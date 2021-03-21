---
title: Включение AAD для Integration Runtime Azure SSIS
description: В этой статье описывается включение аутентификации Azure Active Directory с использованием управляемого удостоверения для службы "Фабрика данных Azure" с целью создания среды выполнения интеграции Azure-SSIS.
ms.service: data-factory
ms.devlang: powershell
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.custom: seo-lt-2019
ms.date: 07/09/2020
ms.openlocfilehash: cd3f590e1869b28f0ac08ce98da32a98160e4e86
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100392737"
---
# <a name="enable-azure-active-directory-authentication-for-azure-ssis-integration-runtime"></a>Включение аутентификации Azure Active Directory для среды выполнения интеграции Azure-SSIS

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этой статье показано, как включить проверку подлинности Azure Active Directory (Azure AD) с помощью управляемого удостоверения для фабрики данных Azure (ADF) и использовать его вместо обычных методов проверки подлинности (например, проверки подлинности SQL) в следующих целях:

- Создайте Azure-SSIS Integration Runtime (IR), который будет в свою очередь подготавливать базу данных каталога SSIS (SSISDB) в базе данных SQL или Управляемый экземпляр SQL от вашего имени.

- Подключайтесь к различным ресурсам Azure при запуске пакетов служб SSIS на Azure-SSIS IR.

Дополнительные сведения об управляемом удостоверении для ADF см. в статье [управляемое удостоверение для фабрики данных](./data-factory-service-identity.md).

> [!NOTE]
>
> - В этом сценарии проверка подлинности Azure AD с управляемым удостоверением для ADF используется только в операциях создания и последующих запусков среды выполнения интеграции SSIS, которые в свою очередь будут подключаться к SSISDB и подключиться к нему. Для выполнения пакетов служб SSIS IR будет по-прежнему подключаться к SSISDB с помощью проверки подлинности SQL с полностью управляемыми учетными записями, созданными во время подготовки SSISDB.
> - Если вы уже создали среду SSIS с помощью проверки подлинности SQL, вы не сможете перенастроить ее для использования проверки подлинности Azure AD через PowerShell, но это можно сделать с помощью приложения портал Azure/ADF. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="enable-azure-ad-on-azure-sql-database"></a>Включение Azure AD в службе "База данных SQL Azure"

База данных SQL поддерживает создание базы данных с пользователем Azure AD. Для начала нужно создать группу Azure AD с управляемым удостоверением, членом которой станет ADF. Затем необходимо задать пользователя Azure AD в качестве администратора Active Directory для базы данных SQL, а затем подключиться к нему на SQL Server Management Studio (SSMS) с помощью этого пользователя. И в конечном итоге потребуется создать автономного пользователя, представляющего группу Azure AD, чтобы позволить Azure-SSIS IR использовать управляемое удостоверение для ADF с целью создания SSISDB от вашего имени.

### <a name="create-an-azure-ad-group-with-the-managed-identity-for-your-adf-as-a-member"></a>Создание группы Azure AD с управляемым удостоверением, членом которой станет ADF

Можно использовать существующую группу Azure AD или создать новую с помощью Azure AD PowerShell.

1.  Установите модуль [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2) .

2.  Войдите с помощью `Connect-AzureAD` , выполните следующий командлет, чтобы создать группу и сохранить ее в переменной:

    ```powershell
    $Group = New-AzureADGroup -DisplayName "SSISIrGroup" `
                              -MailEnabled $false `
                              -SecurityEnabled $true `
                              -MailNickName "NotSet"
    ```

    Результат будет выглядеть как в следующем примере и содержать значение переменной.

    ```powershell
    $Group

    ObjectId DisplayName Description
    -------- ----------- -----------
    6de75f3c-8b2f-4bf4-b9f8-78cc60a18050 SSISIrGroup
    ```

3.  Добавьте в группу управляемое удостоверение службы "Фабрика данных Azure". Чтобы получить идентификатор управляемого объекта удостоверения (например, 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc, но не использовать идентификатор приложения управляемого удостоверения для этой цели), см. статью [Управление удостоверениями для фабрики данных](./data-factory-service-identity.md) .

    ```powershell
    Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc
    ```

    Членство в группе можно также проверить позже.

    ```powershell
    Get-AzureAdGroupMember -ObjectId $Group.ObjectId
    ```

### <a name="configure-azure-ad-authentication-for-sql-database"></a>Настройка проверки подлинности Azure AD для базы данных SQL

Вы можете [настроить аутентификацию Azure AD и управлять ею с помощью SQL](../azure-sql/database/authentication-aad-configure.md) , выполнив следующие действия:

1.  В портал Azure выберите **все службы**  ->  **серверы SQL Server** в области навигации слева.

2.  Выберите сервер в базе данных SQL для настройки аутентификации Azure AD.

3.  В разделе колонки **Параметры** выберите **Администратор Active Directory**.

4.  На панели команд щелкните **Задать администратора**.

5.  Выберите учетную запись пользователя Azure AD, которую необходимо сделать администратором сервера, а затем нажмите **кнопку Выбрать.**

6.  На панели команд нажмите кнопку **Сохранить**.

### <a name="create-a-contained-user-in-sql-database-representing-the-azure-ad-group"></a>Создание автономного пользователя в базе данных SQL, представляющей группу Azure AD

На следующем шаге потребуется [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).

1. Запустите SSMS.

2. В диалоговом окне **соединение с сервером** введите имя сервера в поле **имя сервера** .

3. В поле **authentication (проверка подлинности** ) выберите **Active Directory-Universal с поддержкой MFA** (также можно использовать другие два типа проверки подлинности Active Directory, см. раздел [Настройка и Управление аутентификацией Azure AD с помощью SQL](../azure-sql/database/authentication-aad-configure.md)).

4. В поле **имя пользователя** введите имя учетной записи Azure AD, заданной в качестве администратора сервера, например testuser@xxxonline.com .

5. Выберите **подключить** и завершите процесс входа.

6. В **обозревателе объектов** разверните папку **Базы данных** -> **Системные базы данных**.

7. Щелкните правой кнопкой мыши базу данных **master** и выберите **создать запрос**.

8. В окне запроса введите следующую команду T-SQL и нажмите кнопку **выполнить** на панели инструментов.

   ```sql
   CREATE USER [SSISIrGroup] FROM EXTERNAL PROVIDER
   ```

   В результате выполнения команды должен быть создан автономный пользователь для представления группы.

9. Сотрите данные в окне запроса, введите следующую команду T-SQL и щелкните **Выполнить** на панели инструментов:

   ```sql
   ALTER ROLE dbmanager ADD MEMBER [SSISIrGroup]
   ```

   После выполнения этой команды автономный пользователь сможет создать базу данных (SSISDB).

10. Если база данных SSISDB создана с использованием проверки подлинности SQL и вы хотите переключиться на использование проверки подлинности Azure AD для Azure-SSIS IR доступа к ней, сначала убедитесь в том, что инструкции по предоставлению **разрешения на** успешное завершение. Затем щелкните правой кнопкой мыши базу данных **SSISDB** и выберите **создать запрос**.

11. В окне запроса введите следующую команду T-SQL и нажмите кнопку **выполнить** на панели инструментов.

    ```sql
    CREATE USER [SSISIrGroup] FROM EXTERNAL PROVIDER
    ```

    В результате выполнения команды должен быть создан автономный пользователь для представления группы.

12. Сотрите данные в окне запроса, введите следующую команду T-SQL и щелкните **Выполнить** на панели инструментов:

    ```sql
    ALTER ROLE db_owner ADD MEMBER [SSISIrGroup]
    ```

    В результате выполнения команды автономный пользователь получает права на доступ к SSISDB.

## <a name="enable-azure-ad-on-sql-managed-instance"></a>Включение Azure AD на Управляемый экземпляр SQL

SQL Управляемый экземпляр поддерживает создание базы данных с управляемым удостоверением для ADF напрямую. Вам не нужно присоединить управляемое удостоверение для ADF-файла к группе Azure AD или создать автономного пользователя, представляющего эту группу в Управляемый экземпляр SQL.

### <a name="configure-azure-ad-authentication-for-azure-sql-managed-instance"></a>Настройка аутентификации Azure AD для Управляемый экземпляр Azure SQL

Выполните действия, описанные в разделе [предоставление администратора Azure Active Directory для SQL управляемый экземпляр](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

### <a name="add-the-managed-identity-for-your-adf-as-a-user-in-sql-managed-instance"></a>Добавление управляемого удостоверения для ADF в качестве пользователя в SQL Управляемый экземпляр

На следующем шаге потребуется [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).

1.  Запустите SSMS.

2.  Подключитесь к SQL Управляемый экземпляр SQL Server с помощью учетной записи администратора, которая является **sysadmin**. Это временное ограничение, которое будет удалено после того, как участники сервера Azure AD (имена входа) для Azure SQL Управляемый экземпляр станут общедоступными. При попытке использовать учетную запись администратора Azure AD для создания имени входа вы увидите следующую ошибку: сообщение 15247, уровень 16, состояние 1, пользователь строки 1 не имеет разрешения на выполнение этого действия.

3.  В **обозревателе объектов** разверните папку **Базы данных** -> **Системные базы данных**.

4.  Щелкните правой кнопкой мыши базу данных **master** и выберите **создать запрос**.

5.  В окне запроса выполните следующий скрипт T-SQL, чтобы добавить управляемое удостоверение для ADF-файла в качестве пользователя.

    ```sql
    CREATE LOGIN [{your ADF name}] FROM EXTERNAL PROVIDER
    ALTER SERVER ROLE [dbcreator] ADD MEMBER [{your ADF name}]
    ALTER SERVER ROLE [securityadmin] ADD MEMBER [{your ADF name}]
    ```
    
    После выполнения этой команды управляемое удостоверение для ADF получит возможность для создания базы данных (SSISDB).

6.  Если база данных SSISDB создана с использованием проверки подлинности SQL и вы хотите переключиться на использование проверки подлинности Azure AD для Azure-SSIS IR доступа к ней, сначала убедитесь в том, что инструкции по предоставлению **разрешения на** успешное завершение. Затем щелкните правой кнопкой мыши базу данных **SSISDB** и выберите **создать запрос**.

7.  В окне запроса введите следующую команду T-SQL и нажмите кнопку **выполнить** на панели инструментов.

    ```sql
    CREATE USER [{your ADF name}] FOR LOGIN [{your ADF name}] WITH DEFAULT_SCHEMA = dbo
    ALTER ROLE db_owner ADD MEMBER [{your ADF name}]
    ```

    После выполнения этой команды управляемое удостоверение для ADF получит возможность для доступа к SSISDB.

## <a name="provision-azure-ssis-ir-in-azure-portaladf-app"></a>Подготовка Azure SSIS IR на портале Azure или в приложении ADF

При подготовке Azure-SSIS IR на портале Azure или в приложении ADF на странице **Параметры SQL** установите флажок **Использовать аутентификацию AAD с управляемым удостоверением для ADF**. На следующем снимке экрана показаны параметры для IR с базой данных SQL, размещенной в SSISDB. Для IR с SQL Управляемый экземпляр, на котором размещается SSISDB, **уровень служб базы данных каталога** и **Разрешить службам Azure доступ к** параметрам неприменимы, в то время как другие параметры одинаковы.

Дополнительные сведения о создании Azure-SSIS IR см. в статье [Создание среды выполнения интеграции Azure-SSIS в Фабрике данных Azure](./create-azure-ssis-integration-runtime.md).

![Параметры среды выполнения интеграции Azure-SSIS](media/enable-aad-authentication-azure-ssis-ir/enable-aad-authentication.png)

## <a name="provision-azure-ssis-ir-with-powershell"></a>Подготовка Azure-SSIS IR с помощью PowerShell

Чтобы подготовить к работе среду выполнения интеграции Azure-SSIS с помощью PowerShell, сделайте следующее:

1.  Установите модуль [Azure PowerShell](https://github.com/Azure/azure-powershell/releases/tag/v5.5.0-March2018) .

2.  Не задавайте в скрипте параметр `CatalogAdminCredential`. Пример:

    ```powershell
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                               -DataFactoryName $DataFactoryName `
                                               -Name $AzureSSISName `
                                               -Description $AzureSSISDescription `
                                               -Type Managed `
                                               -Location $AzureSSISLocation `
                                               -NodeSize $AzureSSISNodeSize `
                                               -NodeCount $AzureSSISNodeNumber `
                                               -Edition $AzureSSISEdition `
                                               -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
                                               -CatalogServerEndpoint $SSISDBServerEndpoint `
                                               -CatalogPricingTier $SSISDBPricingTier

    Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                                 -DataFactoryName $DataFactoryName `
                                                 -Name $AzureSSISName
    ```

## <a name="run-ssis-packages-with-managed-identity-authentication"></a>Запуск пакетов SSIS с проверкой подлинности с помощью управляемого удостоверения

При запуске пакетов служб SSIS на Azure-SSIS IR можно использовать проверку подлинности управляемого удостоверения для подключения к различным ресурсам Azure. Сейчас мы уже поддерживали аутентификацию управляемого удостоверения в следующих диспетчерах соединений.

- [Диспетчер соединений OLE DB](/sql/integration-services/connection-manager/ole-db-connection-manager#managed-identities-for-azure-resources-authentication)

- [Диспетчер подключений ADO.NET](/sql/integration-services/connection-manager/ado-net-connection-manager#managed-identities-for-azure-resources-authentication)

- [Диспетчер подключений службы хранилища Azure](/sql/integration-services/connection-manager/azure-storage-connection-manager#managed-identities-for-azure-resources-authentication)

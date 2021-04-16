---
title: Управление доступом к учетной записи хранения в бессерверном пуле SQL
description: В этой статье приведены сведения о том, как бессерверный пул SQL обращается к службе хранилища Azure и как можно управлять доступом к хранилищу для бессерверного пула SQL в Azure Synapse Analytics.
services: synapse-analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 06/11/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 726395e9f004130699dab061cfa752a2e516c834
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106552960"
---
# <a name="control-storage-account-access-for-serverless-sql-pool-in-azure-synapse-analytics"></a>Управление доступом к учетной записи хранения в бессерверном пуле SQL в Azure Synapse Analytics

Бессерверный пул SQL позволяет создавать запросы для чтения файлов непосредственно из службы хранилища Azure. Разрешения на доступ к файлам в службе хранилища Azure контролируются на двух уровнях.
- **Уровень хранилища** — пользователь должен иметь разрешение на доступ к базовым файлам хранилища. Администратор хранилища должен разрешить субъекту Azure AD чтение и запись файлов или создать ключ SAS, который будет использоваться для доступа к хранилищу.
- **Уровень службы SQL** — пользователь должен иметь разрешение на чтение данных с использованием [внешней таблицы](develop-tables-external-tables.md) или на выполнение функции `OPENROWSET`. Дополнительные сведения о необходимых разрешениях см. в [этом разделе](develop-storage-files-overview.md#permissions).

В этой статье описываются типы учетных данных, которые вы можете использовать, и способ поиска учетных данных для пользователей SQL и Azure AD.

## <a name="supported-storage-authorization-types"></a>Поддерживаемые типы авторизации в службе хранилища

Пользователь, выполнивший вход в бессерверный пул SQL, должен иметь права на доступ и отправку запросов к файлам в службе хранилища Azure, если эти файлы не являются общедоступными. Вы можете использовать три типа авторизации для доступа к хранилищу, отличному от общедоступного: [идентификатор пользователя](?tabs=user-identity), [подписанный URL-адрес](?tabs=shared-access-signature) и [управляемое удостоверение](?tabs=managed-identity).

> [!NOTE]
> По умолчанию при создании рабочей области действует **сквозная аутентификация Azure AD**.

### <a name="user-identity"></a>[Удостоверение пользователя](#tab/user-identity)

**Удостоверение пользователя** используется для типа авторизации "сквозная аутентификация Azure", при которой удостоверение пользователя Azure AD, выполнившего вход в бессерверный пул SQL, применяется для авторизации доступа к данным. Перед обращением к данным администратор службы хранилища Azure должен предоставить разрешения соответствующему пользователю Azure AD. Как указано в таблице ниже, этот тип не поддерживается для типа пользователя SQL.

> [!IMPORTANT]
> Маркер проверки подлинности AAD может кэшироваться клиентскими приложениями. Например, PowerBI кэширует токен AAD и повторно использует тот же маркер в течение часа. Длительные запросы которых запущены могут завершиться ошибкой, если срок действия маркера истекает в середине выполнения запроса. При возникновении ошибок запросов, вызванных маркером доступа AAD, срок действия которого истекает в середине запроса, рассмотрите возможность переключения на [управляемое удостоверение](develop-storage-files-storage-access-control.md?tabs=managed-identity#supported-storage-authorization-types) или [подписание общего доступа](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#supported-storage-authorization-types).

Чтобы использовать удостоверение для доступа к данным, необходимо иметь роль владельца, участника или читателя данных для BLOB-объекта в хранилище. В качестве альтернативы можно указать детализированные правила ACL для доступа к файлам и папкам. Даже если вы являетесь владельцем учетной записи хранения, вам придется добавить себя в одну из ролей для данных BLOB-объекта хранилища.
Дополнительные сведения см. в статье [Access control in Azure Data Lake Storage Gen2](../../storage/blobs/data-lake-storage-access-control.md) (Контроль доступа в Azure Data Lake Storage 2-го поколения).


### <a name="shared-access-signature"></a>[Подписанный URL-адрес](#tab/shared-access-signature)

**Подписанный URL-адрес (SAS)** предоставляет делегированный доступ к ресурсам в учетной записи хранения. С помощью SAS можно предоставить клиентам доступ к ресурсам в учетной записи хранения, не передавая им ключи учетной записи. SAS обеспечивает детальный контроль над типом доступа, который вы предоставляете клиентам с конкретным SAS, включая период действия, предоставленные разрешения, допустимый диапазон IP-адресов и допустимый протокол (HTTPS или HTTP).

Маркер SAS можно получить на портале Azure, последовательно открыв пункты **Учетная запись хранения —> Подписанный URL-адрес —> Настроить разрешения —> Создать SAS и строку подключения**.

> [!IMPORTANT]
> Созданный маркер SAS содержит вопросительный знак ("?") в начале строки. Чтобы применить этот маркер для бессерверного пула SQL, удалите символ вопросительного знака ("?") при создании учетных данных. Пример:
>
> Маркер SAS: ?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-04-18T20:42:12Z&st=2019-04-18T12:42:12Z&spr=https&sig=lQHczNvrk1KoYLCpFdSsMANd0ef9BrIPBNJ3VYEIq78%3D

Чтобы разрешить доступ с помощью маркера SAS, необходимо создать учетные данные уровня базы данных или уровня сервера. 


> [!IMPORTANT]
> У вас нет доступа к частным учетным записям хранения с маркером SAS. Попробуйте переключиться на [управляемое удостоверение](develop-storage-files-storage-access-control.md?tabs=managed-identity#supported-storage-authorization-types) или сквозную проверку подлинности [Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity#supported-storage-authorization-types) для доступа к защищенному хранилищу.

### <a name="managed-identity"></a>[Управляемое удостоверение](#tab/managed-identity)

**Управляемое удостоверение** иногда обозначается как MSI. Это функция Azure Active Directory (Azure AD), которая предоставляет службы Azure для бессерверного пула SQL. Также она развертывает автоматически управляемое удостоверение в Azure AD. Это удостоверение можно использовать для авторизации запроса на доступ к данным в службе хранилища Azure.

Перед обращением к данным администратор службы хранилища Azure должен предоставить управляемому удостоверению разрешения на доступ к данным. Предоставление разрешений управляемому удостоверению выполняется точно так же, как любому другому пользователю Azure AD.

### <a name="anonymous-access"></a>[Анонимный доступ](#tab/public-access)

Вы можете использовать файлы, размещенные в общедоступных расположениях в учетных записях хранения Azure, которые [допускают анонимный доступ](../../storage/blobs/anonymous-read-access-configure.md).

---

### <a name="supported-authorization-types-for-databases-users"></a>Поддерживаемые типы авторизации для пользователей баз данных

В таблице ниже перечислены доступные типы авторизации.

| Тип авторизации                    | *Пользователь SQL*    | *Пользователь Azure AD*     |
| ------------------------------------- | ------------- | -----------    |
| [Удостоверение пользователя](?tabs=user-identity#supported-storage-authorization-types)       | Не поддерживается | Поддерживается      |
| [SAS](?tabs=shared-access-signature#supported-storage-authorization-types)       | Поддерживается     | Поддерживается      |
| [Управляемое удостоверение](?tabs=managed-identity#supported-storage-authorization-types) | Не поддерживается | Поддерживается      |

### <a name="supported-storages-and-authorization-types"></a>Поддерживаемые типы авторизации и хранилища

Вы можете использовать следующие сочетания типов авторизации и хранилища Azure.

| Тип авторизации  | Хранилище BLOB-объектов   | ADLS 1-го поколения        | ADLS 2-го поколения     |
| ------------------- | ------------   | --------------   | -----------   |
| [SAS](?tabs=shared-access-signature#supported-storage-authorization-types)    | Поддерживается\*      | Не поддерживается   | Поддерживается\*     |
| [Управляемое удостоверение](?tabs=managed-identity#supported-storage-authorization-types) | Поддерживается      | Поддерживается        | Поддерживается     |
| [Удостоверение пользователя](?tabs=user-identity#supported-storage-authorization-types)    | Поддерживается\*      | Поддерживается\*        | Поддерживается\*     |

\* Маркер SAS и удостоверение Azure AD можно использовать для доступа к хранилищу, не защищенному брандмауэром.


### <a name="querying-firewall-protected-storage"></a>Запросы к хранилищу, защищенному брандмауэром

Для обращения к хранилищу, защищенному брандмауэром, можно использовать **удостоверение пользователя** или **управляемое удостоверение**.

> [!NOTE]
> Функция брандмауэра в службе хранилища предоставляется в общедоступной предварительной версии и доступна во всех регионах общедоступного облака. 

#### <a name="user-identity"></a>Удостоверение пользователя

Чтобы обратиться к хранилищу, защищенному брандмауэром, с помощью удостоверения пользователя, можно использовать модуль Az.Storage в PowerShell.
#### <a name="configuration-via-azure-portal"></a>Настройка через портал Azure

1. Войдите в свою учетную запись хранения на портале Azure.
1. В разделе "Параметры" выберите "Сеть".
1. В разделе "Экземпляры ресурсов" добавьте исключение для рабочей области Synapse.
1. Выберите Microsoft.Synapse/workspaces в качестве типа ресурса.
1. Выберите имя рабочей области в качестве имени экземпляра.
1. Нажмите кнопку «Сохранить».

#### <a name="configuration-via-powershell"></a>Настройка через PowerShell

Выполните описанные ниже действия, чтобы настроить брандмауэр учетной записи хранения и добавить исключение для рабочей области Synapse.

1. Откройте или [установите PowerShell](/powershell/scripting/install/installing-powershell-core-on-windows).
2. Установите модуль Az.Storage 3.4.0 и Az.Synapse 0.7.0: 
    ```powershell
    Install-Module -Name Az.Storage -RequiredVersion 3.4.0
    Install-Module -Name Az.Synapse -RequiredVersion 0.7.0
    ```
    > [!IMPORTANT]
    > Обязательно используйте **версию 3.4.0**. Чтобы проверить версию Az.Storage, выполните следующую команду:  
    > ```powershell 
    > Get-Module -ListAvailable -Name  Az.Storage | select Version
    > ```
    > 

3. Подключитесь к своему клиенту Azure: 
    ```powershell
    Connect-AzAccount
    ```
4. Определите переменные в PowerShell: 
    - Имя группы ресурсов можно найти на портале Azure в общих сведениях об учетной записи хранения.
    - Имя учетной записи — это имя учетной записи хранения, защищенной правилами брандмауэра.
    - Идентификатор клиента можно найти на портале Azure в разделе информации о клиенте Azure Active Directory.
    - Имя рабочей области — это имя рабочей области Synapse.

    ```powershell
        $resourceGroupName = "<resource group name>"
        $accountName = "<storage account name>"
        $tenantId = "<tenant id>"
        $workspaceName = "<synapse workspace name>"
        
        $workspace = Get-AzSynapseWorkspace -Name $workspaceName
        $resourceId = $workspace.Id
        $index = $resourceId.IndexOf("/resourceGroups/", 0)
        # Replace G with g - /resourceGroups/ to /resourcegroups/
        $resourceId = $resourceId.Substring(0,$index) + "/resourcegroups/" + $resourceId.Substring($index + "/resourceGroups/".Length)
        $resourceId
    ```
    > [!IMPORTANT]
    > Убедитесь, что идентификатор ресурса соответствует этому шаблону в выводе переменной resourceId.
    >
    > Значение **resourcegroups** обязательно нужно указывать в нижнем регистре.
    > Пример идентификатора ресурса: 
    > ```
    > /subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Synapse/workspaces/{name-of-workspace}
    > ```
    > 
5. Добавьте правило сети хранения: 
    ```powershell
        Add-AzStorageAccountNetworkRule -ResourceGroupName $resourceGroupName -Name $accountName -TenantId $tenantId -ResourceId $resourceId
    ```
6. Убедитесь, что правило было применено в учетной записи хранения: 
    ```powershell
        $rule = Get-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName
        $rule.ResourceAccessRules | ForEach-Object { 
            if ($_.ResourceId -cmatch "\/subscriptions\/(\w\-*)+\/resourcegroups\/(.)+") { 
                Write-Host "Storage account network rule is successfully configured." -ForegroundColor Green
                $rule.ResourceAccessRules
            } else {
                Write-Host "Storage account network rule is not configured correctly. Remove this rule and follow the steps in detail." -ForegroundColor Red
                $rule.ResourceAccessRules
            }
        }
    ```

#### <a name="managed-identity"></a>Управляемое удостоверение
Необходимо [разрешить доверенные службы Майкрософт](../../storage/common/storage-network-security.md#trusted-microsoft-services) и явно [назначить роли Azure](../../storage/common/storage-auth-aad.md#assign-azure-roles-for-access-rights) [назначаемому системой управляемому удостоверению](../../active-directory/managed-identities-azure-resources/overview.md) для этого экземпляра ресурса. В таком случае область доступа для этого экземпляра соответствует роли Azure, назначенной управляемому удостоверению.

## <a name="credentials"></a>Учетные данные

Чтобы выполнить запрос по файлу, который размещен в службе хранилища Azure, конечная точка бессерверного пула SQL должна иметь учетные данные с информацией об аутентификации. Используются два типа учетных данных.
- Объект CREDENTIAL уровня сервера используется для нерегламентированных запросов, выполняемых с помощью функции `OPENROWSET`. Имя этого объекта должно соответствовать URL-адресу хранилища.
- DATABASE SCOPED CREDENTIAL используется для внешних таблиц. Внешняя таблица ссылается на `DATA SOURCE` с учетными данными, которые должны использоваться для доступа к хранилищу.

Чтобы разрешить пользователю создавать или удалять учетные данные, администратор может предоставить пользователю разрешение GRANT/DENY ALTER ANY CREDENTIAL.

```sql
GRANT ALTER ANY CREDENTIAL TO [user_name];
```

Пользователи базы данных, которые обращаются к внешнему хранилищу, должны иметь разрешение на использование учетных данных.

### <a name="grant-permissions-to-use-credential"></a>Предоставление разрешений на использование учетных данных

Чтобы использовать учетные данные, пользователь должен иметь разрешение `REFERENCES` для этих учетных данных. Чтобы предоставить разрешение `REFERENCES` для учетных данных хранилища (storage_credential) конкретному пользователю (specific_user), выполните такую команду:

```sql
GRANT REFERENCES ON CREDENTIAL::[storage_credential] TO [specific_user];
```

## <a name="server-scoped-credential"></a>Учетные данные уровня сервера

Учетные данные уровня сервера используются при вызове функции `OPENROWSET` из процесса входа в SQL без `DATA_SOURCE` для чтения файлов в определенной учетной записи хранения. Имя учетных данных уровня сервера **должно** соответствовать базовому URL-адресу службы хранилища Azure (и при необходимости сопровождаться именем контейнера). Для добавления учетных данных выполните инструкцию [CREATE CREDENTIAL](/sql/t-sql/statements/create-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true). Необходимо указать аргумент CREDENTIAL NAME.

> [!NOTE]
> Аргумент `FOR CRYPTOGRAPHIC PROVIDER` не поддерживается.

Имя объекта CREDENTIAL уровня сервера должно содержать полный путь к учетной записи хранения и к контейнеру (необязательно) в следующем формате: `<prefix>://<storage_account_path>[/<container_name>]` Описание путей к учетной записи хранения приведено в следующей таблице.

| Внешний источник данных       | Prefix | Путь к учетной записи хранения                                |
| -------------------------- | ------ | --------------------------------------------------- |
| хранилище BLOB-объектов Azure         | HTTPS  | <учетная_запись_хранилища>.blob.core.windows.net             |
| Хранилище Azure Data Lake Storage 1-го поколения | HTTPS  | <учетная_запись_хранилища>.azuredatalakestore.net/webhdfs/v1 |
| Azure Data Lake Storage 2-го поколения | HTTPS  | <учетная_запись_хранения>.dfs.core.windows.net              |

Учетные данные уровня сервера позволяют получить доступ к службе хранилища Azure через следующие типы проверки подлинности:

### <a name="user-identity"></a>[Удостоверение пользователя](#tab/user-identity)

Пользователи Azure AD могут получить доступ к любому файлу в хранилище Azure, если им назначена роль `Storage Blob Data Owner`, `Storage Blob Data Contributor` или `Storage Blob Data Reader`. Пользователям Azure AD не нужно создавать учетные данные для доступа к хранилищу. 

Пользователи SQL не могут использовать аутентификацию Azure AD для доступа к хранилищу.

### <a name="shared-access-signature"></a>[Подписанный URL-адрес](#tab/shared-access-signature)

Следующий скрипт создает учетные данные уровня сервера, которые могут использоваться функцией `OPENROWSET` для доступа к любому файлу в службе хранилища Azure через токен SAS. Создайте такие учетные данные, чтобы разрешить субъекту SQL, который выполняет функцию `OPENROWSET`, считывать файлы, защищенные ключом SAS в службе хранилища Azure, которая соответствует URL-адресу в имени учетных данных.

Замените <*mystorageaccountname*> реальным именем учетной записи хранения, а <*mystorageaccountcontainername*> — именем контейнера:

```sql
CREATE CREDENTIAL [https://<mystorageaccountname>.dfs.core.windows.net/<mystorageaccountcontainername>]
WITH IDENTITY='SHARED ACCESS SIGNATURE'
, SECRET = 'sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-04-18T20:42:12Z&st=2019-04-18T12:42:12Z&spr=https&sig=lQHczNvrk1KoYLCpFdSsMANd0ef9BrIPBNJ3VYEIq78%3D';
GO
```

При необходимости вы можете использовать только базовый URL-адрес учетной записи хранения без имени контейнера.

### <a name="managed-identity"></a>[Управляемое удостоверение](#tab/managed-identity)

Следующий скрипт создает учетные данные уровня сервера, которые могут использоваться функцией `OPENROWSET` для доступа к любому файлу в службе хранилища Azure через управляемое рабочей областью удостоверение.

```sql
CREATE CREDENTIAL [https://<storage_account>.dfs.core.windows.net/<container>]
WITH IDENTITY='Managed Identity'
```

При необходимости вы можете использовать только базовый URL-адрес учетной записи хранения без имени контейнера.

### <a name="public-access"></a>[Открытый доступ](#tab/public-access)

Учетные данные уровня базы данных не обязательны для обращения к общедоступным файлам. Создайте [источник данных без учетных данных уровня базы данных](develop-tables-external-tables.md?tabs=sql-ondemand#example-for-create-external-data-source), чтобы обращаться к общедоступным файлам в службе хранилища Azure.

---

## <a name="database-scoped-credential"></a>Учетные данные уровня базы данных

Учетные данные уровня базы данных используются, когда какой-либо субъект вызывает функцию `OPENROWSET` с `DATA_SOURCE` или выбирает данные из [внешней таблицы](develop-tables-external-tables.md) без использования общедоступных файлов. Учетные данные уровня базы данных не должны совпадать с именем учетной записи хранения, так как они будут явно указаны в объекте DATA SOURCE, который определяет расположение хранилища.

Такие учетные данные позволяют получить доступ к службе хранилища Azure через следующие типы проверки подлинности:

### <a name="azure-ad-identity"></a>[Удостоверение Azure AD](#tab/user-identity)

Пользователи Azure AD могут получить доступ к любому файлу в хранилище Azure, если им назначена роль не ниже `Storage Blob Data Owner`, `Storage Blob Data Contributor` или `Storage Blob Data Reader`. Пользователям Azure AD не нужно создавать учетные данные для доступа к хранилищу.

```sql
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>'
)
```

Пользователи SQL не могут использовать аутентификацию Azure AD для доступа к хранилищу.

### <a name="shared-access-signature"></a>[Подписанный URL-адрес](#tab/shared-access-signature)

Следующий скрипт создает учетные данные, которые используются для доступа к файлам в хранилище с помощью маркера SAS, указанного в этих учетных данных. Скрипт создаст образец внешнего источника данных, который использует этот маркер SAS для доступа к хранилищу.

```sql
-- Optional: Create MASTER KEY if not exists in database:
-- CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<Very Strong Password>'
GO
CREATE DATABASE SCOPED CREDENTIAL [SasToken]
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
     SECRET = 'sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-04-18T20:42:12Z&st=2019-04-18T12:42:12Z&spr=https&sig=lQHczNvrk1KoYLCpFdSsMANd0ef9BrIPBNJ3VYEIq78%3D';
GO
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>',
          CREDENTIAL = SasToken
)
```

### <a name="managed-identity"></a>[Управляемое удостоверение](#tab/managed-identity)

Следующий скрипт создает учетные данные уровня базы данных, которые можно использовать для олицетворения текущего пользователя Azure AD в управляемом удостоверении службы. Скрипт создаст образец внешнего источника данных, который использует удостоверение рабочей области для доступа к хранилищу.

```sql
-- Optional: Create MASTER KEY if not exists in database:
-- CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<Very Strong Password>
CREATE DATABASE SCOPED CREDENTIAL SynapseIdentity
WITH IDENTITY = 'Managed Identity';
GO
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>',
          CREDENTIAL = SynapseIdentity
)
```

Учетные данные уровня базы данных не должны совпадать с именем учетной записи хранения, так как они будут явно указаны в объекте DATA SOURCE, который определяет расположение хранилища.

### <a name="public-access"></a>[Открытый доступ](#tab/public-access)

Учетные данные уровня базы данных не обязательны для обращения к общедоступным файлам. Создайте [источник данных без учетных данных уровня базы данных](develop-tables-external-tables.md?tabs=sql-ondemand#example-for-create-external-data-source), чтобы обращаться к общедоступным файлам в службе хранилища Azure.

```sql
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.blob.core.windows.net/<container>/<path>'
)
```
---

Учетные данные уровня базы данных используются во внешних источниках данных для указания метода проверки подлинности, который будет использоваться для доступа к этому хранилищу.

```sql
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>',
          CREDENTIAL = <name of database scoped credential> 
)
```

## <a name="examples"></a>Примеры

### <a name="access-a-publicly-available-data-source"></a>**Обращение к общедоступному источнику данных**

Используйте следующий скрипт, чтобы создать таблицу, которая обращается к общедоступному источнику данных.

```sql
CREATE EXTERNAL FILE FORMAT [SynapseParquetFormat]
       WITH ( FORMAT_TYPE = PARQUET)
GO
CREATE EXTERNAL DATA SOURCE publicData
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<public_container>/<path>' )
GO

CREATE EXTERNAL TABLE dbo.userPublicData ( [id] int, [first_name] varchar(8000), [last_name] varchar(8000) )
WITH ( LOCATION = 'parquet/user-data/*.parquet',
       DATA_SOURCE = [publicData],
       FILE_FORMAT = [SynapseParquetFormat] )
```

Пользователь базы данных может считывать содержимое файлов из такого источника данных, используя внешнюю таблицу или функцию [OPENROWSET](develop-openrowset.md), которая ссылается на источник данных:

```sql
SELECT TOP 10 * FROM dbo.userPublicData;
GO
SELECT TOP 10 * FROM OPENROWSET(BULK 'parquet/user-data/*.parquet',
                                DATA_SOURCE = 'mysample',
                                FORMAT='PARQUET') as rows;
GO
```

### <a name="access-a-data-source-using-credentials"></a>**Доступ к источнику данных с помощью учетных данных**

Измените следующий скрипт, чтобы создать внешнюю таблицу, которая обращается к службе хранилища Azure с маркером SAS, удостоверением пользователя Azure AD или управляемым удостоверением рабочей области.

```sql
-- Create master key in databases with some password (one-off per database)
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Y*********0'
GO

-- Create databases scoped credential that use Managed Identity or SAS token. User needs to create only database-scoped credentials that should be used to access data source:

CREATE DATABASE SCOPED CREDENTIAL WorkspaceIdentity
WITH IDENTITY = 'Managed Identity'
GO
CREATE DATABASE SCOPED CREDENTIAL SasCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 'sv=2019-10-1********ZVsTOL0ltEGhf54N8KhDCRfLRI%3D'

-- Create data source that one of the credentials above, external file format, and external tables that reference this data source and file format:

CREATE EXTERNAL FILE FORMAT [SynapseParquetFormat] WITH ( FORMAT_TYPE = PARQUET)
GO

CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>'
-- Uncomment one of these options depending on authentication method that you want to use to access data source:
--,CREDENTIAL = WorkspaceIdentity 
--,CREDENTIAL = SasCredential 
)

CREATE EXTERNAL TABLE dbo.userData ( [id] int, [first_name] varchar(8000), [last_name] varchar(8000) )
WITH ( LOCATION = 'parquet/user-data/*.parquet',
       DATA_SOURCE = [mysample],
       FILE_FORMAT = [SynapseParquetFormat] );

```

Пользователь базы данных может считывать содержимое файлов из такого источника данных, используя [внешнюю таблицу](develop-tables-external-tables.md) или функцию [OPENROWSET](develop-openrowset.md), которая ссылается на источник данных:

```sql
SELECT TOP 10 * FROM dbo.userdata;
GO
SELECT TOP 10 * FROM OPENROWSET(BULK 'parquet/user-data/*.parquet', DATA_SOURCE = 'mysample', FORMAT='PARQUET') as rows;
GO
```

## <a name="next-steps"></a>Дальнейшие действия

Приведенные ниже статьи помогут узнать, как включать в запрос разные типы папок и файлов, а также создавать и использовать представления.

- [Запрашивание одного CSV-файла](query-single-csv-file.md)
- [Запрашивание папок и нескольких CSV-файлов](query-folders-multiple-csv-files.md)
- [Запрашивание конкретных файлов](query-specific-files.md)
- [Запрашивание файлов Parquet](query-parquet-files.md)
- [Создание и использование представлений](create-use-views.md)
- [Запрашивание JSON-файлов](query-json-files.md)
- [Запрашивание вложенных типов Parquet](query-parquet-nested-types.md)

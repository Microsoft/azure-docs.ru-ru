---
title: Создание общей локальной среды выполнения интеграции с помощью PowerShell
description: Узнайте, как создать общую локальную среду выполнения интеграции в Фабрике данных Azure, обеспечивающую нескольким фабрикам данных доступ к среде выполнения интеграции.
ms.service: data-factory
ms.topic: conceptual
ms.author: abnarain
author: nabhishek
ms.custom: seo-lt-2019
ms.date: 06/10/2020
ms.openlocfilehash: 16feeb124859fa6199303d9d590fa0a286033ef9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100389439"
---
# <a name="create-a-shared-self-hosted-integration-runtime-in-azure-data-factory"></a>Создание общей локальной среды выполнения интеграции в Фабрике данных Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве описывается создание общей локальной среды выполнения интеграции в Фабрике данных Azure. Общую локальную среду выполнения интеграции затем можно использовать в другой фабрике данных.

## <a name="create-a-shared-self-hosted-integration-runtime-in-azure-data-factory"></a>Создание общей локальной среды выполнения интеграции в Фабрике данных Azure

Вы можете повторно использовать существующую инфраструктуру локальной среды IR, которую вы уже настроили в фабрике данных. Это повторное использование позволяет создать связанную локальную среду выполнения интеграции в другой фабрике данных, ссылаясь на существующий общий объект IR, размещенный на собственном сервере.

Чтобы ознакомиться с введением и демонстрацией этой функции, просмотрите следующее 12-минутное видео:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Hybrid-data-movement-across-multiple-Azure-Data-Factories/player]

### <a name="terminology"></a>Терминология

- **Общие IR**— это изначально размещенная локальная среда IR, которая работает в физической инфраструктуре.  
- **Связанный IR**— IR, который ссылается на другой общий IR. Связанный IR является логическим IR и использует инфраструктуру другого общедоступного саморазмещенного ИК-соединения.

## <a name="create-a-shared-self-hosted-ir-using-azure-data-factory-ui"></a>Создание общей локальной среды выполнения интеграции с помощью пользовательского интерфейса Фабрики данных Azure

Чтобы создать общую локальную среду выполнения интеграции с помощью пользовательского интерфейса Фабрики данных Azure, можно выполнить следующие действия:

1. В локальной среде IR, для которой предоставляется общий доступ, выберите **предоставить разрешение для другой фабрики данных** и на странице "Настройка среды выполнения интеграции" выберите фабрику данных, в которой необходимо создать связанный IR.
      
    ![Кнопка предоставления разрешений на вкладке "Общий доступ"](media/create-self-hosted-integration-runtime/grant-permissions-IR-sharing.png)  
    
2. Запишите и скопируйте приведенный выше "идентификатор ресурса" автономного IR-файла для совместного размещения.
         
3. В фабрике данных, которой были предоставлены разрешения, создайте новую локальную среду IR (связанную) и введите идентификатор ресурса.
      
    ![Кнопка для создания локальной среды выполнения интеграции](media/create-self-hosted-integration-runtime/create-linkedir-1.png)
   
    ![Кнопка создания связанной локальной среды выполнения интеграции](media/create-self-hosted-integration-runtime/create-linkedir-2.png) 

    ![Поля для имени и идентификатора ресурса](media/create-self-hosted-integration-runtime/create-linkedir-3.png)

## <a name="create-a-shared-self-hosted-ir-using-azure-powershell"></a>Создание общей локальной среды выполнения интеграции с помощью Azure PowerShell

Чтобы создать общую локальную среду выполнения интеграции с помощью Azure PowerShell, можно выполнить следующие действия: 
1. Создали фабрику данных. 
1. Создайте локальную среду выполнения интеграции.
1. Предоставьте общий доступ к локальной среде выполнения интеграции другим фабриками данных.
1. Создайте связанную среду выполнения интеграции.
1. Отмените общий доступ.

### <a name="prerequisites"></a>Предварительные требования 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Подписка Azure**. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу. 

- **Azure PowerShell**. Выполните инструкции из руководства [Install Azure PowerShell on Windows with PowerShellGet](/powershell/azure/install-az-ps) (Установка Azure PowerShell в Windows c помощью PowerShellGet). С помощью PowerShell выполните скрипт, чтобы создать локальную среду выполнения интеграции, которую можно использовать совместно с другими фабриками данных. 

> [!NOTE]  
> Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на странице [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory).

### <a name="create-a-data-factory"></a>Создание фабрики данных

1. Откройте интегрированную среду сценариев (ISE) Windows PowerShell.

1. Создайте переменные. Скопируйте и вставьте следующий скрипт. Замените переменные, такие как **SubscriptionName** и **ResourceGroupName**, фактическими значениями: 

    ```powershell
    # If input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$". 
    $SubscriptionName = "[Azure subscription name]" 
    $ResourceGroupName = "[Azure resource group name]" 
    $DataFactoryLocation = "EastUS" 

    # Shared Self-hosted integration runtime information. This is a Data Factory compute resource for running any activities 
    # Data factory name. Must be globally unique 
    $SharedDataFactoryName = "[Shared Data factory name]" 
    $SharedIntegrationRuntimeName = "[Shared Integration Runtime Name]" 
    $SharedIntegrationRuntimeDescription = "[Description for Shared Integration Runtime]"

    # Linked integration runtime information. This is a Data Factory compute resource for running any activities
    # Data factory name. Must be globally unique
    $LinkedDataFactoryName = "[Linked Data factory name]"
    $LinkedIntegrationRuntimeName = "[Linked Integration Runtime Name]"
    $LinkedIntegrationRuntimeDescription = "[Description for Linked Integration Runtime]"
    ```

1. Войдите в систему и выберите подписку. Добавьте следующий код в скрипт, чтобы войти и выбрать подписку Azure.

    ```powershell
    Connect-AzAccount
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

1. Создайте группу ресурсов и фабрику данных.

    > [!NOTE]  
    > Это необязательный шаг. Если у вас уже есть фабрика данных, пропустите этот шаг. 

    Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа. В следующем примере создается группа ресурсов с именем `myResourceGroup` в расположении WestEurope. 

    ```powershell
    New-AzResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
    ```

    Чтобы создать фабрику данных, выполните следующие команды: 

    ```powershell
    Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
                             -Location $DataFactoryLocation `
                             -Name $SharedDataFactoryName
    ```

### <a name="create-a-self-hosted-integration-runtime"></a>Создание локальной среды выполнения интеграции

> [!NOTE]  
> Это необязательный шаг. Если у вас уже есть локальная среда выполнения интеграции, которую вы хотите использовать совместно с другими фабриками данных, пропустите этот шаг.

Выполните следующую команду, чтобы создать локальную среду выполнения интеграции.

```powershell
$SharedIR = Set-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName `
    -Type SelfHosted `
    -Description $SharedIntegrationRuntimeDescription
```

#### <a name="get-the-integration-runtime-authentication-key-and-register-a-node"></a>Получение ключа проверки подлинности для среды выполнения интеграции и регистрация узла

Выполните следующую команду, чтобы получить ключ проверки подлинности для локальной среды выполнения интеграции.

```powershell
Get-AzDataFactoryV2IntegrationRuntimeKey `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName
```

Ответ содержит ключ для локальной среды выполнения интеграции. Используйте этот ключ при регистрации узла среды выполнения интеграции.

#### <a name="install-and-register-the-self-hosted-integration-runtime"></a>Установка и регистрация локальной среды выполнения интеграции

1. Скачайте установщик локальной среды выполнения интеграции из [Integration Runtime Фабрики данных Azure](https://aka.ms/dmg).

2. Запустите установщик, чтобы установить локальную среду выполнения интеграции на локальном компьютере.

3. Зарегистрируйте новую локальную среду выполнения интеграции с помощью ключа проверки подлинности, полученного на предыдущем этапе.

### <a name="share-the-self-hosted-integration-runtime-with-another-data-factory"></a>Предоставление общего доступа к локальной среде выполнения интеграции другой фабрике данных

#### <a name="create-another-data-factory"></a>Создание другой фабрикой данных

> [!NOTE]  
> Это необязательный шаг. Если у вас уже есть фабрика данных, которой вы хотите предоставить общий доступ, пропустите этот шаг. Однако для добавления или удаления назначений ролей в другой фабрике данных необходимы `Microsoft.Authorization/roleAssignments/write` `Microsoft.Authorization/roleAssignments/delete` разрешения и, например [администратор доступа пользователей](../role-based-access-control/built-in-roles.md#user-access-administrator) или [владелец](../role-based-access-control/built-in-roles.md#owner).

```powershell
$factory = Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $LinkedDataFactoryName
```
#### <a name="grant-permission"></a>Предоставление разрешения

Предоставьте разрешение фабрике данных, которой требуется доступ к созданной и зарегистрированной локальной среде выполнения интеграции.

> [!IMPORTANT]  
> Не пропускайте этот шаг!

```powershell
New-AzRoleAssignment `
    -ObjectId $factory.Identity.PrincipalId ` #MSI of the Data Factory with which it needs to be shared
    -RoleDefinitionName 'Contributor' `
    -Scope $SharedIR.Id
```

### <a name="create-a-linked-self-hosted-integration-runtime"></a>Создание связанной локальной среды выполнения интеграции

Выполните следующую команду, чтобы создать связанную локальную среду выполнения интеграции.

```powershell
Set-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $LinkedDataFactoryName `
    -Name $LinkedIntegrationRuntimeName `
    -Type SelfHosted `
    -SharedIntegrationRuntimeResourceId $SharedIR.Id `
    -Description $LinkedIntegrationRuntimeDescription
```

Теперь можно использовать эту связанную среду выполнения интеграции в любой связанной службе. Связанная среда выполнения интеграции использует общую среду выполнения интеграции для выполнения действий.

### <a name="revoke-integration-runtime-sharing-from-a-data-factory"></a>Отмена для фабрики данных общего доступа к среде выполнения интеграции

Выполните следующую команду, чтобы отменить для фабрики данных доступ к общей среде выполнения интеграции:

```powershell
Remove-AzRoleAssignment `
    -ObjectId $factory.Identity.PrincipalId `
    -RoleDefinitionName 'Contributor' `
    -Scope $SharedIR.Id
```

Выполните следующую команду для общей среды выполнения интеграции, чтобы удалить имеющуюся связанную среду выполнения интеграции:

```powershell
Remove-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName `
    -Links `
    -LinkedDataFactoryName $LinkedDataFactoryName
```

### <a name="monitoring"></a>Мониторинг

#### <a name="shared-ir"></a>Общие IR

![Варианты поиска общей среды выполнения интеграции](media/create-self-hosted-integration-runtime/Contoso-shared-IR.png)

![Мониторинг общей среды выполнения интеграции](media/create-self-hosted-integration-runtime/contoso-shared-ir-monitoring.png)

#### <a name="linked-ir"></a>Связанный IR

![Варианты поиска связанной среды выполнения интеграции](media/create-self-hosted-integration-runtime/Contoso-linked-ir.png)

![Мониторинг связанной среды выполнения интеграции](media/create-self-hosted-integration-runtime/Contoso-linked-ir-monitoring.png)


### <a name="known-limitations-of-self-hosted-ir-sharing"></a>Известные ограничения общего доступа к локальной среде IR

* Фабрика данных, в которой создается связанный IR, должна иметь [управляемое удостоверение](../active-directory/managed-identities-azure-resources/overview.md). По умолчанию фабрики данных, созданные в портал Azure или командлетах PowerShell, имеют неявно созданное управляемое удостоверение. Но при создании фабрики данных с помощью шаблона Azure Resource Manager или пакета SDK необходимо явно задать свойство **Identity** . Этот параметр гарантирует, что диспетчер ресурсов создаст фабрику данных, содержащую управляемое удостоверение.

* Пакет SDK для .NET фабрики данных, поддерживающий эту функцию, должен иметь версию 1.1.0 или более позднюю.

* Чтобы предоставить разрешение, требуется роль владельца или Наследуемая роль владельца в фабрике данных, где существует общий IR.

* Функция общего доступа работает только для фабрик данных в одном клиенте Azure AD.

* Для [гостевых пользователей](../active-directory/governance/manage-guest-access-with-access-reviews.md)Azure AD функция поиска в пользовательском интерфейсе, в которой перечислены все фабрики данных с помощью ключевого слова поиска, не работает. Но при условии, что гостевой пользователь является владельцем фабрики данных, вы можете совместно использовать IR без функции поиска. Для управляемого удостоверения фабрики данных, которая должна предоставлять общий доступ к IR, введите это управляемое удостоверение в поле **назначить разрешение** и выберите **Добавить** в пользовательском интерфейсе фабрики данных.

  > [!NOTE]
  > Эта функция доступна только в фабрике данных версии 2.


### <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [основными понятиями среды выполнения интеграции в Фабрике данных Azure](./concepts-integration-runtime.md).

- Узнайте, как [создать локальную среду выполнения интеграции на портале Azure](./create-self-hosted-integration-runtime.md).
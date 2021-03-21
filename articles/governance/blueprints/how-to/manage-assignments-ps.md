---
title: Управление назначениями с помощью PowerShell
description: Узнайте, как управлять назначениями схем с помощью официального модуля PowerShell для чертежей Azure, AZ. чертеж.
ms.date: 01/27/2021
ms.topic: how-to
ms.openlocfilehash: d60fb887e07b4697b8e86a4e2fd74a735ac0bb58
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98919382"
---
# <a name="how-to-manage-assignments-with-powershell"></a>Управление назначениями с помощью PowerShell

Назначение схемы можно управлять с помощью модуля **AZ. чертеж** Azure PowerShell. Модуль поддерживает выборку, создание, обновление и удаление назначений. Модуль также может получать сведения о существующих определениях схем. В этой статье описывается, как установить модуль и приступить к его использованию.

## <a name="add-the-azblueprint-module"></a>Добавьте модуль AZ. чертеж

Чтобы разрешить Azure PowerShell управлять назначениями схем, необходимо добавить модуль. Этот модуль можно использовать с локально установленной средой PowerShell, с [Azure Cloud Shell](https://shell.azure.com) либо с [образом Docker Azure PowerShell](https://hub.docker.com/r/azuresdk/azure-powershell/).

### <a name="base-requirements"></a>Основные требования

Для модуля "проекты Azure" требуется следующее программное обеспечение:

- Azure PowerShell 1.5.0 или более поздней версии. Если она еще не установлена, выполните [эти инструкции](/powershell/azure/install-az-ps).
- Модуль PowerShellGet 2.0.1 или более поздней версии. Если он еще не установлен или не обновлен, выполните [эти инструкции](/powershell/scripting/gallery/installing-psget).

### <a name="install-the-module"></a>Установка модуля

Модуль "планы Azure" для PowerShell — **AZ. чертеж**.

1. Запустите следующие команды из командной строки PowerShell **с правами администратора**.

   ```azurepowershell-interactive
   # Install the Azure Blueprints module from PowerShell Gallery
   Install-Module -Name Az.Blueprint
   ```

   > [!NOTE]
   > Если **AZ. Accounts** уже установлено, для `-AllowClobber` принудительной установки может потребоваться использовать.

1. Убедитесь, что модуль импортирован и является правильной версией (0.2.6):

   ```azurepowershell-interactive
   # Get a list of commands for the imported Az.Blueprint module
   Get-Command -Module 'Az.Blueprint' -CommandType 'Cmdlet'
   ```

## <a name="get-blueprint-definitions"></a>Получение определений схем

Первый шаг для работы с назначением часто получает ссылку на определение схемы.
`Get-AzBlueprint`Командлет возвращает одно или несколько определений схем. Командлет может получать определения схем из группы управления с помощью `-ManagementGroupId {mgId}` или подписки с `-SubscriptionId {subId}` . Параметр **Name** получает определение схемы, но должен использоваться с **ManagementGroupId** или **SubscriptionId**. **Версию** можно использовать с **именем** , чтобы более явно определить, какое определение схемы возвращается. Вместо **Version**, параметр `-LatestPublished` извлекает последнюю опубликованную версию.

В следующем примере используется `Get-AzBlueprint` для получения всех версий определения схемы с именем "101-чертежи-Definition-Subscription" из конкретной подписки, представленной в виде `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get all versions of the blueprint definition in the specified subscription
$blueprints = Get-AzBlueprint -SubscriptionId '{subId}' -Name '101-blueprints-definition-subscription'

# Display the blueprint definition object
$blueprints
```

Пример выходных данных для определения схемы с несколькими версиями выглядит следующим образом:

```output
Name                 : 101-blueprints-definition-subscription
Id                   : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprints/101
                       -blueprints-definition-subscription
DefinitionLocationId : {subId}
Versions             : {1.0, 1.1}
TimeCreated          : 2019-02-25
TargetScope          : Subscription
Parameters           : {storageAccount_storageAccountType, storageAccount_location,
                       allowedlocations_listOfAllowedLocations, [Usergrouporapplicationname]:Reader_RoleAssignmentName}
ResourceGroups       : ResourceGroup
```

[Параметры](../concepts/parameters.md#blueprint-parameters) схемы для определения схемы можно расширить, чтобы предоставить дополнительные сведения.

```azurepowershell-interactive
$blueprints.Parameters
```

```output
Key                                                    Value
---                                                    -----
storageAccount_storageAccountType                      Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
storageAccount_location                                Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
allowedlocations_listOfAllowedLocations                Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
[Usergrouporapplicationname]:Reader_RoleAssignmentName Microsoft.Azure.Commands.Blueprint.Models.PSParameterDefinition
```

## <a name="get-blueprint-assignments"></a>Получение назначений схем

Если назначение схемы уже существует, можно получить ссылку на него с помощью `Get-AzBlueprintAssignment` командлета. Командлет принимает **SubscriptionId** и **Name** как необязательные параметры. Если значение **SubscriptionId** не указано, используется текущий контекст подписки.

В следующем примере используется `Get-AzBlueprintAssignment` для получения одного назначения схемы с именем "назначение-блокировка-ресурс-группы" из конкретной подписки, представленной в виде `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the blueprint assignment in the specified subscription
$blueprintAssignment = Get-AzBlueprintAssignment -SubscriptionId '{subId}' -Name 'Assignment-lock-resource-groups'

# Display the blueprint assignment object
$blueprintAssignment
```

Пример выходных данных для назначения схемы выглядит следующим образом:

```output
Name              : Assignment-lock-resource-groups
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssignme
                    nts/Assignment-lock-resource-groups
Scope             : /subscriptions/{subId}
LastModified      : 2019-02-19
LockMode          : AllResourcesReadOnly
ProvisioningState : Succeeded
Parameters        :
ResourceGroups    : ResourceGroup
```

## <a name="create-blueprint-assignments"></a>Создание назначений схем

Если назначение схемы еще не существует, его можно создать с помощью `New-AzBlueprintAssignment` командлета. Этот командлет использует следующие параметры:

- **Имя** [обязательный]
  - Задает имя назначения схемы
  - Должно быть уникальным и не существовать в **SubscriptionId**
- **Чертеж** [обязательный]
  - Указывает определение схемы для назначения
  - Используйте `Get-AzBlueprint` для получения объекта Reference
- **Расположение** [обязательный]
  - Задает регион для назначаемого системой объекта развертывания управляемого удостоверения и подписки, который будет создан в
- **Подписка** (необязательно)
  - Указывает подписку, в которой развернуто назначение
  - Если не указано, по умолчанию используется текущий контекст подписки
- **Блокировка** (необязательно)
  - Определяет [блокировку ресурсов](../concepts/resource-locking.md) схемы для использования в развернутых ресурсах
  - Поддерживаемые параметры: _None_, _аллресаурцесреадонли_, _аллресаурцесдонотделете_
  - Если не указано, по умолчанию используется значение _None_
- **Системассигнедидентити** (необязательно)
  - Выберите, чтобы создать управляемое системой удостоверение для назначения и развертывания ресурсов.
  - Значение по умолчанию для набора параметров "Identity"
  - Не может использоваться с **усерассигнедидентити**
- **Усерассигнедидентити** (необязательно)
  - Указывает назначаемое пользователем управляемое удостоверение, которое будет использоваться для назначения и развертывания ресурсов.
  - Часть набора параметров "Identity"
  - Не может использоваться с **системассигнедидентити**
- **Параметр** (необязательно)
  - [Хэш-таблица](/powershell/module/microsoft.powershell.core/about/about_hash_tables) пар "ключ-значение" для установки [динамических параметров](../concepts/parameters.md#dynamic-parameters) в назначении схемы
  - Значение по умолчанию для динамического параметра — это **DefaultValue** в определении
  - Если параметр не указан и не имеет **DefaultValue**, параметр не является обязательным.

    > [!NOTE]
    > **Параметр** не поддерживает секурестрингс.

- **Ресаурцеграуппараметер** (необязательно)
  - [Хэш-таблица](/powershell/module/microsoft.powershell.core/about/about_hash_tables) артефактов группы ресурсов
  - Каждый заполнитель артефакта группы ресурсов имеет пары "ключ-значение" для динамического задания **имени** и **расположения** в этом артефакте группы ресурсов.
  - Если параметр группы ресурсов не указан и не имеет **DefaultValue**, параметр группы ресурсов не является обязательным.
- **Ассигнментфиле** (необязательно)
  - Путь к представлению JSON-файла для назначения схемы
  - Этот параметр является частью набора параметров PowerShell, который включает в себя только **имя**, **проект** и идентификатор **подписки**, а также общие параметры.

### <a name="example-1-provide-parameters"></a>Пример 1. Указание параметров

В следующем примере создается новое назначение версии "1,1" определения схемы "My-чертеж", полученного с помощью `Get-AzBlueprint` , устанавливается расположение управляемого удостоверения и объекта назначения в "westus2", блокируются ресурсы с помощью _аллресаурцесреадонли_ и задаются хэш-таблицы для **параметров** и **ресаурцеграуппараметер** в конкретной подписке, представленной в виде `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'

# Create the hash table for Parameters
$bpParameters = @{storageAccount_storageAccountType='Standard_GRS'}

# Create the hash table for ResourceGroupParameters
# ResourceGroup is the resource group artifact placeholder name
$bpRGParameters = @{ResourceGroup=@{name='storage_rg';location='westus2'}}

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Location 'westus2' -Lock AllResourcesReadOnly `
    -Parameter $bpParameters -ResourceGroupParameter $bpRGParameters
```

Пример выходных данных для создания назначения схемы выглядит следующим образом:

```output
Name              : my-blueprint-assignment
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssi
                    gnments/my-blueprint-assignment
Scope             : /subscriptions/{subId}
LastModified      : 2019-03-13
LockMode          : AllResourcesReadOnly
ProvisioningState : Creating
Parameters        : {storageAccount_storageAccountType}
ResourceGroups    : ResourceGroup
```

### <a name="example-2-use-a-json-assignment-definition-file"></a>Пример 2. Использование файла определения назначения JSON

В следующем примере создается почти такое же назначение, как и в [примере 1](#example-1-provide-parameters). Вместо передачи параметров в командлет, в примере показано использование файла определения назначения JSON и параметра **ассигнментфиле** . Кроме того, свойство **ексклудедпринЦипалс** настраивается как часть **блокировок**. Отсутствует параметр PowerShell для **ексклудедпринЦипалс** , и свойство можно настроить только с помощью файла определения назначения JSON.

```json
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "westus2",
  "properties": {
    "description": "Assignment of the 101-blueprint-definition-subscription",
    "blueprintId": "/subscriptions/{subId}/providers/Microsoft.Blueprint/blueprints/101-blueprints-definition-subscription",
    "locks": {
      "mode": "AllResourcesReadOnly",
      "excludedPrincipals": [
          "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
          "38833b56-194d-420b-90ce-cff578296714"
      ]
    },
    "parameters": {
      "storageAccount_storageAccountType": {
        "value": "Standard_GRS"
      }
    },
    "resourceGroups": {
      "ResourceGroup": {
        "name": "storage_rg",
        "location": "westus2"
      }
    }
  }
}
```

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -SubscriptionId '{subId}' `
    -AssignmentFile '.\assignment.json'
```

Пример файла определения назначения JSON для управляемого удостоверения, назначаемого пользователем, см. в тексте запроса в [примере: назначение с управляемым пользователем удостоверением](/rest/api/blueprints/assignments/createorupdate#examples) для REST API.

## <a name="update-blueprint-assignments"></a>Обновление назначений схем

Иногда необходимо обновить назначение схемы, которое уже было создано. `Set-AzBlueprintAssignment`Командлет обрабатывает это действие. Командлет принимает большинство тех же параметров, что и `New-AzBlueprintAssignment` командлет, что позволяет обновлять все, что было установлено в назначении. Исключениями являются _имя_, _проект_ и идентификатор _подписки_. Обновляются только указанные значения.

Сведения о том, что происходит при обновлении назначения схемы, см. в разделе [правила обновления назначений](./update-existing-assignments.md#rules-for-updating-assignments).

- **Имя** [обязательный]
  - Задает имя обновляемого назначения схемы
  - Используется для нахождение назначения для обновления, а не для изменения назначения
- **Чертеж** [обязательный]
  - Задает определение схемы для назначения схемы
  - Используйте `Get-AzBlueprint` для получения объекта Reference
  - Используется для нахождение назначения для обновления, а не для изменения назначения
- **Расположение** (необязательно)
  - Задает регион для назначаемого системой объекта развертывания управляемого удостоверения и подписки, который будет создан в
- **Подписка** (необязательно)
  - Указывает подписку, в которой развернуто назначение
  - Если не указано, по умолчанию используется текущий контекст подписки
  - Используется для нахождение назначения для обновления, а не для изменения назначения
- **Блокировка** (необязательно)
  - Определяет [блокировку ресурсов](../concepts/resource-locking.md) схемы для использования в развернутых ресурсах
  - Поддерживаемые параметры: _None_, _аллресаурцесреадонли_, _аллресаурцесдонотделете_
- **Системассигнедидентити** (необязательно)
  - Выберите, чтобы создать управляемое системой удостоверение для назначения и развертывания ресурсов.
  - Значение по умолчанию для набора параметров "Identity"
  - Не может использоваться с **усерассигнедидентити**
- **Усерассигнедидентити** (необязательно)
  - Указывает назначаемое пользователем управляемое удостоверение, которое будет использоваться для назначения и развертывания ресурсов.
  - Часть набора параметров "Identity"
  - Не может использоваться с **системассигнедидентити**
- **Параметр** (необязательно)
  - [Хэш-таблица](/powershell/module/microsoft.powershell.core/about/about_hash_tables) пар "ключ-значение" для установки [динамических параметров](../concepts/parameters.md#dynamic-parameters) в назначении схемы
  - Значение по умолчанию для динамического параметра — это **DefaultValue** в определении
  - Если параметр не указан и не имеет **DefaultValue**, параметр не является обязательным.

    > [!NOTE]
    > **Параметр** не поддерживает секурестрингс.

- **Ресаурцеграуппараметер** (необязательно)
  - [Хэш-таблица](/powershell/module/microsoft.powershell.core/about/about_hash_tables) артефактов группы ресурсов
  - Каждый заполнитель артефакта группы ресурсов имеет пары "ключ-значение" для динамического задания **имени** и **расположения** в этом артефакте группы ресурсов.
  - Если параметр группы ресурсов не указан и не имеет **DefaultValue**, параметр группы ресурсов не является обязательным.

В следующем примере обновляется назначение версии "1,1" определения схемы "My-чертеж", полученное с `Get-AzBlueprint` помощью изменения режима блокировки:

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'

# Update the existing blueprint assignment
$bpAssignment = Set-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Lock AllResourcesDoNotDelete
```

Пример выходных данных для создания назначения схемы выглядит следующим образом:

```output
Name              : my-blueprint-assignment
Id                : /subscriptions/{subId}/providers/Microsoft.Blueprint/blueprintAssi
                    gnments/my-blueprint-assignment
Scope             : /subscriptions/{subId}
LastModified      : 2019-03-13
LockMode          : AllResourcesDoNotDelete
ProvisioningState : Updating
Parameters        : {storageAccount_storageAccountType}
ResourceGroups    : ResourceGroup
```

## <a name="remove-blueprint-assignments"></a>Удалить назначения чертежей

При удалении назначения схемы `Remove-AzBlueprintAssignment` это действие обрабатывается командлетом. Командлет принимает либо **имя** , либо **InputObject** , чтобы указать назначение схемы для удаления. Идентификатор **SubscriptionId** является _обязательным_ и должен быть указан во всех случаях.

В следующем примере выполняется выборка существующего назначения схемы с помощью `Get-AzBlueprintAssignment` , а затем она удаляется из конкретной подписки, представленной следующим образом `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the blueprint assignment in the specified subscription
$blueprintAssignment = Get-AzBlueprintAssignment -Name 'Assignment-lock-resource-groups'

# Remove the existing blueprint assignment
Remove-AzBlueprintAssignment -InputObject $blueprintAssignment -SubscriptionId '{subId}'
```

## <a name="code-example"></a>Пример кода

Выполнив все шаги вместе, в следующем примере получается определение схемы, затем создается, обновляется и удаляется назначение схемы в конкретной подписке, представленной в виде `{subId}` :

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

#region GetBlueprint
# Get version '1.1' of the blueprint definition in the specified subscription
$bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'my-blueprint' -Version '1.1'
#endregion

#region CreateAssignment
# Create the hash table for Parameters
$bpParameters = @{storageAccount_storageAccountType='Standard_GRS'}

# Create the hash table for ResourceGroupParameters
# ResourceGroup is the resource group artifact placeholder name
$bpRGParameters = @{ResourceGroup=@{name='storage_rg';location='westus2'}}

# Create the new blueprint assignment
$bpAssignment = New-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Location 'westus2' -Lock AllResourcesReadOnly `
    -Parameter $bpParameters -ResourceGroupParameter $bpRGParameters
#endregion CreateAssignment

# Wait for the blueprint assignment to finish deployment prior to the next steps

#region UpdateAssignment
# Update the existing blueprint assignment
$bpAssignment = Set-AzBlueprintAssignment -Name 'my-blueprint-assignment' -Blueprint $bpDefinition `
    -SubscriptionId '{subId}' -Lock AllResourcesDoNotDelete
#endregion UpdateAssignment

# Wait for the blueprint assignment to finish deployment prior to the next steps

#region RemoveAssignment
# Remove the existing blueprint assignment
Remove-AzBlueprintAssignment -InputObject $bpAssignment -SubscriptionId '{subId}'
#endregion
```

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь со сведениями о [жизненном цикле схем](../concepts/lifecycle.md).
- Узнайте, как использовать [статические и динамические параметры](../concepts/parameters.md).
- Научитесь настраивать [последовательность схемы](../concepts/sequencing-order.md).
- Узнайте, как применять [блокировку ресурсов схемы](../concepts/resource-locking.md).
- Устраняйте проблемы, возникающие во время назначения схемы, с помощью [общих инструкций по устранению неполадок](../troubleshoot/general.md).
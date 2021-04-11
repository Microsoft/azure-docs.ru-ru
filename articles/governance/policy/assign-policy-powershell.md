---
title: Краткое руководство. Создание назначения политики с помощью PowerShell
description: В этом кратком руководстве с помощью Azure PowerShell вы создадите назначение в службе "Политика Azure", позволяющее определить ресурсы, которые не соответствуют требованиям.
ms.date: 03/31/2021
ms.topic: quickstart
ms.openlocfilehash: 1043bd53fa2c494d34b0007a851b95afd937d991
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106091418"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-azure-powershell"></a>Краткое руководство. Создание назначения политики для идентификации ресурсов, не соответствующих требованиям, с помощью Azure PowerShell

Чтобы понять, соответствуют ли ресурсы требованиям в Azure, прежде всего нужно определить их состояние. В рамках этого краткого руководства вы создадите назначение политики для выявления виртуальных машин, которые не используют управляемые диски. После выполнения задач в руководстве вы сможете выявлять _несоответствующие_ виртуальные машины.

Модуль Azure PowerShell используется для управления ресурсами Azure из командной строки или в скриптах.
В этом руководстве объясняется, как при помощи модуля Azure создавать назначение политики.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Перед началом работы убедитесь, что установлена последняя версия Azure PowerShell. Дополнительные сведения см. в статье [Install the Azure PowerShell module](/powershell/azure/install-az-ps) (Установка модуля Azure PowerShell).

- Зарегистрируйте поставщика ресурсов Insights в Политике Azure с помощью Azure PowerShell. Регистрация поставщика ресурсов необходима для надлежащей работы вашей подписки с этим поставщиком. Чтобы зарегистрировать поставщик ресурсов, необходимо разрешение на регистрацию для поставщика ресурсов. Эта операция включается в роли участника и владельца. Выполните указанную ниже команду для регистрации поставщика ресурсов.

  ```azurepowershell-interactive
  # Register the resource provider if it's not already registered
  Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'
  ```

  Дополнительные сведения о регистрации и просмотре поставщиков ресурсов см. в статье [Поставщики и типы ресурсов](../../azure-resource-manager/management/resource-providers-and-types.md).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-policy-assignment"></a>Создание назначения политики

В рамках этого краткого руководства вы создадите назначение политики для определения _виртуальных машин аудита без управляемых дисков_. Это определение политики указывает на виртуальные машины, которые не используют управляемые диски.

Чтобы создать назначение политики, выполните следующие команды:

```azurepowershell-interactive
# Get a reference to the resource group that is the scope of the assignment
$rg = Get-AzResourceGroup -Name '<resourceGroupName>'

# Get a reference to the built-in policy definition to assign
$definition = Get-AzPolicyDefinition | Where-Object { $_.Properties.DisplayName -eq 'Audit VMs that do not use managed disks' }

# Create the policy assignment with the built-in definition against your resource group
New-AzPolicyAssignment -Name 'audit-vm-manageddisks' -DisplayName 'Audit VMs without managed disks Assignment' -Scope $rg.ResourceId -PolicyDefinition $definition
```

В указанных выше командах используются следующие сведения:

- **Name** — фактическое имя назначения. В этом примере использовалось имя _audit-vm-manageddisks_.
- **DisplayName** — отображаемое имя назначения политики. В этом случае используется определение _аудита виртуальных машин без назначения управляемых дисков_.
- **Definition.** Определение политики, на основе которого вы создаете назначение. В нашем случае это идентификатор определения политики _аудита виртуальных машин, которые не используют управляемые диски_.
- **Scope.** Область определяет, к каким ресурсам или группе ресурсов принудительно применяется назначение политики. Политика может назначаться разным ресурсам: от подписки до групп ресурсов. Обязательно замените значение &lt;scope&gt; именем своей группы ресурсов.

Теперь все готово к обнаружению ресурсов, которые не соответствуют требованиям, что позволит оценить состояние соответствия в среде.

## <a name="identify-non-compliant-resources"></a>Выявление несоответствующих ресурсов

Используйте приведенные ниже сведения для идентификации ресурсов, которые не соответствуют созданному вами назначению политики. Выполните следующие команды:

```azurepowershell-interactive
# Get the resources in your resource group that are non-compliant to the policy assignment
Get-AzPolicyState -ResourceGroupName $rg.ResourceGroupName -PolicyAssignmentName 'audit-vm-manageddisks' -Filter 'IsCompliant eq false'
```

Дополнительные сведения о получении состояния политики см. в описании командлета [Get-AzPolicyState](/powershell/module/az.policyinsights/Get-AzPolicyState).

Результаты должны выглядеть примерно так:

```output
Timestamp                   : 3/9/19 9:21:29 PM
ResourceId                  : /subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmId}
PolicyAssignmentId          : /subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/audit-vm-manageddisks
PolicyDefinitionId          : /providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d
IsCompliant                 : False
SubscriptionId              : {subscriptionId}
ResourceType                : /Microsoft.Compute/virtualMachines
ResourceTags                : tbd
PolicyAssignmentName        : audit-vm-manageddisks
PolicyAssignmentOwner       : tbd
PolicyAssignmentScope       : /subscriptions/{subscriptionId}
PolicyDefinitionName        : 06a78e20-9358-41c9-923c-fb736d382a4d
PolicyDefinitionAction      : audit
PolicyDefinitionCategory    : Compute
ManagementGroupIds          : {managementGroupId}
```

Результаты соответствуют тому, что вы видите на вкладке **Соответствие ресурсов** назначения политики в представлении портала Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить созданное назначение, выполните следующую команду:

```azurepowershell-interactive
# Removes the policy assignment
Remove-AzPolicyAssignment -Name 'audit-vm-manageddisks' -Scope '/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>'
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы назначили определение политики для идентификации ресурсов, не соответствующих требованиям, в среде Azure.

Следующее руководство серии содержит сведения о назначении политик для проверки новых ресурсов на соответствие требованиям:

> [!div class="nextstepaction"]
> [Создание политик и управление ими](./tutorials/create-and-manage.md)
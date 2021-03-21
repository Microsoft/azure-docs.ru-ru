---
title: Использование политики Azure для ограничения установки расширения виртуальной машины (Windows)
description: Служба "Политика Azure" позволяет ограничить развертывание расширений.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: windows
ms.date: 03/23/2018
ms.openlocfilehash: 0587c2af8a90ce362fa6243e9da8a05734f0d8ec
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102559755"
---
# <a name="use-azure-policy-to-restrict-extensions-installation-on-windows-vms"></a>Ограничение установки расширений на виртуальных машинах Windows с помощью службы "Политика Azure"

Если вы хотите предотвратить использование или установку определенных расширений на виртуальных машинах Windows, можно создать определение политики Azure с помощью PowerShell, чтобы ограничить расширения для виртуальных машин в группе ресурсов. 

В этом руководстве используется интерфейс Azure PowerShell в Cloud Shell, который постоянно обновляется до последней версии. 

 

## <a name="create-a-rules-file"></a>Создание файла правил

Чтобы ограничить допустимые для установки расширения, вам потребуется [правило](../../governance/policy/concepts/definition-structure.md#policy-rule) с логикой определения расширений.

В этом примере показано, как можно запретить расширения, опубликованные издателем Microsoft.Compute, создав файл правил в Azure Cloud Shell. Если вы используете PowerShell локально, можете создать локальный файл и заменить путь $home/clouddrive фактическим расположением файла на компьютере.

В [Cloud Shell](https://shell.azure.com/powershell) введите такой текст:

```azurepowershell-interactive
nano $home/clouddrive/rules.json
```

Скопируйте следующий код JSON и вставьте его в файл.

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.Compute/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                "equals": "Microsoft.Compute"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/type",
                "in": "[parameters('notAllowedExtensions')]"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

Закончив, нажмите клавиши **Ctrl+O** и подтвердите сохранение файла клавишей **ВВОД**. Затем нажмите **Ctrl+X**, чтобы закрыть файл и редактор.

## <a name="create-a-parameters-file"></a>Создание файла параметров

Вам понадобится также файл [параметров](../../governance/policy/concepts/definition-structure.md#parameters), который позволяет создать структуру для передачи списка блокируемых расширений. 

В этом примере показано, как создать в Cloud Shell файл параметров для виртуальных машин Windows. Если вы используете PowerShell локально, можете создать локальный файл и заменить путь $home/clouddrive фактическим расположением файла на компьютере.

В [Cloud Shell](https://shell.azure.com/powershell) введите такой текст:

```azurepowershell-interactive
nano $home/clouddrive/parameters.json
```

Скопируйте следующий код JSON и вставьте его в файл.

```json
{
    "notAllowedExtensions": {
        "type": "Array",
        "metadata": {
            "description": "The list of extensions that will be denied.",
            "displayName": "Denied extension"
        }
    }
}
```

Закончив, нажмите клавиши **Ctrl+O** и подтвердите сохранение файла клавишей **ВВОД**. Затем нажмите **Ctrl+X**, чтобы закрыть файл и редактор.

## <a name="create-the-policy"></a>Создание политики

Определение политики — это объект для хранения конфигурации, которую нужно использовать. Для определения политики используются файлы правил и параметров. Создайте определение политики с помощью командлета [New-AzPolicyDefinition](/powershell/module/az.resources/new-azpolicydefinition).

 Правила и параметры этой политики представлены в виде файлов, которые вы создали в формате JSON и сохранили с помощью Cloud Shell.


```azurepowershell-interactive
$definition = New-AzPolicyDefinition `
   -Name "not-allowed-vmextension-windows" `
   -DisplayName "Not allowed VM Extensions" `
   -description "This policy governs which VM extensions that are explicitly denied."   `
   -Policy 'C:\Users\ContainerAdministrator\clouddrive\rules.json' `
   -Parameter 'C:\Users\ContainerAdministrator\clouddrive\parameters.json'
```




## <a name="assign-the-policy"></a>Назначение политики

В этом примере назначается политика для группы ресурсов с помощью командлета [New-AzPolicyAssignment](/powershell/module/az.resources/new-azpolicyassignment). На всех виртуальных машинах, созданных в группе ресурсов **myResourceGroup**, невозможно будет установить агент доступа к виртуальной машине и пользовательские расширения. 

Командлет [Get-AzSubscription | Format-Table](/powershell/module/az.accounts/get-azsubscription) предоставит вам идентификатор подписки, который следует указать вместо образца в примере.

```azurepowershell-interactive
$scope = "/subscriptions/<subscription id>/resourceGroups/myResourceGroup"
$assignment = New-AzPolicyAssignment `
   -Name "not-allowed-vmextension-windows" `
   -Scope $scope `
   -PolicyDefinition $definition `
   -PolicyParameter '{
    "notAllowedExtensions": {
        "value": [
            "VMAccessAgent",
            "CustomScriptExtension"
        ]
    }
}'
$assignment
```

## <a name="test-the-policy"></a>Проверка политики

Чтобы проверить политику, попробуйте применить расширение для доступа к виртуальной машине. Следующая ошибка должна завершиться с сообщением "Set-Азвмакцессекстенсион: Resource" Мивмакцесс "запрещено политикой".

```azurepowershell-interactive
Set-AzVMAccessExtension `
   -ResourceGroupName "myResourceGroup" `
   -VMName "myVM" `
   -Name "myVMAccess" `
   -Location EastUS 
```

Попытка сменить пароль на портале должна завершиться ошибкой с сообщением "Не удалось развернуть шаблон, нарушена политика" .

## <a name="remove-the-assignment"></a>Удаление назначения

```azurepowershell-interactive
Remove-AzPolicyAssignment -Name not-allowed-vmextension-windows -Scope $scope
```

## <a name="remove-the-policy"></a>Удаление политики

```azurepowershell-interactive
Remove-AzPolicyDefinition -Name not-allowed-vmextension-windows
```
    
## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения см. в статье [Что такое служба "Политика Azure"?](../../governance/policy/overview.md)

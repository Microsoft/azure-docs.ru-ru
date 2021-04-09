---
title: Краткое руководство. Создание общего запроса с помощью Azure PowerShell
description: В этом кратком руководстве показано, как создать общий запрос к Resource Graph с помощью Azure PowerShell.
ms.date: 01/11/2021
ms.topic: quickstart
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2b771253b1dea4bd1d2913bf7c48062112019a19
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99981550"
---
# <a name="quickstart-create-a-resource-graph-shared-query-using-azure-powershell"></a>Краткое руководство. Создание общего запроса к Resource Graph с помощью Azure PowerShell

В этой статье показано, как создать общий запрос к Azure Resource Graph с помощью модуля [Az.ResourceGraph](/powershell/module/az.resourcegraph) в PowerShell.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [azure-powershell-requirements-no-header.md](../../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Так как модуль **Az.ResourceGraph** в PowerShell предоставляется в предварительной версии, его нужно установить отдельно с помощью командлета `Install-Module`.

  ```azurepowershell-interactive
  Install-Module -Name Az.ResourceGraph
  ```

- Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите требуемую подписку с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

  ```azurepowershell-interactive
  Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
  ```

## <a name="create-a-resource-graph-shared-query"></a>Создание общего запроса к Resource Graph

Теперь, когда модуль **Az.ResourceGraph** PowerShell добавлен в выбранную среду, создадим общий запрос к Resource Graph. Общий запрос представляет собой объект Azure Resource Manager, которому можно предоставить разрешения на доступ к обозревателю Azure Resource Graph или на выполнение в нем. Этот запрос вычисляет общее количество ресурсов с группировкой по _расположению_.

1. Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup), чтобы сохранить общий запрос к Azure Resource Graph. Эта группа ресурсов имеет имя `resource-graph-queries` и расположение `westus2`.

   ```azurepowershell-interactive
   # Login first with `Connect-AzAccount` if not using Cloud Shell

   # Create the resource group
   New-AzResourceGroup -Name resource-graph-queries -Location westus2
   ```

1. Создайте общий запрос к Azure Resource Graph с помощью модуля **Az.ResourceGraph** PowerShell и командлета [New-AzResourceGraphQuery](/powershell/module/az.resourcegraph/new-azresourcegraphquery):

   ```azurepowershell-interactive
   # Create the Azure Resource Graph shared query
   $Params = @{
     Name = 'Summarize resources by location'
     ResourceGroupName = 'resource-graph-queries'
     Location = 'westus2'
     Description = 'This shared query summarizes resources by location for a pinnable map graphic.'
     Query = 'Resources | summarize count() by location'
   }
   New-AzResourceGraphQuery @Params
   ```

1. Получите список общих запросов в новой группе ресурсов. Командлет [Get-AzResourceGraphQuery](/powershell/module/az.resourcegraph/get-azresourcegraphquery) возвращает массив значений.

   ```azurepowershell-interactive
   # List all the Azure Resource Graph shared queries in a resource group
   Get-AzResourceGraphQuery -ResourceGroupName resource-graph-queries
   ```

1. Чтобы получить в качестве результата только один общий запрос, используйте `Get-AzResourceGraphQuery` с параметром `Name`.

   ```azurepowershell-interactive
   # Show a specific Azure Resource Graph shared query
   Get-AzResourceGraphQuery -ResourceGroupName resource-graph-queries -Name 'Summarize resources by location'
   ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить общий запрос к Resource Graph и группу ресурсов из среды Azure, воспользуйтесь следующими командами:

- [Remove-AzResourceGraphQuery](/powershell/module/az.resourcegraph/remove-azresourcegraphquery)
- [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup)

```azurepowershell-interactive
# Delete the Azure Resource Graph shared query
Remove-AzResourceGraphQuery -ResourceGroupName resource-graph-queries -Name 'Summarize resources by location'

# Remove the resource group
# WARNING: This command deletes ALL resources you've added to this resource group
Remove-AzResourceGroup -Name resource-graph-queries
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать общий запрос к Resource Graph с помощью Azure PowerShell. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).
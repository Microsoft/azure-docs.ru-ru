---
title: Краткое руководство. Первый запрос PowerShell
description: В этом кратком руководстве вы выполняете шаги по включению модуля Resource Graph для Azure PowerShell и выполнению первого запроса.
ms.date: 01/27/2021
ms.topic: quickstart
ms.custom:
- mode-api
ms.openlocfilehash: e5e276e3be80354eeaaeba2821eb9e3242b368ad
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533075"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-azure-powershell"></a>Краткое руководство. Выполните первый запрос графика ресурсов с помощью Azure PowerShell

При использовании Azure Resource Graph прежде всего необходимо убедиться, что модуль для Azure PowerShell установлен. В этом кратком руководстве описывается процесс добавления модуля к установке Azure PowerShell.

В конце этого процесса вы добавите модуль к установке Azure PowerShell на свой выбор и выполните первый запрос Resource Graph.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-module"></a>Добавить модуль "График ресурсов"

Чтобы позволить Azure PowerShell выполнять запрос графических ресурсов Azure, необходимо добавить модуль. Этот модуль можно использовать с локально установленной средой PowerShell, с [Azure Cloud Shell](https://shell.azure.com) либо с [образом Docker PowerShell](https://hub.docker.com/_/microsoft-powershell).

### <a name="base-requirements"></a>Основные требования

Модуль Azure Resource Graph требует следующее программное обеспечение.

- Azure PowerShell 1.0.0 или более поздней версии. Если она еще не установлена, выполните [эти инструкции](/powershell/azure/install-az-ps).

- Модуль PowerShellGet 2.0.1 или более поздней версии. Если он еще не установлен или не обновлен, выполните [эти инструкции](/powershell/scripting/gallery/installing-psget).

### <a name="install-the-module"></a>Установка модуля

Модуль Resource Graph для PowerShell — **Az.ResourceGraph**.

1. Запустите следующие команды из командной строки PowerShell **с правами администратора**.

   ```azurepowershell-interactive
   # Install the Resource Graph module from PowerShell Gallery
   Install-Module -Name Az.ResourceGraph
   ```

1. Убедитесь, что импортирован модуль последней версии (0.7.5).

   ```azurepowershell-interactive
   # Get a list of commands for the imported Az.ResourceGraph module
   Get-Command -Module 'Az.ResourceGraph' -CommandType 'Cmdlet'
   ```

## <a name="run-your-first-resource-graph-query"></a>Выполните первый запрос график ресурсов

Теперь, когда модуль PowerShell для Azure добавлен в выбранную среду, попробуем выполнить простой запрос Resource Graph. Запрос возвращает первые пять ресурсов Azure с указанными значениями **Имя** и **Тип ресурса** для каждого ресурса.

1. Запустите ваш первый запрос графика ресурсов Azure, используя командлет `Search-AzGraph`:

   ```azurepowershell-interactive
   # Login first with Connect-AzAccount if not using Cloud Shell

   # Run Azure Resource Graph query
   Search-AzGraph -Query 'Resources | project name, type | limit 5'
   ```

   > [!NOTE]
   > Так как этот пример запроса не меняет сортировку, например `order by`, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса.

1. Обновите запрос свойством `order by`**Имя**:

   ```azurepowershell-interactive
   # Run Azure Resource Graph query with 'order by'
   Search-AzGraph -Query 'Resources | project name, type | limit 5 | order by name asc'
   ```

   > [!NOTE]
   > Как и с первым запросом, выполнение этого запроса несколько раз может получить различные наборы ресурсов для каждого запроса. Важен порядок команд запроса. В этом примере `order by` следует после `limit`. Эта последовательность команд сначала ограничивает результаты запроса, а затем упорядочивает их.

1. Обновите запрос, чтобы сначала выполнить сортировку (`order by`) по свойству **Имя**, а затем ограничить (`limit`) вывод пятью первыми результатами:

   ```azurepowershell-interactive
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   Search-AzGraph -Query 'Resources | project name, type | order by name asc | limit 5'
   ```

Повторные выполнения последней версии запроса, если ничего не изменяется в самой среде, возвращают стабильные результаты, отсортированные по свойству **Name** и ограниченные пятью первыми результатами.

> [!NOTE]
> Если запрос не возвращает результаты из подписки, к которой у вас уже есть доступ, имейте ввиду, что командлет `Search-AzGraph` использует значения по умолчанию для подписок в контексте по умолчанию. Чтобы просмотреть список идентификаторов подписок, которые входят в контекст по умолчанию, выполните `(Get-AzContext).Account.ExtendedProperties.Subscriptions`. Если требуется выполнить поиск во всех подписках, к которым у вас есть доступ, можно задать PSDefaultParameterValues для командлета `Search-AzGraph`, выполнив команду `$PSDefaultParameterValues=@{"Search-AzGraph:Subscription"= $(Get-AzSubscription).ID}`.
   
## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите удалить модуль "График ресурсов" из среды Azure PowerShell, это можно сделать с помощью следующей команды:

```azurepowershell-interactive
# Remove the Resource Graph module from the current session
Remove-Module -Name 'Az.ResourceGraph'

# Uninstall the Resource Graph module from the environment
Uninstall-Module -Name 'Az.ResourceGraph'
```

> [!NOTE]
> Это не приводит к удалению скачанного ранее файла модуля. Он удаляется только из открытого сеанса PowerShell.

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы добавили модуль Resource Graph в среду Azure PowerShell и выполнили первый запрос. Чтобы узнать больше о языке Resource Graph, перейдите на страницу сведений о языке запросов.

> [!div class="nextstepaction"]
> [Получите более подробную информацию о языке запросов](./concepts/query-language.md).

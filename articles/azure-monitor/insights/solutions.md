---
title: Решения мониторинга в Azure Monitor | Документация Майкрософт
description: Решения мониторинга в Azure Monitor — это коллекция правил логики, визуализации и получения данных, которые предоставляют метрики, связанные с определенной проблемной областью.  Эта статья содержит сведения об установке и использовании решений мониторинга.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/16/2020
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: a63db154d055675b834e2949a330375633a5761d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101728592"
---
# <a name="monitoring-solutions-in-azure-monitor"></a>Решения мониторинга в Azure Monitor

Решения мониторинга в Azure Monitor обеспечивают анализ операций конкретного приложения или службы Azure. Эта статья содержит краткий обзор решений мониторинга в Azure и подробные сведения об их использовании и установке. В Azure Monitor вы можете добавить решения мониторинга для всех используемых приложений и служб. Обычно они предоставляются бесплатно, но сбор данных может повлечь определенную плату за использование ресурсов.

## <a name="use-monitoring-solutions"></a>Использование решений для мониторинга

На странице **Обзор** решений в Azure Monitor отображается плитка для каждого решения, установленного в log Analytics рабочей области. Чтобы открыть эту страницу, перейдите к **Azure Monitor** в [портал Azure](https://ms.portal.azure.com). В меню **Insights (аналитика** ) выберите **Дополнительно** , чтобы открыть **центр Insights**, а затем щелкните **log Analytics рабочие области**.

[![Центр Insights](media/solutions/insights-hub.png)](media/solutions/insights-hub.png#lightbox)


Используйте раскрывающиеся списки в верхней части экрана, чтобы менять рабочую область и диапазон времени, используемые для плиток. Щелкните плитку решения, чтобы открыть его представление с подробным анализом собранных данных.

[![На снимке экрана показано меню портал Azure с выбранными решениями и решениями, отображаемыми в области решения.](media/solutions/overview.png)](media/solutions/overview.png#lightbox)

Решения мониторинга могут содержать несколько типов ресурсов Azure, и все ресурсы в составе решения можно просмотреть точно так же, как любой другой ресурс. Например, любые запросы журнала, включенные в решение, отображаются в разделе **запросы решения** в [обозревателе запросов](../logs/log-analytics-tutorial.md). Эти запросы можно использовать при выполнении нерегламентированного анализа с [запросами журналов](../logs/log-query-overview.md).

## <a name="list-installed-monitoring-solutions"></a>Список установленных решений мониторинга

### <a name="portal"></a>[Портал](#tab/portal)

Используйте следующую процедуру, чтобы вывести список решений мониторинга, которые установлены в вашей подписке.

1. Перейдите на [портал Azure](https://ms.portal.azure.com). Найдите в поиске и выберите пункт **Решения**.
1. Отображается список решений, установленных во всех рабочих областях. После имени каждого решения указано имя рабочей области, в которой оно установлено.
1. Раскрывающиеся списки в верхней части экрана позволяют отфильтровать список по подписке или группе ресурсов.

![Просмотр всех решений](media/solutions/list-solutions-all.png)

Щелкните имя решения, чтобы открыть для него страницу сводки. На этой странице отображаются все представления, включенные в это решение, а также различные параметры самого решения и его рабочей области. Откройте страницу со сводной информацией о решении, используя одну из описанных выше процедур, а затем щелкните в списке решений имя нужного решения.

![Свойства решения](media/solutions/solution-properties.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы получить список решений для мониторинга, установленных в вашей подписке, используйте команду [AZ Monitor log-Analytics Solution List](/cli/azure/ext/log-analytics-solution/monitor/log-analytics/solution#ext-log-analytics-solution-az-monitor-log-analytics-solution-list) .   Перед выполнением `list` команды следуйте предварительным требованиям, приведенным в [статье Установка решения для мониторинга](#install-a-monitoring-solution).

```azurecli
# List all log-analytics solutions in the current subscription.
az monitor log-analytics solution list

# List all log-analytics solutions for a specific subscription
az monitor log-analytics solution list --subscription MySubscription

# List all log-analytics solutions in a resource group
az monitor log-analytics solution list --resource-group MyResourceGroup
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Используйте командлет [Get-азмониторлоганалитикссолутион](/powershell/module/az.monitoringsolutions/get-azmonitorloganalyticssolution) , чтобы получить список решений мониторинга, установленных в вашей подписке. Перед выполнением этих команд следуйте предварительным требованиям, приведенным в [статье Установка решения для мониторинга](#install-a-monitoring-solution).

```azurepowershell-interactive
# List all log-analytics solutions in the current subscription.
Get-AzMonitorLogAnalyticsSolution

# List all log-analytics solutions for a specific subscription
Get-AzMonitorLogAnalyticsSolution -SubscriptionId 00000000-0000-0000-0000-000000000000

# List all log-analytics solutions in a resource group
Get-AzMonitorLogAnalyticsSolution -ResourceGroupName MyResourceGroup
```

* * *

## <a name="install-a-monitoring-solution"></a>Установка решения для мониторинга

### <a name="portal"></a>[Портал](#tab/portal)

В [Azure Marketplace](https://azuremarketplace.microsoft.com) доступны решения мониторинга, предлагаемые корпорацией Майкрософт и ее партнерами. Вы можете выполнять поиск доступных решений и устанавливать их с помощью описанной ниже процедуры. При установке решения необходимо выбрать [рабочую область Log Analytics](../logs/manage-access.md), в которой требуется установить решение, а также расположение для хранения его данных.

1. В [списке решений в вашей подписке](#list-installed-monitoring-solutions) щелкните **Добавить**.
1. Просмотрите решения или выполните поиск. Вы также можете просмотреть решения по [этой ссылке](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/management-tools?page=1&subcategories=management-solutions).
1. Найдите нужное решение для мониторинга и прочитайте его описание.
1. Нажмите кнопку **Создать**, чтобы запустить процесс установки.
1. Когда начнется процесс установки, вам будет предложено указать рабочую область Log Analytics и требуемую конфигурацию для решения.

![Установка решения](media/solutions/install-solution.png)

### <a name="install-a-solution-from-the-community"></a>Установка решения из сообщества

Участники сообщества могут отправлять решения для управления в шаблоны быстрого запуска Azure. Все эти решения вы можете установить сразу или скачать в виде шаблонов для последующей установки.

1. Выполните действия, описанные в разделе [Рабочая область Log Analytics и учетная запись службы автоматизации](#log-analytics-workspace-and-automation-account), чтобы установить связь между рабочей областью и учетной записью.
2. Перейдите к [шаблонам быстрого запуска Azure](https://azure.microsoft.com/documentation/templates/).
3. Найдите интересующее вас решение.
4. Выберите решение из результатов, чтобы просмотреть сведения о нем.
5. Нажмите кнопку **Deploy to Azure** (Развернуть в Azure).
6. Вам предложат указать сведения, например группу ресурсов и расположение, а также значения для любых параметров в решении.
7. Щелкните **Приобрести** для установки решения.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="prepare-your-environment"></a>Подготовка среды

1. Установка Azure CLI

   Необходимо [установить Azure CLI](/cli/azure/install-azure-cli) перед выполнением справочных команд CLI.  При желании можно также использовать Azure Cloud Shell для выполнения действий, описанных в этой статье.  Azure Cloud Shell — это интерактивная оболочка среды, с которой можно работать в браузере.  Запустите Cloud Shell с помощью одного из этих методов:

   - Откройте Cloud Shell, перейдя по ссылке: [https://shell.azure.com](https://shell.azure.com)

   - Нажмите кнопку **Cloud Shell** в строке меню в правом верхнем углу окна [портала Azure](https://portal.azure.com).

1. Войдите.

   Если вы используете локальную установку CLI, выполните вход с помощью команды [AZ login](/cli/azure/reference-index#az-login) .  Выполните аутентификацию, следуя инструкциям в окне терминала.

    ```azurecli
    az login
    ```

1. Установка расширения `log-analytics-solution`

   `log-analytics-solution`Команда является экспериментальным расширением основного Azure CLI. Дополнительные сведения о расширениях см. в статье [Использование расширения с Azure CLI](/cli/azure/azure-cli-extensions-overview?).

   ```azurecli
   az extension add --name log-analytics-solution
   ```

   Ожидается следующее предупреждение.

   ```output
   The installed extension `log-analytics-solution` is experimental and not covered by customer support.  Please use with discretion.
   ```

### <a name="install-a-solution-with-the-azure-cli"></a>Установка решения с Azure CLI

При установке решения необходимо выбрать [рабочую область Log Analytics](../logs/manage-access.md), в которой требуется установить решение, а также расположение для хранения его данных.  С Azure CLI Управление рабочими областями осуществляется с помощью команд ссылки на [рабочую область "az Monitor журнала-Analytics](/cli/azure/monitor/log-analytics/workspace) ".  Выполните действия, описанные в разделе [Рабочая область Log Analytics и учетная запись службы автоматизации](#log-analytics-workspace-and-automation-account), чтобы установить связь между рабочей областью и учетной записью.

Используйте команду [AZ Monitor log-Analytics](/cli/azure/ext/log-analytics-solution/monitor/log-analytics/solution) для установки решения для мониторинга.  Параметры в квадратных скобках являются необязательными.

```azurecli
az monitor log-analytics solution create --name
                                         --plan-product
                                         --plan-publisher
                                         --resource-group
                                         --workspace
                                         [--no-wait]
                                         [--tags]
```

Ниже приведен пример кода, который создает решение аналитики журналов для продукта плана Омсгаллери/Containers.

```azurecli
az monitor log-analytics solution create --resource-group MyResourceGroup \
                                         --name Containers({SolutionName}) \
                                         --tags key=value \
                                         --plan-publisher Microsoft  \
                                         --plan-product "OMSGallery/Containers" \
                                         --workspace "/subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/ \
                                           Microsoft.OperationalInsights/workspaces/{WorkspaceName}"
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

### <a name="prepare-your-environment"></a>Подготовка среды

1. Установка Azure PowerShell

   Перед выполнением команд Azure PowerShell REFERENCES необходимо [установить Azure PowerShell](/powershell/azure/install-az-ps) . При желании можно также использовать Azure Cloud Shell для выполнения действий, описанных в этой статье. Azure Cloud Shell — это интерактивная оболочка среды, с которой можно работать в браузере. Запустите Cloud Shell с помощью одного из этих методов:

   - Откройте Cloud Shell, перейдя по ссылке: [https://shell.azure.com](https://shell.azure.com)

   - Нажмите кнопку **Cloud Shell** в строке меню в правом верхнем углу окна [портала Azure](https://portal.azure.com).

   > [!IMPORTANT]
   > Пока модуль PowerShell **AZ. мониторингсолутионс** находится на этапе предварительной версии, его необходимо установить отдельно с помощью `Install-Module` командлета. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков модуля Az PowerShell и по умолчанию встроен в Azure Cloud Shell.

   ```azurepowershell-interactive
   Install-Module -Name Az.MonitoringSolutions
   ```

1. Войдите.

   Если вы используете локальную установку PowerShell, выполните вход с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) . Выполните действия, показанные в PowerShell, чтобы завершить процесс проверки подлинности.

   ```azurepowershell
   Connect-AzAccount
   ```

### <a name="install-a-solution-with-azure-powershell"></a>Установка решения с Azure PowerShell

При установке решения необходимо выбрать [рабочую область Log Analytics](../logs/manage-access.md), в которой требуется установить решение, а также расположение для хранения его данных. Используя Azure PowerShell, вы управляете рабочими областями с помощью командлетов в модуле PowerShell [AZ. мониторингсолутионс](/powershell/module/az.monitoringsolutions) . Выполните действия, описанные в разделе [Рабочая область Log Analytics и учетная запись службы автоматизации](#log-analytics-workspace-and-automation-account), чтобы установить связь между рабочей областью и учетной записью.

Используйте командлет [New-азмониторлоганалитикссолутион](/powershell/module/az.monitoringsolutions/new-azmonitorloganalyticssolution) , чтобы установить решение для мониторинга. Параметры в квадратных скобках являются необязательными.

```azurepowershell
New-AzMonitorLogAnalyticsSolution -ResourceGroupName <string> -Type <string> -Location <string>
-WorkspaceResourceId <string> [-SubscriptionId <string>] [-Tag <hashtable>]
[-DefaultProfile <psobject>] [-Break] [-HttpPipelineAppend <SendAsyncStep[]>]
[-HttpPipelinePrepend <SendAsyncStep[]>] [-Proxy <uri>] [-ProxyCredential <pscredential>]
[-ProxyUseDefaultCredentials] [-WhatIf] [-Confirm] [<CommonParameters>]
```

В следующем примере создается решение Monitor log Analytics для рабочей области log Analytics.

```azurepowershell-interactive
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName MyResourceGroup -Name WorkspaceName
New-AzMonitorLogAnalyticsSolution -Type Containers -ResourceGroupName MyResourceGroup -Location $workspace.Location -WorkspaceResourceId $workspace.ResourceId
```

* * *

## <a name="log-analytics-workspace-and-automation-account"></a>Рабочая область Log Analytics и учетная запись службы автоматизации

Всем решениям мониторинга требуется [рабочая область Log Analytics](../logs/manage-access.md) для хранения собранных решением данных и размещения его поисков по журналам и представлений. Для некоторых решений дополнительно требуется [учетная запись службы автоматизации](../../automation/automation-security-overview.md), в которой они хранят модули Runbook и связанные ресурсы. Рабочая область и учетная запись должны отвечать следующим требованиям.

* Каждая установка решения может использовать только одну рабочую область Log Analytics и одну учетную запись службы автоматизации. Решение можно установить отдельно в нескольких рабочих областях.
* Если для решения нужна учетная запись службы автоматизации, она должна быть связана с рабочей областью Log Analytics для этого же решения. Рабочая область Log Analytics может быть связана только с одной учетной записью службы автоматизации и наоборот.

При установке решения с помощью Azure Marketplace вам будет предложено ввести рабочую область и учетную запись службы автоматизации. При необходимости связь между ними создается автоматически.

### <a name="verify-the-link-between-a-log-analytics-workspace-and-automation-account"></a>Проверка связи между рабочей областью Log Analytics и учетной записью службы автоматизации

Связь между рабочей областью Log Analytics и учетной записью службы автоматизации можно проверить, выполнив перечисленные ниже действия.

1. На портале Azure выберите учетную запись службы автоматизации.
1. Прокрутите страницу к разделу меню **Связанные ресурсы** и выберите пункт **Связанная рабочая область**.
1. Если **рабочая область** связана с учетной записью службы автоматизации, то на этой странице указывается рабочая область, с которой она связана. Если выбрать указанное имя рабочей области, вы будете перенаправлены на страницу с общими сведениями об этой рабочей области.

## <a name="remove-a-monitoring-solution"></a>Удаление решения для мониторинга

### <a name="portal"></a>[Портал](#tab/portal)

Чтобы удалить установленное решение с помощью портала, выберите его в [списке установленных решений](#list-installed-monitoring-solutions). Щелкните имя решения, чтобы открыть страницу сводки, и выберите **Удалить**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы удалить установленное решение с помощью Azure CLI, используйте команду [AZ Monitor log-Analytics решение Delete](/cli/azure/ext/log-analytics-solution/monitor/log-analytics/solution#ext-log-analytics-solution-az-monitor-log-analytics-solution-delete) .

```azurecli
az monitor log-analytics solution delete --name
                                         --resource-group
                                         [--no-wait]
                                         [--yes]
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Чтобы удалить установленное решение с помощью Azure PowerShell, используйте командлет [Remove-азмониторлоганалитикссолутион](/powershell/module/az.monitoringsolutions/remove-azmonitorloganalyticssolution) .

```azurepowershell-interactive
Remove-AzMonitorLogAnalyticsSolution  -ResourceGroupName MyResourceGroup -Name WorkspaceName
```

* * *

## <a name="next-steps"></a>Дальнейшие действия

* Получите список решений мониторинга от корпорации Майкрософт [здесь](../monitor-reference.md).
* Узнайте из статьи [Анализ данных Log Analytics в Azure Monitor](../logs/log-query-overview.md), как создавать запросы для анализа данных, собранных решением для мониторинга.
* См. все [команды Azure CLI для Azure Monitor](/cli/azure/azure-cli-reference-for-monitor).
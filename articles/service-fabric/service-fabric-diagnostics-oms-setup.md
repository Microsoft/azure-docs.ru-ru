---
title: Настройка мониторинга с помощью журналов Azure Monitor
description: Узнайте, как настроить журналы Azure Monitor для визуализации и анализа событий, чтобы отслеживать кластеры Service Fabric Azure.
ms.topic: conceptual
ms.date: 02/20/2019
ms.openlocfilehash: 79dbc38f9d177bc3decf11673a68ce434f9a83bc
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105625965"
---
# <a name="set-up-azure-monitor-logs-for-a-cluster"></a>Настройка журналов Azure Monitor для кластера

Для мониторинга событий на уровне кластера мы рекомендуем использовать журналы Azure Monitor. Рабочую область Log Analytics можно настроить в Azure Resource Manager, PowerShell или с помощью Azure Marketplace. Если вы сохраняете обновленный шаблон диспетчер ресурсов развертывания для будущего использования, используйте тот же шаблон для настройки среды журналов Azure Monitor. Развертывание через Marketplace проще, если вы уже развернули кластер с включенной диагностикой. Если у вас нет доступа на уровне подписки в учетной записи, в которую производится развертывание, используйте PowerShell или выполните развертывание с помощью шаблона Resource Manager.

> [!NOTE]
> Чтобы настроить журналы Azure Monitor для мониторинга кластера, необходимо включить диагностику для просмотра событий уровня кластера или платформы. Дополнительные сведения можно получить из статей [Агрегирование и сбор событий с помощью системы диагностики Microsoft Azure](service-fabric-diagnostics-event-aggregation-wad.md) и [Агрегирование и сбор событий с помощью системы диагностики Azure для Linux](service-fabric-diagnostics-oms-syslog.md)

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="deploy-a-log-analytics-workspace-by-using-azure-marketplace"></a>Развертывание рабочей области Log Analytics с помощью Azure Marketplace

Если нужно добавить рабочую область Log Analytics после развертывания кластера, перейдите в Azure Marketplace на портале и найдите решение **аналитики Service Fabric**. Это пользовательское решение для развертывания Service Fabric, содержащее данные, относящиеся к Service Fabric. При этом будет создано и решение (панель мониторинга для просмотра аналитики), и рабочая область (агрегирование базовых данных кластера).

1. В области навигации слева щелкните **Создать**. 

2. Найдите **Аналитика Service Fabric**. Выберите появившийся ресурс.

3. Нажмите кнопку **создания**.

    ![Аналитика Service Fabric в Marketplace](media/service-fabric-diagnostics-event-analysis-oms/service-fabric-analytics.png)

4. В окне создания аналитики Service Fabric щелкните **Выберите рабочую область** для поля **Рабочая область OMS**, а затем выберите **Создание рабочей области**. Укажите необходимые записи. Единственным требованием является совпадение подписки для кластера Service Fabric и рабочей области. Когда завершится проверка записей, начнется развертывание рабочей области. Это займет всего несколько минут.

5. Когда процесс завершится, снова выберите **Создать** в нижней части окна создания аналитики Service Fabric. Убедитесь, что новая рабочая область отображается в разделе **Рабочая область OMS**. В таком случае решение добавится в созданную рабочую область.

Если вы используете Windows, выполните следующие действия, чтобы подключить журналы Azure Monitor к учетной записи хранения, в которой хранятся события кластера. 

>[!NOTE]
>Решение Аналитика Service Fabric поддерживается только для кластеров Windows. Для кластеров Linux ознакомьтесь со статьей [о настройке журналов Azure Monitor для кластеров Linux](service-fabric-diagnostics-oms-syslog.md).  

### <a name="connect-the-log-analytics-workspace-to-your-cluster"></a>Подключение рабочей области Log Analytics к кластеру 

1. Рабочая область должна быть подключена к данным диагностики, поступающим из кластера. Перейдите в группу ресурсов, в которой вы создали решение "Аналитика Service Fabric". Выберите **ServiceFabric \<nameOfWorkspace\>** и перейдите на страницу обзора. На этой странице можно изменить параметры решения, рабочей области и получить доступ к рабочей области Log Analytics.

2. В меню навигации слева в разделе **Источники данных рабочей области** выберите **Журналы учетных записей хранения**.

3. На странице **Журналы учетной записи хранения** выберите **Добавить** в верхней части, чтобы добавить в рабочую область журналы кластера.

4. Выберите **Учетная запись хранения**, чтобы добавить учетную запись хранения, созданную в кластере. Если вы использовали имя по умолчанию, то учетная запись хранения будет **сфдг \<resourceGroupName\>**. Вы можете проверить, какое значение указано для параметра **applicationDiagnosticsStorageAccountName** в шаблоне Azure Resource Manager, на основе которого вы развертывали кластер. Если имя не отображается, прокрутите список вниз и выберите **Загрузить еще**. Выберите имя учетной записи хранения.

5. Укажите тип данных. Задайте для параметра типа данных значение **События Service Fabric**.

6. При этом для параметра "Источник" будет автоматически установлено значение **WADServiceFabric\*EventTable**.

7. Нажмите кнопку **ОК**, чтобы подключить рабочую область к журналам кластера.

    ![Добавление журналов учетной записи хранения в журналы Azure Monitor](media/service-fabric-diagnostics-event-analysis-oms/add-storage-account.png)

Теперь учетная запись будет отображаться в списке журналов учетной записи хранения в разделе источников данных для рабочей области.

Вы успешно добавили решение "Аналитика Service Fabric" в рабочую область Log Analytics, подключенную должным образом к платформе кластера и таблице журналов приложения. Аналогичным образом вы можете добавить в рабочую область дополнительные источники.


## <a name="deploy-azure-monitor-logs-with-azure-resource-manager"></a>Развертывание журналов Azure Monitor с помощью Azure Resource Manager

При развертывании кластера с помощью шаблона Resource Manager этот шаблон создает рабочую область Log Analytics, добавляет в нее решение Service Fabric и настраивает в ней чтение данных из соответствующих таблиц службы хранилища.

Вы можете использовать [этот пример шаблона](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-OMS-UnSecure), а также изменить его в соответствии со своими потребностями. Этот шаблон делает следующее:

* создает кластер Service Fabric с 5 узлами;
* создает рабочую область Log Analytics и решение Service Fabric;
* настраивает агент Log Analytics для сбора и отправки данных с 2 счетчиков производительности в рабочую область;
* настраивает WAD для сбора данных Service Fabric и отправляет их в таблицы хранилища Azure (WADServiceFabric*EventTable);
* настраивает рабочую область Log Analytics для считывания событий из этих таблиц.


Вы можете развернуть шаблон в качестве диспетчер ресурсов обновления в кластере с помощью `New-AzResourceGroupDeployment` API в модуле Azure PowerShell. Пример команды будет выглядеть так:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resourceGroupName>" -TemplateFile "<templatefile>.json" 
``` 

Azure Resource Manager определит, что эта команда — это обновление имеющегося ресурса. Он обработает только изменения между шаблоном, управляющим имеющимся развертыванием, и новым предоставленным шаблоном.

## <a name="deploy-azure-monitor-logs-with-azure-powershell"></a>Развертывание журналов Azure Monitor с помощью Azure PowerShell

Вы также можете развернуть ресурс log Analytics с помощью PowerShell, выполнив `New-AzOperationalInsightsWorkspace` команду. Чтобы использовать этот метод, обязательно установите [Azure Powershell](/powershell/azure/install-az-ps). Чтобы создать рабочую область Log Analytics и добавить в нее решение Service Fabric, используйте следующий сценарий: 

```powershell

$SubID = "<subscription ID>"
$ResourceGroup = "<Resource group name>"
$Location = "<Resource group location>"
$WorkspaceName = "<Log Analytics workspace name>"
$solution = "ServiceFabric"

# Sign in to Azure and access the correct subscription
Connect-AzAccount
Select-AzSubscription -SubscriptionId $SubID 

# Create the resource group if needed
try {
    Get-AzResourceGroup -Name $ResourceGroup -ErrorAction Stop
} catch {
    New-AzResourceGroup -Name $ResourceGroup -Location $Location
}

New-AzOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku Standard -ResourceGroupName $ResourceGroup
Set-AzOperationalInsightsIntelligencePack -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -IntelligencePackName $solution -Enabled $true

```

Когда все будет готово, выполните действия, описанные в предыдущем разделе, чтобы подключить журналы Azure Monitor к соответствующей учетной записи хранения.

Вы также можете добавить другие решения или внести другие изменения в рабочую область Log Analytics с помощью PowerShell. Дополнительные сведения см. в статье [Управление журналами Azure Monitor с помощью PowerShell](../azure-monitor/logs/powershell-workspace-configuration.md).

## <a name="next-steps"></a>Дальнейшие шаги
* [Разверните агент Log Analytics](service-fabric-diagnostics-oms-agent.md) на узлах для сбора данных счетчиков производительности и статистики Docker, а также журналов для контейнеров.
* Ознакомьтесь с функциями [поиска по журналам и запросов к журналам](../azure-monitor/logs/log-query-overview.md), которые являются частью журналов Azure Monitor
* [Использование конструктора представлений для создания пользовательских представлений в журналах Azure Monitor](../azure-monitor/visualize/view-designer.md)

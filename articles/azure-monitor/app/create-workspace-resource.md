---
title: Создание ресурса рабочей области Azure Monitor Application Insights | Документация Майкрософт
description: Узнайте, что нужно сделать, чтобы использовать новые ресурсы на основе рабочих областей Azure Monitor Application Insights.
ms.topic: conceptual
ms.date: 10/06/2020
ms.openlocfilehash: 3ec0b25a24af13b29a3568165009f8a6d66e0218
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100578333"
---
# <a name="workspace-based-application-insights-resources"></a>Ресурсы рабочей области Application Insights.

Ресурсы рабочей области поддерживают полную интеграцию Application Insights и Log Analytics. Теперь вы можете отправлять данные телеметрии Application Insights в общую рабочую область Log Analytics, которая обеспечивает полный доступ ко всем функциям Log Analytics, сохраняя журналы приложений, инфраструктуры и платформы в централизованном расположении.

Это также обеспечивает общий контроль доступа на основе ролей Azure (Azure RBAC) между ресурсами и устраняет необходимость в перекрестных запросах между приложениями и рабочими областями.

> [!NOTE]
> Прием и хранение данных ресурсов рабочей области Application Insights оплачиваются по тарифам рабочей области Log Analytics, в которой находятся данные. [Узнайте больше]( ./pricing.md#workspace-based-application-insights) о выставлении счетов за ресурсы рабочей области Application Insights.

## <a name="new-capabilities"></a>Новые возможности

Application Insights на основе рабочей области позволяет использовать преимущества новейших возможностей Azure Monitor и Log Analytics включая:

* [Ключи, управляемые клиентом (CMK)](../logs/customer-managed-keys.md) , обеспечивают шифрование неактивных данных с ключами шифрования, к которым у вас есть доступ.
* [Приватный канал Azure](../logs/private-link-security.md) позволяет безопасно связать службы Azure PaaS с виртуальной сетью с помощью частных конечных точек.
* [Приведите свое хранилище (BYOS) для профайлера и snapshot Debugger](./profiler-bring-your-own-storage.md) предоставляет полный контроль над политикой шифрования неактивных данных, политикой управления жизненным циклом и сетевым доступом для всех связанных с Application Insights Profiler и snapshot Debugger. 
* [Уровни резервирования емкости](../logs/manage-cost-storage.md#pricing-model) позволяют сэкономить до 25% по сравнению с ценой оплаты по мере использования. 
* Ускорение приема данных за счет Log Analytics приема потоковой передачи.

## <a name="create-workspace-based-resource"></a>Создание ресурса на основе рабочей области

Войдите в [портал Azure](https://portal.azure.com)и создайте ресурс Application Insights:

> [!div class="mx-imgBorder"]
> ![Ресурс рабочей области Application Insights](./media/create-workspace-resource/create-workspace-based.png)

Если у вас еще нет рабочей области Log Analytics, [обратитесь к документации по созданию рабочей области Log Analytics](../logs/quick-create-workspace.md).

**Ресурсы на основе рабочей области в настоящее время доступны во всех коммерческих регионах и в Azure для государственных организаций**

После создания ресурса информация о соответствующей рабочей области отобразится в области **Обзор**.

![Имя рабочей области](./media/create-workspace-resource/workspace-name.png)

Щелкнув синий текст ссылки, вы перейдете в соответствующую рабочую область Log Analytics, где сможете воспользоваться преимуществами новой унифицированной среды обработки запросов рабочей области.

> [!NOTE]
> Мы по-прежнему предоставляем полную обратную совместимость с классическими запросами, ресурсами, книгами и оповещениями на основе журналов Application Insights. Для запроса или просмотра [структуры или схемы новой таблицы рабочей области](apm-tables.md), необходимо сначала перейти в рабочую область Log Analytics. Выбор **журналов (Analytics)** в панелях Application Insights предоставит доступ к классическому интерфейсу Application Insights запросов.

## <a name="copy-the-connection-string"></a>Копирование строки подключения

[Строка подключения](./sdk-connection-string.md?tabs=net) определяет ресурс, с которым необходимо связать данные телеметрии. Она также позволяет изменить конечные точки, которые ресурс будет использовать в качестве назначения для данных телеметрии. Необходимо скопировать строку подключения и добавить ее в код приложения или переменную среды.

## <a name="monitoring-configuration"></a>Конфигурация мониторинга

После создания ресурса рабочей области Application Insights настроить мониторинг относительно просто.

### <a name="code-based-application-monitoring"></a>Мониторинг приложений на основе кода

Чтобы реализовать мониторинг приложений на основе кода, нужно просто установить соответствующий пакет SDK для Application Insights и указать ключ инструментирования или строку подключения к только что созданному ресурсу.  

Подробную информацию по настройке пакета SDK для Application Insights для мониторинга на основе кода можно найти в соответствующей документации для языка или платформы.

- [ASP.NET](./asp-net.md)
- [ASP.NET Core ](./asp-net-core.md)
- [Фоновые задачи и современные консольные приложения (.NET и .NET Core)](./worker-service.md)
- [Классические консольные приложения (.NET)](./console.md) 
- [Java ](./java-get-started.md?tabs=maven)
- [JavaScript](./javascript.md)
- [Node.js](./nodejs.md)
- [Python](./opencensus-python.md)

### <a name="codeless-monitoring-and-visual-studio-resource-creation"></a>Мониторинг без написания кода и создание ресурсов Visual Studio

Чтобы использовать мониторинг без написания кода для таких служб, как Функции Azure и службы приложений Azure, необходимо сначала создать ресурс рабочей области Application Insights, а затем указать этот ресурс на этапе настройки мониторинга.

Хотя эти службы дают возможность создать ресурс Application Insights в рамках собственного процесса создания ресурсов, ресурсы, созданные с помощью этих параметров пользовательского интерфейса, в настоящее время ограничены классическим интерфейсом Application Insights.

То же самое относится к интерфейсу создания ресурсов Application Insights в Visual Studio для ASP.NET и ASP.NET Core. Необходимо выбрать существующий ресурс рабочей области с помощью пользовательского интерфейса включения мониторинга Visual Studio. Если выбрать создание ресурса в Visual Studio, то можно будет создать только классический ресурс Application Insights.

## <a name="creating-a-resource-automatically"></a>Автоматическое создание ресурса

### <a name="azure-cli"></a>Azure CLI

Чтобы получить доступ к командам Azure CLI для предварительной версии Application Insights, сначала необходимо выполнить следующее.

```azurecli
 az extension add -n application-insights
```

Если не выполнить команду `az extension add`, появится сообщение об ошибке, в котором будет указано следующее: `az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`

Теперь можно выполнить следующие действия, чтобы создать ресурс Application Insights.

```azurecli
az monitor app-insights component create --app
                                         --location
                                         --resource-group
                                         [--application-type]
                                         [--ingestion-access {Disabled, Enabled}]
                                         [--kind]
                                         [--only-show-errors]
                                         [--query-access {Disabled, Enabled}]
                                         [--tags]
                                         [--workspace]
```

#### <a name="example"></a>Пример

```azurecli
az monitor app-insights component create --app demoApp --location eastus --kind web -g my_resource_group --workspace "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/test1234/providers/microsoft.operationalinsights/workspaces/test1234555"
```

Полную документацию Azure CLI по этой команде можно найти [здесь](/cli/azure/ext/application-insights/monitor/app-insights/component#ext-application-insights-az-monitor-app-insights-component-create).

### <a name="azure-powershell"></a>Azure PowerShell

В настоящее время команда PowerShell `New-AzApplicationInsights` не поддерживает создание ресурса рабочей области Application Insights. Чтобы создать ресурс рабочей области с помощью PowerShell, можно использовать приведенные ниже шаблоны Azure Resource Manager и выполнить развертывание с помощью PowerShell.

### <a name="azure-resource-manager-templates"></a>Шаблоны Azure Resource Manager

#### <a name="template-file"></a>Файл шаблона

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string"
        },
        "type": {
            "type": "string"
        },
        "regionId": {
            "type": "string"
        },
        "tagsArray": {
            "type": "object"
        },
        "requestSource": {
            "type": "string"
        },
        "workspaceResourceId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('name')]",
            "type": "microsoft.insights/components",
            "location": "[parameters('regionId')]",
            "tags": "[parameters('tagsArray')]",
            "apiVersion": "2020-02-02-preview",
            "properties": {
                "ApplicationId": "[parameters('name')]",
                "Application_Type": "[parameters('type')]",
                "Flow_Type": "Redfield",
                "Request_Source": "[parameters('requestSource')]",
                "WorkspaceResourceId": "[parameters('workspaceResourceId')]"
            }
        }
    ]
}
```

#### <a name="parameters-file"></a>Файл параметров

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "type": {
            "value": "web"
        },
        "name": {
            "value": "customresourcename"
        },
        "regionId": {
            "value": "eastus"
        },
        "tagsArray": {
            "value": {}
        },
        "requestSource": {
            "value": "Custom"
        },
        "workspaceResourceId": {
            "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my_resource_group/providers/microsoft.operationalinsights/workspaces/myworkspacename"
        }
    }
}

```

## <a name="modifying-the-associated-workspace"></a>Изменение связанной рабочей области

После создания ресурса рабочей области Application Insights можно изменить связанную с ним рабочую область Log Analytics.

В области ресурсов Application Insights выберите **Свойства** > **Изменить рабочую область** > **Рабочие области Log Analytics**.

## <a name="export-telemetry"></a>Экспорт данных телеметрии

Устаревшие функции непрерывного экспорта не поддерживаются для ресурсов рабочей области. Вместо этого в ресурсе Application Insights выберите **Параметры диагностики** > **Добавить параметр диагностики**. Можно выбрать все таблицы или подмножество таблиц для архивации в учетную запись хранения или использовать потоковую передачу в концентратор событий Azure.

> [!NOTE]
> В настоящее время дополнительная плата за экспорт телеметрии отсутствует. Сведения о ценах на этот компонент будут доступны на [странице цен на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).  До начала выставления счетов уведомления будут отправляться. Если вы решили продолжить использование <feature name> после периода уведомления, плата будет взиматься по соответствующей ставке. 
 

## <a name="next-steps"></a>Дальнейшие действия

* [Изучение метрик](../essentials/metrics-charts.md)
* [Написание запросов аналитики](../logs/log-query-overview.md)

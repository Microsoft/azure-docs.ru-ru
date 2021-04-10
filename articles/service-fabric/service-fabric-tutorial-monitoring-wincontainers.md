---
title: Мониторинг и диагностика состояния контейнеров Windows
description: В этом руководстве вы настроите журналы Azure Monitor для мониторинга и диагностики контейнеров Windows в Azure Service Fabric.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: b7689d6e259055137a8d1d3c61552790ab9f28d3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100588245"
---
# <a name="tutorial-monitor-windows-containers-on-service-fabric-using-azure-monitor-logs"></a>Руководство по Мониторинг контейнеров Windows в Service Fabric с помощью журналов Azure Monitor

Это третья часть руководства, в которой описывается настройка журналов Azure Monitor для мониторинга контейнеров Windows, управляемых в Service Fabric.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка журналов Azure Monitor для кластера Service Fabric;
> * использование рабочей области Log Analytics для просмотра и запроса журналов из контейнеров и узлов;
> * настройка агента Log Analytics для сбора метрик контейнеров и узлов.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется:

* использовать имеющийся кластер в Azure или [создать новый с помощью данного руководства](service-fabric-tutorial-create-vnet-and-windows-cluster.md);
* [развернуть в нем контейнерное приложение](service-fabric-host-app-in-a-container.md).

## <a name="setting-up-azure-monitor-logs-with-your-cluster-in-the-resource-manager-template"></a>Настройка журналов Azure Monitor для кластера в шаблоне Resource Manager

В случае, если вы использовали [шаблон](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-OMS-UnSecure), предоставленный в первой части этого руководства, он должен содержать указанные ниже дополнения к универсальному шаблону Azure Resource Manager для Service Fabric. В случае, если вы использовали собственный кластер, для которого необходимо настроить мониторинг контейнеров с помощью журналов Azure Monitor, сделайте следующее.

* Внесите приведенные ниже изменения в шаблон Resource Manager.
* [Разверните этот шаблон](./service-fabric-cluster-creation-via-arm.md) с помощью PowerShell, чтобы обновить кластер. Azure Resource Manager определит, что ресурс уже существует, поэтому обновит его.

### <a name="adding-azure-monitor-logs-to-your-cluster-template"></a>Добавление журналов Azure Monitor в шаблон кластера

Внесите следующие изменения в файл *template.json*.

1. Добавьте расположение рабочей области Log Analytics и имя раздела *parameters*:

    ```json
    "omsWorkspacename": {
      "type": "string",
      "defaultValue": "[toLower(concat('sf',uniqueString(resourceGroup().id)))]",
      "metadata": {
        "description": "Name of your Log Analytics Workspace"
      }
    },
    "omsRegion": {
      "type": "string",
      "defaultValue": "East US",
      "allowedValues": [
        "West Europe",
        "East US",
        "Southeast Asia"
      ],
      "metadata": {
        "description": "Specify the Azure Region for your Log Analytics workspace"
      }
    }
    ```

    Чтобы изменить значение какого-либо параметра, добавьте такой же параметр в свой файл *template.parameters.json* и измените его значение.

2. Добавьте имя решения и решение в раздел *variables*.

    ```json
    "omsSolutionName": "[Concat('ServiceFabric', '(', parameters('omsWorkspacename'), ')')]",
    "omsSolution": "ServiceFabric"
    ```

3. Добавьте Microsoft Monitoring Agent в качестве расширения виртуальной машины. Найдите ресурс масштабируемых наборов виртуальных машин: *resources* >  *"apiVersion": "[variables('vmssApiVersion')]"* . В раздел *properties* > *virtualMachineProfile* > *extensionProfile* > *extensions* добавьте следующее описание расширения под расширением *ServiceFabricNode*. 
    
    ```json
    {
        "name": "[concat(variables('vmNodeType0Name'),'OMS')]",
        "properties": {
            "publisher": "Microsoft.EnterpriseCloud.Monitoring",
            "type": "MicrosoftMonitoringAgent",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
            },
            "protectedSettings": {
                "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
            }
        }
    },
    ```

4. Добавьте рабочую область Log Analytics в качестве отдельного ресурса. Добавьте приведенный ниже код в раздел *resources* после ресурса масштабируемых наборов виртуальных машин.

    ```json
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[parameters('omsWorkspacename')]",
        "type": "Microsoft.OperationalInsights/workspaces",
        "properties": {
            "sku": {
                "name": "Free"
            }
        },
        "resources": [
            {
                "apiVersion": "2015-11-01-preview",
                "name": "[concat(variables('applicationDiagnosticsStorageAccountName'),parameters('omsWorkspacename'))]",
                "type": "storageinsightconfigs",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]",
                    "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
                ],
                "properties": {
                    "containers": [ ],
                    "tables": [
                        "WADServiceFabric*EventTable",
                        "WADWindowsEventLogsTable",
                        "WADETWEventTable"
                    ],
                    "storageAccount": {
                        "id": "[resourceId('Microsoft.Storage/storageaccounts/', variables('applicationDiagnosticsStorageAccountName'))]",
                        "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-06-15').key1]"
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "name": "System",
                "type": "datasources",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsEvent",
                "properties": {
                    "eventLogName": "System",
                    "eventTypes": [
                        {
                            "eventType": "Error"
                        },
                        {
                            "eventType": "Warning"
                        },
                        {
                            "eventType": "Information"
                        }
                    ]
                }
            }
        ]
    },
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[variables('omsSolutionName')]",
        "type": "Microsoft.OperationsManagement/solutions",
        "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('OMSWorkspacename'))]"
        ],
        "properties": {
            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        },
        "plan": {
            "name": "[variables('omsSolutionName')]",
            "publisher": "Microsoft",
            "product": "[Concat('OMSGallery/', variables('omsSolution'))]",
            "promotionCode": ""
        }
    },
    ```

[Здесь](https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/d2ffa318581fc23ac7f1b0ab2b52db1a0d7b4ba7/5-VM-Windows-OMS-UnSecure/sfclusteroms.json) доступен пример шаблона (из в первой части данного руководства) со всеми этими изменениями, который можно использовать для справки, если потребуется. Описанные изменения позволяют добавить рабочую область Log Analytics в группу ресурсов. Рабочая область продолжит собирать события платформы Service Fabric из таблиц хранилища, настроенных для агента [Диагностики Azure для Windows](service-fabric-diagnostics-event-aggregation-wad.md). Агент Log Analytics (Microsoft Monitoring Agent) также был добавлен на каждый узел кластера как расширение виртуальной машины. Это означает, что при масштабировании кластера агент автоматически настраивается на каждом компьютере и привязывается к той же рабочей области.

Разверните шаблон с новыми изменениями, чтобы обновить текущий кластер. После этого ресурсы Log Analytics должны отобразиться в вашей группе ресурсов. Когда кластер будет готов к работе, разверните в нем свое контейнерное приложение. На следующем шаге мы настроим мониторинг контейнеров.

## <a name="add-the-container-monitoring-solution-to-your-log-analytics-workspace"></a>Добавление решения для мониторинга контейнеров в рабочую область Log Analytics

Чтобы настроить решение для мониторинга контейнеров в рабочей области, найдите *решение для мониторинга контейнеров* и создайте ресурс контейнеров (в категории "Мониторинг и управление").

![Добавление решения "Контейнеры"](./media/service-fabric-tutorial-monitoring-wincontainers/containers-solution.png)

При появлении запроса о *рабочей области Log Analytics* выберите рабочую область, которая была создана в вашей группе ресурсов, и нажмите кнопку **Создать**. *Решение для мониторинга контейнеров* будет добавлено в вашу рабочую область, инициируя агент Log Analytics, развернутый с помощью шаблона, чтобы начать сбор журналов и статистики Docker.

Вернитесь к своей *группе ресурсов*, в которой должно появиться добавленное решение для мониторинга. Если выбрать его, должна отобразиться целевая страница с информацией о числе выполняемых образов контейнеров.

*Обратите внимание на то, что у нас выполняются пять экземпляров контейнера fabrikam из [второй части](service-fabric-host-app-in-a-container.md) руководства*

![Целевая страница решения мониторинга контейнеров](./media/service-fabric-tutorial-monitoring-wincontainers/solution-landing.png)

Если выбрать **Решение мониторинга контейнеров**, можно перейти к более подробной панели мониторинга. В ней можно прокручивать несколько панелей, а также выполнять запросы в журналах Azure Monitor.

Так как агент собирает журналы Docker, по умолчанию он отображает *stdout* и *stderr*. Если прокрутить страницу горизонтально, вы увидите перечень образов контейнеров, состояние, метрики и примеры запросов, которые можно выполнить, чтобы получить более полезные данные.

![Панель мониторинга решения мониторинга контейнеров](./media/service-fabric-tutorial-monitoring-wincontainers/container-metrics.png)

Щелкните любую из этих панелей, чтобы перейти к запросу Kusto, создающему отображаемое значение. Измените запрос на *\** , чтобы просмотреть различные виды журналов, сбор которых выполняется. Здесь можно выполнить запрос или применить фильтр по производительности контейнеров или журналам. Можно также просмотреть события платформы Service Fabric. Агенты также постоянно передают пульс с каждого узла, который можно просмотреть, чтобы в случае изменения конфигурации кластера проверить, собираются по-прежнему ли данные со всех компьютеров.

![Запрос контейнера](./media/service-fabric-tutorial-monitoring-wincontainers/query-sample.png)

## <a name="configure-log-analytics-agent-to-pick-up-performance-counters"></a>Настройка агента Log Analytics для сбора счетчиков производительности

Еще одним преимуществом агента Log Analytics является то, что он дает возможность изменить собираемые счетчики производительности, воспользовавшись пользовательским интерфейсом Log Analytics. Вам не придется настраивать агент системы диагностики Azure и каждый раз выполнять обновление с помощью шаблона Resource Manager. Чтобы сделать это, на целевой странице решения мониторинга контейнеров (или Service Fabric) выберите **Рабочая область OMS**.

В рабочей области Log Analytics можно просматривать решения, создавать настраиваемые панели мониторинга, а также настраивать агент Log Analytics. 
* Чтобы открыть меню дополнительных параметров, выберите **Дополнительные параметры**;
* выберите **Подключенные источники** > **Серверы с Windows**, чтобы убедиться в наличии *5 подключенных компьютеров Windows*;
* выберите **Данные** > **Счетчики производительности Windows**, чтобы найти и добавить новые счетчики производительности. Здесь можно увидеть список рекомендаций журналов Azure Monitor по собранным счетчикам производительности, а также производить поиск других счетчиков. Убедитесь, что собираются данные со счетчиков **Процессор(_всего)\% Загруженность процессора** и **Память(*)\Доступно мегабайт**.

Через несколько минут **обновите** решение для мониторинга контейнеров. В нем должны начать появляться данные о *производительности компьютеров*. Это поможет вам понять, как используются ресурсы. Кроме того, эти метрики можно использовать для принятия правильных решений о масштабировании кластера или для проверки соответствия балансировки нагрузки кластера ожидаемым показателям.

*Примечание. Убедитесь, что фильтры времени правильно настроены для использования этих метрик.*

![Счетчики производительности 2](./media/service-fabric-tutorial-monitoring-wincontainers/perf-counters2.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * настройка журналов Azure Monitor для кластера Service Fabric;
> * использование рабочей области Log Analytics для просмотра и запроса журналов из контейнеров и узлов;
> * настройка агента Log Analytics для сбора метрик контейнеров и узлов.

Теперь, когда вы настроили мониторинг для контейнерного приложения, попробуйте выполнить следующее:

* настройте журналы Azure Monitor для кластера Linux, выполнив действия, аналогичные приведенным выше. Используйте в качестве образца [этот шаблон](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeType-Secure-OMS), чтобы внести изменения в шаблон Resource Manager.
* В журналах Azure Monitor настройте [автоматические оповещения](../azure-monitor/alerts/alerts-overview.md), которые помогут выполнять обнаружение и диагностику.
* Просмотрите список [рекомендуемых счетчиков производительности](service-fabric-diagnostics-event-generation-perf.md) Service Fabric, которые можно настроить для кластеров.
* Ознакомьтесь с функциями [поиска по журналам и запросов к журналам](../azure-monitor/logs/log-query-overview.md), которые являются частью журналов Azure Monitor.

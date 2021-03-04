---
title: Поддерживаемые ресурсы для оповещений метрик в Azure Monitor
description: Справочник по поддерживаемым метрикам и журналам для оповещений метрик в Azure Monitor
author: harelbr
ms.author: harelbr
services: monitoring
ms.topic: conceptual
ms.date: 02/10/2021
ms.openlocfilehash: c282e6890d56fe047b319f72e05cdc97de76cfcf
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102038192"
---
# <a name="supported-resources-for-metric-alerts-in-azure-monitor"></a>Поддерживаемые ресурсы для оповещений метрик в Azure Monitor

Azure Monitor теперь поддерживает [новый тип оповещений о метриках](./alerts-overview.md), который имеет ряд преимуществ перед [классическими оповещениями](./alerts-classic.overview.md). Метрики доступны для [большого числа служб Azure](../essentials/metrics-supported.md). Список типов ресурсов, поддерживаемых новыми оповещениями, постоянно расширяется. В этой статье рассматриваются новые возможности.

Вы также можете использовать новые оповещения метрики для популярных данных журналов, хранящихся в Log Analytics рабочей области, извлеченной в качестве метрик. Дополнительные сведения см. в статье [Создание оповещений о метриках для журналов в Azure Monitor](./alerts-metric-logs.md).

## <a name="portal-powershell-cli-rest-support"></a>Поддержка портала, PowerShell, CLI и REST
Сейчас можно создавать новые оповещения метрик только в [шаблонах](./alerts-metric-create-templates.md)портал Azure, [REST API](/rest/api/monitor/metricalerts/)или диспетчер ресурсов. Возможность настройки новых оповещений с помощью PowerShell и Azure CLI версии 2.0 и выше будет реализована в ближайшее время.

## <a name="metrics-and-dimensions-supported"></a>Поддерживаемые метрики и измерения
Новые оповещения на основе метрик поддерживают оповещения для метрик, использующих измерения. Измерения можно использовать для фильтрации метрик до необходимого уровня. Все поддерживаемые метрики с применимыми измерениями можно изучить и визуализировать с помощью [обозревателя метрик Azure Monitor](../essentials/metrics-charts.md).

Ниже приведен полный список источников метрик Azure Monitor, поддерживаемых новыми оповещениями.

|Тип ресурса  |Поддерживаемые измерения |Оповещения о нескольких ресурсах| Доступные метрики|
|---------|---------|-----|----------|
|Microsoft. Аадиам/Азуреадметрикс | Да | Нет | |
|Microsoft.ApiManagement/service | Да | Нет | [Управление API](../essentials/metrics-supported.md#microsoftapimanagementservice) |
|Microsoft.AppConfiguration/configurationStores |Да | Нет | [Конфигурация приложений](../essentials/metrics-supported.md#microsoftappconfigurationconfigurationstores) |
|Microsoft.AppPlatform/Spring | Да | Нет | [Azure Spring Cloud](../essentials/metrics-supported.md#microsoftappplatformspring) |
|Microsoft.Automation/automationAccounts | Да| Нет | [Учетные записи автоматизации](../essentials/metrics-supported.md#microsoftautomationautomationaccounts) |
|Microsoft. AVS/Приватеклаудс | Нет | Нет | [Решение Azure VMware](../essentials/metrics-supported.md#microsoftavsprivateclouds) |
|Microsoft.Batch/batchAccounts | Да | Нет | [Ученые записи пакетной службы](../essentials/metrics-supported.md#microsoftbatchbatchaccounts) |
|Microsoft.Cache/Redis; | Да | Да | [Кэш Azure для Redis](../essentials/metrics-supported.md#microsoftcacheredis) |
|Microsoft.ClassicCompute/domainNames/slots/roles | Нет | Нет | [Классические облачные службы](../essentials/metrics-supported.md#microsoftclassiccomputedomainnamesslotsroles) |
|Microsoft.classicСompute/virtualMachines | Нет | Нет | [Классические виртуальные машины](../essentials/metrics-supported.md#microsoftclassiccomputevirtualmachines) |
|Microsoft.ClassicStorage/storageAccounts | Да | Нет | [Учетные записи хранения (классические)](../essentials/metrics-supported.md#microsoftclassicstoragestorageaccounts) |
|Microsoft.ClassicStorage/storageAccounts/blobServices | Да | Нет | [Учетные записи хранения (классические) — BLOB-объекты](../essentials/metrics-supported.md#microsoftclassicstoragestorageaccountsblobservices) |
|Microsoft.ClassicStorage/storageAccounts/fileServices | Да | Нет | [Учетные записи хранения (классические) — файлы](../essentials/metrics-supported.md#microsoftclassicstoragestorageaccountsfileservices) |
|Microsoft.ClassicStorage/storageAccounts/queueServices | Да | Нет | [Учетные записи хранения (классические) — очереди](../essentials/metrics-supported.md#microsoftclassicstoragestorageaccountsqueueservices) |
|Microsoft.ClassicStorage/storageAccounts/tableServices | Да | Нет | [Учетные записи хранения (классические) — таблицы](../essentials/metrics-supported.md#microsoftclassicstoragestorageaccountstableservices) |
|Microsoft.CognitiveServices/accounts | Да | Нет | [Cognitive Services](../essentials/metrics-supported.md#microsoftcognitiveservicesaccounts) |
|Microsoft.Compute/virtualMachines | Да | Да<sup>1</sup> | [Виртуальные машины](../essentials/metrics-supported.md#microsoftcomputevirtualmachines) |
|Microsoft.Compute/virtualMachineScaleSets; | Да | Нет |[Масштабируемые наборы виртуальных машин](../essentials/metrics-supported.md#microsoftcomputevirtualmachinescalesets) |
|Microsoft.ContainerInstance/containerGroups | Да| Нет | [Группы контейнеров](../essentials/metrics-supported.md#microsoftcontainerinstancecontainergroups) |
|Microsoft.ContainerRegistry/registries | Нет | Нет | [Реестры контейнеров](../essentials/metrics-supported.md#microsoftcontainerregistryregistries) |
|Microsoft.ContainerService/managedClusters | Да | Нет | [Управляемые кластеры](../essentials/metrics-supported.md#microsoftcontainerservicemanagedclusters) |
|Microsoft.DataBoxEdge/dataBoxEdgeDevices | Да | Да | [Data Box](../essentials/metrics-supported.md#microsoftdataboxedgedataboxedgedevices) |
|Microsoft.DataFactory/datafactories| Да| Нет | [Фабрики данных V1](../essentials/metrics-supported.md#microsoftdatafactorydatafactories) |
|Microsoft.DataFactory/factories; |Да | Нет | [Фабрики данных V2](../essentials/metrics-supported.md#microsoftdatafactoryfactories) |
|Microsoft.DataShare/accounts | Да | Нет | [Общие ресурсы данных](../essentials/metrics-supported.md#microsoftdatashareaccounts) |
|Microsoft.DBforMariaDB/servers | Нет | Нет | [База данных для MariaDB](../essentials/metrics-supported.md#microsoftdbformariadbservers) |
|Microsoft.DBforMySQL/servers | Нет | Нет |[База данных для MySQL](../essentials/metrics-supported.md#microsoftdbformysqlservers)|
|Microsoft.DBforPostgreSQL/servers | Нет | Нет | [База данных для PostgreSQL](../essentials/metrics-supported.md#microsoftdbforpostgresqlservers)|
|Microsoft.DBforPostgreSQL/serversv2 | Нет | Нет | [База данных для PostgreSQL v2](../essentials/metrics-supported.md#microsoftdbforpostgresqlserversv2)|
|Microsoft.DBforPostgreSQL/flexibleServers | Да | Нет | [База данных для PostgreSQL (гибкие серверы)](../essentials/metrics-supported.md#microsoftdbforpostgresqlflexibleservers)|
|Microsoft.Devices/IotHubs | Да | Нет |[Центр Интернета вещей](../essentials/metrics-supported.md#microsoftdevicesiothubs) |
|Microsoft.Devices/provisioningServices| Да | Нет | [Службы подготовки устройств](../essentials/metrics-supported.md#microsoftdevicesprovisioningservices) |
|Microsoft. Дигиталтвинс/Дигиталтвинсинстанцес | Да | Нет | [Digital Twins](../essentials/metrics-supported.md#microsoftdigitaltwinsdigitaltwinsinstances) |
|Microsoft.DocumentDB/databaseAccounts | Да | Нет | [База данных Cosmos](../essentials/metrics-supported.md#microsoftdocumentdbdatabaseaccounts) |
|Microsoft.EventGrid/domains | Да | Нет | [Домены Сетки событий](../essentials/metrics-supported.md#microsofteventgriddomains) |
|Microsoft. EventGrid/Системтопикс | Да | Нет | [Статьи по системе сетки событий](../essentials/metrics-supported.md#microsofteventgridsystemtopics) |
|Microsoft.EventGrid/topics |Да | Нет | [Разделы Сетки событий](../essentials/metrics-supported.md#microsofteventgridtopics) |
|Microsoft.EventHub/clusters |Да| Нет | [Кластеры концентраторов событий](../essentials/metrics-supported.md#microsofteventhubclusters) |
|Microsoft.EventHub/namespaces |Да| Нет | [Центры событий](../essentials/metrics-supported.md#microsofteventhubnamespaces) |
|Microsoft.HDInsight/clusters | Да | Нет | [Кластеры HDInsight](../essentials/metrics-supported.md#microsofthdinsightclusters) |
|Microsoft.Insights/Components | Да | Нет | [Application Insights](../essentials/metrics-supported.md#microsoftinsightscomponents) |
|Microsoft.KeyVault/vaults | Да |Да |[Хранилища](../essentials/metrics-supported.md#microsoftkeyvaultvaults)|
|Microsoft.Kusto/Clusters | Да |Нет |[Кластеры обозреватель данных](../essentials/metrics-supported.md#microsoftkustoclusters)|
|Microsoft.Logic/integrationServiceEnvironments | Да | Нет |[Среды службы интеграции](../essentials/metrics-supported.md#microsoftlogicintegrationserviceenvironments) |
|Microsoft.Logic/workflows | Нет | Нет |[Logic Apps](../essentials/metrics-supported.md#microsoftlogicworkflows) |
|Microsoft.MachineLearningServices/workspaces | Да | Нет | [Машинное обучение](../essentials/metrics-supported.md#microsoftmachinelearningservicesworkspaces) |
|Microsoft.Maps/accounts | Да | Нет | [Сопоставление учетных записей](../essentials/metrics-supported.md#microsoftmapsaccounts) |
|Microsoft.Media/mediaservices | Нет | Нет | [Службы мультимедиа](../essentials/metrics-supported.md#microsoftmediamediaservices) |
|Microsoft.Media/mediaservices/streamingEndpoints | Да | Нет | [Конечные точки потоковой передачи служб мультимедиа](../essentials/metrics-supported.md#microsoftmediamediaservicesstreamingendpoints) |
|Microsoft.NetApp/netAppAccounts/capacityPools | Да | Да | [Пулы ресурсов Azure NetApp](../essentials/metrics-supported.md#microsoftnetappnetappaccountscapacitypools) |
|Microsoft.NetApp/netAppAccounts/capacityPools/volumes | Да | Да | [Тома NetApp для Azure](../essentials/metrics-supported.md#microsoftnetappnetappaccountscapacitypoolsvolumes) |
|Microsoft.Network/applicationGateways | Да | Нет | [Шлюзы приложений](../essentials/metrics-supported.md#microsoftnetworkapplicationgateways) |
|Microsoft.Network/azurefirewalls | Да | Нет | [брандмауэры;](../essentials/metrics-supported.md#microsoftnetworkazurefirewalls) |
|Microsoft.Network/dnsZones | Нет | Нет | [Зоны DNS](../essentials/metrics-supported.md#microsoftnetworkdnszones) |
|Microsoft.Network/expressRouteCircuits | Да | Нет |[Каналы ExpressRoute](../essentials/metrics-supported.md#microsoftnetworkexpressroutecircuits) |
|Microsoft.Network/expressRoutePorts | Да | Нет |[ExpressRoute Direct](../essentials/metrics-supported.md#microsoftnetworkexpressrouteports) |
|Microsoft.Network/loadBalancers (только для SKU "Стандартный")| Да| Нет | [Подсистемы балансировки нагрузки](../essentials/metrics-supported.md#microsoftnetworkloadbalancers) |
|Microsoft. Network/Натгатевайс| Нет | Нет | [Шлюзы NAT](../essentials/metrics-supported.md#microsoftnetworknatgateways) |
|Microsoft. Network/Приватиндпоинтс| Нет | Нет | [Частные конечные точки](../essentials/metrics-supported.md#microsoftnetworkprivateendpoints) |
|Microsoft.Network/privateLinkServices| Нет | Нет | [Службы частной связи](../essentials/metrics-supported.md#microsoftnetworkprivatelinkservices) |
|Microsoft.Network/publicipaddresses; | Нет | Нет | [общедоступные IP-адреса](../essentials/metrics-supported.md#microsoftnetworkpublicipaddresses)|
|Microsoft.Network/trafficManagerProfiles | Да | Нет | [Профили диспетчера трафика](../essentials/metrics-supported.md#microsoftnetworktrafficmanagerprofiles) |
|Microsoft.OperationalInsights/workspaces| Да | Нет | [Рабочие области Log Analytics](../essentials/metrics-supported.md#microsoftoperationalinsightsworkspaces)|
|Microsoft. пиринг/пиринг | Да | Нет | [Пиринги](../essentials/metrics-supported.md#microsoftpeeringpeerings) |
|Microsoft. пиринг/Пирингсервицес | Да | Нет | [Службы пиринга](../essentials/metrics-supported.md#microsoftpeeringpeeringservices) |
|Microsoft.PowerBIDedicated/capacities | Нет | Нет | [Производительность](../essentials/metrics-supported.md#microsoftpowerbidedicatedcapacities) |
|Microsoft.Relay/namespaces | Да | Нет | [Ретрансляторы](../essentials/metrics-supported.md#microsoftrelaynamespaces) |
|Microsoft.Search/searchServices | Нет | Нет | [Службы поиска](../essentials/metrics-supported.md#microsoftsearchsearchservices) |
|Microsoft.ServiceBus/namespaces | Да | Нет | [Служебная шина](../essentials/metrics-supported.md#microsoftservicebusnamespaces) |
|Microsoft.Sql/managedInstances | Нет | Да | [Управляемые экземпляры SQL](../essentials/metrics-supported.md#microsoftsqlmanagedinstances) |
|Microsoft.Sql/servers/databases | Нет | Да | [Базы данных SQL](../essentials/metrics-supported.md#microsoftsqlserversdatabases) |
|Microsoft.Sql/servers/elasticPools | Нет | Да | [Эластичные пулы SQL](../essentials/metrics-supported.md#microsoftsqlserverselasticpools) |
|Microsoft.Storage/storageAccounts |Да | Нет | [Учетные записи хранения](../essentials/metrics-supported.md#microsoftstoragestorageaccounts)|
|Microsoft.Storage/storageAccounts/blobServices | Да| Нет | [Учетные записи хранения — BLOB-объекты](../essentials/metrics-supported.md#microsoftstoragestorageaccountsblobservices) |
|Microsoft.Storage/storageAccounts/fileServices | Да| Нет | [Учетные записи хранения — файлы](../essentials/metrics-supported.md#microsoftstoragestorageaccountsfileservices) |
|Microsoft.Storage/storageAccounts/queueServices | Да| Нет | [Учетные записи хранения — очереди](../essentials/metrics-supported.md#microsoftstoragestorageaccountsqueueservices) |
|Microsoft.Storage/storageAccounts/tableServices | Да| Нет | [Учетные записи хранения — таблицы](../essentials/metrics-supported.md#microsoftstoragestorageaccountstableservices) |
|Microsoft.StorageCache/caches | Да | Нет | [Кэши HPC](../essentials/metrics-supported.md#microsoftstoragecachecaches) |
|Microsoft. StorageSync/Сторажесинксервицес | Да | Нет | [Службы синхронизации хранилища](../essentials/metrics-supported.md#microsoftstoragesyncstoragesyncservices) |
|Microsoft.StreamAnalytics/streamingjobs | Да | Нет | [Stream Analytics](../essentials/metrics-supported.md#microsoftstreamanalyticsstreamingjobs) |
|Microsoft.Synapse/workspaces | Да | Нет | [Synapse Analytics](../essentials/metrics-supported.md#microsoftsynapseworkspaces) |
|Microsoft. синапсе/workspaces/Бигдатапулс | Да | Нет | [Пулы Apache Spark аналитики синапсе](../essentials/metrics-supported.md#microsoftsynapseworkspacesbigdatapools) |
|Microsoft. синапсе/workspaces/Склпулс | Да | Нет | [Пулы SQL Analytics синапсе](../essentials/metrics-supported.md#microsoftsynapseworkspacessqlpools) |
|Microsoft. Вмвареклаудсимпле/virtualMachines | Да | Нет | [Виртуальные машины CloudSimple](../essentials/metrics-supported.md#microsoftvmwarecloudsimplevirtualmachines) |
|Microsoft.Web/hostingEnvironments/multiRolePools | Да | Нет | [Среда службы приложений пулов с несколькими ролями](../essentials/metrics-supported.md#microsoftwebhostingenvironmentsmultirolepools)|
|Microsoft.Web/hostingEnvironments/workerPools | Да | Нет | [Пулы рабочих ролей Среда службы приложений](../essentials/metrics-supported.md#microsoftwebhostingenvironmentsworkerpools)|
|Microsoft.Web/serverfarms | Да | Нет | [Планы службы приложений](../essentials/metrics-supported.md#microsoftwebserverfarms)|
|Microsoft.Web/sites | Да | Нет | [Службы приложений и Функции](../essentials/metrics-supported.md#microsoftwebsites)|
|Microsoft.Web/sites/slots | Да | Нет | [Слоты Службы приложений](../essentials/metrics-supported.md#microsoftwebsitesslots)|

<sup>1</sup> не поддерживается для метрик сети виртуальной машины (общая сеть, общий объем сети, входящие потоки, исходящие потоки, максимальная частота создания входящих потоков, максимальная скорость создания исходящих потоков) и пользовательские метрики.

## <a name="payload-schema"></a>Схема полезных данных

> [!NOTE]
> Вы также можете использовать [общую схему предупреждений](./alerts-common-schema.md), которая предоставляет преимущества единого расширяемого и унифицированного набора полезных данных оповещений во всех службах предупреждений в Azure Monitor для интеграции веб-перехватчика. [Сведения об общих определениях схемы предупреждений.](./alerts-common-schema-definitions.md)


Если используется соответствующим образом настроенная [группа действий](./action-groups.md), то операция POST содержит следующие полезные данные и схему JSON для всех новых оповещений на основе метрик:

```json
{
  "schemaId": "AzureMonitorMetricAlert",
  "data": {
    "version": "2.0",
    "status": "Activated",
    "context": {
      "timestamp": "2018-02-28T10:44:10.1714014Z",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Contoso/providers/microsoft.insights/metricAlerts/StorageCheck",
      "name": "StorageCheck",
      "description": "",
      "conditionType": "SingleResourceMultipleMetricCriteria",
      "severity":"3",
      "condition": {
        "windowSize": "PT5M",
        "allOf": [
          {
            "metricName": "Transactions",
            "metricNamespace":"microsoft.storage/storageAccounts",
            "dimensions": [
              {
                "name": "AccountResourceId",
                "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Contoso/providers/Microsoft.Storage/storageAccounts/diag500"
              },
              {
                "name": "GeoType",
                "value": "Primary"
              }
            ],
            "operator": "GreaterThan",
            "threshold": "0",
            "timeAggregation": "PT5M",
            "metricValue": 1
          }
        ]
      },
      "subscriptionId": "00000000-0000-0000-0000-000000000000",
      "resourceGroupName": "Contoso",
      "resourceName": "diag500",
      "resourceType": "Microsoft.Storage/storageAccounts",
      "resourceId": "/subscriptions/1e3ff1c0-771a-4119-a03b-be82a51e232d/resourceGroups/Contoso/providers/Microsoft.Storage/storageAccounts/diag500",
      "portalLink": "https://portal.azure.com/#resource//subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Contoso/providers/Microsoft.Storage/storageAccounts/diag500"
    },
    "properties": {
      "key1": "value1",
      "key2": "value2"
    }
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Получите дополнительные сведения об [интерфейсе оповещений](./alerts-overview.md).
* Ознакомьтесь со сведениями об [оповещениях журналов в Azure](./alerts-unified-log.md).
* Сведения об [оповещениях в Azure](./alerts-overview.md).
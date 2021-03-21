---
title: Поддерживаемые типы ресурсов в службе работоспособности ресурсов Azure | Документация Майкрософт
description: Поддерживаемые типы ресурсов в службе работоспособности ресурсов Azure
ms.topic: conceptual
ms.date: 01/29/2019
ms.openlocfilehash: 1b0958732a96786b2fb86b2b145f5cb8d36c0c5a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97883373"
---
# <a name="resource-types-and-health-checks-in-azure-resource-health"></a>Типы ресурсов и проверки работоспособности в службе работоспособности ресурсов Azure
Ниже приведен полный список проверок, выполняемых в системе работоспособности ресурсов для разных типов ресурсов.

## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers
|Выполняемые проверки|
|---|
|<ul><li>Сервер запущен и работает?</li><li>На сервере недостаточно памяти?</li><li>Сервер запускается?</li><li>Сервер восстанавливается?</li></ul>|

## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service
|Выполняемые проверки|
|---|
|<ul><li>Служба Api Management запущена и работает?</li></ul>|

## <a name="microsoftappplatformspring"></a>Microsoft.AppPlatform/Spring
|Выполняемые проверки|
|---|
|<ul><li>Доступен ли экземпляр облака Azure весны?</li></ul>|

## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts
|Выполняемые проверки|
|---|
|<ul><li>Работает ли учетная запись пакетной службы?</li><li>Не превышена ли квота пула для этой учетной записи пакетной службы?</li></ul>|

## <a name="microsoftcacheredis"></a>Microsoft.Cache/Redis;
|Выполняемые проверки|
|---|
|<ul><li>Все ли узлы кэша запущены и работают?</li><li>Доступен ли кэш из центра обработки данных?</li><li>Достигнуто ли для кэша максимальное число подключений?</li><li> Исчерпан ли лимит доступной памяти кэша? </li><li>Возникает ли в кэше большое число ошибок страниц?</li><li>Работает ли кэш с повышенной загрузкой?</li></ul>|

## <a name="microsoftcdnprofile"></a>Microsoft.CDN/profile
|Выполняемые проверки|
|---|
|<ul> <li>Доступен ли дополнительный портал для операций настройки CDN?</li><li>Существуют ли проблемы с доставкой между конечными точками CDN?</li><li>Могут ли пользователи изменять настройки ресурсов CDN?</li><li>Изменения конфигурации публикуются с ожидаемой скоростью?</li><li>Могут ли пользователи управлять конфигурацией CDN с помощью портала Azure, PowerShell или API?</li> </ul>|

## <a name="microsoftclassiccomputevirtualmachines"></a>Microsoft.classiccompute/virtualmachines
|Выполняемые проверки|
|---|
|<ul><li>Сервер узла запущен и работает?</li><li>Завершена ли загрузка ОС на узле?</li><li>Контейнер виртуальной машины подготовлен и включен?</li><li>Есть ли сетевое подключение между узлом и учетной записи хранения?</li><li>Завершена ли загрузка гостевой операционной системы?</li><li>Есть ли назначенные работы по техническому обслуживанию?</li><li>Ухудшается ли работоспособность оборудования узла и прогнозируется ли его сбой в ближайшее время?</li></ul>|

## <a name="microsoftclassiccomputedomainnames"></a>Microsoft. classiccompute/имя_домена
|Выполняемые проверки|
|---|
|<ul><li>Является ли развертывание производственного слота работоспособным для всех экземпляров роли?</li><li>Является ли роль работоспособной для всех экземпляров виртуальных машин?</li><li>Каково состояние работоспособности каждой виртуальной машины в роли облачной службы?</li><li>Изменилось ли состояние виртуальной машины из-за операции, инициированной платформой или клиентом?</li><li>Завершена ли загрузка гостевой операционной системы?</li><li>Есть ли назначенные работы по техническому обслуживанию?</li><li>Ухудшается ли работоспособность оборудования узла и прогнозируется ли его сбой в ближайшее время?</li><li>Дополнительные [сведения](../cloud-services/resource-health-for-cloud-services.md) о выполненных проверках</li></ul>|

## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.cognitiveservices/accounts
|Выполняемые проверки|
|---|
|<ul><li>Доступна ли учетная запись из центра обработки данных?</li><li>Доступен ли поставщик ресурсов Cognitive Services?</li><li>Доступна ли служба Cognitive Services из соответствующего региона?</li><li>Успешно ли выполняются операции чтения для учетной записи хранения, в которой хранятся метаданные ресурсов?</li><li>Достигнута ли квота на вызовы API?</li><li>Достигнут ли лимит чтения на вызовы API?</li></ul>|

## <a name="microsoftcomputehostgroupshosts"></a>Microsoft.compute/hostgroups/hosts
|Выполняемые проверки|
|---|
|<ul><li>Работает ли узел?</li><li>Ухудшается ли работоспособность оборудования узла?</li><li>Не отменено ли выделение узла?</li><li>Не выполняла ли служба оборудования узла восстановление на другое оборудование?</li></ul>|

## <a name="microsoftcomputevirtualmachines"></a>Microsoft.compute/virtualmachines
|Выполняемые проверки|
|---|
|<ul><li>Сервер, размещающий эту виртуальную машину, включен и работает?</li><li>Завершена ли загрузка ОС на узле?</li><li>Контейнер виртуальной машины подготовлен и включен?</li><li>Есть ли сетевое подключение между узлом и учетной записи хранения?</li><li>Завершена ли загрузка гостевой операционной системы?</li><li>Есть ли назначенные работы по техническому обслуживанию?</li><li>Ухудшается ли работоспособность оборудования узла и прогнозируется ли его сбой в ближайшее время?</li></ul>|

## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters
|Выполняемые проверки|
|---|
|<ul><li>Работает ли кластер?</li><li>Доступны ли в кластере основные службы?</li><li>Все ли узлы кластера готовы?</li><li>Является ли субъект-служба текущим и действительным?</li></ul>|

## <a name="microsoftdatafactoryfactories"></a>Microsoft.datafactory/factories
|Выполняемые проверки|
|---|
|<ul><li>Есть ли ошибки выполнения конвейеров?</li><li>Является ли работоспособным кластер, на котором размещена Фабрика данных?</li></ul>|

## <a name="microsoftdatalakeanalyticsaccounts"></a>Microsoft.datalakeanalytics/accounts
|Выполняемые проверки|
|---|
|<ul><li>У пользователей возникали проблемы с отправкой или размещением заданий Data Lake Analytics?</li><li>Возможно ли, что задания Data Lake Analytics не выполняются из-за системных ошибок?</li></ul>|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.datalakestore/accounts
|Выполняемые проверки|
|---|
|<ul><li>У пользователей возникали проблемы с отправкой данных в Data Lake Store?</li><li>У пользователей возникали проблемы с загрузкой данных из Data Lake Store?</li></ul>|

## <a name="microsoftdatamigrationservices"></a>Microsoft.datamigration/services
|Выполняемые проверки|
|---|
|<ul><li>Возможно, не удалось подготовить службу переноса базы данных?</li><li>Возможно, служба переноса базы данных остановлена из-за бездействия или по запросу пользователя?</li></ul>|

## <a name="microsoftdatashareaccounts"></a>Microsoft.DataShare/accounts
|Выполняемые проверки|
|---|
|<ul><li>Работает ли учетная запись Data Share?</li><li>Является ли работоспособным кластер, на котором размещена служба Data Share?</li></ul>|

## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers
|Выполняемые проверки|
|---|
|<ul><li>Возможно, сервер временно недоступен из-за операций обслуживания?</li><li>Возможно, сервер временно недоступен из-за перенастройки?</li></ul>|

## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers
|Выполняемые проверки|
|---|
|<ul><li>Возможно, сервер временно недоступен из-за операций обслуживания?</li><li>Возможно, сервер временно недоступен из-за перенастройки?</li></ul>|

## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers
|Выполняемые проверки|
|---|
|<ul><li>Возможно, сервер временно недоступен из-за операций обслуживания?</li><li>Возможно, сервер временно недоступен из-за перенастройки?</li></ul>|

## <a name="microsoftdevicesiothubs"></a>Microsoft.devices/iothubs
|Выполняемые проверки|
|---|
|<ul><li>Работает ли Центр Интернета вещей?</li></ul>|

## <a name="microsoftdigitaltwinsdigitaltwinsinstances"></a>Microsoft. Дигиталтвинс/Дигиталтвинсинстанцес
|Выполняемые проверки|
|---|
|<ul><li>Работает ли экземпляр Azure Digital двойников?</li></ul>|

## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.documentdb/databaseAccounts
|Выполняемые проверки|
|---|
|<ul><li>Существуют ли запросы к базам данных или коллекциям, которые не были выполнены из-за недоступности службы Azure Cosmos DB?</li><li>Существуют ли запросы к документам, которые не были выполнены из-за недоступности службы Azure Cosmos DB?</li></ul>|

## <a name="microsofteventhubnamespaces"></a>Microsoft.eventhub/namespaces
|Выполняемые проверки|
|---|
|<ul><li>Возникают ли в пространстве имен службы "Центры событий" ошибки, вызванные пользователем?</li><li>Возможно, пространство имен службы "Центры событий" сейчас обновляется?</li></ul>|

## <a name="microsofthdinsightclusters"></a>Microsoft.hdinsight/clusters
|Выполняемые проверки|
|---|
|<ul><li>Доступны ли базовые службы в кластере HDInsight?</li><li>Может ли кластер HDInsight обращаться к ключу для шифрования BYOK в хранилище?</li></ul>|

## <a name="microsoftiotcentraliotapps"></a>Microsoft.IoTCentral/IoTApps.
|Выполняемые проверки|
|---|
|<ul><li>Доступно ли IoT Central приложение?</li></ul>|

## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults
|Выполняемые проверки|
|---|
|<ul><li>Запросы в хранилище ключей завершались сбоем из-за проблем с платформой Azure Key Vault?</li><li>Для запросов в хранилище ключей требовалось регулирование из-за слишком большого количества запросов от клиента?</li></ul>|

## <a name="microsoftkustoclusters"></a>Microsoft. Kusto/кластеры
|Выполняемые проверки|
|---|
|<ul><li>Работает ли кластер с низкой скоростью приема?</li><li>Кластер испытывает высокую задержку приема?</li><li>Возникает ли в кластере большое количество ошибок запросов?</li></ul>|

## <a name="microsoftmachinelearningwebservices"></a>Microsoft.MachineLearning/webServices
|Выполняемые проверки|
|---|
|<ul><li>Работает ли веб-служба?</li></ul>|

## <a name="microsoftmediamediaservices"></a>Microsoft.Media/mediaservices
|Выполняемые проверки|
|---|
|<ul><li>Работает ли служба мультимедиа?</li></ul>|

## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.network/applicationgateways
|Выполняемые проверки|
|---|
|<ul><li>Не ухудшается ли работоспособность Шлюза приложений?</li><li>Доступен ли Шлюз приложений?</li></ul>|

## <a name="microsoftnetworkbastionhosts"></a>Microsoft. Network/бастионхостс
|Выполняемые проверки|
|---|
|<ul><li>Работает ли узел бастиона?</li></ul>|

## <a name="microsoftnetworkconnections"></a>Microsoft.network/connections
|Выполняемые проверки|
|---|
|<ul><li>Подключен ли VPN-туннель?</li><li>Существуют ли конфликты конфигурации для соединения?</li><li>Правильно ли настроены предварительные общие ключи?</li><li>Доступно ли VPN-устройство в локальной среде?</li><li>Есть ли несоответствия в политике безопасности IPSec/IKE?</li><li>Правильно ли подготовлено подключение S2S VPN или оно находится в состоянии сбоя?</li><li>Правильно ли подготовлено подключение между виртуальными сетями или оно находится в состоянии сбоя?</li></ul>|

## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.network/expressroutecircuits
|Выполняемые проверки|
|---|
|<ul><li>Работоспособен ли канал ExpressRoute?</li></ul>|

## <a name="microsoftnetworkfrontdoors"></a>Microsoft.network/frontdoors
|Выполняемые проверки|
|---|
|<ul><li>Возвращают ли некоторые внутренние серверы Front Door ошибки при проверке работоспособности?</li><li>Существуют ли отложенные изменения в конфигурации?</li></ul>|

## <a name="microsoftnetworkloadbalancers"></a>Microsoft.network/LoadBalancers
|Выполняемые проверки|
|---|
|<ul><li>Доступны ли конечные точки балансировки нагрузки?</li></ul>|

## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft. Network/trafficmanagerprofiles
|Выполняемые проверки|
|---|
|<ul><li>Какие проблемы влияют на профиль диспетчера трафика?</li></ul>|

## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.network/virtualNetworkGateways
|Выполняемые проверки|
|---|
|<ul><li>Доступен ли VPN-шлюз из Интернета?</li><li>Находится ли VPN-шлюз в режиме ожидания?</li><li>Запущена ли служба VPN на шлюзе?</li></ul>|

## <a name="microsoftnotificationhubsnamespace"></a>Microsoft.NotificationHubs/namespace
|Выполняемые проверки|
|---|
|<ul><li>Успешно ли выполняются для пространства имен операции времени выполнения, такие как регистрации, установки и отправки?</li></ul>|

## <a name="microsoftoperationalinsightsworkspaces"></a>Microsoft.operationalinsights/workspaces
|Выполняемые проверки|
|---|
|<ul><li>Возникают ли задержки индексации для рабочей области?</li></ul>|

## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/Capacities
|Выполняемые проверки|
|---|
|<ul><li>Запущен и работает ли ресурс производительности?</li><li>Все ли рабочие нагрузки запущены и работают?</li></ul>

## <a name="microsoftsearchsearchservices"></a>Microsoft.search/searchServices
|Выполняемые проверки|
|---|
|<ul><li>Успешно ли выполняются операции диагностики для кластера?</li></ul>|

## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces
|Выполняемые проверки|
|---|
|<ul><li>Сталкиваются ли клиенты с ошибками, связанными со Служебной шиной?</li><li>Возникает ли у пользователей увеличение переходных ошибок из-за обновления пространства имен Служебной шины?</li></ul>|

## <a name="microsoftservicefabricclusters"></a>Microsoft.ServiceFabric/clusters
|Выполняемые проверки|
|---|
|<ul><li>Работает ли кластер Service Fabric?</li><li>Возможно ли управление кластером Service Fabric через Azure Resource Manager?</li></ul>|

## <a name="microsoftsqlmanagedinstancesdatabases"></a>Microsoft.SQL/managedInstances/databases
|Выполняемые проверки|
|---|
|<ul><li>Работает ли база данных?</li></ul>|

## <a name="microsoftsqlserversdatabases"></a>Microsoft.SQL/servers/databases
|Выполняемые проверки|
|---|
|<ul><li>Выполнялись ли входы в базу данных?</li></ul>|

## <a name="microsoftstoragestorageaccounts"></a>Microsoft.Storage/storageAccounts
|Выполняемые проверки|
|---|
|<ul><li>Запросы на чтение данных из учетной записи хранения завершались сбоем из-за проблем с платформой Azure Storage?</li><li>Запросы на запись данных в учетную запись хранения завершались сбоем из-за проблем с платформой Azure Storage?</li><li>Кластер хранилищ, в котором размещена учетная запись хранения, недоступен?</li></ul>|

## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs
|Выполняемые проверки|
|---|
|<ul><li>Все ли узлы, на которых выполняются задания, успешно работают?</li><li>Существуют ли задания, которые не удалось запустить?</li><li>Существуют ли запланированные обновления среды выполнения?</li><li>Находится ли задание в запланированном состоянии (запущено или остановлено клиентом)?</li><li>Возникали ли исключения нехватки памяти при выполнении задания?</li><li>Существуют ли запланированные обновления вычислительной среды?</li><li>Доступен ли диспетчер выполнения (план управления)?</li></ul>|

## <a name="microsoftwebserverfarms"></a>Microsoft.web/serverFarms
|Выполняемые проверки|
|---|
|<ul><li>Сервер узла запущен и работает?</li><li>Работает ли служба Internet Information Services?</li><li>Работает ли подсистема балансировки нагрузки?</li><li>Доступен ли план службы приложений из центра обработки данных?</li><li>Доступна ли учетная запись хранилища, в которой размещается содержимое узлов для фермы серверов?</li></ul>|

## <a name="microsoftwebsites"></a>Microsoft.web/sites
|Выполняемые проверки|
|---|
|<ul><li>Сервер узла запущен и работает?</li><li>Работает ли сервер Internet Information?</li><li>Работает ли подсистема балансировки нагрузки?</li><li>Доступно ли веб-приложение из центра обработки данных?</li><li>Доступна ли учетная запись хранения, в которой размещается контент сайта?</li></ul>|

## <a name="next-steps"></a>Next Steps
-  Дополнительные сведения см. в [общих сведениях о панели мониторинга работоспособности службы Azure](service-health-overview.md) и [общих сведениях о службе "Работоспособность ресурсов Azure"](resource-health-overview.md). 
-  [Azure Resource Health FAQ](resource-health-faq.md) (Часто задаваемые вопросы о службе работоспособности ресурсов Azure)
- Настройте оповещения о проблемах, связанных с работоспособностью. Дополнительные сведения см. в статье [Создание оповещений журнала действий для уведомлений службы](./alerts-activity-log-service-notifications-portal.md).
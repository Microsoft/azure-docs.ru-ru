---
title: Поддерживаемые службы и схемы в журналах ресурсов Azure
description: Узнайте о поддерживаемых службах и схеме событий для журналов ресурсов Azure.
ms.topic: reference
ms.date: 09/01/2020
ms.openlocfilehash: 661cba2d78dbb176bb04e6831cf8e045a285535e
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102033101"
---
# <a name="common-and-service-specific-schema-for-azure-resource-logs"></a>Распространенная и зависящая от службы схема для журналов ресурсов Azure

> [!NOTE]
> Журналы ресурсов ранее назывались журналами диагностики. Имя было изменено в октябре 2019, так как типы журналов, собранные Azure Monitor перемещены, чтобы включать больше, чем просто ресурс Azure. Кроме того, список категорий журналов ресурсов, которые можно было бы использовать для перечисления в этой статье. Они были перемещены в [Категории журнала ресурсов](resource-logs-categories.md). 

[Azure Monitor журналы ресурсов](../essentials/platform-logs-overview.md) — это журналы, создаваемые службами Azure, которые описывают работу этих служб или ресурсов. Все журналы ресурсов, доступные через Azure Monitor, используют общую схему верхнего уровня, обеспечивая гибкость для каждой службы, чтобы создавать уникальные свойства для собственных событий.

Сочетание типа ресурса (доступного в свойстве `resourceId`) и свойства `category` является уникальным идентификатором схемы. В этой статье описывается схема верхнего уровня для журналов ресурсов и ссылки на Schemata для каждой службы.


## <a name="top-level-common-schema"></a>Общая схема верхнего уровня

| Имя | Обязательный/необязательный | Описание |
|---|---|---|
| time | Обязательно | Метка времени события (UTC). |
| resourceId | Обязательно | Идентификатор ресурса, создавшего событие. Для служб клиента он имеет формат "/tenants/ИД_клиента/providers/имя_поставщика". |
| tenantId | Требуется для журналов клиента | Идентификатор клиента Active Directory клиента, к которому привязано это событие. Это свойство используется только для журналов уровня клиентов и не отображается в журналах уровня ресурсов. |
| operationName | Обязательно | Имя операции, которую представляет это событие. Если событие представляет операцию Azure RBAC, это имя операции RBAC Azure (например, Microsoft. Storage/storageAccounts/Блобсервицес/blobs/Read). Обычно моделируются в виде операции Resource Manager, даже если они фактически не являются задокументированными операциями Resource Manager (`Microsoft.<providerName>/<resourceType>/<subtype>/<Write/Read/Delete/Action>`) |
| operationVersion | Необязательно | Версия API, связанная с операцией, если Имя_операции была выполнена с помощью API (например, `http://myservice.windowsazure.net/object?api-version=2016-06-01` ). Если для этой операции не существует соответствующего API, то версия представляет версию этой операции, чтобы в дальнейшем изменить свойства, связанные с операцией. |
| категория | Обязательно | Категория журнала для события. Категория — степень детализации, при которой можно включать или отключать журналы для определенного ресурса. Свойства, которые отображаются в свойствах BLOB-объекта события, одинаковы в пределах определенной категории журнала и типа ресурса. Типичными категориями журналов являются "Аудит", "выполнение" и "запрос". |
| resultType | Необязательно | Состояние события. Обычные значения: "Запущен", "Выполняется", "Успешно", "Сбой", "Активно", "Разрешено". |
| resultSignature | Необязательно | Дополнительное состояние события. Если эта операция соответствует вызову REST API, то это поле содержит код состояния HTTP соответствующего вызова REST. |
| resultDescription | Необязательно | Статическое текстовое описание этой операции, например "получение файла хранилища". |
| durationMs | Необязательно | Время выполнения операции в миллисекундах. |
| callerIpAddress | Необязательно | IP-адрес вызывающего объекта, если операция соответствует вызову API, который будет приходить с объекта с общедоступным IP-адресом. |
| correlationId | Необязательно | Идентификатор GUID, используемый для формирования набора связанных событий. Как правило, если два события имеют одну и ту же Имя_операции, но два разных состояния (например, "запущено" и "успешно"), они используют один и тот же идентификатор корреляции. Идентификатор может также представлять другие связи между событиями. |
| удостоверение | Необязательно | BLOB-объект типа JSON, описывающий идентификацию пользователя или приложения, выполнившего операцию. Обычно это поле содержит маркер авторизации и утверждений/JWT из Active Directory. |
| Level | Необязательно | Уровень серьезности события. Должен быть одним из следующих значений: "Информационное", "Предупреждение", "Ошибка" или "Критическое". |
| location | Необязательно | Регион ресурса, на который порождается событие, например "Восточная часть США" или "Французская Южная" |
| properties | Необязательно | Любые расширенные свойства, связанные с этой конкретной категорией событий. Все пользовательские или уникальные свойства должны быть размещены внутри этой "части B" схемы. |

## <a name="service-specific-schemas"></a>Зависящие от службы схемы

Схема журналов ресурсов зависит от категории ресурса и журнала. В этом списке перечислены службы, которые делают доступными журналы ресурсов и ссылки на схему службы и конкретной категории, где это возможно. Этот список изменяется все время, когда добавляются новые службы, поэтому если вы не видите, что вам нужно, воспользуйтесь поисковой системой для поиска дополнительной документации. Вы можете открыть в этой статье ошибку GitHub, чтобы мы могли обновить ее.

| Служба | Схемы и документы |
| --- | --- |
| Azure Active Directory | [Общие сведения](../../active-directory/reports-monitoring/concept-activity-logs-azure-monitor.md), [схема журнала аудита](../../active-directory/reports-monitoring/reference-azure-monitor-audit-log-schema.md) и [схема входа](../../active-directory/reports-monitoring/reference-azure-monitor-sign-ins-log-schema.md) |
| Службы Analysis Services | [Azure Analysis Services — ведение журнала диагностики](../../analysis-services/analysis-services-logging.md) |
| Управление API | [Журналы ресурсов управления API](../../api-management/api-management-howto-use-azure-monitor.md#resource-logs) |
| Служба приложений | [Журналы службы приложений](../../app-service/troubleshoot-diagnostic-logs.md)
| Шлюзы приложений |[Ведение журнала для шлюза приложений](../../application-gateway/application-gateway-diagnostics.md) |
| Служба автоматизации Azure |[Log Analytics для службы автоматизации Azure](../../automation/automation-manage-send-joblogs-log-analytics.md) |
| Пакетная служба Azure |[Ведение журнала пакетной службы Azure](../../batch/batch-diagnostics.md) |
| Cognitive Services | [Ведение журнала для Cognitive Services Azure](../../cognitive-services/diagnostic-logging.md) |
| Реестр контейнеров | [Ведение журнала для реестра контейнеров Azure](../../container-registry/container-registry-diagnostics-audit-logs.md) |
| Сеть доставки содержимого (CDN) | [Журналы Azure для CDN](../../cdn/cdn-azure-diagnostic-logs.md) |
| Cosmos DB | [Журнал ведения диагностики Azure Cosmos DB](../../cosmos-db/monitor-cosmos-db.md) |
| Фабрика данных | [Мониторинг фабрик данных с помощью Azure Monitor](../../data-factory/monitor-using-azure-monitor.md) |
| Data Lake Analytics |[Доступ к журналам для Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-diagnostic-logs.md) |
| Data Lake Storage |[Доступ к журналам для Azure Data Lake Store](../../data-lake-store/data-lake-store-diagnostic-logs.md) |
| Azure Data Explorer | [Журналы обозреватель данных Azure](/azure/data-explorer/using-diagnostic-logs) |
| База данных Azure для MySQL | [Журналы сервера в базе данных Azure для MySQL](../../mysql/concepts-server-logs.md#diagnostic-logs) |
| База данных Azure для PostgreSQL | [Журналы базы данных Azure для PostgreSQL](../../postgresql/concepts-server-logs.md#resource-logs) |
| Azure Databricks | [Журнал ведения диагностики в Azure Databricks](/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs) |
| Azure Digital Twins | [Настройка диагностики Azure Digital двойников](../../digital-twins/troubleshoot-diagnostics.md#log-schemas)
| Центры событий |[Журналы концентраторов событий Azure](../../event-hubs/event-hubs-diagnostic-logs.md) |
| ExpressRoute | Схема недоступна. |
| Брандмауэр Azure | Схема недоступна. |
| Front Door | [Ведение журнала для передней дверцы](../../frontdoor/front-door-diagnostics.md) |
| Центр Интернета вещей | [Использование Azure Monitor](../../iot-hub/monitor-iot-hub-reference.md#resource-logs) |
| Key Vault |[Ведение журнала хранилища ключей Azure](../../key-vault/general/logging.md) |
| Служба Kubernetes |[Ведение журнала Kubernetes для Azure](../../aks/view-control-plane-logs.md#log-event-schema) |
| Load Balancer |[Служба анализа журналов для балансировщика нагрузки Azure](../../load-balancer/load-balancer-monitor-log.md) |
| Logic Apps |[Настраиваемая схема отслеживания Logic Apps B2B](../../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md) |
| группы сетевой безопасности; |[Аналитика журналов для групп безопасности сети](../../virtual-network/virtual-network-nsg-manage-log.md) |
| Защита от атак DDoS | [Ведение журнала для стандарта защиты Azure от атак DDoS Standard](../../ddos-protection/diagnostic-logging.md#log-schemas) |
| Power BI (цен. категория "Выделенный") | [Ведение журнала для Power BI Embedded в Azure](/power-bi/developer/azure-pbie-diag-logs) |
| Службы восстановления | [Модель данных для Azure Backup](../../backup/backup-azure-reports-data-model.md)|
| Поиск |[Включение и использование аналитики поискового трафика](../../search/search-traffic-analytics.md) |
| Служебная шина Azure |[Журналы служебной шины Azure](../../service-bus-messaging/service-bus-diagnostic-logs.md) |
| База данных SQL | [Ведение журнала базы данных SQL Azure](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md) |
| Stream Analytics |[Журналы заданий](../../stream-analytics/stream-analytics-job-diagnostic-logs.md) |
| Диспетчер трафика | [Схема журнала диспетчера трафика](../../traffic-manager/traffic-manager-diagnostic-logs.md) |
| Виртуальные сети | Схема недоступна. |
| Шлюзы виртуальной сети | Схема недоступна. |



## <a name="next-steps"></a>Next Steps

* [Просмотр категорий журналов ресурсов, которые можно собираются](resource-logs-categories.md)
* [Дополнительные сведения о журналах ресурсов](../essentials/platform-logs-overview.md)
* [Потоковая передача журналов ресурсов в **концентраторы событий**](./resource-logs.md#send-to-azure-event-hubs)
* [Изменение параметров диагностики журнала ресурсов с помощью Azure Monitor REST API](/rest/api/monitor/diagnosticsettings)
* [Сбор журналов и метрик для служб Azure для использования в Log Analytics](./resource-logs.md#send-to-log-analytics-workspace)
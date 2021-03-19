---
title: Журналы — база данных Azure для PostgreSQL — гибкий сервер
description: Описание конфигурации ведения журнала, хранилища и анализа в базе данных Azure для PostgreSQL-гибкого сервера
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 852bce26e348c817b609d5ff837641741afe4461
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90940391"
---
# <a name="logs-in-azure-database-for-postgresql---flexible-server"></a>Журналы в базе данных Azure для PostgreSQL-гибкого сервера

> [!IMPORTANT]
> Гибкий сервер Базы данных Azure для PostgreSQL предоставляется в режиме предварительной версии

База данных Azure для PostgreSQL позволяет настраивать и получать доступ к стандартным журналам postgres. Журналы могут использоваться для обнаружения, устранения и исправления ошибок конфигурации и неоптимальной производительности. Сведения о ведении журнала, которые можно настроить и получить доступ, включают ошибки, сведения о запросах, записи автоочистки, подключения и контрольные точки. (Доступ к журналам транзакций недоступен).

Ведение журнала аудита предоставляется через расширение postgres `pgaudit` . Дополнительные сведения см. в статье [Основные понятия аудита](concepts-audit.md) .

## <a name="configure-logging"></a>Настройка журнала

Вы можете настроить стандартное ведение журнала Postgres на сервере с помощью параметров сервера ведения журнала. Дополнительные сведения о параметрах журнала postgres см. в разделах о [времени](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHEN) и [журналах](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHAT) документации по postgres. Большинство, но не все параметры ведения журнала postgres доступны для настройки в базе данных Azure для PostgreSQL.

Чтобы узнать, как настроить параметры в базе данных Azure для PostgreSQL, см. [документацию по порталу](howto-configure-server-parameters-using-portal.md) или [документации по CLI](howto-configure-server-parameters-using-cli.md).

> [!NOTE]
> Настройка большого объема журналов, например ведение журнала инструкций, может значительно увеличить производительность. 

## <a name="accessing-logs"></a>Доступ к журналам

База данных Azure для PostgreSQL интегрирована с параметрами диагностики Azure Monitor. Параметры диагностики позволяют отправлять журналы postgres в формате JSON, чтобы Azure Monitor журналы для аналитики и оповещений, концентраторов событий для потоковой передачи и службу хранилища Azure для архивации. 

### <a name="log-format"></a>Формат журнала

В следующей таблице описаны поля для типа **постгрескллогс** . Порядок появления выбранных полей зависит от выбранной конечной точки вывода. 

|**Поле** | **Описание** |
|---|---|
| TenantId | Идентификатор клиента |
| SourceSystem | `Azure` |
| TimeGenerated [UTC] | Метка времени, когда журнал был записан в формате UTC |
| Type | Тип журнала Всегда `AzureDiagnostics`. |
| SubscriptionId | Идентификатор GUID для подписки, принадлежащей серверу |
| ResourceGroup | Имя группы ресурсов, принадлежащей серверу |
| ResourceProvider | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORPOSTGRESQL`. |
| ResourceType | `Servers` |
| ResourceId | Универсальный код ресурса (URI) |
| Ресурс | Имя сервера |
| Category | `PostgreSQLLogs` |
| OperationName | `LogEvent` |
| errorLevel | Уровень ведения журнала, например: LOG, ERROR, NOTICE |
| Сообщение | Первичное сообщение журнала | 
| Домен | Версия сервера, например: postgres 10 |
| Подробный сведения | Второстепенное сообщение журнала (если применимо) |
| ColumnName | Имя столбца (если применимо) |
| SchemaName | Имя схемы (если применимо) |
| DatatypeName | Имя типа данных (если применимо) |
| LogicalServerName | Имя сервера | 
| _ResourceId | Универсальный код ресурса (URI) |
| Prefix | Префикс строки журнала |


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [настройке журналов и доступе к ним](howto-configure-and-access-logs.md).
- Дополнительные сведения о [ценах на Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).
- Дополнительные сведения о [журналах аудита](concepts-audit.md)

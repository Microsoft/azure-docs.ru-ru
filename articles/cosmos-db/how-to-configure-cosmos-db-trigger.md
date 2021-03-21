---
title: Триггер функций Azure для расширенной конфигурации Cosmos DB
description: Узнайте, как настроить ведение журнала и политику подключения, используемые триггером функций Azure для Cosmos DB
author: ealsur
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/19/2020
ms.author: maquaran
ms.openlocfilehash: 30328db465e0d9bf8c1ce67d92e48c688c51e043
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100574617"
---
# <a name="how-to-configure-logging-and-connectivity-with-the-azure-functions-trigger-for-cosmos-db"></a>Настройка ведения журнала и подключения с помощью триггера функций Azure для Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описываются дополнительные параметры конфигурации, которые можно задать при использовании триггера функций Azure для Cosmos DB.

## <a name="enabling-trigger-specific-logs"></a>Включение журналов, связанных с триггерами

При активации функций Azure для Cosmos DB для используется [библиотека обработчика канала изменений](./change-feed-processor.md) на внутреннем уровне. Библиотека создает набор журналов работоспособности, которые могут использоваться для мониторинга внутренних операций в целях [устранения неполадок](./troubleshoot-changefeed-functions.md).

В журналах работоспособности описывается, как ведет себя активация функций Azure для Cosmos DB при выполнении операций в сценариях балансировки нагрузки или инициализации.

### <a name="enabling-logging"></a>Включение ведения журналов

Чтобы включить ведение журналов при использовании активации функций Azure для Cosmos DB, найдите файл `host.json` в проекте Функций Azure или приложении Функций Azure и [настройте требуемый уровень ведения журнала](../azure-functions/functions-monitoring.md#log-levels-and-categories). Включите трассировки для  `Host.Triggers.CosmosDB` , как показано в следующем примере:

```js
{
  "version": "2.0",
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "Host.Triggers.CosmosDB": "Trace"
    }
  }
}
```

Когда будут развернуты Функции Azure с обновленной конфигурацией, журналы активации функций Azure для Cosmos DB отобразятся как часть трассировки. Журналы можно просмотреть в настроенном поставщике ведения журнала в разделе *Категории* `Host.Triggers.CosmosDB`.

### <a name="query-the-logs"></a>Выполнение запросов к журналам

Выполните следующий запрос к журналам, созданным с помощью активации функций Azure для Cosmos DB в [Overview of log queries in Azure Monitor](../azure-monitor/logs/log-query-overview.md) (Обзор запросов журнала в Azure Monitor):

```sql
traces
| where customDimensions.Category == "Host.Triggers.CosmosDB"
```

## <a name="configuring-the-connection-policy"></a>Настройка политики подключения

Существует два режима подключения: режим прямого подключения и режим шлюза. Дополнительные сведения об этих режимах подключения см. в статье [режимы подключения](sql-sdk-connection-modes.md) . По умолчанию для установки всех подключений на триггере Функций Azure для Cosmos DB используется **шлюз**. Тем не менее, это может быть не лучшим вариантом для сценариев, ориентированных на производительность.

### <a name="changing-the-connection-mode-and-protocol"></a>Изменение режима и протокола подключения

Для настройки политики подключения для клиента используются два основных параметра конфигурации: **режим** и **протокол** подключения. Вы можете изменить режим подключения по умолчанию и протокол, который используется триггером Функций Azure для Cosmos DB и всеми [привязками Azure Cosmos DB](../azure-functions/functions-bindings-cosmosdb-v2-output.md). Чтобы изменить параметры по умолчанию, вам потребуется найти `host.json` в своем проекте или приложении Функций Azure и добавить следующий [дополнительный параметр](../azure-functions/functions-bindings-cosmosdb-v2-output.md#hostjson-settings).

```js
{
  "cosmosDB": {
    "connectionMode": "Direct",
    "protocol": "Tcp"
  }
}
```

В `connectionMode` должен быть указан необходимый режим подключения (прямое подключение или шлюз), а в `protocol` — желаемый протокол подключения (Tcp или Https). 

Если проект Функций Azure работает со средой выполнения Функций Azure версии 1, в имени конфигурации есть небольшое различие. Вместо `cosmosDB` следует использовать `documentDB`.

```js
{
  "documentDB": {
    "connectionMode": "Direct",
    "protocol": "Tcp"
  }
}
```

> [!NOTE]
> При размещении приложения-функции в плане потребления каждый экземпляр имеет ограничение на количество соединений сокета, которое он может поддерживать. При работе с прямым или TCP-режимом при проектировании создается больше подключений, что может привести к [ограничению плана потребления](../azure-functions/manage-connections.md#connection-limit). в этом случае можно либо использовать режим шлюза, либо разместить приложение-функцию в [плане Premium](../azure-functions/functions-premium-plan.md) или [выделенном (службе приложений) плане](../azure-functions/dedicated-plan.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Ограничение числа подключений](../azure-functions/manage-connections.md#connection-limit)
* [Включение мониторинга](../azure-functions/functions-monitoring.md) в приложениях Функций Azure.
* Узнайте, как выполнять [диагностику и устранять неполадки при использовании активации функций Azure для Cosmos DB](./troubleshoot-changefeed-functions.md).
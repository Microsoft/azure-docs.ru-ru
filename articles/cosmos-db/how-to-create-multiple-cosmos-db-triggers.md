---
title: Создание нескольких независимых триггеров функций Azure для Cosmos DB
description: Из этой статьи вы узнаете, как настроить несколько независимых активаций функций Azure для Cosmos DB, чтобы создать архитектуры на основе событий.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 07/17/2019
ms.author: maquaran
ms.custom: devx-track-csharp
ms.openlocfilehash: 78fff48a97965f0b80456cd3e56ed1507bc784fc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93336696"
---
# <a name="create-multiple-azure-functions-triggers-for-cosmos-db"></a>Создание несколько активаций функций Azure для Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описывается, как настроить несколько активаций функций Azure для Cosmos DB, чтобы работать в параллельном режиме и независимого реагирования на изменения.

:::image type="content" source="./media/change-feed-functions/multi-trigger.png" alt-text="Бессерверные функции на основе событий, которые работают с активацией функций Azure для Cosmos DB и совместно используют контейнер аренды" border="false":::

## <a name="event-based-architecture-requirements"></a>Требования к архитектуре на основе событий

Если вы создаете бессерверные архитектуры с помощью решения [Функции Azure](../azure-functions/functions-overview.md), мы [рекомендуем](../azure-functions/functions-best-practices.md#avoid-long-running-functions) создавать небольшие наборы функций, которые работают совместно (а не крупные длительные функции).

Создавая бессерверные потоки на основе событий с помощью [активации функций Azure для Cosmos DB](./change-feed-functions.md), вы можете столкнуться с ситуацией, когда нужно выполнить несколько действий при появлении нового события в определенном [контейнере Azure Cosmos](./account-databases-containers-items.md#azure-cosmos-containers). Если действия, которые нужно активировать, не зависят друг от друга, лучше всего **создать по одной активации функций Azure для Cosmos DB на каждое действие**. Каждый из этих триггеров будет прослушивать изменения в одном и том же контейнере Azure Cosmos.

## <a name="optimizing-containers-for-multiple-triggers"></a>Оптимизация контейнеров для нескольких триггеров

Учитывая *требования* к активации функций Azure для Cosmos DB, нам потребуется второй контейнер для хранения состояния, который называется *контейнером аренды*. Означает ли это, что для каждой функции Azure требуется отдельный контейнер аренды?

В этом случае существует два подхода.

* Создайте **один контейнер аренды для каждой функции**: этот подход может быть переведен на дополнительные затраты, если только не используется [Общая база данных пропускной способности](./set-throughput.md#set-throughput-on-a-database). Помните, что минимальное значение пропускной способности на уровне контейнера равно 400 [единиц запросов](./request-units.md), а при наличии контейнера аренды она используется только для установки контрольных точек выполнения и поддержания состояния.
* У вас есть **один контейнер аренды и общий доступ к ним** для всех ваших функций. Этот второй вариант позволяет лучше использовать подготовленные единицы запросов в контейнере, так как позволяет нескольким функциям Azure совместно использовать одну подготовленную пропускную способность.

В этой статье подробно описывается реализация второго подхода.

## <a name="configuring-a-shared-leases-container"></a>Настройка общего контейнера аренды

Чтобы настроить общий контейнер аренды, вам нужно выполнить с триггерами одно дополнительное действие — добавить  [атрибут](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#attributes-and-annotations)`LeaseCollectionPrefix` при использовании C# или `leaseCollectionPrefix` [атрибут](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md) при использовании JavaScript. Значением атрибута должен быть логический дескриптор, описывающий, что собой представляет определенный триггер.

Например, у вас есть три триггера: один отправляет сообщения электронной почты, другой использует статистическую обработку для создания материализованного представления, а третий отправляет изменения в другое хранилище для последующего анализа. Вы можете назначить атрибуту `LeaseCollectionPrefix` значение emails для первого триггера, значение materialized для второго триггера, и значение analytics для третьего.

Важно то, что все три триггера **могут использовать одну и ту же конфигурацию контейнера аренды** (имя учетной записи, базы данных и контейнера).

Простой пример кода с использованием атрибута `LeaseCollectionPrefix` на C# будет выглядеть так:

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

[FunctionName("SendEmails")]
public static void SendEmails([CosmosDBTrigger(
    databaseName: "ToDoItems",
    collectionName: "Items",
    ConnectionStringSetting = "CosmosDBConnection",
    LeaseCollectionName = "leases",
    LeaseCollectionPrefix = "emails")]IReadOnlyList<Document> documents,
    ILogger log)
{
    ...
}

[FunctionName("MaterializedViews")]
public static void MaterializedViews([CosmosDBTrigger(
    databaseName: "ToDoItems",
    collectionName: "Items",
    ConnectionStringSetting = "CosmosDBConnection",
    LeaseCollectionName = "leases",
    LeaseCollectionPrefix = "materialized")]IReadOnlyList<Document> documents,
    ILogger log)
{
    ...
}
```

Для JavaScript вы можете применить конфигурацию с атрибутом `leaseCollectionPrefix` к файлу `function.json`:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "CosmosDBConnection",
    "databaseName": "ToDoItems",
    "collectionName": "Items",
    "leaseCollectionPrefix": "emails"
},
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "CosmosDBConnection",
    "databaseName": "ToDoItems",
    "collectionName": "Items",
    "leaseCollectionPrefix": "materialized"
}
```

> [!NOTE]
> Всегда отслеживайте использование единиц запросов, подготовленных на общем контейнере аренды. Каждый триггер, использующий этот контейнер, увеличит среднюю нагрузку на пропускную способность, поэтому при увеличении числа использующих ее функций Azure может потребоваться увеличить подготовленную пропускную способность.

## <a name="next-steps"></a>Дальнейшие действия

* См. полную конфигурацию [активации функций Azure для Cosmos DB](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration).
* Ознакомьтесь с расширенным [списком примеров](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md) для всех языков.
* Изучите другие примеры сценариев бессерверных вычислений с Функциями Azure и Azure Cosmos DB в [репозитории GitHub](https://github.com/ealsur/serverless-recipes/tree/master/cosmosdbtriggerscenarios).

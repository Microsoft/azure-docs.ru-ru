---
title: Запись хранимых процедур и триггеров с помощью API запросов JavaScript в Azure Cosmos DB
description: Узнайте, как записывать хранимые процедуры и триггеры с помощью API запросов JavaScript в Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/07/2020
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: 449af63ef3409b8c8173edef7a5d77074cca02b9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93335846"
---
# <a name="how-to-write-stored-procedures-and-triggers-in-azure-cosmos-db-by-using-the-javascript-query-api"></a>Как записывать хранимые процедуры и триггеры в Azure Cosmos DB с помощью API запросов JavaScript
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB позволяет создавать оптимизированные запросы с помощью гибкого интерфейса JavaScript (без знания языка SQL), который может использоваться для написания хранимых процедур или триггеров. Дополнительные сведения о поддержке API запросов JavaScript в Azure Cosmos DB см. в статье о [работе с API запросов с интегрированным языком JavaScript в Azure Cosmos DB](javascript-query-api.md).

## <a name="stored-procedure-using-the-javascript-query-api"></a><a id="stored-procedures"></a>Создание хранимой процедуры с использованием API запросов JavaScript

Ниже приведен пример кода, где хранимая процедура создается с использованием API запросов JavaScript. Хранимая процедура вставляет заданный входным параметром элемент Azure Cosmos, после чего обновляет документ метаданных с помощью метода `__.filter()`, а также на основе свойств размера входного элемента minSize, maxSize и totalSize.

> [!NOTE]
> `__` (двойное подчеркивание) является псевдонимом для `getContext().getCollection()` при использовании API запросов JavaScript.

```javascript
/**
 * Insert an item and update metadata doc: minSize, maxSize, totalSize based on item.size.
 */
function insertDocumentAndUpdateMetadata(item) {
  // HTTP error codes sent to our callback function by CosmosDB server.
  var ErrorCode = {
    RETRY_WITH: 449,
  }

  var isAccepted = __.createDocument(__.getSelfLink(), item, {}, function(err, item, options) {
    if (err) throw err;

    // Check the item (ignore items with invalid/zero size and metadata itself) and call updateMetadata.
    if (!item.isMetadata && item.size > 0) {
      // Get the metadata. We keep it in the same container. it's the only item that has .isMetadata = true.
      var result = __.filter(function(x) {
        return x.isMetadata === true
      }, function(err, feed, options) {
        if (err) throw err;

        // We assume that metadata item was pre-created and must exist when this script is called.
        if (!feed || !feed.length) throw new Error("Failed to find the metadata item.");

        // The metadata item.
        var metaItem = feed[0];

        // Update metaDoc.minSize:
        // for 1st document use doc.Size, for all the rest see if it's less than last min.
        if (metaItem.minSize == 0) metaItem.minSize = item.size;
        else metaItem.minSize = Math.min(metaItem.minSize, item.size);

        // Update metaItem.maxSize.
        metaItem.maxSize = Math.max(metaItem.maxSize, item.size);

        // Update metaItem.totalSize.
        metaItem.totalSize += item.size;

        // Update/replace the metadata item in the store.
        var isAccepted = __.replaceDocument(metaItem._self, metaItem, function(err) {
          if (err) throw err;
          // Note: in case concurrent updates causes conflict with ErrorCode.RETRY_WITH, we can't read the meta again
          //       and update again because due to Snapshot isolation we will read same exact version (we are in same transaction).
          //       We have to take care of that on the client side.
        });
        if (!isAccepted) throw new Error("replaceDocument(metaItem) returned false.");
      });
      if (!result.isAccepted) throw new Error("filter for metaItem returned false.");
    }
  });
  if (!isAccepted) throw new Error("createDocument(actual item) returned false.");
}
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о хранимых процедурах, триггерах и определяемых пользователем функциях в Azure Cosmos DB см. по следующим ссылкам:

* [Как работать с хранимыми процедурами, триггерами и определяемыми пользователем функциями в Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md)

* [Как зарегистрировать и использовать хранимые процедуры в Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#stored-procedures)

* Как зарегистрировать и использовать [триггеры предварительного](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) и [последовательного выполнения](how-to-use-stored-procedures-triggers-udfs.md#post-triggers) в Azure Cosmos DB

* [Как зарегистрировать и использовать определяемые пользователем функции в Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#udfs)

* [Искусственные ключи секций в Azure Cosmos DB](synthetic-partition-keys.md)

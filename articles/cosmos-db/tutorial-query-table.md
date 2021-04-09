---
title: Как выполнять запросы к данным таблиц в базе данных Azure Cosmos DB
description: Узнайте, как запрашивать данные, хранящиеся в учетной записи API таблиц Azure Cosmos DB, с помощью фильтров OData и запросов LINQ.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 06/05/2020
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: e184d85e3daee41f530334aa0034fc98f40a8766
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93099230"
---
# <a name="tutorial-query-azure-cosmos-db-by-using-the-table-api"></a>Руководство по Выполнение запросов в Azure Cosmos DB с использованием API таблиц
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[API таблицы](table-introduction.md) в службе Azure Cosmos DB поддерживает OData и запросы [LINQ](/rest/api/storageservices/fileservices/writing-linq-queries-against-the-table-service) к данным "ключ — значение" (таблицы).  

В этой статье рассматриваются следующие задачи:

> [!div class="checklist"]
> * Запрос данных с помощью API таблицы.

Запросы в этой статье используют следующий пример таблицы `People`:

| PartitionKey | RowKey | Email | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0101 |
| Смит | Ben | Ben@contoso.com| 425-555-0102 |
| Смит | Jeff | Jeff@contoso.com| 425-555-0104 |

Прочитайте раздел [База данных Azure Cosmos DB. Запрос табличных данных с помощью API таблицы](/rest/api/storageservices/fileservices/querying-tables-and-entities), чтобы получить подробные сведения о том, как отправить запрос с помощью API таблиц.

Дополнительные сведения о расширенных возможностях, предлагаемых базой данных Azure Cosmos DB, см. в статьях [Знакомство со службой Azure Cosmos DB. API таблицы](table-introduction.md) и [Разработка с помощью API таблицы базы данных Azure Cosmos DB на языке .NET](tutorial-develop-table-dotnet.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы эти запросы работали, у вас должна быть учетная запись базы данных Azure Cosmos DB и данные сущности в контейнере. У вас их нет? Выполните процедуры [краткого руководства](create-table-dotnet.md) или [руководства разработчика](tutorial-develop-table-dotnet.md), чтобы создать учетную запись и заполнить базу данных.

## <a name="query-on-partitionkey-and-rowkey"></a>Запросы PartitionKey и RowKey

Так как свойства PartitionKey и RowKey образуют первичный ключ сущности, вы можете использовать специальный синтаксис, чтобы идентифицировать сущность:

**Запрос**

```
https://<mytableendpoint>/People(PartitionKey='Harp',RowKey='Walter')  
```

**Результаты**

| PartitionKey | RowKey | Email | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0104 |

В качестве альтернативы вы можете указать эти свойства как часть параметра `$filter`, как показано в следующем разделе. Обратите внимание, что в именах ключевых свойств и значениях констант учитывается регистр. Свойства PartitionKey и RowKey имеют тип String.

## <a name="query-by-using-an-odata-filter"></a>Запрос с помощью фильтра OData

При построении строки фильтра помните о следующих правилах.

* Используйте логические операторы, определенные спецификацией протокола OData, чтобы сравнить свойство со значением. Обратите внимание, что нельзя сравнивать свойство с динамическим значением. Одна часть выражения должна быть константой.
* Имя свойства, оператор и значение константы должны быть разделены пробелами, закодированными в формате URL-адреса. Пробелы кодируются в формате URL-адреса как `%20`.
* Во всех частях строки фильтра учитывается регистр.
* Для получения допустимых результатов фильтра значения константы и свойства должны иметь одинаковый тип данных. Дополнительные сведения о поддерживаемых типах свойств см. в статье [Общие сведения о модели данных службы таблиц](/rest/api/storageservices/understanding-the-table-service-data-model).

Вот пример запроса, который показывает, как выполнить фильтрацию по свойствам PartitionKey и свойству Email с помощью `$filter` OData.

**Запрос**

```
https://<mytableapi-endpoint>/People()?$filter=PartitionKey%20eq%20'Smith'%20and%20Email%20eq%20'Ben@contoso.com'
```

Дополнительные сведения о том, как строить выражения фильтра для разных типов данных, см. в статье [Запрос таблиц и сущностей](/rest/api/storageservices/querying-tables-and-entities).

**Результаты**

| PartitionKey | RowKey | Email | PhoneNumber |
| --- | --- | --- | --- |
| Смит |Ben | Ben@contoso.com| 425-555-0102 |

Запросы к свойствам DateTime не возвращают данные при выполнении в API таблиц Azure Cosmos DB. Хотя в хранилище таблиц Azure хранятся значения даты с детализацией по времени тактов, API таблиц в Azure Cosmos DB использует свойство `_ts`. Свойство `_ts` находится на втором уровне детализации, который не является фильтром OData. Таким образом, запросы к свойствам метки времени блокируются Azure Cosmos DB. В качестве обходного решения можно определить свойство пользовательского типа DateTime или long и задать значение даты для клиента.

## <a name="query-by-using-linq"></a>Запросы с помощью LINQ 
Вы также можете выполнить запрос с помощью LINQ, который преобразуется в соответствующие выражения запроса OData. Ниже приведен пример создания запросов с использованием пакета SDK .NET.

```csharp
IQueryable<CustomerEntity> linqQuery = table.CreateQuery<CustomerEntity>()
            .Where(x => x.PartitionKey == "4")
            .Select(x => new CustomerEntity() { PartitionKey = x.PartitionKey, RowKey = x.RowKey, Email = x.Email });
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы выполнили следующее:

> [!div class="checklist"]
> * Научились выполнять запросы с помощью API таблицы

Теперь вы можете приступать к следующему руководству, чтобы узнать, как глобально распределять данные.

> [!div class="nextstepaction"]
> [Глобальное распределение данных](tutorial-global-distribution-table.md)

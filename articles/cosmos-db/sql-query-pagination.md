---
title: Разбиение на страницы в Azure Cosmos DB
description: Сведения о концепциях подкачки и маркерах продолжения
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/29/2020
ms.openlocfilehash: 5faff410fa18c5161d93f739f77eeb9c85d581a8
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102430961"
---
# <a name="pagination-in-azure-cosmos-db"></a>Разбиение на страницы в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В Azure Cosmos DB запросы могут содержать несколько страниц результатов. В этом документе описываются условия, которые обработчик запросов Azure Cosmos DB использует, чтобы решить, следует ли разбивать результаты запроса на несколько страниц. При необходимости можно использовать маркеры продолжения для управления результатами запросов, охватывающими несколько страниц.

## <a name="understanding-query-executions"></a>Основные сведения о выполнении запросов

Иногда результаты запроса будут разбиты на несколько страниц. Результаты каждой страницы создаются отдельным выполнением запроса. Если результаты запроса нельзя вернуть в одном выполнении, Azure Cosmos DB автоматически разбивает результаты на несколько страниц.

Можно указать максимальное число элементов, возвращаемых запросом, задав `MaxItemCount` . `MaxItemCount`Задается для каждого запроса и сообщает обработчику запросов, что это число элементов будет возвращено или меньше. Можно задать значение `MaxItemCount` , `-1` если не нужно устанавливать ограничение на количество результатов для каждого выполнения запроса.

Кроме того, существуют и другие причины, по которым механизму запросов может потребоваться разделить результаты запроса на несколько страниц. Сюда входит следующее.

- Контейнер был отрегулирован и еще не был доступен для получения дополнительных результатов запроса.
- Отклик на выполнение запроса слишком большой
- Время выполнения запроса слишком длинное
- Обработчику запросов было более эффективно возвращать результаты при дополнительных выполнениях.

Число возвращаемых элементов на выполнение запроса всегда будет меньше или равно `MaxItemCount` . Однако возможно, что другие критерии могли бы ограничить количество результатов, возвращаемых запросом. Если один и тот же запрос выполняется несколько раз, число страниц может быть непостоянным. Например, если запрос регулируется, в каждой странице может быть меньше доступных результатов, что означает, что запрос будет содержать дополнительные страницы. В некоторых случаях также возможно, что запрос может вернуть пустую страницу результатов.

## <a name="handling-multiple-pages-of-results"></a>Обработка нескольких страниц результатов

Чтобы обеспечить точный результат запроса, необходимо выполнить все страницы. Следует продолжать выполнять запросы до тех пор, пока не будут отсутствовать дополнительные страницы.

Ниже приведены некоторые примеры обработки результатов из запросов с несколькими страницами.

- [Пакет SDK для .NET](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L280)
- [пакет SDK для Java](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L162-L176)
- [Пакет SDK для Node.js](https://github.com/Azure/azure-sdk-for-js/blob/83fcc44a23ad771128d6e0f49043656b3d1df990/sdk/cosmosdb/cosmos/samples/IndexManagement.ts#L128-L140)
- [Пакет SDK для Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/examples.py#L89)

## <a name="continuation-tokens"></a>Маркеры продолжения

В пакете SDK для .NET и пакета SDK для Java можно при необходимости использовать маркеры продолжения в качестве закладки для хода выполнения запроса. Azure Cosmos DB выполнения запросов не имеют состояния на стороне сервера и могут возобновляться в любое время с помощью токена продолжения. Маркеры продолжения не поддерживаются в пакете SDK для Node.js. Для пакета SDK для Python поддерживается односекционные запросы, и значение PK должно быть указано в объекте Options, так как его не хватает в самом запросе.

Ниже приведен пример использования маркеров продолжения.

- [Пакет SDK для .NET](https://github.com/Azure/azure-cosmos-dotnet-v2/blob/master/samples/code-samples/Queries/Program.cs#L699-L734)
- [пакет SDK для Java](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L216)
- [Пакет SDK для Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/test/test_query.py#L533)

Если запрос возвращает токен продолжения, то имеются дополнительные результаты запроса.

В REST API Azure Cosmos DB можно управлять маркерами продолжения с помощью `x-ms-continuation` заголовка. Как и при выполнении запросов к пакету SDK для .NET или Java, если `x-ms-continuation` заголовок ответа не пуст, это означает, что запрос содержит дополнительные результаты.

Пока используется одна и та же версия пакета SDK, срок действия маркеров продолжения никогда не истекает. При необходимости можно [ограничить размер токена продолжения](/dotnet/api/microsoft.azure.documents.client.feedoptions.responsecontinuationtokenlimitinkb#Microsoft_Azure_Documents_Client_FeedOptions_ResponseContinuationTokenLimitInKb). Независимо от объема данных или числа физических секций в контейнере, запросы возвращают один маркер продолжения.

Маркеры продолжения нельзя использовать для запросов с [ключевым словом](sql-query-keywords.md#distinct) [Group By](sql-query-group-by.md) или DISTINCT, поскольку для этих запросов требуется хранить значительный объем состояний. Для запросов с `DISTINCT` можно использовать маркеры продолжения при добавлении `ORDER BY` в запрос.

Ниже приведен пример запроса `DISTINCT` , который может использовать маркер продолжения:

```sql
SELECT DISTINCT VALUE c.name
FROM c
ORDER BY c.name
```

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об Azure Cosmos DB](introduction.md)
- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Предложение ORDER BY](sql-query-order-by.md)

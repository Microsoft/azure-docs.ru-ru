---
title: Часто задаваемые вопросы об API Gremlin в Azure Cosmos DB
description: Получите ответы на часто задаваемые вопросы об API Gremlin в Azure Cosmos DB
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: conceptual
ms.date: 04/28/2020
ms.author: sngun
ms.openlocfilehash: 0374886482b43eefe6df5e107fd4da371633099d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359792"
---
# <a name="frequently-asked-questions-about-the-gremlin-api-in-azure-cosmos-db"></a>Часто задаваемые вопросы об API Gremlin в Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

В этой статье содержатся ответы на некоторые часто задаваемые вопросы об API Gremlin в Azure Cosmos DB.

## <a name="how-to-evaluate-the-efficiency-of-gremlin-queries"></a>Оценка эффективности запросов Gremlin

Чтобы получить предварительный план выполнения запроса, можно использовать **ExecutionProfile()** в конце запроса Gremlin, как показано в примере ниже:

**Пример запроса**

```
g.V('mary').out('knows').executionProfile()
```

**Пример выходных данных**

```json
[
  {
    "gremlin": "g.V('mary').out('knows').executionProfile()",
    "totalTime": 8,
    "metrics": [
      {
        "name": "GetVertices",
        "time": 3,
        "annotations": {
          "percentTime": 37.5
        },
        "counts": {
          "resultCount": 1
        }
      },
      {
        "name": "GetEdges",
        "time": 5,
        "annotations": {
          "percentTime": 62.5
        },
        "counts": {
          "resultCount": 0
        },
        "storeOps": [
          {
            "count": 0,
            "size": 0,
            "time": 0.6
          }
        ]
      },
      {
        "name": "GetNeighborVertices",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      },
      {
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      }
    ]
  }
]
```

Выходные данные указанного выше профиля показывают, сколько времени затрачено на получение объектов вершин, пограничных объектов и размера рабочего набора данных. На основе этих значений и формируется стоимость запросов Azure Cosmos DB.

## <a name="other-frequently-asked-questions"></a>Другие часто задаваемые вопросы

### <a name="how-are-rus-charged-when-running-queries-on-a-graph-database"></a>Как оплачиваются единицы запроса (ЕЗ) при выполнении запросов к базе данных графа?

Все объекты графа (вершины и ребра) описаны в документации JSON, относящейся к серверной части. Так как запрос к Gremlin может одновременно модифицировать несколько объектов графа, стоимость будет зависеть от количества объектов (ребер), обработанных в ходе этого запроса. Точно таким же образом Azure Cosmos DB взаимодействует со всеми API. См. дополнительные сведения о [единицах запросов в базе данных Azure Cosmos DB](request-units.md).

Плата за ЕЗ формируется на основе обрабатываемого, но не результирующего набора данных. Например, если нужно получить одну вершину, но при этом будет затронуто более одного другого объекта, оплата будет учитывать все объекты графа, используемые при вычислении этой вершины.

### <a name="whats-the-maximum-scale-that-a-graph-database-can-have-in-azure-cosmos-db-gremlin-api"></a>Каков максимальный масштаб, который может иметь база данных Graph в Azure Cosmos DB API Gremlin?

Azure Cosmos DB выполняет [горизонтальное секционирование](partitioning-overview.md), чтобы автоматически увеличивать объем хранилища и пропускную способность в соответствии с требованиями. Максимальная пропускная способность и емкость хранилища рабочей нагрузки определяются числом секций, связанных с заданным контейнером. Однако контейнер API Gremlin имеет специальный набор руководств, обеспечивающих правильную работу в масштабе. Дополнительные сведения о секционировании данных и рекомендации см. в статье о [секционировании и масштабировании в Azure Cosmos DB](partitioning-overview.md).

### <a name="for-cnet-development-should-i-use-the-microsoftazuregraphs-package-or-gremlinnet"></a>Можно ли использовать для разработки на C# или .NET пакет Microsoft.Azure.Graphs или Gremlin.NET?

В качестве основных соединителей для службы API Gremlin для Azure Cosmos DB использует драйверы с открытым кодом. Следовательно, рекомендуется использовать [драйверы, поддерживаемые Apache Tinkerpop](https://tinkerpop.apache.org/).

### <a name="how-can-i-protect-against-injection-attacks-using-gremlin-drivers"></a>Как можно защитится от атак путем внедрения кода с помощью драйверов Gremlin?

Большинство драйверов машинного кода Apache Tinkerpop Gremlin позволяют указать словарь параметров для выполнения запроса. См. пример с использованием [Gremlin.Net](https://tinkerpop.apache.org/docs/3.2.7/reference/#gremlin-DotNet) и [Gremlin-Javascript](https://github.com/Azure-Samples/azure-cosmos-db-graph-nodejs-getting-started/blob/main/app.js).

### <a name="why-am-i-getting-the-gremlin-query-compilation-error-unable-to-find-any-method-error"></a>Почему я получаю сообщение "ошибка компиляции запроса Gremlin: не удается найти какой бы то ни было метод"?

API Gremlin для Azure Cosmos DB реализует ряд функций, определенных в Gremlin. Информацию о поддерживаемых шагах и другие дополнительные сведения см. в статье о [поддержке Gremlin](gremlin-support.md).

Лучшее решение — использовать вместо требуемых (основных) функций Gremlin соответствующие функции, которые поддерживаются в Azure Cosmos DB.

### <a name="why-am-i-getting-the-websocketexception-the-server-returned-status-code-200-when-status-code-101-was-expected-error"></a>Почему я получаю "Вебсоккетексцептион: сервер вернул код состояния" 200 ", когда ожидался код состояния" 101 "?

Чаще всего эта ошибка возникает при использовании неправильной конечной точки. Конечная точка, которая возвращает эту ошибку, имеет следующий адрес:

`https:// YOUR_DATABASE_ACCOUNT.documents.azure.com:443/`

Это конечная точка документов для базы данных графа.  Следует использовать правильную конечную точку Gremlin со следующим адресом:

`https://YOUR_DATABASE_ACCOUNT.gremlin.cosmosdb.azure.com:443/`

### <a name="why-am-i-getting-the-requestrateistoolarge-error"></a>Почему я получаю сообщение об ошибке «Рекуестратеистуларже»?

Эта ошибка означает, что выделенного количества единиц запросов (ЕЗ) в секунду недостаточно для обслуживания запроса. Такая ошибка обычно возникает, когда вы выполняете запрос для получения всех вершин:

```
// Query example:
g.V()
```

Такой запрос попытается получить все вершины графа. Следовательно, стоимость такого запроса будет включать как минимум стоимость ЕЗ, требуемых для получения всех вершин. Чтобы такой запрос был выполнен, нужно указать требуемое количество ЕЗ/с.

### <a name="why-do-my-gremlin-driver-connections-get-dropped-eventually"></a>Почему не удается установить подключение драйвера Gremlin?

Подключение Gremlin устанавливается по протоколу WebSocket. Хотя подключение по WebSocket не имеет определенного срока жизни, API Gremlin для Azure Cosmos DB разрывает неактивные подключения через 30 минут ожидания.

### <a name="why-cant-i-use-fluent-api-calls-in-the-native-gremlin-drivers"></a>Почему я не могу использовать вызовы API Fluent в собственных драйверах Gremlin?

Вызовы текучих API пока не поддерживаются API Gremlin для Azure Cosmos DB. Для таких вызовов требуется функция внутреннего форматирования (поддержка байт-кода), которая сейчас недоступна в API Gremlin для Azure Cosmos DB. По этой же причине сейчас не поддерживается и последняя версия драйвера Gremlin-JavaScript.

## <a name="next-steps"></a>Дальнейшие действия

* [Поддержка сетевого протокола Azure Cosmos DB Gremlin](gremlin-support.md)
* Создание, запрос и обход базы данных Azure Cosmos DB Graph с помощью [консоли Gremlin](create-graph-gremlin-console.md)

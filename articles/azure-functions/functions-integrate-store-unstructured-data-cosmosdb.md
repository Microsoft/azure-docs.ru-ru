---
title: Хранение неструктурированных данных с помощью Azure Cosmos DB и службы "Функции"
description: Хранение неструктурированных данных с помощью служб Функции Azure и Cosmos DB
ms.topic: quickstart
ms.date: 10/01/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 5bc3895cb219338acde492b871dce806db70622b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91661165"
---
# <a name="store-unstructured-data-using-azure-functions-and-azure-cosmos-db"></a>Хранение неструктурированных данных с помощью Azure Cosmos DB и службы "Функции Azure"

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) — это отличный способ хранения неструктурированных данных и данных в формате JSON. В сочетании с функциями Azure Cosmos DB позволяет быстро и просто сохранять данные, используя код гораздо меньшего объема, чем требуется для хранения данных в реляционной базе данных.

> [!NOTE]
> Сейчас триггер, входные и выходные привязки Azure Cosmos DB работают только с учетными записями API SQL и API Graph.

В Функциях Azure входные и выходные привязки предоставляют декларативный способ подключения данных внешних служб к функции. В этой статье вы узнаете, как обновить имеющуюся функцию, чтобы добавить выходную привязку, которая сохраняет неструктурированные данные в документе Azure Cosmos DB.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

[!INCLUDE [Previous quickstart note](../../includes/functions-quickstart-previous-topics.md)]

## <a name="create-an-azure-cosmos-db-account"></a>создание учетной записи Azure Cosmos DB;

Перед созданием выходной привязки необходимо получить учетную запись Azure Cosmos DB с API SQL.

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-an-output-binding"></a>Добавление выходной привязки

1. На портале Azure перейдите к ранее созданному приложению-функции и выберите его.

1. Выберите **Функции**, а затем выберите функцию HttpTrigger.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-select-http-function.png" alt-text="Выберите нужную функцию на портале Azure." border="true":::

1. Последовательно выберите **Интеграция** и **Добавить вывод**.

     :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-add-output-binding.png" alt-text="Добавление выходной привязки к Azure Cosmos DB." border="true":::

1. Укажите приведенные в таблице значение в области **Добавить вывод**.

     :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-configure-cosmosdb-binding.png" alt-text="Настройка выходной привязки к Azure Cosmos B." border="true":::

    | Параметр      | Рекомендуемое значение  | Описание                                |
    | ------------ | ---------------- | ------------------------------------------ |
    | **Тип привязки** | Azure Cosmos DB | Имя типа привязки, которое нужно выбрать для создания выходной привязки к Azure Cosmos DB. |
    | **Имя параметра документа** | taskDocument | Имя, которое ссылается на объект Cosmos DB в коде. |
    | **Имя базы данных** | taskDatabase | Имя базы данных для сохранения документов. |
    | **Имя коллекции** | taskCollection | Имя коллекции базы данных. |
    | **If true, creates the Cosmos DB database and collection** (Если значение равно true, создается база данных Cosmos DB и коллекция) | Да | Коллекция не существует, создайте ее. |
    | **Подключение к учетной записи Cosmos DB** | Новый параметр | Щелкните **Создать**, выберите вариант **Учетная запись Azure Cosmos DB** и созданную ранее **учетную запись базы данных**, а затем щелкните **OK**. Создает параметр приложения для подключения к учетной записи. Этот параметр используется в привязке для подключения к базе данных. |

1. Нажмите **ОК**, чтобы создать привязку.

## <a name="update-the-function-code"></a>Обновление кода функции

Замените имеющийся код функции следующим кодом для выбранного языка:

# <a name="c"></a>[C#](#tab/csharp)

Замените имеющуюся функцию C# следующим кодом:

```csharp
#r "Newtonsoft.Json"

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

public static IActionResult Run(HttpRequest req, out object taskDocument, ILogger log)
{
    string name = req.Query["name"];
    string task = req.Query["task"];
    string duedate = req.Query["duedate"];

    // We need both name and task parameters.
    if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(task))
    {
        taskDocument = new
        {
            name,
            duedate,
            task
        };

        return (ActionResult)new OkResult();
    }
    else
    {
        taskDocument = null;
        return (ActionResult)new BadRequestResult();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Замените имеющуюся функцию JavaScript следующим кодом:

```js
module.exports = async function (context, req) {

    // We need both name and task parameters.
    if (req.query.name && req.query.task) {

        // Set the output binding data from the query object.
        context.bindings.taskDocument = req.query;

        // Success.
        context.res = {
            status: 200
        };
    }
    else {
        context.res = {
            status: 400,
            body: "The query options 'name' and 'task' are required."
        };
    }
};
```
---

Этот пример кода считывает строки HTTP-запроса и назначает их в качестве полей объекта `taskDocument`. Привязка `taskDocument` отправляет данные объекта из этого параметра привязки для их сохранения в связанной базе данных документов. База данных создается при первом запуске функции.

## <a name="test-the-function-and-database"></a>Тестирование функции и базы данных

1. Выберите **Тест** и "Выполнить". В разделе **Запрос** щелкните **Добавить параметр** и добавьте в строку запроса следующие параметры.

    + `name`
    + `task`
    + `duedate`

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-test-function.png" alt-text="Тестирование функции." border="true":::


1. Щелкните **Выполнить** и убедитесь, что возвращается состояние 200.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-test-function-output.png" alt-text="Снимок экрана: код HTTP-отклика с состоянием 200, выделенный после выбора пункта &quot;Выполнить&quot;." border="true":::


1. На портале Azure найдите и выберите **Azure Cosmos DB**.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-search-cosmos-db.png" alt-text="Поиск службы Cosmos DB." border="true":::

1. Выберите нужную учетную запись Azure Cosmos DB, а затем щелкните **Обозреватель данных**.

1. Разверните узлы **TaskCollection**, выберите новый документ и убедитесь, что этот документ содержит ваши значения для строки запроса, а также некоторые дополнительные метаданные.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-data-explorer-check-document.png" alt-text="Проверка строковых значений в документе." border="true":::

Вы успешно добавили привязку к триггеру HTTP, которая сохраняет неструктурированные данные в Azure Cosmos DB.

[!INCLUDE [Clean-up section](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о привязке к базе данных Cosmos DB см. в статье [Привязки Cosmos DB в Функциях Azure](functions-bindings-cosmosdb.md).

[!INCLUDE [functions-quickstart-next-steps](../../includes/functions-quickstart-next-steps-2.md)]

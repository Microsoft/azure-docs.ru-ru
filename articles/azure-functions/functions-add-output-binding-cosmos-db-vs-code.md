---
title: Подключение Функций Azure к Azure Cosmos DB с помощью Visual Studio Code
description: Узнайте, как подключить Функции Azure к учетной записи Azure Cosmos DB путем добавления выходной привязки к проекту Visual Studio Code.
author: ThomasWeiss
ms.date: 03/23/2021
ms.topic: quickstart
ms.author: thweiss
zone_pivot_groups: programming-languages-set-functions-temp
ms.openlocfilehash: 0a0c63ee54699185bcd02104b1a3f4d0070ea808
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105023254"
---
# <a name="connect-azure-functions-to-azure-cosmos-db-using-visual-studio-code"></a>Подключение Функций Azure к Azure Cosmos DB с помощью Visual Studio Code

[!INCLUDE [functions-add-storage-binding-intro](../../includes/functions-add-storage-binding-intro.md)]

В этой статье описано, как с помощью Visual Studio Code подключить [Azure Cosmos DB](../cosmos-db/introduction.md) к функции, которую вы создали с помощью предыдущего краткого руководства. Выходная привязка, которую вы добавите к этой функции, записывает данные из HTTP-запроса в документ JSON, который хранится в контейнере Azure Cosmos DB. 

::: zone pivot="programming-language-csharp"
Перед началом работы необходимо выполнить действия, описанные в статье [Краткое руководство. Создание функции C# в Azure с помощью Visual Studio Code](create-first-function-vs-code-csharp.md). Если вы уже удалили ресурсы по окончании работы по этой статье, выполните шаги еще раз, чтобы повторно создать приложение-функцию и связанные ресурсы в Azure.
::: zone-end
::: zone pivot="programming-language-javascript"  
Перед началом работы необходимо выполнить действия, описанные в статье [Краткое руководство. Создание функции JavaScript в Azure с помощью Visual Studio Code](create-first-function-vs-code-node.md). Если вы уже удалили ресурсы по окончании работы по этой статье, выполните шаги еще раз, чтобы повторно создать приложение-функцию и связанные ресурсы в Azure.  
::: zone-end

## <a name="configure-your-environment"></a>Настройка среды

Перед началом работы обязательно установите расширение [Базы данных Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb) для Visual Studio Code.

## <a name="create-your-azure-cosmos-db-account"></a>Создание учетной записи Azure Cosmos DB

> [!IMPORTANT]
> Сейчас предлагается предварительная версия [бессерверного режима Azure Cosmos DB](../cosmos-db/serverless.md). Этот режим на основе потребления делает Azure Cosmos DB надежным вариантом для бессерверных рабочих нагрузок. Чтобы использовать Azure Cosmos DB в бессерверном режиме, выберите вариант **Бессерверный** для параметра **Режим емкости** при создании учетной записи.

1. В новом окне браузера войдите на [портал Azure](https://portal.azure.com/).

2. Последовательно щелкните **Создать ресурс** > **Базы данных** > **Azure Cosmos DB**.
   
    :::image type="content" source="../../includes/media/cosmos-db-create-dbaccount/create-nosql-db-databases-json-tutorial-1.png" alt-text="Область Базы данных на портале Azure" border="true":::

3. На странице **Создание учетной записи Azure Cosmos DB** настройте параметры для новой учетной записи Azure Cosmos DB. 
 
    Параметр|Значение|Описание
    ---|---|---
    Подписка|*Ваша подписка*|Выберите подписку Azure, в которой вы создали приложение-функцию, как описано в [предыдущей статье](./create-first-function-vs-code-csharp.md).
    Группа ресурсов|*Используемая группа ресурсов*|Выберите группу ресурсов, в которой вы создали приложение-функцию, как описано в [предыдущей статье](./create-first-function-vs-code-csharp.md).
    Имя учетной записи|*Укажите уникальное имя*|Введите уникальное имя для идентификации вашей учетной записи Azure Cosmos DB.<br><br>Имя может содержать только строчные буквы, цифры и дефисы. Его длина должна быть от 3 до 31 знаков.
    API|Core (SQL)|Выберите **Core (SQL)** , чтобы создать базу данных документов для запросов с использованием синтаксиса SQL. [См. дополнительные сведения об API SQL для Azure Cosmos DB.](../cosmos-db/introduction.md)|
    Расположение|*Выберите ближайший к вашему расположению регион.*|Выберите географическое расположение для размещения учетной записи Azure Cosmos DB. Используйте ближайшее к вам или вашим пользователям расположение, чтобы обеспечить максимально быстрый доступ к данным.
    Режим емкости|"Бессерверный" или "Подготовленная пропускная способность".|Выберите **Бессерверный**, чтобы создать учетную запись в режиме [Бессерверный](../cosmos-db/serverless.md). Выберите **Подготовленная пропускная способность**, чтобы создать учетную запись в режиме [подготовленной пропускной способности](../cosmos-db/set-throughput.md).<br><br>Выберите **Бессерверный**, если вы только начинаете работу с Azure Cosmos DB.

4. Щелкните **Просмотреть и создать**. Можете пропустить разделы **Сеть** и **Теги**. 

5. Просмотрите сводную информацию и нажмите кнопку **Создать**. 

6. Дождитесь, пока завершится создание Azure Cosmos DB, а затем щелкните **Перейти к ресурсу**.

    :::image type="content" source="../cosmos-db/media/create-cosmosdb-resources-portal/azure-cosmos-db-account-deployment-successful.png" alt-text="Создание учетной записи Azure Cosmos DB завершено" border="true":::

## <a name="create-an-azure-cosmos-db-database-and-container"></a>Создание базы данных и контейнера Azure Cosmos DB

Теперь в учетной записи Azure Cosmos DB выберите **Обозреватель данных** и щелкните **Создать контейнер**. Создайте базу данных с именем *my-database* и контейнер с именем *my-container*, а также выберите `/id` в качестве [ключа секции](../cosmos-db/partitioning-overview.md).

:::image type="content" source="./media/functions-add-output-binding-cosmos-db-vs-code/create-container.png" alt-text="Создание контейнера Azure Cosmos DB на портале Azure" border="true":::

## <a name="update-your-function-app-settings"></a>Обновление параметров приложения-функции

В [предыдущем кратком руководстве](./create-first-function-vs-code-csharp.md) показано, как создать в Azure приложение-функцию. Теперь вы измените это приложение-функцию так, чтобы оно создавало документы JSON в только что созданном контейнере Azure Cosmos DB. Чтобы подключиться к учетной записи Azure Cosmos DB, нужно добавить соответствующую строку подключения в параметры приложения. Затем вы скачаете новый файл параметров local.settings.json, чтобы подключаться к учетной записи Azure Cosmos DB при работе в локальной среде.

1. В Visual Studio Code найдите учетную запись Azure Cosmos DB, которую вы недавно создали. Щелкните ее имя правой кнопкой мыши и выберите **Copy Connection String** (Копировать строку подключения).

    :::image type="content" source="./media/functions-add-output-binding-cosmos-db-vs-code/copy-connection-string.png" alt-text="Копирование строки подключения к Azure Cosmos DB" border="true":::

1. Нажав клавишу <kbd>F1</kbd>, откройте палитру команд, а затем найдите и выполните команду `Azure Functions: Add New Setting...`.

1. Выберите созданное в рамках предыдущей статьи приложение-функцию. Введите следующие сведения по соответствующим запросам:

    + **Enter new app setting name** (Введите имя нового параметра приложения): введите `CosmosDbConnectionString`.

    + **Enter value for "CosmosDbConnectionString"** (Введите значение строки подключения к Cosmos DB): вставьте строку подключения к учетной записи Azure Cosmos DB, которую вы скопировали ранее.

1. Снова нажмите клавишу <kbd>F1</kbd>, чтобы открыть палитру команд, а затем найдите и выполните команду `Azure Functions: Download Remote Settings...`. 

1. Выберите созданное в рамках предыдущей статьи приложение-функцию. Выберите **Да, для всех**, чтобы перезаписать имеющиеся локальные параметры. 

## <a name="register-binding-extensions"></a>Регистрация расширений привязки

Так как вы используете выходную привязку Azure Cosmos DB, перед выполнением проекта нужно установить соответствующее расширение привязок. 

::: zone pivot="programming-language-csharp"

За исключением триггеров HTTP и таймера, привязки реализованы в виде пакетов расширений. Выполните следующую команду [dotnet add package](/dotnet/core/tools/dotnet-add-package) в окне терминала, чтобы добавить пакет расширений службы хранилища в свой проект.

```bash
dotnet add package Microsoft.Azure.WebJobs.Extensions.CosmosDB
```

::: zone-end

::: zone pivot="programming-language-javascript"

В проекте настроено использование [пакетов расширений](functions-bindings-register.md#extension-bundles), которые автоматически устанавливают предопределенный набор пакетов расширений. 

Возможность использования пакетов расширений включена в файле host.json в корне проекта, который выглядит следующим образом.

:::code language="json" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/host.json":::

::: zone-end

Теперь вы можете добавить в проект выходную привязку Azure Cosmos DB.

## <a name="add-an-output-binding"></a>Добавление выходной привязки

В службе "Функции" для каждого типа привязок требуется `direction`, `type` и уникальное `name`, которое определяется в файле function.json. Способ определения этих атрибутов зависит от языка приложения-функции.

::: zone pivot="programming-language-csharp"

В проекте библиотеки классов C# привязки определяются как атрибуты привязки в методе функции. Затем на основе этих атрибутов автоматически создается файл *function.json*, обязательный для службы Функции Azure.

Откройте в проекте файл *HttpExample.cs* и добавьте следующий параметр в определение метода `Run`:

```csharp
[CosmosDB(
    databaseName: "my-database",
    collectionName: "my-container",
    ConnectionStringSetting = "CosmosDbConnectionString")]IAsyncCollector<dynamic> documentsOut,
```

Параметр `documentsOut` имеет тип IAsyncCollector<T>, который представляет коллекцию документов JSON для записи в контейнер Azure Cosmos DB по завершении работы функции. В отдельных атрибутах указываются имена контейнера и родительской базы данных. Строка подключения к учетной записи Azure Cosmos DB задается атрибутом `ConnectionStringSettingAttribute`.

Определение метода Run должно выглядеть следующим образом:  

```csharp
[FunctionName("HttpExample")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
    [CosmosDB(
        databaseName: "my-database",
        collectionName: "my-container",
        ConnectionStringSetting = "CosmosDbConnectionString")]IAsyncCollector<dynamic> documentsOut,
    ILogger log)
```

::: zone-end

::: zone pivot="programming-language-javascript"

Атрибуты привязки определяются непосредственно в файле function.json. В зависимости от типа привязки могут потребоваться дополнительные свойства. [Конфигурация вывода в Azure Cosmos DB](./functions-bindings-cosmosdb-v2-output.md#configuration) описывает поля, которые необходимо задать для выходной привязки Azure Cosmos DB. Расширение позволяет легко добавлять привязки в файл function.json. 

Чтобы создать привязку, щелкните правой кнопкой мыши (CTRL+щелчок в macOS) файл `function.json` в папке HttpTrigger и выберите **Add binding...** (Добавить привязку...). Выполните инструкции, указанные на экране, чтобы определить следующие свойства для новой привязки:

| prompt | Значение | Описание |
| -------- | ----- | ----------- |
| **Select binding direction (Выберите направление привязки)** | `out` | Привязка является выходной привязкой. |
| **Выберите привязку с направлением "out"** | `Azure Cosmos DB` | В качестве привязки нужно выбрать привязку Azure Cosmos DB. |
| **The name used to identify this binding in your code** (Имя, используемое для идентификации этой привязки в коде) | `outputDocument` | Имя, которое используется для идентификации параметров привязки, указанных в коде. |
| **The Cosmos DB database where data will be written** (База данных Cosmos DB, в которую будут записаны данные) | `my-database` | Имя базы данных Azure Cosmos DB, которая содержит целевой контейнер. |
| **Database collection where data will be written** (Коллекция баз данных, в которую будут записаны данные) | `my-container` | Имя контейнера Azure Cosmos DB, в который будут записаны документы JSON. |
| **If true, creates the Cosmos DB database and collection** (Если значение равно true, создается база данных Cosmos DB и коллекция) | `false` | Целевая база и контейнер уже существуют. |
| **Select setting from "local.setting.json"** (Выберите параметр из файла local.setting.json) | `CosmosDbConnectionString` | Имя параметра приложения, который содержит строку подключения к учетной записи Azure Cosmos DB. |
| **Ключ раздела (дополнительно)** | *Не указывайте* | Требуется, только если выходная привязка создает контейнер. |
| **Пропускная способность коллекции (необязательно)** | *Не указывайте* | Требуется, только если выходная привязка создает контейнер. |

Привязка добавляется к массиву `bindings` в function.json, который теперь должен выглядеть примерно так:

```json
{
    "type": "cosmosDB",
    "direction": "out",
    "name": "outputDocument",
    "databaseName": "my-database",
    "collectionName": "my-container",
    "createIfNotExists": "false",
    "connectionStringSetting": "CosmosDbConnectionString"
}
```

::: zone-end

## <a name="add-code-that-uses-the-output-binding"></a>Добавление кода, который использует выходную привязку

::: zone pivot="programming-language-csharp"  

Добавьте код, который использует объект выходной привязки `documentsOut` для создания документа JSON. Сделайте это перед возвращением метода.

```csharp
if (!string.IsNullOrEmpty(name))
{
    // Add a JSON document to the output container.
    await documentsOut.AddAsync(new
    {
        // create a random ID
        id = System.Guid.NewGuid().ToString(),
        name = name
    });
}
```

На этом этапе ваша функция должна выглядеть следующим образом:

```csharp
[FunctionName("HttpExample")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
    [CosmosDB(
        databaseName: "my-database",
        collectionName: "my-container",
        ConnectionStringSetting = "CosmosDbConnectionString")]IAsyncCollector<dynamic> documentsOut,
    ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    if (!string.IsNullOrEmpty(name))
    {
        // Add a JSON document to the output container.
        await documentsOut.AddAsync(new
        {
            // create a random ID
            id = System.Guid.NewGuid().ToString(),
            name = name
        });
    }

    string responseMessage = string.IsNullOrEmpty(name)
        ? "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."
        : $"Hello, {name}. This HTTP triggered function executed successfully.";

    return new OkObjectResult(responseMessage);
}
```

::: zone-end

::: zone pivot="programming-language-javascript"  

Добавьте код, который применяет объект выходной привязки `outputDocument` к `context.bindings` для создания документа JSON. Добавьте этот код перед инструкцией `context.res`.

```javascript
if (name) {
    context.bindings.outputDocument = JSON.stringify({
        // create a random ID
        id: new Date().toISOString() + Math.random().toString().substr(2,8),
        name: name
    });
}
```

На этом этапе ваша функция должна выглядеть следующим образом:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    const name = (req.query.name || (req.body && req.body.name));
    const responseMessage = name
        ? "Hello, " + name + ". This HTTP triggered function executed successfully."
        : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";

    if (name) {
        context.bindings.outputDocument = JSON.stringify({
            // create a random ID
            id: new Date().toISOString() + Math.random().toString().substr(2,8),
            name: name
        });
    }

    context.res = {
        // status: 200, /* Defaults to 200 */
        body: responseMessage
    };
}
```

::: zone-end  

## <a name="run-the-function-locally"></a>Локальное выполнение функции

1. Как и при работе с предыдущей статьей, нажмите клавишу <kbd>F5</kbd>, чтобы запустить проект приложения функции и Core Tools. 

1. При запущенных Core Tools перейдите в область **Azure: Функции**. В разделе **Функции** разверните **Локальный проект** > **Функции**. Щелкните правой кнопкой мыши (Ctrl-щелчок на компьютерах Mac) функцию `HttpExample` и выберите **Выполнить функцию...** .

    :::image type="content" source="../../includes/media/functions-run-function-test-local-vs-code/execute-function-now.png" alt-text="Вариант &quot;Выполнить функцию&quot; из Visual Studio Code":::

1. В поле **Ввести текст запроса** вы увидите значение текста запроса `{ "name": "Azure" }`. Нажмите клавишу ВВОД, чтобы отправить это сообщение запроса в свою функцию.  
 
1. После возврата ответа нажмите клавиши <kbd>CTRL+C</kbd>, чтобы остановить работу Core Tools.

### <a name="verify-that-a-json-document-has-been-created"></a>Проверка успешного создания документа JSON

1. На портале Azure вернитесь к учетной записи Azure Cosmos DB и выберите **Обозреватель данных**.

1. Разверните базу данных, затем контейнер, и выберите **Элементы**, чтобы увидеть список документов, созданных в этом контейнере.

1. Убедитесь, что выходная привязка успешно создала документ JSON.

    :::image type="content" source="./media/functions-add-output-binding-cosmos-db-vs-code/verify-output.png" alt-text="Проверка наличия нового документа в контейнере Azure Cosmos DB" border="true":::

## <a name="redeploy-and-verify-the-updated-app"></a>Повторное развертывание и проверка обновленного приложения

1. В Visual Studio Code нажмите клавишу F1, чтобы открыть палитру команд. В палитре команд найдите и щелкните `Azure Functions: Deploy to function app...`.

1. Выберите созданное в рамках первой статьи приложение-функцию. Так как вы повторно развертываете свой проект в том же приложении, щелкните **Развернуть**, чтобы закрыть предупреждение о перезаписи файлов.

1. После завершения развертывания вы можете повторно использовать возможность **Выполнить функцию...** , чтобы запустить функцию в Azure.

1. Еще раз [проверьте наличие созданных документов в контейнере Azure Cosmos DB](#verify-that-a-json-document-has-been-created) и убедитесь, что выходная привязка создала документ JSON.

## <a name="clean-up-resources"></a>Очистка ресурсов

*Ресурсы* в Azure — это приложения-функции, функции, учетные записи хранения и т. д. Они объединяются в *группы ресурсов*, при удалении которых удаляются и все данные в них.

Вы создали ресурсы для завершения этих кратких руководств. Вам могут быть выставлены счета за эти ресурсы в зависимости от [состояния учетной записи](https://azure.microsoft.com/account/) и [цен на службы](https://azure.microsoft.com/pricing/). Если вам больше не нужны ресурсы, их можно удалить следующим образом:

[!INCLUDE [functions-cleanup-resources-vs-code-inner.md](../../includes/functions-cleanup-resources-vs-code-inner.md)]

## <a name="next-steps"></a>Дальнейшие действия

Итак, вы обновили активируемую HTTP-запросом функцию, и она записывает документы JSON в контейнер Azure Cosmos DB. Теперь вы можете посмотреть дополнительные сведения о разработке Функций с помощью Visual Studio Code.

+ [Разработка Функций Azure с помощью Visual Studio Code](functions-develop-vs-code.md)

+ [Azure Functions triggers and bindings](functions-triggers-bindings.md) (Основные понятия триггеров и привязок в Функциях Azure)
::: zone pivot="programming-language-csharp"  
+ [Примеры полных проектов Функций на C#](/samples/browse/?products=azure-functions&languages=csharp).

+ [Справочник разработчика C# по функциям Azure](functions-dotnet-class-library.md)  
::: zone-end 
::: zone pivot="programming-language-javascript"  
+ [Примеры полных проектов Функций на JavaScript](/samples/browse/?products=azure-functions&languages=javascript).

+ [Руководство разработчика JavaScript для Функций Azure](functions-reference-node.md)  
::: zone-end  
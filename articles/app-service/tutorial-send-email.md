---
title: Руководство. Отправка электронной почты с помощью Logic Apps
description: Узнайте о том, как вызывать бизнес-процессы из приложения Службы приложений. Отправляйте сообщения электронной почты, твиты и сообщения Facebook, а также создавайте списки рассылки и многое другое.
ms.topic: tutorial
ms.date: 04/08/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 8072a941cd89290af3e25cc63c4fccccce705df9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95014668"
---
# <a name="tutorial-send-email-and-invoke-other-business-processes-from-app-service"></a>Руководство по Отправка сообщений электронной почты и вызов других бизнес-процессов из Службы приложений

Из этого руководства вы узнаете, как интегрировать приложение Службы приложений с бизнес-процессами. Это поможет реализовать следующие распространенные сценарии использования веб-приложений:

- отправка сообщения электронной почты с подтверждением покупки;
- добавление пользователя в группу Facebook;
- подключение к сторонним системам, таким как SAP, SalesForce и т. д;
- обмен стандартными сообщениями B2B.

В этом руководстве показано, как отправить сообщения электронной почты через Gmail из приложения Службы приложений, используя [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Есть и другие способы отправки сообщений электронной почты из веб-приложения, например с помощью конфигурации SMTP для используемой языковой платформы. Но Logic Apps предоставляет приложениям Службы приложений гораздо больше возможностей без необходимости усложнять код. Logic Apps предоставляет простой интерфейс конфигурации для интеграции наиболее популярных бизнес-систем, которые можно в любое время вызывать из приложения с помощью HTTP-запроса.

## <a name="prerequisite"></a>Предварительные требования

Разверните приложение в Службе приложений с помощью любой удобной языковой платформы. Инструкции по развертыванию примера приложения см. ниже.

### <a name="aspnet"></a>[ASP.NET](#tab/dotnet)

[Руководство. Создание приложения ASP.NET в Azure с подключением к Базе данных SQL](app-service-web-tutorial-dotnet-sqldatabase.md)

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/dotnetcore)

[Руководство. Создание приложения ASP.NET Core и Базы данных SQL в Службе приложений Azure](tutorial-dotnetcore-sqldb-app.md)

### <a name="nodejs"></a>[Node.js](#tab/node)

[Руководство. Разработка приложения на основе Node.js и MongoDB в Azure](tutorial-nodejs-mongodb-app.md)

### <a name="php"></a>[PHP](#tab/php)

[Руководство. Разработка приложения на основе PHP и MySQL в Azure](tutorial-php-mysql-app.md)

### <a name="python"></a>[Python](#tab/python)

[Руководство. Запуск веб-приложения Python (Django) с PostgreSQL в Службе приложений Azure](tutorial-python-postgresql-app.md)

### <a name="ruby"></a>[Ruby](#tab/ruby)

[Разработка приложения на основе Ruby и Postgres в Службе приложений Azure в Linux](tutorial-ruby-postgres-app.md)

---

## <a name="create-the-logic-app"></a>Создание приложения логики

1. На [портале Azure](https://portal.azure.com) [создайте пустое приложение логики](../logic-apps/quickstart-create-first-logic-app-workflow.md#create-your-logic-app). Когда откроется **конструктор Logic Apps**, вернитесь к этому руководству.
1. На странице заставки конструктора Logic Apps выберите вариант **При получении HTTP-запроса** в разделе **Начать с общего триггера**.

    ![Снимок экрана: страница заставки для конструктора Logic Apps с выделенным вариантом When an HTTP request is received (При получении HTTP-запроса).](./media/tutorial-send-email/receive-http-request.png)
1. В диалоговом окне **При получении HTTP-запроса** выберите **Использовать пример полезной нагрузки, чтобы создать схему**.

    ![Снимок экрана: диалоговое окно When an HTTP request is received (При получении HTTP-запроса) и выбор параметра "Использовать пример полезной нагрузки, чтобы создать схему". ](./media/tutorial-send-email/generate-schema-with-payload.png)

1. Скопируйте следующий пример кода JSON в текстовое поле и щелкните **Готово**.

    ```json
    {
        "task": "<description>",
        "due": "<date>",
        "email": "<email-address>"
    }
    ```

    Теперь для нужных данных запроса будет создана схема. На практике вы можете просто записать реальные данные запроса, созданные кодом приложения, и Azure создаст схему JSON. 
1. В верхней части конструктора Logic Apps, щелкните **Сохранить**. 

    Теперь вы увидите URL-адрес триггера HTTP-запроса. Щелкните значок копирования, чтобы сохранить его для последующего использования.

    ![Снимок экрана: выделенный значок для копирования URL-адреса триггера HTTP-запроса.](./media/tutorial-send-email/http-request-url.png)

    Это определение HTTP-запроса является триггером для любого нужного действия в приложении логики, например для обращения к Gmail. Позже вы будете вызывать этот URL-адрес из приложения Службы приложений. См. сведения о триггере запроса в [справочнике по HTTP-запросам и ответам](../connectors/connectors-native-reqres.md).

1. В нижней части конструктора щелкните **Новый шаг**, введите **Gmail** в поле поиска действий, найдите и выберите **Отправка электронной почты (v2)** .
    
    > [!TIP]
    > Вы можете найти другие типы интеграции, например SendGrid, MailChimp, Microsoft 365 и SalesForce. См. сведения в документации по [Logic Apps](../logic-apps/index.yml).

1. В диалоговом окне **Gmail** щелкните **Вход** и войдите в учетную запись Gmail, из которой нужно отправить электронное письмо.

    ![Снимок экрана: диалоговое окно Gmail, используемое для входа в учетную запись Gmail, из которой необходимо отправить электронное письмо.](./media/tutorial-send-email/gmail-sign-in.png)

1. Войдя, щелкните текстовое поле **Кому**, чтобы открыть диалоговое окно динамического содержимого.

1. Рядом с действием **При получении HTTP-запроса** щелкните **Показать больше**.

    ![Снимок экрана: кнопка "Подробнее" рядом с действием When an HTTP request is received (При получении HTTP-запроса).](./media/tutorial-send-email/expand-dynamic-content.png)

    Здесь вы увидите три свойства из примера данных JSON, которые использовались ранее. На этом шаге вы создадите сообщение электронной почты на основе этих свойств HTTP-запроса.
1. Для поля **Кому** выберите вариант **email**. При желании диалоговое окно динамического содержимого можно убрать, щелкнув действие **Добавить динамическое содержимое**.

    ![Снимок экрана: параметр для выбора электронной почты и выделенный пункт "Добавить динамическое содержимое".](./media/tutorial-send-email/hide-dynamic-content.png)

1. В раскрывающемся списке **Добавить новый параметр** выберите **Тема** и **Текст**.

1. Щелкните текстовое поле **Тема** и выберите **task** аналогичным образом. Пока курсор находится в поле **Тема**, введите *создано*. 

1. Теперь щелкните **Текст** и выберите **due**. Поместите курсор слева от метки **due** и введите *Срок действия этого рабочего элемента*.

    > [!TIP]
    > Если вы хотите изменить содержимое HTML прямо в тексте сообщения электронной почты, выберите **Представление кода** в верхней части окна конструктора Logic Apps. Соблюдайте осторожность, чтобы сохранить код динамического содержимого (например, `@{triggerBody()?['due']}`).
    >
    > ![Снимок экрана: представление кода для просмотра содержимого HTML прямо в тексте сообщения электронной почты.](./media/tutorial-send-email/edit-rich-html-email.png) 

1. Затем добавьте асинхронный ответ HTTP в триггер HTTP. Щелкните значок **+** между триггером HTTP и действием Gmail и выберите **Добавить параллельную ветвь**.

    ![Снимок экрана: значок "+" и выделенный параметр "Добавить параллельную ветвь".](./media/tutorial-send-email/add-http-response.png)

1. В поле поиска действий введите **ответ** и добавьте действие **Ответ**.

    ![Снимок экрана: панель поиска и выделенное действие "Ответ".](./media/tutorial-send-email/choose-response-action.png)

    По умолчанию действие ответа отправляет код HTTP 200. Этого нам достаточно для работы с этим руководством. См. сведения в справочнике по [HTTP-запросам и ответам](../connectors/connectors-native-reqres.md).

1. В верхней части конструктора Logic Apps снова щелкните **Сохранить**. 

## <a name="add-http-request-code-to-app"></a>Добавление кода HTTP-запроса в приложение

Убедитесь, что вы скопировали URL-адрес настроенного ранее триггера HTTP-запроса. Так как он содержит конфиденциальную информацию, мы не рекомендуем размещать его непосредственно в коде. В Службе приложений можно указать ссылку на него через переменную среды, используя параметры приложения. 

В [Cloud Shell](https://shell.azure.com) создайте параметр приложения с помощью следующей команды (замените *\<app-name>* , *\<resource-group-name>* и *\<logic-app-url>* ):

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings LOGIC_APP_URL="<your-logic-app-url>"
```

В коде клиента HTTP выполните стандартный HTTP-запрос POST на URL-адрес, используя любой доступный для вашей языковой платформы язык, настроив следующую конфигурацию.

- Текст запроса содержит код JSON в том же формате, который вы указали в приложении логики:

    ```json
    {
        "task": "<description>",
        "due": "<date>",
        "email": "<email-address>"
    }
    ```

- Запрос содержит заголовок `Content-Type: application/json`. 
- Чтобы оптимизировать производительность, по возможности отправьте асинхронный запрос.

Щелкните вкладку для используемого языка и платформы ниже, чтобы просмотреть пример.

### <a name="aspnet"></a>[ASP.NET](#tab/dotnet)

В ASP.NET можно отправить HTTP-запрос POST с помощью класса [System.Net.Http.HttpClient](/dotnet/api/system.net.http.httpclient). Пример:

```csharp
// requires using System.Net.Http;
var client = new HttpClient();
// requires using System.Text.Json;
var jsonData = JsonSerializer.Serialize(new
{
    email = "a-valid@emailaddress.com",
    due = "4/1/2020",
    task = "My new task!"
});

HttpResponseMessage result = await client.PostAsync(
    // requires using System.Configuration;
    ConfigurationManager.AppSettings["LOGIC_APP_URL"],
    new StringContent(jsonData, Encoding.UTF8, "application/json"));
    
var statusCode = result.StatusCode.ToString();
```

Если вы тестируете этот код в примере приложения для руководства по [ разработке приложения на основе ASP.NET и Базы данных SQL в Azure](app-service-web-tutorial-dotnet-sqldatabase.md), вы можете с его помощью отправить подтверждение по электронной почте в [действии Create](https://github.com/Azure-Samples/dotnet-sqldb-tutorial/blob/master/DotNetAppSqlDb/Controllers/TodosController.cs#L52-L63) после добавления элемента `Todo`. Чтобы использовать приведенный выше асинхронный код, преобразуйте действие Create в асинхронное.

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/dotnetcore)

В ASP.NET Core можно отправить HTTP-запрос POST с помощью класса [System.Net.Http.HttpClient](/dotnet/api/system.net.http.httpclient). Пример:

```csharp
// requires using System.Net.Http;
var client = new HttpClient();
// requires using System.Text.Json;
var jsonData = JsonSerializer.Serialize(new
{
    email = "a-valid@emailaddress.com",
    due = "4/1/2020",
    task = "My new task!"
});

HttpResponseMessage result = await client.PostAsync(
    // Requires DI configuration to access app settings. See https://docs.microsoft.com/azure/app-service/configure-language-dotnetcore#access-environment-variables
    _configuration["LOGIC_APP_URL"],
    new StringContent(jsonData, Encoding.UTF8, "application/json"));
    
var statusCode = result.StatusCode.ToString();
```

> [!NOTE]
> Этот код специально упрощен для ознакомления. В реальной системе не следует создавать новый экземпляр объекта `HttpClient` для каждого запроса. Соблюдайте рекомендации из руководства по [использованию IHttpClientFactory для реализации устойчивых запросов HTTP](/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests).

Если вы тестируете этот код в примере приложения для руководства по [ разработке приложения на основе ASP.NET Core с базой данных SQL в Службе приложений Azure](tutorial-dotnetcore-sqldb-app.md), вы можете с его помощью отправить подтверждение по электронной почте в [действии Create](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial/blob/master/Controllers/TodosController.cs#L56-L65) после добавления элемента `Todo`.

### <a name="nodejs"></a>[Node.js](#tab/node)

В Node.js можно легко отправлять HTTP-сообщения с помощью пакета npm, например [axios](https://www.npmjs.com/package/axios). Пример:

```javascript
// Requires npm install --save axios
const axios = require('axios');

var jsonData = {
        email: "a-valid@emailaddress.com",
        due: "4/1/2020",
        task: "My new task!"
};

(async function(data) {
    try {
        const response = await axios.post(process.env.LOGIC_APP_URL, jsonData);
        console.log(response.status);
    } catch (error) {
        console.log(error);
    }
})(jsonData);

```

Если вы тестируете этот код в примере приложения для руководства по [ разработке приложения на основе Node.js и MongoDB в Azure](tutorial-nodejs-mongodb-app.md), вы можете с его помощью отправить подтверждение по электронной почте в [функции create](https://github.com/Azure-Samples/meanjs/blob/master/modules/articles/server/controllers/articles.server.controller.js#L14-L27) после [успешного сохранения статьи](https://github.com/Azure-Samples/meanjs/blob/master/modules/articles/server/controllers/articles.server.controller.js#L24).

### <a name="php"></a>[PHP](#tab/php)

В PHP можно легко отправлять HTTP-сообщения с помощью [Guzzle](http://docs.guzzlephp.org/en/stable/index.html). Пример:

```php
// Requires composer require guzzlehttp/guzzle:~6.0
use GuzzleHttp\Client;
...
$client = new Client();
$options = [
    'json' => [ 
        'email' => 'a-valid@emailaddress.com',
        'due' => '4/1/2020',
        'task' => "My new task!"
    ]
];

$promise = $client-> postAsync(getenv($LOGIC_APP_URL), $options)->then( 
    function ($response) {
        return $response->getStatusCode();
    }, function ($exception) {
        return $exception->getResponse();
    }
);

$response = $promise->wait();
// Requires Laravel to run Log::info(). Check the documentation of your preferred framework for logging instructions.
Log::info(print_r($response, TRUE));
```

Если вы тестируете этот код в примере приложения для руководства по [ разработке приложения на основе PHP и MySQL в Azure](tutorial-php-mysql-app.md), вы можете с его помощью отправлять подтверждение по электронной почте из [функции Route::post](https://github.com/Azure-Samples/laravel-tasks/blob/master/routes/web.php#L30-L48) непосредственно перед оператором return.

### <a name="python"></a>[Python](#tab/python)

В Python можно легко отправлять HTTP-сообщения с помощью [requests](https://pypi.org/project/requests/). Пример:

```python
# Requires pip install requests && pip freeze > requirements.txt
import requests
...
payload = {
    "email": "a-valid@emailaddress.com",
    "due": "4/1/2020",
    "task": "My new task!"
}
response = requests.post("https://prod-112.westeurope.logic.azure.com:443/workfl$
print(response.status_code)
```
<!-- ```python
# Requires pip install aiohttp && pip freeze > requirements.txt
import aiohttp
...
payload = {
        'email': 'a-valid@emailaddress.com',
        'due': '4/1/2020',
        'task': 'My new task!'
}
async with aiohttp.post('http://httpbin.org/post', data=json.dump(payload)) as resp:
    print(await resp.status())
``` -->

Если вы тестируете этот код в примере приложения для руководства по [ разработке веб-приложения на основе Python (Django) и PostgreSQL в Службе приложений Azure](tutorial-python-postgresql-app.md), вы можете с его помощью отправлять подтверждение по электронной почте из [функции Route::post](https://github.com/Azure-Samples/laravel-tasks/blob/master/routes/web.php#L30-L48) непосредственно перед оператором return.

### <a name="ruby"></a>[Ruby](#tab/ruby)

В Ruby можно легко отправлять HTTP-сообщения с помощью JSONClient. Пример:

```ruby
clnt = JSONClient.new
body = { 
    'email' => 'a-valid@emailaddress.com',
    'due' => '4/1/2020',
    'task' => "My new task!"
}

connection = clnt.post_async(ENV['LOGIC_APP_URL'], body)
```

Если вы тестируете этот код в примере приложения для руководства по [разработке приложения на основе Ruby и Postgres в Службе приложений Azure в Linux](tutorial-ruby-postgres-app.md), вы можете использовать его для отправки подтверждения по электронной почте в действии [create](https://github.com/Azure-Samples/rubyrails-tasks/blob/master/app/controllers/tasks_controller.rb#L26-L38) [при успешном выполнении @task.save](https://github.com/Azure-Samples/rubyrails-tasks/blob/master/app/controllers/tasks_controller.rb#L30).

---

## <a name="more-resources"></a>Дополнительные ресурсы

[Руководство. Размещение API RESTful с поддержкой CORS в Службе приложений Azure](app-service-web-tutorial-rest-api.md)  
[Справочник по HTTP-запросам и ответам для Logic Apps](../connectors/connectors-native-reqres.md)  
[Краткое руководство. Создание первого рабочего процесса с помощью Azure Logic Apps — портал Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md)
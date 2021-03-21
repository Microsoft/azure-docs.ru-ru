---
title: Вызов приложений логики с помощью решения "Функции Azure"
description: Вызов или Активация приложений логики с помощью функций Azure и служебной шины Azure
services: logic-apps
ms.suite: integration
ms.reviewer: jehollan, klam, logicappspm
ms.topic: article
ms.date: 11/08/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: a7df9ba1318f40de8af392cfaedbe51d7a5df755
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98784941"
---
# <a name="call-or-trigger-logic-apps-by-using-azure-functions-and-azure-service-bus"></a>Вызов или Активация приложений логики с помощью функций Azure и служебной шины Azure

Вы можете использовать [функции Azure](../azure-functions/functions-overview.md) для активации приложения логики, когда требуется развернуть долгосрочный прослушиватель или задачу. Например, можно создать функцию, которая прослушивает очередь [служебной шины Azure](../service-bus-messaging/service-bus-messaging-overview.md) и сразу же запускает приложение логики в качестве триггера push-уведомлений.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, [зарегистрируйтесь для получения бесплатной учетной записи Azure](https://azure.microsoft.com/free/).

* Пространство имен служебной шины Azure. Если у вас нет пространства имен, [сначала создайте пространство имен](../service-bus-messaging/service-bus-create-namespace-portal.md).

* Приложение-функция, представляющее собой контейнер для функций. Если у вас нет приложения-функции, [сначала создайте приложение-функцию](../azure-functions/functions-get-started.md)и убедитесь, что в качестве стека среды выполнения выбрано .NET.

* Базовые знания [создания приложений логики](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-logic-app"></a>Создание приложения логики

В этом сценарии у вас есть функция, выполняющая каждое приложение логики, которое необходимо активировать. Сначала создайте приложение логики, которое начинается с триггера HTTP-запроса. Функция Azure вызывает эту конечную точку при получении каждого сообщения из очереди.

1. Войдите на [портал Azure](https://portal.azure.com) и создайте пустое приложение логики.

   Если вы пока не знакомы с приложениями логики, ознакомьтесь со статьей [Краткое руководство. Создание первого автоматизированного рабочего процесса с помощью Azure Logic Apps на портале Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. В поле поиска введите `http request`. В списке триггеры выберите **время получения HTTP-запроса** триггера.

   ![Выбор триггера](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger.png)

   С помощью триггера запроса можно также ввести схему JSON, которая будет использоваться с сообщением очереди. Схемы JSON помогают конструктору приложений логики понять структуру входных данных и упростить их использование в рабочем процессе.

1. Чтобы указать схему, введите ее в поле **Схема JSON текста запроса**, например:

   ![Указание схемы JSON](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger-schema.png)

   Если у вас нет схемы, но у вас есть пример полезных данных в формате JSON, можно создать схему на основе полезных данных.

   1. В триггере запросов выберите **Использовать пример полезной нагрузки, чтобы создать схему**.

   1. В разделе **введите или вставьте пример полезных данных JSON**, введите Пример полезных данных и нажмите кнопку **Готово**.

      ![Ввод примера полезных данных](./media/logic-apps-scenario-function-sb-trigger/enter-sample-payload.png)

   Этот пример полезных данных создает схему, которая появляется в триггере:

   ```json
   {
      "type": "object",
      "properties": {
         "address": {
            "type": "object",
            "properties": {
               "number": {
                  "type": "integer"
               },
               "street": {
                  "type": "string"
               },
               "city": {
                  "type": "string"
               },
               "postalCode": {
                  "type": "integer"
               },
               "country": {
                  "type": "string"
               }
            }
         }
      }
   }
   ```

1. Добавьте любые другие действия, которые необходимо выполнить после получения сообщения очереди.

   Например, можно отправить сообщение электронной почты с соединителем Office 365 Outlook.

1. Сохраните свое приложение логики, которое создает URL-адрес обратного вызова для триггера в этом приложении логики. Позже этот URL-адрес обратного вызова будет использоваться в коде триггера очереди служебной шины Azure.

   URL-адрес обратного вызова отображается в свойстве **URL-адреса HTTP POST** .

   ![Созданный URL-адрес обратного вызова для триггера](./media/logic-apps-scenario-function-sb-trigger/callback-URL-for-trigger.png)

## <a name="create-a-function"></a>Создание функции

Далее создайте функцию, которая выступает в качестве триггера и ожидает передачи данных из очереди.

1. На портале Azure откройте и разверните приложение-функцию, если оно еще не открыто. 

1. В разделе с именем приложения-функции разверните **Функции**. На панели **функции** выберите **создать функцию**.

   ![Разверните узел "функции" и выберите "создать функцию".](./media/logic-apps-scenario-function-sb-trigger/add-new-function-to-function-app.png)

1. Выберите этот шаблон в зависимости от того, создано ли новое приложение-функция, в котором вы выбрали .NET в качестве стека выполнения, или используете существующее приложение-функцию.

   * Для новых приложений функций выберите этот шаблон: **триггер очереди служебной шины** .

     ![Выберите шаблон для нового приложения функции](./media/logic-apps-scenario-function-sb-trigger/current-add-queue-trigger-template.png)

   * Для существующего приложения-функции выберите этот шаблон: **триггер очереди служебной шины — C#** .

     ![Выбор шаблона для существующего приложения функции](./media/logic-apps-scenario-function-sb-trigger/legacy-add-queue-trigger-template.png)

1. В области **триггер очереди служебной шины Azure** укажите имя для триггера и настройте подключение служебной **шины** для очереди, в которой используется прослушиватель пакета SDK служебной шины Azure `OnMessageReceive()` , и нажмите кнопку **создать**.

1. Напишите базовую функцию для вызова ранее созданной конечной точки приложения логики, используя сообщение очереди в качестве триггера. Прежде чем писать функцию, ознакомьтесь со следующими соображениями.

   * В примере используется сообщение с типом содержимого `application/json`. При необходимости этот тип можно изменить.
   
   * Из-за возможных одновременно выполняемых функций, больших объемов или интенсивных загрузок не рекомендуется создавать экземпляры [класса HttpClient](/dotnet/api/system.net.http.httpclient) с помощью `using` инструкции и напрямую создавать экземпляр HttpClient для каждого запроса. Дополнительные сведения см. [в статье Использование хттпклиентфактори для реализации устойчивых HTTP-запросов](/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests#issues-with-the-original-httpclient-class-available-in-net-core).
   
   * По возможности повторно используйте экземпляр HTTP-клиентов. Дополнительные сведения см. [в статье Управление подключениями в функциях Azure](../azure-functions/manage-connections.md).

   В этом примере используется [ `Task.Run` метод](/dotnet/api/system.threading.tasks.task.run) в [асинхронном](/dotnet/csharp/language-reference/keywords/async) режиме. Дополнительные сведения см. в разделе [Асинхронное программирование с использованием ключевых слов async и await](/dotnet/csharp/programming-guide/concepts/async/).

   ```csharp
   using System;
   using System.Threading.Tasks;
   using System.Net.Http;
   using System.Text;

   // Can also fetch from App Settings or environment variable
   private static string logicAppUri = @"https://prod-05.westus.logic.azure.com:443/workflows/<remaining-callback-URL>";

   // Reuse the instance of HTTP clients if possible: https://docs.microsoft.com/azure/azure-functions/manage-connections
   private static HttpClient httpClient = new HttpClient();

   public static async Task Run(string myQueueItem, TraceWriter log) 
   {
      log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
      var response = await httpClient.PostAsync(logicAppUri, new StringContent(myQueueItem, Encoding.UTF8, "application/json")); 
   }
   ```

1. Чтобы протестировать функцию, добавьте сообщение очереди с помощью такого средства, как [Обозреватель служебной шины](https://github.com/paolosalvatori/ServiceBusExplorer).

   Приложение логики должно активироваться сразу же, как только функция получит это сообщение.

## <a name="next-steps"></a>Дальнейшие действия

* [Вызов, активация или вложение рабочих процессов с помощью конечных точек HTTP](../logic-apps/logic-apps-http-endpoint.md)
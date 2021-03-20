---
title: Как использовать хранилище очередей Azure из Node.js — службы хранилища Azure
description: Узнайте, как использовать хранилище очередей Azure для создания и удаления очередей. Узнайте, как вставлять, получать и удалять сообщения с помощью Node.js.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 12/21/2020
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.custom: seo-javascript-september2019, devx-track-js
ms.openlocfilehash: 12ae05e10cdf0fa9a5f0725acaa1784eedc3612c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97803686"
---
# <a name="how-to-use-azure-queue-storage-from-nodejs"></a>Как использовать хранилище очередей Azure из Node.js

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>Обзор

В этом руководство показано, как выполнять типичные сценарии с использованием хранилища очередей Azure. Примеры написаны с использованием интерфейса API Node.js. К рассматриваемым сценариям относятся Вставка, просмотр, получение и удаление сообщений очереди. Также научитесь создавать и удалять очереди.

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-nodejs-application"></a>Создание приложения Node.js

Сведения о создании пустого Node.jsного приложения см. в статьях [Создание веб-приложения Node.js в службе приложений Azure](../../app-service/quickstart-nodejs.md), создание [и развертывание приложения Node.js в облачных службах Azure](../../cloud-services/cloud-services-nodejs-develop-deploy-app.md) с помощью PowerShell или [Visual Studio Code](https://code.visualstudio.com/docs/nodejs/nodejs-tutorial).

## <a name="configure-your-application-to-access-storage"></a>Настройка приложения для доступа к хранилищу

[Клиентская библиотека службы хранилища Azure для JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage#azure-storage-client-library-for-javascript) содержит набор удобных библиотек, взаимодействующих со службами RESTful службы хранилища.

<!-- docutune:ignore Terminal -->

### <a name="use-node-package-manager-npm-to-obtain-the-package"></a>Использование диспетчера пакетов node (NPM) для получения пакета

1. Используйте интерфейс командной строки, например PowerShell (Windows), Terminal (Mac) или bash (UNIX), перейдите в папку, в которой был создан пример приложения.

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

1. Введите `npm install @azure/storage-queue` в командном окне.

1. Убедитесь, что `node_modules` папка создана. Внутри этой папки вы найдете `@azure/storage-queue` пакет, который содержит клиентскую библиотеку, необходимую для доступа к хранилищу.

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

1. Введите `npm install azure-storage` в командном окне.

1. Убедитесь, что `node_modules` папка создана. Внутри этой папки вы найдете `azure-storage` пакет, который содержит библиотеки, необходимые для доступа к хранилищу.

---

### <a name="import-the-package"></a>Импорт пакета

В редакторе кода добавьте следующий код в начало файла JavaScript, в котором планируется использовать очереди.

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_ImportStatements":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

```javascript
var azure = require('azure-storage');
```

---

## <a name="how-to-create-a-queue"></a>Как создать очередь

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Следующий код возвращает значение переменной среды `AZURE_STORAGE_CONNECTION_STRING` с именем и использует ее для создания [`QueueServiceClient`](/javascript/api/@azure/storage-queue/queueserviceclient) объекта. Затем этот объект используется для создания [`QueueClient`](/javascript/api/@azure/storage-queue/queueclient) объекта, который позволяет работать с определенной очередью.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_CreateQueue":::

Если очередь уже существует, возникает исключение.

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Модуль Azure будет считывать переменные среды `AZURE_STORAGE_ACCOUNT` `AZURE_STORAGE_ACCESS_KEY` , а также сведения, `AZURE_STORAGE_CONNECTION_STRING` необходимые для подключения к учетной записи хранения Azure. Если эти переменные среды не заданы, необходимо указать сведения об учетной записи при вызове `createQueueService` .

Следующий код создает `QueueService` объект, который позволяет работать с очередями.

```javascript
var queueSvc = azure.createQueueService();
```

Вызовите `createQueueIfNotExists` метод, чтобы создать новую очередь с указанным именем или вернуть очередь, если она уже существует.

```javascript
queueSvc.createQueueIfNotExists('myqueue', function(error, results, response){
  if(!error){
    // Queue created or exists
  }
});
```

Если очередь создана, значение `result.created` будет истинно. Если очередь уже существует, значение `result.created` будет ложно.

---

## <a name="how-to-insert-a-message-into-a-queue"></a>Вставка сообщения в очередь

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Чтобы добавить сообщение в очередь, вызовите [`sendMessage`](/javascript/api/@azure/storage-queue/queueclient#sendmessage-string--queuesendmessageoptions-) метод.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_AddMessage":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Чтобы вставить сообщение в очередь, вызовите метод, `createMessage` чтобы создать новое сообщение и добавить его в очередь.

```javascript
queueSvc.createMessage('myqueue', "Hello, World", function(error, results, response){
  if(!error){
    // Message inserted
  }
});
```

---

## <a name="how-to-peek-at-the-next-message"></a>Просмотр следующего сообщения

Вы можете просматривать сообщения в очереди, не удаляя их из очереди, вызывая `peekMessages` метод.

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

По умолчанию [`peekMessages`](/javascript/api/@azure/storage-queue/queueclient#peekmessages-queuepeekmessagesoptions-) просматривает одно сообщение. В следующем примере просматривается первые пять сообщений в очереди. Если отображается менее пяти сообщений, возвращаются только видимые сообщения.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_PeekMessage":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

По умолчанию `peekMessages` просматривает одно сообщение.

```javascript
queueSvc.peekMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
  }
});
```

Его содержит объект `result` .

---

Вызов `peekMessages` , если в очереди нет сообщений, не возвращает ошибку. Однако сообщения не возвращаются.

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Изменение содержимого сообщения в очереди

В следующем примере обновляется текст сообщения.

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Измените содержимое сообщения на месте в очереди, вызвав метод [`updateMessage`](/javascript/api/@azure/storage-queue/queueclient#updatemessage-string--string--string--number--queueupdatemessageoptions-) .

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_UpdateMessage":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Измените содержимое сообщения на месте в очереди, вызвав метод `updateMessage` .

```javascript
queueSvc.getMessages('myqueue', function(error, getResults, getResponse){
  if(!error){
    // Got the message
    var message = getResults[0];
    queueSvc.updateMessage('myqueue', message.messageId, message.popReceipt, 10, {messageText: 'new text'}, function(error, updateResults, updateResponse){
      if(!error){
        // Message updated successfully
      }
    });
  }
});
```

---

## <a name="how-to-dequeue-a-message"></a>Вывод сообщения из очереди

Вывод сообщения из очереди состоит из двух этапов:

1. Получите сообщение.

1. удаление сообщения.

В следующем примере возвращается сообщение, а затем удаляется.

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Чтобы получить сообщение, вызовите [`receiveMessages`](/javascript/api/@azure/storage-queue/queueclient#receivemessages-queuereceivemessageoptions-) метод. Этот вызов делает сообщения невидимыми в очереди, поэтому другие клиенты не могут их обработать. Когда приложение обработало сообщение, вызовите метод, [`deleteMessage`](/javascript/api/@azure/storage-queue/queueclient#deletemessage-string--string--queuedeletemessageoptions-) чтобы удалить его из очереди.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_DequeueMessage":::

По умолчанию сообщение скрыто только в течение 30 секунд. Через 30 секунд она будет видна другим клиентам. Можно указать другое значение, задав [`options.visibilityTimeout`](/javascript/api/@azure/storage-queue/queuereceivemessageoptions#visibilitytimeout) при вызове метода `receiveMessages` .

Вызов `receiveMessages` , если в очереди нет сообщений, не возвращает ошибку. Однако сообщения не возвращаются.

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Чтобы получить сообщение, вызовите `getMessages` метод. Этот вызов делает сообщения невидимыми в очереди, поэтому другие клиенты не могут их обработать. Когда приложение обработало сообщение, вызовите метод, `deleteMessage` чтобы удалить его из очереди.

```javascript
queueSvc.getMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
    var message = results[0];
    queueSvc.deleteMessage('myqueue', message.messageId, message.popReceipt, function(error, response){
      if(!error){
        //message deleted
      }
    });
  }
});
```

По умолчанию сообщение скрыто только в течение 30 секунд. Через 30 секунд она будет видна другим клиентам. Можно указать другое значение, используя `options.visibilityTimeout` WITH `getMessages` .

Использование `getMessages` при отсутствии сообщений в очереди не возвращает ошибку. Однако сообщения не возвращаются.

---

## <a name="additional-options-for-dequeuing-messages"></a>Дополнительные варианты удаления сообщений из очереди

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Существует два способа настройки извлечения сообщения из очереди:

- [`options.numberOfMessages`](/javascript/api/@azure/storage-queue/queuereceivemessageoptions#numberofmessages): Получение пакета сообщений (до 32).
- [`options.visibilityTimeout`](/javascript/api/@azure/storage-queue/queuereceivemessageoptions#visibilitytimeout): Задайте более длительное или короткое время ожидания невидимости.

В следующем примере метод используется `receiveMessages` для получения пяти сообщений в одном вызове. Затем он обрабатывает каждое сообщение с помощью цикла `for`. Он также задает время ожидания невидимости пять минут для всех сообщений, возвращенных методом.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_DequeueMessages":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Существует два способа настройки извлечения сообщения из очереди:

- `options.numOfMessages`: Получение пакета сообщений (до 32).
- `options.visibilityTimeout`: Задайте более длительное или короткое время ожидания невидимости.

В следующем примере метод используется `getMessages` для получения 15 сообщений в одном вызове. Затем он обрабатывает каждое сообщение с помощью цикла `for`. Он также задает время ожидания невидимости пять минут для всех сообщений, возвращенных методом.

```javascript
queueSvc.getMessages('myqueue', {numOfMessages: 15, visibilityTimeout: 5 * 60}, function(error, results, getResponse){
  if(!error){
    // Messages retrieved
    for(var index in results){
      // text is available in result[index].messageText
      var message = results[index];
      queueSvc.deleteMessage(queueName, message.messageId, message.popReceipt, function(error, deleteResponse){
        if(!error){
          // Message deleted
        }
      });
    }
  }
});
```

---

## <a name="how-to-get-the-queue-length"></a>Как получить длину очереди

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

[`getProperties`](/javascript/api/@azure/storage-queue/queueclient#getproperties-queuegetpropertiesoptions-)Метод возвращает метаданные об очереди, включая приблизительное количество сообщений, ожидающих в очереди.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_QueueLength":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

`getQueueMetadata`Метод возвращает метаданные об очереди, включая приблизительное количество сообщений, ожидающих в очереди.

```javascript
queueSvc.getQueueMetadata('myqueue', function(error, results, response){
  if(!error){
    // Queue length is available in results.approximateMessageCount
  }
});
```

---

## <a name="how-to-list-queues"></a>Как получить список очередей

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Чтобы получить список очередей, вызовите [`QueueServiceClient.listQueues`](/javascript/api/@azure/storage-queue/servicelistqueuesoptions#prefix) . Чтобы получить список, отфильтрованный по определенному префиксу, задайте [Параметры. prefix](/javascript/api/@azure/storage-queue/servicelistqueuesoptions#prefix) в вызове `listQueues` .

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_ListQueues":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Чтобы получить список очередей, используйте `listQueuesSegmented` . Чтобы получить список, отфильтрованный по определенному префиксу, используйте `listQueuesSegmentedWithPrefix` .

```javascript
queueSvc.listQueuesSegmented(null, function(error, results, response){
  if(!error){
    // results.entries contains the list of queues
  }
});
```

Если не удается вернуть все очереди, передайте в `result.continuationToken` качестве первого параметра `listQueuesSegmented` или второго параметра `listQueuesSegmentedWithPrefix` в значение, чтобы получить дополнительные результаты.

---

## <a name="how-to-delete-a-queue"></a>Удаление очереди

# <a name="javascript-v12"></a>[JavaScript версии 12](#tab/javascript)

Чтобы удалить очередь и все сообщения, содержащиеся в ней, вызовите [`DeleteQueue`](/javascript/api/@azure/storage-queue/queueclient#delete-queuedeleteoptions-) метод для `QueueClient` объекта.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_DeleteQueue":::

Чтобы очистить все сообщения из очереди, не удаляя их, вызовите [`ClearMessages`](/javascript/api/@azure/storage-queue/queueclient#clearmessages-queueclearmessagesoptions-) .

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Чтобы удалить очередь и все сообщения, содержащиеся в ней, вызовите `deleteQueue` метод для объекта очереди.

```javascript
queueSvc.deleteQueue(queueName, function(error, response){
  if(!error){
    // Queue has been deleted
  }
});
```

Чтобы очистить все сообщения из очереди, не удаляя их, вызовите `clearMessages` .

---

[!INCLUDE [storage-check-out-samples-all](../../../includes/storage-check-out-samples-all.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали основные сведения о хранилище очередей, перейдите по следующим ссылкам, чтобы узнать о более сложных задачах хранилища.

- Посетите [блог команды разработчиков службы хранилища Azure](https://techcommunity.Microsoft.com/t5/Azure-storage/bg-p/azurestorageblog) , чтобы узнать о новых возможностях
- Посетите веб [клиентскую библиотеку службы хранилища Azure для JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage#Azure-storage-client-library-for-JavaScript) в репозитории на сайте GitHub

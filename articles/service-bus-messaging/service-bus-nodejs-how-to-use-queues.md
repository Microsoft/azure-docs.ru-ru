---
title: Использование очередей служебной шины Azure в JavaScript
description: Узнайте, как написать программу JavaScript, которая использует последнюю версию пакета @azure/service-bus для отправки сообщений в очередь служебной шины и получения сообщений из нее.
author: spelluru
ms.author: spelluru
ms.date: 11/09/2020
ms.topic: quickstart
ms.devlang: nodejs
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: aee67becf7519f03839eafbd897838f938871307
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537227"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-javascript"></a>Отправка сообщений в очереди Служебной шины и получение сообщений из них (JavaScript)
Из этого учебника вы узнаете, как использовать пакет [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) в программе JavaScript для отправки и получения сообщений для очереди Служебной шины.

## <a name="prerequisites"></a>Предварительные требования
- Подписка Azure. Для работы с этим учебником требуется учетная запись Azure. Вы можете активировать [преимущества подписчика MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Если у вас нет подходящей очереди служебной шины, [создайте ее с помощью портала Azure](service-bus-quickstart-portal.md). Запишите **строку подключения** к пространству имен Служебной шины и имя созданной **очереди**.

> [!NOTE]
> - Примеры для этого учебника вы можете скопировать и запустить с помощью [Nodejs](https://nodejs.org/). Указания по созданию приложения Node.js можно найти в статьях [Создание веб-приложений Node.js в Azure](../app-service/quickstart-nodejs.md) или [Build and deploy a Node.js application to an Azure Cloud Service](../cloud-services/cloud-services-nodejs-develop-deploy-app.md) (Построение и развертывание приложения Node.js в облачной службе Azure).

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Использование диспетчера пакета Node (NPM) для установки пакета
Чтобы установить пакет npm для служебной шины, откройте командную строку, где `npm` сохранен в переменной окружения path, перейдите в папку, в которой вы хотите сохранить примеры, и выполните эту команду.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-queue"></a>Отправка сообщений в очередь
В следующем примере кода показано, как отправить сообщение в очередь.

1. Откройте редактор, например [Visual Studio Code](https://code.visualstudio.com/).
2. Создайте файл с именем `send.js` и вставьте в него приведенный ниже код. Этот код отправляет сообщение в указанную очередь. Сообщение имеет метку (Ученый) и текст (Эйнштейн).

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus");
    
    // connection string to your Service Bus namespace
    const connectionString = "<CONNECTION STRING TO SERVICE BUS NAMESPACE>"

    // name of the queue
    const queueName = "<QUEUE NAME>"
    
    const messages = [
        { body: "Albert Einstein" },
        { body: "Werner Heisenberg" },
        { body: "Marie Curie" },
        { body: "Steven Hawking" },
        { body: "Isaac Newton" },
        { body: "Niels Bohr" },
        { body: "Michael Faraday" },
        { body: "Galileo Galilei" },
        { body: "Johannes Kepler" },
        { body: "Nikolaus Kopernikus" }
     ];
    
    async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createSender() can also be used to create a sender for a topic.
        const sender = sbClient.createSender(queueName);
     
        try {
            // Tries to send all messages in a single batch.
            // Will fail if the messages cannot fit in a batch.
            // await sender.sendMessages(messages);
     
            // create a batch object
            let batch = await sender.createMessageBatch(); 
            for (let i = 0; i < messages.length; i++) {
                // for each message in the array            
    
                // try to add the message to the batch
                if (!batch.tryAddMessage(messages[i])) {            
                    // if it fails to add the message to the current batch
                    // send the current batch as it is full
                    await sender.sendMessages(batch);
    
                    // then, create a new batch 
                    batch = await sender.createMessageBatch();
     
                    // now, add the message failed to be added to the previous batch to this batch
                    if (!batch.tryAddMessage(messages[i])) {
                        // if it still can't be added to the batch, the message is probably too big to fit in a batch
                        throw new Error("Message too big to fit in a batch");
                    }
                }
            }
    
            // Send the last created batch of messages to the queue
            await sender.sendMessages(batch);
     
            console.log(`Sent a batch of messages to the queue: ${queueName}`);
                    
            // Close the sender
            await sender.close();
        } finally {
            await sbClient.close();
        }
    }
    
    // call the main function
    main().catch((err) => {
        console.log("Error occurred: ", err);
        process.exit(1);
     });
    ```
3. Замените `<CONNECTION STRING TO SERVICE BUS NAMESPACE>` строкой подключения к пространству имен служебной шины.
1. Замените `<QUEUE NAME>` именем очереди. 
1. Затем в окне командной строки выполните команду, которая запускает этот файл.

    ```console
    node send.js 
    ```
1. Вы должны увидеть следующие выходные данные.

    ```console
    Sent a batch of messages to the queue: myqueue
    ```

## <a name="receive-messages-from-a-queue"></a>Получение сообщений из очереди

1. Откройте редактор, например [Visual Studio Code](https://code.visualstudio.com/).
2. Создайте файл с именем `receive.js` и вставьте в него следующий код.

    ```javascript
    const { delay, ServiceBusClient, ServiceBusMessage } = require("@azure/service-bus");

    // connection string to your Service Bus namespace
    const connectionString = "<CONNECTION STRING TO SERVICE BUS NAMESPACE>"

    // name of the queue
    const queueName = "<QUEUE NAME>"

     async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createReceiver() can also be used to create a receiver for a subscription.
        const receiver = sbClient.createReceiver(queueName);
    
        // function to handle messages
        const myMessageHandler = async (messageReceived) => {
            console.log(`Received message: ${messageReceived.body}`);
        };
    
        // function to handle any errors
        const myErrorHandler = async (error) => {
            console.log(error);
        };
    
        // subscribe and specify the message and error handlers
        receiver.subscribe({
            processMessage: myMessageHandler,
            processError: myErrorHandler
        });
    
        // Waiting long enough before closing the sender to send messages
        await delay(5000);
    
        await receiver.close(); 
        await sbClient.close();
    }    
    // call the main function
    main().catch((err) => {
        console.log("Error occurred: ", err);
        process.exit(1);
     });
    ```
3. Замените `<CONNECTION STRING TO SERVICE BUS NAMESPACE>` строкой подключения к пространству имен служебной шины.
1. Замените `<QUEUE NAME>` именем очереди. 
1. Затем в окне командной строки выполните команду, которая запускает этот файл.

    ```console
    node receive.js
    ```
1. Вы должны увидеть следующие выходные данные.

    ```console
    Received message: Albert Einstein
    Received message: Werner Heisenberg
    Received message: Marie Curie
    Received message: Steven Hawking
    Received message: Isaac Newton
    Received message: Niels Bohr
    Received message: Michael Faraday
    Received message: Galileo Galilei
    Received message: Johannes Kepler
    Received message: Nikolaus Kopernikus
    ```

На странице **Обзор** для пространства имен служебной шины в портале Azure можно видеть количество **входящих** и **исходящих** сообщений. Чтобы просмотреть последние значения, возможно, потребуется подождать около минуты, а затем обновить страницу. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Количество входящих и исходящих сообщений":::

Выберите очередь на этой странице **Обзор**, чтобы перейти на страницу **Очередь служебной шины**. На этой странице отображаются количество **входящих** и **исходящих** сообщений. Вы также можете увидеть другие сведения, например **текущий размер** очереди, **максимальный размер**, **количество активных сообщений** и т. д. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/queue-details.png" alt-text="Сведения об очереди":::
## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими примерами и документацией: 

- [Клиентская библиотека Служебной шины Azure для JavaScript](https://www.npmjs.com/package/@azure/service-bus)
- [Примеры JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/)
- [Примеры TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/)
- [Справочная документация по API](/javascript/api/overview/azure/service-bus)

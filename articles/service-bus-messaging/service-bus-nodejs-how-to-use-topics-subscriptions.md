---
title: Использование предварительной версии пакета azure/service-bus для разделов и подписок
description: Узнайте, как написать программу JavaScript, которая использует последнюю предварительную версию пакета @azure/service-bus для отправки сообщений в раздел служебной шины и получения сообщений из подписки в разделе.
author: spelluru
ms.author: spelluru
ms.date: 11/09/2020
ms.topic: quickstart
ms.devlang: nodejs
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: 59f1bb29099f2b921e687ca9de46365bc34f1b91
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537245"
---
# <a name="quickstart-service-bus-topics-and-subscriptions-with-nodejs-and-the-preview-azureservice-bus-package"></a>Краткое руководство. Использование для разделов и подписок служебной шины Node.js и предварительной версией пакета azure/service-bus
Из этого учебника вы узнаете, как использовать пакет [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) в программе JavaScript для отправки сообщений в раздел служебной шины и получения сообщений из подписки служебной шины в этом разделе.

## <a name="prerequisites"></a>Предварительные требования
- Подписка Azure. Для работы с этим учебником требуется учетная запись Azure. Вы можете активировать [преимущества подписчика MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Выполните шаги из [краткого руководства по созданию раздела Служебной шины и подписок на него с помощью портала Azure](service-bus-quickstart-topics-subscriptions-portal.md). Запишите строку подключения, имя раздела и имя подписки. При работе с этим кратким руководством вы будете использовать только одну подписку. 

> [!NOTE]
> - Примеры для этого учебника вы можете скопировать и запустить с помощью [Nodejs](https://nodejs.org/). Указания по созданию приложения Node.js можно найти в статьях [Создание веб-приложений Node.js в Azure](../app-service/quickstart-nodejs.md) или [Build and deploy a Node.js application to an Azure Cloud Service](../cloud-services/cloud-services-nodejs-develop-deploy-app.md) (Построение и развертывание приложения Node.js в облачной службе Azure).

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Использование диспетчера пакета Node (NPM) для установки пакета
Чтобы установить пакет npm для служебной шины, откройте командную строку, где `npm` сохранен в переменной окружения path, перейдите в папку, в которой вы хотите сохранить примеры, и выполните эту команду.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-topic"></a>Отправка сообщений в раздел
В следующем примере кода показано, как отправить пакет сообщений в раздел служебной шины. Дополнительные сведения см. в комментариях к коду. 

1. Откройте редактор, например [Visual Studio Code](https://code.visualstudio.com/).
2. Создайте файл с именем `sendtotopic.js` и вставьте в него приведенный ниже код. Этот код отправляет сообщение в указанный раздел.

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus");
    
    const connectionString = "<SERVICE BUS NAMESPACE CONNECTION STRING>"
    const topicName = "<TOPIC NAME>";
    
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
     
        // createSender() can also be used to create a sender for a queue.
        const sender = sbClient.createSender(topicName);
     
        try {
            // Tries to send all messages in a single batch.
            // Will fail if the messages cannot fit in a batch.
            // await sender.sendMessages(messages);
     
            // create a batch object
            let batch = await sender.createMessageBatch(); 
            for (let i = 0; i < messages.length; i++) {
                // for each message in the arry         
    
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
    
            // Send the last created batch of messages to the topic
            await sender.sendMessages(batch);
     
            console.log(`Sent a batch of messages to the topic: ${topicName}`);
                    
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
3. Замените `<SERVICE BUS NAMESPACE CONNECTION STRING>` строкой подключения к пространству имен Служебной шины.
1. Затем замените `<TOPIC NAME>` именем раздела. 
1. Затем в окне командной строки выполните команду, которая запускает этот файл.

    ```console
    node sendtotopic.js 
    ```
1. Вы должны увидеть следующие выходные данные.

    ```console
    Sent a batch of messages to the topic: mytopic
    ```

## <a name="receive-messages-from-a-subscription"></a>Получение сообщений из подписки
1. Откройте редактор, например [Visual Studio Code](https://code.visualstudio.com/).
2. Создайте файл с именем **receivefromsubscription.js** и вставьте в него следующий код. Дополнительные сведения см. в комментариях к коду. 

    ```javascript
    const { delay, ServiceBusClient, ServiceBusMessage } = require("@azure/service-bus");
    
    const connectionString = "<SERVICE BUS NAMESPACE CONNECTION STRING>"
    const topicName = "<TOPIC NAME>";
    const subscriptionName = "<SUBSCRIPTION NAME>";
    
     async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createReceiver() can also be used to create a receiver for a queue.
        const receiver = sbClient.createReceiver(topicName, subscriptionName);
    
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
3. Замените `<SERVICE BUS NAMESPACE CONNECTION STRING>` строкой подключения к пространству имен. 
1. Затем замените `<TOPIC NAME>` именем раздела. 
1. Замените `<SUBSCRIPTION NAME>` именем подписки на раздел. 
1. Затем в окне командной строки выполните команду, которая запускает этот файл.

    ```console
    node receivefromsubscription.js
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

Перейдите к пространству имен служебной шины на портале Azure и выберите раздел в нижней области, чтобы просмотреть страницу **Раздел служебной шины** своего раздела. На этой странице вы увидите три входящих и три исходящих сообщения в диаграмме **Сообщения**. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/topic-page-portal.png" alt-text="Экран &quot;Входящие и исходящие сообщения&quot;":::

Если в следующий раз запустить только приложение отправки, на странице **Раздел служебной шины** вы увидите шесть входящих сообщений (3 новых),но только три исходящих. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/updated-topic-page.png" alt-text="Экран &quot;Обновленная страница раздела&quot;":::

При выборе подписки на этой странице вы будете перенаправлены на страницу **Service Bus Subscription** (Подписка служебной шины). Также на этой странице можно увидеть количество активных и недоставленных сообщений и многое другое. Из этого примера видно, что получатель еще не получил три активных сообщения. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/active-message-count.png" alt-text="Экран &quot;Количество активных сообщений&quot;":::

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими примерами и документацией: 

- [Клиентская библиотека Служебной шины Azure для Python](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/servicebus/service-bus/README.md)
- [Примеры.](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/samples) В папке **javascript** можно найти примеры для JavaScript, а в папке **typescript** — для TypeScript. 
- [Справочная документация по использованию пакета azure-servicebus](/javascript/api/overview/azure/service-bus)

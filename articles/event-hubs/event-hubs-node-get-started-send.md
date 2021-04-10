---
title: Отправка и получение событий через Центры событий Azure с помощью JavaScript (последних версий)
description: В статье описано, как создать приложение JavaScript, которое отправляет события или получает их через службу "Центры событий Azure" с помощью пакета azure/event-hubs, выпущенного последним.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: c418c13346fb1ec8ba16965fa1020c676ddf3ac6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104602559"
---
# <a name="send-events-to-or-receive-events-from-event-hubs-by-using-javascript--azureevent-hubs"></a>Отправка или получение событий через концентраторы событий с помощью JavaScript (azure/event-hubs)
В этом кратком руководстве показано, как отправлять и получать события через концентратор событий с помощью пакета JavaScript **azure/event-hubs**. 


## <a name="prerequisites"></a>Предварительные требования
Если вы впервые используете Центры событий Azure, ознакомьтесь с общими сведениями в [этой статье](event-hubs-about.md), прежде чем приступить к работе с этим руководством. 

Для работы с данным руководством необходимо следующее:

- **Подписка Microsoft Azure.** Чтобы использовать службы Azure, в том числе Центры событий Azure, потребуется действующая подписка.  Если у вас еще нет учетной записи Azure, [зарегистрируйтесь для работы с бесплатной пробной версией](https://azure.microsoft.com/free/) или [активируйте преимущества для подписчиков MSDN при создании учетной записи](https://azure.microsoft.com).
- Node.js 8.x или более поздней версии. Скачайте последнюю версию [долгосрочной поддержки (LTS)](https://nodejs.org).  
- Visual Studio Code (рекомендовано) или любая другая интегрированная среда разработки (IDE).  
- Активное пространство имен Центров событий и концентратор событий. Чтобы создать их, выполните приведенные ниже действия. 

   1. Создайте на [портале Azure](https://portal.azure.com) пространство имен типа *Центров событий* и получите учетные данные для управления, необходимые приложению для взаимодействия с концентратором событий. 
   1. Чтобы создать пространство имен и концентратор событий, следуйте инструкциям в статье [Краткое руководство. Создание концентратора событий с помощью портала Azure](event-hubs-create.md).
   1. Продолжайте, выполнив инструкции в этом кратком руководстве. 
   1. Чтобы получить строку подключения для пространства имен концентратора событий, следуйте инструкциям в разделе [Get connection string from the portal](event-hubs-get-connection-string.md#get-connection-string-from-the-portal) (Получение строки подключения на портале). Запишите строку подключения для дальнейшего использования в этом руководстве.
- **Создайте пространство имен Центров событий и концентратор событий**. Первым шагом является использование [портала Azure](https://portal.azure.com) для создания пространства имен типа Центров событий и получение учетных данных управления, необходимых приложению для взаимодействия с концентратором событий. Чтобы создать пространство имен и концентратор событий, выполните инструкции из [этой статьи](event-hubs-create.md). Получите **строку подключения для пространства имен Центров событий**, следуя инструкциям из статьи [Получение строки подключения на портале](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Строка подключения понадобится нам позже в рамках этого краткого руководства.

### <a name="install-the-npm-package"></a>Установка пакета npm
Чтобы установить [Пакет Node Package Manager (npm) для Центров событий](https://www.npmjs.com/package/@azure/event-hubs), откройте командную строку, в пути которой сохранен *npm*, перейдите в папку, в которой вы хотите сохранить примеры, и выполните эту команду.

```shell
npm install @azure/event-hubs
```

Для принимающей стороны необходимо установить еще два пакета. В этом кратком руководстве вы используете хранилище BLOB-объектов Azure для сохранения контрольных точек, чтобы программа не читала уже прочитанные события. Оно регулярно выполняет фиксацию метаданных для полученных сообщений в большом двоичном объекте. Благодаря такому подходу позже можно легко продолжить получать сообщения с того места, где вы остановились.

Выполните следующие команды:

```shell
npm install @azure/storage-blob
```

```shell
npm install @azure/eventhubs-checkpointstore-blob
```

## <a name="send-events"></a>Отправка событий

В этом разделе вы создадите приложение JavaScript, которое отправляет события в концентратор событий.

1. Откройте редактор, например [Visual Studio Code](https://code.visualstudio.com).
1. Создайте файл с именем *send.js* и вставьте в него следующий код.

    ```javascript
    const { EventHubProducerClient } = require("@azure/event-hubs");

    const connectionString = "EVENT HUBS NAMESPACE CONNECTION STRING";
    const eventHubName = "EVENT HUB NAME";

    async function main() {

      // Create a producer client to send messages to the event hub.
      const producer = new EventHubProducerClient(connectionString, eventHubName);

      // Prepare a batch of three events.
      const batch = await producer.createBatch();
      batch.tryAdd({ body: "First event" });
      batch.tryAdd({ body: "Second event" });
      batch.tryAdd({ body: "Third event" });    

      // Send the batch to the event hub.
      await producer.sendBatch(batch);

      // Close the producer client.
      await producer.close();

      console.log("A batch of three events have been sent to the event hub");
    }

    main().catch((err) => {
      console.log("Error occurred: ", err);
    });
    ```
1. Используйте в коде реальные значения, чтобы заменить следующее:
    * `EVENT HUBS NAMESPACE CONNECTION STRING` 
    * `EVENT HUB NAME`
1. запустите `node send.js`, чтобы выполнить этот файл. Эта команда отправляет в концентратор событий пакет из трех событий.
1. Проверьте получение сообщения концентратором событий на портале Azure. В разделе **Метрики** переключитесь на представление **Сообщения**. Обновите страницу, чтобы обновить диаграмму. На отображение полученных сообщений может уйти несколько секунд.

    [![Проверка получения сообщения концентратором событий](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png)](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png#lightbox)

    > [!NOTE]
    > Полный исходный код, включая дополнительные информационные комментарии, можно найти на странице [GitHub sendEvents.js](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/event-hubs/samples/javascript/sendEvents.js).

Поздравляем! Вы отправили события в концентратор событий.


## <a name="receive-events"></a>Получение событий
В этом разделе вы получаете события из концентратора событий с помощью хранилища контрольных точек хранилища BLOB-объектов Azure в приложении JavaScript. Оно регулярно создает контрольные точки метаданных для полученных сообщений в Azure Storage blob. Благодаря такому подходу позже можно легко продолжить получать сообщения с того места, где вы остановились.

> [!WARNING]
> Запустив этот код в Azure Stack Hub, вы получите ошибки выполнения, если не укажете конкретную версию API службы хранилища. Это связано с тем, что пакет SDK для Центров событий использует последний доступный в Azure API службы хранилища Azure, который может быть недоступным на вашей платформе Azure Stack Hub. Azure Stack Hub может поддерживать версию пакета SDK для хранилища BLOB-объектов, отличающуюся от доступных в Azure. Если вы используете Хранилище BLOB-объектов Azure в качестве хранилища контрольных точек, проверьте [поддерживаемую версию API службы хранилища Azure для сборки Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) и включите эту версию в код. 
>
> Например, если вы используете Azure Stack Hub версии 2005, последней доступной версией для службы хранилища будет 2019-02-02. По умолчанию клиентская библиотека пакета SDK для Центров событий использует последнюю доступную версию в Azure (2019-07-07 на момент выпуска пакета SDK). В таком случае кроме действий, описанных в этом разделе, вам нужно добавить код для указания API службы хранилища версии 2019-02-02. См. сведения о том, как включить поддержку определенной версии API службы хранилища для примеров [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript/receiveEventsWithApiSpecificStorage.js) и [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript/src/receiveEventsWithApiSpecificStorage.ts) на GitHub. 


### <a name="create-an-azure-storage-account-and-a-blob-container"></a>Создание учетной записи хранения Azure и контейнера больших двоичных объектов
Для создания учетной записи хранения Azure и контейнера больших двоичных объектов в ней, выполните действия в следующих ресурсах.

1. [Create an Azure Storage account](../storage/common/storage-account-create.md?tabs=azure-portal) (Создание учетной записи хранения Azure)  
2. [Создание контейнера](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)  
3. [Получение строки подключения к учетной записи хранения](../storage/common/storage-configure-connection-string.md).

Обязательно запишите строку подключения и имя контейнера для последующего использования в коде получения.

### <a name="write-code-to-receive-events"></a>Написание кода для получения событий

1. Откройте редактор, например [Visual Studio Code](https://code.visualstudio.com).
1. Создайте файл с именем *receive.js* и вставьте в него следующий код.

    ```javascript
    const { EventHubConsumerClient } = require("@azure/event-hubs");
    const { ContainerClient } = require("@azure/storage-blob");    
    const { BlobCheckpointStore } = require("@azure/eventhubs-checkpointstore-blob");

    const connectionString = "EVENT HUBS NAMESPACE CONNECTION STRING";    
    const eventHubName = "EVENT HUB NAME";
    const consumerGroup = "$Default"; // name of the default consumer group
    const storageConnectionString = "AZURE STORAGE CONNECTION STRING";
    const containerName = "BLOB CONTAINER NAME";

    async function main() {
      // Create a blob container client and a blob checkpoint store using the client.
      const containerClient = new ContainerClient(storageConnectionString, containerName);
      const checkpointStore = new BlobCheckpointStore(containerClient);

      // Create a consumer client for the event hub by specifying the checkpoint store.
      const consumerClient = new EventHubConsumerClient(consumerGroup, connectionString, eventHubName, checkpointStore);

      // Subscribe to the events, and specify handlers for processing the events and errors.
      const subscription = consumerClient.subscribe({
          processEvents: async (events, context) => {
            if (events.length === 0) {
              console.log(`No events received within wait time. Waiting for next interval`);
              return;
            }
          
            for (const event of events) {
              console.log(`Received event: '${event.body}' from partition: '${context.partitionId}' and consumer group: '${context.consumerGroup}'`);
            }
            // Update the checkpoint.
            await context.updateCheckpoint(events[events.length - 1]);
          },

          processError: async (err, context) => {
            console.log(`Error : ${err}`);
          }
        }
      );

      // After 30 seconds, stop processing.
      await new Promise((resolve) => {
        setTimeout(async () => {
          await subscription.close();
          await consumerClient.close();
          resolve();
        }, 30000);
      });
    }

    main().catch((err) => {
      console.log("Error occurred: ", err);
    });    
    ```
1. Используйте в коде реальные значения, чтобы заменить следующие значения:
    - `EVENT HUBS NAMESPACE CONNECTION STRING`
    - `EVENT HUB NAME`
    - `AZURE STORAGE CONNECTION STRING`
    - `BLOB CONTAINER NAME`
1. В окне командной строки выполните команду `node receive.js`, которая запускает этот файл. Сообщения о полученных событиях должны отображаться в окне.

    > [!NOTE]
    > Полный исходный код, включая дополнительные информационные комментарии, можно найти на странице [GitHub receiveEventsUsingCheckpointStore](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript/receiveEventsUsingCheckpointStore.js).

Поздравляем! Теперь вы получили события из вашего концентратора событий. Программа-получатель получит события из всех разделов группы потребителей по умолчанию в указанном центре событий.

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с примерами в GitHub:

- [Примеры JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/javascript)
- [Примеры TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/typescript)

---
title: Пересылать события пограничных событий в службу "Сетка событий Azure" в службе "Сетка событий" IoT Edge | Документация Майкрософт
description: Пересылка событий пограничной стороны в облако сетки событий
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: bfe150a45c70bc5bed18f8e929c9567905cd38f5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86171607"
---
# <a name="tutorial-forward-events-to-event-grid-cloud"></a>Учебник. Пересылка событий в облако сетки событий

В этой статье рассматриваются все шаги, необходимые для переадресации пограничных событий в службу "Сетка событий" в облаке Azure. Это может потребоваться по следующим причинам.

* Реагирование на события пограничных событий в облаке.
* Пересылка событий в службу "Сетка событий" в облаке и использование концентраторов событий Azure или очередей службы хранилища Azure для буферизации событий перед их обработкой в облаке.

 Для работы с этим руководством необходимо иметь представление о концепциях службы "Сетка событий" в [пограничных](concepts.md) и [Azure](../concepts.md). Дополнительные типы назначения см. в разделе [обработчики событий](event-handlers.md). 

## <a name="prerequisites"></a>Предварительные требования 
Для выполнения шагов, описанных в данном учебнике, потребуется следующее.

* **Подписка Azure** . Создайте [бесплатную учетную запись](https://azure.microsoft.com/free) , если она еще не создана. 
* **Центр Интернета вещей Azure и устройство IOT Edge** . выполните действия, описанные в разделе Краткое руководство для устройств [Linux](../../iot-edge/quickstart-linux.md) или [Windows](../../iot-edge/quickstart.md) , если у вас его еще нет.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)] 
## <a name="create-event-grid-topic-and-subscription-in-cloud"></a>Создание раздела и подписки в сетке событий в облаке

Создайте раздел и подписку на сетку событий в облаке, следуя [этим руководствам](../custom-event-quickstart-portal.md). Запишите `topicURL` `sasKey` и, и `topicName` из вновь созданного раздела, который будет использоваться далее в этом руководстве.

Например, если вы создали раздел с именем `testegcloudtopic` в западной части США, то значения будут выглядеть примерно следующим образом:

* **Топикурл**: `https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events`
* **Топикнаме**: `testegcloudtopic`
* **Саскэй**: доступно в **AccessKey** вашего раздела. Используйте **Key1**.

## <a name="create-event-grid-topic-at-the-edge"></a>Создать раздел сетки событий на границе

1. Создайте topic3.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

    ```json
        {
          "name": "sampleTopic3",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```
1. Выполните следующую команду, чтобы создать раздел. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```
1. Выполните следующую команду, чтобы проверить, успешно ли создан раздел. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```

   Пример результатов выполнения:

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic3",
            "name": "sampleTopic3",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic3/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```
  
## <a name="create-event-grid-subscription-at-the-edge"></a>Создать подписку на сетку событий на границе

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Создайте subscription3.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

   ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                        "endpointUrl": "<your-event-grid-cloud-topic-endpoint-url>?api-version=2018-01-01",
                        "sasKey": "<your-event-grid-topic-saskey>",
                        "topicName": null
              }
            }
          }
    }
   ```

   >[!NOTE]
   > **EndpointUrl** указывает, что URL-адрес раздела сетки событий в облаке. **Саскэй** относится к разделу "облачная таблица событий". Значение в **топикнаме** будет использоваться для отметки всех исходящих событий на сетку событий. Это может быть полезно при публикации в предметной области сетки событий. Дополнительные сведения о домене "Сетка событий" см. в разделе [домены событий](../event-domains.md) .

    Например,
  
    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                "endpointUrl": "https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01",
                 "sasKey": "<your-event-grid-topic-saskey>",
                 "topicName": null
              }
            }
          }
        }
    ```

2. Выполните следующую команду, чтобы создать подписку. Должен возвращаться код состояния HTTP, 200 ОК.

     ```sh
     curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/eventSubscriptions/sampleSubscription3?api-version=2019-01-01-preview
     ```

3. Выполните следующую команду, чтобы проверить, успешно ли создана подписка. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/eventSubscriptions/sampleSubscription3?api-version=2019-01-01-preview
    ```

    Пример результатов выполнения:

    ```json
         {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic3/eventSubscriptions/sampleSubscription3",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription3",
          "properties": {
            "Topic": "sampleTopic3",
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                "endpointUrl": "https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01",
                "sasKey": "<your-event-grid-topic-saskey>",
                "topicName": null
              }
            }
          }
        }
    ```

## <a name="publish-an-event-at-the-edge"></a>Опубликовать событие на границе

1. Создайте event3.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

    ```json
        [
          {
            "id": "eventId-egcloud-0",
            "eventType": "recordInserted",
            "subject": "myapp/vehicles/motorcycles",
            "eventTime": "2019-07-28T21:03:07+00:00",
            "dataVersion": "1.0",
            "data": {
              "make": "Ducati",
              "model": "Monster"
            }
          }
        ]
    ```

1. Выполните следующую команду.

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/events?api-version=2019-01-01-preview
    ```

## <a name="verify-edge-event-in-cloud"></a>Проверка пограничных событий в облаке

Сведения о просмотре событий, предоставляемых облачными разделами, см. в этом [руководстве](../custom-event-quickstart-portal.md).

## <a name="cleanup-resources"></a>Очистка ресурсов

* Выполните следующую команду, чтобы удалить раздел и все его подписки

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```

* Удалите раздел и подписки, созданные в облаке (сетка событий Azure).

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы опубликовали событие в службе "Сетка событий" в облаке Azure. Теперь, когда вы знакомы с основными шагами для пересылки в сетку событий в облаке, сделайте следующее:

* Сведения об устранении неполадок с использованием службы "Сетка событий Azure" на IoT Edge см. в разделе [руководство по устранению неполадок](troubleshoot.md).
* Пересылать события в IoTHub, следуя этому [учебнику](forward-events-iothub.md)
* Пересылка событий в веб-перехватчик в облаке с помощью этого [руководства](pub-sub-events-webhook-cloud.md)
* [Мониторинг разделов и подписок на границе](monitor-topics-subscriptions.md)

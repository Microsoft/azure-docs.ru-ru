---
title: Публикация, подписка на события в облаке — сетка событий Azure IoT Edge | Документация Майкрософт
description: Публикация, подписка на события в облаке с помощью веб-перехватчика со службой "Сетка событий" на IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: ec7ecb77d37ed1cdf1d13aa7191f5d50e0008c20
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790805"
---
# <a name="tutorial-publish-subscribe-to-events-in-cloud"></a>Учебник. Публикация, подписка на события в облаке

В этой статье рассматриваются все шаги, необходимые для публикации и подписки на события с помощью сетки событий на IoT Edge. В этом руководстве используется и функция Azure в качестве обработчика событий. Дополнительные типы назначения см. в разделе [обработчики событий](event-handlers.md).

Прежде чем продолжать, ознакомьтесь со статьей [Основные понятия сетки событий](concepts.md) .

## <a name="prerequisites"></a>Предварительные условия 
Для выполнения шагов, описанных в данном учебнике, потребуется следующее.

* **Подписка Azure** . Создайте [бесплатную учетную запись](https://azure.microsoft.com/free) , если она еще не создана. 
* **Центр Интернета вещей Azure и устройство IOT Edge** . выполните действия, описанные в разделе Краткое руководство для устройств [Linux](../../iot-edge/quickstart-linux.md) или [Windows](../../iot-edge/quickstart.md) , если у вас его еще нет.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)]

## <a name="create-an-azure-function-in-the-azure-portal"></a>Создание функции Azure в портал Azure

Выполните действия, описанные в этом [руководстве](../../azure-functions/functions-get-started.md) , чтобы создать функцию Azure. 

Замените фрагмент кода следующим кодом:

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);

    log.LogInformation($"C# HTTP trigger received {data}.");
    return data != null
        ? (ActionResult)new OkResult()
        : new BadRequestObjectResult("Please pass in the request body");
}
```

В новой функции выберите **получить URL-адрес функции** в правом верхнем углу, выберите значение по умолчанию (**функциональная клавиша**), а затем щелкните **Копировать**. Значение URL-адреса функции будет использоваться позже в этом руководстве.

> [!NOTE]
> Дополнительные примеры и руководства по выполнению событий с помощью триггеров событий EventGrid см. в документации по [функциям Azure](../../azure-functions/functions-overview.md) .

## <a name="create-a-topic"></a>Создание раздела

Как издатель события, необходимо создать раздел «Сетка событий». Раздел относится к конечной точке, в которой издатели могут отсылать события.

1. Создайте topic2.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

    ```json
         {
          "name": "sampleTopic2",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```
1. Выполните следующую команду, чтобы создать раздел. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```
1. Выполните следующую команду, чтобы проверить, успешно ли создан раздел. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```

   Пример результатов выполнения:

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic2",
            "name": "sampleTopic2",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic2/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-an-event-subscription"></a>Создание подписки на событие

Подписчики могут регистрироваться для событий, опубликованных в разделе. Чтобы получить любое событие, подписчикам нужно будет создать подписку на сетку событий в интересующей теме.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Создайте subscription2.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-az-func-cloud-url>"
              }
            }
          }
        }
    ```

   >[!NOTE]
   > **EndpointType** указывает, что подписчик является веб-перехватчиком.  **EndpointUrl** указывает URL-адрес, по которому подписчик прослушивает события. Этот URL-адрес соответствует образцу функции Azure, который вы настроили ранее.
2. Выполните следующую команду, чтобы создать подписку. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/eventSubscriptions/sampleSubscription2?api-version=2019-01-01-preview
    ```
3. Выполните следующую команду, чтобы проверить, успешно ли создана подписка. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/eventSubscriptions/sampleSubscription2?api-version=2019-01-01-preview
    ```

    Пример результатов выполнения:

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic2/eventSubscriptions/sampleSubscription2",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription2",
          "properties": {
            "Topic": "sampleTopic2",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-az-func-cloud-url>"
              }
            }
          }
        }
    ```

## <a name="publish-an-event"></a>Публикация события

1. Создайте event2.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

    ```json
        [
          {
            "id": "eventId-func-1",
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
1. Выполните следующую команду для публикации события

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Проверка доставки событий

Вы можете просмотреть событие, доставленное в портал Azure в разделе " **мониторинг** " функции.

## <a name="cleanup-resources"></a>Очистка ресурсов

* Выполните следующую команду, чтобы удалить раздел и все его подписки

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```

* Удалите функцию Azure, созданную в портал Azure.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы создали раздел, подписку и опубликованные события сетки событий. Теперь, когда вы знакомы с основными шагами, ознакомьтесь со следующими статьями:

* Сведения об устранении неполадок с использованием службы "Сетка событий Azure" на IoT Edge см. в разделе [руководство по устранению неполадок](troubleshoot.md).
* Создайте или обновите подписку с помощью [фильтров](advanced-filtering.md).
* Настройка сохраняемости модуля службы "Сетка событий" в [Linux](persist-state-linux.md) или [Windows](persist-state-windows.md)
* Следуйте инструкциям [по](configure-client-auth.md) настройке проверки подлинности клиента
* Пересылка событий в службу "Сетка событий Azure" в облаке, следуя этому [учебнику](forward-events-event-grid-cloud.md)
* [Мониторинг разделов и подписок на границе](monitor-topics-subscriptions.md)
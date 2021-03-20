---
title: Публикация, подписка на события локально — сетка событий Azure IoT Edge | Документация Майкрософт
description: Публикация, подписка на события локально с помощью веб-перехватчика со службой "Сетка событий" на IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 2a7cc864366bd9a35c96dd453c0dc68f77d8abd9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86171454"
---
# <a name="tutorial-publish-subscribe-to-events-locally"></a>Учебник. Публикация, подписка на события локально

В этой статье рассматриваются все шаги, необходимые для публикации событий и подписки на них с помощью сетки событий на IoT Edge.

> [!NOTE]
> Сведения о разделах и подписках службы "Сетка событий Azure" см. в разделе [Основные понятия сетки событий](concepts.md).

## <a name="prerequisites"></a>Предварительные требования 
Для выполнения шагов, описанных в данном учебнике, потребуется следующее.

* **Подписка Azure** . Создайте [бесплатную учетную запись](https://azure.microsoft.com/free) , если она еще не создана. 
* **Центр Интернета вещей Azure и устройство IOT Edge** . выполните действия, описанные в кратком руководстве для устройств [Linux](../../iot-edge/quickstart-linux.md) или [Windows](../../iot-edge/quickstart.md) , если у вас его еще нет.

## <a name="deploy-event-grid-iot-edge-module"></a>Развертывание модуля IoT Edge сетки событий

Существует несколько способов развертывания модулей на IoT Edge устройстве, и все они работают в службе "Сетка событий Azure" на IoT Edge. В этой статье описаны шаги по развертыванию сетки событий на IoT Edge из портал Azure.

>[!NOTE]
> В этом руководстве вы развернете модуль службы "Сетка событий" без сохранения. Это означает, что все разделы и подписки, создаваемые в этом учебнике, будут удалены при повторном развертывании модуля. Дополнительные сведения о настройке сохраняемости см. в следующих статьях: [Сохранение состояния в Linux](persist-state-linux.md) или [Сохранение состояния в Windows](persist-state-windows.md). Для рабочих нагрузок рекомендуется установить модуль службы "Сетка событий" с сохраняемостью.


### <a name="select-your-iot-edge-device"></a>Выберите устройство IoT Edge

1. Войдите на [портал Azure](https://portal.azure.com)
1. Перейдите в Центр Интернета вещей.
1. Выберите **IOT Edge** в меню в разделе **Автоматическое управление устройствами** . 
1. Щелкните идентификатор целевого устройства в списке устройств.
1. Выберите **задать модули**. Не закрывайте страницу. Шаги, описанные в следующем разделе, будут продолжены.

### <a name="configure-a-deployment-manifest"></a>Настройка манифеста развертывания

Манифест развертывания — это документ JSON, в котором определены развертываемые модули, способ передачи данных между этими модулями и требуемые свойства для двойников модулей. В портал Azure имеется мастер, который поможет вам создать манифест развертывания вместо создания документа JSON вручную.  В этом мастере есть три шага: **Добавление модулей**, **Указание маршрутов** и **Просмотр развертываний**.

### <a name="add-modules"></a>Добавление модулей

1. В разделе **модули развертывания** выберите **Добавить** .
1. В раскрывающемся списке типы модулей выберите **IOT Edge Module (модуль** ).
1. Укажите параметры "имя", "изображение", "создать контейнер" для контейнера:

   * **Имя**: евентгридмодуле
   * **URI изображения**: `mcr.microsoft.com/azure-event-grid/iotedge:latest`
   * **Параметры создания контейнера**:

   [!INCLUDE [event-grid-edge-module-version-update](../../../includes/event-grid-edge-module-version-update.md)]

    ```json
        {
          "Env": [
            "inbound__clientAuth__clientCert__enabled=false"
          ],
          "HostConfig": {
            "PortBindings": {
              "4438/tcp": [
                {
                    "HostPort": "4438"
                }
              ]
            }
          }
        }
    ```    
 1. Щелкните **Сохранить**.
 1. Перейдите к следующему разделу, чтобы добавить модуль подписчика службы "Сетка событий Azure", прежде чем развертывать их вместе.

    >[!IMPORTANT]
    > В этом руководстве вы развернете модуль сетки событий с отключенной проверкой подлинности клиента. Для рабочих нагрузок рекомендуется включить проверку подлинности клиента. Дополнительные сведения о безопасной настройке модуля сетки событий см. в разделе [безопасность и проверка подлинности](security-authentication.md).
    > 
    > Если вы используете виртуальную машину Azure в качестве пограничной устройства, добавьте правило для входящих портов, разрешающее входящий трафик через порт 4438. Инструкции по добавлению правила см. в статье [как открыть порты для виртуальной машины](../../virtual-machines/windows/nsg-quickstart-portal.md).
    

## <a name="deploy-event-grid-subscriber-iot-edge-module"></a>Развертывание модуля IoT Edge подписчика для сетки событий

В этом разделе показано, как развернуть другой модуль IoT, который будет использоваться в качестве обработчика событий, в который могут доставляться события.

### <a name="add-modules"></a>Добавление модулей

1. В разделе **модули развертывания** выберите **Добавить** еще раз. 
1. В раскрывающемся списке типы модулей выберите **IOT Edge Module (модуль** ).
1. Укажите параметры имя, образ и контейнер для создания контейнера:

   * **Имя**: подписчик
   * **URI изображения**: `mcr.microsoft.com/azure-event-grid/iotedge-samplesubscriber:latest`
   * **Параметры создания контейнера**: нет
1. Щелкните **Сохранить**.
1. Нажмите кнопку " **Далее** ", чтобы перейти к разделу "маршруты"

 ### <a name="setup-routes"></a>Настройка маршрутов

Сохранить маршруты по умолчанию и нажать кнопку **Далее** , чтобы перейти к разделу "Проверка"

### <a name="submit-the-deployment-request"></a>Отправка запроса на развертывание

1. В разделе "Проверка" показан манифест развертывания JSON, созданный на основе параметров, сделанных в предыдущем разделе. Убедитесь, что в JSON отображаются оба модуля: **евентгридмодуле** и **подписчик** . 
1. Просмотрите сведения о развертывании, а затем выберите **Отправить**. После отправки развертывания вы вернетесь на страницу **устройства** .
1. Убедитесь, что в **разделе модули** указаны оба модуля: **eventgrid** и **подписчик** . И убедитесь, что для столбцов, **указанных в** параметре развертывание и **отчеты по** столбцам устройства, задано значение **Да**.

    Запуск модуля на устройстве может занять несколько секунд. Затем информация о нем передается в Центр Интернета вещей. Обновите страницу, чтобы увидеть обновленное состояние.

## <a name="create-a-topic"></a>Создание раздела

Как издатель события, необходимо создать раздел «Сетка событий». В службе "Сетка событий Azure" раздел ссылается на конечную точку, в которой издатели могут отсылать события.

1. Создайте topic.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md).

    ```json
        {
          "name": "sampleTopic1",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```

1. Выполните следующую команду, чтобы создать раздел "Сетка событий". Убедитесь, что отображается код состояния HTTP `200 OK` .

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```

1. Выполните следующую команду, чтобы проверить, успешно ли создан раздел. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```

   Пример результатов выполнения:

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic1",
            "name": "sampleTopic1",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic1/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-an-event-subscription"></a>Создание подписки на событие

Подписчики могут регистрироваться для событий, опубликованных в разделе. Чтобы получить любое событие, необходимо создать подписку на сетку событий для интересующей темы.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Создайте subscription.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
    ```

    >[!NOTE]
    > Свойство **endpointType** указывает, что подписчик является веб- **перехватчиком**.  **EndpointUrl** указывает URL-адрес, по которому подписчик прослушивает события. Этот URL-адрес соответствует образцу подписчика Azure, который вы развернули ранее.
2. Выполните следующую команду, чтобы создать подписку для раздела. Убедитесь, что отображается код состояния HTTP `200 OK` .

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/eventSubscriptions/sampleSubscription1?api-version=2019-01-01-preview
    ```
3. Выполните следующую команду, чтобы проверить, успешно ли создана подписка. Должен возвращаться код состояния HTTP, 200 ОК.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/eventSubscriptions/sampleSubscription1?api-version=2019-01-01-preview
    ```

    Пример результатов выполнения:

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic1/eventSubscriptions/sampleSubscription1",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription1",
          "properties": {
            "Topic": "sampleTopic1",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
    ```

## <a name="publish-an-event"></a>Публикация события

1. Создайте event.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md).

    ```json
        [
          {
            "id": "eventId-func-0",
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
1. Выполните следующую команду, чтобы опубликовать событие.

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Проверка доставки событий

1. SSH или RDP в IoT Edge виртуальной машине.
1. Проверьте журналы подписчика:

    В Windows выполните следующую команду:

    ```sh
    docker -H npipe:////./pipe/iotedge_moby_engine container logs subscriber
    ```

   В Linux выполните следующую команду:

    ```sh
    sudo docker logs subscriber
    ```

    Пример результатов выполнения:

    ```sh
        Received Event:
            {
              "id": "eventId-func-0",
              "topic": "sampleTopic1",
              "subject": "myapp/vehicles/motorcycles",
              "eventType": "recordInserted",
              "eventTime": "2019-07-28T21:03:07+00:00",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "make": "Ducati",
                "model": "Monster"
              }
            }
    ```

## <a name="cleanup-resources"></a>Очистка ресурсов

* Выполните следующую команду, чтобы удалить раздел и все его подписки.

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```
* Удалите модуль подписчика с устройства IoT Edge.


## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве вы создали раздел, подписку и опубликованные события сетки событий. Теперь, когда вы знакомы с основными шагами, ознакомьтесь со следующими статьями: 

- Сведения об устранении неполадок с использованием службы "Сетка событий Azure" на IoT Edge см. в разделе [руководство по устранению неполадок](troubleshoot.md).
- Создайте или обновите подписку с помощью [фильтров](advanced-filtering.md).
- Включение сохраняемости модуля службы "Сетка событий" в [Linux](persist-state-linux.md) или [Windows](persist-state-windows.md)
- Следуйте инструкциям [по](configure-client-auth.md) настройке проверки подлинности клиента
- Пересылка событий в функции Azure в облаке с помощью этого [руководства](pub-sub-events-webhook-cloud.md)
- [Реагирование на события хранилища BLOB-объектов в IoT Edge](react-blob-storage-events-locally.md)
- [Мониторинг разделов и подписок на границе](monitor-topics-subscriptions.md)


---
title: Реагирование на события модуля хранилища BLOB-объектов — Служба "Сетка событий Azure" IoT Edge | Документация Майкрософт
description: Реагирование на события модуля хранилища BLOB-объектов
author: arduppal
manager: brymat
ms.author: arduppal
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 230e158a970f8c815b1575403c013e30749124c5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96005067"
---
# <a name="tutorial-react-to-blob-storage-events-on-iot-edge-preview"></a>Учебник. реагирование на события хранилища BLOB-объектов на IoT Edge (Предварительная версия)
В этой статье показано, как развернуть хранилище BLOB-объектов Azure в модуле IoT, который будет служить издателем службы "Сетка событий" для отправки событий при создании большого двоичного объекта и удалении большого двоичного объекта в службе "Сетка событий".  

Общие сведения о хранилище BLOB-объектов Azure на IoT Edge см. в статье [хранилище BLOB-объектов Azure на IOT Edge](../../iot-edge/how-to-store-data-blob.md) и его функциях.

> [!WARNING]
> Хранилище BLOB-объектов Azure в IoT Edge интеграция с сеткой событий находится на этапе предварительной версии

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

    ```json
        {
          "Env": [
           "inbound__serverAuth__tlsPolicy=enabled",
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
    > В этом руководстве вы узнаете, как развернуть модуль "Сетка событий", чтобы разрешить запросы HTTP/HTTPs и отключить проверку подлинности клиента. Для рабочих нагрузок рекомендуется включить проверку подлинности клиента только по протоколам HTTPs и подписчикам. Дополнительные сведения о безопасной настройке модуля сетки событий см. в разделе [безопасность и проверка подлинности](security-authentication.md).
    

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
1. Перейдите к следующему разделу, чтобы добавить модуль хранилища BLOB-объектов Azure.

## <a name="deploy-azure-blob-storage-module"></a>Развертывание модуля хранилища BLOB-объектов Azure

В этом разделе показано, как развернуть модуль хранилища BLOB-объектов Azure, который будет действовать как издатель службы "Сетка событий", который создает и удаляет события.

### <a name="add-modules"></a>Добавление модулей

1. В разделе **модули развертывания** выберите **Добавить** .
2. В раскрывающемся списке типы модулей выберите **IOT Edge Module (модуль** ).
3. Укажите параметры имя, образ и контейнер для создания контейнера:

   * **Имя**: азуреблобсторажеониотедже
   * **URI изображения**: MCR.Microsoft.com/Azure-Blob-Storage:Latest
   * **Параметры создания контейнера**:

   ```json
       {
         "Env":[
           "LOCAL_STORAGE_ACCOUNT_NAME=<your storage account name>",
           "LOCAL_STORAGE_ACCOUNT_KEY=<your storage account key>",
           "EVENTGRID_ENDPOINT=http://<event grid module name>:5888"
         ],
         "HostConfig":{
           "Binds":[
               "<storage mount>"
           ],
           "PortBindings":{
             "11002/tcp":[{"HostPort":"11002"}]
           }
         }
       }
   ```

   > [!IMPORTANT]
   > - Модуль хранилища BLOB-объектов может публиковать события с помощью HTTPS и HTTP. 
   > - Если вы включили проверку подлинности на основе клиента для EventGrid, убедитесь, что вы обновляете значение EVENTGRID_ENDPOINT, чтобы разрешить HTTPS, например: `EVENTGRID_ENDPOINT=https://<event grid module name>:4438` .
   > - Кроме того, добавьте еще одну переменную среды `AllowUnknownCertificateAuthority=true` в приведенный выше код JSON. При взаимодействии с EventGrid по протоколу HTTPS **алловункновнцертификатеаусорити** позволяет модулю хранилища доверять самозаверяющим сертификатам EventGrid Server.

4. Обновите в скопированном JSON-файле следующие сведения:

   - Замените `<your storage account name>` запоминаемым именем. Длина имени учетной записи должна составлять от 3 до 24 символов, а строчные буквы и цифры. Нельзя использовать пробелы.

   - Замените `<your storage account key>` на 64-байтовый ключ Base64. Вы можете создать ключ с помощью таких средств, как [GeneratePlus](https://generate.plus/en/base64?gp_base64_base[length]=64). Вы будете использовать эти учетные данные для доступа к хранилищу BLOB-объектов из других модулей.

   - Замените на `<event grid module name>` имя модуля сетки событий.
   - Замените в `<storage mount>` соответствии с операционной системой контейнера.
     - Для контейнеров Linux **My-Volume:/блобрут**
     - Для контейнеров Windows **My-Volume: C:/блобрут**

5. Щелкните **Сохранить**.
6. Нажмите кнопку " **Далее** ", чтобы перейти к разделу "маршруты"

    > [!NOTE]
    > Если вы используете виртуальную машину Azure в качестве пограничной устройства, добавьте правило для входящих портов, разрешающее входящий трафик на портах узлов, используемых в этом учебнике: 4438, 5888, 8080 и 11002. Инструкции по добавлению правила см. в статье [как открыть порты для виртуальной машины](../../virtual-machines/windows/nsg-quickstart-portal.md).

### <a name="setup-routes"></a>Настройка маршрутов

Сохранить маршруты по умолчанию и нажать кнопку **Далее** , чтобы перейти к разделу "Проверка"

### <a name="review-deployment"></a>Проверка развертывания

1. В разделе "Проверка" показан манифест развертывания JSON, созданный на основе параметров, сделанных в предыдущем разделе. Убедитесь, что вы видите следующие четыре модуля: **$edgeAgent**, **$edgeHub**, **евентгридмодуле**, **Subscriber** и **азуреблобсторажеониотедже** , которые развертываются.
2. Просмотрите сведения о развертывании, а затем выберите **Отправить**.

## <a name="verify-your-deployment"></a>Проверка развертывания

1. Сразу после отправки развертывания можно вернуться на страницу IoT Edge Центра Интернета вещей.
2. Выберите **устройство IOT Edge** , для которого предназначается развертывание, чтобы открыть сведения о нем.
3. В сведениях об устройстве убедитесь, что модули евентгридмодуле, подписчик и азуреблобсторажеониотедже указаны как **указанные в развертывании** и **переданы по устройствам**.

   Запуск модуля на устройстве может занять несколько секунд. Затем информация о нем передается в Центр Интернета вещей. Обновите страницу, чтобы увидеть обновленное состояние.

## <a name="publish-blobcreated-and-blobdeleted-events"></a>Публикация событий BlobCreated и BlobDeleted

1. Этот модуль автоматически создает раздел **микрософтстораже**. Убедитесь, что он существует
    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview
    ```

    Пример результатов выполнения:

    ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/MicrosoftStorage",
            "name": "MicrosoftStorage",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/MicrosoftStorage/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
    ```

    > [!IMPORTANT]
    > - Для потока HTTPS, если проверка подлинности клиента включена через ключ SAS, указанный ранее ключ SAS следует добавить в качестве заголовка. Следовательно, запрос с фигурой будет следующим: `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview`
    > - Для потока HTTPS, если проверка подлинности клиента включена через сертификат, запрос на фигуру будет следующим: `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview`

2. Подписчики могут регистрироваться для событий, опубликованных в разделе. Чтобы получить любое событие, необходимо создать подписку на сетку событий для **микрософтстораже** .
    1. Создайте blobsubscription.jsсо следующим содержимым. Дополнительные сведения о полезных данных см. в [документации по API](api.md) .

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
       > Свойство **endpointType** указывает, что подписчик является веб- **перехватчиком**.  **EndpointUrl** указывает URL-адрес, по которому подписчик прослушивает события. Этот URL-адрес соответствует образцу функции Azure, который вы развернули ранее.

    2. Выполните следующую команду, чтобы создать подписку для раздела. Убедитесь, что отображается код состояния HTTP `200 OK` .

       ```sh
       curl -k -H "Content-Type: application/json" -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview
       ```

       > [!IMPORTANT]
       > - Для потока HTTPS, если проверка подлинности клиента включена через ключ SAS, указанный ранее ключ SAS следует добавить в качестве заголовка. Следовательно, запрос с фигурой будет следующим: `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview` 
       > - Для потока HTTPS, если проверка подлинности клиента включена через сертификат, запрос на фигуру будет следующим:`curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`

    3. Выполните следующую команду, чтобы проверить, успешно ли создана подписка. Должен возвращаться код состояния HTTP, 200 ОК.

       ```sh
       curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview
       ```

       Пример результатов выполнения:

       ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription5",
          "properties": {
            "Topic": "MicrosoftStorage",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
       ```

       > [!IMPORTANT]
       > - Для потока HTTPS, если проверка подлинности клиента включена через ключ SAS, указанный ранее ключ SAS следует добавить в качестве заголовка. Следовательно, запрос с фигурой будет следующим: `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`
       > - Для потока HTTPS, если проверка подлинности клиента включена через сертификат, запрос на фигуру будет следующим: `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`

3. Скачайте [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/) и [подключите его к локальному хранилищу](../../iot-edge/how-to-store-data-blob.md#connect-to-your-local-storage-with-azure-storage-explorer)

## <a name="verify-event-delivery"></a>Проверка доставки событий

### <a name="verify-blobcreated-event-delivery"></a>Проверка доставки событий BlobCreated

1. Отправьте файлы как блочные BLOB-объекты в локальное хранилище из Обозреватель службы хранилища Azure, и модуль будет автоматически публиковать события создания. 
2. Просмотрите журналы подписчика для создания события. Выполните действия для [проверки доставки событий](pub-sub-events-webhook-local.md#verify-event-delivery) .

    Вывод образца:

    ```json
            Received Event:
            {
              "id": "d278f2aa-2558-41aa-816b-e6d8cc8fa140",
              "topic": "MicrosoftStorage",
              "subject": "/blobServices/default/containers/cont1/blobs/Team.jpg",
              "eventType": "Microsoft.Storage.BlobCreated",
              "eventTime": "2019-10-01T21:35:17.7219554Z",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "api": "PutBlob",
                "clientRequestId": "00000000-0000-0000-0000-000000000000",
                "requestId": "ef1c387b-4c3c-4ac0-8e04-ff73c859bfdc",
                "eTag": "0x8D746B740DA21FB",
                "url": "http://azureblobstorageoniotedge:11002/myaccount/cont1/Team.jpg",
                "contentType": "image/jpeg",
                "contentLength": 858129,
                "blobType": "BlockBlob"
              }
            }
    ```

### <a name="verify-blobdeleted-event-delivery"></a>Проверка доставки событий BlobDeleted

1. Удалите BLOB-объекты из локального хранилища с помощью Обозреватель службы хранилища Azure, и модуль будет автоматически публиковать события удаления. 
2. Просмотрите журналы подписчика для события удаления. Выполните действия для [проверки доставки событий](pub-sub-events-webhook-local.md#verify-event-delivery) .

    Вывод образца:
    
    ```json
            Received Event:
            {
              "id": "ac669b6f-8b0a-41f3-a6be-812a3ce6ac6d",
              "topic": "MicrosoftStorage",
              "subject": "/blobServices/default/containers/cont1/blobs/Team.jpg",
              "eventType": "Microsoft.Storage.BlobDeleted",
              "eventTime": "2019-10-01T21:36:09.2562941Z",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "api": "DeleteBlob",
                "clientRequestId": "00000000-0000-0000-0000-000000000000",
                "requestId": "2996bbfb-c819-4d02-92b1-c468cc67d8c6",
                "eTag": "0x8D746B740DA21FB",
                "url": "http://azureblobstorageoniotedge:11002/myaccount/cont1/Team.jpg",
                "contentType": "image/jpeg",
                "contentLength": 858129,
                "blobType": "BlockBlob"
              }
            }
    ```

Поздравляем! Вы завершили работу с руководством. В следующих разделах содержатся сведения о свойствах событий.

### <a name="event-properties"></a>Свойства события

Ниже приведен список поддерживаемых свойств событий, их типов и описаний. 

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| Раздел | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| subject | строка | Определенный издателем путь к субъекту событий. |
| eventType | строка | Один из зарегистрированных типов событий для этого источника событий. |
| eventTime | строка | Время создания события с учетом времени поставщика в формате UTC. |
| идентификатор | строка | Уникальный идентификатор события. |
| . | объект | Данные события хранилища BLOB-объектов. |
| dataVersion | строка | Версия схемы для объекта данных. Версию схемы определяет издатель. |
| metadataVersion | строка | Версия схемы для метаданных события. Служба "Сетка событий" определяет схему свойств верхнего уровня. Это значение предоставляет служба "Сетка событий". |

Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| api | строка | Операция, вызвавшая событие. Может иметь одно из следующих значений. <ul><li>BlobCreated — допустимые значения: `PutBlob` и `PutBlockList`</li><li>BlobDeleted — допустимые значения: `DeleteBlob` , `DeleteAfterUpload` и `AutoDelete` . <p>`DeleteAfterUpload`Событие создается при автоматическом удалении большого двоичного объекта, так как требуемое свойство делетеафтеруплоад имеет значение true. </p><p>`AutoDelete` событие создается при автоматическом удалении большого двоичного объекта из-за истечения срока действия требуемого значения свойства Делетеафтерминутес.</p></li></ul>|
| clientRequestId | строка | предоставленный клиентом идентификатор запроса для операции API хранилища. Этот идентификатор можно использовать для сопоставления с журналами диагностики службы хранилища Azure с помощью поля "Client-Request-ID" в журналах и может быть предоставлено в запросах клиента с помощью заголовка "x-MS-Client-Request-ID". Дополнительные сведения см. в разделе [Формат журнала](/rest/api/storageservices/storage-analytics-log-format). |
| requestId | строка | Идентификатор запроса, формируемый службой для операции API хранилища. Может использоваться для корреляции журналов диагностики службы хранилища Azure с помощью поля request-id-header в журналах. Возвращается при инициации вызова API в заголовке x-ms-request-id. Ознакомьтесь со статьей [Storage Analytics Log Format](/rest/api/storageservices/storage-analytics-log-format) (Формат журналов Аналитики Службы хранилища). |
| eTag | строка | Значение, которое позволяет выполнять операции условно. |
| сontentType | строка | Тип содержимого, указанный для BLOB-объекта. |
| contentLength | Целое число | Размер большого двоичного объекта в байтах. |
| blobType | строка | Тип большого двоичного объекта. Допустимые значения: BlockBlob или PageBlob. |
| url | строка | Путь к BLOB-объекту. <br>Если клиент использует REST API больших двоичных объектов, URL-адрес имеет следующую структуру: *\<storage-account-name\> . \<container-name\> / \<file-name\> BLOB.Core.Windows.NET/*. <br>Если клиент использует Data Lake Storage REST API, URL-адрес имеет следующую структуру: *\<storage-account-name\> . DFS.Core.Windows.NET/ \<file-system-name\> / \<file-name\>*. |


## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь со следующими статьями в документации по хранилищу BLOB-объектов:

- [Фильтрация событий хранилища BLOB-объектов](../../storage/blobs/storage-blob-event-overview.md#filtering-events)
- [Рекомендуемые методики использования событий хранилища BLOB-объектов](../../storage/blobs/storage-blob-event-overview.md#practices-for-consuming-events)

В этом руководстве вы опубликовали события, создавая или удаляя большие двоичные объекты в хранилище BLOB-объектов Azure. Дополнительные сведения о пересылке событий в облако (концентратор событий Azure или центр Интернета вещей Azure) см. в других руководствах. 

- [Переадресация событий в Сетку событий Azure](forward-events-event-grid-cloud.md)
- [Переадресация событий в Сетку событий Azure](forward-events-iothub.md)

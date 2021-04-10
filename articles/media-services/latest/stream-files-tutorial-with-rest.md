---
title: Кодирование и потоковая передача файла из удаленного расположения с помощью Служб мультимедиа
description: Следуйте инструкциям в этом руководстве, чтобы закодировать файл на основе URL-адреса и потоковой передачи содержимого с помощью Служб мультимедиа Azure и REST.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 03/17/2021
ms.author: inhenkel
ms.openlocfilehash: 139970bb043c745d63f2ef795ae1c8aef4bda0fa
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106108136"
---
# <a name="tutorial-encode-a-remote-file-based-on-url-and-stream-the-video---rest"></a>Руководство по Кодирование удаленного файла на основе URL-адреса и потоковой передачи видео с помощью REST

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Службы мультимедиа Azure позволяют кодировать файлы мультимедиа в разные форматы, пригодные для воспроизведения в разных браузерах и на различных устройствах. Например, можно организовать потоковую передачу содержимого в форматах HLS или MPEG DASH от Apple. Перед тем как передавать файл мультимедиа высокого качества, его нужно закодировать. Рекомендации по кодировке см. в статье [Encoding with Azure Media Services](encode-concept.md) (Кодирование в Службах мультимедиа Azure).

В этом руководстве показано, как кодировать файлы на основе URL-адреса и потоковой передачи видео с помощью Служб мультимедиа Azure и REST. 

![Воспроизведение видео](./media/stream-files-tutorial-with-api/final-video.png)

В этом учебнике описаны следующие процедуры.    

> [!div class="checklist"]
> * Создание учетной записи служб мультимедиа
> * Доступ к API Служб мультимедиа.
> * Скачивание файлов Postman
> * Настройка Postman
> * Отправка запросов с помощью Postman
> * Тестирование URL-адреса потоковой передачи.
> * Очистка ресурсов

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- [Создание учетной записи Служб мультимедиа](./account-create-how-to.md).

    Запишите значения, которые вы использовали в качестве имени группы ресурсов и имени учетной записи Служб мультимедиа.

- Установите клиент REST [Postman](https://www.getpostman.com/) для выполнения REST API, как показано в некоторых руководствах по REST AMS. 

    Мы используем **Postman**, но подойдет любое средство REST. Другие варианты включают: **Visual Studio Code** с подключаемым модулем REST или **Telerik Fiddler**. 

## <a name="download-postman-files"></a>Скачивание файлов Postman

Клонируйте репозиторий GitHub, который содержит файлы коллекции и среды Postman.

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-rest-postman.git
 ```

## <a name="access-api"></a>Доступ к API

Подробные сведения см. в статье [Получение учетных данных для доступа к API Служб мультимедиа](access-api-howto.md).

## <a name="configure-postman"></a>Настройка Postman

### <a name="configure-the-environment"></a>Настройка среды 

1. Откройте приложение **Postman**.
2. В правой части экрана выберите параметр **Управление средой**.

    ![Управление средой](./media/develop-with-postman/postman-import-env.png)
4. В диалоговом окне **Управление средой** нажмите кнопку **Импорт**.
2. Перейдите к файлу `Azure Media Service v3 Environment.postman_environment.json`, который вы скачали при клонировании `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`.
6. Будет добавлена среда **Azure Media Service v3 Environment**.

    > [!Note]
    > Задайте переменным доступа новые значения, полученные в разделе **Доступ к API Служб мультимедиа** выше.

7. Дважды щелкните выбранный файл и введите значения, которые вы получили, получив [доступ к API](#access-api).
8. Закройте диалоговое окно.
9. Выберите среду **Azure Media Service v3 Environment** в раскрывающемся списке.

    ![Выбор среды](./media/develop-with-postman/choose-env.png)
   
### <a name="configure-the-collection"></a>Настройка коллекции

1. Нажмите кнопку **импорта**, чтобы импортировать файл коллекции.
1. Перейдите к файлу `Media Services v3.postman_collection.json`, который вы скачали при клонировании `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`.
3. Выберите файл **Media Services v3.postman_collection.json**.

    ![Импорт файла](./media/develop-with-postman/postman-import-collection.png)

## <a name="send-requests-using-postman"></a>Отправка запросов с помощью Postman

В этом разделе описано, как отправить запросы, относящиеся к кодированию и созданию URL-адресов, которые нужны для потоковой передачи файла. В частности, отправляются следующие запросы:

1. получение маркера безопасности Azure AD для аутентификации субъекта-службы;
1. Запуск конечной точки потоковой передачи
2. Создание выходного ресурса
3. Создайте преобразование.
4. Создание задания
5. Создание указателя потоковой передачи
6. Получение списка путей для указателя потоковой передачи

> [!Note]
>  В этом руководстве предполагается, что всем создаваемым ресурсам вы присваиваете уникальные имена.  

### <a name="get-azure-ad-token"></a>Получение маркера Azure AD 

1. В левом окне приложения Postman выберите "Шаг 1. Получить маркер проверки подлинности AAD".
2. Затем выберите Get Azure AD Token for Service Principal Authentication (Получение маркера безопасности Azure AD для аутентификации субъекта-службы).
3. Нажмите кнопку **Отправить**.

    Это действие отправляет следующую операцию **POST**.

    ```
    https://login.microsoftonline.com/:aadTenantDomain/oauth2/token
    ```

4. Ответ на этот запрос возвращает маркер и присваивает его значение переменной среды AccessToken. Чтобы просмотреть код, изменяющий значение AccessToken, щелкните вкладку **Тесты**. 

    ![Получение маркера AAD](./media/develop-with-postman/postman-get-aad-auth-token.png)


### <a name="start-a-streaming-endpoint"></a>Запуск конечной точки потоковой передачи

Чтобы начать потоковую передачу видео, сначала вам потребуется запустить [конечную точку потоковой передачи](./streaming-endpoint-concept.md), из которой предполагается транслировать видео.

> [!NOTE]
> Плата взимается, только когда конечная точка потоковой передачи используется.

1. В левом окне приложения Postman выберите Streaming and Live (Потоковая передача и трансляция).
2. Затем выберите Start StreamingEndpoint (Запустить StreamingEndpoint).
3. Нажмите кнопку **Отправить**.

    * Это действие отправляет следующую операцию **POST**:

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaservices/:accountName/streamingEndpoints/:streamingEndpointName/start?api-version={{api-version}}
        ```
    * Если запрос выполнен успешно, возвращается `Status: 202 Accepted`.

        Это состояние означает, что запрос принят для обработки, но обработка не завершена. Вы можете запросить состояние операции на основе значения в заголовке ответа `Azure-AsyncOperation`.

        Например, следующая операция GET возвращает состояние операции:
        
        `https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/<resourceGroupName>/providers/Microsoft.Media/mediaservices/<accountName>/streamingendpointoperations/1be71957-4edc-4f3c-a29d-5c2777136a2e?api-version=2018-07-01`

        В статье [Track asynchronous Azure operations](../../azure-resource-manager/management/async-operations.md) (Отслеживание асинхронных операций Azure) приведены подробные сведения о том, как отслеживать состояние асинхронных операций Azure с помощью значений, возвращаемых в ответе.

### <a name="create-an-output-asset"></a>Создание выходного ресурса

Выходной [ресурс](/rest/api/media/assets) сохраняет результаты задания кодирования. 

1. В левом окне приложения Postman выберите "Активы".
2. Затем выберите действие Create or update an Asset (Создать или обновить ресурс).
3. Нажмите кнопку **Отправить**.

    * Это действие отправляет следующую операцию **PUT**:

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/assets/:assetName?api-version={{api-version}}
        ```
    * Эта операция включает следующий текст:

        ```json
        {
        "properties": {
            "description": "My Asset",
            "alternateId" : "some GUID",
            "storageAccountName": "<replace from environment file>",
            "container": "<supply any valid container name of your choosing>"
         }
        }
        ```

> [!NOTE]
> Обязательно замените учетную запись хранения и имена контейнеров соответствующими значениями из файла среды или укажите собственные значения.
>
> При выполнении действий, описанных в оставшейся части этой статьи, обязательно указывайте допустимые параметры в теле запроса.

### <a name="create-a-transform"></a>Создание преобразования

При кодировании или обработке содержимого в Службах мультимедиа параметры кодировки часто задают в качестве набора инструкций. Затем необходимо отправить **задание**, чтобы применить этот набор к видео. Отправляя новые задания для каждого нового видео, вы применяете этот набор ко всем видео в своей библиотеке. Набор инструкций в Службах мультимедиа называется **Преобразование**. Дополнительные сведения см. в статье [Преобразования и задания](./transforms-jobs-concept.md). Пример кода, описанный в руководстве, определяет набор инструкций, которые кодируют видео для его потоковой передачи на различные устройства под управлением iOS и Android. 

При создании нового экземпляра [преобразования](/rest/api/media/transforms) необходимо указать, что требуется создать в качестве выходных данных. В качестве обязательного параметра передается объект **TransformOutput**. Каждый объект **TransformOutput** содержит **предустановку** (Preset). **Предустановка** описывает пошаговые инструкции для операций обработки видео и звука, которые будут использоваться для создания нужного объекта **TransformOutput**. Пример, описанный в этой статье, использует встроенную предустановку, называемую **AdaptiveStreaming**. Предустановка кодирует входное видео в автоматически созданную схему скоростей и разрешений (пары "скорость — разрешение") на основе входного разрешения и скорости и создает файлы ISO MP4 с видео H.264 и аудио AAC, соответствующие каждой паре "скорость — разрешение". Дополнительные сведения об этой предустановке см. в [этой статье](encode-autogen-bitrate-ladder.md).

Вы можете использовать встроенную предустановку кодирования или пользовательскую предустановку. 

> [!Note]
> Перед созданием [преобразования](/rest/api/media/transforms) с помощью метода **Get** убедитесь, что такое преобразование еще не существует. В этом руководстве предполагается, что всем создаваемым преобразованиям вы присваиваете уникальные имена.

1. В левом окне приложения Postman выберите Encoding and Analysis (Кодирование и анализ).
2. Затем щелкните Create Transform (Создать преобразование).
3. Нажмите кнопку **Отправить**.

    * Это действие отправляет следующую операцию **PUT**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName?api-version={{api-version}}
        ```
    * Эта операция включает следующий текст:

        ```json
        {
            "properties": {
                "description": "Standard Transform using an Adaptive Streaming encoding preset from the library of built-in Standard Encoder presets",
                "outputs": [
                    {
                    "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                    "preset": {
                        "@odata.type": "#Microsoft.Media.BuiltInStandardEncoderPreset",
                        "presetName": "AdaptiveStreaming"
                    }
                    }
                ]
            }
        }
        ```

### <a name="create-a-job"></a>Создание задания

Объект [задание](/rest/api/media/jobs) содержит фактический запрос к Службам мультимедиа Azure, который применяет созданное **Преобразование** к указанному видео- и (или) аудиосодержимому. **Задание** указывает такую информацию, как расположение входного и выходного видео.

В этом примере входные данные задания основываются на URL-адресе HTTPS ("https:\//nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/").

1. В левом окне приложения Postman выберите Encoding and Analysis (Кодирование и анализ).
2. Затем выберите действие Create or update an Job (Создать или обновить задание).
3. Нажмите кнопку **Отправить**.

    * Это действие отправляет следующую операцию **PUT**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName/jobs/:jobName?api-version={{api-version}}
        ```
    * Эта операция включает следующий текст:

        ```json
        {
        "properties": {
            "input": {
            "@odata.type": "#Microsoft.Media.JobInputHttp",
            "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
            "files": [
                    "Ignite-short.mp4"
                ]
            },
            "outputs": [
            {
                "@odata.type": "#Microsoft.Media.JobOutputAsset",
                "assetName": "testAsset1"
            }
            ]
        }
        }
        ```

Выполнение задания занимает некоторое время. По его завершению вы будете уведомлены. Чтобы просмотреть ход выполнения задания, мы рекомендуем использовать Сетку событий. Она обеспечивает высокий уровень доступности, стабильную производительность и динамическое масштабирование. С помощью службы "Сетка событий Azure" приложения могут ожидать передачи данных и реагировать на события, поступающие буквально из всех служб Azure и пользовательских источников. Простая реактивная обработка событий на основе HTTP позволяет создавать эффективные решения с использованием интеллектуальной фильтрации и маршрутизации событий.  Дополнительные сведения см. в статье [Route Azure Media Services events to a custom web endpoint using CLI](monitoring/job-state-events-cli-how-to.md) (Маршрутизация событий Служб мультимедиа в пользовательскую конечную точку с помощью CLI).

Для **задания** обычно последовательно устанавливаются следующие состояния: **Запланировано**, **В очереди**, **Идет обработка**, **Завершено** (конечное состояние). Если в задании обнаружена ошибка, вы получите состояние **Ошибка**. Если задание находится в процессе отмены, вы получите состояние **Выполнение отмены** и **Отменено** по завершении.

#### <a name="job-error-codes"></a>Коды ошибок задания

См. статью о [кодах ошибок](/rest/api/media/jobs/get#joberrorcode).

### <a name="create-a-streaming-locator"></a>Создание указателя потоковой передачи

Выполнив задание кодирования необходимо сделать видео в выходном **ресурсе** доступным для воспроизведения клиентами. Это можно сделать в два этапа. Сначала создайте [StreamingLocator](/rest/api/media/streaminglocators), а затем URL-адреса потоковой передачи, которые клиенты могут использовать. 

Процесс создания указателя потоковой передачи называется публикацией. По умолчанию указатель потоковой передачи допустим сразу после выполнения вызова API и действует, пока не будет удален, если начальное и конечное время не настроены отдельно. 

При создании [StreamingLocator](/rest/api/media/streaminglocators) необходимо указать желаемое имя **StreamingPolicyName**. В этом примере выполняется потоковая передача незашифрованного содержимого, поэтому используется предварительно определенная политика прозрачной потоковой передачи Predefined_ClearStreamingOnly.

> [!IMPORTANT]
> При использовании пользовательской политики [StreamingPolicy](/rest/api/media/streamingpolicies) следует разработать ограниченный набор таких политик для учетной записи Служб мультимедиа и повторно использовать их для StreamingLocators всякий раз, когда требуются те же параметры шифрования и протоколы. 

У вашей учетной записи служб мультимедиа есть квота на количество входов в **политику потоковой передачи**. Вам не нужно создавать новую **политику потоковой передачи** для каждого указателя потоковой передачи.

1. В левом окне приложения Postman выберите Streaming Policies and Locators (Политики и указатели потоковой передачи).
2. Затем щелкните "Create a Streaming Locator (clear)" (Создать указатель потоковой передачи (чистый)).
3. Нажмите кнопку **Отправить**.

    * Это действие отправляет следующую операцию **PUT**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingLocators/:streamingLocatorName?api-version={{api-version}}
        ```
    * Эта операция включает следующий текст:

        ```json
        {
          "properties": {
            "streamingPolicyName": "Predefined_ClearStreamingOnly",
            "assetName": "testAsset1",
            "contentKeys": [],
            "filters": []
         }
        }
        ```

### <a name="list-paths-and-build-streaming-urls"></a>Получение списка путей и создание URL-адреса потоковой передачи

#### <a name="list-paths"></a>Вывод списка путей

Создав [указатель потоковой передачи](/rest/api/media/streaminglocators), вы можете получить URL-адреса потоковой передачи.

1. В левом окне приложения Postman выберите Streaming Policies (Политики потоковой передачи).
2. Затем выберите List Paths (Отобразить список путей).
3. Нажмите кнопку **Отправить**.

    * Это действие отправляет следующую операцию **POST**.

        ```
        https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/streamingLocators/:streamingLocatorName/listPaths?api-version={{api-version}}
        ```
        
    * Эта операция не содержит текст:
        
4. Запишите один из путей, который вы хотите применить для потоковой передачи. Он потребуется вам в следующем разделе. В нашем примере возвращаются следующие пути:
    
    ```
    "streamingPaths": [
        {
            "streamingProtocol": "Hls",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)"
            ]
        },
        {
            "streamingProtocol": "Dash",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=mpd-time-csf)"
            ]
        },
        {
            "streamingProtocol": "SmoothStreaming",
            "encryptionScheme": "NoEncryption",
            "paths": [
                "/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest"
            ]
        }
    ]
    ```

#### <a name="build-the-streaming-urls"></a>Создание URL-адресов потоковой передачи

В этом разделе описано, как создать URL-адрес потоковой передачи в формате HLS. URL-адрес состоит из следующих значений:

1. Протокол, используемый для отправки данных. В нашем примере это HTTPS.

    > [!NOTE]
    > Если проигрыватель размещен на сайте HTTPS, обновите URL-адрес до HTTPS.

2. Имя узла конечной точки потоковой передачи. В нашем случае это amsaccount-usw22.streaming.media.azure.net.

    Чтобы получить имя узла, можно использовать следующую операцию GET:
    
    ```
    https://management.azure.com/subscriptions/00000000-0000-0000-0000-0000000000000/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaservices/:accountName/streamingEndpoints/default?api-version={{api-version}}
    ```
    Убедитесь, что параметры `resourceGroupName` и `accountName` соответствуют файлу среды. 
    
3. Путь, который вы получили в разделе, посвященном перечислению путей.  

Из этих значений собирается следующий URL-адрес HLS.

```
https://amsaccount-usw22.streaming.media.azure.net/cdb80234-1d94-42a9-b056-0eefa78e5c63/Ignite-short.ism/manifest(format=m3u8-aapl)
```

## <a name="test-the-streaming-url"></a>Тестирование URL-адреса потоковой передачи.


> [!NOTE]
> Убедитесь, что **конечная точка потоковой передачи**, из которой нужно передавать содержимое, запущена.

Для тестирования потоковой передачи в этой статье используется Проигрыватель мультимедиа Azure. 

1. Откройте браузер и перейдите по ссылке [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. В поле **URL-адрес** вставьте созданный ранее URL-адрес. 
3. Щелкните **Update Player** (Обновить проигрыватель).

Проигрыватель мультимедиа Azure можно использовать для тестирования, но его нельзя применять в рабочей среде. 

## <a name="clean-up-resources-in-your-media-services-account"></a>Очистка ресурсов в учетной записи Служб мультимедиа

Как правило, необходимо очистить все, кроме объектов, которые вы планируете использовать повторно. Обычно повторно используются **преобразования**, **указатели потоковой передачи** и пр. Если учетную запись требуется очистить после эксперимента, следует удалить ресурсы, которые не планируется использовать повторно.  

Чтобы удалить ресурс, выберите операцию "Удалить" под тем ресурсом, который нужно удалить.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не нужны какие-либо ресурсы в группе ресурсов, включая Службы мультимедиа и учетные записи хранения, созданные для этого руководства, удалите группу ресурсов, созданную ранее.  

Выполните следующую команду CLI:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="ask-questions-give-feedback-get-updates"></a>Получение справки, отправка отзывов, получение обновлений

Прочитайте статью [сообщества Служб мультимедиа Azure](media-services-community.md), чтобы узнать, как задавать вопросы, оставлять отзывы и получать новости о Службах мультимедиа.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знаете, как отправлять, кодировать и выполнять потоковую передачу своего видео, перейдите к следующей статье: 

> [!div class="nextstepaction"]
> [Анализ видео](analyze-videos-tutorial.md)

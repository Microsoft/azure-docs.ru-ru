---
title: Как закодировать ресурс Azure с помощью Media Encoder Standard | Документация Майкрософт
description: Узнайте, как использовать Media Encoder Standard для кодирования мультимедийного содержимого в службах мультимедиа Azure. В примерах кода используется REST API.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 2a7273c6-8a22-4f82-9bfe-4509ff32d4a4
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: d4b7a188a9509c3510de2843b232f57e3437177b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103009833"
---
# <a name="how-to-encode-an-asset-by-using-media-encoder-standard"></a>Как закодировать ресурс с помощью Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-encode-with-media-encoder-standard.md)
> * [REST](media-services-rest-encode-asset.md)
> * [Портал](media-services-portal-encode.md)
>
>

## <a name="overview"></a>Обзор

Для поставки цифрового видео по Интернету необходимо сжатие мультимедиа. Размер цифровых видеофайлов достаточно велик и может быть слишком большим для доставки через Интернет или правильного отображения на устройствах клиентов. Кодирование — это процесс сжатия аудио- и видеофайлов, чтобы их могли просматривать клиенты.

Задания кодирования — одни из самых распространенных операций обработки в службах мультимедиа Azure. Они создаются для преобразования файлов мультимедиа из одного формата кодирования в другой. При кодировании можно использовать встроенный кодировщик служб мультимедиа (стандартный кодировщик мультимедиа). Можно также использовать кодировщик, предоставленный партнером служб мультимедиа. Кодировщики сторонних производителей доступны в магазине Azure. Информацию о задачах кодировки можно указать с помощью строк предустановок, заданных для кодировщика, или файлов конфигурации. Типы доступных предустановок см. в разделе [Task Presets for Media Encoder Standard](./media-services-mes-presets-overview.md) (Предустановки задач для Media Encoder Standard).

Каждое задание может состоять из одной или нескольких задач в зависимости от типа обработки, которую необходимо выполнить. REST API позволяет создавать задания и связанные с ними задачи одним из двух способов:

* задачи можно определить встроенными средствами с помощью свойства навигации Tasks в сущностях Job;
* можно использовать пакетную обработку OData.

Мы рекомендуем всегда кодировать исходные файлы в набор MP4-файлов с переменной скоростью, а затем преобразовывать его в нужный формат, используя [динамическую упаковку](media-services-dynamic-packaging-overview.md).

Если выходной ресурс зашифрован в хранилище, необходимо настроить политику доставки ресурсов. Дополнительные сведения см. в статье [Настройка политик доставки ресурсов-контейнеров](media-services-rest-configure-asset-delivery-policy.md).

## <a name="considerations"></a>Рекомендации

При доступе к сущностям в службах мультимедиа необходимо задать определенные поля и значения заголовков в HTTP-запросах. Дополнительную информацию см. в статье [Обзор интерфейса REST API служб мультимедиа](media-services-rest-how-to-use.md).

Прежде чем начать ссылаться на обработчики мультимедиа, убедитесь, что у вас есть правильный идентификатор обработчика мультимедиа. Дополнительные сведения см. в статье [Получение экземпляра процессора мультимедиа](media-services-rest-get-media-processor.md).

## <a name="connect-to-media-services"></a>Подключение к службам мультимедиа

Сведения о подключении к API AMS см. в разделе [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="create-a-job-with-a-single-encoding-task"></a>Создание задания с одной задачей кодирования

> [!NOTE]
> При работе с REST API служб мультимедиа следует руководствоваться следующими рекомендациями.
>
> При доступе к сущностям в службах мультимедиа необходимо задать определенные поля и значения заголовков в HTTP-запросах. Дополнительную информацию см. в статье [Обзор интерфейса REST API служб мультимедиа](media-services-rest-how-to-use.md).
>
> Если вы используете JSON и указали ключевое слово **__metadata** в запросе (например, для ссылки на связанный объект), вам НЕОБХОДИМО задать для заголовка **Accept**[подробный формат JSON](https://www.odata.org/documentation/odata-version-3-0/json-verbose-format/): Accept: application/json;odata=verbose.
>
>

В следующем примере показано, как создать и опубликовать задание с одной задачей, предназначенной для кодирования видео с определенным разрешением и качеством. При кодировании с помощью Media Encoder Standard вы можете использовать предустановки конфигурации задач, указанные [здесь](./media-services-mes-presets-overview.md).

Запрос:

```console
POST https://media.windows.net/API/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
    Host: media.windows.net

{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Aaab7f15b-3136-4ddf-9962-e9ecb28fb9d2')"}}],  "Tasks" : [{"Configuration" : "Adaptive Streaming", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",  "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"}]}
```

Ответ:

```console
HTTP/1.1 201 Created

. . .
```

### <a name="set-the-output-assets-name"></a>Задание имени выходного актива
В следующем примере показано, как установить атрибут assetName:

```console
{ "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"}`
```

### <a name="considerations"></a>Рекомендации
* Чтобы определить количество входных или выходных ресурсов, используемых задачей, в свойствах TaskBody необходимо использовать XML-литерал. В статье о задаче приведено определение схемы XML.
* В определении TaskBody каждое внутреннее значение `<inputAsset>` и `<outputAsset>` необходимо установить как JobInputAsset(value) или JobOutputAsset(value).
* В задаче может содержаться несколько выходных ресурсов. Значение JobOutputAsset(x) может использоваться только один раз в качестве выходных данных задачи в задании.
* В качестве входного ресурса задачи можно указать JobInputAsset или JobOutputAsset.
* Задачи не должны образовывать цикл.
* Значение параметра, которое передается в JobInputAsset или JobOutputAsset, — это значение индекса для ресурса. Фактические ресурсы определяются в свойствах навигации ресурса InputMediaAsset и OutputMediaAsset в определении сущности Job.
* Так как службы мультимедиа созданы на платформе OData версии 3, ссылки на отдельные ресурсы в коллекциях свойств навигации ресурсов InputMediaAsset и OutputMediaAsset добавляются в виде пары "имя — значение": "__metadata : uri".
* Ресурсы InputMediaAsset сопоставляются с одним или несколькими ресурсами, созданными в службах мультимедиа. Ресурсы OutputMediaAsset создаются системой. Они не ссылаются на существующий ресурс.
* Ресурсам OutputMediaAsset можно присвоить имя с помощью атрибута assetName. Если этот атрибут отсутствует, то именем ресурса OutputMediaAsset будет внутреннее текстовое значение элемента `<outputAsset>` с суффиксом, которым может быть значение параметра имени задания или параметра идентификатора задания (если свойство Name не определено). Например, если задать для атрибута assetName значение Sample, то для свойства Name ресурса OutputMediaAsset будет задано значение Sample. Тем не менее, если значение атрибута assetName не задано, но в качестве имени задания задано NewJob, имя OutputMediaAsset будет выглядеть следующим образом: JobOutputAsset(значение)_NewJob.

## <a name="create-a-job-with-chained-tasks"></a>Создание задания с цепными задачами
Во многих сценариях приложений разработчикам необходимо создать серию задач обработки. В службах мультимедиа вы можете создавать серии цепных задач. Каждая задача выполняет разные шаги обработки. Эти задачи также могут использовать разные обработчики мультимедиа. Цепные задачи могут передать ресурсы из одной задачи в другую, следуя линейной последовательности задач ресурса. Тем не менее, в последовательность не должны быть включены задачи, выполняемые в задании. При создании цепной задачи цепные объекты **ITask** создаются в одном объекте **IJob**.

> [!NOTE]
> Сейчас количество задач ограничено 30 задачами на задание. Если вам необходимо создать цепь из более чем 30 задач, создайте для них несколько заданий.
>
>

```console
POST https://media.windows.net/api/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{  
   "Name":"NewTestJob",
   "InputMediaAssets":[  
      {  
         "__metadata":{  
            "uri":"https://testrest.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A910ffdc1-2e25-4b17-8a42-61ffd4b8914c')"
         }
      }
   ],
   "Tasks":[  
      {  
         "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
      },
      {  
         "Configuration":"H264 Smooth Streaming 720p",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-16\"?><taskBody><inputAsset>JobOutputAsset(0)</inputAsset><outputAsset>JobOutputAsset(1)</outputAsset></taskBody>"
      }
   ]
}
```

### <a name="considerations"></a>Рекомендации
Чтобы включить создание цепных задач:

* в задании должно быть по крайней мере две задачи;
* должна существовать хотя бы одна задача, входные данные которой являются выходными данными другой задачи в задании.

## <a name="use-odata-batch-processing"></a>Использование пакетной обработки OData
В следующем примере показано, как использовать пакетную обработку OData для создания задания и задач. Чтобы узнать больше о пакетной обработке, ознакомьтесь с [пакетной обработкой посредством протокола OData](https://www.odata.org/documentation/odata-version-3-0/batch-processing/).

```console
POST https://media.windows.net/api/$batch HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Content-Type: multipart/mixed; boundary=batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
Accept: multipart/mixed
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
Host: media.windows.net


--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
Content-Type: multipart/mixed; boundary=changeset_122fb0a4-cd80-4958-820f-346309967e4d

--changeset_122fb0a4-cd80-4958-820f-346309967e4d
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://media.windows.net/api/Jobs HTTP/1.1
Content-ID: 1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{"Name" : "NewTestJob", "InputMediaAssets@odata.bind":["https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A2a22445d-1500-80c6-4b34-f1e5190d33c6')"]}

--changeset_122fb0a4-cd80-4958-820f-346309967e4d
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://media.windows.net/api/$1/Tasks HTTP/1.1
Content-ID: 2
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{  
   "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
   "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
   "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"Custom output name\">JobOutputAsset(0)</outputAsset></taskBody>"
}

--changeset_122fb0a4-cd80-4958-820f-346309967e4d--
--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e--
```


## <a name="create-a-job-by-using-a-jobtemplate"></a>Создание задания с помощью сущности JobTemplate
При обработке нескольких ресурсов с помощью обычного набора задач используйте сущности JobTemplate для определения предустановок задач по умолчанию или установления порядка задач.

В следующем примере показано, как создать сущность JobTemplate, определив сущность TaskTemplate внутренними средствами. Сущность TaskTemplate кодирует файл ресурса, используя в качестве обработчика мультимедиа Media Encoder Standard. Но также можно использовать и другие обработчики.

```console
POST https://media.windows.net/API/JobTemplates HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net


{"Name" : "NewJobTemplate25", "JobTemplateBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><jobTemplate><taskBody taskTemplateId=\"nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789\"><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody></jobTemplate>", "TaskTemplates" : [{"Id" : "nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789", "Configuration" : "H264 Smooth Streaming 720p", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56", "Name" : "SampleTaskTemplate2", "NumberofInputAssets" : 1, "NumberofOutputAssets" : 1}] }
```

> [!NOTE]
> В отличие от других сущностей служб мультимедиа вы должны определить новый идентификатор GUID для каждой сущности TaskTemplate и указать его в тексте запроса в taskTemplateId и свойстве Id. Схема идентификации содержимого должна соответствовать схеме, описанной в статье "Идентификация сущностей служб мультимедиа Azure". Кроме того, сущности JobTemplates нельзя обновлять. Вместо этого необходимо создавать новые сущности с необходимыми изменениями.
>
>

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 201 Created

. . .
```

В следующем примере показано, как создать задание, которое ссылается на идентификатор JobTemplate.

```console
POST https://media.windows.net/API/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net


{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A3f1fe4a2-68f5-4190-9557-cd45beccef92')"}}], "TemplateId" : "nb:jtid:UUID:15e6e5e6-ac85-084e-9dc2-db3645fbf0aa"}
```

При успешном выполнении возвращается следующий ответ:

```console
HTTP/1.1 201 Created

. . .
```


## <a name="advanced-encoding-features-to-explore"></a>Функции расширенного кодирования для ознакомления
* [Как создавать эскизы](media-services-dotnet-generate-thumbnail-with-mes.md)
* [Создание эскизов во время кодирования](media-services-dotnet-generate-thumbnail-with-mes.md#example-of-generating-a-thumbnail-while-encoding)
* [Обрезка видео во время кодирования](media-services-crop-video.md)
* [Настройка предустановок для кодирования](media-services-custom-mes-presets-with-dotnet.md)
* [Создание наложения или подложки для видео с использованием изображением](media-services-advanced-encoding-with-mes.md#overlay)

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы узнали, как создать задание для кодирования ресурса, ознакомьтесь со статьей [Практическое руководство. Проверка хода выполнения задания](media-services-rest-check-job-progress.md).

## <a name="see-also"></a>См. также раздел
[Получение обработчиков мультимедиа](media-services-rest-get-media-processor.md)
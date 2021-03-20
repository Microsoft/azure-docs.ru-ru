---
title: Создание проверок видео с помощью .NET в Content Moderator
titleSuffix: Azure Cognitive Services
description: 'В этой статье содержатся сведения и примеры кода, которые помогут вам быстро приступить к работе по созданию проверок видео с помощью пакета SDK Content Moderator для C#. Вы научитесь выполнять такие задачи:'
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: d9f80de5a18e27de4a9f8e85613e3c2eee6c111c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96853481"
---
# <a name="create-video-reviews-using-net"></a>Создание проверок видео с помощью .NET

В этой статье содержатся сведения и примеры кода, которые помогут вам быстро приступить к работе с [пакетом SDK Content Moderator для C#](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/). Вы научитесь выполнять такие задачи:

- создание проверки видео для модераторов-пользователей;
- добавление кадров в проверку;
- получение кадров для проверки;
- получение состояния и сведений о проверке;
- публикация проверки.

## <a name="prerequisites"></a>Предварительные условия

- Войдите или создайте учетную запись на сайте [средства проверки](https://contentmoderator.cognitive.microsoft.com/) Content Moderator.
- В этой статье предполагается, что вы уже [модерировали видео (ознакомьтесь с кратким руководством)](video-moderation-api.md) и у вас есть данные ответа. Они требуются для создания проверок по кадрам для модераторов-пользователей.

## <a name="ensure-your-api-key-can-call-the-review-api-for-review-creation"></a>Проверка, может ли ключ API вызвать API проверки для создания соответствующих заданий

Если вы начали с портала Azure, после выполнения предыдущих шагов у вас может получиться два ключа Content Moderator.

Если в своем примере пакета SDK вы планируете использовать ключ API, предоставленный платформой Azure, выполните [эти действия](review-tool-user-guide/configure.md#use-your-azure-account-with-the-review-apis), чтобы разрешить приложению вызывать API проверки и создавать соответствующие задания.

Если вы используете бесплатный пробный ключ, сгенерированный средством проверки, ваша учетная запись средства проверки уже знает об этом ключе, поэтому никакие дополнительные действия не требуются.

### <a name="prepare-your-video-and-the-video-frames-for-review"></a>Подготовка видео и кадров видео к проверке

Видео и примеры кадров видео должны быть опубликованы в Интернете, так как вам нужны их URL-адреса.

> [!NOTE]
> Чтобы проиллюстрировать использование API проверки, программа использует сохраненные вручную снимки экрана из видео со случайными оценками зрелости и неприличности. В реальной ситуации для создания изображений и получения оценок используются [выходные данные модерации видео](video-moderation-api.md#run-the-program-and-review-the-output). 

Для видео требуется конечная точка потоковой передачи, чтобы инструмент проверки мог воспроизводить видео в представлении проигрывателя.

![Эскиз демонстрационного видео](images/ams-video-demo-view.PNG)

- Скопируйте **URL-адрес**, опубликованный на странице [демонстрации для Служб мультимедиа Azure](https://aka.ms/azuremediaplayer?url=https%3A%2F%2Famssamples.streaming.mediaservices.windows.net%2F91492735-c523-432b-ba01-faba6c2206a2%2FAzureMediaServicesPromo.ism%2Fmanifest), и сохраните его в качестве URL-адреса манифеста.

В качестве кадров видео (изображений) используйте следующие изображения:

![Эскиз видеокадра 1](images/ams-video-frame-thumbnails-1.PNG) | ![Эскиз видеокадра 2](images/ams-video-frame-thumbnails-2.PNG) | ![Эскиз видеокадра 3](images/ams-video-frame-thumbnails-3.PNG) |
| :---: | :---: | :---: |
[Кадр 1](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG) | [Кадр 2](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG) | [Кадр 3](https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG) |

## <a name="create-your-visual-studio-project"></a>Создание проекта Visual Studio

1. Добавьте в свое решение новый проект **Консольное приложение (.NET Framework)**.

1. Назовите проект **VideoReviews**.

1. Выберите этот проект в качестве единственного запускаемого проекта для решения.

### <a name="install-required-packages"></a>Установка необходимых пакетов

Установите следующие пакеты NuGet для проекта TermLists.

- Microsoft.Azure.CognitiveServices.ContentModerator;
- Microsoft.Rest.ClientRuntime
- Microsoft.Rest.ClientRuntime.Azure
- Newtonsoft.Json.

### <a name="update-the-programs-using-statements"></a>Обновление инструкций using в программе

Измените инструкции using в программе следующим образом.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
using Microsoft.Azure.CognitiveServices.ContentModerator;
using Microsoft.Azure.CognitiveServices.ContentModerator.Models;
using Newtonsoft.Json;
```

### <a name="add-private-properties"></a>Добавление частных свойств

Добавьте следующие закрытые свойства в пространство имен **видеоревиевс**, класс **Program**. Обновите `AzureEndpoint` поля и, `CMSubscriptionKey` указав значения URL-адреса конечной точки и ключа подписки. Их можно найти на вкладке **Быстрый запуск** ресурса в портал Azure.


```csharp
namespace VideoReviews
{
    class Program
    {
        // NOTE: Enter a valid endpoint URL
        /// <summary>
        /// The endpoint URL of your subscription
        /// </summary>
        private static readonly string AzureEndpoint = "YOUR ENDPOINT URL";

        // NOTE: Enter a valid subscription key.
        /// <summary>
        /// Your Content Moderator subscription key.
        /// </summary>
        private static readonly string CMSubscriptionKey = "YOUR CONTENT MODERATOR KEY";

        // NOTE: Replace this example team name with your Content Moderator team name.
        /// <summary>
        /// The name of the team to assign the job to.
        /// </summary>
        /// <remarks>This must be the team name you used to create your 
        /// Content Moderator account. You can retrieve your team name from
        /// the Content Moderator web site. Your team name is the Id associated 
        /// with your subscription.</remarks>
        private const string TeamName = "YOUR CONTENT MODERATOR TEAM ID";

        /// <summary>
        /// The minimum amount of time, in milliseconds, to wait between calls
        /// to the Content Moderator APIs.
        /// </summary>
        private const int throttleRate = 2000;
```

### <a name="create-content-moderator-client-object"></a>Создание объекта клиента Content Moderator

Добавьте следующее определение метода в пространство имен **видеоревиевс**, класс **Program**.

```csharp
/// <summary>
/// Returns a new Content Moderator client for your subscription.
/// </summary>
/// <returns>The new client.</returns>
/// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
/// When you have finished using the client,
/// you should dispose of it either directly or indirectly. </remarks>
public static ContentModeratorClient NewClient()
{
    return new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey))
    {
        Endpoint = AzureEndpoint
    };
}
```

## <a name="create-a-video-review"></a>Создание проверки видео

Создайте проверку видео с помощью **ContentModeratorClient.Reviews.CreateVideoReviews**. Дополнительные сведения см. в [справочнике по API](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4).

**CreateVideoReviews** принимает следующие обязательные параметры.
1. Строка, обозначающая тип MIME, которая должна иметь значение "application/json". 
1. Имя команды Content Moderator.
1. Объект **IList\<CreateVideoReviewsBodyItem>**. Каждый объект **CreateVideoReviewsBodyItem** представляет отдельную проверку видео. В этом кратком руководстве проверки создаются по одной.

**CreateVideoReviewsBodyItem** имеет несколько свойств. Необходимо задать по меньшей мере следующие свойства.
- **Содержимое**. URL-адрес видео для проверки.
- Идентификатор **ContentId**. Идентификатор, который будет присвоен проверке видео.
- **Состояние**. Укажите здесь значение "Unpublished" (Неопубликованное). Если значение не задано, по умолчанию используется значение "Pending" (Ожидание), что означает, что проверка видео уже опубликована и ожидает пользовательской проверки. После публикации проверки видео вы не сможете добавить в нее видеокадры, расшифровку или результат модерации расшифровки.

> [!NOTE]
> **CreateVideoReviews** возвращает IList\<string>. Каждая из этих строк содержит идентификатор проверки видео. Эти идентификаторы являются глобально уникальными и их значения не совпадают со значениями свойства **ContentId**. 

Добавьте следующее определение метода в пространство имен VideoReviews в классе Program.

```csharp
/// <summary>
/// Create a video review. For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4 
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="id">The ID to assign to the video review.</param>
/// <param name="content">The URL of the video to review.</param>
/// <returns>The ID of the video review.</returns>
private static string CreateReview(ContentModeratorClient client, string id, string content)
{
    Console.WriteLine("Creating a video review.");

    List<CreateVideoReviewsBodyItem> body = new List<CreateVideoReviewsBodyItem>() {
        new CreateVideoReviewsBodyItem
        {
            Content = content,
            ContentId = id,
            /* Note: to create a published review, set the Status to "Pending".
            However, you cannot add video frames or a transcript to a published review. */
            Status = "Unpublished",
        }
    };

    var result = client.Reviews.CreateVideoReviews("application/json", TeamName, body);

    Thread.Sleep(throttleRate);

    // We created only one review.
    return result[0];
}
```

> [!NOTE]
> Для ключа службы Content Moderator установлен лимит количества запросов в секунду. При его превышении пакет SDK создаст исключение с кодом ошибки 429.
>
> Для ключа бесплатного уровня предусмотрен лимит в один запрос в секунду.

## <a name="add-video-frames-to-the-video-review"></a>Добавление кадров в проверку видео

Для добавления кадров в проверку видео используется метод **ContentModeratorClient.Reviews.AddVideoFrameUrl** (если видеокадры размещенный в Интернете) или метод **ContentModeratorClient.Reviews.AddVideoFrameStream** (если видеокадры размещены локально). В этом кратком руководстве предполагается, что видеокадры размещены в Интернете, поэтому в нем используется метод **AddVideoFrameUrl**. Дополнительные сведения см. в [справочнике по API](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd).

**AddVideoFrameUrl** принимает следующие обязательные параметры.
1. Строка, обозначающая тип MIME, которая должна иметь значение "application/json".
1. Имя команды Content Moderator.
1. Идентификатор проверки видео, полученный от **CreateVideoReviews**.
1. Объект **IList\<VideoFrameBodyItem>**. Каждый объект **VideoFrameBodyItem** представляет видеокадр.

**VideoFrameBodyItem** имеет следующие свойства.
- **Метка времени**. Строка время создания видеокадра в видео (в секундах).
- **FrameImage**. URL-адрес видеокадра.
- **Метаданные**. Объект IList\<VideoFrameBodyItemMetadataItem>. **VideoFrameBodyItemMetadataItem** представляет собой просто пару "ключ-значение". Допустимые ключи перечислены ниже.
- **reviewRecommended**. Имеет значение true, если рекомендуется пользовательская проверка видеокадра.
- **adultScore**. Имеет значение от 0 до 1, которое оценивает серьезность содержимого для взрослых в видеокадре.
- **а**. Имеет значение true, если в видео есть содержимое для взрослых.
- **racyScore**. Имеет значение от 0 до 1, которое оценивает серьезность непристойного содержимого в видеокадре.
- **r**. Имеет значение true, если в видеокадре есть непристойное содержимое.
- **ReviewerResultTags**. Объект IList\<VideoFrameBodyItemReviewerResultTagsItem>. **VideoFrameBodyItemReviewerResultTagsItem** представляет собой просто пару "ключ-значение". Приложение может использовать эти теги для упорядочивания видеокадров.

> [!NOTE]
> В этом кратком руководство создаются случайные значения свойств **adultScore** и **racyScore**. В рабочем приложении эти значения будут получены [службой модерации видео](video-moderation-api.md), развернутой в качестве службы мультимедиа Azure.

Добавьте следующие определения методов в пространство имен VideoReviews в классе Program.

```csharp
<summary>
/// Create a video frame to add to a video review after the video review is created.
/// </summary>
/// <param name="url">The URL of the video frame image.</param>
/// <returns>The video frame.</returns>
private static VideoFrameBodyItem CreateFrameToAddToReview(string url, string timestamp_seconds)
{
    // We generate random "adult" and "racy" scores for the video frame.
    Random rand = new Random();

    var frame = new VideoFrameBodyItem
    {
        // The timestamp is measured in milliseconds. Convert from seconds.
        Timestamp = (int.Parse(timestamp_seconds) * 1000).ToString(),
        FrameImage = url,

        Metadata = new List<VideoFrameBodyItemMetadataItem>
        {
            new VideoFrameBodyItemMetadataItem("reviewRecommended", "true"),
            new VideoFrameBodyItemMetadataItem("adultScore", rand.NextDouble().ToString()),
            new VideoFrameBodyItemMetadataItem("a", "false"),
            new VideoFrameBodyItemMetadataItem("racyScore", rand.NextDouble().ToString()),
            new VideoFrameBodyItemMetadataItem("r", "false")
        },

        ReviewerResultTags = new List<VideoFrameBodyItemReviewerResultTagsItem>()
        {
            new VideoFrameBodyItemReviewerResultTagsItem("tag1", "value1")
        }
    };

    return frame;
}
```

```csharp
/// <summary>
/// Add a video frame to the indicated video review. For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7b76ae7151f0b10d451fd
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
/// <param name="url">The URL of the video frame image.</param>
static void AddFrame(ContentModeratorClient client, string review_id, string url, string timestamp_seconds)
{
    Console.WriteLine("Adding a frame to the review with ID {0}.", review_id);

    var frames = new List<VideoFrameBodyItem>()
    {
        CreateFrameToAddToReview(url, timestamp_seconds)
    };
        
    client.Reviews.AddVideoFrameUrl("application/json", TeamName, review_id, frames);

    Thread.Sleep(throttleRate);
```

## <a name="get-video-frames-for-video-review"></a>Получение видеокадров для проверки видео

Видеокадры для проверки видео можно получить с помощью метода **ContentModeratorClient.Reviews.GetVideoFrames**. **GetVideoFrames** принимает следующие обязательные параметры.
1. Имя команды Content Moderator.
1. Идентификатор проверки видео, полученный от **CreateVideoReviews**.
1. Отсчитываемый от нуля индекс первого получаемого видеокадра.
1. Число получаемых видеокадров.

Добавьте следующее определение метода в пространство имен VideoReviews в классе Program.

```csharp
/// <summary>
/// Get the video frames assigned to the indicated video review.  For more information, see the API reference:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7ba43e7151f0b10d45200
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
static void GetFrames(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Getting frames for the review with ID {0}.", review_id);

    Frames result = client.Reviews.GetVideoFrames(TeamName, review_id, 0);
    Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

    Thread.Sleep(throttleRate);
}
```

## <a name="get-video-review-information"></a>Получение сведений о проверке видео

Для получения сведений о проверке видео используется метод **ContentModeratorClient.Reviews.GetReview**. **GetReview** принимает следующие обязательные параметры.
1. Имя команды Content Moderator.
1. Идентификатор проверки видео, полученный от **CreateVideoReviews**.

Добавьте следующее определение метода в пространство имен VideoReviews в классе Program.

```csharp
/// <summary>
/// Get the information for the indicated video review. For more information, see the reference API:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
private static void GetReview(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Getting the status for the review with ID {0}.", review_id);

    var result = client.Reviews.GetReview(ModeratorHelper.Clients.TeamName, review_id);
    Console.WriteLine(JsonConvert.SerializeObject(result, Formatting.Indented));

    Thread.Sleep(throttleRate);
}
```

## <a name="publish-video-review"></a>Публикация проверки видео

Для публикации проверки видео используется метод **ContentModeratorClient.Reviews.CreateVideoReviews**. **PublishVideoReview** принимает следующие обязательные параметры.
1. Имя команды Content Moderator.
1. Идентификатор проверки видео, полученный от **CreateVideoReviews**.

Добавьте следующее определение метода в пространство имен VideoReviews в классе Program.

```csharp
/// <summary>
/// Publish the indicated video review. For more information, see the reference API:
/// https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/59e7bb29e7151f0b10d45201
/// </summary>
/// <param name="client">The Content Moderator client.</param>
/// <param name="review_id">The video review ID.</param>
private static void PublishReview(ContentModeratorClient client, string review_id)
{
    Console.WriteLine("Publishing the review with ID {0}.", review_id);
    client.Reviews.PublishVideoReview(TeamName, review_id);
    Thread.Sleep(throttleRate);
}
```

## <a name="putting-it-all-together"></a>Сборка

Добавьте определение метода **Main** в пространство имен VideoReviews в классе Program. Наконец, закройте класс Program и пространство имен VideoReviews.

```csharp
static void Main(string[] args)
{
    using (ContentModeratorClient client = NewClient())
    {
        // Create a review with the content pointing to a streaming endpoint (manifest)
        var streamingcontent = "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest";
        string review_id = CreateReview(client, "review1", streamingcontent);

        var frame1_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame1-00-17.PNG";
        var frame2_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-2-01-04.PNG";
        var frame3_url = "https://blobthebuilder.blob.core.windows.net/sampleframes/ams-video-frame-3-02-24.PNG";

        // Add the frames from 17, 64, and 144 seconds.
        AddFrame(client, review_id, frame1_url, "17");
        AddFrame(client, review_id, frame2_url, "64");
        AddFrame(client, review_id, frame3_url, "144");

        // Get frames information and show
        GetFrames(client, review_id);
        GetReview(client, review_id);

        // Publish the review
        PublishReview(client, review_id);

        Console.WriteLine("Open your Content Moderator Dashboard and select Review > Video to see the review.");
        Console.WriteLine("Press any key to close the application.");
        Console.ReadKey();
    }
}
```

## <a name="run-the-program-and-review-the-output"></a>Запуск программы и просмотр выходных данных
Запустив созданное приложение, вы увидите следующие строки выходных данных.

```json
Creating a video review.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Adding a frame to the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Getting frames for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
{
    "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
    "VideoFrames": [
    {
        "Timestamp": "17000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_17000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.808312381528463"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.846378884206702"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
    ]
    },
    {
        "Timestamp": "64000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_64000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.576078300166912"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.244768953064815"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
    ]
    },
    {
        "Timestamp": "144000",
        "FrameImage": "https://reviewcontentprod.blob.core.windows.net/testreview6/FRM_201801v3212bda70ced4928b2cd7459c290c7dc_144000.PNG",
        "Metadata": [
        {
            "Key": "reviewRecommended",
            "Value": "true"
        },
        {
            "Key": "adultScore",
            "Value": "0.664480847150311"
        },
        {
            "Key": "a",
            "Value": "false"
        },
        {
            "Key": "racyScore",
            "Value": "0.933817870418456"
        },
        {
            "Key": "r",
            "Value": "false"
        }
        ],
        "ReviewerResultTags": [
        {
            "Key": "tag1",
            "Value": "value1"
        }
        ]
    }
    ]
}

Getting the status for the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
{
    "ReviewId": "201801v3212bda70ced4928b2cd7459c290c7dc",
    "SubTeam": "public",
    "Status": "UnPublished",
    "ReviewerResultTags": [],
    "CreatedBy": "testreview6",
    "Metadata": [
    {
        "Key": "FrameCount",
        "Value": "3"
    }
    ],
    "Type": "Video",
    "Content": "https://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
    "ContentId": "review1",
    "CallbackEndpoint": null
}

Publishing the review with ID 201801v3212bda70ced4928b2cd7459c290c7dc.
Open your Content Moderator Dashboard and select Review > Video to see the review.
Press any key to close the application.
```

## <a name="check-out-your-video-review"></a>Просмотр проверки видео

Наконец, вы увидите проверку видео в учетной записи средства проверки Content Moderator на экране **просмотра** > **видео** .

![Проверка видео модераторами-пользователями](images/ams-video-review.PNG)

## <a name="next-steps"></a>Дальнейшие действия

Получите [пакет SDK Content Moderator для .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) и [решение для Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator). Они вам понадобятся для работы с этим и другими руководствами по Content Moderator для .NET.

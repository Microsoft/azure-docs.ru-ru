---
title: Создание проверок на основе REST API консоли Content Moderator
titleSuffix: Azure Cognitive Services
description: Используйте API-интерфейсы проверки Content Moderator Azure, чтобы создавать изображения или текстовые рецензии для человеческого контроля.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: pafarley
ms.openlocfilehash: 479c7c455f07d098edd327196803e85df24dfb6d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96905156"
---
# <a name="create-human-reviews-api-console"></a>Создание пользовательских рецензий (консоль API)

[Проверяет](./review-api.md#reviews) хранение и отображение содержимого для оценки человеческим модератором. Когда пользователь завершает проверку, результаты отправляются в указанную конечную точку обратного вызова. В этом руководство вы узнаете, как настроить рецензии с помощью API-интерфейсов API-интерфейса RESTFUL через консоль. Когда вы понимаете структуру интерфейсов API, вы можете легко перенести эти вызовы на любую платформу, совместимую с остальными частями.

## <a name="prerequisites"></a>Предварительные условия

- Войдите или создайте учетную запись на сайте [средства проверки](https://contentmoderator.cognitive.microsoft.com/) Content Moderator.

## <a name="create-a-review"></a>Создание проверки

Чтобы создать рецензию, перейдите на страницу " **[Проверка-создание](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4)** справочника по API" и нажмите кнопку для своего региона. (это можно найти в URL-адресе конечной точки на странице "Проверка **учетных данных** " [средства проверки](https://contentmoderator.cognitive.microsoft.com/)). Запустится консоль API, где можно легко создавать и выполнять вызовы REST API.

![Проверка — получение выбора региона](images/test-drive-region.png)

### <a name="enter-rest-call-parameters"></a>Введите параметры вызова RESTFUL

Введите значения для **теамнаме** и **OCP-Apim-Subscription-Key**:

- **теамнаме**: идентификатор команды, созданный при настройке учетной записи [средства проверки](https://contentmoderator.cognitive.microsoft.com/) (находится в поле **идентификатор** на экране учетных данных средства проверки).
- **OCP-Apim-Subscription-Key**— ключ Content Moderator. Его можно найти на вкладке **Параметры** [средства проверки](https://contentmoderator.cognitive.microsoft.com).

### <a name="enter-a-review-definition"></a>Введите определение проверки

Измените поле **текст запроса** , чтобы ввести запрос JSON со следующими полями:

- **Метаданные**: пользовательские пары "ключ-значение", возвращаемые в конечную точку обратного вызова. Если ключ является коротким кодом, определенным в [средстве проверки](https://contentmoderator.cognitive.microsoft.com), он отображается в виде тега.
- **Содержимое**: в случае изображения и видео это строка URL-адреса, указывающая на содержимое. Для текстового содержимого это собственно текстовая строка.
- Идентификатор **ContentId**: строка настраиваемого идентификатора. эта строка передается в API и возвращается с помощью обратного вызова. Это полезно для связывания внутренних идентификаторов или метаданных с результатами задания по сопоставлению.
- **Каллбаккендпоинт**: (необязательный) URL-адрес для получения сведений о обратном вызове по завершении проверки.

В тексте запроса по умолчанию показаны примеры различных типов рецензий, которые можно создать.

```json
[Image]
[
  {
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      }
    ],
    "Type": "Image",
    "Content": "<Content Url>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>"
  }
]
[Text]
[
  {
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      }
    ],
    "Type": "Text",
    "Content": "<Your Text Content>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>"
  }
]
[Video]
[
  {
    "VideoFrames":[
      {
          "Id": "<Frame Id>",
          "Timestamp": "<Frame Timestamp",
          "FrameImage":"<Frame Image URL",
          "Metadata": [
            {
              "Key": "<Key>",
              "Value": "<Value"
            }
          ],
          "ReviewerResultTags": [
          ]
    ], 
    "Metadata": [
      {
        "Key": "string",
        "Value": "string"
      },
      //For encrypted Videos
        {
          "Key": "protectedType",
          "Value": "AES or FairPlay or Widevine or Playready"
        },
        {
          "Key": "authenticationToken",
          "Value": "your viewtoken(In case of Video Indexer AES encryption type, this value is viewtoken from breakdown json)"
        },
      //For FairPlay encrypted type video include certificateUrl as well
        {
          "Key": "certificateUrl",
          "Value": "your certificate url"
        }
    ],
    "Type": "Video",
    "Content": "<Stream Url>",
    "ContentId": "<Your identifier for this content>",
    "CallbackEndpoint": "<Url where you would receive callbacks>",
    [Optional]
    "Timescale": "<Timescale of the video>
  }
]
```

### <a name="submit-your-request"></a>Отправка запроса
  
Нажмите кнопку **Отправить**. Если операция выполнена, **состояние ответа** равно `200 OK` , а в поле **ответное содержимое** отображается идентификатор проверки. Скопируйте этот идентификатор, чтобы использовать на следующих шагах.

![Идентификатор проверки в поле "Response content" (Содержимое ответа) в консоли "Review - Create" (Проверка — создание)](images/test-drive-review-2.PNG)

### <a name="examine-the-new-review"></a>Проверка новой проверки

В [средстве проверки](https://contentmoderator.cognitive.microsoft.com)выберите **Проверка**  >   / **текста** изображения / **видео** (в зависимости от используемого содержимого). Содержимое, которое вы перегрузили, должно отобразиться для человеческого анализа.

![Изображение футбольного мяча в инструменте проверки](images/test-drive-review-5.PNG)

## <a name="get-review-details"></a>Получение сведений о проверке

Чтобы получить сведения о существующей проверке, перейдите на страницу [проверки и получение](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c2) справочника по API и нажмите кнопку для своего региона (регион, в котором находится ваш ключ).

![Выбор региона на странице "Workflow - Get" (Рабочий процесс — получение)](images/test-drive-region.png)

Введите параметры вызова RESTFUL, как показано в приведенном выше разделе. Для этого шага **ревиевид** — это уникальная строка идентификатора, полученная при создании проверки.

![Получение результатов в консоли "Review - Create" (Проверка — создание)](images/test-drive-review-3.PNG)
  
Нажмите кнопку **Отправить**. Если операция выполнена, **состояние отклика** — `200 OK` , а в поле **ответное содержимое** отображаются подробные сведения о проверке в формате JSON, как показано ниже:

```json
{  
  "reviewId":"201712i46950138c61a4740b118a43cac33f434",
  "subTeam":"public",
  "status":"Complete",
  "reviewerResultTags":[  
    {  
      "key":"a",
      "value":"False"
    },
    {  
      "key":"r",
      "value":"True"
    },
    {  
      "key":"sc",
      "value":"True"
    }
  ],
  "createdBy":"<teamname>",
  "metadata":[  
    {  
      "key":"sc",
      "value":"true"
    }
  ],
  "type":"Image",
  "content":"https://reviewcontentprod.blob.core.windows.net/<teamname>/IMG_201712i46950138c61a4740b118a43cac33f434",
  "contentId":"0",
  "callbackEndpoint":"<callbackUrl>"
}
```

Обратите внимание на следующие поля в ответе:

- **status**
- **ревиеверресулттагс**. отображается, если какие-либо теги были добавлены вручную командой экспертной проверки (отображается поле **createdBy** ).
- **метаданные**. здесь показаны теги, которые изначально были добавлены в проверку перед тем, как группа разработчиков по анализу внес изменения.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как создавать проверки на основе содержимого с помощью REST API. Затем интегрируйте обзоры в сценарий комплексного контроля, например в учебнике по внедрению [электронной коммерции](./ecommerce-retail-catalog-moderation.md) .
---
title: Краткое руководство по REST API Распознавания лиц
description: Использование REST API Распознавания лиц с cURL позволяет обнаруживать лица, находить похожие лица (поиск лиц по изображению), идентифицировать лица (поиск с распознаванием лиц) и переносить данные о лицах.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 12/06/2020
ms.author: pafarley
ms.openlocfilehash: 1a360a19da8006a8ab6d918560f1b1d566fdd66d
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105104265"
---
Из этого руководства вы узнаете, как начать работу с REST API Распознавания лиц. В службе "Распознавание лиц" доступны передовые алгоритмы обнаружения и распознавания лиц на изображениях.

Использование REST API Распознавания лиц позволяет выполнять следующие задачи:

* [Определение лиц на изображении](#detect-faces-in-an-image)
* [поиск похожих лиц](#find-similar-faces);

> [!NOTE]
> В рамках этого краткого руководства для вызова REST API используются команды cURL. Вы также можете вызывать REST API с помощью языка программирования. Примеры см. в репозиториях GitHub для [C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/Face/rest), [Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/Face/rest), [Java](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/Face/rest), [JavaScript](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/Face/rest) и [Go](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/go/Face/rest).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, перейдите к <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Создание ресурса Распознавания лиц"  target="_blank">созданию ресурса Распознавания лиц</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Распознавания лиц потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* [PowerShell версии 6.0 и выше](/powershell/scripting/install/installing-powershell-core-on-windows) или аналогичное приложение командной строки.


## <a name="detect-faces-in-an-image"></a>Определение лиц на изображении

Для вызова API Распознавания лиц и получения данных об атрибутах лиц из изображения будет использоваться приведенная ниже команда. Сначала скопируйте код в текстовый редактор. Вам потребуется внести изменения в определенные части команды перед ее запуском.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detection_model_2":::

Внесите следующие изменения:
1. Назначьте `Ocp-Apim-Subscription-Key` действительному ключу подписки на службу "Распознавание лиц".
1. Измените первую часть URL-адреса запроса, указав конечную точку, соответствующую ключу подписки.
   [!INCLUDE [subdomains-note](../../../../../includes/cognitive-services-custom-subdomains-note.md)]
1. При необходимости измените URL-адрес в тексте запроса, указав ссылку на другое изображение.

После внесения изменений откройте командную строку и введите новую команду. 

### <a name="examine-the-results"></a>Просмотр результатов

В окне консоли должна появиться информация о лицах в виде данных JSON. Пример:

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    }
  }
]  
```

### <a name="get-face-attributes"></a>Получение атрибутов лиц
 
Чтобы извлечь атрибуты лиц, снова вызовите API обнаружения, но задайте для `detectionModel` значение `detection_01` и добавьте параметр запроса `returnFaceAttributes`. Команда должна выглядеть, как представленный ниже пример. Как и ранее, добавьте реальные значения ключа подписки и конечной точки Распознавания лиц.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detection_model_1":::

### <a name="examine-the-results"></a>Просмотр результатов

Теперь возвращаемая информация о лицах содержит атрибуты. Пример:

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    },
    "faceAttributes": {
      "smile": 0,
      "headPose": {
        "pitch": 0,
        "roll": 0.1,
        "yaw": -32.9
      },
      "gender": "female",
      "age": 22.9,
      "facialHair": {
        "moustache": 0,
        "beard": 0,
        "sideburns": 0
      },
      "glasses": "NoGlasses",
      "emotion": {
        "anger": 0,
        "contempt": 0,
        "disgust": 0,
        "fear": 0,
        "happiness": 0,
        "neutral": 0.986,
        "sadness": 0.009,
        "surprise": 0.005
      },
      "blur": {
        "blurLevel": "low",
        "value": 0.06
      },
      "exposure": {
        "exposureLevel": "goodExposure",
        "value": 0.67
      },
      "noise": {
        "noiseLevel": "low",
        "value": 0
      },
      "makeup": {
        "eyeMakeup": true,
        "lipMakeup": true
      },
      "accessories": [],
      "occlusion": {
        "foreheadOccluded": false,
        "eyeOccluded": false,
        "mouthOccluded": false
      },
      "hair": {
        "bald": 0,
        "invisible": false,
        "hairColor": [
          {
            "color": "brown",
            "confidence": 1
          },
          {
            "color": "black",
            "confidence": 0.87
          },
          {
            "color": "other",
            "confidence": 0.51
          },
          {
            "color": "blond",
            "confidence": 0.08
          },
          {
            "color": "red",
            "confidence": 0.08
          },
          {
            "color": "gray",
            "confidence": 0.02
          }
        ]
      }
    }
  }
]
```

## <a name="find-similar-faces"></a>Поиск похожих лиц

В ходе этой операции принимается одно обнаруженное лицо (лицо-источник) и выполняется поиск совпадений (поиск лиц по изображению) в коллекции других лиц (цель). При обнаружении совпадения в консоль выводится идентификатор лица, с которым найдено совпадение.

### <a name="detect-faces-for-comparison"></a>Обнаружение лиц для сравнения

Чтобы сравнить лица, их сначала нужно обнаружить на изображениях. Выполните эту команду, как описано в разделе [Определение лиц на изображении](#detect-faces-in-an-image). Этот метод обнаружения оптимизирован для операций сравнения. Он не извлекает подробные атрибуты лица, как в предыдущем разделе, и использует другую модель определения.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detect_for_similar":::

Найдите в ответе JSON значение `"faceId"` и сохраните его во временном расположении. Затем снова выполните приведенную выше команду для URL-адресов других изображений и сохраните соответствующие идентификаторы лиц. Эти идентификаторы будут использоваться в качестве целевой группы, в которой будет выполняться поиск похожих лиц.

:::code source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_group":::

Затем определите одно лицо-источник, которое будет использоваться для сопоставления, и сохраните его идентификатор. Сохраните этот идентификатор отдельно от других.

:::code source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_matcher":::

### <a name="find-matches"></a>Поиск совпадений

Скопируйте приведенную ниже команду в текстовый редактор.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar":::

Внесите в нее следующие изменения:
1. Назначьте `Ocp-Apim-Subscription-Key` действительному ключу подписки на службу "Распознавание лиц".
1. Измените первую часть URL-адреса запроса, указав конечную точку, соответствующую ключу подписки.

В качестве значения `body` используйте следующий код JSON:

:::code language="JSON" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_body":::

1. Для параметра `"faceId"` укажите идентификатор лица-источника.
1. В массив `"faceIds"` вставьте другие идентификаторы лиц (в качестве условий поиска).

### <a name="examine-the-results"></a>Просмотр результатов

Вы получите ответ JSON, в котором будут перечислены идентификаторы лиц, соответствующих вашему запросу.

```json
[
    {
        "persistedFaceId" : "015839fb-fbd9-4f79-ace9-7675fc2f1dd9",
        "confidence" : 0.82
    },
    ...
] 
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как с помощью REST API Распознавания лиц выполнять базовые задачи по распознаванию лиц. Далее ознакомьтесь со справочной документацией, чтобы узнать больше о библиотеке.

> [!div class="nextstepaction"]
> [Справочник по API Распознавания лиц](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)

* [Что представляет собой служба "Распознавание лиц"?](../../overview.md)
---
title: Краткое руководство. Оптическое распознавание символов REST API
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве описано, как приступить к работе с функцией оптического распознавания символов REST API.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: quickstart
ms.date: 12/02/2020
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 01149eed5cc4195ca501507e7fe1d66fffecb84d
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107327654"
---
Используйте возможности оптического распознавания символов REST API для чтения печатных и рукописных текстов.

> [!NOTE]
> В рамках этого краткого руководства для вызова REST API используются команды cURL. Вы также можете вызывать REST API с помощью языка программирования. Примеры см. в репозиториях GitHub для [C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/ComputerVision/REST), [Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/ComputerVision/REST), [Java](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/ComputerVision/REST), [JavaScript](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/ComputerVision/REST) и [Go](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/go/ComputerVision/REST).

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/). 
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="создайте ресурс Компьютерного зрения"  target="_blank">create a Computer Vision resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
  * Для подключения приложения к API Компьютерного зрения потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
  * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* Установленная программа [cURL](https://curl.haxx.se/).



## <a name="read-printed-and-handwritten-text"></a>Чтение печатного и рукописного текста

Служба OCR может считывать видимый текст в образе и преобразовывать его в поток символов. Дополнительные сведения о распознавании текста см. в обзорной статье [Оптическое распознавание текста (OCR)](../overview-ocr.md).

Чтобы создать и запустить пример, сделайте следующее.

1. Скопируйте приведенную ниже команду в текстовый редактор.
1. При необходимости внесите следующие изменения в команду.
    1. Замените значение `<subscriptionKey>` своим ключом подписки.
    1. Замените первую часть URL-адреса запроса (`westcentralus`) текстом из URL-адреса своей конечной точки.
        [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]
    1. При необходимости замените URL-адрес изображения в тексте запроса (`https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/Atomist_quote_from_Democritus.png/338px-Atomist_quote_from_Democritus.png\`) URL-адресом другого изображения для анализа.
1. Откройте окно командной строки.
1. Вставьте команду из текстового редактора в окно командной строки и выполните команду.

```bash
curl -v -X POST "https://westcentralus.api.cognitive.microsoft.com/vision/v3.2/read/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <subscription key>" --data-ascii "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/Atomist_quote_from_Democritus.png/338px-Atomist_quote_from_Democritus.png\"}"
```

Ответ будет включать заголовок `Operation-Location`, значение которого является уникальным URL-адресом. Этот URL-адрес будет использоваться для запрашивания результатов операции Read. Срок действия URL-адреса истекает через 48 часов.

1. Скопируйте приведенную ниже команду в свой текстовый редактор.
1. Замените URL-адрес значением `Operation-Location`, которое вы скопировали на предыдущем шаге.
1. При необходимости внесите следующие изменения в команду.
    1. Замените значение `<subscriptionKey>` своим ключом подписки.
1. Откройте окно командной строки.
1. Вставьте команду из текстового редактора в окно командной строки и выполните команду.

```bash
curl -v -X GET "https://westcentralus.api.cognitive.microsoft.com/vision/v3.2/read/analyzeResults/{operationId}" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{body}" 
```

### <a name="examine-the-response"></a>Изучите ответ.

Успешный ответ будет возвращен в формате JSON. После этого запустится синтаксический анализ примера приложения и в окне командной строки отобразится успешный ответ, аналогичный следующему.

```json
{
  "status": "succeeded",
  "createdDateTime": "2021-04-08T21:56:17.6819115+00:00",
  "lastUpdatedDateTime": "2021-04-08T21:56:18.4161316+00:00",
  "analyzeResult": {
    "version": "3.2",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 338,
        "height": 479,
        "unit": "pixel",
        "lines": [
          {
            "boundingBox": [
              25,
              14,
              318,
              14,
              318,
              59,
              25,
              59
            ],
            "text": "NOTHING",
            "appearance": {
              "style": {
                "name": "other",
                "confidence": 0.971
              }
            },
            "words": [
              {
                "boundingBox": [
                  27,
                  15,
                  294,
                  15,
                  294,
                  60,
                  27,
                  60
                ],
                "text": "NOTHING",
                "confidence": 0.994
              }
            ]
          }
        ]
      }
    ]
  }
}

```



## <a name="next-steps"></a>Дальнейшие действия

Изучите OCR API подробнее. Для быстрых экспериментов с API можно использовать [открытую консоль тестирования API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/5d986960601faab4bf452005/console).

> [!div class="nextstepaction"]
> [Подробнее об OCR API](https://centraluseuap.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/5d986960601faab4bf452005)

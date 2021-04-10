---
title: Краткое руководство. REST API Content Moderator
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве показано, как начать работу с REST API Azure Content Moderator. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 12/08/2020
ms.author: pafarley
ms.openlocfilehash: 747f97e54dbe10ca055281c189c805e3dd97c867
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105582815"
---
Сведения о начале работы с REST API Azure Content Moderator. 

Content Moderator — это служба ИИ, позволяющая управлять содержимым, которое может носить оскорбительный характер или представлять опасность, либо нежелательным содержимым иного рода. Используйте службу модерации содержимого на основе ИИ, чтобы проверить текст, изображения и видео, а также автоматически применить флаги содержимого. Затем интегрируйте свое приложение со средством проверки — средой модерации для команды рецензентов для работы в подключенном режиме. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.

Используйте REST API Content Moderator для выполнения таких задач:

* Модерация текста
* Модерация изображений

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="Создание ресурса Content Moderator"  target="_blank">создайте ресурс Content Moderator </a> на портале Azure, чтобы получить ключ и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Для подключения приложения к Content Moderator потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* [PowerShell версии 6.0 и выше](/powershell/scripting/install/installing-powershell-core-on-windows) или аналогичное приложение командной строки.


## <a name="moderate-text"></a>Модерация текста

Вы будете использовать команды, аналогичные приведенным ниже, чтобы вызывать API Content Moderator для анализа текста и вывода результатов в консоль.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/content-moderator/quickstart.sh" ID="textmod":::

Скопируйте команду в текстовый редактор и внесите следующие изменения:

1. Назначьте `Ocp-Apim-Subscription-Key` действительному ключу подписки на службу "Распознавание лиц".
1. Измените первую часть URL-адреса запроса, указав конечную точку, соответствующую ключу подписки.
   [!INCLUDE [subdomains-note](../../../../../includes/cognitive-services-custom-subdomains-note.md)]
1. При необходимости измените текст запроса на любую строку текста, которую нужно проанализировать.

После внесения изменений откройте командную строку и введите новую команду. 

### <a name="examine-the-results"></a>Просмотр результатов

В окне консоли отобразятся результаты модерации текста в формате JSON. Пример:

```json
{
  "OriginalText": "Is this a crap email abcdef@abcd.com, phone: 6657789887, IP: 255.255.255.255,\n1 Microsoft Way, Redmond, WA 98052\n",
  "NormalizedText": "Is this a crap email abide@ abed. com, phone: 6657789887, IP: 255. 255. 255. 255, \n1 Microsoft Way, Redmond, WA 98052",
  "AutoCorrectedText": "Is this a crap email abide@ abed. com, phone: 6657789887, IP: 255. 255. 255. 255, \n1 Microsoft Way, Redmond, WA 98052",
  "Misrepresentation": null,
  "PII": {
    "Email": [
      {
        "Detected": "abcdef@abcd.com",
        "SubType": "Regular",
        "Text": "abcdef@abcd.com",
        "Index": 21
      }
    ],
    "IPA": [
      {
        "SubType": "IPV4",
        "Text": "255.255.255.255",
        "Index": 61
      }
    ],
    "Phone": [
      {
        "CountryCode": "US",
        "Text": "6657789887",
        "Index": 45
      }
    ],
    "Address": [
      {
        "Text": "1 Microsoft Way, Redmond, WA 98052",
        "Index": 78
      }
    ]
  },
 "Classification": {
    "Category1": 
    {
      "Score": 0.5
    },
    "Category2": 
    {
      "Score": 0.6
    },
    "Category3": 
    {
      "Score": 0.5
    },
    "ReviewRecommended": true
  },
  "Language": "eng",
  "Terms": [
    {
      "Index": 10,
      "OriginalIndex": 10,
      "ListId": 0,
      "Term": "crap"
    }
  ],
  "Status": {
    "Code": 3000,
    "Description": "OK",
    "Exception": null
  },
  "TrackingId": "1717c837-cfb5-4fc0-9adc-24859bfd7fac"
}
```

Дополнительные сведения об атрибутах текста, доступных в Content Moderator, см. в разделе [Основные принципы модерации текста](../../text-moderation-api.md).

## <a name="moderate-images"></a>Модерация изображений

Вы будете использовать команды, аналогичные приведенным ниже, чтобы вызывать API Content Moderator для модерации изображения из удаленного расположения и вывода результатов в консоль.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/content-moderator/quickstart.sh" ID="imagemod":::

Скопируйте команду в текстовый редактор и внесите следующие изменения:

1. Назначьте `Ocp-Apim-Subscription-Key` действительному ключу подписки на службу "Распознавание лиц".
1. Измените первую часть URL-адреса запроса, указав конечную точку, соответствующую ключу подписки.
1. При необходимости измените URL-адрес `"Value"` в тексте запроса на любое изображение из удаленного расположения, для которого нужно выполнить модерацию.

> [!TIP]
> Вы также можете выполнить модерацию локальных изображений, передав их байтовые данные в текст запроса. Инструкции см. в [справочной документации](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c).

После внесения изменений откройте командную строку и введите новую команду. 

### <a name="examine-the-results"></a>Просмотр результатов

В окне консоли отобразятся результаты модерации изображения в формате JSON. 

```json
{
  "AdultClassificationScore": x.xxx,
  "IsImageAdultClassified": <Bool>,
  "RacyClassificationScore": x.xxx,
  "IsImageRacyClassified": <Bool>,
  "AdvancedInfo": [],
  "Result": false,
  "Status": {
    "Code": 3000,
    "Description": "OK",
    "Exception": null
  },
  "TrackingId": "<Request Tracking Id>"
}
```

Дополнительные сведения об атрибутах изображения, доступных в Content Moderator, см. в разделе [Основные принципы модерации изображений](../../image-moderation-api.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководстве вы узнали, как модерировать содержимое с помощью REST API Content Moderator. Теперь узнайте больше о модерации изображений или другого мультимедиа, прочитав концептуальное руководство.

* [Принципы модерации изображений](../../image-moderation-api.md)
* [Понятия модерации текста](../../text-moderation-api.md)
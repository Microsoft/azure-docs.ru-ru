---
title: Модерация текстового содержимого с использованием API модерации текста — Content Moderator
titleSuffix: Azure Cognitive Services
description: Опробуйте модерацию текста с помощью API модерации текста в веб-консоли.
services: cognitive-services
author: PatrickFarley
ms.author: pafarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 05/29/2019
ms.openlocfilehash: 664c4289cbfa1f6ce2fce9f9f83b0240bd2d592c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96002006"
---
# <a name="moderate-text-from-the-api-console"></a>Модерация текста с помощью консоли API

Используйте [API-интерфейс](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f) для контроля текста в Azure Content Moderator, чтобы проверить текстовое содержимое на ненормативную лексику и сравнить его с настраиваемыми и общими списками.

## <a name="get-your-api-key"></a>Получение ключа API

Прежде чем можно будет опробовать API в веб-консоли, необходимо получить ключ подписки. Он указан на вкладке **Settings** (Параметры) в поле **Ocp-Apim-Subscription-Key**. Дополнительные сведения см. в этом [обзоре](overview.md).

## <a name="navigate-to-the-api-reference"></a>Переход к справочнику по API

Перейдите к [справочнику по API модерации текста](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f). 

  Откроется страница **Text - Screen** (Текст — проверка).

## <a name="open-the-api-console"></a>Открытие консоли API

Для параметра **Open API testing console** (Открытая тестовая консоль API) выберите регион, лучше всего подходящий для вашего расположения. 

  ![Выбор региона на странице "Text - Screen" (Текст — проверка)](images/test-drive-region.png)

  Откроется консоль API **Text - Screen** (Текст — проверка).

## <a name="select-the-inputs"></a>Выбор входных данных

### <a name="parameters"></a>Параметры

Выберите параметры запроса, которые требуется использовать при проверке текста. В этом примере используется значение по умолчанию для параметра **language** (Язык). Можно также оставить его пустым, так как эта операция автоматически определяет предполагаемый язык в ходе выполнения.

> [!NOTE]
> Для параметра **language** (язык) задайте значение `eng` или оставьте это поле пустым, чтобы в ответе отобразился результат **classification** (классификация), которая сейчас выполняется в режиме предварительной версии. **Эта функция поддерживает только английский язык**.
>
> Для обнаружения **ненормативной лексики** укажите [код ISO 639-3](http://www-01.sil.org/iso639-3/codes.asp) для поддерживаемых языков, которые перечислены в этой статье, или оставьте это поле пустым.

Для параметров **autocorrect** (Автозамена), **PII** (Личные сведения) и **classify (preview)** (Классификация (предварительная версия)) выберите значение **true** (Истина). Оставьте поле **ListId** (ИД списка) пустым.

  ![Параметры запроса в консоли "Text - Screen" (Текст — проверка)](images/text-api-console-inputs.PNG)

### <a name="content-type"></a>Тип содержимого

Для параметра **Content-Type** (Тип содержимого) выберите тип проверяемого содержимого. В этом примере используйте тип содержимого по умолчанию, **text/plain**. В поле **Ocp-Apim-Subscription-Key** введите свой ключ подписки.

### <a name="sample-text-to-scan"></a>Пример текста для проверки

В поле **Request body** (Текст запроса) введите какой-либо текст. В следующем примере показана преднамеренная опечатка в тексте.

```
Is this a grabage or crap email abcdef@abcd.com, phone: 4255550111, IP: 
255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.
```

## <a name="analyze-the-response"></a>Анализ ответа

Приведенный ниже ответ показывает различные важные сведения из API. Она содержит потенциально ненормативную лексику, персональные данные, классификацию (Предварительная версия) и автоматическую исправленную версию.

> [!NOTE]
> Функция машинной классификации находится в режиме предварительной версии и поддерживает только английский язык.

```json
{
   "original_text":"Is this a grabage or crap email abcdef@abcd.com, phone: 
   6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.",
   "normalized_text":"   grabage  crap email abcdef@abcd.com, phone: 
   6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.",
   "auto_corrected_text":"Is this a garbage or crap email abcdef@abcd.com, phone: 
   6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.",
   "status":{
      "code":3000,
      "description":"OK"
   },
   "pii":{
      "email":[
         {
            "detected":"abcdef@abcd.com",
            "sub_type":"Regular",
            "text":"abcdef@abcd.com",
            "index":32
         }
      ],
      "ssn":[

      ],
      "ipa":[
         {
            "sub_type":"IPV4",
            "text":"255.255.255.255",
            "index":72
         }
      ],
      "phone":[
         {
            "country_code":"US",
            "text":"6657789887",
            "index":56
         }
      ],
      "address":[
         {
            "text":"1 Microsoft Way, Redmond, WA 98052",
            "index":89
         }
      ]
   },
   "language":"eng",
   "terms":[
      {
         "index":12,
         "original_index":21,
         "list_id":0,
         "term":"crap"
      }
   ],
   "tracking_id":"WU_ibiza_65a1016d-0f67-45d2-b838-b8f373d6d52e_ContentModerator.
   F0_fe000d38-8ecd-47b5-a8b0-4764df00e3b5"
}
```

Подробное описание всех разделов в ответе JSON см. в концептуальном руководстве по поиску [текста](text-moderation-api.md) .

## <a name="next-steps"></a>Дальнейшие действия

Используйте REST API в коде или следуйте инструкциям в [кратком руководстве по пакету SDK для .NET](./client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp) , чтобы интегрировать его с приложением.
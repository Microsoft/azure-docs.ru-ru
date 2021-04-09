---
title: Поддержка языков — API проверки орфографии Bing
titleSuffix: Azure Cognitive Services
description: Список естественных языков, поддерживаемых API проверки орфографии Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: aahi
ms.openlocfilehash: a97bd6bfc1cb7a0760894faaf3fd0617e6165f58
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "96351203"
---
# <a name="language-and-region-support-for-bing-spell-check-api"></a>Поддержка языков и регионов в API проверки орфографии Bing

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

В API проверки орфографии Bing поддерживаются перечисленные ниже языки (только в режиме `spell`).

Учтите, что для работы с любым языком, кроме `en-US`, необходимо задать `mkt`, а также `Accept-Language` или `setLang`, как описано в [справочнике по API проверки орфографии версии 7](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference).

| Язык    | Код языка |
|:----------- |:-------------:|
| Арабский      | `ar`          |
| Китайский (Китай)     | `zh-CN`          |
| китайский (Гонконг, CАР)    | `zh-HK`          |
| Китайский (Тайвань)     | `zh-TW`          |
| Датский      | `da`          |
| Нидерландский (Бельгия)       | `nl-BE`          |
| Нидерландский (Нидерланды)      | `nl-NL`          |
| Английский (Австралия)    | `en-AU`          |
| Английский (Канада)     | `en-CA`          |
| Английский (Индия)    | `en-IN`          |
| Английский (Индонезия)     | `en-ID`          |
| Английский (Малайзия)     | `en-MY`          |
| Английский (Новая Зеландия)    | `en-NZ`          |
| Английский (Филиппины)     | `en-PH`          |
| Английский (Южная Африка)    | `en-ZA`          |
| Английский (Великобритания)    | `en-GB`          |
| Английский (США)    | `en-US`          |
| Финский     | `fi`          |
| Французский (Бельгия)     | `fr-BE`          |
| Французский (Канада)     | `fr-CA`          |
| Французский (Франция)     | `fr-FR`          |
| Французский (Швейцария)      | `fr-CH`          |
| Немецкий (Австрия)      | `de-AT`          |
| Немецкий (Германия)      | `de-DE`          |
| Немецкий (Швейцария)      | `de-CH`          |
| Итальянский     | `it`          |
| Японский    | `ja`          |
| Корейский      | `ko`          |
| Норвежский   | `no`          |
| Польский      | `pl`          |
| Португальский (Бразилия)   | `pt-BR`|
| Португальский (Португалия) | `pt-PT`|
| русском языке     | `ru`          |
| Испанский (Аргентина)    | `es-AR`          |
| Испанский (Чили)     | `es-CL`          |
| Испанский (Мексика)    | `es-MX`          |
| испанский (Испания)    | `es-ES`          |
| Испанский (США)    | `es-US`          |
| Шведский     | `sv`          |
| Турецкий     | `tr`          |

## <a name="see-also"></a>См. также раздел

- [Страница документации по службам Cognitive Services](../index.yml)
- [Страница продукта Cognitive Services](https://azure.microsoft.com/services/cognitive-services/)
---
title: Руководством по миграции для версии 2 API анализа текста
titleSuffix: Azure Cognitive Services
description: Узнайте, как переместить приложения для использования версии 3 API анализа текста.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 01/22/2021
ms.author: aahi
ms.openlocfilehash: 416ef4ceddbb43e9f1606d44a66ffd5295cee4e6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101699901"
---
# <a name="migrate-to-version-3x-of-the-text-analytics-api"></a>Переход на версию 3. x API анализа текста

Если вы используете версию 2,1 API анализа текста, эта статья поможет вам обновить приложение для использования версии 3. x. Версия 3,0 является общедоступной и содержит новые функции, такие как расширенное [Распознавание сущностей (NER)](how-tos/text-analytics-how-to-entity-linking.md#named-entity-recognition-features-and-versions) и [Управление версиями модели](concepts/model-versioning.md). Также доступна предварительная версия версии 3.1 (версия 3.1-Preview. x), которая добавляет такие функции, как [интеллектуальный анализ данных об мнениях](how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features). Модели, используемые в версии 2, не получат будущие обновления. 

## <a name="sentiment-analysis"></a>[Пример. Как определить тональность с помощью Анализа текста](#tab/sentiment-analysis)

### <a name="feature-changes"></a>Изменения функций 

Анализ тональности в версии 2,1 возвращает баллы тональности в диапазоне от 0 до 1 для каждого документа, отправленного в API, с оценками ближе к 1, что означает более положительный тональности. Версия 3 вместо этого возвращает метки тональности (например, "положительный" или "отрицательный") как для предложений, так и для документа в целом и связанных с ними оценок достоверности. 

### <a name="steps-to-migrate"></a>Шаги для миграции

#### <a name="rest-api"></a>REST API

Если приложение использует REST API, обновите конечную точку запроса до конечной точки v3 для анализа тональности. Например: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/sentiment` . Кроме того, необходимо обновить приложение, чтобы использовать метки тональности, возвращаемые в [ответе API](how-tos/text-analytics-how-to-sentiment-analysis.md#view-the-results). 

Примеры ответа JSON см. в справочной документации.
* [Версия 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c9)
* [Версия 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Sentiment) 
* [Версия 3.1 (предварительная версия)](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Sentiment)

#### <a name="client-libraries"></a>Клиентские библиотеки

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

## <a name="ner-and-entity-linking"></a>[NER и связывание сущностей](#tab/named-entity-recognition)

### <a name="feature-changes"></a>Изменения функций

В версии 2,1 API анализа текста использует одну конечную точку для распознавания именованных сущностей (NER) и связывания сущностей. Версия 3 предоставляет Расширенное обнаружение именованных сущностей и использует отдельные конечные точки для запросов NER и связывания сущностей. Начиная с версии 3.1 – Preview. 1, NER может дополнительно определять персональные `pii` данные и `phi` сведения о работоспособности. 

### <a name="steps-to-migrate"></a>Шаги для миграции

#### <a name="rest-api"></a>REST API

Если приложение использует REST API, обновите конечную точку запроса до конечных точек v3 для связывания NER и (или) сущности.

Связывание сущностей
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/linking`

NER
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/recognition/general`

Кроме того, необходимо обновить приложение, чтобы использовать [категории сущностей](named-entity-types.md) , возвращенные в [ответе API](how-tos/text-analytics-how-to-entity-linking.md#view-results).

Примеры ответа JSON см. в справочной документации.
* [Версия 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/5ac4251d5b4ccd1554da7634)
* [Версия 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/EntitiesRecognitionGeneral) 
* [Версия 3.1 (предварительная версия)](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/EntitiesRecognitionGeneral)

#### <a name="client-libraries"></a>Клиентские библиотеки

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

#### <a name="version-21-entity-categories"></a>Категории сущностей версии 2,1

В следующей таблице перечислены категории сущностей, возвращаемые для NER версии 2.1.

| Категория   | Описание                          |
|------------|--------------------------------------|
| Человек   |   Имена людей.  |
|Расположение    | Естественные и человеческие ориентиры, структуры, географические функции и геофункциональные объекты |
|Организация | Компании, «неправительственные группы», «музыкальные зоны», «спорт», «государственные органы» и «общественные организации». Национальные и религионсы не включаются в этот тип сущности. |
| PhoneNumber | Номера телефонов (только для телефонных номеров США и ЕС). |
| Email | Адреса электронной почты. |
| URL-адрес | URL-адреса веб-сайтов. |
| IP-адрес | Сетевые IP-адреса. |
| Дата/время | Даты и время суток.| 
| Дата | Календарные даты. |
| время; | Время суток |
| Диапазон дат | Диапазоны дат. |
| Диапазон времени | Диапазоны времени. |
| Duration | Длительности. |
| Присвойте параметру | Набор, повторяющиеся значения времени. |
| Количество | Числа и числовые величины. |
| Число | числа. |
| Процент | Процентов.|
| Порядковый номер | Порядковые номера. |
| возраст; | Устаревают. |
| Валюта | Друг. |
| Измерение | Измерения и измерения. |
| температура; | Измеряем. |

## <a name="language-detection"></a>[Пример. Как определить язык с помощью Анализа текста](#tab/language-detection)

### <a name="feature-changes"></a>Изменения функций 

В версии 3 изменились выходные данные компонента определения языка. Ответ JSON будет содержать `ConfidenceScore` вместо `score` . V3 также возвращает только один язык в  `detectedLanguage` атрибуте для каждого документа.

### <a name="steps-to-migrate"></a>Шаги для миграции

#### <a name="rest-api"></a>REST API

Если приложение использует REST API, обновите конечную точку запроса до конечной точки v3 для определения языка. Например: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/languages` . Также потребуется обновить приложение для использования `ConfidenceScore` вместо `score` в [ответе API](how-tos/text-analytics-how-to-language-detection.md#step-3-view-the-results). 

Примеры ответа JSON см. в справочной документации.
* [Версия 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7)
* [Версия 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages) 
* [Версия 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Languages)

#### <a name="client-libraries"></a>Клиентские библиотеки

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

## <a name="key-phrase-extraction"></a>[Пример. Как извлечь ключевые фразы с помощью Анализа текста](#tab/key-phrase-extraction)

### <a name="feature-changes"></a>Изменения функций 

Функция извлечения ключевых фраз не изменилась в версии v3 за пределами конечной точки.

### <a name="steps-to-migrate"></a>Шаги для миграции

#### <a name="rest-api"></a>REST API

Если приложение использует REST API, обновите конечную точку запроса до конечной точки v3 для извлечения ключевых фраз. Пример: `https://<your-custom-subdomain>.api.cognitiveservices.azure.com/text/analytics/v3.0/keyPhrases`

Примеры ответа JSON см. в справочной документации.
* [Версия 2.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c6)
* [Версия 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) 
* [Версия 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/KeyPhrases)

#### <a name="client-libraries"></a>Клиентские библиотеки

[!INCLUDE [Client library migration information](includes/client-library-migration-section.md)]

---

## <a name="see-also"></a>См. также раздел

* [Что такое API "Анализ текста"?](overview.md)
* [Поддержка языков](language-support.md)
* [управления версиями моделей;](concepts/model-versioning.md)

---
title: Ссылки на документацию по обогащению данных с помощью ИИ
titleSuffix: Azure Cognitive Search
description: Аннотированный перечень статей, учебников, примеров и записей блога, связанных с рабочими нагрузками обогащения данных с помощью ИИ в службе "Когнитивный поиск Azure".
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: eaafb9536a8667417c0abf4ab570609f2db290bc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88935353"
---
# <a name="documentation-resources-for-ai-enrichment-in-azure-cognitive-search"></a>Ресурсы документации для обогащения данных с помощью ИИ в Когнитивном поиске Azure

Обогащение искусственного интеллекта — это надстройка индексирования на основе индексатора, которая находит скрытые сведения в нетекстовых источниках и неотличающемся тексте, преобразуя ее в полнотекстовый Поисковый контент в Когнитивный поиск Azure. 

Для встроенной обработки предварительно обученные модели искусственного интеллекта в Cognitive Services вызываются внутренним образом для выполнения анализа. Вы также можете интегрировать пользовательские модели с помощью Машинное обучение Azure, функций Azure или других подходов.

Ниже приведен объединенный список документации для обогащения искусственного интеллекта.

## <a name="concepts"></a>Основные понятия

+ [Дополнения искусственного интеллекта](cognitive-search-concept-intro.md)
+ [Наборы навыков](cognitive-search-working-with-skillsets.md)
+ [Сеансы отладки](cognitive-search-debug-session.md)
+ [Хранилища знаний](knowledge-store-concept-intro.md)
+ [Проекции](knowledge-store-projection-overview.md)
+ [Добавочное дополнение (повторное использование кэшированного обогащенного документа)](cognitive-search-incremental-indexing-conceptual.md)

## <a name="hands-on-walkthroughs"></a>Практические руководства

+ [Краткое руководство. Создание набора квалификационных навыков в портал Azure](cognitive-search-quickstart-blob.md)
+ [Учебник. обогащенное индексирование с помощью искусственного интеллекта](cognitive-search-tutorial-blob.md)
+ [Учебник. Диагностика, восстановление и фиксация изменений в наборе навыков с помощью сеансов отладки](cognitive-search-tutorial-debug-sessions.md)

## <a name="knowledge-stores"></a>Хранилища знаний

+ [Краткое руководство. Создание хранилища знаний в портал Azure](knowledge-store-create-portal.md)
+ [Создание хранилища знаний с помощью REST и Postman](knowledge-store-create-rest.md)
+ [Просмотр хранилища знаний с помощью Обозревателя службы хранилища](knowledge-store-view-storage-explorer.md)
+ [Подключение хранилища знаний с помощью Power BI](knowledge-store-connect-power-bi.md)
+ [Примеры проекции (как формировать и экспортировать дополнения)](knowledge-store-projections-examples.md)

## <a name="custom-skills-advanced"></a>Пользовательские навыки (дополнительно)

+ [How to add a custom skill to a cognitive search pipeline](cognitive-search-custom-skill-interface.md) (Добавление пользовательского навыка в конвейер когнитивного поиска)
+ [Пример. Создание пользовательского навыка с помощью функций Azure (и Поиск сущностей Bing API)](cognitive-search-create-custom-skill-example.md)
+ [Пример. Создание пользовательского навыка с помощью Python](cognitive-search-custom-skill-python.md)
+ [Пример. Создание пользовательского навыка с помощью распознавателя форм](cognitive-search-custom-skill-form.md) 
+ [Пример. Создание пользовательского навыка с помощью Машинное обучение Azure](cognitive-search-tutorial-aml-custom-skill.md) 

## <a name="how-to-guidance"></a>Практические руководства

+ [Вложение ресурса Cognitive Services](cognitive-search-attach-cognitive-services.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)
+ [ссылки на аннотации в наборе навыков](cognitive-search-concept-annotations-syntax.md)
+ [Сопоставьте поля с индексом](cognitive-search-output-field-mapping.md)
+ [Извлечение информации из изображений и ее обработка](cognitive-search-concept-image-scenarios.md)
+ [Настройка кэширования для добавочного дополнения](search-howto-incremental-index.md)
+ [Как перестроить индекс службы "Когнитивный поиск Azure"](search-howto-reindex.md)
+ [Советы по проектированию](cognitive-search-concept-troubleshooting.md)
+ [Распространенные ошибки и предупреждения](cognitive-search-common-errors-warnings.md)

## <a name="skills-reference"></a>Справочник по навыкам

+ [Встроенные навыки](cognitive-search-predefined-skills.md)
  + [Microsoft.Skills.Text.KeyPhraseExtractionSkill](cognitive-search-skill-keyphrases.md)
  + [Microsoft.Skills.Text.LanguageDetectionSkill](cognitive-search-skill-language-detection.md)
  + [Microsoft.Skills.Text.EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md)
  + [Microsoft.Skills.Text.MergeSkill](cognitive-search-skill-textmerger.md)
  + [Microsoft.Skills.Text.PIIDetectionSkill](cognitive-search-skill-pii-detection.md)
  + [Microsoft.Skills.Text.SplitSkill](cognitive-search-skill-textsplit.md)
  + [Microsoft.Skills.Text.SentimentSkill](cognitive-search-skill-sentiment.md)
  + [Microsoft.Skills.Text.TranslationSkill](cognitive-search-skill-text-translation.md)
  + [Microsoft.Skills.Vision.ImageAnalysisSkill](cognitive-search-skill-image-analysis.md)
  + [Microsoft.Skills.Vision.OcrSkill](cognitive-search-skill-ocr.md)
  + [Microsoft.Skills.Util.ConditionalSkill](cognitive-search-skill-conditional.md)
  + [Microsoft.Skills.Util.DocumentExtractionSkill](cognitive-search-skill-document-extraction.md)
  + [Microsoft.Skills.Util.ShaperSkill](cognitive-search-skill-shaper.md)

+ Пользовательские навыки
  + [Microsoft. Skills. Custom. Амлскилл](cognitive-search-aml-skill.md)
  + [Microsoft.Skills.Custom.WebApiSkill](cognitive-search-custom-skill-web-api.md)

+ [Устаревшие навыки](cognitive-search-skill-deprecated.md)
  + [Microsoft.Skills.Text.NamedEntityRecognitionSkill](cognitive-search-skill-named-entity-recognition.md)

## <a name="apis"></a>API-интерфейсы

+ [REST API](/rest/api/searchservice/)
  + [Создание набора навыков (API-Version = 2020-06-30)](/rest/api/searchservice/create-skillset)
  + [Create индексатор (API-Version = 2020-06-30)](/rest/api/searchservice/create-indexer)

## <a name="see-also"></a>См. также раздел

+ [REST API Когнитивного поиска Azure](/rest/api/searchservice/)
+ [Indexers in Azure Cognitive Search](search-indexer-overview.md) (Индексаторы в службе "Когнитивный поиск Azure")
+ [Что собой представляет Когнитивный поиск Azure?](search-what-is-azure-search.md)
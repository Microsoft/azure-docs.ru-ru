---
title: Новые возможности API анализа текста
titleSuffix: Text Analytics - Azure Cognitive Services
description: В этой статье содержатся сведения о новых выпусках и функциях Анализ текста Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 03/18/2021
ms.author: aahi
ms.custom: references_regions
ms.openlocfilehash: a2b001d34d265c8e7246b03875c32168f2c5c962
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104598904"
---
# <a name="whats-new-in-the-text-analytics-api"></a>Новые возможности в API "Анализ текста"

API анализа текста обновляется на постоянной основе. В этой статье содержатся сведения о новых выпусках и функциях, чтобы оставаться в курсе последних нововведений.

## <a name="march-2021"></a>Март 2021 г.

### <a name="general-api-updates"></a>Общие обновления API
* Выпуск нового API версии 3.1-Preview. 4, который включает 
   * Изменения в тексте ответа JSON интеллектуального анализа данных. 
      * `aspects` Сейчас `targets` и `opinions` сейчас `assessments` . 
   * Изменения в тексте ответа JSON размещенного веб-API Анализ текста для работоспособности: 
      * `isNegated`Логическое имя обнаруженного объекта сущности для отрицания является устаревшим и заменяется функцией обнаружения утверждения.
      * Новое свойство, именуемое `role` , теперь является частью извлеченной связи между атрибутом и сущностью, а также отношением между сущностями.  Это добавляет конкретную определенную связь к обнаруженному типу связи.
   * Связывание сущностей теперь доступно в качестве асинхронной задачи в `/analyze` конечной точке.
   * `pii-categories`Теперь в конечной точке доступен новый параметр `/pii` .
      * Этот параметр позволяет указать вариант выбора личных объектов, а также тех, которые не поддерживаются по умолчанию для языка ввода.
* Обновлены клиентские библиотеки, которые включают в себя асинхронное анализ и Анализ текста для операций работоспособности. Примеры можно найти на сайте GitHub:

    * [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
    * [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
    * [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)
    * [JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples/javascript)
    
> [!div class="nextstepaction"]
> [Дополнительные сведения о API анализа текста версии 3.1-Preview. 4](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-4/operations/Languages)

### <a name="text-analytics-for-health-updates"></a>Анализ текста обновлений работоспособности

* Новая версия модели `2021-03-01` для `/health` конечной точки и локального контейнера, которые предоставляют
    * Переименование `Gene` типа сущности в `GeneOrProtein` .
    * Новый `Date` тип сущности.
    * Обнаружение утверждения, которое заменяет обнаружение отрицания (доступно только в API версии 3.1-Preview. 4).
    * Новое предпочитаемое `name` свойство для связанных сущностей, нормализованное из различных онтологиес и систем кодирования (доступно только в API версии 3.1 – Preview. 4). 
* В `3.0.015370001-onprem-amd64` `2021-03-01` репозиторий предварительной версии контейнера была выпущен новый образ контейнера с тегом, а также новая версия модели. 
* Анализ текста для образа контейнера работоспособности будет перемещен в новый репозиторий в следующем месяце.  Просмотрите сообщения электронной почты о расположении новой домашней папки.
> [!div class="nextstepaction"]
> [Дополнительные сведения о Анализ текста для работоспособности](how-tos/text-analytics-for-health.md)
>

### <a name="text-analytics-resource-portal-update"></a>Обновление портала ресурсов Анализ текста
* **Обработанные текстовые записи** теперь доступны в качестве метрики в разделе " **мониторинг** " для ресурса анализ текста в портал Azure.  

## <a name="february-2021"></a>Февраль 2021 года

* `2021-01-15`Версия модели для конечной точки PII в [названии сущности](how-tos/text-analytics-how-to-entity-linking.md) версии 3.1-Preview. x, которая предоставляет 
  * Расширенная поддержка 9 новых языков
  * Улучшенное качество AI категорий именованных сущностей для поддерживаемых языков.
* Ценовые категории S0 – S4 выводятся на 8 марта 2021 г. Если у вас есть ресурс Анализ текста с использованием ценовой категории S0 – S4, необходимо обновить его для использования стандартной [ценовой категории](how-tos/text-analytics-how-to-call-api.md#change-your-pricing-tier)(S).
* [Контейнер обнаружения языка](how-tos/text-analytics-how-to-install-containers.md?tabs=sentiment) теперь общедоступен.
* Версия 2.1 API прекращена. 

## <a name="january-2021"></a>Январь 2021 г.

* `2021-01-15`Версия модели для [имени объекта распознавания сущности](how-tos/text-analytics-how-to-entity-linking.md) v3. x, которая предоставляет 
  * Расширенная языковая поддержка для [нескольких общих категорий сущностей](named-entity-types.md). 
  * Улучшенные категории качества искусственного интеллекта общих сущностей для всех поддерживаемых языков версии v3. 

* `2021-01-05`Версия модели для [определения языка](how-tos/text-analytics-how-to-language-detection.md), которая обеспечивает дополнительную [поддержку языка](language-support.md?tabs=language-detection).

В настоящее время эти версии модели недоступны в регионе Восточной части США. 

> [!div class="nextstepaction"]
> [Дополнительные сведения о новой модели NER](https://azure.microsoft.com/updates/text-analytics-ner-improved-ai-quality)

## <a name="december-2020"></a>Декабрь 2020 г.

* [Обновлены](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) сведения о ценах на API анализа текста.

## <a name="november-2020"></a>Ноябрь 2020 г.

* [Новая конечная точка](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Analyze) с API анализа текста v 3.1-Preview. 3 для нового асинхронного [API-интерфейса анализа](how-tos/text-analytics-how-to-call-api.md?tabs=analyze), который поддерживает пакетную обработку для операций извлечения NER, PII и ключевых фраз.
* [Новая конечная точка](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Health) с API анализа текста версии 3.1-Preview. 3 для нового асинхронного [анализ текста для API службы работоспособности](how-tos/text-analytics-for-health.md) с поддержкой пакетной обработки.
* Новые функции, перечисленные выше, доступны только в следующих регионах: `West US 2` , `East US 2` , `Central US` `North Europe` и `West Europe` .
* Поддержка португальского языка (Бразилия) `pt-BR` теперь поддерживается в [Анализ тональности](how-tos/text-analytics-how-to-sentiment-analysis.md) v3. x, начиная с версии модели `2020-04-01` . Он добавляет к существующей `pt-PT` поддержке португальского языка.
* Обновлены клиентские библиотеки, которые включают в себя асинхронное анализ и Анализ текста для операций работоспособности. Примеры можно найти на сайте GitHub:

    * [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
    * [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
    * [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)
    * 
> [!div class="nextstepaction"]
> [Дополнительные сведения о API анализа текста версии 3.1-Preview. 3](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Languages)

## <a name="october-2020"></a>Октябрь 2020 г.

* Поддержка хинди для анализ тональности v3. x, начиная с версии модели `2020-04-01` . 
* Версия модели `2020-09-01` для конечной точки/лангуажес v3, которая расширяет возможности определения и точности языка.
* v3 доступность в Центральной Индии и Северной ОАЭ.

## <a name="september-2020"></a>Сентябрь 2020 г.

### <a name="general-api-updates"></a>Общие обновления API

* Выпустите новый URL-адрес для общедоступной предварительной версии Анализ текста v 3.1 для поддержки обновлений следующих конечных точек распознавания сущностей V3: 
    * `/pii` Конечная точка теперь включает в себя новое `redactedText` свойство в JSON ответа, где обнаруженные персональные объекты в входном тексте заменяются на `*` для каждого символа этих сущностей.
    * `/linking` Теперь конечная точка содержит `bingID` свойство в JSON ответа для связанных сущностей.
* Следующие конечные точки API предварительной версии Анализ текста были сняты 4 сентября 2020:
    * Версия 2.1 — Предварительная версия
    * Версия 3.0 (предварительная версия)
    * Версия 3.0-Preview. 1
    
> [!div class="nextstepaction"]
> [Дополнительные сведения о API анализа текста версии 3.1-Preview. 2](quickstarts/client-libraries-rest-api.md)

### <a name="text-analytics-for-health-container-updates"></a>Анализ текста для обновлений контейнера работоспособности

Следующие обновления относятся только к выпуску Анализ текста только для контейнера работоспособности.
* В `1.1.013530001-amd64-preview` `2020-09-03` репозиторий предварительной версии контейнера была выпущен новый образ контейнера с тегом с новой версией Model-Version. 
* Эта версия модели предоставляет улучшенные возможности распознавания сущностей, определения сокращения и улучшения задержки.

> [!div class="nextstepaction"]
> [Дополнительные сведения о Анализ текста для работоспособности](how-tos/text-analytics-for-health.md)

## <a name="august-2020"></a>Август 2020 г.

### <a name="general-api-updates"></a>Общие обновления API

* Версия модели `2020-07-01` для v3 `/keyphrases` , `/pii` а также `/languages` конечные точки, которые добавляют:
    * Дополнительные [категории сущностей](named-entity-types.md?tabs=personal) для государственных организаций и страны для распознавания именованных сущностей.
    * Поддержка норвежского и турецкого языков в анализ тональности v3.
* Теперь будет возвращена ошибка HTTP 400 для запросов API V3, превышающих [пределы](concepts/data-limits.md)опубликованных данных. 
* Конечные точки, которые возвращают смещение, теперь поддерживают необязательный `stringIndexType` параметр, который корректирует возвращаемые `offset` `length` значения и в соответствии с поддерживаемой [схемой индексов строк](concepts/text-offsets.md).

### <a name="text-analytics-for-health-container-updates"></a>Анализ текста для обновлений контейнера работоспособности

Следующие обновления относятся только к выпуску Анализ текста за Август только для контейнера работоспособности.

* Новая модель — версия для Анализ текста работоспособности: `2020-07-24`
* Новый URL-адрес для отправки Анализ текста для запросов на работоспособность: `http://<serverURL>:5000/text/analytics/v3.2-preview.1/entities/health` (Обратите внимание, что для использования демонстрационного веб-приложения, входящего в этот новый образ контейнера, требуется очистка кэша браузера)

Изменены следующие свойства в ответе JSON:

* `type` был переименован в `category`. 
* `score` был переименован в `confidenceScore`.
* Сущности в `category` поле выходных данных JSON теперь находятся в стиле Pascal. Следующие сущности были переименованы:
    * `EXAMINATION_RELATION` переименован в `RelationalOperator`.
    * `EXAMINATION_UNIT` переименован в `MeasurementUnit`.
    * `EXAMINATION_VALUE` переименован в `MeasurementValue`.
    * `ROUTE_OR_MODE` был переименован `MedicationRoute` .
    * Реляционная сущность `ROUTE_OR_MODE_OF_MEDICATION` была переименована в `RouteOfMedication` .

Добавлены следующие сущности:

* NER
    * `AdministrativeEvent`
    * `CareEnvironment`
    * `HealthcareProfession`
    * `MedicationForm` 

* Извлечение связей
    * `DirectionOfCondition`
    * `DirectionOfExamination`
    * `DirectionOfTreatment`

> [!div class="nextstepaction"]
> [Дополнительные сведения о Анализ текста контейнере работоспособности](how-tos/text-analytics-for-health.md)

## <a name="july-2020"></a>Июль 2020 г. 

### <a name="text-analytics-for-health-container---public-gated-preview"></a>Анализ текста для контейнера работоспособности — Предварительная версия с открытой проверкой

Анализ текста для контейнера работоспособности теперь находится в общедоступной предварительной версии, которая позволяет извлекать сведения из неструктурированного текста на английском языке в клинической практике документах, таких как формы пациента принимать, заметки Doctor, справочные материалы и сводки о разрядах. В настоящее время не взимается плата за Анализ текста для использования контейнеров работоспособности.

Контейнер предоставляет следующие возможности:

* Распознавание именованных сущностей
* Извлечение связей
* Связывание сущностей
* Отрицание

## <a name="may-2020"></a>Май 2020 г.

### <a name="text-analytics-api-v3-general-availability"></a>Общая доступность API анализа текста v3

API анализа текста v3 теперь общедоступен со следующими обновлениями:

* Версия модели `2020-04-01`
* Новые [ограничения данных](concepts/data-limits.md) для каждого компонента
* Обновленная [языковая поддержка](language-support.md) для [Анализ тональности (SA) v3](how-tos/text-analytics-how-to-sentiment-analysis.md)
* Отдельная конечная точка для связывания сущностей 
* Новая категория сущностей "адрес" в [распознавании имен сущностей (NER) v3](how-tos/text-analytics-how-to-entity-linking.md).
* Новые подкатегории в NER V3:
   * Location — География
   * Расположение — структурное
   * Организация — обмен на бирже
   * Организация — медицинских
   * Организация — Спорт
   * Событие — язык и региональные стандарты
   * Естественное событие
   * Событие — Спорт

Были добавлены следующие свойства в ответе JSON:
   * `SentenceText` в анализ тональности
   * `Warnings` для каждого документа 

Имена следующих свойств в ответе JSON были изменены, где это применимо:

* `score` был переименован в `confidenceScore`.
    * `confidenceScore` имеет две десятичные точки точности. 
* `type` был переименован в `category`.
* `subtype` был переименован в `subcategory`.

> [!div class="nextstepaction"]
> [Дополнительные сведения о API анализа текста v3](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages)

### <a name="text-analytics-api-v31-public-preview"></a>Общедоступная Предварительная версия API анализа текста версии 3.1
   * Новое анализ тональности функции — [интеллектуальный анализ данных](how-tos/text-analytics-how-to-sentiment-analysis.md#opinion-mining)
   * Новый личный `PII` фильтр домена () для защищенных сведений о работоспособности ( `PHI` ).

> [!div class="nextstepaction"]
> [Дополнительные сведения о предварительной версии API анализа текста v 3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-1/operations/Languages)

## <a name="february-2020"></a>Февраль 2020 г.

### <a name="sdk-support-for-text-analytics-api-v3-public-preview"></a>Поддержка пакета SDK для общедоступной предварительной версии API анализа текста v3

В рамках [единой версии пакета SDK для Azure](https://techcommunity.microsoft.com/t5/azure-sdk/january-2020-unified-azure-sdk-release/ba-p/1097290)пакет sdk для API анализа текста v3 теперь доступен в виде общедоступной предварительной версии для следующих языков программирования:
   * [C#](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-csharp&tabs=version-3)
   * [Python](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-python&tabs=version-3)
   * [JavaScript (Node.js)](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-javascript&tabs=version-3)
   * [Java](./quickstarts/client-libraries-rest-api.md?pivots=programming-language-java&tabs=version-3)
   
> [!div class="nextstepaction"]
> [Дополнительные сведения о пакете SDK для API анализа текста v3](./quickstarts/client-libraries-rest-api.md?tabs=version-3)

### <a name="named-entity-recognition-v3-public-preview"></a>Общедоступная Предварительная версия распознавания имен сущностей v3

Дополнительные типы сущностей теперь доступны в общедоступной предварительной версии службы распознавания сущностей (NER) v3, так как мы расширяем обнаружение сущностей общих и личных сведений, найденных в тексте. В этом обновлении представлена [версия модели](concepts/model-versioning.md) `2020-02-01` , которая включает в себя:

* Распознавание следующих общих типов сущностей (только на английском языке):
    * персонтипе
    * Продукт
    * Событие
    * Геоадминистративная сущность (ГПЕ) в качестве подтипа в расположении
    * Навык

* Распознавание следующих типов сущностей личных сведений (только на английском языке):
    * Человек
    * Организация
    * Возраст в качестве подтипа по количеству
    * Дата в качестве подтипа в разделе DateTime
    * электронная почта; 
    * Номер телефона (только США)
    * URL-адрес
    * IP-адрес

### <a name="october-2019"></a>Октябрь 2019 г.

#### <a name="named-entity-recognition-ner"></a>Распознавание именованных сущностей (NER)

* [Новая конечная точка](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/EntitiesRecognitionPii) для распознавания типов сущностей личных сведений (только на английском языке)

* Разделяйте конечные точки для [распознавания сущностей](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/EntitiesRecognitionGeneral) и [связывания сущностей](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/EntitiesLinking).

* [Версия модели](concepts/model-versioning.md) `2019-10-01` , которая включает в себя:
    * Расширенное обнаружение и классификация сущностей, найденных в тексте. 
    * Распознавание следующих новых типов сущностей:
        * Номер телефона
        * IP-адрес

Связывание сущностей поддерживает английский и испанский языки. Языковая поддержка NER зависит от типа сущности.

#### <a name="sentiment-analysis-v3-public-preview"></a>Общедоступная предварительная версия 3 анализа тональности

* [Новая конечная точка](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-2/operations/Sentiment) для анализа тональности.
* [Версия модели](concepts/model-versioning.md) `2019-10-01` , которая включает в себя:

    * Значительные улучшения в точности и подробностях классификации и оценки текста API.
    * Автоматическая маркировка для различных тональности в тексте.
    * Тональности анализ и вывод на уровне документа и предложения. 

Он поддерживает английский ( `en` ), японский ( `ja` ), китайский упрощенный () `zh-Hans` , китайский (традиционное письмо `zh-Hant` ), французский ( `fr` ), итальянский ( `it` ), Испанский ( `es` ), Нидерландский ( `nl` ), португальский () и `pt` немецкий ( `de` ) и доступен в следующих регионах:,,,,, `Australia East` `Central Canada` `Central US` `East Asia` `East US` `East US 2` , `North Europe` , `Southeast Asia` ,,, `South Central US` `UK South` `West Europe` и `West US 2` . 

> [!div class="nextstepaction"]
> [Дополнительные сведения о анализ тональности v3](how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features)

## <a name="next-steps"></a>Дальнейшие действия

* [Что такое API "Анализ текста"?](overview.md)  
* [Примеры пользовательских сценариев](text-analytics-user-scenarios.md)
* [Пример. Как определить тональность с помощью Анализа текста](how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Пример. Как определить язык с помощью Анализа текста](how-tos/text-analytics-how-to-language-detection.md)
* [Распознавание сущностей](how-tos/text-analytics-how-to-entity-linking.md)
* [Пример. Как извлечь ключевые фразы с помощью Анализа текста](how-tos/text-analytics-how-to-keyword-extraction.md)

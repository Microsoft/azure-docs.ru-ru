---
title: Выберите домен для Пользовательское визуальное распознавание проекта — Компьютерное зрение
titleSuffix: Azure Cognitive Services
description: В этой статье показано, как выбрать домен для проекта в Пользовательская служба визуального распознавания.
services: cognitive-services
author: shonohs
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 03/06/2020
ms.author: shono
ms.openlocfilehash: 8aba6f13957d37f843114572f001029baf41ded6
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889354"
---
# <a name="select-a-domain-for-a-custom-vision-project"></a>Выберите домен для Пользовательское визуальное распознавание проекта

На вкладке Параметры проекта Пользовательское визуальное распознавание можно выбрать домен для проекта. Выберите домен, ближайший к вашему сценарию. Если вы получаете доступ к Пользовательское визуальное распознавание через клиентскую библиотеку или REST API, вам потребуется указать идентификатор домена при создании проекта. Вы можете получить список идентификаторов доменов и [получить домены](https://westus2.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeab)или использовать приведенную ниже таблицу.

## <a name="image-classification"></a>Классификация изображений

|Домен|Назначение|
|---|---|
|__Общие сведения__| Рассчитан на самые разные задачи классификации изображений. Если ни один из других доменов не подходит или если вы не знаете, какой домен следует выбрать, выберите один из общих доменов. Идентификатор: `ee85a74c-405e-4adc-bb47-ffa8ca0c9f31`|
|__Общие [a1]__| Оптимизировано для повышения точности благодаря сравнимому времени вывода как к общему домену. Рекомендуется для больших наборов данных или более сложных пользовательских сценариев. Для этого домена требуется дополнительное время обучения. Идентификатор: `a8e3c40f-fb4a-466f-832a-5e457ae4a344`|
|__Общие [a2]__| Оптимизировано для повышения точности с более быстрым временем вывода, чем общие [a1] и общие домены. Рекомендуется для большинства наборов данных. Для этого домена требуется меньше времени обучения, чем для общих и общих доменов [a1]. Идентификатор: `2e37d7fb-3a54-486a-b4d6-cfc369af0018` |
|__еда;__|Рассчитан для фотографий блюд, которые будут отображаться в меню ресторана. Если вы хотите классифицировать фотографии отдельных фруктов или овощей, используйте домен Food. Идентификатор: `c151d5b5-dd07-472a-acc8-15d29dea8518`|
|__Ориентиры__|Рассчитан на распознавание естественных и искусственных ориентиров. Этот домен работает лучше всего, когда ориентир четко виден на фотографии. Этот домен работает, даже если ориентир немного заслоняют люди. Идентификатор: `ca455789-012d-4b50-9fec-5bb63841c793`|
|__Розничная версия__|Рассчитан на изображения из каталогов товаров и торговых веб-сайтов. Если вы хотите классифицировать с высокой точностью между Дрессес, штаны и рубашки, используйте этот домен. Идентификатор: `b30a91ae-e3c1-4f73-a81e-c270bff27c39`|
|__Домены Compact__| Оптимизировано для ограничений классификации в реальном времени на пограничных устройствах.|


> [!NOTE]
> Общие домены [a1] и общие [a2] можно использовать для широкого набора сценариев и оптимизированы для точности. Используйте общую модель [a2], чтобы улучшить скорость вывода и сократить время обучения. Для больших наборов данных может потребоваться использовать общие [a1] для визуализации лучшей точности, чем общая [a2], хотя это требует больше времени на обучение и вывод. Общая модель требует больше времени на вывод, чем общие [a1] и общие [a2].

## <a name="object-detection"></a>Обнаружение объектов

|Домен|Назначение|
|---|---|
|__Общие сведения__| Рассчитан на самые разные задачи обнаружения объектов. Если ни один из других доменов не подходит или вы не знаете, какой домен выбрать, выберите общий домен. Идентификатор: `da2e3a8a-40a5-4171-82f4-58522f70fbc1`|
|__Общие [a1]__| Оптимизировано для повышения точности благодаря сравнимому времени вывода как к общему домену. Рекомендуется для более точных потребностей в расположении регионов, больших наборов данных или более сложных пользовательских сценариев. Для этого домена требуется дополнительное время обучения, и результаты не являются детерминированными: предполагается, что разница в значении +-1% средней точности (mAP) с теми же данными обучения предоставлено. Идентификатор: `9c616dff-2e7d-ea11-af59-1866da359ce6`|
|__Логотип__|Оптимизировано для поиска марочных логотипов в изображениях. Идентификатор: `1d8ffafe-ec40-4fb2-8f90-72b3b6cecea4`|
|__Продукты на полках__|Оптимизировано для обнаружения и классификации продуктов на полках. Идентификатор: `3780a898-81c3-4516-81ae-3a139614e1f3`|
|__Домены Compact__| Оптимизировано для ограничений обнаружения объектов в режиме реального времени на пограничных устройствах.|

## <a name="compact-domains"></a>Домены Compact

Модели, созданные доменами Compact, можно экспортировать для локального запуска. В API общедоступной предварительной версии Пользовательское визуальное распознавание 3,4 можно получить список экспортируемых платформ для компактных доменов, вызвав API-интерфейс-домены.

Производительность модели зависит от выбранного домена. В таблице ниже мы получаем размер модели и время их вывода на ЦП Intel Desktop и NVidia GPU \[ 1 \] . Эти числа не включают предварительную обработку и время обработки.

|Задача|Домен|ID|Размер модели|Время вывода ЦП|Время вывода GPU|
|---|---|---|---|---|---|
|Классификация|General (compact) (Общий (компактный))|`0732100f-1a38-4e49-a514-c9b44c697ab5`|6 МБ|10 мс|5 мс|
|Классификация|Общие (Compact) [S1]|`a1db07ca-a19a-4830-bae8-e004a42dc863`|43 МБ|50 мс|5 мс|
|Обнаружение объектов|General (compact) (Общий (компактный))|`a27d5ca5-bb19-49d8-a70a-fec086c47f5b`|45 МБ|35 МС|5 мс|
|Обнаружение объектов|Общие (Compact) [S1]|`7ec2ac80-887b-48a6-8df9-8b1357765430`|14 МБ|27 МС|7 мс|

>[!NOTE]
>Для обнаружения объектов в __общем (компактном)__ домене требуется специальная логика обработки попроцессоров. Дополнительные сведения см. в примере сценария в экспортированном ZIP-пакете. Если требуется модель без логики обработки попроцессоров, используйте __Общие (Compact) [S1]__.

>[!IMPORTANT]
>Нет никакой гарантии, что экспортированные модели дают точно тот же результат, что и API-интерфейс прогнозирования в облаке. Небольшая разница в выполняющейся платформе или реализации предварительной обработки может привести к увеличению разницы в выходных данных модели. Подробные сведения о логике предварительной обработки см. в [этом документе](quickstarts/image-classification.md).

\[1 процессор \] Intel Xeon 2690 и NVIDIA Tesla M60

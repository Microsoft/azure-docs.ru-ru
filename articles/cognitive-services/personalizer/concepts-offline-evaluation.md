---
title: Использование автономного метода оценки — Персонализация
titleSuffix: Azure Cognitive Services
description: В этой статье объясняется, как использовать автономную оценку для измерения эффективности работы приложения и анализа цикла обучения.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: 627f511bb12c16c8f54935d1f782cb7c2c962163
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87132761"
---
# <a name="offline-evaluation"></a>Автономная оценка

Автономная оценка — это метод, позволяющий тестировать и оценивать эффективность службы "Персонализатор" без изменения кода или влияния на пользовательский интерфейс. Автономная Оценка использует последние данные, отправленные из приложения в API ранжирования и вознаграждений, для сравнения того, как выполняются разные ранги.

Автономная оценка выполняется по диапазону дат. Диапазон может достигать текущего времени. Начало диапазона не может превышать число дней, заданных для [хранения данных](how-to-settings.md).

Автономная оценка может помочь ответить на следующие вопросы:

* Насколько эффективны ранги службы "Персонализатор" для успешной персонализации?
    * Сколько в среднем вознаграждений достигается политикой машинного онлайн-обучения службы "Персонализатор"?
    * Как служба "Персонализатор" сравнивает эффективность того, что приложение сделало бы по умолчанию?
    * Какова была бы сравнительная эффективность случайного выбора для персонализации?
    * Какова бы была сравнительная эффективность различных политики обучения, указанных вручную?
* Какие функции контекста в большей или меньшей степени способствуют персонализации?
* Какие функции действий в большей или меньшей степени способствуют персонализации?

Кроме того, автономную оценку можно использовать для обнаружения более оптимизированных политик обучения, которые Персонализатор может использовать для улучшения результатов в будущем.

Автономные оценки не предоставляют руководств относительно процентной доли событий для использования в исследованиях.

## <a name="prerequisites-for-offline-evaluation"></a>Предварительные требования для автономной оценки

Ниже приведены важные рекомендации для репрезентативной автономной оценки:

* Достаточное количество данных. Рекомендуемое минимальное значение — по крайней мере 50 000 событий.
* Сбор данных в периоды репрезентативного поведения пользователей и трафика.

## <a name="discovering-the-optimized-learning-policy"></a>Обнаружение оптимизированной политики обучения

Служба "Персонализатор" может использовать процесс автономной оценки для автоматического обнаружения более оптимальной политики обучения.

После автономной оценки вы увидите эффективность службы "Персонализатор" с этой новой политикой по сравнению с текущей онлайн-политикой. Затем можно применить эту политику обучения, чтобы она немедленно стала эффективной в персонализации, загрузив ее и отгрузив на панели моделей и политики. Вы также можете скачать его для дальнейшего анализа или использования.

Текущие политики, включаемые в оценку:

| Параметры обучения | Назначение|
|--|--|
|**Политика в сети**| Текущая политика обучения, используемая в Персонализаторе. |
|**Базовый**|Значение по умолчанию для приложения (определяется первым действием, отправленным в ранжированных вызовах).|
|**Произвольная политика**|Мнимое поведение ранга, которое всегда возвращает случайный выбор действий из предоставленных.|
|**Настраиваемые политики**|Дополнительные политики обучения загружены при запуске оценки.|
|**Оптимизированная политика**|Если оценка была начата с возможностью обнаружения оптимизированной политики, она также будет сравниваться, и вы сможете загрузить ее или сделать подключенной политикой обучения, заменив ею текущую.|

## <a name="understanding-the-relevance-of-offline-evaluation-results"></a>Основные сведения о релевантности результатов автономной оценки

При выполнении автономной оценки очень важно проанализировать _доверительные пределы_ результатов. Если они широки, это означает, что приложение не получило достаточно данных для точной или значимой оценки вознаграждения. По мере накопления системой большего объема данных и удлинения автономных оценок доверительные интервалы сужаются.

## <a name="how-offline-evaluations-are-done"></a>Как выполняется автономная оценка

Автономные оценки выполняются с помощью метода под названием **контрфактивная проверка**.

Служба "Персонализатор" основана на предположении, что поведение пользователей (и таким образом вознаграждения) невозможно предсказать в ретроспективе (служба "Персонализатор" не может знать, что бы произошло, если бы пользователю было показано нечто иное, чем то, что он видел), и только учится на измеряемых вознаграждениях.

Это концептуальный процесс, используемый для оценки:

```
[For a given _learning policy), such as the online learning policy, uploaded learning policies, or optimized candidate policies]:
{
    Initialize a virtual instance of Personalizer with that policy and a blank model;

    [For every chronological event in the logs]
    {
        - Perform a Rank call

        - Compare the reward of the results against the logged user behavior.
            - If they match, train the model on the observed reward in the logs.
            - If they don't match, then what the user would have done is unknown, so the event is discarded and not used for training or measurement.

    }

    Add up the rewards and statistics that were predicted, do some aggregation to aid visualizations, and save the results.
}
```

Автономная оценка использует только наблюдаемое поведение пользователя. Этот процесс удаляет большие объемы данных, особенно в том случае, если приложение вызывает ранги с большим количеством действий.


## <a name="evaluation-of-features"></a>Оценка функций

Автономные оценки могут предоставить информацию о том, насколько важны конкретные характеристики действий или контекста для получения более высоких вознаграждений. Информация рассчитывается с использованием оценки по отношению к периоду времени и данным, и может меняться со временем.

Мы рекомендуем ознакомиться с оценками функций и задать такие вопросы:

* Какие другие, дополнительные, возможности может предоставить ваше приложение или система, вроде тех, которые более эффективны?
* Какие функции можно удалить из-за низкой эффективности? Низкоэффективные функции добавляют _шум_ в машинное обучение.
* Существуют ли функции, которые включены случайно? Примеры: сведения, идентифицирующие пользователя, дублирующиеся идентификаторы и т. д.
* Существуют ли нежелательные функции, которые не должны использоваться при персонализации из-за нормативных требований или правил ответственного использования? Существуют ли функции, которые схожи с (т. е точно повторяют или коррелируют с) нежелательными?


## <a name="next-steps"></a>Дальнейшие действия

[Настройка персонализации](how-to-settings.md) 
 [Запуск автономных вычислений](how-to-offline-evaluation.md) Узнайте, [как работает Персонализация](how-personalizer-works.md)

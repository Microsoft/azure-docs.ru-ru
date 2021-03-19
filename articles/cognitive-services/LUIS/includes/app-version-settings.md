---
title: включить файл
ms.topic: include
ms.custom: include file
ms.date: 5/17/2020
ms.openlocfilehash: 79a45825be66b44d869ef741917436de87a1f5d0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83653199"
---
Изучите [основные понятия](../luis-concept-utterance.md#utterance-normalization-for-diacritics-and-punctuation) нормализации и узнайте о том, как с помощью API-интерфейсов [версии](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) обновить эти параметры. Кроме того вы можете использовать раздел **Управление** на портале LUIS на странице **Параметры**.


|Параметры пользовательского интерфейса|Параметр API|Сведения|
|--|--|--|
|Использование недетерминированного обучения|`UseAllTrainingData`|При обучении используется небольшой процент отрицательных выборок. Если вы хотите использовать все данные вместо небольшой отрицательной выборки, задайте значение `true`. |
|Нормализация диакритических знаков|`NormalizeDiacritics`|При нормализации диакритических знаков символы с диакритическими знаками заменяются в речевых фрагментах обычными символами. Этот параметр доступен только для [языков](../luis-reference-application-settings.md#diacritics-normalization), поддерживающих диакритические знаки.|
|Нормализация знаков препинания|`NormalizePunctuation`|Нормализация знаков препинания означает, что прежде чем модели будут обучены и запросы конечной точки спрогнозированы, знаки препинания будут удалены из речевых фрагментов.|
|Нормализация словоформ|`NormalizeWordForm`|Игнорируются все словоформы, кроме корня слова.|

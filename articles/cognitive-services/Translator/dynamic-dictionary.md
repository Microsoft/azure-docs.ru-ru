---
title: Динамический словарь — переводчик
titleSuffix: Azure Cognitive Services
description: В этой статье объясняется, как использовать функцию динамического словаря транслятора Azure Cognitive Services.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: lajanuar
ms.openlocfilehash: ebfccb6fed46db65f6a70937516552f5280c8016
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98897770"
---
# <a name="how-to-use-a-dynamic-dictionary"></a>Использование динамического словаря

Если вы уже знаете перевод, который хотите применить к слову или фразе, вы можете указать его в запросе как исправление. Динамический словарь является надежным только для составных существительных, таких как правильные имена и названия продуктов.

**Синтаксис**

<мстранс: словарный перевод = "перевод фразы" >фраза</мстранс: dictionary>

**Требования.**

* `From`Языки и `To` должны включать английский и другой поддерживаемый язык. 
* Необходимо включить `From` параметр в запрос перевода API, а не использовать функцию автоопределения. 

**Пример: en-de:**

Вход источника: `The word <mstrans:dictionary translation=\"wordomatic\">word or phrase</mstrans:dictionary> is a dictionary entry.`

Выходные данные целевого объекта: `Das Wort "wordomatic" ist ein Wörterbucheintrag.`

Эта возможность работает одинаково как в режиме HTML, так и без него.

Используйте эту функцию с осторожностью. Лучшим способом настройки перевода является использование настраиваемого транслятора. Custom Translator обеспечивает полное использование контекста и статистические значения вероятности. Если вы имеете или можете создавать учебные данные, которые показывают вашу работу или фразу в контексте, полученные результаты будут гораздо лучше. Дополнительные сведения о настраиваемом трансляторе см. в статье [https://aka.ms/CustomTranslator](https://aka.ms/CustomTranslator) .
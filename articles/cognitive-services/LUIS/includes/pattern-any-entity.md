---
title: включить файл
description: включить файл
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 02/14/2020
ms.topic: include
ms.openlocfilehash: 5eb2517b4b71b61dd70406f345c005d007d9e3ef
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91535969"
---
Сущность Pattern.any позволяет найти данные произвольной формы, в которых формулировка сущности затрудняет определение ее окончание из остальной части высказывания.

Приложение для управления персоналом помогает сотрудникам находить формы организации.

|Фраза|
|--|
|Где находится **HRF-123456**?|
|Кто написал **HRF-123234**?|
|Было ли **HRF-456098** опубликовано на французском языке?|

Тем не менее в каждой из форм присутствует как форматированное имя, используемое в предыдущей таблице, так и такое понятное имя, как `Request relocation from employee new to the company 2018 version 5`.

Фразы с понятным названием формы выглядят следующим образом.

|Фраза|
|--|
|Где находится **Request relocation from employee new to the company 2018 version 5**?|
|Кто написал **Request relocation from employee new to the company 2018 version 5**?|
|Был ли **Request relocation from employee new to the company 2018 version 5** опубликован во Франции?|

Заголовки переменной длины включают машинные слова, из-за которых LUIS может неправильно определить окончание сущности. С помощью шаблонной сущности Pattern.any можно указать начало и конец названия формы, что позволит LUIS правильно извлекать ее имя.

|Пример шаблона высказывания|
|--|
|Где находится {FormName} [?]|
|Кто создатель {FormName} [?]|
|Было ли {FormName} опубликовано на французском языке[?]|

См. справочник по [Pattern.any](../reference-entity-pattern-any.md).
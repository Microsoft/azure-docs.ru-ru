---
title: Обзор миграции — LUIS
description: Общие сведения о пути перехода
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/22/2020
ms.openlocfilehash: d6ecacf9aa1a7e650de74a412ed4f161ed0e0790
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91253808"
---
# <a name="migration-in-luis"></a>Миграция в LUIS

Несколько элементов находятся на пути миграции. Используйте следующую таблицу, чтобы понять, что влияет на воздействие и когда требуется выполнить полную миграцию.

|Область|Описание|Дата завершения миграции|
|--|--|--|
|[API-интерфейсы прогнозирования](luis-migration-api-v3.md)|Переход на API V3.|TBD|
|[Разработка API-интерфейсов](luis-migration-authoring-entities.md)|Переход на API V3.|TBD|
|[Создание ресурса](luis-migration-authoring.md)|Выполните миграцию без ресурсов разработки, чтобы использовать ресурс LUIS Authoring на портале LUIS.|2 ноября 2020 г. |
|[Обязательный компонент](luis-migration-authoring-entities.md#api-change-constraint-replaced-with-required-feature)|Это изменение было внесено в мае 2020 на Конференции//Build и относится только к API-интерфейсам разработки версии 3, где приложение использует ограниченную функцию.|19 июня 2020 г.|
|[Составная сущность](migrate-from-composite-entity.md)|Обновите составной объект до сущности машинного обучения, чтобы создать сущность, которая получает более полные прогнозы с улучшенными декомпосабилити для отладки сущности.|TBD|

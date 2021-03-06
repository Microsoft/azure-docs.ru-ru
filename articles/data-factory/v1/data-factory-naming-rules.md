---
title: Правила именования сущностей фабрики данных Azure (версия 1)
description: Описывает правила именования для сущностей фабрики данных v1.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 3c68159f20873aeff5938ab21f348be4a922041c
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104779483"
---
# <a name="rules-for-naming-azure-data-factory-entities"></a>Правила именования сущностей фабрики данных Azure
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [правилах именования в службе "Фабрика данных Azure"](../naming-rules.md).

В следующей таблице приведены правила именования для артефактов фабрики данных.

| Имя | Уникальность имени | Проверки |
|:--- |:--- |:--- |
| Фабрика данных |Уникально в рамках Microsoft Azure. Регистр в именах не учитывается, т. е. `MyDF` и `mydf` ссылаются на одну и ту же фабрику данных. |<ul><li>Каждая фабрика данных привязывается ровно к одной подписке Azure.</li><li>Имена объектов должны начинаться с буквы или цифры и могут содержать только буквы, цифры и дефисы (-).</li><li>Перед каждым знаком тире (-) и после него должна стоять буква или цифра. Последовательные тире в именах контейнеров использовать нельзя.</li><li>Имя может содержать от 3 до 63 символов.</li></ul> |
| Связанные службы/таблицы/конвейеры |Уникально в рамках фабрики данных. Регистр в именах не учитывается. |<ul><li>Максимальное количество символов в имени таблицы: 260.</li><li>Имена объектов должны начинаться с буквы, цифры или символа подчеркивания (_).</li><li>Следующие символы не допускаются: ".", "+", "?", "/", "<", ">", "*", "%", "&", ":", " \\ "</li></ul> |
| Группа ресурсов |Уникально в рамках Microsoft Azure. Регистр в именах не учитывается. |<ul><li>Максимальное количество символов: 1000.</li><li>Имя может содержать буквы, цифры и следующие символы: "-", "_", "," и "."</li></ul> |


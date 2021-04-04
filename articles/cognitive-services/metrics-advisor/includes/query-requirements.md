---
title: Требования к запросам
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 09/30/2020
ms.author: mbullwin
ms.openlocfilehash: 777e3261bd6f842861ed6622b78589eedaaf65d7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "92043219"
---
В запросе используйте параметр `@StartTime`, чтобы получить данные метрик для одной метки времени. Помощник по метрикам заменит параметр строкой формата `yyyy-MM-ddTHH:mm:ss` при выполнении запроса.

> [!IMPORTANT]
> Запрос должен возвращать не более одной записи для каждой комбинации измерения в каждой метке времени. Все записи, возвращаемые запросом, должны иметь одинаковые метки времени. Помощник по метрикам запустит этот запрос для каждой метки времени, чтобы принять данные. Дополнительные сведения и примеры см. в разделе с [вопросами и ответами о запросах](../faq.md#how-do-i-write-a-valid-query-for-ingesting-my-data). 
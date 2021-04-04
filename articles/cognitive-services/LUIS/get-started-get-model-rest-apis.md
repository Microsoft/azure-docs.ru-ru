---
title: Изменение модели LUIS с помощью REST API
titleSuffix: Azure Cognitive Services
description: В этой статье описывается, как добавить примеры речевых фрагментов для изменения модели и обучения приложения.
services: cognitive-services
manager: nitinme
ms.custom: seodec18, devx-track-python, devx-track-js, devx-track-csharp
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 11/30/2020
zone_pivot_groups: programming-languages-set-one
ms.openlocfilehash: 7346037ad804da4f68a2bc6512494c461d7fc5bd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "96436187"
---
# <a name="how-to-change-the-luis-model-with-rest-apis"></a>Изменение модели LUIS с помощью REST API

В этой статье показано, как добавить речевые фрагменты в приложение Pizza и обучить его. Примерами высказываний называют фразы пользователя на обычном языке, сопоставленные с тем или иным намерением. Предоставляя фразы для настроенных намерений, вы сообщаете LUIS ожидаемые варианты сообщений пользователя для каждого намерения.

::: zone pivot="programming-language-csharp"
[!INCLUDE [Get intent with C# and REST](./includes/get-started-get-model-rest-csharp.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Get intent with Java and REST](./includes/get-started-get-model-rest-java.md)]
::: zone-end

::: zone pivot="programming-language-go"
[!INCLUDE [Get intent with Go and REST](./includes/get-started-get-model-rest-go.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Get intent with Node.js and REST](./includes/get-started-get-model-rest-nodejs.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Get intent with Python and REST](./includes/get-started-get-model-rest-python.md)]
::: zone-end

---
title: Запись асинхронного диалога — служба речи
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать асинхронную транскрипцию беседы с помощью службы распознавания речи. Доступно только для Java и C#.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: amishu
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-set-twenty-one
ms.openlocfilehash: 4b2dfa8d474f10d6b4ca1c46ac2b575e8d8407ff
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88934653"
---
# <a name="asynchronous-conversation-transcription"></a>Асинхронное транскрибирование бесед

В этой статье функция записи асинхронного диалога демонстрируется с помощью API **ремотеконверсатионтранскриптионклиент** . Если вы настроили запись разговора для асинхронного транскрипции и имеете `conversationId` возможность, вы можете получить сопоставленную с ним транскрипцию `conversationId` с помощью API **ремотеконверсатионтранскриптионклиент** .

## <a name="asynchronous-vs-real-time--asynchronous"></a>Асинхронное и в режиме реального времени + асинхронное

С помощью асинхронной записи сообщений потоковая передача передается в режиме реального времени, но не требуется. Вместо этого после отправки звука используйте предложение `conversationId` из `Conversation` для запроса состояния асинхронной транскрипции. Когда асинхронная транскрипция готова, вы получите `RemoteConversationTranscriptionResult` .

В режиме реального времени и асинхронных событий вы получаете запись в режиме реального времени, но также можете получить транскрипцию с помощью запроса `conversationId` (аналогично асинхронному сценарию).

Для выполнения асинхронной записи требуются два действия. Первым шагом является Отправка звукового сигнала, выбор асинхронного режима только для асинхронного выполнения или в режиме реального времени. Второй шаг — получение результатов транскрипции.

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/how-to/remote-conversation/csharp/examples.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [prerequisites](includes/how-to/remote-conversation/java/examples.md)]
::: zone-end


## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Примеры на сайте GitHub](https://aka.ms/csspeech/samples)

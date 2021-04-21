---
title: Краткое руководство по транскрибированию бесед в реальном времени — служба "Речь"
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать транскрибирование разговоров в режиме реального времени с помощью речевого пакета SDK. Функция транскрибирования бесед позволяет переводить в текстовую форму встречи и другие разговоры с возможностью добавлять, удалять и обнаруживать несколько участников путем передачи аудиопотока в службу распознавания речи.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/20/2020
ms.author: trbye
zone_pivot_groups: acs-js-csharp
ms.openlocfilehash: 48cd4c7996eabad7293aa2429c76b8943e0ab3da
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "100368478"
---
# <a name="get-started-with-real-time-conversation-transcription"></a>Начало работы с транскрибированием бесед в реальном времени

API-интерфейс **ConversationTranscriber** речевого пакета SDK позволяет переводить в текстовую форму встречи и другие разговоры с возможностью добавлять, удалять и определять несколько участников путем передачи аудиопотоков в службу распознавания речи с помощью `PullStream` или `PushStream`. Сначала нужно создать голосовые подписи для каждого участника с помощью API-интерфейса REST, а затем использовать эти голосовые подписи с пакетом SDK для транскрибирования бесед. С дополнительными сведениями можно ознакомиться в [обзоре](conversation-transcription.md) "Транскрибирование бесед".

## <a name="limitations"></a>Ограничения

* Доступно только в следующих регионах подписки: `centralus`, `eastasia`, `eastus`, `westeurope`
* Необходимо наличие массива из 7-микрофонов с круговым расположением. Массив микрофонов должен соответствовать [нашим техническим требованиям](./speech-devices-sdk-microphone.md).
* В [пакете SDK для речевых устройств](speech-devices-sdk.md) предоставлены совместимые устройства, а также приводится пример приложения с демонстрацией транскрибирования бесед.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](overview.md#try-the-speech-service-for-free).

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript Basics include](includes/how-to/conversation-transcription/real-time-javascript.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [C# Basics include](includes/how-to/conversation-transcription/real-time-csharp.md)]
::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Асинхронное транскрибирование бесед](how-to-async-conversation-transcription.md)
> [Пример кода для устройства ROOBO](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK/blob/master/Samples/Java/Android/Speech%20Devices%20SDK%20Starter%20App/example/app/src/main/java/com/microsoft/cognitiveservices/speech/samples/sdsdkstarterapp/ConversationTranscription.java)
> [Пример кода из пакета для разработчиков Azure Kinect](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK/blob/master/Samples/Java/Windows_Linux/SampleDemo/src/com/microsoft/cognitiveservices/speech/samples/Cts.java)
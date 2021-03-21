---
title: Установка речевых контейнеров
titleSuffix: Azure Cognitive Services
description: Сведения о параметрах конфигурации диаграммы Helm "речь".
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 0257f3af44cc85d0a3656472db224ae5a7e19161
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96002303"
---
### <a name="speech-umbrella-chart"></a>Речь (схема-тег)

Значения на диаграмме "тег-элемент" верхнего уровня переопределяют соответствующие значения во вспомогательной диаграмме. Поэтому необходимо добавить все локальные настраиваемые значения.

|Параметр|Описание|По умолчанию|
| -- | -- | -- | -- |
| `speechToText.enabled` | Включена ли служба **распознавания речи в текст** . | `true` |
| `speechToText.verification.enabled` | Включена ли поддержка `helm test` **распознавания речи в текст** . | `true` |
| `speechToText.verification.image.registry` | Репозиторий образов DOCKER, который `helm test` использует для тестирования службы преобразования **речи в текст** . Helm создает отдельный модуль Pod внутри кластера для тестирования и извлекает из этого реестра образ, *используемый для тестирования* . | `docker.io` |
| `speechToText.verification.image.repository` | Репозиторий образов DOCKER, который `helm test` использует для тестирования службы преобразования **речи в текст** . Модуль тестирования Helm использует этот репозиторий для извлечения образа *тестового использования* . | `antsu/on-prem-client` |
| `speechToText.verification.image.tag` | Тег образа DOCKER, используемый с `helm test` для службы преобразования **речи в текст** . Модуль тестирования Helm использует этот тег для извлечения изображения, *используемого для тестирования* . | `latest` |
| `speechToText.verification.image.pullByHash` | Указывает, извлекается ли в *тесте* образ DOCKER с помощью хэша. `true`Значение, если `speechToText.verification.image.hash` необходимо добавить, с допустимым значением хэша изображения. | `false` |
| `speechToText.verification.image.arguments` | Аргументы, используемые для выполнения *теста — использовать* образ DOCKER. Модуль тестирования Helm передает эти аргументы в контейнер при запуске `helm test` . | `"./speech-to-text-client"`<br/> `"./audio/whatstheweatherlike.wav"` <br/> `"--expect=What's the weather like"`<br/>`"--host=$(SPEECH_TO_TEXT_HOST)"`<br/>`"--port=$(SPEECH_TO_TEXT_PORT)"` |
| `textToSpeech.enabled` | Включена ли служба преобразования **текста в речь** . | `true` |
| `textToSpeech.verification.enabled` | Включена ли поддержка `helm test` **распознавания речи в текст** . | `true` |
| `textToSpeech.verification.image.registry` | Репозиторий образов DOCKER, который `helm test` использует для тестирования службы преобразования **речи в текст** . Helm создает отдельный модуль Pod внутри кластера для тестирования и извлекает из этого реестра образ, *используемый для тестирования* . | `docker.io` |
| `textToSpeech.verification.image.repository` | Репозиторий образов DOCKER, который `helm test` использует для тестирования службы преобразования **речи в текст** . Модуль тестирования Helm использует этот репозиторий для извлечения образа *тестового использования* . | `antsu/on-prem-client` |
| `textToSpeech.verification.image.tag` | Тег образа DOCKER, используемый с `helm test` для службы преобразования **речи в текст** . Модуль тестирования Helm использует этот тег для извлечения изображения, *используемого для тестирования* . | `latest` |
| `textToSpeech.verification.image.pullByHash` | Указывает, извлекается ли в *тесте* образ DOCKER с помощью хэша. `true`Значение, если `textToSpeech.verification.image.hash` необходимо добавить, с допустимым значением хэша изображения. | `false` |
| `textToSpeech.verification.image.arguments` | Аргументы для выполнения *с помощью образа DOCKER.* Модуль тестирования Helm передает эти аргументы в контейнер при выполнении `helm test` . | `"./text-to-speech-client"`<br/> `"--input='What's the weather like'"` <br/> `"--host=$(TEXT_TO_SPEECH_HOST)"`<br/>`"--port=$(TEXT_TO_SPEECH_PORT)"` |
---
title: Обзор перевода речи — служба речи
titleSuffix: Azure Cognitive Services
description: Трансляция речи позволяет добавлять в приложения, средства и устройства полноценный перевод речи в реальном времени, а также многоязыковую поддержку. Один API можно использовать как для преобразования речи в речь, так и для преобразования речи в текст. В этой статье приводятся общие сведения о преимуществах и возможностях службы перевода речи.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: erhopf
ms.custom: devx-track-csharp, cog-serv-seo-aug-2020
keywords: перевод речи
ms.openlocfilehash: 94ddd06068513261b5b73b313877e273c7251d62
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104954967"
---
# <a name="what-is-speech-translation"></a>Что такое перевод речи?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

В этом обзоре вы узнаете о преимуществах и возможностях службы перевода речи, которая обеспечивает преобразование речевых потоков [в](language-support.md#speech-translation) режиме реального времени и преобразования речи в текст. С помощью пакета SDK для распознавания речи вашим приложениям, инструментам и устройствам предоставляется доступ к функциям преобразования поступающих аудиоданных в текстовую форму на языке источника и получения перевода. Промежуточные записи и результаты перевода возвращаются при обнаружении речи, а окончательные результаты могут быть преобразованы в синтезированный речевой режим.

## <a name="core-features"></a>Основные компоненты

* Преобразование речи в текст с результатами распознавания.
* Преобразование речи в речь.
* Поддержка перевода на несколько целевых языков.
* Промежуточные результаты распознавания и перевода.

## <a name="get-started"></a>Начало работы 

Чтобы начать работу с переводом речи, ознакомьтесь с [кратким](get-started-speech-translation.md) руководством. Служба перевода речи доступна через [речевой пакет SDK](speech-sdk.md) и [РЕЧЕВОЙ интерфейс командной строки](spx-overview.md).

## <a name="sample-code"></a>Пример кода

Пример кода для пакета SDK для распознавания речи доступен на сайте GitHub. В этих примерах рассматриваются распространенные сценарии, такие как чтение звука из файла или потока, непрерывное распознавание и преобразование с одним снимком, а также работа с пользовательскими моделями.

* [Примеры преобразования речи в текст и перевода (пакет SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)

## <a name="migration-guides"></a>Руководства по переносу

Если приложения, средства или продукты используют [API перевода речи](./how-to-migrate-from-translator-speech-api.md), мы создали руководства, которые помогут вам перейти на службу речи.

* [Миграция из API перевода речи в службу речи](how-to-migrate-from-translator-speech-api.md)

## <a name="reference-docs"></a>Справочная документация

* [пакет SDK для службы "Речь"](./speech-sdk.md);
* [Пакет SDK для речевых устройств](speech-devices-sdk.md)
* [REST API: Преобразование речи в текст](rest-speech-to-text.md)
* [REST API: Преобразование текста в речь](rest-text-to-speech.md)
* [REST API: Пакетное транскрибирование и настройка](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)

## <a name="next-steps"></a>Дальнейшие действия

* Завершение [краткого руководства](get-started-speech-translation.md) по переводу речи
* [Try the Speech service for free](overview.md#try-the-speech-service-for-free) (Бесплатное использование службы "Речь")
* [Получение пакета SDK для службы "Речь"](speech-sdk.md)
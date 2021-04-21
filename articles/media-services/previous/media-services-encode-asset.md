---
title: Обзор кодировщиков мультимедиа Azure по запросу | Документация Майкрософт
description: Службы мультимедиа Azure предоставляют несколько вариантов для кодирования мультимедиа в облаке. В этой статье приводится обзор кодировщиков мультимедиа Azure по запросу.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: 0fcf3aaa830c092dfa225f55fef882e2263b16cd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103013556"
---
# <a name="overview-of-azure-on-demand-media-encoders"></a>Обзор кодировщиков мультимедиа Azure по запросу

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Службы мультимедиа Azure предоставляют несколько вариантов для кодирования мультимедиа в облаке.

При начале работы со службами мультимедиа важно понимать разницу между кодеками и форматами файлов.
Кодеки — это программное обеспечение, которое реализует алгоритмы сжатия и распаковки, тогда как форматы файлов являются контейнерами, в которых хранится сжатое видео.

Службы мультимедиа обеспечивают динамическую упаковку, которая позволяет доставлять закодированное содержимое MP4-файлов с переменной скоростью или Smooth Streaming в форматах потоковой передачи с поддержкой служб мультимедиа (MPEG-DASH, HLS, Smooth Streaming) без необходимости повторной упаковки в эти форматы потоковой передачи.

При создании учетной записи служб мультимедиа в нее добавляется конечная точка потоковой передачи **по умолчанию** в состоянии **Остановлена**. Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи, из которой необходимо выполнять потоковую передачу содержимого, должна находиться в состоянии **Выполняется**. Выставление счетов для конечных точек потоковой передачи происходит, когда конечная точка находится в состоянии **Запущено**.

Службы мультимедиа поддерживают следующий кодировщик по запросу:

* [Стандартный кодировщик служб мультимедиа](media-services-encode-asset.md#media-encoder-standard)

В этом разделе предоставлен краткий обзор кодировщиков мультимедиа по запросу, а также приведены ссылки на разделы с подробными сведениями.

По умолчанию каждая учетная запись служб мультимедиа может выполнять одну активную задачу кодирования в текущий момент. Можно зарезервировать единицы кодирования, которые позволят выполнять несколько задач кодирования одновременно, по одной для каждой приобретенной единицы. Дополнительные сведения см. в статье [Обзор масштабирования обработка мультимедиа](media-services-scale-media-processing-overview.md).

## <a name="media-encoder-standard"></a>Стандартный кодировщик служб мультимедиа

### <a name="how-to-use"></a>Использование
[Кодирование с помощью стандартного кодировщика мультимедиа](media-services-dotnet-encode-with-media-encoder-standard.md)

### <a name="formats"></a>Форматы
[Форматы и кодеки](media-services-media-encoder-standard-formats.md)

### <a name="presets"></a>Предустановки
Стандартный кодировщик мультимедиа настраивается с помощью одной из предустановок кодировщика, описанных [здесь](./media-services-mes-presets-overview.md).

### <a name="input-and-output-metadata"></a>Входные и выходные метаданные
Входные метаданные кодировщиков описаны [здесь](media-services-input-metadata-schema.md).

Выходные метаданные кодировщиков описаны [здесь](media-services-output-metadata-schema.md).

### <a name="generate-thumbnails"></a>Создание эскизов
Дополнительные сведения см. в разделе о [создании эскизов с помощью стандартного кодировщика служб мультимедиа](media-services-advanced-encoding-with-mes.md).

### <a name="trim-videos-clipping"></a>Усечение видео (обрезка)
Дополнительные сведения см. в разделе об [обрезке видео с помощью стандартного кодировщика мультимедиа](media-services-advanced-encoding-with-mes.md#trim_video).

### <a name="create-overlays"></a>Создание наложений
Дополнительные сведения см. в разделе о [создании наложения с помощью стандартного кодировщика мультимедиа](media-services-advanced-encoding-with-mes.md#overlay).

### <a name="see-also"></a>См. также раздел
[Блог, посвященный службам мультимедиа](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)

### <a name="known-issues"></a>Известные проблемы
Если входящее видео не содержит скрытых субтитров, выходящий ресурс по-прежнему будет содержать пустой файл TTML.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-articles"></a>Похожие статьи
* [Выполняйте расширенные задачи кодирования путем настройки предустановок стандартного кодировщика служб мультимедиа](media-services-custom-mes-presets-with-dotnet.md)
* [Квоты и ограничения](media-services-quotas-and-limitations.md)

<!--Reference links in article-->
[1]: https://azure.microsoft.com/pricing/details/media-services/
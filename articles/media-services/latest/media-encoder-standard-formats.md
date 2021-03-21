---
title: Стандартные форматы и кодеки кодировщика — Azure
description: В этой статье содержится список наиболее распространенных форматов для импорта и экспорта файлов, которые можно использовать с StandardEncoderPreset.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.reviewer: anilmur
ms.openlocfilehash: f0cf89a755090010bb80507c52f90baa2a4e804a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102617630"
---
# <a name="standard-encoder-formats-and-codecs"></a>Стандартные форматы и кодеки кодировщика

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье содержится список наиболее распространенных форматов для импорта и экспорта файлов, которые можно использовать с [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset). Сведения о создании настраиваемых предустановок с помощью **StandardEncoderPreset** см. в разделе [Создание преобразования с помощью настраиваемой предустановки](customize-encoder-presets-how-to.md).

## <a name="input-containerfile-formats"></a>Контейнер ввода и форматы файлов

| Форматы файлов (расширения файлов) | Поддерживается |
| --- | --- |
| FLV (с кодеками H.264 и AAC) (.flv) |Да |
| MXF (.mxf) |Да |
| GXF (.gxf) |Да |
| MPEG2-PS, MPEG2-TS, 3GP (.ts, .ps, .3gp, .3gpp, .mpg) |Да |
| Windows Media Video (WMV)/ASF (.wmv, .asf) |Да |
| AVI (без сжатия 8 бит/10 бит) (.avi) |Да |
| MP4 (MP4, M4A, M4V) и ISMV (ISMA, ISMV) |Да |
| [Microsoft Digital Video Recording(DVR-MS)](/previous-versions/windows/desktop/mstv/about-the-dvr-ms-file-format) (.dvr-ms) |Да |
| Matroska/WebM (.mkv) |Да |
| WAVE/WAV (.wav) |Да |
| QuickTime (.mov) |Да |

### <a name="audio-formats-in-input-containers"></a>Форматы аудио во входных контейнерах

Стандартный кодировщик поддерживает следующие звуковые форматы во входных контейнерах:

* файлы MXF, GXF и QuickTime со звуковыми дорожками с чередованием стерео или форматом 5.1;

или диспетчер конфигурации служб

* файлы MXF, GXF и QuickTime, где звук переносится как отдельные дорожки PCM, но сопоставление каналов (стерео или 5.1) можно определить по метаданным файла.

## <a name="input-video-codecs"></a>Входные видеокодеки
| Входные видеокодеки | Поддерживается |
| --- | --- |
| AVC 8-разрядный/10-разрядный, до 4:2:2, включая AVCIntra |8 бит 4:2:0 и 4:2:2 |
| Avid DNxHD (в MXF) |Да |
| DVCPro/DVCProHD (в MXF) |Да |
| Цифровое видео (DV) (в AVI-файлах) |Да |
| JPEG 2000 |Да |
| MPEG-2 (до 422 Profile и High Level, включая такие варианты, как XDCAM, XDCAM HD, XDCAM IMX, CableLabs® и D10) |До 422 Profile |
| MPEG-1 |Да |
| VC-1/WMV9 |Да |
| Canopus HQ/HQX |Нет |
| MPEG-4, часть 2 |Да |
| [Theora](https://en.wikipedia.org/wiki/Theora) |Да |
| YUV420 без сжатия или мезонинный файл |Да |
| Apple ProRes 422 |Да |
| Apple ProRes 422 LT |Да |
| Apple ProRes 422 HQ |Да |
| Apple ProRes Proxy |Да |
| Apple ProRes 4444 |Да |
| Apple ProRes 4444 XQ |Да |
| HEVC/H.265| Профиль Main|

## <a name="input-audio-codecs"></a>Входные аудиокодеки
| Входные аудиокодеки | Поддерживается |
| --- | --- |
| AAC (AAC-LC, AAC-HE и AAC-HEv2; до 5.1) |Да |
| MPEG Layer 2 |Да |
| MP3 (MPEG-1 Audio Layer 3) |Да |
| Windows Media Audio |Да |
| WAV/PCM |Да |
| [FLAC](https://en.wikipedia.org/wiki/FLAC)</a> |Да |
| [Opus](https://go.microsoft.com/fwlink/?LinkId=822667) |Да |
| [Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a> |Да |
| AMR (adaptive multi-rate) |Да |
| AES (SMPTE 331M и 302M, AES3-2003) |Нет |
| Dolby® E |Нет |
| Dolby® Digital (AC3) |Нет |
| Dolby® Digital Plus (E-AC3) |Нет |

## <a name="output-formats-and-codecs"></a>Выходные форматы и кодеки
В следующей таблице перечислены кодеки и форматы файлов, поддерживаемые для экспорта.

| Формат файла | Видеокодек | Аудиокодек |
| --- | --- | --- |
| MP4 <br/><br/>(включая контейнеры MP4 с несколькими скоростями) |H. 264 (высокий, основной и базовый профили), HEVC (H. 265) 8-bit |AAC-LC, HE-AAC v1, HE-AAC v2 |
| MPEG2-TS |H.264 (профили High, Main и Baselin) |AAC-LC, HE-AAC v1, HE-AAC v2 |

## <a name="next-steps"></a>Следующие шаги

[Создание преобразования с помощью настраиваемой предустановки](customize-encoder-presets-how-to.md)

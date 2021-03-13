---
title: Обзор динамической упаковки служб мультимедиа Azure | Документация Майкрософт
description: В этой статье приводятся общие сведения о Cлужбы мультимедиа Microsoft Azure динамической упаковки.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: d2394cba157a7049e3b63b09fda9e018f0a0b32f
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103013624"
---
# <a name="dynamic-packaging"></a>Динамическая упаковка

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Выберите версию служб мультимедиа, которую вы используете:"]
> * [Версия 3](../latest/dynamic-packaging-overview.md)
> * [Версия 2](media-services-dynamic-packaging-overview.md)

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Службы мультимедиа Microsoft Azure можно использовать для передачи содержимого исходных файлов, потоков мультимедиа и защищенного содержимого в различных форматах на вход клиентов, реализованных на базе различных технологий (например, iOS, XBOX, Silverlight или Windows 8). Эти клиенты работают по разным протоколам: например, для iOS используется формат потоковой трансляции HTTP (HLS) V4, а для технологий Silverlight и Xbox необходимо использовать формат Smooth Streaming. Если у вас есть набор файлов MP4 (ISO Base Media 14496-12) с адаптивной скоростью (мультискоростных) или аналогичных файлов Smooth Streaming, которые вам нужно передать клиентам, принимающим форматы MPEG DASH, HLS или Smooth Streaming, вы можете воспользоваться функциями динамической упаковки служб мультимедиа.

В случае динамической упаковки необходимо создать ресурс, содержащий набор MP4-файлов с адаптивной скоростью или файлы с адаптивной скоростью Smooth Streaming. Затем с учетом формата, указанного в манифесте или запросе фрагмента, сервер потоковой передачи по запросу организует передачу содержимого по выбранному протоколу. В результате вы сможете хранить и оплачивать файлы только в одном формате, а службы мультимедиа выполнят сборку и будут обслуживать соответствующий ответ на основе запросов клиента.

На схеме ниже показан рабочий процесс, в котором используются традиционные технологии кодирования и статической упаковки.

![Статическое кодирование](./media/media-services-dynamic-packaging-overview/media-services-static-packaging.png)

На схеме ниже показан рабочий процесс, в котором используются технологии динамической упаковки.

![Динамическое кодирование](./media/media-services-dynamic-packaging-overview/media-services-dynamic-packaging.png)

## <a name="common-scenario"></a>Стандартный сценарий

1. Отправьте входной файл (он также называется мезонинным). Например, это может быть файл в формате H.264, MP4 или WMV (список поддерживаемых форматов см. в статье [Форматы и кодеки стандартного кодировщика служб мультимедиа](media-services-media-encoder-standard-formats.md)).
2. Преобразуйте мезонинный файл в набор мультискоростных MP4-файлов в формате H.264.
3. Опубликуйте ресурс, содержащий набор мультискоростных MP4-файлов, создав указатель OnDemand.
4. Создайте URL-адреса для доступа к содержимому и его потоковой передачи.

## <a name="preparing-assets-for-dynamic-streaming"></a>Подготовка ресурсов для динамической потоковой передачи

Чтобы подготовить ресурс для динамической потоковой передачи, доступны следующие варианты.

- [Отправьте главный файл](media-services-dotnet-upload-files.md).
- [Сформируйте наборы мультискоростных MP4-файлов в формате H.264 с помощью стандартного кодировщика служб мультимедиа](media-services-dotnet-encode-with-media-encoder-standard.md).
- [Выполните потоковую передачу содержимого](media-services-deliver-content-overview.md).

## <a name="audio-codecs-supported-by-dynamic-packaging"></a>Аудиокодеки, поддерживаемые для динамической упаковки

Динамическая упаковка поддерживает MP4-файлы, которые содержат аудио, закодированную с помощью [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding) (AAC-LC, HE-AAC v1, AAC v2), [Dolby Digital Plus](https://en.wikipedia.org/wiki/Dolby_Digital_Plus)(Улучшенный AC-3 или E-AC3), Dolby атмос или [DTS](https://en.wikipedia.org/wiki/DTS_%28sound_system%29) (DTS Express, DTS ЛБР, DTS HD без потерь). Потоковая передача содержимого Dolby атмос поддерживается для таких стандартов, как протокол MPEG-ТИРЕ, с обычным потоковым форматом (CSF) или стандартным форматом приложения мультимедиа (КМАФ) фрагментированным MP4, а также через HTTP Live Streaming (HLS) с КМАФ.

> [!NOTE]
> Для динамической упаковки не поддерживаются файлы с аудиоданными в формате [Dolby Digital](https://en.wikipedia.org/wiki/Dolby_Digital) (AC3) (это устаревший кодек).

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

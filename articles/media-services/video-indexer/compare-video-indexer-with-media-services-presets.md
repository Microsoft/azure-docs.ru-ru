---
title: Сравнение индексаторов видео и предварительных установок служб мультимедиа Azure v3
description: В этой статье сравниваются возможности индексатора видео и предустановки служб мультимедиа Azure версии 3.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.subservice: video-indexer
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/24/2020
ms.author: juliako
ms.openlocfilehash: 0961aa3e573e511a6941bc59ddc335f64799abb4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87047341"
---
# <a name="compare-azure-media-services-v3-presets-and-video-indexer"></a>Сравнение предустановок Служб мультимедиа версии 3 и Индексатора видео 

В этой статье сравниваются возможности **API Индексатора видео** и **API Служб мультимедиа (версия 3)**. 

В настоящее время существует перекрытие функций, предоставляемых [API индексатора видео](https://api-portal.videoindexer.ai/) и API- [интерфейсов служб мультимедиа v3](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01/Encoding.json). В следующей таблице представлены сведения о текущих различиях и сходстве. 

## <a name="compare"></a>Сравнение

|Компонент|API Индексатора видео |Предустановки анализатора видео и звука<br/>в Службах мультимедиа Azure, версия 3|
|---|---|---|
|Аналитические сведения о мультимедиа|[Расширенная архитектура](video-indexer-output-json-v2.md) |[Базовый](../latest/analyzing-video-audio-files-concept.md)|
|Возможности|Ознакомьтесь с полным списком поддерживаемых возможностей в следующей статье: <br/> [Обзор](video-indexer-overview.md)|Возвращает только аналитические сведения о видео|
|Выставление счетов|[Цены на службы мультимедиа](https://azure.microsoft.com/pricing/details/media-services/#analytics)|[Цены на службы мультимедиа](https://azure.microsoft.com/pricing/details/media-services/#analytics)|
|Соответствие нормативным требованиям|Чтобы узнать о последних обновлениях соответствия требованиям, посетите [Offerings.pdfсоответствие требованиям Azure ](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942/file/178110/23/Microsoft%20Azure%20Compliance%20Offerings.pdf) и выполните поиск по запросу "индексатор видео", чтобы проверить, соответствует ли он интересующему сертификату.|Чтобы узнать о последних обновлениях соответствия требованиям, посетите [Offerings.pdfсоответствие требованиям Azure ](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942/file/178110/23/Microsoft%20Azure%20Compliance%20Offerings.pdf) и найдите "службы мультимедиа", чтобы проверить, соответствует ли он интересующему нас сертификату.|
|Пробная версия|Восточная часть США|Недоступно|
|Доступность по регионам|См. раздел [доступность Cognitive Services по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services) .|См. раздел [доступность служб мультимедиа по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=media-services).|

## <a name="next-steps"></a>Дальнейшие действия

[Общие сведения об Индексаторе видео](video-indexer-overview.md)

[Что такое Службы мультимедиа Azure версии 3?](../latest/media-services-overview.md)

---
title: Устаревшие компоненты служб мультимедиа Azure | Документация Майкрософт
description: В этом разделе обсуждаются устаревшие компоненты служб мультимедиа Azure.
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
ms.date: 10/21/2020
ms.author: inhenkel
ms.openlocfilehash: 495b6bc68e79e50a0bda773e7229225595236fac
ms.sourcegitcommit: 77afc94755db65a3ec107640069067172f55da67
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/22/2021
ms.locfileid: "98694348"
---
# <a name="azure-media-services-legacy-components"></a>Устаревшие компоненты служб мультимедиа Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Со временем мы улучшаем компоненты службы мультимедиа и выбытие устаревших компонентов. Эта статья поможет вам перенести приложение из устаревшего компонента в текущий компонент.
 
## <a name="retirement-plans-of-legacy-components-and-migration-guidance"></a>Планы выбытия устаревших компонентов и руководство по миграции

Обработчики мультимедиа *Windows Azure* (Ваме) и устройства мультимедиа *кодировщика мультимедиа Azure* не рекомендуются.

* [Миграция из кодировщика мультимедиа Windows Azure в Media Encoder Standard](migrate-windows-azure-media-encoder.md)
* [Миграция из кодировщика мультимедиа Azure в Media Encoder Standard](migrate-azure-media-encoder.md)

Следующие Аналитика мультимедиа обработчики мультимедиа либо устарели, либо скоро будут нерекомендуемыми.

  
 
| **Имя обработчика мультимедиа** | **Дата выбытия** | **Дополнительные примечания** |
| --- | --- | ---|
| Azure Media Indexer 2 | 1 января 2020 г. | Этот обработчик мультимедиа будет заменен [режимом "службы мультимедиа v3 Аудиоанализерпресет Basic](../latest/analyzing-video-audio-files-concept.md)". Дополнительные сведения см. в статье [Миграция из Azure Media indexer 2 в индексатор видео служб мультимедиа Azure](migrate-indexer-v1-v2.md). |
| Azure Media Indexer | 1 марта 2023 г. | Этот обработчик мультимедиа будет заменен [режимом "службы мультимедиа v3 Аудиоанализерпресет Basic](../latest/analyzing-video-audio-files-concept.md)". Дополнительные сведения см. в статье [Миграция из Azure Media indexer 2 в индексатор видео служб мультимедиа Azure](migrate-indexer-v1-v2.md). |
| Обнаружение движения | 1 июня 2020 г.|В настоящее время нет планов замены. |
| Формирование сводных данных видео |1 июня 2020 г.|В настоящее время нет планов замены.|
| Оптическое распознавание символов видео | 1 июня 2020 г. |Этот обработчик мультимедиа был заменен [индексатором видео служб мультимедиа Azure](../video-indexer/index.yml). Кроме того, рассмотрите возможность использования [API служб мультимедиа Azure v3](../latest/analyzing-video-audio-files-concept.md). <br/>См. статью [Сравнение предустановок и индексатора видео служб мультимедиа Azure v3](../video-indexer/compare-video-indexer-with-media-services-presets.md). |
| Обнаружение лиц | 1 июня 2020 г. | Этот обработчик мультимедиа был заменен [индексатором видео служб мультимедиа Azure](../video-indexer/index.yml). Кроме того, рассмотрите возможность использования [API служб мультимедиа Azure v3](../latest/analyzing-video-audio-files-concept.md). <br/>См. статью [Сравнение предустановок и индексатора видео служб мультимедиа Azure v3](../video-indexer/compare-video-indexer-with-media-services-presets.md). |
| Content Moderator | 1 июня 2020 г. |Этот обработчик мультимедиа был заменен [индексатором видео служб мультимедиа Azure](../video-indexer/index.yml). Кроме того, рассмотрите возможность использования [API служб мультимедиа Azure v3](../latest/analyzing-video-audio-files-concept.md). <br/>См. статью [Сравнение предустановок и индексатора видео служб мультимедиа Azure v3](../video-indexer/compare-video-indexer-with-media-services-presets.md). |

## <a name="next-steps"></a>Дальнейшие шаги

[Руководство по миграции из версии 2 в версию 3 Служб мультимедиа](../latest/migrate-v-2-v-3-migration-introduction.md)

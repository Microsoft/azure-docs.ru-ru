---
title: Руководство по миграции пакетов и поставок
description: Эта статья содержит рекомендации по упаковке и доставке, которые помогут вам переходить со служб мультимедиа Azure версии 2 на v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.workload: media
ms.date: 03/25/2021
ms.author: inhenkel
ms.openlocfilehash: 3a09e3f2bf29c09066e9414f9aa02a7879375425
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105563537"
---
# <a name="packaging-and-delivery-scenario-based-migration-guidance"></a>Руководство по миграции на основе сценариев упаковки и доставки

![логотип с руководством по миграции](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![шаги миграции 2](./media/migration-guide/steps-4.svg)

В этой статье приводятся рекомендации по упаковке и доставке, которые помогут вам переходить со служб мультимедиа Azure версии 2 на v3.

Значительные изменения в способе публикации содержимого в API V3. Новая модель публикации упрощена и использует меньше сущностей для создания указателя потоковой передачи. API уменьшился до двух сущностей и только что требовалось четыре сущности. Политики ключей содержимого и указатели потоковой передачи теперь заменяют необходимость в `ContentKeyAuthoriationPolicy` , `AssetDeliveyPolicy` , `ContentKey` и `AccessPolicy` .

## <a name="packaging-and-delivery-in-v3"></a>Упаковка и доставка в версии 3

1. Создайте [политики ключей содержимого](content-key-policy-concept.md).
1. Создание [указателей потоковой передачи](streaming-locators-concept.md).
1. Получение [путей потоковой передачи](create-streaming-locator-build-url.md) 
    1. Настройте его для [тире](dynamic-packaging-overview.md#mpeg-dash-protocol) или [HLS](dynamic-packaging-overview.md#hls-protocol) проигрывателя.

Подробные инструкции см. в разделе Основные понятия публикации, руководства и инструкции.

## <a name="publishing-concepts-tutorials-and-how-to-guides"></a>Основные понятия публикации, руководства и инструкции

### <a name="concepts"></a>Основные понятия

- [Динамическая упаковка в Службах мультимедиа версии 3](dynamic-packaging-overview.md)
- [Фильтры](filters-concept.md)
- [Фильтрация манифестов с помощью динамического упаковщика](filters-dynamic-manifest-overview.md)
- [Конечные точки потоковой передачи (источник) в службах мультимедиа Azure](streaming-endpoint-concept.md)
- [Потоковая передача содержимого с помощью интеграции CDN](scale-streaming-cdn.md)
- [Указатели потоковой передачи](streaming-locators-concept.md)

### <a name="how-to-guides"></a>Практические руководства

- [Управление конечными точками потоковой передачи с помощью служб мультимедиа v3](manage-streaming-endpoints-howto.md)
- [Пример CLI. Публикация ресурса](cli-publish-asset.md)
- [Создание указателя потоковой передачи и формирование URL-адресов](create-streaming-locator-build-url.md)
- [Скачать результаты задания](download-results-howto.md)
- [Сигнальные звуковые дорожки сигнала](signal-descriptive-audio-howto.md)
- [Полная настройка Проигрыватель мультимедиа Azure](../azure-media-player/azure-media-player-full-setup.md)
- [Использование проигрывателя Video.js с помощью служб мультимедиа Azure](how-to-video-js-player.md)
- [Как использовать проигрыватель Шака со службами мультимедиа Azure](how-to-shaka-player.md)

## <a name="samples"></a>Примеры

Кроме того, можно [сравнить код v2 и v3 в примерах кода](migrate-v-2-v-3-migration-samples.md).

---
title: Руководство по миграции защиты содержимого
description: Эта статья содержит рекомендации по защите содержимого на основе сценариев, которые помогут вам в минимальном переходе с служб мультимедиа Azure версии 2 на v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.workload: media
ms.date: 1/14/2020
ms.author: inhenkel
ms.openlocfilehash: 9c0040c0ad34b019eaa90bbd1571377ad3539578
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98940106"
---
# <a name="content-protection-scenario-based-migration-guidance"></a>Руководство по миграции на основе сценариев защиты содержимого

![логотип с руководством по миграции](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![шаги миграции 2](./media/migration-guide/steps-4.svg)

Эта статья содержит рекомендации по защите содержимого на основе сценариев, которые помогут вам в минимальном переходе с служб мультимедиа Azure версии 2 на v3.

## <a name="protect-content-in-v3-api"></a>Защита содержимого в API V3

Используйте поддержку [многоключовых](design-multi-drm-system-with-access-control.md) функций в новом API V3.

Подробные инструкции см. в разделе Основные понятия защиты содержимого, руководства и инструкции.

## <a name="content-protection-concepts-tutorials-and-how-to-guides"></a>Основные понятия защиты содержимого, руководства и инструкции

### <a name="concepts"></a>Основные понятия

- [Защита содержимого с помощью динамического шифрования служб мультимедиа](content-protection-overview.md)
- [Проектирование системы для защиты содержимого с несколькими подсистемами DRM и управлением доступом](design-multi-drm-system-with-access-control.md)
- [Шаблон лицензии служб мультимедиа v3 с PlayReady](playready-license-template-overview.md)
- [Обзор служб мультимедиа v3 с использованием шаблона лицензии Widevine](widevine-license-template-overview.md)
- [Требования и конфигурация лицензии Apple FairPlay](fairplay-license-overview.md)
- [Политики потоковой передачи](streaming-policy-concept.md)
- [Политики ключей содержимого](content-key-policy-concept.md)

### <a name="tutorials"></a>Учебники

[Краткое руководство. Шифрование содержимого с помощью портала](encrypt-content-quickstart.md)

### <a name="how-to-guides"></a>Практические руководства

- [Получение ключа подписи из существующей политики](get-content-key-policy-dotnet-howto.md)
- [Автономная потоковая передача FairPlay для iOS с помощью служб мультимедиа v3](offline-fairplay-for-ios.md)
- [Автономная потоковая передача Widevine для Android с помощью служб мультимедиа v3](offline-widevine-for-android.md)
- [Автономная потоковая передача PlayReady для Windows 10 с помощью служб мультимедиа v3](offline-plaready-streaming-for-windows-10.md)

## <a name="samples"></a>Примеры

Кроме того, можно [сравнить код v2 и v3 в примерах кода](migrate-v-2-v-3-migration-samples.md).

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [migration guide next steps](./includes/migration-guide-next-steps.md)]
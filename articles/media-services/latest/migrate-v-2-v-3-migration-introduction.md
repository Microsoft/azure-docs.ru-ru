---
title: Общие сведения о переходе с служб мультимедиа v2 на v3
description: Эта статья содержит вводные сведения о переходе с служб мультимедиа версии 2 на v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.workload: media
ms.date: 1/14/2020
ms.author: inhenkel
ms.openlocfilehash: 3514a7c809e939ea2c45afa5ab60539232b8781f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98953297"
---
# <a name="migrate-from-media-services-v2-to-v3-introduction"></a>Общие сведения о переходе с служб мультимедиа v2 на v3

![логотип с руководством по миграции](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

С помощью инструкции по миграции служб мультимедиа вы можете переходить от API служб мультимедиа v2 к API-интерфейсам версии 3 на основе миграции, которая использует преимущества новых функций и функций, которые теперь доступны. Следует использовать наиболее сомнение и определить, что лучше подходит для вашего сценария.

## <a name="how-to-use-this-guide"></a>Как пользоваться руководством

### <a name="navigating"></a>Навигация

В рамках этого руководством вы увидите следующий рисунок.

![этапы миграции](./media/migration-guide/steps.svg)<br/>

На шаге, на который вы указываете, будет указано изменение цвета в номере шага, например так:

![шаги миграции 2](./media/migration-guide/steps-2.svg)<br/>

В конце каждой страницы вы увидите ссылки на остальные документы миграции, которые можно прочитать под заголовком **дальнейшие действия** .

### <a name="guidance"></a>Руководство

Здесь приведены *Общие* рекомендации. В него входит содержимое для улучшения осведомленности о том, что теперь доступно в версии 3, а также об изменениях в рабочих процессах служб мультимедиа.

Для получения более подробных руководств, в том числе снимков экрана и концептуальной графики, существуют ссылки на основные понятия, учебники, краткие руководства, примеры и справочники по API в каждом разделе на основе сценариев. Мы также перечислены примеры, которые помогут сравнить API v2 с API V3.

## <a name="migration-guidance-overview"></a>Обзор руководств по миграции

Во время миграции необходимо выполнить четыре основных шага.

## <a name="step-1-benefits"></a>Шаг 1. преимущества

<hr color="#5ea0ef" size="10">

[Узнайте о преимуществах](migrate-v-2-v-3-migration-benefits.md) перехода на API служб мультимедиа версии 3.

## <a name="step-2-differences"></a>Различия в шагах 2

<hr color="#5ea0ef" size="10">

Ознакомьтесь с различиями между API служб мультимедиа версии 2 и API V3.

- [Доступ через API](migrate-v-2-v-3-differences-api-access.md)
- [Недостатки функций](migrate-v-2-v-3-differences-feature-gaps.md)
- [Изменения терминологии и сущностей](migrate-v-2-v-3-differences-terminology.md)

## <a name="step-3-sdk-setup"></a>Этап 3. Установка пакета SDK

<hr color="#5ea0ef" size="10">

Ознакомьтесь с различиями в пакете SDK и [Настройте миграцию на версии 3 REST API или клиентского пакета SDK](migrate-v-2-v-3-migration-setup.md).

## <a name="step-4-scenario-based-guidance"></a>Шаг 4. рекомендации на основе сценариев

<hr color="#5ea0ef" size="10">

Приложение служб мультимедиа версии 2 может быть уникальным. Поэтому мы предоставили рекомендации на основе сценариев на основе того, как вы *могли* использовать службы мультимедиа в прошлом, и действия для каждого компонента службы, например:

- [Кодирование](migrate-v-2-v-3-migration-scenario-based-encoding.md)
- [Потоковая передача в реальном времени](migrate-v-2-v-3-migration-scenario-based-live-streaming.md)
- [Упаковка и доставка](migrate-v-2-v-3-migration-scenario-based-publishing.md)
- [Защита содержимого](migrate-v-2-v-3-migration-scenario-based-content-protection.md)
- [Зарезервированные единицы мультимедиа](migrate-v-2-v-3-migration-scenario-based-media-reserved-units.md)

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [migration guide next steps](./includes/migration-guide-next-steps.md)]

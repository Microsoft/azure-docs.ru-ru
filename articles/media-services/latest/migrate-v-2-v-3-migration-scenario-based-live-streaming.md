---
title: Руководство по миграции служб мультимедиа на динамическую потоковую передачу
description: Эта статья содержит инструкции по использованию сценария потоковой передачи в реальном времени, которые помогут вам в минимальном переходе с служб мультимедиа Azure версии 2 на v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 1/14/2020
ms.author: inhenkel
ms.openlocfilehash: 6a2c6495ca3685aec1bc132ec7f8a88809ad2d87
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104598292"
---
# <a name="live-streaming-scenario-based-migration-guidance"></a>Руководство по миграции на основе сценариев динамической потоковой передачи

![логотип с руководством по миграции](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![шаги миграции 2](./media/migration-guide/steps-4.svg)

Портал Azure теперь поддерживает интерактивную настройку и управление событиями.  Рекомендуется испытать его при тестировании миграции версии 2 и 3.

## <a name="test-the-v3-live-event-workflow"></a>Тестирование рабочего процесса динамического события v3

> [!NOTE]
> С помощью API V3 можно управлять каналами и программами, созданными с помощью версии 2 (которые сопоставлены с динамическими событиями и Live Outputs в v3). Руководство заключается в том, чтобы переключиться на v3 Live Events и Live Outputs в удобное время, когда можно будет закрыть существующий канал v2. Сейчас нет поддержки беспрепятственного переноса непрерывно запущенного Круглосуточная оперативного канала в новые динамические события v3. Таким образом, время простоя обслуживания должно быть согласовано с наилучшим удобством для вашей аудитории и бизнеса.

Протестируйте новый способ доставки событий в реальном времени с помощью служб мультимедиа, прежде чем перемещать содержимое из v2 в v3. Ниже приведены функции v3, с которыми можно работать и которые следует учитывать при миграции.

- Создайте новое [динамическое событие](live-events-outputs-concept.md#live-events) v3 для кодирования. Вы можете включить [предустановки кодирования 1080p и 720P](live-event-types-comparison.md#system-presets).
- Использование [активной выходной](live-events-outputs-concept.md#live-outputs) сущности вместо программ
- Создание [указателей потоковой передачи](streaming-locators-concept.md).
- Учитывайте потребность в потоковой передаче [HLS и тире](dynamic-packaging-overview.md) .
- Если вам требуется быстрый запуск интерактивных событий, ознакомьтесь с новыми возможностями [режима ожидания](live-events-outputs-concept.md#standby-mode) .
- Если вы хотите транскрипция интерактивное событие во время его выполнения, изучите новую функцию [транскрипции в реальном времени](live-transcription.md) .
- Если требуется более длительная потоковая передача, создайте события круглосуточной в режиме 3.
- Используйте [сетку событий](monitoring/monitor-events-portal-how-to.md) для мониторинга динамических событий.

Конкретные действия см. в статьях Основные понятия, учебники и инструкции по работе с Live Events.

## <a name="live-events-concepts-tutorials-and-how-to-guides"></a>Основные понятия, учебники и руководства по интерактивным событиям

### <a name="concepts"></a>Основные понятия

- [Потоковая трансляция в Службах мультимедиа Azure версии 3](live-streaming-overview.md)
- [События и выходные данные прямой трансляции в Службах мультимедиа](live-events-outputs-concept.md)
- [Проверенные локальные кодировщики динамической потоковой передачи](recommended-on-premises-live-encoders.md)
- [Используйте сдвиги времени и динамические выходные данные, чтобы создать воспроизведение видео по запросу](live-event-cloud-dvr.md)
- [Динамическая транскрипция (Предварительная версия)](live-transcription.md)
- [Сравнение типов событий Live](live-event-types-comparison.md)
- [Активные состояния мероприятий и выставление счетов](live-event-states-billing.md)
- [Параметры низкой задержки в динамических событиях](live-event-latency.md)
- [Коды ошибок динамических событий служб мультимедиа](live-event-error-codes.md)

### <a name="tutorials-and-quickstarts"></a>Учебники и краткие руководства

- [Руководство по Потоковая трансляция в реальном времени с помощью Служб мультимедиа](stream-live-tutorial-with-api.md)
- [Создание прямой трансляции в Службах мультимедиа Azure с помощью OBS](live-events-obs-quickstart.md)
- [Краткое руководство. Отправка, кодирование и потоковая передача содержимого с помощью портала](manage-assets-quickstart.md)
- [Краткое руководство. Создание динамического потока служб мультимедиа Azure с помощью Wirecast](live-events-wirecast-quickstart.md)

## <a name="samples"></a>Примеры

Кроме того, можно [сравнить код v2 и v3 в примерах кода](migrate-v-2-v-3-migration-samples.md).

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [migration guide next steps](./includes/migration-guide-next-steps.md)]

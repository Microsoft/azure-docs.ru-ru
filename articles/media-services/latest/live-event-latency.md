---
title: Лививент параметры низкой задержки в службах мультимедиа Azure
description: В этом разделе приводятся общие сведения о параметрах низкой задержки Лививент и показано, как задать низкую задержку.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: f9f4476e932df9a5f4c093968b2e7c4840e7ff39
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102122907"
---
# <a name="live-event-low-latency-settings"></a>Параметры низкой задержки в динамических событиях

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье показано, как настроить низкую задержку [события потоковой трансляции](/rest/api/media/liveevents). Здесь также рассмотрены результаты настройки низкой задержки в разных проигрывателях. Результаты различаются в зависимости от задержки сети и CDN.

Чтобы использовать новую функцию **LowLatency**, настройте для параметра **StreamOptionsFlag** значение **LowLatency** в **LiveEvent**. При создании [LiveOutput](/rest/api/media/liveoutputs) для воспроизведения HLS задайте для [LiveOutput.Hls.fragmentsPerTsSegment](/rest/api/media/liveoutputs/create#hls) значение 1. После запуска потока можно использовать [проигрыватель мультимедиа Azure](https://ampdemo.azureedge.net/) (демонстрационная страница amp) и задать параметры воспроизведения для использования профиля эвристики с низкой задержкой.

> [!NOTE]
> В настоящее время Ловлатенци Хеуристикпрофиле в Проигрыватель мультимедиа Azure предназначен для воспроизведения потоков в протоколе MPEG-ТИРЕ с помощью формата CSF или КМАФ (например, `format=mdp-time-csf` или `format=mdp-time-cmaf` ). 

В следующем примере .NET показано, как задать **LowLatency** в **LiveEvent**:

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#NewLiveEvent)]

См. полный пример: [Live Event с DVR](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/Live/LiveEventWithDVR/Program.cs).

## <a name="live-events-latency"></a>Задержка событий потоковой трансляции

В следующих таблицах приведены результаты настройки задержки (установленный флаг LowLatency) в Службах мультимедиа. Задержка измеряется с момента, когда веб-канал входного потока достигает службы, и до момента, когда проигрыватель может запросить воспроизведение. Чтобы оптимально использовать низкую задержку, в параметрах кодировщика задайте для длительности группы изображений (GOP) значение менее 1 секунды. Если задать меньшую длительность GOP, вы уменьшите использование пропускной способности и скорость при одинаковой частоте кадров. Это особенно подходит для видео, где меньше движения.

### <a name="pass-through"></a>Сквозной режим 

||Включена двухсекундная GOP с малой задержкой|Включена односекундная GOP с малой задержкой|
|---|---|---|
|**DASH в AMP**|10 с|8 с|
|**HLS на собственном проигрывателе iOS**|14 с|10 с|

### <a name="live-encoding"></a>Кодирование в реальном времени

||Включена двухсекундная GOP с малой задержкой|Включена односекундная GOP с малой задержкой|
|---|---|---|
|**DASH в AMP**|14 с|10 с|
|**HLS на собственном проигрывателе iOS**|18 с|13 с|

> [!NOTE]
> Значение сквозной задержки может зависеть от условий локальной сети или наличия уровня кэширования CDN. Следует протестировать используемые конфигурации.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о потоковой трансляции](live-streaming-overview.md)
- [Руководство по потоковой трансляции](stream-live-tutorial-with-api.md)

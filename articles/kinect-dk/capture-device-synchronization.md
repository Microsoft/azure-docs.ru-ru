---
title: Сбор данных синхронизации устройств Azure Kinect
description: Узнайте, как синхронизировать устройства записи Azure Kinect с помощью пакета SDK для датчика Kinect для Azure.
author: xthexder
ms.author: jawirth
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, Azure, датчик, пакет SDK, глубина, RGB, внутренняя, внешняя, синхронизация, цепочка, смещение этапа
ms.openlocfilehash: ce0c72d3d708d5696a9775b3885f278f0c23cac1
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102034322"
---
# <a name="capture-azure-kinect-device-synchronization"></a>Сбор данных синхронизации устройств Azure Kinect

Оборудование Azure Kinect может сопоставлять время записи изображений цветов и глубины. Выравнивание между камерами на одном устройстве является **внутренней синхронизацией**. Выравнивание времени записи между несколькими подключенными устройствами является **внешней синхронизацией**. Массив микрофона работает независимо от цветов и камер глубины.

## <a name="device-internal-synchronization"></a>Внутренняя синхронизация устройства

Запись образов между отдельными камерами синхронизируется с оборудованием. В каждой [k4a_capture_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__capture__t.html) , содержащей изображения из датчиков цвета и глубины, метки времени изображений вычисляются в зависимости от режима работы оборудования. По умолчанию изображения записи выравниваются по центру. Относительный интервал отслеживания глубины и цвета можно изменить с помощью `depth_delay_off_color_usec` поля [k4a_device_configuration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

## <a name="device-external-synchronization"></a>Внешняя синхронизация устройства

См. раздел [Настройка внешней синхронизации](https://support.microsoft.com/help/4494429/sync-multiple-azure-kinect-dk-devices) для настройки оборудования.

Программное обеспечение для каждого подключенного устройства должно быть настроено для функционирования в **основном** или **подчиненном** режиме. Этот параметр настраивается на [k4a_device_configuration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

При использовании внешней синхронизации подчиненные камеры всегда должны быть запущены до того, как Главная метка времени будет правильно согласована.

### <a name="subordinate-mode"></a>Подчиненный режим

```C
k4a_device_configuration_t deviceConfig;
deviceConfig.wired_sync_mode = K4A_WIRED_SYNC_MODE_SUBORDINATE
```

### <a name="master-mode"></a>Главный режим

```C
k4a_device_configuration_t deviceConfig;
deviceConfig.wired_sync_mode = K4A_WIRED_SYNC_MODE_MASTER;
```

### <a name="retrieving-synchronization-jack-state"></a>Получение состояния розетки синхронизации

Для программного получения текущего состояния входных данных синхронизации и гнезд выходных данных синхронизации используйте функцию [k4a_device_get_sync_jack](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0209ac87bfd055163677321b0304e962.html#ga0209ac87bfd055163677321b0304e962) .

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как включить и записать синхронизацию устройств. Вы также можете ознакомиться с использованием 

>[!div class="nextstepaction"]
>[API-интерфейс для записи и воспроизведения пакета SDK для датчика Kinect Azure](record-playback-api.md)

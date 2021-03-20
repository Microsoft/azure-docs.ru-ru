---
title: Получение результатов отслеживания текста в Azure Kinect
description: Узнайте, как получить результаты отслеживания текста с помощью пакета SDK для отслеживания текста Kinect для Azure.
author: qm13
ms.prod: kinect-dk
ms.author: quentinm
ms.reviewer: yijwan
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, Azure, датчик, пакет SDK, тело, отслеживание, совместное соединение
ms.openlocfilehash: 1b62022242144d5db51455a32ac04b67c3e5dd7a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85277037"
---
# <a name="get-body-tracking-results"></a>Получение результатов отслеживания тела

Пакет SDK для отслеживания текста использует объект отслеживания текста для обработки записей Azure Kinect DK и формирует результаты отслеживания текста. Он также поддерживает глобальное состояние инспектора, обработку очередей и выходную очередь. Использование средства "Инспектор текста" состоит из трех этапов.

- Создание средства записи
- Снимки глубины и IR-изображения с помощью Azure Kinect DK
- Помещает запись в очередь и выводят на экран результаты.

## <a name="create-a-tracker"></a>Создание средства записи


Первым шагом при использовании отслеживания текста является создание средства отслеживания и передача его в виде калибровки датчика [k4a_calibration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__calibration__t.html) структуре. Для запроса калибровки датчика можно использовать функцию [k4a_device_get_calibration ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4e43940d8d8db48da266c7a7842c8d78.html#ga4e43940d8d8db48da266c7a7842c8d78) в SDK для датчика Kinect Azure.

```C
k4a_calibration_t sensor_calibration;
if (K4A_RESULT_SUCCEEDED != k4a_device_get_calibration(device, device_config.depth_mode, K4A_COLOR_RESOLUTION_OFF, &sensor_calibration))
{
    printf("Get depth camera calibration failed!\n");
    return 0;
}

k4abt_tracker_t tracker = NULL;
k4abt_tracker_configuration_t tracker_config = K4ABT_TRACKER_CONFIG_DEFAULT;
if (K4A_RESULT_SUCCEEDED != k4abt_tracker_create(&sensor_calibration, tracker_config, &tracker))
{
    printf("Body tracker initialization failed!\n");
    return 0;
}
```

## <a name="capture-depth-and-ir-images"></a>Глубина захвата и образы IR

Запись образа с помощью Azure Kinect DK рассматривается на странице [получения изображений](retrieve-images.md) .

>[!NOTE]
> `K4A_DEPTH_MODE_NFOV_UNBINNED``K4A_DEPTH_MODE_WFOV_2X2BINNED`для лучшей производительности и точности рекомендуется использовать режимы или. Не используйте `K4A_DEPTH_MODE_OFF` `K4A_DEPTH_MODE_PASSIVE_IR` режимы или.

Поддерживаемые режимы Azure Kinect DK описаны в [спецификации оборудования](hardware-specification.md) Azure Kinect dk и [k4a_depth_mode_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___enumerations_ga3507ee60c1ffe1909096e2080dd2a05d.html#ga3507ee60c1ffe1909096e2080dd2a05d) перечисления.

```C
// Capture a depth frame
switch (k4a_device_get_capture(device, &capture, TIMEOUT_IN_MS))
{
case K4A_WAIT_RESULT_SUCCEEDED:
    break;
case K4A_WAIT_RESULT_TIMEOUT:
    printf("Timed out waiting for a capture\n");
    continue;
    break;
case K4A_WAIT_RESULT_FAILED:
    printf("Failed to read a capture\n");
    goto Exit;
}
```

## <a name="enqueue-the-capture-and-pop-the-results"></a>Помещение захвата в очередь и извлечение результатов

Средство отслеживания поддерживает внутренние очереди ввода и вывода для более эффективной асинхронной обработки захватов Azure Kinect DK. Используйте функцию [k4abt_tracker_enqueue_capture ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga093becd9bb4a63f5f4d56f58097a7b1e.html#ga093becd9bb4a63f5f4d56f58097a7b1e) , чтобы добавить новую запись в очередь ввода. Используйте функцию [k4abt_tracker_pop_result ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gaaf446fb1579cbbe0b6af824ee0a7458b.html#gaaf446fb1579cbbe0b6af824ee0a7458b) o вывод результата из очереди вывода. Использование значения времени ожидания зависит от приложения и управляет временем ожидания очереди.

### <a name="real-time-processing"></a>Обработка в режиме реального времени
Используйте этот шаблон для многопоточных приложений, требующих результатов в реальном времени и поддерживающих удаленные кадры. `simple_3d_viewer`Примером обработки в режиме реального времени является пример, расположенный в [GitHub Azure-Kinect-Samples](https://github.com/microsoft/Azure-Kinect-Samples) .

```C
k4a_wait_result_t queue_capture_result = k4abt_tracker_enqueue_capture(tracker, sensor_capture, 0);
k4a_capture_release(sensor_capture); // Remember to release the sensor capture once you finish using it
if (queue_capture_result == K4A_WAIT_RESULT_FAILED)
{
    printf("Error! Adding capture to tracker process queue failed!\n");
    break;
}

k4abt_frame_t body_frame = NULL;
k4a_wait_result_t pop_frame_result = k4abt_tracker_pop_result(tracker, &body_frame, 0);
if (pop_frame_result == K4A_WAIT_RESULT_SUCCEEDED)
{
    // Successfully popped the body tracking result. Start your processing
    ...

    k4abt_frame_release(body_frame); // Remember to release the body frame once you finish using it
}
```

### <a name="synchronous-processing"></a>Синхронная обработка
Используйте этот шаблон для приложений, которые не требуют результатов в режиме реального времени или не могут разместить удаленные кадры.

Пропускная способность обработки может быть ограничена.

Пример `simple_sample.exe` синхронной обработки приведен в [GitHub Azure-Kinect-Samples](https://github.com/microsoft/Azure-Kinect-Samples) .

```C
k4a_wait_result_t queue_capture_result = k4abt_tracker_enqueue_capture(tracker, sensor_capture, K4A_WAIT_INFINITE);
k4a_capture_release(sensor_capture); // Remember to release the sensor capture once you finish using it
if (queue_capture_result != K4A_WAIT_RESULT_SUCCEEDED)
{
    // It should never hit timeout or error when K4A_WAIT_INFINITE is set.
    printf("Error! Adding capture to tracker process queue failed!\n");
    break;
}

k4abt_frame_t body_frame = NULL;
k4a_wait_result_t pop_frame_result = k4abt_tracker_pop_result(tracker, &body_frame, K4A_WAIT_INFINITE);
if (pop_frame_result != K4A_WAIT_RESULT_SUCCEEDED)
{
    // It should never hit timeout or error when K4A_WAIT_INFINITE is set.
    printf("Error! Popping body tracking result failed!\n");
    break;
}
// Successfully popped the body tracking result. Start your processing
...

k4abt_frame_release(body_frame); // Remember to release the body frame once you finish using it
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Получение доступа к данным о корпусе](access-data-body-frame.md)

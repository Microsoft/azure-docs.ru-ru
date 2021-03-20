---
title: Получение данных образа Azure Kinect
description: Узнайте, как получить данные образа Azure Kinect с помощью пакета SDK для датчика Kinect.
author: cdedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, Azure, извлечение, датчик, Камера, пакет SDK, глубина, RGB, изображения, цвет, захват, разрешение, буфер
ms.openlocfilehash: fed5c7340d287d9103ba35f0fd3d80c0fff6e3ec
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87538920"
---
# <a name="retrieve-azure-kinect-image-data"></a>Получение данных образа Azure Kinect

На этой странице приводятся сведения о том, как получить изображения из Kinect Azure. В статье показано, как записывать и получать доступ к изображениям, скоординированным между цветом и глубиной устройства. Чтобы получить доступ к образам, необходимо сначала открыть и настроить устройство, после чего вы сможете записывать образы.
Перед настройкой и записью образа необходимо [найти и открыть устройство](find-then-open-device.md).

Вы также можете ознакомиться с [примером потоковой передачи SDK](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/streaming) , в котором показано, как использовать функции, описанные в этой статье.

Рассматриваются следующие функции:

- [`k4a_device_start_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaad7a85e1e5471810262442fc4a8e217a.html#gaad7a85e1e5471810262442fc4a8e217a)
- [`k4a_device_get_capture()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4dac757a33657f4d3dbf1ae8b21c158a.html#ga4dac757a33657f4d3dbf1ae8b21c158a)
- [`k4a_capture_get_depth_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafa03513da96bf6b8c254fc11a04ee6d6.html#gafa03513da96bf6b8c254fc11a04ee6d6)
- [`k4a_image_get_buffer()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ef070cf4e543cd0f726478af332546e.html#ga2ef070cf4e543cd0f726478af332546e)
- [`k4a_image_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga27c81863b13fafc3934a32935a014e9f.html#ga27c81863b13fafc3934a32935a014e9f)
- [`k4a_capture_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6)
- [`k4a_device_stop_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4fa0e0a011a7105309ad97f081a5d6b8.html#ga4fa0e0a011a7105309ad97f081a5d6b8)
  
## <a name="configure-and-start-the-device"></a>Настройка и запуск устройства

Две камеры, доступные на устройстве Kinect, поддерживают несколько режимов, разрешений и форматов вывода. Полный список см. в [спецификации оборудования](hardware-specification.md)для пакета Azure Kinect Development Kit.

Конфигурация потоковой передачи задается с помощью значений в [`k4a_device_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html) структуре.

```C
k4a_device_configuration_t config = K4A_DEVICE_CONFIG_INIT_DISABLE_ALL;
config.camera_fps = K4A_FRAMES_PER_SECOND_30;
config.color_format = K4A_IMAGE_FORMAT_COLOR_MJPG;
config.color_resolution = K4A_COLOR_RESOLUTION_2160P;
config.depth_mode = K4A_DEPTH_MODE_NFOV_UNBINNED;

if (K4A_RESULT_SUCCEEDED != k4a_device_start_cameras(device, &config))
{
    printf("Failed to start device\n");
    goto Exit;
}
```

После запуска камер они будут продолжать записывать данные, пока [`k4a_device_stop_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4fa0e0a011a7105309ad97f081a5d6b8.html#ga4fa0e0a011a7105309ad97f081a5d6b8) не будет вызван или устройство будет закрыто.

## <a name="stabilization"></a>Стабилизация

При запуске устройств с помощью функции синхронизации нескольких устройств настоятельно рекомендуется использовать фиксированный параметр экспозиции.
При наличии ручного набора выдержки может потребоваться до восьми записей с устройства, прежде чем образы и частота кадров стабилизироваться. При автоматической раскрытии может потребоваться до 20 захватов, прежде чем образы и частота кадров стабилизироваться.

## <a name="get-a-capture-from-the-device"></a>Получение записи с устройства

Образы записываются с устройства согласованным образом. Каждый захваченный образ содержит глубину изображения, изображение для инфракрасной связи, цветное изображение или сочетание изображений.

По умолчанию API будет возвращать запись только после получения всех запрошенных образов в режиме потоковой передачи. Вы можете настроить API так, чтобы он возвращал частичные записи только с глубиной или цветными изображениями, как только они станут доступны, сняв [`synchronized_images_only`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t_a8208974f05d89fc1362c6a0900bdef4d.html#a8208974f05d89fc1362c6a0900bdef4d) параметр [`k4a_device_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html) .

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

После того как API успешно вернул запись, необходимо вызвать, [`k4a_capture_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6) когда вы завершите работу с объектом Capture.

## <a name="get-an-image-from-the-capture"></a>Получение образа из записи

Чтобы получить захваченный образ, вызовите соответствующую функцию для каждого типа изображения. Одно из двух значений:

- [`k4a_capture_get_color_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga683e440b5f22215a2de58d7fa140488c.html#ga683e440b5f22215a2de58d7fa140488c)
- [`k4a_capture_get_depth_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafa03513da96bf6b8c254fc11a04ee6d6.html#gafa03513da96bf6b8c254fc11a04ee6d6)
- [`k4a_capture_get_ir_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga1531c3fa76a7c174b8f2eab24de91794.html#ga1531c3fa76a7c174b8f2eab24de91794)

Необходимо вызвать [`k4a_image_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga27c81863b13fafc3934a32935a014e9f.html#ga27c81863b13fafc3934a32935a014e9f) для любого [`k4a_image_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__image__t.html) маркера, возвращаемого этими функциями, после завершения работы с изображением.

## <a name="access-image-buffers"></a>Доступ к буферам изображений

[`k4a_image_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__image__t.html) имеет множество функций доступа для получения свойств изображения.

Чтобы получить доступ к буферу памяти образа, используйте [k4a_image_get_buffer](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ef070cf4e543cd0f726478af332546e.html#ga2ef070cf4e543cd0f726478af332546e).

В следующем примере показано, как получить доступ к изображению с изображением глубины. Этот же принцип применяется к другим типам изображений. Однако убедитесь, что переменная типа Image заменена правильным типом изображения, например IR или Color.

```C
// Access the depth16 image
k4a_image_t image = k4a_capture_get_depth_image(capture);
if (image != NULL)
{
    printf(" | Depth16 res:%4dx%4d stride:%5d\n",
            k4a_image_get_height_pixels(image),
            k4a_image_get_width_pixels(image),
            k4a_image_get_stride_bytes(image));

    // Release the image
    k4a_image_release(image);
}

// Release the capture
k4a_capture_release(capture);
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как записывать и координировать изображения камер между цветом и глубиной с помощью устройства Azure Kinect. Вы также можете:

>[!div class="nextstepaction"]
>[Получение примеров IMU](retrieve-imu-samples.md)

>[!div class="nextstepaction"]
>[Доступ к микрофонам](access-mics.md)

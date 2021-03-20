---
title: Получение примеров Azure Kinect иму
description: Узнайте, как получить примеры Azure Kinect иму с помощью пакета SDK для Azure Kinect.
author: qm13
ms.author: cedmonds
ms.date: 06/26/2019
ms.prod: kinect-dk
ms.topic: conceptual
keywords: Kinect, Azure, Настройка, глубина, цвет, РБГ, Камера, датчик, пакет SDK, иму, датчик движения, движение, гироскопом, Гиро, акселерометр, кадров/с
ms.openlocfilehash: 649dd5b9de62d43d59e74d53adff1ec7de8dfd32
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276944"
---
# <a name="retrieve-azure-kinect-imu-samples"></a>Получение примеров Azure Kinect иму

Устройство Azure Kinect предоставляет доступ к единицам движения инерции (ИМУС), включая типы акселерометр и гироскопом. Чтобы получить доступ к примерам ИМУС, необходимо сначала открыть и настроить устройство, а затем записать данные иму. Дополнительные сведения см. в разделе [Поиск и открытие устройства](find-then-open-device.md).

Примеры иму создаются с более высокой частотой, чем изображения. Образцы выводятся на узел с более низкой скоростью, чем выборка. При ожидании образца иму несколько примеров часто становятся доступными одновременно.

Подробные сведения о частоте отчетов иму см. в [спецификации оборудования](hardware-specification.md) Azure Kinect DK.

## <a name="configure-and-start-cameras"></a>Настройка и запуск камер

> [!NOTE]
> Датчики иму могут работать только при работе с цветами и (или) камерами глубины. Датчики иму не могут работать отдельно.

Чтобы запустить камеры, используйте [k4a_device_start_cameras ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaad7a85e1e5471810262442fc4a8e217a.html#gaad7a85e1e5471810262442fc4a8e217a).

```C
k4a_device_configuration_t config = K4A_DEVICE_CONFIG_INIT_DISABLE_ALL;
config.camera_fps = K4A_FRAMES_PER_SECOND_30;
config.color_format = K4A_IMAGE_FORMAT_COLOR_MJPG;
config.color_resolution = K4A_COLOR_RESOLUTION_2160P;

if (K4A_RESULT_SUCCEEDED != k4a_device_start_cameras(device, &config))
{
    printf("Failed to start cameras\n");
    goto Exit;
}

if (K4A_RESULT_SUCCEEDED != k4a_device_start_imu(device))
{
    printf("Failed to start imu\n");
    goto Exit;
}
```

## <a name="access-imu-samples"></a>Примеры доступа к иму

 Каждый [k4a_imu_sample_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__imu__sample__t.html#details) содержит запись акселерометр и гироскопом, записываемую практически в одно и то же время.

Примеры иму можно получить в том же потоке, в котором будут записываться образы, или в отдельных потоках.

Чтобы получить образцы иму, как только они станут доступны, можно вызвать метод [`k4a_device_get_imu_sample()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga8e5913b3bb94a453c7143bbd6e399a0e.html#ga8e5913b3bb94a453c7143bbd6e399a0e) в собственном потоке. API также имеет достаточно внутренней очереди, чтобы можно было проверять только выборки после возвращения каждой записи изображения.

Из-за некоторых внутренних постановки примеров иму можно использовать следующий шаблон без удаления данных:

1. Ожидание записи с частотой кадров.
2. Обработка записи.
3. Получение всех примеров иму в очереди.
4. Повторите ожидание следующей записи.

Чтобы получить все текущие образцы иму в очереди, можно вызвать [`k4a_device_get_imu_sample()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga8e5913b3bb94a453c7143bbd6e399a0e.html#ga8e5913b3bb94a453c7143bbd6e399a0e) с `timeout_in_ms` 0 в цикле до тех пор, пока функция не вернет значение `K4A_WAIT_RESULT_TIMEOUT` . `K4A_WAIT_RESULT_TIMEOUT` Указывает, что отсутствуют образцы в очереди, и они не поступили за указанное время ожидания.

## <a name="usage-example"></a>Пример использования

```C
k4a_imu_sample_t imu_sample;

// Capture a imu sample
switch (k4a_device_get_imu_sample(device, &imu_sample, TIMEOUT_IN_MS))
{
case K4A_WAIT_RESULT_SUCCEEDED:
    break;
case K4A_WAIT_RESULT_TIMEOUT:
    printf("Timed out waiting for a imu sample\n");
    continue;
    break;
case K4A_WAIT_RESULT_FAILED:
    printf("Failed to read a imu sample\n");
    goto Exit;
}

// Access the accelerometer readings
if (imu_sample != NULL)
{
    printf(" | Accelerometer temperature:%.2f x:%.4f y:%.4f z: %.4f\n",
            imu_sample.temperature,
            imu_sample.acc_sample.xyz.x,
            imu_sample.acc_sample.xyz.y,
            imu_sample.acc_sample.xyz.z);
}

```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы умеете работать с примерами иму, вы также можете
>[!div class="nextstepaction"]
>[Доступ к входным данным микрофона](access-mics.md)

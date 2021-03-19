---
title: Найдите и откройте устройство Azure Kinect.
description: Узнайте, как найти и открыть устройство Azure Kinect с помощью пакета SDK для Azure Kinect Сенор.
author: cedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, Azure, датчик, пакет SDK, глубина, RGB, устройство, найти, открыть
ms.openlocfilehash: 67fc93b924d5d663bb43098969c54d1975bd5895
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276920"
---
# <a name="find-then-open-the-azure-kinect-device"></a>Найдите и откройте устройство Azure Kinect.

В этой статье описывается, как можно найти, а затем открыть Azure Kinect DK. В этой статье объясняется, как справиться с тем, когда к компьютеру подключено несколько устройств.

Можно также обратиться к [примеру пакета SDK](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/enumerate) , в котором показано, как использовать функции, описанные в этой статье.

Рассматриваются следующие функции:
 * [`k4a_device_get_installed_count()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf7d19df0f73f8e4dfaa21e1b4b719ecc.html#gaf7d19df0f73f8e4dfaa21e1b4b719ecc)
 * [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113)
 * [`k4a_device_get_serialnum()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga798489af207ff1c99f2285ff6b08bc22.html#ga798489af207ff1c99f2285ff6b08bc22)
 * [`k4a_device_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7a3931d9a690b3971caaac83b43f9423.html#ga7a3931d9a690b3971caaac83b43f9423)

## <a name="discover-the-number-of-connected-devices"></a>Определение числа подключенных устройств

Сначала получите число подключенных в настоящее время устройств Azure Kinect с помощью [`k4a_device_get_installed_count()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf7d19df0f73f8e4dfaa21e1b4b719ecc.html#gaf7d19df0f73f8e4dfaa21e1b4b719ecc) .

```C
uint32_t device_count = k4a_device_get_installed_count();

printf("Found %d connected devices:\n", device_count);
```

## <a name="open-a-device"></a>Открытие устройства

Чтобы получить сведения об устройстве или прочитать данные из него, необходимо сначала открыть обработчик для устройства с помощью [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113) .

```C
k4a_device_t device = NULL;

for (uint8_t deviceIndex = 0; deviceIndex < device_count; deviceIndex++)
{
    if (K4A_RESULT_SUCCEEDED != k4a_device_open(deviceIndex, &device))
    {
        printf("%d: Failed to open device\n", deviceIndex);
        continue;
    }

    ...

    k4a_device_close(device);
}
```

`index`Параметр параметра [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113) указывает, какое устройство следует открыть при наличии нескольких подключенных. Если предполагается подключение только одного устройства, можно передать аргумент `K4A_DEVICE_DEFAULT` или значение 0, чтобы указать первое устройство.

В любой момент, когда вы открываете устройство, его необходимо вызвать по завершении работы [`k4a_device_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7a3931d9a690b3971caaac83b43f9423.html#ga7a3931d9a690b3971caaac83b43f9423) с этим маркером. Никакие другие дескрипторы не могут быть открыты на одном устройстве, пока вы не закроете дескриптор.

## <a name="identify-a-specific-device"></a>Указание конкретного устройства

Порядок устройств, перечисленных по индексу, не изменится, пока устройства не будут присоединены или отсоединены. Для обнаружения физического устройства следует использовать серийный номер устройства.

Чтобы прочитать серийный номер с устройства, используйте [`k4a_device_get_serialnum()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga798489af207ff1c99f2285ff6b08bc22.html#ga798489af207ff1c99f2285ff6b08bc22) функцию после открытия маркера.

В этом примере показано, как выделить правильный объем памяти для хранения серийного номера.

```C
char *serial_number = NULL;
size_t serial_number_length = 0;

if (K4A_BUFFER_RESULT_TOO_SMALL != k4a_device_get_serialnum(device, NULL, &serial_number_length))
{
    printf("%d: Failed to get serial number length\n", deviceIndex);
    k4a_device_close(device);
    device = NULL;
    continue;
}

serial_number = malloc(serial_number_length);
if (serial_number == NULL)
{
    printf("%d: Failed to allocate memory for serial number (%zu bytes)\n", deviceIndex, serial_number_length);
    k4a_device_close(device);
    device = NULL;
    continue;
}

if (K4A_BUFFER_RESULT_SUCCEEDED != k4a_device_get_serialnum(device, serial_number, &serial_number_length))
{
    printf("%d: Failed to get serial number\n", deviceIndex);
    free(serial_number);
    serial_number = NULL;
    k4a_device_close(device);
    device = NULL;
    continue;
}

printf("%d: Device \"%s\"\n", deviceIndex, serial_number);
```

## <a name="open-the-default-device"></a>Открыть устройство по умолчанию

В большинстве приложений к одному и тому же компьютеру будет присоединен только один сервер Azure Kinect DK. Если необходимо подключиться только к одному ожидаемому устройству, можно вызвать метод с, [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113) `index` `K4A_DEVICE_DEFAULT` чтобы открыть первое устройство.

```C
k4a_device_t device = NULL;
uint32_t device_count = k4a_device_get_installed_count();

if (device_count != 1)
{
    printf("Unexpected number of devices found (%d)\n", device_count);
    goto Exit;
}

if (K4A_RESULT_SUCCEEDED != k4a_device_open(K4A_DEVICE_DEFAULT, &device))
{
    printf("Failed to open device\n");
    goto Exit;
}
```

## <a name="next-steps"></a>Дальнейшие действия

>[!div class="nextstepaction"]
>[Получение изображений](retrieve-images.md)

>[!div class="nextstepaction"]
>[Получение примеров иму](retrieve-imu-samples.md)


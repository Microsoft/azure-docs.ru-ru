---
title: API воспроизведения Kinect Azure
description: Узнайте, как использовать пакет SDK для датчика Kinect для Azure, чтобы открыть файл записи с помощью API воспроизведения.
author: xthexder
ms.author: jawirth
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, Azure, датчик, пакет SDK, глубина, RGB, запись, воспроизведение, матроска, MKV
ms.openlocfilehash: fe403f314c1df415537d090433f34627eb1249e7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276914"
---
# <a name="the-azure-kinect-playback-api"></a>API воспроизведения Kinect Azure

Пакет SDK для датчика предоставляет API для записи данных устройства в файл матроска (. MKV). В формате контейнера матроска хранятся видеодорожки, примеры иму и калибровка устройств. Записи можно создавать с помощью предоставленной служебной программы командной строки [k4arecorder](record-sensor-streams-file.md) . Записи также можно настраивать и записывать непосредственно с помощью API записи.

Дополнительные сведения об API записи см. в разделе [`k4a_record_create()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gae14f4181e9688e710d1c80b215413831.html#gae14f4181e9688e710d1c80b215413831) .

Дополнительные сведения о спецификациях формата файла матроска см. на странице [Формат записи файлов](record-file-format.md) .

## <a name="use-the-playback-api"></a>Использование API воспроизведения

Файлы записи можно открыть с помощью API воспроизведения. API воспроизведения предоставляет доступ к данным датчика в том же формате, что и остальная часть пакета SDK для датчика.

### <a name="open-a-record-file"></a>Открытие файла записи

В следующем примере мы откроем запись с помощью [`k4a_playback_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gacb254ac941b2ab3c202ca68f4537f368.html#gacb254ac941b2ab3c202ca68f4537f368) , распечатайте длину записи, а затем закроете файл с помощью [`k4a_playback_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga76f415f2076f1c8c544e094a649306ff.html#ga76f415f2076f1c8c544e094a649306ff) .

```C
k4a_playback_t playback_handle = NULL;
if (k4a_playback_open("recording.mkv", &playback_handle) != K4A_RESULT_SUCCEEDED)
{
    printf("Failed to open recording\n");
    return 1;
}

uint64_t recording_length = k4a_playback_get_last_timestamp_usec(playback_handle);
printf("Recording is %lld seconds long\n", recording_length / 1000000);

k4a_playback_close(playback_handle);
```

### <a name="read-captures"></a>Чтение записей

После открытия файла можно приступить к чтению записей из записи. В следующем примере будут считываться все записи в файле.

```C
k4a_capture_t capture = NULL;
k4a_stream_result_t result = K4A_STREAM_RESULT_SUCCEEDED;
while (result == K4A_STREAM_RESULT_SUCCEEDED)
{
    result = k4a_playback_get_next_capture(playback_handle, &capture);
    if (result == K4A_STREAM_RESULT_SUCCEEDED)
    {
        // Process capture here
        k4a_capture_release(capture);
    }
    else if (result == K4A_STREAM_RESULT_EOF)
    {
        // End of file reached
        break;
    }
}
if (result == K4A_STREAM_RESULT_FAILED)
{
    printf("Failed to read entire recording\n");
    return 1;
}
```

### <a name="seek-within-a-recording"></a>Поиск в записи

Достиг конца файла, нам может потребоваться вернуться и прочитать его снова. Этот процесс можно выполнить, прочитав его в обратном направлении [`k4a_playback_get_previous_capture()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga54732e3aa0717e1ca4eb76ee385e878c.html#ga54732e3aa0717e1ca4eb76ee385e878c) , но это может быть очень зависеть от продолжительности записи.
Вместо этого можно использовать [`k4a_playback_seek_timestamp()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaea748994a121543bd77f90417cf428f6.html#gaea748994a121543bd77f90417cf428f6) функцию для перехода к определенной точке файла.

В этом примере мы указываем метки времени в микросекундах для поиска различных точек в файле.

```C
// Seek to the beginning of the file
if (k4a_playback_seek_timestamp(playback_handle, 0, K4A_PLAYBACK_SEEK_BEGIN) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to the end of the file
if (k4a_playback_seek_timestamp(playback_handle, 0, K4A_PLAYBACK_SEEK_END) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to 10 seconds from the start
if (k4a_playback_seek_timestamp(playback_handle, 10 * 1000000, K4A_PLAYBACK_SEEK_BEGIN) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to 10 seconds from the end
if (k4a_playback_seek_timestamp(playback_handle, -10 * 1000000, K4A_PLAYBACK_SEEK_END) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}
```

### <a name="read-tag-information"></a>Чтение сведений о теге

Записи также могут содержать различные метаданные, такие как серийный номер устройства и версии встроенного по. Эти метаданные хранятся в тегах записи, к которым можно получить доступ с помощью [`k4a_playback_get_tag()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga320f966fc89b4ba0d758f787f70d5143.html#ga320f966fc89b4ba0d758f787f70d5143) функции.

```C
// Print the serial number of the device used to record
char serial_number[256];
size_t serial_number_size = 256;
k4a_buffer_result_t buffer_result = k4a_playback_get_tag(playback_handle, "K4A_DEVICE_SERIAL_NUMBER", &serial_number, &serial_number_size);
if (buffer_result == K4A_BUFFER_RESULT_SUCCEEDED)
{
    printf("Device serial number: %s\n", serial_number);
}
else if (buffer_result == K4A_BUFFER_RESULT_TOO_SMALL)
{
    printf("Device serial number too long.\n");
}
else
{
    printf("Tag does not exist. Device serial number was not recorded.\n");
}
```

### <a name="record-tag-list"></a>Список тегов записи

Ниже приведен список всех тегов по умолчанию, которые могут быть добавлены в файл записи. Многие из этих значений доступны как часть [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html) структуры и могут быть считаны с помощью [`k4a_playback_get_record_configuration()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaab54a85c1f1e98d170d009042b449255.html#gaab54a85c1f1e98d170d009042b449255) функции.

Если тег не существует, предполагается, что оно имеет значение по умолчанию.

| Имя тега                     | Значение по умолчанию      | [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html) Полями | Примечания     |
|------------------------------|--------------------|--------------------------------------|----------------------------------------------------------------------------------------------------------------|
| `K4A_COLOR_MODE`             | АВТОНОМ              | `color_format` / `color_resolution`  | Возможные значения: "OFF", "MJPG_1080P", "NV12_720P", "YUY2_720P" и т. д.                                      |
| `K4A_DEPTH_MODE`             | АВТОНОМ              | `depth_mode` / `depth_track_enabled` | Возможные значения: "OFF", "NFOV_UNBINNED", "PASSIVE_IR" и т. д.                                                |
| `K4A_IR_MODE`                | АВТОНОМ              | `depth_mode` / `ir_track_enabled`    | Возможные значения: "Выкл.", "ACTIVE", "PASSIVE"                                                                    |
| `K4A_IMU_MODE`               | АВТОНОМ              | `imu_track_enabled`                  | Возможные значения: "ON", "OFF"                                                                                   |
| `K4A_CALIBRATION_FILE`       | "calibration.jsна" | Н/Д                                  | См. [`k4a_device_get_raw_calibration()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga8c4e46642cee3115aeb0b33e2b43b24f.html#ga8c4e46642cee3115aeb0b33e2b43b24f) |
| `K4A_DEPTH_DELAY_NS`         | "0"                | `depth_delay_off_color_usec`         | Значение, хранящееся в наносекундах, API — микросекунды.                                                        |
| `K4A_WIRED_SYNC_MODE`        | АВТОНОМНОГО       | `wired_sync_mode`                    | Возможные значения: «STANDALONE», «MASTER», «ПОДЧИНЕНный»                                                         |
| `K4A_SUBORDINATE_DELAY_NS`   | "0"                | `subordinate_delay_off_master_usec`  | Значение, хранящееся в наносекундах, API — микросекунды.                                                        |
| `K4A_COLOR_FIRMWARE_VERSION` | ""                 | Н/Д                                  | Версия встроенного по цвета устройства, например "1. x. XX"                                                            |
| `K4A_DEPTH_FIRMWARE_VERSION` | ""                 | Н/Д                                  | Версия встроенного по глубины устройства, например "1. x. XX"                                                            |
| `K4A_DEVICE_SERIAL_NUMBER`   | ""                 | Н/Д                                  | Запись серийного номера устройства                                                                                 |
| `K4A_START_OFFSET_NS`        | "0"                | `start_timestamp_offset_usec`        | См. сведения о [синхронизации меток времени](record-playback-api.md#timestamp-synchronization) ниже.                       |
| `K4A_COLOR_TRACK`            | Нет               | Н/Д                                  | См. раздел [запись формата файла — определение дорожек](record-file-format.md#identifying-tracks).                     |
| `K4A_DEPTH_TRACK`            | Нет               | Н/Д                                  | См. раздел [запись формата файла — определение дорожек](record-file-format.md#identifying-tracks).                     |
| `K4A_IR_TRACK`               | Нет               | Н/Д                                  | См. раздел [запись формата файла — определение дорожек](record-file-format.md#identifying-tracks).                     |
| `K4A_IMU_TRACK`              | Нет               | Н/Д                                  | См. раздел [запись формата файла — определение дорожек](record-file-format.md#identifying-tracks).                     |

## <a name="timestamp-synchronization"></a>Синхронизация меток времени

Для формата матроска необходимо, чтобы записи начинались с нулевой метки времени. При [внешней синхронизации камер](record-external-synchronized-units.md)Первая метка времени каждого устройства может быть не равна нулю.

Чтобы сохранить исходные метки времени с устройств между записью и воспроизведением, в файле хранится смещение, применяемое к отметкам времени.

`K4A_START_OFFSET_NS`Тег используется для указания смещения отметки времени, чтобы файлы можно было повторно синхронизировать после записи. Это смещение отметки времени может быть добавлено к каждой метке времени в файле для восстановления исходных меток времени для устройств.

Начальное смещение также доступно в [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html) структуре.

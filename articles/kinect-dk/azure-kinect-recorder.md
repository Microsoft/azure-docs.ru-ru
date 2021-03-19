---
title: Средство записи Azure Kinect DK
description: Узнайте, как записывать потоки данных из пакета SDK для датчика в файл с помощью средства записи Azure Kinect.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, запись, воспроизведение, читатель, матроска, MKV, потоки, глубина, RGB, Камера, цвет, иму, аудио
ms.openlocfilehash: 5fb6895d4102956a991c67ffab836a26b7a3abb0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276962"
---
# <a name="azure-kinect-dk-recorder"></a>Средство записи Azure Kinect DK

В этой статье описывается, как можно использовать `k4arecorder` программу командной строки для записи потоков данных из пакета SDK для датчика в файл.

>[!NOTE]
>Средство записи Azure Kinect не записывает звук.

## <a name="recorder-options"></a>Параметры средства записи

`k4arecorder`Имеет различные аргументы командной строки для указания выходного файла и режимов записи.

Записи хранятся в [формате матроска. MKV](record-file-format.md). В записи используется несколько видеодорожек для цвета и глубины, а также дополнительные сведения, такие как калибровка и метаданные камеры.

```console
k4arecorder [options] output.mkv

 Options:
  -h, --help              Prints this help
  --list                  List the currently connected K4A devices
  --device                Specify the device index to use (default: 0)
  -l, --record-length     Limit the recording to N seconds (default: infinite)
  -c, --color-mode        Set the color sensor mode (default: 1080p), Available options:
                            3072p, 2160p, 1536p, 1440p, 1080p, 720p, 720p_NV12, 720p_YUY2, OFF
  -d, --depth-mode        Set the depth sensor mode (default: NFOV_UNBINNED), Available options:
                            NFOV_2X2BINNED, NFOV_UNBINNED, WFOV_2X2BINNED, WFOV_UNBINNED, PASSIVE_IR, OFF
  --depth-delay           Set the time offset between color and depth frames in microseconds (default: 0)
                            A negative value means depth frames will arrive before color frames.
                            The delay must be less than 1 frame period.
  -r, --rate              Set the camera frame rate in Frames per Second
                            Default is the maximum rate supported by the camera modes.
                            Available options: 30, 15, 5
  --imu                   Set the IMU recording mode (ON, OFF, default: ON)
  --external-sync         Set the external sync mode (Master, Subordinate, Standalone default: Standalone)
  --sync-delay            Set the external sync delay off the master camera in microseconds (default: 0)
                            This setting is only valid if the camera is in Subordinate mode.
  -e, --exposure-control  Set manual exposure value (-11 to 1) for the RGB camera (default: auto exposure)
```

## <a name="record-files"></a>Файлы записей

Пример 1. Режим записи глубины НФОВ унбиннед (640x576), RGB 1080p с частотой 30 кадров/с и иму.
Нажмите клавиши **CTRL + C** , чтобы прерывать запись.

```
k4arecorder.exe output.mkv
```

Пример 2. Запись ВФОВ non-Binned (1MP), RGB 3072p с частотой 15 кадров в секунду без иму, в течение 10 секунд.

```
k4arecorder.exe -d WFOV_UNBINNED -c 3072p -r 15 -l 10 --imu OFF output.mkv
```

Пример 3. Запишите ВФОВ 2x2 Binned в 30 кадров/с в течение 5 секунд и сохраните в Output. MKV.

```
k4arecorder.exe -d WFOV_2X2BINNED -c OFF --imu OFF -l 5 output.mkv
```

>[!TIP]
>Вы можете использовать [средство просмотра Kinect Azure](azure-kinect-viewer.md) для настройки элементов управления камеры RGB перед записью (например, для задания баланса белого цвета вручную).

## <a name="verify-recording"></a>Проверка записи

Файл Output. MKV можно открыть с помощью [средства просмотра Kinect Azure](azure-kinect-viewer.md).

Для извлечения дорожек или просмотра сведений о файле средства, такие как, `mkvinfo` доступны в составе набора средств [мквтулникс](https://mkvtoolnix.download/) .

## <a name="next-steps"></a>Дальнейшие действия

[Использование средства записи с внешними синхронизированными единицами](record-external-synchronized-units.md)
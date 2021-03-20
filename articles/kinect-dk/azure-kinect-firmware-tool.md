---
title: Средство обновления встроенного ПО Azure Kinect
description: Узнайте, как запрашивать и обновлять встроенное по устройства с помощью средства встроенного по Azure Kinect.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, встроенное по, обновление
ms.openlocfilehash: f8b89751d27fb5a4b18d635f45f63f4f36bd05f6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276974"
---
# <a name="azure-kinect-dk-firmware-tool"></a>Средство встроенного по Azure Kinect DK

Средство встроенного по Azure Kinect можно использовать для запроса и обновления встроенного по устройства в Azure Kinect DK.

## <a name="list-connected-devices"></a>Перечисление подключенных устройств

Список подключенных устройств можно получить с помощью параметра-l.  `AzureKinectFirmwareTool.exe -l`

```console
 == Azure Kinect DK Firmware Tool ==
Found 2 connected devices:
0: Device "000036590812"
1: Device "000274185112"
```

## <a name="check-device-firmware-version"></a>Проверка версии встроенного по устройства

Вы можете проверить текущие версии встроенного по первого подключенного устройства с помощью параметра-q, например `AzureKinectFirmwareTool.exe -q` .

```console
 == Azure Kinect DK Firmware Tool ==
Device Serial Number: 000036590812
Current Firmware Versions:
    RGB camera firmware:      1.5.92
    Depth camera firmware:    1.5.66
    Depth config file:        6109.7
    Audio firmware:           1.5.14
    Build Config:             Production
    Certificate Type:         Microsoft
```

Если подключено несколько устройств, можно указать, к какому устройству нужно выполнить запрос, добавив полный серийный номер в команду, например:

`AzureKinectFirmwareTool.exe -q 000036590812`

## <a name="update-device-firmware"></a>Обновление встроенного ПО устройства

Чаще всего это средство используется для обновления встроенного по устройства. Выполните обновление, вызвав средство с помощью `-u` параметра. Обновление встроенного по может занять несколько минут в зависимости от того, какие файлы микропрограмм необходимо обновить.

Пошаговые инструкции по обновлению встроенного по см. в статье [Обновление встроенного по Kinect для Azure](update-device-firmware.md).  

`AzureKinectFirmwareTool.exe -u firmware\AzureKinectDK_Fw_1.5.926614.bin`

Если подключено несколько устройств, можно указать, к какому устройству нужно выполнить запрос, добавив полный серийный номер в команду.

`AzureKinectFirmwareTool.exe -u firmware\AzureKinectDK_Fw_1.5.926614.bin 000036590812`

## <a name="reset-device"></a>Сброс устройства

Подключенный экземпляр Azure Kinect DK можно сбросить с помощью параметра-r, если необходимо вернуть устройство в известное состояние.

Если подключено несколько устройств, можно указать, к какому устройству нужно выполнить запрос, добавив полный серийный номер в команду.

`AzureKinectFirmwareTool.exe -r 000036590812`

## <a name="inspect-firmware"></a>Проверка встроенного по

Проверка встроенного по позволяет получить сведения о версии из файла BIN встроенного по перед обновлением фактического устройства.

`AzureKinectFirmwareTool.exe -i firmware\AzureKinectDK_Fw_1.5.926614.bin`

```console
 == Azure Kinect DK Firmware Tool ==
Loading firmware package ..\tools\updater\firmware\AzureKinectDK_Fw_1.5.926614.bin.
File size: 1228844 bytes
This package contains:
  RGB camera firmware:      1.5.92
  Depth camera firmware:    1.5.66
  Depth config files: 6109.7 5006.27
  Audio firmware:           1.5.14
  Build Config:             Production
  Certificate Type:         Microsoft
  Signature Type:           Microsoft
```

## <a name="firmware-update-tool-options"></a>Параметры средства обновления встроенного по

```console
 == Azure Kinect DK Firmware Tool ==
* Usage Info *
    AzureKinectFirmwareTool.exe <Command> <Arguments>

Commands:
    List Devices: -List, -l
    Query Device: -Query, -q
        Arguments: [Serial Number]
    Update Device: -Update, -u
        Arguments: <Firmware Package Path and FileName> [Serial Number]
    Reset Device: -Reset, -r
        Arguments: [Serial Number]
    Inspect Firmware: -Inspect, -i
        Arguments: <Firmware Package Path and FileName>

    If no Serial Number is provided, the tool will just connect to the first device.

Examples:
    AzureKinectFirmwareTool.exe -List
    AzureKinectFirmwareTool.exe -Update c:\data\firmware.bin 0123456
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Пошаговые инструкции по обновлению встроенного по устройства](update-device-firmware.md)

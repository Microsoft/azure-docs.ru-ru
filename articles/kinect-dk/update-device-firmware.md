---
title: Обновление встроенного по Azure Kinect DK
description: Узнайте, как обновить встроенное по устройства Azure Kinect DK с помощью средства встроенного по Azure Kinect.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, встроенное по, обновление, восстановление
ms.openlocfilehash: 71557798a584635356cc47fd7c4a0309df4d018a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94356232"
---
# <a name="update-azure-kinect-dk-firmware"></a>Обновление встроенного по Azure Kinect DK

В этом документе содержатся рекомендации по обновлению встроенного по устройства в Azure Kinect DK.

Azure Kinect DK не обновляет встроенное по автоматически. Вы можете использовать [средство встроенного по Azure Kinect](azure-kinect-firmware-tool.md) для обновления микропрограммы вручную до последней доступной версии.

## <a name="prepare-for-firmware-update"></a>Подготовка к обновлению встроенного по

1. [Скачать пакет SDK](sensor-sdk-download.md).
2. Установка пакета SDK.
3. В расположении установки пакета SDK в разделе (расположение установки пакета SDK) \тулс\ необходимо найти следующее:

    - AzureKinectFirmwareTool.exe
    - Файл Firmware. bin в папке встроенного по, например *AzureKinectDK_Fw_1.5.926614. bin*.

4. Подключите устройство к главному компьютеру и включите его.

> [!IMPORTANT]
> Обеспечьте подключение USB и источника питания во время обновления встроенного по. Удаление любого подключения во время обновления может привести к повреждению состояния встроенного по.

## <a name="update-device-firmware"></a>Обновление встроенного ПО устройства

1. Откройте командную строку в папке \тулс\ (расположение установки пакета SDK).
2. Обновление встроенного по с помощью средства встроенного по Azure Kinect

    `AzureKinectFirmwareTool.exe -u <device_firmware_file.bin>`

    Пример

    `AzureKinectFirmwareTool.exe -u firmware\AzureKinectDK_Fw_1.5.926614.bin`

3. Дождитесь завершения обновления встроенного по. Это может занять несколько минут в зависимости от размера образа.

### <a name="verify-device-firmware-version"></a>Проверка версии встроенного по устройства

1. Убедитесь, что встроенное по Обновлено.

    `AzureKinectFirmwareTool.exe -q`

2. Просмотрите следующий пример.

    ```console
       >AzureKinectFirmwareTool.exe -q
       == Azure Kinect DK Firmware Tool ==
        Device Serial Number: 000805192412
        Current Firmware Versions:
        RGB camera firmware:      1.6.102
        Depth camera firmware:    1.6.75
        Depth config file:        6109.7
        Audio firmware:           1.6.14
        Build Config:             Production
        Certificate Type:         Microsoft
    ```

3. Если вы видите приведенный выше результат, встроенное по обновится.

4. После обновления встроенного по можно запустить [средство просмотра Kinect для Azure](azure-kinect-viewer.md) , чтобы убедиться, что все датчики работают должным образом.

## <a name="troubleshooting"></a>Устранение неполадок

Обновление встроенного по может завершиться сбоем по нескольким причинам. При сбое обновления встроенного по попробуйте выполнить следующие действия по устранению.

1. Попробуйте выполнить команду обновления встроенного по еще раз.

2. Убедитесь, что устройство по-прежнему подключено, запрашивая версию встроенного по.        AzureKinectFirmareTool.exe

3. Если все остальное не удается, выполните шаги по [восстановлению](https://support.microsoft.com/help/4494277/reset-azure-kinect-dk) , чтобы вернуться к микропрограмме фабрики, и повторите попытку.

Дополнительные сведения о дополнительных проблемах см. на [странице службы поддержки Майкрософт](./index.yml) .

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Средство встроенного по Azure Kinect](azure-kinect-firmware-tool.md)
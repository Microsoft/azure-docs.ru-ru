---
title: Скачивание пакета SDK для датчика Azure Kinect
description: Узнайте, как скачать и установить пакет SDK для датчика Azure Kinect в Windows и Linux.
author: Brent-A
ms.author: brenta
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: azure, kinect, пакет sdk, скачать обновление, последняя версия, доступно, установка
ms.openlocfilehash: 591fcba4c887e298cf667c5d95c19184bc213ffe
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102179635"
---
# <a name="azure-kinect-sensor-sdk-download"></a>Скачивание пакета SDK для датчика Azure Kinect

На этой странице содержатся ссылки на скачивание каждой версии пакета SDK для датчика Azure Kinect. Установщик предоставляет все необходимые файлы для разработки для Azure Kinect.

## <a name="azure-kinect-sensor-sdk-contents"></a>Содержимое пакета SDK для датчика Azure Kinect

- Заголовки и библиотеки для создания приложения с помощью Azure Kinect DK.
- Распространяемые библиотеки DLL, которые требуются для приложений, использующих Azure Kinect DK.
- [Средство просмотра Azure Kinect.](azure-kinect-viewer.md)
- [Средство записи Azure Kinect.](azure-kinect-recorder.md)
- [Средство обновления встроенного ПО Azure Kinect.](azure-kinect-firmware-tool.md)

## <a name="windows-installation-instructions"></a>Инструкции по установке Windows

Сведения об установке для последних и предыдущих версий пакета SDK и встроенного по для Azure Kinect можно найти [здесь](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md).

Исходный код можно найти [здесь](https://github.com/microsoft/Azure-Kinect-Sensor-SDK).

> [!NOTE]
> При установке пакета SDK запомните путь, по которому выполнена установка. Например, C:\Program Files\Azure Kinect SDK 1.2. Средства, упоминаемые в статьях, находятся в этом каталоге.

## <a name="linux-installation-instructions"></a>Инструкции по установке (Linux)

Сейчас единственным поддерживаемым дистрибутивом является Ubuntu 18.04. Чтобы запросить поддержку других дистрибутивов, перейдите на [эту страницу](https://aka.ms/azurekinectfeedback).

Сначала необходимо настроить [репозиторий пакетов Майкрософт](https://packages.microsoft.com/), следуя инструкциям, приведенным [здесь](/windows-server/administration/linux-package-repository-for-microsoft-software).

Теперь можно установить необходимые пакеты. Пакет `k4a-tools` включает [средство просмотра Azure Kinect](azure-kinect-viewer.md), [средство записи Azure Kinect](record-sensor-streams-file.md), а также [средство обновления программного обеспечения Azure Kinect](azure-kinect-firmware-tool.md). Чтобы установить пакет, выполните:

`sudo apt install k4a-tools`
 
Эта команда устанавливает пакеты зависимостей, необходимые для правильной работы средств, включая последнюю версию `libk4a<major>.<minor>` . Вам потребуется добавить правила udev для доступа к Azure Kinect DK без привилегированного пользователя. Инструкции см. в разделе [Настройка устройств Linux](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md#linux-device-setup). В качестве альтернативы можно запускать приложения, использующие устройство в качестве корневого.

`libk4a<major>.<minor>-dev`Пакет содержит заголовки и файлы CMAK для сборки приложений и исполняемых файлов `libk4a` .

`libk4a<major>.<minor>`Пакет содержит общие объекты, необходимые для запуска приложений и исполняемых файлов, которые зависят от `libk4a` .

Для работы с основными руководствами требуется пакет `libk4a<major>.<minor>-dev`. Чтобы установить пакет, выполните:

`sudo apt install libk4a<major>.<minor>-dev` 

Если команда выполнена, это значит, что пакет SDK готов к использованию.

Обязательно установите соответствующую версию `libk4a<major>.<minor>` с помощью `libk4a<major>.<minor>-dev` . Например, если установить `libk4a4.1-dev` пакет, установите соответствующий `libk4a4.1` пакет, содержащий соответствующую версию общих объектных файлов. Последние версии см `libk4a` . по ссылкам в следующем разделе.

## <a name="change-log-and-older-versions"></a>Журнал изменений и более старые версии

Журнал изменений пакета SDK для датчика Azure Kinect см. [здесь](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/CHANGELOG.md).

Более старую версию пакета SDK для датчика Azure Kinect см. [здесь](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md).

## <a name="next-steps"></a>Дальнейшие действия

[Настройка Azure Kinect DK](set-up-azure-kinect-dk.md)

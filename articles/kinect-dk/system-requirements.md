---
title: Требования к системе пакета SDK для датчика Kinect для Azure
description: Ознакомьтесь с требованиями к системе для пакета SDK для датчика Kinect для Azure в Windows и Linux.
author: qm13
ms.author: quentinm
ms.custom:
- CI 115266
- CSSTroubleshooting
manager: dcscontentpm
ms.prod: kinect-dk
ms.date: 03/05/2021
ms.topic: article
keywords: Azure, Kinect, требования к системе, ЦП, GPU, USB, Настройка, Настройка, минимальные требования
ms.openlocfilehash: 558c1b9ca264874fa808aeba5f1e8a809a8f722d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104656980"
---
# <a name="azure-kinect-sensor-sdk-system-requirements"></a>Требования к системе пакета SDK для датчика Kinect для Azure

В этом документе приводятся сведения о требованиях к системе, необходимых для установки пакета SDK для датчика и успешного развертывания Azure Kinect DK.

## <a name="supported-operating-systems-and-architectures"></a>Поддерживаемые операционные системы и архитектуры

- Windows 10 апреля 2018 (версия 1803, сборка ОС 17134) выпуск (x64) или более поздняя версия
- Linux Ubuntu 18,04 (x64) с драйвером GPU, использующим Опенглв 4.4 или более поздней версии;

Пакет SDK для датчика доступен для Windows API (Win32) для приложений Windows в машинном коде C/C++. В настоящее время пакет SDK недоступен для приложений UWP. Azure Kinect DK не поддерживается для Windows 10 в режиме S.

## <a name="development-environment-requirements"></a>Требования к среде разработки

Чтобы внести изменения в разработку пакета SDK для датчика, посетите сайт [GitHub](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK).

## <a name="minimum-host-pc-hardware-requirements"></a>Минимальные требования к оборудованию для главного компьютера

Требования к оборудованию для узла ПК зависят от частоты и разрешения кадров, выполняемых на хост-компьютере. Рекомендуемая конфигурация пакета SDK для минимального датчика для Windows:

- Седьмой &reg; процессор Intel коретм i3 (два ядра с частотой 2,4 ГГц с HD620 GPU или быстрее)
- 4 ГБ памяти
- Выделенный порт USB3
- Поддержка графического драйвера для OpenGL 4,4 или DirectX 11,0

Более низкие или более старые процессоры могут также работать в зависимости от вашего варианта использования.

Кроме того, производительность отличается для операционных систем Windows и Linux и используемых графических драйверов.

## <a name="body-tracking-host-pc-hardware-requirements"></a>Основные требования к оборудованию для отслеживания текста

Требования к узлу для отслеживания текста являются более жесткими по сравнению с требованиями к узлу общего компьютера. Рекомендуемая минимальная конфигурация пакета SDK для отслеживания текста для Windows:

- Седьмой &reg; процессор Intel коретм i5 (четыре ядра с частотой 2,4 ГГц или выше)
- 4 ГБ памяти
- NVIDIA GEFORCE ГТКС 1050 или аналог
- Выделенный порт USB3

Рекомендуемая минимальная конфигурация предполагает K4A_DEPTH_MODE_NFOV_UNBINNED режим глубины в 30fps отслеживания 5 пользователей. Более низкие или более старые процессоры и видеопроцессоры NVIDIA могут также работать в зависимости от вашего варианта использования.

## <a name="usb3"></a>USB3

Существуют известные проблемы совместимости с контроллерами USB хоста. Дополнительные сведения см. на [странице Устранение неполадок](troubleshooting.md#usb3-host-controller-compatibility)

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор Azure Kinect DK](about-azure-kinect-dk.md)

- [Настройка Azure Kinect DK](set-up-azure-kinect-dk.md)

- [Настройка отслеживания текста Azure Kinect](body-sdk-setup.md)

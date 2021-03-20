---
title: Сведения о пакете SDK для датчика Kinect для Azure
description: Общие сведения о пакете средств разработки программного обеспечения (SDK) для Azure Kinect, его функциях и средствах.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: article
keywords: Azure, Kinect, RGB, IR, запись, датчик, пакет SDK, доступ, глубина, видео, Камера, иму, движение, датчик, аудио, микрофон, матроска, пакет SDK для датчика, Загрузка
ms.openlocfilehash: 17c1b33120eacb5d0c6d3c02e692d1488ef474e6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276986"
---
# <a name="about-azure-kinect-sensor-sdk"></a>Сведения о пакете SDK для датчика Kinect для Azure

В этой статье содержатся общие сведения о пакете средств разработки программного обеспечения (SDK) для Azure Kinect, его функциях и средствах.

## <a name="features"></a>Компоненты

Пакет SDK для датчика Kinect для Azure предоставляет межплатформенный доступ низкого уровня для конфигурации устройств Azure Kinect и потоков датчиков оборудования, в том числе:

- доступ к камере глубины и управление режимом (пассивный ИК-режим, режимы глубины с широким и узким полем зрения); 
- доступ к RGB-камере и управление ее параметрами (например, экспозицией и балансом белого); 
- доступ к датчику движения (гироскоп и акселерометр); 
- синхронизированная потоковая передача данных с камер RGB и глубины с настраиваемой задержкой между ними; 
- управление синхронизацией внешнего устройства с настраиваемой задержкой между устройствами; 
- доступ к метаданным кадров камеры (разрешение изображения, метки времени и т. д.); 
- доступ к данным калибровки устройства. 

## <a name="tools"></a>Инструменты

- [Средство просмотра Kinect Azure](azure-kinect-viewer.md) для отслеживания потоков данных устройства и настройки различных режимов.
- API средства записи и воспроизведения [Azure Kinect](azure-kinect-recorder.md) , который использует [Формат контейнера матроска](record-file-format.md).
- [Средство обновления встроенного по](azure-kinect-firmware-tool.md)Azure Kinect DK.

## <a name="sensor-sdk"></a>Пакет SDK для датчика

- [Скачать пакет SDK для датчика](sensor-sdk-download.md).
- Пакет SDK для датчика доступен на сайте [GitHub с открытым кодом](https://github.com/microsoft/Azure-Kinect-Sensor-SDK).
- Дополнительные сведения об использовании см. в [документации по API-интерфейсу Sensor SDK](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/index.html).

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали о пакете SDK для датчика Kinect для Azure, а также:
>[!div class="nextstepaction"]
>[Скачать код пакета SDK для датчика](sensor-sdk-download.md)

>[!div class="nextstepaction"]
>[Поиск и открытие устройства](find-then-open-device.md)

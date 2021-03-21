---
title: Получение пакета SDK для устройств, подключаемых к службе "Речь"
titleSuffix: Azure Cognitive Services
description: Служба распознавания речи работает с самыми разнообразными устройствами и звуковыми источниками. Теперь ваши речевые приложения могут стать еще эффективнее благодаря соответствующему оборудованию и программному обеспечению. В этой статье вы узнаете, как получить доступ к пакету SDK для речевых устройств и приступить к разработке.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/14/2019
ms.author: erhopf
ms.openlocfilehash: 69898e64934c363a18a3ce2fa5e678116624bd24
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026375"
---
# <a name="get-the-cognitive-services-speech-devices-sdk"></a>Получение пакета SDK для речевых устройств в Cognitive Services

Пакет SDK для речевых устройств — это предварительно настроенная Библиотека, предназначенная для работы с готовыми комплектами разработки и различными конфигурациями массивов микрофона.

## <a name="choose-a-development-kit"></a>Выбор набора средств разработки

|Устройства|Спецификация|Описание|Сценарии|
|--|--|--|--|
|[Урбеттер Dev Kit](http://www.urbetter.com/products_56/278.html) ![ Урбеттер DDK](media/speech-devices-sdk/device-urbetter.jpg)|7. массив MIC, ARM SOC, WIFI, Ethernet, HDMI, USB-камера. <br>Linux|Пакет SDK для речевых устройств, который адаптирует микрофонный массив Майкрософт и поддерживает расширенные операции ввода-вывода, такие как HDMI/Ethernet и другие периферийные устройства USB. <br> [Обратитесь в Урбеттер](http://www.urbetter.com/products_56/278.html)|Транскрипция беседы, образование, больницы, роботы, OTT Box, агент передачи голоса, диск|
|[Рубо Smart Audio Dev Kit](http://ddk.roobo.com)<br>[Программа установки](speech-devices-sdk-roobo-v1.md)  /  [Краткое руководство](./speech-devices-sdk-quickstart.md?pivots=platform-android%253fpivots%253dplatform-android) ![ Рубо Smart Audio Dev Kit](media/speech-devices-sdk/device-roobo-v1.jpg)|7. массив MIC, ARM SOC, WIFI, выход из аудиоподсистемы, IO. <br>[Android](./speech-devices-sdk-quickstart.md?pivots=platform-android%253fpivots%253dplatform-android)|Первый пакет SDK для речевых устройств, предназначенный для адаптации Microsoft MIC Array и Front Processing SDK, предназначен для разработки речевых сценариев и речи с высоким качеством.|Запись беседы, смарт-динамик, агент передачи голоса, носимого пользователем|
|[Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/)<br>[Программа установки](../../kinect-dk/set-up-azure-kinect-dk.md)  /  [Краткое руководство](./speech-devices-sdk-quickstart.md?pivots=platform-windows%253fpivots%253dplatform-windows) ![ Azure Kinect DK](media/speech-devices-sdk/device-azure-kinect-dk.jpg)|7 видеокамер и глубиной цвета массива MIC. <br>[Windows](./speech-devices-sdk-quickstart.md?pivots=platform-windows%253fpivots%253dplatform-windows) / [Linux](./speech-devices-sdk-quickstart.md?pivots=platform-linux%253fpivots%253dplatform-linux)|Комплект разработчика с усовершенствованными датчиками искусственного интеллекта (ии) для создания сложных моделей компьютерных концепций и речи. Он сочетает в себе высококлассный пространственный массив и фотокамеру с видеокамерой и датчиком ориентации — всего на одном маленьком устройстве с несколькими режимами, вариантами и пакетами SDK для размещения различных типов вычислений.|Транскрипция беседы, Robotics, интеллектуальное создание|
|Рубо Smart Audio Dev Kit 2<br>[Установка](speech-devices-sdk-roobo-v2.md)<br>![Рубо Smart Audio Dev Kit 2](media/speech-devices-sdk/device-roobo-v2.jpg)|7. массив MIC, ARM SOC, WIFI, Bluetooth, IO. <br>Linux|Пакет SDK для речевых устройств второго поколения, предоставляющий альтернативную ОС и дополнительные функции в экономичной справочной структуре.|Запись беседы, смарт-динамик, агент передачи голоса, носимого пользователем|


## <a name="download-the-speech-devices-sdk"></a>Загрузка пакета SDK для речевых устройств

Скачайте [пакет SDK для речевых устройств](./speech-devices-sdk.md).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Начало работы с пакетом SDK для речевых устройств](./speech-devices-sdk-quickstart.md?pivots=platform-android)
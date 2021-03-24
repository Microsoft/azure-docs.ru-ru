---
title: Лицензирование &reg; пакета Microsoft Smooth Streaming Client Licensing Kit
description: Сведения о лицензировании &reg; пакета Microsoft Smooth Streaming clienting Kit.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: e3b488e7-8428-4c10-a072-eb3af46c82ad
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: xpouyat
ms.openlocfilehash: f2c66b08d399ddf53686074079665e36be9aace7
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870476"
---
# <a name="licensing-microsoftreg-smooth-streaming-client-porting-kit"></a>Лицензирование &reg; пакета Microsoft Smooth Streaming Client Licensing Kit

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
## <a name="overview"></a>Обзор
Пакет для переноса клиента бесперебойной потоковой передачи Microsoft Smooth Streaming (далее для краткости называется **SSPK**) представляет собой реализацию клиента Smooth Streaming. Он оптимизирован так, чтобы помочь производителям встроенных устройств, операторам кабельной и мобильной связи, поставщикам служб содержимого, производителям мобильных устройств, независимым поставщикам программного обеспечения и поставщикам решений создавать продукты и услуги, которые поддерживают потоковую передачу адаптивного содержимого в формате Smooth Streaming. Клиент SSPK не зависит от оборудования и платформы. Лицензиат может легко перенести его на любое устройство и любую платформу. 

Ниже вы видите схему архитектуры высокого уровня. Поле IIS Smooth Streaming Porting Kit здесь обозначает клиента Smooth Streaming, который предоставляет корпорация Майкрософт. Этот клиент содержит всю основную логику для воспроизведения контента Smooth Streaming. Затем партнеры переносят содержимое на конкретное устройство или платформу, создавая соответствующие интерфейсы. 

![SSPK](./media/media-services-sspk/sspk-arch.png)

## <a name="description"></a>Описание
Условия лицензирования SSPK повышают коммерческую ценность этого предложения. Лицензия SSPK предоставляет предприятиям отрасли следующие возможности.

* Исходный код пакета для портирования Smooth Streaming на языке C++, в котором: 
  * реализованы функции клиента Smooth Streaming;
  * добавлены возможности синтаксического и эвристического анализа, логика буферизации и другие функции.
* API-интерфейсы приложения проигрывателя. 
  * Программные интерфейсы для взаимодействия с приложением Проигрывателя мультимедиа.
* Интерфейс слоя платформенных абстракций (PAL). 
  * Программные интерфейсы для взаимодействия с операционной системой (потоки, сокеты).
* Интерфейс слоя аппаратных абстракций (HAL). 
  * Программные интерфейсы для взаимодействия с аппаратным оборудованием аудио- и видеодекодеров (декодирование, отрисовка).
* Интерфейс управления цифровыми правами (DRM). 
  * Программные интерфейсы для обработки ограничений DRM через уровень абстракции DRM (DAL).
  * Пакет для портирования Microsoft PlayReady поставляется отдельно, но интегрируется через этот интерфейс. [См. Дополнительные сведения о лицензировании устройств Microsoft PlayReady](https://www.microsoft.com/playready/licensing/device_technology.mspx#pddipdl).
* Примеры реализации 
  * Пример реализации PAL для Linux
  * Пример реализации HAL для GStreamer

## <a name="licensing-options"></a>Варианты лицензирования
Пакет для портирования клиента Microsoft Smooth Streaming предлагается лицензиатам на условиях одного из двух различных лицензионных соглашений. Одно соглашение предназначено для разработки промежуточных продуктов Smooth Streaming, а другое — для распространения конечных клиентских продуктов Smooth Streaming среди конечных пользователей.

* Изготовители микросхем, системные интеграторы и независимые поставщики программного обеспечения, которые используют исходный код пакета для разработки промежуточных продуктов, должны выполнять условия **лицензии на промежуточный продукт** пакета для портирования клиента Microsoft Smooth Streaming.
* Производители устройств и независимые поставщики программного обеспечения, которым нужны права на передачу конечных продуктов конечным пользователям, должны выполнять условия **лицензии на конечный продукт** пакета портирования клиента Microsoft Smooth Streaming.

### <a name="microsoft-smooth-streaming-client-porting-kit-interim-product-license"></a>Лицензия на промежуточный продукт пакета портирования клиента Microsoft Smooth Streaming.
По этой лицензии корпорация Майкрософт предоставляет пакет портирования клиента Smooth Streaming и необходимые права на интеллектуальную собственность для разработки промежуточных клиентских продуктов Smooth Streaming и их распространения среди других лицензиатов пакета портирования клиента Smooth Streaming, которые распространяют конечные клиентские продукты Smooth Streaming.

#### <a name="fee-structure"></a>Структура выплат
Лицензия на пакет портирования клиента Smooth Streaming предоставляется с выплатой единовременного лицензионного сбора в размере 50 000 долларов США. 

### <a name="microsoft-smooth-streaming-client-porting-kit-final-product-license"></a>Лицензия на конечный продукт пакета портирования клиента Microsoft Smooth Streaming.
По этой лицензии корпорация Майкрософт предоставляет все необходимые права на интеллектуальную собственность для получения промежуточных продуктов от других лицензиатов промежуточных продуктов пакета портирования клиента Smooth Streaming и для распространения продуктов Smooth Streaming под своей торговой маркой среди конечных пользователей.

#### <a name="fee-structure"></a>Структура выплат
Лицензия на конечный продукт пакета портирования клиента Smooth Streaming предоставляется на следующих условиях лицензионных отчислений.

* По 0,10 доллара США за каждое отправленное устройство, в котором используется конечный продукт.
* Общая сумма составляет не более 50 000 долларов США ежегодно.
* Отсутствие отчислений за первые 10 000 устройств в течение каждого года. 

## <a name="licensing-procedure-and-sspk-access"></a>Процедура лицензирования и доступ к SSPK.
Электронная почта [sspkinfo@microsoft.com](mailto:sspkinfo@microsoft.com) для всех запросов на лицензирование.

Зарегистрированные обладатели лицензии на промежуточный продукт получают доступ к Порталу распространения SSPK .

Промежуточные и окончательные лицензии SSPK могут отправлять технические вопросы по [smoothpk@microsoft.com](mailto:smoothpk@microsoft.com) .

## <a name="microsoft-smooth-streaming-client-interim-product-agreement-licensees"></a>Обладатели лицензий на промежуточный продукт пакета для портирования клиента Microsoft Smooth Streaming.

* Adroit Business Solutions, Inc
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* Albis Technologies Ltd.
* Alticast Corporation
* Amazon Digital Services, Inc.
* Arion Technology, Inc.
* AVC Multimedia Software Co., Ltd.
* Cavium, Inc.
* EchoStar Purchasing Corporation
* Enseo, Inc.
* Fluendo S.A.
* Гуангзау Шикун электроника., Ltd.
* HANDAN BroadInfoCom Co., Ltd.
* Infomir GMBH
* Irdeto USA Inc.
* Liberty Global Services BV
* MediaTek Inc.
* MStar Co, Ltd
* Nintendo Co., Ltd.
* OpenTV, Inc.
* Saffron Digital Limited
* Sichuan Changhong Electric Co., Ltd
* SoftAtHome
* Sony Corporation
* Tatung Technology Inc.
* Top Victory Investments, Ltd.
* Vestel Elektronik Sanayi ve Ticaret A.S.
* VisualOn, Inc.
* ZTE Corporation

## <a name="microsoft-smooth-streaming-client-final-product-agreement-licensees"></a>Обладатели лицензий на конечный продукт пакета для портирования клиента Microsoft Smooth Streaming
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* Амтран технология Co., Ltd 
* Arcadyan Technology Corporation
* Арцелик A. S
* Compal Electronics, Inc.
* С ОГРАНИЧЕННЫМ ЧИСЛОМ ТЕХНОЛОГИЙ EXPRESS
* Fluendo S.A.
* ФУНАИ ЭЛЕКТРИЧЕСКИЙ CO, LTD
* Hisense International Co., Ltd.
* Хисенсе визуальных технологий для совместной работы, Ltd
* Ограниченная корпорация ХКК
* Гонконг, Конка Ltd
* Корпорация иннолукс
* Жинпин электрическая компания Ltd. Зухаи. S. E. Z
* Технология K-Троникс (Сузау) CO., Ltd. 
* Kaonmedia Co., Ltd.
* KDDI Corporation
* Мегасимволов славы электроника
* Ограниченная электроника МИРК
* МОКА, ОГРАНИЧЕННЫЙ МЕЖДУНАРОДНЫЙ
* ОНЕПЛУС ЭЛЕКТРОНИКА (ШЕНЗЕН) CO., LTD.
* Корпорация Майкрософт, Panasonic
* Кингдао Хаиер Оптроникс Co., Ltd.
* Шензен Атеко, Co., Ltd.
* Шензен Чуангвеи — модель бытовой электроники по RGB, Ltd.
* Shenzhen Jiuzhou Electric Co., Ltd
* Шензен КТК технология Co., Ltd. 
* Шензен Максмаде технология Co., Ltd
* Шензен МТК Co., Ltd
* Shenzhen Skyworth Digital Technology Co., Ltd
* Sichuan Changhong Electric Co., Ltd.
* Skardin Industrial Corp.
* Небесно CP Ltd
* СМАРДТВ ГЛОБАЛЬНЫЙ SAS
* SoftAtHome
* Sony Corporation
* Technicolor Delivery Technologies, SAS
* Top Victory Investments, Ltd.
* UMC Польша SP. z. о.о.
* Визио, Inc.
* ZTE Corporation

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]


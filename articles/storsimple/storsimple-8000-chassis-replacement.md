---
title: Замена корпуса устройства StorSimple серии 8000 | Документация Майкрософт
description: Эта статья описывает снятие и установку основного корпуса или корпуса EBOD устройства StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: troubleshooting
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: de0f6299f35f9d76fdade976bf70456426e5ec51
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85513440"
---
# <a name="replace-the-chassis-on-your-storsimple-device"></a>Замена корпуса устройства StorSimple
## <a name="overview"></a>Обзор
В этом учебнике объясняется, как снять и заменить корпус устройства StorSimple серии 8000. Модель StorSimple 8100 — устройство в отдельном корпусе (один корпус), тогда как модель 8600 состоит из двух корпусов (два корпуса). В модели 8600 неисправность может возникнуть в двух корпусах: в основном корпусе и в корпусе EBOD.

В каждом из этих случаев Майкрософт предоставляет на замену пустой корпус. В комплект поставки не входят блоки питания и охлаждения (БПО), модули контроллера, твердотельные дисковые накопители (SSD), жесткие диски (HDD) и модули EBOD.

> [!IMPORTANT]
> Перед снятием и заменой корпуса ознакомьтесь со сведениями о безопасности в разделе [Замена компонентов оборудования StorSimple](storsimple-8000-hardware-component-replacement.md).


## <a name="remove-the-chassis"></a>Снятие корпуса
Выполните следующие действия для снятия корпуса устройства StorSimple.

#### <a name="to-remove-a-chassis"></a>Снятие корпуса
1. Убедитесь, что устройство StorSimple выключено и отключено от всех источников питания.
2. Отключите все сетевые кабели и кабели SAS, если они используются.
3. Извлеките устройство из стойки.
4. Извлеките каждый из дисков и запишите номера слотов, из которых они извлекаются. Дополнительные сведения см. в разделе [Снять жесткий диск](storsimple-8000-disk-drive-replacement.md#remove-the-disk-drive).
5. Для корпуса EBOD (при неисправности корпуса) снимите модули контроллеров EBOD. Дополнительные сведения см. в разделе [Снятие контроллера EBOD](storsimple-8000-ebod-controller-replacement.md#remove-an-ebod-controller).
   
    Для основного корпуса (при неисправности корпуса) снимите модули контроллеров и запишите номера слотов, в которых они находились. Дополнительные сведения см. [в разделе Удаление контроллера](storsimple-8000-controller-replacement.md#remove-a-controller).

## <a name="install-the-chassis"></a>Установка корпуса
Чтобы установить корпус устройства StorSimple, выполните следующие действия.

#### <a name="to-install-a-chassis"></a>Установка корпуса
1. Установите корпус в стойку. Дополнительные сведения см. в разделах [Установка устройства StorSimple 8100 в стойку](storsimple-8100-hardware-installation.md#rack-mount-your-storsimple-8100-device) или [Установка устройства StorSimple 8600 в стойку](storsimple-8600-hardware-installation.md#rack-mount-your-storsimple-8600-device).
2. После установки корпуса в стойку установите модули контроллера так же, как они были установлены ранее.
3. Установите диски в те же слоты, в которые они были установлены ранее.
   
   > [!NOTE]
   > Рекомендуется сначала устанавливать твердотельные жесткие диски, а затем жесткие диски.
  
4. После установки корпуса в стойку и установки компонентов подключите устройство к соответствующим источникам питания и включите устройство. Дополнительные сведения см. в разделах [Подключение кабельного хозяйства к устройству StorSimple 8100](storsimple-8100-hardware-installation.md#cable-your-storsimple-8100-device) или [Подключение кабельного хозяйства к устройству StorSimple 8600](storsimple-8600-hardware-installation.md#cable-your-storsimple-8600-device).

## <a name="next-steps"></a>Дальнейшие действия
Узнайте подробнее о [замене компонентов оборудования StorSimple](storsimple-8000-hardware-component-replacement.md).


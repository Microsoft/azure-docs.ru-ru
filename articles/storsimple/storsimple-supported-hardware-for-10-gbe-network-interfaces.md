---
title: Оборудование для интерфейсов 10 GbE устройства StorSimple | Документация Майкрософт
description: Описаны поддерживаемые подключаемые приемопередатчики небольшого размера (SFP), кабели и коммутаторы для сетевых интерфейсов 10 GbE на устройстве StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: df8d40c7-f5ad-4f84-93eb-779fbd5f7243
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 09/21/2016
ms.author: alkohli
ms.openlocfilehash: fd30c7ec15eadd3d7945349a8aae4ac824bfdc7f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94954092"
---
# <a name="supported-hardware-for-the-10-gbe-network-interfaces-on-your-storsimple-device"></a>Поддерживаемое оборудование для сетевых интерфейсов 10 GbE на устройстве StorSimple

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Обзор
Также приводятся сведения о дополнительном оборудовании, которое работает с устройством Microsoft Azure StorSimple.

## <a name="list-of-devices-tested-by-microsoft"></a>Список устройств, протестированных в корпорации Майкрософт
Корпорация Майкрософт проверила следующие подключаемые приемопередатчики небольшого размера (SFP), кабели и коммутаторы, чтобы убедиться, что они оптимально работают с устройствами. (Следующие таблицы будут обновляться по мере тестирования оборудования.)

### <a name="sfp-transceivers"></a>Приемопередатчики SFP +
| Производитель | Модель |
| --- | --- |
| Cisco |SFP-10G-SR |

### <a name="cables"></a>Кабели
| Х. Нет. | Производитель | Модель |
| --- | --- | --- |
| 1. |Cisco |SFP-H10GB-CU1M |
| 2. |Cisco |SFP-H10GB-CU2M |
| 3. |Cisco |SFP-H10GB-CU3M |
| 4. |Tripp-Lite |N820-05M (OM3) |

### <a name="switches"></a>коммутаторы;
| Х. Нет. | Производитель | Модель |
| --- | --- | --- |
| 1. |Cisco |N3K-C3172PQ-10GE |
| 2. |Cisco |N3K-C3048-ZM-F |
| 3. |Cisco |N5K-C5596UP-FA |

## <a name="list-of-devices-tested-in-the-field"></a>Список устройств, протестированных на местах
В этом разделе содержится список устройств, которые были успешно развернуты на местах клиентами StorSimple. Они не были проверены корпорацией Майкрософт, но, скорее всего, подходят для работы с устройством StorSimple.

| Параметр | Значение |
| --- | --- |
| Производитель коммутатора |Juniper |
| Модель коммутатора |ex4550-32F |
| Версия операционной системы коммутатора |JunOS 12.3R9.4 |
| Модель блейд-модуля |Встроенные порты (PIC 0) |
| Производитель приемопередатчика |Juniper |
| Модель приемопередатчика |Артикул 740-021308  <br></br>  Артикул 740-030658 |
| Версия встроенного ПО приемопередатчика |Редакция 01 версии 0.0 (сообщенная) |
| Модель кабеля |Дуплексная перемычка LC/LC 50/125μ, OM3, ЛСЗ |
| Модель StorSimple |8600 |
| Версия программного обеспечения StorSimple |6.3.9600.17491 |

## <a name="list-of-devices-tested-by-oem-provider-mellanox"></a>Список устройств, протестированных поставщиком OEM (Mellanox)
Mellanox проверила следующие подключаемые приемопередатчики небольшого размера (SFP), кабели и коммутаторы, чтобы убедиться, что они оптимально работают с сетевыми интерфейсами Mellanox, например с сетевыми интерфейсами 10 GbE на устройстве StorSimple.

### <a name="cables-and-modules-supported-by-mellanox"></a>Кабели и модули, поддерживаемые Mellanox
Ниже перечислены кабели и модули, поддерживаемые Mellanox. Они не были проверены корпорацией Майкрософт, но, скорее всего, подходят для работы с устройством StorSimple.

| Х. Нет. | Speed | Модель | Описание | Производитель |
| --- | --- | --- | --- | --- |
| 1. |10 GbE |CAB-SFP-SFP-1M |Пассивный медный кабель SFP + 10 Гбит/с, 1 м |Arista |
| 2. |10 GbE |CAB-SFP-SFP-2M |Пассивный медный кабель SFP + 10 Гбит/с, 2 м |Arista |
| 3. |10 GbE |CAB-SFP-SFP-3M |Пассивный медный кабель SFP + 10 Гбит/с, 3 м |Arista |
| 4. |10 GbE |CAB-SFP-SFP-5M |Пассивный медный кабель SFP + 10 Гбит/с, 5 м |Arista |
| 5. |10 GbE |Cisco SFP-H10GBCU1M |Кабель SFP + Cisco |Cisco |
| 6. |10 GbE |Cisco SFP-H10GBCU3M |Кабель SFP + Cisco |Cisco |
| 7. |10 GbE |Cisco SFP-H10GBCU5M |Кабель SFP + Cisco |Cisco |
| 8. |10 GbE |J9281B HP X242 10G |Медный кабель с прямым подключением SFP + на SFP + 1 м |HP |
| 9. |10 GbE |455883-B21 HP BLc |SR SFP+ Opt 10 Гбит |HP |
| 10. |10 GbE |455886-B21 HP BLc |LR SFP+ Opt 10 Гбит |HP |
| 11. |10 GbE |487649-B21 HP BLc |Медный кабель SFP+ 0,5 м 10GbE |HP |
| 12. |10 GbE |487652-B21 HP BLc |Медный кабель SFP+ 1 м 10GbE |HP |
| 13. |10 GbE |487655-B21 HP BLc |Медный кабель SFP+ 3 м 10GbE |HP |
| 14. |10 GbE |487658-B21 HP BLc |Медный кабель SFP+ 7 м 10GbE |HP |
| 15. |10 GbE |537963-B21 HP BLc |Медный кабель SFP+ 5 м 10GbE |HP |
| 16. |10 GbE |AP784A HP |Пассивный медный кабель C-серии SFP+ 3 м |HP |
| 17. |10 GbE |AP785A HP |Пассивный медный кабель C-серии SFP+ 5 м |HP |
| 18. |10 GbE |AP818A HP |Активный медный кабель B-серии SFP+ 1 м |HP |
| 19. |10 GbE |AP819A HP |Активный медный кабель B-серии SFP+ 3 м |HP |
| 20. |10 GbE |J9150A HP |Приемопередатчик X132 10G SFP+ LC SR |HP |
| 21. |10 GbE |J9151A HP |Приемопередатчик X132 10G SFP+ LC LR |HP |
| 22. |10 GbE |J9283B HP |Кабель DAC X242 10G SFP+ SFP+ 3 м |HP |
| 23. |10 GbE |J9285B HP |Кабель DAC X242 10G SFP+ SFP+ 7 м |HP |
| 24. |10 GbE |JD095B HP |Кабель DAC X240 10G SFP+ SFP+ 0,65 м |HP |
| 25. |10 GbE |JD096B HP |Кабель DAC X240 10G SFP+ SFP+ 1,2 м |HP |
| 26. |10 GbE |JD097B HP |Кабель DAD X240 10G SFP+ SFP+ 3 м |HP |
| 27. |10 GbE |MAM1Q00A-QSA Mellanox |Адаптер QSFP To SFP+ |Mellanox Technologies |
| 28. |10 GbE |MC2309124-006 Mt |Пассивный медный кабель 1x SFP+ To QSFP 10 Гбит/с 24awg 7 м |Mellanox Technologies |
| 29. |10 GbE |MC2309124-007 Mt |Пассивный медный кабель 1x SFP+ To QSFP 10 Гбит/с 24awg 7 м |Mellanox Technologies |
| 30. |10 GbE |MC2309130-003 Mt |Пассивный медный кабель 1x SFP+ To QSFP 10 Гбит/с 30awg 3 м |Mellanox Technologies |
| 31. |10 GbE |MC2309130-00A Mt |Пассивный медный кабель 1x SFP+ To QSFP 10 Гбит/с 30awg 0,5 м |Mellanox Technologies |
| 32. |10 GbE |MC3309124-005 Mt |Пассивный медный кабель 1x SFP+ 10 Гбит/с 24awg 5 м |Mellanox Technologies |
| 33. |10 GbE |MC3309124-007 Mt |Пассивный медный кабель 1x SFP+ 10 Гбит/с 24awg 7 м |Mellanox Technologies |
| 34. |10 GbE |MC3309130-003 Mt |Пассивный медный кабель 1x SFP+ 10 Гбит/с 30awg 3 м |Mellanox Technologies |
| 35. |10 GbE |MC3309130-00A Mt |Пассивный медный кабель 1x SFP+ 10 Гбит/с 30awg 0,5 м |Mellanox Technologies |

### <a name="switches-supported-by-mellanox"></a>Коммутаторы, поддерживаемые Mellanox
Ниже перечислены коммутаторы, поддерживаемые Mellanox. Они не были проверены корпорацией Майкрософт, но, скорее всего, подходят для работы с устройством StorSimple.

| Х. Нет. | Speed | Модель | Описание | Производитель |
| --- | --- | --- | --- | --- |
| 1. |10GbE |516733-B21 |Коммутатор HP ProCurve 6120XG 10GbE Ethernet Blade Switch |HP |
| 2. |10GbE |538113-B21 |Переходный модуль (PTM) HP 10GbE |HP |
| 3. |10GbE |EN4093 |Масштабируемый модуль коммутатора IBM PureFlex System Fabric EN4093 10 Gigabit |IBM |
| 4. |1GbE |3020 |Колонка коммутатора Cisco Catalyst 3020 1GbE |Cisco |
| 5. |1GbE |3020X |Колонка коммутатора Cisco Catalyst 3020X 1GbE |Cisco |
| 6. |1GbE |438030-B21 |Модуль коммутатора HP 1GbE — GbE2c Layer 2/3 Ethernet Blade Switch |HP |
| 7. |1GbE |6120G |Колонка коммутатора HP ProCurve 6120G/XG 1GbE |HP |

## <a name="next-steps"></a>Дальнейшие действия
[Дополнительные сведения об аппаратных компонентах и состоянии StorSimple](./storsimple-8000-monitor-hardware-status.md).
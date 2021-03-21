---
title: Местонахождение данных клиента в Azure IoT Central | Документация Майкрософт
description: В этой статье описывается местонахождение данных клиентов в приложениях IoT Central Azure.
author: dominicbetts
ms.author: dobett
ms.date: 11/02/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 9cf803ddd5a3c9329cadefffc35ae6d6999323a1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93280707"
---
# <a name="azure-iot-central-customer-data-residency"></a>Местонахождение данных клиента Azure IoT Central

IoT Central не хранит данные клиента за пределами указанного пользователем географического региона, за исключением следующих сценариев:

- Когда в существующее приложение IoT Central добавляется новый пользователь, идентификатор электронной почты пользователя может храниться за пределами географического времени, пока приглашенный пользователь не попытается получить первый доступ к приложению.

- Плитки карт панели мониторинга IoT Central используют [Azure Maps](../../azure-maps/about-azure-maps.md). При добавлении плитки карты в существующее IoT Central приложение данные о расположении могут обрабатываться или храниться в соответствии с правилами географического расположения службы Azure Maps.

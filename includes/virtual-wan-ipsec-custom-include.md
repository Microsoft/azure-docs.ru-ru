---
title: включить файл
description: включить файл
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 10/07/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 83f0ce27172879a37de9488499e46de30b8e112c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98147346"
---
При работе с пользовательскими политиками IPsec учитывайте следующие требования.

* **IKE** — для IKE можно выбрать любой параметр из шифрования IKE, а также любой параметр из группы Диффи-Хелмана.
* **IPSec** — для IPSec можно выбрать любой параметр из шифрования IPSec, а также любой параметр из целостности IPSec, а также PFS. Если какой-либо из параметров для шифрования IPsec или IPsec имеет значение GCM, то параметры обоих параметров должны быть GCM.

>[!NOTE]
> В пользовательских политиках IPsec нет концепции ответчика и инициатора (в отличие от политик IPsec по умолчанию). Обе стороны (локальный и VPN-шлюз Azure) будут использовать те же параметры, что и на этапе IKE 1 и на этапе 2 IKE. Поддерживаются протоколы IKEv1 и IKEv2. Azure не поддерживается только в качестве респондента.
>

**Доступные параметры и параметры**

| Параметр | Параметры |
|--- |--- |
| Шифрование IKE | GCMAES256, GCMAES128, AES256, AES128 |
| Целостность IKE | SHA384, SHA256 |
| Группа DH | ECP384, ECP256, DHGroup24, DHGroup14 |
| Шифрование IPsec | GCMAES256, GCMAES128, AES256, AES128, нет |
| Целостность IPsec | GCMAES256, GCMAES128, SHA256 |
| Группа PFS | ECP384, ECP256, PFS24, PFS14, нет |
| Срок действия SA |цело мин. 300/по умолчанию 27000 с |

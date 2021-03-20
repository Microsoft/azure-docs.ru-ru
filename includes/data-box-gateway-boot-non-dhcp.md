---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: db4e27c0972a93d870d0a6565f1bd3212146df62
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96582735"
---
При загрузке в среде без DHCP выполните следующие действия, чтобы развернуть виртуальную машину для Шлюз Data Box.

1. [Подключитесь к интерфейсу Windows PowerShell устройства](#connect-to-the-powershell-interface).
2. Используйте `Get-HcsIpAddress` командлет, чтобы вывести список сетевых интерфейсов, включенных на вашем виртуальном устройстве. Если на устройстве включен один сетевой интерфейс, по умолчанию ему назначается имя `Ethernet`.

    В следующем примере показано использование этого командлета:

    ```
    [10.100.10.10]: PS>Get-HcsIpAddress

    OperationalStatus : Up
    Name              : Ethernet
    UseDhcp           : True
    IpAddress         : 10.100.10.10
    Gateway           : 10.100.10.1
    ```

3. Для настройки сети используйте командлет `Set-HcsIpAddress` . См. следующий пример.

    ```
    Set-HcsIpAddress –Name Ethernet –IpAddress 10.161.22.90 –Netmask 255.255.255.0 –Gateway 10.161.22.1
    ```


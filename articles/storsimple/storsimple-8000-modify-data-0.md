---
title: Изменение параметров DATA 0 на устройстве StorSimple серии 8000 | Документация Майкрософт
description: Узнайте, как перенастроить сетевой интерфейс DATA 0 на устройстве StorSimple с помощью Windows PowerShell для StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/27/2017
ms.author: alkohli
ms.openlocfilehash: 85d7114f419266124d0d23368b24700af025758a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94961045"
---
# <a name="modify-the-data-0-network-interface-settings-on-your-storsimple-8000-series-device"></a>Изменение параметров сетевого интерфейса DATA 0 на устройстве StorSimple серии 8000

## <a name="overview"></a>Обзор

Устройство Microsoft Azure StorSimple имеет шесть сетевых интерфейсов, от DATA 0 до DATA 5. Интерфейс DATA 0 всегда настраивается с помощью интерфейса Windows PowerShell или последовательной консоли и располагает автоматическим подключением к облаку. Обратите внимание на то, что сетевой интерфейс DATA 0 невозможно настроить через портал Azure.

Интерфейс DATA 0 изначально настраивается через мастер установки во время первоначального развертывания устройства StorSimple. Если устройство находится в рабочем режиме, может потребоваться перенастроить параметры DATA 0. В этом учебнике приведены два метода изменения сетевых параметров DATA 0. В обоих используется Windows PowerShell для StorSimple.

После изучения этого учебника вы сможете сделать следующее.

* Изменить сетевые параметры DATA 0 с помощью мастера установки.
* Изменить сетевые параметры DATA 0 с помощью командлета `Set-HcsNetInterface` .

## <a name="modify-data-0-network-settings-through-setup-wizard"></a>Изменение сетевых параметров DATA 0 с помощью мастера установки
Можно перенастроить сетевые параметры DATA 0 путем подключения к интерфейсу Windows PowerShell устройства StorSimple и запуска сеанса мастера установки. Чтобы изменить параметры DATA 0, выполните указанные ниже действия.

#### <a name="to-modify-data-0-network-settings-through-setup-wizard"></a>Изменение сетевых параметров DATA 0 с помощью мастера установки
1. В меню последовательной консоли выберите параметр 1 **Войти с полным доступом**. При появлении запроса укажите **пароль администратора устройства**. Пароль по умолчанию: `Password1`.
2. В командной строке введите следующее:
   
    `Invoke-HcsSetupWizard`
3. Отобразится мастер установки, который поможет настроить интерфейс DATA 0 устройства. Введите новые значения для IP-адреса, шлюза и маски подсети.

> [!NOTE]
> Фиксированные IP-адреса контроллеров необходимо будет перенастроить в колонке **Параметры сети** устройства StorSimple на портале Azure. Дополнительные сведения см. в статье [Изменение сетевых интерфейсов](storsimple-8000-modify-device-config.md#modify-network-interfaces).

## <a name="modify-data-0-network-settings-through-set-hcsnetinterface-cmdlet"></a>Изменение сетевых параметров DATA 0 с помощью командлета Set-HcsNetInterface
Альтернативный способ перенастроить сетевой интерфейс DATA 0 — воспользоваться командлетом `Set-HcsNetInterface` . Командлет выполняется из интерфейса Windows PowerShell устройства StorSimple. При использовании этой процедуры здесь также можно настроить статические IP-адреса контроллера. Чтобы изменить параметры DATA 0, выполните указанные ниже действия. 

#### <a name="to-modify-data-0-network-settings-through-the-set-hcsnetinterface-cmdlet"></a>Изменение сетевых параметров DATA 0 с помощью командлета Set-HcsNetInterface
1. В меню последовательной консоли выберите параметр 1 **Войти с полным доступом**. При выводе запроса введите пароль администратора устройства. Пароль по умолчанию: `Password1`.
2. В командной строке введите следующее:
   
    `Set-HCSNetInterface -InterfaceAlias Data0 -IPv4Address <> -IPv4Netmask <> -IPv4Gateway <> -Controller0IPv4Address <> -Controller1IPv4Address <> -IsiScsiEnabled 1 -IsCloudEnabled 1`
   
    В угловых скобках введите следующие значения для DATA 0:
   
   * IPv4-адрес
   * шлюза IPv4;
   * маски подсети IPv4;
   * фиксированного IPv4-адреса для контроллера 0;
   * фиксированного IPv4-адреса для контроллера 1.
     
     Дополнительные сведения об использовании этого командлета см. в [справочнике по командлетам Windows PowerShell для StorSimple](/previous-versions/windows/powershell-scripting/dn688161(v=wps.630)).

## <a name="next-steps"></a>Дальнейшие действия
* Сетевые интерфейсы, отличные от DATA 0, можно [настроить на портале Azure](storsimple-8000-modify-device-config.md). 
* Если у вас возникли проблемы при настройке сетевых интерфейсов, см. статью [Устранение неполадок в развертывании устройства StorSimple](./storsimple-8000-troubleshoot-deployment.md).
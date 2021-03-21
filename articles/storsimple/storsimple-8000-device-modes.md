---
title: Изменение режима устройства StorSimple | Документация Майкрософт
description: В статье описываются режимы устройства StorSimple и объясняется, как изменить режим устройства StorSimple с помощью Windows PowerShell.
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
ms.date: 06/29/2017
ms.author: alkohli
ms.openlocfilehash: c7b0ea489c1d70ab86d677aad666ea6728fa76b4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96017089"
---
# <a name="change-the-device-mode-on-your-storsimple-device"></a>Переключение режима устройства StorSimple

В этой статье представлено краткое описание различных режимов, в которых могут работать устройства StorSimple. Устройство StorSimple может работать в трех режимах: нормальном, обслуживания и восстановления.

После прочтения этой статьи вы узнаете:

* о режимах устройств StorSimple;
* о том, как выяснить, в каком режиме находится устройство StorSimple;
* о том, как переключиться из нормального режима в режим обслуживания и *наоборот*

Эти задачи можно выполнить только через интерфейс Windows PowerShell устройства StorSimple.

## <a name="about-storsimple-device-modes"></a>Режимы устройств StorSimple

Устройство StorSimple может работать в нормальном режиме, режиме обслуживания и режиме восстановления. Ниже кратко описан каждый из этих режимов.

### <a name="normal-mode"></a>Нормальный режим

Это нормальный рабочий режим полностью настроенного устройства StorSimple. По умолчанию устройство должно находиться в нормальном режиме.

### <a name="maintenance-mode"></a>Режим обслуживания

Иногда может потребоваться перевести устройство StorSimple в режим обслуживания. Этот режим позволяет выполнять обслуживание устройства и устанавливать критические обновления, например обновления встроенного ПО дисков.

Перевести систему в режим обслуживания можно только через Windows PowerShell для StorSimple. В этом режиме приостанавливаются все запросы ввода-вывода. Также останавливаются некоторые другие службы, например энергонезависимое ОЗУ (NVRAM) или служба кластеров. При входе в этот режим и выходе из него перезапускаются оба контроллера. При выходе из режима обслуживания все службы возобновляют работу и должны работать без ошибок. Это может занять несколько минут.

> [!NOTE]
> **Режим обслуживания поддерживается только на правильно работающем устройстве. Он не поддерживается на устройстве, в котором один или оба контроллера не работают.**


### <a name="recovery-mode"></a>Режим восстановления

Режим восстановления можно описать как "безопасный режим Windows с поддержкой сети". Режим восстановления включает взаимодействие со службой поддержки Майкрософт, сотрудники которой выполняют диагностику системы. Основная цель режима восстановления — получение системных журналов.

Если система переходит в режим восстановления, вы должны обратиться в службу технической поддержки Майкрософт за дальнейшими действиями. Для получения дополнительных сведений перейдите по [адресу служба поддержки Майкрософт](storsimple-8000-contact-microsoft-support.md).

> [!NOTE]
> **Переключить устройство в режим восстановления нельзя. Если устройство находится в неисправном состоянии, то в режиме восстановления будет предпринята попытка переключить его в то состояние, в котором его смогут изучить специалисты технической поддержки Майкрософт.**

## <a name="determine-storsimple-device-mode"></a>Определение режима устройства StorSimple

#### <a name="to-determine-the-current-device-mode"></a>Порядок определения текущего режима устройства

1. Войдите в последовательную консоль устройства, выполнив действия, описанные в разделе [Использование PuTTY для подключения к последовательной консоли устройства](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
2. Просмотрите заглавное сообщение в меню последовательной консоли устройства. Это сообщение явно указывает, в каком режиме находится устройство: обслуживания или восстановления. Если сообщение не содержит никакой специальной информации, относящейся к режиму работы системы, то устройство находится в нормальном режиме.

## <a name="change-the-storsimple-device-mode"></a>Изменение режима устройства StorSimple

Устройство можно переключить в режим обслуживания (из нормального режима) для проведения обслуживания или установки обновлений режима обслуживания. Для ввода в режим обслуживания или выхода из него выполните следующие действия.

> [!IMPORTANT]
> Перед переключением в режим обслуживания убедитесь, что оба контроллера устройства работоспособны. Для этого откройте меню **Параметры устройства > Работоспособность оборудования** своего устройства на портале Azure. Если один или оба контроллера неработоспособны, обратитесь в службу поддержки Майкрософт для получения дальнейших указаний. Для получения дополнительных сведений перейдите по [адресу служба поддержки Майкрософт](storsimple-8000-contact-microsoft-support.md).
 

#### <a name="to-enter-maintenance-mode"></a>Переход в режим обслуживания

1. Войдите в последовательную консоль устройства, выполнив действия, описанные в разделе [Использование PuTTY для подключения к последовательной консоли устройства](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
2. В меню последовательной консоли выберите параметр 1 **Войти с полным доступом**. При выводе запроса введите **пароль администратора устройства**. Пароль по умолчанию: `Password1`.
3. В командной строке выполните следующую команду: 
   
    `Enter-HcsMaintenanceMode`
4. Отобразится предупреждающее сообщение о том, что режим обслуживания будет мешать работе всех запросов ввода-вывода и приведет к разрыву подключения к порталу Azure, в результате чего появится запрос на подтверждение. Нажмите клавишу **Y** , чтобы перейти в режим обслуживания.
5. Оба контроллера будут перезапущены. После завершения перезапуска баннер последовательной консоли отобразит сообщение о том, что устройство находится в режиме обслуживания. Результат выполнения команды показан ниже.

```
    ---------------------------------------------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0 - Passive
    ---------------------------------------------------------------

    Controller0>Enter-HcsMaintenanceMode
    Checking device state...

    In maintenance mode, your device will not service IOs and will be disconnected from the Microsoft Azure StorSimple Manager service. Entering maintenance mode will end the current session and reboot both controllers, which takes a few minutes to complete. Are you sure you want to enter maintenance mode?
    [Y] Yes [N] No (Default is "Y"): Y

    <BOTH CONTROLLERS RESTART>

    -----------------------MAINTENANCE MODE------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0 - Passive
    ---------------------------------------------------------------

    Serial Console Menu
    [1] Log in with full access
    [2] Log into peer controller with full access
    [3] Connect with limited access
    [4] Change language
    Please enter your choice>

```

#### <a name="to-exit-maintenance-mode"></a>Выход из режима обслуживания

1. Войдите в последовательную консоль устройства. Проверьте, находится ли устройство в режиме обслуживания, с помощью заглавного сообщения.
2. В командной строке введите следующее:
   
    `Exit-HcsMaintenanceMode`
3. Отобразится сообщение с предупреждением и сообщение с запросом на подтверждение. Нажмите клавишу **Y** , чтобы выйти из режима обслуживания.
4. Оба контроллера будут перезапущены. После завершения перезапуска баннер последовательной консоли отобразит сообщение о том, что устройство находится в обычном режиме. Результат выполнения команды показан ниже.

```
    -----------------------MAINTENANCE MODE------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0
    ---------------------------------------------------------------

    Controller0>Exit-HcsMaintenanceMode
    Checking device state...

    Before exiting maintenance mode, ensure that any updates that are required on both controllers have been applied. Failure to install on each controller could result in data corruption. Exiting maintenance mode will end the current session and reboot both controllers, which takes a few minutes to complete. Are you sure you want to exit maintenance mode?
    [Y] Yes [N] No (Default is "Y"): Y

    <BOTH CONTROLLERS RESTART>

    ---------------------------------------------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0 - Active
    ---------------------------------------------------------------

    Serial Console Menu
    [1] Log in with full access
    [2] Log into peer controller with full access
    [3] Connect with limited access
    [4] Change language
    Please enter your choice>
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [применить обновления нормального режима и режима обслуживания](storsimple-update-device.md) к своему устройству StorSimple.


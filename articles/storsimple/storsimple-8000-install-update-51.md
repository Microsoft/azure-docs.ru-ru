---
title: Установка обновления 5,1 на устройство StorSimple 8000 Series | Документация Майкрософт
description: В этой статье объясняется, как установить обновление 5,1 для серии StorSimple 8000 на устройство StorSimple 8000 Series.
services: storsimple
documentationcenter: NA
author: alkohli
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 03/05/2020
ms.author: alkohli
ms.openlocfilehash: 5b9958f3dd497aa612a92947b8d968439ef9d0e4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91575968"
---
# <a name="install-update-51-on-your-storsimple-device"></a>Установка обновления 5,1 на устройстве StorSimple

## <a name="overview"></a>Обзор

В этом учебнике объясняется, как установить обновление 5,1 на устройстве StorSimple с более ранней версией программного обеспечения с помощью портал Azure. <!--The hotfix method is used when you are trying to install Update 5.1 on a device running pre-Update 3 versions. The hotfix method is also used when a gateway is configured on a network interface other than DATA 0 of the StorSimple device and you are trying to update from a pre-Update 1 software version.-->

Обновление 5,1 включает в себя обновления для системы безопасности без сбоев. Непостоянное или регулярное обновление можно применить с помощью портал Azure <!--or by the hotfix method-->.

> [!IMPORTANT]
>
> * Обновление 5,1 является обязательным и должно быть установлено немедленно. Дополнительные сведения см. в разделе [обновление 5,1. заметки о выпуске](storsimple-update51-release-notes.md).
> * Набор предварительных проверок (выполняются вручную и автоматически перед установкой), которые позволяют определить работоспособность устройства, то есть состояние его оборудования и качество сетевого подключения. Эти предварительные проверки выполняются только при применении обновлений из портал Azure.
> * Если вы хотите установить с помощью метода исправления, обратитесь в [Служба поддержки Майкрософт](mailto:support@microsoft.com).

<!--
> * We strongly recommend that when updating a device running versions prior to Update 3, you install the updates using hotfix method. If you encounter any issues, [log a support ticket](storsimple-8000-contact-microsoft-support.md). 


> * We recommend that you install the software and other regular updates via the Azure portal. You should only go to the Windows PowerShell interface of the device (to install updates) if the pre-update gateway check fails in the portal. Depending upon the version you are updating from, the updates may take 4 hours (or greater) to install. The maintenance mode updates must be installed through the Windows PowerShell interface of the device. As maintenance mode updates are disruptive updates, these result in a down time for your device.


> * If running the optional StorSimple Snapshot Manager, ensure that you have upgraded your Snapshot Manager version to Update 5.1 prior to updating the device.
-->

[!INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## <a name="install-update-51-through-the-azure-portal"></a>Установите обновление 5,1 с помощью портал Azure

Чтобы обновить устройство до [обновления 5,1](storsimple-update51-release-notes.md), выполните следующие действия.

> [!NOTE]
> Майкрософт получает дополнительную диагностическую информацию от устройства. Поэтому при обнаружении нашей операционной группой проблем на устройствах мы имеем более широкие возможности для сбора информации с устройства и диагностики проблем.

#### <a name="to-install-an-update-from-the-azure-portal"></a>Установка обновления с портала Azure

1. На странице службы StorSimple выберите свое устройство.

    ![Выбор устройства](./media/storsimple-8000-install-update-51/update1.png)

2. Выберите **Параметры устройства**  >  **обновления устройства**.

    ![Снимок экрана: колонка параметров с вызываемым параметром обновления устройства.](./media/storsimple-8000-install-update-51/update2.png)

3. Если доступны новые обновления, появится уведомление. Либо в колонке **Обновления устройства** щелкните **Проверить обновления**. Будет создано задание поиска доступных обновлений. Вы получите уведомление об успешном завершении задания.

    ![Снимок экрана: колонка параметров с вызываемым параметром обновления устройства и колонка обновления устройства с новыми регулярными обновлениями доступно сообщение с именем out.](./media/storsimple-8000-install-update-51/update3.png)

4. Перед установкой обновления на устройство рекомендуется ознакомиться с заметками о выпуске. Чтобы применить обновления, щелкните **Установить обновления**. Перед установкой обновлений ознакомьтесь с предварительными условиями в колонке **подтверждения регулярных обновлений**. Установите флажок, чтобы указать, что вы готовы приступить к обновлению устройства, а затем щелкните **Установить**.

    ![Снимок экрана: колонка "обновления устройства" с вызываемым параметром "установить обновления" и подтверждением регулярных обновлений с параметром "принять" и параметром "установить с именем".](./media/storsimple-8000-install-update-51/update4.png)

5. Начинается выполнение ряда предварительных проверок. Эти проверки включают в себя:
   
   * **Проверки работоспособности контроллера** подтверждают, что оба контроллера устройства готовы к работе и подключены к сети.
   * **Проверки работоспособности компонентов оборудования** подтверждают, что все компоненты оборудования устройства StorSimple готовы к работе.
   * **Проверки DATA 0** подтверждают, что интерфейс DATA 0 включен на устройстве. Если этот интерфейс не включен, его следует включить и повторить проверку.

     Обновление будет скачано и установлено только в том случае, если все проверки успешно пройдены. Вы получите уведомление о выполнении проверок. Если предварительные проверки завершатся сбоем, вы получите уведомление о причинах сбоя. Устраните эти проблемы и продолжите выполнение операции. Если вы не можете устранить обнаруженные проблемы самостоятельно, обратитесь в службу поддержки Майкрософт.

7. После успешного завершения предварительных проверок будет создано задание обновления. После его создания вы получите соответствующее уведомление.
   
    ![Снимок экрана уведомления о запуске задания обновления программного обеспечения.](./media/storsimple-8000-install-update-51/update6.png)
   
    Затем обновление будет установлено на вашем устройстве.

9. Процедура обновления занимает несколько часов. Выберите задание обновления и нажмите кнопку **Сведения** , чтобы в любой момент просмотреть сведения о задании.

    ![Снимок экрана: колонка "обновления устройства" с загрузкой и установкой обновлений программного обеспечения в колонке "выполняется" и "Установка обновлений".](./media/storsimple-8000-install-update-51/update8.png)

     Вы также можете отслеживать ход выполнения задания обновления в колонке **Параметры устройства > Задания**. В колонке **Задания** отображается ход обновления.

     ![Снимок экрана: колонка "Параметры" с вызываемыми заданиями и колонка "задания", показывающая ход выполнения обновления.](./media/storsimple-8000-install-update-51/update7.png)

10. После завершения задания выберите **Параметры устройства > Обновления устройства**. Теперь версия программного обеспечения должна быть обновлена.


Убедитесь, что на устройстве установлено **обновление 5,1 для серии StorSimple 8000 (6.3.9600.17885)**. Также должна измениться **дата последнего обновления**.
<!-- 5.1 - KB 4542887-->

<!--You will now see that the Maintenance mode updates are available (this message might continue to be displayed for up to 24 hours after you install the updates). The steps to install maintenance mode update are detailed in the next section.

[!INCLUDE [storsimple-8000-install-maintenance-mode-updates](../../includes/storsimple-8000-install-maintenance-mode-updates.md)]

## Install Update 5.1 as a hotfix

The software versions that can be upgraded using the hotfix method are:

* Update 0.1, 0.2, 0.3
* Update 1, 1.1, 1.2
* Update 2, 2.1, 2.2
* Update 3, 3.1
* Update 4
* Update 5

> [!NOTE]
> The recommended method to install Update 5.1 is through the Azure portal when trying to update from Update 3 and later version. When updating a device running versions prior to Update 3, use this procedure. You can also use this procedure if you fail the gateway check when trying to install the updates through the Azure portal. The check fails when you have a gateway assigned to a non-DATA 0 network interface and your device is running a software version earlier than Update 1.

The hotfix method involves the following three steps:

1. Download the hotfixes from the Microsoft Update Catalog.
2. Install and verify the regular mode hotfixes.
3. Install and verify the maintenance mode hotfix.

#### Download updates for your device

You must download and install the following hotfixes in the prescribed order and the suggested folders:

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 1. |KB4037264 |Software update<br> Download both _HcsSoftwareUpdate.exe_ and _CisMSDAgent.exe_ |Regular <br></br>Non-disruptive |~ 25 mins |FirstOrderUpdate|

If updating from a device running Update 4, you only need to install the OS cumulative updates as second order updates.

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 2A. |KB4025336 |OS cumulative updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|

If installing from a device running Update 3 or earlier, install the following in addition to the cumulative updates.

| Order | KB | Description | Update type | Install time |Install in folder|
| --- | --- | --- | --- | --- | --- |
| 2B. |KB4011841 <br> KB4011842 |LSI driver and firmware updates <br> USM firmware update (version 3.38) |Regular <br></br>Non-disruptive |~ 3 hrs <br> (includes 2A. + 2B. + 2C.)|SecondOrderUpdate|
| 2C. |KB3139398 <br> KB3142030 <br> KB3108381 <br> KB3153704 <br> KB3174644 <br> KB3139914   |OS security updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|
| 2D. |KB3146621 <br> KB3103616 <br> KB3121261 <br> KB3123538 |OS updates package <br> Download Windows Server 2012 R2 version |Regular <br></br>Non-disruptive |- |SecondOrderUpdate|



You may also need to install disk firmware updates on top of all the updates shown in the preceding tables. You can verify whether you need the disk firmware updates by running the `Get-HcsFirmwareVersion` cmdlet. If you are running these firmware versions: `XMGJ`, `XGEG`, `KZ50`, `F6C2`, `VR08`, `N003`, `0107`, then you do not need to install these updates.

| Order | KB | Description | Update type | Install time | Install in folder|
| --- | --- | --- | --- | --- | --- |
| 3. |KB4037263 |Disk firmware |Maintenance <br></br>Disruptive |~ 30 mins | ThirdOrderUpdate |

<br></br>

> [!IMPORTANT]
> * If updating from Update 4, the total install time is close to 4 hours.
> * Before using this procedure to apply the update, make sure that both the device controllers are online and all the hardware components are healthy.

Perform the following steps to download and install the hotfixes.

[!INCLUDE [storsimple-install-update5-hotfix](../../includes/storsimple-install-update5-hotfix.md)]
-->
<!--
[!INCLUDE [storsimple-8000-install-troubleshooting](../../includes/storsimple-8000-install-troubleshooting.md)]
-->

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [выпуске обновления 5,1](storsimple-update51-release-notes.md).

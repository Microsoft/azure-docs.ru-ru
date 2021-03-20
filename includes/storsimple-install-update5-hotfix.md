---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: cbc7c82803115f71db233be94d62a857ba050b63
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86050396"
---
#### <a name="to-download-hotfixes"></a>Загрузка исправления

Для загрузки обновления программного обеспечения из каталога обновления Майкрософт выполните следующие действия.

1. Запустите Internet Explorer и перейдите по адресу [http://catalog.update.microsoft.com](https://catalog.update.microsoft.com) .
2. Если вы впервые используете каталог Центра обновления Майкрософт на этом компьютере, нажмите кнопку **Установить** , когда будет предложено установить надстройку каталога Центра обновления Майкрософт.

    ![Установка каталога](./media/storsimple-install-update2-hotfix/HCS_InstallCatalog-include.png)

3. В поле поиска каталога Центра обновления Майкрософт введите номер необходимого исправления в базе знаний, например **4037264**, а затем нажмите кнопку **Найти**.
   
    Появится список исправлений, например **Совокупное обновление программного обеспечения версии 5.0 для StorSimple серии 8000**.
   
    ![Поиск в каталоге](./media/storsimple-install-update5-hotfix/update-catalog-search.png)

4. Щелкните элемент **Загрузить**. Укажите или **выберите** локальное расположение, в которое хотите скачать файлы. Выберите файлы для загрузки в указанное расположение и папку. Этот каталог также можно скопировать в сетевую папку, которая доступна с вашего устройства.
5. Найдите дополнительные исправления, перечисленные в приведенной выше таблице (**4037266**), и скачайте соответствующие файлы в папки, указанные в этой таблице.

> [!NOTE]
> Исправления должны быть доступны с обоих контроллеров, чтобы можно было получить любые возможные сообщения об ошибке от однорангового контроллера.
>
> Исправления необходимо скопировать в 3 отдельные папки. Например, обновление агента программного обеспечения устройства Cis/MDS необходимо скопировать в папку _FirstOrderUpdate_. Все остальные обновления, не нарушающие работу устройства, можно скопировать в папку _SecondOrderUpdate_, а обновления в режиме обслуживания — в папку _ThirdOrderUpdate_.

#### <a name="to-install-and-verify-regular-mode-hotfixes"></a>Установка и проверка исправлений обычного режима

Выполните следующие действия для установки и проверки обычных исправлений. Если вы уже установили их с помощью портала Azure, перейдите к [установке и проверке исправлений режима обслуживания](#to-install-and-verify-maintenance-mode-hotfixes).

1. Чтобы установить исправления, откройте интерфейс Windows PowerShell на последовательной консоли своего устройства StorSimple. Подробные инструкции см. в разделе [Подключение к последовательной консоли устройства с помощью PuTTY](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console). В командной строке нажмите клавишу **ВВОД**.
2. Выберите вариант 1, **Log in with full access** (Вход с правами полного доступа). Рекомендуем сначала установить исправление на пассивный контроллер.
3. Чтобы установить исправление, в командной строке введите следующую команду:
   
    `Start-HcsHotfix -Path <path to update file> -Credential <credentials in domain\username format>`
   
    В указанной выше команде вместо DNS в адресе общей папки укажите IP-адрес. Параметр "credential" используется только для доступа к общему ресурсу с проверкой подлинности.
   
    Для доступа к общим ресурсам рекомендуем использовать параметр учетных данных. Даже те общие ресурсы, которые доступны для всех, обычно закрыты для пользователей, не прошедших проверку подлинности.
   
4. В ответ на запрос введите пароль. Ниже приведен пример выходных данных для установки обновлений первого типа. Для обновления первого типа необходимо указать определенный файл.

    >[!NOTE] 
    > Сначала установите _HcsSoftwareUpdate.exe_. Затем установите _CisMdsAgentUpdate.exe_.
   
    ```output
    Controller0>Start-HcsHotfix -Path \\10.100.100.100\share
    \FirstOrderUpdate\HcsSoftwareUpdate.exe -Credential contoso\John
   
    Confirm
   
    This operation starts the hotfix installation and could reboot one or
    both of the controllers. If the device is serving I/Os, these will not
    be disrupted. Are you sure you want to continue?
    [Y] Yes [N] No [?] Help (default is "Y"): Y
   
    ```
5. Введите **Y** , когда будет предложено подтвердить установку исправлений.
6. Проверьте обновление с помощью командлета `Get-HcsUpdateStatus` . Сначала обновится пассивный контроллер. После этого будет выполнена отработка отказа. Затем обновление будет применено на другом контроллере. Установка обновления завершится, когда оба контроллера будут обновлены.
   
    Ниже приведен пример выходных данных для обновления, которое выполняется. Пока выполняется обновление, `RunInprogress` имеет значение `True`.

    ```
    Controller0>Get-HcsUpdateStatus
    RunInprogress       : True
    LastHotfixTimestamp :
    LastUpdateTimestamp : 07/28/2017 2:04:02 AM
    Controller0Events   :
    Controller1Events   :
    ```
   
     Ниже приведен пример выходных данных для обновления, установка которого завершена. После того как обновление выполнено, `RunInProgress` принимает значение `False`.
   
    ```
    Controller0>Get-HcsUpdateStatus
    RunInprogress       : False
    LastHotfixTimestamp : 07/28/2017 9:15:55 AM
    LastUpdateTimestamp : 07/28/2017 9:06:07 AM
    Controller0Events   :
    Controller1Events   :
    ```

    > [!NOTE]
    > В некоторых случаях командлет выдает значение `False`, пока обновление еще устанавливается. Чтобы проверить установку обновления, подождите несколько минут, выполните эту команду еще раз и убедитесь в том, что `RunInProgress` имеет значение `False`. Если это так, значит, исправление установлено.

7. Когда установка обновления будет завершена, проверьте версии программного обеспечения системы. Тип:
   
    `Get-HcsSystem`
   
    Номера версий должны быть следующими:
   
   * `FriendlySoftwareVersion: StorSimple 8000 Series Update 5.0`
   * `HcsSoftwareVersion: 6.3.9600.17845`
   
     Если после установки обновления номер версии не изменился, значит исправление установить не удалось. В этом случае обратитесь за дополнительной помощью в [службу поддержки Майкрософт](../articles/storsimple/storsimple-8000-contact-microsoft-support.md).
     
     > [!IMPORTANT]
     > Прежде чем применять следующее обновление, необходимо перезапустить активный контроллер с помощью командлета `Restart-HcsController`.
     
8. Повторите шаги 3–6, чтобы установить агент _CisMDSAgentupdate.exe_, скачанный в папку _FirstOrderUpdate_.
8. Повторите шаги 3–6, чтобы установить обновления второго типа. 

    > [!NOTE] 
    > Можно установить несколько обновлений второго типа, просто запустив командлет `Start-HcsHotfix cmdlet` и указав папку, где они находятся. Командлет выполнит все обновления, доступные в папке. Если обновление уже установлено, логика обновления обнаружит это и не установит его.

    После установки всех исправлений используйте командлет `Get-HcsSystem`. Необходимые версии:
    
    * `CisAgentVersion:  1.0.9724.0`
    * `MdsAgentVersion: 35.2.2.0`
    * `Lsisas2Version: 2.0.78.00`


#### <a name="to-install-and-verify-maintenance-mode-hotfixes"></a>Установка и проверка исправлений режима обслуживания

Используйте KB4037263, чтобы установить обновления встроенного ПО дисков. Эти обновления прерывают работу, и на их завершение может потребоваться около 30 минут. Данные обновления можно установить в период планового техобслуживания, подключившись к последовательной консоли устройства.

> [!NOTE] 
> Если встроенное ПО диска уже обновлено, устанавливать эти обновления не требуется. Выполните командлет `Get-HcsUpdateAvailability` в последовательной консоли устройства, чтобы проверить, доступны ли обновления и являются ли они критическими (режим обслуживания) или некритическими (обычный режим).

Чтобы установить обновления встроенного ПО диска, следуйте приведенным ниже инструкциям.

1. Переведите устройство в режим обслуживания. 

    > [!NOTE] 
    > Не используйте удаленное взаимодействие Windows PowerShell при подключении к устройству, находящемуся в режиме обслуживания. Вместо этого запустите следующий командлет при подключении через последовательную консоль устройства.

    Чтобы установить контроллер в режим обслуживания, введите:
   
    `Enter-HcsMaintenanceMode`
   
    Результат выполнения команды показан ниже.

    ```output
    Controller0>Enter-HcsMaintenanceMode
    Checking device state...
   
    In maintenance mode, your device will not service IOs and will be disconnected from the Microsoft Azure StorSimple Manager service. Entering maintenance mode will end the current session and reboot both controllers, which takes a few minutes to complete. Are you sure you want to enter maintenance mode?
    [Y] Yes [N] No (Default is "Y"): Y
   
    -----------------------MAINTENANCE MODE------------------------
    Microsoft Azure StorSimple Appliance Model 8600
    Name: Update4-8600-mystorsimple
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

    После этого оба контроллера перезапускаются и переходят в режим обслуживания.
2. Чтобы установить обновление встроенного ПО диска, введите:
   
    `Start-HcsHotfix -Path <path to update file> -Credential <credentials in domain\username format>`
   
    Результат выполнения команды показан ниже.

    ```output
    Controller1>Start-HcsHotfix -Path \\10.100.100.100\share\ThirdOrderUpdates\ -Credential contoso\john
    Enter Password:
    WARNING: In maintenance mode, hotfixes should be installed on each controller sequentially. After the hotfix is installed on this controller, install it on the peer controller.
    Confirm
    This operation starts a hotfix installation and could reboot one or both of the controllers. By installing new updates you agree to, and accept any additional terms associated with, the new functionality listed in the release notes (https://go.microsoft.com/fwLink/?LinkID=613790). Are you sure you want to continue?
    [Y] Yes [N] No (Default is "Y"): Y
    WARNING: Installation is currently in progress. This operation can take several minutes to complete.
    ```

3. Отслеживайте ход выполнения установки с помощью команды `Get-HcsUpdateStatus` . Обновление завершится, когда `RunInProgress` поменяется на `False`.
4. После завершения установки контроллер, на котором установлено исправление режима обслуживания, будет перезагружен. Выберите вариант 1, **Log in with full access** (Вход с правами полного доступа), и проверьте версию встроенного ПО диска. Тип:
   
   `Get-HcsFirmwareVersion`
   
   Ожидаемые версии встроенного ПО диска:
   
   `XMGJ, XGEG, KZ50, F6C2, VR08, N003, 0107`
   
   Результат выполнения команды показан ниже.

    ```output
       -----------------------MAINTENANCE MODE------------------------
       Microsoft Azure StorSimple Appliance Model 8600
       Name: Update4-8600-mystorsimple
       Software Version: 6.3.9600.17845
       Copyright (C) 2014 Microsoft Corporation. All rights reserved.
       You are connected to Controller1
       ---------------------------------------------------------------
   
       Controller1>Get-HcsFirmwareVersion
   
       Controller0 : TalladegaFirmware
        ActiveBIOS:0.45.0010
           BackupBIOS:0.45.0006
           MainCPLD:17.0.000b
           ActiveBMCRoot:2.0.001F
           BackupBMCRoot:2.0.001F
           BMCBoot:2.0.0002
           LsiFirmware:20.00.04.00
           LsiBios:07.37.00.00
           Battery1Firmware:06.2C
           Battery2Firmware:06.2C
           DomFirmware:X231600
           CanisterFirmware:3.5.0.56
           CanisterBootloader:5.03
           CanisterConfigCRC:0x9134777A
           CanisterVPDStructure:0x06
           CanisterGEMCPLD:0x19
           CanisterVPDCRC:0x142F7DC2
           MidplaneVPDStructure:0x0C
           MidplaneVPDCRC:0xA6BD4F64
           MidplaneCPLD:0x10
           PCM1Firmware:1.00|1.05
           PCM1VPDStructure:0x05
           PCM1VPDCRC:0x41BEF99C
           PCM2Firmware:1.00|1.05
           PCM2VPDStructure:0x05
           PCM2VPDCRC:0x41BEF99C

        EbodFirmware
           CanisterFirmware:3.5.0.56
           CanisterBootloader:5.03
           CanisterConfigCRC:0xB23150F8
           CanisterVPDStructure:0x06
           CanisterGEMCPLD:0x14
           CanisterVPDCRC:0xBAA55828
           MidplaneVPDStructure:0x0C
           MidplaneVPDCRC:0xA6BD4F64
           MidplaneCPLD:0x10
           PCM1Firmware:3.11
           PCM1VPDStructure:0x03
           PCM1VPDCRC:0x6B58AD13
           PCM2Firmware:3.11
           PCM2VPDStructure:0x03
           PCM2VPDCRC:0x6B58AD13

        DisksFirmware
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           SmrtStor:TXA2D20800GA6XYR:KZ50
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
           WD:WD4001FYYG-01SL3:VR08
    ```
   
    Выполните команду `Get-HcsFirmwareVersion` на втором контроллере, чтобы проверить обновление версии программного обеспечения. Затем можно выйти из режима обслуживания. Для этого введите следующую команду для каждого контроллера устройства:
   
   `Exit-HcsMaintenanceMode`

5. При выходе из режима обслуживания контроллеры перезапускаются. Когда обновления встроенного ПО диска будут установлены, а устройство будет выведено из режима обслуживания, вернитесь на портал Azure. Обратите внимание, что для отображения установленных обновлений режима обслуживания на портале может потребоваться до 24 часов.


---
title: Включение или отключение правила брандмауэра в гостевой ОС на виртуальной машине Azure | Документация Майкрософт
description: Узнайте, как использовать сетевые или автономные средства удаленного доступа или параметры реестра для включения или отключения правил брандмауэра гостевой ОС на удаленной виртуальной машине Azure.
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 11/22/2018
ms.author: delhan
ms.openlocfilehash: 9ee27f429dbfd1e550a45bbc26413a1c259c4fbe
ms.sourcegitcommit: b4e6b2627842a1183fce78bce6c6c7e088d6157b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/30/2021
ms.locfileid: "99092374"
---
# <a name="enable-or-disable-a-firewall-rule-on-an-azure-vm-guest-os"></a>Включение или отключение правила брандмауэра в гостевой ОС на виртуальной машине Azure

Инструкции по устранению неполадок в этой статье относятся к ситуациям, в которых возможна частичная блокировка трафика виртуальной машины брандмауэром гостевой ОС. Это может быть полезным в следующих случаях:

*   Если в брандмауэре намеренно настроены дополнительные параметры, приводящие к сбою подключения по протоколу RDP, функция расширения пользовательских скриптов поможет решить эту проблему.

*   Отключение всех профилей брандмауэра будет более надежным действием для устранения проблемы, чем настройка правил брандмауэра для протокола RDP.

## <a name="solution"></a>Решение

Настройка правил брандмауэра зависит от требуемого уровня доступа к виртуальной машине. Ниже представлены примеры использования правил RDP. Те же самые методы можно применить и для любого другого вида трафика, указав правильный раздел реестра.

### <a name="online-troubleshooting"></a>Сетевое устранение неполадок 

#### <a name="mitigation-1-custom-script-extension"></a>Решение 1. Расширение пользовательских скриптов

1.  Создайте скрипт на основе следующего шаблона.

    *   Чтобы включить правило:
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   Чтобы отключить правило:
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

2.  Отправьте этот скрипт в Azure с помощью функции [расширения пользовательских скриптов](../extensions/custom-script-windows.md). 

#### <a name="mitigation-2-remote-powershell"></a>Решение 2. Удаленный сеанс PowerShell

Если виртуальная машина подключена к сети и к ней есть доступ с другой виртуальной машины в той же виртуальной сети, проблемы можно устранить с помощью второй виртуальной машины.

1.  На виртуальной машине восстановления откройте окно консоли PowerShell.

2.  Выполните следующие команды, внеся необходимые изменения.

    *   Чтобы включить правило:
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Enable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

    *   Чтобы отключить правило:
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Disable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

#### <a name="mitigation-3-pstools-commands"></a>Решение 3. Команды PSTools

Если виртуальная машина подключена к сети и к ней есть доступ с другой виртуальной машины в той же виртуальной сети, проблемы можно устранить с помощью второй виртуальной машины.

1.  На виртуальной машине для устранения неполадок скачайте средство [PSTools](/sysinternals/downloads/pstools).

2.  Откройте экземпляр командной строки и создайте подключение к виртуальной машине по внутреннему (выделенному) IP-адресу. 

    * Чтобы включить правило:
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   Чтобы отключить правило:
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

#### <a name="mitigation-4-remote-registry"></a>Решение 4. Удаленный реестр

Если виртуальная машина подключена к сети и к ней есть доступ с другой виртуальной машины в той же виртуальной сети, можно применить [удаленный реестр](https://www.betaarchive.com/wiki/index.php?title=Microsoft_KB_Archive/314837) на второй виртуальной машине.

1.  На виртуальной машине для устранения неполадок запустите редактор реестра (regedit.exe), а затем выберите **файл**  >  **Подключить сетевой реестр**.

2.  Откройте ветвь \SYSTEM *целевого компьютера* и укажите следующие значения:

    * Чтобы включить правило, откройте следующий параметр реестра:
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
    
        Измените **Active=FALSE** на **Active=TRUE** в следующей строке:

        `v2.22|Action=Allow|Active=TRUE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`
    
    * Чтобы отключить правило, откройте следующий параметр реестра:
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        Измените **Active=TRUE** на **Active=FALSE**:
        
        `v2.22|Action=Allow|Active=FALSE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

3.  Чтобы применить эти изменения, перезапустите виртуальную машину.

### <a name="offline-troubleshooting"></a>Автономное устранение неполадок 

Если доступ к виртуальной машины не удается получить любым методом, расширение пользовательского скрипта не сработает, и проблему придется решать в автономном режиме, то есть прямо через системный диск.

Прежде чем выполнять какие-либо действия, сделайте моментальный снимок диска системы затронутой виртуальной машины в качестве резервной копии. Дополнительные сведения см. в статье [Создание моментального снимка](../windows/snapshot-copy-managed-disk.md).

1.  [Подключите системный диск к виртуальной машине восстановления](troubleshoot-recovery-disks-portal-windows.md).

2.  Установите подключение с помощью удаленного рабочего стола к виртуальной машине, используемой для восстановления.

3.  Убедитесь, что диск помечен как " **подключенный** " в консоли управления дисками. Запишите или запомните букву диска, которая присвоена подключенному системному диску.

4.  Прежде чем вносить любые изменения, создайте копию папки \windows\system32\config на случай, если потребуется откат изменений.

5.  На виртуальной машине для устранения неполадок откройте редактор реестра (regedit.exe).

6.  Выделите ключ **HKEY_LOCAL_MACHINE** , а затем в меню выберите **файл**  >  **Загрузить куст** .

    ![Regedit](./media/enable-or-disable-firewall-rule-guest-os/load-registry-hive.png)

7.  Найдите и откройте файл \windows\system32\config\SYSTEM. 

    > [!Note]
    > Появляется сообщение с предложением ввести имя. Введите **BROKENSYSTEM**, а затем разверните **HKEY_LOCAL_MACHINE**. Теперь появится дополнительный ключ с именем **BROKENSYSTEM**. Для устранения неполадок мы подключив эти проблемы Hive как **BROKENSYSTEM**.

8.  Примените следующие изменения к ветви раздела BROKENSYSTEM.

    1.  Проверьте, с каким значением раздела реестра **ControlSet** запускается виртуальная машина. Номер используемого раздела можно найти в разделе HKLM\BROKENSYSTEM\Select\Current.

    2.  Чтобы включить правило, откройте следующий параметр реестра:
    
        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
        
        Измените здесь **Active=FALSE** на **Active=TRUE**.
        
        `v2.22|Action=Allow|Active=TRUE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

    3.  Чтобы отключить правило, откройте следующий раздел реестра:

        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        Измените **Active=TRUE** на **Active=FALSE**.
        
        `v2.22|Action=Allow|Active=FALSE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

9.  Выделите **BROKENSYSTEM**, а затем в меню выберите **файл**  >  **Выгрузить куст Hive** .

10. [Отсоедините системный диск и повторно создайте виртуальную машину](troubleshoot-recovery-disks-portal-windows.md).

11. Убедитесь, что проблема устранена.

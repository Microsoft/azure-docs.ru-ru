---
title: Внутренняя ошибка при попытке подключения к Виртуальным машинам Azure по протоколу удаленного рабочего стола | Документация Майкрософт
description: Узнайте, как устранить внутренние ошибки RDP в Microsoft Azure. | Документация Майкрософт
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/22/2018
ms.author: genli
ms.openlocfilehash: 5c8bd335832a950385f88f13dc31eb7f6159f831
ms.sourcegitcommit: 5a999764e98bd71653ad12918c09def7ecd92cf6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/16/2021
ms.locfileid: "100548137"
---
#  <a name="an-internal-error-occurs-when-you-try-to-connect-to-an-azure-vm-through-remote-desktop"></a>Внутренняя ошибка при попытке подключения к виртуальной Машине Azure через удаленный рабочий стол

В этой статье описаны ошибки, которые могут возникнуть при подключении к виртуальной машине в Microsoft Azure.


## <a name="symptoms"></a>Симптомы

Вы не можете подключиться к виртуальной машине Azure с помощью протокола удаленного рабочего стола (RDP). Подключение зависает при **настройке удаленного** раздела или появляется следующее сообщение об ошибке:

- внутренняя ошибка RDP;
- произошла внутренняя ошибка;
- не удается подключиться к удаленному компьютеру с этого компьютера. Повторите попытку подключения. Если проблема не исчезнет, ​​обратитесь к владельцу удаленного компьютера или к администратору сети.


## <a name="cause"></a>Причина

Эта проблема может возникать по следующим причинам.

- Возможно, виртуальная машина была атакована.
- Не удается получить доступ к локальным ключам шифрования RSA.
- протокол TLS отключен;
- сертификат поврежден или истек срок его действия.

## <a name="solution"></a>Решение

Чтобы устранить эту проблему, выполните действия, описанные в следующих разделах. Прежде чем начать, Создайте моментальный снимок диска операционной системы затронутой виртуальной машины в качестве резервной копии. Дополнительные сведения см. в статье [Создание моментального снимка](../windows/snapshot-copy-managed-disk.md).

### <a name="check-rdp-security"></a>Проверка безопасности RDP

Сначала убедитесь, что группа безопасности сети для порта RDP 3389 не защищена (открыта). Если он не защищен и отображается \* как исходный IP-адрес для входящего трафика, ограничьте порт RDP IP-адресом пользователя спеЦифк, а затем проверьте доступ по протоколу RDP. Если это не удается, выполните действия, описанные в следующем разделе.

### <a name="use-serial-control"></a>Использование последовательной консоли

Используйте последовательную консоль или [восстановите виртуальную машину в автономном режиме](#repair-the-vm-offline) , подключив диск операционной системы виртуальной машины к виртуальной машине восстановления.

Чтобы начать, подключитесь к [последовательной консоли и откройте экземпляр PowerShell](./serial-console-windows.md#use-cmd-or-powershell-in-serial-console
). Если последовательную консоль не включено на виртуальной машине, перейдите к разделу [repair the VM offline](#repair-the-vm-offline)(Автономное восстановление виртуальной машины).

#### <a name="step-1-check-the-rdp-port"></a>Шаг 1. Проверка порта RDP

1. В экземпляре PowerShell используйте команду [netstat](/windows-server/administration/windows-commands/netstat) , чтобы проверить, используется ли порт 3389 другими приложениями:

    ```powershell
    Netstat -anob |more
    ```

2. Если Termservice.exe использует порт 3389, перейдите к шагу 2. Если для другой службы или приложения, кроме Termservice.exe, используется порт 3389, выполните следующие действия.

    1. Остановите службу для приложения, которое использует службу 3389.

        ```powershell
        Stop-Service -Name <ServiceName> -Force
        ```

    2. Запустите службу терминалов.

        ```powershell
        Start-Service -Name Termservice
        ```

2. Если не удается остановить приложение, или этот метод не подходит, измените порт для RDP.

    1. Измените порт.

        ```powershell
        Set-ItemProperty -Path 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name PortNumber -value <Hexportnumber>

        Stop-Service -Name Termservice -Force

        Start-Service -Name Termservice
        ```

    2. Настройте брандмауэр в соответствии с новым портом.

        ```powershell
        Set-NetFirewallRule -Name "RemoteDesktop-UserMode-In-TCP" -LocalPort <NEW PORT (decimal)>
        ```

    3. [Обновите группу безопасности сети для нового порта](../../virtual-network/network-security-groups-overview.md) на порте RDP портала Azure.

#### <a name="step-2-set-correct-permissions-on-the-rdp-self-signed-certificate"></a>Шаг 2. Установка правильных разрешений на самозаверяющем сертификате RDP

1. Чтобы обновить самозаверяющий сертификат RDP, по очереди выполните следующие команды в экземпляре PowerShell.

    ```powershell
    Import-Module PKI

    Set-Location Cert:\LocalMachine 

    $RdpCertThumbprint = 'Cert:\LocalMachine\Remote Desktop\'+((Get-ChildItem -Path 'Cert:\LocalMachine\Remote Desktop\').thumbprint) 

    Remove-Item -Path $RdpCertThumbprint

    Stop-Service -Name "SessionEnv"

    Start-Service -Name "SessionEnv"
    ```

2. Если не удается обновить сертификат с помощью этого метода, попробуйте обновить самозаверяющий сертификат RDP удаленно.

    1. С работающей виртуальной машины с подключением к той виртуальной машине, на которой возникли проблемы, введите **mmc** в окне **Запуск**, чтобы открыть консоль управления (MMC).
    2. В меню **Файл** выберите **Add/Remove Snap-in** (Добавить или удалить оснастку), выберите **Сертификаты**, а затем выберите **Добавить**.
    3. Выберите **Computer accounts** (учетные записи компьютера), выберите **Another Computer** (другой компьютер), а затем добавьте IP-адрес проблемной виртуальной машины.
    4. Перейдите в папку **Remote Desktop\Certificates** (Удаленный рабочий стол или сертификаты), щелкните правой кнопкой мыши сертификат и затем щелкните **Удалить**.
    5. В экземпляре PowerShell из последовательной консоли перезапустите службу настройки удаленного рабочего стола.

        ```powershell
        Stop-Service -Name "SessionEnv"

        Start-Service -Name "SessionEnv"
        ```

3. Сбросьте разрешение для папки MachineKeys

    ```powershell
    remove-module psreadline 

    md c:\temp

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\BeforeScript_permissions.txt 

    takeown /f "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys" /a /r

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\System:(F)"

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\NETWORK SERVICE:(R)"

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"

    icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\AfterScript_permissions.txt 

    Restart-Service TermService -Force
    ```

4. Перезапустите виртуальную машину, а затем повторите попытку подключения к виртуальной машине с удаленного рабочего стола. Если ошибку не удалось устранить, перейдите к следующему шагу.

#### <a name="step-3-enable-all-supported-tls-versions"></a>Шаг 3. Включение всех поддерживаемых версий протокола TLS

Клиент RDP использует TLS 1.0 в качестве протокола по умолчанию. Тем не менее его можно изменить на TLS 1.1, который является новым стандартом. Если протокол TLS 1.1 отключен на виртуальной машине, произойдет сбой подключения.

1. В экземпляре CMD включите протокол TLS.

    ```console
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f
    ```

2. Чтобы предотвратить перезапись изменений политики AD, временно остановите обновление групповой политики.

    ```console
    REG add "HKLM\SYSTEM\CurrentControlSet\Services\gpsvc" /v Start /t REG_DWORD /d 4 /f
    ```

3. Чтобы изменения вступили в силу, перезапустите виртуальную машину. Если проблема устранена, выполните следующую команду, чтобы снова включить групповую политику.

    ```console
    sc config gpsvc start= auto sc start gpsvc

    gpupdate /force
    ```

    Если изменение отменено, это означает, что в домене вашей компании уже имеется политика Active Directory. Чтобы впредь избежать этой проблемы, необходимо изменить эту политику.

### <a name="repair-the-vm-offline"></a>Автономное восстановление виртуальной машины

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Подключите диск ОС к виртуальной машине восстановления.

1. [Подключите диск операционной системы к виртуальной машине восстановления](./troubleshoot-recovery-disks-portal-windows.md).
2. После подключения диска ОС к виртуальной машине восстановления убедитесь, что в консоли управления дисками он помечен как **В сети**. Запишите или запомните букву диска, которая присвоена подключенному диску ОС.
3. Установите подключение с помощью удаленного рабочего стола к виртуальной машине, используемой для восстановления.

#### <a name="enable-dump-log-and-serial-console"></a>Включение журнала дампа и последовательной консоли

Чтобы включить журнал дампа и последовательную консоль, выполните следующий сценарий.

1. Откройте сеанс командной строки с повышенными привилегиями (**Запуск от имени администратора**).
2. Выполните следующий сценарий:

    В этом сценарии мы предполагаем, что подключенному диску ОС присвоена буква F. Замените ее соответствующим значением для своей виртуальной машины.

    ```console
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REM Enable Serial Console
    bcdedit /store F:\boot\bcd /set {bootmgr} displaybootmenu yes
    bcdedit /store F:\boot\bcd /set {bootmgr} timeout 5
    bcdedit /store F:\boot\bcd /set {bootmgr} bootems yes
    bcdedit /store F:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store F:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200

    REM Suggested configuration to enable OS Dump
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    reg unload HKLM\BROKENSYSTEM
    ```

#### <a name="reset-the-permission-for-machinekeys-folder"></a>Сброс разрешения для папки MachineKeys

1. Откройте сеанс командной строки с повышенными привилегиями (**Запуск от имени администратора**).
2. Выполните следующий сценарий. В этом сценарии мы предполагаем, что подключенному диску ОС присвоена буква F. Замените ее соответствующим значением для своей виртуальной машины.

    ```console
    Md F:\temp

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\BeforeScript_permissions.txt

    takeown /f "F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys" /a /r

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\System:(F)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\NETWORK SERVICE:(R)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"

    icacls F:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\AfterScript_permissions.txt
    ```

#### <a name="enable-all-supported-tls-versions"></a>Шаг 3. Включение всех поддерживаемых версий протокола TLS

1. Отройте сеанс командной строки с повышенными привилегиями (**Запуск от имени администратора**) и выполните приведенные ниже команды. В этом сценарии мы предполагаем, что подключенному диску ОС присвоена буква F. Замените ее соответствующим значением для своей виртуальной машины.
2. Проверьте, который протокол TLS включен.

    ```console
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWO
    ```

3. Если ключ не существует, или его значение **0**, включите протокол, выполнив следующие сценарии.

    ```console
    REM Enable TLS 1.0, TLS 1.1 and TLS 1.2

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f
    ```

4. Включите NLA.

    ```console
    REM Enable NLA

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f reg unload HKLM\BROKENSYSTEM
    ```

5. [Отключите диск ОС и повторно создайте виртуальную машину](./troubleshoot-recovery-disks-portal-windows.md), а затем проверьте, устранена ли проблема.

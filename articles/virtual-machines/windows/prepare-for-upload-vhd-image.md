---
title: Подготовка виртуального жесткого диска Windows к передаче в Azure
description: Узнайте, как подготовить VHD или VHDX Windows к отправке в Azure.
author: glimoli
manager: dcscontentpm
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.workload: infrastructure-services
ms.topic: troubleshooting
ms.date: 09/02/2020
ms.author: genli
ms.openlocfilehash: 12ef839cbbbc69230b314bf7c56a63f57a0d6b20
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102556270"
---
# <a name="prepare-a-windows-vhd-or-vhdx-to-upload-to-azure"></a>Подготовка диска VHD или VHDX для Windows к отправке в Azure

Перед отправкой виртуальной машины Windows из локальной среды в Azure необходимо подготовить виртуальный жесткий диск (VHD или VHDX). Azure поддерживает виртуальные машины поколения 1 и 2, которые имеют формат VHD-файла и имеют диск фиксированного размера. Максимально допустимый размер виртуального жесткого диска для ОС на виртуальной машине поколения 1 составляет 2 ТБ.

Вы можете преобразовать VHDX-файл в VHD, преобразовать динамически расширяемый диск в диск фиксированного размера, но нельзя изменить поколение виртуальной машины. Дополнительные сведения см. [в статье Создание виртуальной машины поколения 1 или 2 в Hyper-V](/windows-server/virtualization/hyper-v/plan/Should-I-create-a-generation-1-or-2-virtual-machine-in-Hyper-V) и [Поддержка виртуальных машин поколения 2 в Azure](../generation-2.md).

Сведения о политике поддержки для виртуальных машин Azure см. в [статье поддержка серверного программного обеспечения Майкрософт для виртуальных машин Azure](https://support.microsoft.com/help/2721672/).

> [!NOTE]
> Инструкции, приведенные в этой статье, относятся к:
>
> - 64-разрядная версия операционных систем Windows Server 2008 R2 и более поздних версий. Сведения о выполнении 32-разрядной операционной системы в Azure см. [в статье поддержка 32-разрядных операционных систем на виртуальных машинах Azure](https://support.microsoft.com/help/4021388/).
> - Если для миграции рабочей нагрузки будет использоваться любое средство аварийного восстановления, например Azure Site Recovery или миграция Azure, этот процесс по-прежнему потребуется в гостевой ОС для подготовки образа перед миграцией.

## <a name="system-file-checker"></a> средство проверки системных файлов

### <a name="run-windows-system-file-checker-utility-before-generalization-of-os-image"></a>Запустите программу проверки системных файлов Windows перед обобщением образа ОС

Средство проверки системных файлов (SFC) используется для проверки и замены системных файлов Windows.

> [!IMPORTANT]
> Используйте сеанс PowerShell с повышенными привилегиями для выполнения примеров в этой статье.

Выполните команду SFC:

```powershell
sfc.exe /scannow
```

```Output
Beginning system scan.  This process will take some time.

Beginning verification phase of system scan.
Verification 100% complete.

Windows Resource Protection did not find any integrity violations.
```

После завершения проверки SFC установите обновления Windows и перезагрузите компьютер.

## <a name="set-windows-configurations-for-azure"></a>Настройка конфигурации Windows для Azure

> [!NOTE]
> Платформа Azure подключает файл ISO к DVD-ROM при создании виртуальной машины Windows из обобщенного образа. По этой причине DVD-диск должен быть включен в ОС в обобщенном образе. Если она отключена, виртуальная машина Windows будет заблокирована при первом включении (OOBE).

1. Удалите все статические постоянные маршруты в таблице маршрутизации.

   - Чтобы просмотреть таблицу маршрутизации, выполните команду `route.exe print` .
   - Ознакомьтесь с разделом **маршруты сохраняемости** . Если имеется постоянный маршрут, `route.exe delete` удалите его с помощью команды.

1. Удалите прокси-сервер WinHTTP.

   ```powershell
   netsh.exe winhttp reset proxy
   ```

    Если виртуальная машина должна работать с конкретным прокси-сервером, добавьте исключение прокси для IP-адреса Azure ([168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md)), чтобы виртуальная машина могла подключаться к Azure:

    ```
    $proxyAddress='<your proxy server>'
    $proxyBypassList='<your list of bypasses>;168.63.129.16'
    netsh.exe winhttp set proxy $proxyAddress $proxyBypassList
    ```

1. Откройте программу DiskPart:

   ```powershell
   diskpart.exe
   ```

   Задайте для политики SAN диска значение [`Onlineall`](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/gg252636(v=ws.11)) :

   ```DiskPart
   DISKPART> san policy=onlineall
   DISKPART> exit
   ```

1. Установить время в формате UTC для Windows. Кроме того, задайте **Автоматический** тип запуска службы времени Windows **W32Time** :

   ```powershell
   Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation -Name RealTimeIsUniversal -Value 1 -Type DWord -Force
   Set-Service -Name w32time -StartupType Automatic
   ```

1. Установите для профиля питания высокую производительность:

   ```powershell
   powercfg.exe /setactive SCHEME_MIN
   ```

1. Убедитесь, что для переменных среды **TEMP** и **tmp** заданы значения по умолчанию:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -Name TEMP -Value "%SystemRoot%\TEMP" -Type ExpandString -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -Name TMP -Value "%SystemRoot%\TEMP" -Type ExpandString -Force
   ```

## <a name="check-the-windows-services"></a>Настройка служб Windows

Убедитесь, что для каждой из следующих служб Windows задано значение Windows по умолчанию. Эти службы являются минимальными, которые необходимо настроить для обеспечения подключения к виртуальной машине. Чтобы задать параметры запуска, выполните следующий пример:

```powershell
Get-Service -Name BFE, Dhcp, Dnscache, IKEEXT, iphlpsvc, nsi, mpssvc, RemoteRegistry |
  Where-Object StartType -ne Automatic |
    Set-Service -StartupType Automatic

Get-Service -Name Netlogon, Netman, TermService |
  Where-Object StartType -ne Manual |
    Set-Service -StartupType Manual
```

## <a name="update-remote-desktop-registry-settings"></a>Обновление параметров реестра для удаленного рабочего стола

Убедитесь, что для удаленного доступа правильно настроены следующие параметры:

> [!NOTE]
> Если при запуске появляется сообщение об ошибке `Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services -Name <string> -Value <object>` , его можно спокойно проигнорировать. Это означает, что домен не задает эту конфигурацию с помощью объекта групповая политика.

1. Включите протокол удаленного рабочего стола (RDP):

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   ```

1. Порт RDP настроен правильно, используя порт по умолчанию 3389:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name PortNumber -Value 3389 -Type DWord -Force
   ```

   При развертывании виртуальной машины создаются правила по умолчанию для порта 3389. Чтобы изменить номер порта, сделайте это после развертывания виртуальной машины в Azure.

1. Прослушиватель прослушивает каждый сетевой интерфейс:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name LanAdapter -Value 0 -Type DWord -Force
   ```

1. Настройте режим проверки подлинности на уровне сети (NLA) для подключений по протоколу RDP:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name UserAuthentication -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name SecurityLayer -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name fAllowSecProtocolNegotiation -Value 1 -Type DWord -Force
   ```

1. Задайте значение для проверки активности:

   ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveEnable -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveInterval -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name KeepAliveTimeout -Value 1 -Type DWord -Force
   ```

1. Задайте параметры повторного подключения:

   ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDisableAutoReconnect -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fInheritReconnectSame -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fReconnectSame -Value 0 -Type DWord -Force
   ```

1. Ограничьте количество одновременных подключений:

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name MaxInstanceCount -Value 4294967295 -Type DWord -Force
   ```

1. Удалите все самозаверяющие сертификаты, привязанные к прослушивателю RDP:

   ```powershell
   if ((Get-Item -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp').Property -contains 'SSLCertificateSHA1Hash')
   {
       Remove-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name SSLCertificateSHA1Hash -Force
   }
   ```

   Этот код гарантирует, что вы сможете подключиться при развертывании виртуальной машины. Эти параметры также можно просматривать после развертывания виртуальной машины в Azure.

1. Если виртуальная машина является частью домена, проверьте следующие политики, чтобы убедиться, что предыдущие параметры не отменены.

    |                 Цель                  |                                                                            Политика                                                                            |                           Значение                            |
    | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
    | Включение RDP                        | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Компоненты\Службы удаленных рабочих столов\Узел сеансов удаленных рабочих столов\Подключения         | Разрешить пользователям удаленное подключение с использованием служб удаленных рабочих столов    |
    | Групповая политика с проверкой подлинности на уровне сети                      | Параметры\Административные шаблоны\Компоненты\Службы удаленных рабочих столов\Узел сеансов удаленных рабочих столов\Безопасность                                                    | Требовать проверку подлинности пользователя для удаленного доступа с помощью NLA |
    | Параметры сохранения активности                   | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Компоненты Windows\Службы удаленных рабочих столов\Узел сеансов удаленных рабочих столов\Подключения | Настроить интервал проверяемых на активность подключений                   |
    | Параметры переподключения                    | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Компоненты Windows\Службы удаленных рабочих столов\Узел сеансов удаленных рабочих столов\Подключения | Автоматическое подключение                                    |
    | Ограниченное число параметров подключения | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Компоненты Windows\Службы удаленных рабочих столов\Узел сеансов удаленных рабочих столов\Подключения | Ограничить количество подключений                                |

## <a name="configure-windows-firewall-rules"></a>Настройка правил брандмауэра Windows

1. Включите брандмауэр Windows в трех профилях (домен, Стандартный и общедоступный):

   ```powershell
   Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled True
   ```

1. Выполните следующий пример, чтобы разрешить WinRM через три профиля брандмауэра (домен, частный и общедоступный) и включить удаленную службу PowerShell:

   ```powershell
   Enable-PSRemoting -Force
   Set-NetFirewallRule -DisplayName 'Windows Remote Management (HTTP-In)' -Enabled True
   ```

1. Включите следующие правила брандмауэра, чтобы разрешить трафик RDP:

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Desktop' -Enabled True
   ```

1. Включите правило для общего доступа к файлам и принтерам, чтобы виртуальная машина могла отвечать на запросы проверки связи в виртуальной сети:

   ```powershell
   Set-NetFirewallRule -DisplayName 'File and Printer Sharing (Echo Request - ICMPv4-In)' -Enabled True
   ```

1. Создайте правило для сети платформы Azure.

   ```powershell
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Inbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow -EdgeTraversalPolicy Allow
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Outbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow
   ```

1. Если виртуальная машина является частью домена, проверьте следующие политики Azure AD, чтобы убедиться, что предыдущие параметры не отменены.

    |                 Цель                 |                                                                         Политика                                                                          |                  Значение                  |
    | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
    | Включите профили брандмауэра Windows: | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Сеть\Сетевое подключение\Брандмауэр Windows\Профиль домена\Брандмауэр Windows   | Защита всех сетевых подключений         |
    | Включите RDP:                           | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Сеть\Сетевое подключение\Брандмауэр Windows\Профиль домена\Брандмауэр Windows   | Разрешить исключения для входящих сообщений удаленного управления рабочим столом |
    |                                      | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Сеть\Сетевое подключение\Брандмауэр Windows\Стандартный профиль\Брандмауэр Windows | Разрешить исключения для входящих сообщений удаленного управления рабочим столом |
    | Включите ICMP-V4:                       | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Сеть\Сетевое подключение\Брандмауэр Windows\Профиль домена\Брандмауэр Windows   | Разрешить исключения ICMP                   |
    |                                      | Конфигурация компьютера\Политики\Параметры Windows\Административные шаблоны\Сеть\Сетевое подключение\Брандмауэр Windows\Стандартный профиль\Брандмауэр Windows | Разрешить исключения ICMP                   |

## <a name="verify-the-vm"></a>Проверка виртуальной машины

Убедитесь, что виртуальная машина работоспособна, защищена и RDP доступна:

1. Чтобы убедиться, что диск является работоспособным и последовательным, проверьте диск при следующей перезагрузке виртуальной машины:

   ```powershell
   chkdsk.exe /f
   ```

   Убедитесь, что в отчете отображается чистый и работоспособный диск.

1. Задайте параметры данных конфигурации загрузки.

   ```powershell
   bcdedit.exe /set "{bootmgr}" integrityservices enable
   bcdedit.exe /set "{default}" device partition=C:
   bcdedit.exe /set "{default}" integrityservices enable
   bcdedit.exe /set "{default}" recoveryenabled Off
   bcdedit.exe /set "{default}" osdevice partition=C:
   bcdedit.exe /set "{default}" bootstatuspolicy IgnoreAllFailures

   #Enable Serial Console Feature
   bcdedit.exe /set "{bootmgr}" displaybootmenu yes
   bcdedit.exe /set "{bootmgr}" timeout 5
   bcdedit.exe /set "{bootmgr}" bootems yes
   bcdedit.exe /ems "{current}" ON
   bcdedit.exe /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```

1. Журнал дампа может быть полезен при устранении неполадок, связанных с аварийным завершением работы Windows. Включите сбор журналов дампа:

   ```powershell
   # Set up the guest OS to collect a kernel dump on an OS crash event
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name CrashDumpEnabled -Type DWord -Force -Value 2
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name DumpFile -Type ExpandString -Force -Value "%SystemRoot%\MEMORY.DMP"
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name NMICrashDump -Type DWord -Force -Value 1

   # Set up the guest OS to collect user mode dumps on a service crash event
   $key = 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps'
   if ((Test-Path -Path $key) -eq $false) {(New-Item -Path 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting' -Name LocalDumps)}
   New-ItemProperty -Path $key -Name DumpFolder -Type ExpandString -Force -Value 'C:\CrashDumps'
   New-ItemProperty -Path $key -Name CrashCount -Type DWord -Force -Value 10
   New-ItemProperty -Path $key -Name DumpType -Type DWord -Force -Value 2
   Set-Service -Name WerSvc -StartupType Manual
   ```

1. Убедитесь, что репозиторий инструментарий управления Windows (WMI) (WMI) является согласованным:

   ```powershell
   winmgmt.exe /verifyrepository
   ```

   Если репозиторий поврежден, см. раздел [WMI: повреждение репозитория](https://techcommunity.microsoft.com/t5/ask-the-performance-team/wmi-repository-corruption-or-not/ba-p/375484).

1. Убедитесь, что никакое другое приложение не использует порт 3389. Этот порт используется для службы RDP в Azure. Чтобы узнать, какие порты используются на виртуальной машине, выполните `netstat.exe -anob` :

   ```powershell
   netstat.exe -anob
   ```

1. Чтобы отправить виртуальный жесткий диск Windows, который является контроллером домена, выполните следующие действия.

   - Выполните [эти дополнительные шаги](https://support.microsoft.com/kb/2904015), чтобы подготовить диск.

   - Убедитесь, что вы знаете пароль режима восстановления служб каталогов (DSRM), если вам когда-либо потребуется запустить виртуальную машину в режиме DSRM. Дополнительные сведения см. [в разделе Установка пароля DSRM](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc754363(v=ws.11)).

1. Убедитесь, что вы знаете встроенную учетную запись администратора и пароль. Вам может потребоваться сбросить текущий пароль локального администратора и убедиться, что вы можете использовать эту учетную запись для входа в Windows через подключение по протоколу RDP. Это разрешение на доступ управляется объектом групповая политика "разрешить вход через службы удаленных рабочих столов". Просмотрите этот объект в редактор локальных групповых политик:

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights Assignment`

1. Проверьте следующие политики Azure AD, чтобы убедиться, что они не блокируют доступ по протоколу RDP:

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights
      Assignment\Deny access to this computer from the network`

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights
      Assignment\Deny log on through Remote Desktop Services`

1. Проверьте следующую политику Azure AD, чтобы убедиться, что они не удаляют ни одну из необходимых учетных записей доступа:

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights Assignment\Access this computer from the network`

   Политика должна перечислять следующие группы:

   - Администраторы

   - Операторы архива

   - Все

   - Пользователи

1. Перезапустите виртуальную машину, чтобы убедиться в работоспособности Windows и доступе через RDP-подключение. На этом этапе рекомендуется создать виртуальную машину на локальном сервере Hyper-V, чтобы убедиться, что виртуальная машина запускается полностью. Затем протестируйте, чтобы убедиться, что вы можете подключиться к виртуальной машине по протоколу RDP.

1. Удалите все лишние фильтры TDI (TDI). Например, удалите программное обеспечение, которое анализирует TCP-пакеты или дополнительные брандмауэры.

1. Удалите все другие сторонние программы или драйверы, связанные с физическими компонентами или любыми другими технологиями виртуализации.

### <a name="install-windows-updates"></a>Установка обновлений Windows

В идеале следует следить за тем, чтобы компьютер был обновлен до *уровня обновления*. Если это невозможно, убедитесь, что установлены следующие обновления. Чтобы получить последние обновления, см. страницы журнала центра обновления Windows: [Windows 10 и Windows server 2019](https://support.microsoft.com/help/4000825), [Windows 8.1, Windows Server 2012 R2](https://support.microsoft.com/help/4009470) и Windows [7 SP1 и Windows Server 2008 R2 с пакетом обновления 1 (SP1)](https://support.microsoft.com/help/4009469).

<br />

|        Компонент        |     Двоичные данные     | Windows 7 SP1, Windows Server 2008 R2 SP1 |       Windows 8, Windows Server 2012        | Windows 8.1, Windows Server 2012 R2 | Windows 10 v1607, Windows Server 2016 v1607 |      Windows 10 версии 1703      | Windows 10 v1709, Windows Server 2016 v1709 | Windows 10 v1803, Windows Server 2016 v1803 |
| ----------------------- | -------------- | ----------------------------------------- | ------------------------------------------- | ----------------------------------- | ------------------------------------------- | -------------------------- | ------------------------------------------- | ------------------------------------------- |
| Память                 | disk.sys       | 6.1.7601.23403 — KB3125574                | 6.2.9200.17638 / 6.2.9200.21757 — KB3137061 | 6.3.9600.18203 — KB3137061          | -                                           | -                          | -                                           | -                                           |
|                         | storport.sys   | 6.1.7601.23403 — KB3125574                | 6.2.9200.17188 / 6.2.9200.21306 — KB3018489 | 6.3.9600.18573 — KB4022726          | 10.0.14393.1358 — KB4022715                 | 10.0.15063.332             | -                                           | -                                           |
|                         | ntfs.sys       | 6.1.7601.23403 — KB3125574                | 6.2.9200.17623 / 6.2.9200.21743 — KB3121255 | 6.3.9600.18654 — KB4022726          | 10.0.14393.1198 — KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | Iologmsg.dll   | 6.1.7601.23403 — KB3125574                | 6.2.9200.16384 — KB2995387                  | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | Classpnp.sys   | 6.1.7601.23403 — KB3125574                | 6.2.9200.17061 / 6.2.9200.21180 — KB2995387 | 6.3.9600.18334 — KB3172614          | 10.0.14393.953 — KB4022715                  | -                          | -                                           | -                                           |
|                         | Volsnap.sys    | 6.1.7601.23403 — KB3125574                | 6.2.9200.17047 / 6.2.9200.21165 — KB2975331 | 6.3.9600.18265 — KB3145384          | -                                           | 10.0.15063.0               | -                                           | -                                           |
|                         | partmgr.sys    | 6.1.7601.23403 — KB3125574                | 6.2.9200.16681 — KB2877114                  | 6.3.9600.17401 — KB3000850          | 10.0.14393.953 — KB4022715                  | 10.0.15063.0               | -                                           | -                                           |
|                         | volmgr.sys     |                                           |                                             |                                     |                                             | 10.0.15063.0               | -                                           | -                                           |
|                         | Volmgrx.sys    | 6.1.7601.23403 — KB3125574                | -                                           | -                                   | -                                           | 10.0.15063.0               | -                                           | -                                           |
|                         | Msiscsi.sys    | 6.1.7601.23403 — KB3125574                | 6.2.9200.21006 — KB2955163                  | 6.3.9600.18624 — KB4022726          | 10.0.14393.1066 — KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | Msdsm.sys      | 6.1.7601.23403 — KB3125574                | 6.2.9200.21474 — KB3046101                  | 6.3.9600.18592 — KB4022726          | -                                           | -                          | -                                           | -                                           |
|                         | Mpio.sys       | 6.1.7601.23403 — KB3125574                | 6.2.9200.21190 — KB3046101                  | 6.3.9600.18616 — KB4022726          | 10.0.14393.1198 — KB4022715                 | -                          | -                                           | -                                           |
|                         | vmstorfl.sys   | 6.3.9600.18907 — KB4072650                | 6.3.9600.18080 — KB3063109                  | 6.3.9600.18907 — KB4072650          | 10.0.14393.2007 — KB4345418                 | 10.0.15063.850 — KB4345419 | 10.0.16299.371 — KB4345420                  | -                                           |
|                         | Fveapi.dll     | 6.1.7601.23311 — KB3125574                | 6.2.9200.20930 — KB2930244                  | 6.3.9600.18294 — KB3172614          | 10.0.14393.576 — KB4022715                  | -                          | -                                           | -                                           |
|                         | Fveapibase.dll | 6.1.7601.23403 — KB3125574                | 6.2.9200.20930 — KB2930244                  | 6.3.9600.17415 — KB3172614          | 10.0.14393.206 — KB4022715                  | -                          | -                                           | -                                           |
| Сеть                 | netvsc.sys     | -                                         | -                                           | -                                   | 10.0.14393.1198 — KB4022715                 | 10.0.15063.250 — KB4020001 | -                                           | -                                           |
|                         | mrxsmb10.sys   | 6.1.7601.23816 — KB4022722                | 6.2.9200.22108 — KB4022724                  | 6.3.9600.18603 — KB4022726          | 10.0.14393.479 — KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | mrxsmb20.sys   | 6.1.7601.23816 — KB4022722                | 6.2.9200.21548 — KB4022724                  | 6.3.9600.18586 — KB4022726          | 10.0.14393.953 — KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | mrxsmb.sys     | 6.1.7601.23816 — KB4022722                | 6.2.9200.22074 — KB4022724                  | 6.3.9600.18586 — KB4022726          | 10.0.14393.953 — KB4022715                  | 10.0.15063.0               | -                                           | -                                           |
|                         | tcpip.sys      | 6.1.7601.23761 — KB4022722                | 6.2.9200.22070 — KB4022724                  | 6.3.9600.18478 — KB4022726          | 10.0.14393.1358 — KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | http.sys       | 6.1.7601.23403 — KB3125574                | 6.2.9200.17285 — KB3042553                  | 6.3.9600.18574 — KB4022726          | 10.0.14393.251 — KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | vmswitch.sys   | 6.1.7601.23727 — KB4022719                | 6.2.9200.22117 — KB4022724                  | 6.3.9600.18654 — KB4022726          | 10.0.14393.1358 — KB4022715                 | 10.0.15063.138             | -                                           | -                                           |
| Основные сведения                    | ntoskrnl.exe   | 6.1.7601.23807 — KB4022719                | 6.2.9200.22170 — KB4022718                  | 6.3.9600.18696 — KB4022726          | 10.0.14393.1358 — KB4022715                 | 10.0.15063.483             | -                                           | -                                           |
| Службы удаленных рабочих столов | rdpcorets.dll  | 6.2.9200.21506 — KB4022719                | 6.2.9200.22104 — KB4022724                  | 6.3.9600.18619 — KB4022726          | 10.0.14393.1198 — KB4022715                 | 10.0.15063.0               | -                                           | -                                           |
|                         | termsrv.dll    | 6.1.7601.23403 — KB3125574                | 6.2.9200.17048 — KB2973501                  | 6.3.9600.17415 — KB3000850          | 10.0.14393.0 — KB4022715                    | 10.0.15063.0               | -                                           | -                                           |
|                         | termdd.sys     | 6.1.7601.23403 — KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | win32k.sys     | 6.1.7601.23807 — KB4022719                | 6.2.9200.22168 — KB4022718                  | 6.3.9600.18698 — KB4022726          | 10.0.14393.594 — KB4022715                  | -                          | -                                           | -                                           |
|                         | rdpdd.dll      | 6.1.7601.23403 — KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | rdpwd.sys      | 6.1.7601.23403 — KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
| Безопасность                | MS17-010       | KB4012212                                 | KB4012213                                   | KB4012213                           | KB4012606                                   | KB4012606                  | -                                           | -                                           |
|                         |                |                                           | KB4012216                                   |                                     | KB4013198                                   | KB4013198                  | -                                           | -                                           |
|                         |                | KB4012215                                 | KB4012214                                   | KB4012216                           | KB4013429                                   | KB4013429                  | -                                           | -                                           |
|                         |                |                                           | KB4012217                                   |                                     | KB4013429                                   | KB4013429                  | -                                           | -                                           |
|                         | CVE-2018-0886  | KB4103718                                 | KB4103730                                   | KB4103725                           | KB4103723                                   | KB4103731                  | KB4103727                                   | KB4103721                                   |
|                         |                | KB4103712                                 | KB4103726                                   | KB4103715                           |                                             |                            |                                             |                                             |

> [!NOTE]
> Чтобы избежать случайной перезагрузки во время подготовки виртуальной машины, рекомендуется убедиться, что все Центр обновления Windows установки завершены и обновления не ожидаются. Один из способов сделать это — установить все возможные обновления Windows и перезагрузить один раз перед выполнением `sysprep.exe` команды.

## <a name="determine-when-to-use-sysprep"></a>Определение необходимости использования Sysprep

Средство подготовки системы ( `sysprep.exe` ) — это процесс, который можно выполнить для сброса установки Windows.
Средство Sysprep обеспечивает работу, удаляя все персональные данные и переустанавливая несколько компонентов.

Обычно выполняется `sysprep.exe` Создание шаблона, из которого можно развернуть несколько других виртуальных машин с определенной конфигурацией. Шаблон называется *обобщенным изображением*.

Чтобы создать только одну виртуальную машину с одного диска, не нужно использовать Sysprep. Вместо этого можно создать виртуальную машину на основе *специализированного образа*. Сведения о том, как создать виртуальную машину на основе специализированного диска, см. в следующих статьях:

- [Создание виртуальной машины Windows из специализированного диска](create-vm-specialized.md)
- [Create a VM from a specialized VHD disk](./create-vm-specialized-portal.md) (Создание виртуальной машины из специализированного VHD-диска)

Для создания обобщенного образа необходимо запустить Sysprep. Дополнительные сведения см. в разделе [Использование Sysprep: введение](/previous-versions/windows/it-pro/windows-xp/bb457073(v=technet.10)).

Не все роли или приложения, установленные на компьютере под управлением Windows, поддерживают обобщенные образы. Перед использованием этой процедуры убедитесь, что Sysprep поддерживает роль компьютера. Дополнительные сведения см. в разделе [Поддержка Sysprep для ролей сервера](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles).

В частности, Sysprep требует полной расшифровки дисков перед выполнением. Если вы включили шифрование на виртуальной машине, отключите ее перед запуском Sysprep.


### <a name="generalize-a-vhd"></a>Подготовка виртуального жесткого диска к использованию

>[!NOTE]
> Выполнив `sysprep.exe` следующие действия, выключите виртуальную машину. Не включайте его снова, пока вы не создадите образ в Azure.

1. Войдите на виртуальную машину Windows.
1. Запустите сеанс PowerShell от имени администратора.
1. Удалите каталог пансер (К:\виндовс\пансер).
1. Измените каталог на `%windir%\system32\sysprep` . Затем выполните `sysprep.exe`.
1. В диалоговом окне **средство подготовки системы** выберите вход в систему по раскрывающемсяу **интерфейсу (OOBE)** и убедитесь, что установлен флажок **generalize** .

    ![Средство SysPrep](media/prepare-for-upload-vhd-image/syspre.png)
1. В разделе **Параметры завершения работы** выберите **Завершение работы**.
1. Щелкните **ОК**.
1. После завершения работы программы Sysprep завершите работу виртуальной машины. Не используйте **перезагрузку** для завершения работы виртуальной машины.

Теперь диск VHD можно отправлять. Дополнительные сведения о том, как создать виртуальную машину на основе обобщенного диска, см. в разделе [Отправка обобщенного виртуального жесткого диска и его использование для создания новой виртуальной машины в Azure](/previous-versions/azure/virtual-machines/windows/sa-upload-generalized).

>[!NOTE]
> Пользовательский файл *unattend.xml* не поддерживается. Хотя мы поддерживаем свойство **аддитионалунаттендконтент** , которое предоставляет ограниченную поддержку добавления параметров [Microsoft-Windows-Shell-setup](/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup) в файл *unattend.xml* , используемый агентом подготовки Azure. Можно использовать, например, [аддитионалунаттендконтент](/dotnet/api/microsoft.azure.management.compute.models.additionalunattendcontent) , чтобы добавить Фирстлогонкоммандс и логонкоммандс. Дополнительные сведения см. в разделе [Аддитионалунаттендконтент фирстлогонкоммандс example](https://github.com/Azure/azure-quickstart-templates/issues/1407).

## <a name="convert-the-virtual-disk-to-a-fixed-size-vhd"></a>Преобразование виртуального диска в виртуальный жесткий диск фиксированного размера

Используйте один из методов в этом разделе, чтобы преобразовать виртуальный диск и изменить его размер в требуемый формат для Azure:

1. Создайте резервную копию виртуальной машины перед выполнением преобразования или изменения размера виртуального диска.

1. Убедитесь, что виртуальный жесткий диск Windows правильно работает на локальном сервере. Устраните все ошибки на виртуальной машине, прежде чем преобразовывать ее или отправлять в Azure.

1. Преобразуйте виртуальный диск в тип fixed.

1. Измените размер виртуального диска в соответствии с требованиями Azure:

   1. Дискам в Azure должен быть назначен виртуальный размер, который соответствует 1 MiB. Если виртуальный жесткий диск является частью 1 MiB, необходимо изменить размер диска до кратного 1 MiB. Диски, которые являются частью MiB, вызывают ошибки при создании образов из отправленного виртуального жесткого диска. Чтобы проверить размер, можно использовать командлет PowerShell [Get-VHD](/powershell/module/hyper-v/get-vhd) , чтобы отобразить "size", который должен быть кратен 1 MiB в Azure, и "Размер файла", который будет равен "size" плюс 512 байта для нижнего колонтитула VHD.
   
   1. Максимально допустимый размер виртуального жесткого диска для ОС с виртуальной машиной 1 поколения — 2 048 гиб (2 Тиб). 
   1. Максимальный размер диска данных — 32 767 гиб (32 Тиб).

> [!NOTE]
> - Если вы готовите диск операционной системы Windows после преобразования в фиксированный диск и при необходимости измените его размер, создайте виртуальную машину, использующую этот диск. Запустите и войдите на виртуальную машину и продолжите подразделы в этой статье, чтобы завершить подготовку к отправке.  
> - При подготовке диска данных вы можете перезапустить этот раздел и приступить к отправке диска.

### <a name="use-hyper-v-manager-to-convert-the-disk"></a>Преобразование диска с помощью диспетчера Hyper-V

1. Откройте диспетчер Hyper-V и выберите свой локальный компьютер в левой части окна. В меню над списком компьютер выберите **действие**  >  **изменить диск**.
1. На странице **Обнаружение виртуального жесткого диска** выберите виртуальный диск.
1. На странице **Выбор действия** выберите **преобразовать**  >  **Далее**.
1. Чтобы преобразовать VHDX-файл, выберите **VHD**  >  **Далее**.
1. Чтобы преобразовать динамически расширяемый диск, выберите **фиксированный размер**  >  **Далее**.
1. Выберите путь, чтобы сохранить новый VHD-файл.
1. Нажмите кнопку **Готово**.

### <a name="use-powershell-to-convert-the-disk"></a>Преобразование диска с помощью PowerShell

Вы можете преобразовать виртуальный диск с помощью командлета [Convert-VHD](/powershell/module/hyper-v/convert-vhd) в PowerShell. Сведения об установке этого командлета см. [в статье Установка роли Hyper-V](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).

В следующем примере показано преобразование диска из формата VHDX в VHD. Он также преобразует диск из динамически расширяющегося диска в диск фиксированного размера.

```powershell
Convert-VHD -Path C:\test\MyVM.vhdx -DestinationPath C:\test\MyNewVM.vhd -VHDType Fixed
```

В этом примере замените значение **path** на путь к виртуальному жесткому диску, который требуется преобразовать. Замените значение **DestinationPath** на новый путь и имя преобразованного диска.

### <a name="use-hyper-v-manager-to-resize-the-disk"></a>Использование диспетчера Hyper-V для изменения размера диска

1. Откройте диспетчер Hyper-V и выберите свой локальный компьютер в левой части окна. В меню над списком компьютер выберите **действие**  >  **изменить диск**.
1. На странице **Обнаружение виртуального жесткого диска** выберите виртуальный диск.
1. На странице **Выбор действия** нажмите кнопку **развернуть**  >  **Далее**.
1. На странице **определение виртуального жесткого диска** введите новый размер в гиб > **Далее**.
1. Нажмите кнопку **Готово**.

### <a name="use-powershell-to-resize-the-disk"></a>Использование PowerShell для изменения размера диска

Вы можете изменить размер виртуального диска с помощью командлета [resize-VHD](/powershell/module/hyper-v/resize-vhd) в PowerShell. Сведения об установке этого командлета см. [в статье Установка роли Hyper-V](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).

В следующем примере показано, как изменить размер диска с 100,5 MiB на 101 MiB в соответствии с требованиями к выравниванию Azure.

```powershell
Resize-VHD -Path C:\test\MyNewVM.vhd -SizeBytes 105906176
```

В этом примере замените значение **path** на путь к виртуальному жесткому диску, размер которого нужно изменить. Замените значение **сизебитес** на новый размер в байтах для диска.

### <a name="convert-from-vmware-vmdk-disk-format"></a>Преобразование диска VMware в формате VMDK

Если вы используете образ виртуальной машины Windows в [формате VMDK](https://en.wikipedia.org/wiki/VMDK), то можете использовать службу " [Миграция Azure](../../migrate/server-migrate-overview.md) " для преобразования VMDK-файла и его передачи в Azure.

## <a name="complete-the-recommended-configurations"></a>Выполнение рекомендуемых конфигураций

Следующие параметры не влияют на отправку виртуального жесткого диска. Тем не менее, мы настоятельно рекомендуем их настроить.

- Установите [Агент виртуальной машины Azure](https://go.microsoft.com/fwlink/?LinkID=394789). Затем можно включить расширения виртуальной машины. Расширения виртуальных машин реализуют большинство важных функций, которые могут потребоваться для использования с виртуальными машинами. Для сброса паролей или настройки протокола удаленного рабочего стола требуются расширения, например. Дополнительные сведения см. в [обзоре агента виртуальной машины Azure](../extensions/agent-windows.md).
- После создания виртуальной машины в Azure рекомендуется разместить файл подкачки на *томе временного диска* , чтобы повысить производительность. Расположение файла можно настроить следующим образом:

  ```powershell
  Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management' -Name PagingFiles -Value 'D:\pagefile.sys' -Type MultiString -Force
  ```

  Если диск данных подключен к виртуальной машине, то буква тома временного диска обычно равна *D*. Это может быть различным, в зависимости от параметров и количества доступных дисков.

  - Рекомендуется отключить блокирование сценариев, которое может быть предоставлено антивирусным программным обеспечением. Они могут мешать работе и блокировать сценарии агента подготовки Windows, выполняемые при развертывании новой виртуальной машины из образа.

## <a name="next-steps"></a>Дальнейшие действия

- [Отправка образа виртуальной машины Windows в Azure для развертываний Resource Manager](upload-generalized-managed.md)
- [Устранение неполадок при активации виртуальной машины Windows в Azure](../troubleshooting/troubleshoot-activation-problems.md)

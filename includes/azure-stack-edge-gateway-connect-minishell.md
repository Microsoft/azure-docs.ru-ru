---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/08/2021
ms.author: alkohli
ms.openlocfilehash: 0ad760caedffa97599548b8dd1b59a887b5690af
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105144"
---
В зависимости от операционной системы клиента процедуры удаленного подключения к устройству отличаются.

### <a name="remotely-connect-from-a-windows-client"></a>Удаленное подключение из клиента Windows


#### <a name="prerequisites"></a>Предварительные требования

Перед тем как начать, убедитесь в следующем.

- Клиент Windows работает под управлением Windows PowerShell 5,0 или более поздней версии.
- У клиента Windows есть цепочка подписывания (корневой сертификат), соответствующая сертификату узла, установленному на устройстве. Подробные инструкции см. [в статье Установка сертификата на клиенте Windows](../articles/databox-online/azure-stack-edge-gpu-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).
- `hosts`Файл, расположенный в папке `C:\Windows\System32\drivers\etc` для клиента Windows, имеет запись, соответствующую сертификату узла в следующем формате:

    `<Device IP>    <Node serial number>.<DNS domain of the device>`

    Ниже приведен пример записи для `hosts` файла.
 
    `10.100.10.10    1HXQG13.wdshcsso.com`
  

#### <a name="detailed-steps"></a>Подробные инструкции

Выполните следующие действия, чтобы удаленно подключиться из клиента Windows.

1. Запустите сеанс Windows PowerShell от имени администратора.
2. Убедитесь, что на вашем клиенте запущена служба служба удаленного управления Windows. В командной строке введите следующее:

    `winrm quickconfig`

    Дополнительные сведения см. в разделе [Установка и настройка для службы удаленного управления Windows](/windows/win32/winrm/installation-and-configuration-for-windows-remote-management#quick-default-configuration).

3. Назначьте переменную IP-адресу устройства.

    $ip = "<device_ip>"

    Замените `<device_ip>` IP-адресом вашего устройства.

4. Чтобы добавить IP-адрес устройства в список доверенных узлов клиента, введите следующую команду:

    `Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force`

5. Запустите сеанс Windows PowerShell на устройстве:

    `Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell -UseSSL`

    Если отображается ошибка, связанная с отношением доверия, убедитесь, что цепочка подписывания сертификата узла, переданного на устройство, также установлена на клиенте, обращающемся к устройству.

    > [!NOTE] 
    > При использовании этого `-UseSSL` параметра выполняется удаленное взаимодействие через PowerShell по *протоколу HTTPS*. Рекомендуется всегда использовать *протокол HTTPS* для удаленного подключения с помощью PowerShell. Хотя сеанс *http* не является самым безопасным методом подключения, он приемлем в доверенных сетях.

6. При появлении запроса введите пароль. Используйте тот же пароль, который используется для входа в локальный веб-интерфейс. Локальный пароль пользовательского веб-интерфейса по умолчанию — *password1*. При успешном подключении к устройству с помощью удаленной PowerShell вы увидите следующий пример выходных данных:  

    ```
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.
    
    PS C:\WINDOWS\system32> winrm quickconfig
    WinRM service is already running on this machine.
    PS C:\WINDOWS\system32> $ip = "10.100.10.10"
    PS C:\WINDOWS\system32> Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force
    PS C:\WINDOWS\system32> Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell -UseSSL

    WARNING: The Windows PowerShell interface of your device is intended to be used only for the initial network configuration. Please engage Microsoft Support if you need to access this interface to troubleshoot any potential issues you may be experiencing. Changes made through this interface without involving Microsoft Support could result in an unsupported configuration.
    [10.100.10.10]: PS>
    ```

### <a name="remotely-connect-from-a-linux-client"></a>Удаленное подключение из клиента Linux

На клиенте Linux, который будет использоваться для подключения:

- [Установите последнюю версию PowerShell Core для Linux](/powershell/scripting/install/installing-powershell-core-on-linux) из GitHub, чтобы получить возможность удаленного взаимодействия SSH. 
- [Установите только `gss-ntlmssp` пакет из модуля NTLM](https://github.com/Microsoft/omi/blob/master/Unix/doc/setup-ntlm-omi.md). Для клиентов Ubuntu используйте следующую команду:
    - `sudo apt-get install gss-ntlmssp`

Дополнительные сведения см. в подразделах [удаленное взаимодействие PowerShell по протоколу SSH](/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core).

Выполните следующие действия, чтобы удаленно подключиться из клиента NFS.

1. Чтобы открыть сеанс PowerShell, введите:

    `pwsh`
 
2. Для подключения с помощью удаленного клиента введите:

    `Enter-PSSession -ComputerName $ip -Authentication Negotiate -ConfigurationName Minishell -Credential ~\EdgeUser`

    При появлении запроса введите пароль, используемый для входа на устройство.
 
> [!NOTE]
> Эта процедура не работает на Mac OS.
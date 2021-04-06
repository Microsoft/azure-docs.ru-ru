---
title: Руководство. Базовая локальная среда Active Directory и среда Azure AD
services: active-directory
description: Узнайте, как создать базовую среду AD и Azure AD.
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 12/02/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: de59b3f26a9a4696c721cab94cd5b96bd0200e9e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98612357"
---
# <a name="tutorial-basic-active-directory-environment"></a>Руководство по базовой среде Active Directory

В этом руководстве показано, как создать базовую среду Active Directory. 

![Схема, на которой показана базовая среда Azure AD.](media/tutorial-single-forest/diagram-2.png)

Созданную по инструкциям из этого руководства среду можно использовать для тестирования разных аспектов в гибридных сценариях идентификации. Прохождение этого руководства является необходимым условием для работы с некоторыми другими руководствами.  Если у вас уже есть среда Active Directory, вы можете использовать ее.  Приведенная здесь информация предназначена для тех пользователей, которые только начинают работу.

Это руководство состоит из следующих разделов.
## <a name="prerequisites"></a>Предварительные требования
Для работы с этим учебником требуется следующее:
- Компьютер с установленным [Hyper-V](/windows-server/virtualization/hyper-v/hyper-v-technology-overview).  Рекомендуется использовать компьютер с ОС [Windows 10](/virtualization/hyper-v-on-windows/about/supported-guest-os) или [Windows Server 2016](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows).
- [Внешний сетевой адаптер](/virtualization/hyper-v-on-windows/quick-start/connect-to-network) для связи виртуальной машины с Интернетом.
- [Подписка Azure](https://azure.microsoft.com/free)
- Копия Windows Server 2016.
- [Microsoft .NET Framework 4.7.1](https://www.microsoft.com/download/details.aspx?id=56115)

> [!NOTE]
> В этом учебнике используются скрипты PowerShell, что позволяет создать учебную среду в минимальные сроки.  В каждом скрипте применяются переменные, которые объявляются в начале скрипта.  Эти переменные можно и нужно изменить в соответствии с особенностями вашей среды.
>
>Эти скрипты создают общую среду Active Directory перед установкой агента облачной подготовки Azure AD Connect.  Они актуальны для всех руководств.
>
> Копии сценариев PowerShell, используемых в этом руководстве, можно найти на сайте GitHub [здесь](https://github.com/billmath/tutorial-phs).

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины
Первое, что нужно сделать для подготовки среды гибридной идентификации к работе, — это создать виртуальную машину, которая будет служить локальным сервером Active Directory.  Выполните следующие действия.

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell
    #Declare variables
    $VMName = 'DC1'
    $Switch = 'External'
    $InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
    $Path = 'D:\VM'
    $VHDPath = 'D:\VM\DC1\DC1.vhdx'
    $VHDSize = '64424509440'

    #Create New Virtual Machine
    New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

    #Set the memory to be non-dynamic
    Set-VMMemory $VMName -DynamicMemoryEnabled $false

    #Add DVD Drive to Virtual Machine
    Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

    #Mount Installation Media
    $DVDDrive = Get-VMDvdDrive -VMName $VMName

    #Configure Virtual Machine to Boot from DVD
    Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive 
    ```

## <a name="complete-the-operating-system-deployment"></a>Завершение развертывания операционной системы
Чтобы завершить создание виртуальной машины, необходимо завершить установку операционной системы.

1. В диспетчере Hyper-V дважды щелкните виртуальную машину.
2. Нажмите кнопку "Запустить".
3. Вам будет предложено нажать любую клавишу для загрузки с компакт-диска или DVD-диска. Сделайте это.
4. На начальном экране Windows Server выберите язык и нажмите кнопку **Далее**.
5. Нажмите **Установить**.
6. Введите ключ лицензии и нажмите кнопку **Далее**.
7. Установите флажок "Я принимаю условия лицензии" и нажмите кнопку **Далее**.
8. Выберите **Custom:  Install Windows Only (Advanced)** (Пользовательская: установить только Windows (расширенная).
9. Нажмите кнопку **Далее**
10. Когда установка завершится, перезапустите виртуальную машину, выполните вход и запустите обновление Windows, чтобы обеспечить актуальность виртуальной машины.  Установите последние обновления.

## <a name="install-active-directory-prerequisites"></a>Установка необходимых компонентов для Active Directory
После этого нужно выполнить еще несколько действий перед установкой Active Directory.  В частности, вам нужно переименовать виртуальную машину, задать статический IP-адрес и указать сведения DNS, а также установить средства удаленного администрирования сервера.   Выполните следующие действия.

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell
    #Declare variables
    $ipaddress = "10.0.1.117" 
    $ipprefix = "24" 
    $ipgw = "10.0.1.1" 
    $ipdns = "10.0.1.117"
    $ipdns2 = "8.8.8.8" 
    $ipif = (Get-NetAdapter).ifIndex 
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $newname = "DC1"
    $addsTools = "RSAT-AD-Tools" 

    #Set static IP address
    New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

    # Set the DNS servers
    Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

    #Rename the computer 
    Rename-Computer -NewName $newname -force 

    #Install features 
    New-Item $featureLogPath -ItemType file -Force 
    Add-WindowsFeature $addsTools 
    Get-WindowsFeature | Where installed >>$featureLogPath 

    #Restart the computer 
    Restart-Computer
    ```

## <a name="create-a-windows-server-ad-environment"></a>Создание среды AD для Windows Server
Итак, вы создали виртуальную машину, переименовали ее и назначили ей статический IP-адрес. Теперь вы можете установить и настроить доменные службы Active Directory.  Выполните следующие действия.

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell 
    #Declare variables
    $DatabasePath = "c:\windows\NTDS"
    $DomainMode = "WinThreshold"
    $DomainName = "contoso.com"
    $DomaninNetBIOSName = "CONTOSO"
    $ForestMode = "WinThreshold"
    $LogPath = "c:\windows\NTDS"
    $SysVolPath = "c:\windows\SYSVOL"
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $Password = "Pass1w0rd"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force

    #Install AD DS, DNS and GPMC 
    start-job -Name addFeature -ScriptBlock { 
    Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
    Wait-Job -Name addFeature 
    Get-WindowsFeature | Where installed >>$featureLogPath

    #Create New AD Forest
    Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $SecureString -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
    ```

## <a name="create-a-windows-server-ad-user"></a>Создание пользователя AD для Windows Server
Теперь, когда среда Active Directory создана, вам потребуется тестовая учетная запись.  Она будет создана в локальной среде AD, а затем синхронизирована с Azure AD.  Выполните следующие действия.

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell 
    # Filename:    4_CreateUser.ps1
    # Description: Creates a user in Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $Givenname = "Allie"
    $Surname = "McCray"
    $Displayname = "Allie McCray"
    $Name = "amccray"
    $Password = "Pass1w0rd"
    $Identity = "CN=ammccray,CN=Users,DC=contoso,DC=com"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


    #Create the user
    New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

    #Set the password to never expire
    Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
    ```


## <a name="create-an-azure-ad-tenant"></a>Создание клиента Azure AD
Теперь создайте клиент Azure AD, чтобы синхронизировать пользователей с облаком.  Чтобы создать клиент Azure AD, выполните указанные ниже действия.

1. Перейдите на [портал Azure](https://portal.azure.com) и выполните вход с учетной записью, имеющей подписку Azure.
2. Щелкните **значок "плюс" (+)** и выполните поиск по запросу **Azure Active Directory**.
3. В результатах поиска выберите **Azure Active Directory**.
4. Нажмите кнопку **создания**.</br>
![Снимок экрана: страница Azure Active Directory на портале Azure.](media/tutorial-single-forest/create-1.png)</br>
5. Укажите **имя организации** с **первоначальным доменным именем**. Щелкните **Создать**. В результате будет создан каталог.
6. По завершении щелкните ссылку **здесь**, чтобы перейти к управлению каталогом.

## <a name="create-a-global-administrator-in-azure-ad"></a>Создание глобального администратора в Azure AD
Получив клиент Azure AD, создайте учетную запись глобального администратора.  Чтобы создать учетную запись глобального администратора, выполните указанные ниже действия.

1.  В разделе **Управление** выберите **Пользователи**.</br>
![Снимок экрана: меню "Обзор" с выбранным пунктом "Пользователи".](media/tutorial-single-forest/administrator-1.png)</br>
2.  Выберите пункт **Все пользователи**, а затем выберите **+ Новый пользователь**.
3.  Укажите имя и имя пользователя. Этот пользователь будет глобальным администратором клиента. Кроме того, может потребоваться изменить **роль каталога** на **Глобальный администратор**. Можно также отобразить временный пароль. Когда все будет готово, нажмите кнопку **Создать**.</br>
![Создание](media/tutorial-single-forest/administrator-2.png)</br>
4. По завершении откройте новую вкладку в браузере и выполните вход на странице myapps.microsoft.com, используя новую учетную запись глобального администратора и временный пароль.
5. Смените пароль глобального администратора и запомните новый пароль.

## <a name="optional--additional-server-and-forest"></a>Необязательное действие:  Дополнительный сервер и лес
В этом необязательном разделе показано, как создать дополнительный сервер и (или) лес.  Эти элементы могут пригодиться в некоторых руководствах более сложного уровня (например, [Пилотная облачная синхронизация для существующего синхронизированного леса AD](tutorial-pilot-aadc-aadccp.md)).

Если вам нужен только дополнительный сервер, завершите процесс после шага **Создание виртуальной машины** и присоедините полученный сервер к ранее созданному домену.  

### <a name="create-a-virtual-machine"></a>Создание виртуальной машины

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell
    # Filename:    1_CreateVM_CP.ps1
    # Description: Creates a VM to be used in the tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. #This script is made available to you without any express, implied or statutory warranty, not even the implied warranty of merchantability or fitness for a particular purpose, or the warranty of title or non-infringement. The entire risk of the use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $VMName = 'CP1'
    $Switch = 'External'
    $InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
    $Path = 'D:\VM'
    $VHDPath = 'D:\VM\CP1\CP1.vhdx'
    $VHDSize = '64424509440'

    #Create New Virtual Machine
    New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

    #Set the memory to be non-dynamic
    Set-VMMemory $VMName -DynamicMemoryEnabled $false

    #Add DVD Drive to Virtual Machine
    Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

    #Mount Installation Media
    $DVDDrive = Get-VMDvdDrive -VMName $VMName

    #Configure Virtual Machine to Boot from DVD
    Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
    ```

### <a name="complete-the-operating-system-deployment"></a>Завершение развертывания операционной системы
Чтобы завершить создание виртуальной машины, необходимо завершить установку операционной системы.

1. В диспетчере Hyper-V дважды щелкните виртуальную машину.
2. Нажмите кнопку "Запустить".
3. Вам будет предложено нажать любую клавишу для загрузки с компакт-диска или DVD-диска. Сделайте это.
4. На начальном экране Windows Server выберите язык и нажмите кнопку **Далее**.
5. Нажмите **Установить**.
6. Введите ключ лицензии и нажмите кнопку **Далее**.
7. Установите флажок "Я принимаю условия лицензии" и нажмите кнопку **Далее**.
8. Выберите **Custom:  Install Windows Only (Advanced)** (Пользовательская: установить только Windows (расширенная).
9. Нажмите кнопку **Далее**
10. Когда установка завершится, перезапустите виртуальную машину, выполните вход и запустите обновление Windows, чтобы обеспечить актуальность виртуальной машины.  Установите последние обновления.

### <a name="install-active-directory-prerequisites"></a>Установка необходимых компонентов для Active Directory
После этого нужно выполнить еще несколько действий перед установкой Active Directory.  В частности, вам нужно переименовать виртуальную машину, задать статический IP-адрес и указать сведения DNS, а также установить средства удаленного администрирования сервера.   Выполните следующие действия.

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell
    # Filename:    2_ADPrep_CP.ps1
    # Description: Prepares your environment for Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $ipaddress = "10.0.1.118" 
    $ipprefix = "24" 
    $ipgw = "10.0.1.1" 
    $ipdns = "10.0.1.118"
    $ipdns2 = "8.8.8.8" 
    $ipif = (Get-NetAdapter).ifIndex 
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $newname = "CP1"
    $addsTools = "RSAT-AD-Tools" 

    #Set static IP address
    New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

    #Set the DNS servers
    Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

    #Rename the computer 
    Rename-Computer -NewName $newname -force 

    #Install features 
    New-Item $featureLogPath -ItemType file -Force 
    Add-WindowsFeature $addsTools 
    Get-WindowsFeature | Where installed >>$featureLogPath 

    #Restart the computer 
    Restart-Computer
    ```
### <a name="create-a-windows-server-ad-environment"></a>Создание среды AD для Windows Server
Итак, вы создали виртуальную машину, переименовали ее и назначили ей статический IP-адрес. Теперь вы можете установить и настроить доменные службы Active Directory.  Выполните следующие действия.

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell
    # Filename:    3_InstallAD_CP.ps1
    # Description: Creates an on-premises AD environment.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $DatabasePath = "c:\windows\NTDS"
    $DomainMode = "WinThreshold"
    $DomainName = "fabrikam.com"
    $DomaninNetBIOSName = "FABRIKAM"
    $ForestMode = "WinThreshold"
    $LogPath = "c:\windows\NTDS"
    $SysVolPath = "c:\windows\SYSVOL"
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $Password = "Pass1w0rd"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force

    #Install AD DS, DNS and GPMC 
    start-job -Name addFeature -ScriptBlock { 
    Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
    Wait-Job -Name addFeature 
    Get-WindowsFeature | Where installed >>$featureLogPath

    #Create New AD Forest
    Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $SecureString -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
    ```

### <a name="create-a-windows-server-ad-user"></a>Создание пользователя AD для Windows Server
Теперь, когда среда Active Directory создана, вам потребуется тестовая учетная запись.  Она будет создана в локальной среде AD, а затем синхронизирована с Azure AD.  Выполните следующие действия.

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

    ```powershell 
    # Filename:    4_CreateUser_CP.ps1
    # Description: Creates a user in Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $Givenname = "Anna"
    $Surname = "Ringdal"
    $Displayname = "Anna Ringdal"
    $Name = "aringdal"
    $Password = "Pass1w0rd"
    $Identity = "CN=aringdal,CN=Users,DC=fabrikam,DC=com"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


    #Create the user
    New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

    #Set the password to never expire
    Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
    ```

## <a name="conclusion"></a>Заключение
Теперь у вас есть среда, которую можно использовать для работы с доступными руководствами и тестирования дополнительных функций, предоставляемых облачной синхронизацией.

## <a name="next-steps"></a>Дальнейшие действия 

- [Что собой представляет подготовка?](what-is-provisioning.md)
- [Что представляет собой облачная синхронизация Azure AD Connect?](what-is-cloud-sync.md)
---
title: Развертывание службы "Синхронизация файлов Azure" | Документация Майкрософт
description: Узнайте, как развернуть Синхронизация файлов Azure, от начала до конца, с помощью портал Azure, PowerShell или Azure CLI.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 11/05/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 4814a12a870d5317ad91c3514327ba0daad7ed69
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99225376"
---
# <a name="deploy-azure-file-sync"></a>Развертывание Синхронизации файлов Azure
Используйте службу "Синхронизация файлов Azure", чтобы централизованно хранить файловые ресурсы организации в службе файлов Azure, обеспечивая гибкость, производительность и совместимость локального файлового сервера. Это достигается путем преобразования Windows Server в быстрый кэш общего файлового ресурса Azure. Для локального доступа к данным вы можете использовать любой протокол, доступный в Windows Server, в том числе SMB, NFS и FTPS. Кроме того, вы можете создать любое количество кэшей в любом регионе.

Перед выполнением шагов, описанных в этом руководстве, настоятельно рекомендуем ознакомиться со статьями [Планирование развертывания службы файлов Azure](storage-files-planning.md) и [Планирование развертывания службы синхронизации файлов Azure (предварительная версия)](storage-sync-files-planning.md).

## <a name="prerequisites"></a>Предварительные требования

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Файловый ресурс Azure в том же регионе, который требуется развернуть Синхронизация файлов Azure. Дополнительные сведения см. в следующих статьях:
    - [Доступность по регионам](storage-sync-files-planning.md#azure-file-sync-region-availability). Здесь представлены сведения о регионах службы синхронизации файлов Azure.
    - [Создание общей папки в хранилище файлов Azure](storage-how-to-create-file-share.md). Здесь содержатся пошаговые инструкции по созданию общего файлового ресурса.
1. По крайней мере один поддерживаемый экземпляр Windows Server или кластера Windows Server для синхронизации с Синхронизация файлов Azure. Дополнительные сведения о поддерживаемых версиях Windows Server и рекомендуемых системных ресурсах см. в статье [рекомендации по файловым серверам Windows](storage-sync-files-planning.md#windows-file-server-considerations).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Файловый ресурс Azure в том же регионе, который требуется развернуть Синхронизация файлов Azure. Дополнительные сведения см. в следующих статьях:
    - [Доступность по регионам](storage-sync-files-planning.md#azure-file-sync-region-availability). Здесь представлены сведения о регионах службы синхронизации файлов Azure.
    - [Создание общей папки в хранилище файлов Azure](storage-how-to-create-file-share.md). Здесь содержатся пошаговые инструкции по созданию общего файлового ресурса.
1. По крайней мере один поддерживаемый экземпляр Windows Server или кластера Windows Server для синхронизации с Синхронизация файлов Azure. Дополнительные сведения о поддерживаемых версиях Windows Server и рекомендуемых системных ресурсах см. в статье [рекомендации по файловым серверам Windows](storage-sync-files-planning.md#windows-file-server-considerations).

1. Модуль AZ PowerShell можно использовать с PowerShell 5,1 или PowerShell 6 +. Модуль AZ PowerShell можно использовать для Синхронизация файлов Azure в любой поддерживаемой системе, в том числе в системах, отличных от Windows, однако командлет регистрации сервера всегда должен выполняться в регистрируемом экземпляре Windows Server (это можно сделать напрямую или с помощью удаленного взаимодействия PowerShell). В Windows Server 2012 R2 можно проверить, что вы используете по крайней мере PowerShell 5,1. \* Просмотрев значение свойства **PSVersion** объекта **$PSVersionTable** :

    ```powershell
    $PSVersionTable.PSVersion
    ```

    Если значение **PSVersion** меньше 5,1 \* , как в случае с последними установками Windows Server 2012 R2, можно легко выполнить обновление, загрузив и установив [Windows Management Framework (WMF) 5,1](https://www.microsoft.com/download/details.aspx?id=54616). Соответствующий пакет для загрузки и установки для Windows Server 2012 R2 — **Win 8.1 andw2k12r2-KB \* \* \* \* \* \* \* -x64. msu**. 

    PowerShell 6 + можно использовать с любой поддерживаемой системой и может быть загружена с помощью [страницы GitHub](https://github.com/PowerShell/PowerShell#get-powershell). 

    > [!Important]  
    > Если вы планируете использовать пользовательский интерфейс регистрации сервера вместо регистрации непосредственно из PowerShell, необходимо использовать PowerShell 5,1.

1. Если вы выбрали использование PowerShell 5,1, убедитесь, что установлена хотя бы платформа .NET 4.7.2. Дополнительные сведения о [платформа .NET Framework версиях и зависимостях](/dotnet/framework/migration-guide/versions-and-dependencies) в системе.

    > [!Important]  
    > При установке .NET 4.7.2 + в Windows Server Core необходимо установить с `quiet` `norestart` флагами и, иначе установка завершится ошибкой. Например, при установке .NET 4,8 команда будет выглядеть следующим образом:
    > ```PowerShell
    > Start-Process -FilePath "ndp48-x86-x64-allos-enu.exe" -ArgumentList "/q /norestart" -Wait
    > ```

1. Модуль AZ PowerShell, который можно установить, следуя приведенным ниже инструкциям: [Установка и настройка Azure PowerShell](/powershell/azure/install-Az-ps).
     
    > [!Note]  
    > Теперь модуль AZ. StorageSync устанавливается автоматически при установке модуля AZ PowerShell.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Файловый ресурс Azure в том же регионе, который требуется развернуть Синхронизация файлов Azure. Дополнительные сведения см. в следующих статьях:
    - [Доступность по регионам](storage-sync-files-planning.md#azure-file-sync-region-availability). Здесь представлены сведения о регионах службы синхронизации файлов Azure.
    - [Создание общей папки в хранилище файлов Azure](storage-how-to-create-file-share.md). Здесь содержатся пошаговые инструкции по созданию общего файлового ресурса.
1. По крайней мере один поддерживаемый экземпляр Windows Server или кластера Windows Server для синхронизации с Синхронизация файлов Azure. Дополнительные сведения о поддерживаемых версиях Windows Server и рекомендуемых системных ресурсах см. в статье [рекомендации по файловым серверам Windows](storage-sync-files-planning.md#windows-file-server-considerations).

1. [Установка Azure CLI](/cli/azure/install-azure-cli)

   При желании для выполнения шагов, описанных в этом учебнике, вы можете использовать Azure Cloud Shell.  Azure Cloud Shell — это интерактивная оболочка среды, с которой можно работать в браузере.  Запустите Cloud Shell с помощью одного из этих методов:

   - Нажмите кнопку **Попробовать** в правом верхнем углу блока с кодом. **Попробуйте** открыть Azure Cloud Shell, но автоматически не скопирует код в Cloud Shell.

   - Откройте Cloud Shell, перейдя по ссылке: [https://shell.azure.com](https://shell.azure.com)

   - Нажмите кнопку **Cloud Shell** в строке меню в правом верхнем углу окна [портала Azure](https://portal.azure.com).

1. Войдите.

   Выполните вход с помощью команды [az login](/cli/azure/reference-index#az-login), если вы используете локальную установку CLI.

   ```azurecli
   az login
   ```

    Выполните аутентификацию, следуя инструкциям в окне терминала.

1. Установите расширение [AZ филесинк](/cli/azure/ext/storagesync/storagesync) Azure CLI.

   ```azurecli
   az extension add --name storagesync
   ```

   После установки ссылки на расширение **storagesync** вы получите следующее предупреждение.

   ```output
   The installed extension 'storagesync' is experimental and not covered by customer support. Please use with discretion.
   ```

---

## <a name="prepare-windows-server-to-use-with-azure-file-sync"></a>Подготовка сервера Windows к работе со службой синхронизации файлов Azure
Для каждого сервера, который вы собираетесь использовать с функцией "Синхронизация файлов Azure", включая каждый узел сервера в отказоустойчивом кластере, отключите **конфигурацию усиленной безопасности Internet Explorer**. Это необходимо только при начальной регистрации на сервере. Конфигурацию можно включить повторно после регистрации сервера.

# <a name="portal"></a>[Портал](#tab/azure-portal)
> [!Note]  
> Этот шаг можно пропустить при развертывании Синхронизация файлов Azure в Windows Server Core.

1. Откройте диспетчер сервера.
2. Щелкните **Локальный сервер**:  
    ![Элемент "Локальный сервер" в левой части пользовательского интерфейса диспетчера сервера](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-1.PNG)
3. Перейдите по ссылке **Конфигурация усиленной безопасности Internet Explorer** на вложенной панели **Свойства**.  
    ![Панель "Конфигурация усиленной безопасности Internet Explorer" в пользовательском интерфейсе диспетчера сервера](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-2.PNG)
4. В диалоговом окне **Конфигурация усиленной безопасности Internet Explorer** выберите пункт **выкл** . для **администраторов** и **пользователей**.  
    ![Всплывающее окно "Конфигурация усиленной безопасности Internet Explorer" с установленным переключателем "Выкл."](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-3.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Чтобы отключить конфигурацию усиленной безопасности Internet Explorer, выполните следующую команду PowerShell с повышенными правами:

```powershell
$installType = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\").InstallationType

# This step is not required for Server Core
if ($installType -ne "Server Core") {
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Administrators
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Users
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Force Internet Explorer closed, if open. This is required to fully apply the setting.
    # Save any work you have open in the IE browser. This will not affect other browsers,
    # including Microsoft Edge.
    Stop-Process -Name iexplore -ErrorAction SilentlyContinue
}
``` 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Следуйте инструкциям по портал Azure или PowerShell.

---

## <a name="deploy-the-storage-sync-service"></a>Развертывание службы синхронизации хранилища 
Развертывание функции "Синхронизация файлов Azure" начинается с размещения ресурса **службы синхронизации хранилища** в группе ресурсов выбранной вами подписки. Мы рекомендуем выполнять подготовку по мере необходимости. Вы создадите отношение доверия между серверами и этим ресурсом, и сервер можно будет зарегистрировать только в одной службе синхронизации хранилища. В результате рекомендуется развертывать столько служб синхронизации хранилища, сколько необходимо для разделения групп серверов. Имейте в виду, что серверы из разных служб синхронизации хранилища не могут синхронизироваться друг с другом.

> [!Note]
> Служба синхронизации хранилища наследует разрешения на доступ из подписки и группы ресурсов, в которой она была развернута. Рекомендуется тщательно проверить, кому предоставлен доступ. Сущности с доступом на запись могут начать синхронизацию новых наборов файлов с серверов, зарегистрированных в этой службе синхронизации хранилища, и вызвать отправку потока данных в доступное для них хранилище Azure.

# <a name="portal"></a>[Портал](#tab/azure-portal)
Чтобы развернуть службу синхронизации хранилища, перейдите в [портал Azure](https://portal.azure.com/), щелкните *создать ресурс* , а затем выполните поиск синхронизация файлов Azure. В результатах поиска выберите **Синхронизация файлов Azure** и нажмите кнопку **создать** , чтобы открыть вкладку **Развертывание синхронизации хранилища** .

В открывшейся области введите следующие сведения:

- **Имя**: уникальное имя (для каждого региона) службы синхронизации хранилища.
- **Подписка.** Подписка, в которой будет создана служба синхронизации хранилища. В зависимости от стратегии конфигурации организации можно получить доступ к одной или нескольким подпискам. Подписка Azure — это базовый контейнер при выставлении счетов для каждой облачной службы (такой как "Файлы Azure").
- **Группа ресурсов.** Это логическая группа ресурсов Azure, таких как учетная запись хранения или служба синхронизации хранилища. Для Синхронизация файлов Azure можно создать новую группу ресурсов или использовать существующую. (Мы рекомендуем использовать группы ресурсов в качестве контейнеров для логической изоляции ресурсов в Организации, например для группирования ресурсов или ресурсов отдела кадров для конкретного проекта.)
- **Location**— регион, в котором требуется развернуть синхронизация файлов Azure. В этом списке доступны только поддерживаемые регионы.

По завершении выберите **Создать**, чтобы развернуть службу синхронизации хранилища.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Замените `<Az_Region>` , `<RG_Name>` и `<my_storage_sync_service>` собственными значениями, а затем используйте следующие команды для создания и развертывания службы синхронизации хранилища.

```powershell
$hostType = (Get-Host).Name

if ($installType -eq "Server Core" -or $hostType -eq "ServerRemoteHost") {
    Connect-AzAccount -UseDeviceAuthentication
}
else {
    Connect-AzAccount
}

# this variable holds the Azure region you want to deploy 
# Azure File Sync into
$region = '<Az_Region>'

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = @()
Get-AzLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the selected Azure Region or the region is mistyped.")
}

# the resource group to deploy the Storage Sync Service into
$resourceGroup = '<RG_Name>'

# Check to ensure resource group exists and create it if doesn't
$resourceGroups = @()
Get-AzResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    New-AzResourceGroup -Name $resourceGroup -Location $region
}

$storageSyncName = "<my_storage_sync_service>"
$storageSync = New-AzStorageSyncService -ResourceGroupName $resourceGroup -Name $storageSyncName -Location $region
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Следуйте инструкциям по портал Azure или PowerShell.

---

## <a name="install-the-azure-file-sync-agent"></a>Установка агента службы синхронизации файлов Azure
Агент службы синхронизации файлов Azure — это скачиваемый пакет, позволяющий синхронизировать Windows Server с общим файловым ресурсом Azure. 

# <a name="portal"></a>[Портал](#tab/azure-portal)
Его можно скачать в [Центре загрузки Майкрософт](https://go.microsoft.com/fwlink/?linkid=858257). После скачивания дважды щелкните пакет MSI, чтобы начать установку агента службы синхронизации файлов Azure.

> [!Important]  
> Если службу синхронизации файлов Azure нужно использовать с отказоустойчивым кластером, агент службы синхронизации файлов Azure должен быть установлен на каждом узле в кластере. Каждый узел в кластере должен быть зарегистрирован для работы с Синхронизация файлов Azure.

Таким образом, мы рекомендуем сделать следующее:
- Оставить путь установки по умолчанию (C:\Program Files\Azure\StorageSyncAgent), чтобы упростить устранение неполадок и обслуживание сервера.
- Включить Центр обновления Майкрософт, чтобы служба синхронизации файлов Azure поддерживалась в актуальном состоянии. Все обновления для агента службы синхронизации файлов Azure, включая обновления компонентов и исправления, будут выполняться из Центра обновления Майкрософт. Мы рекомендуем установить Последнее обновление для Синхронизация файлов Azure. Дополнительные сведения см. в разделе [Синхронизация файлов Azure политика обновления](storage-sync-files-planning.md#azure-file-sync-agent-update-policy).

По завершении установки агента службы синхронизации файлов Azure пользовательский интерфейс регистрации сервера запустится автоматически. Перед регистрацией установите службу синхронизации хранилища. В разделе ниже ознакомьтесь со сведениями о создании службы синхронизации хранилища.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Выполните следующий код PowerShell, чтобы загрузить соответствующую версию агента Синхронизации файлов Azure для своей ОС и установите ее в своей системе.

> [!Important]  
> Если службу синхронизации файлов Azure нужно использовать с отказоустойчивым кластером, агент службы синхронизации файлов Azure должен быть установлен на каждом узле в кластере. Для работы с Синхронизацией файлов Azure нужно зарегистрировать каждый узел в кластере.

```powershell
# Gather the OS version
$osver = [System.Environment]::OSVersion.Version

# Download the appropriate version of the Azure File Sync agent for your OS.
if ($osver.Equals([System.Version]::new(10, 0, 17763, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2019 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(10, 0, 14393, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2016 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(6, 3, 9600, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2012R2 `
        -OutFile "StorageSyncAgent.msi" 
} else {
    throw [System.PlatformNotSupportedException]::new("Azure File Sync is only supported on Windows Server 2012 R2, Windows Server 2016, and Windows Server 2019")
}

# Install the MSI. Start-Process is used to PowerShell blocks until the operation is complete.
# Note that the installer currently forces all PowerShell sessions closed - this is a known issue.
Start-Process -FilePath "StorageSyncAgent.msi" -ArgumentList "/quiet" -Wait

# Note that this cmdlet will need to be run in a new session based on the above comment.
# You may remove the temp folder containing the MSI and the EXE installer
Remove-Item -Path ".\StorageSyncAgent.msi" -Recurse -Force
```
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Следуйте инструкциям по портал Azure или PowerShell.

---

## <a name="register-windows-server-with-storage-sync-service"></a>Регистрация Windows Server в службе синхронизации хранилища
При регистрации Windows Server в службе синхронизации хранилища между сервером (или кластером) и службой синхронизации хранилища устанавливаются отношения доверия. Сервер можно зарегистрировать только в одной службе синхронизации хранилища, и он может синхронизироваться с другими серверами и файловыми ресурсами Azure, связанными с той же службой синхронизации хранилища.

> [!Note]
> При регистрации сервера используются ваши учетные данные Azure для создания отношения доверия между службой синхронизации хранилища и сервером Windows Server, но впоследствии сервер создает и использует собственное удостоверение, действительное, пока он остается зарегистрированным и пока действительным является текущий маркер SAS (SAS хранилища). Новый маркер SAS нельзя выдать серверу после сбоя его регистрации. Таким образом, исключается способность сервера получать доступ к файловым ресурсам Azure и любая синхронизация становится невозможной.

Администратор, регистрирующий сервер, должен быть членом **владельца** ролей управления или **участника** для данной службы синхронизации хранилища. Это можно настроить в разделе **Управление доступом (IAM)** в портал Azure для службы синхронизации хранилища.

Кроме того, можно отличать администраторов от разрешенных для регистрации серверов, которые также могут настроить синхронизацию в службе синхронизации хранилища. Для этого необходимо создать пользовательскую роль, в которой перечислены администраторы, которым разрешено регистрировать серверы, и предоставить настраиваемой роли следующие разрешения:

* "Microsoft. StorageSync/Сторажесинксервицес/Регистередсерверс/Write"
* "Microsoft. StorageSync/Сторажесинксервицес/Read"
* "Microsoft. StorageSync/Сторажесинксервицес/рабочие процессы/чтение"
* "Microsoft. StorageSync/Сторажесинксервицес/рабочие процессы/операции/чтение"

# <a name="portal"></a>[Портал](#tab/azure-portal)
После установки агента службы синхронизации файлов Azure пользовательский интерфейс регистрации сервера запустится автоматически. Если он не запустился, его можно открыть вручную, перейдя в расположение файла: C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe. После открытия пользовательского интерфейса регистрации сервера выберите **Вход**, чтобы приступить к работе.

После входа в систему вам будет предложено ввести следующие сведения:

![Снимок экрана пользовательского интерфейса регистрации сервера](media/storage-sync-files-deployment-guide/register-server-scubed-1.png)

- **Подписка Azure.** Подписка, содержащая службу синхронизации хранилища (дополнительные сведения см. в разделе [Развертывание службы синхронизации хранилища](#deploy-the-storage-sync-service)). 
- **Группа ресурсов.** Группа ресурсов, содержащая службу синхронизации хранилища.
- **Служба синхронизации хранилища.** Имя службы синхронизации хранилища, которую необходимо зарегистрировать.

Выбрав соответствующую информацию, выберите **Зарегистрировать**, чтобы завершить регистрацию сервера. В рамках процесса регистрации будет запрошен дополнительный вход.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
$registeredServer = Register-AzStorageSyncServer -ParentObject $storageSync
```
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Следуйте инструкциям по портал Azure или PowerShell.

---

## <a name="create-a-sync-group-and-a-cloud-endpoint"></a>Создание группы синхронизации и конечной точки облака
Группа синхронизации определяет топологию синхронизации для набора файлов. Конечные точки в группе синхронизации синхронизируются. Группа синхронизации должна содержать одну облачную конечную точку, которая представляет собой общий файловый ресурс Azure, и одну или несколько конечных точек сервера. Конечная точка сервера представляет путь на зарегистрированном сервере. У сервера конечные точки могут быть в нескольких группах синхронизации. Вы можете создать необходимое число групп синхронизации, чтобы соответствующим образом описать желаемую топологию синхронизации.

Конечная точка облака является указателем файлового ресурса Azure. Все конечные точки сервера будут синхронизироваться с облачной конечной точкой, превращая ее в центр. Учетная запись хранения файлового ресурса Azure должна располагаться в том же регионе, что и служба синхронизации хранилища. Общий файловый ресурс Azure будет синхронизирован с одним исключением: будет подготовлена специальная папка, сопоставимая со скрытой папкой "System Volume Information" на томе NTFS. Этот каталог называется ".SystemShareInformation". Он содержит важные метаданные синхронизации, которые не синхронизируются с другими конечными точками. Не используйте и не удаляйте его!

> [!Important]  
> В группе синхронизации можно внести изменения в любую облачную конечную точку или конечную точку сервера в группе синхронизации и синхронизировать файлы в другие конечные точки. Если вы вносите изменения непосредственно в облачную конечную точку (общий файловый ресурс Azure), эти изменения сначала должны быть определены заданием обнаружения изменений службы "Синхронизация файлов Azure". Задание обнаружения изменений инициируется для облачной конечной точки каждые 24 часа. Дополнительные сведения см. в статье [Часто задаваемые вопросы о службе файлов Azure](storage-files-faq.md#afs-change-detection).

Администратор, создающий облачную конечную точку, должен быть членом **владельца** роли управления для учетной записи хранения, содержащей файловый ресурс Azure, на который указывает облачная конечная точка. Это можно настроить в разделе **Управление доступом (IAM)** в портал Azure учетной записи хранения.

# <a name="portal"></a>[Портал](#tab/azure-portal)
Чтобы создать группу синхронизации, в [портал Azure](https://portal.azure.com/)перейдите в службу синхронизации хранилища и выберите **+ Группа синхронизации**:

![Создание группы синхронизации на портале Azure](media/storage-sync-files-deployment-guide/create-sync-group-1.png)

В открывшейся области введите следующую информацию, чтобы создать группу синхронизации с облачной конечной точкой:

- **Имя группы синхронизации**: имя создаваемой группы синхронизации. Это имя должно быть уникальным в службе синхронизации хранилища. Вы можете выбрать любое подходящее имя.
- **Подписка.** Подписка, в которой развернута служба синхронизации хранилища из шага [Развертывание службы синхронизации хранилища](#deploy-the-storage-sync-service).
- **Учетная запись хранения.** Если щелкнуть **Выберите учетную запись хранения**, отобразится другая область. В ней можно выбрать учетную запись хранения с файловым ресурсом Azure, с которым нужно синхронизироваться.
- **Общая папка Azure.** Имя синхронизируемого файлового ресурса Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Чтобы создать группу синхронизации, выполните следующую команду PowerShell. Замените `<my-sync-group>` нужным именем группы синхронизации.

```powershell
$syncGroupName = "<my-sync-group>"
$syncGroup = New-AzStorageSyncGroup -ParentObject $storageSync -Name $syncGroupName
```

После успешного создания группы синхронизации можно создать облачную конечную точку. Замените `<my-storage-account>` и `<my-file-share>` фактическими значениями.

```powershell
# Get or create a storage account with desired name
$storageAccountName = "<my-storage-account>"
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup | Where-Object {
    $_.StorageAccountName -eq $storageAccountName
}

if ($storageAccount -eq $null) {
    $storageAccount = New-AzStorageAccount `
        -Name $storageAccountName `
        -ResourceGroupName $resourceGroup `
        -Location $region `
        -SkuName Standard_LRS `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly:$true
}

# Get or create an Azure file share within the desired storage account
$fileShareName = "<my-file-share>"
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    $fileShare = New-AzStorageShare -Context $storageAccount.Context -Name $fileShareName
}

# Create the cloud endpoint
New-AzStorageSyncCloudEndpoint `
    -Name $fileShare.Name `
    -ParentObject $syncGroup `
    -StorageAccountResourceId $storageAccount.Id `
    -AzureFileShareName $fileShare.Name
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используйте команду [AZ storagesync Sync-Group](/cli/azure/ext/storagesync/storagesync/sync-group#ext-storagesync-az-storagesync-sync-group-create) , чтобы создать новую группу синхронизации.  Чтобы настроить группу ресурсов по умолчанию для всех команд интерфейса командной строки, используйте команду [AZ configure](/cli/azure/reference-index#az-configure).

```azurecli
az storagesync sync-group create --resource-group myResourceGroupName \
                                 --name myNewSyncGroupName \
                                 --storage-sync-service myStorageSyncServiceName \
```

Чтобы создать облачную конечную точку, используйте команду [AZ storagesync Sync-Group облака-Endpoint](/cli/azure/ext/storagesync/storagesync/sync-group/cloud-endpoint#ext-storagesync-az-storagesync-sync-group-cloud-endpoint-create) .

```azurecli
az storagesync sync-group cloud-endpoint create --resource-group myResourceGroup \
                                                --storage-sync-service myStorageSyncServiceName \
                                                --sync-group-name mySyncGroupName \
                                                --name myNewCloudEndpointName \
                                                --storage-account mystorageaccountname \
                                                --azure-file-share-name azure-file-share-name
```

---

## <a name="create-a-server-endpoint"></a>Создание конечной точки сервера
Конечная точка сервера представляет собой определенное расположение на зарегистрированном сервере, например папку в томе сервера. Конечная точка сервера должна представлять собой путь на зарегистрированном сервере (а не на установленном ресурсе), а для распределения по уровням облака путь должен располагаться в несистемном томе. Запоминающее устройство, подключаемое к сети (NAS), не поддерживается.

> [!NOTE]
> Изменение пути или буквы диска после установки конечной точки сервера в томе не поддерживается. Убедитесь, что вы используете окончательный путь на зарегистрированном сервере.

# <a name="portal"></a>[Портал](#tab/azure-portal)
Чтобы добавить конечную точку сервера, перейдите к только что созданной группе синхронизации и выберите **Добавить конечную точку сервера**.

![Добавление новой конечной точки сервера в области группы синхронизации](media/storage-sync-files-deployment-guide/create-sync-group-2.png)

В области **Добавить конечную точку сервера** введите следующие сведения, чтобы создать конечную точку сервера.

- **Зарегистрированный сервер**. имя сервера или кластера, в котором требуется создать конечную точку сервера.
- **Путь**: путь Windows Server для синхронизации в составе группы синхронизации.
- **Уровни в облаке.** Параметр, позволяющий включать и отключать распределение по уровням облака. С помощью этого параметра редко используемые файлы можно распределить по уровням компонента "Файлы Azure".
- **Свободное место в томе**: объем свободного места, резервируемого на томе, на котором находится конечная точка сервера. Например, если объем свободного места тома с единственной конечной точкой сервера равен 50 %, примерно половина объема данных будет распределена по уровням компонента "Файлы Azure". Независимо от того, включено ли распределение по уровням в облаке, общий файловый ресурс Azure всегда содержит полную копию данных в группе синхронизации.
- **Начальный режим загрузки**. Это необязательный вариант, начиная с версии агента 11, который может быть полезен при наличии файлов в общей папке Azure, но не на сервере. Такая ситуация может существовать, например, при создании конечной точки сервера для добавления другого сервера филиала в группу синхронизации или при аварийном восстановлении сервера, на котором произошел сбой. Если включено распределение по уровням облака, по умолчанию используется только для повторного вызова пространства имен без содержимого файла. Это полезно, если вы считаете, что вместо запросов доступа пользователей следует решить, какое содержимое файлов будет отозвано на сервере. Если распределение по уровням облака отключено, по умолчанию пространство имен будет скачиваться первыми, а затем файлы будут отозваны на основе метки времени последнего изменения, пока не будет достигнута локальная емкость. Однако можно изменить режим начальной загрузки на только пространство имен. Третий режим можно использовать, только если для этой конечной точки сервера отключено распределение по уровням облака. Этот режим позволяет избежать повторного вызова пространства имен. Файлы будут отображаться только на локальном сервере, если у них есть возможность полной загрузки. Этот режим удобен, если для экземпляра приложения требуется наличие полных файлов, которые не допускают использования многоуровневых файлов в пространстве имен.

Чтобы добавить конечную точку сервера, нажмите кнопку **создать**. Теперь ваши файлы будут синхронизироваться по всем общим файловым ресурсам Azure и Windows Server. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Выполните следующие команды PowerShell, чтобы создать конечную точку сервера, и обязательно замените `<your-server-endpoint-path>` `<your-volume-free-space>` и с нужными значениями и проверьте дополнительный параметр для необязательной политики начальной загрузки.

```powershell
$serverEndpointPath = "<your-server-endpoint-path>"
$cloudTieringDesired = $true
$volumeFreeSpacePercentage = <your-volume-free-space>
# Optional property. Choose from: [NamespaceOnly] default when cloud tiering is enabled. [NamespaceThenModifiedFiles] default when cloud tiering is disabled. [AvoidTieredFiles] only available when cloud tiering is disabled.
$initialDownloadPolicy = NamespaceOnly

if ($cloudTieringDesired) {
    # Ensure endpoint path is not the system volume
    $directoryRoot = [System.IO.Directory]::GetDirectoryRoot($serverEndpointPath)
    $osVolume = "$($env:SystemDrive)\"
    if ($directoryRoot -eq $osVolume) {
        throw [System.Exception]::new("Cloud tiering cannot be enabled on the system volume")
    }

    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath `
        -CloudTiering `
        -VolumeFreeSpacePercent $volumeFreeSpacePercentage `
        -InitialDownloadPolicy $initialDownloadPolicy
} else {
    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath `
        -InitialDownloadPolicy $initialDownloadPolicy
}
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы создать новую конечную точку сервера, используйте команду [AZ storagesync Sync-Group Server-Endpoint](/cli/azure/ext/storagesync/storagesync/sync-group/server-endpoint#ext-storagesync-az-storagesync-sync-group-server-endpoint-create) .

```azurecli
# Create a new sync group server endpoint 
az storagesync sync-group server-endpoint create --resource-group myResourceGroupName \
                                                 --name myNewServerEndpointName
                                                 --registered-server-id 91beed22-7e9e-4bda-9313-fec96c286e0
                                                 --server-local-path d:\myPath
                                                 --storage-sync-service myStorageSyncServiceNAme
                                                 --sync-group-name mySyncGroupName

# Create a new sync group server endpoint with additional optional parameters
az storagesync sync-group server-endpoint create --resource-group myResourceGroupName \
                                                 --storage-sync-service myStorageSyncServiceName \
                                                 --sync-group-name mySyncGroupName \
                                                 --name myNewServerEndpointName \
                                                 --registered-server-id 91beed22-7e9e-4bda-9313-fec96c286e0 \
                                                 --server-local-path d:\myPath \
                                                 --cloud-tiering on \
                                                 --volume-free-space-percent 85 \
                                                 --tier-files-older-than-days 15 \
                                                 --initial-download-policy NamespaceOnly [OR] NamespaceThenModifiedFiles [OR] AvoidTieredFiles
                                                 --offline-data-transfer on \
                                                 --offline-data-transfer-share-name myfilesharename \

```

---

## <a name="configure-firewall-and-virtual-network-settings"></a>Настройка параметров брандмауэра и виртуальной сети

### <a name="portal"></a>Портал
Если вы хотите настроить службу синхронизации файлов Azure для работы с параметрами брандмауэра и виртуальной сети, выполните следующие действия.

1. В портал Azure перейдите к учетной записи хранения, которую необходимо защитить.
1. Нажмите кнопку **брандмауэры и виртуальные сети** в меню слева.
1. Выберите **Выбранные сети** в разделе **Разрешить доступ из**.
1. Убедитесь, что IP-адрес или виртуальная сеть серверов указана в соответствующем разделе.
1. Убедитесь, что установлен флажок **разрешить доверенным службам Майкрософт доступ к этой учетной записи хранения** .
1. Нажмите кнопку **Save** (Сохранить), чтобы сохранить настройки.

![Настройка параметров брандмауэра и виртуальной сети для работы со службой синхронизации файлов Azure](media/storage-sync-files-deployment-guide/firewall-and-vnet.png)

## <a name="onboarding-with-azure-file-sync"></a>Подключение с помощью службы синхронизации файлов Azure
Рекомендуемые действия по подключению Синхронизация файлов Azure в первый раз с нулевым временем простоя при сохранении полной точности файла и списка управления доступом (ACL), приведены ниже.
 
1. Разверните службу синхронизации хранилища.
1. Создайте группу синхронизации.
1. Установите агент службы синхронизации файлов Azure на сервере с полным набором данных.
1. Зарегистрируйте этот сервер и создайте конечную точку сервера на общем ресурсе. 
1. Дождитесь, пока служба синхронизация выполнит полную отправку данных на общий файловый ресурс Azure (в облачную конечную точку).  
1. После завершения первоначальной отправки установите агент синхронизации файлов Azure на всех оставшихся серверах.
1. Создайте новые общие файловые ресурсы на всех оставшихся серверах.
1. Создайте серверные конечные точки на новых файловых ресурсах с политикой распределения по уровням облака, если необходимо. (Для этого требуется дополнительное хранилище, доступное для первоначальной настройки.)
1. Пусть Синхронизация файлов Azure Agent выполняет быстрое восстановление полного пространства имен без фактической пересылки данных. После полной синхронизации пространства имен служба синхронизации заполнит место на локальном диске на основе политики распределения по уровням облака для серверной конечной точки. 
1. Дождитесь завершения синхронизации и протестируйте топологию нужным образом. 
1. Перенаправьте пользователей и приложения в новую общую папку.
1. При необходимости можно удалить все повторяющиеся общие папки на серверах.
 
Если дополнительное место для первоначального подключения отсутствует, но требуется присоединиться к существующим общим папкам, можно предварительно заполнить данными общие файловые ресурсы Azure. Такой подход рекомендуется, только если вы допускаете время простоя и можете гарантировать отсутствие изменений данных на общих ресурсах сервера в процессе исходного подключения. 
 
1. Убедитесь, что данные на любом из серверов не могут быть изменены во время процесса адаптации.
1. Предварительное заполнение файловых ресурсов Azure данными сервера с помощью любого средства передачи данных по протоколу SMB. Например, Robocopy. Вы также можете использовать AzCopy поверх остальных. Не забудьте использовать AzCopy с соответствующими параметрами для сохранения меток времени и атрибутов ACL.
1. Создайте топологию службы синхронизации файлов Azure с нужными серверными конечными точками, указывающими на существующие общие папки.
1. Дождитесь, пока процесс синхронизации завершит сверку на всех конечных точках. 
1. После этого можно открыть общие ресурсы для внесения изменений.
 
В настоящее время подход предварительного заполнения имеет несколько ограничений. 
- Изменения данных на сервере до полного запуска топологии синхронизации могут вызывать конфликты на серверных конечных точках.  
- После создания облачной конечной точки Синхронизация файлов Azure запускает процесс обнаружения файлов в облаке перед началом начальной синхронизации. Время, затраченное на выполнение этого процесса, зависит от различных факторов, таких как скорость сети, доступная пропускная способность и число файлов и папок. По грубой оценке в рамках предварительной версии процесс обнаружения выполняется со скоростью приблизительно 10 файлов в секунду. Таким образом, даже если предварительное заполнение завершится быстро, полный запуск системы может выполняться значительно дольше, когда данные предварительно заполняются в облаке.

## <a name="self-service-restore-through-previous-versions-and-vss-volume-shadow-copy-service"></a>Самостоятельное восстановление с помощью предыдущих версий и VSS (служба теневого копирования томов)

> [!IMPORTANT]
> Следующие сведения можно использовать только с версией 9 (или более поздней) агента синхронизации хранилища. Версии ниже 9 не будут иметь командлеты Сторажесинкселфсервице.

Предыдущие версии — это функция Windows, которая позволяет использовать на стороне сервера моментальные снимки VSS тома для представления restorable версий файла клиенту SMB.
Это позволяет использовать мощный сценарий, обычно называемый самостоятельной восстановлением, непосредственно для информационных работников, а не в зависимости от восстановления от ИТ-администратора.

Моментальные снимки VSS и предыдущие версии работают независимо от Синхронизация файлов Azure. Однако распределение по уровням облака должно быть установлено в совместимом режиме. На одном томе может существовать множество конечных точек сервера Синхронизация файлов Azure. Необходимо сделать следующий вызов PowerShell для каждого тома, который содержит даже одну конечную точку сервера, где вы планируете или используете распределение по уровням облака.

```powershell
Import-Module '<SyncAgentInstallPath>\StorageSync.Management.ServerCmdlets.dll'
Enable-StorageSyncSelfServiceRestore [-DriveLetter] <string> [[-Force]] 
```

Моментальные снимки VSS берутся из всего тома. По умолчанию для данного тома может существовать до 64 моментальных снимков, что дает достаточно места для хранения моментальных снимков. VSS обрабатывает это автоматически. Расписание моментальных снимков по умолчанию принимает два моментальных снимка в день с понедельника по пятницу. Это расписание можно настроить с помощью запланированной задачи Windows. Приведенный выше командлет PowerShell выполняет два действия:
1. Он настраивает синхронизацию облачных уровней на указанном томе, чтобы обеспечить совместимость с предыдущими версиями, и гарантирует, что файл можно восстановить из предыдущей версии, даже если он был распределен по уровням в облаке на сервере. 
1. Он включает расписание VSS по умолчанию. Затем его можно будет изменить позже. 

> [!Note]  
> При этом необходимо обратить внимание на два важных момента.
>- Если вы используете параметр-Force и служба VSS в настоящее время включена, она перезапишет текущее расписание моментальных снимков VSS и заменит его расписанием по умолчанию. Перед запуском командлета убедитесь, что вы сохранили пользовательскую конфигурацию.
> - Если вы используете этот командлет на узле кластера, необходимо также запустить его на всех остальных узлах в кластере. 

Чтобы узнать, включена ли совместимость самостоятельного восстановления, можно выполнить следующий командлет.

```powershell
Get-StorageSyncSelfServiceRestore [[-Driveletter] <string>]
```

В нем будут перечислены все тома на сервере, а также количество дней, совместимых с распределением по уровням облака для каждого из них. Это значение вычисляется автоматически на основе максимально возможного количества моментальных снимков на том и расписания моментальных снимков по умолчанию. Поэтому по умолчанию все предыдущие версии, представленные для информационного работника, можно использовать для восстановления из. То же самое справедливо и при изменении расписания по умолчанию для создания большего количества моментальных снимков.
Однако если изменить расписание таким образом, что приведет к доступному моментальному снимку тома, который старше значения "совместимые дни", пользователи не смогут использовать этот старый моментальный снимок (Предыдущая версия) для восстановления из.

> [!Note]
> Включение самостоятельного восстановления может повлиять на потребление и выставление счета за использование службы хранилища Azure. Это влияние ограничивается файлами, которые в настоящее время распределены по уровням на сервере. Включение этой функции гарантирует, что в облаке доступна версия файла, на которую можно ссылаться с помощью записи предыдущих версий (моментальных снимков VSS).
>
> Если отключить эту функцию, использование хранилища Azure будет медленно отклоняться до тех пор, пока не будет пройдено окно "совместимые дни". Это не позволяет ускорить работу. 

По умолчанию максимальное количество моментальных снимков VSS на том (64), а также расписание, используемое по умолчанию, приводят к максимуму 45 дней предыдущих версий, в зависимости от количества моментальных снимков VSS, которые можно хранить на томе.

Если параметр max. 64 моментальных снимков VSS на том не подходит для вас, это значение можно [изменить с помощью раздела реестра](/windows/win32/backup/registry-keys-for-backup-and-restore#maxshadowcopies).
Чтобы новое ограничение вступило в силу, необходимо повторно запустить командлет, чтобы включить предыдущую совместимость версий для всех ранее включенных томов, с флагом-Force, который принимает новое максимальное число моментальных снимков VSS на том. Это приведет к появлению нового вычисленного числа совместимых дней. Обратите внимание, что это изменение вступит в силу только для новых многоуровневых файлов и перезапишет настройки, которые могли быть сделаны в расписании VSS.

<a id="proactive-recall"></a>
## <a name="proactively-recall-new-and-changed-files-from-an-azure-file-share"></a>Заранее отозвать новые и измененные файлы из общей папки Azure

При использовании агента версии 11 в конечной точке сервера будет доступен новый режим. Этот режим позволяет глобально распределенным компаниям использовать кэш сервера в удаленном регионе, предварительно заполненный до тех пор, пока локальные пользователи не будут получать доступ к файлам. При включении в конечной точке сервера этот режим приведет к тому, что этот сервер будет отзывать файлы, созданные или измененные в общем файловом ресурсе Azure.

### <a name="scenario"></a>Сценарий

Глобально распределенная компания имеет филиалы в США и Индии. В утром (США) информационные работники создают новую папку и новые файлы для нового проекта, а также работают со всеми днями. Синхронизация файлов Azure будет синхронизировать папки и файлы в общую папку Azure (облачная конечная точка). Информационные работники в Индии продолжат работу над проектом в своем часовом поясе. Когда они поступают утром, локальный сервер Синхронизация файлов Azure с поддержкой в Индии должен иметь доступ к этим новым файлам локально, чтобы группа Индии могла эффективно работать с локальным кэшем. Включение этого режима предотвращает медленный доступ к файлам из-за отзыва по требованию и позволяет серверу заранее отозвать файлы сразу после их изменения или создания в общей папке Azure.

> [!IMPORTANT]
> Важно понимать, что отслеживание изменений в файловом ресурсе Azure, которые близко к серверу, может увеличить трафик исходящего трафика и счет за использование Azure. Если файлы, отозванные на сервере, на самом деле не требуются локально, ненужный отзыв на сервере может негативно сказаться. Используйте этот режим, если вы уверены, что предварительно заполнение кэша на сервере с недавними изменениями в облаке повлияет на пользователей или приложения, использующие файлы на этом сервере.

### <a name="enable-a-server-endpoint-to-proactively-recall-what-changed-in-an-azure-file-share"></a>Включение конечной точки сервера для упреждающего отзыва изменений в общей папке Azure

# <a name="portal"></a>[Портал](#tab/proactive-portal)

1. В [портал Azure](https://portal.azure.com/)перейдите к службе синхронизации хранилища, выберите правильную группу синхронизации, а затем укажите конечную точку сервера, для которой необходимо тщательно отслеживать изменения в общей папке Azure (облачная конечная точка).
1. В разделе распределение по уровням облака найдите раздел "Загрузка общей папки Azure". Вы увидите выбранный режим и можете изменить его, чтобы более точно отслеживать изменения в общей папке Azure и заранее отзывать их на сервере.

:::image type="content" source="media/storage-sync-files-deployment-guide/proactive-download.png" alt-text="Изображение, показывающее поведение скачивания файлового ресурса Azure для конечной точки сервера, которое в настоящий момент действует, и кнопку для открытия меню, позволяющее изменить его.":::

# <a name="powershell"></a>[PowerShell](#tab/proactive-powershell)

Вы можете изменить свойства конечной точки сервера в PowerShell с помощью командлета [Set-азсторажесинксерверендпоинт](/powershell/module/az.storagesync/set-azstoragesyncserverendpoint) .

```powershell
# Optional parameter. Default: "UpdateLocallyCachedFiles", alternative behavior: "DownloadNewAndModifiedFiles"
$recallBehavior = "DownloadNewAndModifiedFiles"

Set-AzStorageSyncServerEndpoint -InputObject <PSServerEndpoint> -LocalCacheMode $recallBehavior
```

---

## <a name="migrate-a-dfs-replication-dfs-r-deployment-to-azure-file-sync"></a>Перенос развертывания репликации DFS (DFS-R) в службу синхронизации файлов Azure
Чтобы перенести развертывание DFS-R в службу "Синхронизация файлов Azure", сделайте следующее:

1. Создайте группу синхронизации для представления топологии DFS-R, которую требуется заменить.
2. Начните с сервера с полным набором данных в топологии DFS-R, которую следует перенести. Установите службу "Синхронизация файлов" Azure на этом сервере.
3. Зарегистрируйте этот сервер и создайте серверную конечную точку для переноса первого сервера. Не включайте распределение по уровням облака.
4. Выполните синхронизацию всех данных с файловым ресурсом Azure (облачной конечной точкой).
5. Установите и зарегистрируйте агент синхронизации файлов Azure на каждом из оставшихся серверов DFS-R.
6. Отключите DFS-R. 
7. Создайте серверную конечную точку на каждом из серверов DFS-R. Не включайте распределение по уровням облака.
8. Дождитесь завершения синхронизации и протестируйте топологию нужным образом.
9. Снимите DFS-R с учета.
10. Теперь можно включить распределение по уровням облака на любой серверной конечной точке.

Дополнительные сведения см. в статье [Azure File Sync interop with Distributed File System (DFS)](storage-sync-files-planning.md#distributed-file-system-dfs) (Взаимодействие службы "Синхронизация файлов Azure" с распределенной файловой системой (DFS)).

## <a name="next-steps"></a>Дальнейшие действия
- [Добавление и удаление конечных точек сервера службы синхронизации файлов Azure](storage-sync-files-server-endpoint.md)
- [Регистрация и отмена регистрации сервера в службе синхронизации файлов Azure (предварительная версия)](storage-sync-files-server-registration.md)
- [Мониторинг Синхронизации файлов Azure](storage-sync-files-monitoring.md)

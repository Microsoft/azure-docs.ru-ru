---
title: Инструкции по настройке компьютера с шаблонами Windows | Документация Майкрософт
description: Универсальные шаги для подготовки компьютера шаблона Windows в службах лаборатории.  Эти действия включают в себя настройку Центр обновления Windows расписания, установку OneDrive и установку Office.
author: EMaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 216dc843b31eac355e1d818014f3d70b2ef83132
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94647908"
---
# <a name="guide-to-setting-up-a-windows-template-machine-in-azure-lab-services"></a>Инструкции по настройке компьютера шаблона Windows в службах лаборатории Azure

Если вы настраиваете компьютер с шаблоном Windows 10 для служб лаборатории Azure, вот несколько рекомендаций и советов, которые следует учитывать. Ниже приведены все этапы настройки.  Однако эти подготовительные шаги помогут повысить производительность учащихся, сократить время работы в классе и убедиться, что они используют новейшие технологии.

>[!IMPORTANT]
>В этой статье содержатся фрагменты кода PowerShell для упрощения процесса изменения шаблона компьютера.  Для всех показанных сценариев PowerShell их необходимо запустить в Windows PowerShell с правами администратора. В Windows 10 это быстрый способ сделать это, щелкнув правой кнопкой мыши меню Пуск и выбрав пункт "Windows PowerShell (администратор)".

## <a name="install-and-configure-onedrive"></a>Установка и настройка OneDrive

Чтобы защитить данные учащихся от потери при сбросе виртуальной машины, мы рекомендуем учащимся передачу данных в облако.  Microsoft OneDrive может помочь учащимся защитить свои данные.  

### <a name="install-onedrive"></a>Установка OneDrive

Чтобы вручную скачать и установить OneDrive, см. страницы скачивания [onedrive](https://onedrive.live.com/about/download/) или [onedrive для бизнеса](https://onedrive.live.com/about/business/) .

Можно также использовать следующий сценарий PowerShell.  Будет автоматически загружена и установлена последняя версия OneDrive.  После установки клиента OneDrive запустите установщик.  В нашем примере мы используем параметр, `/allUsers` чтобы установить OneDrive для всех пользователей на компьютере. Мы также используем `/silent` параметр для автоматической установки OneDrive.

```powershell
Write-Host "Downloading OneDrive Client..."
$DownloadPath = "$env:USERPROFILE/Downloads/OneDriveSetup.exe"
if((Test-Path $DownloadPath) -eq $False )
{
    Write-Host "Downloading OneDrive..."
    $web = new-object System.Net.WebClient
    $web.DownloadFile("https://go.microsoft.com/fwlink/p/?LinkId=248256",$DownloadPath)
} else {
    Write-Host "OneDrive installer already exists at " $DownloadPath
}

Write-Host "Installing OneDrive..."
& $env:USERPROFILE/Downloads/OneDriveSetup.exe /allUsers /silent
```

### <a name="onedrive-customizations"></a>Настройки OneDrive

Существует множество [настроек, которые можно выполнить в OneDrive](/onedrive/use-group-policy). Рассмотрим некоторые из наиболее распространенных настроек.

#### <a name="silently-move-windows-known-folders-to-onedrive"></a>Автоматическое перемещение известных папок Windows в OneDrive

Папки, такие как документы, загружаемые файлы и рисунки, часто используются для хранения файлов учащихся. Чтобы обеспечить резервное копирование этих папок в OneDrive, рекомендуется переместить эти папки в OneDrive.

Если вы используете компьютер, который не использует Active Directory, пользователи могут вручную переместить эти папки в OneDrive после проверки подлинности в OneDrive.

1. Открытие проводника
2. Щелкните правой кнопкой мыши папку документы, загрузки или изображения.
3. Перейдите в раздел свойства > расположение.  Переместите папку в новую папку в каталоге OneDrive.

Если виртуальная машина подключена к Active Directory, можно настроить компьютер-шаблон, чтобы автоматически запрашивать у учащихся перемещение известных папок в OneDrive.  

Сначала необходимо получить идентификатор организации.  Дополнительные инструкции см. [в статье Поиск идентификатора Microsoft 365 организации](/onedrive/find-your-office-365-tenant-id).  ИДЕНТИФИКАТОР организации также можно получить с помощью следующей команды PowerShell.

```powershell
Install-Module MSOnline -Confirm
Connect-MsolService
$officeTenantID = Get-MSOLCompanyInformation |
    Select-Object -expand objectID |
    Select-Object -expand Guid
```

Получив идентификатор организации, укажите в OneDrive запрос на перемещение известных папок в OneDrive с помощью следующей команды PowerShell.

```powershell
if ($officeTenantID -eq $null)
{
        Write-Error "Variable `$officeTenantId must be set to your Office Tenant Id before continuing."
}
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "KFMSilentOptIn" -Value $officeTenantID -PropertyType STRING
```

### <a name="use-onedrive-files-on-demand"></a>Использование файлов OneDrive по запросу

Студенты могут иметь много файлов в своих учетных записях OneDrive. Чтобы сэкономить место на компьютере и сократить время загрузки, рекомендуется сделать так, чтобы все файлы, хранящиеся в учетной записи OneDrive учащегося, были по запросу.  Файлы по требованию скачиваются только после того, как пользователь обращается к файлу.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive" -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "FilesOnDemandEnabled" -Value "00000001" -PropertyType DWORD
```

### <a name="silently-sign-in-users-to-onedrive"></a>Автоматический вход пользователей в OneDrive

В OneDrive можно настроить автоматический вход с использованием учетных данных Windows вошедшего в систему пользователя.  Автоматический вход в систему полезен для классов, в которых учащийся входит в учетные данные учебного заведения.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "SilentAccountConfig" -Value "00000001" -PropertyType DWORD
```

### <a name="disable-the-tutorial-that-appears-at-the-end-of-onedrive-setup"></a>Отключение учебника, который отображается в конце установки OneDrive

Этот параметр позволяет запретить запуск руководства в веб-браузере в конце установки OneDrive.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive" -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
    -Name "DisableTutorial" -Value "00000001" -PropertyType DWORD -Force
```

### <a name="set-the-maximum-size-of-a-file-that-to-be-download-automatically"></a>Задать максимальный размер файла для автоматического скачивания

Этот параметр используется вместе с автоматическим входом пользователей в клиент синхронизации OneDrive с учетными данными Windows на устройствах, на которых не включены файлы OneDrive по запросу. Любой пользователь, имеющий OneDrive, размер которого превышает указанное пороговое значение (в МБ), будет получать запрос на выбор папок, которые требуется синхронизировать, прежде чем клиент синхронизации OneDrive (OneDrive.exe) загрузит эти файлы.  В нашем примере "1111-2222-3333-4444" — это идентификатор организации, а 0005000 устанавливает пороговое значение 5 ГБ.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive"
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive\DiskSpaceCheckThresholdMB"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\OneDrive\DiskSpaceCheckThresholdMB"
    -Name "1111-2222-3333-4444" -Value "0005000" -PropertyType DWORD
```

## <a name="install-and-configure-microsoft-365"></a>Установка и настройка Microsoft 365

### <a name="install-microsoft-365"></a>Установка Microsoft 365

Если компьютеру-шаблону требуется Office, рекомендуется установить Office с помощью [средства развертывания Office (ODT)](https://www.microsoft.com/download/details.aspx?id=49117). Вам потребуется создать файл конфигурации с возможностью повторного использования с помощью [центра администрирования Microsoft 365 Apps](https://config.office.com/) , чтобы выбрать архитектуру, функции, необходимые для Office, и частоту обновления.

1. Перейдите в [центр администрирования Microsoft 365 Apps](https://config.office.com/) и скачайте собственный файл конфигурации.
2. Загрузите [средство развертывания Office](https://www.microsoft.com/download/details.aspx?id=49117).  Скачанный файл будет иметь расширение `setup.exe` .
3. Запустите `setup.exe /download configuration.xml` , чтобы скачать компоненты Office.
4. Выполните команду `setup.exe /configure configuration.xml` , чтобы установить компоненты Office.

### <a name="change-the-microsoft-365-update-channel"></a>Изменение канала обновления Microsoft 365

С помощью средства настройки Office можно задать частоту, с которой Office будет получать обновления. Однако если необходимо изменить частоту получения обновлений Office после установки, можно изменить URL-адрес канала обновления. URL-адреса каналов обновления можно найти по адресу [изменение канала обновления Microsoft 365Ных приложений для устройств в вашей организации](/deployoffice/change-update-channels). В приведенном ниже примере показано, как настроить Microsoft 365 для использования ежемесячного канала обновления.

```powershell
# Update to the Microsoft 365 Monthly Channel
Set-ItemProperty
    -Path "HKLM:\SOFTWARE\Microsoft\Office\ClickToRun\Configuration\CDNBaseUrl"
    -Name "CDNBaseUrl"
    -Value "http://officecdn.microsoft.com/pr/492350f6-3a01-4f97-b9c0-c7c6ddf67d60"
```

## <a name="install-and-configure-updates"></a>Установка и настройка обновлений

### <a name="install-the-latest-windows-updates"></a>Установка последних обновлений Windows

Рекомендуется установить последние обновления Майкрософт на компьютере шаблона для обеспечения безопасности перед публикацией виртуальной машины шаблона.  Это также позволяет избежать нарушения работы учащихся при выполнении обновлений в непредвиденное время.

1. Запуск **параметров** из меню "Пуск"
2. Щелкните **обновление** & безопасность.
3. Щелкните " **проверить наличие обновлений** "
4. Обновления будут скачаны и установлены.

Вы также можете использовать PowerShell для обновления шаблона компьютера.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Confirm
Install-Module PSWindowsUpdate -Confirm
Install-WindowsUpdate -MicrosoftUpdate
Set-ExecutionPolicy default -Force
```

>[!NOTE]
>Для некоторых обновлений может потребоваться перезапустить компьютер.  Вам будет предложено, требуется ли перезагрузка.

### <a name="install-the-latest-updates-for-microsoft-store-apps"></a>Установка последних обновлений для приложений Microsoft Store

Рекомендуется, чтобы все Microsoft Store приложения были обновлены до последних версий.  Ниже приведены инструкции по обновлению приложений вручную с Microsoft Store.  

1. Запустите приложение **Microsoft Store** .
2. Нажмите кнопку с многоточием (...) рядом с фотографией пользователя в верхнем углу приложения.
3. В раскрывающемся меню выберите **скачать** и обновить.
4. Нажмите кнопку **получить обновление** .

Вы также можете использовать PowerShell для обновления Microsoft Store уже установленных приложений.

```powershell
(Get-WmiObject -Namespace "root\cimv2\mdm\dmmap" -Class "MDM_EnterpriseModernAppManagement_AppManagement01").UpdateScanMethod()
```

### <a name="stop-automatic-windows-updates"></a>Отключить автоматические обновления Windows

После обновления Windows до последней версии может быть необходимо остановить обновления Windows.  Автоматическое обновление может привести к конфликту с запланированным временем класса.  Если ваш курс больше всего работает, рассмотрите вопрос о том, следует ли учащимся вручную проверять наличие обновлений или устанавливать автоматическое обновление в течение времени, находящегося за пределами запланированных часов класса.  Дополнительные сведения о параметрах настройки для Центр обновления Windows см. в разделе [Управление дополнительными параметрами центр обновления Windows](/windows/deployment/update/waas-wu-settings).

Автоматическое обновление Windows может быть остановлено с помощью следующего сценария PowerShell.

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AU"
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AU"
    -Name "NoAutoUpdate" -Value "1" -PropertyType DWORD
```

## <a name="install-foreign-language-packs"></a>Установка иностранных языковых пакетов

Если на виртуальной машине должны быть установлены дополнительные языки, их можно добавить с помощью Microsoft Store.

1. Запустить Microsoft Store
2. Выполните поиск по фразе "языковой пакет"
3. Выберите язык для установки

Если вы уже выполнили вход в виртуальную машину шаблона, воспользуйтесь сочетанием клавиш "установить языковой пакет" ( `ms-settings:regionlanguage?activationSource=SMC-IA-4027670` ), чтобы перейти непосредственно к соответствующей странице параметров.

## <a name="remove-unneeded-built-in-apps"></a>Удаление ненужных встроенных приложений

В состав Windows 10 входит множество встроенных приложений, которые могут не потребоваться для конкретного класса. Чтобы упростить образ компьютера для учащихся, может потребоваться удалить некоторые приложения с компьютера с шаблоном.  Чтобы просмотреть список установленных приложений, используйте `Get-AppxPackage` командлет PowerShell.  В следующем примере показаны все установленные приложения, которые можно удалить.

```powershell
Get-AppxPackage | Where {$_.NonRemovable -eq $false} | select Name
```

Чтобы удалить приложение, используйте командлет Remove-Appx.  В приведенном ниже примере показано, как удалить все связанные с XBox устройства.

```powershell
Get-AppxPackage -Name *xbox* | foreach { if (-not $_.NonRemovable) { Remove-AppxPackage $_} }
```

## <a name="install-common-teaching-related-applications"></a>Установка общих приложений, связанных с обобучением

Установите другие приложения, которые обычно используются для обучения в приложении Магазина Windows. К таким предложениям относятся такие приложения, как [Microsoft доски](https://www.microsoft.com/store/productId/9MSPC6MP8FM4), [Microsoft Teams](https://www.microsoft.com/store/productId/9MSPC6MP8FM4)и [Minecraft образовательный выпуск](https://education.minecraft.net/). Эти приложения необходимо установить вручную в магазине Windows или на соответствующих веб-сайтах на виртуальной машине шаблона.

## <a name="conclusion"></a>Заключение

В этой статье показаны необязательные шаги по подготовке виртуальной машины шаблона Windows для эффективного класса.  Шаги включают установку OneDrive и установку Microsoft 365, установку обновлений для Windows и установку обновлений для приложений Microsoft Store.  Мы также рассмотрели, как настроить обновления по расписанию, которое лучше подходит для вашего класса.  

## <a name="next-steps"></a>Дальнейшие действия
См. статью о том, как управлять поведением завершения работы Windows, чтобы упростить управление затратами: [руководство по управлению работой при завершении работы Windows](how-to-windows-shutdown.md)
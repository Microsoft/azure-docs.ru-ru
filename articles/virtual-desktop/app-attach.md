---
title: Настройка виртуальных рабочих столов Windows для подключения приложения MSIX сценарии PowerShell в Azure
description: Создание скриптов PowerShell для подключения приложения MSIX к виртуальному рабочему столу Windows.
author: Heidilohr
ms.topic: how-to
ms.date: 12/14/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 5e45c51735e0b7ab4b263d3f3047b5848c82439d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98185773"
---
# <a name="create-powershell-scripts-for-msix-app-attach-preview"></a>Создание скриптов PowerShell для присоединения приложения MSIX (Предварительная версия)

> [!IMPORTANT]
> Присоединение приложения MSIX в настоящее время находится в общедоступной предварительной версии.
> без соглашений об уровне обслуживания. Мы не рекомендуем использовать ее для выполнения производственных рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этом разделе описано, как настроить сценарии PowerShell для подключения к приложению MSIX.

>[!IMPORTANT]
>Прежде чем приступить к работе, обязательно заполните и отправьте [эту форму](https://aka.ms/enablemsixappattach) , чтобы включить присоединение приложения MSIX в вашей подписке. Если у вас нет утвержденного запроса, присоединение приложения MSIX не будет работать. Утверждение запросов может занять до 24 часов в рабочие дни. После принятия и завершения запроса вы получите сообщение электронной почты.

## <a name="install-certificates"></a>Установка сертификатов

Необходимо установить сертификаты на всех узлах сеансов в пуле узлов, где будет размещаться ТД из приложения MSIX, которое присоединяет пакеты.

Если в приложении используется сертификат, который не является общедоступным доверенным или является самозаверяющим, установите его следующим образом:

1. Щелкните пакет правой кнопкой мыши и выберите **Свойства**.
2. В открывшемся окне выберите вкладку **Цифровые подписи**. В списке на этой вкладке должен быть указан только один элемент, как показано ниже. Выдерите этот элемент, а затем выберите **Сведения**.
3. Когда появится окно сведения о цифровой подписи, перейдите на вкладку **Общие** , выберите **Просмотр сертификата**, а затем — **установить сертификат**.
4. Когда откроется установщик, выберите установку на **локальный компьютер** и нажмите **Далее**.
5. Если установщик спросит, нужно ли разрешить приложению вносить изменения на вашем устройстве, выберите **Да**.
6. Выберите **Поместить все сертификаты в следующее хранилище** и нажмите **Обзор**.
7. Когда откроется окно хранилища сертификатов, выберите **Доверенные лица** и нажмите **ОК**.
8. Выберите **Далее** и **Готово**.

## <a name="enable-microsoft-hyper-v"></a>Включить Microsoft Hyper-V

Microsoft Hyper-V должны быть включены, так как `Mount-VHD` команда необходима для ее размещения и `Dismount-VHD` необходима для ее отмены.

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

>[!NOTE]
>Чтобы это вступило в силу, перезапустите виртуальную машину.

## <a name="prepare-powershell-scripts-for-msix-app-attach"></a>Подготовка сценариев PowerShell для подключения к приложению MSIX

Подключение приложения MSIX состоит из четырех этапов, которые необходимо выполнить в следующем порядке:

1. Этап
2. Зарегистрировать
3. Отмена регистрации
4. Отмена размещения

На каждом этапе создается сценарий PowerShell. Примеры сценариев для каждого этапа см. [здесь](https://github.com/Azure/RDS-Templates/tree/master/msix-app-attach).

### <a name="stage-powershell-script"></a>Скрипт этапа PowerShell

Перед обновлением сценариев PowerShell убедитесь, что вы знаете GUID тома на виртуальном жестком диске. Чтобы узнать GUID тома, выполните следующие действия.

1.  Откройте сетевую папку с виртуальным жестким диском на той виртуальной машине, где вы собираетесь выполнить сценарий.

2.  Щелкните виртуальный жесткий диск правой кнопкой мыши файл и выберите **Подключить**. Виртуальный жесткий диск будет подключен к соответствующему диску.

3.  После подключения виртуального жесткого диска откроется **Проводник**. Запишите родительскую папку и обновите переменную **$parentFolder**.

    >[!NOTE]
    >Если родительская папка не отображается, это значит, что MSIX был развернут неправильно. Повторите действия, описанные в предыдущем разделе, и попробуйте еще раз.

4.  Откройте родительскую папку. При правильном развертывании вы увидите папку с таким же именем, как у пакета. Обновите переменную **$packageName**, присвоив ей имя этой папки.

    Например, `VSCodeUserSetup-x64-1.38.1_1.38.1.0_x64__8wekyb3d8bbwe`.

5.  Откройте окно командной строки и введите **mountvol**. Эта команда открывает список томов и их GUID. Скопируйте GUID тома, в котором буква диска соответствует диску, к которому вы подключили виртуальный жесткий диск в шаге 2.

    Например, в этом примере выходных данных команды mountvol при подключении виртуального жесткого диска к диску C необходимо скопировать значение, которое находится над `C:\`.

    ```cmd
    Possible values for VolumeName along with current mount points are:

    \\?\Volume{a12b3456-0000-0000-0000-10000000000}\
    *** NO MOUNT POINTS ***

    \\?\Volume{c78d9012-0000-0000-0000-20000000000}\
        E:\

    \\?\Volume{d34e5678-0000-0000-0000-30000000000}\
        C:\

    ```


6.  Обновите переменную **$volumeGuid**, присвоив ей только что скопированный GUID тома.

7. Откройте командную строку PowerShell с правами администратора и обновите следующий сценарий PowerShell, указав переменные, которые относятся к вашей среде.

    ```powershell
    #MSIX app attach staging sample

    #region variables
    $vhdSrc="<path to vhd>"
    $packageName = "<package name>"
    $parentFolder = "<package parent folder>"
    $parentFolder = "\" + $parentFolder + "\"
    $volumeGuid = "<vol guid>"
    $msixJunction = "C:\temp\AppAttach\"
    #endregion

    #region mountvhd
    try
    {
          Mount-Diskimage -ImagePath $vhdSrc -NoDriveLetter -Access ReadOnly
          Write-Host ("Mounting of " + $vhdSrc + " was completed!") -BackgroundColor Green
    }
    catch
    {
          Write-Host ("Mounting of " + $vhdSrc + " has failed!") -BackgroundColor Red
    }
    #endregion

    #region makelink
    $msixDest = "\\?\Volume{" + $volumeGuid + "}\"
    if (!(Test-Path $msixJunction))
    {
         md $msixJunction
    }

    $msixJunction = $msixJunction + $packageName
    cmd.exe /c mklink /j $msixJunction $msixDest
    #endregion

    #region stage
    [Windows.Management.Deployment.PackageManager,Windows.Management.Deployment,ContentType=WindowsRuntime] | Out-Null
    Add-Type -AssemblyName System.Runtime.WindowsRuntime
    $asTask = ([System.WindowsRuntimeSystemExtensions].GetMethods() | Where { $_.ToString() -eq 'System.Threading.Tasks.Task`1[TResult] AsTask[TResult,TProgress](Windows.Foundation.IAsyncOperationWithProgress`2[TResult,TProgress])'})[0]
    $asTaskAsyncOperation = $asTask.MakeGenericMethod([Windows.Management.Deployment.DeploymentResult], [Windows.Management.Deployment.DeploymentProgress])
    $packageManager = [Windows.Management.Deployment.PackageManager]::new()
    $path = $msixJunction + $parentFolder + $packageName # needed if we do the pbisigned.vhd
    $path = ([System.Uri]$path).AbsoluteUri
    $asyncOperation = $packageManager.StagePackageAsync($path, $null, "StageInPlace")
    $task = $asTaskAsyncOperation.Invoke($null, @($asyncOperation))
    $task
    #endregion
    ```

### <a name="register-powershell-script"></a>Регистрация сценария PowerShell

Чтобы выполнить сценарий регистрации, выполните следующие командлеты PowerShell, заменив значения заполнителей на значения, которые относятся к вашей среде.

```powershell
#MSIX app attach registration sample

#region variables
$packageName = "<package name>"
$path = "C:\Program Files\WindowsApps\" + $packageName + "\AppxManifest.xml"
#endregion

#region register
Add-AppxPackage -Path $path -DisableDevelopmentMode -Register
#endregion
```

### <a name="deregister-powershell-script"></a>Отмена регистрации сценария PowerShell

Для этого сценария замените заполнитель переменной **$packageName** на имя проверяемого пакета.

```powershell
#MSIX app attach deregistration sample

#region variables
$packageName = "<package name>"
#endregion

#region deregister
Remove-AppxPackage -PreserveRoamableApplicationData $packageName
#endregion
```

### <a name="destage-powershell-script"></a>Отмена размещения сценария PowerShell

Для этого сценария замените заполнитель переменной **$packageName** на имя проверяемого пакета. В рабочем развертывании лучше запустить его при завершении работы.

```powershell
#MSIX app attach de staging sample

$vhdSrc="<path to vhd>"

#region variables
$packageName = "<package name>"
$msixJunction = "C:\temp\AppAttach"
#endregion

#region deregister
Remove-AppxPackage -AllUsers -Package $packageName
Remove-Item "$msixJunction\$packageName" -Recurse -Force -Verbose
#endregion

#region Detach VHD
Dismount-DiskImage -ImagePath $vhdSrc -Confirm:$false
#endregion
```

## <a name="set-up-simulation-scripts-for-the-msix-app-attach-agent"></a>Настройка сценариев моделирования для агента подключения приложения MSIX

После создания сценариев пользователи могут запустить их вручную или настроить на автоматический запуск в качестве сценариев запуска, входа в систему, выхода из системы и завершения работы. Дополнительные сведения об этих типах сценариев см. в разделе [Использование сценариев запуска, завершения работы, входа и выхода из системы в групповой политике](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn789196(v=ws.11)/).

Каждый из этих автоматических сценариев запускает один из этапов сценариев подключения приложения:

- Сценарий запуска запускает сценарий размещения.
- Сценарий входа запускает сценарий регистрации.
- Сценарий выхода из системы запускает сценарий отмены регистрации.
- Сценарий завершения работы запускает сценарий отмены размещения.

## <a name="use-packages-offline"></a>Использование пакетов в автономном режиме

При использовании пакетов из [Магазина Майкрософт для бизнеса](https://businessstore.microsoft.com/) или [Магазина Майкрософт для образовательных учреждений](https://educationstore.microsoft.com/) в сети или на устройствах, не подключенных к Интернету, для успешного запуска приложения необходимо получить лицензии пакета из Магазина и установить их. Если устройство подключено к сети и может подключаться к Магазину Майкрософт для бизнеса, необходимые лицензии должны загружаться автоматически, но, если вы работаете в автономном режиме, лицензии нужно будет настроить вручную.

Чтобы установить файлы лицензий, необходимо использовать сценарий PowerShell, который вызывает класс MDM_EnterpriseModernAppManagement_StoreLicenses02_01 в поставщике WMI Bridge.

Чтобы настроить лицензии для автономного использования, выполните следующие действия.

1. Скачайте пакет приложения, лицензии и необходимые платформы из Магазина Майкрософт для бизнеса. Вам нужны как закодированные, так и незакодированные файлы лицензий. Подробные инструкции по скачиванию см. [здесь](/microsoft-store/distribute-offline-apps#download-an-offline-licensed-app).
2. Обновите следующие переменные в сценарии для шага 3:
      1. `$contentID` — это значение ContentID из незакодированного файла лицензии (файл с расширением XML). Вы можете открыть файл лицензии в любом текстовом редакторе по своему усмотрению.
      2. `$licenseBlob` — это вся строка BLOB-объекта лицензии в закодированном файле лицензии (файл с расширением BIN). Вы можете открыть закодированный файл лицензии в любом текстовом редакторе по своему усмотрению.
3. Выполните следующий сценарий в командной строке администратора PowerShell. Лицензии лучше всего устанавливать в конце [сценария размещения](#stage-powershell-script), который также нужно выполнять из командной строки администратора PowerShell.

```powershell
$namespaceName = "root\cimv2\mdm\dmmap"
$className = "MDM_EnterpriseModernAppManagement_StoreLicenses02_01"
$methodName = "AddLicenseMethod"
$parentID = "./Vendor/MSFT/EnterpriseModernAppManagement/AppLicenses/StoreLicenses"

#TODO - Update $contentID with the ContentID value from the unencoded license file (.xml)
$contentID = "{'ContentID'_in_unencoded_license_file}"

#TODO - Update $licenseBlob with the entire String in the encoded license file (.bin)
$licenseBlob = "{Entire_String_in_encoded_license_file}"

$session = New-CimSession

#The final string passed into the AddLicenseMethod should be of the form <License Content="encoded license blob" />
$licenseString = '<License Content='+ '"' + $licenseBlob +'"' + ' />'

$params = New-Object Microsoft.Management.Infrastructure.CimMethodParametersCollection
$param = [Microsoft.Management.Infrastructure.CimMethodParameter]::Create("param",$licenseString ,"String", "In")
$params.Add($param)


try
{
   $instance = New-CimInstance -Namespace $namespaceName -ClassName $className -Property @{ParentID=$parentID;InstanceID=$contentID}
   $session.InvokeMethod($namespaceName, $instance, $methodName, $params)

}
catch [Exception]
{
     write-host $_ | out-string
}
```

## <a name="next-steps"></a>Дальнейшие действия

Эта функция пока не поддерживается, но вы можете задать вопросы [техническому сообществу по виртуальным рабочим столам Windows](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

Вы также можете оставить отзыв о виртуальном рабочем столе Windows в [центре обратной связи по виртуальным рабочим столам Windows](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).

---
title: Установка языковых пакетов на виртуальных машинах Windows 10 в виртуальном рабочем столе Windows — Azure
description: Установка языковых пакетов для многосеансовых виртуальных машин Windows 10 в виртуальном рабочем столе Windows.
author: Heidilohr
ms.topic: how-to
ms.date: 12/03/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: fc7892db2ca11ab7970835f8979360961ee01104
ms.sourcegitcommit: afb9e9d0b0c7e37166b9d1de6b71cd0e2fb9abf5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/14/2021
ms.locfileid: "103463346"
---
# <a name="add-language-packs-to-a-windows-10-multi-session-image"></a>Добавление языковых пакетов в многосеансовый образ Windows 10

Виртуальный рабочий стол Windows — это служба, которую пользователи могут развертывать в любое время в любом месте. Поэтому важно, чтобы пользователи могли настраивать, на каком языке отображается многосеансовый образ Windows 10 Enterprise.

Существует два способа, которым можно удовлетворять языковые потребности пользователей:

- Создавайте выделенные пулы узлов с настроенным образом для каждого языка.
- Помещайте пользователей с различными требованиями к языку и локализации в одном пуле узлов, но настраивайте свои образы, чтобы убедиться, что они могут выбрать нужный язык.

Последний метод является гораздо более эффективным и экономичным. Тем не менее, вам нужно решить, какой метод лучше подходит для ваших нужд. В этой статье показано, как настроить языки для образов.

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить многосеансовые образы Windows 10 Enterprise для добавления нескольких языков, необходимо выполнить следующие действия.

- Виртуальная машина Azure с многосеансовой поддержкой Windows 10 Enterprise, версия 1903 или более поздняя

- ISO-файл языка ISO, функция по запросу (FOD) и папка "Входящие" для приложений версии ОС, используемой в образе. Их можно загрузить здесь: 
     
     - ISO-язык:
        - [Windows 10, версия 1903 или 1909 языковой пакет (ISO)](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_CLIENTLANGPACKDVD_OEM_MULTI.iso)
        - [Windows 10, версия 2004 или 20H2 языковой пакет ISO](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_CLIENTLANGPACKDVD_OEM_MULTI.iso)

     - FOD диск 1 ISO:
        - [Windows 10, версия 1903 или 1909 FOD диск 1 ISO](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_amd64fre_FOD-PACKAGES_OEM_PT1_amd64fre_MULTI.iso)
        - [Windows 10, версия 2004 или 20H2 FOD диск 1 ISO](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_amd64fre_FOD-PACKAGES_OEM_PT1_amd64fre_MULTI.iso)
        
     - ISO-файл входящих приложений:
        - [Windows 10, версия 1903 или 1909 Входящие приложения (ISO)](https://software-download.microsoft.com/download/pr/18362.1.190318-1202.19h1_release_amd64fre_InboxApps.iso)
        - [Windows 10, версия 2004 Входящие приложения ISO](https://software-download.microsoft.com/download/pr/19041.1.191206-1406.vb_release_amd64fre_InboxApps.iso)
        - [Windows 10, версия 20H2 Входящие приложения ISO](https://software-download.microsoft.com/download/pr/19041.508.200905-1327.vb_release_svc_prod1_amd64fre_InboxApps.iso)
     
     - Если для локализации образов вы используете ISO-файлы с локальным пакетом поддержки (ЛКСП), вам также потребуется скачать соответствующий ЛКСП ISO для лучшего языкового интерфейса.
        - Если вы используете Windows 10 версии 1903 или 1909:
          - [Windows 10, версия 1903 или 1909 ЛКСП ISO](https://software-download.microsoft.com/download/pr/Win_10_1903_32_64_ARM64_MultiLng_LngPkAll_LXP_ONLY.iso)
        - Если вы используете Windows 10 версии 2004 или 20H2, используйте сведения в статьи [Добавление языков в Windows 10: известные проблемы](/windows-hardware/manufacture/desktop/language-packs-known-issue) , чтобы выяснить, какой из следующих лксп ISO подходит вам:
          - [Windows 10, версия 2004 или 20H2 **9Б** лксп ISO](https://software-download.microsoft.com/download/pr/Win_10_2004_64_ARM64_MultiLang_LangPckAll_LIP_LXP_ONLY)
          - [Windows 10, версия 2004 или 20H2 **9C** лксп ISO](https://software-download.microsoft.com/download/pr/Win_10_2004_32_64_ARM64_MultiLng_LngPkAll_LIP_9C_LXP_ONLY)
          - [Windows 10, версия 2004 или 20H2 **10C** лксп ISO](https://software-download.microsoft.com/download/pr/LanguageExperiencePack.2010C.iso)
          - [Windows 10, версия 2004 или 20H2 **11C** лксп ISO](https://software-download.microsoft.com/download/pr/LanguageExperiencePack.2011C.iso)
          - [Windows 10, версия 2004 или 20H2 **1C** лксп ISO](https://software-download.microsoft.com/download/pr/LanguageExperiencePack.2101C.iso)
          - [Windows 10, версия 2004 или 20H2 **2c** лксп ISO](https://software-download.microsoft.com/download/pr/LanguageExperiencePack.2102C.iso)

- Общая папка службы файлов Azure или общая папка на виртуальной машине файлового сервера Windows

>[!NOTE]
>Общая папка (репозиторий) должна быть доступна из виртуальной машины Azure, которую вы планируете использовать для создания пользовательского образа.

## <a name="create-a-content-repository-for-language-packages-and-features-on-demand"></a>Создание репозитория содержимого для языковых пакетов и компонентов по требованию

Чтобы создать репозиторий содержимого для языковых пакетов и Фодс, а также репозиторий для пакетов входящих приложений, выполните следующие действия.

1. На виртуальной машине Azure Скачайте многоязыковые приложения для Windows 10, Фодс и входящие, для образов Windows 10 Корпоративная с несколькими сеансами, версии 1903/1909 и 2004, используя ссылки в [предварительных требованиях](#prerequisites).

2. Откройте и подключите ISO-файлы на виртуальной машине.

3. Перейдите к ISO-образу языкового пакета и скопируйте содержимое из папок **локалекспериенцепаккс** и **x64 \\ лангпаккс** , а затем вставьте содержимое в общую папку.

4. Перейдите в **ISO-файл FOD**, скопируйте все его содержимое, а затем вставьте его в общую папку.
5. Перейдите в папку **amd64fre** в формате ISO "Входящие приложения" и скопируйте содержимое в репозиторий для подготовленных Вами входящих вами приложений.

     >[!NOTE]
     > Если вы работаете с ограниченным объемом хранилища, копируйте файлы только для тех языков, которые вам нужны пользователям. Файлы можно указать отдельно, взглянув на коды языка в именах файлов. Например, французский файл содержит код "fr-FR" в его имени. Полный список кодов языков для всех доступных языков см. в разделе [Доступные языковые пакеты для Windows](/windows-hardware/manufacture/desktop/available-language-packs-for-windows).

     >[!IMPORTANT]
     > Для некоторых языков требуются дополнительные шрифты, содержащиеся в вспомогательных пакетах, которые соответствуют различным соглашениям об именовании. Например, имена файлов японского шрифта включают "Жпан".
     >
     > [!div class="mx-imgBorder"]
     > ![Пример японских языковых пакетов с тегом языка "Жпан" в именах файлов.](media/language-pack-example.png)

6. Задайте разрешения для общей папки репозитория содержимого языка, чтобы у вас был доступ на чтение с виртуальной машины, которая будет использоваться для создания пользовательского образа.

## <a name="create-a-custom-windows-10-enterprise-multi-session-image-manually"></a>Создание пользовательского многосеансового образа Windows 10 Enterprise вручную

Для создания пользовательского многосеансового образа Windows 10 Enterprise вручную выполните следующие действия.

1. Разверните виртуальную машину Azure, перейдите в коллекцию Azure и выберите текущую версию многосеансовой версии Windows 10 Enterprise, которую вы используете.
2. После развертывания виртуальной машины подключитесь к ней, используя RDP в качестве локального администратора.
3. Убедитесь, что на виртуальной машине установлены все последние обновления Windows. При необходимости скачайте обновления и перезапустите виртуальную машину.
4. Подключитесь к репозиторию файлового пакета, FOD и папке "Входящие приложения" и подключите его к букве диска (например, на диске E).

## <a name="create-a-custom-windows-10-enterprise-multi-session-image-automatically"></a>Автоматическое создание пользовательского многосеансового образа Windows 10 Enterprise

Если вы предпочитаете устанавливать языки с помощью автоматизированного процесса, можно настроить сценарий в PowerShell. Приведенный ниже пример сценария можно использовать для установки языковых пакетов для испанского (Испания), французского (Франция) и китайского (КНР) языков и вспомогательных пакетов для многосеансовой версии Windows 10 Enterprise, версия 2004. Сценарий интегрирует пакет интерфейса языка и все необходимые вспомогательные пакеты в образ. Однако этот скрипт также можно изменить для установки других языков. Просто убедитесь, что сценарий запущен из сеанса PowerShell с повышенными привилегиями, или в противном случае он не будет работать.

```powershell
########################################################
## Add Languages to running Windows Image for Capture##
########################################################

##Disable Language Pack Cleanup##
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\AppxDeploymentClient\" -TaskName "Pre-staged app cleanup"

##Set Language Pack Content Stores##
[string]$LIPContent = "E:"

##Spanish##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\es-es\LanguageExperiencePack.es-es.Neutral.appx -LicensePath $LIPContent\es-es\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_es-es.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-es-es-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~es-es~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~es-es~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("es-es")
Set-WinUserLanguageList $LanguageList -force

##French##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\fr-fr\LanguageExperiencePack.fr-fr.Neutral.appx -LicensePath $LIPContent\fr-fr\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_fr-fr.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-fr-fr-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~fr-fr~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~fr-FR~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~fr-FR~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("fr-fr")
Set-WinUserLanguageList $LanguageList -force

##Chinese(PRC)##
Add-AppProvisionedPackage -Online -PackagePath $LIPContent\zh-cn\LanguageExperiencePack.zh-cn.Neutral.appx -LicensePath $LIPContent\zh-cn\License.xml
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Client-Language-Pack_x64_zh-cn.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Basic-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Fonts-Hans-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Handwriting-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-OCR-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-Speech-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-LanguageFeatures-TextToSpeech-zh-cn-Package~31bf3856ad364e35~amd64~~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-NetFx3-OnDemand-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-MSPaint-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Notepad-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-PowerShell-ISE-FOD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-Printing-WFS-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-StepsRecorder-Package~31bf3856ad364e35~amd64~zh-cn~.cab
Add-WindowsPackage -Online -PackagePath $LIPContent\Microsoft-Windows-WordPad-FoD-Package~31bf3856ad364e35~amd64~zh-cn~.cab
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("zh-cn")
Set-WinUserLanguageList $LanguageList -force
```

Сценарий может занять некоторое время в зависимости от числа языков, которые необходимо установить.

После завершения выполнения скрипта убедитесь, что языковые пакеты правильно установлены, перейдя к   >  **параметру**  >  **время запуска &**  >  **язык**. Если языковые файлы установлены, все они будут установлены.

После добавления дополнительных языков в образ Windows также необходимо обновить Входящие приложения для поддержки добавленных языков. Это можно сделать, обновив предварительно установленные приложения содержимым из ISO-файла входящих приложений. Чтобы выполнить это обновление в отключенной среде (нет доступа к Интернету из виртуальной машины), можно использовать следующий пример сценария PowerShell для автоматизации процесса.

```powershell
#########################################
## Update Inbox Apps for Multi Language##
#########################################
##Set Inbox App Package Content Stores##
[string]$InboxApps = "F:\"
##Update Inbox Store Apps##
$AllAppx = Get-Item $inboxapps\*.appx | Select-Object name
$AllAppxBundles = Get-Item $inboxapps\*.appxbundle | Select-Object name
$allAppxXML = Get-Item $inboxapps\*.xml | Select-Object name
foreach ($Appx in $AllAppx) {
    $appname = $appx.name.substring(0,$Appx.name.length-5)
    $appnamexml = $appname + ".xml"
    $pathappx = $InboxApps + "\" + $appx.Name
    $pathxml = $InboxApps + "\" + $appnamexml
    
    if($allAppxXML.name.Contains($appnamexml)){
    
    Write-Host "Handeling with xml $appname"  
  
    Add-AppxProvisionedPackage -Online -PackagePath $pathappx -LicensePath $pathxml
    } else {
      
      Write-Host "Handeling without xml $appname"
      
      Add-AppxProvisionedPackage -Online -PackagePath $pathappx -skiplicense
    }
}
foreach ($Appx in $AllAppxBundles) {
    $appname = $appx.name.substring(0,$Appx.name.length-11)
    $appnamexml = $appname + ".xml"
    $pathappx = $InboxApps + "\" + $appx.Name
    $pathxml = $InboxApps + "\" + $appnamexml
    
    if($allAppxXML.name.Contains($appnamexml)){
    Write-Host "Handeling with xml $appname"
    
    Add-AppxProvisionedPackage -Online -PackagePath $pathappx -LicensePath $pathxml
    } else {
       Write-Host "Handeling without xml $appname"
      Add-AppxProvisionedPackage -Online -PackagePath $pathappx -skiplicense
    }
}
```

>[!IMPORTANT]
>Входящие в состав ISO приложения не являются последними версиями предварительно установленных приложений Windows. Чтобы получить последнюю версию всех приложений, необходимо обновить приложения с помощью приложения для Магазина Windows и выполнить поиск обновлений вручную после установки дополнительных языков.

Когда все будет готово, не забудьте отключить общую папку.

## <a name="finish-customizing-your-image"></a>Завершение настройки образа

После установки языковых пакетов можно установить любое другое программное обеспечение, которое нужно добавить в пользовательский образ.

После завершения настройки образа необходимо запустить средство подготовки системы (Sysprep).

Чтобы запустить Sysprep, выполните следующие действия.

1. Откройте командную строку с повышенными привилегиями и выполните следующую команду, чтобы обобщать образ:  
   
     ```cmd
     C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown
     ```

2. Закройте виртуальную машину, а затем запишите ее в управляемом образе, следуя инструкциям в статье [Создание управляемого образа универсальной виртуальной машины в Azure](../virtual-machines/windows/capture-image-resource.md).

3. Теперь можно использовать настроенный образ для развертывания пула узлов виртуальных рабочих столов Windows. Сведения о развертывании пула узлов см. в разделе [учебник. Создание пула узлов с портал Azure](create-host-pools-azure-marketplace.md).

## <a name="enable-languages-in-windows-settings-app"></a>Включение языков в приложении "Параметры Windows"

Наконец, после развертывания пула узлов необходимо добавить язык в список языков каждого пользователя, чтобы он мог выбрать предпочитаемый язык в меню Параметры.

Чтобы пользователи могли выбрать установленные языки, войдите в систему от имени пользователя, а затем выполните следующий командлет PowerShell, чтобы добавить установленные языковые пакеты в меню языки. Этот сценарий также можно настроить как автоматизированную задачу или сценарий входа, который активируется при входе пользователя в сеанс.

```powershell
$LanguageList = Get-WinUserLanguageList
$LanguageList.Add("es-es")
$LanguageList.Add("fr-fr")
$LanguageList.Add("zh-cn")
Set-WinUserLanguageList $LanguageList -force
```

После того как пользователь изменит параметры языка, ему потребуется выйти из сеанса виртуальных рабочих столов Windows и снова войти, чтобы изменения вступили в силу. 

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать об известных проблемах, связанных с языковыми пакетами, см. статью [Добавление языковых пакетов в Windows 10, версия 1803 и более поздние версии: известные проблемы](/windows-hardware/manufacture/desktop/language-packs-known-issue).

Если у вас есть вопросы о многосеансовой среде Windows 10 Enterprise, ознакомьтесь с вопросами [и](windows-10-multisession-faq.md)ответами.

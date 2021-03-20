---
title: Установка Office на главном образе VHD в Azure
description: Как установить и настроить Office в основном образе Windows для виртуальных рабочих столов в Azure.
author: Heidilohr
ms.topic: how-to
ms.date: 05/02/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: b6369013d605ae538ad611a28a90e9c099bb7d80
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91326054"
---
# <a name="install-office-on-a-master-vhd-image"></a>Установка Office в главный образ VHD

В этой статье объясняется, как установить Microsoft 365 приложения для корпоративных, OneDrive и других распространенных приложений на основном образе виртуального жесткого диска (VHD) для отправки в Azure. Если пользователям требуется доступ к определенным бизнес-приложениям, рекомендуется установить их после выполнения инструкций, описанных в этой статье.

В этой статье предполагается, что вы уже создали виртуальную машину (ВМ). Если нет, см. статью [Подготовка и настройка главного образа VHD](set-up-customize-master-image.md#create-a-vm) .

Кроме того, в этой статье предполагается, что у вас есть повышенный доступ к виртуальной машине, будь то ее подготовка в Azure или диспетчере Hyper-V. Если нет, см. статью [повышение уровня доступа для управления всеми подписками Azure и группами управления](../role-based-access-control/elevate-access-global-admin.md).

>[!NOTE]
>Эти инструкции предназначены для конфигурации Виртуального рабочего стола Windows, которую можно использовать в имеющихся процессах вашей организации.

## <a name="install-office-in-shared-computer-activation-mode"></a>Установить Office в режиме активации общего компьютера

Активация на общем компьютере позволяет развертывать Microsoft 365ные приложения для предприятия на компьютере в Организации, к которому имеют доступ несколько пользователей. Дополнительные сведения об активации общего компьютера см. в разделе [Обзор активации общего компьютера для Microsoft 365 приложений](/deployoffice/overview-shared-computer-activation).

Используйте [средство развертывания Office](https://www.microsoft.com/download/details.aspx?id=49117) для установки Office. Многосеансовая поддержка Windows 10 Enterprise поддерживает только следующие версии Office:

   - Приложения Microsoft 365 для предприятия
   - Microsoft 365 приложения для бизнеса, поставляемые с подпиской Microsoft 365 бизнес Premium

Средству развертывания Office требуется XML-файл конфигурации. Чтобы настроить следующий пример, см. раздел [Параметры конфигурации средства развертывания Office](/deployoffice/configuration-options-for-the-office-2016-deployment-tool/).

Этот пример XML-файла конфигурации, который мы предоставили, выполняет следующие действия:

   - Установите Office из ежемесячного корпоративного канала и доставьте обновления из ежемесячного корпоративного канала.
   - Используйте архитектуру x64.
   - Отключение автоматического обновления.
   - Удалите все существующие установки Office и перенесите их параметры.
   - Включите активацию общего компьютера.

>[!NOTE]
>Функция поиска в наборе элементов Visio может не работать должным образом в виртуальном рабочем столе Windows.

Ниже приведен пример XML-файла конфигурации.

   - Установка Skype для бизнеса
   - Установите OneDrive в режиме "на пользователя". Дополнительные сведения см. в статье [Установка OneDrive в режиме "на компьютер](#install-onedrive-in-per-machine-mode)".

>[!NOTE]
>Активацию на общем компьютере можно настроить с помощью групповая политика объектов (GPO) или параметров реестра. Объект GPO расположен в папке **"политики конфигурации компьютера" \\ \\ административные шаблоны \\ \\ параметры лицензирования Microsoft Office 2016 (компьютер)** .

Средство развертывания Office содержит setup.exe. Чтобы установить Office, выполните в командной строке следующую команду:

```cmd
Setup.exe /configure configuration.xml
```

#### <a name="sample-configurationxml"></a>Пример configuration.xml

В следующем примере XML будет установлен ежемесячный выпуск корпоративного канала.

```xml
<Configuration>
  <Add OfficeClientEdition="64" Channel="MonthlyEnterprise">
    <Product ID="O365ProPlusRetail">
      <Language ID="en-US" />
      <Language ID="MatchOS" />
      <ExcludeApp ID="Groove" />
      <ExcludeApp ID="Lync" />
      <ExcludeApp ID="OneDrive" />
      <ExcludeApp ID="Teams" />
    </Product>
  </Add>
  <RemoveMSI/>
  <Updates Enabled="FALSE"/>
  <Display Level="None" AcceptEULA="TRUE" />
  <Logging Level="Standard" Path="%temp%\WVDOfficeInstall" />
  <Property Name="FORCEAPPSHUTDOWN" Value="TRUE"/>
  <Property Name="SharedComputerLicensing" Value="1"/>
</Configuration>
```

>[!NOTE]
>Команда разработчиков Office рекомендует использовать 64-разрядную установку для параметра **оффицеклиентедитион** .

После установки Office можно обновить поведение Office по умолчанию. Выполните следующие команды по отдельности или в пакетном файле, чтобы обновить поведение.

```cmd
rem Mount the default user registry hive
reg load HKU\TempDefault C:\Users\Default\NTUSER.DAT
rem Must be executed with default registry hive mounted.
reg add HKU\TempDefault\SOFTWARE\Policies\Microsoft\office\16.0\common /v InsiderSlabBehavior /t REG_DWORD /d 2 /f
rem Set Outlook's Cached Exchange Mode behavior
rem Must be executed with default registry hive mounted.
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v enable /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v syncwindowsetting /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v CalendarSyncWindowSetting /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v CalendarSyncWindowSettingMonths  /t REG_DWORD /d 1 /f
rem Unmount the default user registry hive
reg unload HKU\TempDefault

rem Set the Office Update UI behavior.
reg add HKLM\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate /v hideupdatenotifications /t REG_DWORD /d 1 /f
reg add HKLM\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate /v hideenabledisableupdates /t REG_DWORD /d 1 /f
```

## <a name="install-onedrive-in-per-machine-mode"></a>Установка OneDrive в режиме "на компьютер"

Как правило, OneDrive устанавливается отдельно для каждого пользователя. В этой среде она должна быть установлена для каждого компьютера.

Вот как можно установить OneDrive в режиме "на компьютер":

1. Сначала создайте расположение для размещения установщика OneDrive. Папка локального диска или [ \\ \\ UNC] (file://UNC) — это нормально.

2. Скачайте OneDriveSetup.exe в промежуточное расположение с помощью этой ссылки: <https://aka.ms/OneDriveWVD-Installer>

3. Если вы установили Office в OneDrive, опустив установку **\<ExcludeApp ID="OneDrive" /\>** , удалите все существующие установки onedrive для каждого пользователя из командной строки с повышенными привилегиями, выполнив следующую команду:

    ```cmd
    "[staged location]\OneDriveSetup.exe" /uninstall
    ```

4. Выполните эту команду из командной строки с повышенными привилегиями, чтобы задать значение реестра **аллусерсинсталл** :

    ```cmd
    REG ADD "HKLM\Software\Microsoft\OneDrive" /v "AllUsersInstall" /t REG_DWORD /d 1 /reg:64
    ```

5. Выполните следующую команду, чтобы установить OneDrive в режиме "на компьютер":

    ```cmd
    Run "[staged location]\OneDriveSetup.exe" /allusers
    ```

6. Выполните следующую команду, чтобы настроить OneDrive для запуска при входе для всех пользователей:

    ```cmd
    REG ADD "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDrive /t REG_SZ /d "C:\Program Files (x86)\Microsoft OneDrive\OneDrive.exe /background" /f
    ```

7. Включите **автоматическую настройку учетной записи пользователя** , выполнив следующую команду.

    ```cmd
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "SilentAccountConfig" /t REG_DWORD /d 1 /f
    ```

8. Перенаправьте и переместите известные папки Windows в OneDrive, выполнив следующую команду.

    ```cmd
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "KFMSilentOptIn" /t REG_SZ /d "<your-AzureAdTenantId>" /f
    ```

## <a name="microsoft-teams-and-skype-for-business"></a>Microsoft Teams и Skype для бизнеса

Виртуальный рабочий стол Windows не поддерживает Skype для бизнеса.

Сведения об установке Microsoft Teams см. в статье [использование Microsoft Teams на виртуальных рабочих столах Windows](teams-on-wvd.md). Оптимизация мультимедиа для Microsoft Teams на виртуальных рабочих столах Windows доступна в предварительной версии.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы добавили Office в образ, вы можете продолжить настройку главного образа VHD. См. раздел [Подготовка и настройка главного образа VHD](set-up-customize-master-image.md).

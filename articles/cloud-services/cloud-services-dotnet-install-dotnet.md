---
title: Установка .NET в облачных службах Azure (классические роли) | Документация Майкрософт
description: В этой статье описывается, как вручную установить платформу .NET Framework для веб-роли и рабочей роли облачной службы.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: aa05fc9f02c26192762ed34db54b60b4760bf3bf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99061857"
---
# <a name="install-net-on-azure-cloud-services-classic-roles"></a>Установка .NET в облачных службах Azure (классические роли)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

В этой статье описывается установка версий платформы .NET Framework, которые не входят в состав гостевой ОС Azure. .NET в гостевой ОС можно использовать для настройки веб-ролей и рабочих ролей облачной службы.

Например, можно установить платформа .NET Framework 4.6.2 в семействе 4 гостевой ОС, который не поставляется с выпуском платформа .NET Framework 4,6. (Семейство гостевых ОС версии 5 поставляется с платформа .NET Framework 4,6). Последние сведения о выпусках гостевой ОС Azure см. в [статье новости о выпуске гостевой ОС Azure](cloud-services-guestos-update-matrix.md). 

>[!IMPORTANT]
>Пакет Azure SDK 2,9 содержит ограничение на развертывание платформа .NET Framework 4,6 на гостевой ОС семейства 4 или более ранней версии. Исправление ограничения доступно на сайте [документации Майкрософт](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9).

Чтобы установить .NET для веб-ролей и рабочих ролей, включите веб-установщик .NET в качестве части проекта облачной службы. Запустите установщик в рамках задач запуска роли. 

## <a name="add-the-net-installer-to-your-project"></a>Добавление установщика .NET в проект
Чтобы скачать веб-установщик для платформы .NET Framework, выберите версию, которую требуется установить:

* [Веб-установщик платформа .NET Framework 4,8](https://go.microsoft.com/fwlink/?LinkId=2150985)
* [Веб-установщик платформа .NET Framework 4.7.2](https://go.microsoft.com/fwlink/?LinkId=863262)
* [Веб-установщик платформа .NET Framework 4.6.2](https://www.microsoft.com/download/details.aspx?id=53345)

Чтобы добавить установщик для *веб*-роли:
  1. В **обозревателе решений** в разделе **Роли** проекта облачной службы щелкните правой кнопкой мыши *веб-роль* и выберите **Добавить** > **Новая папка**. Создайте папку с именем **bin**.
  2. Щелкните правой кнопкой мыши папку bin и выберите **Добавить**  >  **существующий элемент**. Выберите установщик .NET и добавьте его в папку bin.
  
Чтобы добавить установщик для *рабочей* роли:
* Щелкните правой кнопкой мыши *рабочую* роль и выберите **Добавить** > **Существующий элемент**. Выберите установщик .NET и добавьте его в роль. 

При добавлении файлов таким образом в папку содержимого роли они автоматически добавляются в пакет облачной службы. Файлы затем развертываются в согласованное расположение на виртуальной машине. Повторите эту процедуру для каждой веб-роли и рабочей роли в облачной службе, чтобы у всех ролей была копия установщика.

> [!NOTE]
> Необходимо установить платформа .NET Framework 4.6.2 в роли облачной службы, даже если приложение предназначено для платформа .NET Framework 4,6. Гостевая ОС включает [обновление 3098779](https://support.microsoft.com/kb/3098779) и [обновление 3097997](https://support.microsoft.com/kb/3097997) базы знаний. При запуске приложений .NET могут возникнуть проблемы, если платформа .NET Framework 4,6 устанавливается поверх обновлений базы знаний. Чтобы избежать этих проблем, установите платформа .NET Framework 4.6.2, а не версию 4,6. Дополнительные сведения см. в статьях базы знаний [3118750](https://support.microsoft.com/kb/3118750) и [4340191](https://support.microsoft.com/kb/4340191).
> 
> 

![Роль с файлами установщика][1]

## <a name="define-startup-tasks-for-your-roles"></a>Определение начальных задач для ролей
С помощью задач запуска вы можете выполнять различные операции перед запуском роли. Установка платформы .NET Framework как части начальной задачи позволяет установить платформу до того, как будет запущено выполнение программного кода. Дополнительные сведения о задачах запуска см. [в статье запуск задач запуска в Azure](cloud-services-startup-tasks.md). 

1. В файл ServiceDefinition.csdef в узле **WebRole** или **WorkerRole** добавьте для всех ролей следующее содержимое:
   
    ```xml
    <LocalResources>
      <LocalStorage name="NETFXInstall" sizeInMB="1024" cleanOnRoleRecycle="false" />
    </LocalResources>    
    <Startup>
      <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
        <Environment>
          <Variable name="PathToNETFXInstall">
            <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='NETFXInstall']/@path" />
          </Variable>
          <Variable name="ComputeEmulatorRunning">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
    ```
   
    Предыдущая конфигурация выполняет консольную команду `install.cmd` с правами администратора и устанавливает платформу .NET Framework. Конфигурация также создает элемент **LocalStorage** с именем **NETFXInstall**. Сценарий запуска задает временную папку для использования этого локального ресурса хранилища. 
    
    > [!IMPORTANT]
    > Для обеспечения правильной установки платформы размер этого ресурса должен быть не менее 1024 МБ.
    
    Дополнительные сведения о задачах запуска см. в статье [Стандартные задачи запуска в облачной службе](cloud-services-startup-tasks-common.md).

2. Создайте файл с именем **install.cmd** и добавьте в него следующий скрипт установки.

   Скрипт путем поиска по реестру проверяет, установлена ли на компьютере выбранная версия .NET Framework. Если платформа .NET Framework версия не установлена, открывается веб-установщик платформа .NET Framework. Для помощи в устранении возникших проблем скрипт регистрирует все действия в файл startuptasklog-(текущая дата и время).txt, который хранится в локальном хранилище **InstallLogs**.
   
   > [!IMPORTANT]
   > Для создания файла install.cmd используйте простой текстовый редактор, например Блокнот. Если для создания текстового файла и изменения расширения на .cmd используется Visual Studio, файл по-прежнему может содержать метку порядка байтов UTF-8. Эта метка может вызвать ошибку при выполнении первой строки скрипта. Чтобы избежать этой ошибки, сделайте первую строку скрипта оператором REM, который может быть пропущен при обработке порядка байтов. 
   > 
   >
   
   ```cmd
   REM Set the value of netfx to install appropriate .NET Framework. 
   REM ***** To install .NET 4.5.2 set the variable netfx to "NDP452" *****
   REM ***** To install .NET 4.6 set the variable netfx to "NDP46" *****
   REM ***** To install .NET 4.6.1 set the variable netfx to "NDP461" ***** https://go.microsoft.com/fwlink/?LinkId=671729
   REM ***** To install .NET 4.6.2 set the variable netfx to "NDP462" ***** https://www.microsoft.com/download/details.aspx?id=53345
   REM ***** To install .NET 4.7 set the variable netfx to "NDP47" ***** 
   REM ***** To install .NET 4.7.1 set the variable netfx to "NDP471" ***** https://go.microsoft.com/fwlink/?LinkId=852095
   REM ***** To install .NET 4.7.2 set the variable netfx to "NDP472" ***** https://go.microsoft.com/fwlink/?LinkId=863262
   set netfx="NDP472"
   REM ***** To install .NET 4.8 set the variable netfx to "NDP48" ***** https://dotnet.microsoft.com/download/thank-you/net48
      
   REM ***** Set script start timestamp *****
   set timehour=%time:~0,2%
   set timestamp=%date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2%
   set "log=install.cmd started %timestamp%."
   
   REM ***** Exit script if running in Emulator *****
   if "%ComputeEmulatorRunning%"=="true" goto exit
   
   REM ***** Needed to correctly install .NET 4.6.1, otherwise you may see an out of disk space error *****
   set TMP=%PathToNETFXInstall%
   set TEMP=%PathToNETFXInstall%
   
   REM ***** Setup .NET filenames and registry keys *****
   if %netfx%=="NDP472" goto NDP472
   if %netfx%=="NDP471" goto NDP471
   if %netfx%=="NDP47" goto NDP47
   if %netfx%=="NDP462" goto NDP462
   if %netfx%=="NDP461" goto NDP461
   if %netfx%=="NDP46" goto NDP46
   
   set "netfxinstallfile=NDP452-KB2901954-Web.exe"
   set netfxregkey="0x5cbf5"
   goto logtimestamp
   
   :NDP46
   set "netfxinstallfile=NDP46-KB3045560-Web.exe"
   set netfxregkey="0x6004f"
   goto logtimestamp
   
   :NDP461
   set "netfxinstallfile=NDP461-KB3102438-Web.exe"
   set netfxregkey="0x6040e"
   goto logtimestamp
   
   :NDP462
   set "netfxinstallfile=NDP462-KB3151802-Web.exe"
   set netfxregkey="0x60632"
   goto logtimestamp
   
   :NDP47
   set "netfxinstallfile=NDP47-KB3186500-Web.exe"
   set netfxregkey="0x707FE"
   goto logtimestamp
   
   :NDP471
   set "netfxinstallfile=NDP471-KB4033344-Web.exe"
   set netfxregkey="0x709fc"
   goto logtimestamp
   
   :NDP472
   set "netfxinstallfile=NDP472-KB4054531-Web.exe"
   set netfxregkey="0x70BF6"
   goto logtimestamp
   
   :logtimestamp
   REM ***** Setup LogFile with timestamp *****
   md "%PathToNETFXInstall%\log"
   set startuptasklog="%PathToNETFXInstall%log\startuptasklog-%timestamp%.txt"
   set netfxinstallerlog="%PathToNETFXInstall%log\NetFXInstallerLog-%timestamp%"
   echo %log% >> %startuptasklog%
   echo Logfile generated at: %startuptasklog% >> %startuptasklog%
   echo TMP set to: %TMP% >> %startuptasklog%
   echo TEMP set to: %TEMP% >> %startuptasklog%
   
   REM ***** Check if .NET is installed *****
   echo Checking if .NET (%netfx%) is installed >> %startuptasklog%
   set /A netfxregkeydecimal=%netfxregkey%
   set foundkey=0
   FOR /F "usebackq skip=2 tokens=1,2*" %%A in (`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v Release 2^>nul`) do @set /A foundkey=%%C
   echo Minimum required key: %netfxregkeydecimal% -- found key: %foundkey% >> %startuptasklog%
   if %foundkey% GEQ %netfxregkeydecimal% goto installed
   
   REM ***** Installing .NET *****
   echo Installing .NET with commandline: start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog%  /chainingpackage "CloudService Startup Task" >> %startuptasklog%
   start /wait %~dp0%netfxinstallfile% /q /serialdownload /log %netfxinstallerlog% /chainingpackage "CloudService Startup Task" >> %startuptasklog% 2>>&1
   if %ERRORLEVEL%== 0 goto installed
       echo .NET installer exited with code %ERRORLEVEL% >> %startuptasklog%    
       if %ERRORLEVEL%== 3010 goto restart
       if %ERRORLEVEL%== 1641 goto restart
       echo .NET (%netfx%) install failed with Error Code %ERRORLEVEL%. Further logs can be found in %netfxinstallerlog% >> %startuptasklog%
       goto exit
       
   :restart
   echo Restarting to complete .NET (%netfx%) installation >> %startuptasklog%
   shutdown.exe /r /t 5 /c "Installed .NET framework" /f /d p:2:4
   
   :installed
   echo .NET (%netfx%) is installed >> %startuptasklog%
   
   :end
   echo install.cmd completed: %date:~-4,4%%date:~-10,2%%date:~-7,2%-%timehour: =0%%time:~3,2% >> %startuptasklog%
   
   :exit
   EXIT /B 0
   ```

3. Добавьте файл install. cmd в каждую роль с помощью команды **Добавить**  >  **существующий элемент** в **Обозреватель решений** , как описано ранее в этом разделе. 

    После завершения этого шага все роли должны иметь файл установщика .NET и файл install.cmd.

   ![Роль со всеми файлами][2]

## <a name="configure-diagnostics-to-transfer-startup-logs-to-blob-storage"></a>Настройка диагностики для передачи журналов задач запуска в хранилище BLOB-объектов
Чтобы упростить устранение неполадок при установке, можно настроить в Диагностике Azure передачу всех файлов журналов, созданных скриптом запуска или установщиком .NET, в хранилище BLOB-объектов Azure. Это позволит просматривать журналы, скачивая их из хранилища BLOB-объектов, а не на удаленном рабочем столе, где хранится роль.


Чтобы настроить систему диагностики, откройте файл diagnostics.wadcfgx и добавьте в узел **Directories** следующее содержимое: 

```xml 
<DataSources>
 <DirectoryConfiguration containerName="netfx-install">
  <LocalResource name="NETFXInstall" relativePath="log"/>
 </DirectoryConfiguration>
</DataSources>
```

Этот XML-код позволяет системе диагностики передавать файлы из каталога log в ресурсе **NETFXInstall** в учетную запись хранения диагностических данных в контейнере больших двоичных объектов **netfx-install**.

## <a name="deploy-your-cloud-service"></a>Развертывание облачной службы
При развертывании облачной службы задачи запуска устанавливают платформу .NET Framework, если она еще не установлена. Роли облачной службы находятся в состоянии *занятости* при установке платформы. Если при установке платформы необходима перезагрузка, роли службы также могут перезапуститься. 

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Установка платформы .NET Framework][Installing the .NET Framework]
* [Определение установленных версий платформы .NET Framework][How to: Determine Which .NET Framework Versions Are Installed]
* [Устранение неполадок платформа .NET Framework установки][Troubleshooting .NET Framework Installations]

[How to: Determine Which .NET Framework Versions Are Installed]: /dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed
[Installing the .NET Framework]: /dotnet/framework/install/guide-for-developers
[Troubleshooting .NET Framework Installations]: /dotnet/framework/install/troubleshoot-blocked-installations-and-uninstallations

<!--Image references-->
[1]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithinstallerfiles.png
[2]: ./media/cloud-services-dotnet-install-dotnet/rolecontentwithallfiles.png




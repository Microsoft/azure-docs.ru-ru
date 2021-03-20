---
title: Создание веб-ролей и рабочих ролей Azure для PHP
description: Руководство по настройке среды выполнения PHP и созданию веб-ролей и рабочих ролей PHP в облачной службе Azure.
services: ''
documentationcenter: php
author: msangapu-msft
manager: gwallace
ms.assetid: 9f7ccda0-bd96-4f7b-a7af-fb279a9e975b
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 04/11/2018
ms.author: msangapu
ms.openlocfilehash: 0b02662658b2199a13f117ad95c6402f9b2974f9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95519852"
---
# <a name="create-php-web-and-worker-roles"></a>Создание веб-ролей и рабочих ролей PHP

## <a name="overview"></a>Обзор

В этом руководстве мы расскажем, как создать веб-роль и рабочую роль PHP в среде разработки Windows, выбрать определенную версию PHP из доступных «встроенных» версий, изменить конфигурацию PHP, включить расширения и развернуть решение в Azure. Здесь также описывается, как создать веб-роль или рабочую роль для использования среды выполнения PHP (с настраиваемой конфигурацией и расширениями), которая предоставляется пользователем.

Azure предоставляет три вычислительные модели для запуска приложений: веб-приложения Azure, виртуальные машины Azure и облачные службы Azure. Все три модели поддерживают PHP. Облачные службы — это модель обслуживания типа *платформа как услуга (PaaS)*, в рамках которой используются веб-роли и рабочие роли. Веб-роль облачной службы предоставляет выделенный веб-сервер служб IIS, на котором размещаются интерфейсные веб-приложения. В рабочей роли могут выполняться асинхронные, долгосрочные или постоянные задачи, для которых не требуется взаимодействие с пользователем.

Подробнее об этих моделях можно узнать в статье о [вариантах размещения вычислительных сред, предоставляемых Azure](cloud-services/cloud-services-choose-me.md).

## <a name="download-the-azure-sdk-for-php"></a>Загрузка пакета Azure SDK для PHP

[Пакет Azure SDK для PHP](https://github.com/Azure/azure-sdk-for-php) состоит из нескольких компонентов. В данной статье будут использоваться два из них: Azure PowerShell и эмуляторы Azure. Эти два компонента можно установить при помощи установщика веб-платформы Майкрософт. Подробнее: [Установка и настройка Azure PowerShell](/powershell/azure/).

## <a name="create-a-cloud-services-project"></a>Создание проекта облачных служб

Первым шагом в создании веб-роли или рабочей роли PHP является создание проекта службы Azure. Он служит логическим контейнером для обеих ролей и содержит файлы [определения службы (CSDEF)] и [конфигурации службы (CSCFG)].

Чтобы создать новый проект службы Azure, запустите Azure PowerShell от имени администратора и выполните следующую команду.

```powershell
PS C:\>New-AzureServiceProject myProject
```

Эта команда создаст новый каталог (`myProject`), в который можно добавить веб-роли и рабочие роли.

## <a name="add-php-web-or-worker-roles"></a>Добавление веб-ролей PHP и ролей работников

Чтобы добавить в проект веб-роль PHP, выполните следующую команду из корневого каталога проекта.

```powershell
PS C:\myProject> Add-AzurePHPWebRole roleName
```

Для рабочей роли используйте следующую команду:

```powershell
PS C:\myProject> Add-AzurePHPWorkerRole roleName
```

> [!NOTE]
> Параметр `roleName` не обязателен. Если он опущен, имя роли будет создано автоматически. Первая веб-роль создается с именем `WebRole1`, вторая — `WebRole2` и т. д. Первая рабочая роль создается с именем `WorkerRole1`, вторая — `WorkerRole2` и т. д.
>
>

## <a name="use-your-own-php-runtime"></a>Использование собственной среды выполнения PHP

В некоторых случаях вместо выбора встроенной среды выполнения PHP и настройки в соответствии с указанными выше инструкциями вам может понадобиться предоставить собственную среду выполнения PHP. Например, вы можете использовать в рабочей роли или веб-роли ту же среду выполнения PHP, что и в среде разработки. Это поможет гарантировать, что приложение не изменит свое поведение в рабочей среде.

### <a name="configure-a-web-role-to-use-your-own-php-runtime"></a>Настройка собственной среды выполнения PHP для веб-роли

Чтобы использовать в веб-роли собственную среду выполнения PHP, выполните приведенные ниже действия.

1. Создайте проект службы Azure и добавьте веб-роль PHP, как описано в этом разделе ранее.
2. Создайте папку `php` в папке `bin`, которая находится в корневом каталоге веб-роли, затем добавьте в папку `php` среду выполнения PHP (все двоичные файлы, файлы конфигурации, вложенные папки и т. д.).
3. (Дополнительно.) Если среда выполнения PHP использует [драйверы Microsoft для PHP для SQL Server][sqlsrv drivers], то необходимо настроить веб-роль для установки [SQL Server Native Client 2012][sql native client] после ее подготовки. Для этого добавьте установщик [sqlncli.msi x64] в папку `bin` в корневом каталоге веб-роли. Скрипт запуска, описанный на следующем шаге, запустит установщик без вмешательства пользователя при подготовке роли. Если среда выполнения PHP не использует драйверы Microsoft для PHP для SQL Server, можно удалить следующую строку из скрипта, показанного на следующем шаге.

   ```console
   msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES
   ```

4. Чтобы [службы IIS][iis.net] использовали вашу среду PHP для обработки запросов для страниц `.php`, определите задачу запуска, изменяющую настройки IIS. Для этого откройте файл`setup_web.cmd` (в файле `bin` корневого каталога веб-роли) в текстовом редакторе и замените его содержимое на следующий скрипт:

    ```cmd
    @ECHO ON
    cd "%~dp0"

    if "%EMULATED%"=="true" exit /b 0

    msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

    SET PHP_FULL_PATH=%~dp0php\php-cgi.exe
    SET NEW_PATH=%PATH%;%RoleRoot%\base\x86

    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%',maxInstances='12',idleTimeout='60000',activityTimeout='3600',requestTimeout='60000',instanceMaxRequests='10000',protocol='NamedPipe',flushNamedPipe='False']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PATH',value='%NEW_PATH%']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='%PHP_FULL_PATH%'].environmentVariables.[name='PHP_FCGI_MAX_REQUESTS',value='10000']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/handlers /+"[name='PHP',path='*.php',verb='GET,HEAD,POST',modules='FastCgiModule',scriptProcessor='%PHP_FULL_PATH%',resourceType='Either',requireAccess='Script']" /commit:apphost
    %WINDIR%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /"[fullPath='%PHP_FULL_PATH%'].queueLength:50000"
    ```
5. Добавьте файлы вашего приложения в корневой каталог веб-роли. Это будет корневой каталог веб-сервера.
6. Опубликуйте приложение, как описано ниже в разделе [Публикация приложения](#publish-your-application).

> [!NOTE]
> Сценарий `download.ps1` (в папке `bin` корневого каталога веб-роли) можно удалить после выполнения описанной выше настройки собственной среды выполнения PHP.
>
>

### <a name="configure-a-worker-role-to-use-your-own-php-runtime"></a>Настройка собственной среды выполнения PHP для рабочей роли

Чтобы использовать в рабочей роли собственную среду выполнения PHP, выполните приведенные ниже действия.

1. Создайте проект службы Azure и добавьте рабочую роль PHP, как описано в этом разделе ранее.
2. Создайте папку `php` в корневом каталоге рабочей роли, а затем добавьте в папку `php` среду выполнения PHP (все двоичные файлы, файлы конфигурации, вложенные папки и т. д.).
3. (Дополнительно.) Если среда выполнения PHP использует [драйверы Microsoft для PHP для SQL Server][sqlsrv drivers], то необходимо настроить рабочую роль для установки [SQL Server Native Client 2012][sql native client] после ее подготовки. Для этого добавьте установщик [sqlncli.msi x64] в корневой каталог рабочей роли. Скрипт запуска, описанный на следующем шаге, запустит установщик без вмешательства пользователя при подготовке роли. Если среда выполнения PHP не использует драйверы Microsoft для PHP для SQL Server, можно удалить следующую строку из скрипта, показанного на следующем шаге.

   ```console
   msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES
   ```

4. Определите задачу запуска, которая при подготовке роли добавляет исполняемый файл `php.exe` в переменную среды PATH рабочей роли. Для этого откройте файл `setup_worker.cmd` (в корневом каталоге рабочей роли) в текстовом редакторе и замените его содержимое на следующий скрипт:

    ```cmd
    @echo on

    cd "%~dp0"

    echo Granting permissions for Network Service to the web root directory...
    icacls ..\ /grant "Network Service":(OI)(CI)W
    if %ERRORLEVEL% neq 0 goto error
    echo OK

    if "%EMULATED%"=="true" exit /b 0

    msiexec /i sqlncli.msi /qn IACCEPTSQLNCLILICENSETERMS=YES

    setx Path "%PATH%;%~dp0php" /M

    if %ERRORLEVEL% neq 0 goto error

    echo SUCCESS
    exit /b 0

    :error

    echo FAILED
    exit /b -1
    ```
5. Добавьте файлы вашего приложения в корневой каталог рабочей роли.
6. Опубликуйте приложение, как описано ниже в разделе [Публикация приложения](#publish-your-application).

## <a name="run-your-application-in-the-compute-and-storage-emulators"></a>Запуск приложения в эмуляторах вычисления и хранения

Эмуляторы Azure представляют собой локальную среду, в которой можно тестировать приложения Azure перед их развертыванием в облаке. Существует несколько различий между эмуляторами и средой Azure. Чтобы лучше понять это, см. статью [использование эмулятора хранения Azure для разработки и тестирования](storage/common/storage-use-emulator.md).

Обратите внимание, что для использования эмулятора вычислений следует установить PHP локально. Эмулятор вычислений будет использовать локальную установку PHP для запуска приложения.

Чтобы выполнить свой проект в эмуляторах, выполните следующую команду из корневого каталога своего проекта.

```powershell
PS C:\MyProject> Start-AzureEmulator
```

Вы увидите примерно такие выходные данные:

```output
Creating local package...
Starting Emulator...
Role is running at http://127.0.0.1:81
Started
```

Запущенное в эмуляторе приложение можно просмотреть в браузере, перейдя по адресу, который указан в выходных данных (в приведенном выше примере это`http://127.0.0.1:81` ).

Чтобы остановить эмуляторы, выполните следующую команду:

```powershell
PS C:\MyProject> Stop-AzureEmulator
```

## <a name="publish-your-application"></a>Публикация приложения

Чтобы опубликовать приложение, сначала импортируйте параметры публикации с помощью командлета [Import-AzurePublishSettingsFile](/powershell/module/servicemanagement/azure.service/import-azurepublishsettingsfile) . После этого опубликуйте приложение, используя командлет [Publish-AzureServiceProject](/powershell/module/servicemanagement/azure.service/publish-azureserviceproject) . Подробнее о входе: [Установка и настройка Azure PowerShell](/powershell/azure/).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительную информацию можно найти в [Центре разработчика PHP](https://azure.microsoft.com/develop/php/).

[install ps and emulators]: https://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
[определение службы (CSDEF)]: /previous-versions/azure/reference/ee758711(v=azure.100)
[Конфигурация службы (. cscfg)]: /previous-versions/azure/reference/ee758710(v=azure.100)
[iis.net]: https://www.iis.net/
[sql native client]: /sql/sql-server/sql-server-technical-documentation
[sqlsrv drivers]: https://php.net/sqlsrv
[sqlncli.msi x64]: https://go.microsoft.com/fwlink/?LinkID=239648
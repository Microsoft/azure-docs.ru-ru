---
title: Общие задачи запуска для облачных служб (классическая модель) | Документация Майкрософт
description: Содержит некоторые примеры стандартных задач запуска, которые можно выполнить в веб-роли или рабочей роли облачной службы.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: f55b225e615a3e7a5fbcf56b405054883d3b5413
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98741202"
---
# <a name="common-cloud-service-classic-startup-tasks"></a>Стандартные задачи запуска облачной службы (классические)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

В этой статье приведены некоторые примеры стандартных задач запуска, которые можно выполнить в облачной службе. С помощью задач запуска вы можете выполнять различные операции перед запуском роли. Это может быть установка компонента, регистрация компонентов COM, установка разделов реестра или запуск длительного процесса. 

[Эта статья](cloud-services-startup-tasks.md) поможет понять, как работают задачи запуска, и, в частности, как создавать записи, которые определяют задачу запуска.

> [!NOTE]
> Задачи запуска неприменимы к виртуальным машинам, они подходят только для веб-ролей и рабочих ролей облачной службы.
> 

## <a name="define-environment-variables-before-a-role-starts"></a>Определение переменных среды до запуска роли
Если требуется определить переменные среды для конкретной задачи, можно использовать элемент [Environment] внутри элемента [Task]

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
                <Environment>
                    <Variable name="MyEnvironmentVariable" value="MyVariableValue" />
                </Environment>
            </Task>
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

В переменных также может использоваться [допустимое значение XPath Azure](cloud-services-role-config-xpath.md) для ссылки на что-либо, относящееся к развертыванию. Вместо использования атрибута `value` определите дочерний элемент [RoleInstanceValue] .

```xml
<Variable name="PathToStartupStorage">
    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
</Variable>
```


## <a name="configure-iis-startup-with-appcmdexe"></a>Настройка запуска IIS с помощью AppCmd.exe
С помощью программы командной строки [AppCmd.exe](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj635852(v=ws.11)) можно управлять параметрами IIS при запуске в Azure. *AppCmd.exe* предоставляет удобный доступ из командной строки к параметрам конфигурации, используемым в задачах запуска в Azure. С помощью *AppCmd.exe* можно добавить, изменить или удалить параметры веб-сайта для приложений и сайтов.

Однако следует знать о некоторых особенностях использования *AppCmd.exe* в качестве задачи запуска:

* Задачи запуска могут несколько раз выполняться между перезагрузками. Например, при перезапуске роли.
* Если действие *AppCmd.exe* выполняется несколько раз, может произойти ошибка. Например, попытка дважды добавить раздел в *Web.config* приведет к ошибке.
* Задачи запуска завершаются ошибкой, если они возвращают ненулевой код выхода или **errorlevel**. Например, если *AppCmd.exe* выдает ошибку.

Рекомендуется проверять **errorlevel** после вызова *AppCmd.exe*. Это легко сделать, если создать программу-оболочку вызова *AppCmd.exe* в *CMD-файле*. При обнаружении известного ответа **errorlevel** его можно пропустить или вернуть.

Значения errorlevel, возвращаемые *AppCmd.exe*, перечислены в файле winerror.h. Кроме того, их можно просмотреть на сайте [MSDN](/windows/desktop/Debug/system-error-codes--0-499-).

### <a name="example-of-managing-the-error-level"></a>Пример управления уровнем ошибок
Этот пример добавляет раздел сжатия и запись сжатия для JSON в файл *Web.config* с обработкой ошибок и ведением журнала.

Здесь показаны соответствующие разделы файла [ServiceDefinition.csdef], которые включают в себя присвоение атрибуту [executionContext](/previous-versions/azure/reference/gg557552(v=azure.100)#task) значения `elevated`. Так *AppCmd.exe* предоставляются достаточные разрешения для изменения параметров в файле *Web.config*:

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

Пакетный файл *Startup.cmd* использует *AppCmd.exe* для добавления раздела сжатия и записи сжатия для JSON в файл *Web.config*. Ожидаемый **errorlevel** 183 получает значение 0 с помощью программы командной строки VERIFY.EXE. Непредвиденные значения errorlevel заносятся в файл StartupErrorLog.txt.

```cmd
REM   *** Add a compression section to the Web.config file. ***
%windir%\system32\inetsrv\appcmd set config /section:urlCompression /doDynamicCompression:True /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1

REM   ERRORLEVEL 183 occurs when trying to add a section that already exists. This error is expected if this
REM   batch file were executed twice. This can occur and must be accounted for in an Azure startup
REM   task. To handle this situation, set the ERRORLEVEL to zero by using the Verify command. The Verify
REM   command will safely set the ERRORLEVEL to zero.
IF %ERRORLEVEL% EQU 183 VERIFY > NUL

REM   If the ERRORLEVEL is not zero at this point, some other error occurred.
IF %ERRORLEVEL% NEQ 0 (
    ECHO Error adding a compression section to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
    GOTO ErrorExit
)

REM   *** Add compression for json. ***
%windir%\system32\inetsrv\appcmd set config  -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1
IF %ERRORLEVEL% EQU 183 VERIFY > NUL
IF %ERRORLEVEL% NEQ 0 (
    ECHO Error adding the JSON compression type to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
    GOTO ErrorExit
)

REM   *** Exit batch file. ***
EXIT /b 0

REM   *** Log error and exit ***
:ErrorExit
REM   Report the date, time, and ERRORLEVEL of the error.
DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
ECHO An error occurred during startup. ERRORLEVEL = %ERRORLEVEL% >> "%TEMP%\StartupLog.txt" 2>&1
EXIT %ERRORLEVEL%
```

## <a name="add-firewall-rules"></a>Добавление правил брандмауэра
Фактически в Azure есть два брандмауэра. Первый брандмауэр управляет подключениями между виртуальной машиной и внешним миром. Для управления им используется элемент [EndPoints] в файле [ServiceDefinition.csdef].

Второй брандмауэр управляет подключениями между виртуальной машиной и процессами в этой виртуальной машине. Им можно управлять с помощью программы командной строки `netsh advfirewall firewall`.

Azure создает правила брандмауэра для процессов, запущенных в ваших ролях. Например, при запуске службы или программы Azure автоматически создает необходимые правила брандмауэра, чтобы служба могла взаимодействовать с Интернетом. Тем не менее если создать службу, которую запускает процесс извне роли (например, службу COM+ или запланированную задачу Windows), необходимо будет вручную создать правило брандмауэра, чтобы разрешить доступ к этой службе. Эти правила брандмауэра можно создавать с помощью задачи запуска.

У задачи запуска, которая создает правило брандмауэра, должен быть [executionContext][Task] со значением **elevated**. Добавьте следующую задачу запуска в файл [ServiceDefinition.csdef] .

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WorkerRole name="WorkerRole1">
        ...
        <Startup>
            <Task commandLine="AddFirewallRules.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WorkerRole>
</ServiceDefinition>
```

Чтобы добавить правило брандмауэра, необходимо использовать в пакетном файле запуска соответствующие команды `netsh advfirewall firewall` . В этом примере задаче запуска требуется обеспечить безопасность и шифрование данных для TCP-порта 80.

```cmd
REM   Add a firewall rule in a startup task.

REM   Add an inbound rule requiring security and encryption for TCP port 80 traffic.
netsh advfirewall firewall add rule name="Require Encryption for Inbound TCP/80" protocol=TCP dir=in localport=80 security=authdynenc action=allow >> "%TEMP%\StartupLog.txt" 2>&1

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

## <a name="block-a-specific-ip-address"></a>Блокирование определенного IP-адреса
Доступ к веб-роли Azure можно ограничить набором указанных IP-адресов, изменив файл IIS **web.config**. Кроме того, нужно использовать командный файл, который разблокирует раздел **ipSecurity** файла **ApplicationHost.config**.

Чтобы разблокировать раздел **ipSecurity** файла **ApplicationHost.config**, создайте командный файл, который запускается при запуске роли. Создайте папку **startup** на корневом уровне веб-роли, а в этой папке создайте пакетный файл **startup.cmd**. Добавьте этот файл в проект Visual Studio и задайте для свойств значение **Всегда копировать**, чтобы убедиться, что он включен в пакет.

Добавьте следующую задачу запуска в файл [ServiceDefinition.csdef] .

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="startup.cmd" executionContext="elevated" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

Добавьте эту команду в файл **startup.cmd** :

```cmd
@echo off
@echo Installing "IPv4 Address and Domain Restrictions" feature 
powershell -ExecutionPolicy Unrestricted -command "Install-WindowsFeature Web-IP-Security"
@echo Unlocking configuration for "IPv4 Address and Domain Restrictions" feature 
%windir%\system32\inetsrv\AppCmd.exe unlock config -section:system.webServer/security/ipSecurity
```

В результате пакетный файл **startup.cmd** будет запускаться каждый раз при инициализации веб-роли, что разблокирует необходимый раздел **ipSecurity**.

Наконец, измените [раздел system.webServer](https://www.iis.net/configreference/system.webserver/security/ipsecurity#005) в файле **web.config** веб-роли, чтобы добавить список IP-адресов, к которым предоставлен доступ, как показано в следующем примере:

В этом примере конфигурации доступ к серверу **разрешен** всем IP-адресам, за исключением двух определенных.

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are granted access-->
    <ipSecurity>
        <!--The following IP addresses are denied access-->
        <add allowed="false" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="false" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

В этом примере конфигурации доступ к серверу **запрещен** всем IP-адресам, за исключением двух определенных.

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are denied access-->
    <ipSecurity allowUnlisted="false">
        <!--The following IP addresses are granted access-->
        <add allowed="true" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="true" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

## <a name="create-a-powershell-startup-task"></a>Создание задачи запуска PowerShell
Сценарии Windows PowerShell нельзя вызывать непосредственно из файла [ServiceDefinition.csdef] , но их можно вызывать из пакетного файла запуска.

PowerShell по умолчанию не запускает неподписанные сценарии. Если сценарии не подписаны, необходимо настроить PowerShell для запуска неподписанных сценариев. Для запуска неподписанных сценариев параметру **ExecutionPolicy** необходимо присвоить значение **Unrestricted**. Используемый параметр **ExecutionPolicy** зависит от версии Windows PowerShell.

```cmd
REM   Run an unsigned PowerShell script and log the output
PowerShell -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

Если вы используете гостевую ОС, на которой выполняется PowerShell 2.0 или 1.0, можно принудительно выполнить версию 2, и если она недоступна, использовать версию 1.

```cmd
REM   Attempt to set the execution policy by using PowerShell version 2.0 syntax.
PowerShell -Version 2.0 -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   If PowerShell version 2.0 isn't available. Set the execution policy by using the PowerShell
IF %ERRORLEVEL% EQU -393216 (
   PowerShell -Command "Set-ExecutionPolicy Unrestricted" >> "%TEMP%\StartupLog.txt" 2>&1
   PowerShell .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1
)

REM   If an error occurred, return the errorlevel.
EXIT /B %errorlevel%
```

## <a name="create-files-in-local-storage-from-a-startup-task"></a>Создание файлов в локальном хранилище из задачи запуска
Локальный ресурс хранилища можно использовать для хранения файлов, созданных задачей запуска, которые приложение использует позже.

Для создания локального ресурса хранилища добавьте раздел [LocalResources] в файл [ServiceDefinition.csdef], затем добавьте дочерний элемент [LocalStorage]. Присвойте локальному ресурсу хранилища уникальное имя и задайте соответствующий размер для задачи запуска.

Чтобы использовать локальный ресурс хранилища в задаче запуска, необходимо создать переменную среды для ссылки на расположение локального ресурса хранилища. Затем задача запуска и приложение смогут выполнять чтение и запись файлов в локальный ресурс хранилища.

Соответствующие разделы файла **ServiceDefinition.csdef** показаны ниже.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WorkerRole name="WorkerRole1">
    ...

    <LocalResources>
      <LocalStorage name="StartupLocalStorage" sizeInMB="5"/>
    </LocalResources>

    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>
          <Variable name="PathToStartupStorage">
            <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
  </WorkerRole>
</ServiceDefinition>
```

Например, этот пакетный файл **Startup.cmd** использует переменную среды **PathToStartupStorage**, чтобы создать файл **MyTest.txt** в локальном хранилище.

```cmd
REM   Create a simple text file.

ECHO This text will go into the MyTest.txt file which will be in the    >  "%PathToStartupStorage%\MyTest.txt"
ECHO path pointed to by the PathToStartupStorage environment variable.  >> "%PathToStartupStorage%\MyTest.txt"
ECHO The contents of the PathToStartupStorage environment variable is   >> "%PathToStartupStorage%\MyTest.txt"
ECHO "%PathToStartupStorage%".                                          >> "%PathToStartupStorage%\MyTest.txt"

REM   Exit the batch file with ERRORLEVEL 0.

EXIT /b 0
```

Обратиться к папке локального хранилища можно из пакета SDK для Azure с помощью метода [GetLocalResource](/previous-versions/azure/reference/ee772845(v=azure.100)).

```csharp
string localStoragePath = Microsoft.WindowsAzure.ServiceRuntime.RoleEnvironment.GetLocalResource("StartupLocalStorage").RootPath;

string fileContent = System.IO.File.ReadAllText(System.IO.Path.Combine(localStoragePath, "MyTestFile.txt"));
```

## <a name="run-in-the-emulator-or-cloud"></a>Запуск в эмуляторе или облаке
С помощью задачи запуска можно выполнять другие действия, когда она выполняется в облаке, по сравнению с выполнением в эмуляторе вычислений. Например, можно понадобиться использовать новую копию данных SQL только при выполнении в эмуляторе. Кроме того, может понадобиться каким-либо образом оптимизировать производительность для облака, что не требуется при выполнении в эмуляторе.

Эту возможность выполнять различные действия в эмуляторе вычислений и облаке можно получить, создав переменную среды в файле [ServiceDefinition.csdef]. Затем переменная проверяется в задаче запуска.

Чтобы создать переменную среды, добавьте элемент [variable] / [RoleInstanceValue] и создайте значение XPath `/RoleEnvironment/Deployment/@emulated` . При выполнении в эмуляторе вычислений переменная среды **%ComputeEmulatorRunning%** приобретает значение `true`, а при выполнении в облаке — `false`.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WorkerRole name="WorkerRole1">

    ...

    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>
          <Variable name="ComputeEmulatorRunning">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>

  </WorkerRole>
</ServiceDefinition>
```

Теперь задача может проверить переменную среды **%ComputeEmulatorRunning%** для выполнения разных действий в зависимости от того, где выполняется роль —в облаке или эмуляторе. Вот CMD-файл сценария оболочки для проверки этой переменной среды.

```cmd
REM   Check if this task is running on the compute emulator.

IF "%ComputeEmulatorRunning%" == "true" (
    REM   This task is running on the compute emulator. Perform tasks that must be run only in the compute emulator.
) ELSE (
    REM   This task is running on the cloud. Perform tasks that must be run only in the cloud.
)
```


## <a name="detect-that-your-task-has-already-run"></a>Обнаружение запуска задачи
Роль может перезапускаться без перезагрузки, вызывая повторный запуск задачи запуска. Показателей, которые указывают, что задача уже запущена на виртуальной машине размещения, не существует. Возможно, для некоторых задач не важно, сколько раз они выполняются. Однако возможны ситуации, когда нужно предотвратить выполнение задачи несколько раз.

Самый простой способ определить, что задача уже была запущена, —создавать файл в папке **%TEMP%** , если задача выполнена успешно, и искать его при запуске задачи. Ниже приведен пример CMD-файла сценария оболочки, который это делает.

```cmd
REM   If Task1_Success.txt exists, then Application 1 is already installed.
IF EXIST "%PathToApp1Install%\Task1_Success.txt" (
  ECHO Application 1 is already installed. Exiting. >> "%TEMP%\StartupLog.txt" 2>&1
  GOTO Finish
)

REM   Run your real exe task
ECHO Running XYZ >> "%TEMP%\StartupLog.txt" 2>&1
"%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

IF %ERRORLEVEL% EQU 0 (
  REM   The application installed without error. Create a file to indicate that the task
  REM   does not need to be run again.

  ECHO This line will create a file to indicate that Application 1 installed correctly. > "%PathToApp1Install%\Task1_Success.txt"

) ELSE (
  REM   An error occurred. Log the error and exit with the error code.

  DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
  TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
  ECHO  An error occurred running task 1. Errorlevel = %ERRORLEVEL%. >> "%TEMP%\StartupLog.txt" 2>&1

  EXIT %ERRORLEVEL%
)

:Finish

REM   Exit normally.
EXIT /B 0
```

## <a name="task-best-practices"></a>Рекомендации по задачам
Ниже приведены некоторые рекомендации, которые необходимо выполнять при настройке задачи для рабочей роли или веб-роли.

### <a name="always-log-startup-activities"></a>Всегда ведите журнал операций запуска
В Visual Studio нет отладчика для пошагового выполнения пакетных файлов, поэтому рекомендуется получить столько данных о выполнении пакетных файлов, сколько возможно. Ведение журнала выходных данных пакетных файлов, **stdout** и **stderr**, может предоставить вам важную информацию при попытке отладить и исправить пакетные файлы. Чтобы записывать **stdout** и **stderr** в файл StartupLog.txt в каталоге, указанном переменной среды **%TEMP%**, добавьте текст `>>  "%TEMP%\\StartupLog.txt" 2>&1` в конец конкретных строк, которые нужно добавлять в журнал. Например, чтобы выполнить setup.exe в каталоге **% PathToApp1Install%** : `"%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1`

Чтобы упростить XML, можно создать файл-оболочку *CMD*, который вызывает все задачи запуска, ведет журналы и гарантирует, что все дочерние задачи используют одинаковые переменные среды

Иногда может быть неудобно использовать `>> "%TEMP%\StartupLog.txt" 2>&1` в конце каждой задачи запуска. Чтобы настроить принудительную регистрацию задач, можно создать оболочку, которая выполняет ведение журнала автоматически. Эта оболочка вызывает действительный пакетный файл, который нужно запустить. Любые выходные данные целевого пакетного файла будут перенаправляться в файл *Startuplog.txt*.

В примере ниже показано, как перенаправить все выходные данные из пакетного файла запуска. В этом примере файл ServerDefinition.csdef создает задачу запуска, которая называется *logwrap.cmd*. *logwrap.cmd* вызывает *Startup2.cmd*, перенаправляя все выходные данные в **%TEMP%\\StartupLog.txt**.

ServiceDefinition.cmd:

```xml
<Startup>
    <Task commandLine="logwrap.cmd startup2.cmd" executionContext="limited" taskType="simple" />
</Startup>
```

**logwrap.cmd:**

```cmd
@ECHO OFF

REM   logwrap.cmd calls passed in batch file, redirecting all output to the StartupLog.txt log file.

ECHO [%date% %time%] == START logwrap.cmd ============================================== >> "%TEMP%\StartupLog.txt" 2>&1
ECHO [%date% %time%] Running %1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   Call the child command batch file, redirecting all output to the StartupLog.txt log file.
START /B /WAIT %1 >> "%TEMP%\StartupLog.txt" 2>&1

REM   Log the completion of child command.
ECHO [%date% %time%] Done >> "%TEMP%\StartupLog.txt" 2>&1

IF %ERRORLEVEL% EQU 0 (

   REM   No errors occurred. Exit logwrap.cmd normally.
   ECHO [%date% %time%] == END logwrap.cmd ================================================ >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO.  >> "%TEMP%\StartupLog.txt" 2>&1
   EXIT /B 0

) ELSE (

   REM   Log the error.
   ECHO [%date% %time%] An error occurred. The ERRORLEVEL = %ERRORLEVEL%.  >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO [%date% %time%] == END logwrap.cmd ================================================ >> "%TEMP%\StartupLog.txt" 2>&1
   ECHO.  >> "%TEMP%\StartupLog.txt" 2>&1
   EXIT /B %ERRORLEVEL%

)
```

**Startup2.cmd:**

```cmd
@ECHO OFF

REM   This is the batch file where the startup steps should be performed. Because of the
REM   way Startup2.cmd was called, all commands and their outputs will be stored in the
REM   StartupLog.txt file in the directory pointed to by the TEMP environment variable.

REM   If an error occurs, the following command will pass the ERRORLEVEL back to the
REM   calling batch file.

ECHO [%date% %time%] Some log information about this task
ECHO [%date% %time%] Some more log information about this task

EXIT %ERRORLEVEL%
```

Пример выходных данных в файле **StartupLog.txt**:

```txt
[Mon 10/17/2016 20:24:46.75] == START logwrap.cmd ============================================== 
[Mon 10/17/2016 20:24:46.75] Running command1.cmd 
[Mon 10/17/2016 20:24:46.77] Some log information about this task
[Mon 10/17/2016 20:24:46.77] Some more log information about this task
[Mon 10/17/2016 20:24:46.77] Done 
[Mon 10/17/2016 20:24:46.77] == END logwrap.cmd ================================================ 
```

> [!TIP]
> Файл **StartupLog.txt** находится в папке *C:\Resources\temp\\{идентификатор_роли}\RoleTemp*.
> 
> 

### <a name="set-executioncontext-appropriately-for-startup-tasks"></a>Правильная настройка executionContext для задач запуска
Задайте соответствующие привилегии для задачи запуска. Иногда задачи запуска должны выполняться с повышенными привилегиями, даже несмотря на то, что роль запускается с обычными привилегиями.

С помощью программы командной строки [executionContext][Task] задает уровень привилегий задачи запуска. Использование `executionContext="limited"` означает, что задаче запуска назначен тот же уровень привилегий, что и роли. Использование `executionContext="elevated"` означает, что задаче запуска назначены привилегии администратора, разрешающие задаче запуска выполнять операции администрирования без предоставления привилегий администратора вашей роли.

Примером задачи запуска, которой требуются повышенные привилегии, может послужить задача запуска, которая использует **AppCmd.exe** для настройки IIS. **AppCmd.exe** требуется `executionContext="elevated"`.

### <a name="use-the-appropriate-tasktype"></a>Использование соответствующего taskType
С помощью программы командной строки [TaskType][Task] определяет способ выполнения задачи запуска. Он имеет три значения: **simple**, **background** и **foreground**. Фоновые задачи (background) и задачи переднего плана (foreground) запускаются асинхронно, а затем по одной синхронно выполняются простые задачи (simple).

С помощью **простых** задач запуска можно задать порядок выполнения задач, упорядочив их список в файле ServiceDefinition.csdef. Если **простая** задача завершится с ненулевым кодом выхода, то процедура запуска остановится и роль не запустится.

Разница между **фоновыми** задачами запуска и задачами запуска **переднего плана** состоит в том, что задачи **переднего плана** оставляют роль запущенной до **своего** завершения. Это также означает, что если задача **переднего плана** перестает отвечать на запросы или дает сбой, роль не перезапустится, пока задача **переднего плана** не будет принудительно закрыта. По этой причине рекомендуется использовать **фоновые** задачи для асинхронных задач запуска, если только не требуется использовать возможности задачи **переднего плана**.

### <a name="end-batch-files-with-exit-b-0"></a>Завершайте пакетные файлы командой EXIT /B 0
Роль запустится, только если **errorlevel** каждой простой задачи запуска равен нулю. Не все программы корректно задают **errorlevel** (код завершения), поэтому пакетный файл должен заканчиваться `EXIT /B 0`, если все выполнено правильно.

Отсутствие `EXIT /B 0` в конце пакетного файла запуска — распространенная причина, по которой роли не запускаются.

> [!NOTE]
> Я заметил, что вложенные пакетные файлы иногда перестают отвечать на запросы при использовании `/B` параметра. Может потребоваться убедиться, что эта проблема не возникает, если другой пакетный файл вызывает текущий пакетный файл, например, если используется [оболочка журнала](#always-log-startup-activities). В этом случае можно опустить параметр `/B`.
> 
> 

### <a name="expect-startup-tasks-to-run-more-than-once"></a>Ожидайте многократного выполнения задач запуска
Не все перезапуски роли включают в себя перезагрузку, но все они вызывают выполнение всех задач запуска. Это означает, что задачи запуска должны иметь возможность многократно выполняться между перезагрузками без каких-либо проблем. Эта процедура рассматривается в [предыдущем разделе](#detect-that-your-task-has-already-run).

### <a name="use-local-storage-to-store-files-that-must-be-accessed-in-the-role"></a>Используйте локальное хранилище для хранения файлов, которые должны использоваться в роли
Если вы хотите во время выполнения задачи запуска скопировать или создать файл, который затем будет доступен вашей роли, этот файл необходимо поместить в локальное хранилище. См. [предыдущий раздел](#create-files-in-local-storage-from-a-startup-task).

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с [моделью и пакетом облачной службы](cloud-services-model-and-package.md)

Узнайте, как работают [задачи](cloud-services-startup-tasks.md) .

[Создайте и разверните](cloud-services-how-to-create-deploy-portal.md) свой пакет облачной службы.

[ServiceDefinition. csdef]: cloud-services-model-and-package.md#csdef
[Задача]: /previous-versions/azure/reference/gg557552(v=azure.100)#Task
[Startup]: /previous-versions/azure/reference/gg557552(v=azure.100)#Startup
[Runtime]: /previous-versions/azure/reference/gg557552(v=azure.100)#Runtime
[Среда]: /previous-versions/azure/reference/gg557552(v=azure.100)#Environment
[Переменная]: /previous-versions/azure/reference/gg557552(v=azure.100)#Variable
[RoleInstanceValue]: /previous-versions/azure/reference/gg557552(v=azure.100)#RoleInstanceValue
[RoleEnvironment]: /previous-versions/azure/reference/ee773173(v=azure.100)
[Конечные точки]: /previous-versions/azure/reference/gg557552(v=azure.100)#Endpoints
[LocalStorage]: /previous-versions/azure/reference/gg557552(v=azure.100)#LocalStorage
[LocalResources]: /previous-versions/azure/reference/gg557552(v=azure.100)#LocalResources
[RoleInstanceValue]: /previous-versions/azure/reference/gg557552(v=azure.100)#RoleInstanceValue
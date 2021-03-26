---
title: Справочник по API агента .NET для Azure Application Insights
description: Справочник по API агента Application Insights. Отслеживайте производительность веб-сайта без повторного развертывания веб-сайта. Работает с веб-приложениями ASP.NET, размещенными локально, в виртуальных машинах или в Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 04/23/2019
ms.openlocfilehash: 7c5c2f1d055ec6270892873548872b20b17b4158
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105566903"
---
# <a name="azure-monitor-application-insights-agent-api-reference"></a>Справочник по API агента Application Insights Azure Monitor

В этой статье описывается командлет, который является членом [модуля PowerShell AZ. аппликатионмонитор](https://www.powershellgallery.com/packages/Az.ApplicationMonitor/).

> [!NOTE] 
> - Чтобы приступить к работе, необходим ключ инструментирования. Дополнительные сведения см. [в разделе Создание ресурса](create-new-resource.md#copy-the-instrumentation-key).
> - Для этого командлета необходимо проверить и принять нашу лицензию и заявление о конфиденциальности.

> [!IMPORTANT] 
> Для этого командлета требуется сеанс PowerShell с разрешениями администратора и политикой выполнения с повышенными правами. Дополнительные сведения см. в статье [Запуск PowerShell от имени администратора с повышенной политикой выполнения](status-monitor-v2-detailed-instructions.md#run-powershell-as-admin-with-an-elevated-execution-policy).
> - Для этого командлета необходимо проверить и принять нашу лицензию и заявление о конфиденциальности.
> - Модуль инструментирования добавляет дополнительные издержки и по умолчанию отключен.


## <a name="enable-instrumentationengine"></a>Enable-InstrumentationEngine

Включает модуль инструментирования, настроив некоторые разделы реестра.
Перезапустите IIS, чтобы изменения вступили в силу.

Модуль инструментирования может дополнять данные, собираемые пакетами SDK для .NET.
Он собирает события и сообщения, описывающие выполнение управляемого процесса. Эти события и сообщения включают в себя коды результатов зависимостей, глаголы HTTP и [текст команды SQL](asp-net-dependencies.md#advanced-sql-tracking-to-get-full-sql-query).

Включите модуль инструментирования, если:
- Вы уже включили наблюдение с помощью командлета Enable, но не включили модуль инструментирования.
- Вы вручную разрешили свое приложение с помощью пакетов SDK для .NET и хотите получить дополнительные данные телеметрии.

### <a name="examples"></a>Примеры

```powershell
PS C:\> Enable-InstrumentationEngine
```

### <a name="parameters"></a>Параметры

#### <a name="-acceptlicense"></a>-AcceptLicense
**Необязательный элемент.** Используйте этот параметр, чтобы принять условия лицензии и конфиденциальности в установках без монитора.

#### <a name="-verbose"></a>-Verbose
**Общий параметр.** Используйте этот параметр для вывода подробных журналов.

### <a name="output"></a>Выходные данные


##### <a name="example-output-from-successfully-enabling-the-instrumentation-engine"></a>Пример выходных данных для успешного включения модуля инструментирования

```
Configuring IIS Environment for instrumentation engine...
Configuring registry for instrumentation engine...
```

## <a name="enable-applicationinsightsmonitoring"></a>Enable-ApplicationInsightsMonitoring

Включает некодированный мониторинг приложений IIS на конечном компьютере.

Этот командлет изменит applicationHost.config IIS и установит некоторые разделы реестра.
Кроме того, будет создан файл applicationinsights.ikey.config, который определяет ключ инструментирования, используемый каждым приложением.
Службы IIS загружают Редфиелдмодуле при запуске, что приводит к внедрению пакета SDK Application Insights в приложения при запуске приложений.
Перезапустите IIS, чтобы изменения вступили в силу.

После включения мониторинга рекомендуется использовать [динамические метрики](live-stream.md) , чтобы быстро проверить, отправляет ли ваше приложение данные телеметрии.

### <a name="examples"></a>Примеры

#### <a name="example-with-a-single-instrumentation-key"></a>Пример с одним ключом инструментирования
В этом примере всем приложениям на текущем компьютере назначается один ключ инструментирования.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### <a name="example-with-an-instrumentation-key-map"></a>Пример с картой ключа инструментирования
В этом примере:
- `MachineFilter` соответствует текущему компьютеру с помощью `'.*'` подстановочного знака.
- `AppFilter='WebAppExclude'` предоставляет `null` ключ инструментирования. Указанное приложение не будет инструментировано.
- `AppFilter='WebAppOne'` Присваивает указанному приложению уникальный ключ инструментирования.
- `AppFilter='WebAppTwo'` Присваивает указанному приложению уникальный ключ инструментирования.
- Наконец, `AppFilter` также использует `'.*'` подстановочный знак для сопоставления всех веб-приложений, которые не соответствуют предыдущим правилам, и назначения ключа инструментирования по умолчанию.
- Для удобочитаемости добавляются пробелы.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap 
    @(@{MachineFilter='.*';AppFilter='WebAppExclude'},
      @{MachineFilter='.*';AppFilter='WebAppOne';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1'}},
      @{MachineFilter='.*';AppFilter='WebAppTwo';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2'}},
      @{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault'}})

```

> [!NOTE]
> Именование Аппфилтер в этом контексте может быть запутанным, `AppFilter` задает фильтр имени приложения Regex (HostingEnvironment. sitename в случае .NET на IIS). `VirtualPathFilter` Задает фильтр регулярных выражений виртуального пути (HostingEnvironment. Аппликатионвиртуалпас в случае .NET на IIS). Для инструментирования отдельного приложения используйте Виртуалпасфилтер следующим образом: `Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap @(@{VirtualPathFilter="^/MyAppName$"; InstrumentationSettings=@{InstrumentationKey='<your ikey>'}})`

### <a name="parameters"></a>Параметры

#### <a name="-instrumentationkey"></a>-InstrumentationKey
**Обязательный.** Используйте этот параметр, чтобы указать один ключ инструментирования для использования всеми приложениями на целевом компьютере.

#### <a name="-instrumentationkeymap"></a>-Инструментатионкэймап
**Обязательный.** Этот параметр используется для предоставления нескольких ключей инструментирования и сопоставления ключей инструментирования, используемых каждым приложением.
Можно создать один скрипт установки для нескольких компьютеров, установив `MachineFilter` .

> [!IMPORTANT]
> Приложения будут соответствовать правилам в том порядке, в котором предоставляются правила. Поэтому необходимо сначала указать наиболее конкретные правила, а также наиболее общие правила.

##### <a name="schema"></a>схема
`@(@{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'}})`

- **Мачинефилтер** — это обязательное регулярное выражение C# для имени компьютера или виртуальной машины.
    - ". *" будет соответствовать всем
    - "ComputerName" будет соответствовать только компьютерам с указанным именем.
- **Аппфилтер** — это обязательное регулярное выражение C# имени веб-сайта IIS. Список сайтов на сервере можно получить, выполнив команду [Get-ииссите](/powershell/module/iisadministration/get-iissite).
    - ". *" будет соответствовать всем
    - "SiteName" будет соответствовать только сайту IIS с указанным точным именем.
- **InstrumentationKey** требуется для включения мониторинга приложений, соответствующих двум предыдущим фильтрам.
    - Если необходимо определить правила для исключения мониторинга, оставьте значение null.


#### <a name="-enableinstrumentationengine"></a>-Енаблеинструментатионенгине
**Необязательный элемент.** Используйте этот параметр, чтобы модуль инструментирования собирал события и сообщения о том, что происходит во время выполнения управляемого процесса. Эти события и сообщения включают в себя коды результатов зависимостей, глаголы HTTP и текст команды SQL.

Модуль инструментирования добавляет издержки и по умолчанию отключен.

#### <a name="-acceptlicense"></a>-AcceptLicense
**Необязательный элемент.** Используйте этот параметр, чтобы принять условия лицензии и конфиденциальности в установках без монитора.

#### <a name="-ignoresharedconfig"></a>-Игнорешаредконфиг
При наличии кластера веб-серверов можно использовать [общую конфигурацию](/iis/web-hosting/configuring-servers-in-the-windows-web-platform/shared-configuration_211).
Не удается внедрить HttpModule в эту общую конфигурацию.
Этот сценарий завершится ошибкой с сообщением о необходимости дополнительных шагов установки.
Используйте этот параметр, чтобы пропустить эту проверку и продолжить установку необходимых компонентов. Дополнительные сведения см. в разделе [Известные конфликты — с-IIS-Shared-Configuration](status-monitor-v2-troubleshoot.md#conflict-with-iis-shared-configuration) .

#### <a name="-verbose"></a>-Verbose
**Общий параметр.** Используйте этот параметр для просмотра подробных журналов.

#### <a name="-whatif"></a>-WhatIf 
**Общий параметр.** Используйте этот переключатель для проверки и проверки входных параметров без фактического включения мониторинга.

### <a name="output"></a>Выходные данные

#### <a name="example-output-from-a-successful-enablement"></a>Пример выходных данных успешного включения

```powershell
Initiating Disable Process
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-52z'
in :1,237
No element in the source document matches '/configuration/location[@path='']/system.webServer/modules/add[@name='ManagedHttpModuleHelper']'
Not executing RemoveAll (transform line 1, 546)
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'disable'
GAC Module will not be removed, since this operation might cause IIS instabilities
Configuring IIS Environment for codeless attach...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring IIS Environment for instrumentation engine...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring registry for instrumentation engine...
Successfully disabled Application Insights Status Monitor
Installing GAC module 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\0.2.0\content\Runtime\Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll'
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
Found GAC module Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.ManagedHttpModuleHelper, Microsoft.AppInsights.IIS.ManagedHttpModuleHelper, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-52z_1'
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'enable'
Configuring IIS Environment for codeless attach...
Configuring IIS Environment for instrumentation engine...
Configuring registry for instrumentation engine...
Updating app pool permissions...
Successfully enabled Application Insights Status Monitor
```

## <a name="disable-instrumentationengine"></a>Disable-InstrumentationEngine

Отключает модуль инструментирования, удаляя некоторые разделы реестра.
Перезапустите IIS, чтобы изменения вступили в силу.

### <a name="examples"></a>Примеры

```powershell
PS C:\> Disable-InstrumentationEngine
```

### <a name="parameters"></a>Параметры 

#### <a name="-verbose"></a>-Verbose
**Общий параметр.** Используйте этот параметр для вывода подробных журналов.

### <a name="output"></a>Выходные данные


##### <a name="example-output-from-successfully-disabling-the-instrumentation-engine"></a>Пример выходных данных для успешного отключения модуля инструментирования

```powershell
Configuring IIS Environment for instrumentation engine...
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]'
Configuring registry for instrumentation engine...
```

## <a name="disable-applicationinsightsmonitoring"></a>Disable-ApplicationInsightsMonitoring

Отключает наблюдение на целевом компьютере.
Этот командлет удалит изменения в applicationHost.config IIS и удалит разделы реестра.

### <a name="examples"></a>Примеры

```powershell
PS C:\> Disable-ApplicationInsightsMonitoring
```

### <a name="parameters"></a>Параметры 

#### <a name="-verbose"></a>-Verbose
**Общий параметр.** Используйте этот параметр для просмотра подробных журналов.

### <a name="output"></a>Выходные данные


##### <a name="example-output-from-successfully-disabling-monitoring"></a>Пример выходных данных для успешного отключения мониторинга

```powershell
Initiating Disable Process
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-00z'
in :1,237
No element in the source document matches '/configuration/location[@path='']/system.webServer/modules/add[@name='ManagedHttpModuleHelper']'
Not executing RemoveAll (transform line 1, 546)
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'disable'
GAC Module will not be removed, since this operation might cause IIS instabilities
Configuring IIS Environment for codeless attach...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring IIS Environment for instrumentation engine...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring registry for instrumentation engine...
Successfully disabled Application Insights Status Monitor
```


## <a name="get-applicationinsightsmonitoringconfig"></a>Get-ApplicationInsightsMonitoringConfig

Возвращает файл конфигурации и выводит значения на консоль.

### <a name="examples"></a>Примеры

```powershell
PS C:\> Get-ApplicationInsightsMonitoringConfig
```

### <a name="parameters"></a>Параметры

Параметры не требуются.

### <a name="output"></a>Выходные данные


##### <a name="example-output-from-reading-the-config-file"></a>Пример выходных данных чтения файла конфигурации

```
RedfieldConfiguration:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
```

## <a name="get-applicationinsightsmonitoringstatus"></a>Get-ApplicationInsightsMonitoringStatus

Этот командлет предоставляет сведения об устранении неполадок монитор состояния.
Используйте этот командлет для изучения состояния мониторинга, версии модуля PowerShell и для проверки выполняющегося процесса.
Этот командлет будет сообщать сведения о версии и о файлах ключей, необходимых для мониторинга.

### <a name="examples"></a>Примеры

#### <a name="example-application-status"></a>Пример: состояние приложения

Выполните команду, `Get-ApplicationInsightsMonitoringStatus` чтобы отобразить состояние мониторинга веб-сайтов.

```powershell

PS C:\Windows\system32> Get-ApplicationInsightsMonitoringStatus

IIS Websites:

SiteName               : Default Web Site
ApplicationPoolName    : DefaultAppPool
SiteId                 : 1
SiteState              : Stopped

SiteName               : DemoWebApp111
ApplicationPoolName    : DemoWebApp111
SiteId                 : 2
SiteState              : Started
ProcessId              : not found

SiteName               : DemoWebApp222
ApplicationPoolName    : DemoWebApp222
SiteId                 : 3
SiteState              : Started
ProcessId              : 2024
Instrumented           : true
InstrumentationKey     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx123

SiteName               : DemoWebApp333
ApplicationPoolName    : DemoWebApp333
SiteId                 : 4
SiteState              : Started
ProcessId              : 5184
AppAlreadyInstrumented : true
```

В этом примере —
- **Идентификатор компьютера** — это АНОНИМный идентификатор, используемый для уникальной идентификации сервера. При создании запроса в службу поддержки этот идентификатор потребуется для поиска журналов на сервере.
- **Веб-сайт по умолчанию** остановлен в службах IIS
- **DemoWebApp111** был запущен в службах IIS, но не получил никаких запросов. В этом отчете показано, что нет выполняющегося процесса (ProcessId: не найдено).
- **DemoWebApp222** работает и отслеживается (инструментировано: true). На основе пользовательской конфигурации для этого сайта был сопоставлен ключ инструментирования XXXXXXXX-XXXX-XXXX-XXXX-xxxxxxxxx123.
- **DemoWebApp333** был вручную инструментирован с помощью пакета SDK для Application Insights. Монитор состояния обнаружил пакет SDK и не отслеживает этот сайт.


#### <a name="example-powershell-module-information"></a>Пример: сведения о модуле PowerShell

Выполните команду, `Get-ApplicationInsightsMonitoringStatus -PowerShellModule` чтобы отобразить сведения о текущем модуле:

```powershell

PS C:\> Get-ApplicationInsightsMonitoringStatus -PowerShellModule

PowerShell Module version:
0.4.0-alpha

Application Insights SDK version:
2.9.0.3872

Executing PowerShell Module Assembly:
Microsoft.ApplicationInsights.Redfield.Configurator.PowerShell, Version=2.8.14.11432, Culture=neutral, PublicKeyToken=31bf3856ad364e35

PowerShell Module Directory:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\0.2.2\content\PowerShell

Runtime Paths:
ParentDirectory (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content

ConfigurationPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config

ManagedHttpModuleHelperPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll

RedfieldIISModulePath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll

InstrumentationEngine86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\MicrosoftInstrumentationEngine_x86.dll

InstrumentationEngine64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll

InstrumentationEngineExtensionHost86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\Microsoft.ApplicationInsights.ExtensionsHost_x86.dll

InstrumentationEngineExtensionHost64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll

InstrumentationEngineExtensionConfig86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\Microsoft.InstrumentationEngine.Extensions.config

InstrumentationEngineExtensionConfig64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.InstrumentationEngine.Extensions.config

ApplicationInsightsSdkPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.dll
```

#### <a name="example-runtime-status"></a>Пример: состояние среды выполнения

Вы можете проверить процесс на инструментированном компьютере, чтобы узнать, загружены ли все библиотеки DLL. Если наблюдение работает, необходимо загрузить не менее 12 библиотек DLL.

Выполните команду `Get-ApplicationInsightsMonitoringStatus -InspectProcess`.


```
PS C:\> Get-ApplicationInsightsMonitoringStatus -InspectProcess

iisreset.exe /status
Status for IIS Admin Service ( IISADMIN ) : Running
Status for Windows Process Activation Service ( WAS ) : Running
Status for Net.Msmq Listener Adapter ( NetMsmqActivator ) : Running
Status for Net.Pipe Listener Adapter ( NetPipeActivator ) : Running
Status for Net.Tcp Listener Adapter ( NetTcpActivator ) : Running
Status for World Wide Web Publishing Service ( W3SVC ) : Running

handle64.exe -accepteula -p w3wp
BF0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.ServerTelemetryChannel.dll
C58: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.AzureAppServices.dll
C68: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.DependencyCollector.dll
C78: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.WindowsServer.dll
C98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.Web.dll
CBC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.PerfCounterCollector.dll
DB0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.Agent.Intercept.dll
B98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll
BB4: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.Contracts.dll
BCC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.Redfield.Lightup.dll
BE0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.dll

listdlls64.exe -accepteula w3wp
0x0000000019ac0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
0x00000000198b0000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
0x000000000c460000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
0x000000000ad60000  0x108000  C:\Windows\TEMP\2.4.0.0.Microsoft.ApplicationInsights.Extensions.Intercept_x64.dll
```

### <a name="parameters"></a>Параметры

#### <a name="no-parameters"></a>(Нет параметров)

По умолчанию этот командлет выведет отчет о состоянии мониторинга веб-приложений.
Используйте этот параметр, чтобы проверить успешность инструментирования приложения.
Вы также можете проверить, какой ключ инструментирования был сопоставлен с сайтом.


#### <a name="-powershellmodule"></a>-Повершеллмодуле
**Необязательно**. Используйте этот параметр, чтобы сообщить номера версий и пути библиотек DLL, необходимых для мониторинга.
Используйте этот параметр, если необходимо указать версию любой библиотеки DLL, включая пакет SDK для Application Insights.

#### <a name="-inspectprocess"></a>-Инспектпроцесс

**Необязательно**. Используйте этот параметр для сообщения о том, запущен ли сервер IIS.
Также будут скачаны внешние средства, чтобы определить, загружены ли необходимые библиотеки DLL в среду выполнения IIS.


Если по какой бы то ни было причине этот процесс завершится сбоем, можно выполнить следующие команды вручную:
- iisreset.exe/Status
- [handle64.exe](/sysinternals/downloads/handle) -p w3wp | Findstr/I "Инструментатионенгине AI. ApplicationInsights
- [listdlls64.exe](/sysinternals/downloads/listdlls) w3wp | Findstr/I "Инструментатионенгине AI ApplicationInsights"


#### <a name="-force"></a>-Force

**Необязательно**. Используется только с Инспектпроцесс. Используйте этот параметр, чтобы пропустить запрос пользователя, который появляется перед скачиванием дополнительных средств.


## <a name="set-applicationinsightsmonitoringconfig"></a>Set-ApplicationInsightsMonitoringConfig

Задает файл конфигурации без выполнения полной переустановки.
Перезапустите IIS, чтобы изменения вступили в силу.

> [!IMPORTANT] 
> Для этого командлета требуется сеанс PowerShell с разрешениями администратора.


### <a name="examples"></a>Примеры

#### <a name="example-with-a-single-instrumentation-key"></a>Пример с одним ключом инструментирования
В этом примере всем приложениям на текущем компьютере будет назначен один ключ инструментирования.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### <a name="example-with-an-instrumentation-key-map"></a>Пример с картой ключа инструментирования
В этом примере:
- `MachineFilter` соответствует текущему компьютеру с помощью `'.*'` подстановочного знака.
- `AppFilter='WebAppExclude'` предоставляет `null` ключ инструментирования. Указанное приложение не будет инструментировано.
- `AppFilter='WebAppOne'` Присваивает указанному приложению уникальный ключ инструментирования.
- `AppFilter='WebAppTwo'` Присваивает указанному приложению уникальный ключ инструментирования.
- Наконец, `AppFilter` также использует `'.*'` подстановочный знак для сопоставления всех веб-приложений, которые не соответствуют предыдущим правилам, и назначения ключа инструментирования по умолчанию.
- Для удобочитаемости добавляются пробелы.

```powershell
Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap `
       @(@{MachineFilter='.*';AppFilter='WebAppExclude'},
          @{MachineFilter='.*';AppFilter='WebAppOne';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1'}},
          @{MachineFilter='.*';AppFilter='WebAppTwo';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2'}},
          @{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault'}})
```

### <a name="parameters"></a>Параметры

#### <a name="-instrumentationkey"></a>-InstrumentationKey
**Обязательный.** Используйте этот параметр, чтобы указать один ключ инструментирования для использования всеми приложениями на целевом компьютере.

#### <a name="-instrumentationkeymap"></a>-Инструментатионкэймап
**Обязательный.** Этот параметр используется для предоставления нескольких ключей инструментирования и сопоставления ключей инструментирования, используемых каждым приложением.
Можно создать один скрипт установки для нескольких компьютеров, установив `MachineFilter` .

> [!IMPORTANT]
> Приложения будут соответствовать правилам в том порядке, в котором предоставляются правила. Поэтому необходимо сначала указать наиболее конкретные правила, а также наиболее общие правила.

##### <a name="schema"></a>схема
`@(@{MachineFilter='.*';AppFilter='.*';InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'})`

- **Мачинефилтер** — это обязательное регулярное выражение C# для имени компьютера или виртуальной машины.
    - ". *" будет соответствовать всем
    - "ComputerName" будет соответствовать только компьютерам с указанным именем.
- **Аппфилтер** — это обязательное регулярное выражение C# для имени компьютера или виртуальной машины.
    - ". *" будет соответствовать всем
    - "ApplicationName" будет сопоставлять только приложения IIS с указанным именем.
- **InstrumentationKey** требуется для включения мониторинга приложений, соответствующих двум предыдущим фильтрам.
    - Если необходимо определить правила для исключения мониторинга, оставьте значение null.


#### <a name="-verbose"></a>-Verbose
**Общий параметр.** Используйте этот параметр для просмотра подробных журналов.


### <a name="output"></a>Выходные данные

По умолчанию выходные данные отсутствуют.

##### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkey"></a>Пример подробных данных о настройке файла конфигурации с помощью-InstrumentationKey

```
VERBOSE: Operation: InstallWithIkey
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
```

##### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkeymap"></a>Пример подробных данных о настройке файла конфигурации с помощью-Инструментатионкэймап

```
VERBOSE: Operation: InstallWithIkeyMap
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
```

## <a name="start-applicationinsightsmonitoringtrace"></a>Start-ApplicationInsightsMonitoringTrace

Собирает [события ETW](/windows/desktop/etw/event-tracing-portal) из среды выполнения с некодированным подключением. Этот командлет является альтернативой запуску [PerfView](https://github.com/microsoft/perfview).

Собранные события будут выводиться на консоль в режиме реального времени и сохранены в ETL-файл. Выходной ETL-файл можно открыть с помощью [PerfView](https://github.com/microsoft/perfview) для дальнейшего изучения.

Этот командлет будет выполняться до тех пор, пока не достигнет времени ожидания (по умолчанию 5 минут) или остановлен вручную ( `Ctrl + C` ).

### <a name="examples"></a>Примеры

#### <a name="how-to-collect-events"></a>Получение событий

Обычно мы попросим вас собраться с событиями, чтобы выяснить, почему приложение не оснащено.

Среда выполнения присоединения, не имеющая кода, будет создавать события ETW при запуске IIS и при запуске приложения.

Чтобы получить эти события, сделайте следующее:
1. В консоли CMD с правами администратора выполните команду, `iisreset /stop` чтобы отключить службы IIS и все веб-приложения.
2. Выполните этот командлет
3. В консоли CMD с правами администратора выполните команду, `iisreset /start` чтобы запустить IIS.
4. Попробуйте перейти к приложению.
5. После завершения загрузки приложения можно вручную его отключить ( `Ctrl + C` ) или подождать истечения времени ожидания.

#### <a name="what-events-to-collect"></a>События для собраний

При сборе событий можно использовать три варианта:
1. Используйте параметр `-CollectSdkEvents` для получения сведений о событиях, выдаваемых из пакета SDK для Application Insights.
2. Используйте параметр, `-CollectRedfieldEvents` чтобы получить события, созданные монитор состояния и средой выполнения Redfield. Эти журналы полезны при диагностике служб IIS и запуска приложений.
3. Используйте оба параметра для накопления обоих типов событий.
4. По умолчанию, если параметр не указан, будут собираться оба типа событий.


### <a name="parameters"></a>Параметры

#### <a name="-maxdurationinminutes"></a>-Максдуратионинминутес
**Необязательный элемент.** Используйте этот параметр, чтобы задать время, в течение которого этот скрипт должен получать события. Значение по умолчанию — 5 минут.

#### <a name="-logdirectory"></a>-LogDirectory
**Необязательный элемент.** Используйте этот параметр, чтобы задать выходной каталог ETL-файла. По умолчанию этот файл будет создан в каталоге модулей PowerShell. Полный путь будет отображаться во время выполнения скрипта.


#### <a name="-collectsdkevents"></a>-Коллектсдкевентс
**Необязательный элемент.** Используйте этот параметр для получения Application Insights событий пакета SDK.

#### <a name="-collectredfieldevents"></a>-Коллектредфиелдевентс
**Необязательный элемент.** Используйте этот параметр для получения событий из монитор состояния и среды выполнения Redfield.

#### <a name="-verbose"></a>-Verbose
**Общий параметр.** Используйте этот параметр для вывода подробных журналов.



### <a name="output"></a>Выходные данные


#### <a name="example-of-application-startup-logs"></a>Пример журналов запуска приложений
```powershell
PS C:\Windows\system32> Start-ApplicationInsightsMonitoringTrace -ColectRedfieldEvents
Starting...
Log File: C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\logs\20190627_144217_ApplicationInsights_ETW_Trace.etl
Tracing enabled, waiting for events.
Tracing will timeout in 5 minutes. Press CTRL+C to cancel.

2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftAppInsights_ManagedHttpModulePath='C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll', MicrosoftAppInsights_ManagedHttpModuleType='Microsoft.ApplicationInsights.RedfieldIISModule.RedfieldIISModule'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftDiagnosticServices_ManagedHttpModulePath2='', MicrosoftDiagnosticServices_ManagedHttpModuleType2=''
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Environment variable 'MicrosoftDiagnosticServices_ManagedHttpModulePath2' or 'MicrosoftDiagnosticServices_ManagedHttpModuleType2' is null, skipping managed dll loading
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace MulticastHttpModule.constructor, success, 70 ms
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Current assembly 'Microsoft.ApplicationInsights.RedfieldIISModule, Version=2.8.18.27202, Culture=neutral, PublicKeyToken=f23a46de0be5d6f3' location 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Matched filter '.*'~'STATUSMONITORTE', '.*'~'DemoWithSql'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Lightup assembly calculated path: 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.Redfield.Lightup.dll'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-FrameworkLightup Trace Loaded applicationInsights.config from assembly's resource Microsoft.ApplicationInsights.Redfield.Lightup, Version=2.8.18.27202, Culture=neutral, PublicKeyToken=f23a46de0be5d6f3/Microsoft.ApplicationInsights.Redfield.Lightup.ApplicationInsights-recommended.config
2:42:34 PM EVENT: Microsoft-ApplicationInsights-FrameworkLightup Trace Successfully attached ApplicationInsights SDK
2:42:34 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.LoadLightupAssemblyAndGetLightupHttpModuleClass, success, 2687 ms
2:42:34 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.CreateAndInitializeApplicationInsightsHttpModules(lightupHttpModuleClass), success
2:42:34 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace ManagedHttpModuleHelper, multicastHttpModule.Init() success, 3288 ms
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftAppInsights_ManagedHttpModulePath='C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll', MicrosoftAppInsights_ManagedHttpModuleType='Microsoft.ApplicationInsights.RedfieldIISModule.RedfieldIISModule'
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftDiagnosticServices_ManagedHttpModulePath2='', MicrosoftDiagnosticServices_ManagedHttpModuleType2=''
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Environment variable 'MicrosoftDiagnosticServices_ManagedHttpModulePath2' or 'MicrosoftDiagnosticServices_ManagedHttpModuleType2' is null, skipping managed dll loading
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace MulticastHttpModule.constructor, success, 0 ms
2:42:35 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.CreateAndInitializeApplicationInsightsHttpModules(lightupHttpModuleClass), success
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace ManagedHttpModuleHelper, multicastHttpModule.Init() success, 0 ms
Timeout Reached. Stopping...
```

## <a name="next-steps"></a>Дальнейшие действия

  Просмотр телеметрии:
 - [Изучите метрики](../essentials/metrics-charts.md) для мониторинга производительности и использования.
- [Поиск событий и журналов](./diagnostic-search.md) для диагностики проблем.
- Используйте [аналитику](../logs/log-query-overview.md) для более сложных запросов.
- [Создание панелей мониторинга](./overview-dashboard.md).
 
 Добавление данных телеметрии:
 - [Создайте веб-тесты](monitor-web-app-availability.md), чтобы убедиться, что ваш сайт продолжает работать.
- [Добавьте данные телеметрии веб-клиента](./javascript.md) , чтобы просмотреть исключения из кода веб-страницы и включить вызовы трассировки.
- [Добавьте в код пакет SDK для Application Insights](./asp-net.md) , чтобы можно было вставить вызовы трассировки и журнала.
 
 Другие действия с агентом Application Insights:
 - Используйте наше справочное по для [устранения неполадок](status-monitor-v2-troubleshoot.md) агента Application Insights.


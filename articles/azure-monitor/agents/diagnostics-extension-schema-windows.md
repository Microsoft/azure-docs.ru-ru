---
title: Схема расширения диагностики Windows
description: Справочник по схеме конфигурации для расширения диагностики Windows (WAD) в Azure Monitor.
ms.topic: reference
author: bwren
ms.author: bwren
ms.date: 01/20/2020
ms.openlocfilehash: 70b27fec07d074dadb413d1debb098e23b4d33b3
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102428734"
---
# <a name="windows-diagnostics-extension-schema"></a>Схема расширения диагностики Windows
Расширение система диагностики Azure — это агент в Azure Monitor, собирающий данные мониторинга из операционной системы на виртуальной машине и рабочих нагрузок ресурсов вычислений Azure. В этой статье подробно описывается схема, используемая для настройки расширения системы диагностики на виртуальных машинах Windows и других ресурсов вычислений.

> [!NOTE]
> Схема в этой статье действительна для версий 1,3 и более новых (пакет Azure SDK 2,4 и более поздние версии). Новые разделы конфигурации снабжены комментариями, указывающими, в какой версии они были добавлены. Версия 1,0 и 1,2 схемы были заархивированы и больше не доступны. 

## <a name="public-configuration-file-schema"></a>Схема файла общедоступной конфигурации

Скачайте общедоступное определение схемы файла конфигурации, выполнив следующую команду PowerShell:  

```powershell  
(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File –Encoding utf8 -FilePath 'C:\temp\WadConfig.xsd'  
```  


## <a name="common-attribute-types"></a>Общие типы атрибутов  
 Атрибут **scheduledTransferPeriod** присутствует в нескольких элементах. Это интервал между запланированными передачами в службу хранилища, округленный с точностью до ближайшей минуты. Значение относится к [типу данных XML "Duration"](https://www.w3schools.com/xml/schema_dtypes_date.asp).


## <a name="diagnosticsconfiguration-element"></a>Элемент DiagnosticsConfiguration  
 *Дерево: корневой элемент — DiagnosticsConfiguration*

Добавлен в версии 1.3.  

Элемент верхнего уровня в файле конфигурации диагностики.  

**Атрибут**: xmlns. Пространство имен XML для файла конфигурации диагностики:  
`http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration`


|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**PublicConfig**|Обязательный. Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**PrivateConfig**|Необязательный параметр. Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**IsEnabled**|Логическое. Ознакомьтесь с описанием в другом разделе на этой странице.|  

## <a name="publicconfig-element"></a>Элемент PublicConfig  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig*

 Описывает общедоступную конфигурацию диагностики.  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**WadCfg**|Обязательный. Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**StorageAccount**|Имя учетной записи хранения Azure для хранения данных. Может также быть указан как параметр при выполнении командлета Set-AzureServiceDiagnosticsExtension.|  
|**StorageType**|Может быть *таблицей*, *большим двоичным объектом* или *TableAndBlob*. Таблица — это значение по умолчанию. При выборе TableAndBlob диагностические данные записываются дважды (по одному разу на каждый тип).|  
|**LocalResourceDirectory**|Каталог на виртуальной машине, в котором Monitoring Agent хранит данные событий. Если этот параметр не задан, используется каталог по умолчанию:<br /><br /> для рабочей роли или веб-роли: `C:\Resources\<guid>\directory\<guid>.<RoleName.DiagnosticStore\`<br /><br /> для виртуальной машины: `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<WADVersion>\WAD<WADVersion>`<br /><br /> Ниже перечислены обязательные атрибуты.<br /><br /> - **path**: каталог в системе для использования системой диагностики Azure.<br /><br /> - **expandEnvironment**: позволяет раскрыть переменные среды в пути.|  

## <a name="wadcfg-element"></a>Элемент WadCFG  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig, WadCFG*

 Позволяет определить и настроить сбор данных телеметрии.  


## <a name="diagnosticmonitorconfiguration-element"></a>Элемент DiagnosticMonitorConfiguration
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig, WadCFG, DiagnosticMonitorConfiguration*

 Обязательно

|Атрибуты|Описание|  
|----------------|-----------------|  
| **overallQuotaInMB** | Максимальный объем пространства на локальном жестком диске, доступный для диагностических данных различного типа, собранных системой диагностики Azure. Значение по умолчанию составляет 4096 МБ.<br />
|**useProxyServer** | Позволяет настроить систему диагностики Azure для использования параметров прокси-сервера, указанных в настройках IE.|
|**sinks** | Добавлено в версии 1.5. Необязательный параметр. Указывает расположение приемника для отправки диагностических данных для всех дочерних элементов с соответствующей поддержкой. Пример приемника — Application Insights или Центры событий Azure. Обратите внимание, что необходимо добавить свойство *resourceId* в элемент *метрики* , если требуется, чтобы события, отправляемые в концентраторы событий, имели идентификатор ресурса. |  


<br /> <br />

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**CrashDumps**|Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**DiagnosticInfrastructureLogs**|Включает сбор журналов, создаваемых системой диагностикой Azure. Журналы инфраструктуры диагностики удобны для устранения неполадок в самой системе диагностики. Необязательные атрибуты:<br /><br /> - **scheduledTransferLogLevelFilter**: задает минимальный уровень серьезности собираемых журналов.<br /><br /> - **scheduledTransferPeriod** — интервал между запланированными передачами в хранилище, округленный до ближайшей минуты. Значение относится к [типу данных XML "Duration"](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  
|**Каталоги**|Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**EtwProviders**|Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**Метрики**|Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**PerformanceCounters**|Ознакомьтесь с описанием в другом разделе на этой странице.|  
|**WindowsEventLog**|Ознакомьтесь с описанием в другом разделе на этой странице.|
|**DockerSources**|Ознакомьтесь с описанием в другом разделе на этой странице. |



## <a name="crashdumps-element"></a>Элемент CrashDumps  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — CrashDumps*

 Включает сбор аварийных дампов.  

|Атрибуты|Описание|  
|----------------|-----------------|  
|**containerName**|Необязательный параметр. Имя контейнера больших двоичных объектов в вашей учетной записи хранения Azure, используемого для хранения аварийных дампов.|  
|**crashDumpType**|Необязательный параметр.  Позволяет настроить систему диагностики Azure для сбора мини-дампов или полных дампов.|  
|**directoryQuotaPercentage**|Необязательный параметр.  Позволяет настроить процент от значения **overallQuotaInMB**, резервируемый для аварийных дампов на виртуальной машине.|  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**CrashDumpConfiguration**|Обязательный. Определяет значения конфигурации для каждого процесса.<br /><br /> Следующий атрибут также является обязательным:<br /><br /> **processName**: имя процесса, для которого системе диагностики Azure нужно собирать аварийные дампы.|  

## <a name="directories-element"></a>Элемент Directories
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — Directories*

 Включает сбор содержимого каталога, журналов невыполненных запросов на вход IIS и (или) журналов IIS.  

 Необязательный атрибут **scheduledTransferPeriod**. Ознакомьтесь с описанием выше.  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**IISLogs**|Если добавить этот элемент в конфигурацию, то будет включен сбор журналов IIS.<br /><br /> **containerName**: имя контейнера больших двоичных объектов в вашей учетной записи хранения Azure, используемого для хранения журналов IIS.|   
|**FailedRequestLogs**|Если добавить этот элемент в конфигурацию, то будет включен сбор журналов о невыполненных запросах к сайту или приложению IIS. Вам также необходимо включить параметры трассировки в разделе **system.WebServer** файла **Web.config**.|  
|**Коллекция DataSources**|Задает список отслеживаемых каталогов.|




## <a name="datasources-element"></a>Элемент DataSources  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — Directories — DataSources*

 Задает список отслеживаемых каталогов.  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**DirectoryConfiguration**|Обязательный. Обязательный атрибут:<br /><br /> **containerName**: имя контейнера больших двоичных объектов в вашей учетной записи хранения Azure, используемого для хранения файлов журнала.|  





## <a name="directoryconfiguration-element"></a>Элемент DirectoryConfiguration  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — Directories — DataSources — DirectoryConfiguration*

 Может содержать только один из элементов **Absolute** и **LocalResource**, но не оба.  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**Минималь**|Абсолютный путь к отслеживаемому каталогу. Ниже приведены обязательные атрибуты.<br /><br /> - **Path**: абсолютный путь к отслеживаемому каталогу.<br /><br /> - **expandEnvironment**: позволяет раскрыть переменные среды, указанные в пути.|  
|**LocalResource**|Путь относительно отслеживаемого локального ресурса. Ниже перечислены обязательные атрибуты.<br /><br /> - **Name**: локальный ресурс, который содержит каталог для отслеживания.<br /><br /> - **relativePath**: путь относительно значения Name, содержащего отслеживаемый каталог.|  



## <a name="etwproviders-element"></a>Элемент EtwProviders  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — EtwProviders*

 Настраивает сбор событий ETW (трассировка событий Windows) из поставщиков на основе EventSource и (или) манифеста ETW.  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**EtwEventSourceProviderConfiguration**|Позволяет настроить сбор событий, создаваемых из [класса EventSource](/dotnet/api/system.diagnostics.tracing.eventsource). Обязательный атрибут:<br /><br /> **provider**: имя класса события EventSource.<br /><br /> Необязательные атрибуты:<br /><br /> - **scheduledTransferLogLevelFilter**: минимальный уровень серьезности события для переноса в вашу учетную запись хранения.<br /><br /> - **scheduledTransferPeriod** — интервал между запланированными передачами в хранилище, округленный до ближайшей минуты. Значение относится к [типу данных XML "Duration"](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  
|**EtwManifestProviderConfiguration**|Обязательный атрибут:<br /><br /> **provider**: GUID поставщика событий.<br /><br /> Необязательные атрибуты:<br /><br /> - **scheduledTransferLogLevelFilter**: минимальный уровень серьезности события для переноса в вашу учетную запись хранения.<br /><br /> - **scheduledTransferPeriod** — интервал между запланированными передачами в хранилище, округленный до ближайшей минуты. Значение относится к [типу данных XML "Duration"](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  



## <a name="etweventsourceproviderconfiguration-element"></a>Элемент EtwEventSourceProviderConfiguration  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — EtwProviders — EtwEventSourceProviderConfiguration*

 Позволяет настроить сбор событий, создаваемых из [класса EventSource](/dotnet/api/system.diagnostics.tracing.eventsource).  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**DefaultEvents**|Необязательный атрибут:<br/><br/> **eventDestination**: имя таблицы для хранения событий.|  
|**Событие**|Обязательный атрибут:<br /><br /> **id**: идентификатор события.<br /><br /> Необязательный атрибут:<br /><br /> **eventDestination**: имя таблицы для хранения событий.|  



## <a name="etwmanifestproviderconfiguration-element"></a>Элемент EtwManifestProviderConfiguration  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — EtwProviders — EtwManifestProviderConfiguration*

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**DefaultEvents**|Необязательный атрибут:<br /><br /> **eventDestination**: имя таблицы для хранения событий.|  
|**Событие**|Обязательный атрибут:<br /><br /> **id**: идентификатор события.<br /><br /> Необязательный атрибут:<br /><br /> **eventDestination**: имя таблицы для хранения событий.|  



## <a name="metrics-element"></a>Элемент Metrics  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — Metrics*

 Позволяет создать таблицу счетчиков производительности, которая оптимизирована для быстрых запросов. Каждый счетчик производительности, который определен в элементе **PerformanceCounters**, помимо таблицы счетчиков производительности хранится в таблице метрик.  

 Атрибут **ResourceId** является обязательным.  Это идентификатор ресурса виртуальной машины или масштабируемого набора виртуальных машин, в котором развертывается система диагностики Azure. Значение **resourceID** можно получить на [портале Azure](https://portal.azure.com). Выберите **Обзор**  ->  **группы ресурсов**  ->  **<имя \>**. Щелкните элемент **Свойства** и скопируйте значение поля **Идентификатор**.  Это свойство resourceID используется для отправки пользовательских метрик и для добавления свойства resourceID в данные, отправляемые в концентраторы событий. Обратите внимание, что необходимо добавить свойство *resourceId* в элемент *метрики* , если требуется, чтобы события, отправляемые в концентраторы событий, имели идентификатор ресурса.

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**MetricAggregation**|Обязательный атрибут:<br /><br /> **scheduledTransferPeriod**: интервал между запланированными передачами в службу хранилища, округленный с точностью до ближайшей минуты. Значение относится к [типу данных XML "Duration"](https://www.w3schools.com/xml/schema_dtypes_date.asp). |  



## <a name="performancecounters-element"></a>Элемент PerformanceCounters  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — PerformanceCounters*

 Включает сбор данных счетчиков производительности.  

 Необязательный атрибут:  

 Необязательный атрибут **scheduledTransferPeriod**. Ознакомьтесь с описанием выше.

|Дочерний элемент|Описание|  
|-------------------|-----------------|  
|**PerformanceCounterConfiguration**|Ниже приведены обязательные атрибуты.<br /><br /> - **counterSpecifier** — имя счетчика производительности. Например, `\Processor(_Total)\% Processor Time`. Чтобы получить список счетчиков производительности на узле, выполните команду `typeperf`.<br /><br /> - **sampleRate**: частота выборки для счетчика.<br /><br /> Необязательный атрибут:<br /><br /> **unit**: единица измерения счетчика. Значения доступны в [классе единицах UnitType](/dotnet/api/microsoft.azure.management.sql.models.unittype) |
|**sinks** | Добавлено в версии 1.5. Необязательный параметр. Указывает расположение приемника для отправки диагностических данных. Например, Azure Monitor или Центры событий. Обратите внимание, что необходимо добавить свойство *resourceId* в элемент *метрики* , если требуется, чтобы события, отправляемые в концентраторы событий, имели идентификатор ресурса.|    




## <a name="windowseventlog-element"></a>Элемент WindowsEventLog
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — WindowsEventLog*

 Включает сбор журналов событий Windows.  

 Необязательный атрибут **scheduledTransferPeriod**. Ознакомьтесь с описанием выше.  

|Дочерний элемент|Описание|  
|-------------------|-----------------|  
|**DataSource**|Собираемые журналы событий Windows. Обязательный атрибут:<br /><br /> **name** — запрос XPath, описывающий собираемые события Windows. Пример:<br /><br /> `Application!*[System[(Level <=3)]], System!*[System[(Level <=3)]], System!*[System[Provider[@Name='Microsoft Antimalware']]], Security!*[System[(Level <= 3)]`<br /><br /> Для сбора всех событий укажите "*". |
|**sinks** | Добавлено в версии 1.5. Необязательный параметр. Указывает расположение приемника для отправки диагностических данных для всех дочерних элементов с соответствующей поддержкой. Пример приемника — Application Insights или Центры событий Azure.|  


## <a name="logs-element"></a>Элемент Logs  
 *Дерево:корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — Logs*

 Используется в версиях 1.0 и 1.1. Отсутствует в версии 1.2. Снова добавлен в версии 1.3.  

 Определяет конфигурацию буфера для базовых журналов Azure.  

|attribute|Тип|Описание|  
|---------------|----------|-----------------|  
|**bufferQuotaInMB**|**unsignedInt**|Необязательный параметр. Указывает максимальный объем хранилища файловой системы, который доступен для указанных данных.<br /><br /> Значение по умолчанию равно 0.|  
|**scheduledTransferLogLevelFilter**|**строка**|Необязательный параметр. Указывает минимальный уровень серьезности для передаваемых записей журнала. Значение по умолчанию — **Undefined**, при котором передаются все журналы. Другие возможные значения (в порядке убывания информативности): **Verbose**, **Information**, **Warning**, **Error** и **Critical**.|  
|**scheduledTransferPeriod**|**duration**|Необязательный параметр. Указывает интервал между запланированными передачами данных, округленный с точностью до ближайшей минуты.<br /><br /> По умолчанию используется значение PT0S.|  
|**sinks** |**строка**| Добавлено в версии 1.5. Необязательный параметр. Указывает расположение приемника для отправки диагностических данных. Например, Application Insights или Центры событий Azure. Обратите внимание, что необходимо добавить свойство *resourceId* в элемент *метрики* , если требуется, чтобы события, отправляемые в концентраторы событий, имели идентификатор ресурса.|  

## <a name="dockersources"></a>DockerSources
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — DiagnosticMonitorConfiguration — DockerSources*

 Добавлен в версии 1.9.

|Имя элемента|Описание|  
|------------------|-----------------|  
|**Статистика**|Указывает системе, что нужно собрать статистику для контейнеров Docker.|  

## <a name="sinksconfig-element"></a>Элемент SinksConfig  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — SinksConfig*

 Содержит список расположений для отправки диагностических данных и конфигурацию, связанную с этими расположениями.  

|Имя элемента|Описание|  
|------------------|-----------------|  
|**Приемник**|Ознакомьтесь с описанием в другом разделе на этой странице.|  

## <a name="sink-element"></a>Элемент Sink
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — SinksConfig — Sink*

 Добавлен в версии 1.5.  

 Определяет расположение для отправки диагностических данных. Например, это может быть служба Application Insights.  

|attribute|Тип|Описание|  
|---------------|----------|-----------------|  
|**name**|строка|Строка, определяющая имя приемника.|  

|Элемент|Type|Описание|  
|-------------|----------|-----------------|  
|**Application Insights**|строка|Используется только при отправке данных в Application Insights. Содержит ключ инструментирования для активной учетной записи Application Insights, к которой у вас есть доступ.|  
|**Каналы**|строка|Указывается для каждой дополнительной фильтрации, используемой при потоковой передаче.|  

## <a name="channels-element"></a>Элемент Channels  
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — SinksConfig — Sink — Channels*

 Добавлен в версии 1.5.  

 Определяет фильтры для потоков данных журнала, проходящих через приемник.  

|Элемент|Type|Описание|  
|-------------|----------|-----------------|  
|**Канал**|строка|Ознакомьтесь с описанием в другом разделе на этой странице.|  

## <a name="channel-element"></a>Элемент Channel
 *Дерево: корневой элемент — DiagnosticsConfiguration — PublicConfig — WadCFG — SinksConfig — Sink — Channels — Channel*

 Добавлен в версии 1.5.  

 Определяет расположение для отправки диагностических данных. Например, это может быть служба Application Insights.  

|Атрибуты|Type|Описание|  
|----------------|----------|-----------------|  
|**logLevel**|**строка**|Указывает минимальный уровень серьезности для передаваемых записей журнала. Значение по умолчанию — **Undefined**, при котором передаются все журналы. Другие возможные значения (в порядке убывания информативности): **Verbose**, **Information**, **Warning**, **Error** и **Critical**.|  
|**name**|**строка**|Уникальное имя для использования ссылки на канал.|  


## <a name="privateconfig-element"></a>Элемент PrivateConfig
 *Дерево: корневой элемент — DiagnosticsConfiguration — PrivateConfig*

 Добавлен в версии 1.3.  

 Необязательно  

 Хранит частные сведения об учетной записи хранения (имя, ключ и конечную точку). Эта информация отправляется на виртуальную машину, но извлечь ее из виртуальной машины невозможно.  

|Дочерние элементы|Описание|  
|--------------------|-----------------|  
|**StorageAccount**|Используемая учетная запись хранения. Ниже приведены обязательные атрибуты.<br /><br /> - **name**: имя учетной записи хранения.<br /><br /> - **key**: ключ учетной записи хранения.<br /><br /> - **endpoint**: конечная точка для доступа к учетной записи хранения. <br /><br /> -**sasToken** (Добавлено 1.8.1) — вы можете указать маркер SAS вместо ключа учетной записи хранения в частной конфигурации. Если этот параметр указан, ключ учетной записи хранения игнорируется. <br />Требования к маркеру SAS: <br />Поддерживает только маркер SAS учетной записи. <br />Требуемые типы служб: - *b*, *t*. <br /> Требуемые разрешения: - *a*, *c*, *u*, *w*. <br /> Требуемые типы ресурсов: - *c*, *o*. <br /> Поддерживает только протокол HTTPS. <br /> Время начала и окончания срока действия должно быть допустимым.|  


## <a name="isenabled-element"></a>Элемент IsEnabled  
 *Дерево: корневой элемент — DiagnosticsConfiguration — IsEnabled*

 Логическое. Используйте `true`, чтобы включить диагностику, или `false`, чтобы отключить ее.

## <a name="example-configuration"></a>Пример конфигурации
 Ниже приведен полный пример конфигурации для расширения системы диагностики Windows, показанного как в JSON, так и в XML.

 
### <a name="json"></a>JSON

*PublicConfig* и *PrivateConfig* разделены, так как в большинстве случаев использования JSON они передаются как разные переменные. К таким случаям относятся шаблоны диспетчер ресурсов, PowerShell и Visual Studio.

> [!NOTE]
> Определение приемника Azure Monitor общедоступной конфигурации имеет два свойства: *resourceId* и *регион*. Для работы классических Виртуальных машин и классических облачных служб требуются только они. Свойство *Region* не должно использоваться для других ресурсов, свойство *ResourceId* используется на виртуальных машинах ARM для заполнения поля resourceId в журналах, передаваемых в концентраторы событий.

```json
"PublicConfig" {
    "WadCfg": {
        "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": 10000,
            "DiagnosticInfrastructureLogs": {
                "scheduledTransferLogLevelFilter": "Error"
            },
            "PerformanceCounters": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "AzureMonitorSink",
                "PerformanceCounterConfiguration": [
                    {
                        "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                        "sampleRate": "PT1M",
                        "unit": "percent"
                    }
                ]
            },
            "Directories": {
                "scheduledTransferPeriod": "PT5M",
                "IISLogs": {
                    "containerName": "iislogs"
                },
                "FailedRequestLogs": {
                    "containerName": "iisfailed"
                },
                "DataSources": [
                    {
                        "containerName": "mynewprocess",
                        "Absolute": {
                            "path": "C:\\MyNewProcess",
                            "expandEnvironment": false
                        }
                    },
                    {
                        "containerName": "badapp",
                        "Absolute": {
                            "path": "%SYSTEMDRIVE%\\BadApp",
                            "expandEnvironment": true
                        }
                    },
                    {
                        "containerName": "goodapp",
                        "LocalResource": {
                            "relativePath": "..\\PeanutButter",
                            "name": "Skippy"
                        }
                    }
                ]
            },
            "EtwProviders": {
                "sinks": "",
                "EtwEventSourceProviderConfiguration": [
                    {
                        "scheduledTransferPeriod": "PT5M",
                        "provider": "MyProviderClass",
                        "Event": [
                            {
                                "id": 0
                            },
                            {
                                "id": 1,
                                "eventDestination": "errorTable"
                            }
                        ],
                        "DefaultEvents": {
                        }
                    }
                ],
                "EtwManifestProviderConfiguration": [
                    {
                        "scheduledTransferPeriod": "PT2M",
                        "scheduledTransferLogLevelFilter": "Information",
                        "provider": "5974b00b-84c2-44bc-9e58-3a2451b4e3ad",
                        "Event": [
                            {
                                "id": 0
                            }
                        ],
                        "DefaultEvents": {
                        }
                    }
                ]
            },
            "WindowsEventLog": {
                "scheduledTransferPeriod": "PT5M",
                "DataSource": [
                    {
                        "name": "System!*[System[Provider[@Name='Microsoft Antimalware']]]"
                    },
                    {
                        "name": "System!*[System[Provider[@Name='NTFS'] and (EventID=55)]]"
                    },
                    {
                        "name": "System!*[System[Provider[@Name='disk'] and (EventID=7 or EventID=52 or EventID=55)]]"
                    }
                ]
            },
            "Logs": {
                "scheduledTransferPeriod": "PT1M",
                "scheduledTransferLogLevelFilter": "Verbose",
                "sinks": "ApplicationInsights.AppLogs"
            },
            "CrashDumps": {
                "directoryQuotaPercentage": 30,
                "dumpType": "Mini",
                "containerName": "wad-crashdumps",
                "CrashDumpConfiguration": [
                    {
                        "processName": "mynewprocess.exe"
                    },
                    {
                        "processName": "badapp.exe"
                    }
                ]
            }
        },
        "SinksConfig": {
            "Sink": [
                {
                    "name": "AzureMonitorSink",
                    "AzureMonitor":
                    {
                        "ResourceId": "{insert resourceId if a classic VM or cloud service, else property not needed}",
                        "Region": "{insert Azure region of resource if a classic VM or cloud service, else property not needed}"
                    }
                },
                {
                    "name": "ApplicationInsights",
                    "ApplicationInsights": "{Insert InstrumentationKey}",
                    "Channels": {
                        "Channel": [
                            {
                                "logLevel": "Error",
                                "name": "Errors"
                            },
                            {
                                "logLevel": "Verbose",
                                "name": "AppLogs"
                            }
                        ]
                    }
                },
                {
                    "name": "EventHub",
                    "EventHub": {
                        "Url": "https://myeventhub-ns.servicebus.windows.net/diageventhub",
                        "SharedAccessKeyName": "SendRule",
                        "usePublisherId": false
                    }
                },
                {
                    "name": "secondaryEventHub",
                    "EventHub": {
                        "Url": "https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub",
                        "SharedAccessKeyName": "SendRule",
                        "usePublisherId": false
                    }
                },
                {
                    "name": "secondaryStorageAccount",
                    "StorageAccount": {
                        "name": "secondarydiagstorageaccount",
                        "endpoint": "https://core.windows.net"
                    }
                }
            ]
        }
    },
    "StorageAccount": "diagstorageaccount",
    "StorageType": "TableAndBlob"
}
```

> [!NOTE]
> Определение приемника Azure Monitor частного файла конфигурации имеет два свойства: *PrincipalId* и *Secret*. Для работы классических Виртуальных машин и классических облачных служб требуются только они. Эти свойства не следует использовать для других ресурсов.


```json
"PrivateConfig" {
    "storageAccountName": "diagstorageaccount",
    "storageAccountKey": "{base64 encoded key}",
    "storageAccountEndPoint": "https://core.windows.net",
    "storageAccountSasToken": "{sas token}",
    "EventHub": {
        "Url": "https://myeventhub-ns.servicebus.windows.net/diageventhub",
        "SharedAccessKeyName": "SendRule",
        "SharedAccessKey": "{base64 encoded key}"
    },
    "AzureMonitorAccount": {
        "ServicePrincipalMeta": {
            "PrincipalId": "{Insert service principal client Id}",
            "Secret": "{Insert service principal client secret}"
        }
    },
    "SecondaryStorageAccounts": {
        "StorageAccount": [
            {
                "name": "secondarydiagstorageaccount",
                "key": "{base64 encoded key}",
                "endpoint": "https://core.windows.net",
                "sasToken": "{sas token}"
            }
        ]
    },
    "SecondaryEventHubs": {
        "EventHub": [
            {
                "Url": "https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub",
                "SharedAccessKeyName": "SendRule",
                "SharedAccessKey": "{base64 encoded key}"
            }
        ]
    }
}

```

### <a name="xml"></a>XML

```xml  
<?xml version="1.0" encoding="utf-8"?>  
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">   
  <PublicConfig>  
    <WadCfg>  
      <DiagnosticMonitorConfiguration overallQuotaInMB="10000">  

        <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzureMonitorSink">  
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />  
        </PerformanceCounters>  

        <Directories scheduledTransferPeriod="PT5M">  
          <IISLogs containerName="iislogs" />  
          <FailedRequestLogs containerName="iisfailed" />  

          <DataSources>  
            <DirectoryConfiguration containerName="mynewprocess">  
              <Absolute path="C:\MyNewProcess" expandEnvironment="false" />  
            </DirectoryConfiguration>  
            <DirectoryConfiguration containerName="badapp">  
              <Absolute path="%SYSTEMDRIVE%\BadApp" expandEnvironment="true" />  
            </DirectoryConfiguration>  
            <DirectoryConfiguration containerName="goodapp">  
              <LocalResource name="Skippy" relativePath="..\PeanutButter"/>  
            </DirectoryConfiguration>  
          </DataSources>  

        </Directories>  

        <EtwProviders>  
          <EtwEventSourceProviderConfiguration   
                       provider="MyProviderClass"   
                       scheduledTransferPeriod="PT5M">  
            <Event id="0"/>  
            <Event id="1" eventDestination="errorTable"/>  
            <DefaultEvents />  
          </EtwEventSourceProviderConfiguration>  
          <EtwManifestProviderConfiguration provider="5974b00b-84c2-44bc-9e58-3a2451b4e3ad" scheduledTransferLogLevelFilter="Information" scheduledTransferPeriod="PT2M">  
            <Event id="0"/>  
            <DefaultEvents eventDestination="defaultTable"/>  
          </EtwManifestProviderConfiguration>  
        </EtwProviders>  

        <WindowsEventLog scheduledTransferPeriod="PT5M">  
          <DataSource name="System!*[System[Provider[@Name='Microsoft Antimalware']]]"/>  
          <DataSource name="System!*[System[Provider[@Name='NTFS'] and (EventID=55)]]" />  
          <DataSource name="System!*[System[Provider[@Name='disk'] and (EventID=7 or EventID=52 or EventID=55)]]" />  
        </WindowsEventLog>  

        <Logs  bufferQuotaInMB="1024"   
             scheduledTransferPeriod="PT1M"   
             scheduledTransferLogLevelFilter="Verbose"   
             sinks="ApplicationInsights.AppLogs"/>  <!-- sinks attribute added in 1.5 -->  

        <CrashDumps containerName="wad-crashdumps" directoryQuotaPercentage="30" dumpType="Mini">  
          <CrashDumpConfiguration processName="mynewprocess.exe" />  
          <CrashDumpConfiguration processName="badapp.exe"/>  
        </CrashDumps>  

        <DockerSources> <!-- Added in 1.9 -->
          <Stats enabled="true" sampleRate="PT1M" scheduledTransferPeriod="PT1M" />
        </DockerSources>

      </DiagnosticMonitorConfiguration>  

      <SinksConfig>   <!-- Added in 1.5 -->  
        <Sink name="AzureMonitorSink">
            <AzureMonitor> <!-- Added in 1.11 -->
                <resourceId>{insert resourceId}</ResourceId> <!-- Parameter only needed for classic VMs and Classic Cloud Services, exclude VMSS and Resource Manager VMs-->
                <Region>{insert Azure region of resource}</Region> <!-- Parameter only needed for classic VMs and Classic Cloud Services, exclude VMSS and Resource Manager VMs -->
            </AzureMonitor>
        </Sink>
        <Sink name="ApplicationInsights">   
          <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>   
          <Channels>   
            <Channel logLevel="Error" name="Errors"  />   
            <Channel logLevel="Verbose" name="AppLogs"  />   
          </Channels>   
        </Sink>   
        <Sink name="EventHub"> <!-- Added in 1.7 -->
          <EventHub Url="https://myeventhub-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" usePublisherId="false" />
        </Sink>
        <Sink name="secondaryEventHub"> <!-- Added in 1.7 -->
          <EventHub Url="https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub" SharedAccessKeyName="SendRule" usePublisherId="false" />
        </Sink>
        <Sink name="secondaryStorageAccount"> <!-- Added in 1.7 -->
          <StorageAccount name="secondarydiagstorageaccount" endpoint="https://core.windows.net" />
        </Sink>
   </SinksConfig>

  </WadCfg>  

  <StorageAccount>diagstorageaccount</StorageAccount>
  <StorageType>TableAndBlob</StorageType> <!-- Added in 1.8 -->  
  </PublicConfig>  

  <PrivateConfig>  <!-- Added in 1.3 -->  
    <StorageAccount name="" key="" endpoint="" sasToken="{sas token}"  />  <!-- sasToken in Private config added in 1.8.1 -->  
    <EventHub Url="https://myeventhub-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="{base64 encoded key}" />

    <AzureMonitorAccount>
        <ServicePrincipalMeta> <!-- Added in 1.11; only needed for classic VMs and Classic cloud services -->
            <PrincipalId>{Insert service principal clientId}</PrincipalId>
            <Secret>{Insert service principal client secret}</Secret>
        </ServicePrincipalMeta>
    </AzureMonitorAccount>

    <SecondaryStorageAccounts>
       <StorageAccount name="secondarydiagstorageaccount" key="{base64 encoded key}" endpoint="https://core.windows.net" sasToken="{sas token}" />
    </SecondaryStorageAccounts>

    <SecondaryEventHubs>
       <EventHub Url="https://myeventhub-ns.servicebus.windows.net/secondarydiageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="{base64 encoded key}" />
    </SecondaryEventHubs>

  </PrivateConfig>  
  <IsEnabled>true</IsEnabled>  
</DiagnosticsConfiguration>  

```  
> [!NOTE]
> Определение общедоступной конфигурации приемника Azure Monitor включает два свойства: resourceId и region. Для работы классических Виртуальных машин и классических облачных служб требуются только они. Эти свойства не следует использовать для Виртуальных машин Resource Manager или масштабируемых наборов виртуальных машин.
> Также доступен дополнительный элемент закрытой конфигурации приемника Azure Monitor, который передает секрет и идентификатор субъекта. Для работы классических виртуальных машин и классических облачных служб требуются только они. Для работы виртуальных машин Resource Manager и масштабируемого набора виртуальных машин определение Azure Monitor в элементе закрытой конфигурации можно исключить.
>

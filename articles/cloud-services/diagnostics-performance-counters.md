---
title: Получение сведений о счетчиках производительности в облачных службах Azure (классическая модель) | Документация Майкрософт
description: Узнайте, как находить, использовать и создавать счетчики производительности в облачных службах с помощью системы диагностики Azure и Application Insights.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: fa5dd61c0764be45cdba68b73a4f55745ee5e55a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100585505"
---
# <a name="collect-performance-counters-for-your-azure-cloud-service-classic"></a>Получение счетчиков производительности для облачной службы Azure (классическая модель)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

Счетчики производительности предоставляют возможность отслеживать работу приложения и узла. Windows Server предоставляет множество различных счетчиков производительности, связанных с программным обеспечением, приложениями, ОС и т. д. Собирая и отправляя данные счетчика производительности в Azure, вы сможете анализировать их и принимать более взвешенные решения. 

## <a name="discover-available-counters"></a>Поиск доступных счетчиков

Счетчик производительности состоит из двух частей: имени набора (также известного как категория) и одного или нескольких счетчиков. Вы можете получить список доступных счетчиков производительности с помощью PowerShell.

```powershell
Get-Counter -ListSet * | Select-Object CounterSetName, Paths | Sort-Object CounterSetName

CounterSetName                                  Paths
--------------                                  -----
.NET CLR Data                                   {\.NET CLR Data(*)\SqlClient...
.NET CLR Exceptions                             {\.NET CLR Exceptions(*)\# o...
.NET CLR Interop                                {\.NET CLR Interop(*)\# of C...
.NET CLR Jit                                    {\.NET CLR Jit(*)\# of Metho...
.NET Data Provider for Oracle                   {\.NET Data Provider for Ora...
.NET Data Provider for SqlServer                {\.NET Data Provider for Sql...
.NET Memory Cache 4.0                           {\.NET Memory Cache 4.0(*)\C...
AppV Client Streamed Data Percentage            {\AppV Client Streamed Data ...
ASP.NET                                         {\ASP.NET\Application Restar...
ASP.NET Apps v4.0.30319                         {\ASP.NET Apps v4.0.30319(*)...
ASP.NET State Service                           {\ASP.NET State Service\Stat...
ASP.NET v2.0.50727                              {\ASP.NET v2.0.50727\Applica...
ASP.NET v4.0.30319                              {\ASP.NET v4.0.30319\Applica...
Authorization Manager Applications              {\Authorization Manager Appl...

#... results cut to save space ...
```

Свойство `CounterSetName` представляет собой набор (категорию) и указывает на то, с чем связан счетчик производительности. Свойство `Paths` представляет собой коллекцию счетчиков для набора. Дополнительные сведения о наборе счетчиков содержатся в свойстве `Description`.

Все счетчики для набора можно найти в коллекции `Paths` с помощью значения `CounterSetName`. Каждый элемент пути является счетчиком. Его можно запросить. Например, чтобы получить доступные счетчики, связанные с набором `Processor`, раскройте коллекцию `Paths`.

```powershell
Get-Counter -ListSet * | Where-Object CounterSetName -eq "Processor" | Select -ExpandProperty Paths

\Processor(*)\% Processor Time
\Processor(*)\% User Time
\Processor(*)\% Privileged Time
\Processor(*)\Interrupts/sec
\Processor(*)\% DPC Time
\Processor(*)\% Interrupt Time
\Processor(*)\DPCs Queued/sec
\Processor(*)\DPC Rate
\Processor(*)\% Idle Time
\Processor(*)\% C1 Time
\Processor(*)\% C2 Time
\Processor(*)\% C3 Time
\Processor(*)\C1 Transitions/sec
\Processor(*)\C2 Transitions/sec
\Processor(*)\C3 Transitions/sec
```

Эти пути отдельных счетчиков можно добавить на платформу диагностики, используемую облачной службой. Дополнительные сведения о создании пути счетчика производительности см. в статье [Specifying a Counter Path](/windows/win32/perfctrs/specifying-a-counter-path) (Указание пути счетчика).

## <a name="collect-a-performance-counter"></a>Сбор данных счетчика производительности

Счетчик производительности можно добавить в облачную службу для диагностики Azure или Application Insights.

### <a name="application-insights"></a>Application Insights

Azure Application Insights для облачных служб позволяет указать счетчики производительности для сбора данных. После того как вы [добавите Application Insights в свой проект](../azure-monitor/app/cloudservices.md#sdk), в проект Visual Studio добавляется файл конфигурации с именем **ApplicationInsights.config**. Он определяет, какой тип данных Application Insights собирает и отправляет в Azure.

Откройте файл **ApplicationInsights.config** и найдите элемент **ApplicationInsights** > **TelemetryModules**. Каждый дочерний элемент `<Add>` определяет тип телеметрии, собираемый вместе с его конфигурацией. Тип модуля телеметрического счетчика производительности следующий: `Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector`. Если этот элемент уже определен, не добавляйте его снова. Каждый счетчик производительности для сбора определяется в узле с именем `<Counters>`. Ниже приведен пример сбора данных счетчика производительности диска.

```xml
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">

  <TelemetryModules>

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <Add PerformanceCounter="\LogicalDisk(C:)\Disk Write Bytes/sec" ReportAs="Disk write (C:)" />
        <Add PerformanceCounter="\LogicalDisk(C:)\Disk Read Bytes/sec" ReportAs="Disk read (C:)" />
      </Counters>
    </Add>

  </TelemetryModules>

<!-- ... cut to save space ... -->
```

Каждый счетчик производительности представлен как элемент `<Add>` в разделе `<Counters>`. Атрибут `PerformanceCounter` определяет счетчик производительности для сбора данных. Атрибут `ReportAs` — это заголовок, отображаемый на портале Azure для счетчика производительности. Все счетчики производительности, данные которых вы собираете, помещаются в **настраиваемую** категорию на портале. Вы не можете установить интервал сбора и отправки в Azure данных счетчиков производительности, как это возможно в системе диагностики Azure. С Application Insights данные счетчиков производительности собираются и отправляются каждую минуту. 

Application Insights автоматически собирает данные следующих счетчиков производительности:

* \Process (?? APP_WIN32_PROC??) \% Процессорное время
* \Память\доступные байты
* \.NET CLR Exceptions(??APP_CLR_PROC??)\# Исключений в секунду
* \Process(??APP_WIN32_PROC??)\Байт исключительного пользования
* \Process(??APP_WIN32_PROC??)\I/O — обмен данными, байт в секунду
* \Процессор (_общий объем ресурсов)\% загруженности процессора

Дополнительные сведения см. в статье [Системные счетчики производительности в Application Insights](../azure-monitor/app/performance-counters.md) и в разделе [Счетчики производительности](../azure-monitor/app/cloudservices.md#performance-counters).

### <a name="azure-diagnostics"></a>Система диагностики Azure

> [!IMPORTANT]
> Хотя все эти данные объединяются в учетную запись хранения, портал **не предоставляет собственный** способ построения диаграммы данных. Настоятельно рекомендуем интегрировать в приложение другую службу диагностики, например Application Insights.

Расширение системы диагностики Azure для облачных служб позволяет указать счетчики производительности для сбора данных. Сведения об установке диагностики Azure см. в разделе [Настройка расширения системы диагностики](cloud-services-how-to-monitor.md#setup-diagnostics-extension).

Счетчики производительности, данные которых вы хотите собрать, определены в файле **diagnostics.wadcfgx**. Откройте этот файл (он определяется для каждой роли) в Visual Studio и найдите элемент **DiagnosticsConfiguration**  >  **PublicConfig**  >  **WadCfg**  >  **DiagnosticMonitorConfiguration**  >  **PerformanceCounters** . Добавьте новый дочерний элемент **PerformanceCounterConfiguration**. Этот элемент имеет два атрибута: `counterSpecifier` и `sampleRate`. Атрибут `counterSpecifier` определяет набор счетчиков производительности системы (описанный в предыдущем разделе) для сбора данных. Значение `sampleRate` указывает, как часто выполняется опрос. В целом все счетчики производительности передаются в Azure в соответствии со значением атрибута `scheduledTransferPeriod` родительского элемента `PerformanceCounters`.

Дополнительные сведения об элементе схемы `PerformanceCounters` см. в разделе [Схема Диагностики Azure](../azure-monitor/agents/diagnostics-extension-schema-windows.md#performancecounters-element).

Период, определенный атрибутом `sampleRate`, использует тип данных продолжительности XML, чтобы указать, как часто проверяется счетчик производительности. В приведенном ниже примере скорость равна `PT3M`, что означает `[P]eriod[T]ime[3][M]inutes`: каждые три минуты.

Дополнительные сведения об определении `sampleRate` и `scheduledTransferPeriod` см. в разделе **Тип данных о длительности** в руководстве [XSD Date and Time Data Types](https://www.w3schools.com/XML/schema_dtypes_date.asp) (Типы даты и времени XSD).

```xml
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <PublicConfig>
    <WadCfg>
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">

        <!-- ... cut to save space ... -->

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />

          <!-- This is a new perf counter which will track the C: disk read activity in bytes per second, every minute -->
          <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(C:)\Disk Read Bytes/sec" sampleRate="PT1M" />

        </PerformanceCounters>
      </DiagnosticMonitorConfiguration>
    </WadCfg>
    
    <!-- ... cut to save space ... -->

  </PublicConfig>
</DiagnosticsConfiguration>
```

## <a name="create-a-new-perf-counter"></a>Создание счетчика производительности

С помощью вашего кода можно создать и использовать новый счетчик производительности. Уровень вашего кода, создающего счетчик производительности, должен быть повышен, иначе он не будет работать. Код запуска `OnStart` облачной службы может создать счетчик производительности, при этом необходимо, чтобы вы выполняли роль с повышенными правами. Или же вы можете создать задачу запуска, которая работает на повышенном уровне. Она создаст счетчик производительности. Дополнительные сведения см. в статье [Как настроить и выполнить задачи запуска для облачной службы](cloud-services-startup-tasks.md).

Чтобы настроить роль для повышения уровня, добавьте элемент `<Runtime>` в файл формата [CSDEF](cloud-services-model-and-package.md#servicedefinitioncsdef).

```xml
<ServiceDefinition name="CloudServiceLoadTesting" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRoleWithSBQueue1" vmsize="Large">

    <!-- ... cut to save space ... -->

    <Runtime executionContext="elevated">
      
    </Runtime>

    <!-- ... cut to save space ... -->

  </WorkerRole>
</ServiceDefinition>
```

Вы можете создать и зарегистрировать счетчик производительности с несколькими строками кода. Используйте перегрузку метода `System.Diagnostics.PerformanceCounterCategory.Create`, которая создает категорию и счетчик. Следующий код сначала проверяет, существует ли категория (если она отсутствует, он создает категорию и счетчик).

```csharp
using System.Diagnostics;
using Microsoft.WindowsAzure;
using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRoleWithSBQueue1
{
    public class WorkerRole : RoleEntryPoint
    {
        // Perf counter variable representing times service was used.
        private PerformanceCounter counterServiceUsed;

        public override bool OnStart()
        {
            // ... Other startup code here ...

            // Define the category and counter names.
            string perfCounterCatName = "MyService";
            string perfCounterName = "Times Used";

            // Create the counter if needed. Our counter category only has a single counter.
            // Both the category and counter are created in the same method call.
            if (!PerformanceCounterCategory.Exists(perfCounterCatName))
            {
                PerformanceCounterCategory.Create(perfCounterCatName, "Collects information about the cloud service.", 
                                                  PerformanceCounterCategoryType.SingleInstance, 
                                                  perfCounterName, "How many times the cloud service was used.");
            }

            // Get reference to our counter
            counterServiceUsed = new PerformanceCounter(perfCounterCatName, perfCounterName);
            counterServiceUsed.ReadOnly = false;
            
            return base.OnStart();
        }

        // ... cut class code to save space
    }
}
```

Если необходимо использовать счетчик, вызовите метод `Increment` или `IncrementBy`.

```csharp
// Increase the counter by 1
counterServiceUsed.Increment();
```

Теперь, когда приложение использует ваш счетчик, необходимо настроить службу диагностики Azure или Application Insights для отслеживания счетчика.


### <a name="application-insights"></a>Application Insights

Как указывалось ранее, счетчики производительности для Application Insights определены в файле **ApplicationInsights.config**. Откройте файл **ApplicationInsights.config** и найдите элемент **ApplicationInsights** > **TelemetryModules** > **Add** > **Counters**. Создайте дочерний элемент `<Add>` и укажите для атрибута `PerformanceCounter` категорию и имя созданного в коде счетчика производительности. Для атрибута `ReportAs` задайте простое имя, которое вы хотите увидеть на портале.

```xml
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">

  <TelemetryModules>

    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
      <Counters>
        <!-- ... cut other perf counters to save space ... -->

        <!-- This new perf counter matches the [category name]\[counter name] defined in your code -->
        <Add PerformanceCounter="\MyService\Times Used" ReportAs="Service used counter" />
      </Counters>
    </Add>

  </TelemetryModules>

<!-- ... cut to save space ... -->
```

### <a name="azure-diagnostics"></a>Система диагностики Azure

Как указывалось ранее, счетчики производительности, данные которых вы хотите собрать, определены в файле **diagnostics.wadcfgx**. Откройте этот файл (он определяется для каждой роли) в Visual Studio и найдите элемент **DiagnosticsConfiguration**  >  **PublicConfig**  >  **WadCfg**  >  **DiagnosticMonitorConfiguration**  >  **PerformanceCounters** . Добавьте новый дочерний элемент **PerformanceCounterConfiguration**. Укажите для атрибута `counterSpecifier` категорию и имя созданного в коде счетчика производительности. 

```xml
<?xml version="1.0" encoding="utf-8"?>
<DiagnosticsConfiguration  xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <PublicConfig>
    <WadCfg>
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">

        <!-- ... cut to save space ... -->

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <!-- ... cut other perf counters to save space ... -->
          
          <!-- This new perf counter matches the [category name]\[counter name] defined in your code -->
          <PerformanceCounterConfiguration counterSpecifier="\MyService\Times Used" sampleRate="PT1M" />

        </PerformanceCounters>
      </DiagnosticMonitorConfiguration>
    </WadCfg>
    
    <!-- ... cut to save space ... -->

  </PublicConfig>
</DiagnosticsConfiguration>
```

## <a name="more-information"></a>Дополнительные сведения

- [Application Insights для облачных служб Azure](../azure-monitor/app/cloudservices.md#performance-counters)
- [Системные счетчики производительности в Application Insights](../azure-monitor/app/performance-counters.md)
- [Specifying a Counter Path](/windows/win32/perfctrs/specifying-a-counter-path) (Указание пути счетчика)
- [Схема Диагностики Azure — элементы PerformanceCounters](../azure-monitor/agents/diagnostics-extension-schema-windows.md#performancecounters-element)
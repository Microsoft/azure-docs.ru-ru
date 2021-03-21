---
title: Отправка система диагностики Azure данных в Application Insights
description: Обновление открытой конфигурации диагностики Azure для отправки данных в Application Insights.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/19/2016
ms.openlocfilehash: b9e9d6b1b5939804b24fd523bf8b7444ed41178f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708600"
---
# <a name="send-cloud-service-virtual-machine-or-service-fabric-diagnostic-data-to-application-insights"></a>Отправка в Application Insights диагностических данных облачной службы, виртуальной машины или Service Fabric
Облачные службы, виртуальные машины, масштабируемые наборы виртуальных машин и Service Fabric используют расширение системы диагностики Azure для сбора данных.  Система диагностики Azure отправляет данные в таблицы службы хранилища Azure.  Тем не менее эти данные можно также полностью или частично передавать в другие расположения, используя расширение системы диагностики Azure 1.5 или более поздней версии.

В этой статье описывается, как отправлять данные из расширения системы диагностики Azure в Application Insights.

## <a name="diagnostics-configuration-explained"></a>Описание конфигурации системы диагностики
Расширение системы диагностики Azure 1.5 вводит понятие приемников — дополнительных расположений, в которые можно отправлять диагностические данные.

Ниже приведен пример конфигурации приемника для Application Insights.

```XML
<SinksConfig>
    <Sink name="ApplicationInsights">
      <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>
      <Channels>
        <Channel logLevel="Error" name="MyTopDiagData"  />
        <Channel logLevel="Verbose" name="MyLogData"  />
      </Channels>
    </Sink>
</SinksConfig>
```
```JSON
"SinksConfig": {
    "Sink": [
        {
            "name": "ApplicationInsights",
            "ApplicationInsights": "{Insert InstrumentationKey}",
            "Channels": {
                "Channel": [
                    {
                        "logLevel": "Error",
                        "name": "MyTopDiagData"
                    },
                    {
                        "logLevel": "Error",
                        "name": "MyLogData"
                    }
                ]
            }
        }
    ]
}
```
- Атрибут **Sink** *name* — это строковое значение, однозначно определяющее приемник.

- Элемент **ApplicationInsights** указывает ключ инструментирования ресурса Application Insights, в который отправляется Диагностика Azure.
    - Если ресурс Application Insights еще не существует, см. статью [Создание нового ресурса Application Insights](../app/create-new-resource.md), где содержатся дополнительные сведения о создании ресурса и получении ключа инструментирования.
    - При разработке облачной службы с использованием пакета SDK для Azure 2.8 и более поздних версий этот ключ инструментирования заполняется автоматически. При упаковке проекта облачной службы это значение задается на основе параметра конфигурации службы **APPINSIGHTS_INSTRUMENTATIONKEY**. См. статью [Application Insights для облачных служб Azure](../app/cloudservices.md)

- Элемент **Channels** содержит один или несколько элементов **Channel**.
    - Атрибут *name* однозначно ссылается на этот канал.
    - Атрибут *Loglevel* позволяет указать уровень ведения журнала для канала. Доступны следующие уровни ведения журнала (от наиболее к наименее информативным):
        - Подробный
        - Сведения
        - Предупреждение
        - Ошибка
        - Критически важно

Канал действует как фильтр и позволяет выбрать конкретные уровни ведения журнала для отправки в приемник. Например, можно собирать подробные журналы и отправлять их в хранилище, а в приемник отправлять только журнал ошибок.

Эта взаимосвязь показана на следующем рисунке.

![Открытая конфигурация диагностики](media/diagnostics-extension-to-application-insights/AzDiag_Channels_App_Insights.png)

На следующем рисунке показаны значения конфигурации и как они работают. В конфигурацию можно включить несколько приемников на разных уровнях иерархии. Приемник, указанный на верхнем уровне, действует как глобальный параметр, а приемник, указанный на уровне отдельного элемента, действует как переопределение для этого глобального параметра.

![Конфигурация приемников системы диагностики с Application Insights](media/diagnostics-extension-to-application-insights/Azure_Diagnostics_Sinks.png)

## <a name="complete-sink-configuration-example"></a>Полный пример конфигурации приемника
Ниже приведен полный пример файла общедоступной конфигурации, которая:
1. Отправляет все ошибки в службу Application Insights (указанную в узле **DiagnosticMonitorConfiguration**).
2. Отправляет подробные журналы приложений (указанные в узле **Logs**).

```XML
<WadCfg>
  <DiagnosticMonitorConfiguration overallQuotaInMB="4096"
       sinks="ApplicationInsights.MyTopDiagData"> <!-- All info below sent to this channel -->
    <DiagnosticInfrastructureLogs />
    <PerformanceCounters>
      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
      <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
    </PerformanceCounters>
    <WindowsEventLog scheduledTransferPeriod="PT1M">
      <DataSource name="Application!*" />
    </WindowsEventLog>
    <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Verbose"
            sinks="ApplicationInsights.MyLogData"/> <!-- This specific info sent to this channel -->
  </DiagnosticMonitorConfiguration>

<SinksConfig>
    <Sink name="ApplicationInsights">
      <ApplicationInsights>{Insert InstrumentationKey}</ApplicationInsights>
      <Channels>
        <Channel logLevel="Error" name="MyTopDiagData"  />
        <Channel logLevel="Verbose" name="MyLogData"  />
      </Channels>
    </Sink>
  </SinksConfig>
</WadCfg>
```
```JSON
"WadCfg": {
    "DiagnosticMonitorConfiguration": {
        "overallQuotaInMB": 4096,
        "sinks": "ApplicationInsights.MyTopDiagData", "_comment": "All info below sent to this channel",
        "DiagnosticInfrastructureLogs": {
        },
        "PerformanceCounters": {
            "PerformanceCounterConfiguration": [
                {
                    "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                    "sampleRate": "PT3M"
                },
                {
                    "counterSpecifier": "\\Memory\\Available MBytes",
                    "sampleRate": "PT3M"
                }
            ]
        },
        "WindowsEventLog": {
            "scheduledTransferPeriod": "PT1M",
            "DataSource": [
                {
                    "name": "Application!*"
                }
            ]
        },
        "Logs": {
            "scheduledTransferPeriod": "PT1M",
            "scheduledTransferLogLevelFilter": "Verbose",
            "sinks": "ApplicationInsights.MyLogData", "_comment": "This specific info sent to this channel"
        }
    },
    "SinksConfig": {
        "Sink": [
            {
                "name": "ApplicationInsights",
                "ApplicationInsights": "{Insert InstrumentationKey}",
                "Channels": {
                    "Channel": [
                        {
                            "logLevel": "Error",
                            "name": "MyTopDiagData"
                        },
                        {
                            "logLevel": "Verbose",
                            "name": "MyLogData"
                        }
                    ]
                }
            }
        ]
    }
}
```
В предыдущей конфигурации приведенные ниже строки имеют следующий смысл.

### <a name="send-all-the-data-that-is-being-collected-by-azure-diagnostics"></a>Отправка всех данных, собираемых системой диагностики Azure

```xml
<DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="ApplicationInsights">
```

```json
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights",
}
```

### <a name="send-only-error-logs-to-the-application-insights-sink"></a>Отправка в приемник Application Insights только журналов ошибок

```xml
<DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="ApplicationInsights.MyTopDiagdata">
```

```json
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights.MyTopDiagData",
}
```

### <a name="send-verbose-application-logs-to-application-insights"></a>Отправка подробных журналов приложений в Application Insights

```XML
<Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Verbose" sinks="ApplicationInsights.MyLogData"/>
```
```JSON
"DiagnosticMonitorConfiguration": {
    "overallQuotaInMB": 4096,
    "sinks": "ApplicationInsights.MyLogData",
}
```

## <a name="limitations"></a>Ограничения

- **Каналы могут вести журнал типов, но не счетчиков производительности.** Если указать канал с элементом счетчика производительности, он игнорируется.
- **Уровень журнала для канала не может превышать уровень журнала, данные которого собираются с помощью функции диагностики Azure.** Например, нельзя собирать данные об ошибках в журнале приложений в элементе Logs и пытаться отправлять подробные журналы в приемник Application Insight. Атрибут *ScheduledTransferLogLevelFilter* должен всегда собирать равное или большее число журналов, чем число журналов, которые вы пытаетесь отправить в приемник.
- **В Application Insights нельзя отправлять собранные расширением системы диагностики Azure данные больших двоичных объектов.** Например, все, что указано в узле *Directories* . Что касается аварийных дампов, фактический аварийный дамп отправляется в хранилище BLOB-объектов, а в Application Insights отправляется только уведомление о том, что аварийный дамп был создан.

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте, как [просматривать данные диагностики Azure](../app/cloudservices.md) в Application Insights.
* Используйте [PowerShell](../../cloud-services/cloud-services-diagnostics-powershell.md), чтобы включить расширение диагностики Azure для вашего приложения.
* Используйте [Visual Studio](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines) , чтобы включить расширение диагностики Azure для вашего приложения.


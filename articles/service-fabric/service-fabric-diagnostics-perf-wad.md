---
title: Мониторинг производительности с помощью система диагностики Azure Windows
description: Использование Диагностики Azure для Windows для сбора данных счетчиков производительности для кластеров Azure Service Fabric.
ms.topic: conceptual
ms.date: 11/21/2018
ms.openlocfilehash: 6803494d29bf97336e30128f9f5ad20ec73ce202
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105627428"
---
# <a name="performance-monitoring-with-the-windows-azure-diagnostics-extension"></a>Мониторинг производительности с помощью расширения для Диагностики Azure для Windows

В этом документе рассматриваются шаги, необходимые для настройки сбора данных счетчиков производительности через расширение Диагностики Azure для Windows для кластеров Windows. Для кластеров Linux настройте [агент Log Analytics](service-fabric-diagnostics-oms-agent.md), чтобы собрать данные счетчиков производительности для узлов. 

 > [!NOTE]
> Чтобы выполнение этих шагов было эффективным, на кластере необходимо развернуть расширение системы диагностики Microsoft Azure. Если оно не настроено, перейдите к разделу [Агрегирование и сбор событий с помощью Диагностики Azure для Windows](service-fabric-diagnostics-event-aggregation-wad.md).  


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="collect-performance-counters-via-the-wadcfg"></a>Сбор данных счетчиков производительности с помощью раздела WadCfg

Для сбора данных счетчиков производительности через систему диагностики Microsoft Azure необходимо изменить конфигурацию соответствующим образом в шаблоне Resource Manager кластера. Выполните эти шаги, чтобы добавить счетчик производительности, с которого нужно собирать данные, в шаблон и запустить обновление ресурса Resource Manager.

1. Найдите конфигурацию системы диагностики Microsoft Azure в шаблоне кластера, а именно раздел `WadCfg`. Вы добавите счетчики производительности для сбора в разделе `DiagnosticMonitorConfiguration`.

2. Настройте конфигурацию для сбора данных счетчиков производительности, добавив следующий раздел в `DiagnosticMonitorConfiguration`. 

    ```json
    "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": []
    }
    ```

    `scheduledTransferPeriod` определяет частоту передачи собираемых значений счетчиков в таблицу хранилища Azure и в любой настроенный приемник. 

3. Добавьте счетчики производительности, данные которых необходимо собрать, в параметр `PerformanceCounterConfiguration`, объявленный на предыдущем шаге. Каждый счетчик, данные которого необходимо собирать, определен с помощью параметров `counterSpecifier`, `sampleRate`, `unit`, `annotation` и любых соответствующих приемников `sinks`.

Ниже приведен пример конфигурации со счетчиком для *общей загруженности процессора* (периода, в течение которого ЦП был задействован для операций обработки) и *вызовов методов субъекта Service Fabric в секунду* — одним из настраиваемых счетчиков производительности Service Fabric. Полный список настраиваемых счетчиков производительности Service Fabric см. в разделах о [счетчиках производительности Reliable Actors](service-fabric-reliable-actors-diagnostics.md#list-of-events-and-performance-counters) и [счетчиках производительности Reliable Services](service-fabric-reliable-serviceremoting-diagnostics.md#list-of-performance-counters).

 ```json
 "WadCfg": {
         "DiagnosticMonitorConfiguration": {
           "overallQuotaInMB": "50000",
           "EtwProviders": {
             "EtwEventSourceProviderConfiguration": [
               {
                 "provider": "Microsoft-ServiceFabric-Actors",
                 "scheduledTransferKeywordFilter": "1",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricReliableActorEventTable"
                 }
               },
               {
                 "provider": "Microsoft-ServiceFabric-Services",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricReliableServiceEventTable"
                 }
               }
             ],
             "EtwManifestProviderConfiguration": [
               {
                 "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                 "scheduledTransferLogLevelFilter": "Information",
                 "scheduledTransferKeywordFilter": "4611686018427387904",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricSystemEventTable"
                 }
               }
             ]
           },
           "PerformanceCounters": {
                 "scheduledTransferPeriod": "PT1M",
                 "PerformanceCounterConfiguration": [
                     {
                         "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                         "sampleRate": "PT1M",
                         "unit": "Percent",
                         "annotation": [
                         ],
                         "sinks": ""
                     },
                     {
                         "counterSpecifier": "\\Service Fabric Actor Method(*)\\Invocations/Sec",
                         "sampleRate": "PT1M",
                     }
                 ]
             }
         }
       },
  ```

 Частоту выборки счетчика можно изменить согласно вашим потребностям. Допустимый формат — `PT<time><unit>`, поэтому если необходимо собирать данные счетчика каждую секунду, то следует задать значение `"sampleRate": "PT15S"`.

 Кроме того, используя в шаблоне ARM переменные, можно собирать массив счетчиков производительности. Такая возможность может пригодиться при сборе счетчиков производительности для каждого процесса. В примере ниже мы с помощью переменных собираем данные о процессорном времени и времени работы сборщика мусора для каждого процесса, а затем — два счетчика производительности на самих узлах. 

 ```json
"variables": {
  "copy": [
      {
        "name": "processorTimeCounters",
        "count": "[length(parameters('monitoredProcesses'))]",
        "input": {
          "counterSpecifier": "\\Process([parameters('monitoredProcesses')[copyIndex('processorTimeCounters')]])\\% Processor Time",
          "sampleRate": "PT1M",
          "unit": "Percent",
          "sinks": "applicationInsights",
          "annotation": [
            {
              "displayName": "[concat(parameters('monitoredProcesses')[copyIndex('processorTimeCounters')],' Processor Time')]",
              "locale": "en-us"
            }
          ]
        }
      },
      {
        "name": "gcTimeCounters",
        "count": "[length(parameters('monitoredProcesses'))]",
        "input": {
          "counterSpecifier": "\\.NET CLR Memory([parameters('monitoredProcesses')[copyIndex('gcTimeCounters')]])\\% Time in GC",
          "sampleRate": "PT1M",
          "unit": "Percent",
          "sinks": "applicationInsights",
          "annotation": [
            {
              "displayName": "[concat(parameters('monitoredProcesses')[copyIndex('gcTimeCounters')],' Time in GC')]",
              "locale": "en-us"
            }
          ]
        }
      }
    ],
    "machineCounters": [
      {
        "counterSpecifier": "\\Memory\\Available Bytes",
        "sampleRate": "PT1M",
        "unit": "KB",
        "sinks": "applicationInsights",
        "annotation": [
          {
            "displayName": "Memory Available Kb",
            "locale": "en-us"
          }
        ]
      },
      {
        "counterSpecifier": "\\Memory\\% Committed Bytes In Use",
        "sampleRate": "PT15S",
        "unit": "percent",
        "annotation": [
          {
            "displayName": "Memory usage",
            "locale": "en-us"
          }
        ]
      }
    ]
  }
....
"WadCfg": {
    "DiagnosticMonitorConfiguration": {
      "overallQuotaInMB": "50000",
      "Metrics": {
        "metricAggregation": [
          {
            "scheduledTransferPeriod": "PT1M"
          }
        ],
        "resourceId": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', variables('vmNodeTypeApp2Name'))]"
      },
      "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": "[concat(variables ('processorTimeCounters'), variables('gcTimeCounters'),  variables('machineCounters'))]"
      },
....
```

1. Когда вы добавите соответствующие счетчики производительности, данные которых необходимо собрать, потребуется обновить ресурс кластера, чтобы эти изменения отразились в работающем кластере. Сохраните измененный файл `template.json` и откройте PowerShell. Обновление кластера можно выполнить с помощью `New-AzResourceGroupDeployment`. Для вызова необходимо указать имя группы ресурсов, обновленный файл шаблона и файл параметров. Кроме того, вызов отправляет запрос в Resource Manager на внесение соответствующих изменений в обновленные ресурсы. После входа в учетную запись и правильную подписку используйте следующую команду для выполнения обновления:

    ```sh
    New-AzResourceGroupDeployment -ResourceGroupName <ResourceGroup> -TemplateFile <PathToTemplateFile> -TemplateParameterFile <PathToParametersFile> -Verbose
    ```

1. По завершении обновления (оно занимает 15–45 минут в зависимости от ряда факторов, в частности от размера группы ресурсов и того, первый ли раз выполняется развертывание) система диагностики Microsoft Azure должна собирать данные счетчиков производительности и отправлять их в таблицу с именем WADPerformanceCountersTable в учетной записи хранения, связанной с вашим кластером. Просмотрите счетчики производительности в Application Insights, [добавив приемник AI в шаблон диспетчер ресурсов](service-fabric-diagnostics-event-aggregation-wad.md#add-the-application-insights-sink-to-the-resource-manager-template).

## <a name="next-steps"></a>Дальнейшие шаги
* Собирайте данные дополнительных счетчиков производительности для кластера. Список счетчиков, данные которых следует собирать, см. в статье [Метрики производительности](service-fabric-diagnostics-event-generation-perf.md).
* [Используйте мониторинг и систему диагностики с виртуальной машиной Windows и шаблонами Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md), чтобы внести изменения в раздел `WadCfg`, включая настройку дополнительных учетных записей хранения для отправки данных диагностики.
* Посетите [Построитель WadCfg](https://azure.github.io/azure-diagnostics-tools/config-builder/) , чтобы создать шаблон с нуля и убедиться в правильности синтаксиса. ( https://azure.github.io/azure-diagnostics-tools/config-builder/) чтобы создать шаблон с нуля и убедиться в правильности синтаксиса.

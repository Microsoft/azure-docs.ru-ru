---
title: Агрегирование событий Service Fabric Azure с помощью EventFlow
description: Ознакомьтесь со сведениями об агрегировании и сборе событий с использованием EventFlow для мониторинга и диагностики кластеров Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 2/25/2019
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: a80eea5a50aa7b1e441049eeb2cae381994cd3ec
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96006342"
---
# <a name="event-aggregation-and-collection-using-eventflow"></a>Агрегирование и сбор событий с помощью EventFlow

[EventFlow службы диагностики Microsoft](https://github.com/Azure/diagnostics-eventflow) позволяет направлять события от узла к одному или нескольким целевым объектам для мониторинга. Так как это решение включается в проект службы как пакет NuGet, код и конфигурация EventFlow перемещаются вместе со службой, устраняя необходимость отдельной настройки каждого узла для системы диагностики Azure, которую мы обсуждали выше. EventFlow выполняется внутри процесса службы и напрямую подключается к настроенным потокам вывода. Прямое подключение позволяет использовать EventFlow для служб, развернутых в Azure, с помощью контейнера или в локальной среде. Соблюдайте осторожность при выполнении EventFlow в сценариях с высокой плотностью, например в контейнере, так как каждый конвейер EventFlow создает внешнее соединение. Если вы разместите несколько процессов, то получите несколько исходящих подключений. Об этом можно не беспокоиться при использовании приложений Service Fabric, так как все реплики `ServiceType` выполняются в одном процессе, что ограничивает число исходящих подключений. EventFlow также поддерживает фильтрацию событий, то есть позволяет отправлять только события, соответствующие указанному фильтру.

## <a name="set-up-eventflow"></a>Настройка EventFlow

Двоичные файлы EventFlow предоставляются как набор пакетов NuGet. Чтобы добавить библиотеку EventFlow в проект службы Service Fabric, щелкните его правой кнопкой мыши в обозревателе решений и выберите "Управление пакетами NuGet". Перейдите на вкладку "Обзор" и найдите `Diagnostics.EventFlow`.

![Пакеты NuGet EventFlow в диспетчере пакетов NuGet Visual Studio](./media/service-fabric-diagnostics-event-aggregation-eventflow/eventflow-nuget.png)

Появится список различных пакетов, которые помечены как "Входные данные" и "Выходные данные". EventFlow поддерживает различные регистраторы и анализаторы журналов. Служба, в которой размещается EventFlow, должна содержать соответствующие пакеты в зависимости от источника и назначения журналов приложений. Помимо основного пакета ServiceFabric, также требуется настроить по крайней мере один ввод и один вывод. Например, можно добавить следующие пакеты для отправки событий EventSource в Application Insights:

* `Microsoft.Diagnostics.EventFlow.Inputs.EventSource` сбор данных из класса EventSource службы и из стандартных классов EventSource, таких как *Microsoft-ServiceFabric-Services* и *Microsoft-ServiceFabric-* actors.
* `Microsoft.Diagnostics.EventFlow.Outputs.ApplicationInsights` (мы собираемся отправлять журналы в ресурс Azure Application Insights)
* `Microsoft.Diagnostics.EventFlow.ServiceFabric` (позволяет инициализировать конвейер EventFlow из конфигурации службы Service Fabric и сообщать о всех проблемах отправки диагностических данных в виде отчетов о работоспособности Service Fabric)

>[!NOTE]
>Для пакета `Microsoft.Diagnostics.EventFlow.Inputs.EventSource` требуется, чтобы в проекте службы использовалась платформа .NET Framework 4.6 или более поздняя версия. Обязательно задайте соответствующую целевую платформу в свойствах проекта, прежде чем устанавливать этот пакет.

После установки всех пакетов следующим шагом является настройка и включение EventFlow в службе.

## <a name="configure-and-enable-log-collection"></a>Настройка и включение сбора журналов
Конвейер EventFlow, отвечающий за отправку журналов, создается на основе спецификации, хранящейся в файле конфигурации. Пакет `Microsoft.Diagnostics.EventFlow.ServiceFabric` устанавливает начальный файл конфигурации EventFlow в паку решения `PackageRoot\Config` с именем `eventFlowConfig.json`. Этот файл конфигурации нужно изменить, чтобы собирать данные из класса `EventSource` службы по умолчанию, а также другие входные данные, которые вы хотите настроить, и отправлять их в соответствующее место.

>[!NOTE]
>Если используется файл проекта в формате Visual Studio 2017, файл `eventFlowConfig.json` не будет добавлен автоматически. Чтобы это исправить, создайте файл в папке `Config` и задайте действие сборки для `Copy if newer`. 

Вот пример файла *eventFlowConfig.json* на основе упомянутых выше пакетов NuGet:
```json
{
  "inputs": [
    {
      "type": "EventSource",
      "sources": [
        { "providerName": "Microsoft-ServiceFabric-Services" },
        { "providerName": "Microsoft-ServiceFabric-Actors" },
        // (replace the following value with your service's ServiceEventSource name)
        { "providerName": "your-service-EventSource-name" }
      ]
    }
  ],
  "filters": [
    {
      "type": "drop",
      "include": "Level == Verbose"
    }
  ],
  "outputs": [
    {
      "type": "ApplicationInsights",
      // (replace the following value with your AI resource's instrumentation key)
      "instrumentationKey": "00000000-0000-0000-0000-000000000000"
    }
  ],
  "schemaVersion": "2016-08-11"
}
```

Имя ServiceEventSource службы — это значение свойства Name в `EventSourceAttribute`, которое применяется к классу ServiceEventSource. Все это указывается в файле `ServiceEventSource.cs`, который является частью кода службы. Например, в приведенном ниже фрагменте кода именем ServiceEventSource является *MyCompany Application1-Stateless1*.

```csharp
[EventSource(Name = "MyCompany-Application1-Stateless1")]
internal sealed class ServiceEventSource : EventSource
{
    // (rest of ServiceEventSource implementation)
}
```

Обратите внимание, что файл `eventFlowConfig.json` входит в пакет конфигурации службы. Изменения в этот файл могут вноситься только полными обновлениями или обновлениями конфигурации службы, которые проходят проверку работоспособности обновлений Service Fabric и автоматически откатываются в случае сбоя. Дополнительные сведения см. в разделе [Обновление приложения Service Fabric](service-fabric-application-upgrade.md).

Раздел *filters* конфигурации позволяет произвести дополнительную настройку данных, которые будут передаваться по конвейеру EventFlow на вывод, включив или исключив определенную информацию либо изменив структуру данных событий. Дополнительные сведения о фильтрации см. в разделе [Фильтры EventFlow](https://github.com/Azure/diagnostics-eventflow#filters).

Завершающим шагом является создание экземпляра конвейера EventFlow в коде запуска службы, расположенном в файле `Program.cs`.

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using Microsoft.ServiceFabric;
using Microsoft.ServiceFabric.Services.Runtime;

// **** EventFlow namespace
using Microsoft.Diagnostics.EventFlow.ServiceFabric;

namespace Stateless1
{
    internal static class Program
    {
        /// <summary>
        /// This is the entry point of the service host process.
        /// </summary>
        private static void Main()
        {
            try
            {
                // **** Instantiate log collection via EventFlow
                using (var diagnosticsPipeline = ServiceFabricDiagnosticPipelineFactory.CreatePipeline("MyApplication-MyService-DiagnosticsPipeline"))
                {

                    ServiceRuntime.RegisterServiceAsync("Stateless1Type",
                    context => new Stateless1(context)).GetAwaiter().GetResult();

                    ServiceEventSource.Current.ServiceTypeRegistered(Process.GetCurrentProcess().Id, typeof(Stateless1).Name);

                    Thread.Sleep(Timeout.Infinite);
                }
            }
            catch (Exception e)
            {
                ServiceEventSource.Current.ServiceHostInitializationFailed(e.ToString());
                throw;
            }
        }
    }
}
```

Имя, переданное в качестве параметра в метод `CreatePipeline` класса `ServiceFabricDiagnosticsPipelineFactory`, — это имя *сущности работоспособности*, представляющей конвейер EventFlow для сбора журналов. Это имя используется в том случае, если EventFlow обнаруживает ошибку и сообщает о ней через подсистему работоспособности Service Fabric.

### <a name="use-service-fabric-settings-and-application-parameters-in-eventflowconfig"></a>Использование параметров Service Fabric и параметров приложений в eventFlowConfig

EventFlow поддерживает использование параметров Service Fabric и параметров приложения для настройки параметров EventFlow. На параметры Service Fabric можно ссылаться с помощью специального синтаксиса для значений:

```json
servicefabric:/<section-name>/<setting-name>
```

`<section-name>` — это имя раздела конфигурации Service Fabric, а `<setting-name>` — это параметр конфигурации, предоставляющий значение, которое будет использоваться для настройки параметра EventFlow. Дополнительные сведения см. в статье [Поддержка параметров Service Fabric и параметров приложений](https://github.com/Azure/diagnostics-eventflow#support-for-service-fabric-settings-and-application-parameters).

## <a name="verification"></a>Проверка

Запустите службу и просмотрите окно выходных данных отладки в Visual Studio. После запуска службы вы должны увидеть подтверждение того, что она отправляет записи в настроенное расположение вывода. Перейдите в платформу анализа и визуализации событий и убедитесь в том, что журналы начали отображаться (это может занять несколько минут).

## <a name="next-steps"></a>Дальнейшие действия

* [Анализ событий и визуализация с помощью Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Анализ событий и визуализация с помощью журналов Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
* [Документация по EventFlow](https://github.com/Azure/diagnostics-eventflow)

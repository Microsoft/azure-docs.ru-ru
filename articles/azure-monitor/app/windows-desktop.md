---
title: Мониторинг использования и производительности классических приложений для Windows
description: Анализ использования и производительности классического приложения для Windows с помощью Application Insights.
ms.topic: conceptual
ms.date: 06/11/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 15331494b40021f10c162fba75abf6fe88d2d419
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100583312"
---
# <a name="monitoring-usage-and-performance-in-classic-windows-desktop-apps"></a>Мониторинг использования и производительности в классических приложениях для Windows

Все приложения, размещенные на локальном компьютере, в Azure и в других облаках могут воспользоваться преимуществами Application Insights. Все, что нужно — это [разрешить обмен данными](./ip-addresses.md) со службой Application Insights. Для мониторинга приложений универсальной платформы Windows (UWP) мы рекомендуем использовать [Центр приложений Visual Studio](../app/mobile-center-quickstart.md).

## <a name="to-send-telemetry-to-application-insights-from-a-classic-windows-application"></a>Отправка данных телеметрии в Application Insights из классического приложения для Windows
1. На [портале Azure](https://portal.azure.com)[создайте ресурс Application Insights](./create-new-resource.md). 
2. Сделайте копию ключа инструментирования.
3. В Visual Studio измените пакеты NuGet вашего проекта приложения и добавьте Microsoft.ApplicationInsights.WindowsServer. (Или выберите Microsoft. ApplicationInsights, если нужен только базовый API, без стандартных модулей сбора данных телеметрии.)
4. Задайте ключ инструментирования в коде.
   
    `TelemetryConfiguration.Active.InstrumentationKey = "`*Ваш ключ*`";`
   
    Можно также задать его в файле ApplicationInsights.config (если установлен один из пакетов стандартной телеметрии).
   
    `<InstrumentationKey>`*Ваш ключ*`</InstrumentationKey>` 
   
    Если используется файл ApplicationInsights.config, убедитесь, что его свойства в обозревателе решений имеют следующие значения: **"Действие сборки = содержимое", "Копировать в выходной каталог = копировать"**.
5. [Используйте API](./api-custom-events-metrics.md) для отправки данных телеметрии.
6. Запустите приложение и просмотрите данные телеметрии в ресурсе, созданном в портал Azure.

## <a name="example-code"></a><a name="telemetry"></a>Пример кода

```csharp
using Microsoft.ApplicationInsights;

    public partial class Form1 : Form
    {
        private TelemetryClient tc = new TelemetryClient();
        ...
        private void Form1_Load(object sender, EventArgs e)
        {
            // Alternative to setting ikey in config file:
            tc.InstrumentationKey = "key copied from portal";

            // Set session data:
            tc.Context.Session.Id = Guid.NewGuid().ToString();
            tc.Context.Device.OperatingSystem = Environment.OSVersion.ToString();

            // Log a page view:
            tc.TrackPageView("Form1");
            ...
        }

        protected override void OnFormClosing(System.Windows.Forms.FormClosingEventArgs e)
        {
            if (tc != null)
            {
                tc.Flush(); // only for desktop apps

                // Allow time for flushing:
                System.Threading.Thread.Sleep(1000);
            }
            base.OnFormClosing(e);
        }
        
        ...
```

## <a name="override-storage-of-computer-name"></a>Переопределить хранилище имени компьютера

По умолчанию этот пакет SDK будет выполнять сбор и хранение имени компьютера, порожденного системой телеметрии.

Имя компьютера используется Application Insights ценовой категории ["устаревший корпоративный" (на узел)](./pricing.md#legacy-enterprise-per-node-pricing-tier) для внутренних целей выставления счетов. По умолчанию, если для переопределения используется инициализатор телеметрии `telemetry.Context.Cloud.RoleInstance` , будет отправлено отдельное свойство, `ai.internal.nodeName` которое по-прежнему будет содержать значение имени компьютера. Это значение не будет храниться в Application Insights телеметрии, но используется для внутренних целей при приеме, чтобы обеспечить обратную совместимость с устаревшей моделью выставления счетов на основе узлов.

Если вы используете [ценовую категорию устаревшего предприятия (на узел)](./pricing.md#legacy-enterprise-per-node-pricing-tier) и просто переопределите хранилище имени компьютера, используйте инициализатор телеметрии:

**Напишите пользовательский TelemetryInitializer, как показано ниже.**

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

namespace CustomInitializer.Telemetry
{
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleInstance))
            {
                // Set custom role name here. Providing an empty string will result
                // in the computer name still be sent via this property.
                  telemetry.Context.Cloud.RoleInstance = "Custom RoleInstance";
            }
        }
    }
}
```

Создайте экземпляр инициализатора в `Program.cs` `Main()` приведенном ниже методе, указав ключ инструментирования:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;

   static void Main()
        {
            TelemetryConfiguration.Active.InstrumentationKey = "{Instrumentation-key-here}";
            TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
            //...
        }
```

## <a name="override-transmission-of-computer-name"></a>Переопределить передачу имени компьютера

Если вы не используете [ценовую категорию "устаревший корпоративный" (на уровне узла)](./pricing.md#legacy-enterprise-per-node-pricing-tier) и хотите полностью предотвратить отправку данных телеметрии, содержащих имя компьютера, необходимо использовать обработчик данных телеметрии.

### <a name="telemetry-processor"></a>Обработчик данных телеметрии

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;


namespace WindowsFormsApp2
{
    public class CustomTelemetryProcessor : ITelemetryProcessor
    {
        private readonly ITelemetryProcessor _next;

        public CustomTelemetryProcessor(ITelemetryProcessor next)
        {
            _next = next;
        }

        public void Process(ITelemetry item)
        {
            if (item != null)
            {
                item.Context.Cloud.RoleInstance = string.Empty;
            }

            _next.Process(item);
        }
    }
}
```

Создайте экземпляр обработчика данных телеметрии в `Program.cs` `Main()` приведенном ниже методе, указав ключ инструментирования:

```csharp
using Microsoft.ApplicationInsights.Extensibility;

namespace WindowsFormsApp2
{
    static class Program
    {
        static void Main()
        {
            TelemetryConfiguration.Active.InstrumentationKey = "{Instrumentation-key-here}";
            var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
            builder.Use((next) => new CustomTelemetryProcessor(next));
            builder.Build();
            //...
        }
    }
}

```

> [!NOTE]
> Хотя вы можете использовать обработчик данных телеметрии, как описано выше, даже если вы используете [ценовую категорию устаревшего предприятия (на узел)](./pricing.md#legacy-enterprise-per-node-pricing-tier), это приведет к потенциальной оплате из-за невозможности правильного различения узлов для каждого узла.

## <a name="next-steps"></a>Дальнейшие действия
* [Создание панели мониторинга](./overview-dashboard.md)
* [Поиск по журналу диагностики](./diagnostic-search.md)
* [Изучение метрик](../essentials/metrics-charts.md)
* [Написание запросов аналитики](../logs/log-query-overview.md)


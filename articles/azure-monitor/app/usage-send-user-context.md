---
title: Идентификаторы контекста пользователя для мониторинга активности — Azure Application Insights
description: Отслеживайте, как пользователи перемещаются по службе, назначив каждому из них строку уникального постоянного идентификатора в Application Insights.
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 01/03/2019
ms.reviewer: abgreg;mbullwin
ms.openlocfilehash: 021c76bcd03bbe35eabec5611fe0cc1e2c7c4427
ms.sourcegitcommit: e559daa1f7115d703bfa1b87da1cf267bf6ae9e8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/17/2021
ms.locfileid: "100583333"
---
# <a name="send-user-context-ids-to-enable-usage-experiences-in-azure-application-insights"></a>Отправка идентификаторов контекста пользователей для обеспечения оптимальной работы в Azure Application Insights

## <a name="tracking-users"></a>Отслеживание пользователей

Application Insights позволяет отслеживать пользователей с помощью набора инструментов для использования продуктов:

- [Пользователи, сеансы, события](./usage-segmentation.md)
- [Воронки](./usage-funnels.md)
- Когорты [удержания](./usage-retention.md)
- [книги](../visualize/workbooks-overview.md)

Для отслеживания действий пользователя за определенный период Application Insights требуется идентификатор для каждого пользователя или сеанса. Добавьте приведенные идентификаторы в каждое пользовательское событие или представление страницы.

- Для пользователей, воронок, инструментов хранения и когорт: включите идентификатор пользователя.
- Для сеансов: включите идентификатор сеанса.

> [!NOTE]
> В этой расширенной статье приводятся выполняемые вручную шаги для отслеживания действий пользователей с помощью Application Insights. Для многих веб-приложений **эти шаги могут не потребоваться**, так как для автоматического отслеживания действий пользователей часто бывает достаточно заданных по умолчанию серверных пакетов SDK в сочетании с [пакетом SDK для JavaScript на стороне клиента или обозревателя](./website-monitoring.md). Если вы не настроили [наблюдение на стороне клиента](./website-monitoring.md) дополнительно к серверному пакету SDK, сделайте это и проверьте, что средства анализа поведения пользователей работают нормально.

## <a name="choosing-user-ids"></a>Выбор идентификаторов пользователей

Идентификаторы пользователей должны сохраняться в пользовательских сеансах, чтобы можно было отследить поведение пользователей за период времени. Существуют различные подходы к сохранению идентификатора.

- Определение пользователя, уже имеющееся в службе.
- Если служба имеет доступ к браузеру, она может передать ему файл cookie с идентификатором. Идентификатор хранится, пока файл cookie остается в браузере пользователя.
- При необходимости можно использовать новый идентификатор для каждого сеанса, но результаты для пользователей будут ограничены. Например, вы не сможете узнать, как поведение пользователя меняется со временем.

Идентификатор должен быть идентификатором GUID или другой строкой, достаточно сложной для уникальной идентификации каждого пользователя. Например, это может быть длинное случайное число.

Если идентификатор содержит персональные идентификационные сведения о пользователе, его не следует отправлять в качестве идентификатора пользователя в Application Insights. Такой идентификатор можно отправить как [аутентифицированный идентификатор пользователя](./api-custom-events-metrics.md#authenticated-users). Однако такой идентификатор не удовлетворяет требования к идентификатору пользователя для сценариев использования.

## <a name="aspnet-apps-setting-the-user-context-in-an-itelemetryinitializer"></a>Приложения ASP.NET: установка контекста пользователя в ITelemetryInitializer

[Создайте инициализатор телеметрии](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer). Передайте идентификатор сеанса в запросе телеметрии и задайте Context.User.Id и Context.Session.Id.

В этом примере в качестве идентификатора пользователя задается идентификатор, срок действия которого истекает по завершении сеанса. По возможности используйте идентификатор пользователя, который сохраняется между сеансами.

### <a name="telemetry-initializer"></a>Инициализатор телеметрии

```csharp
using System;
using System.Web;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

namespace MvcWebRole.Telemetry
{
  /*
   * Custom TelemetryInitializer that sets the user ID.
   *
   */
  public class MyTelemetryInitializer : ITelemetryInitializer
  {
    public void Initialize(ITelemetry telemetry)
    {
        var ctx = HttpContext.Current;

        // If telemetry initializer is called as part of request execution and not from some async thread
        if (ctx != null)
        {
            var requestTelemetry = ctx.GetRequestTelemetry();
 
            // Set the user and session ids from requestTelemetry.Context.User.Id, which is populated in Application_PostAcquireRequestState in Global.asax.cs.
            if (requestTelemetry != null && !string.IsNullOrEmpty(requestTelemetry.Context.User.Id) &&
                (string.IsNullOrEmpty(telemetry.Context.User.Id) || string.IsNullOrEmpty(telemetry.Context.Session.Id)))
            {
                // Set the user id on the Application Insights telemetry item.
                telemetry.Context.User.Id = requestTelemetry.Context.User.Id;
 
                // Set the session id on the Application Insights telemetry item.
                telemetry.Context.Session.Id = requestTelemetry.Context.User.Id;
            }
        }
    }
  }
}
```

### <a name="globalasaxcs"></a>Global.asax.cs

```csharp
using System.Web;
using System.Web.Http;
using System.Web.Mvc;
using System.Web.Optimization;
using System.Web.Routing;

namespace MvcWebRole.Telemetry
{
    public class MvcApplication : HttpApplication
    {
        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            GlobalConfiguration.Configure(WebApiConfig.Register);
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            BundleConfig.RegisterBundles(BundleTable.Bundles);
        }
 
        protected void Application_PostAcquireRequestState()
        {
            var requestTelemetry = Context.GetRequestTelemetry();
 
            if (HttpContext.Current.Session != null && requestTelemetry != null && string.IsNullOrEmpty(requestTelemetry.Context.User.Id))
            {
                requestTelemetry.Context.User.Id = Session.SessionID;
            }
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы обеспечить оптимальное использование, начните отправлять [пользовательские события](./api-custom-events-metrics.md#trackevent) или [сведения о просмотрах страниц](./api-custom-events-metrics.md#page-views).
- Если вы уже сделали это, изучите инструменты использования, чтобы узнать, как пользователи используют службу.
    - [Общие сведения об использовании](usage-overview.md)
    - [Анализ пользователей, сеансов и событий в Application Insights](usage-segmentation.md)
    - [Воронки](usage-funnels.md)
    - [Сохранение](usage-retention.md)
    - [книги](../visualize/workbooks-overview.md)


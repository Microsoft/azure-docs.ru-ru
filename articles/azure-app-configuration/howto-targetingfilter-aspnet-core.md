---
title: Включение поэтапного развертывания функций для определенных аудиторий
titleSuffix: Azure App Configuration
description: Узнайте, как включить поэтапный выпуск компонентов для целевых аудиторий
ms.service: azure-app-configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.topic: conceptual
ms.date: 11/20/2020
ms.openlocfilehash: c415eaeab2edd0a1b324bba4266266201cb50cbf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96929690"
---
# <a name="enable-staged-rollout-of-features-for-targeted-audiences"></a>Включение поэтапного развертывания функций для определенных аудиторий

Флаги компонентов позволяют динамически активировать или деактивировать функциональные возможности в приложении. Фильтры компонентов определяют состояние флага компонента при каждом его вычислении. `Microsoft.FeatureManagement`Библиотека включает `TargetingFilter` , которая включает флаг компонента для указанного списка пользователей и групп или указанный процент пользователей. `TargetingFilter` «прикреплен». Это означает, что после того, как отдельный пользователь получит функцию, он продолжит видеть эту функцию во всех будущих запросах. Можно использовать `TargetingFilter` , чтобы включить функцию для конкретной учетной записи во время демонстрации, чтобы последовательно развернуть новые функции для пользователей в разных группах или "кольцах" и многое другое.

В этой статье вы узнаете, как развернуть новую функцию в веб-приложении ASP.NET Core для определенных пользователей и групп с помощью `TargetingFilter` приложения Azure.

## <a name="create-a-web-application-with-feature-flags-and-authentication"></a>Создание веб-приложения с флагами компонентов и проверкой подлинности

Чтобы развернуть компоненты на основе пользователей и групп, вам потребуется веб-приложение, позволяющее пользователям входить в систему.

1. Создайте веб-приложение для проверки подлинности в локальной базе данных с помощью следующей команды:

   ```dotnetcli
   dotnet new mvc --auth Individual -o TestFeatureFlags
   ```

1. Выполните сборку и запуск, а затем щелкните ссылку **Регистрация** в правом верхнем углу, чтобы создать новую учетную запись пользователя. Используйте адрес электронной почты `test@contoso.com` . На экране **Подтверждение регистрации** выберите **щелкните здесь, чтобы подтвердить свою учетную запись**.

1. Следуйте инструкциям в [кратком руководстве по добавлению флагов компонентов в приложение ASP.NET Core](./quickstart-feature-flag-aspnet-core.md) , чтобы добавить флаг компонента в новое веб-приложение.

1. Переключите флаг компонента в конфигурации приложения. Убедитесь, что это действие управляет видимостью элемента **бета-тестирования** на панели навигации.

## <a name="update-the-web-application-code-to-use-targetingfilter"></a>Обновление кода веб-приложения для использования Таржетингфилтер

На этом этапе можно использовать флаг функции, чтобы включить или отключить `Beta` функцию для всех пользователей. Чтобы включить флаг компонента для некоторых пользователей, отключив его к другим пользователям, обновите код для использования `TargetingFilter` . В этом примере вы будете использовать адрес электронной почты пользователя, выполнившего вход, в качестве идентификатора пользователя и часть имени домена адреса электронной почты в качестве группы. Вы добавите пользователя и группу в `TargetingContext` . `TargetingFilter`Использует этот контекст для определения состояния флага компонента для каждого запроса.

1. Обновите пакет до последней версии `Microsoft.FeatureManagement.AspNetCore` .

   ```dotnetcli
   dotnet add package Microsoft.FeatureManagement.AspNetCore
   ```

1. Добавьте файл *тесттаржетингконтекстакцессор. CS* :

    ```csharp
    using Microsoft.AspNetCore.Http;
    using Microsoft.FeatureManagement.FeatureFilters;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;

    namespace TestFeatureFlags
    {
        public class TestTargetingContextAccessor : ITargetingContextAccessor
        {
            private const string TargetingContextLookup = "TestTargetingContextAccessor.TargetingContext";
            private readonly IHttpContextAccessor _httpContextAccessor;

            public TestTargetingContextAccessor(IHttpContextAccessor httpContextAccessor)
            {
                _httpContextAccessor = httpContextAccessor ?? throw new ArgumentNullException(nameof(httpContextAccessor));
            }

            public ValueTask<TargetingContext> GetContextAsync()
            {
                HttpContext httpContext = _httpContextAccessor.HttpContext;
                if (httpContext.Items.TryGetValue(TargetingContextLookup, out object value))
                {
                    return new ValueTask<TargetingContext>((TargetingContext)value);
                }
                List<string> groups = new List<string>();
                if (httpContext.User.Identity.Name != null)
                {
                    groups.Add(httpContext.User.Identity.Name.Split("@", StringSplitOptions.None)[1]);
                }
                TargetingContext targetingContext = new TargetingContext
                {
                    UserId = httpContext.User.Identity.Name,
                    Groups = groups
                };
                httpContext.Items[TargetingContextLookup] = targetingContext;
                return new ValueTask<TargetingContext>(targetingContext);
            }
        }
    }
    ```

1. В *Startup. CS* добавьте ссылку на пространство имен *Microsoft. феатуреманажемент. феатурефилтерс* :

    ```csharp
    using Microsoft.FeatureManagement.FeatureFilters;
    ```

1. Обновите метод *ConfigureServices* для регистрации `TargetingFilter` , следуя вызову `AddFeatureManagement()` :

    ```csharp
    services.AddFeatureManagement()
            .AddFeatureFilter<TargetingFilter>();
    ```

1. Обновите метод *ConfigureServices* , чтобы добавить `TestTargetingContextAccessor` созданный ранее шаг в коллекцию служб. *Таржетингфилтер* использует его для определения контекста назначения при каждом вычислении флага компонента.

    ```csharp
      services.AddSingleton<ITargetingContextAccessor, TestTargetingContextAccessor>();
    ```

Весь метод *ConfigureServices* будет выглядеть следующим образом:

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));
    services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
            .AddEntityFrameworkStores<ApplicationDbContext>();
    services.AddControllersWithViews();
    services.AddRazorPages();

    // Add feature management, targeting filter, and ITargetingContextAccessor to service collection
    services.AddFeatureManagement().AddFeatureFilter<TargetingFilter>();
    services.AddSingleton<ITargetingContextAccessor, TestTargetingContextAccessor>();
    }
```

## <a name="update-the-feature-flag-to-use-targetingfilter"></a>Обновление флага компонента для использования Таржетингфилтер

1. В портал Azure перейдите к хранилищу конфигураций приложений и выберите **Диспетчер компонентов**.

1. Выберите контекстное меню для флага *бета-версии Beta* , созданного в кратком руководстве. Выберите **Изменить**.

    > [!div class="mx-imgBorder"]
    > ![Изменение флага бета-версии компонента](./media/edit-beta-feature-flag.png)

1. На экране **редактирования** установите флажок **включить флаг функции** , если он еще не установлен. Затем установите флажок **использовать фильтр компонентов** .

1. Выберите переключатель **нацеливание на целевой объект** .

1. Выберите следующие параметры.

    - **Процент по умолчанию**: 0
    - **Группы**: введите **имя** _contoso.com_ и **процент** от _50_
    - **Пользователи**: `test@contoso.com`

    Экран фильтра компонентов будет выглядеть следующим образом:

    > [!div class="mx-imgBorder"]
    > ![Условный флаг функции](./media/feature-flag-filter-enabled.png)

    Эти параметры приводят к следующему поведению:

    - Флаг функции всегда включен для пользователя `test@contoso.com` , так как `test@contoso.com` он указан в разделе _Пользователи_ .
    - Флаг функции включен для 50% других пользователей в группе _contoso.com_ , так как _contoso.com_ указан в разделе _Groups_ с _процентом_ _50_.
    - Эта функция всегда отключена для всех остальных пользователей, так как _процент по умолчанию_ имеет значение _0_.

1. Нажмите кнопку **Применить** , чтобы сохранить эти параметры и вернуться на экран **диспетчера компонентов** .

1. **Фильтр компонентов** для флага функции теперь отображается как *нацеливание*. Это состояние означает, что флаг компонента будет включен или отключен для каждого запроса в зависимости от критериев, принудительно примененных фильтром функций *нацеливания* .

## <a name="targetingfilter-in-action"></a>Таржетингфилтер в действии

Чтобы увидеть влияние этого флага функции, выполните сборку и запустите приложение. Изначально элемент *бета-версии* не отображается на панели инструментов, так как параметр _процент по умолчанию_ имеет значение 0.

Теперь войдите как `test@contoso.com` , используя пароль, заданный при регистрации. Теперь на панели инструментов отображается *бета-версия* элемента, так как `test@contoso.com` она указана как целевой пользователь.

В следующем видео показано это поведение в действии.

> [!div class="mx-imgBorder"]
> ![Таржетингфилтер в действии](./media/feature-flags-targetingfilter.gif)

Чтобы увидеть поведение параметров группы, можно создать дополнительных пользователей с `@contoso.com` адресами электронной почты. 50% этих пользователей увидят *бета-версию* элемента. Второй 50% не увидит элемент *бета-версии* .

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Общие сведения об управлении функциями](./concept-feature-management.md)
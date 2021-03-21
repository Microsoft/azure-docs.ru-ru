---
title: Использовать конфигурацию для отдельных сред
titleSuffix: Azure App Configuration
description: Используйте метки для предоставления значений конфигурации для отдельных сред.
ms.service: azure-app-configuration
author: AlexandraKemperMS
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 3/12/2020
ms.author: alkemper
ms.openlocfilehash: 84286df063994f3def15079cb9b190550d5bd977
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96929622"
---
# <a name="use-labels-to-enable-configurations-for-different-environments"></a>Использование меток для включения конфигураций для различных сред

Многие приложения должны использовать разные конфигурации для разных сред. Предположим, что приложение имеет значение конфигурации, определяющее строку подключения, которая будет использоваться для ее серверной базы данных. Разработчики приложений используют базу данных, отличную от той, которая используется в рабочей среде. Строка подключения к базе данных, используемая приложением, должна изменяться по мере перемещения приложения из среды разработки в рабочую среду.

В конфигурации приложений Azure *метки* можно использовать для определения различных значений для одного и того же ключа. Например, можно определить один ключ с различными значениями для разработки и рабочей среды. Вы можете указать, какую метку загружать при подключении к конфигурации приложения.

Чтобы продемонстрировать эту функциональность, вы измените веб-приложение, созданное в [кратком руководстве: создание ASP.NET Core приложения с помощью конфигурации приложения Azure](./quickstart-aspnet-core-app.md) для использования различных параметров конфигурации для разработки и рабочей среды. Перед продолжением работы выполните инструкции из краткого руководства.

## <a name="specify-a-label-when-adding-a-configuration-value"></a>Указание метки при добавлении значения конфигурации

В портал Azure перейдите в раздел **Обозреватель конфигураций** и найдите раздел *TestApp: Settings: FontColor* , созданный в кратком руководстве. Выберите его контекстное меню и щелкните **Добавить значение**.

> [!div class="mx-imgBorder"]
> ![Добавить пункт меню "значение"](media/labels-add-value.png)

На экране **Добавить значение** введите **красный** и  **метку** **разработки**. Оставьте поле **тип содержимого** пустым. Нажмите кнопку **Применить**.

## <a name="load-configuration-values-with-a-specified-label"></a>Загрузить значения конфигурации с указанной меткой

По умолчанию в конфигурации приложений Azure загружаются только значения конфигурации без метки. Если вы определили метки для значений конфигурации, необходимо указать метки, которые будут использоваться при подключении к конфигурации приложения.

В предыдущем разделе вы создали другое значение конфигурации для среды разработки. `HostingEnvironment.EnvironmentName`Переменная используется для динамического определения среды, в которой выполняется приложение. Дополнительные сведения см. в статье [Использование нескольких сред в ASP.NET Core](/aspnet/core/fundamentals/environments).

Загрузите значения конфигурации с меткой, соответствующей текущей среде, передав имя среды в `Select` метод:

```csharp
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(options =>
                    options
                        .Connect(Environment.GetEnvironmentVariable("AppConfigConnectionString"))
                        // Load configuration values with no label
                        .Select(KeyFilter.Any, LabelFilter.Null)
                        // Override with any configuration values specific to current hosting env
                        .Select(KeyFilter.Any, hostingContext.HostingEnvironment.EnvironmentName)
                );
            })
            .UseStartup<Startup>());
```

> [!IMPORTANT]
> Предыдущий фрагмент кода загружает строку подключения конфигурации приложения из переменной среды с именем `AppConfigConnectionString` . Убедитесь, что эта переменная среды задана правильно.

Метод `Select` вызывается два раза. В первый раз он загружает значения конфигурации без метки. Затем он загружает значения конфигурации с меткой, соответствующей текущей среде. Эти значения, относящиеся к конкретной среде, переопределяют все соответствующие значения без метки. Для каждого ключа не нужно определять специфические для среды значения. Если у ключа нет значения с меткой, соответствующей текущей среде, используется значение без метки.

## <a name="test-in-different-environments"></a>Тестирование в различных средах

Откройте `launchSettings.json` файл в `Properties` каталоге. Найдите `config` запись в разделе `profiles` . В `environmentVariables` разделе задайте `ASPNETCORE_ENVIRONMENT` для переменной значение `Production` .

После установки новых значений создайте и запустите приложение.

```dotnetcli
dotnet build
dotnet run
```

Используйте веб-браузер для перехода по адресу `http://localhost:5000` . Обратите внимание, что цвет шрифта — черный.

![Веб-приложение, выполняемое с рабочей конфигурацией](media/labels-website-prod.png)

Обновите, `launchSettings.json` чтобы присвоить `ASPNETCORE_ENVIRONMENT` переменной значение `Development` . Еще раз запустите `dotnet run`. 

Вы заметите, что цвет шрифта теперь красный. Это связано с тем, что приложение теперь использует значение `TestApp:Settings:FontColor` , имеющее `Development` метку. Все остальные значения конфигурации остаются теми же, что и их рабочие значения.

![Веб-приложение, выполняемое с конфигурацией разработки](media/labels-website-dev.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Конфигурация в .NET Core](/aspnet/core/fundamentals/configuration/)

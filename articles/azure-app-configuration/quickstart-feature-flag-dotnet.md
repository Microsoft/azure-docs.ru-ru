---
title: Краткое руководство по добавлению флагов функций в приложения .NET Framework | Документация Майкрософт | Документация Майкрософт
description: Краткое руководство по добавлению флагов функций в приложения .NET Framework и управлению ими в Конфигурации приложений Azure
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.tgt_pltfrm: .NET
ms.workload: tbd
ms.date: 10/19/2020
ms.author: alkemper
ms.openlocfilehash: 513c826e11ff9dfe6ea94349c67620da9d1bba48
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96932053"
---
# <a name="quickstart-add-feature-flags-to-a-net-framework-app"></a>Краткое руководство. Добавление флагов функций в приложение .NET Framework

В этом кратком руководстве описано, как внедрить Конфигурацию приложений Azure в приложение .NET Framework, чтобы реализовать комплексное управление функциями. Вы можете использовать службу "Конфигурация приложений" для централизованного хранения всех флагов функций и управления их состояниями. 

Библиотеки управления функциями .NET расширяют возможности платформы за счет поддержки флагов функций. Эти библиотеки создаются на основе системы конфигурации .NET. Они интегрируются со службой "Конфигурация приложений" посредством поставщика конфигураций .NET.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs)
- [.NET Framework 4.8](https://dotnet.microsoft.com/download)

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Выберите **Диспетчер компонентов** >  **+Добавить**, чтобы добавить флаг функции `Beta`.

    > [!div class="mx-imgBorder"]
    > ![Включение флага функции с именем Beta](media/add-beta-feature-flag.png)

    Не определяйте `label` сейчас.

## <a name="create-a-net-console-app"></a>Создайте консольное приложение .NET

1. Запустите Visual Studio и выберите **Файл** > **Создать** > **Проект**.

1. В разделе **Создание проекта** с помощью фильтра найдите тип проекта **Консоль** и щелкните **Консольное приложение (.NET Framework)** . Щелкните **Далее**.

1. В окне **Настроить новый проект** введите имя проекта. В разделе **Платформа** выберите **.NET Framework 4.8** или более позднюю версию. Нажмите кнопку **Создать**.

## <a name="connect-to-an-app-configuration-store"></a>Подключение к хранилищу Конфигурации приложений

1. Щелкните проект правой кнопкой мыши и выберите **Управление пакетами NuGet**. На вкладке **Обзор** найдите и добавьте в проект следующие пакеты NuGet. Если вы не можете их найти, установите флажок **Включить предварительные выпуски**.

    ```
    Microsoft.Extensions.DependencyInjection
    Microsoft.Extensions.Configuration.AzureAppConfiguration
    Microsoft.FeatureManagement
    ```

1. Откройте файл *Program.cs* и добавьте следующие инструкции:

    ```csharp
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    using Microsoft.FeatureManagement;
    using System.Threading.Tasks;
    ```

1. Обновите метод `Main` для подключения к Конфигурации приложения, указав параметр `UseFeatureFlags`, чтобы получить флаги функции. Затем отобразите сообщение, если флаг функции `Beta` включен.

    ```csharp
        public static async Task Main(string[] args)
        {         
            IConfigurationRoot configuration = new ConfigurationBuilder()
                .AddAzureAppConfiguration(options =>
                {
                    options.Connect(Environment.GetEnvironmentVariable("ConnectionString"))
                           .UseFeatureFlags();
                }).Build();

            IServiceCollection services = new ServiceCollection();

            services.AddSingleton<IConfiguration>(configuration).AddFeatureManagement();

            using (ServiceProvider serviceProvider = services.BuildServiceProvider())
            {
                IFeatureManager featureManager = serviceProvider.GetRequiredService<IFeatureManager>();

                if (await featureManager.IsEnabledAsync("Beta"))
                {
                    Console.WriteLine("Welcome to the beta!");
                }
            }

            Console.WriteLine("Hello World!");
            Console.WriteLine("Press any key to continue ...");
            Console.Read();
        }
    ```

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Задайте переменную среды с именем **ConnectionString** для строки подключения к хранилищу службы "Конфигурация приложений". При использовании командной строки Windows выполните следующую команду:

    ```console
        setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете Windows PowerShell, выполните следующую команду:

    ```powershell
        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

1. Перезапустите Visual Studio, чтобы изменение вступило в силу. 

1. Нажмите клавиши CTRL+F5, чтобы скомпилировать и запустить консольное приложение.

    ![Приложение с включенным флагом функций](./media/quickstarts/dotnet-app-feature-flag.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали флаг функции в Конфигурации приложений и использовали его с консольным приложением .NET Framework. Сведения о динамическом обновлении флагов функций и других значениях конфигурации без перезапуска приложения, см. в следующем руководстве.

> [!div class="nextstepaction"]
> [Включение динамической конфигурации](./enable-dynamic-configuration-dotnet.md)

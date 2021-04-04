---
title: Краткое руководство по использованию службы конфигурации приложений Azure с .NET Framework | Документация Майкрософт
description: В этой статье описано, как создать приложение .NET Framework с помощью службы "Конфигурация приложений Azure", чтобы обеспечить централизованное хранение параметров приложения и управление ими отдельно от кода.
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.date: 09/28/2020
ms.author: alkemper
ms.openlocfilehash: 62516218ed2c0249f829ad8d286e4ad8bbc471f8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96932097"
---
# <a name="quickstart-create-a-net-framework-app-with-azure-app-configuration"></a>Краткое руководство. Создание приложения на .NET Framework с помощью службы конфигурации приложений Azure

В этом кратком руководстве описано, как добавить службу конфигурации приложений Azure в консольное приложение .NET Framework, чтобы обеспечить централизованное хранение параметров приложения и управление ими отдельно от кода.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs)
- [.NET Framework 4.7.2](https://dotnet.microsoft.com/download)

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Выберите **Обозреватель конфигураций** > **Создать** > **Ключ-значение**, чтобы добавить следующие пары "ключ-значение":

    | Клавиши | Значение |
    |---|---|
    | TestApp:Settings:FontSize | Данные из конфигурации приложения Azure |

    Поля **Метка** и **Тип контента** пока заполнять не нужно.

8. Нажмите кнопку **Применить**.

## <a name="create-a-net-console-app"></a>Создайте консольное приложение .NET

1. Запустите Visual Studio и выберите **Файл** > **Создать** > **Проект**.

1. В разделе **Создание проекта** с помощью фильтра найдите тип проекта **Консоль** и щелкните **Консольное приложение (.NET Framework)** . Выберите **Далее**.

1. В окне **Настроить новый проект** введите имя проекта. В разделе **Платформа** выберите **.NET Framework 4.7.1** или более поздней версии. Нажмите кнопку **создания**.

## <a name="connect-to-an-app-configuration-store"></a>Подключение к хранилищу Конфигурации приложений

1. Щелкните проект правой кнопкой мыши и выберите **Управление пакетами NuGet**. На вкладке **Обзор** найдите и добавьте в проект следующие пакеты NuGet.

    ```
    Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration 1.0.0 or later
    Microsoft.Configuration.ConfigurationBuilders.Environment 2.0.0 or later
    System.Configuration.ConfigurationManager version 4.6.0 or later
    ```

1. Обновите файл проекта *App.config*:

    ```xml
    <configSections>
        <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
    </configSections>

    <configBuilders>
        <builders>
            <add name="MyConfigStore" mode="Greedy" connectionString="${ConnectionString}" type="Microsoft.Configuration.ConfigurationBuilders.AzureAppConfigurationBuilder, Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration" />
            <add name="Environment" mode="Greedy" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment" />
        </builders>
    </configBuilders>

    <appSettings configBuilders="Environment,MyConfigStore">
        <add key="AppName" value="Console App Demo" />
        <add key="ConnectionString" value ="Set via an environment variable - for example, dev, test, staging, or production connection string." />
    </appSettings>
    ```

   Строка подключения к хранилищу службы "Конфигурация приложений" считывается из переменной среды `ConnectionString`. Добавьте построитель конфигураций `Environment` перед `MyConfigStore` в свойстве `configBuilders` раздела `appSettings`.

1. Откройте файл *Program.cs* и обновите метод `Main` для использования службы "Конфигурация приложений", вызвав метод `ConfigurationManager`.

    ```csharp
    static void Main(string[] args)
    {
        string message = System.Configuration.ConfigurationManager.AppSettings["TestApp:Settings:Message"];

        Console.WriteLine(message);
        Console.ReadKey();
    }
    ```

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Обновите файл **App.config**, заменив `${ConnectionString}` фактической строкой подключения для экземпляра службы "Конфигурация приложений". Эту строку можно найти на вкладке **Ключи доступа** для ресурса службы "Конфигурация приложений" на портале Azure.

1. Нажмите клавиши CTRL+F5, чтобы скомпилировать и запустить консольное приложение.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве описано, как создать хранилище службы "Конфигурация приложений" и использовать его с консольным приложением на основе .NET Framework. После запуска приложения значение `AppSettings` параметра `ConfigurationManager` не изменится. Однако в приложении .NET Framework также может использоваться библиотека поставщика конфигурации .NET Standard службы "Конфигурация приложений". Чтобы узнать, как включить в приложении .NET Framework динамическое обновление параметров конфигурации, перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Включение динамической конфигурации](./enable-dynamic-configuration-dotnet.md)

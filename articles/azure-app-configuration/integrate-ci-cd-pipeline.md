---
title: Интеграция службы "Конфигурация приложений Azure" с помощью конвейера непрерывной интеграции и доставки
description: Узнайте, как реализовать непрерывную интеграцию и доставку с использованием службы "Конфигурация приложений Azure".
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.date: 04/19/2020
ms.author: alkemper
ms.openlocfilehash: 3a4d171f0e3225db195c5c2b71ca99a3386e3a36
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99979850"
---
# <a name="integrate-with-a-cicd-pipeline"></a>Интеграция с конвейером CI/CD

Из этой статьи вы узнаете, как данные из службы "Конфигурация приложений Azure" можно использовать в системе непрерывной интеграции и непрерывного развертывания.

## <a name="use-app-configuration-in-your-azure-devops-pipeline"></a>Использование Конфигурации приложений в конвейере Azure DevOps

Если у вас есть конвейер Azure DevOps, можно извлечь значения ключей из Конфигурации приложений и установить их в качестве переменных задач. [Расширение Конфигурации приложений для Azure DevOps](https://go.microsoft.com/fwlink/?linkid=2091063) — это дополнительный модуль, обеспечивающий эти функциональные возможности. Следуйте его инструкциям по использованию расширения в последовательности задач сборки или выпуска.

## <a name="deploy-app-configuration-data-with-your-application"></a>Развертывание данных Конфигурации приложений с приложением

Ваше приложение может не запуститься, если оно зависит от Конфигурации приложений Azure и не сможет получить к ней доступ. Чтобы повысить устойчивость приложения, упакуйте данные конфигурации в файл, который развертывается вместе с приложением и загружается локально во время запуска приложения. Такой подход гарантирует, что при запуске приложение будет получать стандартные значения параметров. Эти значения будут перезаписаны более новыми изменениями из хранилища Конфигурации приложений, когда оно станет доступно.

С помощью функции [Экспорт](./howto-import-export-data.md#export-data) Конфигурации приложений Azure можно автоматизировать процесс получения текущих данных конфигурации в виде отдельного файла. Затем на этапе сборки или развертывания этот файл можно внедрить в конвейер непрерывной интеграции и непрерывного развертывания (CI/CD).

Ниже приведен пример того, как включить данные Конфигурации приложений в качестве этапа сборки для веб-приложения, представленного в кратких руководствах. Прежде чем продолжить, ознакомьтесь со статьей [Краткое руководство. Создание приложения ASP.NET Core с помощью службы "Конфигурация приложений Azure"](./quickstart-aspnet-core-app.md).

Вы можете выполнять шаги в этом учебнике с помощью любого редактора кода. [Visual Studio Code](https://code.visualstudio.com/) является отличным вариантом, который доступен на платформах Windows, macOS и Linux.

### <a name="prerequisites"></a>Предварительные требования

Если сборка выполняется локально, скачайте и установите [Azure CLI](/cli/azure/install-azure-cli), если этот компонент отсутствует.

Чтобы выполнить облачную сборку, например с помощью DevOps в Azure, убедитесь, что в системе сборки установлен [Azure CLI](/cli/azure/install-azure-cli).

### <a name="export-an-app-configuration-store"></a>Экспорт хранилища Конфигурации приложений

1. Откройте *CSPROJ*-файл и добавьте в него приведенный ниже сценарий.

    ```xml
    <Target Name="Export file" AfterTargets="Build">
        <Message Text="Export the configurations to a temp file. " />
        <Exec WorkingDirectory="$(MSBuildProjectDirectory)" Condition="$(ConnectionString) != ''" Command="az appconfig kv export -d file --path $(OutDir)\azureappconfig.json --format json --separator : --connection-string $(ConnectionString)" />
    </Target>
    ```
1. Откройте файл *Program.cs* и обновите метод `CreateWebHostBuilder`, чтобы использовать экспортированный JSON-файл с помощью вызова метода `config.AddJsonFile()`.  Добавьте также пространство имен `System.Reflection`.

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var directory = System.IO.Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
                var settings = config.Build();

                config.AddJsonFile(Path.Combine(directory, "azureappconfig.json"));
                config.AddAzureAppConfiguration(settings["ConnectionStrings:AppConfig"]);
            })
            .UseStartup<Startup>();
    ```

### <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Задайте переменную среды с именем **ConnectionString** и укажите для нее ключ доступа к хранилищу службы "Конфигурация приложений". 
    Если вы используете командную строку Windows, выполните следующую команду и перезапустите командную строку, чтобы изменения вступили в силу:

    ```console
     setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете Windows PowerShell, выполните следующую команду:

    ```powershell
     $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете macOS или Linux, выполните следующую команду:

    ```console
     export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

2. Чтобы создать приложение с помощью .NET Core CLI, выполните следующую команду в оболочке командной строки:

    ```console
     dotnet build
    ```

3. Когда создание завершится, запустите веб-приложение локально с помощью следующей команды:

    ```console
     dotnet run
    ```

4. Откройте окно браузера и перейдите по адресу `http://localhost:5000`, который является URL-адресом по умолчанию для веб-приложения, размещенного локально.

    ![Краткое руководство. Запуск приложения, размещенного локально](./media/quickstarts/aspnet-core-app-launch-local.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы экспортировали данные Конфигурации приложений Azure для использования в конвейере развертывания. Чтобы узнать больше об использовании службы "Конфигурация приложений", перейдите к примерам скриптов Azure CLI.

> [!div class="nextstepaction"]
> [Azure CLI](/cli/azure/appconfig)

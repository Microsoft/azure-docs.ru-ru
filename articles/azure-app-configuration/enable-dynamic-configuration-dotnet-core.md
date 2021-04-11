---
title: Руководство по Использование динамической конфигурации в приложении .NET Core
titleSuffix: Azure App Configuration
description: Из этого руководства вы узнаете, как динамически обновлять данные конфигурации для приложений .NET Core.
services: azure-app-configuration
documentationcenter: ''
author: GrantMeStrength
manager: zhenlan
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.custom: devx-track-csharp
ms.topic: tutorial
ms.date: 07/01/2019
ms.author: jken
ms.openlocfilehash: bb0c83536f8973a07d656844439a8816dea83346
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105625710"
---
# <a name="tutorial-use-dynamic-configuration-in-a-net-core-app"></a>Руководство по Использование динамической конфигурации в приложении .NET Core

Клиентская библиотека .NET Core для конфигурации приложений поддерживает конфигурации по запросу без перезапуска приложения. Это можно реализовать, сначала получив экземпляр `IConfigurationRefresher` из параметров для поставщика конфигурации, а затем вызвав `TryRefreshAsync` на этом экземпляре, где угодно в коде.

Чтобы поддерживать параметры в актуальном состоянии и избегать слишком большого количества обращений к хранилищу конфигураций, для каждого параметра используется кэш. До истечения срока действия кэшированного значения, операция обновления не выполняет обновление значения, даже если значение изменилось в хранилище конфигураций. Стандартное время истечения для каждого запроса составляет 30 секунд, но при необходимости его можно переопределить.

Из этого руководства вы узнаете, как реализовать динамические обновления конфигурации в коде. При этом в качестве основы используется код приложения, представленный в кратких руководствах. Прежде чем продолжить, ознакомьтесь со статьей [Краткое руководство. Создание приложения .NET Core с помощью службы "Конфигурация приложений Azure"](./quickstart-dotnet-core-app.md).

Вы можете выполнять шаги в этом учебнике с помощью любого редактора кода. [Visual Studio Code](https://code.visualstudio.com/) является отличным вариантом, который доступен на платформах Windows, macOS и Linux.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка приложения .NET Core на обновление конфигурации при изменении данных в хранилище службы "Конфигурация приложений";
> * использование последней конфигурации в приложении.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="reload-data-from-app-configuration"></a>Перезагрузка данных из App Configuration

Откройте и обновите файл *Program.cs*, чтобы добавить ссылку на пространство имен `System.Threading.Tasks`, указать конфигурацию обновления в методе `AddAzureAppConfiguration` и запустить обновление вручную с помощью метода `TryRefreshAsync`.

```csharp
using System;
using System.Threading.Tasks;

namespace TestConsole
{
class Program
{
    private static IConfiguration _configuration = null;
    private static IConfigurationRefresher _refresher = null;

    static void Main(string[] args)
    {
        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(options =>
        {
            options.Connect(Environment.GetEnvironmentVariable("ConnectionString"))
                    .ConfigureRefresh(refresh =>
                    {
                        refresh.Register("TestApp:Settings:Message")
                               .SetCacheExpiration(TimeSpan.FromSeconds(10));
                    });
                    
                    _refresher = options.GetRefresher();
        });

        _configuration = builder.Build();
        PrintMessage().Wait();
    }

    private static async Task PrintMessage()
    {
        Console.WriteLine(_configuration["TestApp:Settings:Message"] ?? "Hello world!");

        // Wait for the user to press Enter
        Console.ReadLine();

        await _refresher.TryRefreshAsync();
        Console.WriteLine(_configuration["TestApp:Settings:Message"] ?? "Hello world!");
    }
}
}
```

Метод `ConfigureRefresh` используется, чтобы указать параметры, используемые для обновления данных конфигурации в хранилище службы "Конфигурация приложений" при запуске операции обновления. Экземпляр `IConfigurationRefresher` можно получить, вызвав метод `GetRefresher` в параметрах, предоставленных методу `AddAzureAppConfiguration`, а метод `TryRefreshAsync` в этом экземпляре можно использовать для запуска операции обновления в любом месте вашего кода.
    
> [!NOTE]
> Стандартное время истечения срока действия кэша для параметра конфигурации составляет 30 секунд, но его можно переопределить, вызвав метод `SetCacheExpiration` в инициализаторе параметров, переданном в качестве аргумента методу `ConfigureRefresh`.

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Задайте переменную среды с именем **ConnectionString** и укажите для нее ключ доступа к хранилищу службы "Конфигурация приложений". Если вы используете командную строку Windows, выполните следующую команду и перезапустите командную строку, чтобы изменения вступили в силу:

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

1. Чтобы создать консольное приложение, выполните следующую команду:

    ```console
     dotnet build
    ```

1. Когда создание завершится, запустите приложение локально с помощью следующей команды:

    ```console
     dotnet run
    ```

    ![Краткое руководство. Запуск приложения, размещенного локально](./media/quickstarts/dotnet-core-app-run.png)

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Все ресурсы** и выберите экземпляр хранилища Конфигурации приложений, который вы создали по инструкциям из краткого руководства.

1. Выберите **Configuration Explorer** (Обозреватель конфигураций) и измените значения следующих ключей.

    | Клавиши | Значение |
    |---|---|
    | TestApp:Settings:FontSize | Данные из конфигурации приложений Azure. Обновлено |

1. Нажмите клавишу Enter, чтобы запустить обновление и вывести обновленное значение в окне командной строки или PowerShell.

    ![Краткое руководство. Обновление размещенного локально приложения](./media/quickstarts/dotnet-core-app-run-refresh.png)
    
    > [!NOTE]
    > Поскольку время истечения срока действия кэша было установлено равным 10 секундам с использованием метода `SetCacheExpiration` при указании конфигурации для операции обновления, то значение параметра конфигурации будет обновляться только в том случае, если с момента последнего обновления этого параметра прошло не менее 10 секунд.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы включили в приложении .NET Core динамическое обновление параметров конфигурации из службы "Конфигурация приложения". Чтобы узнать, как с помощью удостоверения, управляемого Azure, упростить доступ к службе "Конфигурация приложений Azure", перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Руководство по интеграции с управляемыми удостоверениями Azure](./howto-integrate-azure-managed-service-identity.md)

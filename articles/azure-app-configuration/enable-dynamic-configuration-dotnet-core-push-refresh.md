---
title: Руководство. Использование динамической конфигурации путем обновления в режиме отправки в приложении .NET Core
titleSuffix: Azure App Configuration
description: Из этого руководства вы узнаете, как динамически обновлять данные конфигурации для приложений .NET Core путем обновления в режиме отправки
services: azure-app-configuration
documentationcenter: ''
author: GrantMeStrength
manager: zhenlan
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 07/25/2020
ms.author: jken
ms.openlocfilehash: f330e69d37ac36e2a374e81fbffc6235c86feedd
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105625727"
---
# <a name="tutorial-use-dynamic-configuration-using-push-refresh-in-a-net-core-app"></a>Руководство. Использование динамической конфигурации путем обновления в режиме отправки в приложении .NET Core

Клиентская библиотека .NET Core для Конфигурации приложений поддерживает обновление конфигурации по запросу без перезапуска приложения. Приложение можно настроить для обнаружения изменений в Конфигурации приложений с помощью одного из описанных ниже подходов или их обоих.

1. Модель опроса. Это поведение по умолчанию, при котором для обнаружения изменений в конфигурации используется опрос. Когда истекает срок действия кэшированного значения параметра, при следующем вызове к `TryRefreshAsync` или `RefreshAsync` отправляется запрос на сервер, чтобы проверить, изменилась ли конфигурация. При необходимости извлекается обновленная конфигурация.

1. Модель отправки. В этой модели для обнаружения изменений в конфигурации применяются [события Конфигурации приложений](./concept-app-configuration-event.md). Когда Конфигурация приложений будет настроена для отправки событий изменения значения ключа в Сетку событий Azure, с их помощью приложение сможет оптимизировать общее число запросов, необходимых для обновления конфигурации. Приложения могут подписываться на события напрямую из Сетки событий либо с помощью одного из [поддерживаемых обработчиков событий](../event-grid/event-handlers.md), таких как веб-перехватчик, функция Azure либо раздел Служебной шины.

Приложения могут подписываться на эти события непосредственно из Сетки событий, через веб-перехватчик или путем переадресации событий в Служебную шину Azure. Пакет SDK для Служебной шины Azure предоставляет API для регистрации обработчика сообщений, упрощающего процесс для приложений, у которых нет конечной точки HTTP или возможности постоянно опрашивать Сетку событий на предмет изменений.

Из этого руководства вы узнаете, как реализовать динамические обновления конфигурации в коде путем обновления в режиме отправки. При этом в качестве основы используется код приложения, представленный в кратких руководствах. Прежде чем продолжить, ознакомьтесь со статьей [Краткое руководство. Создание приложения .NET Core с помощью службы "Конфигурация приложений Azure"](./quickstart-dotnet-core-app.md).

Вы можете выполнять шаги в этом учебнике с помощью любого редактора кода. [Visual Studio Code](https://code.visualstudio.com/) является отличным вариантом, который доступен на платформах Windows, macOS и Linux.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка подписки на отправку событий изменения конфигурации из Конфигурации приложений в раздел Служебной шины;
> * настройка приложения .NET Core для обновления конфигурации при изменении данных в Конфигурации приложений;
> * использование последней конфигурации в приложении.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-azure-service-bus-topic-and-subscription"></a>Настройка раздела и подписки Служебной шины Azure

Для работы с этим руководством используется интеграция Служебной шины для Сетки событий, чтобы упростить обнаружение изменений конфигурации для приложений, которые не могут опрашивать Конфигурацию приложений на предмет изменений. Пакет SDK для Служебной шины Azure предоставляет API для регистрации обработчика сообщений, с помощью которого можно обновлять конфигурацию при обнаружении изменений в Конфигурации приложений. Выполните шаги из [краткого руководства по созданию раздела и подписки Служебной шины с помощью портала Azure](../service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md), чтобы создать пространство имен, раздел и подписку Служебной шины.

После создания ресурсов добавьте указанные ниже переменные среды. С их помощью будет регистрироваться обработчик событий для изменений конфигурации в коде приложения.

| Ключ | Значение |
|---|---|
| ServiceBusConnectionString | Строка подключения к пространству имен Служебной шины |
| ServiceBusTopic | Имя раздела Служебной шины |
| ServiceBusSubscription | Имя подписки Служебной шины |

## <a name="set-up-event-subscription"></a>Настройка подписки на события

1. Откройте ресурс Конфигурации приложений на портале Azure, а затем щелкните `+ Event Subscription` в области `Events`.

    ![События Конфигурации приложений](./media/events-pane.png)

1. Введите имена для `Event Subscription` и `System Topic`.

    ![Создание подписки на события](./media/create-event-subscription.png)

1. Выберите для `Endpoint Type` значение `Service Bus Topic`, выберите раздел Служебной шины, а затем щелкните `Confirm Selection`.

    ![Конечная точка Служебной шины для подписки на события](./media/event-subscription-servicebus-endpoint.png)

1. Щелкните `Create`, чтобы добавить подписку на событие.

1. Щелкните `Event Subscriptions` в области `Events`, чтобы проверить, создана ли подписка.

    ![Подписки на события Конфигурации приложений](./media/event-subscription-view.png)

> [!NOTE]
> При подписке на изменения конфигурации можно использовать один или несколько фильтров. Они позволяют сократить число событий, отправляемых в приложение. Такие фильтры можно настроить как [фильтры подписки Сетки событий](../event-grid/event-filtering.md) или [фильтры подписки Служебной шины](../service-bus-messaging/topic-filters.md). Например, с помощью фильтра можно подписаться только на события изменений в ключе, который начинается с определенной строки.

## <a name="register-event-handler-to-reload-data-from-app-configuration"></a>Регистрация обработчика событий для повторной загрузки данных из Конфигурации приложений

Откройте файл *Program.cs* и замените код в нем следующим:

```csharp
using Microsoft.Azure.ServiceBus;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Configuration.AzureAppConfiguration;
using System;
using System.Diagnostics;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

namespace TestConsole
{
    class Program
    {
        private const string AppConfigurationConnectionStringEnvVarName = "AppConfigurationConnectionString"; // e.g. Endpoint=https://{store_name}.azconfig.io;Id={id};Secret={secret}
        private const string ServiceBusConnectionStringEnvVarName = "ServiceBusConnectionString"; // e.g. Endpoint=sb://{service_bus_name}.servicebus.windows.net/;SharedAccessKeyName={key_name};SharedAccessKey={key}
        private const string ServiceBusTopicEnvVarName = "ServiceBusTopic";
        private const string ServiceBusSubscriptionEnvVarName = "ServiceBusSubscription";

        private static IConfigurationRefresher _refresher = null;

        static async Task Main(string[] args)
        {
            string appConfigurationConnectionString = Environment.GetEnvironmentVariable(AppConfigurationConnectionStringEnvVarName);

            IConfiguration configuration = new ConfigurationBuilder()
                .AddAzureAppConfiguration(options =>
                {
                    options.Connect(appConfigurationConnectionString);
                    options.ConfigureRefresh(refresh =>
                        refresh
                            .Register("TestApp:Settings:Message")
                            .SetCacheExpiration(TimeSpan.FromDays(30))  // Important: Reduce poll frequency
                    );

                    _refresher = options.GetRefresher();
                }).Build();

            RegisterRefreshEventHandler();
            var message = configuration["TestApp:Settings:Message"];
            Console.WriteLine($"Initial value: {configuration["TestApp:Settings:Message"]}");

            while (true)
            {
                await _refresher.TryRefreshAsync();

                if (configuration["TestApp:Settings:Message"] != message)
                {
                    Console.WriteLine($"New value: {configuration["TestApp:Settings:Message"]}");
                    message = configuration["TestApp:Settings:Message"];
                }
                
                await Task.Delay(TimeSpan.FromSeconds(1));
            }
        }

        private static void RegisterRefreshEventHandler()
        {
            string serviceBusConnectionString = Environment.GetEnvironmentVariable(ServiceBusConnectionStringEnvVarName);
            string serviceBusTopic = Environment.GetEnvironmentVariable(ServiceBusTopicEnvVarName);
            string serviceBusSubscription = Environment.GetEnvironmentVariable(ServiceBusSubscriptionEnvVarName);
            SubscriptionClient serviceBusClient = new SubscriptionClient(serviceBusConnectionString, serviceBusTopic, serviceBusSubscription);

            serviceBusClient.RegisterMessageHandler(
                handler: (message, cancellationToken) =>
               {
                   string messageText = Encoding.UTF8.GetString(message.Body);
                   JsonElement messageData = JsonDocument.Parse(messageText).RootElement.GetProperty("data");
                   string key = messageData.GetProperty("key").GetString();
                   Console.WriteLine($"Event received for Key = {key}");

                   _refresher.SetDirty();
                   return Task.CompletedTask;
               },
                exceptionReceivedHandler: (exceptionargs) =>
                {
                    Console.WriteLine($"{exceptionargs.Exception}");
                    return Task.CompletedTask;
                });
        }
    }
}
```

Метод [SetDirty](/dotnet/api/microsoft.extensions.configuration.azureappconfiguration.iconfigurationrefresher.setdirty) позволяет установить кэшированное значения для значений ключа, зарегистрированных для обновления как "грязные". Это гарантирует, что при следующем вызове `RefreshAsync` или `TryRefreshAsync` с помощью Конфигурации приложений повторно проверяются кэшированные значения. При необходимости они обновляются.

До момента, когда значение помечается как "грязное", добавляется случайная задержка, чтобы уменьшить возможное регулирование в случае одновременного обновления нескольких экземпляров. Максимальная задержка по умолчанию до момента, когда кэшированное значение помечается как "грязное", составляет 30 секунд. Это значение можно переопределить, передав необязательный параметр `TimeSpan` в метод `SetDirty`.

> [!NOTE]
> Чтобы сократить количество запросов к Конфигурации приложений при обновлении в режиме отправки, важно вызвать `SetCacheExpiration(TimeSpan cacheExpiration)` с соответствующим значением параметра `cacheExpiration`, чтобы контролировать срок действия кэша для обновления в режиме отправки и использовать в качестве средства обеспечения безопасности в случае возникновения проблемы с подпиской на событие или подпиской Служебной шины. Мы рекомендуем использовать значение `TimeSpan.FromDays(30)`.

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Задайте переменную среды с именем **AppConfigurationConnectionString** и присвойте ей значение ключа доступа к хранилищу Конфигурации приложений. Если вы используете командную строку Windows, выполните следующую команду и перезапустите командную строку, чтобы изменения вступили в силу:

    ```console
     setx AppConfigurationConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете Windows PowerShell, выполните следующую команду:

    ```powershell
     $Env:AppConfigurationConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете macOS или Linux, выполните следующую команду:

    ```console
     export AppConfigurationConnectionString='connection-string-of-your-app-configuration-store'
    ```

1. Чтобы создать консольное приложение, выполните следующую команду:

    ```console
     dotnet build
    ```

1. Когда создание завершится, запустите приложение локально с помощью следующей команды:

    ```console
     dotnet run
    ```

    ![Запуск режима отправки перед обновлением](./media/dotnet-core-app-pushrefresh-initial.png)

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Все ресурсы** и выберите экземпляр хранилища Конфигурации приложений, который вы создали по инструкциям из краткого руководства.

1. Выберите **Configuration Explorer** (Обозреватель конфигураций) и измените значения следующих ключей.

    | Клавиши | Значение |
    |---|---|
    | TestApp:Settings:FontSize | Данные из конфигурации приложений Azure. Обновлено |

1. Подождите 30 секунд, пока будет обработано событие и обновится конфигурация.

    ![Запуск режима отправки после обновления](./media/dotnet-core-app-pushrefresh-final.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы включили в приложении .NET Core динамическое обновление параметров конфигурации из службы "Конфигурация приложения". Чтобы узнать, как с помощью удостоверения, управляемого Azure, упростить доступ к службе "Конфигурация приложений Azure", перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Руководство по интеграции с управляемыми удостоверениями Azure](./howto-integrate-azure-managed-service-identity.md)
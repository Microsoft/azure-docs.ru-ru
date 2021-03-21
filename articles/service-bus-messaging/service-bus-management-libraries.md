---
title: Программное создание сущностей служебной шины Azure | Документация Майкрософт
description: В этой статье объясняется, как использовать динамически или программно подготавливать пространства имен и сущности служебной шины.
ms.devlang: dotnet
ms.topic: article
ms.date: 01/13/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 57192ab2ee1624cb18de832ac91c95290da727df
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98539868"
---
# <a name="dynamically-provision-service-bus-namespaces-and-entities"></a>Динамическая инициализация пространств имен и сущностей служебной шины 
Библиотеки управления служебной шины Azure могут динамически подготавливать пространства имен и сущности служебной шины. Это дает возможность реализовывать сложные развертывания и сценарии обмена сообщениями и позволяет программно определять, какие сущности следует подготовить. В настоящее время эти библиотеки доступны для .NET.

## <a name="overview"></a>Обзор
Для создания сущностей служебной шины и управления ими доступны три библиотеки управления. Их можно сформулировать следующим образом.

- [Azure. Messaging. ServiceBus. Administration](#azuremessagingservicebusadministration)
- [Microsoft. Azure. ServiceBus. Management](#microsoftazureservicebusmanagement)
- [Microsoft.Azure.Management.ServiceBus.](#microsoftazuremanagementservicebus)

Все эти пакеты поддерживают операции создания, получения, перечисления, удаления, обновления, удаления и обновления **очередей, разделов и подписок**. Но только [Microsoft. Azure. Management. servicebus](#microsoftazuremanagementservicebus) поддерживает операции создания, обновления, перечисления, получения и удаления для **пространств имен**, перечисление и повторное создание ключей SAS и многое другое. 

Библиотека Microsoft. Azure. Management. ServiceBus работает только с проверкой подлинности Azure Active Directory (Azure AD) и не поддерживает использование строки подключения. В то время как другие две библиотеки (Azure. Messaging. ServiceBus и Microsoft. Azure. ServiceBus) поддерживают использование строки подключения для проверки подлинности в службе и проще в использовании. Между этими библиотеками Azure. Messaging. ServiceBus является последней и мы рекомендуем использовать.

В следующих разделах содержатся дополнительные сведения об этих библиотеках. 

## <a name="azuremessagingservicebusadministration"></a>Azure. Messaging. ServiceBus. Administration
Класс [сервицебусадминистратионклиент](/dotnet/api/azure.messaging.servicebus.administration.servicebusadministrationclient) можно использовать в пространстве имен [Azure. Messaging. servicebus. admin](/dotnet/api/azure.messaging.servicebus.administration) для управления пространствами имен, очередями, разделами и подписками. Ниже приведен пример кода. Полный пример см. в разделе [Пример CRUD](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/servicebus/Azure.Messaging.ServiceBus/tests/Samples/Sample07_CrudOperations.cs).

```csharp
using System;
using System.Threading.Tasks;

using Azure.Messaging.ServiceBus.Administration;

namespace adminClientTrack2
{
    class Program
    {
        public static void Main()
        {
            MainAsync().GetAwaiter().GetResult();
        }

        private static async Task MainAsync()
        {
            string connectionString = "SERVICE BUS NAMESPACE CONNECTION STRING";
            string QueueName = "QUEUE NAME";
            string TopicName = "TOPIC NAME";
            string SubscriptionName = "SUBSCRIPTION NAME";

            var adminClient = new ServiceBusAdministrationClient(connectionString);
            bool queueExists = await adminClient.QueueExistsAsync(QueueName);
            if (!queueExists)
            {
                var options = new CreateQueueOptions(QueueName)
                {
                    MaxDeliveryCount = 3                    
                };
                await adminClient.CreateQueueAsync(options);
            }


            bool topicExists = await adminClient.TopicExistsAsync(TopicName);
            if (!topicExists)
            {
                var options = new CreateTopicOptions(TopicName)
                {
                    MaxSizeInMegabytes = 1024
                };
                await adminClient.CreateTopicAsync(options);
            }

            bool subscriptionExists = await adminClient.SubscriptionExistsAsync(TopicName, SubscriptionName);
            if (!subscriptionExists)
            {
                var options = new CreateSubscriptionOptions(TopicName, SubscriptionName)
                {
                    DefaultMessageTimeToLive = new TimeSpan(2, 0, 0, 0)
                };
                await adminClient.CreateSubscriptionAsync(options);
            }
        }
    }
}

```


## <a name="microsoftazureservicebusmanagement"></a>Microsoft. Azure. ServiceBus. Management 
Класс [манажементклиент](/dotnet/api/microsoft.azure.servicebus.management.managementclient) в пространстве имен [Microsoft. Azure. servicebus. Management](/dotnet/api/microsoft.azure.servicebus.management) можно использовать для управления пространствами имен, очередями, разделами и подписками. Вот пример кода: 

> [!NOTE]
> Рекомендуется использовать `ServiceBusAdministrationClient` класс из `Azure.Messaging.ServiceBus.Administration` библиотеки, которая является последним пакетом SDK. Дополнительные сведения см. в [первом разделе](#azuremessagingservicebusadministration). 

```csharp
using System;
using System.Threading.Tasks;

using Microsoft.Azure.ServiceBus.Management;

namespace SBusManagementClient
{
    class Program
    {
        public static void Main()
        {
            MainAsync().GetAwaiter().GetResult();
        }

        private static async Task MainAsync()
        {
            string connectionString = "SERVICE BUS NAMESPACE CONNECTION STRING";
            string QueueName = "QUEUE NAME";
            string TopicName = "TOPIC NAME";
            string SubscriptionName = "SUBSCRIPTION NAME";

            var managementClient = new ManagementClient(connectionString);
            bool queueExists = await managementClient.QueueExistsAsync(QueueName);
            if (!queueExists)
            {
                QueueDescription qd = new QueueDescription(QueueName);
                qd.MaxSizeInMB = 1024;
                qd.MaxDeliveryCount = 3;
                await managementClient.CreateQueueAsync(qd);
            }


            bool topicExists = await managementClient.TopicExistsAsync(TopicName);
            if (!topicExists)
            {
                TopicDescription td = new TopicDescription(TopicName);
                td.MaxSizeInMB = 1024;
                td.DefaultMessageTimeToLive = new TimeSpan(2, 0, 0, 0);
                await managementClient.CreateTopicAsync(td);
            }

            bool subscriptionExists = await managementClient.SubscriptionExistsAsync(TopicName, SubscriptionName);
            if (!subscriptionExists)
            {
                SubscriptionDescription sd = new SubscriptionDescription(TopicName, SubscriptionName);
                sd.DefaultMessageTimeToLive = new TimeSpan(2, 0, 0, 0);
                sd.MaxDeliveryCount = 3;
                await managementClient.CreateSubscriptionAsync(sd);
            }
        }
    }
}
```


## <a name="microsoftazuremanagementservicebus"></a>Microsoft.Azure.Management.ServiceBus. 
Эта библиотека является частью пакета SDK для плоскости управления на основе Azure Resource Manager. 

### <a name="prerequisites"></a>Предварительные условия

Чтобы приступить к работе с этой библиотекой, необходимо пройти проверку подлинности с помощью службы Azure Active Directory (Azure AD). Azure AD требует выполнять аутентификацию в качестве субъекта-службы, который предоставляет доступ к вашим ресурсам Azure. Сведения о создании субъекта-службы см. в одной из приведенных ниже статей:  

* [Используйте портал Azure, чтобы создать Active Directory приложения и субъекта-службы, которые могут получать доступ к ресурсам.](../active-directory/develop/howto-create-service-principal-portal.md)
* [Использование Azure PowerShell для создания субъекта-службы и доступа к ресурсам](../active-directory/develop/howto-authenticate-service-principal-powershell.md)
* [Использование интерфейса командной строки Azure для создания субъекта-службы и доступа к ресурсам](/cli/azure/create-an-azure-service-principal-azure-cli)

В этих руководствах вы получите `AppId` (идентификатор клиента), `TenantId` и `ClientSecret` (ключ аутентификации), которые используются библиотеками управления для аутентификации. Необходимо иметь по крайней мере разрешения владельца данных или [**участника**](../role-based-access-control/built-in-roles.md#contributor) [**служебной шины Azure**](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner) для группы ресурсов, которую вы хотите запустить.

### <a name="programming-pattern"></a>Шаблон программирования

Шаблон обработки любого ресурса служебной шины соответствует общему протоколу.

1. Получите маркер из Azure AD с помощью библиотеки **Microsoft.IdentityModel.Clients.ActiveDirectory**:
   ```csharp
   var context = new AuthenticationContext($"https://login.microsoftonline.com/{tenantId}");

   var result = await context.AcquireTokenAsync("https://management.azure.com/", new ClientCredential(clientId, clientSecret));
   ```
2. Создайте объект `ServiceBusManagementClient`:

   ```csharp
   var creds = new TokenCredentials(token);
   var sbClient = new ServiceBusManagementClient(creds)
   {
       SubscriptionId = SettingsCache["SubscriptionId"]
   };
   ```
3. Присвойте параметрам `CreateOrUpdate` указанные значения:

   ```csharp
   var queueParams = new QueueCreateOrUpdateParameters()
   {
       Location = SettingsCache["DataCenterLocation"],
       EnablePartitioning = true
   };
   ```
4. Выполните вызов:

   ```csharp
   await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, QueueName, queueParams);
   ```

### <a name="complete-code-to-create-a-queue"></a>Завершение кода для создания очереди
Ниже приведен пример кода для создания очереди служебной шины. Полный пример см. в [примере управления .NET на сайте GitHub](https://github.com/Azure-Samples/service-bus-dotnet-management/). 

```csharp
using System;
using System.Threading.Tasks;

using Microsoft.Azure.Management.ServiceBus;
using Microsoft.Azure.Management.ServiceBus.Models;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;

namespace SBusADApp
{
    class Program
    {
        static void Main(string[] args)
        {
            CreateQueue().GetAwaiter().GetResult();
        }

        private static async Task CreateQueue()
        {
            try
            {
                var subscriptionID = "<SUBSCRIPTION ID>";
                var resourceGroupName = "<RESOURCE GROUP NAME>";
                var namespaceName = "<SERVICE BUS NAMESPACE NAME>";
                var queueName = "<NAME OF QUEUE YOU WANT TO CREATE>";

                var token = await GetToken();

                var creds = new TokenCredentials(token);
                var sbClient = new ServiceBusManagementClient(creds)
                {
                    SubscriptionId = subscriptionID,
                };

                var queueParams = new SBQueue();

                Console.WriteLine("Creating queue...");
                await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, queueName, queueParams);
                Console.WriteLine("Created queue successfully.");
            }
            catch (Exception e)
            {
                Console.WriteLine("Could not create a queue...");
                Console.WriteLine(e.Message);
                throw e;
            }
        }

        private static async Task<string> GetToken()
        {
            try
            {
                var tenantId = "<AZURE AD TENANT ID>";
                var clientId = "<APPLICATION/CLIENT ID>";
                var clientSecret = "<CLIENT SECRET>";

                var context = new AuthenticationContext($"https://login.microsoftonline.com/{tenantId}");

                var result = await context.AcquireTokenAsync(
                    "https://management.azure.com/",
                    new ClientCredential(clientId, clientSecret)
                );

                // If the token isn't a valid string, throw an error.
                if (string.IsNullOrEmpty(result.AccessToken))
                {
                    throw new Exception("Token result is empty!");
                }

                return result.AccessToken;
            }
            catch (Exception e)
            {
                Console.WriteLine("Could not get a token...");
                Console.WriteLine(e.Message);
                throw e;
            }
        }

    }
}
```

## <a name="fluent-library"></a>Библиотека Fluent
Пример использования библиотеки Fluent для управления сущностями служебной шины см. в [этом примере](https://github.com/Azure/azure-libraries-for-net/tree/master/Samples/ServiceBus). 

## <a name="next-steps"></a>Дальнейшие действия
См. следующие разделы справки: 

- [Azure. Messaging. ServiceBus. Administration](/dotnet/api/azure.messaging.servicebus.administration.servicebusadministrationclient)
- [Microsoft. Azure. ServiceBus. Management](/dotnet/api/microsoft.azure.servicebus.management.managementclient)
- [Microsoft.Azure.Management.ServiceBus.](/dotnet/api/microsoft.azure.management.servicebus.servicebusmanagementclient)
- [Fluent](/dotnet/api/microsoft.azure.management.servicebus.fluent)
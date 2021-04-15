---
title: Создание учетной записи Purview с помощью пакета SDK для .NET
description: Создание учетной записи Azure Purview с помощью пакета SDK для .NET
author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 4/2/2021
ms.author: nayenama
ms.openlocfilehash: 04ed5cef351c81355a2390dd0b983c162f2b9532
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/05/2021
ms.locfileid: "106387244"
---
# <a name="quickstart-create-a-purview-account-using-net-sdk"></a>Краткое руководство. Создание учетной записи Purview с помощью пакета SDK для .NET

В этом кратком руководстве описано, как создать учетную запись Azure Purview с помощью пакета SDK для .NET. 

> [!IMPORTANT]
> Сейчас предоставляется предварительная версия Azure Purview. [Дополнительные условия использования Azure для предварительных версий в Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) содержат дополнительные юридические условия, применимые к функциям Azure, которые предоставляются в бета-версии, предварительной версии или еще не выпущены в общедоступной версии по другим причинам.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

* Собственный [клиент Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

* Ваша учетная запись должна иметь разрешение на создание ресурсов в подписке.

* Если используется **политика Azure**, которая запрещает всем приложениям создавать **учетные записи хранения** и **пространства имен Центра событий**, следует создать исключение из этой политики с помощью тега, который можно указать при создании учетной записи Azure Purview. Основная причина заключается в том, что для каждой созданной учетной записи Azure Purview необходимо создать управляемую группу ресурсов, а в этой группе ресурсов создать учетную запись хранения и пространство имен EventHub. Дополнительные сведения см. в статье [Создание портала каталога](create-catalog-portal.md) .

### <a name="visual-studio"></a>Visual Studio

В этом пошаговом руководстве используется Visual Studio 2019. Процедуры в Visual Studio 2013, 2015 или 2017 незначительно отличаются.

### <a name="azure-net-sdk"></a>Пакет Azure SDK для .NET

Скачайте и установите [пакет Azure SDK для .NET](https://azure.microsoft.com/downloads/) на компьютер.

## <a name="create-an-application-in-azure-active-directory"></a>Создание приложения в Azure Active Directory

В разделе *Практическое руководство. Создание приложения Azure Active Directory и субъекта-службы с доступом к ресурсам с помощью портала* следуйте инструкциям по выполнению этих задач.

1. В разделе [Создание приложения Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) создайте приложение, которое представляет приложение .NET, создаваемое в этом руководстве. В качестве URL-адреса входа можно указать фиктивный URL-адрес, как показано в статье (`https://contoso.org/exampleapp`).
2. В разделе [Получение значений для входа](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) получите **код приложения**, а также **идентификатор арендатора** и запишите эти значения. Они потребуются в дальнейшем при выполнении инструкций этого руководства. 
3. В разделе [Сертификаты и секреты](../active-directory/develop/howto-create-service-principal-portal.md#authentication-two-options) получите **ключ аутентификации** и запишите это значение. Оно потребуется в дальнейшем при выполнении инструкций этого руководства.
4. В разделе [Назначение приложению роли](../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) назначьте приложению роль **Участник** на уровне подписки, чтобы приложение могло создавать фабрики данных в подписке.

## <a name="create-a-visual-studio-project"></a>Создание проекта Visual Studio

Далее создайте в Visual Studio консольное приложение C# .NET.

1. Запустите **Visual Studio**.
2. В окне запуска выберите **Создать новый проект** > **Консольное приложение (платформа .NET Framework)** . Требуется .NET версии 4.5.2 или более поздней.
3. В окне **Имя проекта** введите **ADFv2QuickStart**.
4. Выберите **Создать**, чтобы создать проект.

## <a name="install-nuget-packages"></a>Установка пакетов Nuget

1. Выберите **Инструменты** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.
2. На панели **Консоль диспетчера пакетов** выполните следующие команды, чтобы установить пакеты. Дополнительные сведения см. в документации по пакету NuGet [Microsoft.Azure.Management.Purview](https://www.nuget.org/packages/Microsoft.Azure.Management.Purview/).

    ```powershell
    Install-Package Microsoft.Azure.Management.Purview
    Install-Package Microsoft.Azure.Management.ResourceManager -IncludePrerelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-purview-client"></a>Создание клиента Purview

1. Откройте файл **Program.cs** и включите следующие инструкции, чтобы добавить ссылки на пространства имен.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Rest;
    using Microsoft.Rest.Serialization;
      using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.Purview;
      using Microsoft.Azure.Management.Purview.Models;
      using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

2. Добавьте следующий код в метод **Main**, который задает следующие переменные. Замените значения заполнителей на собственные. Чтобы получить список регионов Azure, в которых сейчас доступно Purview, выполните поиск по запросу **Azure Purview** и выберите интересующие вас регионы на странице [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/).

   ```csharp
   // Set variables
   string tenantID = "<your tenant ID>";
   string applicationId = "<your application ID>";
   string authenticationKey = "<your authentication key for the application>";
   string subscriptionId = "<your subscription ID where the data factory resides>";
   string resourceGroup = "<your resource group where the data factory resides>";
   string region = "<the location of your resource group>";
   string purviewAccountName = 
       "<specify the name of purview account to create. It must be globally unique.>";
   ```

3. Добавьте в метод **Main** приведенный ниже код, создающий экземпляр класса **PurviewManagementClient**. Этот объект используется для создания учетной записи Purview.

   ```csharp
   // Authenticate and create a purview management client
   var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
   ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
   AuthenticationResult result = context.AcquireTokenAsync(
   "https://management.azure.com/", cc).Result;
   ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
   var client = new PurviewManagementClient(cred)
   {
      SubscriptionId = subscriptionId           
   };
   ```

## <a name="create-a-purview-account"></a>Создание учетной записи Purview

Добавьте следующий код, создающий **учетную запись Purview**, в метод **Main**. 

```csharp
    // Create a purview Account
    Console.WriteLine("Creating Purview Account " + purviewAccountName + "...");
    Account account = new Account()
    {
    Location = region,
    Identity = new Identity(type: "SystemAssigned"),
    Sku = new AccountSku(name: "Standard", capacity: 4)
    };            
    try
    {
      client.Accounts.CreateOrUpdate(resourceGroup, purviewAccountName, account);
      Console.WriteLine(client.Accounts.Get(resourceGroup, purviewAccountName).ProvisioningState);                
    }
    catch (ErrorResponseModelException purviewException)
    {
      Console.WriteLine(purviewException.StackTrace);
    }
    Console.WriteLine(
      SafeJsonConvert.SerializeObject(account, client.SerializationSettings));
    while (client.Accounts.Get(resourceGroup, purviewAccountName).ProvisioningState ==
           "PendingCreation")
    {
      System.Threading.Thread.Sleep(1000);
    }
    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
```

## <a name="run-the-code"></a>Выполнение кода

Создайте и запустите приложение, а затем проверьте выполнение.

В консоли выводится ход создания учетной записи Purview.

### <a name="sample-output"></a>Пример выходных данных

```json
Creating Purview Account testpurview...
Succeeded
{
  "sku": {
    "capacity": 4,
    "name": "Standard"
  },
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "southcentralus"
}

Press any key to exit...
```

## <a name="verify-the-output"></a>Проверка выходных данных

Перейдите на страницу **Учетных записей Purview** на [портале Azure](https://portal.azure.com) и подтвердите созданную учетную запись с помощью кода выше. 

## <a name="delete-purview-account"></a>Удаление учетной записи Purview

Чтобы программно удалить учетную запись Purview, добавьте следующие строки кода в программу: 

```csharp
    Console.WriteLine("Deleting the Purview Account");
    client.Accounts.Delete(resourceGroup, purviewAccountName);
```

## <a name="check-if-purview-account-name-is-available"></a>Проверка доступности имени учетной записи Purview

Чтобы проверить доступность учетной записи Purview, используйте следующий код: 

```csharp
    CheckNameAvailabilityRequest checkNameAvailabilityRequest = new CheckNameAvailabilityRequest()
    {
      Name = purviewAccountName,
      Type =  "Microsoft.Purview/accounts"
    };
    Console.WriteLine("Check Purview account name");
    Console.WriteLine(client.Accounts.CheckNameAvailability(checkNameAvailabilityRequest).NameAvailable);
```

Приведенный выше код выведет true, если имя доступно, и false, если имя недоступно.


## <a name="next-steps"></a>Дальнейшие действия

Код, предоставленный в этом руководстве, позволяет создать или удалить учетную запись Purview, а также проверить доступность имени учетной записи. Теперь вы можете скачать пакет SDK для .NET и узнать о других действиях поставщика ресурсов, которые можно выполнить для учетной записи Purview.

Перейдите к следующей статье, чтобы узнать, как разрешить пользователям доступ к учетной записи Azure Purview. 

> [!div class="nextstepaction"]
> [Добавление пользователей в учетную запись Azure Purview](catalog-permissions.md)

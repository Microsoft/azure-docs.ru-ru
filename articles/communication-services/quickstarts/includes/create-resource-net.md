---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 443595a52458d7ff7c168f4c120257cfb60fad2e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105110901"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet/) бесплатно.
- Последняя версия [пакета SDK для .NET Core](https://dotnet.microsoft.com/download/dotnet-core) для вашей операционной системы.
- Получите последнюю версию [пакета SDK для .NET Identity](/dotnet/api/azure.identity).
- Получите последнюю версию [пакета SDK управления .NET](../../concepts/sdk-options.md).

## <a name="installing-the-sdk"></a>Установка пакета SDK

Во-первых, добавьте пакет SDK для управления Служб коммуникации в проект C#:

```csharp
using Azure.ResourceManager.Communication;
```

## <a name="subscription-id"></a>Идентификатор подписки

Вам потребуется идентификатор подписки Azure. Его можно получить на портале:

1.  Войдите в учетную запись Azure.
2.  Выберите "Подписки" на левой боковой панели.
3.  Выберите нужную подписку.
4.  Щелкните "Обзор".
5.  Выберите "Идентификатор подписки".

В этом кратком руководстве предполагается, что вы сохранили идентификатор подписки в переменной среды `AZURE_SUBSCRIPTION_ID`.

## <a name="authentication"></a>Аутентификация

Для взаимодействия со Службами коммуникации Azure необходимо сначала пройти проверку подлинности в Azure. Обычно для этого можно использовать удостоверение субъекта-службы.

### <a name="option-1-managed-identity"></a>Вариант 1. Управляемое удостоверение

Если код выполняется в качестве службы в Azure, самый простой способ пройти проверку подлинности — получить управляемое удостоверение Azure. Дополнительные сведения об [управляемых удостоверениях](../../../active-directory/managed-identities-azure-resources/overview.md).

[Службы Azure с поддержкой управляемых удостоверений](../../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

[Использование управляемых удостоверений в Службе приложений и Функциях Azure](../../../app-service/overview-managed-identity.md?tabs=dotnet)

#### <a name="system-assigned-managed-identity"></a>[Управляемое удостоверение, назначаемое системой](../../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity)

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = "AZURE_SUBSCRIPTION_ID";
var acsClient = new CommunicationManagementClient(subscriptionId, new ManagedIdentityCredential());
```

#### <a name="user-assigned-managed-identity"></a>[Управляемое удостоверение, назначаемое пользователем](../../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-user-assigned-identity)

ClientId созданного управляемого удостоверения должен быть явно передан `ManagedIdentityCredential`.

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = "AZURE_SUBSCRIPTION_ID";
var managedIdentityCredential = new ManagedIdentityCredential("AZURE_CLIENT_ID");
var acsClient = new CommunicationManagementClient(subscriptionId, managedIdentityCredential);
```

### <a name="option-2-service-principal"></a>Вариант 2. Субъект-служба

Вместо применения управляемого удостоверения проверку подлинности в Azure можно пройти, используя субъект-службу, которым вы самостоятельно управляете. Дополнительные сведения см. в документации по [созданию субъекта-службы в Azure Active Directory и управлении им](../../../active-directory/develop/howto-create-service-principal-portal.md).

После создания субъекта-службы необходимо собрать следующие сведения о нем на портале Azure.

- **Идентификатор клиента**
- **Секрет клиента**
- **Идентификатор клиента**

Сохраните эти значения в переменных среды `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` и `AZURE_TENANT_ID` соответственно. Затем вы можете создать клиент управления Служб коммуникации следующим образом.

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var acsClient = new CommunicationManagementClient(subscriptionId, new EnvironmentCredential());
```

### <a name="option-3-user-identity"></a>Способ 3. Удостоверение пользователя

Если вместо использования удостоверения службы вы хотите вызвать Azure от имени интерактивного пользователя, можно применить следующий код и создать клиент управления Служб коммуникации Azure. Откроется окно браузера, где пользователю необходимо будет ввести MSA или учетные данные Azure AD.

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var communicationServiceClient = new CommunicationManagementClient(subscriptionId, new InteractiveBrowserCredential());
```

## <a name="managing-communication-services-resources"></a>Управление ресурсами Служб коммуникации

### <a name="interacting-with-azure-resources"></a>Взаимодействие с ресурсами Azure

После проверки подлинности можно использовать клиент управления для вызова API.

Для каждого из приведенных ниже примеров мы будем назначать ресурсы Служб коммуникации имеющейся группе ресурсов.

Если вам необходимо создать группу ресурсов, можете использовать [портал Azure](../../../azure-resource-manager/management/manage-resource-groups-portal.md) или [пакет SDK для Azure Resource Manager](https://github.com/Azure/azure-sdk-for-net/blob/master/doc/mgmt_preview_quickstart.md).

### <a name="create-and-manage-a-communication-services-resource"></a>Создание ресурсов Служб коммуникации Azure и управление ими

Для выполнения операций с ресурсами Служб коммуникации можно использовать экземпляр клиента пакета SDK для управления Служб коммуникации (``Azure.ResourceManager.Communication.CommunicationManagementClient``).

#### <a name="create-a-communication-services-resource"></a>Создание ресурса Служб коммуникации

При создании ресурса Служб коммуникации необходимо указать имя группы ресурсов и имя ресурса. Обратите внимание, что значение свойства `Location` всегда будет `global`, а во время общедоступной предварительной версии значение `DataLocation` должно быть `UnitedStates`.

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var resource = new CommunicationServiceResource { Location = "Global", DataLocation = "UnitedStates"  };
var operation = await acsClient.CommunicationService.StartCreateOrUpdateAsync(resourceGroupName, resourceName, resource);
await operation.WaitForCompletionAsync();
```

#### <a name="update-a-communication-services-resource"></a>Обновление ресурса Служб коммуникации

```csharp
...
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var resource = new CommunicationServiceResource { Location = "Global", DataLocation = "UnitedStates" };
resource.Tags.Add("environment","test");
resource.Tags.Add("department","tech");
// Use existing resource name and new resource object
var operation = await acsClient.CommunicationService.StartCreateOrUpdateAsync(resourceGroupName, resourceName, resource);
await operation.WaitForCompletionAsync();
```

#### <a name="list-all-communication-services-resources"></a>Список всех ресурсов Служб коммуникации

```csharp
var resources = acsClient.CommunicationService.ListBySubscription();
foreach (var resource in resources)
{
    Console.WriteLine(resource.Name);
}
```

#### <a name="delete-a-communication-services-resource"></a>Удаление ресурса Служб коммуникации

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
await acsClient.CommunicationService.StartDeleteAsync(resourceGroupName, resourceName);
```

## <a name="managing-keys-and-connection-strings"></a>Управление ключами и строками подключения

Каждый ресурс Служб коммуникации имеет пару ключей доступа и соответствующие строки подключения. Доступ к этим ключам можно получить с помощью пакета SDK управления. Затем их можно использовать в других пакетах SDK Служб коммуникации для проверки подлинности в Службах коммуникации Azure.

#### <a name="get-access-keys-for-a-communication-services-resource"></a>Получение ключей доступа для ресурса Служб коммуникации

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var keys = await acsClient.CommunicationService.ListKeysAsync(resourceGroupName, resourceName);

Console.WriteLine(keys.Value.PrimaryConnectionString);
Console.WriteLine(keys.Value.SecondaryConnectionString);
```

#### <a name="regenerate-an-access-key-for-a-communication-services-resource"></a>Повторное создание ключа доступа для ресурса Служб коммуникации

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var keyParams = new RegenerateKeyParameters { KeyType = KeyType.Primary };
var keys = await acsClient.CommunicationService.RegenerateKeyAsync(resourceGroupName, resourceName, keyParams);

Console.WriteLine(keys.Value.PrimaryKey);
```
---
title: Создание SAS учетной записи с помощью .NET
titleSuffix: Azure Storage
description: Узнайте, как создать подписанный URL-адрес учетной записи (SAS) с помощью клиентской библиотеки .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/12/2020
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 2918b845430a6fc6dc59eca7041c114fc9d06515
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97092216"
---
# <a name="create-an-account-sas-with-net"></a>Создание SAS учетной записи с помощью .NET

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

В этой статье показано, как использовать ключ учетной записи хранения для создания SAS учетной записи с [клиентской библиотекой службы хранилища Azure для .NET](/dotnet/api/overview/azure/storage).

## <a name="create-an-account-sas"></a>Create an account SAS (Создание SAS на уровне учетной записи)

### <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

SAS учетной записи подписывается с помощью ключа доступа к учетной записи. Используйте класс [сторажешаредкэйкредентиал](/dotnet/api/azure.storage.storagesharedkeycredential) для создания учетных данных, которые используются для подписания SAS. Затем создайте новый объект [аккаунтсасбуилдер](/dotnet/api/azure.storage.sas.accountsasbuilder) и вызовите метод [тосаскуерипараметерс](/dotnet/api/azure.storage.sas.accountsasbuilder.tosasqueryparameters) , чтобы получить строку токена SAS.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_GetAccountSASToken":::

### <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

Чтобы создать SAS учетной записи для контейнера, вызовите метод [CloudStorageAccount. GetSharedAccessSignature](/dotnet/api/microsoft.azure.storage.cloudstorageaccount.getsharedaccesssignature) .

В следующем примере кода показано создание подписанного URL-адреса учетной записи, который действует для службы BLOB-объектов или службы файлов и предоставляет клиенту права на чтение, запись и получение списков через API-интерфейсы уровня службы. В SAS учетной записи указано ограничение на использование протоколов, поэтому запрос должен быть выполнен с помощью протокола HTTPS. Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```csharp
static string GetAccountSASToken()
{
    // To create the account SAS, you need to use Shared Key credentials. Modify for your account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=<storage-account>;AccountKey=<account-key>";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create a new access policy for the account.
    SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
        {
            Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
            Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
            ResourceTypes = SharedAccessAccountResourceTypes.Service,
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Protocols = SharedAccessProtocol.HttpsOnly
        };

    // Return the SAS token.
    return storageAccount.GetSharedAccessSignature(policy);
}
```

---

## <a name="use-an-account-sas-from-a-client"></a>Использование SAS учетной записи из клиента

Чтобы использовать SAS учетной записи для доступа к API уровня службы для службы BLOB-объектов, создайте объект клиента службы BLOB-объектов, используя SAS и конечную точку хранилища BLOB-объектов для вашей учетной записи хранения.

### <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_UseAccountSAS":::

### <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

В этом фрагменте кода замените `<storage-account>` заполнитель именем вашей учетной записи хранения.

```csharp
static void UseAccountSAS(string sasToken)
{
    // Create new storage credentials using the SAS token.
    StorageCredentials accountSAS = new StorageCredentials(sasToken);
    // Use these credentials and the account name to create a Blob service client.
    CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "<storage-account>", endpointSuffix: null, useHttps: true);
    CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

    // Now set the service properties for the Blob client created with the SAS.
    blobClientWithSAS.SetServiceProperties(new ServiceProperties()
    {
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        Logging = new LoggingProperties()
        {
            LoggingOperations = LoggingOperations.All,
            RetentionDays = 14,
            Version = "1.0"
        }
    });

    // The permissions granted by the account SAS also permit you to retrieve service properties.
    ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
}
```

---

## <a name="next-steps"></a>Дальнейшие действия

- [Предоставление ограниченного доступа к ресурсам службы хранилища Azure с помощью подписанных URL-адресов (SAS)](storage-sas-overview.md)
- [Create an account SAS](/rest/api/storageservices/create-account-sas) (Создание SAS на уровне учетной записи)

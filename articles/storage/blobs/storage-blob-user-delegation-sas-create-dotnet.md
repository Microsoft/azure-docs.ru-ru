---
title: Использование .NET для создания SAS делегирования пользователя для контейнера, каталога или большого двоичного объекта
titleSuffix: Azure Storage
description: Узнайте, как создать SAS делегирования пользователя с учетными данными Azure Active Directory с помощью клиентской библиотеки .NET для службы хранилища Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/03/2021
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.openlocfilehash: 13491735f73cb1696f3c36f3434cc781a1e2b739
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99526811"
---
# <a name="create-a-user-delegation-sas-for-a-container-directory-or-blob-with-net"></a>Создание SAS для делегирования пользователя для контейнера, каталога или большого двоичного объекта с помощью .NET

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

В этой статье показано, как использовать учетные данные Azure Active Directory (Azure AD) для создания SAS делегирования пользователя для контейнера, каталога или большого двоичного объекта с клиентской библиотекой службы хранилища Azure для .NET версии 12.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="assign-azure-roles-for-access-to-data"></a>Назначение ролей Azure для доступа к данным

Когда субъект безопасности Azure AD пытается получить доступ к данным большого двоичного объекта, этот субъект безопасности должен иметь разрешения для ресурса. Независимо от того, является ли участник безопасности управляемым удостоверением в Azure или учетной записью пользователя Azure AD, выполняющего код в среде разработки, участнику безопасности должна быть назначена роль Azure, которая предоставляет доступ к данным большого двоичного объекта в службе хранилища Azure. Сведения о назначении разрешений с помощью Azure RBAC см. в разделе **назначение ролей Azure для прав доступа** в статье [авторизация доступа к BLOB-объектам и очередям azure с помощью Azure Active Directory](../common/storage-auth-aad.md#assign-azure-roles-for-access-rights).

[!INCLUDE [storage-install-packages-blob-and-identity-include](../../../includes/storage-install-packages-blob-and-identity-include.md)]

Дополнительные сведения о проверке подлинности с помощью клиентской библиотеки удостоверений Azure из службы хранилища Azure см. в разделе **Проверка подлинности с помощью библиотеки удостоверений Azure** в статье [авторизация доступа к BLOB-объектам и очередям с Azure Active Directory и управляемыми удостоверениями для ресурсов Azure](../common/storage-auth-aad-msi.md?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json#authenticate-with-the-azure-identity-library).

## <a name="get-an-authenticated-token-credential"></a>Получение учетных данных маркера, прошедшего проверку подлинности

Чтобы получить учетные данные маркера, которые ваш код может использовать для авторизации запросов в службе хранилища Azure, создайте экземпляр класса [дефаултазурекредентиал](/dotnet/api/azure.identity.defaultazurecredential) .

В следующем фрагменте кода показано, как получить учетные данные маркера, прошедшего проверку подлинности, и использовать его для создания клиента службы для хранилища BLOB-объектов:

```csharp
// Construct the blob endpoint from the account name.
string blobEndpoint = string.Format("https://{0}.blob.core.windows.net", accountName);

// Create a new Blob service client with Azure AD credentials.
BlobServiceClient blobClient = new BlobServiceClient(new Uri(blobEndpoint),
                                                     new DefaultAzureCredential());
```

## <a name="get-the-user-delegation-key"></a>Получение ключа делегирования пользователя

Каждый SAS подписывается ключом. Чтобы создать SAS для делегирования пользователей, необходимо сначала запросить ключ делегирования пользователя, который затем используется для подписания SAS. Ключ делегирования пользователя аналогичен ключу учетной записи, используемому для подписания SAS службы или SAS учетной записи, за исключением того, что он использует учетные данные Azure AD. Когда клиент запрашивает ключ делегирования пользователя с помощью маркера OAuth 2,0, служба хранилища Azure Возвращает ключ делегирования пользователя от имени пользователя.

После получения ключа делегирования пользователя можно использовать этот ключ, чтобы создать любое количество подписанных URL для общего доступа пользователя в течение времени существования ключа. Ключ делегирования пользователя не зависит от токена OAuth 2,0, используемого для его получения, поэтому токен не нуждается в продлении, пока ключ остается действительным. Можно указать, что ключ действителен в течение периода до 7 дней.

Используйте один из следующих методов, чтобы запросить ключ делегирования пользователя:

- [жетусерделегатионкэй](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.getuserdelegationkey)
- [жетусерделегатионкэйасинк](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.getuserdelegationkeyasync)

Следующий фрагмент кода получает ключ делегирования пользователя и записывает его свойства:

```csharp
// Get a user delegation key for the Blob service that's valid for seven days.
// You can use the key to generate any number of shared access signatures over the lifetime of the key.
UserDelegationKey key = await blobClient.GetUserDelegationKeyAsync(DateTimeOffset.UtcNow,
                                                                   DateTimeOffset.UtcNow.AddDays(7));

// Read the key's properties.
Console.WriteLine("User delegation key properties:");
Console.WriteLine("Key signed start: {0}", key.SignedStartsOn);
Console.WriteLine("Key signed expiry: {0}", key.SignedExpiresOn);
Console.WriteLine("Key signed object ID: {0}", key.SignedObjectId);
Console.WriteLine("Key signed tenant ID: {0}", key.SignedTenantId);
Console.WriteLine("Key signed service: {0}", key.SignedService);
Console.WriteLine("Key signed version: {0}", key.SignedVersion);
```

## <a name="get-a-user-delegation-sas-for-a-blob"></a>Получение SAS для делегирования пользователя для большого двоичного объекта

В следующем примере кода показан полный код для проверки подлинности субъекта безопасности и создания SAS для делегирования пользователя для большого двоичного объекта:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_GetUserDelegationSasBlob":::

В следующем примере выполняется проверка SAS делегирования пользователя, созданного в предыдущем примере, из имитации клиентского приложения. Если SAS является допустимым, клиентское приложение может прочитать содержимое большого двоичного объекта. Если SAS является недопустимым, например, если срок его действия истек, служба хранилища Azure возвращает код ошибки 403 (запрещено).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_ReadBlobWithSasAsync":::

## <a name="get-a-user-delegation-sas-for-a-container"></a>Получение SAS для делегирования пользователя для контейнера

В следующем примере кода показано, как создать SAS делегирования пользователя для контейнера:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_GetUserDelegationSasContainer":::

В следующем примере выполняется проверка SAS делегирования пользователя, созданного в предыдущем примере, из имитации клиентского приложения. Если SAS является допустимым, клиентское приложение может прочитать содержимое большого двоичного объекта. Если SAS является недопустимым, например, если срок его действия истек, служба хранилища Azure возвращает код ошибки 403 (запрещено).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_ListBlobsWithSasAsync":::

## <a name="get-a-user-delegation-sas-for-a-directory"></a>Получение SAS пользователя о делегировании для каталога

В следующем примере кода показано, как создать SAS делегирования пользователя для каталога, если для учетной записи хранения включено иерархическое пространство имен:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_GetUserDelegationSasDirectory":::

В следующем примере выполняется проверка SAS делегирования пользователя, созданного в предыдущем примере, из имитации клиентского приложения. Если SAS является допустимым, клиентское приложение может перечислить пути к файлам для этого каталога. Если SAS является недопустимым, например, если срок его действия истек, служба хранилища Azure возвращает код ошибки 403 (запрещено).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Sas.cs" id="Snippet_ListFilePathsWithDirectorySasAsync":::

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>См. также раздел

- [Предоставление ограниченного доступа к ресурсам службы хранилища Azure с помощью подписанных URL-адресов (SAS)](../common/storage-sas-overview.md)
- [Операция получения ключа делегирования пользователя](/rest/api/storageservices/get-user-delegation-key)
- [Создание SAS для делегирования пользователей (REST API)](/rest/api/storageservices/create-user-delegation-sas)

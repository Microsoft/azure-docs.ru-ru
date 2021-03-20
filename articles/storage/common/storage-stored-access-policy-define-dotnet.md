---
title: Создание хранимой политики доступа с помощью .NET
titleSuffix: Azure Storage
description: Используйте службу хранилища Azure и .NET для создания хранимой политики доступа. Проведите дополнительные уровни контроля над подписанными на уровне службы подписями общего доступа на сервере.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 6cd6147fbe38710bcefd580e71be1d6f5d446a21
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89010755"
---
# <a name="create-a-stored-access-policy-with-net"></a>Создание хранимой политики доступа с помощью .NET

Хранимая политика доступа предоставляет дополнительный уровень контроля над ПОДПИСАНными URL-адресами уровня службы на стороне сервера. Определение хранимой политики доступа служит для группировки подписанных URL-групп и предоставления дополнительных ограничений для подписанных URL-подписей, привязанных к политике. Хранимую политику доступа можно использовать для изменения времени запуска, времени окончания срока действия или разрешений для SAS, а также для отмены ее после ее выдачи.
  
Следующие ресурсы службы хранилища Azure поддерживают хранимые политики доступа:  
  
- контейнеров BLOB-объектов;  
- общих папок;  
- очередей;  
- Таблицы  
  
> [!NOTE]
> Хранимую политику доступа в контейнере можно связать с подписанным URL-путем, предоставив разрешения для самого контейнера или содержащихся в нем больших двоичных объектов. Аналогичным образом хранимая политика доступа в общей папке может быть связана с подписанным URL-путем, предоставляющим разрешения для самой общей папки или содержащихся в ней файлов.  
>
> Хранимые политики доступа поддерживаются только для SAS службы. Хранимые политики доступа не поддерживаются для SAS учетной записи или SAS делегирования пользователя.  

Дополнительные сведения о хранимых политиках доступа см. [в разделе Определение хранимой политики доступа](/rest/api/storageservices/define-stored-access-policy).

## <a name="create-a-stored-access-policy"></a>Создание хранимой политики доступа

Базовая операция RESTFUL для создания хранимой политики доступа — [Установка ACL контейнера](/rest/api/storageservices/set-container-acl). Необходимо авторизовать операцию для создания хранимой политики доступа с помощью общего ключа, используя ключи доступа к учетной записи в строке подключения. Авторизация операции **задания ACL контейнеров** с учетными данными Azure AD не поддерживается. Дополнительные сведения см. в разделе [разрешения на вызов операций с данными большого двоичного объекта и очереди](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations).

В следующих примерах кода создается хранимая политика доступа для контейнера. Политики доступа можно использовать для указания ограничений для SAS службы, накладываемые на контейнер или BLOB-объекты.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Чтобы создать хранимую политику доступа для контейнера с клиентской библиотекой .NET версии 12 для службы хранилища Azure, вызовите один из следующих методов.

- [Блобконтаинерклиент. Сетакцессполици](/dotnet/api/azure.storage.blobs.blobcontainerclient.setaccesspolicy)
- [Блобконтаинерклиент. Сетакцессполициасинк](/dotnet/api/azure.storage.blobs.blobcontainerclient.setaccesspolicyasync)

В следующем примере создается хранимая политика доступа, которая действует в течение одного дня и предоставляет разрешения на чтение и запись:

```csharp
async static Task CreateStoredAccessPolicyAsync(string containerName)
{
    string connectionString = "";

    // Use the connection string to authorize the operation to create the access policy.
    // Azure AD does not support the Set Container ACL operation that creates the policy.
    BlobContainerClient containerClient = new BlobContainerClient(connectionString, containerName);

    try
    {
        await containerClient.CreateIfNotExistsAsync();

        // Create one or more stored access policies.
        List<BlobSignedIdentifier> signedIdentifiers = new List<BlobSignedIdentifier>
        {
            new BlobSignedIdentifier
            {
                Id = "mysignedidentifier",
                AccessPolicy = new BlobAccessPolicy
                {
                    StartsOn = DateTimeOffset.UtcNow.AddHours(-1),
                    ExpiresOn = DateTimeOffset.UtcNow.AddDays(1),
                    Permissions = "rw"
                }
            }
        };
        // Set the container's access policy.
        await containerClient.SetAccessPolicyAsync(permissions: signedIdentifiers);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.ErrorCode);
        Console.WriteLine(e.Message);
    }
    finally
    {
        await containerClient.DeleteAsync();
    }
}
```

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Чтобы создать хранимую политику доступа для контейнера с клиентской библиотекой .NET версии 12 для службы хранилища Azure, вызовите один из следующих методов.

- [CloudBlobContainer. SetPermissions](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setpermissions)
- [CloudBlobContainer. Сетпермиссионсасинк](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.setpermissionsasync)

В следующем примере создается хранимая политика доступа, которая действует в течение одного дня и предоставляет разрешения на чтение, запись и перечисление.

```csharp
private static async Task CreateStoredAccessPolicyAsync(CloudBlobContainer container, string policyName)
{
    // Create a new stored access policy and define its constraints.
    // The access policy provides create, write, read, list, and delete permissions.
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
    {
        // When the start time for the SAS is omitted, the start time is assumed to be the time when Azure Storage receives the request.
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
        Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List |
            SharedAccessBlobPermissions.Write
    };

    // Get the container's existing permissions.
    BlobContainerPermissions permissions = await container.GetPermissionsAsync();

    // Add the new policy to the container's permissions, and set the container's permissions.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    await container.SetPermissionsAsync(permissions);
}
```

---

## <a name="see-also"></a>См. также раздел

- [Предоставление ограниченного доступа к ресурсам службы хранилища Azure с помощью подписанных URL-адресов (SAS)](storage-sas-overview.md)
- [Определение хранимой политики доступа](/rest/api/storageservices/define-stored-access-policy)
- [Настройка строк подключения службы хранилища Azure](storage-configure-connection-string.md)

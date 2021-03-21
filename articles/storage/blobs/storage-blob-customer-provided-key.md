---
title: Указание предоставленного клиентом ключа для запроса к хранилищу BLOB-объектов с помощью .NET в службе хранилища Azure
description: Узнайте, как указать предоставленный клиентом ключ для запроса к хранилищу BLOB-объектов с помощью .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: c3096da8b3c83dbfe8cfdd6a5fa4d177241334de
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97693517"
---
# <a name="specify-a-customer-provided-key-on-a-request-to-blob-storage-with-net"></a>Указание предоставленного клиентом ключа для запроса к хранилищу BLOB-объектов с помощью .NET

Клиенты, выполняющие запросы к хранилищу больших двоичных объектов Azure, имеют возможность предоставить ключ шифрования AES-256 для отдельного запроса. Включение ключа шифрования в запрос обеспечивает детальный контроль над параметрами шифрования для операций с хранилищем BLOB-объектов. Предоставляемые клиентом ключи могут храниться в Azure Key Vault или в другом хранилище ключей.

В этой статье показано, как указать предоставленный клиентом ключ для запроса с помощью .NET.

[!INCLUDE [storage-install-packages-blob-and-identity-include](../../../includes/storage-install-packages-blob-and-identity-include.md)]

Дополнительные сведения о проверке подлинности с помощью клиентской библиотеки удостоверений Azure см. в статье [Клиентская библиотека удостоверений Azure для .NET](/dotnet/api/overview/azure/identity-readme).

## <a name="use-a-customer-provided-key-to-write-to-a-blob"></a>Использование предоставленного клиентом ключа для записи в большой двоичный объект

В следующем примере показан ключ AES-256 при отправке большого двоичного объекта с клиентской библиотекой версии 12 для хранилища BLOB-объектов. В примере используется объект [дефаултазурекредентиал](/dotnet/api/azure.identity.defaultazurecredential) для авторизации запроса на запись в Azure AD, но можно также авторизовать запрос с учетными данными общего ключа.

```csharp
async static Task UploadBlobWithClientKey(Uri blobUri,
                                          Stream data,
                                          byte[] key,
                                          string keySha256)
{
    // Create a new customer-provided key.
    // Key must be AES-256.
    var cpk = new CustomerProvidedKey(key);

    // Check the key's encryption hash.
    if (cpk.EncryptionKeyHash != keySha256)
    {
        throw new InvalidOperationException("The encryption key is corrupted.");
    }

    // Specify the customer-provided key on the options for the client.
    BlobClientOptions options = new BlobClientOptions()
    {
        CustomerProvidedKey = cpk
    };

    // Create the client object with options specified.
    BlobClient blobClient = new BlobClient(
        blobUri,
        new DefaultAzureCredential(),
        options);

    // If the container may not exist yet,
    // create a client object for the container.
    // The container client retains the credential and client options.
    BlobContainerClient containerClient =
        blobClient.GetParentBlobContainerClient();

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload the data using the customer-provided key.
        await blobClient.UploadAsync(data);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Укажите ключ шифрования для запроса к хранилищу BLOB-объектов](encryption-customer-provided-keys.md)
- [Шифрование службы хранилища Azure для неактивных данных](../common/storage-service-encryption.md)

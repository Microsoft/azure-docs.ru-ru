---
title: Получение имени типа учетной записи хранения и номера SKU с помощью .NET
titleSuffix: Azure Storage
description: Узнайте, как получить имя типа учетной записи хранения Azure и номера SKU с помощью клиентской библиотеки .NET.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 11/12/2020
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 0fd693573858df095b62a7a7917563141ac19c5b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94579340"
---
# <a name="get-storage-account-type-and-sku-name-with-net"></a>Получение имени типа учетной записи хранения и номера SKU с помощью .NET

В этой статье показано, как получить имя типа учетной записи хранения Azure и номер SKU для большого двоичного объекта с помощью [клиентской библиотеки службы хранилища Azure для .NET](/dotnet/api/overview/azure/storage).

## <a name="about-account-type-and-sku-name"></a>Сведения о типе учетной записи и имени SKU

**Тип учетной записи**: допустимые типы учетных записей включают `BlobStorage` , `BlockBlobStorage` ,, `FileStorage` `Storage` и `StorageV2` . В разделе [Общие сведения об учетной записи хранения Azure](storage-account-overview.md) содержится дополнительная информация, включая описания различных учетных записей хранения.

**Имя SKU**: допустимые имена SKU включают `Premium_LRS` , `Premium_ZRS` , `Standard_GRS` , `Standard_GZRS` , `Standard_LRS` ,, и `Standard_RAGRS` `Standard_RAGZRS` `Standard_ZRS` . Имена SKU учитывают регистр и являются строковыми полями в [классе SkuName](/dotnet/api/microsoft.azure.management.storage.models.skuname).

## <a name="retrieve-account-information"></a>Получение сведений об учетной записи

В следующем примере кода извлекаются и отображаются свойства учетной записи только для чтения.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Чтобы получить тип учетной записи хранения и имя SKU, связанные с большим двоичным объектом, вызовите метод [жетаккаунтинфо](/dotnet/api/azure.storage.blobs.blobserviceclient.getaccountinfo) или [жетаккаунтинфоасинк](/dotnet/api/azure.storage.blobs.blobserviceclient.getaccountinfoasync) .

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Account.cs" id="Snippet_GetAccountInfo":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Чтобы получить тип учетной записи хранения и имя SKU, связанные с большим двоичным объектом, вызовите метод [жетаккаунтпропертиес](/dotnet/api/microsoft.azure.storage.blob.cloudblob.getaccountproperties) или [жетаккаунтпропертиесасинк](/dotnet/api/microsoft.azure.storage.blob.cloudblob.getaccountpropertiesasync) .

```csharp
private static async Task GetAccountInfoAsync(CloudBlob blob)
{
    try
    {
        // Get the blob's storage account properties.
        AccountProperties acctProps = await blob.GetAccountPropertiesAsync();

        // Display the properties.
        Console.WriteLine("Account properties");
        Console.WriteLine("  AccountKind: {0}", acctProps.AccountKind);
        Console.WriteLine("      SkuName: {0}", acctProps.SkuName);
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Узнайте о других операциях, которые можно выполнять с учетной записью хранения, используя [портал Azure](https://portal.azure.com) и REST API Azure.

- [Операция получения сведений об учетной записи (ОСТАВШАЯся)](/rest/api/storageservices/get-account-information)

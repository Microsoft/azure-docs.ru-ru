---
title: Настройка протокола TLS для клиентского приложения
titleSuffix: Azure Storage
description: Настройте клиентское приложение для взаимодействия с хранилищем Azure, используя минимальную версию протокола TLS.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/08/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 37b8c79df5b208feea185292fa09c323b64fa27d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89001813"
---
# <a name="configure-transport-layer-security-tls-for-a-client-application"></a>Настройка протокола TLS для клиентского приложения

В целях безопасности учетной записи хранения Azure может потребоваться, чтобы клиенты использовали для отправки запросов минимальную версию протокола TLS. Вызовы службы хранилища Azure завершатся ошибкой, если клиент использует версию TLS, которая ниже минимальной требуемой версии. Например, если для учетной записи хранения требуется TLS 1,2, запрос, отправленный клиентом, который использует TLS 1,1, завершится ошибкой.

В этой статье описывается, как настроить клиентское приложение для использования определенной версии TLS. Сведения о том, как настроить минимально необходимую версию TLS для учетной записи хранения Azure, см. [в статье Настройка минимальной требуемой версии протокола TLS для учетной записи хранения](transport-layer-security-configure-minimum-version.md).

## <a name="configure-the-client-tls-version"></a>Настройка версии TLS клиента

Чтобы клиент мог отправить запрос с определенной версией протокола TLS, операционная система должна поддерживать эту версию.

В следующих примерах показано, как задать версию TLS клиента 1,2 из PowerShell или .NET. Платформа .NET Framework, используемый клиентом, должен поддерживать TLS 1,2. Дополнительные сведения см. в разделе [поддержка TLS 1,2](/dotnet/framework/network-programming/tls#support-for-tls-12).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как включить TLS 1,2 в клиенте PowerShell.

```powershell
# Set the TLS version used by the PowerShell client to TLS 1.2.
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;

# Create a new container.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context
New-AzStorageContainer -Name "sample-container" -Context $ctx
```

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

В следующем примере показано, как включить TLS 1,2 в клиенте .NET с помощью клиентской библиотеки службы хранилища Azure версии 12:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Networking.cs" id="Snippet_ConfigureTls12":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

В следующем примере показано, как включить TLS 1,2 в клиенте .NET с помощью клиентской библиотеки службы хранилища Azure версии 11:

```csharp
static void EnableTls12()
{
    // Enable TLS 1.2 before connecting to Azure Storage
    System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;

    // Add your connection string here.
    string connectionString = "";

    // Connect to Azure Storage and create a new container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
    container.CreateIfNotExists();
}
```

---

## <a name="verify-the-tls-version-used-by-a-client"></a>Проверка версии TLS, используемой клиентом

Чтобы убедиться, что клиент использовал указанную версию протокола TLS для отправки запроса, можно использовать [Fiddler](https://www.telerik.com/fiddler) или аналогичное средство. Откройте Fiddler, чтобы начать запись сетевого трафика клиента, а затем выполните один из примеров из предыдущего раздела. Просмотрите трассировку Fiddler, чтобы убедиться, что для отправки запроса использовалась правильная версия протокола TLS, как показано на следующем рисунке.

:::image type="content" source="media/transport-layer-security-configure-client-version/fiddler-trace-tls-version.png" alt-text="Снимок экрана, на котором показана трассировка Fiddler, указывающая версию TLS, используемую в запросе":::

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка минимальной требуемой версии протокола TLS для учетной записи хранения](transport-layer-security-configure-minimum-version.md)
- [Рекомендации по обеспечению безопасности для хранилища BLOB-объектов](../blobs/security-recommendations.md)

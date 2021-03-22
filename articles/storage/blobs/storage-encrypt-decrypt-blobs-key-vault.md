---
title: Руководство. Шифрование и расшифровка больших двоичных объектов с помощью Azure Key Vault
titleSuffix: Azure Storage
description: Узнайте, как зашифровать и расшифровать большой двоичный объект путем шифрования на стороне клиента с помощью Azure Key Vault.
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 02/18/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: c2daed4a8df89ed176749900dc75eb231c00af87
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102049276"
---
# <a name="tutorial---encrypt-and-decrypt-blobs-using-azure-key-vault"></a>Руководство. Шифрование и расшифровка больших двоичных объектов с помощью Azure Key Vault

В этом учебнике описывается, как использовать хранилище ключей Azure для шифрования хранилища на стороне клиента. Учебник включает пошаговое руководство по шифрованию и расшифровке большого двоичного объекта в консольном приложении с помощью этих технологий.

**Предполагаемое время выполнения:** 20 минут

Общие сведения о хранилище ключей Azure см. в статье [Что такое хранилище ключей Azure?](../../key-vault/general/overview.md)

Общие сведения о шифровании на стороне клиента для службы хранилища Azure см. в статье [Шифрование на стороне клиента для службы хранилища Microsoft Azure](../common/storage-client-side-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником необходимо наличие следующих компонентов.

* Учетная запись хранения Azure
* Visual Studio 2013 или более поздней версии
* Azure PowerShell

## <a name="overview-of-client-side-encryption"></a>Обзор шифрования на стороне клиента

Общие сведения о шифровании на стороне клиента для службы хранилища Azure см. в статье [Шифрование на стороне клиента для службы хранилища Microsoft Azure](../common/storage-client-side-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

Вот краткое описание того, как работает шифрование на стороне клиента.

1. Пакет SDK клиента хранилища Azure создает ключ шифрования содержимого (CEK), который является одноразовым симметричным ключом.
2. Данные пользователей шифруются с помощью этого ключа CEK.
3. Ключ CEK, в свою очередь, шифруется с помощью ключа шифрования ключа KEK. Ключ KEK определяется идентификатором ключа и может быть парой асимметричных ключей или симметричным ключом. Вы можете управлять им локально или хранить его в хранилище ключей Azure. У самого клиента хранилища нет доступа к KEK. Он просто вызывает алгоритм шифрования ключа, предоставляемый хранилищем ключей. Пользователи могут при необходимости использовать настраиваемые поставщики для шифрования и расшифровки ключа.
4. Зашифрованные данные затем передаются в службу хранилища Azure.

## <a name="set-up-your-azure-key-vault"></a>Настройка хранилища ключей Azure

Чтобы продолжить работу с этим руководством, необходимо выполнить действия, описанные в статье [Краткое руководство. Использование клиентской библиотеки секретов Azure Key Vault для .NET (пакет SDK версии 4)](../../key-vault/secrets/quick-create-net.md):

* Создать хранилище ключей.
* Добавить ключ или секрет в хранилище ключей.
* Зарегистрируйте приложение в Azure Active Directory.
* Разрешить приложению использовать ключ или секрет.

Запишите ClientID и ClientSecret, сформированные при регистрации приложения в Azure Active Directory.

Создайте оба ключа в хранилище ключей. В этом учебнике подразумевается, что вы использовали следующие имена: ContosoKeyVault и TestRSAKey1.

## <a name="create-a-console-application-with-packages-and-appsettings"></a>Создание консольного приложения с пакетами и AppSettings

Создайте новое консольное приложение в Visual Studio.

Добавьте необходимые пакеты NuGet в консоли диспетчера пакетов.

```powershell
Install-Package Microsoft.Azure.ConfigurationManager
Install-Package Microsoft.Azure.Storage.Common
Install-Package Microsoft.Azure.Storage.Blob
Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory

Install-Package Microsoft.Azure.KeyVault
Install-Package Microsoft.Azure.KeyVault.Extensions
```

Добавьте AppSettings в файл App.Config.

```xml
<appSettings>
    <add key="accountName" value="myaccount"/>
    <add key="accountKey" value="theaccountkey"/>
    <add key="clientId" value="theclientid"/>
    <add key="clientSecret" value="theclientsecret"/>
    <add key="container" value="stuff"/>
</appSettings>
```

Добавьте следующие директивы `using` и обязательно добавьте в проект ссылку на System.Configuration.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Configuration;
using Microsoft.Azure;
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Auth;
using Microsoft.Azure.Storage.Blob;
using Microsoft.Azure.KeyVault;
using System.Threading;
using System.IO;
```
---

## <a name="add-a-method-to-get-a-token-to-your-console-application"></a>Добавление метода получения токена в консольное приложение

Следующий метод используется классами хранилища ключей, когда необходимо пройти проверку подлинности для доступа к вашему хранилищу ключей.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
private async static Task<string> GetToken(string authority, string resource, string scope)
{
    var authContext = new AuthenticationContext(authority);
    ClientCredential clientCred = new ClientCredential(
        CloudConfigurationManager.GetSetting("clientId"),
        CloudConfigurationManager.GetSetting("clientSecret"));
    AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

    if (result == null)
        throw new InvalidOperationException("Failed to obtain the JWT token");

    return result.AccessToken;
}
```
---

## <a name="access-azure-storage-and-key-vault-in-your-program"></a>Получение доступа к хранилищу данных Azure и Key Vault в программе

В методе main() добавьте следующий код.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
// This is standard code to interact with Blob storage.
StorageCredentials creds = new StorageCredentials(
    CloudConfigurationManager.GetSetting("accountName"),
    CloudConfigurationManager.GetSetting("accountKey")
);
CloudStorageAccount account = new CloudStorageAccount(creds, useHttps: true);
CloudBlobClient client = account.CreateCloudBlobClient();
CloudBlobContainer contain = client.GetContainerReference(CloudConfigurationManager.GetSetting("container"));
contain.CreateIfNotExists();

// The Resolver object is used to interact with Key Vault for Azure Storage.
// This is where the GetToken method from above is used.
KeyVaultKeyResolver cloudResolver = new KeyVaultKeyResolver(GetToken);
```
---

> [!NOTE]
> Объектные модели хранилища ключей
> 
> Важно понимать, что фактически существуют две объектные модели хранилища ключей, которые необходимо принимать во внимание: одна основана на API REST (пространство имен KeyVault), а другая представляет собой расширение для шифрования на стороне клиента.
> 
> Клиент хранилища ключей взаимодействует с REST API и понимает веб-ключи и секреты JSON для двух видов элементов, содержащихся в хранилище ключей.
> 
> Расширения хранилища ключей — это классы, которые специально созданы для шифрования на стороне клиента в хранилище Azure. Они содержат интерфейс для ключей (IKey) и классов, основанный на концепции сопоставителя ключей. Существуют две реализации IKey, которые необходимо знать, — RSAKey и SymmetricKey. Они совпадают с элементами, которые находятся в хранилище ключей, но на данный момент остаются независимыми классами (таким образом, ключ и секретный код, получаемые клиентом хранилища ключей, не реализуют IKey).
> 
> 

## <a name="encrypt-blob-and-upload"></a>Шифрование и передача BLOB-объекта

Добавьте следующий код, чтобы зашифровать BLOB-объект и передать его в свою учетную запись хранения Azure. Затем используется метод **ResolveKeyAsync**, который возвращает IKey.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
// Retrieve the key that you created previously.
// The IKey that is returned here is an RsaKey.
var rsa = cloudResolver.ResolveKeyAsync(
            "https://contosokeyvault.vault.azure.net/keys/TestRSAKey1", 
            CancellationToken.None).GetAwaiter().GetResult();

// Now you simply use the RSA key to encrypt by setting it in the BlobEncryptionPolicy.
BlobEncryptionPolicy policy = new BlobEncryptionPolicy(rsa, null);
BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };

// Reference a block blob.
CloudBlockBlob blob = contain.GetBlockBlobReference("MyFile.txt");

// Upload using the UploadFromStream method.
using (var stream = System.IO.File.OpenRead(@"C:\Temp\MyFile.txt"))
    blob.UploadFromStream(stream, stream.Length, null, options, null);
```
---

> [!NOTE]
> Если вы рассмотрите конструктор BlobEncryptionPolicy, вы увидите, что он может принимать ключ и (или) сопоставитель. Необходимо помнить, что сейчас сопоставитель нельзя использовать для шифрования, так как он не поддерживает ключ по умолчанию.


## <a name="decrypt-blob-and-download"></a>Расшифруйте BLOB-объект и загрузите его

Расшифровка работает, когда использование классов сопоставителя имеет смысл. Идентификатор ключа, используемого для шифрования, связан с BLOB-объектом в своих метаданных, поэтому не нужно извлекать ключ и запоминать связь между ключом и BLOB-объектом. Необходимо просто убедиться, что ключ по-прежнему находится в хранилище ключей.   

Закрытый ключ RSA Key остается в хранилище ключей, поэтому для расшифровки зашифрованный ключ из метаданных BLOB-объекта, содержащего CEК, отправляется в хранилище ключей.

Добавьте следующий код для расшифровки отправленного большого двоичного объекта.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
// In this case, we will not pass a key and only pass the resolver because
// this policy will only be used for downloading / decrypting.
BlobEncryptionPolicy policy = new BlobEncryptionPolicy(null, cloudResolver);
BlobRequestOptions options = new BlobRequestOptions() { EncryptionPolicy = policy };

using (var np = File.Open(@"C:\data\MyFileDecrypted.txt", FileMode.Create))
    blob.DownloadToStream(np, null, options, null);
```
---

> [!NOTE]
> Существует несколько других видов сопоставителей, которые облегчают управление ключами, например AggregateKeyResolver и CachingKeyResolver.

## <a name="use-key-vault-secrets"></a>Использование секретов хранилища ключей

Секрет можно использовать для шифрования на стороне клиента с помощью класса SymmetricKey, так как он фактически является симметричным ключом. Но, как указано выше, секрет в хранилище ключей не сопоставляется точно с SymmetricKey. Необходимо понять несколько вещей.

* Ключ в SymmetricKey должен быть фиксированной длины: 128, 192, 256, 384 или 512 бит.
* Ключ в SymmetricKey должен быть в кодировке Base64.
* Секрет хранилища ключей, который будет использоваться в качестве SymmetricKey, должен иметь тип содержимого application/octet-stream в хранилище ключей.

Ниже приведен пример использования PowerShell для создания в хранилище ключей секрета, который можно использовать как SymmetricKey.
Имейте ввиду, что жестко запрограммированное значение $key приводится только для примера. Сгенерируйте этот ключ в своем собственном коде.

```powershell
// Here we are making a 128-bit key so we have 16 characters.
//     The characters are in the ASCII range of UTF8 so they are
//    each 1 byte. 16 x 8 = 128.
$key = "qwertyuiopasdfgh"
$b = [System.Text.Encoding]::UTF8.GetBytes($key)
$enc = [System.Convert]::ToBase64String($b)
$secretvalue = ConvertTo-SecureString $enc -AsPlainText -Force

// Substitute the VaultName and Name in this command.
$secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'TestSecret2' -SecretValue $secretvalue -ContentType "application/octet-stream"
```

Для получения этого секрета как SymmetricKey в консольном приложении можно использовать тот же вызов, что и ранее.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Сейчас мы работаем над созданием фрагментов кода для версии 12.x клиентских библиотек службы хранилища Azure. Дополнительные сведения см. в [объявлении о клиентских библиотеках службы хранилища Azure версии 12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
SymmetricKey sec = (SymmetricKey) cloudResolver.ResolveKeyAsync(
    "https://contosokeyvault.vault.azure.net/secrets/TestSecret2/",
    CancellationToken.None).GetAwaiter().GetResult();
```
---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании службы хранилища Microsoft Azure с C# см. в статье [Клиентская библиотека службы хранилища Microsoft Azure для .NET](/previous-versions/azure/dn261237(v=azure.100)).

Дополнительные сведения о REST API для больших двоичных объектов см. в статье [REST API службы BLOB-объектов](/rest/api/storageservices/Blob-Service-REST-API).

Новости о службе хранилища Microsoft Azure см. в [блоге команды разработчиков службы хранилища Microsoft Azure](/archive/blogs/windowsazurestorage/).
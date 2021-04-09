---
title: Краткое руководство. Использование клиентской библиотеки сертификатов Azure Key Vault для .NET (версия 4)
description: Узнайте, как создавать, извлекать и удалять сертификаты в Azure Key Vault с помощью клиентской библиотеки .NET (версия 4)
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/23/2020
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 4dd216f4018feca8c3461104a5beb220d90fd743
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97932849"
---
# <a name="quickstart-azure-key-vault-certificate-client-library-for-net-sdk-v4"></a>Краткое руководство. Использование клиентской библиотеки сертификатов Azure Key Vault для .NET (пакет SDK версии 4)

Приступите к работе с клиентской библиотекой сертификатов Azure Key Vault для .NET. [Azure Key Vault](../general/overview.md) — это облачная служба, которая предоставляет защищенное хранилище для сертификатов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Создать хранилища Azure Key Vault и управлять ими можно на портале Azure. Из этого краткого руководства вы узнаете, как создавать, извлекать и удалять сертификаты в Azure Key Vault с помощью клиентской библиотеки .NET.

Ресурсы клиентской библиотеки Key Vault:

[Справочная документация по API](/dotnet/api/azure.security.keyvault.certificates) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.Certificates/).

Дополнительные сведения о Key Vault и сертификатах см. в следующих статьях:
- [Общие сведения о Key Vault](../general/overview.md)
- [Общие сведения о сертификатах](about-certificates.md).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
* [Пакет SDK для .NET Core 3.1 или более поздней версии](https://dotnet.microsoft.com/download/dotnet-core)
* [Azure CLI](/cli/azure/install-azure-cli)
* Хранилище Key Vault можно создать с помощью [портала Azure](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md) или [Azure PowerShell](../general/quick-create-powershell.md).

В этом кратком руководстве используется `dotnet` и Azure CLI

## <a name="setup"></a>Настройка

В этом кратком руководстве используется библиотека удостоверений Azure и Azure CLI для проверки подлинности пользователя в службах Azure. Разработчики также могут использовать Visual Studio или Visual Studio Code для проверки подлинности своих вызовов. Дополнительные сведения см. в статье [Проверка подлинности клиента с помощью клиентской библиотеки удостоверений Azure](/dotnet/api/overview/azure/identity-readme?#authenticate-the-client&preserve-view=true).

### <a name="sign-in-to-azure"></a>Вход в Azure

1. Выполните команду `login`.

    ```azurecli-interactive
    az login
    ```

    Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа.

    Если нет, самостоятельно откройте в браузере страницу [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и введите код авторизации, отображаемый в терминале.

2. Выполните вход в браузере с помощью учетных данных.

#### <a name="grant-access-to-your-key-vault"></a>Предоставление доступа к хранилищу ключей

Создайте политику доступа для хранилища ключей, которая предоставляет разрешение сертификата для учетной записи пользователя.

```console
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --certificate-permissions delete get list create purge
```

### <a name="create-new-net-console-app"></a>Создание консольного приложения .NET

1. В командной оболочке выполните следующую команду, чтобы создать проект с именем `key-vault-console-app`.

    ```dotnetcli
    dotnet new console --name key-vault-console-app
    ```

1. Перейдите ко вновь созданному каталогу *key-store-console-app* и выполните следующую команду, чтобы начать сборку проекта.

    ```dotnetcli
    dotnet build
    ```

    Выходные данные сборки не должны содержать предупреждений или ошибок.
    
    ```console
    Build succeeded.
     0 Warning(s)
     0 Error(s)
    ```

### <a name="install-the-packages"></a>Установка пакетов

Из командной оболочки установите клиентскую библиотеку сертификатов Azure Key Vault для .NET:

```dotnetcli
dotnet add package Azure.Security.KeyVault.Certificates
```

Для этого краткого руководства вам также потребуется установить клиентскую библиотеку Azure SDK для Удостоверения Azure:

```dotnetcli
dotnet add package Azure.Identity
```

#### <a name="set-environment-variables"></a>Настройка переменных среды

Это приложение использует имя хранилища ключей в качестве переменной среды с именем `KEY_VAULT_NAME`.

Windows
```cmd
set KEY_VAULT_NAME=<your-key-vault-name>
````
Windows PowerShell
```powershell
$Env:KEY_VAULT_NAME="<your-key-vault-name>"
```

macOS или Linux
```bash
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="object-model"></a>Объектная модель

Клиентская библиотека сертификатов Azure Key Vault для .NET позволяет управлять сертификатами. В разделе [Примеры кода](#code-examples) показано, как создать клиент, а также как указать, получить и удалить сертификат.

## <a name="code-examples"></a>Примеры кода

### <a name="add-directives"></a>Добавление директив

Добавьте следующие директивы в начало *Program.cs*.

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Certificates;
```

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс [DefaultAzureCredential()](/dotnet/api/azure.identity.defaultazurecredential) из [библиотеки удостоверений Azure](https://docs.microsoft.com/dotnet/api/overview/azure/identity-readme), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения о проверке подлинности в хранилище ключей см. в [руководстве для разработчиков](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

```csharp
string keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
var kvUri = "https://" + keyVaultName + ".vault.azure.net";

var client = new CertificateClient(new Uri(kvUri), new DefaultAzureCredential());
```

### <a name="save-a-certificate"></a>Сохранение сертификата

В этом примере для удобства можно использовать самозаверяющий сертификат с политикой выдачи по умолчанию. Для этой задачи используйте метод [StartCreateCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.startcreatecertificateasync). Параметры метода принимают имя и [политику сертификата](https://docs.microsoft.com/dotnet/api/azure.security.keyvault.certificates.certificatepolicy).

```csharp
var operation = await client.StartCreateCertificateAsync("myCertificate", CertificatePolicy.Default);
var certificate = await operation.WaitForCompletionAsync();
```

> [!NOTE]
> Если сертификат с таким именем уже существует, выполните приведенный выше код, чтобы создать версию этого сертификата.

### <a name="retrieve-a-certificate"></a>Получение сертификата

Теперь вы можете получить ранее созданный сертификат с помощью метода [GetCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.getcertificateasync).

```csharp
var certificate = await client.GetCertificateAsync("myCertificate");
```

### <a name="delete-a-certificate"></a>Удаление сертификата

Теперь удалите сертификат из хранилища ключей и очистите его с помощью методов [StartDeleteCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.startdeletecertificateasync) и [PurgeDeletedCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.purgedeletedcertificateasync).

```csharp
var operation = await client.StartDeleteCertificateAsync("myCertificate");

// You only need to wait for completion if you want to purge or recover the certificate.
await operation.WaitForCompletionAsync();

var certificate = operation.Value;
await client.PurgeDeletedCertificateAsync("myCertificate");
```

## <a name="sample-code"></a>Образец кода

Измените консольное приложение .NET Core для взаимодействия с Azure Key Vault, выполнив следующие действия.

- Замените код в *Program.cs* на следующий код:

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Azure.Identity;
    using Azure.Security.KeyVault.Certificates;
    
    namespace key_vault_console_app
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                const string certificateName = "myCertificate";
                var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
                var kvUri = $"https://{keyVaultName}.vault.azure.net";
    
                var client = new CertificateClient(new Uri(kvUri), new DefaultAzureCredential());
    
                Console.Write($"Creating a certificate in {keyVaultName} called '{certificateName}' ...");
                CertificateOperation operation = await client.StartCreateCertificateAsync(certificateName, CertificatePolicy.Default);
                await operation.WaitForCompletionAsync();
                Console.WriteLine(" done.");
    
                Console.WriteLine($"Retrieving your certificate from {keyVaultName}.");
                var certificate = await client.GetCertificateAsync(certificateName);
                Console.WriteLine($"Your certificate version is '{certificate.Value.Properties.Version}'.");
    
                Console.Write($"Deleting your certificate from {keyVaultName} ...");
                DeleteCertificateOperation deleteOperation = await client.StartDeleteCertificateAsync(certificateName);
                // You only need to wait for completion if you want to purge or recover the certificate.
                await deleteOperation.WaitForCompletionAsync();
                Console.WriteLine(" done.");

                Console.Write($"Purging your certificate from {keyVaultName} ...");
                await client.PurgeDeletedCertificateAsync(certificateName);
                Console.WriteLine(" done.");
            }
        }
    }
    ```
### <a name="test-and-verify"></a>Тестирование и проверка

Чтобы создать проект, выполните указанную ниже команду.

```dotnetcli
dotnet build
```

Отображаются выходные данные, аналогичные следующим:

```console
Creating a certificate in mykeyvault called 'myCertificate' ... done.
Retrieving your certificate from mykeyvault.
Your certificate version is '8532359bced24e4bb2525f2d2050738a'.
Deleting your certificate from mykeyvault ... done
Purging your certificate from mykeyvault ... done
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы создали хранилище ключей, сохранили в нем сертификат и извлекли его. 

Дополнительные сведения об Azure Key Vault и о том, как интегрировать его с вашими приложениями, см. в следующих статьях:

- [Обзор Azure Key Vault](../general/overview.md)
- [Обзор сертификатов](about-certificates.md)
- См. [Руководство по доступу к Key Vault из приложения службы приложений](../general/tutorial-net-create-vault-azure-web-app.md)
- См. [Руководство по доступу к Key Vault из виртуальной машины](../general/tutorial-net-virtual-machine.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)

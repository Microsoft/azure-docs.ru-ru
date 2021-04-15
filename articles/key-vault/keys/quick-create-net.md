---
title: Краткое руководство по использованию клиентской библиотеки ключей Azure Key Vault для .NET (версия 4)
description: Узнайте, как создавать, извлекать и удалять ключи в Azure Key Vault с помощью клиентской библиотеки .NET (версия 4).
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/23/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-csharp, devx-track-azurepowershell
ms.openlocfilehash: 52e33660b4076f8119ddb5d77e2dbbf7ee201913
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105968464"
---
# <a name="quickstart-azure-key-vault-key-client-library-for-net-sdk-v4"></a>Краткое руководство. Использование клиентской библиотеки ключей Azure Key Vault для .NET (пакет SDK версии 4)

Начало работы с клиентской библиотекой ключей Azure Key Vault для .NET. [Azure Key Vault](../general/overview.md) — это облачная служба, которая предоставляет защищенное хранилище для ключей шифрования. Вы можете безопасно хранить ключи шифрования, пароли, сертификаты и другие секреты. Создать хранилища Azure Key Vault и управлять ими можно на портале Azure. Из этого краткого руководства вы узнаете, как создавать, извлекать и удалять ключи в Azure Key Vault с помощью клиентской библиотеки ключей .NET.

Ресурсы клиентской библиотеки ключей Key Vault:

[Справочная документация по API](/dotnet/api/azure.security.keyvault.keys) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.keys/).

Дополнительные сведения о Key Vault и ключах см. в следующих статьях:
- [Общие сведения о Key Vault](../general/overview.md)
- [Общие сведения о ключах](about-keys.md).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
* [Пакет SDK для .NET Core 3.1 или более поздней версии](https://dotnet.microsoft.com/download/dotnet-core)
* [Azure CLI](/cli/azure/install-azure-cli)
* Хранилище Key Vault можно создать с помощью [портала Azure](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md) или [Azure PowerShell](../general/quick-create-powershell.md).

## <a name="setup"></a>Настройка

В этом кратком руководстве используется библиотека удостоверений Azure для проверки подлинности пользователя в службах Azure. Разработчики также могут использовать Visual Studio или Visual Studio Code для проверки подлинности своих вызовов. Дополнительные сведения см. в статье [Проверка подлинности клиента с помощью клиентской библиотеки удостоверений Azure](/dotnet/api/overview/azure/identity-readme?#authenticate-the-client&preserve-view=true).

### <a name="sign-in-to-azure"></a>Вход в Azure

1. Выполните команду `login`.

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    ```azurecli-interactive
    az login
    ```
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```
    ---

    Если в Azure CLI или Azure PowerShell можно запустить браузер по умолчанию, откроется браузер со страницей входа.

    Если нет, самостоятельно откройте в браузере страницу [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и введите код авторизации, отображаемый в терминале.

2. Выполните вход в браузере с помощью учетных данных.

#### <a name="grant-access-to-your-key-vault"></a>Предоставление доступа к хранилищу ключей

Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование ключа.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli-interactive
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --key-permissions delete get list create purge
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName <your-key-vault-name> -UserPrincipalName user@domain.com -PermissionsToSecrets delete,get,list,set,purge
```
---

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

Из командной оболочки установите клиентскую библиотеку ключей Azure Key Vault для .NET:

```dotnetcli
dotnet add package Azure.Security.KeyVault.Keys
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
```azurepowershell
$Env:KEY_VAULT_NAME="<your-key-vault-name>"
```

macOS или Linux
```bash
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="object-model"></a>Объектная модель

Клиентская библиотека ключей Azure Key Vault для .NET позволяет управлять ключами. В разделе [Примеры кода](#code-examples) показано, как создать клиент, а также задать, получить и удалить ключ.

## <a name="code-examples"></a>Примеры кода

### <a name="add-directives"></a>Добавление директив

Добавьте следующие директивы в начало *Program.cs*.

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Keys;
```

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс [DefaultAzureCredential()](/dotnet/api/azure.identity.defaultazurecredential) из [библиотеки удостоверений Azure](https://docs.microsoft.com/dotnet/api/overview/azure/identity-readme), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения о проверке подлинности в хранилище ключей см. в [руководстве для разработчиков](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

```csharp
var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
var kvUri = "https://" + keyVaultName + ".vault.azure.net";

var client = new KeyClient(new Uri(kvUri), new DefaultAzureCredential());
```

### <a name="save-a-key"></a>Сохранение ключа

Для этой задачи используйте метод [CreateKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.createkeyasync). Параметры метода принимают имя и [тип ключа](https://docs.microsoft.com/dotnet/api/azure.security.keyvault.keys.keytype).

```csharp
var key = await client.CreateKeyAsync("myKey", KeyType.Rsa);
```

> [!NOTE]
> Если ключ с таким именем уже существует, выполните приведенный выше код, чтобы создать другую версию этого ключа.

### <a name="retrieve-a-key"></a>Получение ключа

Теперь вы можете получить ранее созданный ключ с помощью метода [GetKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.getkeyasync).

```csharp
var key = await client.GetKeyAsync("myKey");
```

### <a name="delete-a-key"></a>Удаление ключа

Наконец, удалите ключ из хранилища ключей и очистите его с помощью методов [StartDeleteKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.startdeletekeyasync) и [PurgeDeletedKeyAsync](/dotnet/api/azure.security.keyvault.keys.keyclient.purgedeletedkeyasync).

```csharp
var operation = await client.StartDeleteKeyAsync("myKey");

// You only need to wait for completion if you want to purge or recover the key.
await operation.WaitForCompletionAsync();

var key = operation.Value;
await client.PurgeDeletedKeyAsync("myKey");
```

## <a name="sample-code"></a>Образец кода

Измените консольное приложение .NET Core для взаимодействия с Azure Key Vault, выполнив следующие действия.

- Замените код в *Program.cs* на следующий код:

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Azure.Identity;
    using Azure.Security.KeyVault.Keys;
    
    namespace key_vault_console_app
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                const string keyName = "myKey";
                var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
                var kvUri = $"https://{keyVaultName}.vault.azure.net";
    
                var client = new KeyClient(new Uri(kvUri), new DefaultAzureCredential());
    
                Console.Write($"Creating a key in {keyVaultName} called '{keyName}' ...");
                var createdKey = await client.CreateKeyAsync(keyName, KeyType.Rsa);
                Console.WriteLine("done.");
    
                Console.WriteLine($"Retrieving your key from {keyVaultName}.");
                var key = await client.GetKeyAsync(keyName);
                Console.WriteLine($"Your key version is '{key.Value.Properties.Version}'.");
    
                Console.Write($"Deleting your key from {keyVaultName} ...");
                var deleteOperation = await client.StartDeleteKeyAsync(keyName);
                // You only need to wait for completion if you want to purge or recover the key.
                await deleteOperation.WaitForCompletionAsync();
                Console.WriteLine("done.");

                Console.Write($"Purging your key from {keyVaultName} ...");
                await client.PurgeDeletedKeyAsync(keyName);
                Console.WriteLine(" done.");
            }
        }
    }
    ```
### <a name="test-and-verify"></a>Тестирование и проверка

1.  Чтобы создать проект, выполните указанную ниже команду.

    ```dotnetcli
    dotnet build
    ```

1. Выполните следующую команду для запуска этого приложения.

    ```dotnetcli
    dotnet run
    ```

1. При появлении запроса введите значение секрета. Например, mySecretPassword.

    Отображаются выходные данные, аналогичные следующим:

    ```console
    Creating a key in mykeyvault called 'myKey' ... done.
    Retrieving your key from mykeyvault.
    Your key version is '8532359bced24e4bb2525f2d2050738a'.
    Deleting your key from jl-kv ... done
    Purging your key from <your-unique-keyvault-name> ... done.   
    ```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать хранилище ключей, сохранить в нем ключ и извлечь его. 

Дополнительные сведения об Azure Key Vault и о том, как интегрировать его с вашими приложениями, см. в следующих статьях:

- [Обзор Azure Key Vault](../general/overview.md)
- [Общие сведения о ключах](about-keys.md)
- См. [Руководство по доступу к Key Vault из приложения службы приложений](../general/tutorial-net-create-vault-azure-web-app.md)
- См. [Руководство по доступу к Key Vault из виртуальной машины](../general/tutorial-net-virtual-machine.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)

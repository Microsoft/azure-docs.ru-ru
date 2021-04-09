---
title: Краткое руководство по использованию клиентской библиотеки секретов Azure Key Vault для .NET (версия 4)
description: Узнайте, как создавать, извлекать и удалять секреты в Azure Key Vault с помощью клиентской библиотеки .NET (версия 4)
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/23/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: a82c2cdf7084b31eb6ba861e48ecffb81e6d1363
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102453705"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-net-sdk-v4"></a>Краткое руководство. Использование клиентской библиотеки секретов Azure Key Vault для .NET (пакет SDK версии 4)

Начало работы с клиентской библиотекой секретов Azure Key Vault для .NET. [Azure Key Vault](../general/overview.md) — это облачная служба, которая предоставляет защищенное хранилище для секретов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Создать хранилища Azure Key Vault и управлять ими можно на портале Azure. В рамках этого краткого руководства вы узнаете, как создавать, извлекать и удалять секреты в Azure Key Vault с помощью клиентской библиотеки .NET.

Ресурсы клиентской библиотеки Key Vault:

[Справочная документация по API](/dotnet/api/azure.security.keyvault.secrets) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.Secrets/).

Дополнительные сведения о Key Vault и секретах см. в следующих статьях:
- [Общие сведения о Key Vault](../general/overview.md)
- [Общие сведения о секретах](about-secrets.md)

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

### <a name="grant-access-to-your-key-vault"></a>Предоставление доступа к хранилищу ключей

Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование секрета.

```console
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --secret-permissions delete get list set purge
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

Из командной оболочки установите клиентскую библиотеку секретов Azure Key Vault для .NET:

```dotnetcli
dotnet add package Azure.Security.KeyVault.Secrets
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

Клиентская библиотека секретов Azure Key Vault для .NET позволяет управлять секретами. В разделе [Примеры кода](#code-examples) показано, как создать клиент, задать секрет, получить секрет и удалить секрет.

## <a name="code-examples"></a>Примеры кода

### <a name="add-directives"></a>Добавление директив

Добавьте следующие директивы в начало *Program.cs*.

[!code-csharp[](~/samples-key-vault-dotnet-quickstart/key-vault-console-app/Program.cs?name=directives)]

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс [DefaultAzureCredential()](/dotnet/api/azure.identity.defaultazurecredential) из [библиотеки удостоверений Azure](https://docs.microsoft.com/dotnet/api/overview/azure/identity-readme), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения о проверке подлинности в хранилище ключей см. в [руководстве для разработчиков](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

[!code-csharp[](~/samples-key-vault-dotnet-quickstart/key-vault-console-app/Program.cs?name=authenticate)]

### <a name="save-a-secret"></a>Сохранение секрета

Теперь, когда консольное приложение прошло проверку подлинности, добавьте в хранилище ключей секрет. Для этой задачи используйте метод [SetSecretAsync](/dotnet/api/azure.security.keyvault.secrets.secretclient.setsecretasync). Первый параметр метода принимает в этом примере имя секрета&mdash;"mySecret".

```csharp
await client.SetSecretAsync(secretName, secretValue);
```

> [!NOTE]
> Если имя секрета существует, с помощью приведенного выше кода можно создать новую версию этого секрета.


### <a name="retrieve-a-secret"></a>Получение секрета

Теперь вы можете получить ранее заданное значение с помощью метода [GetSecretAsync](/dotnet/api/azure.security.keyvault.secrets.secretclient.getsecretasync).

```csharp
var secret = await client.GetSecretAsync(secretName);
```

Секрет теперь сохраняется как `secret.Value`.

### <a name="delete-a-secret"></a>Удаление секрета.

Наконец, мы удалим секрет из хранилища ключей с помощью методов [StartDeleteSecretAsync](/dotnet/api/azure.security.keyvault.secrets.secretclient.startdeletesecretasync) и [PurgeDeletedSecretAsync](/dotnet/api/azure.security.keyvault.keys.keyclient).

```csharp
var operation = await client.StartDeleteSecretAsync("mySecret");
// You only need to wait for completion if you want to purge or recover the key.
await operation.WaitForCompletionAsync();

await client.PurgeDeletedKeyAsync("mySecret");
```

## <a name="sample-code"></a>Образец кода

Измените консольное приложение .NET Core для взаимодействия с Azure Key Vault, выполнив следующие действия.

1. Замените код в *Program.cs* на следующий код:

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Azure.Identity;
    using Azure.Security.KeyVault.Secrets;
    
    namespace key_vault_console_app
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                const string secretName = "mySecret";
                var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
                var kvUri = $"https://{keyVaultName}.vault.azure.net";
    
                var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential());
    
                Console.Write("Input the value of your secret > ");
                var secretValue = Console.ReadLine();
    
                Console.Write($"Creating a secret in {keyVaultName} called '{secretName}' with the value '{secretValue}' ...");
                await client.SetSecretAsync(secretName, secretValue);
                Console.WriteLine(" done.");
    
                Console.WriteLine("Forgetting your secret.");
                secretValue = string.Empty;
                Console.WriteLine($"Your secret is '{secretValue}'.");
    
                Console.WriteLine($"Retrieving your secret from {keyVaultName}.");
                var secret = await client.GetSecretAsync(secretName);
                Console.WriteLine($"Your secret is '{secret.Value.Value}'.");
    
                Console.Write($"Deleting your secret from {keyVaultName} ...");
                DeleteSecretOperation operation = await client.StartDeleteSecretAsync(secretName);
                // You only need to wait for completion if you want to purge or recover the secret.
                await operation.WaitForCompletionAsync();
                Console.WriteLine(" done.");
                
                Console.Write($"Purging your secret from {keyVaultName} ...");
                await client.PurgeDeletedSecretAsync(secretName);
                Console.WriteLine(" done.");
            }
        }
    }
    ```
### <a name="test-and-verify"></a>Тестирование и проверка

1. Выполните следующую команду для запуска этого приложения.

    ```dotnetcli
    dotnet run
    ```

1. При появлении запроса введите значение секрета. Например, mySecretPassword.

Отображаются выходные данные, аналогичные следующим:

```console
Input the value of your secret > mySecretPassword
Creating a secret in <your-unique-keyvault-name> called 'mySecret' with the value 'mySecretPassword' ... done.
Forgetting your secret.
Your secret is ''.
Retrieving your secret from <your-unique-keyvault-name>.
Your secret is 'mySecretPassword'.
Deleting your secret from <your-unique-keyvault-name> ... done.    
Purging your secret from <your-unique-keyvault-name> ... done.
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure Key Vault и о том, как интегрировать его с вашими приложениями, см. в следующих статьях:

- [Обзор Azure Key Vault](../general/overview.md)
- См. [Руководство по доступу к Key Vault из приложения службы приложений](../general/tutorial-net-create-vault-azure-web-app.md)
- См. [Руководство по доступу к Key Vault из виртуальной машины](../general/tutorial-net-virtual-machine.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)

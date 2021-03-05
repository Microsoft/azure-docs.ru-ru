---
title: Учебник. Использование Azure Key Vault с виртуальной машиной в .NET | Документация Майкрософт
description: В этом руководстве показано, как, используя приложение ASP.NET Core, настроить виртуальную машину для чтения секрета из хранилища ключей.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/20/2020
ms.author: mbaldwin
ms.custom: mvc, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: e69e5d9b94a47bf7db21ef3732a4ddcba7c2cf5c
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102181556"
---
# <a name="tutorial-use-azure-key-vault-with-a-virtual-machine-in-net"></a>Руководство по Использование Azure Key Vault с виртуальной машиной в .NET

Azure Key Vault помогает защитить секреты, такие как ключи API, строки подключения к базам данных, необходимые для доступа к приложениям, службам и ИТ-ресурсам.

Из этого руководства вы узнаете, как настроить консольное приложение для считывания сведений из Azure Key Vault. Приложение будет использовать управляемое удостоверение виртуальной машины для аутентификации в Key Vault. 

В этом руководстве описаны следующие процедуры.

> [!div class="checklist"]
> * Создайте группу ресурсов.
> * Создать хранилище ключей.
> * Добавление секрета в хранилище ключей.
> * получение секрета из хранилища ключей;
> * Создайте виртуальную машину Azure.
> * Включить [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) для виртуальной машины.
> * Назначение разрешений для идентификатора виртуальной машины.

Перед началом работы ознакомьтесь c [основными понятиями службы Key Vault](basic-concepts.md). 

Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Предварительные требования

Для Windows, Mac и Linux:
  * [Git](https://git-scm.com/downloads);
  * [Пакет SDK для .NET Core 3.1 или более поздней версии](https://dotnet.microsoft.com/download/dotnet-core/3.1).
  * [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="create-resources-and-assign-permissions"></a>Создание ресурсов и назначение разрешений

Перед началом программирования необходимо создать некоторые ресурсы, поместить секрет в хранилище ключей и назначить разрешения.

### <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы войти в Azure с помощью Azure CLI, введите следующую команду:

```azurecli
az login
```

## <a name="create-a-resource-group-and-key-vault"></a>Создание группы ресурсов и хранилища ключей

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

## <a name="populate-your-key-vault-with-a-secret"></a>Запись секрета в хранилище ключей

[!INCLUDE [Create a secret](../../../includes/key-vault-create-secret.md)]

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины
Создайте виртуальную машину Windows или Linux одним из следующих способов.

| Windows | Linux |
|--|--|
| [Azure CLI](../../virtual-machines/windows/quick-create-cli.md) | [Azure CLI](../../virtual-machines/linux/quick-create-cli.md) |  
| [PowerShell](../../virtual-machines/windows/quick-create-powershell.md) | [PowerShell](../../virtual-machines/linux/quick-create-powershell.md) |
| [Портал Azure](../../virtual-machines/windows/quick-create-portal.md) | [Портал Azure](../../virtual-machines/linux/quick-create-portal.md) |

## <a name="assign-an-identity-to-the-vm"></a>Назначение удостоверения виртуальной машине
Создайте назначаемое системой удостоверение для виртуальной машины с помощью команды [az vm identity assign](/cli/azure/vm/identity?view=azure-cli-latest#az-vm-identity-assign):

```azurecli
az vm identity assign --name <NameOfYourVirtualMachine> --resource-group <YourResourceGroupName>
```

Обратите внимание на присвоенный системой идентификатор, который отображается в следующем коде. Выходные данные приведенной выше команды выглядят следующим образом. 

```output
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## <a name="assign-permissions-to-the-vm-identity"></a>Назначение разрешения для идентификатора виртуальной машины
Предоставьте созданному выше удостоверению разрешение для хранилища ключей, выполнив команду [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy).

```azurecli
az keyvault set-policy --name '<your-unique-key-vault-name>' --object-id <VMSystemAssignedIdentity> --secret-permissions get list
```

## <a name="sign-in-to-the-virtual-machine"></a>Вход в виртуальную машину

Чтобы войти в виртуальную машину, следуйте инструкциям по подключению или входу в виртуальную машину на платформе [Windows](../../virtual-machines/windows/connect-logon.md) или [Linux](../../virtual-machines/linux/login-using-aad.md).

## <a name="set-up-the-console-app"></a>Настройка консольного приложения

Создайте консольное приложение и установите необходимые пакеты с помощью команды `dotnet`.

### <a name="install-net-core"></a>Установка .NET Core

Чтобы установить .NET Core, перейдите на страницу [​​загрузки .NET](https://www.microsoft.com/net/download).

### <a name="create-and-run-a-sample-net-app"></a>Создание и запуск примера приложения .NET

Откройте командную строку.

Вы можете вывести Hello World на консоль, выполнив следующие команды.

```console
dotnet new console -n keyvault-console-app
cd keyvault-console-app
dotnet run
```

### <a name="install-the-package"></a>Установка пакета

В окне консоли установите клиентскую библиотеку секретов Azure Key Vault для .NET:

```console
dotnet add package Azure.Security.KeyVault.Secrets
```

Для работы с этим кратким руководством необходимо установить следующий пакет удостоверений для аутентификации в Azure Key Vault:

```console
dotnet add package Azure.Identity
```

## <a name="edit-the-console-app"></a>Изменение консольного приложения

Откройте файл *Program.cs* и добавьте следующие пакеты.

```csharp
using System;
using Azure.Core;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
```

Добавьте эти строки, обновив URI, чтобы отразить `vaultUri` хранилища ключей. Для проверки подлинности в хранилище ключей в следующем примере кода используется функция [DefaultAzureCredential()](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet), которая использует токен из управляемого удостоверения приложения. В ней также реализована экспоненциальная задержка для повторных попыток на случай регулирования запросов к хранилищу ключей.

```csharp
  class Program
    {
        static void Main(string[] args)
        {
            string secretName = "mySecret";

            var kvUri = "https://<your-key-vault-name>.vault.azure.net";
            SecretClientOptions options = new SecretClientOptions()
            {
                Retry =
                {
                    Delay= TimeSpan.FromSeconds(2),
                    MaxDelay = TimeSpan.FromSeconds(16),
                    MaxRetries = 5,
                    Mode = RetryMode.Exponential
                 }
            };

            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential(),options);

            Console.Write("Input the value of your secret > ");
            string secretValue = Console.ReadLine();

            Console.Write("Creating a secret in " + keyVaultName + " called '" + secretName + "' with the value '" + secretValue + "` ...");

            client.SetSecret(secretName, secretValue);

            Console.WriteLine(" done.");

            Console.WriteLine("Forgetting your secret.");
            secretValue = "";
            Console.WriteLine("Your secret is '" + secretValue + "'.");

            Console.WriteLine("Retrieving your secret from " + keyVaultName + ".");

            KeyVaultSecret secret = client.GetSecret(secretName);

            Console.WriteLine("Your secret is '" + secret.Value + "'.");

            Console.Write("Deleting your secret from " + keyVaultName + " ...");

            client.StartDeleteSecret(secretName);

            System.Threading.Thread.Sleep(5000);
            Console.WriteLine(" done.");

        }
    }
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите виртуальную машину и хранилище ключей, если они больше не нужны.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Справочник по REST API для Azure Key Vault](/rest/api/keyvault/)

---
title: Краткое руководство по использованию клиентской библиотеки ключей Azure Key Vault для JavaScript (версия 4)
description: Сведения о том, как создавать, извлекать и удалять ключи в Azure Key Vault с помощью клиентской библиотеки JavaScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: c0066409732f4492186ea0bf604261e1ab59ca9f
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750308"
---
# <a name="quickstart-azure-key-vault-key-client-library-for-javascript-version-4"></a>Краткое руководство. Использование клиентской библиотеки ключей Azure Key Vault для JavaScript (версия 4)

Начало работы с клиентской библиотекой ключей Azure Key Vault для JavaScript. [Azure Key Vault](../general/overview.md) — это облачная служба, которая предоставляет защищенное хранилище для ключей шифрования. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Создать хранилища Azure Key Vault и управлять ими можно на портале Azure. Из этого краткого руководства вы узнаете, как создавать, извлекать и удалять ключи в Azure Key Vault с помощью клиентской библиотеки ключей JavaScript.

Ресурсы клиентской библиотеки Key Vault:

[Справочная документация по API](/javascript/api/overview/azure/key-vault-index) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Пакет (npm)](https://www.npmjs.com/package/@azure/keyvault-keys).

Дополнительные сведения о Key Vault и ключах см. в следующих статьях:
- [Общие сведения о Key Vault](../general/overview.md)
- [Общие сведения о ключах](about-keys.md).

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Последняя версия [Node.js](https://nodejs.org) для вашей операционной системы.
- [Azure CLI](/cli/azure/install-azure-cli)
- Хранилище Key Vault можно создать с помощью [портала Azure](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md) или [Azure PowerShell](../general/quick-create-powershell.md).

В этом кратком руководстве предполагается, что вы используете [Azure CLI](/cli/azure/install-azure-cli).

## <a name="sign-in-to-azure"></a>Вход в Azure

1. Выполните команду `login`.

    ```azurecli-interactive
    az login
    ```

    Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа.

    Если нет, самостоятельно откройте в браузере страницу [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и введите код авторизации, отображаемый в терминале.

2. Выполните вход в браузере с помощью учетных данных.

## <a name="create-new-nodejs-application"></a>Создание приложения Node.js

Теперь создайте приложение Node. js, которое можно развернуть в облаке. 

1. В командной оболочке создайте папку с именем `key-vault-node-app`.

```azurecli
mkdir key-vault-node-app
```

1. Перейдите к созданному новому каталогу *key-vault-node-app* и выполните команду init, чтобы инициализировать проект узла.

```azurecli
cd key-vault-node-app
npm init -y
```

## <a name="install-key-vault-packages"></a>Установка пакетов Key Vault

Из окна консоли установите [библиотеку ключей](https://www.npmjs.com/package/@azure/keyvault-keys) Azure Key Vault для Node.js.

```azurecli
npm install @azure/keyvault-keys
```

Установите пакет [azure.identity](https://www.npmjs.com/package/@azure/identity) для проверки подлинности в Key Vault.

```azurecli
npm install @azure/identity
```

## <a name="set-environment-variables"></a>Настройка переменных среды

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
```cmd
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="grant-access-to-your-key-vault"></a>Предоставление доступа к хранилищу ключей

Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование ключа.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --key-permissions delete get list create purge
```

## <a name="code-examples"></a>Примеры кода

В приведенных ниже примерах кода показано, как создать клиент, а также как задать, извлечь и удалить ключ. 

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

1. Создать новый текстовый файл и сохранить его как "index.js".

1. Добавьте в него вызовы require для загрузки модулей Azure и Node.js.

1. Создайте структуру программы, включая простую обработку исключений.

```javascript
const readline = require('readline');

function askQuestion(query) {
    const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
    });

    return new Promise(resolve => rl.question(query, ans => {
        rl.close();
        resolve(ans);
    }))
}

async function main() {
    
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));
```

### <a name="add-directives"></a>Добавление директив

Добавьте в верхнюю часть кода следующие директивы:

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { KeyClient } = require("@azure/keyvault-keys");
```

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс [DefaultAzureCredential()](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) из [библиотеки удостоверений Azure](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения о проверке подлинности в хранилище ключей см. в [руководстве для разработчиков](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Добавьте следующий код в функцию main():

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new KeyClient(KVUri, credential);
```

### <a name="save-a-key"></a>Сохранение ключа

Теперь, когда приложение прошло проверку подлинности, можно поместить ключ в хранилище ключей с помощью метода [createKey](/javascript/api/@azure/keyvault-keys/keyclient?#createKey_string__KeyType__CreateKeyOptions_). Параметры этого метода принимают имя и [тип](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keytype) ключа.

```javascript
await client.createKey(keyName, keyType);
```

### <a name="retrieve-a-key"></a>Получение ключа

Теперь вы можете получить ранее заданное значение с помощью метода [getKey](/javascript/api/@azure/keyvault-keys/keyclient?#getKey_string__GetKeyOptions_).

```javascript
const retrievedKey = await client.getKey(keyName);
 ```

### <a name="delete-a-key"></a>Удаление ключа

Наконец, удалите ключ из хранилища ключей и очистите его с помощью методов [beginDeleteKey](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keyclient?#beginDeleteKey_string__BeginDeleteKeyOptions_) и [purgeDeletedKey](https://docs.microsoft.com/javascript/api/@azure/keyvault-keys/keyclient?#purgeDeletedKey_string__PurgeDeletedKeyOptions_).

```javascript
const deletePoller = await client.beginDeleteKey(keyName);
await deletePoller.pollUntilDone();
await client.purgeDeletedKey(keyName);
```

## <a name="sample-code"></a>Образец кода

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { KeyClient } = require("@azure/keyvault-keys");

const readline = require('readline');

function askQuestion(query) {
    const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
    });

    return new Promise(resolve => rl.question(query, ans => {
        rl.close();
        resolve(ans);
    }))
}

async function main() {

  const keyVaultName = process.env["KEY_VAULT_NAME"];
  const KVUri = "https://" + keyVaultName + ".vault.azure.net";

  const credential = new DefaultAzureCredential();
  const client = new KeyClient(KVUri, credential);

  const keyName = "myKey";
  
  console.log("Creating a key in " + keyVaultName + " called '" + keyName + "` ...");
  await client.createKey(keyName, "RSA");

  console.log("Done.");

  console.log("Retrieving your key from " + keyVaultName + ".");

  const retrievedKey = await client.getKey(keyName);

  console.log("Your key version is '" + retrievedKey.properties.version + "'.");

  console.log("Deleting your key from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteKey(keyName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your key from {keyVaultName} ...");
  await client.purgeDeletedKey(keyName);
  
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));

```

## <a name="test-and-verify"></a>Тестирование и проверка

Воспользуйтесь следующей командой для запуска этого приложения.

```azurecli
npm install
npm index.js
```

Отображаются выходные данные, аналогичные следующим:

```azurecli
Creating a key in <your-unique-keyvault-name> called 'myKey' ... done.
Retrieving your key from mykeyvault.
Your key version is '8532359bced24e4bb2525f2d2050738a'.
Deleting your key from <your-unique-keyvault-name> ... done.  
Purging your key from <your-unique-keyvault-name> ... done.   
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать хранилище ключей, сохранить в нем ключ и извлечь его. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- [Обзор ключей Azure Key Vault](about-keys.md)
- [Безопасный доступ к хранилищу ключей](../general/security-overview.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)

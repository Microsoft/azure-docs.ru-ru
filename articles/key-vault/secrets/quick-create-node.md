---
title: Краткое руководство по использованию клиентской библиотеки секретов Azure Key Vault для JavaScript (версия 4)
description: Сведения о том, как создавать, извлекать и удалять секреты в Azure Key Vault с помощью клиентской библиотеки JavaScript.
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: 70416daced2cbdebb70fb8e1defbcbcb599710f1
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101705489"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-javascript-version-4"></a>Краткое руководство. Использование клиентской библиотеки секретов Azure Key Vault для JavaScript (версия 4)

Начало работы с клиентской библиотекой секретов Azure Key Vault для JavaScript. [Azure Key Vault](../general/overview.md) — это облачная служба, которая предоставляет защищенное хранилище для секретов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Создать хранилища Azure Key Vault и управлять ими можно на портале Azure. В рамках этого краткого руководства вы узнаете, как создавать, извлекать и удалять секреты в Azure Key Vault с помощью клиентской библиотеки JavaScript.

Ресурсы клиентской библиотеки Key Vault:

[Справочная документация по API](/javascript/api/overview/azure/key-vault-index) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Пакет (npm)](https://www.npmjs.com/package/@azure/keyvault-secrets).

Дополнительные сведения о Key Vault и секретах см. в следующих статьях:
- [Общие сведения о Key Vault](../general/overview.md)
- [Общие сведения о секретах](about-secrets.md)

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

```terminal
mkdir key-vault-node-app
```

1. Перейдите к созданному новому каталогу *key-vault-node-app* и выполните команду init, чтобы инициализировать проект узла.

```terminal
cd key-vault-node-app
npm init -y
```

## <a name="install-key-vault-packages"></a>Установка пакетов Key Vault

Через окно консоли установите клиентскую [библиотеку секретов](https://www.npmjs.com/package/@azure/keyvault-secrets) Azure Key Vault для Node.js.

```terminal
npm install @azure/keyvault-secrets
```

Установите пакет [azure.identity](https://www.npmjs.com/package/@azure/identity) для проверки подлинности в Key Vault.

```terminal
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

Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование секрета.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --secret-permissions delete get list set purge
```

## <a name="code-examples"></a>Примеры кода

В приведенных ниже примерах кода показано, как создать клиент, а также как задать, извлечь и удалить секрет. 

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
const { SecretClient } = require("@azure/keyvault-secrets");
```

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс [DefaultAzureCredential()](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) из [библиотеки удостоверений Azure](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), который позволяет использовать один и тот же код в различных средах с различными параметрами для предоставления удостоверения. Дополнительные сведения о проверке подлинности в хранилище ключей см. в [руководстве для разработчиков](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Добавьте следующий код в функцию main():

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new SecretClient(KVUri, credential);
```

### <a name="save-a-secret"></a>Сохранение секрета

Теперь, когда приложение прошло проверку подлинности, вы можете разместить секрет в хранилище ключей с помощью [метода setSecret](/javascript/api/@azure/keyvault-secrets/secretclient?view=azure-node-latest#setSecret_string__string__SetSecretOptions_). Для этого нужно указать имя секрета — в нашем примере это mySecret.  

```javascript
await client.setSecret(secretName, secretValue);
```

### <a name="retrieve-a-secret"></a>Получение секрета

Теперь вы можете получить ранее заданное значение с помощью [метода getSecret](/javascript/api/@azure/keyvault-secrets/secretclient?view=azure-node-latest#getSecret_string__GetSecretOptions_).

```javascript
const retrievedSecret = await client.getSecret(secretName);
 ```

Секрет теперь сохраняется как `retrievedSecret.value`.

### <a name="delete-a-secret"></a>Удаление секрета.

Наконец, удалите секрет из хранилища ключей и очистите его с помощью методов [beginDeleteSecret](https://docs.microsoft.com/javascript/api/@azure/keyvault-secrets/secretclient?#beginDeleteSecret_string__BeginDeleteSecretOptions_) и [purgeDeletedSecret](https://docs.microsoft.com/javascript/api/@azure/keyvault-secrets/secretclient?#purgeDeletedSecret_string__PurgeDeletedSecretOptions_).

```javascript
const deletePoller = await client.beginDeleteSecret(secretName);
await deletePoller.pollUntilDone();
await client.purgeDeletedSecret(secretName);
```

## <a name="sample-code"></a>Образец кода

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { SecretClient } = require("@azure/keyvault-secrets");

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
  const client = new SecretClient(KVUri, credential);

  const secretName = "mySecret";
  var secretValue = await askQuestion("Input the value of your secret > ");

  console.log("Creating a secret in " + keyVaultName + " called '" + secretName + "' with the value '" + secretValue + "` ...");
  await client.setSecret(secretName, secretValue);

  console.log("Done.");

  console.log("Forgetting your secret.");
  secretValue = "";
  console.log("Your secret is '" + secretValue + "'.");

  console.log("Retrieving your secret from " + keyVaultName + ".");

  const retrievedSecret = await client.getSecret(secretName);

  console.log("Your secret is '" + retrievedSecret.value + "'.");

  console.log("Deleting your secret from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteSecret(secretName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your secret from {keyVaultName} ...");
  await client.purgeDeletedSecret(secretName);
  
}

main().then(() => console.log('Done')).catch((ex) => console.log(ex.message));

```

## <a name="test-and-verify"></a>Тестирование и проверка

1. Воспользуйтесь следующей командой для запуска этого приложения.

    ```terminal
    npm install
    node index.js
    ```

1. При появлении запроса введите значение секрета. Например, mySecretPassword.

    Отображаются выходные данные, аналогичные следующим:

    ```azurecli
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

При работе с этим кратким руководством вы создали хранилище ключей, сохранили в нем секрет и извлекли его. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- [Обзор секретов Azure Key Vault](about-secrets.md)
- [Безопасный доступ к хранилищу ключей](../general/secure-your-key-vault.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)

---
title: Краткое руководство по использованию клиентской библиотеки сертификатов Azure Key Vault для JavaScript (версия 4)
description: Узнайте, как создавать, извлекать и удалять сертификаты в Azure Key Vault с помощью клиентской библиотеки JavaScript
author: msmbaldwin
ms.author: mbaldwin
ms.date: 12/6/2020
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: 1064c6a1e2dddaae98e94ccceca7b1d550897393
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97930860"
---
# <a name="quickstart-azure-key-vault-certificate-client-library-for-javascript-version-4"></a>Краткое руководство. Использование клиентской библиотеки сертификатов Azure Key Vault для JavaScript (версия 4)

Приступите к работе с клиентской библиотекой сертификатов Azure Key Vault для JavaScript. [Azure Key Vault](../general/overview.md) — это облачная служба, которая предоставляет защищенное хранилище для сертификатов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Создать хранилища Azure Key Vault и управлять ими можно на портале Azure. Из этого краткого руководства вы узнаете, как создавать, извлекать и удалять сертификаты в Azure Key Vault с помощью клиентской библиотеки JavaScript.

Ресурсы клиентской библиотеки Key Vault:

[Справочная документация по API](/javascript/api/overview/azure/key-vault-index) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault) | [Пакет (npm)](https://www.npmjs.com/package/@azure/keyvault-certificates).

Дополнительные сведения о Key Vault и сертификатах см. в следующих статьях:
- [Общие сведения о Key Vault](../general/overview.md)
- [Общие сведения о сертификатах](about-certificates.md).

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

Через окно консоли установите клиентскую [библиотеку сертификатов](https://www.npmjs.com/package/@azure/keyvault-certificates) Azure Key Vault для Node.js.

```azurecli
npm install @azure/keyvault-certificates
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

Создайте политику доступа для хранилища ключей, которая предоставляет разрешение сертификата для учетной записи пользователя.

```azurecli
az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --certificate-permissions delete get list create purge
```

## <a name="code-examples"></a>Примеры кода

В приведенных ниже примерах кода показано, как создать клиент, задать сертификат, получить сертификат и удалить его. 

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
const { CertificateClient } = require("@azure/keyvault-certificates");
```

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента

В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в разделе [Обзор управляемых удостоверений](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс [DefaultAzureCredential()](https://docs.microsoft.com/javascript/api/@azure/identity/defaultazurecredential) из [библиотеки удостоверений Azure](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения о проверке подлинности в хранилище ключей см. в [руководстве для разработчиков](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

Добавьте следующий код в функцию main():

```javascript
const keyVaultName = process.env["KEY_VAULT_NAME"];
const KVUri = "https://" + keyVaultName + ".vault.azure.net";

const credential = new DefaultAzureCredential();
const client = new Certificate(KVUri, credential);
```

### <a name="save-a-certificate"></a>Сохранение сертификата

Теперь, когда приложение прошло проверку подлинности, можно поместить сертификат в хранилище ключей с помощью метода [beginCreateCertificate](/javascript/api/@azure/keyvault-certificates/certificateclient?#beginCreateCertificate_string__CertificatePolicy__BeginCreateCertificateOptions_). Для этого требуется имя и [политика](https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificatepolicy) сертификата со [свойствами этой политики](https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificatepolicyproperties).

```javascript
const certificatePolicy = {
  issuerName: "Self",
  subject: "cn=MyCert"
};
const createPoller = await client.beginCreateCertificate(certificateName, certificatePolicy);
const certificate = await poller.pollUntilDone();
```

> [!NOTE]
> Если сертификат с таким именем уже существует, выполните приведенный выше код, чтобы создать версию этого сертификата.
### <a name="retrieve-a-certificate"></a>Получение сертификата

Теперь вы можете получить ранее заданное значение с помощью метода [getCertificate](/javascript/api/@azure/keyvault-certificates/certificateclient?#getCertificate_string__GetCertificateOption).

```javascript
const retrievedCertificate = await client.getCertificate(certificateName);
 ```

### <a name="delete-a-certificate"></a>Удаление сертификата

Теперь удалите сертификат из хранилища ключей и очистите его с помощью методов [beginDeleteCertificate]https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificateclient?#beginDeleteCertificate_string__BeginDeleteCertificateOptions_) и [purgeDeletedCertificate](https://docs.microsoft.com/javascript/api/@azure/keyvault-certificates/certificateclient?#purgeDeletedCertificate_string__PurgeDeletedCertificateOptions_).

```javascript
const deletePoller = await client.beginDeleteCertificate(certificateName);
await deletePoller.pollUntilDone();
await client.purgeDeletedCertificate(certificateName);
```

## <a name="sample-code"></a>Образец кода

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { CertificateClient } = require("@azure/keyvault-certificates");

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

  const string certificateName = "myCertificate";
  const keyVaultName = process.env["KEY_VAULT_NAME"];
  const KVUri = "https://" + keyVaultName + ".vault.azure.net";

  const credential = new DefaultAzureCredential();
  const client = new CertificateClient(KVUri, credential);

  console.log("Creating a certificate in " + keyVaultName + " called '" + certificateName +  "` ...");
  const certificatePolicy = {
  issuerName: "Self",
  subject: "cn=MyCert"
  };
  const createPoller = await client.beginCreateCertificate(certificateName, certificatePolicy);
  const certificate = await poller.pollUntilDone();

  console.log("Done.");

  console.log("Retrieving your certificate from " + keyVaultName + ".");

  const retrievedCertificate = await client.getCertificate(certificateName);

  console.log("Your certificate version is '" + retrievedCertificate.properties.version + "'.");

  console.log("Deleting your certificate from " + keyVaultName + " ...");
  const deletePoller = await client.beginDeleteCertificate(certificateName);
  await deletePoller.pollUntilDone();
  console.log("Done.");
  
  console.log("Purging your certificate from {keyVaultName} ...");
  await client.purgeDeletedCertificate(certificateName);
  
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
Creating a certificate in mykeyvault called 'myCertificate' ... done.
Retrieving your certificate from mykeyvault.
Your certificate version is '8532359bced24e4bb2525f2d2050738a'.
Deleting your certificate from mykeyvault ... done
Purging your certificate from mykeyvault ... done 
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы создали хранилище ключей, сохранили в нем сертификат и извлекли его. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- [Обзор сертификатов](about-certificates.md)
- См. [Руководство по доступу к Key Vault из приложения службы приложений](../general/tutorial-net-create-vault-azure-web-app.md)
- См. [Руководство по доступу к Key Vault из виртуальной машины](../general/tutorial-net-virtual-machine.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)

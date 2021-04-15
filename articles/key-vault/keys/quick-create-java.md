---
title: Краткое руководство. Клиентская библиотека ключей Azure Key Vault для Java
description: Сведения о клиентской библиотеке ключей Azure Key Vault для Java.
author: msmbaldwin
ms.custom: devx-track-java
ms.author: mbaldwin
ms.date: 01/05/2021
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.openlocfilehash: 124e56fad35be0f3ac5b08ee9dd66454b9d077c5
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374698"
---
# <a name="quickstart-azure-key-vault-key-client-library-for-java"></a>Краткое руководство. Клиентская библиотека ключей Azure Key Vault для Java
Начало работы с клиентской библиотекой ключей Azure Key Vault для Java. Чтобы установить пакет и испробовать пример кода для выполнения базовых задач, выполните описанные ниже шаги.

Дополнительные ресурсы:

* [Исходный код](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-keys).
* [Справочная документация по API](https://azure.github.io/azure-sdk-for-java/keyvault.html)
* [Документация по продукту](index.yml)
* [Примеры](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-keys/src/samples/java/com/azure/security/keyvault/keys)

## <a name="prerequisites"></a>Предварительные требования
- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Пакет разработчиков Java (JDK)](/java/azure/jdk/) версии 8 или более поздней версии.
- [Apache Maven](https://maven.apache.org)
- [Azure CLI](/cli/azure/install-azure-cli)

В этом кратком руководстве предполагается, что вы используете [Azure CLI](/cli/azure/install-azure-cli) и [Apache Maven](https://maven.apache.org) в окне терминала Linux.

## <a name="setting-up"></a>Настройка
В этом кратком руководстве используется библиотека удостоверений Azure и Azure CLI для проверки подлинности пользователя в службах Azure. Разработчики также могут использовать Visual Studio или Visual Studio Code для проверки подлинности своих вызовов. Дополнительные сведения см. в статье [Проверка подлинности клиента с помощью клиентской библиотеки удостоверений Azure](/java/api/overview/azure/identity-readme).

### <a name="sign-in-to-azure"></a>Вход в Azure
1. Выполните команду `login`.

    ```azurecli-interactive
    az login
    ```

   Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа.

   В противном случае самостоятельно откройте в браузере страницу [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и введите код авторизации, отображаемый в терминале.

2. Выполните вход в браузере с помощью учетных данных.

### <a name="create-a-new-java-console-app"></a>Создание консольного приложения Java
В окне консоли выполните команду `mvn`, чтобы создать консольное приложение Java с именем `akv-keys-java`.

```console
mvn archetype:generate -DgroupId=com.keyvault.keys.quickstart
                       -DartifactId=akv-keys-java
                       -DarchetypeArtifactId=maven-archetype-quickstart
                       -DarchetypeVersion=1.4
                       -DinteractiveMode=false
```

Примерный результат создания проекта показан ниже:

```console
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.keyvault.keys.quickstart
[INFO] Parameter: artifactId, Value: akv-keys-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.keyvault.keys.quickstart
[INFO] Parameter: packageInPathFormat, Value: com/keyvault/quickstart
[INFO] Parameter: package, Value: com.keyvault.keys.quickstart
[INFO] Parameter: groupId, Value: com.keyvault.keys.quickstart
[INFO] Parameter: artifactId, Value: akv-keys-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Project created from Archetype in dir: /home/user/quickstarts/akv-keys-java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  38.124 s
[INFO] Finished at: 2019-11-15T13:19:06-08:00
[INFO] ------------------------------------------------------------------------
```

Измените каталог на созданную папку `akv-keys-java/`.

```console
cd akv-keys-java
```

### <a name="install-the-package"></a>Установка пакета
Откройте файл *pom.xml* в текстовом редакторе. Добавьте приведенные ниже элементы зависимости в группу зависимостей.

```xml
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-security-keyvault-keys</artifactId>
      <version>4.2.3</version>
    </dependency>

    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-identity</artifactId>
      <version>1.2.0</version>
    </dependency>
```

### <a name="create-a-resource-group-and-key-vault"></a>Создание группы ресурсов и хранилища ключей
[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

#### <a name="grant-access-to-your-key-vault"></a>Предоставление доступа к хранилищу ключей
Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование ключа.

```console
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --key-permissions delete get list create purge
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
```cmd
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="object-model"></a>Объектная модель
Клиентская библиотека ключей Azure Key Vault для Java позволяет управлять ключами. В разделе [Примеры кода](#code-examples) показано, как создать клиент, а также создать, получить и удалить ключ.

Полная версия консольного приложения представлена [ниже](#sample-code).

## <a name="code-examples"></a>Примеры кода
### <a name="add-directives"></a>Добавление директив
Добавьте в верхнюю часть кода следующие директивы:

```java
import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.DeletedKey;
import com.azure.security.keyvault.keys.models.KeyType;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
```

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента
В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в статье [Что такое управляемые удостоверения для ресурсов Azure?](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс ["DefaultAzureCredential()"](https://docs.microsoft.com/java/api/com.azure.identity.defaultazurecredential), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения см. в статье [Проверка подлинности учетных данных Azure по умолчанию](https://docs.microsoft.com/java/api/overview/azure/identity-readme).

```java
String keyVaultName = System.getenv("KEY_VAULT_NAME");
String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

KeyClient keyClient = new KeyClientBuilder()
    .vaultUrl(keyVaultUri)
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

### <a name="create-a-key"></a>Создание ключа
Теперь, когда приложение прошло аутентификацию, можно создать ключ в хранилище ключей с помощью метода `keyClient.createKey`. Для этого требуется имя и тип ключа — мы присвоили значение myKey переменной `keyName` и используем `KeyType` RSA в этом примере.

```java
keyClient.createKey(keyName, KeyType.RSA);
```

Команда [az keyvault key show](/cli/azure/keyvault/key?#az-keyvault-key-show) позволяет убедиться, что ключ задан успешно:

```azurecli
az keyvault key show --vault-name <your-unique-key-vault-name> --name myKey
```

### <a name="retrieve-a-key"></a>Получение ключа
Теперь вы можете получить ранее созданный ключ с помощью метода `keyClient.getKey`.

```java
KeyVaultKey retrievedKey = keyClient.getKey(keyName);
 ```

Теперь вы можете получить доступ к сведениям о полученном ключе с помощью таких операций, как `retrievedKey.getProperties`, `retrievedKey.getKeyOperations` и т. д.

### <a name="delete-a-key"></a>Удаление ключа
Наконец, мы удалим ключ из хранилища ключей с помощью метода `keyClient.beginDeleteKey`.

Удаление ключа — это длительная операция, для которой можно опросить ход выполнения или дождаться ее завершения.

```java
SyncPoller<DeletedKey, Void> deletionPoller = keyClient.beginDeleteKey(keyName);
deletionPoller.waitForCompletion();
```

Команда [az keyvault key show](/cli/azure/keyvault/key?#az-keyvault-key-show) позволяет убедиться, что ключ успешно удален:

```azurecli
az keyvault key show --vault-name <your-unique-key-vault-name> --name myKey
```

## <a name="clean-up-resources"></a>Очистка ресурсов
Если ресурсы больше не нужны, вы можете использовать интерфейс командной строки Azure или Azure PowerShell, чтобы удалить хранилище ключей и соответствующую группу ресурсов.

```azurecli
az group delete -g "myResourceGroup"
```

```azurepowershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="sample-code"></a>Образец кода
```java
package com.keyvault.keys.quickstart;

import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.DeletedKey;
import com.azure.security.keyvault.keys.models.KeyType;
import com.azure.security.keyvault.keys.models.KeyVaultKey;

public class App {
    public static void main(String[] args) throws InterruptedException, IllegalArgumentException {
        String keyVaultName = System.getenv("KEY_VAULT_NAME");
        String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

        System.out.printf("key vault name = %s and key vault URI = %s \n", keyVaultName, keyVaultUri);

        KeyClient keyClient = new KeyClientBuilder()
                .vaultUrl(keyVaultUri)
                .credential(new DefaultAzureCredentialBuilder().build())
                .buildClient();

        String keyName = "myKey";

        System.out.print("Creating a key in " + keyVaultName + " called '" + keyName + " ... ");

        keyClient.createKey(keyName, KeyType.RSA);

        System.out.print("done.");
        System.out.println("Retrieving key from " + keyVaultName + ".");

        KeyVaultKey retrievedKey = keyClient.getKey(keyName);

        System.out.println("Your key's ID is '" + retrievedKey.getId() + "'.");
        System.out.println("Deleting your key from " + keyVaultName + " ... ");

        SyncPoller<DeletedKey, Void> deletionPoller = keyClient.beginDeleteKey(keyName);
        deletionPoller.waitForCompletion();

        System.out.print("done.");
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
В процессе работы с этим кратким руководством вы создали хранилище ключей, создали ключ, извлекли его, а затем удалили. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- Изучите статью [Обзор системы безопасности Key Vault](../general/security-overview.md).
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- [Безопасный доступ к хранилищу ключей](../general/secure-your-key-vault.md)
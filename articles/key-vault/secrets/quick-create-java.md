---
title: Краткое руководство по использованию клиентской библиотеки секретов Azure Key Vault для Java
description: Краткое описание использования клиентской библиотеки секретов Azure Key Vault для Java
author: msmbaldwin
ms.custom: devx-track-java
ms.author: mbaldwin
ms.date: 10/20/2019
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.openlocfilehash: b45bb353ae32f1037fde7dc5d518472089edf12f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766341"
---
# <a name="quickstart-azure-key-vault-secret-client-library-for-java"></a>Краткое руководство. Использование клиентской библиотеки секретов Azure Key Vault для Java
Начало работы с клиентской библиотекой секретов Azure Key Vault для Java. Чтобы установить пакет и испробовать пример кода для выполнения базовых задач, выполните описанные ниже шаги.

Дополнительные ресурсы:

* [Исходный код](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets).
* [Справочная документация по API](https://azure.github.io/azure-sdk-for-java/keyvault.html)
* [Документация по продукту](index.yml)
* [Примеры](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets/src/samples/java/com/azure/security/keyvault/secrets)

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
В окне консоли выполните команду `mvn`, чтобы создать консольное приложение Java с именем `akv-secrets-java`.

```console
mvn archetype:generate -DgroupId=com.keyvault.secrets.quickstart
                       -DartifactId=akv-secrets-java
                       -DarchetypeArtifactId=maven-archetype-quickstart
                       -DarchetypeVersion=1.4
                       -DinteractiveMode=false
```

Примерный результат создания проекта показан ниже:

```console
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: artifactId, Value: akv-secrets-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: packageInPathFormat, Value: com/keyvault/quickstart
[INFO] Parameter: package, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: groupId, Value: com.keyvault.secrets.quickstart
[INFO] Parameter: artifactId, Value: akv-secrets-java
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Project created from Archetype in dir: /home/user/quickstarts/akv-secrets-java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  38.124 s
[INFO] Finished at: 2019-11-15T13:19:06-08:00
[INFO] ------------------------------------------------------------------------
```

Измените каталог на созданную папку `akv-secrets-java/`.

```console
cd akv-secrets-java
```

### <a name="install-the-package"></a>Установка пакета
Откройте файл *pom.xml* в текстовом редакторе. Добавьте приведенные ниже элементы зависимости в группу зависимостей.

```xml
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-security-keyvault-secrets</artifactId>
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
Создайте для хранилища ключей политику доступа, которая предоставляет учетной записи пользователя разрешения на использование секрета.

```console
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --secret-permissions delete get list set purge
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
Клиентская библиотека секретов Azure Key Vault для Java позволяет управлять секретами. В разделе [Примеры кода](#code-examples) показано, как создать клиент, задать секрет, получить секрет и удалить секрет.

Полная версия консольного приложения представлена [ниже](#sample-code).

## <a name="code-examples"></a>Примеры кода
### <a name="add-directives"></a>Добавление директив
Добавьте в верхнюю часть кода следующие директивы:

```java
import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.secrets.SecretClient;
import com.azure.security.keyvault.secrets.SecretClientBuilder;
import com.azure.security.keyvault.secrets.models.DeletedSecret;
import com.azure.security.keyvault.secrets.models.KeyVaultSecret;
```

### <a name="authenticate-and-create-a-client"></a>Аутентификация и создание клиента
В этом кратком руководстве пользователь, вошедший в систему, проходит проверку подлинности в хранилище ключей, который является предпочтительным методом при локальной разработке. Для приложений, развернутых в Azure, службе приложений или виртуальной машине должно быть назначено управляемое удостоверение. Дополнительные сведения см. в статье [Что такое управляемые удостоверения для ресурсов Azure?](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

В примере ниже имя хранилища ключей расширяется до универсального кода ресурса (URI) хранилища ключей в формате "https://\<your-key-vault-name\>.vault.azure.net". В этом примере показан класс ["DefaultAzureCredential()"](https://docs.microsoft.com/java/api/com.azure.identity.defaultazurecredential), который для предоставления удостоверения позволяет использовать один и тот же код в различных средах с различными параметрами. Дополнительные сведения см. в статье [Проверка подлинности учетных данных Azure по умолчанию](https://docs.microsoft.com/java/api/overview/azure/identity-readme).

```java
String keyVaultName = System.getenv("KEY_VAULT_NAME");
String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

SecretClient secretClient = new SecretClientBuilder()
    .vaultUrl(keyVaultUri)
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

### <a name="save-a-secret"></a>Сохранение секрета
Теперь, когда приложение прошло проверку подлинности, можно добавить секрет в хранилище ключей с помощью метода `secretClient.setSecret`. Для этого требуется имя секрета. Мы присвоили значение mySecret переменной `secretName` в этом примере.

```java
secretClient.setSecret(new KeyVaultSecret(secretName, secretValue));
```

Команда [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show) позволяет убедиться, что секрет задан успешно:

```azurecli
az keyvault secret show --vault-name <your-unique-key-vault-name> --name mySecret
```

### <a name="retrieve-a-secret"></a>Получение секрета
Теперь вы можете получить ранее заданный секрет с помощью метода `secretClient.getSecret`.

```java
KeyVaultSecret retrievedSecret = secretClient.getSecret(secretName);
 ```

Теперь можно получить доступ к значению полученного секрета с помощью `retrievedSecret.getValue()`.

### <a name="delete-a-secret"></a>Удаление секрета.
Наконец, давайте удалим секрет из хранилища ключей с помощью метода `secretClient.beginDeleteSecret`.

Удаление секрета — это длительная операция, для которой можно опросить ход выполнения или дождаться ее завершения.

```java
SyncPoller<DeletedSecret, Void> deletionPoller = secretClient.beginDeleteSecret(secretName);
deletionPoller.waitForCompletion();
```

Команда [az keyvault secret show](/cli/azure/keyvault/secret?#az_keyvault_secret_show) позволяет убедиться, что секрет удален успешно:

```azurecli
az keyvault secret show --vault-name <your-unique-key-vault-name> --name mySecret
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
package com.keyvault.secrets.quickstart;

import java.io.Console;

import com.azure.core.util.polling.SyncPoller;
import com.azure.identity.DefaultAzureCredentialBuilder;

import com.azure.security.keyvault.secrets.SecretClient;
import com.azure.security.keyvault.secrets.SecretClientBuilder;
import com.azure.security.keyvault.secrets.models.DeletedSecret;
import com.azure.security.keyvault.secrets.models.KeyVaultSecret

public class App {
    public static void main(String[] args) throws InterruptedException, IllegalArgumentException {
        String keyVaultName = System.getenv("KEY_VAULT_NAME");
        String keyVaultUri = "https://" + keyVaultName + ".vault.azure.net";

        System.out.printf("key vault name = %s and key vault URI = %s \n", keyVaultName, keyVaultUri);

        SecretClient secretClient = new SecretClientBuilder()
            .vaultUrl(keyVaultUri)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();

        Console con = System.console();  

        String secretName = "mySecret";

        System.out.println("Please provide the value of your secret > ");
        
        String secretValue = con.readLine();

        System.out.print("Creating a secret in " + keyVaultName + " called '" + secretName + "' with value '" + secretValue + "` ... ");

        secretClient.setSecret(new KeyVaultSecret(secretName, secretValue));

        System.out.println("done.");
        System.out.println("Forgetting your secret.");
        
        secretValue = "";
        System.out.println("Your secret's value is '" + secretValue + "'.");

        System.out.println("Retrieving your secret from " + keyVaultName + ".");

        KeyVaultSecret retrievedSecret = secretClient.getSecret(secretName);

        System.out.println("Your secret's value is '" + retrievedSecret.getValue() + "'.");
        System.out.print("Deleting your secret from " + keyVaultName + " ... ");

        SyncPoller<DeletedSecret, Void> deletionPoller = secretClient.beginDeleteSecret(secretName);
        deletionPoller.waitForCompletion();

        System.out.println("done.");
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
При работе с этим кратким руководством вы создали хранилище ключей, сохранили в нем секрет, извлекли его, а затем удалили. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- [Руководство разработчика Azure Key Vault](../general/developers-guide.md)
- [Безопасный доступ к хранилищу ключей](../general/security-overview.md)

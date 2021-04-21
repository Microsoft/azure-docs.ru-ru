---
title: Руководство. Управляемое удостоверение для подключения Key Vault
description: Настройка управляемого удостоверения для подключения Key Vault к приложению Azure Spring Cloud
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 07/08/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 870a04af244d18826e1041316895f746e27870eb
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786591"
---
# <a name="tutorial-use-a-managed-identity-to-connect-key-vault-to-an-azure-spring-cloud-app"></a>Руководство по использованию управляемого удостоверения для подключения Key Vault к приложению Azure Spring Cloud

**Эта статья применима к:** ✔️ Java

В этой статье показано, как создать управляемое удостоверение для приложения Azure Spring Cloud и использовать его для получения доступа к Azure Key Vault.

В Azure Key Vault можно безопасно хранить токены, пароли, сертификаты, ключи API и другие секреты для приложения, а также контролировать доступ к этой информации. Вы можете создать управляемое удостоверение в Azure Active Directory (AAD) и пройти проверку подлинности в любой службе, поддерживающей проверку подлинности AAD, в том числе Key Vault, без необходимости отображать учетные данные в коде.

## <a name="prerequisites"></a>Предварительные требования

* [Регистрация для получения подписки Azure](https://azure.microsoft.com/free/)
* [установите Azure CLI (версии 2.0.67 или выше)](/cli/azure/install-azure-cli);
* [установите Maven 3.0 или более поздней версии](https://maven.apache.org/download.cgi);

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Создайте группу ресурсов, которая будет содержать Key Vault и Spring Cloud, с помощью команды [az group create](/cli/azure/group#az_group_create).

```azurecli-interactive
az group create --name "myResourceGroup" -l "EastUS"
```

## <a name="set-up-your-key-vault"></a>Настройка Key Vault
Чтобы создать Key Vault, используйте команду [az keyvault create](/cli/azure/keyvault#az_keyvault_create).

> [!Important]
> Каждый Key Vault должен иметь уникальное имя. В следующих примерах замените заполнитель <your-keyvault-name> именем своего экземпляра Key Vault.

```azurecli-interactive
az keyvault create --name "<your-keyvault-name>" -g "myResourceGroup"
```

Запишите полученное значение `vaultUri` в формате https://<your-keyvault-name>.vault.azure.net. Оно понадобится вам позже.

Теперь поместите секрет в Key Vault с помощью команды [az keyvault secret set](/cli/azure/keyvault/secret#az_keyvault_secret_set).

```azurecli-interactive
az keyvault secret set --vault-name "<your-keyvault-name>" \
    --name "connectionString" \
    --value "jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE;"
```

## <a name="create-azure-spring-cloud-service-and-app"></a>Создание приложения и службы Azure Spring Cloud
Установив соответствующее расширение, создайте экземпляр Azure Spring Cloud с помощью команды `az spring-cloud create` Azure CLI. 

```azurecli-interactive
az extension add --name spring-cloud
az spring-cloud create -n "myspringcloud" -g "myResourceGroup"
```
В приведенном ниже примере создается приложение `springapp` с управляемым удостоверением, назначаемым системой, как определено параметром `--assign-identity`.

```azurecli
az spring-cloud app create -n "springapp" -s "myspringcloud" -g "myResourceGroup" --assign-endpoint true --assign-identity
export SERVICE_IDENTITY=$(az spring-cloud app show --name "springapp" -s "myspringcloud" -g "myResourceGroup" | jq -r '.identity.principalId')
```

Запишите полученное значение `url` в формате `https://<your-app-name>.azuremicroservices.io`. Оно понадобится вам позже.


## <a name="grant-your-app-access-to-key-vault"></a>Предоставление приложению доступа к Key Vault
Используйте `az keyvault set-policy`, чтобы предоставить приложению доступ к Key Vault.
```azurecli
az keyvault set-policy --name "<your-keyvault-name>" --object-id ${SERVICE_IDENTITY} --secret-permissions set get list
```
> [!NOTE]
> Используйте `az keyvault delete-policy --name "<your-keyvault-name>" --object-id ${SERVICE_IDENTITY}`, чтобы удалить доступ к приложению после отключения удостоверения, управляемого системой.

## <a name="build-a-sample-spring-boot-app-with-spring-boot-starter"></a>Создание примера приложения Spring Boot с помощью стартера Spring Boot
Этому приложению будет предоставлен доступ на получение секретов из Azure Key Vault. См. сведения об использовании [стартера Spring Boot для секретов Azure Key Vault](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-keyvault-secrets).  Azure Key Vault добавляется в качестве экземпляра **PropertySource** Spring.  Секреты, хранимые в Azure Key Vault, можно легко получить и использовать как любое внешнее свойство конфигурации, например свойства в файлах. 

1. Создайте пример проекта на основе start.spring.io с помощью стартеров Spring для Azure Key Vault. 
    ```azurecli
    curl https://start.spring.io/starter.tgz -d dependencies=web,azure-keyvault-secrets -d baseDir=springapp -d bootVersion=2.3.1.RELEASE -d javaVersion=1.8 | tar -xzvf -
    ```

2. Определите Key Vault в приложении. 

    ```azurecli
    cd springapp
    vim src/main/resources/application.properties
    ```

    Чтобы использовать управляемое удостоверение для приложений Azure Spring Cloud, добавьте свойства со следующим содержимым в src/main/resources/application.properties.

    ```
    azure.keyvault.enabled=true
    azure.keyvault.uri=https://<your-keyvault-name>.vault.azure.net
    ```
    > [!Note] 
    > Вам нужно добавить URL-адрес хранилища ключей в `application.properties`, как показано выше. В противном случае URL-адрес хранилища ключей не будет использоваться во время выполнения.

3. Добавьте пример кода в src/main/java/com/example/demo/DemoApplication.java. Он получает строку подключения из Key Vault. 

    ```Java
    package com.example.demo;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

    @SpringBootApplication
    @RestController
    public class DemoApplication implements CommandLineRunner {

        @Value("${connectionString}")
        private String connectionString;

        public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
        }

        @GetMapping("get")
        public String get() {
          return connectionString;
        }

        public void run(String... varl) throws Exception {
          System.out.println(String.format("\nConnection String stored in Azure Key Vault:\n%s\n",connectionString));
        }
      }
    ```

    При открытии pom.xml вы увидите зависимость от `azure-keyvault-secrets-spring-boot-starter`. Добавьте эту зависимость в проект в pom.xml. 

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-keyvault-secrets-spring-boot-starter</artifactId>
    </dependency>
    ```

4. Упакуйте пример приложения. 

    ```azurecli
    mvn clean package
    ```

5. Теперь вы можете развернуть приложение в Azure с помощью команды `az spring-cloud app deploy` Azure CLI. 

    ```azurecli
    az spring-cloud app deploy -n "springapp" -s "myspringcloud" -g "myResourceGroup" --jar-path target/demo-0.0.1-SNAPSHOT.jar
    ```

6.  Чтобы протестировать приложение, получите доступ к общедоступной конечной точке или тестовой конечной точке.

    ```azurecli
    curl https://myspringcloud-springapp.azuremicroservices.io/get
    ```

    Вы увидите сообщение с информацией о том, что значение секретной строки подключения от Key Vault успешно получено (Successfully got the value of secret connectionString from Key Vault https://<your-keyvault-name>.vault.azure.net/: jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE;).

## <a name="build-sample-spring-boot-app-with-java-sdk"></a>Создание примера приложения Spring Boot с помощью пакета SDK для Java

Этот пример может определять и получать секреты из Azure Key Vault. [Клиентская библиотека секретов Azure Key Vault для Java](/java/api/overview/azure/security-keyvault-secrets-readme) поддерживает проверку подлинности Azure Active Directory на основе токенов в пакете SDK Azure. Она предоставляет набор реализаций **TokenCredential**, которые можно использовать для создания клиентов пакета SDK Azure и включения поддержки проверки подлинности AAD на основе токенов.

Эта библиотека позволяет безопасно хранить токены, пароли, ключи API и другие секреты, а также контролировать доступ к этой информации. Библиотека предоставляет операции для создания, извлечения, обновления, удаления, очистки, резервного копирования, восстановления и перечисления секретов и их версий.

1. Клонируйте пример проекта. 

    ```azurecli
    git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples.git
    ```

2. Определите Key Vault в приложении. 

    ```azurecli
    cd Azure-Spring-Cloud-Samples/managed-identity-keyvault
    vim src/main/resources/application.properties
    ```

    Чтобы использовать управляемое удостоверение для приложений Azure Spring Cloud, добавьте свойства со следующим содержимым в *src/main/resources/application.properties*.

    ```
    azure.keyvault.enabled=true
    azure.keyvault.uri=https://<your-keyvault-name>.vault.azure.net
    ```

3. Включите [ManagedIdentityCredentialBuilder](/java/api/com.azure.identity.managedidentitycredentialbuilder), чтобы получить токен из Azure Active Directory и [SecretClientBuilder](/java/api/com.azure.security.keyvault.secrets.secretclientbuilder), чтобы определить или получить секреты из Key Vault в коде.

    Получите пример из [MainController.java](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/managed-identity-keyvault/src/main/java/com/microsoft/azure/MainController.java#L28) клонированного примера проекта.

    Также включите `azure-identity` и `azure-security-keyvault-secrets` в качестве зависимости в pom.xml. Получите пример из [pom.xml](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/managed-identity-keyvault/pom.xml#L21) клонированного примера проекта. 

4. Упакуйте пример приложения. 

    ```azurecli
    mvn clean package
    ```

5. Теперь вы разверните приложение в Azure с помощью команды `az spring-cloud app deploy` Azure CLI. 

    ```azurecli
    az spring-cloud app deploy -n "springapp" -s "myspringcloud" -g "myResourceGroup" --jar-path target/asc-managed-identity-keyvault-sample-0.1.0.jar
    ```

6. Чтобы протестировать приложение, получите доступ к общедоступной конечной точке или тестовой конечной точке. 

    Сначала получите значение секрета, определенное в Azure Key Vault. 
    ```azurecli
    curl https://myspringcloud-springapp.azuremicroservices.io/secrets/connectionString
    ```

    Вы увидите сообщение с информацией о том, что значение секретной строки подключения от Key Vault успешно получено (Successfully got the value of secret connectionString from Key Vault https://<your-keyvault-name>.vault.azure.net/: jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE;).

    Теперь создайте секрет, а затем извлеките его с помощью пакета SDK для Java. 
    ```azurecli
    curl -X PUT https://myspringcloud-springapp.azuremicroservices.io/secrets/test?value=success

    curl https://myspringcloud-springapp.azuremicroservices.io/secrets/test
    ```

    Вы увидите сообщение с информацией о том, что значение секретного теста от Key Vault успешно получено (Successfully got the value of secret connectionString from Key Vault https://<your-keyvault-name>.vault.azure.net: success). 

## <a name="next-steps"></a>Дальнейшие действия

* [Как получить доступ к большому двоичному объекту службы хранилища с помощью управляемого удостоверения в Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/managed-identity-storage-blob)
* [Как включить управляемое удостоверение, назначенное системой, для приложения Azure Spring Cloud](./spring-cloud-howto-enable-system-assigned-managed-identity.md)
* [Дополнительные сведения об управляемых удостоверениях для ресурсов Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Проверка подлинности Azure Spring Cloud с использованием Key Vault в GitHub Actions](./spring-cloud-github-actions-key-vault.md)

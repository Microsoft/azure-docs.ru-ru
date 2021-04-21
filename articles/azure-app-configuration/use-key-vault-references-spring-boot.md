---
title: Учебник по использованию ссылок Key Vault службы "Конфигурация приложений Azure" в приложении Java Spring Boot | Документация Майкрософт
description: В этом учебнике описано, как использовать ссылки Key Vault службы "Конфигурация приложений Azure" из приложения Java Spring Boot.
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 08/11/2020
ms.author: alkemper
ms.custom: mvc, devx-track-java, devx-track-azurecli
ms.openlocfilehash: 7c5534ab836968bc4e72a54db1ddb9667d366558
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768855"
---
# <a name="tutorial-use-key-vault-references-in-a-java-spring-app"></a>Руководство по Использование ссылок Key Vault в приложении Java Spring

В этом учебнике вы узнаете, как использовать службу "Конфигурация приложений Azure" с Azure Key Vault. Конфигурация приложений Azure и Key Vault — это дополняющие службы, которые используются параллельно в большинстве развертываний приложений.

Служба "Конфигурация приложений Azure" упрощает их использование, позволяя создавать ключи, ссылающиеся на значения, хранящиеся в Key Vault. Когда служба "Конфигурация приложений Azure" создает такие ключи, она сохраняет URI значений Key Vault, а не сами значения.

С помощью поставщика клиента "Конфигурация приложений Azure" ваше приложение получает ссылки на Key Vault так же, как и любые другие ключи, хранящиеся в службе "Конфигурация приложений Azure". В этом случае значения, хранящиеся в службе "Конфигурация приложений Azure", являются URI, которые ссылаются на значения в Key Vault. Это не значения или учетные данные Key Vault. Так как поставщик клиента распознает ключи как ссылки на Key Vault, для получения их значений он использует Key Vault.

Приложение отвечает за правильную проверку подлинности как в службе "Конфигурация приложений", так и в Key Vault. Две службы не обмениваются данными напрямую.

В этом учебнике показано, как можно реализовать ссылки на Key Vault в коде. При этом в качестве основы используется код веб-приложения, представленный в кратких руководствах. Прежде чем продолжить, прочитайте краткое руководство [Создание приложения Java Spring с помощью службы "Конфигурация приложений Azure"](./quickstart-java-spring-app.md).

Вы можете выполнять шаги в этом учебнике с помощью любого редактора кода. Например, [Visual Studio Code](https://code.visualstudio.com/) — это кроссплатформенный редактор кода, доступный для операционных систем Windows, macOS и Linux.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание ключа службы "Конфигурация приложений Azure", который ссылается на значение, хранящееся в Key Vault;
> * доступ к значению этого ключа из веб-приложения Java Spring.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
* Поддерживаемый [комплект разработчика Java (JDK)](/java/azure/jdk) версии 8.
* [Apache Maven](https://maven.apache.org/download.cgi) версии 3.0 или более поздней.

## <a name="create-a-vault"></a>Создание хранилища

1. Выберите параметр **Создать ресурс** в верхнем левом углу окна портала Azure.

    ![Снимок экрана: вариант "Создать ресурс" на портале Azure.](./media/quickstarts/search-services.png)
1. В поле поиска введите **Key Vault**.
1. В списке результатов выберите **Key Vault** слева.
1. В разделе **Хранилища ключей** выберите **Добавить**.
1. Справа в разделе **Создать Key Vault** введите приведенные ниже сведения.
    * Выберите **Подписка**, чтобы выбрать подписку.
    * В разделе **Группа ресурсов** выберите **Создать** и введите имя группы ресурсов.
    * В поле **Имя Key Vault** укажите уникальное имя. Для работы с этим учебником введите **Contoso-vault2**.
    * В раскрывающемся списке **Регион** выберите расположение.
1. Для других свойств раздела **создания Key Vault** оставьте значения по умолчанию.
1. Нажмите кнопку **создания**.

На этом этапе доступ к этому новому хранилищу может получать только учетная запись Azure.

![Снимок экрана: ваше хранилище ключей.](./media/quickstarts/vault-properties.png)

## <a name="add-a-secret-to-key-vault"></a>Добавление секрета в Key Vault

Чтобы добавить секрет в хранилище, вам просто нужно выполнить несколько дополнительных шагов. В этом случае мы добавим сообщение, которое можно использовать для проверки возвращения Key Vault. Сообщение называется **Message**, и в нем сохраняется значение "Hello from Key Vault".

1. На странице свойств Key Vault выберите **Секреты**.
1. Выберите **Создать или импортировать**.
1. В области **Создание секрета** выберите следующие значения:
    * **Параметры отправки.** Введите **Вручную**.
    * **Name** (Имя). Введите **Message**.
    * **Value** (Значение): Введите **Hello from Key Vault**.
1. Для других свойств раздела **Создание секрета** оставьте значения по умолчанию.
1. Нажмите кнопку **создания**.

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Добавление ссылки на Key Vault в службу "Конфигурация приложений Azure"

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Все ресурсы** и выберите экземпляр хранилища "Конфигурация приложений Azure", который вы создали по инструкциям из краткого руководства.

1. Выберите **Обозреватель конфигураций**.

1. Выберите **+ Создать** > **Ссылка на хранилище ключей** и укажите следующие значения:
    * **Ключ**: Выберите **/application/config.keyvaultmessage**
    * **Метка** — оставьте это значение пустым.
    * **Подписка**, **Группа ресурсов** и **Хранилище ключей**: введите значения, соответствующие значениям хранилища ключей, созданного в предыдущем разделе.
    * **Секрет**: выберите секрет с именем **Message**, созданный в предыдущем разделе.

## <a name="connect-to-key-vault"></a>Подключение к Key Vault

1. В этом учебнике вы используете субъект-службу для проверки подлинности в KeyVault. Создайте субъект-службу с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) Azure CLI:

    ```azurecli
    az ad sp create-for-rbac -n "http://mySP" --sdk-auth
    ```

    Эта операция возвращает ряд пар "ключ — значение".

    ```json
    {
    "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
    "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
    "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
    "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
    }
    ```

1. Выполните следующую команду, чтобы предоставить субъекту-службе доступ к хранилищу ключей:

    ```azurecli
    az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --secret-permissions delete get
    ```

1. Выполните следующую команду, чтобы получить идентификатор объекта, затем добавьте его в Конфигурацию приложений.

    ```azurecli
    az ad sp show --id <clientId-of-your-service-principal>
    az role assignment create --role "App Configuration Data Reader" --assignee-object-id <objectId-of-your-service-principal> --resource-group <your-resource-group>
    ```

1. Создайте переменные среды **AZURE_CLIENT_ID**, **AZURE_CLIENT_SECRET** и **AZURE_TENANT_ID**. Для этого используйте значения для субъекта-службы, которые были показаны на предыдущих шагах. В командной строке выполните следующие команды и перезапустите командную строку, чтобы изменения вступили в силу:

    ```cmd
    setx AZURE_CLIENT_ID "clientId"
    setx AZURE_CLIENT_SECRET "clientSecret"
    setx AZURE_TENANT_ID "tenantId"
    ```

    Если вы используете Windows PowerShell, выполните следующую команду:

    ```azurepowershell
    $Env:AZURE_CLIENT_ID = "clientId"
    $Env:AZURE_CLIENT_SECRET = "clientSecret"
    $Env:AZURE_TENANT_ID = "tenantId"
    ```

    Если вы используете macOS или Linux, выполните следующую команду:

    ```cmd
    export AZURE_CLIENT_ID ='clientId'
    export AZURE_CLIENT_SECRET ='clientSecret'
    export AZURE_TENANT_ID ='tenantId'
    ```


> [!NOTE]
> Эти учетные данные Key Vault используются только в вашем приложении.  Приложение выполняет проверку подлинности напрямую с Key Vault, используя эти учетные данные без привлечения службы "Конфигурация приложений".  Key Vault обеспечивает проверку подлинности как для приложения, так и для службы "Конфигурация приложений" без общего доступа или предоставления ключей.

## <a name="update-your-code-to-use-a-key-vault-reference"></a>Обновление кода для использования ссылки Key Vault

1. Создайте переменную среды с именем **APP_CONFIGURATION_ENDPOINT**. Задайте в качестве значения конечную точку хранилища Конфигурации приложений. Конечную точку можно найти в колонке **Ключи доступа** на портале Azure. Перезапустите командную строку, чтобы изменение вступило в силу. 


1. Откройте *bootstrap.properties* в папке *resources*. Обновите этот файл, чтобы использовать значение **APP_CONFIGURATION_ENDPOINT**. Удалите все ссылки на строку подключения в этом файле. 

    ```properties
    spring.cloud.azure.appconfiguration.stores[0].endpoint= ${APP_CONFIGURATION_ENDPOINT}
    ```

1. Откройте *MessageProperties.java*. Добавьте новую переменную с именем *keyVaultMessage*:

    ```java
    private String keyVaultMessage;

    public String getKeyVaultMessage() {
        return keyVaultMessage;
    }

    public void setKeyVaultMessage(String keyVaultMessage) {
        this.keyVaultMessage = keyVaultMessage;
    }
    ```

1. Откройте *HelloController.java*. Обновите метод *getMessage*, чтобы включить сообщение, полученное от Key Vault.

    ```java
    @GetMapping
    public String getMessage() {
        return "Message: " + properties.getMessage() + "\nKey Vault message: " + properties.getKeyVaultMessage();
    }
    ```

1. Создайте файл с именем *AzureCredentials.java* и добавьте приведенный ниже код.

    ```java
    package com.example.demo;

    import com.azure.core.credential.TokenCredential;
    import com.azure.identity.EnvironmentCredentialBuilder;
    import com.microsoft.azure.spring.cloud.config.AppConfigurationCredentialProvider;
    import com.microsoft.azure.spring.cloud.config.KeyVaultCredentialProvider;

    public class AzureCredentials implements AppConfigurationCredentialProvider, KeyVaultCredentialProvider{

        @Override
        public TokenCredential getKeyVaultCredential(String uri) {
            return getCredential();
        }

        @Override
        public TokenCredential getAppConfigCredential(String uri) {
            return getCredential();
        }

        private TokenCredential getCredential() {
            return new EnvironmentCredentialBuilder().build();
        }

    }
    ```

1. Создайте файл с именем *AppConfiguration.java*. И добавьте приведенный ниже код.

    ```java
    package com.example.demo;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class AppConfiguration {

        @Bean
        public AzureCredentials azureCredentials() {
            return new AzureCredentials();
        }
    }
    ```

1. Создайте файл в каталоге ресурсов META-INF с именем *spring.factories* и добавьте приведенный ниже код.

    ```factories
    org.springframework.cloud.bootstrap.BootstrapConfiguration=\
    com.example.demo.AppConfiguration
    ```

1. Создайте приложение Spring Boot с помощью Maven и запустите его, например, следующим образом:

    ```shell
    mvn clean package
    mvn spring-boot:run
    ```

1. После запуска приложение можно протестировать с помощью средства *curl*, например:

      ```shell
      curl -X GET http://localhost:8080/
      ```

    Вы увидите сообщение, которое указали в хранилище конфигураций приложений. Вы также увидите сообщение, которое ввели в Key Vault.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы создали ключ службы "Конфигурация приложений Azure", который ссылается на значение, хранящееся в Key Vault. Чтобы узнать, как использовать флаги функций в приложении Java Spring, перейдите к следующему руководству.

> [!div class="nextstepaction"]
> [Руководство по интеграции с управляемыми удостоверениями Azure](./quickstart-feature-flag-spring-boot.md)

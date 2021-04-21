---
title: Руководство. Управляемое удостоверение для вызова Функций Azure
description: Использование управляемого удостоверения для вызова Функций Azure из приложения Azure Spring Cloud
author: MarkGardner
ms.author: margard
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 07/10/2020
ms.openlocfilehash: b737ea751d3b3d2132691e04a1a2cd853748db65
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107792513"
---
# <a name="tutorial-use-a-managed-identity-to-invoke-azure-functions-from-an-azure-spring-cloud-app"></a>Руководство по использованию управляемого удостоверения для вызова Функций Azure из приложения Azure Spring Cloud

В этой статье показано, как создать управляемое удостоверение для приложения Azure Spring Cloud и использовать его для вызова функций, активируемых по протоколу HTTP.

Функции Azure и службы приложений имеют встроенную поддержку проверки подлинности Azure Active Directory (Azure AD). Используя встроенную функцию проверки подлинности вместе с управляемыми удостоверениями для Azure Spring Cloud, мы сможем вызывать службы RESTful с помощью современных семантик OAuth. Этот метод не требует хранения секретов в коде и предоставляет более детализированные элементы управления доступом к внешним ресурсам. 


## <a name="prerequisites"></a>Предварительные требования

* [Регистрация для получения подписки Azure](https://azure.microsoft.com/free/)
* [установите Azure CLI (версии 2.0.67 или выше)](/cli/azure/install-azure-cli);
* [установите Maven 3.0 или более поздней версии](https://maven.apache.org/download.cgi);
* [Установите Azure Functions Core Tools версии 3.0.2009 или более поздней](../azure-functions/functions-run-local.md#install-the-azure-functions-core-tools)


## <a name="create-a-resource-group"></a>Создание группы ресурсов
Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Создайте группу ресурсов, которая будет содержать приложение-функцию и Spring Cloud, с помощью команды [az group create](/cli/azure/group#az_group_create).

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```


## <a name="create-a-function-app"></a>Создание приложения-функции
Чтобы создать приложение-функцию, необходимо сначала создать учетную запись хранения для архивации. Для этого выполните команду [az storage account create](/cli/azure/storage/account#az_storage_account_create):

> [!Important]
> Каждое приложение-функция и каждая учетная запись хранения должны иметь уникальные имена. Замените в следующих примерах <your-functionapp-name> именем своего приложения-функции, а <your-storageaccount-name> — именем вашей учетной записи хранения.

```azurecli-interactive
az storage account create --name <your-storageaccount-name> --resource-group myResourceGroup --location eastus --sku Standard_LRS
```

Создав учетную запись хранения, можно приступить к созданию приложения-функции.

```azurecli-interactive
az functionapp create --name <your-functionapp-name> --resource-group myResourceGroup --consumption-plan-location eastus --os-type windows --runtime node --storage-account <your-storageaccount-name> --functions-version 3
```

Запишите возвращенные **имена узлов** в формате "https://<your-functionapp-name>.azurewebsites.net". Они понадобятся вам позже.


## <a name="enable-azure-active-directory-authentication"></a>Включение проверки подлинности Azure Active Directory

Откройте только что созданное приложение-функцию на [портале Azure](https://portal.azure.com) и выберите в меню параметров пункт "Проверка подлинности/авторизация". Включите проверку подлинности Службы приложений Azure и установите для параметра "Предпринимаемое действие, если проверка подлинности для запроса не выполнена" значение "Войти с использованием Azure Active Directory". Этот параметр обеспечит отклонение всех запросов, не прошедших проверку подлинности (ответ 401).

![Параметры проверки подлинности, отображающие Azure Active Directory в качестве поставщика по умолчанию](media/spring-cloud-tutorial-managed-identities-functions/function-auth-config-1.jpg)

В разделе "Поставщики проверки подлинности" выберите Azure Active Directory, чтобы настроить регистрацию приложения. Выбор режима управления "Экспресс" приведет к автоматическому созданию регистрации приложения в клиенте Azure AD с правильной конфигурацией.

![Поставщик Azure Active Directory с заданным режимом управления "Экспресс"](media/spring-cloud-tutorial-managed-identities-functions/function-auth-config-2.jpg)

После сохранения параметров приложение-функция запустится и все последующие запросы будут требовать входа через Azure AD. Вы можете убедиться, что запросы, не прошедшие проверку подлинности, теперь будут отклоняться, перейдя к корневому URL-адресу приложения-функции (возвращаемому в выходных данных **имен узлов** на предыдущем шаге). После этого вы должны перейти на экран входа Azure AD вашей организации.


## <a name="create-an-http-triggered-function"></a>Создание функции, активируемой HTTP

Создайте новое приложение-функцию в пустом локальном каталоге и добавьте функцию, активируемую HTTP.

```console
func init --worker-runtime node
func new --template HttpTrigger --name HttpTrigger
```

По умолчанию Функции используют проверку подлинности на основе ключей, чтобы обезопасить конечные точки HTTP. Поскольку мы будем включать проверку подлинности Azure AD, чтобы предоставить безопасный доступ к Функциям, нам понадобится [установить для уровня проверки подлинности функции значение "анонимно"](../azure-functions/functions-bindings-http-webhook-trigger.md#secure-an-http-endpoint-in-production).

```json function.json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      ...
    }
  ]
}
```

Теперь приложение можно опубликовать в экземпляре [приложения-функции](#create-a-function-app), созданном на предыдущем шаге.

```console
func azure functionapp publish <your-functionapp-name>
```

URL-адрес только что созданной функции должен быть указан в выходных данных команды publish.

```output
Deployment completed successfully.
Syncing triggers...
Functions in <your-functionapp-name>:
    HttpTrigger - [httpTrigger]
        Invoke url: https://<your-functionapp-name>.azurewebsites.net/api/httptrigger
```


## <a name="create-azure-spring-cloud-service-and-app"></a>Создание приложения и службы Azure Spring Cloud
Установив расширение spring-cloud, создайте экземпляр Azure Spring Cloud с помощью команды `az spring-cloud create`Azure CLI. 

```azurecli-interactive
az extension add --name spring-cloud
az spring-cloud create --name mymsispringcloud --resource-group myResourceGroup --location eastus
```

В приведенном ниже примере создается приложение `msiapp` с управляемым удостоверением, назначаемым системой, как определено параметром `--assign-identity`.

```azurecli
az spring-cloud app create --name "msiapp" --service "mymsispringcloud" --resource-group "myResourceGroup" --assign-endpoint true --assign-identity
```

## <a name="build-sample-spring-boot-app-to-invoke-the-function"></a>Сборка примера приложения Spring Boot для вызова Функции

Этот пример вызывает функцию, активируемую HTTP, запрашивая маркер доступа из [конечной точки MSI](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md#get-a-token-using-http) и используя его для проверки подлинности HTTP-запроса Функции.

1. Клонируйте пример проекта. 

    ```console
    git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples.git
    ```

2. Укажите URI функции и имя триггера в свойствах приложения. 

    ```bash
    cd Azure-Spring-Cloud-Samples/managed-identity-function
    vim src/main/resources/application.properties
    ```

    Чтобы использовать управляемое удостоверение для приложений Azure Spring Cloud, добавьте свойства со следующим содержимым в *src/main/resources/application.properties*.

    ```
    azure.function.uri=https://<your-functionapp-name>.azurewebsites.net
    azure.function.triggerPath=httptrigger
    ```

3. Упакуйте пример приложения. 

    ```console
    mvn clean package
    ```

4. Теперь вы разверните приложение в Azure с помощью команды `az spring-cloud app deploy` Azure CLI. 

    ```azurecli
    az spring-cloud app deploy  --name "msiapp" --service "mymsispringcloud" --resource-group "myResourceGroup" --jar-path target/sc-managed-identity-function-sample-0.1.0.jar
    ```

5. Чтобы протестировать приложение, получите доступ к общедоступной конечной точке или тестовой конечной точке. 

    ```console
    curl https://mymsispringcloud-msiapp.azuremicroservices.io/func/springcloud
    ```

    В тексте ответа будет выведено следующее сообщение.
    ```output
    Function Response: Hello, springcloud. This HTTP triggered function executed successfully.
    ```
    
    Вы можете попробовать передать функции другие значения, изменив параметр пути.

## <a name="next-steps"></a>Дальнейшие действия

* [Как включить управляемое удостоверение, назначенное системой, для приложения Azure Spring Cloud](./spring-cloud-howto-enable-system-assigned-managed-identity.md)
* [Дополнительные сведения об управляемых удостоверениях для ресурсов Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Настройка клиентских приложений для получения доступа к Службе приложений](../app-service/configure-authentication-provider-aad.md#configure-client-apps-to-access-your-app-service)

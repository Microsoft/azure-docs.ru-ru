---
title: Настройка проверки подлинности
titleSuffix: Azure Machine Learning
description: Узнайте, как настроить проверку подлинности для различных ресурсов и рабочих процессов в Машинном обучении Azure.
services: machine-learning
author: cjgronlund
ms.author: cgronlun
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 11/05/2020
ms.topic: conceptual
ms.custom: how-to, has-adal-ref, devx-track-js, devx-track-azurecli, contperf-fy21q2
ms.openlocfilehash: 30e4fede72df8eaf922745e7781c9e0d11f7ddb4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102210824"
---
# <a name="set-up-authentication-for-azure-machine-learning-resources-and-workflows"></a>Настройка проверки подлинности для ресурсов и рабочих процессов Машинного обучения Azure


Узнайте, как настроить проверку подлинности для Машинное обучение Azure рабочей области. Для большинства задач Проверка подлинности в Машинное обучение Azure рабочей области основана на __Azure Active Directory__ (Azure AD). Как правило, существует три рабочих процесса проверки подлинности, которые можно использовать при подключении к рабочей области.

* __Интерактивный__: ваша учетная запись используется в Azure Active Directory для непосредственной проверки подлинности или для получения токена, используемого для проверки подлинности. Интерактивная проверка подлинности используется во время _экспериментов и итеративной разработки_. Интерактивная проверка подлинности позволяет управлять доступом к ресурсам (например, веб-службе) отдельно для каждого пользователя.

* __Субъект-служба__. вы создаете учетную запись субъекта службы в Azure Active Directory и используете ее для проверки подлинности или получения маркера. Субъект-служба используется, когда требуется _автоматизированный процесс для проверки подлинности_ в службе без вмешательства пользователя. Например, сценарий непрерывной интеграции и развертывания, который проводит обучение и тестирование модели при каждом изменении кода обучения.

* __Управляемое удостоверение__. при использовании пакета SDK машинное обучение Azure _на виртуальной машине Azure_ можно использовать управляемое удостоверение для Azure. Этот рабочий процесс позволяет виртуальной машине подключаться к рабочей области с помощью управляемого удостоверения без сохранения учетных данных в коде Python или при запросе проверки подлинности пользователя. Машинное обучение Azureные кластеры можно настроить для использования управляемого удостоверения для доступа к рабочей области во время _обучения моделей_.

> [!IMPORTANT]
> Независимо от используемого рабочего процесса проверки подлинности, для определения уровня доступа (авторизации), разрешенного для ресурсов, используется управление доступом на основе ролей Azure (Azure RBAC). Например, администратор или процесс автоматизации может иметь доступ к созданию вычислительного экземпляра, но не использовать его, пока анализу данных может использовать его, но не сможет удалить или создать его. Дополнительные сведения см. [в статье Управление доступом к рабочей области машинное обучение Azure](how-to-assign-roles.md).

## <a name="prerequisites"></a>Предварительные требования

* Создайте [рабочую область Машинного обучения Azure](how-to-manage-workspace.md).
* [Настройте среду разработки](how-to-configure-environment.md) , чтобы установить пакет sdk для машинное обучение Azure, или используйте [машинное обучение Azure вычислительный экземпляр](concept-azure-machine-learning-architecture.md#compute-instance) с уже установленным пакетом SDK.

## <a name="azure-active-directory"></a>Azure Active Directory

Все рабочие процессы проверки подлинности для рабочей области зависят от Azure Active Directory. Если вы хотите, чтобы пользователи могли проходить проверку подлинности с помощью отдельных учетных записей, они должны иметь учетные записи в Azure AD. Если вы хотите использовать субъекты-службы, они должны существовать в Azure AD. Управляемые удостоверения также являются функцией Azure AD. 

Дополнительные сведения об Azure AD см. в статье [что такое Azure Active Directory проверка подлинности](..//active-directory/authentication/overview-authentication.md).

Создав учетные записи Azure AD, ознакомьтесь со статьей [Управление доступом к машинное обучение Azure рабочей области](how-to-assign-roles.md) , чтобы получить сведения о предоставлении им доступа к рабочей области и другим операциям в машинное обучение Azure.

## <a name="configure-a-service-principal"></a>Настройка субъекта-службы

Чтобы использовать субъект-службу (SP), необходимо сначала создать пакет обновления и предоставить ему доступ к рабочей области. Как упоминалось ранее, управление доступом на основе ролей в Azure (Azure RBAC) используется для управления доступом, поэтому необходимо также решить, какой доступ следует предоставить пакету обновления.

> [!IMPORTANT]
> При использовании субъекта-службы предоставьте ему __минимальный доступ, необходимый для задачи__ , для которой он используется. Например, владелец субъекта-службы или доступ к участнику не могут быть предоставлены, если все они используются для чтения маркера доступа для развертывания в Интернете.
>
> Причина предоставления минимального доступа заключается в том, что субъект-служба использует пароль для проверки подлинности, а пароль может храниться как часть скрипта автоматизации. При утечке пароля наличие минимального доступа, необходимого для выполнения конкретных задач, позволяет избежать вредоносного использования пакета обновления.

Самый простой способ создать пакет обновления и предоставить доступ к рабочей области — с помощью [Azure CLI](/cli/azure/install-azure-cli). Чтобы создать субъект-службу и предоставить ему доступ к рабочей области, выполните следующие действия.

> [!NOTE]
> Для выполнения всех этих действий необходимо быть администратором подписки.

1. Пройдите аутентификацию в подписке Azure.

    ```azurecli-interactive
    az login
    ```

    Если CLI сможет запустить браузер по умолчанию, он откроет в браузере страницу входа. Или откройте браузер и выполните инструкции из командной строки. В инструкции входит переход к [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и ввод кода авторизации.

    Если у вас несколько подписок Azure, можно использовать `az account set -s <subscription name or ID>` команду, чтобы задать подписку. См. дополнительные сведения в статье [Use multiple Azure subscriptions](/cli/azure/manage-azure-subscriptions-azure-cli) (Использование нескольких подписок Azure).

    Другие методы аутентификации см. в статье [Вход с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

1. Установите расширение Машинное обучение Azure:

    ```azurecli-interactive
    az extension add -n azure-cli-ml
    ```

1. Создайте субъект-службу. В следующем примере создается SP с именем **ML-auth** :

    ```azurecli-interactive
    az ad sp create-for-rbac --sdk-auth --name ml-auth
    ```

    Результатом будет файл JSON, который будет выглядеть примерно так: Запишите значения в полях `clientId`, `clientSecret` и `tenantId`, поскольку они понадобятся для выполнения других действий в этой статье.

    ```json
    {
        "clientId": "your-client-id",
        "clientSecret": "your-client-secret",
        "subscriptionId": "your-sub-id",
        "tenantId": "your-tenant-id",
        "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
        "resourceManagerEndpointUrl": "https://management.azure.com",
        "activeDirectoryGraphResourceId": "https://graph.windows.net",
        "sqlManagementEndpointUrl": "https://management.core.windows.net:5555",
        "galleryEndpointUrl": "https://gallery.azure.com/",
        "managementEndpointUrl": "https://management.core.windows.net"
    }
    ```

1. Получите сведения для субъекта-службы, используя `clientId` значение, возвращенное на предыдущем шаге:

    ```azurecli-interactive
    az ad sp show --id your-client-id
    ```

    Следующий JSON является упрощенным примером выходных данных команды. Запишите значение в поле `objectId`, поскольку оно понадобится на следующем шаге.

    ```json
    {
        "accountEnabled": "True",
        "addIns": [],
        "appDisplayName": "ml-auth",
        ...
        ...
        ...
        "objectId": "your-sp-object-id",
        "objectType": "ServicePrincipal"
    }
    ```

1. Разрешить пакету обновления доступ к рабочей области Машинное обучение Azure. Вам потребуется имя рабочей области и имя группы ресурсов для параметров `-w` и `-g` соответственно. Для параметра `--user` используйте значение `objectId` из предыдущего шага. `--role`Параметр позволяет задать роль доступа для субъекта-службы. В следующем примере для роли **владельца** назначается SP. 

    > [!IMPORTANT]
    > Доступ владельца позволяет субъекту-службе выполнять практически любую операцию в рабочей области. Он используется в этом документе, чтобы продемонстрировать, как предоставить доступ. в рабочей среде Корпорация Майкрософт рекомендует предоставить субъекту-службе минимальный доступ, необходимый для выполнения роли. Сведения о создании пользовательской роли с доступом, необходимым для вашего сценария, см. в разделе [Управление доступом к рабочей области машинное обучение Azure](how-to-assign-roles.md).

    ```azurecli-interactive
    az ml workspace share -w your-workspace-name -g your-resource-group-name --user your-sp-object-id --role owner
    ```

    Этот вызов не создает никаких выходных данных при успешном выполнении.

## <a name="configure-a-managed-identity"></a>Настройка управляемого удостоверения

> [!IMPORTANT]
> Управляемое удостоверение поддерживается только при использовании пакета SDK для Машинное обучение Azure на виртуальной машине Azure или в кластере Машинное обучение Azure COMPUTE. Использование управляемого удостоверения с кластером вычислений в настоящее время находится на этапе предварительной версии.

### <a name="managed-identity-with-a-vm"></a>Управляемое удостоверение с виртуальной машиной

1. Включите [управляемое системой удостоверение для ресурсов Azure на виртуальной машине](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

1. В [портал Azure](https://portal.azure.com)выберите рабочую область, а затем выберите __Управление доступом (IAM)__, __добавить назначение ролей__ и выберите __Виртуальная машина__ из раскрывающегося списка __назначить доступ__ . Наконец, выберите удостоверение виртуальной машины.

1. Выберите роль для назначения этому удостоверению. Например, участник или пользовательская роль. Дополнительные сведения см. в разделе [Управление доступом к ресурсам](how-to-assign-roles.md).

### <a name="managed-identity-with-compute-cluster"></a>Управляемое удостоверение с кластером вычислений

Дополнительные сведения см. в разделе [Настройка управляемого удостоверения для кластера вычислений](how-to-create-attach-compute-cluster.md#managed-identity).

<a id="interactive-authentication"></a>

## <a name="use-interactive-authentication"></a>Использовать интерактивную проверку подлинности

> [!IMPORTANT]
> При интерактивной проверке подлинности используется браузер и требуются файлы cookie (включая сторонние файлы cookie). Если вы отключили файлы cookie, может появиться сообщение об ошибке "не удалось войти". Эта ошибка также может возникать, если включена [многофакторная идентификация Azure AD](../active-directory/authentication/concept-mfa-howitworks.md).

В большинстве примеров в документации и примерах используется интерактивная проверка подлинности. Например, при использовании пакета SDK есть два вызова функций, которые автоматически запрашивают поток проверки подлинности на основе пользовательского интерфейса:

* Вызов функции `from_config()` приведет к выдаче запроса.

    ```python
    from azureml.core import Workspace
    ws = Workspace.from_config()
    ```

    Функция `from_config()` выполняет поиск файла JSON, который содержит сведения о подключении к рабочей области.

* При использовании `Workspace` конструктора для предоставления сведений о подписке, группе ресурсов и рабочей области будет также предложено выполнить интерактивную проверку подлинности.

    ```python
    ws = Workspace(subscription_id="your-sub-id",
                  resource_group="your-resource-group-id",
                  workspace_name="your-workspace-name"
                  )
    ```

> [!TIP]
> При наличии доступа к нескольким клиентам может потребоваться импортировать класс и явно определить целевой клиент. При вызове конструктора для `InteractiveLoginAuthentication` также будет предложено выполнить вход, аналогичный приведенным выше вызовам.
>
> ```python
> from azureml.core.authentication import InteractiveLoginAuthentication
> interactive_auth = InteractiveLoginAuthentication(tenant_id="your-tenant-id")
> ```

При использовании Azure CLI `az login` команда используется для проверки подлинности сеанса CLI. Дополнительные сведения см. в статье [Приступая к работе с Azure CLI](/cli/azure/get-started-with-azure-cli).

> [!TIP]
> Если вы используете пакет SDK из среды, в которой ранее выполнялась проверка подлинности в интерактивном режиме с помощью Azure CLI, можно использовать `AzureCliAuthentication` класс для проверки подлинности в рабочей области с помощью учетных данных, кэшированных CLI:
>
> ```python
> from azureml.core.authentication import AzureCliAuthentication
> cli_auth = AzureCliAuthentication()
> ws = Workspace(subscription_id="your-sub-id",
>                resource_group="your-resource-group-id",
>                workspace_name="your-workspace-name",
>                auth=cli_auth
>                )
> ```

<a id="service-principal-authentication"></a>

## <a name="use-service-principal-authentication"></a>Использование аутентификации субъекта-службы

Чтобы выполнить аутентификацию в рабочей области из пакета SDK с помощью субъекта-службы, используйте `ServicePrincipalAuthentication` конструктор класса. Используйте значения, полученные при создании поставщика услуг в качестве параметров. Параметр `tenant_id` сопоставляется с `tenantId` выше, `service_principal_id` сопоставляется с `clientId`, а `service_principal_password` сопоставляется с `clientSecret`.

```python
from azureml.core.authentication import ServicePrincipalAuthentication

sp = ServicePrincipalAuthentication(tenant_id="your-tenant-id", # tenantID
                                    service_principal_id="your-client-id", # clientId
                                    service_principal_password="your-client-secret") # clientSecret
```

Переменная `sp` теперь содержит объект проверки подлинности, который вы используете непосредственно в пакете SDK. Как правило, идентификаторы и секреты, используемые выше, рекомендуется хранить в переменных среды, как показано в следующем коде. Хранение в переменных среды предотвращает случайную проверку данных в репозитории GitHub.

```python
import os

sp = ServicePrincipalAuthentication(tenant_id=os.environ['AML_TENANT_ID'],
                                    service_principal_id=os.environ['AML_PRINCIPAL_ID'],
                                    service_principal_password=os.environ['AML_PRINCIPAL_PASS'])
```

Для автоматизированных рабочих процессов, которые выполняются в Python и используют пакет SDK, этот объект в большинстве случаев можно использовать как есть для проверки подлинности. Следующий код выполняет проверку подлинности в рабочей области с помощью созданного объекта auth.

```python
from azureml.core import Workspace

ws = Workspace.get(name="ml-example",
                   auth=sp,
                   subscription_id="your-sub-id")
ws.get_details()
```

### <a name="use-a-service-principal-from-the-azure-cli"></a>Использование субъекта-службы из Azure CLI

Для Azure CLIных команд можно использовать субъект-службу. Дополнительные сведения см. [в разделе Вход с использованием субъекта-службы](/cli/azure/create-an-azure-service-principal-azure-cli#sign-in-using-a-service-principal).

### <a name="use-a-service-principal-with-the-rest-api-preview"></a>Использование субъекта-службы с REST API (Предварительная версия)

Субъект-служба также может использоваться для проверки подлинности Машинное обучение Azure [REST API](/rest/api/azureml/) (Предварительная версия). Используется [поток предоставления учетных данных клиента](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) Azure Active Directory, который позволяет выполнять вызовы из одной службы к другой для проверки подлинности в автоматизированных рабочих процессах без графического пользовательского интерфейса. Примеры реализуются с помощью библиотеки [ADAL](../active-directory/azuread-dev/active-directory-authentication-libraries.md) как в Python, так и в Node.js, но также можно использовать любую библиотеку с открытым исходным кодом, поддерживающую OpenID Connect 1.0.

> [!NOTE]
> MSAL.js — это более новая библиотека, чем ADAL, однако она не позволяет выполнять проверку подлинности между службами с помощью учетных данных клиента, поскольку, в первую очередь, это клиентская библиотека, предназначенная для интерактивной проверки подлинности или на базе пользовательского интерфейса, привязанной к конкретному пользователю. Мы рекомендуем использовать ADAL, как показано ниже, чтобы создать автоматические рабочие процессы с REST API.

#### <a name="nodejs"></a>Node.js

Чтобы создать маркер проверки подлинности с помощью Node.js, выполните указанные ниже действия. В среде выполните `npm install adal-node`. Затем используйте `tenantId`, `clientId` и `clientSecret` из субъекта-службы, созданного в приведенных выше шагах, в качестве значений для соответствующих переменных в следующем сценарии.

```javascript
const adal = require('adal-node').AuthenticationContext;

const authorityHostUrl = 'https://login.microsoftonline.com/';
const tenantId = 'your-tenant-id';
const authorityUrl = authorityHostUrl + tenantId;
const clientId = 'your-client-id';
const clientSecret = 'your-client-secret';
const resource = 'https://management.azure.com/';

const context = new adal(authorityUrl);

context.acquireTokenWithClientCredentials(
  resource,
  clientId,
  clientSecret,
  (err, tokenResponse) => {
    if (err) {
      console.log(`Token generation failed due to ${err}`);
    } else {
      console.dir(tokenResponse, { depth: null, colors: true });
    }
  }
);
```

Переменная `tokenResponse` — это объект, включающий маркер и связанные метаданные, такие как время истечения срока действия. Маркеры действительны в течение 1 часа. Обновить их можно путем повторного выполнения того же вызова для получения нового маркера. В следующем фрагменте кода приведен пример ответа.

```javascript
{
    tokenType: 'Bearer',
    expiresIn: 3599,
    expiresOn: 2019-12-17T19:15:56.326Z,
    resource: 'https://management.azure.com/',
    accessToken: "random-oauth-token",
    isMRRT: true,
    _clientId: 'your-client-id',
    _authority: 'https://login.microsoftonline.com/your-tenant-id'
}
```

Используйте свойство `accessToken` для получения маркера проверки подлинности. Примеры использования маркера для выполнения вызовов API см. в [документации по REST API](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API).

#### <a name="python"></a>Python

Чтобы создать маркер проверки подлинности с помощью Python, выполните указанные ниже действия. В среде выполните `pip install adal`. Затем используйте `tenantId`, `clientId` и `clientSecret` из субъекта-службы, созданного в приведенных выше шагах, в качестве значений для соответствующих переменных в следующем сценарии.

```python
from adal import AuthenticationContext

client_id = "your-client-id"
client_secret = "your-client-secret"
resource_url = "https://login.microsoftonline.com"
tenant_id = "your-tenant-id"
authority = "{}/{}".format(resource_url, tenant_id)

auth_context = AuthenticationContext(authority)
token_response = auth_context.acquire_token_with_client_credentials("https://management.azure.com/", client_id, client_secret)
print(token_response)
```

Переменная `token_response` — это словарь, включающий маркер и связанные метаданные, такие как время истечения срока действия. Маркеры действительны в течение 1 часа. Обновить их можно путем повторного выполнения того же вызова для получения нового маркера. В следующем фрагменте кода приведен пример ответа.

```python
{
    'tokenType': 'Bearer',
    'expiresIn': 3599,
    'expiresOn': '2019-12-17 19:47:15.150205',
    'resource': 'https://management.azure.com/',
    'accessToken': 'random-oauth-token',
    'isMRRT': True,
    '_clientId': 'your-client-id',
    '_authority': 'https://login.microsoftonline.com/your-tenant-id'
}
```

Чтобы получить маркер проверки подлинности, используйте `token_response["accessToken"]`. Примеры использования маркера для выполнения вызовов API см. в [документации по REST API](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API).

#### <a name="java"></a>Java

В Java извлеките токен носителя, используя стандартный вызов RESTFUL:

```java
String tenantId = "your-tenant-id";
String clientId = "your-client-id";
String clientSecret = "your-client-secret";
String resourceManagerUrl = "https://management.azure.com";

HttpRequest tokenAuthenticationRequest = tokenAuthenticationRequest(tenantId, clientId, clientSecret, resourceManagerUrl);

HttpClient client = HttpClient.newBuilder().build();
Gson gson = new Gson();
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
if (response.statusCode == 200)
{
     body = gson.fromJson(body, AuthenticationBody.class);

    // ... etc ... 
}
// ... etc ...

static HttpRequest tokenAuthenticationRequest(String tenantId, String clientId, String clientSecret, String resourceManagerUrl){
    String authUrl = String.format("https://login.microsoftonline.com/%s/oauth2/token", tenantId);
    String clientIdParam = String.format("client_id=%s", clientId);
    String resourceParam = String.format("resource=%s", resourceManagerUrl);
    String clientSecretParam = String.format("client_secret=%s", clientSecret);

    String bodyString = String.format("grant_type=client_credentials&%s&%s&%s", clientIdParam, resourceParam, clientSecretParam);

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(authUrl))
            .POST(HttpRequest.BodyPublishers.ofString(bodyString))
            .build();
    return request;
}

class AuthenticationBody {
    String access_token;
    String token_type;
    int expires_in;
    String scope;
    String refresh_token;
    String id_token;
    
    AuthenticationBody() {}
}
```

Приведенный выше код должен был бы выполнять обработку исключений и кодов состояния, отличных от `200 OK` , но показывает шаблон: 

- Используйте идентификатор клиента и секрет, чтобы проверить, что программа должна иметь доступ
- Используйте идентификатор клиента, чтобы указать, где `login.microsoftonline.com` следует искать
- Использование Azure Resource Manager в качестве источника маркера авторизации

## <a name="use-managed-identity-authentication"></a>Использовать аутентификацию управляемого удостоверения

Чтобы пройти проверку подлинности в рабочей области с виртуальной машины или кластера вычислений, для которого настроено управляемое удостоверение, используйте `MsiAuthentication` класс. В следующем примере показано, как использовать этот класс для проверки подлинности в рабочей области.

```python
from azureml.core.authentication import MsiAuthentication

msi_auth = MsiAuthentication()

ws = Workspace(subscription_id="your-sub-id",
                resource_group="your-resource-group-id",
                workspace_name="your-workspace-name",
                auth=msi_auth
                )
```

## <a name="next-steps"></a>Дальнейшие действия

* [Использование секретов в обучении](how-to-use-secrets-in-runs.md).
* [Настройка проверки подлинности для моделей, развернутых в виде веб-службы](how-to-authenticate-web-service.md).
* [Использование модели Машинного обучения Azure, развернутой в качестве веб-службы](how-to-consume-web-service.md).

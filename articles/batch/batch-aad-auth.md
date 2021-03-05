---
title: Проверка подлинности пакетной службы Azure с помощью Azure Active Directory
description: Пакетная служба поддерживает Azure AD для аутентификации из пакетной службы. Узнайте, как пройти проверку подлинности одним из двух способов.
ms.topic: how-to
ms.date: 10/20/2020
ms.custom: has-adal-ref
ms.openlocfilehash: 2ceefa538c44208750da8986fcf3d161f0c0865f
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102180026"
---
# <a name="authenticate-batch-service-solutions-with-active-directory"></a>Аутентификация решений пакетной службы с помощью Active Directory

Пакетная служба Azure поддерживает проверку подлинности [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD). Azure AD — это облачный каталог и служба управления удостоверениями, основанная на нескольких клиентах Майкрософт. Инфраструктура Azure использует Azure AD для аутентификации клиентов, администраторов служб и пользователей организации.

Процесс аутентификации Azure AD в пакетной службе Azure можно выполнить двумя способами:

- Аутентификация пользователя, взаимодействующего с приложением, с использованием **встроенной аутентификации**. Приложение, использующее встроенную аутентификацию, собирает учетные данные пользователя и использует их для аутентификации доступа к ресурсам пакетной службы.
- Аутентификация автоматического приложения с использованием **субъекта-службы**. Субъект-служба определяет политику и разрешения приложения, чтобы представить приложение при получении доступа к ресурсам в среде выполнения.

Дополнительные сведения о службе Azure AD см. в [документации по Azure Active Directory](../active-directory/index.yml).

## <a name="endpoints-for-authentication"></a>Конечные точки аутентификации

Чтобы выполнить аутентификацию приложений пакетной службы с помощью Azure AD, необходимо включить некоторые известные конечные точки в код.

### <a name="azure-ad-endpoint"></a>Конечная точка Azure AD

Базовая конечная точка авторизации Azure AD:

`https://login.microsoftonline.com/`

Чтобы выполнить аутентификацию с помощью Azure AD, вам потребуется эта конечная точка и идентификатор клиента (идентификатором каталога). Идентификатор клиента определяет клиент Azure AD для аутентификации. Чтобы получить идентификатор клиента, выполните указания, описанные в разделе [Получение идентификатора клиента для Azure Active Directory](#get-the-tenant-id-for-your-active-directory):

`https://login.microsoftonline.com/<tenant-id>`

> [!NOTE]
> В процессе аутентификации с использованием субъекта-службы требуется конечная точка определенного клиента.
>
> При аутентификации с помощью встроенной аутентификации эта конечная точка является не обязательной. Тем не менее мы советуем использовать ее. Но вы также можете использовать общую конечную точку Azure AD, которая предоставляет универсальный интерфейс для сбора общих учетных данных в случае отсутствия конкретного клиента. Общая конечная точка выглядит так: `https://login.microsoftonline.com/common`.

Дополнительные сведения о конечных точках Azure AD см. в статье [Проверка подлинности и авторизация](../active-directory/develop/authentication-vs-authorization.md).

### <a name="batch-resource-endpoint"></a>Конечная точка ресурса пакетной службы

**Конечная точка ресурса пакетной службы Azure** позволяет получить токен аутентификации запросов для пакетной службы:

`https://batch.core.windows.net/`

## <a name="register-your-application-with-a-tenant"></a>Регистрация приложения в клиенте

При использовании Azure AD для аутентификации в первую очередь необходимо зарегистрировать приложение в клиенте Azure AD. Зарегистрировав приложение, вы сможете вызвать [библиотеку аутентификации Azure Active Directory ](../active-directory/azuread-dev/active-directory-authentication-libraries.md) (ADAL) из кода. Эта библиотека предоставляет API для аутентификации с помощью Azure AD в приложении. Регистрация приложения необходима, если вы планируете использовать встроенную аутентификацию или аутентификацию с использованием субъекта-службы.

При регистрации приложения вы отправляете сведения о приложении в Azure AD. После этого служба Azure AD предоставит идентификатор приложения (*идентификатор клиента*), позволяющий связать с ней приложение во время выполнения. Дополнительные сведения об идентификаторе приложения см. в статье [Объекты приложения и субъекта-службы в Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

Чтобы зарегистрировать приложение пакетной службы, выполните действия, описанные в разделе " **Регистрация приложения** " статьи [Краткое руководство по регистрации приложения на платформе Microsoft Identity](../active-directory/develop/quickstart-register-app.md). При регистрации приложения как собственного вы можете указать любой допустимый универсальный код ресурса (URI) в качестве **URI перенаправления**. Реальную конечную точку указывать необязательно.

После регистрации приложения отобразится его идентификатор:

![Снимок экрана идентификатора приложения, показанного в портал Azure.](./media/batch-aad-auth/app-registration-data-plane.png)

## <a name="get-the-tenant-id-for-your-active-directory"></a>Получение идентификатора клиента для Active Directory

Идентификатор клиента определяет клиент Azure AD, который предоставляет службы аутентификации приложению. Чтобы получить идентификатор клиента, сделайте следующее:

1. На портале Azure выберите Active Directory.
1. Выберите **Свойства**.
1. Скопируйте значение GUID, указанное для параметра **Идентификатор каталога**. Это значение также называется идентификатором клиента.

![Снимок экрана с ИДЕНТИФИКАТОРом каталога в портал Azure.](./media/batch-aad-auth/aad-directory-id.png)

## <a name="use-integrated-authentication"></a>Использование встроенной аутентификации

Чтобы выполнить аутентификацию с помощью встроенной аутентификации, необходимо предоставить приложению разрешения на подключение к API пакетной службы. Это позволит ему выполнить аутентификацию вызовов к API пакетной службы с помощью Azure AD.

Чтобы предоставить приложению доступ к пакетной службе, после регистрации приложения выполните следующие действия на портале Azure:

1. В портал Azure выберите **все службы**, а затем щелкните **Регистрация приложений**.
1. Найдите имя приложения в списке зарегистрированных приложений.
1. Выберите приложение, а затем выберите **Разрешения API**.
1. В разделе **Разрешения API** щелкните **Добавить разрешение**.
1. В разделе **Выбор API** найдите API пакетной службы. Продолжайте поиск для каждой из этих строк, пока не найдете API:
    1. **Microsoft Azure Batch**
    1. **ddbf3205-c6bd-46ae-8127-60eb93363864** — это идентификатор API пакетной службы.
1. Найдя API пакетной службы, выберите его и нажмите кнопку **выбрать**.
1. В разделе **Выбор разрешений** установите флажок **Access Azure Batch Service** (Доступ к пакетной службе Azure) и выберите **Добавить разрешения**.

Теперь в окне **Разрешения API** указано, что приложение Azure AD имеет доступ и к Microsoft Graph, и к API пакетной службы. Разрешения Microsoft Graph предоставляются автоматически во время первой регистрации приложения в Azure AD.

## <a name="use-a-service-principal"></a>Использование субъекта-службы

Чтобы выполнить аутентификацию приложения, которое выполняется автоматически, необходимо использовать субъект-службу. После регистрации приложения выполните приведенные ниже действия на портале Azure, чтобы настроить субъект-службу.

1. Запросите секрет для приложения.
1. Назначьте приложению управление доступом на основе ролей Azure (Azure RBAC).

### <a name="request-a-secret-for-your-application"></a>Запрос секрета для приложения

При выполнении проверки подлинности с помощью субъекта-службы приложение отправляет в Azure AD идентификатор приложения и секрет. Вам необходимо создать и скопировать секретный ключ, чтобы использовать его из кода.

На портале Azure сделайте следующее:

1. В портал Azure выберите **все службы**. Выберите **Регистрация приложений**.
1. Выберите свое приложение из списка приложений.
1. Выберите приложение, а затем выберите **Сертификаты и секреты**. В разделе **Секреты клиента** выберите **Новый секрет клиента**.
1. Чтобы создать секрет, введите описание секрета. Затем выберите срок действия секрета в один год, два года или без истечения срока действия.
1. Выберите **Добавить**, чтобы создать и отобразить секрет. Скопируйте значение секрета в безопасное место, так как у вас больше не будет доступа к нему после закрытия страницы.

### <a name="assign-azure-rbac-to-your-application"></a>Назначение Azure RBAC приложению

Для проверки подлинности с помощью субъекта-службы необходимо назначить Azure RBAC приложению. Выполните следующие действия.

1. На портале Azure перейдите к учетной записи пакетной службы, используемой приложением.
1. В разделе **Параметры** учетной записи пакетной службы выберите **Управление доступом (IAM)** .
1. Выберите вкладку **Назначения ролей**.
1. Выберите **Добавить назначение ролей**.
1. В раскрывающемся списке **Роль** выберите роль приложения — *Участник* или *Читатель*. Дополнительные сведения об этих ролях см. в статье Начало [работы с управлением доступом на основе ролей Azure в портал Azure](../role-based-access-control/overview.md).
1. В поле **Выбрать** введите имя приложения. Выберите свое приложение в списке, а затем нажмите **Сохранить**.

Теперь приложение должно отображаться в параметрах управления доступом с назначенной ролью Azure.

![Назначение роли Azure для приложения](./media/batch-aad-auth/app-rbac-role.png)

### <a name="assign-a-custom-role"></a>Назначение настраиваемой роли

Настраиваемая роль предоставляет детализированное разрешение пользователю на отправку заданий, задач и т. д. Это дает возможность запретить пользователям выполнять операции, влияющие на стоимость, такие как создание пулов или изменение узлов.

Пользовательскую роль можно использовать для предоставления разрешений пользователю Azure AD, группе или субъекту-службе для следующих операций RBAC Azure.

- Microsoft.Batch/batchAccounts/pools/write
- Microsoft.Batch/batchAccounts/pools/delete
- Microsoft.Batch/batchAccounts/pools/read
- Microsoft.Batch/batchAccounts/jobSchedules/write
- Microsoft.Batch/batchAccounts/jobSchedules/delete
- Microsoft.Batch/batchAccounts/jobSchedules/read
- Microsoft.Batch/batchAccounts/jobs/write
- Microsoft.Batch/batchAccounts/jobs/delete
- Microsoft.Batch/batchAccounts/jobs/read
- Microsoft.Batch/batchAccounts/certificates/write
- Microsoft.Batch/batchAccounts/certificates/delete
- Microsoft.Batch/batchAccounts/certificates/read
- Microsoft.Batch/batchAccounts/read (для любой операции чтения)
- Microsoft.Batch/batchAccounts/listKeys/action (для любой операции)

Настраиваемые роли предназначены для пользователей, прошедших проверку подлинности в Azure AD, а не для учетных данных учетной записи пакетной службы (общий ключ). Обратите внимание, что учетные данные учетной записи пакетной службы предоставляют полный доступ к учетной записи пакетной службы. Также обратите внимание, что для заданий, использующих [автопул](nodes-and-pools.md#autopools) , требуются разрешения уровня пула.

> [!NOTE]
> Некоторые назначения ролей необходимо указать в поле действие, тогда как другие необходимо указать в поле Action. Дополнительные сведения см. в статье [операции с поставщиками ресурсов Azure](../role-based-access-control/resource-provider-operations.md#microsoftbatch).

Вот пример определения настраиваемой роли.

```json
{
 "properties":{
    "roleName":"Azure Batch Custom Job Submitter",
    "type":"CustomRole",
    "description":"Allows a user to submit jobs to Azure Batch but not manage pools",
    "assignableScopes":[
      "/subscriptions/88888888-8888-8888-8888-888888888888"
    ],
    "permissions":[
      {
        "actions":[
          "Microsoft.Batch/*/read",
          "Microsoft.Authorization/*/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*",
          "Microsoft.Insights/alertRules/*"
        ],
        "notActions":[

        ],
        "dataActions":[
          "Microsoft.Batch/batchAccounts/jobs/*",
          "Microsoft.Batch/batchAccounts/jobSchedules/*"
        ],
        "notDataActions":[

        ]
      }
    ]
  }
}
```

Дополнительные сведения о создании настраиваемой роли см. в статье [пользовательские роли Azure](../role-based-access-control/custom-roles.md).

### <a name="get-the-tenant-id-for-your-azure-active-directory"></a>Получение идентификатора клиента для Azure Active Directory

Идентификатор клиента определяет клиент Azure AD, который предоставляет службы аутентификации приложению. Чтобы получить идентификатор клиента, сделайте следующее:

1. На портале Azure выберите Active Directory.
1. Выберите **Свойства**.
1. Скопируйте значение GUID, указанное для параметра **Идентификатор каталога**. Это значение также называется идентификатором клиента.

![Копирование идентификатора каталога](./media/batch-aad-auth/aad-directory-id.png)

## <a name="code-examples"></a>Примеры кода

В примерах кода в этом разделе показано, как выполнить аутентификацию в Azure AD, используя встроенную и субъект-службу. В большинстве примеров кода используется .NET, но общие концепции актуальны и для других языков.

> [!NOTE]
> Срок действия маркера проверки подлинности Azure AD истекает через час. При использовании долговременного объекта **BatchClient** мы советуем получать маркер из ADAL при каждом запросе, чтобы у вас всегда был допустимый маркер.
>
> Чтобы достичь этого в .NET, напишите метод, который получает маркер из Azure AD, и передайте этот метод в качестве делегата в объект **BatchTokenCredentials**. Чтобы гарантировать предоставление допустимого маркера, при каждом запросе к пакетной службе вызывается метод делегата. По умолчанию ADAL кэширует маркеры, поэтому новый маркер извлекается из Azure AD только при необходимости. Дополнительные сведения об маркерах в Azure AD см. в разделе [токены безопасности](../active-directory/develop/security-tokens.md).

### <a name="code-example-using-azure-ad-integrated-authentication-with-batch-net"></a>Пример кода: Использование встроенной аутентификации Azure AD с библиотекой .NET для пакетной службы

Чтобы выполнить аутентификацию с использованием встроенной аутентификации из библиотеки .NET для пакетной службы, укажите ссылку на пакет [.NET для пакетной службы Azure](https://www.nuget.org/packages/Microsoft.Azure.Batch/) и пакет [библиотеки ADAL](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).

Добавьте в код следующие инструкции `using`:

```csharp
using Microsoft.Azure.Batch;
using Microsoft.Azure.Batch.Auth;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
```

Укажите ссылку на конечную точку Azure AD в коде, в том числе идентификатор клиента. Чтобы получить идентификатор клиента, выполните указания, описанные в разделе [Получение идентификатора клиента для Azure Active Directory](#get-the-tenant-id-for-your-active-directory):

```csharp
private const string AuthorityUri = "https://login.microsoftonline.com/<tenant-id>";
```

Укажите ссылку на конечную точку ресурса пакетной службы:

```csharp
private const string BatchResourceUri = "https://batch.core.windows.net/";
```

Укажите ссылку на учетную запись пакетной службы:

```csharp
private const string BatchAccountUrl = "https://myaccount.mylocation.batch.azure.com";
```

Укажите идентификатор приложения (идентификатор клиента) для вашего приложения. Идентификатор приложения становится доступен после регистрации приложения на портале Azure:

```csharp
private const string ClientId = "<application-id>";
```

Скопируйте также указанный URI перенаправления, если приложение зарегистрировано в качестве собственного приложения. URI перенаправления, указанный в коде, должен соответствовать URI перенаправления, который был указан при регистрации приложения:

```csharp
private const string RedirectUri = "http://mybatchdatasample";
```

Напишите метод обратного вызова, чтобы получить маркер проверки подлинности из Azure AD. Приведенный здесь метод обратного вызова **GetAuthenticationTokenAsync** вызывает библиотеку ADAL для аутентификации пользователя, взаимодействующего с приложением. Предоставленный ADAL метод **AcquireTokenAsync** запрашивает учетные данные пользователя. Как только пользователь введет их, приложение продолжит работу (если его учетные данные не кэшированы).

```csharp
public static async Task<string> GetAuthenticationTokenAsync()
{
    var authContext = new AuthenticationContext(AuthorityUri);

    // Acquire the authentication token from Azure AD.
    var authResult = await authContext.AcquireTokenAsync(BatchResourceUri,
                                                        ClientId,
                                                        new Uri(RedirectUri),
                                                        new PlatformParameters(PromptBehavior.Auto));

    return authResult.AccessToken;
}
```

Создайте объект **BatchTokenCredentials**, который принимает делегат в качестве параметра. Используйте эти учетные данные, чтобы открыть объект **BatchClient**. Вы можете использовать объект **BatchClient** для последующих операций в пакетной службе.

```csharp
public static void PerformBatchOperations()
{
    Func<Task<string>> tokenProvider = () => GetAuthenticationTokenAsync();

    using (var client = BatchClient.Open(new BatchTokenCredentials(BatchAccountUrl, tokenProvider)))
    {
        client.JobOperations.ListJobs();
    }
}
```

### <a name="code-example-using-an-azure-ad-service-principal-with-batch-net"></a>Пример кода: Использование субъекта-службы Azure AD с библиотекой .NET для пакетной службы

Чтобы выполнить аутентификацию с использованием субъекта-службы из библиотеки .NET для пакетной службы, укажите ссылки на пакет [.NET для пакетной службы Azure](https://www.nuget.org/packages/Azure.Batch/) и пакет [библиотеки ADAL](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).

Добавьте в код следующие инструкции `using`:

```csharp
using Microsoft.Azure.Batch;
using Microsoft.Azure.Batch.Auth;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
```

Укажите ссылку на конечную точку Azure AD в коде, в том числе идентификатор клиента. При использовании субъекта-службы необходимо указать конечную точку определенного клиента. Чтобы получить идентификатор клиента, выполните указания, описанные в разделе [Получение идентификатора клиента для Azure Active Directory](#get-the-tenant-id-for-your-active-directory):

```csharp
private const string AuthorityUri = "https://login.microsoftonline.com/<tenant-id>";
```

Укажите ссылку на конечную точку ресурса пакетной службы:

```csharp
private const string BatchResourceUri = "https://batch.core.windows.net/";
```

Укажите ссылку на учетную запись пакетной службы:

```csharp
private const string BatchAccountUrl = "https://myaccount.mylocation.batch.azure.com";
```

Укажите идентификатор приложения (идентификатор клиента) для вашего приложения. Идентификатор приложения становится доступен после регистрации приложения на портале Azure:

```csharp
private const string ClientId = "<application-id>";
```

Укажите секретный ключ, скопированный на портале Azure:

```csharp
private const string ClientKey = "<secret-key>";
```

Напишите метод обратного вызова, чтобы получить маркер проверки подлинности из Azure AD. Приведенный здесь метод обратного вызова **GetAuthenticationTokenAsync** вызывает библиотеку ADAL для автоматической аутентификации:

```csharp
public static async Task<string> GetAuthenticationTokenAsync()
{
    AuthenticationContext authContext = new AuthenticationContext(AuthorityUri);
    AuthenticationResult authResult = await authContext.AcquireTokenAsync(BatchResourceUri, new ClientCredential(ClientId, ClientKey));

    return authResult.AccessToken;
}
```

Создайте объект **BatchTokenCredentials**, который принимает делегат в качестве параметра. Используйте эти учетные данные, чтобы открыть объект **BatchClient**. Применяйте этот объект **BatchClient** для всех последующих операций в пакетной службе.

```csharp
public static void PerformBatchOperations()
{
    Func<Task<string>> tokenProvider = () => GetAuthenticationTokenAsync();

    using (var client = BatchClient.Open(new BatchTokenCredentials(BatchAccountUrl, tokenProvider)))
    {
        client.JobOperations.ListJobs();
    }
}
```

### <a name="code-example-using-an-azure-ad-service-principal-with-batch-python"></a>Пример кода: Использование субъекта-службы Azure AD с библиотекой Python для пакетной службы

Чтобы выполнить аутентификацию с использованием субъекта-службы из библиотеки Python для пакетной службы, установите модули [azure-batch](https://pypi.org/project/azure-batch/) и [azure-common](https://pypi.org/project/azure-common/) и укажите ссылки на них.

```python
from azure.batch import BatchServiceClient
from azure.common.credentials import ServicePrincipalCredentials
```

Если вы используете субъект-службу, обязательно укажите идентификатор клиента. Чтобы получить идентификатор клиента, выполните указания, описанные в разделе [Получение идентификатора клиента для Azure Active Directory](#get-the-tenant-id-for-your-active-directory):

```python
TENANT_ID = "<tenant-id>"
```

Укажите ссылку на конечную точку ресурса пакетной службы:

```python
RESOURCE = "https://batch.core.windows.net/"
```

Укажите ссылку на учетную запись пакетной службы:

```python
BATCH_ACCOUNT_URL = "https://myaccount.mylocation.batch.azure.com"
```

Укажите идентификатор приложения (идентификатор клиента) для вашего приложения. Идентификатор приложения становится доступен после регистрации приложения на портале Azure:

```python
CLIENT_ID = "<application-id>"
```

Укажите секретный ключ, скопированный на портале Azure:

```python
SECRET = "<secret-key>"
```

Создайте объект **ServicePrincipalCredentials**.

```python
credentials = ServicePrincipalCredentials(
    client_id=CLIENT_ID,
    secret=SECRET,
    tenant=TENANT_ID,
    resource=RESOURCE
)
```

Используйте учетные данные субъекта-службы, чтобы открыть объект **BatchServiceClient**. Используйте этот объект **BatchClient** для всех последующих операций в пакетной службе.

```python
    batch_client = BatchServiceClient(
    credentials,
    batch_url=BATCH_ACCOUNT_URL
)
```

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [документацией по Azure Active Directory](../active-directory/index.yml). Подробные примеры, показывающие, как использовать ADAL, доступны в библиотеке [примеров кода Azure](https://azure.microsoft.com/resources/samples/?service=active-directory).
- Сведения об [объектах приложений и субъектов-служб в Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md) и [о том, как создать приложение Azure AD и субъект-службу, которые могут получать доступ к ресурсам](../active-directory/develop/howto-create-service-principal-portal.md).
- Узнайте [, как проверить подлинность решений по управлению пакетной службой с помощью Active Directory](batch-aad-auth-management.md).
- Пример Python для создания клиента пакетной службы, подлинность которого проверяется с помощью маркера Azure AD, см. в примере [Deploying Azure Batch Custom Image with a Python Script](https://github.com/azurebigcompute/recipes/blob/master/Azure%20Batch/CustomImages/CustomImagePython.md) (Развертывание пользовательского образа пакетной службы Azure с помощью скрипта Python).


---
title: Краткое руководство. Вызов Microsoft Graph из управляющей программы Python | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как для процесса Python получить маркер доступа и вызвать API, защищенный платформой удостоверений Майкрософт, с помощью собственного удостоверения приложения.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/22/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, devx-track-python, scenarios:getting-started, languages:Python
ms.openlocfilehash: 734fad7d3f4fb7a2a816d9ad10fb6b15e2faf9e2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99820415"
---
# <a name="quickstart-acquire-a-token-and-call-microsoft-graph-api-from-a-python-console-app-using-apps-identity"></a>Краткое руководство. Получение маркера безопасности и вызов API Microsoft Graph из консольного приложения Python с помощью удостоверения приложения

При работе с этим кратким руководством вы скачаете и выполните пример кода. Такой пример кода демонстрирует, как приложение Python может получить маркер доступа с помощью удостоверения приложения для вызова API Microsoft Graph и отобразить [список пользователей](/graph/api/user-list) в каталоге, а также как автоматическое задание или служба Windows могут выполняться с удостоверением приложения, а не пользователя. 

> [!div renderon="docs"]
> ![Схема работы приложения, создаваемого в этом кратком руководстве](media/quickstart-v2-python-daemon/python-console-daemon.svg)

## <a name="prerequisites"></a>Предварительные требования

Для запуска этого примера вам потребуются следующие компоненты:

- [Python 2.7+](https://www.python.org/downloads/release/python-2713) или [Python 3+](https://www.python.org/downloads/release/python-364/)
- [MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве

> [!div renderon="docs" class="sxs-lookup"]
>
> У вас есть два варианта запуска приложения, используемого в этом кратком руководстве: оперативно (вариант 1) и вручную (вариант 2).
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода
>
> 1. Откройте страницу <a href="https://portal.azure.com/?Microsoft_AAD_RegisteredApps=true#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/PythonDaemonQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям, чтобы быстро скачать и автоматически настроить новое приложение.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Вариант 2. Регистрация и настройка приложения и примера кода вручную

> [!div renderon="docs"]
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
> Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение вручную, сделайте следующее:
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите **имя** приложения, например `Daemon-console`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. Выберите **Зарегистрировать**.
> 1. В разделе **Управление** выберите **Сертификаты и секреты**.
> 1. В разделе **Секреты клиента** выберите **Новый секрет клиента**, введите имя, а затем нажмите кнопку **Добавить**. Сохраните значение секрета в надежном месте, так как это значение потребуется вам в дальнейшем.
> 1. В разделе **Управление** выберите **Разрешения API** > **Добавить разрешение**. Выберите **Microsoft Graph**.
> 1. Выберите **Разрешения приложений**.
> 1. В узле **Пользователь** выберите **User.Read.All**, а затем щелкните **Добавить разрешения**.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="download-and-configure-the-quickstart-app"></a>Скачивание и настройка приложения, используемого в этом кратком руководстве
>
> #### <a name="step-1-configure-your-application-in-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Для работы примера кода, приведенного в этом кратком руководстве, необходимо создать секрет клиента и добавить разрешение приложения API Graph **User.Read.All**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-netcore-daemon/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-python-project"></a>Шаг 2. Скачивание проекта Python

> [!div renderon="docs"]
> [Скачивание проекта управляющей программы Python](https://github.com/Azure-Samples/ms-identity-python-daemon/archive/master.zip)

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/ms-identity-python-daemon/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`


> [!div renderon="docs"]
> #### <a name="step-3-configure-the-python-project"></a>Шаг 3. Настройка проекта Python
>
> 1. Извлеките ZIP-файл в локальную папку, расположенную как можно ближе к корню диска (например, **C:\Azure-Samples**).
> 1. Перейдите к вложенной папке **1-Call-MsGraph-WithSecret**.
> 1. Измените файл **parameters.json**, заменив значения полей `authority`, `client_id` и `secret` следующим фрагментом кода:
>
>    ```json
>    "authority": "https://login.microsoftonline.com/Enter_the_Tenant_Id_Here",
>    "client_id": "Enter_the_Application_Id_Here",
>    "secret": "Enter_the_Client_Secret_Here"
>    ```
>    Где:
>    - `Enter_the_Application_Id_Here` — это **идентификатор приложения (клиента)** , которое вы зарегистрировали.
>    - `Enter_the_Tenant_Id_Here` — замените это значение на **идентификатор клиента** или **имя клиента** (например, contoso.microsoft.com).
>    - `Enter_the_Client_Secret_Here` — замените это значение на секрет клиента, созданный на шаге 1.
>
> > [!TIP]
> > Чтобы найти значения параметров **Идентификатор приложения (клиента)** и **Идентификатор каталога (клиента)** , на портале Azure перейдите на страницу приложения **Обзор**. Чтобы создать ключ, перейдите на страницу **Сертификаты и секреты**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-admin-consent"></a>Шаг 3. Согласие администратора

> [!div renderon="docs"]
> #### <a name="step-4-admin-consent"></a>Шаг 4. Согласие администратора

Если попытаться запустить приложение на этом этапе, вы получите ошибку *HTTP 403 — Forbidden* (запрещено): `Insufficient privileges to complete the operation`. Эта ошибка возникает, так как *разрешение только для приложения* предоставляется с согласия администратора. Следовательно, глобальный администратор каталога должен предоставить такое согласие приложению. Выберите один из следующих вариантов с учетом своей роли:

##### <a name="global-tenant-administrator"></a>Глобальный администратор клиента

> [!div renderon="docs"]
> Если вы являетесь глобальным администратором арендатора, перейдите на страницу **Разрешения API** в разделе **Регистрации приложений** на портале Azure и выберите **Предоставить согласие администратора для {имя арендатора}** (где {имя арендатора} — это имя вашего каталога).

> [!div renderon="portal" class="sxs-lookup"]
> Если вы являетесь глобальным администратором, перейдите на страницу **Разрешения API** и выберите **Предоставить согласия администратора для имя_арендаторр**.
> > [!div id="apipermissionspage"]
> > [Переход на страницу "Разрешения API"]()

##### <a name="standard-user"></a>Обычный пользователь

Если вы являетесь обычным пользователем арендатора, попросите глобального администратора предоставить согласие администратора для вашего приложения. Чтобы сделать это, предоставьте следующий URL-адрес администратору:

```url
https://login.microsoftonline.com/Enter_the_Tenant_Id_Here/adminconsent?client_id=Enter_the_Application_Id_Here
```

> [!div renderon="docs"]
>> Где:
>> * `Enter_the_Tenant_Id_Here` — замените это значение на **идентификатор клиента** или **имя клиента** (например, contoso.microsoft.com).
>> * `Enter_the_Application_Id_Here` — это **идентификатор приложения (клиента)** , которое вы зарегистрировали.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-4-run-the-application"></a>Шаг 4. Выполнение приложения

> [!div renderon="docs"]
> #### <a name="step-5-run-the-application"></a>Шаг 5. Выполнение приложения

Зависимости этого образца необходимо установить один раз.

```console
pip install -r requirements.txt
```

Затем запустите приложение с помощью командной строки или консоли:

```console
python confidential_client_secret_sample.py parameters.json
```

Вы должны увидеть на консоли фрагмент Json, представляющий список пользователей в вашем каталоге Azure AD.

> [!IMPORTANT]
> В этом кратком руководстве приложение использует секрет клиента для собственной идентификации в качестве конфиденциального клиента. Так как секрет клиента добавляется в качестве обычного текста в файлы проекта, из соображениям безопасности рекомендуется использовать сертификат вместо секрета клиента, прежде чем использовать приложение в качестве рабочего. Дополнительные сведения об использовании сертификата см. в [этих инструкциях](https://github.com/Azure-Samples/ms-identity-python-daemon/blob/master/2-Call-MsGraph-WithCertificate/README.md) в том же репозитории GitHub для этого примера, но во второй папке **2-Call-MsGraph-WithCertificate**.

## <a name="more-information"></a>Дополнительные сведения

### <a name="msal-python"></a>MSAL Python

[MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) — это библиотека, используемая для выполнения входа пользователей и запросов маркеров, которые нужны для доступа к API, защищенному платформой удостоверений Майкрософт. Как описано выше, в этом кратком руководстве маркеры запрашиваются с использованием собственного удостоверения приложения вместо делегированных разрешений. Поток проверки подлинности, используемый в данном случае, называется *[потоком учетных данных клиента OAuth](v2-oauth2-client-creds-grant-flow.md)* . Дополнительные сведения об использовании MSAL Python с приложениями управляющей программы см. в [этой статье](scenario-daemon-overview.md).

 Установите MSAL Python, выполнив следующую команду pip.

```powershell
pip install msal
```

### <a name="msal-initialization"></a>Инициализация MSAL

Добавив следующий код, вы можете добавить ссылку на MSAL.

```Python
import msal
```

Затем выполните инициализацию MSAL с помощью следующего кода.

```Python
app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential=config["secret"])
```

> | Где: |Описание |
> |---------|---------|
> | `config["secret"]` | Секрет клиента, созданный для приложения на портале Azure. |
> | `config["client_id"]` | **Идентификатор приложения (клиента)** , зарегистрированного на портале Azure. Это значение можно найти на странице приложения **Обзор** на портале Azure. |
> | `config["authority"]`    | Конечная точка STS для проверки подлинности пользователей. Обычно `https://login.microsoftonline.com/{tenant}` для общедоступного облака, где {tenant} — имя или идентификатор вашего клиента.|

Дополнительные сведения см. в [справочной документации по `ConfidentialClientApplication`](https://msal-python.readthedocs.io/en/latest/#confidentialclientapplication).

### <a name="requesting-tokens"></a>Запрос маркеров

Чтобы запросить маркер с помощью удостоверение приложения, используйте метод `AcquireTokenForClient`.

```Python
result = None
result = app.acquire_token_silent(config["scope"], account=None)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    result = app.acquire_token_for_client(scopes=config["scope"])
```

> |Где:| Описание |
> |---------|---------|
> | `config["scope"]` | Содержит запрошенные области. Для конфиденциальных клиентов следует использовать формат, аналогичный `{Application ID URI}/.default`, который указывает, что запрашиваемые области — это те, которые статически определены в объекте приложения, заданном на портале Azure (для Microsoft Graph `{Application ID URI}` указывает на `https://graph.microsoft.com`). Для пользовательских веб-API `{Application ID URI}` определяется в разделе **Предоставление API** в разделе **Регистрации приложений** на портале Azure.|

Дополнительные сведения см. в [справочной документации по `AcquireTokenForClient`](https://msal-python.readthedocs.io/en/latest/#msal.ConfidentialClientApplication.acquire_token_for_client).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управляющих программах см. на главной странице сценария.

> [!div class="nextstepaction"]
> [Создание управляющей программы, которая вызывает веб-API](scenario-daemon-overview.md)

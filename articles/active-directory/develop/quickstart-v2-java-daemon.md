---
title: Краткое руководство. Вызов Microsoft Graph из управляющей программы Java | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как для приложения Java получить маркер доступа и вызвать API, защищенный конечной точкой платформы удостоверений Майкрософт, с помощью собственного удостоверения приложения.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 01/22/2021
ms.author: nacanuma
ms.custom: aaddev, scenarios:getting-started, languages:Java, devx-track-java
ms.openlocfilehash: 196b80a704b8a270a4cbb7d3505d5f9be1e23479
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99820330"
---
# <a name="quickstart-acquire-a-token-and-call-microsoft-graph-api-from-a-java-console-app-using-apps-identity"></a>Краткое руководство. Получение маркера и вызов API Microsoft Graph из консольного приложения Java с помощью удостоверения приложения

При работе с этим кратким руководством вы скачаете и выполните пример кода, который демонстрирует получение маркера доступа с помощью удостоверения приложения из приложения Java для вызова API Microsoft Graph и отображение [списка пользователей](/graph/api/user-list) в каталоге. а также как автоматическое задание или служба Windows могут выполняться с удостоверением приложения, а не пользователя. 

> [!div renderon="docs"]
> ![Схема работы приложения, создаваемого в этом кратком руководстве](media/quickstart-v2-java-daemon/java-console-daemon.svg)

## <a name="prerequisites"></a>Предварительные требования

Для запуска этого примера вам потребуются следующие компоненты:

- [Пакет SDK для Java](https://openjdk.java.net/) версии 8 или более поздней.
- [Maven](https://maven.apache.org/)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве

> [!div renderon="docs" class="sxs-lookup"]
>
> У вас есть два варианта запуска приложения, используемого в этом кратком руководстве: оперативно (вариант 1) и вручную (вариант 2).
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода
>
> 1. Откройте страницу <a href="https://portal.azure.com/?Microsoft_AAD_RegisteredApps=true#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/JavaDaemonQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
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
> #### <a name="step-1-configure-the-application-in-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Для работы примера кода в этом кратком руководстве необходимо создать секрет клиента и добавить разрешение приложения API Graph **User.Read.All**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-netcore-daemon/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-java-project"></a>Шаг 2. Скачивание проекта Java

> [!div renderon="docs"]
> [Скачивание проекта управляющей программы Java](https://github.com/Azure-Samples/ms-identity-java-daemon/archive/master.zip)

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/ms-identity-java-daemon/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`


> [!div renderon="docs"]
> #### <a name="step-3-configure-the-java-project"></a>Шаг 3. Настройка проекта Java
>
> 1. Извлеките ZIP-файл в локальную папку, расположенную как можно ближе к корню диска (например, *C:\Azure-Samples*).
> 1. Перейдите к вложенной папке **msal-client-credential-secret**.
> 1. Откройте в редакторе файл *src\main\resources\application.properties* и замените значения полей `AUTHORITY`, `CLIENT_ID` и `SECRET` следующим фрагментом кода:
>
>    ```
>    AUTHORITY=https://login.microsoftonline.com/Enter_the_Tenant_Id_Here/
>    CLIENT_ID=Enter_the_Application_Id_Here
>    SECRET=Enter_the_Client_Secret_Here
>    ```
>    Где:
>    - `Enter_the_Application_Id_Here` — это **идентификатор приложения (клиента)** , которое вы зарегистрировали.
>    - `Enter_the_Tenant_Id_Here` — замените это значение **идентификатором арендатора** или **именем арендатора** (например, contoso.microsoft.com).
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

Если вы являетесь обычным пользователем клиента, попросите глобального администратора предоставить согласие администратора для вашего приложения. Чтобы сделать это, предоставьте следующий URL-адрес администратору:

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

Вы можете протестировать пример, запустив метод main из ClientCredentialGrant.java в любой интегрированной среде разработки.

Из оболочки или командной строки:

```
$ mvn clean compile assembly:single
```

Это действие создает файл msal-client-credential-secret-1.0.0.jar в каталоге /targets. Его можно выполнить с помощью такого исполняемого файла Java:

```
$ java -jar msal-client-credential-secret-1.0.0.jar
```

Когда приложение завершит работу, вы увидите список пользователей в настроенном клиенте.


> [!IMPORTANT]
> В этом кратком руководстве приложение использует секрет клиента для собственной идентификации в качестве конфиденциального клиента. Так как секрет клиента добавляется в качестве обычного текста в файлы проекта, из соображениям безопасности рекомендуется использовать сертификат вместо секрета клиента, прежде чем использовать приложение в качестве рабочего. Дополнительные сведения об использовании сертификата см. в [этих инструкциях](https://github.com/Azure-Samples/ms-identity-java-daemon/tree/master/msal-client-credential-certificate) в том же репозитории GitHub для этого примера, но во второй папке **msal-client-credential-certificate**.

## <a name="more-information"></a>Дополнительные сведения

### <a name="msal-java"></a>MSAL Java

Библиотека [MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java) нужна для обработки входа пользователей и запросов маркеров, которые используются для доступа к API, защищенному платформой удостоверений Майкрософт. Как описано выше, в этом кратком руководстве маркеры запрашиваются с использованием собственного удостоверения приложения вместо делегированных разрешений. Поток проверки подлинности, используемый в данном случае, называется *[потоком учетных данных клиента OAuth](v2-oauth2-client-creds-grant-flow.md)* . Дополнительные сведения об использовании MSAL Java с приложениями управляющей программы см. в [этой статье](scenario-daemon-overview.md).

Добавьте библиотеку MSAL4J в приложение с помощью Maven или Gradle для управления зависимостями, внеся следующие изменения в файл pom.xml (Maven) или build.gradle (Gradle) приложения.

В файле pom.xml:

```XML
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>msal4j</artifactId>
    <version>1.0.0</version>
</dependency>
```

В файле build.gradle:

```$xslt
compile group: 'com.microsoft.azure', name: 'msal4j', version: '1.0.0'
```

### <a name="msal-initialization"></a>Инициализация MSAL

Добавьте ссылку на MSAL для Java, добавив следующий код в начало файла, в котором будет использоваться MSAL4J:

```Java
import com.microsoft.aad.msal4j.*;
```

Затем выполните инициализацию MSAL с помощью следующего кода.

```Java
IClientCredential credential = ClientCredentialFactory.createFromSecret(CLIENT_SECRET);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

> | Где: |Описание |
> |---------|---------|
> | `CLIENT_SECRET` | Секрет клиента, созданный для приложения на портале Azure. |
> | `CLIENT_ID` | **Идентификатор приложения (клиента)** , зарегистрированного на портале Azure. Это значение можно найти на странице приложения **Обзор** на портале Azure. |
> | `AUTHORITY`    | Конечная точка STS для проверки подлинности пользователей. Обычно `https://login.microsoftonline.com/{tenant}` для общедоступного облака, где {tenant} — имя или идентификатор вашего клиента.|

### <a name="requesting-tokens"></a>Запрос маркеров

Чтобы запросить маркер с помощью удостоверение приложения, используйте метод `acquireToken`.

```Java
IAuthenticationResult result;
     try {
         SilentParameters silentParameters =
                 SilentParameters
                         .builder(SCOPE)
                         .build();

         // try to acquire token silently. This call will fail since the token cache does not
         // have a token for the application you are requesting an access token for
         result = cca.acquireTokenSilently(silentParameters).join();
     } catch (Exception ex) {
         if (ex.getCause() instanceof MsalException) {

             ClientCredentialParameters parameters =
                     ClientCredentialParameters
                             .builder(SCOPE)
                             .build();

             // Try to acquire a token. If successful, you should see
             // the token information printed out to console
             result = cca.acquireToken(parameters).join();
         } else {
             // Handle other exceptions accordingly
             throw ex;
         }
     }
     return result;
```

> |Где:| Описание |
> |---------|---------|
> | `SCOPE` | Содержит запрошенные области. Для конфиденциальных клиентов следует использовать формат, аналогичный `{Application ID URI}/.default`, который указывает, что запрашиваемые области — это те, которые статически определены в объекте приложения, заданном на портале Azure (для Microsoft Graph `{Application ID URI}` указывает на `https://graph.microsoft.com`). Для пользовательских веб-API `{Application ID URI}` определяется в разделе **Предоставление API** в разделе **Регистрации приложений** на портале Azure.|

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управляющих программах см. на главной странице сценария.

> [!div class="nextstepaction"]
> [Создание управляющей программы, которая вызывает веб-API](scenario-daemon-overview.md)

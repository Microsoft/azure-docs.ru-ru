---
title: Аутентификация и авторизация
description: Узнайте о различных способах проверки подлинности приложения или службы в пространственных привязках Azure, а также об уровнях управления, которым требуется шлюзовый доступ к пространственным привязкам.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: 01065f9ac26599d26d6e2a6979eae1e559a82854
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97722969"
---
# <a name="authentication-and-authorization-to-azure-spatial-anchors"></a>Аутентификация и авторизация в службе "Пространственные привязки Azure"

В этой статье вы узнаете о различных способах проверки подлинности в пространственных привязках Azure из приложения или веб-службы. Вы также узнаете о способах использования управления доступом на основе ролей Azure (Azure RBAC) в Azure Active Directory (Azure AD) для управления доступом к учетным записям пространственных привязок.

## <a name="overview"></a>Обзор

![Схема, в которой показан обзор проверки подлинности для пространственных привязок Azure.](./media/spatial-anchors-authentication-overview.png)

Чтобы получить доступ к конкретной учетной записи службы "Пространственные привязки Azure", клиенту необходимо сначала получить маркер доступа от службы токенов безопасности для Смешанной реальности Azure. Токены, полученные от службы STS, имеют время существования 24 часа. Они содержат сведения, используемые службами пространственных привязок для принятия решений об авторизации учетной записи и обеспечения доступа к учетной записи только авторизованным субъектам.

Маркеры доступа можно получить в Exchange для ключей или маркеров учетной записи, выданных Azure AD.

Ключи учетной записи позволяют быстро приступить к работе с помощью службы пространственных привязок Azure. Но перед развертыванием приложения в рабочей среде рекомендуется обновить приложение для использования проверки подлинности Azure AD.

Получить маркеры проверки подлинности Azure AD можно двумя способами.

- Если вы создаете корпоративное приложение и ваша компания использует Azure AD в качестве своей системы идентификации, вы можете использовать в своем приложении проверку подлинности Azure AD на основе пользователей. Затем вы предоставляете доступ к учетным записям пространственных привязок с помощью существующих групп безопасности Azure AD. Вы также можете предоставить доступ непосредственно пользователям в Организации.
- В противном случае рекомендуется получить токены Azure AD из веб-службы, которая поддерживает ваше приложение. Мы советуем использовать этот метод для рабочих приложений, так как он позволяет избежать внедрения учетных данных для доступа к пространственным привязкам Azure в клиентском приложении.

## <a name="account-keys"></a>Ключи учетной записи

Самый простой способ начать работу — использовать ключи учетной записи для доступа к учетной записи пространственных привязок Azure. Ключи учетной записи можно получить на портал Azure. Перейдите к своей учетной записи и перейдите на вкладку **ключи** :

![Снимок экрана, на котором показана вкладка "ключи" с кнопкой "Копировать" для выделенного первичного ключа.](../../../includes/media/spatial-anchors-get-started-create-resource/view-account-key.png)

Доступны два ключа. Оба они одновременно допустимы для доступа к учетной записи пространственных привязок. Рекомендуется регулярно обновлять ключ, используемый для доступа к учетной записи. Наличие двух отдельных допустимых ключей включает эти обновления без простоев. Также необходимо обновить только первичный ключ и вторичный ключ.

Пакет SDK имеет встроенную поддержку проверки подлинности с помощью ключей учетной записи. Необходимо просто задать `AccountKey` свойство для `cloudSession` объекта:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AccountKey = @"MyAccountKey";
```

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
_cloudSession.configuration.accountKey = @"MyAccountKey";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.accountKey = "MyAccountKey"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAccountKey("MyAccountKey");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AccountKey(R"(MyAccountKey)");
```

# <a name="cwinrt"></a>[C++/WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AccountKey(LR"(MyAccountKey)");
```

---

После задания этого свойства пакет SDK будет управлять обменом ключами учетной записи для маркера доступа и необходимым кэшированием токенов для приложения.

> [!WARNING]
> Для быстрой адаптации рекомендуется использовать ключи учетной записи, но только во время разработки или создания прототипов. Мы не рекомендуем поставлять приложение в рабочую среду с внедренным ключом учетной записи. Вместо этого используйте способы проверки подлинности Azure AD на основе пользователей или служб, описанные далее.

## <a name="azure-ad-user-authentication"></a>Аутентификация пользователей с помощью AAD

Для приложений, предназначенных для Azure Active Directory пользователей, рекомендуется использовать маркер Azure AD для пользователя. Этот токен можно получить с помощью [MSAL](../../active-directory/develop/msal-overview.md). Выполните действия, описанные в [кратком руководстве по регистрации приложения](../../active-directory/develop/quickstart-register-app.md), в том числе:

**На портале Azure:**
1.    Зарегистрируйте приложение в Azure AD в качестве собственного приложения. В процессе регистрации необходимо определить, должно ли приложение быть клиентом. Вам также потребуется указать URL-адреса перенаправления для приложения.
1.  Перейдите на вкладку **разрешения API** .
2.  Выберите **Добавить разрешение**.
    1.  Выберите **поставщик ресурсов смешанной реальности** на вкладке API, которые **использует Моя организация** .
    2.  Нажмите **Делегированные разрешения**.
    3.  Выберите **микседреалити. Signing** in **микседреалити**.
    4.  Выберите **Добавить разрешения**.
3.  Выберите **предоставить согласие администратора**.

2. Предоставьте приложению или пользователям доступ к вашему ресурсу:
   1.    Перейдите к ресурсу пространственных привязок в портал Azure.
   2.    Перейдите на вкладку **Управление доступом (IAM)** .
   3.    Выберите **Добавить назначение ролей**.
   1.    [Выберите роль](#azure-role-based-access-control).
   2.    В поле **выбрать** введите имена пользователей, групп и (или) приложений, которым требуется назначить доступ.
   3.    Щелкните **Сохранить**.

**В коде**
1.    Не забудьте использовать идентификатор приложения и URI перенаправления своего приложения Azure AD для параметров **Client ID** и **RedirectUri** в MSAL.
2.    Введите сведения об арендаторе.
        1.    Если приложение поддерживает **только мою организацию**, замените это значение на **идентификатор клиента** или **имя клиента**. Например, contoso.microsoft.com.
        2.    Если приложение поддерживает **учетные записи в любом каталоге Организации**, замените это значение **организациями**.
        3.    Если приложение поддерживает **все учетная запись Майкрософт пользователей**, замените это значение на **Common**.
3.    В запросе токена задайте для **области** значение " **`https://sts.<account-domain>//.default` "**, где `<account-domain>` будет заменен **домен учетной** записи для учетной записи пространственных привязок Azure. В качестве примера области для учетной записи пространственных привязок Azure в домене "Восточная часть США 2" используется учетная запись **" `https://sts.mixedreality.azure.com//.default` "**. Этот параметр будет указывать на службу AAD, у которой приложение запрашивает маркер для службы токенов безопасности для Смешанной реальности.

После выполнения этих действий приложение должно получить доступ с MSAL маркера Azure AD. Вы можете задать маркер Azure AD в качестве значения `authenticationToken` для объекта конфигурации сеанса облака:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AuthenticationToken = @"MyAuthenticationToken";
```

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
_cloudSession.configuration.authenticationToken = @"MyAuthenticationToken";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.authenticationToken = "MyAuthenticationToken"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAuthenticationToken("MyAuthenticationToken");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AuthenticationToken(R"(MyAuthenticationToken)");
```

# <a name="cwinrt"></a>[C++/WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AuthenticationToken(LR"(MyAuthenticationToken)");
```

---

## <a name="azure-ad-service-authentication"></a>Аутентификация с помощью AAD на уровне служб

Для развертывания приложений, использующих пространственные привязки Azure в рабочей среде, рекомендуется использовать серверную службу, которая будет выполнять запросы на проверку подлинности брокера. Ниже приведен обзор этого процесса.

![Схема, которая содержит общие сведения о проверке подлинности для пространственных привязок Azure.](./media/spatial-anchors-aad-authentication.png)

В этом случае предполагается, что приложение использует собственный механизм для аутентификации в серверной службе. (Например, учетная запись Майкрософт, PlayFab, Facebook, идентификатор Google или пользовательское имя пользователя и пароль).  После проверки подлинности пользователей в серверной службе эта служба может получить маркер Azure AD, обмениваться маркером доступа для пространственных привязок Azure и вернуть ее обратно в клиентское приложение.

Маркер доступа Azure AD извлекается через [MSAL](../../active-directory/develop/msal-overview.md). Выполните действия, описанные в [кратком руководстве по регистрации приложений](../../active-directory/develop/quickstart-register-app.md), в том числе:

**На портале Azure:**
1.    Зарегистрируйте приложение в AAD.
        1.    В портал Azure выберите **Azure Active Directory**, а затем выберите **Регистрация приложений**.
        2.    Выберите **Новая регистрация**.
        3.    Введите имя приложения, выберите для него тип **Веб-приложение или API** и введите URL-адрес аутентификации для своей службы. Нажмите кнопку **создания**.
2.    В приложении выберите **Параметры**, а затем выберите вкладку **Сертификаты и секреты** . Создайте новый секрет клиента, выберите длительность и нажмите кнопку **Добавить**. Не забудьте сохранить значение секрета. Его необходимо включить в код веб-службы.
3.    Предоставьте приложению и (или) пользователям доступ к своему ресурсу.
        1.    Перейдите к ресурсу пространственных привязок в портал Azure.
        2.    Перейдите на вкладку **Управление доступом (IAM)** .
        3.    Выберите **Добавить назначение ролей**.
        4.    [Выберите роль](#azure-role-based-access-control).
        5.    В поле **выбрать** введите имя или имена приложений, которым требуется назначить доступ. Если вы хотите, чтобы пользователи приложения имели разные роли для учетной записи пространственных привязок, зарегистрируйте несколько приложений в Azure AD и назначьте каждому из них отдельную роль. Затем реализуйте логику авторизации, чтобы пользователи получали нужные роли.

              > [!NOTE]
              > В области **Добавление назначения ролей** в в поле **назначить доступ к** выберите **пользователь Azure AD, группа или субъект-служба**.

        6.    Щелкните **Сохранить**.

**В коде**

>[!NOTE]
> Вы можете использовать [Пример службы](https://github.com/Azure/azure-spatial-anchors-samples/tree/master/Sharing/SharingServiceSample) , который доступен как часть [примеров приложений пространственных привязок](https://github.com/Azure/azure-spatial-anchors-samples).

1.    Не забудьте использовать идентификатор приложения, секрет приложения и URI перенаправления своего приложения Azure AD в качестве **идентификатора клиента**, **секрета** и параметров **RedirectUri** в MSAL.
2.    Задайте идентификатор клиента в качестве идентификатора клиента Azure AD в параметре **Authority** в MSAL.
3.    В запросе токена задайте для **области** значение " **`https://sts.<account-domain>//.default` "**, где `<account-domain>` будет заменен **домен учетной** записи для учетной записи пространственных привязок Azure. В качестве примера области для учетной записи пространственных привязок Azure в домене "Восточная часть США 2" используется учетная запись **" `https://sts.mixedreality.azure.com//.default` "**.

После выполнения этих действий серверная служба может получить маркер Azure AD. Затем она обменяет его на токен СР, который будет возвращен клиенту. Для получения токена СР в обмен на токен AAD используется вызов REST. Вот пример вызова:

```
GET https://sts.mixedreality.azure.com/Accounts/35d830cb-f062-4062-9792-d6316039df56/token HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni<truncated>FL8Hq5aaOqZQnJr1koaQ
Host: sts.mixedreality.azure.com
Connection: Keep-Alive

HTTP/1.1 200 OK
Date: Sun, 24 Feb 2019 08:00:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 1153
Accept: application/json
MS-CV: 05JLqWeKFkWpbdY944yl7A.0
{"AccessToken":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjI2MzYyMTk5ZTI2NjQxOGU4ZjE3MThlM2IyMThjZTIxIiwidHlwIjoiSldUIn0.eyJqdGkiOiJmMGFiNWIyMy0wMmUxLTQ1MTQtOWEzNC0xNzkzMTA1NTc4NzAiLCJjYWkiOiIzNWQ4MzBjYi1mMDYyLTQwNjItOTc5Mi1kNjMxNjAzOWRmNTYiLCJ0aWQiOiIwMDAwMDAwMC0wMDAwLTAwMDAtMDAwMC0wMDAwMDAwMDAwMDAiLCJhaWQiOiIzNWQ4MzBjYi1mMDYyLTQwNjItOTc5Mi1kNjMxNjAzOWRmNTYiLCJhYW8iOi0xLCJhcHIiOiJlYXN0dXMyIiwicmlkIjoiL3N1YnNjcmlwdGlvbnMvNzIzOTdlN2EtNzA4NC00ODJhLTg3MzktNjM5Y2RmNTMxNTI0L3Jlc291cmNlR3JvdXBzL3NhbXBsZV9yZXNvdXJjZV9ncm91cC9wcm92aWRlcnMvTWljcm9zb2Z0Lk1peGVkUmVhbGl0eS9TcGF0aWFsQW5jaG9yc0FjY291bnRzL2RlbW9fYWNjb3VudCIsIm5iZiI6MTU0NDU0NzkwMywiZXhwIjoxNTQ0NjM0MzAzLCJpYXQiOjE1NDQ1NDc5MDMsImlzcyI6Imh0dHBzOi8vbXJjLWF1dGgtcHJvZC50cmFmZmljbWFuYWdlci5uZXQvIiwiYXVkIjoiaHR0cHM6Ly9tcmMtYW5jaG9yLXByb2QudHJhZmZpY21hbmFnZXIubmV0LyJ9.BFdyCX9UJj0i4W3OudmNUiuaGgVrlPasNM-5VqXdNAExD8acFJnHdvSf6uLiVvPiQwY1atYyPbOnLYhEbIcxNX-YAfZ-xyxCKYb3g_dbxU2w8nX3zDz_X3XqLL8Uha-rkapKbnNgxq4GjM-EBMCill2Svluf9crDmO-SmJbxqIaWzLmlUufQMWg_r8JG7RLseK6ntUDRyDgkF4ex515l2RWqQx7cw874raKgUO4qlx0cpBAB8cRtGHC-3fA7rZPM7UQQpm-BC3suXqRgROTzrKqfn_g-qTW4jAKBIXYG7iDefV2rGMRgem06YH_bDnpkgUa1UgJRRTckkBuLkO2FvA"}
```

Заголовок авторизации имеет следующий формат: `Bearer <Azure_AD_token>`

Ответ содержит токен MR в виде обычного текста.

Этот токен СР возвращается клиенту. Затем клиентское приложение может задать его в качестве маркера доступа в конфигурации сеанса облака:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AccessToken = @"MyAccessToken";
```

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
_cloudSession.configuration.accessToken = @"MyAccessToken";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.accessToken = "MyAccessToken"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAccessToken("MyAccessToken");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AccessToken(R"(MyAccessToken)");
```

# <a name="cwinrt"></a>[C++/WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AccessToken(LR"(MyAccessToken)");
```

---

## <a name="azure-role-based-access-control"></a>Управление доступом на основе ролей в Azure

Чтобы помочь вам управлять уровнем доступа, предоставляемого приложениям, службам или пользователям Azure AD вашей службы, вы можете назначить эти существующие роли в соответствии с учетными записями пространственных привязок Azure:

- **Владелец учетной записи пространственных привязок**. Приложения или пользователи с этой ролью могут создавать пространственные привязки, запрашивать их и удалять. При проверке подлинности учетной записи с помощью ключей учетной записи роль владельца учетной записи пространственных привязок назначается участнику, прошедшему проверку подлинности.
- **Участник учетной записи пространственных привязок**. Приложения или пользователи с этой ролью могут создавать пространственные привязки и запрашивать их, но не могут удалять их.
- **Средство чтения учетной записи пространственных привязок**. Приложения или пользователи с этой ролью могут запрашивать только пространственные привязки. Они не могут создавать новые, удалять существующие или обновлять метаданные. Эта роль обычно используется для приложений, в которых некоторые пользователи изменяют среду, но другие могут отзывать только те привязки, которые были ранее помещены в среду.

## <a name="next-steps"></a>Дальнейшие действия

Создайте свое первое приложение с пространственными привязками Azure:

> [!div class="nextstepaction"]
> [Unity (HoloLens)](../quickstarts/get-started-unity-hololens.md)

> [!div class="nextstepaction"]
> [Unity (iOS)](../quickstarts/get-started-unity-ios.md)

> [!div class="nextstepaction"]
> [Unity (Android)](../quickstarts/get-started-unity-android.md)

> [!div class="nextstepaction"]
> [iOS](../quickstarts/get-started-ios.md)

> [!div class="nextstepaction"]
> [Android](../quickstarts/get-started-android.md)

> [!div class="nextstepaction"]
> [HoloLens](../quickstarts/get-started-hololens.md)

> [!div class="nextstepaction"]
> [Xamarin (Android)](../quickstarts/get-started-xamarin-android.md)

> [!div class="nextstepaction"]
> [Xamarin (iOS)](../quickstarts/get-started-xamarin-ios.md)

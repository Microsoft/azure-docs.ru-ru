---
title: Общедоступные клиентские приложения для одной и нескольких учетных записей | Службы
description: Обзор общедоступных клиентских приложений с одним и несколькими учетными записями.
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/26/2019
ms.author: shoatman
ms.custom: aaddev
ms.reviewer: shoatman
ms.openlocfilehash: 5bea1753c87c11094e78f95a1bbadb02fb0b95e2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98752975"
---
# <a name="single-and-multiple-account-public-client-apps"></a>Общедоступные клиентские приложения для одной и нескольких учетных записей

Эта статья поможет вам понять, какие типы используются в общедоступных клиентских приложениях с одним и несколькими учетными записями, и сосредоточиться на общедоступных клиентских приложениях одной учетной записи. 

Библиотека проверки подлинности Azure Active Directory (ADAL) моделирует сервер.  Вместо этого в библиотеке проверки подлинности Майкрософт (MSAL) клиентское приложение моделируется.  Большинство приложений Android считаются общедоступными клиентами. Общедоступный клиент — это приложение, которое не может безопасно хранить секретный код.  

MSAL специализирует поверхность API `PublicClientApplication` в для упрощения и уточнения процесса разработки для приложений, которые позволяют использовать только одну учетную запись одновременно. `PublicClientApplication` является подклассом `SingleAccountPublicClientApplication` и `MultipleAccountPublicClientApplication` .  На следующей схеме показана связь между этими классами.

![Схема классов UML Синглеаккаунтпубликклиентаппликатион](./media/single-multi-account/single-and-multiple-account.png)

## <a name="single-account-public-client-application"></a>Общедоступное клиентское приложение с одной учетной записью

`SingleAccountPublicClientApplication`Класс позволяет создать приложение на основе MSAL, которое разрешает вход только в одну учетную запись за раз. `SingleAccountPublicClientApplication` имеет следующие отличия от `PublicClientApplication` :

- MSAL отслеживает текущую учетную запись для входа.
  - Если приложение использует брокер (по умолчанию во время регистрации приложения портал Azure) и устанавливается на устройстве, где имеется брокер, MSAL проверит, что учетная запись по-прежнему доступна на устройстве.
- `signIn` позволяет выполнять вход в учетную запись явным образом и отдельно от запроса областей.
- `acquireTokenSilent` не требуется параметр учетной записи.  Если вы предоставляете учетную запись, а указанная учетная запись не соответствует текущей учетной записи, отслеживающей MSAL, `MsalClientException` создается исключение.
- `acquireToken` не позволяет пользователю переключать учетные записи. Если пользователь пытается переключиться на другую учетную запись, возникает исключение.
- `getCurrentAccount` Возвращает результирующий объект, который предоставляет следующие сведения:
  - Логическое значение, указывающее, изменилась ли учетная запись. Например, учетную запись можно изменить в результате удаления с устройства.
  - Прошлая учетная запись. Это полезно, если необходимо выполнить очистку локальных данных при удалении учетной записи с устройства или при входе в новую учетную запись.
  - Куррентаккаунт.
- `signOut` Удаляет все токены, связанные с клиентом, с устройства.  

Если на устройстве установлен брокер проверки подлинности Android, например Microsoft Authenticator или Корпоративный портал Intune, а приложение настроено для использования брокера, `signOut` учетная запись не удаляется с устройства.

## <a name="single-account-scenario"></a>Сценарий с одной учетной записью

Следующий псевдокод иллюстрирует использование `SingleAccountPublicClientApplication` .

```java
// Construct Single Account Public Client Application
ISingleAccountPublicClientApplication app = PublicClientApplication.createSingleAccountPublicClientApplication(getApplicationContext(), R.raw.msal_config);

String[] scopes = {"User.Read"};
IAccount mAccount = null;

// Acquire a token interactively
// The user will get a UI prompt before getting the token.
app.signIn(getActivity(), scopes, new AuthenticationCallback()
{

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) 
        {
            mAccount = authenticationResult.getAccount();
        }

        @Override
        public void onError(MsalException exception)
        {
        }

        @Override
        public void onCancel()
        {
        }
    }
);

// Load Account Specific Data
getDataForAccount(account);

// Get Current Account
ICurrentAccountResult currentAccountResult = app.getCurrentAccount();
if (currentAccountResult.didAccountChange())
{
    // Account Changed Clear existing account data
    clearDataForAccount(currentAccountResult.getPriorAccount());
    mAccount = currentAccountResult.getCurrentAccount();
    if (account != null)
    {
        //load data for new account
        getDataForAccount(account);
    }
}

// Sign out
if (app.signOut())
{
    clearDataForAccount(mAccount);
    mAccount = null;
}
```

## <a name="multiple-account-public-client-application"></a>Общедоступное клиентское приложение с несколькими учетными записями

`MultipleAccountPublicClientApplication`Класс используется для создания приложений на основе MSAL, которые позволяют одновременно войти в несколько учетных записей. Он позволяет получать, добавлять и удалять учетные записи следующим образом.

### <a name="add-an-account"></a>Добавить учетную запись

Используйте одну или несколько учетных записей в приложении, вызвав `acquireToken` один или несколько раз.  

### <a name="get-accounts"></a>Получение учетных записей

- Вызовите `getAccount` , чтобы получить определенную учетную запись.
- Вызовите `getAccounts` , чтобы получить список учетных записей, известных в настоящее время для приложения.

Приложение не сможет перечислять все учетные записи платформы Microsoft Identity на устройстве, известном для приложения брокера. Он может перечислять только учетные записи, используемые приложением.  Учетные записи, удаленные с устройства, не будут возвращены этими функциями.

### <a name="remove-an-account"></a>Удаление учетной записи

Удалите учетную запись, вызвав ее `removeAccount` с помощью идентификатора учетной записи.

Если приложение настроено для использования брокера и на устройстве установлен брокер, то при вызове учетная запись не будет удалена из брокера `removeAccount` .  Удаляются только маркеры, связанные с клиентом.

## <a name="multiple-account-scenario"></a>Сценарий с несколькими учетными записями

В следующем псевдокоде показано, как создать приложение с несколькими учетными записями, вывести список учетных записей на устройстве и получить маркеры.

```java
// Construct Multiple Account Public Client Application
IMultipleAccountPublicClientApplication app = PublicClientApplication.createMultipleAccountPublicClientApplication(getApplicationContext(), R.raw.msal_config);

String[] scopes = {"User.Read"};
IAccount mAccount = null;

// Acquire a token interactively
// The user will be required to interact with a UI to obtain a token
app.acquireToken(getActivity(), scopes, new AuthenticationCallback()
 {

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) 
        {
            mAccount = authenticationResult.getAccount();
        }

        @Override
        public void onError(MsalException exception)
        {
        }

        @Override
        public void onCancel()
        {
        }
 });

...

// Get the default authority
String authority = app.getConfiguration().getDefaultAuthority().getAuthorityURL().toString();

// Get a list of accounts on the device
List<IAccount> accounts = app.getAccounts();

// Pick an account to obtain a token from without prompting the user to sign in
IAccount selectedAccount = accounts.get(0);

// Get a token without prompting the user
app.acquireTokenSilentAsync(scopes, selectedAccount, authority, new SilentAuthenticationCallback()
{

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) 
        {
            mAccount = authenticationResult.getAccount();
        }

        @Override
        public void onError(MsalException exception)
        {
        }
});
```

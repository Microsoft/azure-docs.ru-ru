---
title: Руководством по миграции ADAL в MSAL (MSAL iOS/macOS) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте о различиях между MSAL для iOS и macOS и библиотекой аутентификация Azure AD для ObjectiveC (ADAL. ObjC) и как перейти на MSAL для iOS/macOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev
ms.openlocfilehash: 7dc3241198fbc6eeddba059251f28c6dc35c8a29
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98754942"
---
# <a name="migrate-applications-to-msal-for-ios-and-macos"></a>Перенос приложений в MSAL для iOS и macOS

Библиотека проверки подлинности Azure Active Directory ([ADAL цель-C](https://github.com/AzureAD/azure-activedirectory-library-for-objc)) была создана для работы с учетными записями Azure Active Directory с помощью конечной точки v 1.0.

Библиотека проверки подлинности Microsoft для iOS и macOS (MSAL) предназначена для работы со всеми удостоверениями Майкрософт, такими как учетные записи Azure Active Directory (Azure AD), личные учетные записи Майкрософт и Azure AD B2C учетные записи с помощью платформы идентификации Майкрософт (формально — конечная точка Azure AD версии 2.0).

Платформа Microsoft Identity имеет несколько ключевых отличий с Azure Active Directory v 1.0. В этой статье описываются эти различия и приводятся рекомендации по переносу приложения из ADAL в MSAL.

## <a name="adal-and-msal-app-capability-differences"></a>Отличия от возможностей приложения ADAL и MSAL

### <a name="who-can-sign-in"></a>Каким учетным записям разрешен вход

* ADAL поддерживает только рабочие и учебные учетные записи (также называемые учетными записями Azure AD).
* MSAL поддерживает личные учетные записи Майкрософт (учетные записи MSA), такие как Hotmail.com, Outlook.com и Live.com.
* MSAL поддерживает рабочие и учебные учетные записи, а Azure AD B2C учетные записи.

### <a name="standards-compliance"></a>Соответствие стандартам

* Платформа Microsoft Identity соответствует стандартам OAuth 2,0 и OpenID Connect Connect.

### <a name="incremental-and-dynamic-consent"></a>Добавочное и динамическое согласие

* Конечная точка Azure Active Directory v 1.0 требует, чтобы все разрешения были объявлены заранее при регистрации приложения. Это означает, что эти разрешения являются статическими.
* Платформа Microsoft Identity позволяет запрашивать разрешения динамически. Приложения могут запрашивать разрешения только по мере необходимости и запрашивать больше по мере того, как приложение нуждается в них.

Дополнительные сведения о различиях между Azure Active Directory v 1.0 и платформой идентификации Майкрософт см. [в статье Зачем обновлять платформу Microsoft Identity](../azuread-dev/azure-ad-endpoint-comparison.md).

## <a name="adal-and-msal-library-differences"></a>Отличия библиотек ADAL и MSAL

Общедоступный API MSAL отражает несколько ключевых различий между Azure AD v 1.0 и платформой идентификации Майкрософт.

### <a name="msalpublicclientapplication-instead-of-adauthenticationcontext"></a>Мсалпубликклиентаппликатион вместо Адаусентикатионконтекст

`ADAuthenticationContext` — Первый объект, создаваемый приложением ADAL. Он представляет экземпляр ADAL. Приложения создают новый экземпляр `ADAuthenticationContext` для каждой Azure Active Directory сочетание облака и клиента (центра). То же самое `ADAuthenticationContext` можно использовать для получения маркеров для нескольких общедоступных клиентских приложений.

В MSAL основное взаимодействие осуществляется через `MSALPublicClientApplication` объект, который моделируется после [общедоступного клиента OAuth 2,0](https://tools.ietf.org/html/rfc6749#section-2.1). Один экземпляр `MSALPublicClientApplication` можно использовать для взаимодействия с несколькими облаками AAD и клиентами без необходимости создавать новый экземпляр для каждого центра. Для большинства приложений достаточно одного `MSALPublicClientApplication` экземпляра.

### <a name="scopes-instead-of-resources"></a>Области вместо ресурсов

В ADAL приложению пришлось предоставить идентификатор *ресурса* , например, `https://graph.microsoft.com` для получения маркеров от конечной точки Azure Active Directory v 1.0. Ресурс может определять ряд областей или oAuth2Permissions в манифесте приложения, который он понимает. Это позволяло клиентским приложениям запрашивать маркеры из этого ресурса для определенного набора областей, предварительно определенных во время регистрации приложения.

В MSAL вместо одного идентификатора ресурса приложения предоставляют набор областей для каждого запроса. Область — это идентификатор ресурса, за которым следует имя разрешения в виде ресурса или разрешения. Например: `https://graph.microsoft.com/user.read`

Существует два способа предоставления областей в MSAL:

* Предоставьте список всех разрешений, необходимых вашим приложениям. Пример: 

    `@[@"https://graph.microsoft.com/directory.read", @"https://graph.microsoft.com/directory.write"]`

    В этом случае приложение запрашивает `directory.read` `directory.write` разрешения и. Пользователю будет предложено предоставить согласие на предоставление этих разрешений, если они не были им предоставлены ранее для этого приложения. Приложение может также получить дополнительные разрешения, которые пользователь уже предоставил для приложения. Пользователю будет предложено подтвердить только новые разрешения или разрешения, которые не были предоставлены.

* Область `/.default`.

Это встроенная область для каждого приложения. Он относится к статическому списку разрешений, настроенных при регистрации приложения. Его поведение аналогично поведению `resource` . Это может быть полезно при миграции, чтобы обеспечить соблюдение аналогичного набора областей и взаимодействия с пользователем.

Чтобы использовать `/.default` область, добавьте `/.default` к идентификатору ресурса. Например, `https://graph.microsoft.com/.default`. Если ресурс заканчивается косой чертой ( `/` ), необходимо по-прежнему добавлять `/.default` , включая предшествующую косую черту, что приводит к области с двойной косой чертой ( `//` ) в ней.

Дополнительные сведения об использовании области "/.дефаулт" можно прочитать [здесь](./v2-permissions-and-consent.md#the-default-scope) .

### <a name="supporting-different-webview-types--browsers"></a>Поддержка различных типов WebView & браузерах

ADAL поддерживает только Уивебвиев/Вквебвиев для iOS и WebView для macOS. MSAL для iOS поддерживает больше возможностей для отображения веб-содержимого при запросе кода авторизации и более не поддерживает, `UIWebView` что может улучшить взаимодействие с пользователем и безопасность.

По умолчанию MSAL в iOS использует [асвебаусентикатионсессион](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession?language=objc), который является веб-компонентом Apple рекомендует проверку подлинности на устройствах iOS 12 +. Она предоставляет преимущества единого Sign-On (SSO) через общий доступ к файлам cookie между приложениями и браузер Safari.

Вы можете использовать другой веб-компонент в зависимости от требований приложения и желаемого взаимодействия с конечным пользователем. Дополнительные параметры см. в разделе [Поддерживаемые типы веб-представлений](customize-webviews.md) .

При переходе с ADAL на MSAL `WKWebView` обеспечивает взаимодействие с пользователем, похожее на ADAL в iOS и macOS. Мы рекомендуем перейти на `ASWebAuthenticationSession` iOS, если это возможно. Для macOS мы рекомендуем использовать `WKWebView` .

### <a name="account-management-api-differences"></a>Различия в API управления учетными записями

При вызове методов ADAL `acquireToken()` или `acquireTokenSilent()` вы получаете `ADUserInformation` объект, содержащий список утверждений от `id_token` , представляющий учетную запись, для которой выполняется проверка подлинности. Кроме того, `ADUserInformation` возвращает объект, `userId` основанный на `upn` утверждении. После первоначального обращения к интерактивному получению маркера ADAL требует, чтобы разработчик предоставил `userId` все автоматические вызовы.

ADAL не предоставляет API для получения удостоверений известных пользователей. Он использует приложение для сохранения этих учетных записей и управления ими.

MSAL предоставляет набор API-интерфейсов для перечисления всех учетных записей, известных MSAL, без необходимости получения маркера.

Как и ADAL, MSAL возвращает сведения об учетной записи, содержащую список утверждений из `id_token` . Он является частью `MSALAccount` объекта внутри `MSALResult` объекта.

MSAL предоставляет набор интерфейсов API для удаления учетных записей, делая удаленные учетные записи недоступными для приложения. После удаления учетной записи последующие вызовы для получения маркера будут предлагать пользователю выполнить интерактивное получение маркера. Удаление учетной записи применяется только к клиентскому приложению, которое его запустило, и не удаляет учетную запись из других приложений, запущенных на устройстве или в браузере системы. Это гарантирует, что пользователь будет иметь возможность единого входа на устройстве даже после выхода из отдельного приложения.

Кроме того, MSAL также возвращает идентификатор учетной записи, который можно использовать для автоматического запроса маркера. Тем не менее, идентификатор учетной записи (доступный через `identifier` свойство в `MSALAccount` объекте) не может быть воспроизведен, и вы не можете предположить, какой формат имеет значение, или же вы не должны пытаться интерпретировать или проанализировать его.

### <a name="migrating-the-account-cache"></a>Перенос кэша учетной записи

При переходе с ADAL приложения обычно хранят ADAL `userId` , у которых нет `identifier` необходимых для MSAL. При однократной миграции приложение может запросить учетную запись MSAL с помощью ADAL userId со следующим API:

`- (nullable MSALAccount *)accountForUsername:(nonnull NSString *)username error:(NSError * _Nullable __autoreleasing * _Nullable)error;`

Этот API считывает кэш MSAL и ADAL, чтобы найти учетную запись с помощью ADAL userId (UPN).

Если учетная запись найдена, разработчик должен использовать учетную запись для автоматического получения маркера. Первое получение маркера без уведомления будет эффективно обновлять учетную запись, и разработчик получит MSAL идентификатор учетной записи, совместимый в MSAL Result ( `identifier` ). После этого `identifier` следует использовать только для поиска учетных записей с помощью следующего API:

`- (nullable MSALAccount *)accountForIdentifier:(nonnull NSString *)identifier error:(NSError * _Nullable __autoreleasing * _Nullable)error;`

Хотя можно продолжать использовать ADAL `userId` для всех операций в MSAL, так как оно `userId` основано на имени участника-пользователя, оно подчиняется нескольким ограничениям, которые приводят к неправильному взаимодействиям с пользователем. Например, если имя участника-пользователя изменяется, пользователь должен снова войти в систему. Мы рекомендуем, чтобы все приложения использовали неотображаемую учетную запись `identifier` для всех операций.

Узнайте больше о [миграции состояния кэша](sso-between-adal-msal-apps-macos-ios.md).

### <a name="token-acquisition-changes"></a>Изменения в получении маркера

MSAL вводит некоторые изменения в вызовах для получения маркера:

* Как и ADAL, `acquireTokenSilent` всегда приводит к автоматическому запросу.
* В отличие от ADAL, всегда приводит к пользовательскому `acquireToken` интерфейсу с пользовательскими действиями — через веб-представление или приложение Microsoft Authenticator. В зависимости от состояния единого входа в WebView/Microsoft Authenticator пользователю может быть предложено ввести учетные данные.
* В ADAL при `acquireToken` `AD_PROMPT_AUTO` первом попытается выполнить автоматическое получение маркера и только в случае сбоя автоматического запроса. В MSAL эта логика может быть достигнута первым вызовом `acquireTokenSilent` и вызывается только в `acquireToken` случае сбоя автоматического приобретения. Это позволяет разработчикам настраивать взаимодействие с пользователем перед началом интерактивного получения маркера.

### <a name="error-handling-differences"></a>Различия в обработке ошибок

MSAL обеспечивает более ясность между ошибками, которые могут быть обработаны приложением, и теми, которые нуждаются в вмешательстве пользователя. Существует ограниченное число ошибок, которые разработчик должен иметь в обработке:

* `MSALErrorInteractionRequired`: пользователь должен выполнить интерактивный запрос. Это может быть вызвано разными причинами, такими как сеанс проверки подлинности с истекшим сроком действия, изменение политики условного доступа, срок действия маркера обновления истек или он был отозван, нет допустимых токенов в кэше и т. д.
* `MSALErrorServerDeclinedScopes`: Запрос не полностью завершен, и для некоторых областей не предоставлен доступ. Это может быть вызвано тем, что пользователь отменяет согласие на одну или несколько областей.

Обработка всех остальных ошибок в [ `MSALError` списке](https://github.com/AzureAD/microsoft-authentication-library-for-objc/blob/master/MSAL/src/public/MSALError.h#L128) является необязательной. Сведения в этих ошибках можно использовать для улучшения взаимодействия с пользователем.

Дополнительные сведения об обработке ошибок MSAL см. [в разделе Обработка исключений и ошибок с помощью MSAL](msal-error-handling-ios.md) .

### <a name="broker-support"></a>Поддержка брокера

MSAL, начиная с версии 0.3.0, обеспечивает поддержку проверки подлинности через посредника с помощью приложения Microsoft Authenticator. Microsoft Authenticator также обеспечивает поддержку сценариев условного доступа. Примеры сценариев условного доступа включают политики соответствия устройств, требующие регистрации устройства в Intune или регистрации в AAD для получения маркера. И политики условного доступа управления мобильными приложениями (MAM), требующие подтверждения соответствия, прежде чем приложение сможет получить маркер.

Чтобы включить брокер для приложения, выполните следующие действия.

1. Зарегистрируйте формат URI перенаправления, совместимого с брокером, для приложения. Формат URI для перенаправления, совместимый с брокером: `msauth.<app.bundle.id>://auth` . Замените `<app.bundle.id>` идентификатором пакета своего приложения. Если вы выполняете миграцию из ADAL и ваше приложение уже поддерживает брокер, вам ничего не нужно делать. Предыдущий URI перенаправления полностью совместим с MSAL, поэтому можно перейти к шагу 3.

2. Добавьте схему URI перенаправления приложения в файл info. plist. Для URI перенаправления MSAL по умолчанию используется формат `msauth.<app.bundle.id>` . Пример:

    ```xml
    <key>CFBundleURLSchemes</key>
    <array>
        <string>msauth.<app.bundle.id></string>
    </array>
    ```

3. Добавьте следующие схемы в файле info. plist приложения в разделе Лсаппликатионкуериессчемес:

    ```xml
    <key>LSApplicationQueriesSchemes</key>
    <array>
         <string>msauthv2</string>
         <string>msauthv3</string>
    </array>
    ```

4. Добавьте следующий объект в файл AppDelegate. m для управления обратными вызовами: цель-C:
    
    ```objc
    - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options`
    {
        return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```
    
    Swift:
    
    ```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
    }
    ```

### <a name="business-to-business-b2b"></a>B2B (бизнес-бизнес)

В ADAL вы создаете отдельные экземпляры `ADAuthenticationContext` для каждого клиента, для которого приложение запрашивает маркеры. Это больше не требуется в MSAL. В MSAL можно создать один экземпляр `MSALPublicClientApplication` и использовать его для любого облака AAD и организации, указав другой центр для вызовов acquireToken и acquireTokenSilent.

## <a name="sso-in-partnership-with-other-sdks"></a>Единый вход в связи с другими пакетами SDK

MSAL для iOS может обеспечить единый вход через единый кэш со следующими пакетами SDK:

- Зацель ADAL-C 2.7. x +
- MSAL.NET для Xamarin 2.4. x +
- ADAL.NET для Xamarin 4.4. x +

Единый вход осуществляется через общий доступ к цепочке ключей iOS и доступен только между приложениями, опубликованными из той же учетной записи разработчика Apple.

Единый вход через обмен цепочкой ключей iOS является единственным необъявляемым типом единого входа.

В macOS MSAL может обеспечить единый вход с другими MSAL для приложений на основе iOS и macOS и приложений на основе ADAL, основанных на C.

MSAL в iOS также поддерживает два других типа единого входа:

* Единый вход через веб-браузер. MSAL для iOS поддерживает `ASWebAuthenticationSession` , что обеспечивает единый вход через файлы cookie, совместно используемые другими приложениями на устройстве, а именно в браузере Safari.
* Единый вход через брокер аутентификации. На устройстве iOS Microsoft Authenticator выступает в качестве брокера проверки подлинности. Он может следовать политикам условного доступа, таким как требовать соответствующее устройство, и обеспечивает единый вход для зарегистрированных устройств. Пакеты SDK MSAL, начиная с версии 0.3.0, поддерживают брокер по умолчанию.

## <a name="intune-mam-sdk"></a>Пакет SDK для Intune MAM

[Пакет SDK для INTUNE MAM](/intune/app-sdk-get-started) поддерживает MSAL для iOS начиная с версии [11.1.2](https://github.com/msintuneappsdk/ms-intune-app-sdk-ios/releases/tag/11.1.2)

## <a name="msal-and-adal-in-the-same-app"></a>MSAL и ADAL в одном приложении

ADAL версии 2.7.0 и выше не может сосуществовать с MSAL в одном приложении. Основная причина заключается в том, что общий код подмодуля является общим. Поскольку цель-C не поддерживает пространства имен, при добавлении платформ ADAL и MSAL в приложение будут существовать два экземпляра одного и того же класса. Не гарантируется, какой из них будет подобран во время выполнения. Если оба пакета SDK используют одну и ту же версию конфликтующего класса, приложение может продолжать работать. Однако если это другая версия, в приложении могут возникать непредвиденные сбои, которые трудно диагностировать.

Запуск ADAL и MSAL в одном рабочем приложении не поддерживается. Тем не менее, если вы просто тестируете и переносите пользователей из ADAL цели-C в MSAL для iOS и macOS, вы можете продолжить использовать приложение [ADAL цели-c 2.6.10](https://github.com/AzureAD/azure-activedirectory-library-for-objc/releases/tag/2.6.10). Это единственная версия, которая работает с MSAL в одном и том же приложении. Для этой версии ADAL не будет новых обновлений компонентов, поэтому она должна использоваться только для целей миграции и тестирования. Приложение не должно полагаться на сосуществование ADAL и MSAL в долгосрочной перспективе.

Сосуществование ADAL и MSAL в одном приложении не поддерживается.
Сосуществование ADAL и MSAL между несколькими приложениями полностью поддерживается.

## <a name="practical-migration-steps"></a>Практические действия по переносу

### <a name="app-registration-migration"></a>Миграция регистрации приложений

Вам не нужно изменять существующее приложение AAD для переключения на MSAL и включения учетных записей AAD. Однако если приложение на основе ADAL не поддерживает аутентификацию через посредника, необходимо зарегистрировать новый универсальный код ресурса (URI) перенаправления для приложения, прежде чем можно будет переключиться на MSAL.

URI перенаправления должен иметь следующий формат: `msauth.<app.bundle.id>://auth` . Замените `<app.bundle.id>` идентификатором пакета своего приложения. Укажите URI перенаправления в [портал Azure](https://aka.ms/MobileAppReg).

Только для iOS для поддержки проверки подлинности на основе сертификатов необходимо зарегистрировать дополнительный URI перенаправления в приложении и портал Azure в следующем формате: `msauth://code/<broker-redirect-uri-in-url-encoded-form>` . Например: `msauth://code/msauth.com.microsoft.mybundleId%3A%2F%2Fauth`

Мы рекомендуем, чтобы все приложения регистрировали оба URI перенаправления.

Если вы хотите добавить поддержку добавочного согласия, выберите интерфейсы API и разрешения, настроенные для запроса доступа к в регистрации приложения на вкладке **разрешения API** .

Если вы выполняете миграцию из ADAL и хотите поддерживать учетные записи AAD и MSA, необходимо обновить существующую регистрацию приложения для поддержки обоих. Мы не рекомендуем обновлять существующее рабочее приложение, чтобы обеспечить немедленное поддержку AAD и MSA. Вместо этого создайте другой идентификатор клиента, поддерживающий AAD и MSA для тестирования, и после проверки того, что все сценарии работают, обновите существующее приложение.

### <a name="add-msal-to-your-app"></a>Добавление MSAL в приложение

Пакет SDK для MSAL можно добавить в приложение с помощью предпочтительного средства управления пакетами. [Подробные инструкции](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki/Installation)см. здесь.

### <a name="update-your-apps-infoplist-file"></a>Обновление файла INFO. plist приложения

Для только iOS Добавьте схему URI перенаправления приложения в файл info. plist. Для приложений, совместимых с компонентом ADAL Broker, это уже должно быть. Схема URI перенаправления MSAL по умолчанию будет иметь формат: `msauth.<app.bundle.id>` .  

```xml
<key>CFBundleURLSchemes</key>
<array>
    <string>msauth.<app.bundle.id></string>
</array>
```

Добавьте следующие схемы в файле info. plist приложения в разделе `LSApplicationQueriesSchemes` .

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
     <string>msauthv2</string>
     <string>msauthv3</string>
</array>
```

### <a name="update-your-appdelegate-code"></a>Обновление кода AppDelegate

Для iOS добавьте в файл AppDelegate. m следующее:

Objective-C.

```objc
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options`
{
    return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
}
```

Swift:

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
}
```

**Если вы используете Xcode 11**, вместо этого следует поместить обратный вызов MSAL в `SceneDelegate` файл.
Если UISceneDelegate и UIApplicationDelegate поддерживаются для совместимости с более старыми версиями iOS, то обратный вызов MSAL потребуется поместить в оба файла.

Objective-C.

```objc
 - (void)scene:(UIScene *)scene openURLContexts:(NSSet<UIOpenURLContext *> *)URLContexts
 {
     UIOpenURLContext *context = URLContexts.anyObject;
     NSURL *url = context.URL;
     NSString *sourceApplication = context.options.sourceApplication;
     
     [MSALPublicClientApplication handleMSALResponse:url sourceApplication:sourceApplication];
 }
```

Swift:

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        
        guard let urlContext = URLContexts.first else {
            return
        }
        
        let url = urlContext.url
        let sourceApp = urlContext.options.sourceApplication
        
        MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApp)
    }
```

Это позволяет MSAL обработку ответов от брокера и веб-компонента.
Это не требовалось в ADAL, так как методы делегата приложения "свиззлед" были автоматически. Его Добавление вручную снижает вероятность ошибок и предоставляет приложению больше возможностей управления.

### <a name="enable-token-caching"></a>Включение кэширования маркеров

По умолчанию MSAL кэширует маркеры приложения в цепочку ключей iOS или macOS. 

Чтобы включить кэширования маркеров:
1. Убедитесь, что приложение подписано правильно
2. Перейдите к параметрам проекта Xcode, а затем выберите **Вкладка "Возможности"**  > **Включить общий доступ к цепочке ключей**.
3. Щелкните **+** и введите следующую запись **Групп цепочек ключей**. 3.a) для iOS введите `com.microsoft.adalcache` 3.б) для macOS введите `com.microsoft.identity.universalstorage`

### <a name="create-msalpublicclientapplication-and-switch-to-its-acquiretoken-and-acquiretokesilent-calls"></a>Создайте Мсалпубликклиентаппликатион и переключитесь на вызовы acquireToken и Аккуиретокесилент

Можно создать `MSALPublicClientApplication` с помощью следующего кода:

Objective-C.

```objc
NSError *error = nil;
MSALPublicClientApplicationConfig *configuration = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];
    
MSALPublicClientApplication *application =
[[MSALPublicClientApplication alloc] initWithConfiguration:configuration
                                                     error:&error];
```

Swift:

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
do {
  let application = try MSALPublicClientApplication(configuration: config)
  // continue on with application
            
} catch let error as NSError {
  // handle error here
}
```

Затем вызовите API управления учетными записями, чтобы узнать, есть ли в кэше какие-либо учетные записи:

Objective-C.

```objc
NSString *accountIdentifier = nil /*previously saved MSAL account identifier */;
NSError *error = nil;
MSALAccount *account = [application accountForIdentifier:accountIdentifier error:&error];
```

Swift:

```swift
// definitions that need to be initialized
let application: MSALPublicClientApplication!
let accountIdentifier: String! /*previously saved MSAL account identifier */

do {
  let account = try application.account(forIdentifier: accountIdentifier)
  // continue with account usage
} catch let error as NSError {
  // handle error here
}
```



или прочтите все учетные записи:

Objective-C.

```objc
NSError *error = nil;
NSArray<MSALAccount *> *accounts = [application allAccounts:&error];
```

Swift:

```swift
let application: MSALPublicClientApplication!
do {
  let accounts = try application.allAccounts()
  // continue with account usage
} catch let error as NSError {
  // handle error here
}
```



Если учетная запись найдена, вызовите `acquireTokenSilent` API MSAL:

Objective-C.

```objc
MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"<your-resource-here>/.default"] account:account];
    
[application acquireTokenSilentWithParameters:silentParameters
                              completionBlock:^(MSALResult *result, NSError *error)
{
    if (result)
    {
        NSString *accessToken = result.accessToken;
        // Use your token
    }
    else
    {
        // Check the error
        if ([error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
        {
            // Interactive auth will be required
        }
            
        // Other errors may require trying again later, or reporting authentication problems to the user
    }
}];
```

Swift:

```swift
let application: MSALPublicClientApplication!
let account: MSALAccount!
        
let silentParameters = MSALSilentTokenParameters(scopes: ["<your-resource-here>/.default"], 
                                                 account: account)
application.acquireTokenSilent(with: silentParameters) {
  (result: MSALResult?, error: Error?) in
  if let accessToken = result?.accessToken {
     // use accessToken
  }
  else {
    // Check the error
    guard let error = error else {
      assert(true, "callback should contain a valid result or error")
      return
    }
    
    let nsError = error as NSError
    if (nsError.domain == MSALErrorDomain
        && nsError.code == MSALError.interactionRequired.rawValue) {
      // Interactive auth will be required
    }
                
    // Other errors may require trying again later, or reporting authentication problems to the user
  }
}
```



## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в [Потоки проверки подлинности и сценарии приложений](authentication-flows-app-scenarios.md)
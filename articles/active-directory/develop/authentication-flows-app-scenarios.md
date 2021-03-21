---
title: Сценарии приложений и потоки для проверки подлинности на платформе удостоверений Майкрософт | Azure
description: Сведения о сценариях приложений для платформы Microsoft Identity, включая проверку подлинности удостоверений, получение маркеров и вызов защищенных интерфейсов API.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/03/2020
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: ffa52805a5e2680d534b2b24a210465cb3fc7cac
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100557850"
---
# <a name="authentication-flows-and-application-scenarios"></a>Потоки проверки подлинности и сценарии приложений

Платформа Microsoft Identity поддерживает проверку подлинности для различных типов современных архитектур приложений. Все эти архитектуры основаны на стандартных отраслевых протоколах [OAuth 2.0 и OpenID Connect](active-directory-v2-protocols.md). Используя [библиотеки проверки подлинности для платформы идентификации Майкрософт](reference-v2-libraries.md), приложения проверяют подлинность удостоверений и получают маркеры для доступа к защищенным API.

В этой статье описываются потоки проверки подлинности и сценарии применения приложений, используемые в.

## <a name="application-categories"></a>Категории приложений

Токены могут быть получены из нескольких типов приложений, включая:

- Веб-приложения
- Мобильные приложения
- Классические приложения
- Веб-API

Маркеры также могут быть получены приложениями, работающими на устройствах, у которых нет браузера или которые выполняются на Интернет вещей (IoT).

В следующих разделах описываются категории приложений.

### <a name="protected-resources-vs-client-applications"></a>Защищенные ресурсы или клиентские приложения

Сценарии проверки подлинности предполагают два действия:

- **Получение маркеров безопасности для защищенного веб-API**. рекомендуется использовать [библиотеку проверки подлинности Майкрософт (MSAL)](reference-v2-libraries.md), разработанную и поддерживаемую корпорацией Майкрософт.
- **Защита веб-API или веб-приложения**. Одна из проблем защиты этих ресурсов — проверка маркера безопасности. Для некоторых платформ корпорация Майкрософт предоставляет [библиотеки ПО промежуточного слоя](reference-v2-libraries.md).

### <a name="with-users-or-without-users"></a>С пользователями или без пользователей

Большинство сценариев проверки подлинности получают маркеры от имени пользователей, выполнивших вход.

![Сценарии взаимодействия с пользователями](media/scenarios/scenarios-with-users.svg)

Однако существуют также управляющие приложения. В этих сценариях приложения получают маркеры от имени самих пользователей.

![Сценарии с приложениями управляющей программы](media/scenarios/daemon-app.svg)

### <a name="single-page-public-client-and-confidential-client-applications"></a>Одностраничные приложения, общедоступные клиентские приложения или конфиденциальные клиентские приложения

Маркеры безопасности могут быть получены несколькими типами приложений. Эти приложения, как правило, делятся на следующие три категории. Каждая из них используется с разными библиотеками и объектами.

- **Одностраничные приложения**: также известные как одностраничные приложения, это веб-приложения, в которых маркеры получаются с помощью приложения JavaScript или TypeScript, работающего в браузере. Многие современные приложения имеют одностраничное приложение на внешнем интерфейсе, которое в основном написано на JavaScript. Для таких приложений часто применяются платформы Angular, React, Vue и др. MSAL.js — это единственная библиотека проверки подлинности Майкрософт, которая поддерживает одностраничные приложения.

- **Общедоступные клиентские приложения**: приложения в этой категории, такие как следующие типы, всегда входят в систему пользователей:
  - Классические приложения, вызывающие веб-API от имени пользователей, выполнивших вход
  - Мобильные приложения
  - Приложения, работающие на устройствах без браузера, как и в Интернете вещей

- **Конфиденциальные клиентские приложения**: приложения в этой категории включают:
  - Веб-приложения, вызывающие веб-API
  - Веб-API, вызывающие веб-API
  - приложения управляющей программы, даже если они реализованы как консольная служба (например, управляющая программа в Linux или служба Windows).

### <a name="sign-in-audience"></a>Аудитория для входа

Доступные потоки проверки подлинности зависят от аудитории входа. Некоторые потоки доступны только для рабочих или учебных учетных записей. Другие доступны как для рабочих, так и для учебных учетных записей, а также для личных учетных записей Майкрософт.

Дополнительные сведения см. в разделе [Поддерживаемые типы учетных записей](v2-supported-account-types.md#account-type-support-in-authentication-flows).

## <a name="application-scenarios"></a>Сценарии приложений

Платформа Microsoft Identity поддерживает проверку подлинности для этих архитектур приложений:

- Одностраничные приложения
- Веб-приложения
- Веб-API
- Мобильные приложения
- Собственные приложения
- Управляющие программы
- Приложения серверной части

Приложения используют разные потоки проверки подлинности для входа пользователей и получения маркеров для вызова защищенных API.

### <a name="single-page-application"></a>Одностраничное приложение

Многие современные веб-приложения создаются как одностраничные приложения на стороне клиента. Эти приложения используют JavaScript или платформу, такую как угловой, Vue и реагируют на них. Эти приложения выполняются в веб-браузере.

Одностраничные приложения отличаются от традиционных веб-приложений на стороне сервера с точки зрения характеристик проверки подлинности. Использование платформы удостоверений Майкрософт позволяет одностраничным приложениям обрабатывать вход пользователей и получать маркеры для доступа к серверным службам или веб-API. Платформа Microsoft Identity предлагает два типа предоставления для приложений JavaScript:

| MSAL.js (2. x) | MSAL.js (1. x) |
|---|---|
| ![Проверка подлинности одностраничного приложения](media/scenarios/spa-app-auth.svg) | ![Одностраничное приложение неявным образом](media/scenarios/spa-app.svg) |

### <a name="web-app-that-signs-in-a-user"></a>Веб-приложение, которое подписывает пользователя

![Веб-приложение, которое обрабатывает вход пользователя](media/scenarios/scenario-webapp-signs-in-users.svg)

Чтобы защитить веб-приложение, которое подписывает пользователя, выполните следующие действия.

- При разработке в .NET используется ASP.NET или ASP.NET Core с по промежуточного слоя ASP.NET OpenID Connect Connect. Защита ресурса включает проверку маркера безопасности, который выполняется [расширениями IdentityModel для библиотек .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki) и не MSAL.

- При разработке в Node.js используется [узел MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) или [Passport.js](https://github.com/AzureAD/passport-azure-ad).

См. сведения см. о [веб-приложениях, которые обрабатывают вход пользователей](scenario-web-app-sign-user-overview.md).

### <a name="web-app-that-signs-in-a-user-and-calls-a-web-api-on-behalf-of-the-user"></a>Веб-приложение, которое подписывает пользователя и вызывает веб-API от имени пользователя.

![Веб-приложение, которое вызывает веб-API](media/scenarios/web-app.svg)

Чтобы вызвать веб-API из веб-приложения от имени пользователя, используйте поток кода авторизации и сохраните полученные токены в кэше маркеров. MSAL обновляет маркеры по мере необходимости, а контроллер прозрачно получает маркеры из кэша.

Дополнительные сведения см. в разделе [веб-приложение, вызывающее веб-API](scenario-web-app-call-api-overview.md).

### <a name="desktop-app-that-calls-a-web-api-on-behalf-of-a-signed-in-user"></a>Классическое приложение, вызывающее веб-API от имени пользователя, выполнившего вход

Чтобы классическое приложение вызывало веб-API, который выполняет вход пользователей, используйте интерактивные методы приобретения токена MSAL. Эти интерактивные методы позволяют управлять интерфейсом входа пользователя в систему. Чтобы реализовать такое взаимодействие, MSAL использует веб-браузер.

![Классическое приложение, которое вызывает веб-API](media/scenarios/desktop-app.svg)

Для приложений, выполняющихся на компьютерах Windows, присоединенных к домену Windows или к Azure Active Directory, есть еще одна возможность. Эти приложения могут автоматически получать маркер, используя [встроенную проверку подлинности Windows](https://aka.ms/msal-net-iwa).

Приложения на устройстве без браузера также могут вызывать API от имени пользователя. Для проверки подлинности пользователю нужно войти на другое устройство, где есть веб-браузер. В этом сценарии требуется использовать [поток кода устройства](https://aka.ms/msal-net-device-code-flow).

![Поток кода устройства](media/scenarios/device-code-flow-app.svg)

Хотя мы не рекомендуем использовать его, [поток имени пользователя и пароля](scenario-desktop-acquire-token.md#username-and-password) доступен в общедоступных клиентских приложениях. Этот поток пока еще необходим для некоторых сценариев, например при использовании DevOps.

Использование потока имени пользователя и пароля ограничивает ваши приложения. Например, приложения не могут войти в систему пользователя, который должен использовать многофакторную проверку подлинности или средство условного доступа в Azure AD. Также приложение не сможет использовать механизм единого входа. Проверка подлинности с помощью потока имени пользователя и пароля ведет к принципам современной проверки подлинности и предоставляется только для устаревших целей.

В классических приложениях, если требуется сохранять кэш маркеров, можно настроить [сериализацию кэша маркеров](scenario-desktop-acquire-token.md#file-based-token-cache). Реализуя [сериализацию двойного кэша](scenario-desktop-acquire-token.md#dual-token-cache-serialization-msal-unified-cache--adal-v3), можно использовать обратные и совместимые с ними кэши маркеров. Эти маркеры поддерживают предыдущие поколения библиотек проверки подлинности. Сюда относятся, среди прочего, библиотеки аутентификации Azure AD для .NET (ADAL.NET) версий 3 и 4.

См. сведения о [классическом приложении, которое вызывает веб-API](scenario-desktop-overview.md).

### <a name="mobile-app-that-calls-a-web-api-on-behalf-of-an-interactive-user"></a>Мобильное приложение, вызывающее веб-API от имени интерактивного пользователя

Как и для классического приложения, мобильное приложение вызывает методы интерактивного приобретения токена MSAL, чтобы получить маркер для вызова веб-API.

![Мобильное приложение, которое вызывает веб-API](media/scenarios/mobile-app.svg)

MSAL для iOS и MSAL для Android по умолчанию используют системный веб-браузер. Однако можно направить их на использование встроенного веб-представления. Существуют особенности, зависящие от мобильной платформы: универсальная платформа Windows (UWP), iOS или Android.

В некоторых сценариях, например при использовании условного доступа с привязкой к идентификатору устройства или регистрации устройства, требуется установить на устройство брокер. К таким брокерам относятся корпоративный портал Майкрософт (на Android) и Microsoft Authenticator (Android и iOS). MSAL теперь может взаимодействовать с брокерами. Дополнительные сведения о брокерах см. [в статье использование брокеров в Android и iOS](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/leveraging-brokers-on-Android-and-iOS).

См. сведения о [мобильных приложениях, которые вызывают веб-API](scenario-mobile-overview.md).

> [!NOTE]
> Мобильное приложение, использующее MSAL. iOS, MSAL. Для Android или MSAL.NET в Xamarin могут применяться политики защиты приложений. Например, политика может запретить пользователю копировать защищенный текст. Мобильное приложение управляется Intune и распознается Intune как управляемое приложение. Дополнительные сведения см. в разделе [Обзор пакета SDK для Microsoft Intune приложений](/intune/app-sdk).
>
> [Пакет SDK для приложений в Intune](/intune/app-sdk-get-started) отделен от библиотек MSAL и самостоятельно взаимодействует с AAD.

### <a name="protected-web-api"></a>Защищенный веб-API

Конечную точку платформы Microsoft Identity можно использовать для защиты веб-служб, таких как веб-API RESTFUL вашего приложения. Защищенный веб-API вызывается через маркер доступа. Маркер помогает защитить данные API и пройти проверку подлинности входящих запросов. Объект, вызывающий такой веб-API, добавляет маркер доступа в заголовок авторизации в HTTP-запросе.

Если вы хотите защитить веб-API ASP.NET или ASP.NET Core, проверьте маркер доступа. Для такой проверки применяется ПО промежуточного слоя JWT для ASP.NET. Проверка выполняется с помощью [расширений IdentityModel для библиотеки .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki), а не через MSAL.NET.

См. сведения о [защищенных веб-API](scenario-protected-web-api-overview.md).

### <a name="web-api-that-calls-another-web-api-on-behalf-of-a-user"></a>Веб-API, который вызывает другой веб-API от имени пользователя

Чтобы защищенный веб-API вызывал другой веб-API от имени пользователя, приложение должно получить маркер для подчиненного веб-API. Такие вызовы иногда называются вызовами *между* службами. Веб-API, которые вызывают другие веб-API, должны обеспечивать настраиваемую сериализацию кэша.

![Веб-API, который вызывает другой веб-API](media/scenarios/web-api.svg)

См. сведения о [веб-API, которые вызывают веб-API](scenario-web-api-call-api-overview.md).

### <a name="daemon-app-that-calls-a-web-api-in-the-daemons-name"></a>Управляющее приложение, вызывающее веб-API в имени управляющей программы

Приложениям, которые выполняют долговременные процессы или работают без взаимодействия с пользователем, также нужен метод доступа к защищенным веб-API. Такое приложение может проверять подлинность и получать маркеры с помощью удостоверения приложения. Приложение подтверждает свою идентификацию с помощью секрета или сертификата клиента.

Вы можете написать такие управляющие приложения, которые получают маркер для вызывающего приложения, используя методы получения [учетных данных клиента](scenario-daemon-acquire-token.md#acquiretokenforclient-api) в MSAL. Для этих методов требуется секрет клиента, добавляемый в регистрацию приложения в Azure AD. Затем приложение передает этот секрет управляющей программе, которую оно вызывает. Примеры таких секретов: пароли приложений, утверждение сертификата и утверждение клиента.

![Приложение управляющей программы, к которому обращаются другие приложения и API](media/scenarios/daemon-app.svg)

См. сведения об [управляющих приложениях, которые вызывают веб-API](scenario-daemon-overview.md).

## <a name="scenarios-and-supported-authentication-flows"></a>Сценарии и поддерживаемые потоки проверки подлинности

Потоки проверки подлинности используются для реализации сценариев приложений, запрашивающих маркеры. Между сценариями приложений и потоками проверки подлинности не существует строгого соответствия.

Сценарии, использующие получение маркеров, также сопоставляются с потоками проверки подлинности OAuth 2,0. Дополнительные сведения см. в разделе [протоколы OAuth 2,0 и OpenID Connect Connect на платформе Microsoft Identity](active-directory-v2-protocols.md).

<table>
 <thead>
  <tr><th>Сценарий</th> <th>Подробный разбор сценария</th> <th>Поток и предоставление разрешения OAuth 2.0</th> <th>Аудитория</th></tr>
 </thead>
 <tbody>
  <tr>
   <td><a href="scenario-spa-overview.md"><img alt="Single-Page App with Auth code" src="media/scenarios/spa-app-auth.svg"></a></td>
   <td><a href="scenario-spa-overview.md">Одностраничное приложение</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Код авторизации</a> с PKCE</td>
   <td>Рабочие или учебные учетные записи, личные учетные записи и Azure Active Directory B2C (Azure AD B2C)</td>
 </tr>

  <tr>
   <td><a href="scenario-spa-overview.md"><img alt="Single-Page App with Implicit" src="media/scenarios/spa-app.svg"></a></td>
   <td><a href="scenario-spa-overview.md">Одностраничное приложение</a></td>
   <td><a href="v2-oauth2-implicit-grant-flow.md">Неявно</a></td>
   <td>Рабочие или учебные учетные записи, личные учетные записи и Azure Active Directory B2C (Azure AD B2C)</td>
 </tr>

  <tr>
   <td><a href="scenario-web-app-sign-user-overview.md"><img alt="Web app that signs in users" src="media/scenarios/scenario-webapp-signs-in-users.svg"></a></td>
   <td><a href="scenario-web-app-sign-user-overview.md">Веб-приложение, которое поддерживает вход пользователей</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Код авторизации</a></td>
   <td>Рабочие или учебные учетные записи, личные учетные записи и Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-web-app-call-api-overview.md"><img alt="Web app that calls web APIs" src="media/scenarios/web-app.svg"></a></td>
   <td><a href="scenario-web-app-call-api-overview.md">Веб-приложение, которое вызывает веб-API</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Код авторизации</a></td>
   <td>Рабочие или учебные учетные записи, личные учетные записи и Azure AD B2C</td>
 </tr>

  <tr>
   <td rowspan="3"><a href="scenario-desktop-overview.md"><img alt=Desktop app that calls web APIs" src="media/scenarios/desktop-app.svg"></a></td>
   <td rowspan="4"><a href="scenario-desktop-overview.md">Классическое приложение, которое вызывает веб-API</a></td>
   <td>Интерактивное использование <a href="v2-oauth2-auth-code-flow.md">кода авторизации</a> с PKCE</td>
   <td>Рабочие или учебные учетные записи, личные учетные записи и Azure AD B2C</td>
 </tr>

  <tr>
   <td>Встроенная проверка подлинности Windows</td>
   <td>Рабочие или учебные учетные записи</td>
 </tr>

  <tr>
   <td><a href="v2-oauth-ropc.md">Пароль владельца ресурса</a></td>
   <td>Рабочие или учебные учетные записи и Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-desktop-acquire-token.md#command-line-tool-without-a-web-browser"><img alt="Browserless application" src="media/scenarios/device-code-flow-app.svg"></a></td>
   <td><a href="v2-oauth2-device-code.md">Код устройства</a></td>
   <td>Рабочие или учебные учетные записи, личные учетные записи и Azure AD B2C</td>
 </tr>

 <tr>
   <td rowspan="2"><a href="scenario-mobile-overview.md"><img alt="Mobile app that calls web APIs" src="media/scenarios/mobile-app.svg"></a></td>
   <td rowspan="2"><a href="scenario-mobile-overview.md">Мобильное приложение, которое вызывает веб-API</a></td>
   <td>Интерактивное использование <a href="v2-oauth2-auth-code-flow.md">кода авторизации</a> с PKCE</td>
   <td>Рабочие или учебные учетные записи, личные учетные записи и Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="v2-oauth-ropc.md">Пароль владельца ресурса</a></td>
   <td>Рабочие или учебные учетные записи и Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-daemon-overview.md"><img alt="Daemon app that calls web APIs" src="media/scenarios/daemon-app.svg"></a></td>
   <td><a href="scenario-daemon-overview.md">Управляющее приложение, вызывающее веб-API</a></td>
   <td><a href="v2-oauth2-client-creds-grant-flow.md">Учетные данные клиента</a></td>
   <td>Разрешения только для приложений, не имеющие пользователя и используемые только в организациях Azure AD</td>
 </tr>

  <tr>
   <td><a href="scenario-web-api-call-api-overview.md"><img alt="Web API that calls web APIs" src="media/scenarios/web-api.svg"></a></td>
   <td><a href="scenario-web-api-call-api-overview.md">Веб-API, которые вызывают веб-API</a></td>
   <td><a href="v2-oauth2-on-behalf-of-flow.md">Вызов от имени</a></td>
   <td>Рабочие или учебные учетные записи и личные учетные записи</td>
 </tr>

 </tbody>
</table>

## <a name="scenarios-and-supported-platforms-and-languages"></a>Сценарии, поддерживаемые платформы и языки

Библиотеки проверки подлинности Майкрософт поддерживают несколько платформ:

- .NET Core
- .NET Framework
- Java
- JavaScript
- macOS
- Нативные приложения для Android
- нативные приложения для iOS;
- Node.js
- Python
- Windows 10/UWP
- Xamarin.iOS
- Xamarin.Android

Для создания приложений также можно использовать различные языки.

> [!NOTE]
> Некоторые типы приложений недоступны на каждой платформе.

В приведенной ниже таблице в столбце Windows каждое упоминание .NET Core подразумевает и платформу .NET Framework. Мы опустили эти упоминания, чтобы не загромождать таблицу.

|Сценарий  | Windows | Linux | Mac | iOS | Android
|--|--|--|--|--|--|--|
| [Одностраничное приложение](scenario-spa-overview.md) <br/>[![Проверка подлинности одностраничного приложения](media/scenarios/spa-app-auth.svg)](scenario-spa-overview.md) | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png) MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js
| [Одностраничное приложение](scenario-spa-overview.md) <br/>[![Неявное одностраничное приложение](media/scenarios/spa-app.svg)](scenario-spa-overview.md) | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png) MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js
| [Веб-приложение, которое поддерживает вход пользователей](scenario-web-app-sign-user-overview.md) <br/>[![Веб-приложение, которое входит в систему пользователей](media/scenarios/scenario-webapp-signs-in-users.svg)](scenario-web-app-sign-user-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>
| [Веб-приложение, которое вызывает веб-API](scenario-web-app-call-api-overview.md) <br/> <br/>[![Веб-приложение, которое вызывает веб-API](media/scenarios/web-app.svg)](scenario-web-app-call-api-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png) <br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>Flask + MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>Flask + MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/> ![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>Flask + MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>
| [Классическое приложение, которое вызывает веб-API](scenario-desktop-overview.md) <br/> <br/>[ ![ Классическое приложение, вызывающее веб-API](media/scenarios/desktop-app.svg)](scenario-desktop-overview.md) ![ Поток кода устройства](media/scenarios/device-code-flow-app.svg) | ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/> ![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python <br/> ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/> ![iOS, Objective-C или swift](media/sample-v2-code/small_logo_iOS.png) MSAL.objc |
| [Мобильное приложение, которое вызывает веб-API](scenario-mobile-overview.md) <br/> [![Мобильное приложение, которое вызывает веб-API](media/scenarios/mobile-app.svg)](scenario-mobile-overview.md) | ![UWP](media/sample-v2-code/small_logo_windows.png) MSAL.NET ![Xamarin](media/sample-v2-code/small_logo_xamarin.png) MSAL.NET | | | ![iOS, Objective-C или swift](media/sample-v2-code/small_logo_iOS.png) MSAL.objc | ![Android](media/sample-v2-code/small_logo_Android.png) MSAL.Android
| [Управляющая программа](scenario-daemon-overview.md) <br/> [![Управляющая программа](media/scenarios/daemon-app.svg)](scenario-daemon-overview.md) | ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png) MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>
| [Веб-API, которые вызывают веб-API](scenario-web-api-call-api-overview.md) <br/><br/> [![Веб-API, которые вызывают веб-API](media/scenarios/web-api.svg)](scenario-web-api-call-api-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python ![MSAL Node](media/sample-v2-code/small-logo-nodejs.png) <br/>MSAL Node<br/>

Дополнительные сведения см. в [статье библиотеки проверки подлинности платформ Microsoft Identity](reference-v2-libraries.md).

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте больше об [основах проверки подлинности](./authentication-vs-authorization.md) и [маркерах доступа на платформе Microsoft Identity](access-tokens.md).
* Дополнительные сведения о [защите доступа к приложениям IOT](/azure/architecture/example-scenario/iot-aad/iot-aad).

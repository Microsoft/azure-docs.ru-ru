---
title: Перенос одностраничного приложения JavaScript из неявного потока предоставления разрешения в поток кода авторизации | Azure
titleSuffix: Microsoft identity platform
description: Обновление одностраничного приложения JavaScript, использующего библиотеку MSAL.js 1.x и поток неявного предоставления разрешения, до MSAL.js 2.x и потока кода авторизации с поддержкой PKCE и CORS.
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/17/2020
ms.author: hahamil
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: 3c11334fe1b4d77be6e64febfc1d3de6efa302c3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100365946"
---
# <a name="migrate-a-javascript-single-page-app-from-implicit-grant-to-auth-code-flow"></a>Перенос одностраничного приложения JavaScript из неявного потока предоставления разрешения в поток кода авторизации

Библиотека проверки подлинности Microsoft для JavaScript (MSAL.js) версии 2.0 обеспечивает поддержку потока кода авторизации с PKCE и CORS для одностраничных приложений на платформе Microsoft Identity. Выполните действия, описанные в следующих разделах, чтобы перенести приложение с MSAL.js 1.x, использующее неявный поток предоставления разрешения, на MSAL.js версии 2.0 или выше (далее *2.x*) и поток кода авторизации.

MSAL.js 2.x — это расширенная версия MSAL.js 1.x, которая поддерживает поток кода авторизации в браузере вместо потока неявного предоставления разрешения. MSAL.js 2.x **НЕ** поддерживает неявный поток.

## <a name="migration-steps"></a>Этапы миграции

Чтобы перевести приложение на использование MSAL.js 2.x и потока кода авторизации, нужно выполнить три основных шага:

1. Переключить URI перенаправления для [регистрации приложения](#switch-redirect-uris-to-spa-platform) с **веб-** платформы на платформу **одностраничного приложения**.
1. Обновить [код](#switch-redirect-uris-to-spa-platform), чтобы вместо MSAL.js 1.x в нем использовалась версия **2.x**.
1. Отключить [неявное предоставление разрешения](#disable-implicit-grant-settings) в регистрации приложения, когда все приложения, использующие эту регистрацию, будут перенесены на MSAL.js 2.x и поток кода авторизации.

Далее подробно описан каждый из этих шагов.

## <a name="switch-redirect-uris-to-spa-platform"></a>Переключение URI перенаправления на платформу одностраничного приложения

Если вы хотите использовать существующую регистрацию приложения, переключите URI перенаправления на платформу одностраничного приложения на портале Azure. Это позволит активировать поток кода авторизации с поддержкой PKCE и CORS для приложений, использующих регистрацию (но вам по-прежнему необходимо обновить код приложения, задав в нем использование MSAL.js 2.x).

Выполните следующие действия для регистраций приложений, у которых URI перенаправления сейчас настроены для **веб-** платформы.

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a> и выберите свой клиент **Azure Active Directory**.
1. В разделе **Регистрация приложений** выберите свое приложение и щелкните элемент **Проверка подлинности**.
1. На плитке **веб-** платформы в разделе **URI перенаправления** выберите предупреждающий баннер (он указывает на необходимость переноса URI).

    :::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-01-implicit-warning-banner.png" alt-text="Баннер, предупреждающий об использовании неявного потока, на плитке веб-приложения на портале Azure":::
1. Выберите *только* URI, связанные с приложениями которые будут использовать MSAL.js 2.x, и щелкните **Настроить**.

    :::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-02-select-redirect-uri.png" alt-text="Выбор панели URI перенаправления на панели одностраничного приложения на портале Azure":::

Эти URI перенаправления должны появиться на плитке платформы **одностраничного приложения**, сообщая о том, что для них теперь включена поддержка CORS с потоком кода авторизации и PKCE.

:::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-03-spa-redirect-uri-tile.png" alt-text="Плитка одностраничного приложения в регистрации приложения на портале Azure":::

Можно также [создать новую регистрацию приложения](scenario-spa-app-registration.md), а не обновлять URI перенаправления в существующей регистрации.

## <a name="update-your-code-to-msaljs-2x"></a>Обновление кода для использования MSAL.js 2.x

При использовании MSAL 1.x экземпляр приложения создается путем инициализации [UserAgentApplication][msal-js-useragentapplication]:

```javascript
// MSAL 1.x
import * as msal from "msal";

const msalInstance = new msal.UserAgentApplication(config);
```

Для MSAL 2.x нужно вместо этого инициализировать [PublicClientApplication][msal-js-publicclientapplication]:

```javascript
// MSAL 2.x
import * as msal from "@azure/msal-browser";

const msalInstance = new msal.PublicClientApplication(config);
```

Полное пошаговое руководство по добавлению MSAL 2.x в приложение см. в [Руководстве по входу пользователей и вызову API Microsoft Graph из одностраничного приложения JavaScript с помощью потока кода авторизации](tutorial-v2-javascript-auth-code.md).

Дополнительные сведения об изменениях, которые могут потребоваться в коде, см. в [руководстве по миграции](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/v1-migration.md) на сайте GitHub.

## <a name="disable-implicit-grant-settings"></a>Отключение параметров неявного предоставления разрешений

Когда вы перенесете все рабочие приложения, использующие эту регистрацию приложения и соответствующий идентификатор клиента, на MSAL 2.x и поток кода авторизации, снимите флажки параметров неявного предоставления разрешения в регистрации приложения.

В результате неявный поток будет отключен для всех приложений, использующих эту регистрацию и идентификатор клиента.

**Не** отключайте неявный поток предоставления разрешений, пока не обновите все приложения, настроив их на использование MSAL.js 2.x и [PublicClientApplication][msal-js-publicclientapplication].

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о потоке кода авторизации, включая различия между неявными потоками кода и проверки подлинности, см. в статье [поток кода авторизации для платформы Microsoft Identity и OAuth 2,0](v2-oauth2-auth-code-flow.md).

Если вы хотите узнать больше о разработке одностраничных приложений JavaScript на платформе удостоверений Майкрософт, см. серию статей [Сценарий: одностраничное приложение](scenario-spa-overview.md).

<!-- LINKS - external -->
[msal-js-useragentapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal.useragentapplication.html
[msal-js-publicclientapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal_browser.publicclientapplication.html

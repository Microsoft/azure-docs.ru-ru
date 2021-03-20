---
title: Получение сертификата AppSource для Azure Active Directory | Документация Майкрософт
description: Сведения о получении сертификата AppSource приложения для Azure Active Directory.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.topic: how-to
ms.workload: identity
ms.date: 08/21/2018
ms.author: ryanwi
ms.reviewer: jeedes
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: d9a4da6fe65fda07609c7399518fa324017ea44c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101649351"
---
# <a name="how-to-get-appsource-certified-for-azure-active-directory"></a>Как получить сертификат AppSource для Azure Active Directory

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

С помощью [Microsoft AppSource](https://appsource.microsoft.com/) бизнес-пользователи могут обнаруживать бизнес-приложения SaaS (автономные приложения SaaS и дополнения к имеющимся продуктам SaaS Microsoft), использовать их и управлять ими.

Чтобы отобразить список автономных приложений SaaS на AppSource, приложение должно принять единый вход из рабочих учетных записей любой компании или организации с Azure Active Directory (Azure AD). В процессе входа необходимо использовать протоколы [OpenID Connect](v1-protocols-openid-connect-code.md) или [OAuth 2.0](v1-protocols-oauth-code.md). Для сертификации AppSource интеграции SAML не принимается.

## <a name="guides-and-code-samples"></a>Руководства и примеры кода

Если вы хотите узнать о том, как интегрировать приложение с Azure AD с помощью Open ID Connect, следуйте нашим руководствам и примерам кода в [Руководстве разработчика Azure Active Directory](v1-overview.md#get-started "Приступая к работе с Azure AD для разработчиков").

## <a name="multi-tenant-applications"></a>Мультитенантные приложения

Приложение, принимающее вход пользователя из любой компании или организации с Azure AD, не требуя отдельного экземпляра, конфигурации или развертывания, называется *мультитенантным*. AppSource рекомендует реализовать поддержку мультитенантности в приложениях, чтобы добавить бесплатную пробную версию возможности входа *одним щелчком*.

Чтобы включить мультитенантность в приложении, сделайте следующее.
1. При введении регистрационной информации приложения на [портале Azure](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) задайте для свойства `Multi-Tenanted` значение `Yes`. По умолчанию все приложения, созданные на портале Azure, настроены как *[приложения с одним клиентом](#single-tenant-applications)*.
1. Обновите код для отправки запросов в конечную точку `common`. Чтобы сделать это, обновите конечную точку из `https://login.microsoftonline.com/{yourtenant}` в `https://login.microsoftonline.com/common*`.
1. Для некоторых платформ, например ASP .NET, необходимо также обновить код, чтобы принимать несколько издателей.

Дополнительные сведения о мультитенантности см. в статье [Как реализовать вход любого пользователя Azure Active Directory с помощью шаблона мультитенантного приложения](../develop/howto-convert-app-to-be-multi-tenant.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

### <a name="single-tenant-applications"></a>Приложения с одним клиентом

*Приложения с одним клиентом* являются приложениями, которые принимают вход пользователей только из определенного экземпляра Azure AD. Внешние пользователи (включая рабочие или учебные учетные записи из других организаций или личные учетные записи) могут войти в приложение с одним клиентом после добавления каждого пользователя в качестве гостевой учетной записи в экземпляр Azure AD, в котором регистрируется приложение. 

В Azure AD пользователей можно добавлять в качестве гостевых учетных записей через [службу совместной работы Azure AD B2B](../external-identities/what-is-b2b.md). Это можно сделать [программным образом](../../active-directory-b2c/code-samples.md). При использовании B2B пользователи могут создать портал самообслуживания, для входа на которой не требуется приглашение. Дополнительные сведения см. в статье [Самостоятельная регистрация на портале для службы совместной работы Azure AD B2B](../external-identities/self-service-portal.md).

В приложениях с одним клиентом можно включить возможность *Свяжитесь со мной*. Однако если требуется включить возможность входа одним щелчком или пробную версию возможностей, рекомендуемые AppSource, то включите в приложении мультитенантность.

## <a name="appsource-trial-experiences"></a>Пробные версии возможностей AppSource

### <a name="free-trial-customer-led-trial-experience"></a>Бесплатная пробная версия возможностей (клиентская)

Клиентская пробная версия возможностей рекомендуется AppSource, так как она позволяет воспользоваться доступом к приложению в один щелчок. В следующем примере показано, как выглядит этот интерфейс:

<table >
<tr>    <td valign="top" width="33%">1.<br/><img src="media/active-directory-devhowto-appsource-certified/customer-led-trial-step1.png" width="85%" alt="Shows Free trial for customer-led trial experience."/><ul><li>Пользователь находит ваше приложение на веб-сайте AppSource.</li><li>Выбор параметра "Бесплатная пробная версия"</li></ul></td>
    <td valign="top" width="33%">2.<br/><img src="media/active-directory-devhowto-appsource-certified/customer-led-trial-step2.png" width="85%" alt="Shows how user is redirected to a URL in your web site."/><ul><li>AppSource перенаправляет пользователя по URL-адресу вашего веб-сайта.</li><li>На вашем веб-сайте автоматически запускается <i>единый вход</i> (при загрузке страницы).</li></ul></td>
    <td valign="top" width="33%">3.<br/><img src="media/active-directory-devhowto-appsource-certified/customer-led-trial-step3.png" width="85%" alt="Shows the Microsoft sign-in page."/><ul><li>Пользователь перенаправляется на страницу входа Майкрософт.</li><li>Пользователь указывает учетные данные для входа.</li></ul></td>
</tr>
<tr>
    <td valign="top" width="33%">4.<br/><img src="media/active-directory-devhowto-appsource-certified/customer-led-trial-step4.png" width="85%" alt="Example: Consent page for an application."/><ul><li>Пользователь дает согласие для приложения.</li></ul></td>
    <td valign="top" width="33%">5.<br/><img src="media/active-directory-devhowto-appsource-certified/customer-led-trial-step5.png" width="85%" alt="Shows the experience the user sees when redirected back to your site."/><ul><li>Вход выполняется и пользователь перенаправляется на ваш веб-сайт.</li><li>Пользователь запускает бесплатную пробную версию.</li></ul></td>
    <td></td>
</tr>
</table>

### <a name="contact-me-partner-led-trial-experience"></a>Функция "Связаться со мной" (партнерская пробная версия)

Вы можете использовать партнерскую пробную версию для операций, выполняемых вручную, или длительных операций для подготовки пользователя или компании, например, когда приложению необходимо подготовить виртуальные машины, экземпляры базы данных или операции, которые занимают много времени. В этом случае, когда пользователь нажмет кнопку **Request Trial** (Запросить пробную версию) и заполнит форму, AppSource отправит вам его контактные данные. При получении этих сведений вы подготавливаете среду и отправляете пользователю инструкции по получению доступа к пробной версии.<br/><br/>

<table valign="top">
<tr>
    <td valign="top" width="33%">1.<br/><img src="media/active-directory-devhowto-appsource-certified/partner-led-trial-step1.png" width="85%" alt="Shows Contact me for partner-led trial experience"/><ul><li>Пользователь находит ваше приложение на веб-сайте AppSource.</li><li>Выбор параметра "связаться со мной"</li></ul></td>
    <td valign="top" width="33%">2.<br/><img src="media/active-directory-devhowto-appsource-certified/partner-led-trial-step2.png" width="85%" alt="Shows an example form with contact info"/><ul><li>Заполняет форму контактными данными.</li></ul></td>
     <td valign="top" width="33%">3.<br/><br/>
        <table bgcolor="#f7f7f7">
        <tr>
            <td><img src="media/active-directory-devhowto-appsource-certified/usercontact.png" width="55%" alt="Shows placeholder for user information"/></td>
            <td>Вы получаете сведения о пользователе.</td>
        </tr>
        <tr>
            <td><img src="media/active-directory-devhowto-appsource-certified/setupenv.png" width="55%" alt="Shows placeholder for setup environment info"/></td>
            <td>Настраиваете среду.</td>
        </tr>
        <tr>
            <td><img src="media/active-directory-devhowto-appsource-certified/contactcustomer.png" width="55%" alt="Shows placeholder for trial info"/></td>
            <td>Связываетесь с пользователем по поводу информации о пробной версии.</td>
        </tr>
        </table><br/><br/>
        <ul><li>Вы получаете сведения пользователя и настраиваете пробный экземпляр.</li><li>Вы отправляете пользователю гиперссылку на получение доступа к приложению.</li></ul>
    </td>
</tr>
<tr>
    <td valign="top" width="33%">4.<br/><img src="media/active-directory-devhowto-appsource-certified/partner-led-trial-step3.png" width="85%" alt="Shows the application sign-in screen"/><ul><li>Пользователь получает доступ к приложению и завершает процесс единого входа.</li></ul></td>
    <td valign="top" width="33%">5.<br/><img src="media/active-directory-devhowto-appsource-certified/partner-led-trial-step4.png" width="85%" alt="Shows an example consent page for an application"/><ul><li>Пользователь дает согласие для приложения.</li></ul></td>
    <td valign="top" width="33%">6.<br/><img src="media/active-directory-devhowto-appsource-certified/customer-led-trial-step5.png" width="85%" alt="Shows the experience the user sees when redirected back to your site"/><ul><li>Вход выполняется и пользователь перенаправляется на ваш веб-сайт.</li><li>Пользователь запускает бесплатную пробную версию.</li></ul></td>  
</tr>
</table>

### <a name="more-information"></a>Дополнительные сведения

Дополнительные сведения о пробной версии AppSource см. [в этом видео](https://aka.ms/trialexperienceforwebapps). 

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о разработке приложений, поддерживающих вход с помощью Azure AD, см. в статье [Сценарии проверки подлинности в Azure AD](./v1-authentication-scenarios.md).
- Сведения о выводе списка приложений SaaS в AppSource см. на [странице сведений для партнеров AppSource](https://appsource.microsoft.com/partners).

## <a name="get-support"></a>Получение поддержки

Для интеграции Azure AD мы используем [Microsoft Q&A](/answers/products/) с сообществом для предоставления поддержки.

Мы настоятельно рекомендуем задать свои вопросы в Microsoft Q&и просмотреть существующие проблемы, чтобы узнать, не запрашивал ли пользователь вопрос. Убедитесь, что ваши вопросы или комментарии помечены как [`[azure-active-directory]`](/answers/topics/azure-active-directory.html) .

Оставляйте свои замечания и пожелания в разделе ниже. Они помогают нам улучшать содержимое веб-сайта.

<!--Reference style links -->
[AAD-Auth-Scenarios]:v1-authentication-scenarios.md
[AAD-Auth-Scenarios-Browser-To-WebApp]:v1-authentication-scenarios.md#web-browser-to-web-application
[AAD-Dev-Guide]: v1-overview.md
[AAD-Howto-Multitenant-Overview]: howto-convert-app-to-be-multi-tenant.md
[AAD-QuickStart-Web-Apps]: v1-overview.md#get-started

<!--Image references-->
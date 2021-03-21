---
title: Добавление Google в качестве поставщика удостоверений для B2B — Azure AD
description: Активируйте Федерацию с Google, чтобы позволить гостевым пользователям входить в приложения Azure AD с помощью собственных учетных записей Gmail.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 03/02/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5a397c17821d16594ccfb48175a8a141cb9f390b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101687827"
---
# <a name="add-google-as-an-identity-provider-for-b2b-guest-users"></a>Добавление Google в качестве поставщика удостоверений для гостевых пользователей B2B

Настроив Федерацию с помощью Google, можно разрешить приглашенным пользователям входить в ваши общие приложения и ресурсы с собственными учетными записями Gmail, не создавая учетные записи Майкрософт.

После добавления Google в качестве одного из параметров входа приложения на странице **входа** пользователь может просто ввести электронное сообщение, используемое для входа в Google, или выбрать **Параметры входа** и выбрать **Вход с помощью Google**. В любом случае они будут перенаправлены на страницу входа Google для проверки подлинности.

![Варианты входа для пользователей Google](media/google-federation/sign-in-with-google-overview.png)

> [!NOTE]
> Федерация Google предназначена специально для пользователей Gmail. Чтобы создать федерацию с доменами G Suite, используйте [прямую федерацию](direct-federation.md).

> [!IMPORTANT]
> **Начиная с 4 января 2021 г.** компания Google [прекращает поддержку входа через веб-представление](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html). Если вы используете федерацию Google или самостоятельную регистрацию с помощью Gmail, вам нужно [проверить нативные бизнес-приложения на совместимость](google-federation.md#deprecation-of-webview-sign-in-support).

## <a name="what-is-the-experience-for-the-google-user"></a>Как эта возможность выглядит для пользователя Google?

Когда пользователь Google активирует ваше приглашение, его работа зависит от того, вошли ли они в Google:

- Гостевым пользователям, которые не вошли в Google, будет предложено сделать это.
- Гостевые пользователи, которые уже вошли в Google, получат запрос на выбор учетной записи, которую хотите использовать. Он должен выбрать учетную запись, которую вы использовали, чтобы пригласить его.

Гостевые пользователи, которые видят сообщение об ошибке "слишком длинный заголовок", могут очистить свои файлы cookie или открыть частное или режиме инкогнито окно и повторить попытку входа.

![Снимок экрана, на котором показана страница входа Google.](media/google-federation/google-sign-in.png)

## <a name="sign-in-endpoints"></a>Конечные точки входа

Пользователи Google Guest теперь могут входить в приложения с несколькими клиентами или сторонними приложениями Майкрософт, используя [общую конечную точку](redemption-experience.md#redemption-and-sign-in-through-a-common-endpoint) (иными словами, общий URL-адрес приложения, не включающий контекст клиента). В процессе входа гостевой пользователь выбирает **Параметры входа**, а затем выбирает **Вход в организацию**. Затем пользователь вводит имя Организации и возобновляет вход с использованием учетных данных Google.

Пользователи Google Guest могут также использовать конечные точки приложений, содержащие сведения о клиенте, например:

  * `https://myapps.microsoft.com/?tenantid=<your tenant ID>`
  * `https://myapps.microsoft.com/<your verified domain>.onmicrosoft.com`
  * `https://portal.azure.com/<your tenant ID>`

Вы также можете предоставить гостевым пользователям Google прямую ссылку на приложение или ресурс, например, включив сведения о клиенте `https://myapps.microsoft.com/signin/Twitter/<application ID?tenantId=<your tenant ID>` .

## <a name="deprecation-of-webview-sign-in-support"></a>Нерекомендуемая поддержка входа в WebView

Начиная с 4 января 2021 г. в Google не [рекомендуется использовать встроенную поддержку входа в WebView](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html). Если вы используете Google Federation или [самостоятельную регистрацию с помощью Gmail](identity-providers.md), необходимо протестировать собственные бизнес-приложения для обеспечения совместимости. Если в приложениях есть содержимое WebView, требующее проверки подлинности, пользователи Google Gmail не смогут пройти проверку подлинности. Ниже приведены известные сценарии, которые влияют на пользователей Gmail:

- Приложения Windows, использующие встроенные WebView или Вебаккаунтманажер (WAM) в более старых версиях Windows.
- Другие разрабатываемые собственные приложения, использующие встроенную платформу браузера для проверки подлинности.

Это изменение не влияет на:

- Приложения Windows, использующие встроенные WebView или Вебаккаунтманажер (WAM) в последних версиях Windows
- Приложения Microsoft iOS
- Идентификаторы G Suite, например, если вы используете [прямую федерацию](direct-federation.md) на основе SAML с G Suite

Мы продолжаем тестировать различные платформы и сценарии и соответствующим образом обновите эту статью.
### <a name="to-test-your-apps-for-compatibility"></a>Тестирование приложений на совместимость

1. Следуйте указаниям [Google](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html) , чтобы определить, затрагиваются ли ваши приложения.
2. С помощью Fiddler или другого средства тестирования Вставьте заголовок во время входа и используйте внешний идентификатор Google для проверки входа.

   1. Добавьте Google-accounts-Check-OAuth-login: true в заголовки HTTP-запроса при отправке запросов в accounts.google.com.
   1. Попробуйте войти в приложение, введя адрес Gmail на странице входа в accounts.google.com.
   1. Если вход завершается неудачей и появляется сообщение об ошибке, например "Этот браузер или приложение может быть незащищенным", внешние удостоверения Google будут заблокированы для входа.

3. Устраните проблему, выполнив одно из следующих действий.

   - Если приложение Windows использует внедренные WebView или Вебаккаунтманажер (WAM) в более старой версии Windows, обновите версию Windows до последней версии.
   - Измените приложения, чтобы использовать системный браузер для входа в систему. Дополнительные сведения см. в разделе [внедренный веб-интерфейс VS системы](../develop/msal-net-web-browsers.md#embedded-vs-system-web-ui) в документации по MSAL.NET.  


## <a name="step-1-configure-a-google-developer-project"></a>Шаг 1. Настройка проекта разработчика Google
Сначала создайте новый проект в консоли Google Developers, чтобы получить идентификатор клиента и секрет клиента, которые позже можно будет добавить в Azure Active Directory (Azure AD). 
1. Перейдите к API-интерфейсам Google на странице https://console.developers.google.com и войдите с помощью учетной записи Google. Мы рекомендуем использовать общую учетную запись Google, используемую командой.
2. При появлении запроса примите условия предоставления услуг.
3. Создайте новый проект: в левом верхнем углу страницы выберите список проект, а затем на странице **Выбор проекта** выберите **Новый проект**.
4. На странице **Новый проект** укажите имя проекта (например, **Azure AD B2B**), а затем выберите **создать**: 
   
   ![Снимок экрана, на котором показана новая страница проекта.](media/google-federation/google-new-project.png)

4. На странице **api & службы** выберите **представление** в новом проекте.

5. Выберите пункт **Переход к API обзор** на карточке API. Выберите **экран согласия OAuth**.

6. Щелкните **Внешний**, а затем — **Создать**. 

7. На **экране согласия OAuth** введите **имя приложения**:

   ![Снимок экрана с экраном согласия Google OAuth.](media/google-federation/google-oauth-consent-screen.png)

8. Перейдите к разделу " **полномочные домены** " и введите **microsoftonline.com**:

   ![Снимок экрана, показывающий раздел "полномочные домены".](media/google-federation/google-oauth-authorized-domains.PNG)

9. Щелкните **Сохранить**.

10. Выберите **Credentials** (Учетные данные). В меню **Создание учетных данных** выберите **идентификатор клиента OAuth**:

    ![Снимок экрана, показывающий меню создания учетных данных Google API.](media/google-federation/google-api-credentials.png)

11. Из списка **Application type** (Тип приложения) выберите **Web application** (Веб-приложение). Присвойте приложению подходящее имя, например **Azure AD B2B**. В разделе " **зарегистрированные URI перенаправления**" введите следующие URI:
    - `https://login.microsoftonline.com`
    - `https://login.microsoftonline.com/te/<tenant ID>/oauth2/authresp` <br>(где `<tenant ID>` — идентификатор клиента)
   
    > [!NOTE]
    > Чтобы найти идентификатор клиента, перейдите на [портал Azure](https://portal.azure.com). В разделе **Azure Active Directory** выберите **Свойства** и скопируйте **идентификатор клиента**.

    ![Снимок экрана, показывающий раздел идентификаторов URI для перенаправления.](media/google-federation/google-create-oauth-client-id.png)

12. Нажмите кнопку **создания**. Скопируйте идентификатор клиента и секрет клиента. Они будут использоваться при добавлении поставщика удостоверений в портал Azure.

    ![Снимок экрана, на котором показаны идентификатор клиента OAuth и секрет клиента.](media/google-federation/google-auth-client-id-secret.png)

## <a name="step-2-configure-google-federation-in-azure-ad"></a>Шаг 2. Настройка федерации с Google в Azure AD 
Теперь вы можете задать идентификатор клиента Google и секрет клиента. Для этого можно использовать портал Azure или PowerShell. Не забудьте проверить конфигурацию Google Federation, выполнив приглашение самостоятельно. Используйте адрес Gmail и попробуйте активировать приглашение с помощью приглашенной учетной записи Google. 

**Настройка Google Federation в портал Azure** 
1. Перейдите на [портал Microsoft Azure](https://portal.azure.com). В области слева выберите **Azure Active Directory**. 
2. Выберите **Внешние удостоверения**.
3. Выберите **все поставщики удостоверений**, а затем нажмите кнопку **Google** .
4. Введите идентификатор клиента и секрет клиента, полученные ранее. Выберите **сохранить**: 

   ![Снимок экрана, на котором показана страница добавления поставщика удостоверений Google.](media/google-federation/google-identity-provider.png)

**Настройка федерации Google с помощью PowerShell**
1. Установите последнюю версию модуля Azure AD PowerShell для Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Выполните следующую команду: `Connect-AzureAD`
3. При появлении запроса на вход войдите с помощью управляемой учетной записи глобального администратора.  
4. Выполните следующую команду: 
   
   `New-AzureADMSIdentityProvider -Type Google -Name Google -ClientId <client ID> -ClientSecret <client secret>`
 
   > [!NOTE]
   > Используйте идентификатор клиента и секрет клиента из приложения, созданного на шаге 1: Настройка проекта Google Developer. Дополнительные сведения см. в разделе [New-азуреадмсидентитипровидер](/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview). 
 
## <a name="how-do-i-remove-google-federation"></a>Как удалить федерацию с Google?
Вы можете удалить настройку федерации с Google. В этом случае пользователи Google Guest, которые уже активировали свои приглашения, не смогут войти в систему. Однако вы можете снова предоставить им доступ к ресурсам, удалив их из каталога и пригласив повторно. 
 
**Удаление Google Federation на портале Azure AD**
1. Перейдите на [портал Microsoft Azure](https://portal.azure.com). В области слева выберите **Azure Active Directory**. 
2. Выберите **Внешние удостоверения**.
3. Выберите **Все поставщики удостоверений**.
4. В строке **Google** нажмите кнопку с многоточием (**...**), а затем выберите **Удалить**. 
   
   ![Снимок экрана, на котором показана кнопка "Удалить" для поставщика удостоверений социальных сетей.](media/google-federation/google-social-identity-providers.png)

1. Выберите **Да** , чтобы подтвердить удаление. 

**Удаление Google Federation с помощью PowerShell** 
1. Установите последнюю версию модуля Azure AD PowerShell для Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Выполните `Connect-AzureAD`.  
4. При появлении запроса на вход войдите с помощью управляемой учетной записи глобального администратора.  
5. Введите следующую команду:

    `Remove-AzureADMSIdentityProvider -Id Google-OAUTH`

   > [!NOTE]
   > Дополнительные сведения см. в статье [Remove-AzureADMSIdentityProvider](/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview).

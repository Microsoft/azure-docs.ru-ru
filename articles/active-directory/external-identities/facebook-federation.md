---
title: Добавление Facebook в качестве поставщика удостоверений — Azure AD
description: Настройте федерацию с Facebook, чтобы внешние пользователи (гости) входили в ваши приложения Azure AD с помощью учетных записей Facebook.
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
ms.openlocfilehash: bcacfc8cede0ab91241fd431aace10cbbfbea916
ms.sourcegitcommit: dae6b628a8d57540263a1f2f1cdb10721ed1470d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "105709690"
---
# <a name="add-facebook-as-an-identity-provider-for-external-identities"></a>Добавление Facebook в качестве поставщика удостоверений для внешних удостоверений

Вы можете добавить Facebook в пользовательские потоки для самостоятельной регистрации, чтобы пользователи могли входить в приложения с помощью собственных учетных записей Facebook. Чтобы разрешить пользователям выполнять вход с помощью Facebook, сначала необходимо [включить самостоятельную регистрацию](self-service-sign-up-user-flow.md) для вашего арендатора. Добавив Facebook в качестве поставщика удостоверений, настройте поток пользователя для приложения и выберите Facebook в качестве одного из вариантов входа.

После добавления Facebook в качестве одного из параметров входа приложения на странице **входа** пользователь может просто ввести электронное сообщение, используемое для входа в Facebook, или выбрать **Параметры входа** и выбрать **Вход с помощью Facebook**. В любом случае они будут перенаправлены на страницу входа в Facebook для проверки подлинности.

![Параметры входа для пользователей Facebook](media/facebook-federation/sign-in-with-facebook-overview.png)

> [!NOTE]
> Пользователи могут использовать свои учетные записи Facebook только для регистрации в приложениях с помощью самостоятельной регистрации и пользовательских потоков. Пользователи не могут получать и активировать приглашения с помощью учетной записи Facebook.

## <a name="create-an-app-in-the-facebook-developers-console"></a>Создание приложения в консоли разработчиков Facebook

Чтобы использовать учетную запись Facebook в качестве [поставщика удостоверений](identity-providers.md), необходимо создать приложение в консоли разработчиков Facebook. Если у вас нет учетной записи Facebook, вы можете зарегистрироваться здесь: [https://www.facebook.com/](https://www.facebook.com).

> [!NOTE]  
> Используйте следующие URL-адреса на шагах 9 и 16 ниже.
> - В поле **URL-адрес сайта** введите адрес приложения, например `https://contoso.com` .
> - В качестве **допустимых URI перенаправления OAuth** введите `https://login.microsoftonline.com/te/<tenant-id>/oauth2/authresp`. Значение `<tenant-ID>` можно найти в Azure Active Directory в колонке "Обзор".


1. Выполните вход на [веб-сайт разработчиков Facebook](https://developers.facebook.com/) с учетными данными от учетной записи Facebook.
2. Если вы этого еще не сделали, необходимо зарегистрироваться в качестве разработчика Facebook. Для этого щелкните **Get Started** (Начать работу) в правом верхнем углу страницы, примите политики Facebook и завершите регистрацию.
3. Щелкните **My Apps** (Мои приложения) и **Create App** (Создать приложение).
4. Введите **Отображаемое название** и **Эл. адрес для связи**.
5. Щелкните **Create App ID** (Создать идентификатор приложения). Для этого может потребоваться принять условия политики платформы Facebook и пройти проверку безопасности в сети.
6. Выберите **Параметры** > **Основные**.
7. Щелкните **Category** (Категория) и выберите категорию, например Business and Pages (Бизнес и страницы). Это значение требуется для Facebook, но не для Azure AD.
8. В нижней части страницы, выберите **Добавить платформу**, а затем выберите **Веб-сайт**.
9. В поле **Site URL** (URL-адрес сайта) введите соответствующий URL-адрес (см. выше).
10. В поле **Privacy Policy URL** (URL-адрес политики конфиденциальности) введите URL-адрес страницы, на которой будут храниться сведения о конфиденциальности для вашего приложения, например `http://www.contoso.com`.
11. Щелкните **Save changes** (Сохранить изменения).
12. В верхней части страницы скопируйте значение **APP ID** (Идентификатор приложения).
13. Щелкните **Show** (Показать) и скопируйте значение **App Secret** (Секрет приложения). Оба значения потребуются вам для настройки Facebook в качестве поставщика удостоверений для вашего клиента. **Секрет приложения** — это важные учетные данные безопасности.
14. Щелкните знак плюса рядом с полем **PRODUCTS** (Продукты), а затем выберите **Set up** (Настройка) в разделе **Facebook Login** (Вход в Facebook).
15. В разделе **Facebook Login** (Вход в Facebook) щелкните **Параметры**.
16. В поле **Valid OAuth redirect URIs** (Допустимые URI перенаправления OAuth) введите соответствующий URL-адрес (см. выше).
17. Щелкните **Save Changes** (Сохранить изменения) в нижней части страницы.
18. Чтобы сделать приложение Facebook доступным в Azure AD, щелкните селектор состояния в правой верхней части страницы и установите его в положение **On** (Вкл.) для предоставления общего доступа к приложению, а затем нажмите кнопку **Switch Mode** (Переключить режим). На этом этапе состояние должно измениться с **Development** (Разработка) на **Live** (Активно).
    
## <a name="configure-a-facebook-account-as-an-identity-provider"></a>Настройка учетной записи Facebook в качестве поставщика удостоверений
Теперь необходимо задать идентификатор и секрет клиента Facebook, введя их на портале Azure AD или с помощью PowerShell. Вы можете проверить конфигурацию Facebook, зарегистрировавшись с помощью пользовательского потока в приложении, для которого включена самостоятельная регистрация.

### <a name="to-configure-facebook-federation-in-the-azure-ad-portal"></a>Настройка федерации с Facebook на портале Azure AD
1. Войдите на [портал Azure](https://portal.azure.com) с правами глобального администратора арендатора Azure AD.
2. В разделе **Службы Azure** щелкните **Azure Active Directory**.
3. В меню слева щелкните **Внешние удостоверения**.
4. Щелкните **Все поставщики удостоверений** и выберите **Facebook**.
5. В качестве **идентификатора клиента** введите **идентификатор приложения** Facebook, созданного ранее.
6. В качестве **секрета клиента** введите **секрет приложения**, записанный ранее.

   ![Снимок экрана: страница добавления поставщика удостоверений социальных сетей](media/facebook-federation/add-social-identity-provider-page.png)

7. Щелкните **Сохранить**.
### <a name="to-configure-facebook-federation-by-using-powershell"></a>Настройка федерации с Facebook с помощью PowerShell
1. Установите последнюю версию модуля Azure AD PowerShell для Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Выполните следующую команду: `Connect-AzureAD`.
3. При появлении запроса на вход войдите с помощью управляемой учетной записи глобального администратора.  
4. Выполните следующую команду: 
   
   `New-AzureADMSIdentityProvider -Type Facebook -Name Facebook -ClientId [Client ID] -ClientSecret [Client secret]`
 
   > [!NOTE]
   > Используйте идентификатор и секрет клиента из приложения, созданного ранее в консоли разработчиков Facebook. Дополнительные сведения см. в статье [New-AzureADMSIdentityProvider](/powershell/module/azuread/new-azureadmsidentityprovider?view=azureadps-2.0-preview&preserve-view=true). 

## <a name="how-do-i-remove-facebook-federation"></a>Как удалить федерацию с Facebook?
Вы можете удалить настройку федерации с Facebook. В этом случае все пользователи, которые зарегистрировались с помощью пользовательских потоков, используя свои учетные записи Facebook, больше не смогут выполнить вход. 

### <a name="to-delete-facebook-federation-in-the-azure-ad-portal"></a>Удаление федерации с Facebook на портале Azure AD 
1. Перейдите на [портал Azure](https://portal.azure.com). В области слева выберите **Azure Active Directory**. 
2. Выберите **Внешние удостоверения**.
3. Выберите **Все поставщики удостоверений**.
4. В строке **Facebook** раскройте контекстное меню ( **...** ), а затем выберите **Удалить**. 
5. Выберите **Да**, чтобы подтвердить удаление.

### <a name="to-delete-facebook-federation-by-using-powershell"></a>Удаление федерации с Facebook с помощью PowerShell 
1. Установите последнюю версию модуля Azure AD PowerShell для Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Выполните `Connect-AzureAD`.  
4. При появлении запроса на вход войдите с помощью управляемой учетной записи глобального администратора.  
5. Введите следующую команду:

    `Remove-AzureADMSIdentityProvider -Id Facebook-OAUTH`

   > [!NOTE]
   > Дополнительные сведения см. в статье [Remove-AzureADMSIdentityProvider](/powershell/module/azuread/Remove-AzureADMSIdentityProvider?view=azureadps-2.0-preview&preserve-view=true). 

## <a name="next-steps"></a>Дальнейшие действия

- [Добавление самостоятельной регистрации в приложение](self-service-sign-up-user-flow.md)
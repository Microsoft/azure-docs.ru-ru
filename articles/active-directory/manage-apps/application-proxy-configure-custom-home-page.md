---
title: Настраиваемая домашняя страница для опубликованных приложений — Azure AD Application Proxy
description: Основные сведения о соединителях прокси приложения Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/23/2019
ms.author: kenwith
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: ebef93455414c114fb60df9af8ebc629769a33e6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99254434"
---
# <a name="set-a-custom-home-page-for-published-apps-by-using-azure-ad-application-proxy"></a>Настройка пользовательской домашней страницы для опубликованных приложений с помощью прокси приложения Azure AD

В этой статье описывается настройка приложения для направления пользователя на пользовательскую домашнюю страницу. При публикации приложения с помощью прокси приложения задается внутренний URL-адрес, но иногда это не страница, которую пользователь должен видеть первым. Задайте пользовательскую домашнюю страницу, чтобы пользователь получит нужную страницу при доступе к приложению. Пользователь увидит настраиваемую домашнюю страницу, независимо от того, будут ли они получать доступ к приложению из Azure Active Directory "Мои приложения" или "средство запуска Microsoft 365ного приложения".

Когда пользователь запускает приложение, он по умолчанию перенаправляется на URL-адрес корневого домена опубликованного приложения. Обычно целевой страницей считается URL-адрес домашней страницы приложения. Используйте модуль Azure AD PowerShell для определения настраиваемого URL-адреса домашней страницы, если нужно, чтобы пользователь приложения наземный определенную страницу в приложении.

Ниже приведен один из сценариев, в котором объясняется, почему ваша компания настроила пользовательскую домашнюю страницу:

- В корпоративной сети пользователь переходит на `https://ExpenseApp/login/login.aspx` Вход и доступ к вашему приложению.
- Так как у вас есть другие ресурсы (например, изображения), к которым прокси приложения должны получить доступ на верхнем уровне структуры папок, вы публикуете приложение с `https://ExpenseApp` внутренним URL-адресом.
- Внешний URL-адрес по умолчанию — `https://ExpenseApp-contoso.msappproxy.net` , который не принимает внешнего пользователя на страницу входа.
- Вы хотите задать `https://ExpenseApp-contoso.msappproxy.net/login/login.aspx` вместо этого URL-адрес домашней страницы, поэтому внешний пользователь увидит первую страницу входа.

> [!NOTE]
> При предоставлении пользователям доступа к опубликованным приложениям приложения отображаются в окне " [Мои приложения](../user-help/my-apps-portal-end-user-access.md) " и в средстве [запуска приложений Office 365](https://www.microsoft.com/microsoft-365/blog/2016/09/27/introducing-the-new-office-365-app-launcher/).

## <a name="before-you-start"></a>Перед началом работы

Прежде чем указывать URL-адрес домашней страницы, учтите следующие требования:

- Указываемый путь должен быть путем к поддомену URL-адреса корневого домена.

  Например, если URL-адрес корневого домена — `https://apps.contoso.com/app1/` , то настраиваемый URL-адрес домашней страницы должен начинаться с `https://apps.contoso.com/app1/` .

- Когда вы вносите изменения в опубликованное приложение, URL-адрес домашней страницы может вернуться к значению по умолчанию. Поэтому при дальнейшем обновлении приложения следует повторно проверить URL-адрес домашней страницы и изменить его при необходимости.

URL-адрес домашней страницы можно задать либо с помощью портал Azure, либо с помощью PowerShell.

## <a name="change-the-home-page-in-the-azure-portal"></a>Изменение домашней страницы на портале Azure

Чтобы изменить URL-адрес домашней страницы приложения на портале Azure AD, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/) с использованием учетной записи администратора.
1. Выберите **Azure Active Directory**, а затем **Регистрация приложений**. Появится список зарегистрированных приложений.
1. Выберите приложение из списка. Откроется страница с подробными сведениями о зарегистрированном приложении.
1. В разделе **Управление** выберите **фирменная символика**.
1. Обновите **URL-адрес домашней страницы**  , указав новый путь.

   ![Страница фирменной символики для зарегистрированного приложения, показывающего поле URL-адреса домашней страницы](media/application-proxy-configure-custom-home-page/app-proxy-app-branding.png)

1. Щелкните **Сохранить**.

## <a name="change-the-home-page-with-powershell"></a>Изменение домашней страницы с помощью PowerShell

Чтобы настроить домашнюю страницу приложения с помощью PowerShell, необходимо выполнить следующие действия.

1. Установите модуль Azure AD PowerShell.
1. Найдите значение ObjectId приложения.
1. Обновите URL-адрес домашней страницы приложения с помощью команд PowerShell.

### <a name="install-the-azure-ad-powershell-module"></a>Установка модуля Azure AD PowerShell

Прежде чем настраивать URL-адрес пользовательской домашней страницы с помощью PowerShell, установите модуль Azure AD PowerShell. Этот пакет можно скачать из [коллекции PowerShell](https://www.powershellgallery.com/packages/AzureAD/2.0.2.16), в которой используется конечная точка API Graph.

Для установки пакета выполните следующие действия.

1. Откройте стандартное окно PowerShell и выполните следующую команду.

   ```powershell
   Install-Module -Name AzureAD
   ```

    Если вы запускаете команду не от имени администратора, используйте параметр `-scope currentuser`.

1. Во время установки выберите **Y** , чтобы установить два пакета из NuGet.org. Оба пакета являются обязательными.

### <a name="find-the-objectid-of-the-app"></a>Найти ObjectId приложения

Идентификатор объекта для приложения можно получить, выполнив поиск приложения по его отображаемому имени или домашней странице.

1. В том же окне PowerShell импортируйте модуль Azure AD.

   ```powershell
   Import-Module AzureAD
   ```

1. Войдите в модуль Azure AD в качестве администратора клиента.

   ```powershell
   Connect-AzureAD
   ```

1. Найдите приложение. В этом примере PowerShell используется для поиска ObjectId путем поиска приложения с отображаемым именем `SharePoint` .

   ```powershell
   Get-AzureADApplication | Where-Object { $_.DisplayName -eq "SharePoint" } | Format-List DisplayName, Homepage, ObjectId
   ```

   Вы должны получить результат, похожий на приведенный ниже. Скопируйте идентификатор GUID ObjectId для использования в следующем разделе.

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

   Кроме того, можно просто извлечь список всех приложений, найти приложение в списке с конкретным отображаемым именем или домашней страницей, а затем скопировать ObjectId приложения после обнаружения приложения.

   ```powershell
   Get-AzureADApplication | Format-List DisplayName, Homepage, ObjectId
   ```

### <a name="update-the-home-page-url"></a>Изменение URL-адреса домашней страницы

Создайте URL-адрес домашней страницы и обновите приложение, указав это значение. Продолжайте использовать то же окно PowerShell или, если вы используете новое окно PowerShell, снова войдите в модуль Azure AD с помощью `Connect-AzureAD` . Затем выполните следующие действия:

1. Создайте переменную для хранения значения ObjectId, скопированного в предыдущем разделе. (Замените значение ObjectId, используемое для в этом примере SharePoint, значением ObjectId приложения.)

   ```powershell
   $objguid = "8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4"
   ```

1. Убедитесь, что у вас есть нужное приложение, выполнив следующую команду. Выходные данные должны быть идентичны выходным данным, которые вы видели в предыдущем разделе ([найдите идентификатор ObjectID приложения](#find-the-objectid-of-the-app)).

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

1. Создайте пустой объект приложения, в котором будут храниться ваши изменения.

   ```powershell
   $appnew = New-Object "Microsoft.Open.AzureAD.Model.Application"
   ```

1. Задайте нужное значение для URL-адреса домашней страницы. Это значение должно быть путем к поддомену опубликованного приложения. Например, если изменить URL-адрес домашней страницы с `https://sharepoint-iddemo.msappproxy.net/` на `https://sharepoint-iddemo.msappproxy.net/hybrid/`, пользователи приложения перейдут непосредственно на пользовательскую домашнюю страницу.

   ```powershell
   $homepage = "https://sharepoint-iddemo.msappproxy.net/hybrid/"
   ```

1. Обновите домашнюю страницу.

   ```powershell
   Set-AzureADApplication -ObjectId $objguid -Homepage $homepage
   ```

1. Чтобы убедиться, что изменение прошло успешно, снова выполните следующую команду из шага 2.

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

   В нашем примере выходные данные теперь должны выглядеть следующим образом:

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/hybrid/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

1. Перезапустите приложение, чтобы убедиться, что домашняя страница отображается в качестве первого экрана, как ожидалось.

> [!NOTE]
> Все изменения, которые вы внесли в приложение, могут сбросить URL-адрес домашней страницы. В случае сброса URL-адреса домашней страницы повторите шаги в этом разделе, чтобы снова задать его.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка удаленного доступа к SharePoint с помощью прокси приложения Azure AD](application-proxy-integrate-with-sharepoint-server.md)
- [Руководство по Добавление локального приложения для удаленного доступа через Application Proxy в Azure Active Directory](application-proxy-add-on-premises-application.md)

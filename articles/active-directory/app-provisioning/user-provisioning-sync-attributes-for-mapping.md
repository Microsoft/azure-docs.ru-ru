---
title: Синхронизация атрибутов в Azure AD для сопоставления
description: Узнайте, как синхронизировать атрибуты из локального Active Directory с Azure AD. При настройке подготовки пользователей для приложений SaaS используйте функцию расширения каталога, чтобы добавить исходные атрибуты, которые не синхронизированы по умолчанию.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 03/12/2021
ms.author: kenwith
ms.openlocfilehash: 0f8369c80a7a219b159f31aacb7d10a0dd009d00
ms.sourcegitcommit: df1930c9fa3d8f6592f812c42ec611043e817b3b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/13/2021
ms.locfileid: "103418680"
---
# <a name="sync-an-attribute-from-your-on-premises-active-directory-to-azure-ad-for-provisioning-to-an-application"></a>Синхронизация атрибута из локального Active Directory с Azure AD для подготовки приложения

При настройке сопоставлений атрибутов для подготовки пользователей может оказаться, что атрибут, который нужно сопоставить, не отображается в списке **исходных атрибутов** . В этой статье показано, как добавить недостающий атрибут, синхронизируя его из локальной Active Directory (AD) с Azure Active Directory (Azure AD).

В Azure AD должны содержаться все данные, необходимые для создания профиля пользователя при подготовке учетных записей пользователей из Azure AD в приложение SaaS. В некоторых случаях для обеспечения доступности данных может потребоваться синхронизация атрибутов из локальной службы AD в Azure AD. Azure AD Connect автоматически синхронизирует определенные атрибуты с Azure AD, но не со всеми атрибутами. Кроме того, некоторые атрибуты (например, SAMAccountName), синхронизированные по умолчанию, могут быть недоступны с помощью Microsoft Graph API. В таких случаях можно использовать функцию расширения каталога Azure AD Connect, чтобы синхронизировать атрибут с Azure AD. Таким образом, атрибут будет видим для Microsoft Graph API и службы подготовки Azure AD.

Если данные, необходимые для подготовки, находятся в Active Directory но недоступны для подготовки из-за описанных выше причин, можно использовать Azure AD Connect или PowerShell для создания атрибутов расширения. 
 
## <a name="create-an-extension-attribute-using-azure-ad-connect"></a>Создание атрибута расширения с помощью Azure AD Connect

1. Откройте мастер Azure AD Connect, выберите задачи, а затем выберите **настроить параметры синхронизации**.

   ![Страница "дополнительные задачи" мастера Azure Active Directory Connect](./media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-customize.png)
 
2. Войдите в систему как глобальный администратор Azure AD. 

3. На странице **Дополнительные компоненты** выберите **синхронизировать атрибут расширения каталога**.
 
   ![Страница дополнительных компонентов мастера Azure Active Directory Connect](./media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-directory-extension-attribute-sync.png)

4. Выберите атрибуты, которые требуется расширить в Azure AD.
   > [!NOTE]
   > При поиске в списке **Доступные атрибуты** учитывается регистр.

   ![Снимок экрана, на котором показана страница выбора "расширения каталога"](./media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-directory-extensions.png)

5. Завершите работу мастера Azure AD Connect и дождитесь выполнения цикла полной синхронизации. По завершении цикла схема расширяется, а новые значения синхронизируются между локальной службой AD и Azure AD.
 
6. В портал Azure при [редактировании сопоставлений атрибутов пользователя](customize-application-attributes.md)список **исходных атрибутов** теперь будет содержать добавленный атрибут в формате `<attributename> (extension_<appID>_<attributename>)` . Выберите атрибут и сопоставьте его целевому приложению для подготовки.

   ![Страница выбора расширений каталога мастера Azure Active Directory Connect](./media/user-provisioning-sync-attributes-for-mapping/attribute-mapping-extensions.png)

> [!NOTE]
> Возможность подготавливать ссылочные атрибуты из локальной службы AD, например **ManagedBy** или **DN/distinguishedName**, сейчас не поддерживается. Эту функцию можно запросить на [голоса пользователя](https://feedback.azure.com/forums/169401-azure-active-directory). 

## <a name="create-an-extension-attribute-using-powershell"></a>Создание атрибута расширения с помощью PowerShell
Создайте пользовательское расширение с помощью PowerShell и назначьте ему значение. 

```
#Connect to your Azure AD tenant   
Connect-AzureAD

#Create an application (you can instead use an existing application if you would like)
$App = New-AzureADApplication -DisplayName “test app name” -IdentifierUris https://testapp

#Create a service principal
New-AzureADServicePrincipal -AppId $App.AppId

#Create an extension property
New-AzureADApplicationExtensionProperty -ObjectId $App.ObjectId -Name “TestAttributeName” -DataType “String” -TargetObjects “User”

#List users in your tenant to determine the objectid for your user
Get-AzureADUser

#Set a value for the extension property on the user. Replace the objectid with the id of the user and the extension name with the value from the previous step
Set-AzureADUserExtension -objectid 0ccf8df6-62f1-4175-9e55-73da9e742690 -ExtensionName “extension_6552753978624005a48638a778921fan3_TestAttributeName”

#Verify that the attribute was added correctly.
Get-AzureADUser -ObjectId 0ccf8df6-62f1-4175-9e55-73da9e742690 | Select -ExpandProperty ExtensionProperty

```

## <a name="next-steps"></a>Дальнейшие действия

* [Определение пользователей, которые находятся в области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)

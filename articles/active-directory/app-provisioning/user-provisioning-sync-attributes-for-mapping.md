---
title: Синхронизация атрибутов в Azure AD для сопоставления
description: При настройке подготовки пользователей для приложений SaaS используйте функцию расширения каталога, чтобы добавить исходные атрибуты, которые не синхронизированы по умолчанию.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 03/17/2021
ms.author: kenwith
ms.openlocfilehash: 52f34cdafac76a9bca2b4ff0b00e0b3efaa63f5d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104579439"
---
# <a name="syncing-extension-attributes-attributes"></a>Синхронизация атрибутов атрибутов расширения

При настройке сопоставлений атрибутов для подготовки пользователей может оказаться, что атрибут, который нужно сопоставить, не отображается в списке **исходных атрибутов** . В этой статье показано, как добавить недостающий атрибут, синхронизируя его из локальной Active Directory (AD) с Azure Active Directory (Azure AD) или создав атрибуты расширения в Azure AD для пользователя облака. 

В Azure AD должны содержаться все данные, необходимые для создания профиля пользователя при подготовке учетных записей пользователей из Azure AD в приложение SaaS. В некоторых случаях для обеспечения доступности данных может потребоваться синхронизация атрибутов из локальной службы AD в Azure AD. Azure AD Connect автоматически синхронизирует определенные атрибуты с Azure AD, но не со всеми атрибутами. Кроме того, некоторые атрибуты (например, SAMAccountName), синхронизированные по умолчанию, могут быть недоступны с помощью API Graph Azure AD. В таких случаях можно использовать функцию расширения каталога Azure AD Connect, чтобы синхронизировать атрибут с Azure AD. Таким образом, атрибут будет отображаться для API Graph Azure AD и службы подготовки Azure AD. Если данные, необходимые для подготовки, находятся в Active Directory но недоступны для подготовки из-за описанных выше причин, можно использовать Azure AD Connect для создания атрибутов расширения. 

Хотя большинство пользователей скорее всего являются гибридными пользователями, которые синхронизируются с Active Directory, вы также можете создавать расширения только для облачных пользователей без использования Azure AD Connect. С помощью PowerShell или Microsoft Graph можно расширить схему только облачного пользователя. 

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

## <a name="create-an-extension-attribute-on-a-cloud-only-user"></a>Создание атрибута расширения только для облачного пользователя
Клиенты могут использовать Microsoft Graph и PowerShell для расширения схемы пользователя. Эти атрибуты расширений автоматически обнаруживаются в большинстве случаев, но у клиентов, у которых более 1000 субъектов-служб, могут обнаружиться расширения, отсутствующие в списке исходных атрибутов. Если атрибут, созданный с помощью описанных ниже шагов, не отображается в списке исходных атрибутов автоматически, проверьте использование Graph, который был успешно создан атрибутом расширения, и добавьте его в схему [вручную](https://docs.microsoft.com/azure/active-directory/app-provisioning/customize-application-attributes#editing-the-list-of-supported-attributes). При выполнении запросов графов ниже щелкните Дополнительные сведения, чтобы проверить разрешения, необходимые для выполнения запросов. Для выполнения запросов можно использовать [Обозреватель Graph](https://docs.microsoft.com/graph/graph-explorer/graph-explorer-overview) . 

### <a name="create-an-extension-attribute-on-a-cloud-only-user-using-microsoft-graph"></a>Создание атрибута расширения только для облачного пользователя с помощью Microsoft Graph
Для расширения схемы пользователей необходимо использовать приложение. Перечислите приложения в клиенте, чтобы указать идентификатор приложения, которое вы хотите использовать для расширения схемы пользователя. [Подробнее.](https://docs.microsoft.com/graph/api/application-list?view=graph-rest-1.0&tabs=http)

```json
GET https://graph.microsoft.com/v1.0/applications
```

Создайте атрибут расширения. Замените приведенное ниже свойство **ID** на **идентификатор** , полученный на предыдущем шаге. Необходимо будет использовать атрибут **ID** , а не AppID. [Подробнее.](https://docs.microsoft.com/graph/api/application-post-extensionproperty?view=graph-rest-1.0&tabs=http)
```json
POST https://graph.microsoft.com/v1.0/applications/{id}/extensionProperties
Content-type: application/json

{
    "name": "extensionName",
    "dataType": "string",
    "targetObjects": [
        "User"
    ]
}
```

Предыдущий запрос создал атрибут расширения с форматом "extension_appID_extensionName". Обновите пользователя с помощью атрибута расширения. [Подробнее.](https://docs.microsoft.com/graph/api/user-update?view=graph-rest-1.0&tabs=http)
```json
PATCH https://graph.microsoft.com/v1.0/users/{id}
Content-type: application/json

{
  "extension_inputAppId_extensionName": "extensionValue"
}
```
Проверьте пользователя, чтобы убедиться, что атрибут был успешно обновлен. [Подробнее.](https://docs.microsoft.com/graph/api/user-get?view=graph-rest-1.0&tabs=http#example-3-users-request-using-select)

```json
GET https://graph.microsoft.com/v1.0/users/{id}?$select=displayName,extension_inputAppId_extensionName
```


### <a name="create-an-extension-attribute-on-a-cloud-only-user-using-powershell"></a>Создание атрибута расширения только для облачного пользователя с помощью PowerShell
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

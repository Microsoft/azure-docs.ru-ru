---
title: Проверка подлинности приложения для доступа к службе Azure SignalR
description: В этой статье содержатся сведения о проверке подлинности приложения с помощью Azure Active Directory для доступа к службе Azure SignalR.
author: terencefan
ms.author: tefa
ms.service: signalr
ms.topic: conceptual
ms.date: 08/03/2020
ms.openlocfilehash: 597b69c1180ea1fb2a6812d648f8b8ad37707d66
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101092599"
---
# <a name="authenticate-an-application-with-azure-active-directory-to-access-azure-signalr-service"></a>Проверка подлинности приложения с Azure Active Directory для доступа к службе Azure SignalR
Microsoft Azure предоставляет интегрированную систему управления доступом к ресурсам и приложениям на основе Azure Active Directory (Azure AD). Ключевым преимуществом использования Azure AD со службой SignalR Azure является то, что вам больше не нужно хранить учетные данные в коде. Вместо этого можно запросить маркер доступа OAuth 2,0 на платформе Microsoft Identity. Имя ресурса для запроса маркера — `https://signalr.azure.com/` . Azure AD выполняет проверку подлинности субъекта безопасности (приложения, группы ресурсов или субъекта-службы), на котором выполняется приложение. Если проверка подлинности прошла, Azure AD возвращает маркер доступа к приложению, а приложение может использовать маркер доступа для авторизации запросов к ресурсам службы SignalR Azure.

Когда роль назначается субъекту безопасности Azure AD, Azure предоставляет доступ к этим ресурсам для этого субъекта безопасности. Доступ может быть ограничен уровнем подписки, группой ресурсов или ресурсом SignalR Azure. Безопасность Azure AD может назначать роли пользователю, группе, субъекту-службе приложения или [управляемому удостоверению для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md). 

> [!NOTE]
> Определение роли представляет собой коллекцию разрешений. Управление доступом на основе ролей (RBAC) управляет применением этих разрешений с помощью назначения ролей. Назначение ролей состоит из трех элементов: субъект безопасности, определение роли и область действия. Дополнительные сведения см. [в разделе понимание различных ролей](../role-based-access-control/overview.md).

## <a name="register-your-application-with-an-azure-ad-tenant"></a>Регистрация приложения в клиенте Azure AD
Первым шагом в использовании Azure AD для авторизации ресурсов службы SignalR Azure является регистрация приложения в клиенте Azure AD из [портал Azure](https://portal.azure.com/). При регистрации приложения вы предоставляете сведения о приложении в AD. Затем Azure AD предоставляет идентификатор клиента (также называемый ИДЕНТИФИКАТОРом приложения), с помощью которого можно связать приложение со средой выполнения Azure AD. Дополнительные сведения об идентификаторе клиента см. в статье [Объекты приложения и субъекта-службы в Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md). 

На следующих изображениях показаны шаги для регистрации веб-приложения.

![Регистрация приложения](./media/authenticate/app-registrations-register.png)

> [!Note]
> При регистрации приложения в качестве собственного приложения можно указать любой допустимый универсальный код ресурса (URI) для URI перенаправления. Для собственных приложений это значение не обязательно должно быть действительным URL-адресом. Для веб-приложений URI перенаправления должен быть допустимым URI, так как он указывает URL-адрес, на который предоставляются токены.

После регистрации приложения в разделе **Параметры** отобразится **идентификатор приложения (клиента)** .

![Идентификатор приложения зарегистрированного приложения](./media/authenticate/application-id.png)

Дополнительные сведения о регистрации приложения в Azure AD см. в разделе [Интеграция приложений с Azure Active Directory](../active-directory/develop/quickstart-register-app.md).


### <a name="create-a-client-secret"></a>Создание секрета клиента   
Приложению требуется секрет клиента для подтверждения его личности при запросе маркера. Чтобы добавить секрет клиента, выполните следующие действия.

1. Перейдите к регистрации приложения в портал Azure.
1. Выберите параметр **Сертификаты и секреты**.
1. В разделе **секреты клиента** выберите **новый секрет клиента** , чтобы создать новый секрет.
1. Введите описание секрета и выберите желаемый интервал срока действия.
1. Сразу скопируйте значение нового секрета в безопасное место. Значение заполнения отображается только один раз.

![Создание секрета клиента](./media/authenticate/client-secret.png)

### <a name="upload-a-certificate"></a>Загрузить сертификат

Можно также отправить сертификат вместо создания секрета клиента.

![Отправка сертификата](./media/authenticate/certification.png)

## <a name="assign-azure-roles-using-the-azure-portal"></a>Назначение ролей Azure с помощью портала Azure  
Дополнительные сведения об управлении доступом к ресурсам Azure с помощью Azure RBAC и портал Azure см. в [этой статье](..//role-based-access-control/role-assignments-portal.md). 

Определив соответствующую область для назначения ролей, перейдите к этому ресурсу в портал Azure. Отобразить параметры управления доступом (IAM) для ресурса и выполнить следующие инструкции для управления назначениями ролей:

1. В [портал Azure](https://portal.azure.com/)перейдите к ресурсу SignalR.
1. Выберите **Управление доступом (IAM)** , чтобы отобразить параметры управления доступом для Azure SignalR. 
1. Выберите вкладку **Назначения ролей**, чтобы просмотреть список назначений ролей. Нажмите кнопку **Добавить** на панели инструментов, а затем выберите **добавить назначение ролей**. 

    ![Кнопка "Добавить" на панели инструментов](./media/authenticate/role-assignments-add-button.png)

1. На странице **Добавление назначения ролей** выполните следующие действия.
    1. Выберите **роль Azure SignalR** , которую вы хотите назначить. 
    1. Найдите **участника безопасности** (пользователя, группы, субъекта-службы), которому нужно назначить роль.
    1. Нажмите кнопку **сохранить** , чтобы сохранить назначение ролей. 

        ![Назначение роли приложению](./media/authenticate/assign-role-to-application.png)

    1. Удостоверение, которому назначена роль RBAC, будет отображаться в списке под этой ролью. Например, на следующем рисунке показано приложение `signalr-dev` и `signalr-service` они находятся в роли сервера приложений SignalR. 
        
        ![Список назначений ролей](./media/authenticate/role-assignment-list.png)

Вы можете выполнить аналогичные действия, чтобы назначить роль для группы ресурсов или подписки. Определив роль и ее область, вы можете проверить это поведение с помощью примеров [в этом расположении GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac).

## <a name="sample-codes-while-configuring-your-app-server"></a>Примеры кодов при настройке сервера приложений.

Добавьте следующие параметры, когда `AddAzureSignalR` :

```C#
services.AddSignalR().AddAzureSignalR(option =>
{
    option.ConnectionString = "Endpoint=https://<name>.signalr.net;AuthType=aad;clientId=<clientId>;clientSecret=<clientSecret>;tenantId=<tenantId>";
});
```

Или просто настройте `ConnectionString` в `appsettings.json` файле.

```json
{
"Azure": {
    "SignalR": {
      "Enabled": true,
      "ConnectionString": "Endpoint=https://<name>.signalr.net;AuthType=aad;clientId=<clientId>;clientSecret=<clientSecret>;tenantId=<tenantId>"
    }
  },
}
```

При использовании `certificate` измените в `clientSecret` `clientCert` следующим образом:

```C#
    option.ConnectionString = "Endpoint=https://<name>.signalr.net;AuthType=aad;clientId=<clientId>;clientCert=<clientCertFilepath>;tenantId=<tenantId>";
```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о RBAC см. в статье [что такое управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md)?
- Чтобы узнать, как назначать назначения ролей Azure и управлять ими с помощью Azure PowerShell, Azure CLI или REST API, см. следующие статьи:
    - [Управление доступом на основе ролей с помощью Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)  
    - [Управление доступом на основе ролей с помощью интерфейса командной строки Azure](../role-based-access-control/role-assignments-cli.md)
    - [Управление доступом на основе ролей с помощью REST API](../role-based-access-control/role-assignments-rest.md)
    - [Управление доступом на основе ролей (RBAC) с помощью шаблонов Azure Resource Manager](../role-based-access-control/role-assignments-template.md)

См. следующие статьи:
- [Проверка подлинности управляемого удостоверения с Azure Active Directory для доступа к службе Azure SignalR](authenticate-managed-identity.md)
- [Авторизация доступа к службе Azure SignalR с помощью Azure Active Directory](authorize-access-azure-active-directory.md)
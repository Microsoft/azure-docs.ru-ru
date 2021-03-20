---
title: Проверка подлинности управляемого удостоверения с помощью Azure Active Directory
description: В этой статье содержатся сведения о проверке подлинности управляемого удостоверения с помощью Azure Active Directory для доступа к службе Azure SignalR.
author: terencefan
ms.author: tefa
ms.date: 08/03/2020
ms.service: signalr
ms.topic: conceptual
ms.openlocfilehash: c561bb507a5178f4a838b370a3da8af9447829f4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101092556"
---
# <a name="authenticate-a-managed-identity-with-azure-active-directory-to-access-azure-signalr-resources"></a>Проверка подлинности управляемого удостоверения с Azure Active Directory для доступа к ресурсам Azure SignalR
Служба Azure SignalR поддерживает проверку подлинности Azure Active Directory (Azure AD) с помощью [управляемых удостоверений для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md). Управляемые удостоверения для ресурсов Azure могут авторизовать доступ к ресурсам службы Azure SignalR с помощью учетных данных Azure AD из приложений, работающих на виртуальных машинах Azure, приложений-функций, масштабируемых наборов виртуальных машин и других служб. Используя управляемые удостоверения для ресурсов Azure вместе с проверкой подлинности Azure AD, можно избежать хранения учетных данных в приложениях, выполняемых в облаке.

В этой статье показано, как авторизовать доступ к службе Azure SignalR с помощью управляемого удостоверения на виртуальной машине Azure.

## <a name="enable-managed-identities-on-a-vm"></a>Включение управляемых удостоверений на виртуальной машине
Прежде чем использовать управляемые удостоверения для ресурсов Azure для авторизации ресурсов службы Azure SignalR из виртуальной машины, сначала необходимо включить управляемые удостоверения для ресурсов Azure на виртуальной машине. Сведения о включении управляемых удостоверений для ресурсов Azure см. в одной из следующих статей.

- [Портал Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Шаблон Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Клиентские библиотеки Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-a-managed-identity-in-azure-ad"></a>Предоставление разрешений управляемому удостоверению в Azure AD
Чтобы авторизовать запрос к службе Azure SignalR из управляемого удостоверения в приложении, сначала настройте параметры управления доступом на основе ролей (RBAC) для этого управляемого удостоверения. Служба Azure SignalR определяет роли RBAC, охватывающие разрешения на получение `AccessKey` или `ClientToken` . Когда роль RBAC назначается управляемому удостоверению, управляемому удостоверению предоставляется доступ к службе Azure SignalR в соответствующей области.

Дополнительные сведения о назначении ролей RBAC см. [в статье аутентификация с помощью Azure Active Directory для доступа к ресурсам службы Azure SignalR](authorize-access-azure-active-directory.md).

## <a name="connect-to-azure-signalr-service-with-managed-identities"></a>Подключение к службе Azure SignalR с помощью управляемых удостоверений
Чтобы подключиться к службе Azure SignalR с управляемыми удостоверениями, необходимо назначить удостоверение роли и соответствующую область. Процедура в этом разделе использует простое приложение, которое выполняется под управляемым удостоверением и обращается к ресурсам службы SignalR Azure.

Здесь мы используем пример ресурса виртуальной машины Azure.

1. Перейдите в раздел **Параметры** и выберите **удостоверение**. 
1. Выберите **состояние** **включено**. 
1. Нажмите кнопку **Сохранить**, чтобы сохранить параметры. 

    ![Управляемое удостоверение для виртуальной машины](./media/authenticate/identity-virtual-machine.png)

После включения этого параметра в Azure Active Directory (Azure AD) создается новое удостоверение службы, которое настраивается в узле службы приложений.

Теперь назначьте это удостоверение службы роли в требуемой области в ресурсах службы Azure SignalR.

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

## <a name="samples-code-while-configuring-your-app-server"></a>Примеры кода при настройке сервера приложений

Добавьте следующие параметры, когда `AddAzureSignalR` :

```C#
services.AddSignalR().AddAzureSignalR(option =>
{
    option.ConnectionString = "Endpoint=https://<name>.signalr.net;AuthType=aad;";
});
```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о RBAC см. в статье [что такое управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md)?
- Чтобы узнать, как назначать назначения ролей Azure и управлять ими с помощью Azure PowerShell, Azure CLI или REST API, см. следующие статьи:
    - [Управление доступом на основе ролей с помощью Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)  
    - [Управление доступом на основе ролей с помощью интерфейса командной строки Azure](../role-based-access-control/role-assignments-cli.md)
    - [Управление доступом на основе ролей с помощью REST API](../role-based-access-control/role-assignments-rest.md)
    - [Управление доступом на основе ролей (RBAC) с помощью шаблонов Azure Resource Manager](../role-based-access-control/role-assignments-template.md)

См. следующие статьи:
- [Проверка подлинности приложения с Azure Active Directory для доступа к службе Azure SignalR](authenticate-application.md)
- [Авторизация доступа к службе Azure SignalR с помощью Azure Active Directory](authorize-access-azure-active-directory.md)
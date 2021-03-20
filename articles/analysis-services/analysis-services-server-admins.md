---
title: Управление администраторами серверов в службах Azure Analysis Services | Документы Майкрософт
description: В этой статье описывается, как управлять администраторами сервера для Azure Analysis Services сервера с помощью портал Azure, PowerShell или интерфейсов API.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 2/4/2021
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 62acb526a247362b17c4dfd4e26c52760deecd71
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99573506"
---
# <a name="manage-server-administrators"></a>Управление администраторами серверов

Администратор сервера должен быть допустимым пользователем, субъектом-службой или группой безопасности в Azure Active Directory (Azure AD) для клиента, в котором находится сервер. Для управления администраторами сервера можно использовать элемент **Администраторы служб Analysis Services** для сервера на портале Azure или свойств сервера в SSMS, PowerShell или REST API. 

При добавлении **группы безопасности** используйте `obj:groupid@tenantid` . Субъекты-службы не поддерживаются в группах безопасности, добавленных к роли администратора сервера.

Дополнительные сведения о добавлении субъекта-службы к роли администратора сервера см. в разделе [Добавление субъекта-службы к роли администратора сервера](analysis-services-addservprinc-admins.md).

Если включен брандмауэр сервера, то IP-адреса клиентского компьютера администратора сервера должны быть включены в правило брандмауэра. Дополнительные сведения см. в разделе [Настройка брандмауэра сервера](analysis-services-qs-firewall.md).

## <a name="to-add-server-administrators-by-using-azure-portal"></a>Добавление администраторов на портале Azure

1. На портале выберите для сервера **Администраторы служб Analysis Services**.
2. В **\<servername> Analysis Services "Администраторы**" нажмите кнопку **добавить**.
3. В области **добавления администраторов сервера** выберите учетные записи пользователей из Azure AD или пригласите внешних пользователей по электронной почте.

    ![Администраторы сервера на портале Azure](./media/analysis-services-server-admins/aas-manage-users-admins.png)

## <a name="to-add-server-administrators-by-using-ssms"></a>Добавление администраторов сервера с помощью SSMS

1. Щелкните правой кнопкой мыши сервер и выберите пункт **Свойства**.
2. В разделе **Свойства сервера анализа данных** выберите **Безопасность**.
3. Нажмите кнопку **Добавить**, а затем введите адрес электронной почты для пользователя или группы в Azure AD.
   
    ![Добавление администраторов сервера в SSMS](./media/analysis-services-server-admins/aas-manage-users-ssms.png)

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Используйте командлет [New-азаналисиссервицессервер](/powershell/module/az.analysisservices/new-azanalysisservicesserver) , чтобы указать параметр администратора при создании нового сервера. <br>
Используйте командлет [Set-азаналисиссервицессервер](/powershell/module/az.analysisservices/set-azanalysisservicesserver) , чтобы изменить параметр администратора для существующего сервера.

## <a name="rest-api"></a>REST API

Используйте команду [Create](/rest/api/analysisservices/servers/create) для указания свойства asAdministrator при создании нового сервера. <br>
Используйте команду [Update](/rest/api/analysisservices/servers/update) для указания свойства asAdministrator при изменении существующего сервера. <br>



## <a name="next-steps"></a>Дальнейшие действия 

[Аутентификация и пользовательские разрешения](analysis-services-manage-users.md)  
[Управление ролями и пользователями базы данных](analysis-services-database-users.md)  
[Управление доступом Azure на основе ролей (Azure RBAC)](../role-based-access-control/overview.md)

---
title: Документация по ролям администраторов в Microsoft 365 службах Azure AD | Документация Майкрософт
description: Поиск содержимого и справочников по API для ролей администратора служб Microsoft 365 в Azure Active Directory
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: reference
ms.date: 11/05/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 310dd82f9a5f7f98fbca720b48212b7963e25a63
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102551510"
---
# <a name="roles-for-microsoft-365-services-in-azure-active-directory"></a>Роли для служб Microsoft 365 в Azure Active Directory

Все продукты в Microsoft 365 могут управляться административными ролями в Azure Active Directory (Azure AD). Некоторые продукты также предоставляют дополнительные роли, относящиеся к этому продукту. Сведения о ролях, включенных в каждый продукт, см. в следующей таблице. Общие обсуждения проблем делегирования см. в статье [Делегирование администрирования в Azure Active Directory](concept-delegation.md).

## <a name="where-to-find-content"></a>Где найти содержимое

Служба Microsoft 365 | Содержимое роли | API содержимого
---------------------- | ------------------ | -----------------
Роли администратора в бизнес-планах Office 365 и Microsoft 365 | [Роли администратора Microsoft 365](/office365/admin/add-users/about-admin-roles) | Недоступно
Azure Active Directory (Azure AD) и защита удостоверений Azure Active Directory| [Разрешения роли администратора в Azure Active Directory](permissions-reference.md) | [API Graph](/graph/api/overview)<br>[Получение назначений ролей](/graph/api/directoryrole-list)
Exchange Online| [Общие сведения об управлении доступом на основе ролей](/exchange/understanding-role-based-access-control-exchange-2013-help) |  [Add-ManagementRoleEntry](/powershell/module/exchange/role-based-access-control/add-managementroleentry)<br>[Получение назначений ролей](/powershell/module/exchange/role-based-access-control/get-rolegroup)
SharePoint Online | [Разрешения роли администратора в Azure Active Directory](permissions-reference.md)<br>Также [о роли администратора SharePoint в Microsoft 365](/sharepoint/sharepoint-admin-role) | [API Graph](/graph/api/overview)<br>[Получение назначений ролей](/graph/api/directoryrole-list)
Команды и Skype для бизнеса | [Разрешения роли администратора в Azure Active Directory](permissions-reference.md) | [API Graph](/graph/api/overview)<br>[Получение назначений ролей](/graph/api/directoryrole-list)
Центр безопасности и соответствия требованиям (Office 365 Advanced Threat Protection, Exchange Online Protection, защита информации) | [Роли администраторов в Office 365](/office365/SecurityCompliance/permissions-in-the-security-and-compliance-center) | [PowerShell для Exchange](/powershell/module/exchange/role-based-access-control/add-managementroleentry)<br>[Получение назначений ролей](/powershell/module/exchange/role-based-access-control/get-rolegroup)
Оценка безопасности | [Разрешения роли администратора в Azure Active Directory](permissions-reference.md) | [API Graph](/graph/api/overview)<br>[Получение назначений ролей](/graph/api/directoryrole-list)
Диспетчер соответствия требованиям | [Разрешения и управление доступом на основе ролей](/office365/securitycompliance/meet-data-protection-and-regulatory-reqs-using-microsoft-cloud#permissions-and-role-based-access-control) | Недоступно
Azure Information Protection | [Разрешения роли администратора в Azure Active Directory](permissions-reference.md) | [API Graph](/graph/api/overview)<br>[Получение назначений ролей](/graph/api/directoryrole-list)
Microsoft Cloud App Security | [Управление доступом на основе ролей](/cloud-app-security/manage-admins) | [Справочник по API](/cloud-app-security/api-tokens) 
Расширенная защита от угроз Azure | [Группы ролей Azure ATP](/azure-advanced-threat-protection/atp-role-groups) | Недоступно
Advanced Threat Protection в Защитнике Windows | [Управление доступом к порталу с помощью элемента управления доступом на основе ролей](/windows/security/threat-protection/windows-defender-atp/rbac-windows-defender-advanced-threat-protection) | Недоступно
Управление привилегированными пользователями (PIM) | [Разрешения роли администратора в Azure Active Directory](permissions-reference.md) | [API Graph](/graph/api/overview)<br>[Получение назначений ролей](/graph/api/directoryrole-list)
Intune | [Управление доступом на основе ролей (RBAC) с помощью Microsoft Intune](/intune/role-based-access-control) | [API Graph](/graph/api/resources/intune-rbac-conceptual?view=graph-rest-beta&preserve-view=true)<br>[Получение назначений ролей](/graph/api/intune-rbac-roledefinition-list?view=graph-rest-beta&preserve-view=true)
Управляемый рабочий стол | [Разрешения роли администратора в Azure Active Directory](permissions-reference.md) | [API Graph](/graph/api/overview)<br>[Получение назначений ролей](/graph/api/directoryrole-list)

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотр и назначение ролей администратора в Azure Active Directory](manage-roles-portal.md)
* [Administrator role permissions in Azure Active Directory](permissions-reference.md) (Разрешения ролей администратора в Azure Active Directory)
---
title: PowerShell для ролей Azure AD в PIM — Azure AD | Документация Майкрософт
description: Управление ролями Azure AD с помощью командлетов PowerShell в Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: pim
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/15/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2774c63d34de3de951aa8076b56d203b976158ec
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102548756"
---
# <a name="powershell-for-azure-ad-roles-in-privileged-identity-management"></a>PowerShell для ролей Azure AD в управление привилегированными пользователями

Эта статья содержит инструкции по использованию командлетов PowerShell Azure Active Directory (Azure AD) для управления ролями Azure AD в управление привилегированными пользователями (PIM). Кроме того, в ней содержатся сведения о том, как выполнить настройки модуля Azure AD PowerShell.

> [!Note]
> Официальная версия PowerShell поддерживается, только если вы используете новую версию Azure AD Privileged Identity Management. Перейдите в управление привилегированными пользователями и убедитесь, что в колонке быстрого запуска есть следующий баннер.
> [![Проверка версии управление привилегированными пользователями](media/pim-how-to-add-role-to-user/pim-new-version.png "Выберите Azure AD > управление привилегированными пользователями")](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox) Если у вас нет этого баннера, подождите, пока идет развертывание этого обновленного процесса в течение следующих нескольких недель.
> Командлеты PowerShell управление привилегированными пользователями поддерживаются в модуле предварительной версии Azure AD. Если вы использовали другой модуль и этот модуль теперь возвращает сообщение об ошибке, начните использовать этот новый модуль. Если у вас есть рабочие системы, созданные на основе другого модуля, обратитесь к [pim_preview@microsoft.com](mailto:pim_preview@microsoft.com) .

## <a name="installation-and-setup"></a>Установка и настройка

1. Установка модуля предварительной версии Azure AD

    ```powershell
    Install-module AzureADPreview
    ```

1. Перед продолжением убедитесь, что у вас есть необходимые разрешения роли. Если вы пытаетесь выполнить задачи управления, такие как назначение ролей или изменение параметров роли, убедитесь, что у вас есть роль администратора глобального администратора или привилегированной роли. Если вы только пытаетесь активировать собственное назначение, не требуются разрешения, превышающие разрешения пользователя по умолчанию.

1. Подключитесь к Azure AD.

    ```powershell
    $AzureAdCred = Get-Credential  
    Connect-AzureAD -Credential $AzureAdCred
    ```

1. Найдите идентификатор клиента для вашей организации Azure AD, выбрав **Azure Active Directory**  >  **Свойства**  >  **идентификатор каталога**. В разделе командлеты используйте этот идентификатор всякий раз, когда необходимо указать resourceId.

    ![Найдите идентификатор организации в свойствах для Организации Azure AD.](./media/powershell-for-azure-ad-roles/tenant-id-for-Azure-ad-org.png)

> [!Note]
> В следующих разделах приведены простые примеры, которые помогут вам приступить к работе. Более подробную документацию по следующим командлетам можно найти по адресу [https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview&preserve-view=true#privileged_role_management](/powershell/module/azuread/?view=azureadps-2.0-preview&preserve-view=true#privileged_role_management) . Однако необходимо заменить "Азурересаурцес" в параметре providerID на "Аадролес". Кроме того, необходимо забывать об использовании идентификатора клиента для вашей организации Azure AD в качестве параметра resourceId.

## <a name="retrieving-role-definitions"></a>Получение определений ролей

Используйте следующий командлет, чтобы получить все встроенные и пользовательские роли Azure AD в вашей организации Azure AD. Этот важный шаг позволяет сопоставить имя роли и roleDefinitionId. RoleDefinitionId используется во всех этих командлетах для ссылки на определенную роль.

RoleDefinitionId зависит от вашей организации Azure AD и отличается от roleDefinitionId, возвращенного API управления ролями.

```powershell
Get-AzureADMSPrivilegedRoleDefinition -ProviderId aadRoles -ResourceId 926d99e7-117c-4a6a-8031-0cc481e9da26
```

Результат:

![Получение всех ролей для Организации Azure AD](./media/powershell-for-azure-ad-roles/get-all-roles-result.png)

## <a name="retrieving-role-assignments"></a>Получение назначений ролей

Используйте следующий командлет для получения всех назначений ролей в вашей организации Azure AD.

```powershell
Get-AzureADMSPrivilegedRoleAssignment -ProviderId "aadRoles" -ResourceId "926d99e7-117c-4a6a-8031-0cc481e9da26"
```

Используйте следующий командлет для получения всех назначений ролей для конкретного пользователя. Этот список также называется "Мои роли" на портале Azure AD. Единственное отличие заключается в том, что вы добавили фильтр для идентификатора субъекта. ИДЕНТИФИКАТОРом субъекта в этом контексте является идентификатор пользователя или идентификатор группы.

```powershell
Get-AzureADMSPrivilegedRoleAssignment -ProviderId "aadRoles" -ResourceId "926d99e7-117c-4a6a-8031-0cc481e9da26" -Filter "subjectId eq 'f7d1887c-7777-4ba3-ba3d-974488524a9d'" 
```

Используйте следующий командлет, чтобы получить все назначения ролей для конкретной роли. RoleDefinitionId здесь является ИДЕНТИФИКАТОРом, возвращаемым предыдущим командлетом.

```powershell
Get-AzureADMSPrivilegedRoleAssignment -ProviderId "aadRoles" -ResourceId "926d99e7-117c-4a6a-8031-0cc481e9da26" -Filter "roleDefinitionId eq '0bb54a22-a3df-4592-9dc7-9e1418f0f61c'"
```

Командлеты приводят к отображению списка объектов назначения ролей, показанных ниже. ИДЕНТИФИКАТОР субъекта — это идентификатор пользователя, которому назначена роль. Состояние назначения может быть либо активным, либо допустимым. Если пользователь активен и в поле Линкеделигиблеролеассигнментид есть идентификатор, это означает, что роль в данный момент активирована.

Результат:

![Получение всех назначений ролей для Организации Azure AD](./media/powershell-for-azure-ad-roles/get-all-role-assignments-result.png)

## <a name="assign-a-role"></a>Назначение роли

Используйте следующий командлет для создания подходящего назначения.

```powershell
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId 'aadRoles' -ResourceId '926d99e7-117c-4a6a-8031-0cc481e9da26' -RoleDefinitionId 'ff690580-d1c6-42b1-8272-c029ded94dec' -SubjectId 'f7d1887c-7777-4ba3-ba3d-974488524a9d' -Type 'adminAdd' -AssignmentState 'Eligible' -schedule $schedule -reason "dsasdsas" 
```

Расписание, определяющее время начала и окончания назначения, — это объект, который можно создать, как в следующем примере:

```powershell
$schedule = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedSchedule
$schedule.Type = "Once"
$schedule.StartDateTime = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
$schedule.endDateTime = "2020-07-25T20:49:11.770Z"
```
> [!Note]
> Если значение Енддатетиме установлено в NULL, оно указывает на постоянное назначение.

## <a name="activate-a-role-assignment"></a>Активация назначения роли

Используйте следующий командлет для активации подходящего назначения.

```powershell
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId 'aadRoles' -ResourceId '926d99e7-117c-4a6a-8031-0cc481e9da26' -RoleDefinitionId 'f55a9a68-f424-41b7-8bee-cee6a442d418' -SubjectId 'f7d1887c-7777-4ba3-ba3d-974488524a9d' -Type 'UserAdd' -AssignmentState 'Active' -schedule $schedule -reason "dsasdsas"
``` 

Этот командлет почти идентичен командлету для создания назначения роли. Основное различие между командлетами заключается в том, что для параметра – Type Activation — «Усерадд» вместо «Админадд». Другое отличие заключается в том, что параметр – Ассигнментстате — «Active» вместо «право».

> [!Note]
> Существует два сценария ограничения для активации ролей с помощью PowerShell.
> 1. Если в параметрах роли требуется номер системы или билета билета, невозможно указать их в качестве параметра. Таким образом, невозможно будет активировать роль за пределами портал Azure. Эта функция создается в PowerShell в течение следующих нескольких месяцев.
> 1. Если требуется многофакторная проверка подлинности для активации роли, то в настоящее время PowerShell не может вынуждать пользователя при активации своей роли. Вместо этого пользователям потребуется активировать запрос MFA при подключении к Azure AD, следуя [этой записи блога](http://www.anujchaudhary.com/2020/02/connect-to-azure-ad-powershell-with-mfa.html) от одного из наших инженеров. Если вы разрабатываете приложение для PIM, одна из возможных реализаций — выдавать пользователям запрос и повторно подключать их к модулю после получения ошибки "Мфаруле".

## <a name="retrieving-and-updating-role-settings"></a>Получение и обновление параметров роли

Используйте следующий командлет, чтобы получить все параметры роли в вашей организации Azure AD.

```powershell
Get-AzureADMSPrivilegedRoleSetting -ProviderId 'aadRoles' -Filter "ResourceId eq '926d99e7-117c-4a6a-8031-0cc481e9da26'"
```

В параметре имеется четыре основных объекта. В настоящее время PIM использует только три из этих объектов. Усермемберсеттингс — это параметры активации, Админелигиблесеттингс — параметры назначения для соответствующих назначений, а Админмемберсеттингс — параметры назначения для активных назначений.

[![Получение и обновление параметров роли.](media/powershell-for-azure-ad-roles/get-update-role-settings-result.png)](media/powershell-for-azure-ad-roles/get-update-role-settings-result.png#lightbox)

Чтобы обновить параметр роли, необходимо получить существующий объект параметров для конкретной роли и внести в него изменения:

```powershell
Get-AzureADMSPrivilegedRoleSetting -ProviderId 'aadRoles' -Filter "ResourceId eq 'tenant id' and RoleDefinitionId eq 'role id'"
$settinga = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedRuleSetting
$settinga.RuleIdentifier = "JustificationRule"
$settinga.Setting = '{"required":false}'
```

Затем можно применить параметр к одному из объектов для конкретной роли, как показано ниже. ИДЕНТИФИКАТОРом является идентификатор параметра роли, который можно получить из результата выполнения командлета List Role Settings.

```powershell
Set-AzureADMSPrivilegedRoleSetting -ProviderId 'aadRoles' -Id 'ff518d09-47f5-45a9-bb32-71916d9aeadf' -ResourceId '3f5887ed-dd6e-4821-8bde-c813ec508cf9' -RoleDefinitionId '2387ced3-4e95-4c36-a915-73d803f93702' -UserMemberSettings $settinga 
```

## <a name="next-steps"></a>Дальнейшие действия

- [Назначение настраиваемой роли Azure AD](azure-ad-custom-roles-assign.md)
- [Update or remove an assigned Azure AD custom role in Privileged Identity Management](azure-ad-custom-roles-update-remove.md) (Обновление или удаление назначенной настраиваемой роли AAD в Privileged Identity Management)
- [Настройка назначения настраиваемой роли Azure AD](azure-ad-custom-roles-configure.md)
- [Administrator role permissions in Azure Active Directory](../roles/permissions-reference.md) (Разрешения роли администратора в Azure Active Directory)

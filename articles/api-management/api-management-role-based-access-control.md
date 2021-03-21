---
title: Как использовать управление доступом на основе ролей в службе управления API Azure | Документация Майкрософт
description: Узнайте, как использовать встроенные роли и создавать пользовательские роли в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 364cd53e-88fb-4301-a093-f132fa1f88f5
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 06/20/2018
ms.author: apimpm
ms.openlocfilehash: fc41d2904d5a6676f1e9e1e2d178985bef0a9aa3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101095544"
---
# <a name="how-to-use-role-based-access-control-in-azure-api-management"></a>Как использовать управление доступом на основе ролей в службе управления API Azure

Служба управления API Azure использует управление доступом на основе ролей Azure (Azure RBAC) для обеспечения точного управления доступом к службам и сущностям управления API (например, к API и политикам). В этой статье содержатся общие сведения о встроенных и пользовательских ролях в службе управления API. Дополнительные сведения об управлении доступом на портале Azure, см. в руководстве по [началу работы с управлением доступом на портале Azure](../role-based-access-control/overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="built-in-roles"></a>Встроенные роли

Сейчас служба управления API предоставляет три встроенные роли, к которым скоро будут добавлены еще две. Эти роли можно назначать в разных областях, включая подписку, группу ресурсов и отдельный экземпляр службы управления API. Например, если пользователю назначена роль "читатель службы управления API" на уровне группы ресурсов, то он будет иметь доступ для чтения ко всем экземплярам управления API внутри группы ресурсов. 

В таблице ниже содержатся краткие описания встроенных ролей. Эти роли можно назначать с помощью портала Azure или других инструментов, включая Azure [PowerShell](../role-based-access-control/role-assignments-powershell.md), [Azure CLI](../role-based-access-control/role-assignments-cli.md) и [REST API](../role-based-access-control/role-assignments-rest.md). Дополнительные сведения о назначении встроенных ролей см. в статье [назначение ролей Azure для управления доступом к ресурсам подписки Azure](../role-based-access-control/role-assignments-portal.md).

| Роль          | Доступ на чтение<sup>[1]</sup> | Доступ на запись<sup>[2]</sup> | Создание, удаление и масштабирование служб, настройка VPN и личных доменов | Доступ к устаревшему порталу издателя | Описание
| ------------- | ---- | ---- | ---- | ---- | ---- 
| Участник службы управления API | ✓ | ✓ | ✓ | ✓ | Суперпользователь. Имеет полный доступ CRUD к службам и сущностям управления API (например, к интерфейсам API и политикам). Имеет доступ к устаревшему порталу издателя. |
| Средство чтения службы управления API | ✓ | | || Имеет доступ на чтение к службам и сущностям управления API. |
| Оператор службы управления API | ✓ | | ✓ | | Может управлять службами управления API, но не может управлять сущностями.|
| Редактор службы управления API<sup>*</sup> | ✓ | ✓ | |  | Может управлять сущностями управления API, но не может управлять службами.|
| Диспетчер содержимого API Management<sup>*</sup> | ✓ | | | ✓ | Может управлять порталом разработчика. Имеет доступ на чтение к службам и сущностям.|

<sup>[1] Полный доступ к службам и сущностям управления API (например, к интерфейсам API и политикам).</sup>

<sup>[2] Доступ на запись к службам и сущностям управления API, за исключением следующих операций: создание, удаление и масштабирование экземпляров; настройка VPN; настройка пользовательского домена.</sup>

<sup>\* Роль редактора служб будет доступна после переноса всего пользовательского интерфейса администратора с существующего портала издателя на портал Azure. Роль Диспетчер содержимого будет доступна после рефакторинга портала издателя, чтобы он содержал только функциональные возможности, связанные с управлением порталом разработчика.</sup>  

## <a name="custom-roles"></a>Пользовательские роли

Если ни одна из встроенных ролей не удовлетворяет вашим потребностям, можно создать пользовательские роли, которые обеспечат более детализированное управление доступом к сущностям управления API. Например, можно создать пользовательскую роль, которая имеет доступ на чтение к службе управления API и доступ на запись к одному конкретному API. Дополнительные сведения о пользовательских ролях см. в описании [пользовательских ролей в Azure RBAC](../role-based-access-control/custom-roles.md). 

> [!NOTE]
> Чтобы иметь возможность видеть экземпляр управления API в портал Azure, пользовательская роль должна включать ```Microsoft.ApiManagement/service/read``` действие.

При создании пользовательской роли проще всего начать с одной из встроенных ролей. Измените атрибуты и добавьте необходимые действия **Actions**, **notActions** или области **AssignableScopes**, а затем сохраните изменения как новую роль. Следующий пример начинается с роли "читатель службы управления API" и создает пользовательскую роль с именем "редактор API калькулятора". Пользовательскую роль можно назначить конкретному API. Следовательно, эта роль получит доступ только к этому API. 

```powershell
$role = Get-AzRoleDefinition "API Management Service Reader Role"
$role.Id = $null
$role.Name = 'Calculator API Contributor'
$role.Description = 'Has read access to Contoso APIM instance and write access to the Calculator API.'
$role.Actions.Add('Microsoft.ApiManagement/service/apis/write')
$role.Actions.Add('Microsoft.ApiManagement/service/apis/*/write')
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add('/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.ApiManagement/service/<service name>/apis/<api ID>')
New-AzRoleDefinition -Role $role
New-AzRoleAssignment -ObjectId <object ID of the user account> -RoleDefinitionName 'Calculator API Contributor' -Scope '/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.ApiManagement/service/<service name>/apis/<api ID>'
```

В статье [Операции поставщиков ресурсов Azure Resource Manager](../role-based-access-control/resource-provider-operations.md#microsoftapimanagement) содержится список разрешений, которые могут быть предоставлены на уровне управления API.

## <a name="video"></a>Видеоролик


> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Role-Based-Access-Control-in-API-Management/player]
>
>

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше об управлении доступом на основе ролей в Azure, см. следующие статьи:
  * [Начало работы с управлением доступом на портале Azure](../role-based-access-control/overview.md)
  * [Назначение ролей Azure для управления доступом к ресурсам подписки Azure](../role-based-access-control/role-assignments-portal.md)
  * [Пользовательские роли в Azure RBAC](../role-based-access-control/custom-roles.md)
  * [Операции поставщика ресурсов Azure Resource Manager](../role-based-access-control/resource-provider-operations.md#microsoftapimanagement)

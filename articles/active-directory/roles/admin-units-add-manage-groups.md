---
title: Добавление, удаление и перечисление групп в административной единице Azure Active Directory | Документация Майкрософт
description: Управляйте группами и разрешениями их ролей в административной единице в Azure Active Directory.
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: roles
ms.workload: identity
ms.date: 03/10/2021
ms.author: rolyon
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: f88a0818d93c33b6265cc8c695479d2a42678ada
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103011040"
---
# <a name="add-and-manage-groups-in-an-administrative-unit-in-azure-active-directory"></a>Добавление групп в административную единицу и управление ими в Azure Active Directory

В Azure Active Directory (Azure AD) можно добавлять группы в административную единицу для более детального управления областью администрирования.

Чтобы подготовиться к использованию PowerShell и Microsoft Graph для управления административными единицами, см. статью Приступая к [работе](admin-units-manage.md#get-started).

## <a name="add-groups-to-an-administrative-unit"></a>Добавление групп в административную единицу

Добавить группы в административную единицу можно с помощью портал Azure, PowerShell или Microsoft Graph.

### <a name="use-the-azure-portal"></a>Использование портала Azure

Административную единицу можно назначить только отдельные группы. Назначение групп в качестве групповой операции не предусмотрено. В портал Azure группу можно назначить административной единице одним из двух способов:

* В области **группы** :

  1. В портал Azure перейдите в **Azure AD**.
  1. Выберите **группы**, а затем выберите группу, которую необходимо назначить административной единице. 
  1. В левой области выберите **административные единицы** , чтобы отобразить список административных единиц, которым назначена данная группа. 

     ![Снимок экрана: ссылка "назначить в административную единицу" в области "административные единицы".](./media/admin-units-add-manage-groups/assign-to-group-1.png)

  1. Выберите **назначить административной единице**.
  1. На правой панели выберите административную единицу.

* В области **административные единицы**  >  **все группы** :

  1. В портал Azure перейдите в **Azure AD**.
  
  1. В левой области выберите **административные единицы**, а затем выберите **все группы**. 
     Все группы, которые уже назначены административной единице, отображаются на правой панели. 

  1. В области **группы** выберите **Добавить**.
    В правой области перечислены все доступные группы в вашей организации Azure AD. 

     ![Снимок экрана кнопки "Добавить" для добавления группы в административную единицу.](./media/admin-units-add-manage-groups/assign-to-admin-unit.png)

  1. Выберите одну или несколько групп для назначения административной единице, а затем нажмите кнопку **выбрать** .

### <a name="use-powershell"></a>Использование PowerShell

В следующем примере используйте `Add-AzureADMSAdministrativeUnitMember` командлет, чтобы добавить группу в административную единицу. Идентификатор объекта административной единицы и идентификатор объекта добавляемой группы берутся в качестве аргументов. Измените выделенный раздел в соответствии с требованиями конкретной среды.


```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
$GroupObj = Get-AzureADGroup -Filter "displayname eq 'TestGroup'"
Add-AzureADMSAdministrativeUnitMember -ObjectId $adminUnitObj.ObjectId -RefObjectId $GroupObj.ObjectId
```

### <a name="use-microsoft-graph"></a>Использование Microsoft Graph

Выполните следующие команды:

Запрос

```http
POST /administrativeUnits/{admin-unit-id}/members/$ref
```

Текст

```http
{
"@odata.id":"https://graph.microsoft.com/v1.0/groups/{group-id}"
}
```

Пример

```http
{
"@odata.id":"https://graph.microsoft.com/v1.0/groups/871d21ab-6b4e-4d56-b257-ba27827628f3"
}
```

## <a name="view-a-list-of-groups-in-an-administrative-unit"></a>Просмотр списка групп в административной единице

### <a name="use-the-azure-portal"></a>Использование портала Azure

1. В портал Azure перейдите в **Azure AD**.

1. На левой панели выберите **административные единицы**, а затем выберите административную единицу, группы которой необходимо просмотреть. По умолчанию на левой панели выбраны **все пользователи** . 

1. На левой панели выберите **группы**. На правой панели отображается список групп, входящих в выбранную административную единицу.

   ![Снимок экрана: панель "группы", отображающая список групп в административной единице.](./media/admin-units-add-manage-groups/list-groups-in-admin-units.png)

### <a name="use-powershell"></a>Использование PowerShell

Чтобы отобразить список всех членов административной единицы, выполните следующую команду: 

```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
Get-AzureADMSAdministrativeUnitMember -ObjectId $adminUnitObj.ObjectId
```

Чтобы отобразить все группы, входящие в административную единицу, используйте следующий фрагмент кода:

```powershell
foreach ($member in (Get-AzureADMSAdministrativeUnitMember -ObjectId $adminUnitObj.ObjectId)) 
{
if($member.ObjectType -eq "Group")
{
Get-AzureADGroup -ObjectId $member.ObjectId
}
}
```

### <a name="use-microsoft-graph"></a>Использование Microsoft Graph

Выполните следующую команду.

Запрос

```http
GET /directory/administrativeUnits/{admin-unit-id}/members/$/microsoft.graph.group
```

Текст

```http
{}
```

## <a name="view-a-list-of-administrative-units-for-a-group"></a>Просмотр списка административных единиц для группы

### <a name="use-the-azure-portal"></a>Использование портала Azure

1. В портал Azure перейдите в **Azure AD**.

1. На левой панели выберите **группы** , чтобы отобразить список групп.

1. Выберите группу, чтобы открыть ее профиль. 

1. В левой области выберите **административные единицы** , чтобы получить список всех административных единиц, членом которых является группа.

   ![Снимок экрана: панель "административные единицы", в которой отображается список административных единиц, которым назначена группа.](./media/admin-units-add-manage-groups/list-group-au.png)

### <a name="use-powershell"></a>Использование PowerShell

Выполните следующую команду.

```powershell
Get-AzureADMSAdministrativeUnit | where { Get-AzureADMSAdministrativeUnitMember -ObjectId $_.ObjectId | where {$_.ObjectId -eq $groupObjId} }
```

### <a name="use-microsoft-graph"></a>Использование Microsoft Graph

Выполните следующую команду.

```http
https://graph.microsoft.com/v1.0/groups/{group-id}/memberOf/$/Microsoft.Graph.AdministrativeUnit
```

## <a name="remove-a-group-from-an-administrative-unit"></a>Удаление группы из административной единицы

### <a name="use-the-azure-portal"></a>Использование портала Azure

Удалить группу из административной единицы можно в портал Azure одним из двух способов:

- Удалите его из группы Обзор:

  1. В портал Azure перейдите в **Azure AD**.
  1. На левой панели выберите **группы**, а затем откройте профиль для группы, которую необходимо удалить из административной единицы.
  1. В левой области выберите **административные единицы** , чтобы получить список всех административных единиц, которым назначена данная группа. 
  1. Выберите административную единицу, из которой необходимо удалить группу, а затем нажмите **Удалить из административной единицы**.

     ![Снимок экрана: панель "административные единицы", в которой отображается список групп, назначенных выбранной административной единице.](./media/admin-units-add-manage-groups/group-au-remove.png)

- Удалите его из административной единицы:

  1. В портал Azure перейдите в **Azure AD**.
  1. На левой панели выберите **административные единицы**, а затем выберите административную единицу, которой назначена группа.
  1. В области слева выберите **группы** , чтобы вывести список всех групп, назначенных административной единице.
  1. Выберите группу, которую необходимо удалить, а затем выберите **удалить группы**.

    ![Снимок экрана: панель "группы", отображающая список групп в административной единице.](./media/admin-units-add-manage-groups/list-groups-in-admin-units.png)

### <a name="use-powershell"></a>Использование PowerShell

Выполните следующую команду.

```powershell
Remove-AzureADMSAdministrativeUnitMember -ObjectId $adminUnitId -MemberId $memberGroupObjId
```

### <a name="use-microsoft-graph"></a>Использование Microsoft Graph

Выполните следующую команду.

```http
https://graph.microsoft.com/v1.0/directory/administrativeUnits/{admin-unit-id}/members/{group-id}/$ref
```

## <a name="next-steps"></a>Дальнейшие действия

- [Назначение роли административной единице](admin-units-assign-roles.md)
- [Управление пользователями в административной единице](admin-units-add-manage-users.md)

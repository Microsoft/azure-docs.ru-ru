---
title: Настройка разрешений для восстановления учетной записи Azure Cosmos DB.
description: Узнайте, как изолировать и ограничить разрешения на восстановление для учетной записи непрерывного резервного копирования определенной роли или субъекту. В нем показано, как назначить встроенную роль с помощью портал Azure, интерфейса командной строки или определить пользовательскую роль.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 02/01/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 8b3ce2c195dc2fa3dd703306e731aa5b807b78b1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100648609"
---
# <a name="manage-permissions-to-restore-an-azure-cosmos-db-account"></a>Управление разрешениями для восстановления учетной записи Azure Cosmos DB
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> Функция восстановления на момент времени (режим непрерывного резервного копирования) для Azure Cosmos DB в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Cosmos DB позволяет изолировать и ограничить разрешения на восстановление для учетной записи непрерывного резервного копирования (Предварительная версия) для определенной роли или участника. Владелец учетной записи может активировать восстановление и назначить роль другим участникам для выполнения операции восстановления. Эти разрешения могут быть применены к области действия подписки или более детально в области исходной учетной записи, как показано на следующем рисунке:

:::image type="content" source="./media/continuous-backup-restore-permissions/restore-roles-permissions.png" alt-text="Список ролей, необходимых для выполнения операции восстановления." lightbox="./media/continuous-backup-restore-permissions/restore-roles-permissions.png" border="false":::

Область — это набор ресурсов с доступом. Дополнительные сведения об областях см. в документации по [Azure RBAC](../role-based-access-control/scope-overview.md) . В Azure Cosmos DB применимые области — это исходная подписка и учетная запись базы данных для большинства вариантов использования. Участник, выполняющий действия по восстановлению, должен иметь разрешения на запись в целевую группу ресурсов.

## <a name="assign-roles-for-restore-using-the-azure-portal"></a>Назначение ролей для восстановления с помощью портал Azure

Для выполнения восстановления пользователю или участнику требуется разрешение на восстановление (то есть разрешение *RESTORE/Action* ) и разрешение на предоставление новой учетной записи (разрешение на *запись* ).  Чтобы предоставить эти разрешения, владелец может назначить `CosmosRestoreOperator` `Cosmos DB Operator` участнику и встроенные роли.

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Перейдите к своей подписке и перейдите на вкладку **Управление доступом (IAM)** и выберите **Добавить**  >  **Добавление назначения роли** .

1. В области **Добавление назначения ролей** в поле **роль** выберите роль **космосрестореоператор** . Выберите **пользователь, группа или субъект-служба** для поля **назначить доступ к** и найдите имя пользователя или идентификатор электронной почты, как показано на следующем рисунке:

   :::image type="content" source="./media/continuous-backup-restore-permissions/assign-restore-operator-roles.png" alt-text="Назначение ролей операторов Космосрестореоператор и Cosmos DB." border="true":::

1. Выберите **сохранить** , чтобы предоставить разрешение *RESTORE/Action* .

1. Повторите шаг 3 с ролью **оператора Cosmos DB** , чтобы предоставить разрешение на запись. При назначении этой роли из портал Azure она предоставляет разрешение на восстановление всей подписки.

## <a name="permission-scopes"></a>Области разрешений

|Область  |Пример  |
|---------|---------|
|Подписка | /subscriptions/00000000-0000-0000-0000-000000000000 |
|Группа ресурсов | /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-cosmosdb-rg |
|Ресурс учетной записи CosmosDB restorable | /Subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.DocУментдб/Locations/Западная часть США/Ресторабледатабасеаккаунтс/23e99a35-CD36-4df4-9614-f767a03b9995|

Ресурс учетной записи restorable можно извлечь из выходных данных `az cosmosdb restorable-database-account list --name <accountname>` команды в CLI или `Get-AzCosmosDBRestorableDatabaseAccount -DatabaseAccountName <accountname>` командлете в PowerShell. Атрибут Name в выходных данных представляет собой `instanceID` учетную запись restorable. Дополнительные сведения см. в статье [PowerShell](continuous-backup-restore-powershell.md) или [CLI](continuous-backup-restore-command-line.md) .

## <a name="permissions"></a>Разрешения

Для выполнения различных действий, относящихся к восстановлению учетных записей режима непрерывной архивации, требуются следующие разрешения.

|Разрешение  |Влияние  |Минимальная область  |Максимальная область  |
|---------|---------|---------|---------|
|`Microsoft.Resources/deployments/validate/action`, `Microsoft.Resources/deployments/write` | Эти разрешения необходимы для развертывания шаблона ARM для создания восстановленной учетной записи. Чтобы задать эту роль, см. пример разрешения [рестораблеактион](#custom-restorable-action) ниже. | Неприменимо | Неприменимо  |
|`Microsoft.DocumentDB/databaseAccounts/write` | Это разрешение требуется для восстановления учетной записи в группе ресурсов. | Группа ресурсов, в которой создана восстановленная учетная запись. | Подписка, в которой создана восстановленная учетная запись |
|`Microsoft.DocumentDB/locations/restorableDatabaseAccounts/restore/action` |Это разрешение необходимо в области учетной записи базы данных-источника restorable, чтобы разрешить выполнение действий по восстановлению.  | Ресурс *ресторабледатабасеаккаунт* , принадлежащий к восстанавливаемой исходной учетной записи. Это значение также задается `ID` свойством ресурса учетной записи базы данных restorable. Примером учетной записи restorable является */субскриптионс/субскриптионид/провидерс/Microsoft.Docументдб/Locations/RegionName/ресторабледатабасеаккаунтс/<GUID-instanceid>* | Подписка, содержащая учетную запись базы данных restorable. Невозможно выбрать группу ресурсов в качестве области.  |
|`Microsoft.DocumentDB/locations/restorableDatabaseAccounts/read` |Это разрешение необходимо для области учетной записи базы данных-источника restorable, чтобы получить список учетных записей базы данных, которые можно восстановить.  | Ресурс *ресторабледатабасеаккаунт* , принадлежащий к восстанавливаемой исходной учетной записи. Это значение также задается `ID` свойством ресурса учетной записи базы данных restorable. Примером учетной записи restorable является */субскриптионс/субскриптионид/провидерс/Microsoft.Docументдб/Locations/RegionName/ресторабледатабасеаккаунтс/<GUID-instanceid>*| Подписка, содержащая учетную запись базы данных restorable. Невозможно выбрать группу ресурсов в качестве области.  |
|`Microsoft.DocumentDB/locations/restorableDatabaseAccounts/*/read` | Это разрешение необходимо для области учетной записи источника restorable, чтобы разрешить чтение ресурсов restorable, таких как список баз данных и контейнеров для учетной записи restorable.  | Ресурс *ресторабледатабасеаккаунт* , принадлежащий к восстанавливаемой исходной учетной записи. Это значение также задается `ID` свойством ресурса учетной записи базы данных restorable. Примером учетной записи restorable является */субскриптионс/субскриптионид/провидерс/Microsoft.Docументдб/Locations/RegionName/ресторабледатабасеаккаунтс/<GUID-instanceid>*| Подписка, содержащая учетную запись базы данных restorable. Невозможно выбрать группу ресурсов в качестве области. |

## <a name="azure-cli-role-assignment-scenarios-to-restore-at-different-scopes"></a>Azure CLI сценариев назначения ролей для восстановления в разных областях

Роли с разрешением могут быть назначены разным областям, чтобы обеспечить детальный контроль над тем, кто может выполнять операции восстановления в рамках подписки или указанной учетной записи.

### <a name="assign-capability-to-restore-from-any-restorable-account-in-a-subscription"></a>Назначение возможности восстановления из любой учетной записи restorable в подписке

Назначение `CosmosRestoreOperator` встроенной роли на уровне подписки

```azurecli-interactive
az role assignment create --role "CosmosRestoreOperator" --assignee <email> –scope /subscriptions/<subscriptionId>
```

### <a name="assign-capability-to-restore-from-a-specific-account"></a>Назначение возможности восстановления из определенной учетной записи

* Назначение действия записи пользователя для определенной группы ресурсов. Это действие необходимо для создания новой учетной записи в группе ресурсов.

* Назначьте встроенную роль *космосрестореоператор* конкретной учетной записи базы данных restorable, которую необходимо восстановить. В следующей команде область для *ресторабледатабасеаккаунт* извлекается из `ID` свойства в выходных данных `az cosmosdb restorable-database-account` (при использовании интерфейса командной строки) или  `Get-AzCosmosDBRestorableDatabaseAccount` (при использовании PowerShell).

  ```azurecli-interactive
   az role assignment create --role "CosmosRestoreOperator" --assignee <email> –scope <RestorableDatabaseAccount>
  ```

### <a name="assign-capability-to-restore-from-any-source-account-in-a-resource-group"></a>Назначьте возможность восстановления из любой исходной учетной записи в группе ресурсов.
Эта операция в настоящее время не поддерживается.

## <a name="custom-role-creation-for-restore-action-with-cli"></a><a id="custom-restorable-action"></a>Создание настраиваемой роли для действия восстановления с помощью интерфейса командной строки

Владелец подписки может предоставить разрешение на восстановление в любое другое удостоверение Azure AD. Разрешение на восстановление основано на действии: `Microsoft.DocumentDB/locations/restorableDatabaseAccounts/restore/action` и должно быть включено в разрешение на восстановление. Существует встроенная роль с именем *космосрестореоператор* , которая включает эту роль. Можно либо назначить разрешение с помощью этой встроенной роли, либо создать настраиваемую роль.

Рестораблеактион ниже представляет пользовательскую роль. Эту роль необходимо создать явным образом. Следующий шаблон JSON создает пользовательскую роль *рестораблеактион* с разрешением на восстановление:

```json
{
  "assignableScopes": [
    "/subscriptions/23587e98-b6ac-4328-a753-03bcd3c8e744"
  ],
  "description": "Can do a restore request for any Cosmos DB database account with continuous backup",
  "permissions": [
    {
      "actions": [
        "Microsoft.Resources/deployments/validate/action",
        "Microsoft.DocumentDB/databaseAccounts/write",
        "Microsoft.Resources/deployments/write",  
        "Microsoft.DocumentDB/locations/restorableDatabaseAccounts/restore/action",
        "Microsoft.DocumentDB/locations/restorableDatabaseAccounts/read",
        "Microsoft.DocumentDB/locations/restorableDatabaseAccounts/*/read"
        ],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "Name": "RestorableAction",
    "roleType": "CustomRole"
}
```

Затем используйте следующую команду развертывания шаблона, чтобы создать роль с разрешением на восстановление с помощью шаблона ARM:

```azurecli-interactive
az role definition create --role-definition <JSON_Role_Definition_Path>
```

## <a name="next-steps"></a>Дальнейшие действия

* Настройка непрерывного резервного копирования и управление им с помощью [портал Azure](continuous-backup-restore-portal.md), [PowerShell](continuous-backup-restore-powershell.md), [интерфейса командной строки](continuous-backup-restore-command-line.md)или [Azure Resource Manager](continuous-backup-restore-template.md).
* [Модель ресурсов режима непрерывного резервного копирования](continuous-backup-restore-resource-model.md)

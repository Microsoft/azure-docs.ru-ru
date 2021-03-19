---
title: Настройка управления доступом на основе ролей для учетной записи Azure Cosmos DB с помощью Azure AD
description: Узнайте, как настроить управление доступом на основе ролей с помощью Azure Active Directory для учетной записи Azure Cosmos DB
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 03/17/2021
ms.author: thweiss
ms.openlocfilehash: efde86eac3e0830b36eabfc9e80df09daeed9f6f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104586069"
---
# <a name="configure-role-based-access-control-with-azure-active-directory-for-your-azure-cosmos-db-account-preview"></a>Настройка управления доступом на основе ролей с помощью Azure Active Directory для учетной записи Azure Cosmos DB (Предварительная версия)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!IMPORTANT]
> Azure Cosmos DB управление доступом на основе ролей в настоящее время находится на этапе предварительной версии. Эта предварительная версия предоставляется без Соглашение об уровне обслуживания и не рекомендуется для рабочих нагрузок в рабочей среде. Дополнительные сведения см. в статье [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!NOTE]
> Эта статья посвящена управлению доступом на основе ролей для операций с плоскостью данных в Azure Cosmos DB. Если вы используете операции плоскости управления, см. раздел [Управление доступом на основе ролей](role-based-access-control.md) , примененный к статье операции управления плоскостьм.

Azure Cosmos DB предоставляет встроенную систему управления доступом на основе ролей (RBAC), которая позволяет:

- Проверка подлинности запросов к данным с помощью удостоверения Azure Active Directory (Azure AD).
- Авторизация запросов данных с помощью детализированной модели разрешений на основе ролей.

## <a name="concepts"></a>Основные понятия

Azure Cosmos DB плоскость данных RBAC построена на основе концепций, которые обычно находятся в других системах RBAC, таких как [Azure RBAC](../role-based-access-control/overview.md):

- [Модель разрешений](#permission-model) состоит из набора **действий**. Каждое из этих действий сопоставляется с одной или несколькими операциями с базой данных. Некоторые примеры действий: чтение элемента, запись элемента или выполнение запроса.
- Azure Cosmos DB пользователи создают **[определения ролей](#role-definitions)** , содержащие список разрешенных действий.
- Определения ролей назначаются конкретным удостоверениям Azure AD с помощью **[назначений ролей](#role-assignments)**. Назначение роли также определяет область, к которой применяется определение роли. Сейчас в настоящее время три области:
    - Учетная запись Azure Cosmos DB,
    - База данных Azure Cosmos DB,
    - Контейнер Azure Cosmos DB.

  :::image type="content" source="./media/how-to-setup-rbac/concepts.png" alt-text="Понятия RBAC":::

> [!NOTE]
> Azure Cosmos DB RBAC в настоящее время не предоставляет встроенные определения ролей.

## <a name="permission-model"></a><a id="permission-model"></a> Модель разрешений

В следующей таблице перечислены все действия, предоставляемые моделью разрешений.

| Имя | Соответствующие операции с базой данных |
|---|---|
| `Microsoft.DocumentDB/databaseAccounts/readMetadata` | Чтение метаданных учетной записи. Дополнительные сведения см. в разделе [запросы метаданных](#metadata-requests) . |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/create` | Создайте новый элемент. |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/read` | Чтение отдельного элемента по его ИДЕНТИФИКАТОРу и ключу секции (точка-чтение). |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/replace` | Замена существующего элемента. |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/upsert` | "Upsert" элемент, что означает создание его, если он не существует, или замените его, если он существует. |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/delete` | Удаление элемента. |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/executeQuery` | Выполнение [SQL-запроса](sql-query-getting-started.md). |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/readChangeFeed` | Чтение из [веб-канала изменений](read-change-feed.md)контейнера. |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/executeStoredProcedure` | Выполните [хранимую процедуру](stored-procedures-triggers-udfs.md). |
| `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/manageConflicts` | Управление [конфликтами](conflict-resolution-policies.md) для учетных записей регионов с несколькими записями (то есть вывод списка и удаление элементов из веб-канала конфликтов). |

На уровнях *контейнеров* и *элементов* поддерживаются подстановочные знаки:

- `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/*`
- `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/*`

> [!IMPORTANT]
> Эта модель разрешений охватывает только операции базы данных, позволяющие считывать и записывать данные. Он **не** охватывает никаких операций управления, таких как создание контейнеров или изменение их пропускной способности. Для проверки подлинности операций управления с удостоверением AAD используйте [Azure RBAC](role-based-access-control.md) .

### <a name="metadata-requests"></a><a id="metadata-requests"></a> Запросы метаданных

При использовании пакетов SDK для Azure Cosmos DB эти пакеты SDK выдают запросы к метаданным только для чтения во время инициализации и для обслуживания определенных запросов данных. Эти запросы на получение метаданных получают различные сведения о конфигурации, такие как: 

- Глобальная конфигурация учетной записи, включающая регионы Azure, в которых эта учетная запись доступна.
- Ключ секции для контейнеров или их политики индексации.
- Список физических секций, которые делают контейнер и их адреса.

Они *не* выбирали данные, сохраненные в вашей учетной записи.

Чтобы обеспечить наилучшую прозрачность модели разрешений, эти запросы метаданных явным образом попадают в `Microsoft.DocumentDB/databaseAccounts/readMetadata` действие. Это действие должно быть разрешено во всех ситуациях, когда доступ к учетной записи Azure Cosmos DB осуществляется через один из Azure Cosmos DB пакетов SDK. Его можно назначить (с помощью назначения роли) на любом уровне иерархии Azure Cosmos DB (т. е. учетной записи, базе данных или контейнере).

Фактические запросы метаданных, разрешенные `Microsoft.DocumentDB/databaseAccounts/readMetadata` действием, зависят от области, которой назначено действие:

| Область | Запросы, разрешенные действием |
|---|---|
| Учетная запись | — Перечисление баз данных в учетной записи<br>— Для каждой базы данных под учетной записью допустимые действия в области базы данных |
| База данных | -Чтение метаданных базы данных<br>-Перечисление контейнеров в базе данных<br>— Для каждого контейнера в базе данных допустимые действия в области контейнера. |
| Контейнер | -Чтение метаданных контейнера<br>— Вывод списка физических секций в контейнере;<br>— Разрешение адреса каждой физической секции; |

## <a name="create-role-definitions"></a><a id="role-definitions"></a> Создание определений ролей

При создании определения роли необходимо указать:

- Имя учетной записи Azure Cosmos DB.
- Группа ресурсов, содержащая вашу учетную запись.
- Тип определения роли; `CustomRole` поддерживается только в настоящее время.
- Имя определения роли.
- Список [действий](#permission-model) , которые должна разрешить роль.
- Одна или несколько областей, на которые может быть назначено определение роли; поддерживаются следующие области:
    - `/` (уровень учетной записи),
    - `/dbs/<database-name>` (уровень базы данных),
    - `/dbs/<database-name>/colls/<container-name>` (на уровне контейнера).

> [!NOTE]
> Приведенные ниже операции в настоящее время доступны в.
> - Azure PowerShell: [AZ. CosmosDB версии 2.0.1-Preview](https://www.powershellgallery.com/packages/Az.CosmosDB/2.0.1-preview)
> - Azure CLI: [расширение "cosmosdb-Preview" версии 0.4.0](https://github.com/Azure/azure-cli-extensions/tree/master/src/cosmosdb-preview)

### <a name="using-azure-powershell"></a>Использование Azure PowerShell

Создайте роль с именем *миреадонлироле* , которая содержит только действия чтения:

```powershell
$resourceGroupName = "<myResourceGroup>"
$accountName = "<myCosmosAccount>"
New-AzCosmosDBSqlRoleDefinition -AccountName $accountName `
    -ResourceGroupName $resourceGroupName `
    -Type CustomRole -RoleName MyReadOnlyRole `
    -DataAction @( `
        'Microsoft.DocumentDB/databaseAccounts/readMetadata',
        'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/read', `
        'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/executeQuery', `
        'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/readChangeFeed') `
    -AssignableScope "/"
```

Создайте роль с именем *миреадвритероле* , которая содержит все действия:

```powershell
New-AzCosmosDBSqlRoleDefinition -AccountName $accountName `
    -ResourceGroupName $resourceGroupName `
    -Type CustomRole -RoleName MyReadWriteRole `
    -DataAction @( `
        'Microsoft.DocumentDB/databaseAccounts/readMetadata',
        'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/*', `
        'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/*') `
    -AssignableScope "/"
```

Перечислите определения ролей, созданные для выборки идентификаторов:

```powershell
Get-AzCosmosDBSqlRoleDefinition -AccountName $accountName `
    -ResourceGroupName $resourceGroupName
```

```
RoleName         : MyReadWriteRole
Id               : /subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAcc
                   ounts/<myCosmosAccount>/sqlRoleDefinitions/<roleDefinitionId>
Type             : CustomRole
Permissions      : {Microsoft.Azure.Management.CosmosDB.Models.Permission}
AssignableScopes : {/subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAc
                   counts/<myCosmosAccount>}

RoleName         : MyReadOnlyRole
Id               : /subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAcc
                   ounts/<myCosmosAccount>/sqlRoleDefinitions/<roleDefinitionId>
Type             : CustomRole
Permissions      : {Microsoft.Azure.Management.CosmosDB.Models.Permission}
AssignableScopes : {/subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAc
                   counts/<myCosmosAccount>}
```

### <a name="using-the-azure-cli"></a>Использование Azure CLI

Создайте роль с именем *миреадонлироле* , которая содержит только действия чтения:

```json
// role-definition-ro.json
{
    "RoleName": "MyReadOnlyRole",
    "Type": "CustomRole",
    "AssignableScopes": ["/"],
    "Permissions": [{
        "DataActions": [
            "Microsoft.DocumentDB/databaseAccounts/readMetadata",
            "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/read",
            "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/executeQuery",
            "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/readChangeFeed"
        ]
    }]
}
```

```azurecli
resourceGroupName='<myResourceGroup>'
accountName='<myCosmosAccount>'
az cosmosdb sql role definition create --account-name $accountName --resource-group $resourceGroupName --body @role-definition-ro.json
```

Создайте роль с именем *миреадвритероле* , которая содержит все действия:

```json
// role-definition-rw.json
{
    "RoleName": "MyReadWriteRole",
    "Type": "CustomRole",
    "AssignableScopes": ["/"],
    "Permissions": [{
        "DataActions": [
            "Microsoft.DocumentDB/databaseAccounts/readMetadata",
            "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/*",
            "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/*"
        ]
    }]
}
```

```azurecli
az cosmosdb sql role definition create --account-name $accountName --resource-group $resourceGroupName --body @role-definition-rw.json
```

Перечислите определения ролей, созданные для выборки идентификаторов:

```azurecli
az cosmosdb sql role definition list --account-name $accountName --resource-group $resourceGroupName
```

```
[
  {
    "assignableScopes": [
      "/subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAccounts/<myCosmosAccount>"
    ],
    "id": "/subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAccounts/<myCosmosAccount>/sqlRoleDefinitions/<roleDefinitionId>",
    "name": "<roleDefinitionId>",
    "permissions": [
      {
        "dataActions": [
          "Microsoft.DocumentDB/databaseAccounts/readMetadata",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/*",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/*"
        ],
        "notDataActions": []
      }
    ],
    "resourceGroup": "<myResourceGroup>",
    "roleName": "MyReadWriteRole",
    "sqlRoleDefinitionGetResultsType": "CustomRole",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleDefinitions"
  },
  {
    "assignableScopes": [
      "/subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAccounts/<myCosmosAccount>"
    ],
    "id": "/subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDB/databaseAccounts/<myCosmosAccount>/sqlRoleDefinitions/<roleDefinitionId>",
    "name": "<roleDefinitionId>",
    "permissions": [
      {
        "dataActions": [
          "Microsoft.DocumentDB/databaseAccounts/readMetadata",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/read",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/executeQuery",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/readChangeFeed"
        ],
        "notDataActions": []
      }
    ],
    "resourceGroup": "<myResourceGroup>",
    "roleName": "MyReadOnlyRole",
    "sqlRoleDefinitionGetResultsType": "CustomRole",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleDefinitions"
  }
]
```

## <a name="create-role-assignments"></a><a id="role-assignments"></a> Создание назначений ролей

После создания определений ролей их можно связать с удостоверениями AAD. При создании назначения роли необходимо указать:

- Имя учетной записи Azure Cosmos DB.
- Группа ресурсов, содержащая вашу учетную запись.
- Идентификатор назначаемого определения роли.
- ИДЕНТИФИКАТОР субъекта удостоверения, которому должно быть назначено определение роли.
- Область назначения роли; поддерживаются следующие области:
    - `/` (уровень учетной записи)
    - `/dbs/<database-name>` (уровень базы данных)
    - `/dbs/<database-name>/colls/<container-name>` (уровень контейнера)

  Область должна совпадать или быть подобластью одной из назначаемых для определения роли областей.

> [!NOTE]
> Если вы хотите создать назначение роли для субъекта-службы, обязательно используйте **идентификатор объекта** , который находится в разделе **корпоративные приложения** в колонке **Azure Active Directory** портале.

> [!NOTE]
> Приведенные ниже операции в настоящее время доступны в.
> - Azure PowerShell: [AZ. CosmosDB версии 2.0.1-Preview](https://www.powershellgallery.com/packages/Az.CosmosDB/2.0.1-preview)
> - Azure CLI: [расширение "cosmosdb-Preview" версии 0.4.0](https://github.com/Azure/azure-cli-extensions/tree/master/src/cosmosdb-preview)

### <a name="using-azure-powershell"></a>Использование Azure PowerShell

Назначить роль удостоверению:

```powershell
$resourceGroupName = "<myResourceGroup>"
$accountName = "<myCosmosAccount>"
$readOnlyRoleDefinitionId = "<roleDefinitionId>" // as fetched above
$principalId = "<aadPrincipalId>"
New-AzCosmosDBSqlRoleAssignment -AccountName $accountName `
    -ResourceGroupName $resourceGroupName `
    -RoleDefinitionId $readOnlyRoleDefinitionId `
    -Scope $accountName `
    -PrincipalId $principalId
```

### <a name="using-the-azure-cli"></a>Использование Azure CLI

Назначить роль удостоверению:

```azurecli
resourceGroupName='<myResourceGroup>'
accountName='<myCosmosAccount>'
readOnlyRoleDefinitionId = '<roleDefinitionId>' // as fetched above
principalId = '<aadPrincipalId>'
az cosmosdb sql role assignment create --account-name $accountName --resource-group --scope "/" --principal-id $principalId --role-definition-id $readOnlyRoleDefinitionId
```

## <a name="initialize-the-sdk-with-azure-ad"></a>Инициализация пакета SDK с помощью Azure AD

Чтобы использовать Azure Cosmos DB RBAC в приложении, необходимо обновить способ инициализации пакета SDK для Azure Cosmos DB. Вместо того чтобы передавать первичный ключ учетной записи, необходимо передать экземпляр `TokenCredential` класса. Этот экземпляр предоставляет Azure Cosmos DB SDK с контекстом, необходимым для выборки токена AAD от имени удостоверения, которое вы хотите использовать.

Способ создания `TokenCredential` экземпляра выходит за рамки этой статьи. Существует множество способов создания такого экземпляра в зависимости от типа удостоверения AAD, которое вы хотите использовать (субъект-пользователь, субъект-служба, группа и т. д.). Что важнее всего, `TokenCredential` экземпляр должен быть разрешаться в удостоверение (идентификатор участника), которому назначены роли. Примеры создания `TokenCredential` класса:

- [в .NET](https://docs.microsoft.com/dotnet/api/overview/azure/identity-readme#credential-classes)
- [в Java](https://docs.microsoft.com/java/api/overview/azure/identity-readme#credential-classes)
- [в JavaScript](https://docs.microsoft.com/javascript/api/overview/azure/identity-readme#credential-classes)

В приведенных ниже примерах используется субъект-служба с `ClientSecretCredential` экземпляром.

### <a name="in-net"></a>в .NET;

В настоящее время в `preview` версии [пакета SDK v3 для .net](sql-api-sdk-dotnet-standard.md)поддерживается Azure Cosmos DB RBAC.

```csharp
TokenCredential servicePrincipal = new ClientSecretCredential(
    "<azure-ad-tenant-id>",
    "<client-application-id>",
    "<client-application-secret>");
CosmosClient client = new CosmosClient("<account-endpoint>", servicePrincipal);
```

### <a name="in-java"></a>В Java

В настоящее время в [пакете SDK для Java v4](sql-api-sdk-java-v4.md)поддерживается Azure Cosmos DB RBAC.

```java
TokenCredential ServicePrincipal = new ClientSecretCredentialBuilder()
    .authorityHost("https://login.microsoftonline.com")
    .tenantId("<azure-ad-tenant-id>")
    .clientId("<client-application-id>")
    .clientSecret("<client-application-secret>")
    .build();
CosmosAsyncClient Client = new CosmosClientBuilder()
    .endpoint("<account-endpoint>")
    .credential(ServicePrincipal)
    .build();
```

### <a name="in-javascript"></a>В JavaScript

В настоящее время в [пакете SDK v3 для JavaScript](sql-api-sdk-node.md)поддерживается Azure Cosmos DB RBAC.

```javascript
const servicePrincipal = new ClientSecretCredential(
    "<azure-ad-tenant-id>",
    "<client-application-id>",
    "<client-application-secret>");
const client = new CosmosClient({
    "<account-endpoint>",
    aadCredentials: servicePrincipal
});
```

## <a name="auditing-data-requests"></a>Аудит запросов данных

При использовании Azure Cosmos DB RBAC [журналы диагностики](cosmosdb-monitor-resource-logs.md) дополнены сведениями об удостоверениях и авторизации для каждой операции с данными. Это позволяет выполнять подробный аудит и получать удостоверение AAD, используемое для каждого запроса данных, отправленного в учетную запись Azure Cosmos DB.

Эти дополнительные сведения поступают в категорию журнала **DataPlaneRequests** и состоят из двух дополнительных столбцов:

- `aadPrincipalId_g` Показывает идентификатор субъекта для удостоверения AAD, который использовался для проверки подлинности запроса.
- `aadAppliedRoleAssignmentId_g` показывает [назначение роли](#role-assignments) , которое было соблюдаться при авторизации запроса.

## <a name="limits"></a>Ограничения

- Можно создать до 100 определений ролей и 2 000 назначений ролей для каждой учетной записи Azure Cosmos DB.
- Разрешение группы Azure AD в настоящее время не поддерживается для удостоверений, относящихся к более чем 200 группам.
- Маркер Azure AD в настоящее время передается в виде заголовка с каждым отдельным запросом, отправленным в службу Azure Cosmos DB, увеличивая общий размер полезных данных.
- Доступ к данным с помощью Azure AD через [Azure Cosmos DB Explorer](data-explorer.md) пока не поддерживается. Для работы с обозревателем Azure Cosmos DB по-прежнему требуется, чтобы у пользователя был доступ к первичному ключу учетной записи.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="which-azure-cosmos-db-apis-are-supported-by-rbac"></a>Какие интерфейсы API Azure Cosmos DB поддерживаются RBAC?

В настоящее время поддерживается только API SQL.

### <a name="is-it-possible-to-manage-role-definitions-and-role-assignments-from-the-azure-portal"></a>Можно ли управлять определениями и назначениями ролей на портале Azure?

Управление ролями с помощью портала Azure пока не поддерживается.

### <a name="which-sdks-in-azure-cosmos-db-sql-api-support-rbac"></a>Какие пакеты SDK в Azure Cosmos DB поддержки SQL API RBAC?

Пакеты SDK для [.NET v3](sql-api-sdk-dotnet-standard.md) и [Java v4](sql-api-sdk-java-v4.md) в настоящее время поддерживаются.

### <a name="is-the-azure-ad-token-automatically-refreshed-by-the-azure-cosmos-db-sdks-when-it-expires"></a>Будут ли пакеты SDK Azure Cosmos DB автоматически обновлять маркер Azure AD по истечении срока действия?

Да.

### <a name="is-it-possible-to-disable-the-usage-of-the-account-primary-key-when-using-rbac"></a>Можно ли отключить использование первичного ключа учетной записи при использовании RBAC?

Отключение первичного ключа учетной записи сейчас невозможно.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с обзором [безопасного доступа к данным в Cosmos DB](secure-access-to-data.md).
- Дополнительные сведения о [RBAC для управления Azure Cosmos DB](role-based-access-control.md).

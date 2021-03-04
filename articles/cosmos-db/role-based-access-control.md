---
title: Управление доступом на основе ролей в Azure в Azure Cosmos DB
description: Узнайте, как Azure Cosmos DB обеспечивает защиту базы данных с помощью интеграции с Active Directory (Azure RBAC).
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: mjbrown
ms.openlocfilehash: 3a0f0f617e45834f1f205bcb4c8bcbc884b20d22
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102048120"
---
# <a name="azure-role-based-access-control-in-azure-cosmos-db"></a>Управление доступом на основе ролей в Azure в Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

> [!NOTE]
> Эта статья посвящена управлению доступом на основе ролей для операций плоскости управления в Azure Cosmos DB. Если вы используете операции с плоскостью данных, см. раздел [Azure Cosmos DB RBAC](how-to-setup-rbac.md) для управления доступом на основе ролей, примененных к операциям с плоскостью данных.

Azure Cosmos DB предоставляет встроенные возможности управления доступом на основе ролей в Azure (Azure RBAC) для распространенных сценариев управления в Azure Cosmos DB. Пользователь, имеющий профиль в Azure Active Directory, может назначить эти роли Azure пользователям, группам, субъектам-службам или управляемым удостоверениям, чтобы предоставить или запретить доступ к ресурсам и операциям в Azure Cosmos DBных ресурсах. Назначения ролей предоставляются только для доступа к плоскости управления, включая доступ к учетным записям Azure Cosmos, базам данных, контейнерам и предложениям (пропускная способность).

## <a name="built-in-roles"></a>Встроенные роли

Ниже перечислены встроенные роли, поддерживаемые Azure Cosmos DB.

|**Встроенная роль**  |**Описание**  |
|---------|---------|
|[Участник учетной записи DocumentDB](../role-based-access-control/built-in-roles.md#documentdb-account-contributor)|Может управлять учетными записями Azure Cosmos DB|
|[Средство чтения учетных записей Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role)|Позволяет считывать данные учетных записей Azure Cosmos DB.|
|[Оператор резервного копирования Cosmos](../role-based-access-control/built-in-roles.md#cosmosbackupoperator)| Может отправить запрос на восстановление портал Azure для периодической резервной копии базы данных или контейнера. Может изменять интервал и срок хранения резервных копий на портал Azure. Не удается получить доступ к данным или использовать обозреватель данных.  |
| [космосрестореоператор](../role-based-access-control/built-in-roles.md) | Можно выполнить действие восстановления для учетной записи Azure Cosmos DB с режимом непрерывного резервного копирования.|
|[Оператор Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-operator)|Может подготавливать учетные записи, базы данных и контейнеры Azure Cosmos. Не удается получить доступ к данным или использовать обозреватель данных.|

> [!IMPORTANT]
> Поддержка Azure RBAC в Azure Cosmos DB относится только к операциям управления плоскостью. Операции с плоскостью данных защищены с помощью первичных ключей или маркеров ресурсов. Дополнительные сведения см. в разделе [безопасный доступ к данным в Azure Cosmos DB](secure-access-to-data.md)

## <a name="identity-and-access-management-iam"></a>Управление удостоверениями и доступом (IAM)

Панель **управления доступом (IAM)** в портал Azure используется для настройки управления доступом на основе ролей Azure в ресурсах Azure Cosmos. Роли применяются к пользователям, группам, субъектам-службам и управляемым удостоверениям в Active Directory. Для частных пользователей и групп можно использовать встроенные роли или пользовательские роли. На следующем снимке экрана показана интеграция Active Directory (Azure RBAC) с использованием управления доступом (IAM) в портал Azure.

:::image type="content" source="./media/role-based-access-control/database-security-identity-access-management-rbac.png" alt-text="Управление доступом (IAM) в портал Azure, демонстрирующий безопасность базы данных.":::

## <a name="custom-roles"></a>Пользовательские роли

В дополнение к встроенным ролям пользователи также могут создавать [пользовательские роли](../role-based-access-control/custom-roles.md) в Azure и применять эти роли к субъектам-службам во всех подписках в их Active Directory клиенте. Пользовательские роли предоставляют пользователям способ создания определений ролей Azure с помощью настраиваемого набора операций поставщика ресурсов. Чтобы узнать, какие операции доступны для создания пользовательских ролей для Azure Cosmos DB см. [Azure Cosmos DB операции с поставщиками ресурсов](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb) .

> [!TIP]
> Пользовательские роли, которым необходим доступ к данным, хранящимся в Cosmos DB, или использовать обозреватель данных в портал Azure должны иметь `Microsoft.DocumentDB/databaseAccounts/listKeys/*` действие.

## <a name="preventing-changes-from-the-azure-cosmos-db-sdks"></a><a id="prevent-sdk-changes"></a>Предотвращение изменений в пакетах SDK для Azure Cosmos DB

Поставщик ресурсов Azure Cosmos DB можно заблокировать, чтобы предотвратить любые изменения в ресурсах, подключающихся с помощью ключей учетной записи (которые подключаются через пакет SDK Azure Cosmos). Сюда также входят изменения, внесенные из портал Azure. Эта функция может быть предпочтительна для пользователей, которым требуется более высокий уровень контроля и управления для рабочих сред. Предотвращение изменений в пакете SDK также включает такие функции, как блокировки ресурсов и журналы диагностики для операций управления плоскостью. Клиенты, подключающиеся из пакета SDK для Azure Cosmos DB, не смогут изменять свойства для учетных записей, баз данных, контейнеров и пропускной способности Azure Cosmos. Операции, включающие чтение и запись данных в контейнеры Cosmos, не затрагиваются.

Если эта функция включена, изменения в любом ресурсе могут выполняться только от пользователя с правой ролью Azure и Azure Active Directory учетными данными, включая управляемые удостоверения службы.

> [!WARNING]
> Включение этой функции может оказать влияние на приложение. Убедитесь, что вы понимаете последствия, прежде чем включать его.

### <a name="check-list-before-enabling"></a>Проверить список перед включением

Этот параметр предотвратит любые изменения в ресурсе Cosmos с любого клиента, подключающегося с помощью ключей учетной записи, включая любой Cosmos DB SDK, любые средства, подключающиеся через ключи учетной записи или из портал Azure. Чтобы предотвратить возникновение проблем или ошибок в приложениях после включения этой функции, убедитесь, что приложения или портал Azure пользователи выполняют следующие действия перед включением этой функции, включая:

- Любое изменение в учетной записи Cosmos, включая любые свойства или добавление или удаление регионов.

- Создание, удаление дочерних ресурсов, таких как базы данных и контейнеры. Сюда входят ресурсы для других API, таких как Cassandra, MongoDB, Gremlin и табличные ресурсы.

- Обновление пропускной способности для ресурсов уровня базы данных или контейнера.

- Изменение свойств контейнера, включая политику индексов, TTL и уникальные ключи.

- Изменение хранимых процедур, триггеров или определяемых пользователем функций.

Если приложения (или пользователи с помощью портал Azure) выполняют любое из этих действий, их необходимо перенести для выполнения с помощью [шаблонов ARM](./manage-with-templates.md), [PowerShell](manage-with-powershell.md), [Azure CLI](manage-with-cli.md), службы "оставшаяся" или [библиотеки управления Azure](https://github.com/Azure-Samples/cosmos-management-net). Обратите внимание, что Управление Azure доступно на [нескольких языках](/azure/?product=developer-tools#languages-and-tools).

### <a name="set-via-arm-template"></a>Задать с помощью шаблона ARM

Чтобы задать это свойство с помощью шаблона ARM, обновите существующий шаблон или экспортируйте новый шаблон для текущего развертывания, а затем включите в `"disableKeyBasedMetadataWriteAccess": true` свойства `databaseAccounts` ресурсов. Ниже приведен простой пример шаблона Azure Resource Manager с этим параметром свойства.

```json
{
    {
      "type": "Microsoft.DocumentDB/databaseAccounts",
      "name": "[variables('accountName')]",
      "apiVersion": "2020-04-01",
      "location": "[parameters('location')]",
      "kind": "GlobalDocumentDB",
      "properties": {
        "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
        "locations": "[variables('locations')]",
        "databaseAccountOfferType": "Standard",
        "disableKeyBasedMetadataWriteAccess": true
        }
    }
}
```

> [!IMPORTANT]
> Убедитесь, что вы включили другие свойства учетной записи и дочерние ресурсы при редплойинг с этим свойством. Не развертывайте этот шаблон как есть или сбрасывает все свойства учетной записи.

### <a name="set-via-azure-cli"></a>Задать через Azure CLI

Чтобы включить использование Azure CLI, используйте следующую команду:

```azurecli-interactive
az cosmosdb update  --name [CosmosDBAccountName] --resource-group [ResourceGroupName]  --disable-key-based-metadata-write-access true

```

### <a name="set-via-powershell"></a>Настройка с помощью PowerShell

Чтобы включить использование Azure PowerShell, используйте следующую команду:

```azurepowershell-interactive
Update-AzCosmosDBAccount -ResourceGroupName [ResourceGroupName] -Name [CosmosDBAccountName] -DisableKeyBasedMetadataWriteAccess true
```

## <a name="next-steps"></a>Дальнейшие действия

- [Что такое управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md)
- [Настраиваемые роли Azure](../role-based-access-control/custom-roles.md)
- [Операции поставщика ресурсов Azure Cosmos DB](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb)

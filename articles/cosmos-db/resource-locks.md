---
title: Предотвращение удаления или изменения Azure Cosmos DB ресурсов
description: Используйте блокировки ресурсов Azure, чтобы предотвратить удаление или изменение Azure Cosmos DB ресурсов.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/06/2020
ms.author: mjbrown
ms.openlocfilehash: 1c8c766208132aec115e1fbeb15af3a057c3de3e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94636713"
---
# <a name="prevent-azure-cosmos-db-resources-from-being-deleted-or-changed"></a>Предотвращение удаления или изменения Azure Cosmos DB ресурсов
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В качестве администратора может потребоваться заблокировать учетную запись Azure Cosmos, базу данных или контейнер, чтобы предотвратить случайное удаление и изменение критических ресурсов другими пользователями в Организации. Можно установить уровень блокировки CanNotDelete или ReadOnly.

- **CanNotDelete** означает, что авторизованные пользователи смогут читать и изменять ресурс, но не смогут его удалить.
- **ReadOnly** означает, что авторизованные пользователи смогут читать ресурс, но не смогут его удалить или обновить. Применение этой блокировки подобно ограничению авторизованных пользователей с помощью разрешений, предоставляемых для роли Читатель.

## <a name="how-locks-are-applied"></a>Применение блокировок

При применении блокировки к родительской области все ресурсы в этой области наследуют ту же блокировку. Даже ресурсы, добавляемые позже, наследуют блокировку от родительского ресурса. Приоритет имеет наиболее строгая блокировка.

В отличие от управления доступом на основе ролей в Azure, блокировки управления используются для применения ограничений ко всем пользователям и ролям. Дополнительные сведения об Azure RBAC для Azure Cosmos DB см. [в статье Управление доступом на основе ролей Azure в Azure Cosmos DB](role-based-access-control.md).

Блокировки Resource Manager применяются только к операциям, выполняемым в плоскости управления, которая включает в себя операции, передаваемые на https://management.azure.com. Эти блокировки не мешают ресурсам осуществлять свои собственные операции. Ограничиваются изменения ресурсов, но не операции с ними. Например, блокировка ReadOnly в контейнере Cosmos для Azure не позволяет удалить или изменить контейнер. Это не мешает создавать, обновлять и удалять данные в контейнере. Транзакции с данными разрешены, поскольку эти операции не отправляются в https://management.azure.com.

## <a name="manage-locks"></a>Управление блокировками

> [!WARNING]
> Блокировки ресурсов не работают для изменений, внесенных пользователями, которые обращаются к Azure Cosmos DB с помощью ключей учетной записи, если учетная запись Azure Cosmos не была сначала заблокирована путем включения свойства Дисаблекэйбаседметадатавритеакцесс. Следует соблюдать осторожность, прежде чем включать это свойство, чтобы не нарушать работу существующих приложений, которые делают изменения в ресурсах с помощью любого пакета SDK, портал Azure или сторонних средств, подключающихся через ключи учетной записи и изменяя такие ресурсы, как изменение пропускной способности, обновление политик индексов и т. д. Дополнительные сведения и просмотр контрольного списка для обеспечения работоспособности приложений см. в статье [предотвращение изменений в Azure Cosmos DBных](role-based-access-control.md#prevent-sdk-changes) пакетах SDK.

### <a name="powershell"></a>PowerShell

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "my-cosmos-account"
$lockName = "$accountName-Lock"

# First, update the account to prevent changes by anything that connects via account keys
Update-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName -DisableKeyBasedMetadataWriteAccess true

# Create a Delete Lock on an Azure Cosmos account resource and all child resources
New-AzResourceLock `
    -ApiVersion "2020-04-01" `
    -ResourceType "Microsoft.DocumentDB/databaseAccounts" `
    -ResourceGroupName $resourceGroupName `
    -ResourceName $accountName `
    -LockName $lockName `
    -LockLevel "CanNotDelete" # CanNotDelete or ReadOnly
```

### <a name="azure-cli"></a>Azure CLI

```bash
resourceGroupName='myResourceGroup'
accountName='my-cosmos-account'
$lockName="$accountName-Lock"

# First, update the account to prevent changes by anything that connects via account keys
az cosmosdb update  --name $accountName --resource-group $resourceGroupName  --disable-key-based-metadata-write-access true

# Create a Delete Lock on an Azure Cosmos account resource
az lock create --name $lockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/databaseAccount \
    --lock-type 'CanNotDelete' # CanNotDelete or ReadOnly \
    --resource $accountName
```

### <a name="template"></a>Шаблон

При применении блокировки к Azure Cosmos DB ресурсу используйте следующие форматы:

- имя — `{resourceName}/Microsoft.Authorization/{lockName}`;
- тип — `{resourceProviderNamespace}/{resourceType}/providers/locks`.

> [!IMPORTANT]
> При изменении существующей учетной записи Azure Cosmos необходимо включить другие свойства учетной записи и дочерние ресурсы, если редплойинг с этим свойством. Не развертывайте этот шаблон как есть или сбрасывает все свойства учетной записи.

```json
"resources": [
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "name": "[variables('accountName')]",
        "apiVersion": "2020-04-01",
        "kind": "GlobalDocumentDB",
        "location": "[parameters('location')]",
        "properties": {
            "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
            "locations": "[variables('locations')]",
            "databaseAccountOfferType": "Standard",
            "enableAutomaticFailover": "[parameters('automaticFailover')]",
            "disableKeyBasedMetadataWriteAccess": true
        }
    },
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/providers/locks",
        "apiVersion": "2020-04-01",
        "name": "[concat(variables('accountName'), '/Microsoft.Authorization/siteLock')]",
        "dependsOn": [
        "[resourceId('Microsoft.DocumentDB/databaseAccounts', variables('accountName'))]"
        ],
        "properties": {
        "level": "CanNotDelete",
        "notes": "Cosmos account should not be deleted."
        }
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о блокировках Azure Resource Manager](../azure-resource-manager/management/lock-resources.md)

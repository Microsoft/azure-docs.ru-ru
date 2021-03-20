---
title: Шаблоны Resource Manager для API Gremlin Azure Cosmos DB
description: Используйте шаблоны Azure Resource Manager для создания и настройки API Gremlin Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/14/2020
ms.author: mjbrown
ms.openlocfilehash: 2fee06a2ceb9b8062b5150e5716f1ee9abf15cff
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93340638"
---
# <a name="manage-azure-cosmos-db-gremlin-api-resources-using-azure-resource-manager-templates"></a>Управление ресурсами API Gremlin Azure Cosmos DB с использованием шаблонов Azure Resource Manager
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Из этой статьи вы узнаете, как использовать шаблоны Azure Resource Manager для развертывания учетных записей, баз данных и графов Azure Cosmos DB и управления ими.

В этой статье приводятся примеры учетных записей только API Gremlin. Примеры учетных записей других типов API см. в разделах об использовании шаблонов Azure Resource Manager с API Azure Cosmos DB для [Cassandra](templates-samples-cassandra.md), [SQL](templates-samples-sql.md), [MongoDB](templates-samples-mongodb.md) и [Table](templates-samples-table.md).

> [!IMPORTANT]
>
> * Длина имен учетных записей ограничена 44 символами (только строчные буквы).
> * Чтобы изменить значения пропускной способности, повторно разверните шаблон с обновленным значением ЕЗ/с.
> * Одновременно с добавлением или удалением расположений в учетной записи Azure Cosmos нельзя изменять другие свойства. Эти операции должны выполняться отдельно.

Чтобы создать любой из перечисленных ниже ресурсов Azure Cosmos DB, скопируйте следующий пример шаблона в новый JSON-файл. При необходимости можно создать JSON-файл параметров, который будет использоваться при развертывании нескольких экземпляров одного ресурса с разными именами и значениями. Шаблоны Azure Resource Manager можно развертывать множеством способов, в том числе с помощью [портала Azure](../azure-resource-manager/templates/deploy-portal.md), [Azure CLI](../azure-resource-manager/templates/deploy-cli.md), [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) и [GitHub](../azure-resource-manager/templates/deploy-to-azure-button.md).

<a id="create-autoscale"></a>

## <a name="azure-cosmos-db-account-for-gremlin-with-autoscale-provisioned-throughput"></a>Учетная запись Azure Cosmos DB для Gremlin с пропускной способностью, обеспечиваемой автомасштабированием

Этот шаблон используется для создания учетной записи Azure Cosmos для API Gremlin с базой данных и графом с применением автомаштабирования пропускной способности. Этот шаблон также доступен для развертывания одним щелчком из коллекции шаблонов быстрого запуска Azure.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание в Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-gremlin-autoscale%2Fazuredeploy.json)

:::code language="json" source="~/quickstart-templates/101-cosmosdb-gremlin-autoscale/azuredeploy.json":::

<a id="create-manual"></a>

## <a name="azure-cosmos-db-account-for-gremlin-with-standard-provisioned-throughput"></a>Учетная запись Azure Cosmos DB для Gremlin со стандартной пропускной способностью

Этот шаблон используется для создания учетной записи Azure Cosmos для API Gremlin с базой данных и графом со стандартной (регулируемой вручную) пропускной способностью. Этот шаблон также доступен для развертывания одним щелчком из коллекции шаблонов быстрого запуска Azure.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание в Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-gremlin%2Fazuredeploy.json)

:::code language="json" source="~/quickstart-templates/101-cosmosdb-gremlin/azuredeploy.json":::

## <a name="next-steps"></a>Дальнейшие действия

Ниже приведены некоторые дополнительные ресурсы.

* [Документация по Azure Resource Manager](../azure-resource-manager/index.yml)
* [Схема поставщика ресурсов Azure Cosmos DB](/azure/templates/microsoft.documentdb/allversions)
* [Шаблоны быстрого запуска Azure Cosmos DB](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.DocumentDB&pageNumber=1&sort=Popular)
* [Устранение распространенных ошибок при развертывании с помощью Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md)
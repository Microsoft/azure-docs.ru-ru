---
title: Шаблоны Azure Resource Manager для Azure Cosmos DB Core (API SQL)
description: Использование шаблонов Azure Resource Manager для создания и настройки Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/24/2021
ms.author: mjbrown
ms.openlocfilehash: 7163658024d150a7c5d75c3b3ac0b6b6b29cd3cb
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105037314"
---
# <a name="azure-resource-manager-templates-for-azure-cosmos-db"></a>Шаблоны Azure Resource Manager для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье показаны только примеры шаблонов Azure Resource Manager для учетных записей API Core (SQL). Кроме того, можно найти примеры шаблонов для API [Cassandra](templates-samples-cassandra.md), [Gremlin](templates-samples-gremlin.md), [MongoDB](templates-samples-mongodb.md) и [таблиц](templates-samples-table.md).

## <a name="core-sql-api"></a>API Core (SQL)

|**Шаблон**|**Описание**|
|---|---|
|[Создание учетной записи, базы данных и контейнера Azure Cosmos DB c применением автомасштабирования пропускной способности](manage-with-templates.md#create-autoscale) | Этот шаблон используется для создания учетной записи в двух регионах, базы данных и контейнера API Core (SQL) с применением автомасштабирования пропускной способности. |
|[Создание учетной записи, базы данных и контейнера Azure Cosmos DB c аналитическим хранилищем](manage-with-templates.md#create-analytical-store) | Этот шаблон используется для создания учетной записи API Core (SQL) в одном регионе с настроенным контейнером, для которого включена поддержка аналитического срока жизни и возможность использовать для пропускной способности настройку вручную или с помощью автомасштабирования. |
|[Создание учетной записи, базы данных и контейнера Azure Cosmos DB c применением стандартной настройки пропускной способности (вручную)](manage-with-templates.md#create-manual) | Этот шаблон используется для создания учетной записи API Core (SQL) в двух регионах, базы данных и контейнера с применением стандартной настройки пропускной способности. |
|[Создание учетной записи, базы данных и контейнера Azure Cosmos с хранимой процедурой, триггером и UDF](manage-with-templates.md#create-sproc) | Этот шаблон используется для создания учетной записи API Core (SQL) в двух регионах с хранимой процедурой, триггером и UDF для контейнера. |
|[Создание учетной записи Azure Cosmos с удостоверением Azure AD, определениями ролей и назначением ролей](manage-with-templates.md#create-rbac) | Этот шаблон создает основную учетную запись API (SQL) с удостоверением Azure AD, определениями ролей и назначением ролей для субъекта-службы. |
|[Создание частной конечной точки для имеющейся учетной записи Azure Cosmos](how-to-configure-private-endpoints.md#create-a-private-endpoint-by-using-a-resource-manager-template) |  Этот шаблон используется для создания частной конечной точки для имеющейся учетной записи API Azure Cosmos Core (SQL) в существующей виртуальной сети. |
|[Создание учетной записи Azure Cosmos уровня "Бесплатный"](manage-with-templates.md#free-tier) |  Этот шаблон используется для создания учетной записи API Azure Cosmos DB Core (SQL) уровня "Бесплатный". |

Справочную документацию см. на странице [справочника по Azure Resource Manager для Azure Cosmos DB](/azure/templates/microsoft.documentdb/allversions).

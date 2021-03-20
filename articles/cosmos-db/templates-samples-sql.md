---
title: Шаблоны Azure Resource Manager для Azure Cosmos DB Core (API SQL)
description: Использование шаблонов Azure Resource Manager для создания и настройки Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 10/14/2020
ms.author: mjbrown
ms.openlocfilehash: 52fa5957cdbc3ae73ebc4b36e04582741199414a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93340606"
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
|[Создание частной конечной точки для имеющейся учетной записи Azure Cosmos](how-to-configure-private-endpoints.md#create-a-private-endpoint-by-using-a-resource-manager-template) |  Этот шаблон используется для создания частной конечной точки для имеющейся учетной записи API Azure Cosmos Core (SQL) в существующей виртуальной сети. |
|[Создание учетной записи Azure Cosmos уровня "Бесплатный"](manage-with-templates.md#free-tier) |  Этот шаблон используется для создания учетной записи API Azure Cosmos DB Core (SQL) уровня "Бесплатный". |

Справочную документацию см. на странице [справочника по Azure Resource Manager для Azure Cosmos DB](/azure/templates/microsoft.documentdb/allversions).

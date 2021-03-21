---
title: Перемещение учетной записи Azure Cosmos DB в другой регион
description: Узнайте, как переместить учетную запись Azure Cosmos DB в другой регион.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 09/12/2020
ms.author: mjbrown
ms.openlocfilehash: 29b5c5d3cf55cd11fe505c0d9ab9b894dc2ad267
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93342034"
---
# <a name="move-an-azure-cosmos-db-account-to-another-region"></a>Перемещение учетной записи Azure Cosmos DB в другой регион
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описывается, как выполнить одно из следующих действий:

- Перемещение региона, в котором данные реплицируются в Azure Cosmos DB.
- Перенос метаданных и данных учетной записи (Azure Resource Manager) из одного региона в другой.

## <a name="move-data-from-one-region-to-another"></a>Перемещение данных из одного региона в другой

Azure Cosmos DB поддерживает встроенную репликацию данных, поэтому перемещение данных из одного региона в другой очень просто. Это можно сделать с помощью портал Azure, Azure PowerShell или Azure CLI. Он включает следующие шаги:

1. Добавьте новый регион в учетную запись.

    Сведения о добавлении нового региона в учетную запись Azure Cosmos DB см. в разделе [Добавление и удаление регионов в учетной записи Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account).

1. Выполните отработку отказа вручную в новый регион.

    Когда удаляемый регион в настоящее время является регионом записи для учетной записи, необходимо запустить отработку отказа в новом регионе, добавленном на предыдущем шаге. Это операция с нулевым временем простоя. Если вы перемещаете регион чтения в учетную запись с несколькими регионами, этот шаг можно пропустить. 
    
    Чтобы запустить отработку отказа, см. статью [Выполнение перехода на другой ресурс вручную в учетной записи Azure Cosmos](how-to-manage-database-account.md#manual-failover).

1. Удалите исходный регион.

    Сведения об удалении региона из учетной записи Azure Cosmos DB см. в статье [Добавление и удаление регионов из учетной записи Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account).

## <a name="migrate-azure-cosmos-db-account-metadata"></a>Миграция метаданных учетной записи Azure Cosmos DB

Azure Cosmos DB изначально не поддерживает миграцию метаданных учетной записи из одного региона в другой. Чтобы перенести метаданные учетной записи и данные клиента из одного региона в другой, необходимо создать новую учетную запись в нужном регионе, а затем скопировать данные вручную. 

Для миграции практически без простоя для API SQL необходимо использовать [веб-канал изменений](change-feed.md) или средство, которое его использует. Если вы выполняете миграцию API MongoDB, API Cassandra или другого API или можете узнать больше о вариантах переноса данных между учетными записями, см. раздел [Параметры переноса локальных или облачных данных в Azure Cosmos DB](cosmosdb-migrationchoices.md). 

В следующих шагах показано, как перенести учетную запись Azure Cosmos DB для API SQL и его данных из одного региона в другой:

1. Создайте новую учетную запись Azure Cosmos DB в нужном регионе.

    Сведения о создании новой учетной записи с помощью портал Azure, PowerShell или Azure CLI см. в разделе [Создание учетной записи Azure Cosmos DB](how-to-manage-database-account.md#create-an-account).

1. Создайте новую базу данных и контейнер.

    Сведения о создании новой базы данных и контейнера см. в статье [Создание контейнера Azure Cosmos](how-to-create-container.md).

1. Перенос данных с помощью средства Azure Cosmos DB Live Data Migrator.

    Сведения о переносе данных с почти нулевым временем простоя см. в разделе [Azure Cosmos DB Live Data Migrator Tool](https://github.com/Azure-Samples/azure-cosmosdb-live-data-migrator).

1. Обновите строку подключения приложения.

    Когда средство Live Data Migrator продолжает работать, обновите сведения о подключении в новом развертывании приложения. Вы можете получить конечные точки и ключи для приложения из портал Azure.

    :::image type="content" source="./media/secure-access-to-data/nosql-database-security-master-key-portal.png" alt-text="Контроль доступа в портал Azure, демонстрирующий безопасность базы данных NoSQL.":::

1. Перенаправлять запросы в новое приложение.

    После подключения нового приложения к Azure Cosmos DB можно перенаправить клиентские запросы в новое развертывание.

1. Удалите все ресурсы, которые больше не нужны.

    Теперь, когда запросы полностью перенаправляются на новый экземпляр, вы можете удалить старую учетную запись Azure Cosmos DB и средство Live Data Migrator.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения и примеры управления учетной записью Azure Cosmos, а также базами данных и контейнерами см. в следующих статьях:

* [Управление учетной записью Azure Cosmos](how-to-manage-database-account.md)
* [Веб-канал изменений в Azure Cosmos DB](change-feed.md)

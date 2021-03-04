---
title: Настройте частные конечные точки для Azure Cosmos DBного аналитического хранилища.
description: Узнайте, как настроить управляемые частные конечные точки для Azure Cosmos DBного аналитического хранилища, чтобы ограничить доступ к сети.
author: AnithaAdusumilli
ms.service: cosmos-db
ms.topic: how-to
ms.date: 03/02/2021
ms.author: anithaa
ms.openlocfilehash: 2f15b397fbceb9e097d94080ba03fba50a96ed06
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102048511"
---
# <a name="configure-private-endpoints-for-azure-cosmos-db-analytical-store"></a>Настройка частных конечных точек для аналитического хранилища Azure Cosmos DB
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

В этой статье вы узнаете, как настроить управляемые частные конечные точки для Azure Cosmos DBного аналитического хранилища. Если используется хранилище транзакций, см. статью [частные конечные точки в статье о хранилище транзакций](how-to-configure-private-endpoints.md) . Используя управляемые частные конечные точки, можно ограничить сетевой доступ к Azure Cosmos DB аналитического хранилища с помощью управляемой виртуальной сети Azure синапсе. Управляемые частные конечные точки устанавливают закрытую ссылку на Ваше аналитическое хранилище.

## <a name="enable-private-endpoint-for-the-analytical-store"></a>Включить закрытую конечную точку для аналитического хранилища

### <a name="set-up-an-azure-synapse-analytics-workspace-with-a-managed-virtual-network"></a>Настройка рабочей области Azure синапсе Analytics с помощью управляемой виртуальной сети

[Создайте рабочую область в Azure синапсе Analytics с включенным утечка данных.](../synapse-analytics/security/how-to-create-a-workspace-with-data-exfiltration-protection.md) С помощью [защиты данных утечка](../synapse-analytics/security/workspace-data-exfiltration-protection.md)можно гарантировать, что злоумышленники не смогут копировать или передавать данные из ресурсов Azure в расположения за пределами области организации.

При включении защиты утечка данных для рабочей области Azure синапсе Analytics применяются следующие ограничения доступа.

* Если вы используете Azure Spark для Azure синапсе Analytics, доступ разрешен только утвержденным управляемым частным конечным точкам для Azure Cosmos DBного аналитического хранилища.

* Если вы используете несинапсеные пулы SQL Server, вы можете запросить любую учетную запись Azure Cosmos DB с помощью ссылки Azure синапсе. Однако запросы на запись, [создающие внешние таблицы как SELECT (CETAS)](../synapse-analytics/sql/develop-tables-cetas.md) , разрешены только утвержденным конечным точкам управления в виртуальной сети рабочей области.

> [!NOTE]
> Вы не можете изменить управляемую виртуальную сеть и конфигурацию утечка данных после создания рабочей области.

### <a name="add-a-managed-private-endpoint-for-azure-cosmos-db-analytical-store"></a>Добавление управляемой частной конечной точки для Azure Cosmos DBного аналитического хранилища

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В портал Azure перейдите в рабочую область синапсе Analytics и откройте панель **Обзор** .

1. Запустите синапсе Studio, перейдя в область **Начало работы** и выбрав **Открыть в разделе** **Открыть синапсе Studio**.

1. В синапсе Studio откройте вкладку **Управление** .

1. Перейдите к разделу **управляемые частные конечные точки** и выберите **создать** .

   :::image type="content" source="./media/analytical-store-private-endpoints/create-new-private-endpoint.png" alt-text="Создайте новую закрытую конечную точку для аналитического хранилища." border="true":::

1. Выберите тип учетной записи **Azure Cosmos DB (API SQL)** > **продолжить**.

   :::image type="content" source="./media/analytical-store-private-endpoints/select-private-endpoint.png" alt-text="Выберите Azure Cosmos DB API SQL, чтобы создать закрытую конечную точку." border="true":::

1. Заполните **новую форму управляемой частной конечной точки** следующими сведениями:

   * **Name** — имя управляемой частной конечной точки. Это имя невозможно обновить после его создания.
   * **Описание** — введите понятное описание для идентификатора частной конечной точки.
   * **Подписка Azure** . Выберите учетную запись Azure Cosmos DB из списка доступных учетных записей в подписках Azure.
   * **Azure Cosmos DB имя учетной записи** — выберите существующую учетную запись Azure Cosmos DB типа SQL или MongoDB.
   * **Целевой вспомогательный ресурс** — выберите один из следующих вариантов: **аналитическая**: Если вы хотите добавить частную конечную точку для Azure Cosmos DBного аналитического хранилища.
     **SQL** (или **MongoDB**): Если вы хотите добавить конечную точку учетной записи транзакции OLTP или транзакций.

   > [!NOTE]
   > Вы можете добавить частные конечные точки хранилища транзакций и службы аналитики хранилища в одну учетную запись Azure Cosmos DB в рабочей области Azure синапсе Analytics. Если вы хотите выполнять только аналитические запросы, может потребоваться только сопоставленная частная конечная точка аналитики.

   :::image type="content" source="./media/analytical-store-private-endpoints/choose-analytical-private-endpoint.png" alt-text="Выберите аналитический элемент для целевого подресурса." border="true":::

1. После создания перейдите к имени частной конечной точки и выберите **Управление утверждениями в портал Azure**.

1. Перейдите к своей учетной записи Azure Cosmos DB, выберите закрытую конечную точку и нажмите кнопку **утвердить**.

1. Вернитесь в рабочую область синапсе Analytics и нажмите кнопку **Обновить** на панели **управляемые частные конечные точки** . Убедитесь, что частная конечная точка находится в **утвержденном** состоянии.

   :::image type="content" source="./media/analytical-store-private-endpoints/approved-private-endpoint.png" alt-text="Убедитесь, что частная конечная точка утверждена." border="true":::

## <a name="use-apache-spark-for-azure-synapse-analytics"></a>Использование Apache Spark для Azure синапсе Analytics

Если вы создали рабочую область Azure синапсе с включенной защитой утечка данных, исходящий доступ из синапсе Spark к Azure Cosmos DB учетным записям будет заблокирован по умолчанию. Кроме того, если у Azure Cosmos DB уже есть частная конечная точка, синапсе Spark будет блокировать доступ к ней.

Чтобы разрешить доступ к данным Azure Cosmos DB:

* Если вы используете ссылку Azure синапсе для запроса Azure Cosmos DB данных, **добавьте управляемую** личную конечную точку для учетной записи Azure Cosmos DB.

* Если вы используете операции записи или чтения или потоковой передачи в хранилище транзакций, добавьте управляемую конечную точку *SQL* или *MongoDB* для учетной записи Azure Cosmos DB. Кроме того, необходимо задать *connectionMode* в качестве *шлюза* , как показано в следующем фрагменте кода:

  ```python
  # Write a Spark DataFrame into an Azure Cosmos DB container
  # To select a preferred lis of regions in a multi-region account, add .option("spark.cosmos.preferredRegions", "<Region1>, <Region2>")
  
  YOURDATAFRAME.write\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<your-Cosmos-DB-linked-service-name>")\
    .option("spark.cosmos.container","<your-Cosmos-DB-container-name>")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .option("spark.cosmos.connection.mode", "Gateway")\
    .mode('append')\
    .save()
  
  ```

## <a name="using-synapse-serverless-sql-pools"></a>Использование несинапсеных пулов SQL Server

Синапсеные пулы SQL используют возможности нескольких клиентов, которые не развертываются в управляемой виртуальной сети. Если учетная запись Azure Cosmos DB имеет существующую закрытую конечную точку, несинапсеный пул SQL не сможет получить доступ к учетной записи из-за проверки сетевой изоляции в учетной записи Azure Cosmos DB.

Чтобы настроить сетевую изоляцию для этой учетной записи из рабочей области синапсе, выполните следующие действия.

1. Разрешите рабочей области синапсе доступ к учетной записи Azure Cosmos DB, указав `NetworkAclBypassResourceId` параметр в учетной записи.

   **PowerShell**

   ```powershell-interactive
   Update-AzCosmosDBAccount -Name MyCosmosDBDatabaseAccount -ResourceGroupName MyResourceGroup -NetworkAclBypass AzureServices -NetworkAclBypassResourceId "/subscriptions/subId/resourceGroups/rgName/providers/Microsoft.Synapse/workspaces/wsName"
   ```

   **Использование Azure CLI**

   ```azurecli-interactive
   az cosmosdb update --name MyCosmosDBDatabaseAccount --resource-group MyResourceGroup --network-acl-bypass AzureServices --network-acl-bypass-resource-ids "/subscriptions/subId/resourceGroups/rgName/providers/Microsoft.Synapse/workspaces/wsName"
   ```

   > [!NOTE]
   > Учетная запись Azure Cosmos DB и Рабочая область Azure синапсе Analytics должны находиться в одном клиенте Azure Active Directory (AD).

2. Теперь вы можете получить доступ к учетной записи из несерверных пулов SQL с помощью запросов T-SQL через ссылку Azure синапсе. Однако, чтобы обеспечить сетевую изоляцию данных в аналитическом хранилище, необходимо добавить в эту учетную запись собственную управляемую конечную точку с **аналитическим** управлением. В противном случае данные в аналитическом хранилище не будут заблокированы для общего доступа.

> [!IMPORTANT]
> Если вы используете Azure синапсе Link и вам нужна сетевая изоляция данных в аналитическом хранилище, необходимо связать учетную запись Azure Cosmos DB с рабочей областью синапсе с помощью управляемой частной конечной точки **аналитики** .

## <a name="next-steps"></a>Дальнейшие действия

* Приступая к работе с [запросом к аналитическому хранилищу с помощью Azure синапсе Spark](../synapse-analytics/synapse-link/how-to-query-analytical-store-spark.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json)
* Приступая к работе с [запросами к аналитическому хранилищу с помощью ПУЛОВ SQL без сервера Azure синапсе](../synapse-analytics/sql/query-cosmos-db-analytical-store.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json)

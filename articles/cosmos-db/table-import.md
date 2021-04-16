---
title: Перенос существующих данных в учетную запись API таблиц в Azure Cosmos DB
description: Сведения о том, как перенести либо импортировать данные из локальной или облачной среды в учетную запись API таблиц Azure в Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 12/07/2017
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: 77f4c928db695bd4193ad46c93e0efbd16decf29
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106443344"
---
# <a name="migrate-your-data-to-an-azure-cosmos-db-table-api-account"></a>Перенос данных в учетную запись API таблиц в Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

Это руководство содержит инструкции по импорту данных для использования с помощью [API таблицы](table-introduction.md) Azure Cosmos DB. Данные, находящиеся в хранилище таблиц Azure, можно импортировать в API таблиц в Azure Cosmos DB, используя средства переноса данных или средство AzCopy. 

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * импорт данных с помощью средства переноса данных;
> * импорт данных с помощью AzCopy.

## <a name="prerequisites"></a>Предварительные требования

* **Увеличьте пропускную способность.** Продолжительность переноса данных зависит от пропускной способности, настроенной для отдельного контейнера или набора контейнеров. Увеличьте пропускную способность для крупных миграций. После переноса уменьшите пропускную способность для экономии расходов.

* **Создайте ресурсы Azure Cosmos DB.** Перед началом переноса данных создайте все таблицы на портале Azure. Если вы выполняете перенос в учетную запись Azure Cosmos DB, обладающую пропускной способностью уровня базы данных, обязательно укажите ключ раздела при создании таблиц Azure Cosmos DB.

## <a name="data-migration-tool"></a>Инструмент переноса данных

Для импорта существующих данных хранилища таблиц Azure в учетную запись API таблиц можно использовать средство командной строки для переноса данных (dt.exe) в Azure Cosmos DB. 

Для переноса табличных данных выполните следующие действия:

1. Скачайте средство миграции с сайта [GitHub](https://github.com/azure/azure-documentdb-datamigrationtool).
2. Запустите `dt.exe` с аргументами командной строки для вашего сценария. `dt.exe` принимает команду в следующем формате:

   ```bash
    dt.exe [/<option>:<value>] /s:<source-name> [/s.<source-option>:<value>] /t:<target-name> [/t.<target-option>:<value>] 
   ```

Для этой команды поддерживаются следующие параметры.

* **/ErrorLog.** Необязательный параметр. Имя CSV-файла, в который будут перенаправлены ошибки, возникшие при переносе данных.
* **/OverwriteErrorLog.** Необязательный параметр. Разрешает перезапись файла журнала ошибок.
* **/ProgressUpdateInterval.** Необязательный параметр, по умолчанию имеет значение `00:00:01`. Интервал времени для обновления хода выполнения передачи данных на экране.
* **/ErrorDetails.** Необязательный параметр, по умолчанию имеет значение `None`. Позволяет указать, для каких ошибок должны отображаться подробные сведения: `None`, `Critical` или `All`.
* **/EnableCosmosTableLog.** Необязательный параметр. Направляет журнал в учетную запись таблицы Azure Cosmos DB. Если этот параметр установлен, по умолчанию используется строка подключения к конечной учетной записи, если также не указан параметр `/CosmosTableLogConnectionString`. Это удобно в тех случаях, когда несколько экземпляров инструмента выполняются одновременно.
* **/CosmosTableLogConnectionString.** Необязательный параметр. Строка подключения для направления журнала в удаленную учетную запись таблицы Azure Cosmos DB.

### <a name="command-line-source-settings"></a>Параметры источника командной строки

Используйте следующие параметры источника, чтобы определить хранилище таблиц Azure в качестве источника миграции.

* **/s:AzureTable.** Считывает данные из хранилища таблиц.
* **/s.ConnectionString.** Строка подключения к конечной точке таблицы. Ее можно получить на портале Azure.
* **/s.LocationMode.** Необязательный параметр, по умолчанию имеет значение `PrimaryOnly`. Позволяет задать режим расположения, который нужно использовать при подключении к хранилищу таблиц: `PrimaryOnly`, `PrimaryThenSecondary`, `SecondaryOnly`, `SecondaryThenPrimary`.
* **/s.Table.** Имя таблицы Azure.
* **/s.InternalFields.** Задайте значение `All` для миграции таблиц, так как `RowKey` и `PartitionKey` являются обязательными для импорта.
* **/s.Filter.** Необязательный параметр. Применяемая строка фильтра.
* **/s.Projection.** Необязательный параметр. Список столбцов для выбора.

Чтобы получить строку подключения к источнику при импорте из хранилища таблиц, откройте портал Azure. Выберите **Учетные записи хранения** > **Учетная запись** > **Ключи доступа** и скопируйте **строку подключения**.

:::image type="content" source="./media/table-import/storage-table-access-key.png" alt-text="Снимок экрана: выбраны элементы &quot;Учетные записи хранения&quot; > &quot;Учетная запись&quot; > &quot;Ключи доступа&quot; и выделен значок &quot;Копировать&quot;.":::

### <a name="command-line-target-settings"></a>Параметры целевого объекта командной строки

Используйте следующие параметры целевого объекта, чтобы определить API таблиц Azure Cosmos DB в качестве целевого объекта миграции.

* **/t:TableAPIBulk.** Отправляет данные в API таблиц Azure Cosmos DB в пакетном режиме.
* **/t.ConnectionString.** Строка подключения к конечной точке таблицы.
* **/t.TableName.** Задает имя таблицы для записи данных.
* **/t.Overwrite.** Необязательный параметр, по умолчанию имеет значение `false`. Позволяет указать, нужно ли перезаписывать существующие значения.
* **/t.MaxInputBufferSize.** Необязательный параметр, по умолчанию имеет значение `1GB`. Приблизительная оценка байтов входящих данных, которые буферизуются перед записью данных в приемник.
* **/t.Throughput.** Необязательный параметр. Если он не указан, используются значения по умолчанию для службы. Задает пропускную способность для таблицы.
* **/t.MaxBatchSize.** Необязательный параметр, по умолчанию имеет значение `2MB`. Задает размер пакета в байтах.

### <a name="sample-command-source-is-table-storage"></a>Пример команды: источник — хранилище таблиц

Ниже приведен пример команды для импорта данных из хранилища таблиц в API таблиц:

```bash
dt /s:AzureTable /s.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Table storage account name>;AccountKey=<Account Key>;EndpointSuffix=core.windows.net /s.Table:<Table name> /t:TableAPIBulk /t.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Cosmos DB account name>;AccountKey=<Azure Cosmos DB account key>;TableEndpoint=https://<Account name>.table.cosmosdb.azure.com:443 /t.TableName:<Table name> /t.Overwrite
```

## <a name="migrate-data-by-using-azcopy"></a>Перенос данных с помощью AzCopy

С помощью служебной программы командной строки AzCopy можно также перенести данные из хранилища таблиц в API таблиц Azure Cosmos DB. Чтобы использовать AzCopy, сначала экспортируйте данные, как описано в разделе [Экспорт данных из хранилища таблиц](/previous-versions/azure/storage/storage-use-azcopy#export-data-from-table-storage). Затем импортируйте данные в Azure Cosmos DB, как описано в разделе [API таблиц Azure Cosmos DB](/previous-versions/azure/storage/storage-use-azcopy#import-data-into-table-storage).

При импорте в Azure Cosmos DB см. следующий пример. Обратите внимание, что в значении `/Dest` используется `cosmosdb`, а не `core`.

Пример команды импорта:

```bash
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.cosmosdb.windows.net/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как выполнять запросы к данным с помощью API таблиц Azure Cosmos DB. 

> [!div class="nextstepaction"]
>[Как выполнять запросы с помощью SQL в базе данных Azure Cosmos DB](../cosmos-db/tutorial-query-table.md)





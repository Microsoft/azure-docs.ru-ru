---
title: Миграция реляционных данных "один к нескольким" в Azure Cosmos DB API SQL
description: Узнайте, как выполнять комплексную миграцию данных для связей "один к нескольким" в API SQL.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 12/12/2019
ms.author: thvankra
ms.openlocfilehash: 53a3317f38cc22ffa3745f5f0e58cc01a54b825c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93096756"
---
# <a name="migrate-one-to-few-relational-data-into-azure-cosmos-db-sql-api-account"></a>Миграция реляционных данных "один к нескольким" в учетную запись SQL API Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Чтобы выполнить миграцию из реляционной базы данных в Azure Cosmos DB API SQL, может потребоваться внести изменения в модель данных для оптимизации.

Одним из распространенных преобразований является денормализация данных путем встраивания связанных вложенных элементов в один документ JSON. Здесь мы рассмотрим несколько вариантов для этого с помощью фабрики данных Azure или Azure Databricks. Общие рекомендации по моделированию данных для Cosmos DB см. [в статье моделирование данных в Azure Cosmos DB](modeling-data.md).  

## <a name="example-scenario"></a>Пример сценария

Предположим, что в базе данных SQL, Orders и OrderDetails имеются две следующие таблицы:


:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/orders.png" alt-text="Снимок экрана, на котором показаны таблицы Orders и OrderDetails в базе данных SQL." border="false" :::

Мы хотим объединить эту связь "один к нескольким" в один документ JSON во время миграции. Для этого можно создать запрос T-SQL, используя "FOR JSON", как показано ниже:

```sql
SELECT
  o.OrderID,
  o.OrderDate,
  o.FirstName,
  o.LastName,
  o.Address,
  o.City,
  o.State,
  o.PostalCode,
  o.Country,
  o.Phone,
  o.Total,
  (select OrderDetailId, ProductId, UnitPrice, Quantity from OrderDetails od where od.OrderId = o.OrderId for json auto) as OrderDetails
FROM Orders o;
```

Результаты этого запроса будут выглядеть следующим образом: 

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/for-json-query-result.png" alt-text="Сведения о заказе" lightbox="./media/migrate-relational-to-cosmos-sql-api/for-json-query-result.png":::

В идеале вы хотите использовать одно действие копирования фабрики данных Azure (ADF) для запроса данных SQL в качестве источника и записывать выходные данные непосредственно в Azure Cosmos DB приемника в качестве правильных объектов JSON. Сейчас невозможно выполнить необходимое преобразование JSON в одном действии копирования. При попытке скопировать результаты приведенного выше запроса в контейнер API SQL Azure Cosmos DB мы видим поле OrderDetails как строковое свойство документа, а не ожидаемый массив JSON.

Мы можем обойти это текущее ограничение одним из следующих способов:

* **Используйте фабрику данных Azure с двумя действиями копирования**: 
  1. Получение данных в формате JSON из SQL в текстовый файл в промежуточном расположении хранилища BLOB-объектов и 
  2. Загрузка данных из текстового файла JSON в контейнер в Azure Cosmos DB.

* **Используйте Azure Databricks для чтения из SQL и записи в Azure Cosmos DB** — мы представим два варианта.


Рассмотрим эти подходы более подробно:

## <a name="azure-data-factory"></a>Фабрика данных Azure

Несмотря на то, что невозможно внедрить OrderDetails в качестве массива JSON в целевом Cosmos DB документе, можно обойти эту ошибку, используя два отдельных действия копирования.

### <a name="copy-activity-1-sqljsontoblobtext"></a>#1 действия копирования: Склжсонтоблобтекст

Для исходных данных мы используем SQL-запрос для получения результирующего набора в виде одного столбца с одним объектом JSON (представляющим порядок) для каждой строки с помощью SQL Server OPENJSON и возможностей пути JSON:

```sql
SELECT [value] FROM OPENJSON(
  (SELECT
    id = o.OrderID,
    o.OrderDate,
    o.FirstName,
    o.LastName,
    o.Address,
    o.City,
    o.State,
    o.PostalCode,
    o.Country,
    o.Phone,
    o.Total,
    (select OrderDetailId, ProductId, UnitPrice, Quantity from OrderDetails od where od.OrderId = o.OrderId for json auto) as OrderDetails
   FROM Orders o FOR JSON PATH)
)
```

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf1.png" alt-text="Копия ADF":::


Для приемника действия копирования Склжсонтоблобтекст мы выберем «текст с разделителями» и назначим ему определенную папку в хранилище BLOB-объектов Azure с динамически формируемым уникальным именем файла (например, « @concat конвейер ()». RunId, '. JSON ').
Так как наш текстовый файл не является «разделителем», и мы не хотим, чтобы он был проанализирован в отдельные столбцы с помощью запятых и нужно сохранить двойные кавычки ("), мы зададим" разделитель столбцов "на знак табуляции (" \t ") или другой символ, не являющийся в данных, а" символ кавычки "—" без кавычек ".

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf2.png" alt-text="Снимок экрана, на котором выделены параметры разделителя столбцов и кавычек.":::

### <a name="copy-activity-2-blobjsontocosmos"></a>#2 действия копирования: Блобжсонтокосмос

Далее мы изменим конвейер ADF, добавив второе действие копирования, которое ищет в хранилище BLOB-объектов Azure текстовый файл, созданный первым действием. Он обрабатывает его как источник JSON для вставки в Cosmos DB приемника как один документ для каждой строки JSON, найденный в текстовом файле.

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf3.png" alt-text="Снимок экрана, на котором выделены исходный файл JSON и поля пути к файлу.":::

При необходимости мы также добавляем действие "Удалить" в конвейер, чтобы оно удалило все предыдущие файлы, оставшиеся в папке/ордерс/, перед каждым запуском. Наш конвейер ADF теперь выглядит примерно так:

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf4.png" alt-text="Снимок экрана, на котором выделено действие удаления.":::

После запуска конвейера мы видим файл, созданный в нашем промежуточном хранилище BLOB-объектов Azure, содержащем один объект JSON (на строку):

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf5.png" alt-text="Снимок экрана, на котором показан созданный файл, содержащий объекты JSON.":::

Мы также видим документы о заказах с правильно внедренной OrderDetails, вставленными в нашу коллекцию Cosmos DB:

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/adf6.png" alt-text="Снимок экрана, показывающий сведения о заказе в составе Cosmos DB документа":::


## <a name="azure-databricks"></a>Azure Databricks

Кроме того, Spark можно использовать в [Azure Databricks](https://azure.microsoft.com/services/databricks/) , чтобы скопировать данные из источника базы данных SQL в Azure Cosmos DB назначения, не создавая промежуточные файлы Text/JSON в хранилище BLOB-объектов Azure. 

> [!NOTE]
> Для ясности и простоты фрагменты кода ниже содержат явно встроенные пароли базы данных, но всегда следует использовать Azure Databricks секреты.
>

Сначала мы создадим и подключим необходимые [соединители SQL](https://docs.databricks.com/data/data-sources/sql-databases-azure.html) и [Azure Cosmos DB библиотеки соединителей](https://docs.databricks.com/data/data-sources/azure/cosmosdb-connector.html) к Azure Databricksному кластеру. Перезапустите кластер, чтобы убедиться, что библиотеки загружены.

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks1.png" alt-text="Снимок экрана, на котором показано, где создать и подключить требуемый соединитель SQL и библиотеки соединителей для Azure Cosmos DB к Azure Databricksному кластеру.":::

Далее мы представим два примера для Scala и Python. 

### <a name="scala"></a>Scala
Здесь мы получаем результаты SQL-запроса с выходом "FOR JSON" в кадр данных:

```scala
// Connect to Azure SQL https://docs.databricks.com/data/data-sources/sql-databases-azure.html
import com.microsoft.azure.sqldb.spark.config.Config
import com.microsoft.azure.sqldb.spark.connect._
val configSql = Config(Map(
  "url"          -> "xxxx.database.windows.net",
  "databaseName" -> "xxxx",
  "queryCustom"  -> "SELECT o.OrderID, o.OrderDate, o.FirstName, o.LastName, o.Address, o.City, o.State, o.PostalCode, o.Country, o.Phone, o.Total, (SELECT OrderDetailId, ProductId, UnitPrice, Quantity FROM OrderDetails od WHERE od.OrderId = o.OrderId FOR JSON AUTO) as OrderDetails FROM Orders o",
  "user"         -> "xxxx", 
  "password"     -> "xxxx" // NOTE: For clarity and simplicity, this example includes secrets explicitely as a string, but you should always use Databricks secrets
))
// Create DataFrame from Azure SQL query
val orders = sqlContext.read.sqlDB(configSql)
display(orders)
```

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks2.png" alt-text="Снимок экрана, показывающий выходные данные SQL-запроса в кадре данных.":::

Далее мы подключимся к нашей Cosmos DB базе данных и коллекции:

```scala
// Connect to Cosmos DB https://docs.databricks.com/data/data-sources/azure/cosmosdb-connector.html
import org.joda.time._
import org.joda.time.format._
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark.CosmosDBSpark
import com.microsoft.azure.cosmosdb.spark.config.Config
import org.apache.spark.sql.functions._
import org.joda.time._
import org.joda.time.format._
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark.CosmosDBSpark
import com.microsoft.azure.cosmosdb.spark.config.Config
import org.apache.spark.sql.functions._
val configMap = Map(
  "Endpoint" -> "https://xxxx.documents.azure.com:443/",
  // NOTE: For clarity and simplicity, this example includes secrets explicitely as a string, but you should always use Databricks secrets
  "Masterkey" -> "xxxx",
  "Database" -> "StoreDatabase",
  "Collection" -> "Orders")
val configCosmos = Config(configMap)
```

Наконец, мы определим нашу схему и используем from_json, чтобы применить кадр данных перед сохранением в коллекции CosmosDB.

```scala
// Convert DataFrame to proper nested schema
import org.apache.spark.sql.types._
val orderDetailsSchema = ArrayType(StructType(Array(
    StructField("OrderDetailId",IntegerType,true),
    StructField("ProductId",IntegerType,true),
    StructField("UnitPrice",DoubleType,true),
    StructField("Quantity",IntegerType,true)
  )))
val ordersWithSchema = orders.select(
  col("OrderId").cast("string").as("id"),
  col("OrderDate").cast("string"),
  col("FirstName").cast("string"),
  col("LastName").cast("string"),
  col("Address").cast("string"),
  col("City").cast("string"),
  col("State").cast("string"),
  col("PostalCode").cast("string"),
  col("Country").cast("string"),
  col("Phone").cast("string"),
  col("Total").cast("double"),
  from_json(col("OrderDetails"), orderDetailsSchema).as("OrderDetails")
)
display(ordersWithSchema)
// Save nested data to Cosmos DB
CosmosDBSpark.save(ordersWithSchema, configCosmos)
```

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks3.png" alt-text="Снимок экрана, в котором выделяется правильный массив для сохранения в коллекцию Cosmos DB.":::


### <a name="python"></a>Python

В качестве альтернативного подхода может потребоваться выполнить преобразования JSON в Spark (если база данных-источник не поддерживает "FOR JSON" или аналогичную операцию) или вы можете использовать параллельные операции для очень большого набора данных. Здесь представлен пример PySpark. Начните с настройки подключений к исходной и целевой базе данных в первой ячейке:

```python
import uuid
import pyspark.sql.functions as F
from pyspark.sql.functions import col
from pyspark.sql.types import StringType,DateType,LongType,IntegerType,TimestampType
 
#JDBC connect details for SQL Server database
jdbcHostname = "jdbcHostname"
jdbcDatabase = "OrdersDB"
jdbcUsername = "jdbcUsername"
jdbcPassword = "jdbcPassword"
jdbcPort = "1433"
 
connectionProperties = {
  "user" : jdbcUsername,
  "password" : jdbcPassword,
  "driver" : "com.microsoft.sqlserver.jdbc.SQLServerDriver"
}
jdbcUrl = "jdbc:sqlserver://{0}:{1};database={2};user={3};password={4}".format(jdbcHostname, jdbcPort, jdbcDatabase, jdbcUsername, jdbcPassword)
 
#Connect details for Target Azure Cosmos DB SQL API account
writeConfig = {
    "Endpoint": "Endpoint",
    "Masterkey": "Masterkey",
    "Database": "OrdersDB",
    "Collection": "Orders",
    "Upsert": "true"
}
```

Затем мы будем запрашивать базу данных-источник (в данном случае SQL Server) как для записей заказа, так и для записи подробных сведений о заказах, помещая результаты в кадры данных Spark. Также будет создан список, содержащий все идентификаторы заказов и пул потоков для параллельных операций:

```python
import json
import ast
import pyspark.sql.functions as F
import uuid
import numpy as np
import pandas as pd
from functools import reduce
from pyspark.sql import SQLContext
from pyspark.sql.types import *
from pyspark.sql import *
from pyspark.sql.functions import exp
from pyspark.sql.functions import col
from pyspark.sql.functions import lit
from pyspark.sql.functions import array
from pyspark.sql.types import *
from multiprocessing.pool import ThreadPool
 
#get all orders
orders = sqlContext.read.jdbc(url=jdbcUrl, table="orders", properties=connectionProperties)
 
#get all order details
orderdetails = sqlContext.read.jdbc(url=jdbcUrl, table="orderdetails", properties=connectionProperties)
 
#get all OrderId values to pass to map function 
orderids = orders.select('OrderId').collect()
 
#create thread pool big enough to process merge of details to orders in parallel
pool = ThreadPool(10)
```

Затем создайте функцию для записи заказов в целевую коллекцию API SQL. Эта функция фильтрует все сведения о заказе для заданного идентификатора заказа, преобразует их в массив JSON и вставляет массив в документ JSON, который будет записан в целевую коллекцию API SQL для этого заказа:

```python
def writeOrder(orderid):
  #filter the order on current value passed from map function
  order = orders.filter(orders['OrderId'] == orderid[0])
  
  #set id to be a uuid
  order = order.withColumn("id", lit(str(uuid.uuid1())))
  
  #add details field to order dataframe
  order = order.withColumn("details", lit(''))
  
  #filter order details dataframe to get details we want to merge into the order document
  orderdetailsgroup = orderdetails.filter(orderdetails['OrderId'] == orderid[0])
  
  #convert dataframe to pandas
  orderpandas = order.toPandas()
  
  #convert the order dataframe to json and remove enclosing brackets
  orderjson = orderpandas.to_json(orient='records', force_ascii=False)
  orderjson = orderjson[1:-1] 
  
  #convert orderjson to a dictionaory so we can set the details element with order details later
  orderjsondata = json.loads(orderjson)
  
  
  #convert orderdetailsgroup dataframe to json, but only if details were returned from the earlier filter
  if (orderdetailsgroup.count() !=0):
    #convert orderdetailsgroup to pandas dataframe to work better with json
    orderdetailsgroup = orderdetailsgroup.toPandas()
    
    #convert orderdetailsgroup to json string
    jsonstring = orderdetailsgroup.to_json(orient='records', force_ascii=False)
    
    #convert jsonstring to dictionary to ensure correct encoding and no corrupt records
    jsonstring = json.loads(jsonstring)
    
    #set details json element in orderjsondata to jsonstring which contains orderdetailsgroup - this merges order details into the order 
    orderjsondata['details'] = jsonstring
 
  #convert dictionary to json
  orderjsondata = json.dumps(orderjsondata)
 
  #read the json into spark dataframe
  df = spark.read.json(sc.parallelize([orderjsondata]))
  
  #write the dataframe (this will be a single order record with merged many-to-one order details) to cosmos db using spark the connector
  #https://docs.microsoft.com/azure/cosmos-db/spark-connector
  df.write.format("com.microsoft.azure.cosmosdb.spark").mode("append").options(**writeConfig).save()
```

Наконец, мы вызываем приведенный выше код с помощью функции Map в пуле потоков для параллельного выполнения, передавая в список идентификаторов заказов, которые мы создали ранее:

```python
#map order details to orders in parallel using the above function
pool.map(writeOrder, orderids)
```
В любом из подходов в конце мы должны правильно сохранить внедренную OrderDetails в каждом документе в Cosmos DB коллекции:

:::image type="content" source="./media/migrate-relational-to-cosmos-sql-api/databricks4.png" alt-text="Databricks":::

## <a name="next-steps"></a>Дальнейшие действия
* Сведения о [моделировании данных в Azure Cosmos DB](./modeling-data.md)
* Сведения [о том, как моделировать и секционировать данные на Azure Cosmos DB](./how-to-model-partition-example.md)
---
title: Анализ схемы с помощью массивов и вложенных структур
description: Анализ массивов и вложенных структур с помощью Apache Spark и SQL
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: spark
ms.date: 06/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: a35cdebe5ff4db562b19dc4ceed069a831af4305
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98220003"
---
# <a name="analyze-complex-data-types-in-azure-synapse-analytics"></a>Анализ сложных типов данных в Azure синапсе Analytics

Эта статья относится к файлам и контейнерам Parquet в [Azure синапсе Link для Azure Cosmos DB](.\synapse-link\how-to-connect-synapse-link-cosmos-db.md). Spark или SQL можно использовать для чтения или преобразования данных со сложными схемами, такими как массивы или вложенные структуры. Следующий пример завершается одним документом, но его можно легко масштабировать в миллиарды документов с помощью Spark или SQL. Код, прилагаемый к этой статье, использует PySpark (Python).

## <a name="use-case"></a>Вариант использования

Сложные типы данных все чаще встречаются и представляют трудности для инженеров по работе с данными. Анализ вложенной схемы и массивов может занимать длительные и сложные запросы SQL. Кроме того, может быть сложно переименовать или привести тип данных «вложенные столбцы». Кроме того, при работе с глубокими вложенными объектами могут возникнуть проблемы с производительностью.

Специалистам по обработке данных необходимо понять, как эффективно обрабатывать сложные типы данных и сделать их доступными для всех. В следующем примере используется Spark в Azure синапсе Analytics для чтения и преобразования объектов в плоскую структуру с помощью кадров данных. Используйте бессерверную модель SQL в Azure синапсе Analytics, чтобы напрямую запрашивать такие объекты и возвращать эти результаты в виде обычной таблицы.

## <a name="what-are-arrays-and-nested-structures"></a>Что такое массивы и вложенные структуры?

Следующий объект взят из [Application Insights](../azure-monitor/app/app-insights-overview.md). В этом объекте есть вложенные структуры и массивы, содержащие вложенные структуры.

```json
{
    "id": "66532691-ab20-11ea-8b1d-936b3ec64e54",
    "context": {
        "data": {
            "eventTime": "2020-06-10T13:43:34.553Z",
            "samplingRate": "100.0",
            "isSynthetic": "false"
        },
        "session": {
            "isFirst": "false",
            "id": "38619c14-7a23-4687-8268-95862c5326b1"
        },
        "custom": {
            "dimensions": [
                {
                    "customerInfo": {
                        "ProfileType": "ExpertUser",
                        "RoomName": "",
                        "CustomerName": "diamond",
                        "UserName": "XXXX@yahoo.com"
                    }
                },
                {
                    "customerInfo": {
                        "ProfileType": "Novice",
                        "RoomName": "",
                        "CustomerName": "topaz",
                        "UserName": "XXXX@outlook.com"
                    }
                }
            ]
        }
    }
}
```

### <a name="schema-example-of-arrays-and-nested-structures"></a>Пример схемы массивов и вложенных структур
При печати схемы фрейма данных объекта ( **DF**) с помощью команды `df.printschema` вы увидите следующее представление:

* Желтый цвет представляет вложенные структуры.
* Зеленый цвет обозначает массив с двумя элементами.

[![Код с выделением желтого и зеленого цвета, показывающее происхождение схемы](./media/how-to-complex-schema/schema-origin.png)](./media/how-to-complex-schema/schema-origin.png#lightbox)

`_rid`, `_ts` и были `_etag` добавлены в систему во время приема документа в Azure Cosmos DB хранилище транзакций.

Предыдущий кадр данных подсчитывает только 5 столбцов и одну строку. После преобразования проверенный кадр данных будет содержать 13 столбцов и 2 строки в табличном формате.

## <a name="flatten-nested-structures-and-explode-arrays"></a>Сведение вложенных структур и развертывание массивов

С помощью Spark в Azure синапсе Analytics можно легко преобразовать вложенные структуры в столбцы и элементы массива в несколько строк. Для реализации выполните следующие действия.

[![Блок-схема, показывающая шаги для преобразований Spark](./media/how-to-complex-schema/spark-transform-steps.png)](./media/how-to-complex-schema/spark-transform-steps.png#lightbox)

### <a name="define-a-function-to-flatten-the-nested-schema"></a>Определение функции для преобразования вложенной схемы в плоскую

Эту функцию можно использовать без изменения. Создайте ячейку в [записной книжке PySpark](quickstart-apache-spark-notebook.md) , используя следующую функцию:

```python
from pyspark.sql.functions import col

def flatten_df(nested_df):
    stack = [((), nested_df)]
    columns = []

    while len(stack) > 0:
        parents, df = stack.pop()

        flat_cols = [
            col(".".join(parents + (c[0],))).alias("_".join(parents + (c[0],)))
            for c in df.dtypes
            if c[1][:6] != "struct"
        ]

        nested_cols = [
            c[0]
            for c in df.dtypes
            if c[1][:6] == "struct"
        ]

        columns.extend(flat_cols)

        for nested_col in nested_cols:
            projected_df = df.select(nested_col + ".*")
            stack.append((parents + (nested_col,), projected_df))

    return nested_df.select(columns)
```

### <a name="use-the-function-to-flatten-the-nested-schema"></a>Использование функции для преобразования вложенной схемы в плоскую

На этом шаге выполняется сведение вложенной схемы кадра данных (**DF**) в новый кадр данных ( `df_flat` ):

```python
from pyspark.sql.types import StringType, StructField, StructType
df_flat = flatten_df(df)
display(df_flat.limit(10))
```

Функция отображения должна возвращать 10 столбцов и 1 строку. Массив и его вложенные элементы все еще существуют.

### <a name="transform-the-array"></a>Преобразование массива

В этом случае массив, `context_custom_dimensions` в кадре данных, преобразуется `df_flat` в новый кадр данных `df_flat_explode` . В следующем коде также определяется, какой столбец следует выбрать:

```python
from pyspark.sql.functions import explode
from pyspark.sql.functions import flatten
from pyspark.sql.functions import arrays_zip
df_flat_explode = df_flat.select("_rid","_ts","id","_etag",explode(df_flat.context_custom_dimensions),"context_session_isFirst","context_session_id","context_data_eventTime","context_data_samplingRate","context_data_isSynthetic")\
.select("_rid","_ts","id","_etag","col.*","context_session_isFirst","context_session_id","context_data_eventTime","context_data_samplingRate","context_data_isSynthetic")
display(df_flat_explode.limit(10))

```

Функция отображения должна возвращать 10 столбцов и 2 строки. Следующим шагом является сведение вложенных схем с помощью функции, определенной на шаге 1.

### <a name="use-the-function-to-flatten-the-nested-schema"></a>Использование функции для преобразования вложенной схемы в плоскую

Наконец, используйте функцию, чтобы выполнить сведение вложенной схемы фрейма данных `df_flat_explode` в новый кадр данных `df_flat_explode_flat` :
```python
df_flat_explode_flat = flatten_df(df_flat_explode)
display(df_flat_explode_flat.limit(10))
```

Функция отображения должна отображать 13 столбцов и 2 строки.

Функция `printSchema` кадра данных `df_flat_explode_flat` возвращает следующий результат:

[![Код, показывающий окончательную схему](./media/how-to-complex-schema/schema-final.png)](./media/how-to-complex-schema/schema-final.png#lightbox)

## <a name="read-arrays-and-nested-structures-directly"></a>Чтение массивов и вложенных структур напрямую

С бессерверной моделью SQL можно запрашивать и создавать представления и таблицы на таких объектах.

Во первых, в зависимости от способа хранения данных пользователи должны использовать следующую таксономию. Все символы, показанные в верхнем регистре, относятся к Вашему варианту использования:

| Массовость | Формат |
| ------ | ------ |
| 'https://ACCOUNTNAME.dfs.core.windows.net/FILESYSTEM/PATH/FINENAME.parquet' |"Parquet" (ADLSg2)|
| Н'ендпоинт = https://ACCOUNTNAME.documents-staging.windows-ppe.net:443/ ; Account = ACCOUNTNAME; база данных = DATABASENAME; коллекция = COLLECTIONNAME; регион = регионтокуери ', секрет = ' йоурсекрет ' |"CosmosDB" (ссылка Azure синапсе)|


Замените каждое поле следующим образом:
* "Ваша операция выше" представляет собой строку подключения к источнику данных, к которому вы подключаетесь.
* ' Ваш тип выше ' — формат, используемый для подключения к источнику.

```sql
select *
FROM
openrowset(
    BULK 'YOUR BULK ABOVE',
    FORMAT='YOUR TYPE ABOVE'
)
with (id varchar(50),
        contextdataeventTime varchar(50) '$.context.data.eventTime',
        contextdatasamplingRate varchar(50) '$.context.data.samplingRate',
        contextdataisSynthetic varchar(50) '$.context.data.isSynthetic',
        contextsessionisFirst varchar(50) '$.context.session.isFirst',
        contextsessionid varchar(50) '$.context.session.id',
        contextcustomdimensions varchar(max) '$.context.custom.dimensions'
) as q 
cross apply openjson (contextcustomdimensions) 
with ( ProfileType varchar(50) '$.customerInfo.ProfileType',
            RoomName varchar(50) '$.customerInfo.RoomName',
            CustomerName varchar(50) '$.customerInfo.CustomerName',
            UserName varchar(50) '$.customerInfo.UserName'
    )
```

Существует два разных типа операций:

- Тип первой операции указывается в следующей строке кода, которая определяет столбец `contextdataeventTime` с именем, который ссылается на вложенный элемент `Context.Data.eventTime` . 
  ```sql
  contextdataeventTime varchar(50) '$.context.data.eventTime'
  ```

  Эта строка определяет столбец с именем `contextdataeventTime` , который ссылается на вложенный элемент `Context>Data>eventTime` .

- Второй тип операции использует `cross apply` для создания новых строк для каждого элемента в массиве. Затем он определяет каждый вложенный объект. 
  ```sql
  cross apply openjson (contextcustomdimensions) 
  with ( ProfileType varchar(50) '$.customerInfo.ProfileType', 
  ```

  Если массив содержит 5 элементов с 4 вложенными структурами, то бессерверная модель SQL возвращает 5 строк и 4 столбца. Бессерверная модель SQL может выполнять запросы на месте, сопоставлять массив в двух строках и отображать все вложенные структуры в столбцах.

## <a name="next-steps"></a>Дальнейшие действия

* [Узнайте, как запросить ссылку синапсе для Azure Cosmos DB с помощью Spark](./synapse-link/how-to-query-analytical-store-spark.md)
* [Запрашивание вложенных типов Parquet](./sql/query-parquet-nested-types.md)
---
title: Синхронизация Apache Spark для определений внешних таблиц в бессерверном пуле SQL
description: Общие сведения о запросе таблиц Spark с помощью бессерверного пула SQL
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: 057a69881b8b407e5d75fa3510ca1c3eb1830bc7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96446497"
---
# <a name="synchronize-apache-spark-for-azure-synapse-external-table-definitions-in-serverless-sql-pool"></a>Синхронизация Apache Spark для определений внешних таблиц Azure Synapse в бессерверном пуле SQL

Бессерверный пул SQL может автоматически синхронизировать метаданные из Apache Spark. Для каждой базы данных, имеющейся в бессерверных пулах Apache Spark, будет создана база данных бессерверного пула SQL. 

Для каждой внешней таблицы Spark, основанной на Parquet и размещенной в службе хранилища Azure, внешняя таблица создается в базе данных бессерверного пула SQL. Таким образом, вы можете завершить работу пулов Spark и по-прежнему запрашивать внешние таблицы Spark из бессерверного пула SQL.

Если таблица секционирована в Spark, файлы в хранилище упорядочиваются по папкам. Бессерверный пул SQL будет использовать для запроса метаданные секции и только целевые папки и файлы.

Синхронизация метаданных автоматически настраивается для каждого бессерверного пула Apache Spark, подготовленного в рабочей области Azure Synapse. Вы можете сразу же начать выполнение запросов ко внешним таблицам Spark.

Каждая внешняя таблица Spark на основе parquet, размещенная в службе хранилища Azure, представлена внешней таблицей в схеме dbo, которая соответствует базе данных бессерверного пула SQL. 

Для запросов к внешним таблицам Spark выполните запрос, нацеленный на внешнюю таблицу [spark_table]. Перед выполнением примера убедитесь, что у вас есть правильный [доступ к учетной записи хранения](develop-storage-files-storage-access-control.md), в которой находятся файлы.

```sql
SELECT * FROM [db].dbo.[spark_table]
```

> [!NOTE]
> Команды добавления, удаления или изменения столбца внешней таблицы Spark не повлияют на внешнюю таблицу бессерверного пула SQL.

## <a name="apache-spark-data-types-to-sql-data-types-mapping"></a>Сопоставление типов данных Apache Spark с типами данных SQL

| Тип данных Spark | Тип данных SQL               |
| --------------- | --------------------------- |
| ByteType        | smallint                    |
| ShortType      | smallint                    |
| IntegerType     | INT                         |
| LongType        | BIGINT                      |
| FloatType       | real                        |
| DoubleType      | FLOAT                       |
| DecimalType     | Decimal                     |
| TimestampType   | datetime2                   |
| DateType        | Дата                        |
| StringType      | varchar(max)\*               |
| BinaryType      | varbinary                   |
| BooleanType     | bit                         |
| ArrayType       | varchar(max)\* (в JSON)\** |
| MapType         | varchar(max)\* (в JSON)\** |
| StructType      | varchar(max)\* (в JSON)\** |

\* Используемые параметры сортировки — Latin1_General_100_BIN2_UTF8.

\** ArrayType, MapType и StructType представлены в виде JSON.



## <a name="next-steps"></a>Дальнейшие действия

Перейдите к статье [Управление доступом к хранилищу](develop-storage-files-storage-access-control.md), чтобы узнать больше о контроле доступа к хранилищу.

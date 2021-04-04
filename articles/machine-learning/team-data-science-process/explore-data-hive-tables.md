---
title: Просмотр данных в таблицах Hive с помощью запросов Hive — Процесс обработки и анализа данных группы
description: Примеры скриптов Hive, которые используются для анализа данных в таблицах Hive в кластере HDInsight Hadoop.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: a7234a8c45c20c64dddb43a52a099aa92f2d297d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "93321118"
---
# <a name="explore-data-in-hive-tables-with-hive-queries"></a>Просмотр данных в таблицах Hive с помощью запросов Hive

В этой статье представлено несколько примеров скриптов Hive, которые используются для анализа данных в таблицах Hive в кластере HDInsight Hadoop.

Эта задача является одним из этапов [процесса обработки и анализа данных группы](overview.md).

## <a name="prerequisites"></a>Предварительные требования
В этой статье предполагается, что вы:

* Создали учетную запись хранения Azure. Инструкции см. в разделе [Создание учетной записи хранения](../../storage/common/storage-account-create.md).
* Подготовили настраиваемый кластер Hadoop с помощью службы HDInsight. Инструкции см. в статье [Настройка кластеров Azure HDInsight Hadoop для процесса обработки и анализа данных группы](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md).
* Отправили данные в таблицы Hive, которые находятся в кластерах Azure HDInsight Hadoop. Если данные не загружены, необходимо предварительно загрузить их в таблицы Hive, воспользовавшись инструкциями из статьи [Создание и загрузка данных в таблицы Hive из хранилища больших двоичных объектов Azure](move-hive-tables.md) .
* Включили удаленный доступ к кластеру. Инструкции можно найти в разделе [Доступ к головному узлу в кластере Hadoop](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md).
* Инструкции по отправке запросов Hive см. в разделе [Отправка запросов Hive](move-hive-tables.md#submit).

## <a name="example-hive-query-scripts-for-data-exploration"></a>Пример сценариев запроса Hive для просмотра данных
1. Узнайте количество наблюдений на раздел `SELECT <partitionfieldname>, count(*) from <databasename>.<tablename> group by <partitionfieldname>;`
2. Узнайте количество наблюдений в день `SELECT to_date(<date_columnname>), count(*) from <databasename>.<tablename> group by to_date(<date_columnname>);`
3. Получение уровней в столбце категорий   
    `SELECT  distinct <column_name> from <databasename>.<tablename>`
4. Узнайте число уровней в сочетании двух столбцов категорий `SELECT <column_a>, <column_b>, count(*) from <databasename>.<tablename> group by <column_a>, <column_b>`
5. Получение распределения для числовых столбцов  
    `SELECT <column_name>, count(*) from <databasename>.<tablename> group by <column_name>`
6. Извлечение записей из двух объединенных таблиц
   
    ```hiveql
    SELECT
        a.<common_columnname1> as <new_name1>,
        a.<common_columnname2> as <new_name2>,
        a.<a_column_name1> as <new_name3>,
        a.<a_column_name2> as <new_name4>,
        b.<b_column_name1> as <new_name5>,
        b.<b_column_name2> as <new_name6>
    FROM
        (
        SELECT <common_columnname1>,
            <common_columnname2>,
            <a_column_name1>,
            <a_column_name2>,
        FROM <databasename>.<tablename1>
        ) a
        join
        (
        SELECT <common_columnname1>,
            <common_columnname2>,
            <b_column_name1>,
            <b_column_name2>,
        FROM <databasename>.<tablename2>
        ) b
        ON a.<common_columnname1>=b.<common_columnname1> and a.<common_columnname2>=b.<common_columnname2>
    ```

## <a name="additional-query-scripts-for-taxi-trip-data-scenarios"></a>Дополнительные сценарии запросов для сценариев данных о поездках такси
Кроме того, в [репозитории GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts) приведены примеры запросов, которые используются для сценариев наподобие [Данные о поездках в такси по Нью-Йорку](https://chriswhong.com/open-data/foil_nyc_taxi/). Для этих запросов уже задана схема данных, и они готовы к отправке и запуску.
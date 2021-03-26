---
title: Перенос стандартного хранилища метаданных Hive во внешнее хранилище метаданных на базе Azure HDInsight
description: Перенос стандартного хранилища метаданных Hive во внешнее хранилище метаданных на базе Azure HDInsight
author: kevxmsft
ms.author: kevx
ms.reviewer: ''
ms.service: hdinsight
ms.topic: how-to
ms.date: 11/4/2020
ms.openlocfilehash: 4a0258d5e448c59baa1cd63e98058fe7116a8485
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105566121"
---
# <a name="migrate-default-hive-metastore-db-to-external-metastore-db"></a>Миграция базы данных хранилище метаданных Hive по умолчанию в внешнюю базу данных хранилище метаданных

В этой статье показано, как перенести метаданные из [хранилище МЕТАДАННЫХ DB по умолчанию](../hdinsight-use-external-metadata-stores.md#default-metastore) для Hive во внешнюю базу данных SQL в HDInsight. 

## <a name="why-migrate-to-external-metastore-db"></a>Зачем выполнять миграцию во внешнюю базу данных хранилище метаданных

* База данных хранилище метаданных DB по умолчанию ограничена базовой КОНФИГУРАЦИей и не может управлять рабочими нагрузками рабочей среды.

* Внешняя хранилище метаданных DB позволяет клиенту горизонтально масштабировать ресурсы для вычислений Hive, добавляя новые кластеры HDInsight, совместно использующие одну и ту же хранилище метаданных базу данных.

* Для миграции с HDInsight 3,6 на 4,0 перед обновлением версии схемы Hive обязательно нужно перенести метаданные в External хранилище метаданных DB. См. статью [Миграция рабочих нагрузок из hdinsight 3,6 в hdinsight 4,0](./apache-hive-migrate-workloads.md).

Так как база данных хранилище метаданных DB по умолчанию имеет ограниченную емкость вычислений, мы рекомендуем использовать низкий уровень использования других заданий в кластере при переносе метаданных.

Исходная и Целевая баз данных должны использовать одну и ту же версию HDInsight и одни и те же учетные записи хранения. При обновлении версий HDInsight с 3,6 до 4,0 сначала выполните действия, описанные в этой статье. Затем выполните официальные действия по обновлению [.](./apache-hive-migrate-workloads.md)

## <a name="prerequisites"></a>Предварительные условия

При использовании [Azure Data Lake Storage 1-го поколения](../overview-data-lake-storage-gen1.md)расположения таблиц Hive, скорее всего, зависят от конфигурации HDFS кластера для Azure Data Lake Storage 1-го поколения. Выполните следующее действие скрипта, чтобы обеспечить перенос этих расположений в другие кластеры. См. раздел [действие сценария в работающем кластере](../hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster).

Действие аналогично замене символических ссылок своими полными путями.

|Свойство | Значение |
|---|---|
|URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/hive-adl-expand-location-v01.sh`|
|Типы узлов|Head|
|Параметры|""|

## <a name="migrate-with-exportimport-using-sqlpackage"></a>Миграция с использованием экспорта и импорта с помощью SqlPackage

Кластер HDInsight An, созданный только после 2020-10-15, поддерживает экспорт/импорт SQL для хранилище метаданных DB по умолчанию для Hive с помощью `sqlpackage` .

1. Установите [SqlPackage](/sql/tools/sqlpackage-download#get-sqlpackage-net-core-for-linux) в кластер.

2. Экспортируйте базу данных хранилище метаданных DB по умолчанию в BACPAC-файл, выполнив следующую команду.

    ```bash
    wget "https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/hive_metastore_tool.py"
    SQLPACKAGE_FILE='/home/sshuser/sqlpackage/sqlpackage'  # replace with sqlpackage location
    TARGET_FILE='hive.bacpac'
    sudo python hive_metastore_tool.py --sqlpackagefile $SQLPACKAGE_FILE --targetfile $TARGET_FILE
    ```

3. Сохраните BACPAC-файл. Ниже приведен параметр.

    ```bash
    hdfs dfs -mkdir -p /bacpacs
    hdfs dfs -put $TARGET_FILE /bacpacs/
    ```

4. Импортируйте BACPAC-файл в новую базу данных с помощью шагов, перечисленных [здесь](../../azure-sql/database/database-import.md).

5. Новая база данных готова к настройке в [качестве внешней хранилище МЕТАДАННЫХ DB в новом кластере HDInsight](../hdinsight-use-external-metadata-stores.md#select-a-custom-metastore-during-cluster-creation).

## <a name="migrate-using-hive-script"></a>Миграция с помощью скрипта Hive

Кластеры, созданные до 2020-10-15, не поддерживают экспорт и импорт базы данных хранилище метаданных DB по умолчанию.

Для таких кластеров следуйте инструкциям по [копированию таблиц Hive в учетных записях хранения](./hive-migration-across-storage-accounts.md), используя второй кластер с [внешней хранилище метаданных Hive DB](../hdinsight-use-external-metadata-stores.md#select-a-custom-metastore-during-cluster-creation). Второй кластер может использовать ту же учетную запись хранения, но должен использовать новую файловую систему по умолчанию.

### <a name="option-to-shallow-copy"></a>Вариант "неполная" копия
Использование хранилища будет вдвое выше, если таблицы скопированы с помощью приведенного выше руководством. Необходимо вручную очистить данные в исходном контейнере хранилища.
Вместо этого можно «неполная» копирование таблиц, если они не являются транзакционными. Все таблицы Hive в HDInsight 3,6 по умолчанию не являются транзакционными, но в HDInsight 4,0 не поддерживаются транзакции только для внешних таблиц. Транзакционные таблицы должны быть глубоко скопированы. Выполните следующие действия для неполной копии нетранзакционных таблиц:

1. Выполните сценарий [Hive-ddls.sh](https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/hive-ddls.sh) на основном головного узла исходного кластера, чтобы создать DDL для каждой таблицы Hive.
2. DDL записывается в локальный скрипт Hive с именем `/tmp/hdi_hive_ddls.hql` . Выполните это в целевом кластере, который использует внешнюю базу данных хранилище метаданных Hive DB.

## <a name="verify-that-all-hive-tables-are-imported"></a>Проверка импорта всех таблиц Hive

Следующая команда использует SQL-запрос в базе данных хранилище метаданных DB для печати всех таблиц Hive и их расположений. Сравните выходные данные между новыми и старыми кластерами, чтобы убедиться в отсутствии таблиц в новой базе данных хранилище метаданных DB.

```bash
SCRIPT_FNAME='hive_metastore_tool.py'
SCRIPT="/tmp/$SCRIPT_FNAME"
wget -O "$SCRIPT" "https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/$SCRIPT_FNAME"
OUTPUT_FILE='/tmp/hivetables.csv'
QUERY="SELECT DBS.NAME, TBLS.TBL_NAME, SDS.LOCATION FROM SDS, TBLS, DBS WHERE TBLS.SD_ID = SDS.SD_ID AND TBLS.DB_ID = DBS.DB_ID ORDER BY DBS.NAME, TBLS.TBL_NAME ASC;"
sudo python "$SCRIPT" --query "$QUERY" > $OUTPUT_FILE
```

## <a name="further-reading"></a>Дополнительные материалы

* [Перенос рабочих нагрузок из HDInsight 3,6 в 4,0](./apache-hive-migrate-workloads.md)
* [Миграция рабочей нагрузки Hive между учетными записями хранения](./hive-migration-across-storage-accounts.md)
* [Подключение к Beeline в HDInsight](../hadoop/connect-install-beeline.md)
* [Устранение неполадок разрешения ошибка создания таблицы](./interactive-query-troubleshoot-permission-error-create-table.md)
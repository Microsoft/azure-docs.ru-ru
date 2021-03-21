---
title: Миграция рабочей нагрузки Hive в новую учетную запись в службе хранилища Azure
description: Миграция рабочей нагрузки Hive в новую учетную запись в службе хранилища Azure
author: kevxmsft
ms.author: kevx
ms.reviewer: ''
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/11/2020
ms.openlocfilehash: 0d62bebddb7751c168ba2e487b2391a40bbc6e67
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101747226"
---
# <a name="hive-workload-migration-to-new-account-in-azure-storage"></a>Миграция рабочей нагрузки Hive в новую учетную запись в службе хранилища Azure

Узнайте, как использовать действия сценария для копирования таблиц Hive между учетными записями хранения в HDInsight. Это может быть полезно при переходе на [`Azure Data Lake Storage Gen2`](../hdinsight-hadoop-use-data-lake-storage-gen2.md) .

Чтобы вручную скопировать отдельную таблицу Hive в HDInsight 4,0, см. раздел [Экспорт и импорт Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ImportExport).

## <a name="prerequisites"></a>Предварительные условия

* Новый кластер HDInsight со следующими конфигурациями:
  * Его файловая система по умолчанию находится в целевой учетной записи хранения. См. статью [Использование службы хранилища Azure с кластерами Azure HDInsight](../hdinsight-hadoop-use-blob-storage.md).
  * Ее версия кластера должна совпадать с версией исходного кластера.
  * В нем используется новая внешняя хранилище метаданных Hive DB. См. раздел [использование внешних хранилищ метаданных](../hdinsight-use-external-metadata-stores.md#select-a-custom-metastore-during-cluster-creation).
* Учетная запись хранения, доступная как к исходным, так и к новым кластерам. См. раздел [Добавление дополнительных учетных записей хранения в HDInsight](../hdinsight-hadoop-add-storage.md) и [типы хранилищ и функции](../hdinsight-hadoop-compare-storage-options.md#storage-types-and-features) для разрешенных вторичных типов хранилища.

    Ниже приведены некоторые параметры.
  * Добавьте целевую учетную запись хранения в исходный кластер.
  * Добавьте исходную учетную запись хранения в новый кластер.
  * Добавьте промежуточную учетную запись хранения в исходные и новые кластеры.

## <a name="how-it-works"></a>Принцип работы

Мы выполним действие сценария для экспорта таблиц Hive из исходного кластера в указанный каталог HDFS. См. раздел [действие сценария в работающем кластере](../hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster).

Затем мы выполним еще одно действие сценария в новом кластере, чтобы импортировать таблицы Hive из каталога HDFS.

Скрипт повторно создаст таблицы в новой файловой системе кластера по умолчанию. Собственные таблицы также будут копировать свои данные в хранилище. Несобственные таблицы будут копироваться только по определению. Дополнительные сведения о несобственных таблицах см. в разделе [обработчики хранилища Hive](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers) .

Путь внешних таблиц, отсутствующих в каталоге хранилища Hive, будет сохранен. Другие таблицы будут скопированы в путь Hive по умолчанию целевого кластера. См. раздел свойства Hive `hive.metastore.warehouse.external.dir` и `hive.metastore.warehouse.dir` .

Скрипты не будут сохранять разрешения пользовательского файла в целевом кластере.

> [!NOTE]
>
> Это Guide поддерживает копирование объектов метаданных, связанных с базами данных Hive, таблицами и секциями. Другие объекты метаданных необходимо создать повторно вручную.
>
> * Для `Views` команда Hive поддерживает `SHOW VIEWS` команду, 2.2.0 Hive в HDInsight 4,0. Используется `SHOW CREATE TABLE` для определения представления. Для более ранних версий Hive запросите хранилище метаданных базу данных SQL для отображения представлений.
> * Для `Materialized Views` Используйте команды, `SHOW MATERIALIZED VIEWS` `DESCRIBE FORMATTED` и `CREATE MATERIALIZED VIEW` . Дополнительные сведения см. в разделе [материализованные представления](https://cwiki.apache.org/confluence/display/Hive/Materialized+views) .
> * Для `Constraints` , поддерживается в 2.1.0 Hive в HDInsight 4,0, используйте `DESCRIBE EXTENDED` для перечисления ограничений для таблицы и используйте `ALTER TABLE` для добавления ограничений. Дополнительные сведения см. в разделе [ограничения ALTER TABLE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTableConstraints) .

## <a name="copy-hive-tables"></a>Копирование таблиц Hive

1. Примените действие сценария "экспорт" к исходному кластеру со следующими полями.

    При этом будут созданы и выполнены промежуточные скрипты Hive. Они будут сохраняться в указанном `<hdfs-export-path>` .

    При необходимости используйте, `--run-script=false` чтобы настроить их перед выполнением вручную.

    |Свойство. | Значение |
    |---|---|
    |URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/export-hive-data-v01.sh`|
    |Типы узлов|Head|
    |Параметры|`<hdfs-export-path>` `--run-script`|

    ```sh
    usage: generate Hive export and import scripts and export Hive data to specified HDFS path
           [--run-script={true,false}]
           hdfs-export-path

    positional arguments:

        hdfs-export-path      remote HDFS directory to write export data to

    optional arguments:
        --run-script={true,false}
                            whether to execute the generated Hive export script
                            (default: true)
    ```

2. После успешного завершения экспорта примените действие скрипта импорта в новом кластере со следующими полями.

    |Свойство. | Значение |
    |---|---|
    |URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/import-hive-data-v01.sh`|
    |Типы узлов|Head|
    |Параметры|`<hdfs-export-path>`|

    ```sh
    usage: download Hive import script from specified HDFS path and execute it
           hdfs-export-path

    positional arguments:

      hdfs-export-path      remote HDFS directory to download Hive import script from

    ```

## <a name="verification"></a>Проверка

Скачайте и запустите скрипт от имени привилегированного пользователя [`hive_contents.sh`](https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/hive_contents.sh) на основном узле каждого кластера и Сравните содержимое выходного файла `/tmp/hive_contents.out` . См. раздел [Подключение к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="cleanup-additional-storage-usage"></a>Очистка дополнительного использования хранилища

После завершения миграции хранилища и проверки можно удалить данные в указанном пути экспорта HDFS.

Параметр — использовать команду HDFS `hdfs dfs -rm -R` .

## <a name="option-reduce-additional-storage-usage"></a>Параметр: сокращение дополнительного использования хранилища

Действие скрипта экспорта, вероятно, удваивает использование хранилища из-за Hive. Однако можно ограничить дополнительное использование хранилища, переполнив миграцию вручную, по одной базе данных или таблице за раз.

1. Укажите `--run-script=false` , чтобы пропустить выполнение созданного скрипта Hive. Скрипты экспорта и импорта Hive по-прежнему будут сохранены в пути экспорта.

2. Выполните фрагменты кода из скриптов экспорта и импорта Hive база данных или таблица по таблице, вручную очистив путь экспорта после каждой перенесенной базы данных или таблицы.

## <a name="next-steps"></a>Дальнейшие действия

* [Azure Data Lake Storage 2-го поколения](../hdinsight-hadoop-use-data-lake-storage-gen2.md)
* [Использование внешних хранилищ метаданных](../hdinsight-use-external-metadata-stores.md#select-a-custom-metastore-during-cluster-creation)
* [Типы и функции хранения](../hdinsight-hadoop-compare-storage-options.md#storage-types-and-features)
* [Действие скрипта в работающем кластере](../hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster)

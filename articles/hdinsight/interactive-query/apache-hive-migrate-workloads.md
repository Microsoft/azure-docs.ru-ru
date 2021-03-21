---
title: Перенос рабочих нагрузок Hive из Azure HDInsight 3.6 в HDInsight 4.0
description: Узнайте, как перенести рабочие нагрузки Apache Hive в HDInsight 3,6 в HDInsight 4,0.
author: kevxmsft
ms.author: kevx
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 11/4/2020
ms.openlocfilehash: b13e8e088eff95071247a53ad1a4a18879f94053
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101742200"
---
# <a name="migrate-azure-hdinsight-36-hive-workloads-to-hdinsight-40"></a>Перенос рабочих нагрузок Hive из Azure HDInsight 3.6 в HDInsight 4.0

HDInsight 4,0 имеет несколько преимуществ по сравнению с HDInsight 3,6. Ниже приведен [Обзор новых возможностей в HDInsight 4,0](../hdinsight-version-release.md).

В этой статье рассматриваются действия по переносу рабочих нагрузок Hive с HDInsight 3,6 на 4,0, включая

* хранилище метаданных Hive копирования и обновления схемы
* Безопасность миграции для обеспечения совместимости ACID
* Сохранение политик безопасности Hive

Новые и старые кластеры HDInsight должны иметь доступ к одним и тем же учетным записям хранения.

Миграция таблиц Hive в новую учетную запись хранения должна выполняться в отдельном шаге. См. раздел [Миграция Hive между учетными записями хранения](./hive-migration-across-storage-accounts.md).

## <a name="steps-to-upgrade"></a>Действия по обновлению

### <a name="1-prepare-the-data"></a>1. Подготовка данных

* HDInsight 3,6 по умолчанию не поддерживает таблицы ACID. Однако если существуют таблицы ACID, выполните на них "основные" сжатия. Дополнительные сведения о сжатии см. в [руководстве по языку Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable/Partition/Compact) .

* При использовании [Azure Data Lake Storage 1-го поколения](../overview-data-lake-storage-gen1.md)расположение таблиц Hive, скорее всего, зависит от конфигураций HDFS кластера. Выполните следующее действие скрипта, чтобы обеспечить перенос этих расположений в другие кластеры. См. раздел [действие сценария в работающем кластере](../hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster).

    |Свойство. | Значение |
    |---|---|
    |URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/hive-adl-expand-location-v01.sh`|
    |Типы узлов|Head|
    |Параметры||

### <a name="2-copy-the-sql-database"></a>2. копирование базы данных SQL

* Если кластер использует хранилище метаданных Hive по умолчанию, следуйте этому [руководству](./hive-default-metastore-export-import.md) , чтобы экспортировать метаданные во внешний хранилище метаданных. Затем создайте копию внешнего хранилище метаданных для обновления.

* Если кластер использует внешний хранилище метаданных Hive, создайте его копию. К параметрам относятся [Экспорт/импорт](../../azure-sql/database/database-export.md) и [Восстановление до точки во времени](../../azure-sql/database/recovery-using-backups.md#point-in-time-restore).

### <a name="3-upgrade-the-metastore-schema"></a>3. обновите схему хранилище метаданных

Этот шаг использует [`Hive Schema Tool`](https://cwiki.apache.org/confluence/display/Hive/Hive+Schema+Tool) из HDInsight 4,0 для обновления схемы хранилище метаданных.

> [!Warning]
> Этот шаг необратим. Выполните это только для копии хранилище метаданных.

1. Создайте временный кластер HDInsight 4,0 для доступа к кусту 4,0 `schematool` . Для этого шага можно использовать [хранилище метаданных Hive по умолчанию](../hdinsight-use-external-metadata-stores.md#default-metastore) .

1. В кластере HDInsight 4,0 выполните команду, `schematool` чтобы обновить целевой объект HDInsight 3,6 хранилище метаданных:

    ```sh
    SERVER='servername.database.windows.net'  # replace with your SQL Server
    DATABASE='database'  # replace with your 3.6 metastore SQL Database
    USERNAME='username'  # replace with your 3.6 metastore username
    PASSWORD='password'  # replace with your 3.6 metastore password
    STACK_VERSION=$(hdp-select status hive-server2 | awk '{ print $3; }')
    /usr/hdp/$STACK_VERSION/hive/bin/schematool -upgradeSchema -url "jdbc:sqlserver://$SERVER;databaseName=$DATABASE;trustServerCertificate=false;encrypt=true;hostNameInCertificate=*.database.windows.net;" -userName "$USERNAME" -passWord "$PASSWORD" -dbType "mssql" --verbose
    ```

    > [!NOTE]
    > Эта программа использует клиент `beeline` для выполнения скриптов SQL в `/usr/hdp/$STACK_VERSION/hive/scripts/metastore/upgrade/mssql/upgrade-*.mssql.sql` .
    >
    > Синтаксис SQL в этих скриптах не обязательно совместим с другими клиентскими средствами. Например, в [среде SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) и [редакторе запросов на портале Azure](../../azure-sql/database/connect-query-portal.md) требуется ключевое слово `GO` после каждой команды.
    >
    > Если выполнение какого-либо скрипта завершается сбоем вследствие объема ресурсов или превышения времени ожидания транзакций, увеличьте масштаб базы данных SQL.

1. Проверьте окончательную версию с помощью запроса `select schema_version from dbo.version` .

    Выходные данные должны соответствовать следующей команде bash из кластера HDInsight 4,0.

    ```bash
    grep . /usr/hdp/$(hdp-select --version)/hive/scripts/metastore/upgrade/mssql/upgrade.order.mssql | tail -n1 | rev | cut -d'-' -f1 | rev
    ```

1. Удалите временный кластер HDInsight 4,0.

### <a name="4-deploy-a-new-hdinsight-40-cluster"></a>4. Развертывание нового кластера HDInsight 4,0

Создайте новый кластер HDInsight 4,0, [выбрав обновленную хранилище метаданных Hive](../hdinsight-use-external-metadata-stores.md#select-a-custom-metastore-during-cluster-creation) и те же учетные записи хранения.

* Для нового кластера не требуется та же файловая система по умолчанию.

* Если хранилище метаданных содержит таблицы, находящиеся в нескольких учетных записях хранения, необходимо добавить эти учетные записи хранения в новый кластер для доступа к этим таблицам. См. раздел [Добавление дополнительных учетных записей хранения в HDInsight](../hdinsight-hadoop-add-storage.md).

* Если сбой заданий Hive из-за недоступности хранилища, убедитесь, что расположение таблицы находится в учетной записи хранения, добавленной в кластер.

    Используйте следующую команду Hive для определения расположения таблицы:

    ```sql
    SHOW CREATE TABLE ([db_name.]table_name|view_name);
    ```

### <a name="5-convert-tables-for-acid-compliance"></a>5. Преобразование таблиц для обеспечения соответствия ACID

Управляемые таблицы должны быть совместимыми с ACID в HDInsight 4,0. Используйте `strictmanagedmigration` в HDInsight 4,0 для преобразования всех управляемых таблиц, не являющихся ACID, во внешние таблицы со свойством `'external.table.purge'='true'` . Выполните из головного узла:

```bash
sudo su - hive
STACK_VERSION=$(hdp-select status hive-server2 | awk '{ print $3; }')
/usr/hdp/$STACK_VERSION/hive/bin/hive --config /etc/hive/conf --service strictmanagedmigration --hiveconf hive.strict.managed.tables=true -m automatic --modifyManagedTables
```

## <a name="secure-hive-across-hdinsight-versions"></a>Защита Hive в версиях HDInsight

HDInsight при необходимости интегрируется с Azure Active Directory с помощью HDInsight Корпоративный пакет безопасности (ESP). ESP использует Kerberos и Apache Ranger для управления разрешениями конкретных ресурсов в кластере. Политики Ranger, развернутые с помощью Hive в HDInsight 3,6, можно перенести в HDInsight 4,0, выполнив следующие действия:

1. Перейдите на панель Service Manager Ranger в кластере HDInsight 3,6.
2. Перейдите к политике с именем **Hive** и экспортируйте политику в JSON-файл.
3. Убедитесь, что все пользователи, которые ссылаются на экспортированную политику JSON, существуют в новом кластере. Если пользователь упоминается в политике JSON, но не существует в новом кластере, добавьте пользователя в новый кластер или удалите ссылку из политики.
4. Перейдите на панель **Service Manager Ranger** в кластере HDInsight 4,0.
5. Перейдите к политике с именем **Hive** и импортируйте Ranger политику JSON из шага 2.

## <a name="hive-changes-in-hdinsight-40-that-may-require-application-changes"></a>Изменения Hive в HDInsight 4,0, которые могут потребовать изменения в приложении

* См. Дополнительные сведения о [настройке с помощью соединителя хранилища Hive](./apache-hive-warehouse-connector.md) для совместного использования хранилище метаданных между Spark и Hive для таблиц ACID.

* HDInsight 4,0 использует [авторизацию на основе хранилища](https://cwiki.apache.org/confluence/display/Hive/Storage+Based+Authorization+in+the+Metastore+Server). Если вы изменяете разрешения файла или создаете папки от имени пользователя, отличного от Hive, скорее всего, будут обнаружены ошибки Hive на основе разрешений на хранилище. Чтобы исправить это, предоставьте `rw-` доступ пользователю. См. [инструкции по разрешениям HDFS](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html).

* `HiveCLI` заменяется на `Beeline` .

Дополнительные изменения см. в [объявлении HDInsight 4,0](../hdinsight-version-release.md) .

## <a name="further-reading"></a>Дополнительные сведения

* [Объявление о HDInsight 4,0](../hdinsight-version-release.md)
* [HDInsight 4,0. подробное углубление](https://azure.microsoft.com/blog/deep-dive-into-azure-hdinsight-4-0/)
* [Таблицы ACID Hive 3](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/using-hiveql/content/hive_3_internals.html)

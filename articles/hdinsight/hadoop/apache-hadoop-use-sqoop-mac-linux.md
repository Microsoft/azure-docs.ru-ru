---
title: Использование Apache Sqoop с Apache Hadoop в Azure HDInsight
description: Узнайте, как использовать Apache Sqoop для импорта и экспорта между Apache Hadoop в HDInsight и базе данных SQL Azure.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 11/28/2019
ms.openlocfilehash: 2d0d38dee15817e56c2784981365ea331b6a8459
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98943155"
---
# <a name="use-apache-sqoop-to-import-and-export-data-between-apache-hadoop-on-hdinsight-and-azure-sql-database"></a>Использование Apache Sqoop для импорта и экспорта данных между Apache Hadoop в HDInsight и базе данных SQL Azure

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Узнайте, как использовать Apache Sqoop для импорта и экспорта между кластером Apache Hadoop в Azure HDInsight и базой данных SQL Azure или Microsoft SQL Server. В этом руководстве используется команда `sqoop` ​​непосредственно из головного узла кластера Hadoop. В этом документе вы подключитесь к головному узлу, используя SSH, и выполните команды. Эта статья является продолжением статьи [Использование Apache Sqoop с Hadoop в HDInsight](./hdinsight-use-sqoop.md).

## <a name="prerequisites"></a>Предварительные требования

* Должна быть выполнена [настройка тестовой среды](./hdinsight-use-sqoop.md#create-cluster-and-sql-database), описанная в разделе [Использование Apache Sqoop с Hadoop в HDInsight](./hdinsight-use-sqoop.md).

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Опыт работы со Sqoop. Дополнительные сведения см. в [руководстве пользователя по Sqoop](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html).

## <a name="set-up"></a>Настройка

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Для простоты использования задайте переменные. Замените `PASSWORD` , `MYSQLSERVER` и `MYDATABASE` соответствующими значениями, а затем введите следующие команды:

    ```bash
    export password='PASSWORD'
    export sqlserver="MYSQLSERVER"
    export database="MYDATABASE"


    export serverConnect="jdbc:sqlserver://$sqlserver.database.windows.net:1433;user=sqluser;password=$password"
    export serverDbConnect="jdbc:sqlserver://$sqlserver.database.windows.net:1433;user=sqluser;password=$password;database=$database"
    ```

## <a name="sqoop-export"></a>Экспорт Sqoop

Из Hive в SQL.

1. Чтобы убедиться, что Sqoop может видеть вашу базу данных, введите приведенную ниже команду в открытом SSH-подключении. Эта команда возвращает список баз данных.

    ```bash
    sqoop list-databases --connect $serverConnect
    ```

1. Введите следующую команду, чтобы просмотреть список таблиц для указанной базы данных:

    ```bash
    sqoop list-tables --connect $serverDbConnect
    ```

1. Чтобы экспортировать данные из таблицы Hive в `hivesampletable` `mobiledata` таблицу базы данных, введите приведенную ниже команду в открытом SSH-подключении.

    ```bash
    sqoop export --connect $serverDbConnect \
    -table mobiledata \
    --hcatalog-table hivesampletable
    ```

1. Чтобы убедиться, что данные были экспортированы, используйте следующие запросы из SSH-подключения для просмотра экспортированных данных:

    ```bash
    sqoop eval --connect $serverDbConnect \
    --query "SELECT COUNT(*) from dbo.mobiledata WITH (NOLOCK)"


    sqoop eval --connect $serverDbConnect \
    --query "SELECT TOP(10) * from dbo.mobiledata WITH (NOLOCK)"
    ```

## <a name="sqoop-import"></a>Импорт Sqoop

Из SQL в службу хранилища Azure.

1. Введите приведенную ниже команду в открытом SSH-подключении, чтобы импортировать данные из `mobiledata` таблицы в SQL, в `wasbs:///tutorials/usesqoop/importeddata` каталог в HDInsight. Поля в данных разделены знаками табуляции, а строки завершаются символом новой строки.

    ```bash
    sqoop import --connect $serverDbConnect \
    --table mobiledata \
    --target-dir 'wasb:///tutorials/usesqoop/importeddata' \
    --fields-terminated-by '\t' \
    --lines-terminated-by '\n' -m 1
    ```

1. Кроме того, можно указать таблицу Hive:

    ```bash
    sqoop import --connect $serverDbConnect \
    --table mobiledata \
    --target-dir 'wasb:///tutorials/usesqoop/importeddata2' \
    --fields-terminated-by '\t' \
    --lines-terminated-by '\n' \
    --create-hive-table \
    --hive-table mobiledata_imported2 \
    --hive-import -m 1
    ```

1. После завершения импорта введите следующую команду в открытом SSH-подключении, чтобы вывести данные в новом каталоге:

    ```bash
    hadoop fs -tail /tutorials/usesqoop/importeddata/part-m-00000
    ```

1. Используйте [Beeline](./apache-hadoop-use-hive-beeline.md) , чтобы убедиться, что таблица была создана в Hive.

    1. Подключение

        ```bash
        beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
        ```

    1. Выполните каждый запрос по одному за раз и просмотрите выходные данные:

        ```hql
        show tables;
        describe mobiledata_imported2;
        SELECT COUNT(*) FROM mobiledata_imported2;
        SELECT * FROM mobiledata_imported2 LIMIT 10;
        ```

    1. Выйдите из Beeline с помощью `!exit` .

## <a name="limitations"></a>Ограничения

* При выполнении полного экспорта с помощью HDInsight на основе Linux соединитель Sqoop, используемый для экспорта данных в SQL, не поддерживает операции вставки.

* Пакетная обработка: при использовании HDInsight на основе Linux, когда для выполнения вставок применяется параметр`-batch`, Sqoop выполняет несколько вставок вместо пакетной обработки операций вставки.

## <a name="important-considerations"></a>Важные замечания

* HDInsight и SQL Server должны находиться в одной виртуальной сети Azure.

    Пример см. в статье о [подключении HDInsight к локальной сети](./../connect-on-premises-network.md).

    Подробные сведения об использовании HDInsight c виртуальными сетями Azure см. в статье [Расширение возможностей HDInsight с помощью виртуальной сети Azure](../hdinsight-plan-virtual-network-deployment.md). Дополнительные сведения см. в статье [Виртуальная сеть Azure](../../virtual-network/virtual-networks-overview.md).

* Также SQL Server должен разрешать аутентификацию SQL. Дополнительные сведения см. в документе [Выбор режима проверки подлинности](/sql/relational-databases/security/choose-an-authentication-mode).

* Необходимо настроить SQL Server для удаленных соединений. Дополнительные сведения см. в документе [How to troubleshoot connecting to the SQL Server database engine](https://social.technet.microsoft.com/wiki/contents/articles/2102.how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx) (Устранение неполадок при подключении к ядру СУБД SQL Server).

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как использовать Sqoop. Дополнительные сведения см. на следующих ресурсах:

* [Использование Apache Oozie с HDInsight](../hdinsight-use-oozie-linux-mac.md). Используйте действие Sqoop в рабочем процессе Oozie.
* [Анализ данных о задержке рейсов с помощью HDInsight](../interactive-query/interactive-query-tutorial-analyze-flight-data.md): используйте интерактивный запрос для анализа данных о задержке рейсов, а затем используйте Sqoop для экспорта данных в базу данных в Azure.
* [Передача данных в HDInsight](../hdinsight-upload-data.md): узнайте о других способах отправки данных в HDInsight и хранилище больших двоичных объектов Azure.
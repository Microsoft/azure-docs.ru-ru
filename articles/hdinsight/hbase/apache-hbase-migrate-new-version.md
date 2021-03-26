---
title: Перенос кластера HBase в новую версию в Azure HDInsight
description: Как перенести кластеры Apache HBase в более новую версию в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/02/2020
ms.openlocfilehash: 43b46d19503856f5eae38272299f73d9c80055b8
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104868891"
---
# <a name="migrate-an-apache-hbase-cluster-to-a-new-version"></a>Перенос данных кластера Apache HBase в новую версию

В этой статье обсуждаются шаги, необходимые для обновления кластера Apache HBase в Azure HDInsight до более новой версии.

Время простоя при обновлении должно быть минимальным, порядка нескольких минут. Простой связан с действиями по записи всех данных в памяти на диск, а также настройкой и перезапуском служб в новом кластере. Результаты будут различаться в зависимости от количества узлов, объема данных и других переменных.

## <a name="review-apache-hbase-compatibility"></a>Проверка совместимости Apache HBase

Перед обновлением Apache HBase убедитесь, что версии HBase в исходном и конечном кластерах совместимы. Дополнительные сведения см. в статье [Что представляют собой компоненты и версии Apache Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)

> [!NOTE]  
> Настоятельно рекомендуем изучить матрицу совместимости версий в [руководстве по HBase](https://hbase.apache.org/book.html#upgrading). Все критические несовместимости должны быть описаны в заметках о выпуске версии HBase.

Ниже приведен пример матрицы совместимости версий. Y означает совместимость, а N означает потенциальную несовместимость:

| Тип совместимости | Основной номер версии| Дополнительный номер версии | Обновление |
| --- | --- | --- | --- |
| Совместимость клиента и сервера на уровне передачи данных | Нет | Да | Да |
| Совместимость серверов | Нет | Да | Да |
| Совместимость форматов файлов | Нет | Да | Да |
| Совместимость API клиента | Нет | Да | Да |
| Совместимость клиента на уровне двоичного кода | Нет | Нет | Да |
| **Ограниченная совместимость API на стороне сервера** |  |  |  |
| объем стабилен | Нет | Да | Да |
| Развитие | Нет | Нет | Да |
| Работает неустойчиво | Нет | Нет | Нет |
| Совместимость зависимостей | Нет | Да | Да |
| Совместимость операций | Нет | Нет | Да |

## <a name="upgrade-with-same-apache-hbase-major-version"></a>Обновление с тем же основным номером версии Apache HBase

Чтобы обновить кластер Apache HBase в Azure HDInsight, выполните следующие действия.

1. Убедитесь, что приложение совместимо с новой версией, обратившись к матрице совместимости HBase и заметкам о выпуске. Протестируйте приложение в кластере, запустив целевые версии HDInsight и HBase.

1. [Настройте новый кластер назначения HDInsight](../hdinsight-hadoop-provision-linux-clusters.md), используя ту же учетную запись хранения, но другое имя контейнера:

   :::image type="content" source="./media/apache-hbase-migrate-new-version/same-storage-different-container.png" alt-text="Используйте ту же учетную запись хранения, но создайте другой контейнер" border="true":::

1. Очистите исходный кластер HBase, который представляет собой обновляемый кластер. HBase записывает входящие данные в хранилище в памяти, которое называется _memstore_. После того как memstore достигнет определенного размера, HBase записывает его на диск для долгосрочного хранения в учетной записи хранения кластера. При удалении старого кластера memstore очищается, что может привести к потере данных. Чтобы вручную перенести данные memstore для каждой таблицы на диск, выполните следующий скрипт. Последнюю версию этого скрипта можно найти на [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/scripts/flush_all_tables.sh) для Azure.

    ```bash
    #!/bin/bash
    
    #-------------------------------------------------------------------------------#
    # SCRIPT TO FLUSH ALL HBASE TABLES.
    #-------------------------------------------------------------------------------#
    
    LIST_OF_TABLES=/tmp/tables.txt
    HBASE_SCRIPT=/tmp/hbase_script.txt
    TARGET_HOST=$1
    
    usage ()
    {
        if [[ "$1" == "-h" ]] || [[ "$1" == "--help" ]]
        then
            cat << ...
    
    Usage: 
    
    $0 [hostname]
    
    Providing hostname is optional and not required when the script is executed within HDInsight cluster with access to 'hbase shell'.
    
    However hostname should be provided when executing the script as a script-action from HDInsight portal.
    
    For Example:
    
        1.  Executing script inside HDInsight cluster (where 'hbase shell' is 
            accessible):
    
            $0 
    
            [No need to provide hostname]
    
        2.  Executing script from HDinsight Azure portal:
    
            Provide Script URL.
    
            Provide hostname as a parameter (i.e. hn0, hn1, hn2.. or wn2 etc.).
    ...
            exit
        fi
    }
    
    validate_machine ()
    {
        THIS_HOST=`hostname`
    
        if [[ ! -z "$TARGET_HOST" ]] && [[ $THIS_HOST  != $TARGET_HOST* ]]
        then
            echo "[INFO] This machine '$THIS_HOST' is not the right machine ($TARGET_HOST) to execute the script."
            exit 0
        fi
    }
    
    get_tables_list ()
    {
    hbase shell << ... > $LIST_OF_TABLES 2> /dev/null
        list
        exit
    ...
    }
    
    add_table_for_flush ()
    {
        TABLE_NAME=$1
        echo "[INFO] Adding table '$TABLE_NAME' to flush list..."
        cat << ... >> $HBASE_SCRIPT
            flush '$TABLE_NAME'
    ...
    }
    
    clean_up ()
    {
        rm -f $LIST_OF_TABLES
        rm -f $HBASE_SCRIPT
    }
    
    ########
    # MAIN #
    ########
    
    usage $1
    
    validate_machine
    
    clean_up
    
    get_tables_list
    
    START=false
    
    while read LINE 
    do 
        if [[ $LINE == TABLE ]] 
        then
            START=true
            continue
        elif [[ $LINE == *row*in*seconds ]]
        then
            break
        elif [[ $START == true ]]
        then
            add_table_for_flush $LINE
        fi
    
    done < $LIST_OF_TABLES
    
    cat $HBASE_SCRIPT
    
    hbase shell $HBASE_SCRIPT << ... 2> /dev/null
    exit
    ...
    
    ```

1. Остановите прием данных в старом кластере HBase.

1. Чтобы убедиться, что все последние данные в хранилище memstore записаны на диск, еще раз выполните предыдущий скрипт.

1. Войдите в [Apache Ambari](https://ambari.apache.org/) в старом кластере ( `https://OLDCLUSTERNAME.azurehdidnsight.net` ) и завершите работу служб HBase. При появлении запроса подтвердить, что вы хотите отключить службы, установите флажок, чтобы включить режим обслуживания для HBase. Подробные сведения о подключении к платформе Ambari и ее использовании см. в статье [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](../hdinsight-hadoop-manage-ambari.md).

    :::image type="content" source="./media/apache-hbase-migrate-new-version/stop-hbase-services1.png" alt-text="В Ambari щелкните службы > > HBase в разделе действия службы." border="true":::

    :::image type="content" source="./media/apache-hbase-migrate-new-version/turn-on-maintenance-mode.png" alt-text="Проверьте, установлен ли флажок Turn On Maintenance Mode for HBase (Включить режим обслуживания HBase), а затем подтвердите остановку." border="true":::

1. Если вы не используете кластеры HBase с функцией расширенной записи, пропустите этот шаг. Он нужен только для кластеров HBase с функцией расширенной записи.

   Создайте резервную копию WAL dir в HDFS, выполнив приведенные ниже команды из сеанса SSH на любом из узлов Zookeeper или рабочих узлов исходного кластера.
   
   ```bash
   hdfs dfs -mkdir /hbase-wal-backup**
   hdfs dfs -cp hdfs://mycluster/hbasewal /hbase-wal-backup**
   ```
    
1. Войдите в Ambari в новом кластере HDInsight. Измените параметр HDFS `fs.defaultFS`, чтобы указать имя контейнера, используемого в исходном кластере. Этот параметр находится в разделе **HDFS > Configs (Конфигурации) > Advanced (Расширенные) > Advanced core-site (Расширенный основной сайт)**.

   :::image type="content" source="./media/apache-hbase-migrate-new-version/hdfs-advanced-settings.png" alt-text="В Ambari щелкните службы > HDFS > configs > дополнительно." border="true":::

   :::image type="content" source="./media/apache-hbase-migrate-new-version/change-container-name.png" alt-text="Изменение имени контейнера в Ambari" border="true":::

1. Если вы не используете кластеры HBase с функцией расширенной записи, пропустите этот шаг. Он нужен только для кластеров HBase с функцией расширенной записи.

   Измените `hbase.rootdir` путь, чтобы он указывал на контейнер исходного кластера.

   :::image type="content" source="./media/apache-hbase-migrate-new-version/change-container-name-for-hbase-rootdir.png" alt-text="В Ambari измените имя контейнера для рутдир HBase." border="true":::
    
1. Если вы не используете кластеры HBase с функцией расширенной записи, пропустите этот шаг. Он нужен только для кластеров HBase с функцией расширенной записи и только в тех случаях, когда исходный кластер был кластером HBase с функцией расширенной записи.

   Очистите данные Zookeeper и WAL FS для этого нового кластера. Выполните следующие команды в любом из узлов Zookeeper или рабочих узлов:

   ```bash
   hbase zkcli
   rmr /hbase-unsecure
   quit

   hdfs dfs -rm -r hdfs://mycluster/hbasewal**
   ```

1. Если вы не используете кластеры HBase с функцией расширенной записи, пропустите этот шаг. Он нужен только для кластеров HBase с функцией расширенной записи.
   
   Восстановите WAL dir в каталог HDFS нового кластера из сеанса SSH на любом из узлов Zookeeper или рабочих узлов нового кластера.
   
   ```bash
   hdfs dfs -cp /hbase-wal-backup/hbasewal hdfs://mycluster/**
   ```
   
1. Если вы обновляете HDInsight 3,6 до 4,0, выполните описанные ниже действия, в противном случае перейдите к шагу 13.

    1. Перезапустите все необходимые службы в Ambari, выбрав **службы**  >  **все необходимые для перезапуска**.
    1. Завершите работу службы HBase.
    1. Подключитесь к узлу Zookeeper по протоколу SSH и выполните команду [zkCli](https://github.com/go-zkcli/zkcli) , `rmr /hbase-unsecure` чтобы удалить корневой Znode будет удален HBase из Zookeeper.
    1. Перезапустите HBase.

1. При обновлении до любой другой версии HDInsight, кроме 4,0, выполните следующие действия.
    1. Сохраните изменения.
    1. Перезапустите все необходимые службы, как указано в Ambari.

1. Направьте приложение в новый кластер.

    > [!NOTE]  
    > Статическое DNS-имя вашего приложения изменится при обновлении. Вместо того чтобы жестко запрограммировать это DNS-имя, можно настроить запись CNAME в параметрах DNS доменного имени, которая указывает на имя кластера. Другой вариант — использовать файл конфигурации для приложения, которое можно обновить без повторного развертывания.

1. Запустите прием данных, чтобы убедиться, что все работает правильно.

1. Если новый кластер подходит, удалите исходный кластер.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [Apache HBase](https://hbase.apache.org/) и обновлении кластеров HDInsight см. в следующих статьях:

* [Обновление кластера HDInsight до более новой версии](../hdinsight-upgrade-cluster.md)
* [Управление кластерами HDInsight с помощью веб-интерфейса Apache Ambari](../hdinsight-hadoop-manage-ambari.md)
* [Компоненты и версии Apache Hadoop](../hdinsight-component-versioning.md)
* [Оптимизация Apache HBase](../optimize-hbase-ambari.md)

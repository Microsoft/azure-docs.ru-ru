---
title: Устранение неполадок HDFS в Azure HDInsight
description: Получите ответы на распространенные вопросы о работе с HDFS и Azure HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 04/27/2020
ms.custom: seodec18
ms.openlocfilehash: 0be7805493e5acc41254c57ca912b5a2ecf02dae
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98931464"
---
# <a name="troubleshoot-apache-hadoop-hdfs-by-using-azure-hdinsight"></a>Устранение неполадок рабочих нагрузок Apache Hadoop HDFS с помощью Azure HDInsight

Сведения об основных проблемах и способах их решения при работе с Hadoop распределенная файловая система (HDFS). Полный список команд см. в [руководству по командам HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html) и в разделе [руководств по оболочке файловой системы](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html).

## <a name="how-do-i-access-the-local-hdfs-from-inside-a-cluster"></a><a name="how-do-i-access-local-hdfs-from-inside-a-cluster"></a>Как получить доступ к локальной системе HDFS в пределах кластера?

### <a name="issue"></a>Проблема

Вместо хранилища BLOB-объектов Azure или Azure Data Lake Storage для доступа к локальной системе HDFS в пределах кластера HDInsight используется командная строка и код приложения.

### <a name="resolution-steps"></a>Действия по устранению

1. Вставьте в командную строку `hdfs dfs -D "fs.default.name=hdfs://mycluster/" ...` без изменений, как в следующей команде:

    ```output
    hdfs dfs -D "fs.default.name=hdfs://mycluster/" -ls /
    Found 3 items
    drwxr-xr-x   - hdiuser hdfs          0 2017-03-24 14:12 /EventCheckpoint-30-8-24-11102016-01
    drwx-wx-wx   - hive    hdfs          0 2016-11-10 18:42 /tmp
    drwx------   - hdiuser hdfs          0 2016-11-10 22:22 /user
    ```

2. Вставьте в исходный код универсальный код ресурса (URI) `hdfs://mycluster/` без изменений, как показано в примере приложения.

    ```Java
    import java.io.IOException;
    import java.net.URI;
    import org.apache.commons.io.IOUtils;
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.*;

    public class JavaUnitTests {

        public static void main(String[] args) throws Exception {

            Configuration conf = new Configuration();
            String hdfsUri = "hdfs://mycluster/";
            conf.set("fs.defaultFS", hdfsUri);
            FileSystem fileSystem = FileSystem.get(URI.create(hdfsUri), conf);
            RemoteIterator<LocatedFileStatus> fileStatusIterator = fileSystem.listFiles(new Path("/tmp"), true);
            while(fileStatusIterator.hasNext()) {
                System.out.println(fileStatusIterator.next().getPath().toString());
            }
        }
    }
    ```

3. Запустите скомпилированный JAR-файл (например, файл с именем `java-unit-tests-1.0.jar`) в кластере HDInsight с помощью следующей команды:

    ```apache
    hadoop jar java-unit-tests-1.0.jar JavaUnitTests
    hdfs://mycluster/tmp/hive/hive/5d9cf301-2503-48c7-9963-923fb5ef79a7/inuse.info
    hdfs://mycluster/tmp/hive/hive/5d9cf301-2503-48c7-9963-923fb5ef79a7/inuse.lck
    hdfs://mycluster/tmp/hive/hive/a0be04ea-ae01-4cc4-b56d-f263baf2e314/inuse.info
    hdfs://mycluster/tmp/hive/hive/a0be04ea-ae01-4cc4-b56d-f263baf2e314/inuse.lck
    ```

## <a name="storage-exception-for-write-on-blob"></a>Исключение хранилища для записи в большой двоичный объект

### <a name="issue"></a>Проблема

При использовании `hadoop` команд или `hdfs dfs` для записи файлов размером свыше 12 ГБ в кластер HBase может возникнуть следующая ошибка:

```error
ERROR azure.NativeAzureFileSystem: Encountered Storage Exception for write on Blob : example/test_large_file.bin._COPYING_ Exception details: null Error Code : RequestBodyTooLarge
copyFromLocal: java.io.IOException
        at com.microsoft.azure.storage.core.Utility.initIOException(Utility.java:661)
        at com.microsoft.azure.storage.blob.BlobOutputStream$1.call(BlobOutputStream.java:366)
        at com.microsoft.azure.storage.blob.BlobOutputStream$1.call(BlobOutputStream.java:350)
        at java.util.concurrent.FutureTask.run(FutureTask.java:262)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
        at java.util.concurrent.FutureTask.run(FutureTask.java:262)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
Caused by: com.microsoft.azure.storage.StorageException: The request body is too large and exceeds the maximum permissible limit.
        at com.microsoft.azure.storage.StorageException.translateException(StorageException.java:89)
        at com.microsoft.azure.storage.core.StorageRequest.materializeException(StorageRequest.java:307)
        at com.microsoft.azure.storage.core.ExecutionEngine.executeWithRetry(ExecutionEngine.java:182)
        at com.microsoft.azure.storage.blob.CloudBlockBlob.uploadBlockInternal(CloudBlockBlob.java:816)
        at com.microsoft.azure.storage.blob.CloudBlockBlob.uploadBlock(CloudBlockBlob.java:788)
        at com.microsoft.azure.storage.blob.BlobOutputStream$1.call(BlobOutputStream.java:354)
        ... 7 more
```

### <a name="cause"></a>Причина

В HBase в кластерах HDInsight размер блока по умолчанию при записи в службу хранилища Azure равен 256 КБ. Несмотря на то что это подходит при использовании API HBase или REST API, применение служебных программ командной строки `hadoop` или `hdfs dfs` вызовет ошибку.

### <a name="resolution"></a>Решение

Задайте больший размер блока с помощью свойства `fs.azure.write.request.size`. Это изменение можно выполнить отдельно для каждого использования с помощью `-D` параметра. Далее приведен пример использования параметра с командой `hadoop`:

```bash
hadoop -fs -D fs.azure.write.request.size=4194304 -copyFromLocal test_large_file.bin /example/data
```

Можно также глобально увеличить значение `fs.azure.write.request.size` с помощью Apache Ambari. Чтобы изменить значение в веб-интерфейсе Ambari, сделайте следующее:

1. В браузере перейдите к веб-интерфейсу Ambari для кластера URL-адрес — `https://CLUSTERNAME.azurehdinsight.net` , где `CLUSTERNAME` — имя кластера. При появлении запроса введите имя и пароль администратора для кластера.
2. В левой части экрана выберите **HDFS**, а затем перейдите на вкладку **Configs** (Конфигурации).
3. В поле **фильтр...** введите `fs.azure.write.request.size` .
4. Измените значение с 262 144 (256 КБ) на новое. Например, 4 194 304 (4 MB).

    ![Изображение смены значения через веб-интерфейс Ambari](./media/hdinsight-troubleshoot-hdfs/hbase-change-block-write-size.png)

Подробные сведения об использовании Ambari см. в статье [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](hdinsight-hadoop-manage-ambari.md).

## <a name="du"></a>du

[`-du`](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#du)Команда отображает размеры файлов и каталогов, содержащихся в заданном каталоге, или длину файла, если это просто файл.

`-s`Параметр создает сводную сводку по длине файлов.  
`-h`Параметр форматирует размеры файлов.

Пример

```bash
hdfs dfs -du -s -h hdfs://mycluster/
hdfs dfs -du -s -h hdfs://mycluster/tmp
```

## <a name="rm"></a>rm

Команда [-RM](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#rm) удаляет файлы, указанные в качестве аргументов.

Пример

```bash
hdfs dfs -rm hdfs://mycluster/tmp/testfile
```

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]
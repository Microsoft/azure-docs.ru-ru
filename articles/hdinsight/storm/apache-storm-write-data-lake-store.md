---
title: Руководство. Запись данных HDInsight Apache Storm в хранилище в Azure или озеро данных
description: Учебник. Сведения об использовании Apache Storm для записи данных в HDFS-совместимое хранилище для Azure HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 06/24/2019
ms.openlocfilehash: 4e648c57be699620e669ce7db0845dad2b876095
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932548"
---
# <a name="tutorial-write-to-apache-hadoop-hdfs-from-apache-storm-on-azure-hdinsight"></a>Руководство. Запись данных в Apache Hadoop HDFS из Apache Storm в Azure HDInsight

В этом учебнике описано, как с помощью Apache Storm записывать данные в HDFS-совместимое хранилище, используемое Apache Storm в HDInsight. HDInsight может использовать службу хранилища Azure и Azure Data Lake Storage в качестве HDFS-совместимого хранилища. Storm предоставляет компонент [HdfsBolt](https://storm.apache.org/releases/current/javadocs/org/apache/storm/hdfs/bolt/HdfsBolt.html), который записывает данные в HDFS. В этой статье предоставляются сведения о записи данных в хранилища обоих типов из HdfsBolt.

Пример топологии, используемый в этом документе, зависит от компонентов, которые входят в состав Storm в HDInsight. Может потребоваться изменить его для работы с Azure Data Lake Storage при использовании с другими кластерами Apache Storm.

В этом руководстве описано следующее.

> [!div class="checklist"]
> * Настройка кластера с помощью действия сценария
> * Сборка и упаковка топологии
> * Развертывание и запуск топологии
> * Просмотр выходных данных
> * Остановка топологии

## <a name="prerequisites"></a>Предварительные условия

* [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support)

* Средство [Apache Maven](https://maven.apache.org/download.cgi), [установленное](https://maven.apache.org/install.html) согласно инструкций Apache.  Maven — система сборки проектов Java.

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* [Схема универсального кода ресурса (URI)](../hdinsight-hadoop-linux-information.md#URI-and-scheme) для основного хранилища кластеров. Для службы хранилища Azure это будет `wasb://`, для Azure Data Lake Storage 2-го поколения — `abfs://` или `adl://` для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure включено безопасное перемещение, URI будет таким: `wasbs://`.  См. также сведения о [безопасной передаче](../../storage/common/storage-require-secure-transfer.md).

### <a name="example-configuration"></a>Пример конфигурации

Следующий YAML-файл является фрагментом файла `resources/writetohdfs.yaml`, который включен в пример. Этот файл определяет топологию Storm с помощью платформы [Flux](https://storm.apache.org/releases/current/flux.html) для Apache Storm.

```yaml
components:
  - id: "syncPolicy"
    className: "org.apache.storm.hdfs.bolt.sync.CountSyncPolicy"
    constructorArgs:
      - 1000

  # Rotate files when they hit 5 MB
  - id: "rotationPolicy"
    className: "org.apache.storm.hdfs.bolt.rotation.FileSizeRotationPolicy"
    constructorArgs:
      - 5
      - "MB"

  - id: "fileNameFormat"
    className: "org.apache.storm.hdfs.bolt.format.DefaultFileNameFormat"
    configMethods:
      - name: "withPath"
        args: ["${hdfs.write.dir}"]
      - name: "withExtension"
        args: [".txt"]

  - id: "recordFormat"
    className: "org.apache.storm.hdfs.bolt.format.DelimitedRecordFormat"
    configMethods:
      - name: "withFieldDelimiter"
        args: ["|"]

# spout definitions
spouts:
  - id: "tick-spout"
    className: "com.microsoft.example.TickSpout"
    parallelism: 1


# bolt definitions
bolts:
  - id: "hdfs-bolt"
    className: "org.apache.storm.hdfs.bolt.HdfsBolt"
    configMethods:
      - name: "withConfigKey"
        args: ["hdfs.config"]
      - name: "withFsUrl"
        args: ["${hdfs.url}"]
      - name: "withFileNameFormat"
        args: [ref: "fileNameFormat"]
      - name: "withRecordFormat"
        args: [ref: "recordFormat"]
      - name: "withRotationPolicy"
        args: [ref: "rotationPolicy"]
      - name: "withSyncPolicy"
        args: [ref: "syncPolicy"]
```

Это YAML-файл определяет следующие элементы:

* `syncPolicy` — определяет, когда файлы синхронизируются или записываются в файловую систему. В этом примере это происходит после каждой 1000 кортежей.
* `fileNameFormat` — определяет шаблон пути и имени файла, используемый при записи файлов. В этом примере путь предоставляется во время выполнения с помощью фильтра, а файл имеет расширение `.txt`.
* `recordFormat` — определяет внутренний формат записанных файлов. В этом примере поля разделяются с помощью символа `|`.
* `rotationPolicy` — определяет, когда чередуются файлы. В этом примере чередование не выполняется.
* `hdfs-bolt` — использует предыдущие компоненты в качестве параметров конфигурации для класса `HdfsBolt`.

Дополнительные сведения о платформе Flux см. здесь: [https://storm.apache.org/releases/current/flux.html](https://storm.apache.org/releases/current/flux.html).

## <a name="configure-the-cluster"></a>Настройка кластера

По умолчанию Storm в HDInsight не содержит компоненты, которые `HdfsBolt` использует для взаимодействия со службой хранилища Azure или Data Lake Storage в пути к классу Storm. Используйте следующее действие скрипта, чтобы добавить эти компоненты в каталог `extlib` для Storm в кластере:

| Свойство | Значение |
|---|---|
|Тип скрипта |- Custom|
|URI bash-скрипта |`https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh`|
|Типы узлов |Nimbus, Supervisor|
|Параметры |Отсутствуют|

Сведения об использовании этого скрипта с кластером см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](./../hdinsight-hadoop-customize-cluster-linux.md).

## <a name="build-and-package-the-topology"></a>Сборка и упаковка топологии

1. Скачайте пример проекта отсюда [https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store](https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store) в среду разработки.

2. С помощью командной строки, терминала или сеанса работы с оболочкой измените каталоги на корень загруженного проекта. Для сборки и создания пакета топологии выполните следующую команду:

    ```cmd
    mvn compile package
    ```

    По завершении сборки и упаковки будет создан каталог `target`, который содержит файл `StormToHdfs-1.0-SNAPSHOT.jar`. Этот файл содержит скомпилированную топологию.

## <a name="deploy-and-run-the-topology"></a>Развертывание и запуск топологии

1. Для копирования топологии в кластер HDInsight воспользуйтесь следующей командой. Замените `CLUSTERNAME` именем кластера.

    ```cmd
    scp target\StormToHdfs-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:StormToHdfs-1.0-SNAPSHOT.jar
    ```

1. После завершения отправки используйте следующую команду для подключения к кластеру HDInsight с помощью SSH. Замените `CLUSTERNAME` именем кластера.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. После подключения создайте файл `dev.properties` с помощью следующей команды.

    ```bash
    nano dev.properties
    ```

1. В качестве содержимого файла `dev.properties` добавьте приведенный ниже текст. При необходимости внесите изменения на основе [схемы URI](../hdinsight-hadoop-linux-information.md#URI-and-scheme).

    ```
    hdfs.write.dir: /stormdata/
    hdfs.url: wasbs:///
    ```

    Чтобы сохранить этот файл, нажмите клавиши __CTRL+X__, введите __Y__ и нажмите клавишу __ВВОД__. Значения в этом файле позволяют задать URL-адрес хранилища и имя каталога, в который записываются данные.

1. Запустите топологию, используя следующую команду.

    ```bash
    storm jar StormToHdfs-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writetohdfs.yaml --filter dev.properties
    ```

    Эта команда запускает топологию с помощью платформы Flux и отправляет ее в узел Nimbus кластера. Топология определяется в файле `writetohdfs.yaml`, включенном в JAR-файл. Файл `dev.properties` передается в качестве фильтра, и топология считывает содержащиеся в нем значения.

## <a name="view-output-data"></a>Просмотр выходных данных

Для просмотра данных используйте следующую команду.

  ```bash
  hdfs dfs -ls /stormdata/
  ```

Отобразится список файлов, созданных с помощью этой топологии. Ниже приведен пример списка, возвращаемого предыдущими командами.

```output
Found 23 items
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-0-1561407909895.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-1-1561407915577.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-10-1561407943327.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-11-1561407946312.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-12-1561407949320.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-13-1561407952662.txt
-rw-r--r--   1 storm supergroup    5242880 2019-06-24 20:25 /stormdata/hdfs-bolt-3-14-1561407955502.txt
```

## <a name="stop-the-topology"></a>Остановка топологии

Топологии Storm будут выполняться до остановки или удаления кластера. Для остановки топологии введите следующую команду:

```bash
storm kill hdfswriter
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим руководством, удалите группу ресурсов. При этом будет удален связанный кластер HDInsight и другие ресурсы, связанные с этой группой ресурсов.

Чтобы удалить группу ресурсов с помощью портала Azure, сделайте следующее:

1. На портале Azure разверните меню слева, чтобы открыть меню служб, а затем выберите __Группы ресурсов__, чтобы просмотреть список групп ресурсов.
2. Найдите группу ресурсов, которую нужно удалить, и щелкните правой кнопкой мыши кнопку __Дополнительно__ (…) справа от списка.
3. Выберите __Удалить группу ресурсов__ и подтвердите выбор.

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы узнали, как с помощью Apache Storm записывать данные в HDFS-совместимое хранилище, используемое Apache Storm в HDInsight.

> [!div class="nextstepaction"]
> Узнайте о других [примерах Apache Storm для HDInsight](apache-storm-example-topology.md)
---
title: Apache Storm с компонентами Python в Azure HDInsight
description: Узнайте, как создать топологию Apache Storm, в которой используются компоненты Python в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-python
ms.date: 12/16/2019
ms.openlocfilehash: e28d21ed71cf5f485165c639a8bd519b3a2736e1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98928989"
---
# <a name="develop-apache-storm-topologies-using-python-on-hdinsight"></a>Разработка топологий Apache Storm с помощью Python в HDInsight

Узнайте, как создать топологию [Apache Storm](https://storm.apache.org/), использующую компоненты Python. Apache Storm поддерживает несколько языков и даже позволяет объединять компоненты из нескольких языков в одной топологии. Платформа [Flux](https://storm.apache.org/releases/current/flux.html) (которая впервые была представлена в Storm 0.10.0) позволяет легко создавать решения, использующие компоненты Python.

> [!IMPORTANT]  
> Сведения в этом документе были проверены с использованием Storm в HDInsight 3.6.

## <a name="prerequisites"></a>Предварительные условия

* Кластер Apache Storm в HDInsight. Ознакомьтесь со статьей [Create Linux-based clusters in HDInsight by using the Azure portal](../hdinsight-hadoop-create-linux-clusters-portal.md) (Создание кластеров под управлением Linux в HDInsight с помощью портала Azure) и выберите **Storm** для параметра **Тип кластера**.

* Среда разработки в локальной среде (необязательно). Локальная среда разработки Storm требуется только в том случае, если вы хотите запускать топологию локально. Дополнительные сведения см. в разделе [Setting up a development environment](https://storm.apache.org/releases/current/Setting-up-development-environment.html) (Настройка среды разработки).

* [Python 2,7 или более поздней версии](https://www.python.org/downloads/).

* [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* Средство [Apache Maven](https://maven.apache.org/download.cgi), [установленное](https://maven.apache.org/install.html) согласно инструкций Apache.  Maven — система сборки проектов Java.

## <a name="storm-multi-language-support"></a>Многоязыковая поддержка Storm

Среда разработки Apache Storm была разработана для работы с компонентами, написанными на любом языке программирования. Компоненты должны "понимать", как работать с определением Thrift для Storm. В рамках проекта Apache Storm предоставляется модуль для Python, который позволяет легко взаимодействовать со Storm. Этот модуль можно найти по адресу [https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py](https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py) .

Storm является процессом Java, который работает на виртуальной машине Java (JVM). Компоненты, написанные на других языках, выполняются как подпроцессы. Storm взаимодействуют с этими подпроцессами с помощью сообщений JSON, отправляемых через стандартные потоки stdin и stdout. Дополнительные сведения о связи между компонентами можно найти в документации по [многоязыковому протоколу](https://storm.apache.org/releases/current/Multilang-protocol.html) .

## <a name="python-with-the-flux-framework"></a>Python с платформой Flux

Платформа Flux позволяет определять топологии Storm отдельно от компонентов. Она использует YAML для определения топологии Storm. Ниже приведен пример указания ссылки на компонент Python в документе YAML.

```yaml
# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "org.apache.storm.flux.wrappers.spouts.FluxShellSpout"
    constructorArgs:
      # Command line
      - ["python", "sentencespout.py"]
      # Output field(s)
      - ["sentence"]
    # parallelism hint
    parallelism: 1
```

Класс `FluxShellSpout` используется для сценария `sentencespout.py`, который реализует spout.

Flux ожидает, что сценарии Python находятся в каталоге `/resources` в JAR-файле, содержащем топологию. Поэтому в этом примере сценарии Python хранятся в каталоге `/multilang/resources`. В `pom.xml` этот файл указан с помощью приведенного ниже кода XML.

```xml
<!-- include the Python components -->
<resource>
    <directory>${basedir}/multilang</directory>
    <filtering>false</filtering>
</resource>
```

Как упоминалось ранее, существует `storm.py` файл, реализующий определение Thrift для работы с расширением. Платформа Flux добавляет `storm.py` автоматически при выполнении сборки проекта, поэтому не нужно беспокоиться о его добавлении.

## <a name="build-the-project"></a>Сборка проекта

1. Скачайте проект из [https://github.com/Azure-Samples/hdinsight-python-storm-wordcount](https://github.com/Azure-Samples/hdinsight-python-storm-wordcount) .

1. Откройте командную строку и перейдите к корневому каталогу проекта: `hdinsight-python-storm-wordcount-master` . Введите следующую команду:

    ```cmd
    mvn clean compile package
    ```

    Она создает файл `target/WordCount-1.0-SNAPSHOT.jar`, содержащий скомпилированную топологию.

## <a name="run-the-storm-topology-on-hdinsight"></a>Запуск топологии Storm в HDInsight

1. Используйте [команду SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) , чтобы скопировать `WordCount-1.0-SNAPSHOT.jar` файл в свой кластер HDInsight. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    scp target/WordCount-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. После завершения передачи файла подключитесь к кластеру с помощью протокола SSH.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. В сеансе SSH используйте следующую команду, чтобы запустить топологию на кластере.

    ```bash
    storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -r -R /topology.yaml
    ```

    После запуска топология Storm выполняется до тех пор, пока она не будет остановлена.

1. Используйте пользовательский интерфейс для просмотра топологии в кластере. Пользовательский интерфейс Storm находится по адресу `https://CLUSTERNAME.azurehdinsight.net/stormui`. Замените `CLUSTERNAME` именем кластера.

1. Останавливает топологию с. Чтобы прерывать топологию в кластере, используйте следующую команду:

    ```bash
    storm kill wordcount
    ```

    Кроме того, можно использовать пользовательский интерфейс. В разделе **действия топологии** для топологии выберите **Kill**.

## <a name="run-the-topology-locally"></a>Локальный запуск топологии

Для локального запуска топологии введите следующую команду.

```bash
storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -l -R /topology.yaml
```

> [!NOTE]  
> Для ее выполнения нужна локальная среда разработки Storm. Дополнительные сведения см. в разделе [Setting up a development environment](https://storm.apache.org/releases/current/Setting-up-development-environment.html) (Настройка среды разработки).

После запуска топология выдает в локальную консоль информацию следующего вида.

```output
24302 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
24302 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting the
24302 [Thread-28] INFO  o.a.s.t.ShellBolt - ShellLog pid:2437, name:counter-bolt Emitting years:160
24302 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=the, count=599}
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=seven, count=302}
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=dwarfs, count=143}
24303 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
24303 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting cow
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=four, count=160}
```

Чтобы остановить топологию, нажмите сочетание клавиш __CTRL+C__.

## <a name="next-steps"></a>Дальнейшие действия

Другие способы использования Python с HDInsight см. в следующих документах: [как использовать пользовательские функции Python (UDF) в Apache Pig и Apache Hive](../hadoop/python-udf-hdinsight.md).
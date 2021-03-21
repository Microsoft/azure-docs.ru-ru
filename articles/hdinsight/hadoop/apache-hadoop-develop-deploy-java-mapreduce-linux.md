---
title: Создание Java MapReduce для Apache Hadoop в Azure HDInsight
description: Узнайте, как использовать Apache Maven, чтобы создать приложение MapReduce на основе Java, а также запустить его с помощью Hadoop в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-java
ms.date: 01/16/2020
ms.openlocfilehash: 7c1a6623883cbee46ba98982808f3c392dc50ffa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946655"
---
# <a name="develop-java-mapreduce-programs-for-apache-hadoop-on-hdinsight"></a>Разработка программ MapReduce на Java для Apache Hadoop в HDInsight

Узнайте, как использовать Apache Maven, чтобы создать приложение MapReduce на основе Java, а также запустить его с помощью Apache Hadoop в Azure HDInsight.

## <a name="prerequisites"></a>Предварительные условия

* [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* Средство [Apache Maven](https://maven.apache.org/download.cgi), [установленное](https://maven.apache.org/install.html) согласно инструкций Apache.  Maven — система сборки проектов Java.

## <a name="configure-development-environment"></a>Настройка среды разработки

Среда, используемая для этой статьи, была компьютер под Windows 10. Команды были выполнены в командной строке, а различные файлы были изменены с помощью блокнота. Измените соответствующим образом для своей среды.

В командной строке введите приведенные ниже команды, чтобы создать рабочую среду.

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>Создание проекта Maven

1. Введите следующую команду, чтобы создать проект Maven с именем **Каталог wordcountjava**:

   ```bash
   mvn archetype:generate -DgroupId=org.apache.hadoop.examples -DartifactId=wordcountjava -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

    Эта команда создает каталог с именем, указанным в `artifactID` параметре (**Каталог wordcountjava** в этом примере). Этот каталог содержит следующие элементы:

    * `pom.xml` — это [объектная модель проекта (POM)](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html), которая содержит сведения и данные конфигурации, использующиеся при сборке проекта;
    * срк\маин\жава\орг\апаче\хадуп\ексамплес: содержит код приложения.
    * срк\тест\жава\орг\апаче\хадуп\ексамплес: содержит тесты для вашего приложения.

1. Удалите созданный код примера. Удалите созданные файлы тестов и приложений `AppTest.java` , а затем `App.java` введите приведенные ниже команды.

    ```cmd
    cd wordcountjava
    DEL src\main\java\org\apache\hadoop\examples\App.java
    DEL src\test\java\org\apache\hadoop\examples\AppTest.java
    ```

## <a name="update-the-project-object-model"></a>Обновление модели объекта проекта

Все справочные материалы по файлу pom.xml см. по адресу https://maven.apache.org/pom.html. Откройте `pom.xml` , введя следующую команду:

```cmd
notepad pom.xml
```

### <a name="add-dependencies"></a>Добавление зависимостей

В `pom.xml` добавьте в раздел следующий текст `<dependencies>` :

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-examples</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-client-common</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
```

Это означает, что требуются библиотеки (перечисленные в параметре &lt;artifactId\>) определенной версии (указанной в параметре &lt;version\>). При компиляции эти зависимости скачиваются из репозитория Maven по умолчанию. Можно воспользоваться [поиском по репозиторию Maven](https://search.maven.org/#artifactdetails%7Corg.apache.hadoop%7Chadoop-mapreduce-examples%7C2.5.1%7Cjar) , чтобы получить дополнительную информацию.

`<scope>provided</scope>` сообщает Maven, что эти зависимости не должны быть упакованы с приложением, так как они будут предоставлены кластером HDInsight во время выполнения.

> [!IMPORTANT]
> Используемая версия должна соответствовать версии Hadoop в кластере. Дополнительные сведения о версиях см. в статье [Что представляют собой различные компоненты и версии Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)

### <a name="build-configuration"></a>Конфигурация построения

Подключаемые модули Maven позволяют настроить этапы сборки проекта. Этот раздел используется для добавления подключаемых модулей, ресурсов и других параметров конфигурации сборки.

Добавьте в файл следующий код `pom.xml` , а затем сохраните и закройте файл. Эти строки должны находиться в файле внутри тегов `<project>...</project>` (например, между тегами `</dependencies>` и `</project>`).

```xml
<build>
    <plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>2.3</version>
        <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
            </transformer>
        </transformers>
        </configuration>
        <executions>
        <execution>
            <phase>package</phase>
                <goals>
                <goal>shade</goal>
                </goals>
        </execution>
        </executions>
        </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.6.1</version>
        <configuration>
        <source>1.8</source>
        <target>1.8</target>
        </configuration>
    </plugin>
    </plugins>
</build>
```

В этом разделе настраивается подключаемый модуль компилятора Apache Maven и подключаемый модуль Apache Maven. Подключаемый модуль компилятора используется для компиляции топологии. Подключаемый модуль shade используется для предотвращения дублирования лицензии в JAR-файле, собранном Maven. Этот подключаемый модуль используется для предотвращения ошибки с дублированием файлов лицензий, которая появляется во время выполнения на кластере HDInsight. Использование maven-shade-plugin с реализацией `ApacheLicenseResourceTransformer` позволяет избежать этой ошибки.

maven-shade-plugin также создает так называемый uber jar, который содержит все зависимости, требуемые для приложения.

Сохраните файл `pom.xml`.

## <a name="create-the-mapreduce-application"></a>Создание приложения MapReduce

1. Введите следующую команду, чтобы создать и открыть новый файл `WordCount.java` . Выберите **Да** в командной строке, чтобы создать новый файл.

    ```cmd
    notepad src\main\java\org\apache\hadoop\examples\WordCount.java
    ```

2. Затем скопируйте и вставьте приведенный ниже код Java в новый файл. Затем закройте файл.

    ```java
    package org.apache.hadoop.examples;

    import java.io.IOException;
    import java.util.StringTokenizer;
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.util.GenericOptionsParser;

    public class WordCount {

        public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context
                        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
            word.set(itr.nextToken());
            context.write(word, one);
            }
        }
    }

    public static class IntSumReducer
            extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                            Context context
                            ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
            sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage: wordcount <in> <out>");
            System.exit(2);
        }
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
        }
    }
    ```

    Обратите внимание, что пакет имеет имя `org.apache.hadoop.examples`, а класс — имя `WordCount`. Эти имена будут использоваться при отправке задания MapReduce.

## <a name="build-and-package-the-application"></a>Сборка и создание пакета приложения

Выполните следующую команду из каталога `wordcountjava`, чтобы собрать JAR-файл, содержащий приложение:

```cmd
mvn clean package
```

Эта команда удаляет остатки предыдущих сборок, скачивает все неустановленные на текущий момент зависимости, а затем собирает и упаковывает приложение.

После выполнения команды каталог `wordcountjava/target` будет содержать файл с именем `wordcountjava-1.0-SNAPSHOT.jar`.

> [!NOTE]
> Файл `wordcountjava-1.0-SNAPSHOT.jar` является uberjar, который содержит не только задание WordCount, но и зависимости, необходимые заданию во время выполнения.

## <a name="upload-the-jar-and-run-jobs-ssh"></a>Передача JAR-файла и запуск заданий (SSH)

В следующих действиях используется команда `scp` для копирования JAR-файла в головной узел Apache HBase в кластере HDInsight. С помощью команды `ssh` выполняется подключение к кластеру; пример запускается непосредственно на головном узле.

1. Отправьте JAR-файл в кластер. Замените `CLUSTERNAME` именем кластера HDInsight, а затем введите следующую команду:

    ```cmd
    scp target/wordcountjava-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. Подключитесь к кластеру. Замените `CLUSTERNAME` именем кластера HDInsight, а затем введите следующую команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. В сеансе SSH используйте следующую команду для запуска приложения MapReduce:

   ```bash
   yarn jar wordcountjava-1.0-SNAPSHOT.jar org.apache.hadoop.examples.WordCount /example/data/gutenberg/davinci.txt /example/data/wordcountout
   ```

    Эта команда запускает приложение WordCount MapReduce. Входной файл — `/example/data/gutenberg/davinci.txt`, а выходной каталог — `/example/data/wordcountout`. Файлы ввода и вывода хранятся в хранилище кластера по умолчанию.

1. По завершении задания воспользуйтесь следующей командой, чтобы просмотреть результат:

   ```bash
   hdfs dfs -cat /example/data/wordcountout/*
   ```

    Должен появиться список слов и чисел со значениями, похожими на приведенные ниже:

    ```output
    zeal    1
    zelus   1
    zenith  2
    ```

## <a name="next-steps"></a>Дальнейшие действия

В этом документе объясняется, как разработать задание MapReduce на Java. Чтобы узнать о других методах работы с HDInsight, см. следующие документы.

* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hdinsight-use-hive.md)
* [Использование MapReduce в Hadoop в HDInsight](hdinsight-use-mapreduce.md)
* [Центр разработчиков Java](https://azure.microsoft.com/develop/java/)
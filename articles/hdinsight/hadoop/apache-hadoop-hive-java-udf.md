---
title: Определяемая пользователем функция Java с Apache Hive Azure HDInsight
description: Узнайте, как создать определяемую пользователем функцию (UDF) на основе Java, которая работает с Apache Hive. В этом примере определяемая пользователем функция преобразует таблицу текстовых строк в нижний регистр.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-java
ms.date: 11/20/2019
ms.openlocfilehash: bf9c2ea544c6b510a0507b6b020f0eae0b101dc7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946564"
---
# <a name="use-a-java-udf-with-apache-hive-in-hdinsight"></a>Использование определяемых пользователем функций Java с Apache Hive в HDInsight

Узнайте, как создать определяемую пользователем функцию (UDF) на основе Java, которая работает с Apache Hive. Определяемая пользователем функция Java в этом примере преобразует таблицу текстовых строк в символы нижнего регистра.

## <a name="prerequisites"></a>Предварительные условия

* Кластер Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](./apache-hadoop-linux-tutorial-get-started.md).
* [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support)
* Средство [Apache Maven](https://maven.apache.org/download.cgi), [установленное](https://maven.apache.org/install.html) согласно инструкций Apache.  Maven — система сборки проектов Java.
* [Схема универсального кода ресурса (URI)](../hdinsight-hadoop-linux-information.md#URI-and-scheme) для основного хранилища кластеров. Это будет wasb://для службы хранилища Azure, abfs://для Azure Data Lake Storage 2-го поколения или adl://для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure включено безопасное перемещение, URI будет таким: `wasbs://`.  См. также сведения о [безопасной передаче](../../storage/common/storage-require-secure-transfer.md).

* Текстовый редактор или Java IDE.

    > [!IMPORTANT]  
    > Если вы создаете файлы Python в клиенте Windows, следует использовать редактор, который использует LF в качестве символа конца строки. Если вы не уверены, используется ли в редакторе символ LF или CRLF, см. раздел [Устранение неполадок](#troubleshooting), в котором описано, как удалять символы CR.

## <a name="test-environment"></a>Тестовая среда

Среда, используемая для этой статьи, была компьютер под Windows 10.  Команды были выполнены в командной строке, а различные файлы были изменены с помощью блокнота. Измените соответствующим образом для своей среды.

В командной строке введите приведенные ниже команды, чтобы создать рабочую среду.

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-an-example-java-udf"></a>Создание примера определяемой пользователем функции Java

1. Создайте новый проект Maven, введя следующую команду:

    ```cmd
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=ExampleUDF -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

    Эта команда создает каталог с именем `exampleudf` , который содержит проект Maven.

2. После создания проекта удалите `exampleudf/src/test` каталог, созданный как часть проекта, введя следующую команду:

    ```cmd
    cd ExampleUDF
    rmdir /S /Q "src/test"
    ```

3. Откройте `pom.xml` , введя следующую команду:

    ```cmd
    notepad pom.xml
    ```

    Затем замените существующую `<dependencies>` запись следующим кодом XML:

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.3</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>1.2.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    ```

    В этих записях указана версия Hadoop и Hive, используемая в кластерах HDInsight 3.6. Сведения о версиях Hadoop и Hive, включенных в HDInsight, можно найти в статье [Что представляют собой различные компоненты Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md) .

    В конце файла добавьте раздел `<build>` перед строкой `</project>`. Этот раздел должен содержать следующий код XML:

    ```xml
    <build>
        <plugins>
            <!-- build for Java 1.8. This is required by HDInsight 3.6  -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <!-- build an uber jar -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.1</version>
                <configuration>
                    <!-- Keep us from getting a can't overwrite file error -->
                    <transformers>
                        <transformer
                                implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
                        </transformer>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer">
                        </transformer>
                    </transformers>
                    <!-- Keep us from getting a bad signature error -->
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
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
        </plugins>
    </build>
    ```

    Эти записи определяют способ построения проекта. В частности, используемую в проекте версию Java и способ построения uberjar для развертывания в кластер.

    После внесения изменений сохраните файл.

4. Введите следующую команду, чтобы создать и открыть новый файл `ExampleUDF.java` :

    ```cmd
    notepad src/main/java/com/microsoft/examples/ExampleUDF.java
    ```

    Затем скопируйте и вставьте приведенный ниже код Java в новый файл. Затем закройте файл.

    ```java
    package com.microsoft.examples;

    import org.apache.hadoop.hive.ql.exec.Description;
    import org.apache.hadoop.hive.ql.exec.UDF;
    import org.apache.hadoop.io.*;

    // Description of the UDF
    @Description(
        name="ExampleUDF",
        value="returns a lower case version of the input string.",
        extended="select ExampleUDF(deviceplatform) from hivesampletable limit 10;"
    )
    public class ExampleUDF extends UDF {
        // Accept a string input
        public String evaluate(String input) {
            // If the value is null, return a null
            if(input == null)
                return null;
            // Lowercase the input string and return it
            return input.toLowerCase();
        }
    }
    ```

    Этот код реализует определяемую пользователем функцию, которая принимает строковое значение и возвращает строку в нижнем регистре.

## <a name="build-and-install-the-udf"></a>Создание и установка определяемой пользователем функции

В приведенных ниже командах замените `sshuser` фактическим именем пользователя, если оно отличается. Замените `mycluster` фактическим именем кластера.

1. Скомпилируйте и упакуйте определяемую пользователем функцию, введя следующую команду:

    ```cmd
    mvn compile package
    ```

    Эта команда выполняет сборку определяемой пользователем функции и упаковывает ее в файл `exampleudf/target/ExampleUDF-1.0-SNAPSHOT.jar`.

2. Используйте `scp` команду, чтобы скопировать файл в кластер HDInsight, введя следующую команду:

    ```cmd
    scp ./target/ExampleUDF-1.0-SNAPSHOT.jar sshuser@mycluster-ssh.azurehdinsight.net:
    ```

3. Подключитесь к кластеру с помощью SSH, введя следующую команду:

    ```cmd
    ssh sshuser@mycluster-ssh.azurehdinsight.net
    ```

4. В открытом сеансе SSH скопируйте JAR-файл в хранилище HDInsight.

    ```bash
    hdfs dfs -put ExampleUDF-1.0-SNAPSHOT.jar /example/jars
    ```

## <a name="use-the-udf-from-hive"></a>Использование определяемой пользователем функции из Hive

1. Запустите клиент Beeline из сеанса SSH, введя следующую команду:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
    ```

    В этой команде предполагается, что вы использовали для кластера учетную запись входа по умолчанию — **admin** .

2. После появления командной строки `jdbc:hive2://localhost:10001/>` введите следующую команду, чтобы добавить определяемую пользователем функцию в Hive и предоставить ее как функцию.

    ```hiveql
    ADD JAR wasbs:///example/jars/ExampleUDF-1.0-SNAPSHOT.jar;
    CREATE TEMPORARY FUNCTION tolower as 'com.microsoft.examples.ExampleUDF';
    ```

3. Используйте определяемую пользователем функцию, чтобы преобразовать значения, полученные из таблицы, в строки нижнего регистра.

    ```hiveql
    SELECT tolower(state) AS ExampleUDF, state FROM hivesampletable LIMIT 10;
    ```

    Этот запрос выбирает состояние из таблицы, преобразует строку в нижний регистр, а затем отображает их вместе с неизмененным именем. Выходные данные выглядят следующим образом:

    ```output
    +---------------+---------------+--+
    |  exampleudf   |     state     |
    +---------------+---------------+--+
    | california    | California    |
    | pennsylvania  | Pennsylvania  |
    | pennsylvania  | Pennsylvania  |
    | pennsylvania  | Pennsylvania  |
    | colorado      | Colorado      |
    | colorado      | Colorado      |
    | colorado      | Colorado      |
    | utah          | Utah          |
    | utah          | Utah          |
    | colorado      | Colorado      |
    +---------------+---------------+--+
    ```

## <a name="troubleshooting"></a>Устранение неполадок

При выполнении задания Hive может возникнуть ошибка, аналогичная приведенной ниже:

```output
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: [Error 20001]: An error occurred while reading or writing to your custom script. It may have crashed with an error.
```

Эта проблема может быть вызвана символами окончания строк в файле Python. Многие редакторы Windows по умолчанию используют символы CRLF, но в приложениях Linux обычно ожидается использование символа LF.

Вы можете использовать следующие команды PowerShell для удаления символов CR перед передачей файла в HDInsight:

```PowerShell
# Set $original_file to the python file path
$text = [IO.File]::ReadAllText($original_file) -replace "`r`n", "`n"
[IO.File]::WriteAllText($original_file, $text)
```

## <a name="next-steps"></a>Дальнейшие действия

Другие способы работы с Hive в HDInsight см. в [этой статье](hdinsight-use-hive.md).

Дополнительные сведения об определяемых пользователем функциях Hive см. в разделе [Hive Operators and User-Defined Functions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF) (Операторы Hive и определяемые пользователем функции) вики-сайта Hive на сайте apache.org.
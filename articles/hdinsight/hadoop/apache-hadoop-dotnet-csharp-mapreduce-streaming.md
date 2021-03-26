---
title: Использование C# с MapReduce в Hadoop в HDInsight в Azure
description: Узнайте, как использовать C# для создания решений MapReduce, использующих Apache Hadoop в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, seoapr2020, devx-track-csharp
ms.date: 04/28/2020
ms.openlocfilehash: 8093db9c53d9c34014d1b315d53539b3f2cffb30
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104866715"
---
# <a name="use-c-with-mapreduce-streaming-on-apache-hadoop-in-hdinsight"></a>Использование C# для потоковой передачи MapReduce в Apache Hadoop в HDInsight

Узнайте, как использовать C# для создания решения MapReduce в HDInsight.

Apache Hadoop потоковая передача позволяет выполнять задания MapReduce с помощью скрипта или исполняемого файла. Здесь .NET используется для реализации модуля сопоставления и сокращения для решения по подсчету слов.

## <a name="net-on-hdinsight"></a>.NET в HDInsight

Кластеры HDInsight используют [Mono https://mono-project.com) (](https://mono-project.com) для запуска приложений .NET). Mono версии 4.2.1 входит в состав HDInsight версии 3.6. Дополнительные сведения о версии Mono, включенной в HDInsight, см. в разделе [Apache Hadoop компоненты, доступные в версиях hdinsight](../hdinsight-component-versioning.md).

Дополнительные сведения о совместимости Mono с различными версиями платформы .NET Framework см. в разделе [Compatibility](https://www.mono-project.com/docs/about-mono/compatibility/) (Совместимость).

## <a name="how-hadoop-streaming-works"></a>Как работает потоковая передача Hadoop

Базовый процесс потоковой передачи в данном документе выглядит следующим образом.

1. Hadoop передает данные в средство сопоставления (*mapper.exe* в этом примере) на stdin.
2. Модуль сопоставления обрабатывает эти данные и передает пары "ключ-значение", разделенные знаками табуляции, в канал STDOUT.
3. Выходные данные считываются Hadoop, а затем передаются в reduce (*reducer.exe* в этом примере) на stdin.
4. Модуль редукции считывает пары "ключ-значение", разделенные знаками табуляции, обрабатывает данные и передает результат в виде пар "ключ-значение", разделенные знаками табуляции, в канал STDOUT.
5. Выходные данные считываются Hadoop и записываются в выходной каталог.

Дополнительные сведения см. в разделе [Потоковая передача Hadoop](https://hadoop.apache.org/docs/r2.7.1/hadoop-streaming/HadoopStreaming.html).

## <a name="prerequisites"></a>Предварительные требования

* приведенному.

* Опыт написания и выполнения сборки кода C#, предназначенного для платформы .NET Framework 4.5.

* Способ передачи EXE-файлов в кластер. В этом документе для передачи файлов в основное хранилище кластера используются средства Data Lake для Visual Studio.

* При использовании PowerShell вам потребуется [модуль Az](/powershell/azure/).

* Кластер Apache Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](../hadoop/apache-hadoop-linux-tutorial-get-started.md).

* Схема универсального кода ресурса (URI) для основного хранилища кластеров. Эта схема будет использоваться `wasb://` для службы хранилища Azure, `abfs://` Azure Data Lake Storage 2-го поколения или `adl://` для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure или Data Lake Storage 2-го поколения включено безопасное перемещение, URI будет иметь значение `wasbs://` или `abfss://` соответственно.

## <a name="create-the-mapper"></a>Создание модуля сопоставления

В Visual Studio создайте новое платформа .NET Framework консольное приложение с именем *сопоставитель*. Используйте для него следующий код.

```csharp
using System;
using System.Text.RegularExpressions;

namespace mapper
{
    class Program
    {
        static void Main(string[] args)
        {
            string line;
            //Hadoop passes data to the mapper on STDIN
            while((line = Console.ReadLine()) != null)
            {
                // We only want words, so strip out punctuation, numbers, etc.
                var onlyText = Regex.Replace(line, @"\.|;|:|,|[0-9]|'", "");
                // Split at whitespace.
                var words = Regex.Matches(onlyText, @"[\w]+");
                // Loop over the words
                foreach(var word in words)
                {
                    //Emit tab-delimited key/value pairs.
                    //In this case, a word and a count of 1.
                    Console.WriteLine("{0}\t1",word);
                }
            }
        }
    }
}
```

После создания приложения создайте его для создания файла *mapper.exe/бин/дебуг/* в каталоге проекта.

## <a name="create-the-reducer"></a>Создание модуля редукции

В Visual Studio создайте новое платформа .NET Framework консольное приложение с именем *reduce*. Используйте для него следующий код.

```csharp
using System;
using System.Collections.Generic;

namespace reducer
{
    class Program
    {
        static void Main(string[] args)
        {
            //Dictionary for holding a count of words
            Dictionary<string, int> words = new Dictionary<string, int>();

            string line;
            //Read from STDIN
            while ((line = Console.ReadLine()) != null)
            {
                // Data from Hadoop is tab-delimited key/value pairs
                var sArr = line.Split('\t');
                // Get the word
                string word = sArr[0];
                // Get the count
                int count = Convert.ToInt32(sArr[1]);

                //Do we already have a count for the word?
                if(words.ContainsKey(word))
                {
                    //If so, increment the count
                    words[word] += count;
                } else
                {
                    //Add the key to the collection
                    words.Add(word, count);
                }
            }
            //Finally, emit each word and count
            foreach (var word in words)
            {
                //Emit tab-delimited key/value pairs.
                //In this case, a word and a count of 1.
                Console.WriteLine("{0}\t{1}", word.Key, word.Value);
            }
        }
    }
}
```

После создания приложения создайте его для создания файла *reducer.exe/бин/дебуг/* в каталоге проекта.

## <a name="upload-to-storage"></a>Отправка в хранилище

Далее необходимо передать приложения модуля *сопоставления* и программы *сокращения* в хранилище HDInsight.

1. В Visual Studio выберите **Просмотреть**  >  **Обозреватель сервера**.

1. Щелкните правой кнопкой мыши **Azure**, выберите **подключиться к Microsoft Azure подписке...** и завершите процесс входа.

1. Разверните кластер HDInsight, в который нужно развернуть это приложение. Отобразится запись с текстом **(Учетная запись хранения по умолчанию)**.

   :::image type="content" source="./media/apache-hadoop-dotnet-csharp-mapreduce-streaming/hdinsight-storage-account.png" alt-text="Учетная запись хранения, кластер HDInsight, обозреватель сервера, Visual Studio" border="true":::

   * Если вы можете расширить запись **(учетную запись хранения по умолчанию)** , вы используете **учетную запись хранения Azure** в качестве хранилища по умолчанию для кластера. Чтобы просмотреть файлы в хранилище по умолчанию для кластера, разверните запись и дважды щелкните элемент **(контейнер по умолчанию)**.

   * Если запись **(учетная запись хранения по умолчанию)** не может быть расширена, вы используете **Azure Data Lake Storage** в качестве хранилища по умолчанию для кластера. Чтобы просмотреть файлы в хранилище по умолчанию кластера, дважды щелкните запись **(Контейнер по умолчанию)**.

1. Чтобы передать EXE-файлы, используйте один из следующих методов.

    * Если вы используете **учетную запись хранения Azure**, щелкните значок **Отправить BLOB-объект** .

        :::image type="content" source="./media/apache-hadoop-dotnet-csharp-mapreduce-streaming/hdinsight-upload-icon.png" alt-text="Значок отправки HDInsight для модуля сопоставления, Visual Studio" border="true":::

        В диалоговом окне **Отправка нового файла** в поле **имя файла** выберите **Обзор**. В диалоговом окне **Отправка большого двоичного объекта** перейдите в папку *bin\Debug* проекта модуля *сопоставления* , а затем выберите файл *mapper.exe* . Наконец, выберите **Открыть** , а затем нажмите **кнопку ОК** , чтобы завершить отправку.

    * Для **Azure Data Lake Storage** щелкните правой кнопкой мыши пустую область в списке файлов и выберите **Отправить**. Наконец, выберите файл *mapper.exe* и нажмите кнопку **Открыть**.

    После завершения передачи *mapper.exe* повторите этот процесс для передачи файла *reducer.exe*.

## <a name="run-a-job-using-an-ssh-session"></a>Выполнение задания с помощью сеанса SSH

Следующая процедура описывает выполнение задания MapReduce с помощью сеанса SSH.

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Используйте одну из приведенных команд для запуска задания MapReduce.

   * Если хранилищем по умолчанию является служба **хранилища Azure**:

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
            -files wasbs:///mapper.exe,wasbs:///reducer.exe \
            -mapper mapper.exe \
            -reducer reducer.exe \
            -input /example/data/gutenberg/davinci.txt \
            -output /example/wordcountout
        ```

    * Если хранилищем по умолчанию является **Data Lake Storage 1-го поколения**:

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
            -files adl:///mapper.exe,adl:///reducer.exe \
            -mapper mapper.exe \
            -reducer reducer.exe \
            -input /example/data/gutenberg/davinci.txt \
            -output /example/wordcountout
        ```

   * Если хранилищем по умолчанию является **Data Lake Storage 2-го поколения**:

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
            -files abfs:///mapper.exe,abfs:///reducer.exe \
            -mapper mapper.exe \
            -reducer reducer.exe \
            -input /example/data/gutenberg/davinci.txt \
            -output /example/wordcountout
        ```

   В следующем списке описывается, что представляет каждый параметр и параметр.

   |Параметр | Описание |
   |---|---|
   |hadoop-streaming.jar|Указывает JAR-файл, содержащий функцию потоковой передачи MapReduce.|
   |— файлы|Указывает *mapper.exe* и *reducer.exe* файлы для этого задания. `wasbs:///` `adl:///` Объявление протокола, или `abfs:///` перед каждым файлом представляет собой путь к корню хранилища по умолчанию для кластера.|
   |-сопоставитель|Указывает файл, который реализует модуль сопоставления.|
   |-reduce|Указывает файл, который реализует класс reduce.|
   |— входные данные|Задает входные данные.|
   |-Output|Указывает выходной каталог.|

1. После завершения задания MapReduce используйте следующую команду, чтобы просмотреть результаты.

   ```bash
   hdfs dfs -text /example/wordcountout/part-00000
   ```

   Ниже приведен пример данных, возвращаемых этой командой.

   ```output
   you     1128
   young   38
   younger 1
   youngest        1
   your    338
   yours   4
   yourself        34
   yourselves      3
   youth   17
   ```

## <a name="run-a-job-using-powershell"></a>Выполнение задания с помощью PowerShell

Используйте приведенный ниже сценарий PowerShell для запуска задания MapReduce и скачивания результатов.

[!code-powershell[main](../../../powershell_scripts/hdinsight/use-csharp-mapreduce/use-csharp-mapreduce.ps1?range=5-87)]

Этот сценарий предлагает ввести имя учетной записи для входа и пароль для кластера, а также имя кластера HDInsight. После завершения задания выходные данные загружаются в файл с именем *output.txt*. Ниже приведен пример выходных данных в файле `output.txt`.

```output
you     1128
young   38
younger 1
youngest        1
your    338
yours   4
yourself        34
yourselves      3
youth   17
```

## <a name="next-steps"></a>Дальнейшие действия

* [Используйте MapReduce в Apache Hadoop в HDInsight](hdinsight-use-mapreduce.md).
* [Используйте определяемую пользователем функцию C# с Apache Hive и Apache Pig](apache-hadoop-hive-pig-udf-dotnet-csharp.md).
* [Разработка программ Java MapReduce](apache-hadoop-develop-deploy-java-mapreduce-linux.md)
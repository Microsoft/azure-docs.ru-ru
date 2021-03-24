---
title: C#, Apache Hive & Apache Pig на Apache Hadoop в Azure HDInsight
description: Узнайте, как использовать определяемые пользователем функции C# при потоковой передаче Apache Hive и Apache Pig в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/06/2019
ms.openlocfilehash: b5682ff58ad827f5a165342f11f03fb49bbe6d2d
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867888"
---
# <a name="use-c-user-defined-functions-with-apache-hive-and-apache-pig-on-apache-hadoop-in-hdinsight"></a>Использование определяемых пользователем функций C# с Apache Hive и Apache Pig на Apache Hadoop в HDInsight

Узнайте, как использовать определяемые пользователем функции C# с [Apache Hive](https://hive.apache.org) и [Apache Pig](https://pig.apache.org) в HDInsight.

> [!IMPORTANT]
> Действия, описанные в этом документе, работают с кластерами HDInsight под управлением Linux. Linux — это единственная операционная система, используемая для работы с HDInsight 3.4 или более поздних версий. Дополнительные сведения см. в разделе [Что представляют собой различные компоненты и версии Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)

Hive и Pig могут передавать данные во внешние приложения для обработки. Этот процесс называется _потоковой передачей_. При использовании приложения .NET данные передаются в него через поток STDIN, после чего приложение возвращает результаты через поток STDOUT. Для чтения и записи с помощью потоков STDIN и STDOUT можно использовать `Console.ReadLine()` и `Console.WriteLine()` из консольного приложения.

## <a name="prerequisites"></a>Предварительные требования

* Опыт написания и выполнения сборки кода C#, предназначенного для платформы .NET Framework 4.5.

    Используйте любую интегрированную среду разработки. Рекомендуется использовать [Visual Studio](https://www.visualstudio.com/vs) или [Visual Studio Code](https://code.visualstudio.com/). В действиях, описанных в этом документе, используется Visual Studio 2019.

* Способ передачи EXE-файлов в кластер и выполнения заданий Pig и Hive. Рекомендуется [Data Lake средства для Visual Studio](../../data-lake-analytics/data-lake-analytics-data-lake-tools-install.md), [Azure PowerShell](/powershell/azure)и [Azure CLI](/cli/azure/install-azure-cli). В этом документе для передачи файлов и выполнения примера запроса Hive используются средства Data Lake для Visual Studio.

    Дополнительные сведения о других способах выполнения запросов Hive см. [в статье что такое Apache Hive и HiveQL в Azure HDInsight?](hdinsight-use-hive.md).

* Hadoop в кластере HDInsight. Дополнительные сведения о создании кластера см. в статье [Создание кластеров HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

## <a name="net-on-hdinsight"></a>.NET в HDInsight

В кластерах *HDInsight под управлением Linux* для запуска приложений .NET используется [Mono (https://mono-project.com)](https://mono-project.com). Mono версии 4.2.1 входит в состав HDInsight версии 3.6.

Дополнительные сведения о совместимости Mono с различными версиями платформы .NET Framework см. в разделе [Compatibility](https://www.mono-project.com/docs/about-mono/compatibility/) (Совместимость).

Дополнительные сведения о версии платформа .NET Framework и Mono, включенных в версии HDInsight, см. в разделе [версии компонентов hdinsight](../hdinsight-component-versioning.md).

## <a name="create-the-c-projects"></a>Создание проектов C\#

В следующих разделах описано, как создать проект C# в Visual Studio для Apache Hive UDF и UDF Apache Pig.

### <a name="apache-hive-udf"></a>Определяемая пользователем функция Apache Hive

Чтобы создать проект C# для Apache Hive UDF:

1. Запустите Visual Studio.

2. Выберите **Создать новый проект**.

3. В окне **Создание нового проекта** выберите шаблон **консольное приложение (платформа .NET Framework)** (версия C#). Выберите **Далее**.

4. В окне **Настройка нового проекта** введите **имя проекта** *HiveCSharp* и перейдите к или создайте **Расположение** для сохранения нового проекта в. Щелкните **Создать**.

5. В интегрированной среде разработки Visual Studio замените содержимое файла *Program. CS* следующим кодом:

    ```csharp
    using System;
    using System.Security.Cryptography;
    using System.Text;
    using System.Threading.Tasks;

    namespace HiveCSharp
    {
        class Program
        {
            static void Main(string[] args)
            {
                string line;
                // Read stdin in a loop
                while ((line = Console.ReadLine()) != null)
                {
                    // Parse the string, trimming line feeds
                    // and splitting fields at tabs
                    line = line.TrimEnd('\n');
                    string[] field = line.Split('\t');
                    string phoneLabel = field[1] + ' ' + field[2];
                    // Emit new data to stdout, delimited by tabs
                    Console.WriteLine("{0}\t{1}\t{2}", field[0], phoneLabel, GetMD5Hash(phoneLabel));
                }
            }
            /// <summary>
            /// Returns an MD5 hash for the given string
            /// </summary>
            /// <param name="input">string value</param>
            /// <returns>an MD5 hash</returns>
            static string GetMD5Hash(string input)
            {
                // Step 1, calculate MD5 hash from input
                MD5 md5 = System.Security.Cryptography.MD5.Create();
                byte[] inputBytes = System.Text.Encoding.ASCII.GetBytes(input);
                byte[] hash = md5.ComputeHash(inputBytes);

                // Step 2, convert byte array to hex string
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < hash.Length; i++)
                {
                    sb.Append(hash[i].ToString("x2"));
                }
                return sb.ToString();
            }
        }
    }
    ```

6. В строке меню выберите **Build**  >  **Build Solution** (сборка сборки), чтобы построить проект.

7. Закройте решение.

### <a name="apache-pig-udf"></a>Определяемая пользователем функция Apache Pig

Чтобы создать проект C# для Apache Hive UDF:

1. Запустите Visual Studio.

2. В окне **Начало работы** выберите **Создать проект**.

3. В окне **Создание нового проекта** выберите шаблон **консольное приложение (платформа .NET Framework)** (версия C#). Выберите **Далее**.

4. В окне **Настройка нового проекта** введите **имя проекта** *PigUDF*, перейдите в каталог или создайте **Расположение** для сохранения нового проекта в. Щелкните **Создать**.

5. В интегрированной среде разработки Visual Studio замените содержимое файла *Program. CS* следующим кодом:

    ```csharp
    using System;

    namespace PigUDF
    {
        class Program
        {
            static void Main(string[] args)
            {
                string line;
                // Read stdin in a loop
                while ((line = Console.ReadLine()) != null)
                {
                    // Fix formatting on lines that begin with an exception
                    if(line.StartsWith("java.lang.Exception"))
                    {
                        // Trim the error info off the beginning and add a note to the end of the line
                        line = line.Remove(0, 21) + " - java.lang.Exception";
                    }
                    // Split the fields apart at tab characters
                    string[] field = line.Split('\t');
                    // Put fields back together for writing
                    Console.WriteLine(String.Join("\t",field));
                }
            }
        }
    }
    ```

    Этот код анализирует строки, отправленные из Pig, и переформатирует строки, которые начинаются с `java.lang.Exception`.

6. В строке меню выберите **Сборка**  >  **построить решение** , чтобы построить проект.

7. Оставьте решение открытым.

## <a name="upload-to-storage"></a>Отправка в хранилище

Затем отправьте приложения UDF Hive и Pig в хранилище в кластере HDInsight.

1. В Visual Studio перейдите к **просмотру**  >  **Обозреватель сервера**.

1. В **Обозреватель сервера** щелкните правой кнопкой мыши **Azure**, выберите **подключиться к Microsoft Azure подписке** и завершите процесс входа.

1. Разверните кластер HDInsight, в который нужно развернуть это приложение. Отобразится запись с текстом **(Учетная запись хранения по умолчанию)**.

    :::image type="content" source="./media/apache-hadoop-hive-pig-udf-dotnet-csharp/hdinsight-storage-account.png" alt-text="Учетная запись хранения по умолчанию, кластер HDInsight обозреватель сервера" border="true":::

    * Если эту запись можно расширить, вы используете **учетную запись хранения Azure** в качестве хранилища по умолчанию для кластера. Чтобы просмотреть файлы в хранилище по умолчанию кластера, разверните эту запись, а затем дважды щелкните запись **(Контейнер по умолчанию)**.

    * Если эту запись невозможно расширить, вы используете **Azure Data Lake Storage** в качестве хранилища по умолчанию для кластера. Чтобы просмотреть файлы в хранилище по умолчанию кластера, дважды щелкните запись **(Контейнер по умолчанию)**.

1. Чтобы передать EXE-файлы, используйте один из следующих методов.

    * Если вы используете **учетную запись хранения Azure**, щелкните значок **Отправить BLOB-объект** .

        :::image type="content" source="./media/apache-hadoop-hive-pig-udf-dotnet-csharp/hdinsight-upload-icon.png" alt-text="Значок отправки HDInsight для нового проекта" border="true":::

        В диалоговом окне **Отправка нового файла** в поле **имя файла** выберите **Обзор**. В диалоговом окне **Отправка большого двоичного объекта** перейдите в папку *bin\Debug* проекта *HiveCSharp* и выберите файл *HiveCSharp.exe* . Наконец, выберите **Открыть** , а затем нажмите **кнопку ОК** , чтобы завершить отправку.

    * Если вы используете **Azure Data Lake Storage**, щелкните правой кнопкой мыши пустую область в списке файлов и выберите команду **Отправить**. Наконец, выберите файл *HiveCSharp.exe* и нажмите кнопку **Открыть**.

    После завершения передачи *HiveCSharp.exe* повторите этот процесс для передачи файла *PigUDF.exe*.

## <a name="run-an-apache-hive-query"></a>Выполнение запроса Apache Hive

Теперь можно выполнить запрос Hive, использующий приложение UDF для Hive.

1. В Visual Studio перейдите к **просмотру**  >  **Обозреватель сервера**.

2. Разверните пункт **Azure**, а затем — **HDInsight**.

3. Щелкните правой кнопкой мыши кластер, в котором развернуто приложение *HiveCSharp*, а затем выберите **Написать запрос Hive**.

4. В качестве запроса Hive используйте следующий текст:

    ```hiveql
    -- Uncomment the following if you are using Azure Storage
    -- add file wasbs:///HiveCSharp.exe;
    -- Uncomment the following if you are using Azure Data Lake Storage Gen1
    -- add file adl:///HiveCSharp.exe;
    -- Uncomment the following if you are using Azure Data Lake Storage Gen2
    -- add file abfs:///HiveCSharp.exe;

    SELECT TRANSFORM (clientid, devicemake, devicemodel)
    USING 'HiveCSharp.exe' AS
    (clientid string, phoneLabel string, phoneHash string)
    FROM hivesampletable
    ORDER BY clientid LIMIT 50;
    ```

    > [!IMPORTANT]
    > Раскомментируйте инструкцию `add file`, которая соответствует типу хранилища по умолчанию для кластера.

    Этот запрос выбирает `clientid` поля, `devicemake` и `devicemodel` из `hivesampletable` , а затем передает поля в приложение *HiveCSharp.exe* . В результате запроса приложение должно возвратить три поля, которые хранятся под именами `clientid`, `phoneLabel` и `phoneHash`. Запрос также будет искать *HiveCSharp.exe* в корне контейнера хранилища по умолчанию.

5. Переключите **Интерактивный** режим на **пакет** по умолчанию, а затем нажмите кнопку **Отправить** , чтобы отправить задание в кластер HDInsight. Откроется окно **Сводка по заданию Hive**.

6. Выберите **Обновить** , чтобы обновить сводку до тех пор, пока **состояние задания** не изменится на **завершено**. Чтобы просмотреть выходные данные задания, выберите **выходные данные задания**.

## <a name="run-an-apache-pig-job"></a>Выполнение задания Apache Pig

Вы также можете запустить задание Pig, которое использует приложение UDF Pig.

1. Подключитесь к кластеру HDInsight с помощью протокола SSH. (Например, выполните команду `ssh sshuser@<clustername>-ssh.azurehdinsight.net` .) Дополнительные сведения см. в разделе [Использование SSH висхдинсигхт](../hdinsight-hadoop-linux-use-ssh-unix.md).

2. Используйте следующую команду, чтобы запустить командную строку Pig:

    ```shell
    pig
    ```

    Откроется командная строка `grunt>`.

3. Чтобы запустить задание Pig, использующее приложение .NET Framework, введите следующую команду.

    ```pig
    DEFINE streamer `PigUDF.exe` CACHE('/PigUDF.exe');
    LOGS = LOAD '/example/data/sample.log' as (LINE:chararray);
    LOG = FILTER LOGS by LINE is not null;
    DETAILS = STREAM LOG through streamer as (col1, col2, col3, col4, col5);
    DUMP DETAILS;
    ```

    `DEFINE`Инструкция создает псевдоним `streamer` для приложения *PigUDF.exe* и `CACHE` загружает его из хранилища по умолчанию для кластера. Позже `streamer` используется с `STREAM` оператором для обработки отдельных строк, содержащихся в `LOG` , и возврата данных в виде последовательности столбцов.

    > [!NOTE]
    > Имя приложения, используемое для потоковой передачи, должно быть заключено в \` escape-символ (Обратная кавычка) при использовании псевдонима, а также символом "(одинарная цитата), если используется с `SHIP` .

4. После ввода последней строки запустится задание. Появится результат, аналогичный приведенному ниже тексту.

    ```output
    (2019-07-15 16:43:25 SampleClass5 [WARN] problem finding id 1358451042 - java.lang.Exception)
    (2019-07-15 16:43:25 SampleClass5 [DEBUG] detail for id 1976092771)
    (2019-07-15 16:43:25 SampleClass5 [TRACE] verbose detail for id 1317358561)
    (2019-07-15 16:43:25 SampleClass5 [TRACE] verbose detail for id 1737534798)
    (2019-07-15 16:43:25 SampleClass7 [DEBUG] detail for id 1475865947)
    ```

5. Используйте `exit` для выхода из Pig.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как использовать платформа .NET Frameworkное приложение из Hive и Pig в HDInsight. Чтобы узнать об использовании Python с Hive и Pig, ознакомьтесь со статьей [Использование определяемых пользователем функций Python с Apache Hive и Apache Pig в HDInsight](python-udf-hdinsight.md).

Другие способы использования Hive и сведения об использовании MapReduce см. в следующих статьях:

* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hdinsight-use-hive.md)
* [Использование MapReduce в Hadoop в HDInsight](hdinsight-use-mapreduce.md)
* [Основы Pig Latin](https://pig.apache.org/docs/latest/basic.html)

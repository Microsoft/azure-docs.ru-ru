---
title: Apache Hive & Data Lake инструменты для Visual Studio — Azure HDInsight
description: Узнайте, как использовать Средства Data Lake для Visual Studio с целью выполнения запросов Apache Hive с Apache Hadoop в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/27/2019
ms.openlocfilehash: 3e54991f6afd2ace19bf7b70e57bda6f551f7780
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98943158"
---
# <a name="run-apache-hive-queries-using-the-data-lake-tools-for-visual-studio"></a>Выполнение запросов Apache Hive с использованием средств Data Lake для Visual Studio

Узнайте, как использовать Средства Data Lake для Visual Studio с целью выполнения запросов Apache Hive. Средства Data Lake позволяют легко создавать, отправлять и отслеживать запросы Hive к Apache Hadoop в Azure HDInsight.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Hadoop в HDInsight. Сведения о создании этого элемента см. [в статье Создание кластера Apache Hadoop в Azure HDInsight с помощью шаблона диспетчер ресурсов](./apache-hadoop-linux-tutorial-get-started.md).

* [Visual Studio](https://visualstudio.microsoft.com/vs/). В действиях, описанных в этой статье, используется Visual Studio 2019.

* Средства HDInsight для Visual Studio или инструменты Azure Data Lake Tools для Visual Studio. Сведения об установке и настройке средств см. в разделе [Install Data Lake Tools for Visual Studio](apache-hadoop-visual-studio-tools-get-started.md#install-data-lake-tools-for-visual-studio).

## <a name="run-apache-hive-queries-using-the-visual-studio"></a> Выполнение запросов Apache Hive с помощью Visual Studio

Есть два способа создания и выполнения запросов Hive.

* Создание специальных запросов.
* Создайте приложение Hive.

### <a name="create-an-ad-hoc-hive-query"></a>Создание нерегламентированного запроса Hive

Нерегламентированные запросы можно выполнять в **пакетном** или **интерактивном** режиме.

1. Запустите **Visual Studio** и выберите **Продолжить без кода**.

2. В **Обозреватель сервера** щелкните правой кнопкой мыши **Azure**, выберите **подключиться к Microsoft Azure подписке...** и завершите процесс входа.

3. Разверните **HDInsight**, щелкните правой кнопкой мыши кластер, в котором нужно выполнить запрос, и выберите команду **написать запрос Hive**.

4. Введите следующий запрос Hive:

    ```hql
    SELECT * FROM hivesampletable;
    ```

5. Нажмите кнопку **Выполнить**. Режим выполнения по умолчанию — **Interactive**.

    ![Выполнение интерактивного запроса Hive, Visual Studio](./media/apache-hadoop-use-hive-visual-studio/vs-execute-hive-query.png)

6. Чтобы выполнить тот же запрос в **пакетном** режиме, переключите раскрывающийся список с **интерактивного** на **пакет**. Кнопка выполнения изменится с **EXECUTE** на **Submit**.

    ![Отправка запроса Hive пакетной службы, Visual Studio](./media/apache-hadoop-use-hive-visual-studio/visual-studio-batch-query.png)

    Редактор Hive поддерживает технологию IntelliSense. Средства Data Lake для Visual Studio поддерживают загрузку удаленных метаданных при редактировании скрипта Hive. Например, если ввести `SELECT * FROM` , IntelliSense выводит список всех предлагаемых имен таблиц. Если указано имя таблицы, IntelliSense выведет список имен столбцов. Эти инструменты поддерживают почти все инструкции, подзапросы и встроенные определяемые пользователем функции Hive DML. IntelliSense предлагает только метаданные кластеров, выбранных на панели инструментов HDInsight.

7. На панели инструментов запроса (область под вкладкой запрос и выше текста запроса) либо выберите **Отправить**, либо щелкните стрелку раскрывающегося списка рядом с пунктом **Отправить** и выберите пункт **Дополнительно** в раскрывающийся список. Если выбрать второй вариант,

8. Если выбран параметр дополнительные отправки, настройте **имя задания**, **аргументы**, **дополнительные конфигурации** и **Каталог состояния** в диалоговом окне **Отправка скрипта** . Затем нажмите кнопку **Отправить**.

    ![Диалоговое окно "Отправка скрипта", запрос Hive в HDInsight Hadoop](./media/apache-hadoop-use-hive-visual-studio/vs-tools-submit-jobs-advanced.png)

### <a name="create-a-hive-application"></a>Создание приложения Hive

Чтобы выполнить запрос Hive путем создания приложения Hive, выполните следующие действия.

1. Откройте **Visual Studio**.

2. В окне **Начало работы** выберите **Создать проект**.

3. В окне **Создание нового проекта** в поле **Поиск шаблонов** введите *Hive*. Затем выберите **приложение Hive** и нажмите кнопку **Далее**.

4. В окне **Настройка нового проекта** введите **имя проекта**, выберите или создайте **Расположение** для нового проекта, а затем щелкните **создать**.

5. Откройте файл **Script.hql** , созданный в этом проекте, и вставьте следующие операторы HiveQL:

    ```hql
    set hive.execution.engine=tez;
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION '/example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND  INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
    ```

    Эти инструкции выполняют следующие действия:

    * `DROP TABLE`: Удаляет таблицу, если она существует.

    * `CREATE EXTERNAL TABLE`: создает новую "внешнюю" таблицу в Hive. Внешние таблицы хранят только определения таблицы в Hive. (Данные остаются в исходном расположении.)

        > [!NOTE]  
        > Внешние таблицы следует использовать, если предполагается, что базовые данные будут обновляться внешним источником, например заданием MapReduce или службой Azure.
        >
        > Удаление внешней таблицы **не** приводит к удалению данных, будет удалено только определение таблицы.

    * `ROW FORMAT`: инструкции по форматированию данных для Hive. В данном случае поля всех журналов разделены пробелом.

    * `STORED AS TEXTFILE LOCATION`: Указывает Hive, что данные хранятся в каталоге *example/Data* и хранятся в виде текста.

    * `SELECT`: Выбирает количество строк, в которых столбец `t4` содержит значение `[ERROR]` . Эта инструкция возвращает значение `3` , поскольку три строки содержат это значение.

    * `INPUT__FILE__NAME LIKE '%.log'`: Сообщает Hive, что следует возвращать данные только из файлов, оканчивающихся на. log. Это предложение ограничит поиск до файла *Sample. log* , содержащего данные.

6. На панели инструментов файл запроса (с похожим видом на панель инструментов ad-hoc Query) выберите кластер HDInsight, который вы хотите использовать для этого запроса. Затем измените **Interactive** на **Batch** (при необходимости) и нажмите кнопку **Отправить** , чтобы выполнить инструкции в качестве задания Hive.

   Отобразится **сводка по заданию Hive** и информация о его выполнении. Воспользуйтесь ссылкой **Обновить**, чтобы обновить информацию о задании. Обновляйте ее до тех пор, пока **состояние задания** не изменится на **Завершено**.

   ![Завершенные сводки по заданиям Hive, приложение Hive, Visual Studio](./media/apache-hadoop-use-hive-visual-studio/hdinsight-job-summary.png)

7. Выберите **выходные данные задания** , чтобы просмотреть выходные данные этого задания. Они содержат `[ERROR] 3`. Это значение, возвращенное данным запросом.

### <a name="additional-example"></a>Дополнительный пример

В следующем примере используется `log4jLogs` Таблица, созданная в предыдущей процедуре, и [создается приложение Hive](#create-a-hive-application).

1. В **Обозреватель сервера** щелкните правой кнопкой мыши кластер и выберите **написать запрос Hive**.

2. Введите следующий запрос Hive:

    ```hql
    set hive.execution.engine=tez;
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
    ```

    Эти инструкции выполняют следующие действия:

    * `CREATE TABLE IF NOT EXISTS`: Создает таблицу, если она еще не существует. Поскольку `EXTERNAL` ключевое слово не используется, эта инструкция создает внутреннюю таблицу. Внутренние таблицы хранятся в хранилище данных Hive и управляются Hive.

        > [!NOTE]  
        > В отличие от таблиц `EXTERNAL`, удаление внутренней таблицы приводит к удалению ее базовых данных.

    * `STORED AS ORC`: Сохраняет данные в формате *оптимизированной строки по столбцам* (ORC). Это высокооптимизированный и эффективный формат для хранения данных Hive.

    * `INSERT OVERWRITE ... SELECT`: выбирает строки из таблицы `log4jLogs`, которые содержат `[ERROR]`, а затем вставляет эти данные в таблицу `errorLogs`.

3. При необходимости измените **Интерактивный** для **пакета** , а затем нажмите кнопку **Отправить**.

4. Чтобы убедиться, что задание создало таблицу, перейдите в раздел **Обозреватель сервера** и разверните узел **Azure**  >  **HDInsight**. Разверните свой кластер HDInsight, а затем разверните **базы данных Hive**  >  **по умолчанию**. Должны быть отображены таблицы **errorLogs** и **log4jLogs**.

## <a name="next-steps"></a>Дальнейшие действия

Как вы видите, средства HDInsight для Visual Studio упрощают работу с запросами Hive в HDInsight.

* Общие сведения о Hive в HDInsight см. в статье [что такое Apache Hive и HiveQL в Azure hdinsight?](hdinsight-use-hive.md)

* Дополнительные сведения о других способах работы с Hadoop в HDInsight см. [в статье Использование MapReduce в Apache Hadoop в hdinsight](hdinsight-use-mapreduce.md) .

* Дополнительные сведения о средствах HDInsight для Visual Studio см. в статье[использование средств Data Lake для Visual Studio для подключения к Azure HDInsight и выполнения запросов Apache Hive](apache-hadoop-visual-studio-tools-get-started.md)

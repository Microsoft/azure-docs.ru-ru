---
title: Использование представления Hive Apache Ambari с Apache Hadoop в Azure HDInsight
description: Узнайте, как использовать представление Hive в веб-браузере для отправки запросов Hive. Представление Hive — это компонент веб-интерфейса Ambari, поставляемого с кластером HDInsight на основе Linux.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: 87a4d3960937450713747fa16bd473b4c34eff0e
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867871"
---
# <a name="use-apache-ambari-hive-view-with-apache-hadoop-in-hdinsight"></a>Использование представления Hive Apache Ambari с Apache Hadoop в HDInsight

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

Узнайте, как выполнять запросы Hive с использованием представления Hive Apache Ambari. Представление Hive позволяет создавать, оптимизировать и выполнять запросы Hive из веб-браузера.

## <a name="prerequisites"></a>Предварительные требования

Кластер Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](./apache-hadoop-linux-tutorial-get-started.md).

## <a name="run-a-hive-query"></a>Выполнение запроса Hive

1. На [портале Azure](https://portal.azure.com/) выберите свой кластер.  Инструкции см. в разделе [список и отображение кластеров](../hdinsight-administer-use-portal-linux.md#showClusters) . Кластер открывается в новом представлении портала.

1. На **панели мониторинга кластера** выберите **представления Ambari**. Если запрашивается проверка подлинности, используйте имя пользователя и пароль учетной записи входа в кластер (по умолчанию — `admin`), указанные при создании кластера. Можно также выбрать `https://CLUSTERNAME.azurehdinsight.net/#/main/views` в браузере `CLUSTERNAME` , где — это имя кластера.

1. В списке представлений выберите __Представление Hive__.

    :::image type="content" source="./media/apache-hadoop-use-hive-ambari-view/select-apache-hive-view.png" alt-text="Выберите Apache Hive представление Apache Ambari" border="true":::

    Страница представления Hive выглядит следующим образом:

    :::image type="content" source="./media/apache-hadoop-use-hive-ambari-view/ambari-worksheet-view.png" alt-text="Изображение листа запроса для представления Hive" border="true":::

1. На вкладке __Запрос__ вставьте в лист следующие инструкции HiveQL:

    ```hiveql
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs(
        t1 string,
        t2 string,
        t3 string,
        t4 string,
        t5 string,
        t6 string,
        t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION '/example/data/';
    SELECT t4 AS loglevel, COUNT(*) AS count FROM log4jLogs
        WHERE t4 = '[ERROR]'
        GROUP BY t4;
    ```

    Эти инструкции выполняют следующие действия:

    |Инструкция | Описание |
    |---|---|
    |DROP TABLE|удаляет таблицу и файл данных, если таблица уже существует.|
    |CREATE EXTERNAL TABLE|создает "внешнюю" таблицу в Hive. Внешние таблицы хранят только определение таблицы в Hive. Данные остаются в исходном расположении.|
    |ФОРМАТ СТРОКИ|показывает настройку форматирования данных. В данном случае поля всех журналов разделены пробелом.|
    |ХРАНИТСЯ КАК РАСПОЛОЖЕНИЕ TEXTFILE|показывает место хранения данных и их формат (текст).|
    |SELECT|выбирает подсчет количества строк, в которых столбец t4 содержит значение [ERROR].|

   > [!IMPORTANT]  
   > Оставьте для параметра __База данных__ значение __по умолчанию__. В примерах в этом документе используется база данных по умолчанию, входящая в состав HDInsight.

1. Чтобы запустить запрос, выберите **выполнить** под листом. Кнопка станет оранжевой, а текст изменится на **Остановить**.

1. Когда запрос будет выполнен, на вкладке **Результаты** появятся результаты этой операции. Вот пример результата запроса:

    ```output
    loglevel       count
    [ERROR]        3
    ```

    Для просмотра сведений журнала, созданных заданием, можно использовать вкладку **Журнал** .

   > [!TIP]  
   > Скачайте или сохраните результаты из раскрывающегося диалогового окна **действия** на вкладке  **результаты** .

### <a name="visual-explain"></a>Визуальное объяснение

Чтобы отобразить визуализацию плана запроса, выберите под листом вкладку **Visual Explain** (Визуальное пояснение).

Представление запроса **Visual Explain** (Визуальное объяснение) помогает разобраться в потоке сложных запросов.

### <a name="tez-ui"></a>Пользовательский интерфейс Tez

Чтобы отобразить пользовательский интерфейс Tez для запроса, выберите вкладку **Пользовательский интерфейс Tez** под листом.

> [!IMPORTANT]  
> Tez используется не для всех запросов. Многие запросы можно разрешить и без применения Tez.

## <a name="view-job-history"></a>Просмотреть журнал заданий

На вкладке __Задания__ отображается журнал запросов Hive.

:::image type="content" source="./media/apache-hadoop-use-hive-ambari-view/apache-hive-job-history.png" alt-text="Журнал вкладок Apache Hive просмотра заданий" border="true":::

## <a name="database-tables"></a>Таблицы базы данных

Вкладку __Таблицы__ можно использовать для работы с таблицами в базе данных Hive.

:::image type="content" source="./media/apache-hadoop-use-hive-ambari-view/hdinsight-tables-tab.png" alt-text="Изображение вкладки «Apache Hive таблицы»" border="true":::

## <a name="saved-queries"></a>Сохраненные запросы

На вкладке **запрос** можно при необходимости сохранить запросы. После сохранения запроса можно повторно использовать его из вкладки __Saved Queries__ (Сохраненные запросы).

:::image type="content" source="./media/apache-hadoop-use-hive-ambari-view/ambari-saved-queries.png" alt-text="Вкладка «Apache Hive представлений сохраненных запросов»" border="true":::

> [!TIP]  
> Запросы сохраняются в системе хранения данных кластера по умолчанию. Сохраненные запросы можно найти в следующем расположении: `/user/<username>/hive/scripts`. Они хранятся в виде обычных текстовых файлов `.hql`.
>
> Если вы удалите кластер, но сохраните хранилище, для извлечения запросов можно использовать такую служебную программу, как [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/) или обозреватель хранилища Data Lake (на [портале Azure](https://portal.azure.com)).

## <a name="user-defined-functions"></a>Пользовательские функции

Инфраструктуру Hive можно расширить с помощью определяемых пользователем функций (UDF). Они позволяют реализовать функции или логику, сложно моделируемые в HiveQL.

Объявлять и сохранять наборы определяемых пользователем функций можно с помощью вкладки **UDF** вверху представления Hive. Эти функции могут использоваться в **редакторе запросов**.

:::image type="content" source="./media/apache-hadoop-use-hive-ambari-view/user-defined-functions.png" alt-text="Apache Hive просмотре вкладки пользовательских функций" border="true":::

В нижней части **редактора запросов** появится кнопка **Вставить UDF** . Эта запись отображает раскрывающийся список пользовательских функций, определенных в представлении Hive. Выбирая определяемую пользователем функцию, вы добавляете в запрос соответствующие инструкции HiveQL.

Например, если определена определяемая пользователем функция со следующими свойствами:

* имя ресурса — myudfs;

* путь к ресурсу — /myudfs.jar;

* имя определяемой пользователем функции — myawesomeudf;

* имя класса определяемой пользователем функции — com.myudfs.Awesome.

Нажав кнопку **Insert udfs** (Вставить определяемые пользователем функции), вы увидите запись с именем **myudfs**, содержащую раскрывающийся список для каждой функции, определяемой для этого ресурса. В нашем примере это функция **myawesomeudf**. Если вы выберете эту запись, в начало запроса будет добавлен следующий код:

```hiveql
add jar /myudfs.jar;
create temporary function myawesomeudf as 'com.myudfs.Awesome';
```

Затем вы можете использовать эту функцию в своем запросе. Например, `SELECT myawesomeudf(name) FROM people;`.

Дополнительные сведения об использовании определяемых пользователем функций с Hive в HDInsight см. в следующих статьях:

* [Использование определяемых пользователем функций Python с Apache Hive и Apache Pig в HDInsight](python-udf-hdinsight.md)
* [Использование определяемых пользователем функций Java с Apache Hive в HDInsight](./apache-hadoop-hive-java-udf.md)

## <a name="hive-settings"></a>Параметры Hive

Вы можете изменять различные настройки Hive. Например, для изменения механизма выполнения для Hive с Tez (значение по умолчанию) на MapReduce.

## <a name="next-steps"></a>Дальнейшие действия

Общая информация о Hive в HDInsight:

* [Использование Apache Hive с Apache Hadoop в HDInsight](hdinsight-use-hive.md)
* [Use the Apache Beeline client with Apache Hive](apache-hadoop-use-hive-beeline.md) (Использование клиента Apache Beeline с Apache Hive)

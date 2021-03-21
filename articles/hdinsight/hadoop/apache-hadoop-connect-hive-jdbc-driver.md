---
title: Отправка запросов в Apache Hive с помощью драйвера JDBC — Azure HDInsight
description: Используйте драйвер JDBC из приложения Java для отправки запросов Apache Hive в Hadoop в службе HDInsight. Подключение можно осуществлять программными средствами и из клиента SQL SQuirrel.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 04/20/2020
ms.openlocfilehash: d23b376384262c208fed70306e62634592d0b46b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946761"
---
# <a name="query-apache-hive-through-the-jdbc-driver-in-hdinsight"></a>Отправка запросов в Apache Hive с помощью драйвера JDBC в HDInsight

[!INCLUDE [ODBC-JDBC-selector](../../../includes/hdinsight-selector-odbc-jdbc.md)]

Узнайте, как использовать драйвер JDBC из приложения Java. Отправка запросов Apache Hive в Apache Hadoop в Azure HDInsight. Сведения в этом документе показывают, как подключаться программным способом и из клиента SQuirreL SQL.

Дополнительные сведения об интерфейсе JDBC Hive см. в статье [HiveJDBCInterface](https://cwiki.apache.org/confluence/display/Hive/HiveJDBCInterface).

## <a name="prerequisites"></a>Предварительные условия

* Кластер HDInsight Hadoop. Дополнительные сведения о создании кластера см. в статье [Приступая к работе с Hadoop в HDInsight](apache-hadoop-linux-tutorial-get-started.md). Убедитесь, что служба HiveServer2 запущена.
* [Пакет Java Developer Kit (JDK) версии 11](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html) или более поздней.
* [SQUIRREL SQL](http://squirrel-sql.sourceforge.net/). SQuirreL представляет собой клиентское приложение JDBC.

## <a name="jdbc-connection-string"></a>Строка подключения JDBC

Подключения JDBC к кластеру HDInsight в Azure осуществляются через порт 443. Трафик защищен с помощью TLS/SSL. Открытый шлюз, за которым находятся кластеры, перенаправляет трафик к порту, который фактически прослушивается HiveServer2. В следующей строке подключения показан формат для HDInsight:

```http
    jdbc:hive2://CLUSTERNAME.azurehdinsight.net:443/default;transportMode=http;ssl=true;httpPath=/hive2
```

Замените `CLUSTERNAME` на имя вашего кластера HDInsight.

Вы также можете получить подключение через **Пользовательский интерфейс Ambari > Hive > конфигурации > Advanced**.

![Получение строки подключения JDBC через Ambari](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-get-connection-string-through-ambari.png)

### <a name="host-name-in-connection-string"></a>Имя узла в строке подключения

Имя узла "CLUSTERNAME.azurehdinsight.net" в строке подключения совпадает с URL-адресом кластера. Его можно получить с помощью портал Azure.

### <a name="port-in-connection-string"></a>Порт в строке подключения

**Порт 443** можно использовать только для подключения к кластеру из некоторых мест за пределами виртуальной сети Azure. HDInsight — это управляемая служба, которая означает, что все подключения к кластеру управляются через безопасный шлюз. Вы не можете подключиться к HiveServer 2 напрямую через порты 10001 или 10000. Эти порты не видны снаружи.

## <a name="authentication"></a>Аутентификация

При установке подключения используйте имя администратора кластера HDInsight и пароль для проверки подлинности. Из клиентов JDBC, таких как SQuirreL SQL, введите имя администратора и пароль в параметрах клиента.

При подключении из приложения Java необходимо использовать имя пользователя и пароль. Например, следующий код Java открывает новое соединение:

```java
DriverManager.getConnection(connectionString,clusterAdmin,clusterPassword);
```

## <a name="connect-with-squirrel-sql-client"></a>Подключение с использованием клиента SQuirreL SQL

SQuirreL SQL — клиент JDBC, который можно использовать для удаленного выполнения запросов Hive с кластером HDInsight. В следующих шагах предполагается, что SQuirreL SQL уже установлен.

1. Создайте каталог, в котором будут храниться определенные файлы, копируемые из кластера.

2. В следующем сценарии замените `sshuser` именем учетной записи пользователя SSH для кластера.  Замените `CLUSTERNAME` именем кластера HDInsight.  В командной строке измените рабочий каталог на созданный на предыдущем шаге, а затем введите следующую команду, чтобы скопировать файлы из кластера HDInsight:

    ```cmd
    scp sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/usr/hdp/current/hadoop-client/{hadoop-auth.jar,hadoop-common.jar,lib/log4j-*.jar,lib/slf4j-*.jar,lib/curator-*.jar} .

    scp sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/usr/hdp/current/hive-client/lib/{commons-codec*.jar,commons-logging-*.jar,hive-*-*.jar,httpclient-*.jar,httpcore-*.jar,libfb*.jar,libthrift-*.jar} .
    ```

3. Запустите приложение SQuirreL SQL. В левой части окна выберите **Драйверы**.

    ![Вкладка "Драйверы" в левой части окна](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-squirreldrivers.png)

4. Среди значков в верхней части диалогового окна **Drivers** (Драйверы) щелкните значок **+** для создания драйвера.

    ![Значок драйверов приложений SQuirreL SQL](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-driversicons.png)

5. В диалоговом окне Add Driver (Добавление драйвера) укажите следующие сведения.

    |Свойство. | Значение |
    |---|---|
    |Имя|Hive|
    |Пример URL-адреса|`jdbc:hive2://localhost:443/default;transportMode=http;ssl=true;httpPath=/hive2`|
    |Дополнительный путь к классу|Используйте кнопку **Добавить** , чтобы добавить все скачанные ранее JAR-файлы.|
    |Имя класса|org. Apache. Hive. JDBC. HiveDriver|

   ![диалоговое окно добавления драйвера с параметрами](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-add-driver.png)

   Нажмите кнопку **ОК** , чтобы сохранить эти параметры.

6. В левой части окна SQuirreL SQL выберите **Псевдонимы**. Затем щелкните **+** значок, чтобы создать псевдоним подключения.

    !["SQuirreL SQL добавить новый псевдоним"](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-new-aliases.png)

7. Используйте следующие значения в диалоговом окне **Добавление псевдонима** :

    |Свойство. |Значение |
    |---|---|
    |Имя|Hive в HDInsight|
    |Драйвер|Используйте раскрывающийся список, чтобы выбрать драйвер **Hive** .|
    |URL-адрес|`jdbc:hive2://CLUSTERNAME.azurehdinsight.net:443/default;transportMode=http;ssl=true;httpPath=/hive2`. Замените **CLUSTERNAME** именем кластера HDInsight.|
    |Имя пользователя|Имя пользователя для учетной записи входа кластера HDInsight. Значение по умолчанию — **Admin**.|
    |Пароль|Пароль для учетной записи входа кластера.|

    ![диалоговое окно добавления псевдонима с параметрами](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-addalias-dialog.png)

    > [!IMPORTANT]
    > Нажмите кнопку **Проверить**, чтобы убедиться, что подключение работает. При появлении диалогового окна **Connect to: Hive on HDInsight** (Подключение: Hive в HDInsight) выберите **Подключиться**, чтобы выполнить проверку. Если проверка пройдет успешно, вы увидите диалоговое окно **Connection successful** (Подключение выполнено успешно). При возникновении ошибки см. раздел [Устранение неполадок](#troubleshooting).

    Чтобы сохранить псевдоним подключения, в нижней части диалогового окна **Add Alias** (Добавить псевдоним) нажмите кнопку **ОК**.

8. В раскрывающемся списке **Подключиться к** в верхней части окна SQuirreL SQL выберите **Hive on HDInsight** (Hive в HDInsight). При появлении запроса выберите **Подключиться**.

    ![диалоговое окно соединения с параметрами](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-connect-dialog.png)

9. После подключения введите следующий запрос в диалоговом окне SQL-запрос, а затем щелкните значок **запуска** (работающего человека). В области результатов должны появиться результаты запроса.

    ```hiveql
    select * from hivesampletable limit 10;
    ```

    ![диалоговое окно запроса SQL с результатами запроса](./media/apache-hadoop-connect-hive-jdbc-driver/hdinsight-sqlquery-dialog.png)

## <a name="connect-from-an-example-java-application"></a>Подключение из примера приложения Java

Пример использования клиента Java для запроса Hive в HDInsight можно найти по адресу [https://github.com/Azure-Samples/hdinsight-java-hive-jdbc](https://github.com/Azure-Samples/hdinsight-java-hive-jdbc) . Следуйте инструкциям в репозитории, чтобы построить и запустить образец.

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="unexpected-error-occurred-attempting-to-open-an-sql-connection"></a>Непредвиденная ошибка при попытке открыть подключение к SQL

**Признаки.** При подключении к кластеру HDInsight версии 3.3 или более поздней может появиться сообщение о возникновении непредвиденной ошибки. Трассировка стека для этой ошибки начинается со следующих строк:

```java
java.util.concurrent.ExecutionException: java.lang.RuntimeException: java.lang.NoSuchMethodError: org.apache.commons.codec.binary.Base64.<init>(I)V
at java.util.concurrent.FutureTas...(FutureTask.java:122)
at java.util.concurrent.FutureTask.get(FutureTask.java:206)
```

**Причина.** Эту ошибку вызывают прежние версии файла commons-codec.jar в составе SQuirreL.

**Решение**. чтобы устранить эту ошибку, выполните следующие действия.

1. Выйдите из SQuirreL, а затем перейдите в каталог, где SQuirreL установлен в системе, возможно, `C:\Program Files\squirrel-sql-4.0.0\lib` . В каталоге SquirreL в каталоге `lib` замените существующий файл commons-codec.ja файлом, скачанным из кластера HDInsight.

1. Перезапустите SQuirreL. При последующих подключениях к Hive в HDInsight ошибка больше не должна возникать.

### <a name="connection-disconnected-by-hdinsight"></a>Подключение отключено с помощью HDInsight

**Симптомы**. при попытке загрузить огромный объем данных (например, несколько ГБ) через JDBC/ODBC соединение неожиданно отключается из HDInsight при скачивании.

**Причина**: Эта ошибка вызвана ограничением на узлы шлюза. При получении данных из JDBC/ODBC все данные должны проходить через узел шлюза. Однако шлюз не предназначен для загрузки огромного объема данных, поэтому шлюз может закрыть подключение, если он не может обработать трафик.

**Решение**. Избегайте использования драйвера JDBC/ODBC для загрузки огромных объемов данных. Вместо этого следует копировать данные непосредственно из хранилища BLOB-объектов.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как использовать JDBC для работы с Hive, воспользуйтесь следующими ссылками для изучения других способов работы с Azure HDInsight.

* [Визуализируйте Apache Hive данные с помощью Microsoft Power BI в Azure HDInsight](apache-hadoop-connect-hive-power-bi.md).
* [Визуализация данных Hive в интерактивном запросе с помощью Power BI в Azure HDInsight](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md).
* [Подключение Excel к Hadoop в Azure HDInsight с помощью Microsoft Hive ODBC Driver](apache-hadoop-connect-excel-hive-odbc-driver.md)
* [Подключите Excel к Apache Hadoop с помощью Power Query](apache-hadoop-connect-excel-power-query.md).
* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hdinsight-use-hive.md)
* [Использование Pig с Hadoop в HDInsight](../use-pig.md)
* [Использование заданий MapReduce с HDInsight](hdinsight-use-mapreduce.md)

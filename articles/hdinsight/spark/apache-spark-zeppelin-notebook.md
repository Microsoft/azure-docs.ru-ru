---
title: Записные книжки Zeppelin & Apache Spark кластере Azure HDInsight
description: Пошаговые инструкции по использованию записных книжек Zeppelin с кластерами Apache Spark в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: ba58c007dc0bc3776f429366651cb79404b137b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98928475"
---
# <a name="use-apache-zeppelin-notebooks-with-apache-spark-cluster-on-azure-hdinsight"></a>Использование записных книжек Apache Zeppelin с кластером Apache Spark в Azure HDInsight

Кластеры HDInsight Spark включают записные книжки [Apache Zeppelin](https://zeppelin.apache.org/) . Используйте записные книжки для запуска заданий Apache Spark. Из этой статьи вы узнаете, как использовать записную книжку Zeppelin в кластере HDInsight.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md).
* Схема универсального кода ресурса (URI) для основного хранилища кластеров. Схема будет использоваться `wasb://` для хранилища BLOB-объектов Azure, `abfs://` для Azure Data Lake Storage 2-го поколения или `adl://` для Azure Data Lake Storage 1-го поколения. Если для хранилища BLOB-объектов включено безопасное перемещение, URI будет иметь значение `wasbs://` .  Дополнительные сведения см. [в статье обязательное безопасное перемещение в службе хранилища Azure](../../storage/common/storage-require-secure-transfer.md) .

## <a name="launch-an-apache-zeppelin-notebook"></a>Запуск записной книжки Apache Zeppelin

1. В разделе **Общие сведения о** кластере Spark в разделе **панели мониторинга кластера** выберите **Записная книжка Zeppelin** . Введите учетные данные администратора для кластера.  

   > [!NOTE]  
   > Также можно открыть Zeppelin Notebook для своего кластера, открыв следующий URL-адрес в браузере. Замените **CLUSTERNAME** именем кластера:
   >
   > `https://CLUSTERNAME.azurehdinsight.net/zeppelin`

2. Создайте новую записную книжку. На панели заголовка перейдите к **записной книжке**  >  **создать новое примечание**.

    ![Создание новой записной книжки Zeppelin](./media/apache-spark-zeppelin-notebook/hdinsight-create-zeppelin-notebook.png "Создание новой записной книжки Zeppelin")

    Введите имя записной книжки, а затем выберите **создать заметку**.

3. Убедитесь, что в заголовке записной книжки отображается состояние подключено. Он обозначается зеленой точкой в правом верхнем углу.

    ![Состояния записной книжки Zeppelin](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-connected.png "Состояния записной книжки Zeppelin")

4. Загрузите демонстрационные данные во временную таблицу. При создании кластера Spark в HDInsight файл образца данных `hvac.csv` копируется в связанную учетную запись хранения в разделе `\HdiSamples\SensorSampleData\hvac` .

    В пустой абзац, созданный по умолчанию в новой записной книжке, вставьте следующий фрагмент кода.

    ```scala
    %livy2.spark
    //The above magic instructs Zeppelin to use the Livy Scala interpreter

    // Create an RDD using the default Spark context, sc
    val hvacText = sc.textFile("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

    // Define a schema
    case class Hvac(date: String, time: String, targettemp: Integer, actualtemp: Integer, buildingID: String)

    // Map the values in the .csv file to the schema
    val hvac = hvacText.map(s => s.split(",")).filter(s => s(0) != "Date").map(
        s => Hvac(s(0),
                s(1),
                s(2).toInt,
                s(3).toInt,
                s(6)
        )
    ).toDF()

    // Register as a temporary table called "hvac"
    hvac.registerTempTable("hvac")
    ```

    Нажмите клавиши **SHIFT + ВВОД** или нажмите кнопку **воспроизвести** для абзаца, чтобы выполнить фрагмент кода. Состояние, которое отображается в правом верхнем углу абзаца, должно изменяться в следующей последовательности: READY (ГОТОВО), PENDING (ОЖИДАЕТ), RUNNING (ВЫПОЛНЯЕТСЯ) и FINISHED (ЗАВЕРШЕНО). Выходные данные отображаются в нижней части того же абзаца. Снимок экрана выглядит как на следующем рисунке:

    ![Создание временной таблицы из необработанных данных](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-load-data.png "Создание временной таблицы из необработанных данных")

    Можно указать заголовок для каждого абзаца. В правом углу абзаца выберите значок **параметров** (спроккет), а затем щелкните **отобразить заголовок**.  

    > [!NOTE]  
    > Интерпретатор %spark2 не поддерживается в блокнотах Zeppelin во всех версиях HDInsight, а интерпретатор %sh не поддерживается в HDInsight версии 4.0 и выше.

5. Теперь можно выполнять инструкции Spark SQL для `hvac` таблицы. Вставьте следующий запрос в новый абзац. Запрос получает идентификатор здания. Также разница между целевой и фактической температурами для каждого здания в заданной дате. Нажмите **SHIFT + ВВОД**.

    ```sql
    %sql
    select buildingID, (targettemp - actualtemp) as temp_diff, date from hvac where date = "6/1/13"
    ```  

    Инструкция **%sql** в начале сообщает записной книжке, что необходимо использовать интерпретатор Livy Scala.

6. Щелкните значок **линейчатой диаграммы** , чтобы изменить отображение.  **Параметры**, отображаемые после выбора **линейчатой диаграммы**, позволяют выбрать **ключи** и **значения**.  Выходные данные показаны на снимке экрана ниже.

    ![Выполнение инструкции Spark SQL с помощью notebook1](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-spark-query-1.png "Выполнение инструкции Spark SQL с помощью notebook1")

7. Можно также запустить инструкции Spark SQL с помощью переменных в запросе. В следующем фрагменте кода показано, как определить переменную `Temp` в запросе с возможными значениями, с которыми необходимо выполнить запрос. При первом выполнении запроса раскрывающийся список автоматически заполняется значениями, указанными для переменной.

    ```sql
    %sql  
    select buildingID, date, targettemp, (targettemp - actualtemp) as temp_diff from hvac where targettemp > "${Temp = 65,65|75|85}"
    ```

    Вставьте этот фрагмент кода в новый абзац и нажмите клавиши **SHIFT + ВВОД**. Затем выберите **65** в раскрывающемся списке **TEMP** .

8. Щелкните значок **линейчатой диаграммы** , чтобы изменить отображение.  Затем выберите **Параметры** и внесите следующие изменения:

   * **Группы:**  Добавьте **таржеттемп**.  
   * **Значения:** одного. Удалить **дату**.  2. Добавьте **temp_diff**.  3.  Измените агрегатор с **Sum** на **AVG**.  

     Выходные данные показаны на снимке экрана ниже.

     ![Выполнение инструкции Spark SQL с помощью notebook2](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-spark-query-2.png "Выполнение инструкции Spark SQL с помощью notebook2")

## <a name="how-do-i-use-external-packages-with-the-notebook"></a>Использование внешних пакетов с записной книжкой

Записная книжка Zeppelin в кластере Apache Spark в HDInsight может использовать внешние, участвующие в сообществе пакеты, которые не включены в кластер. Полный список доступных пакетов можно найти в [репозитории Maven](https://search.maven.org/) . Его также можно получить из других источников. Например, полный список предоставленных сообществом пакетов можно найти в разделе [Пакеты Spark](https://spark-packages.org/).

В этой статье вы узнаете, как использовать пакет [Spark-CSV](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar) с Jupyter Notebook.

1. Откройте параметры интерпретатора. В правом верхнем углу выберите имя пользователя, выполнившего вход, а затем выберите **интерпретатор**.

    ![Запуск интерпретатора](./media/apache-spark-zeppelin-notebook/zeppelin-launch-interpreter.png "Выходные данные Hive")

2. Прокрутите до **livy2**, а затем выберите **изменить**.

    ![Изменение интерпретатора запуска1](./media/apache-spark-zeppelin-notebook/zeppelin-use-external-package-1.png "Изменение интерпретатора запуска1")

3. Перейдите к разделу key `livy.spark.jars.packages` и задайте его значение в формате `group:id:version` . Если вы хотите использовать пакет [spark-csv](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar), для ключа необходимо задать значение `com.databricks:spark-csv_2.10:1.4.0`.

    ![Изменение интерпретатора Settings2](./media/apache-spark-zeppelin-notebook/zeppelin-use-external-package-2.png "Изменение интерпретатора Settings2")

    Выберите **сохранить** , а затем нажмите **кнопку ОК** , чтобы перезапустить интерпретатор Livy.

4. Вот как можно получить значение указанного выше ключа.

    а. Найдите пакет в репозитории Maven. В этой статье мы использовали [Spark-CSV](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar).

    b. В репозитории найдите значения для параметров **GroupId**, **ArtifactId** и **Version**.

    ![Использование внешних пакетов с Jupyter Notebook](./media/apache-spark-zeppelin-notebook/use-external-packages-with-jupyter.png "Использование внешних пакетов с Jupyter Notebook")

    c. Объедините три значения, разделив их двоеточием (**:**).

    ```
    com.databricks:spark-csv_2.10:1.4.0
    ```

## <a name="where-are-the-zeppelin-notebooks-saved"></a>Место сохранения записных книжек Zeppelin

Записные книжки Zeppelin сохраняются на головных узлах кластера. Поэтому при удалении кластера записные книжки также будут удалены. Если вы хотите сохранить записные книжки для последующего использования в других кластерах, необходимо экспортировать их после выполнения заданий. Чтобы экспортировать записную книжку, выберите значок **экспорта** , как показано на рисунке ниже.

![Скачивание записной книжки](./media/apache-spark-zeppelin-notebook/zeppelin-download-notebook.png "Скачать записную книжку")

Это действие сохраняет записную книжку в виде JSON-файла в расположении загрузки.

## <a name="use-shiro-to-configure-access-to-zeppelin-interpreters-in-enterprise-security-package-esp-clusters"></a>Использование `Shiro` для настройки доступа к интерпретаторам Zeppelin в кластерах корпоративный пакет безопасности (ESP)

Как отмечалось выше, `%sh` интерпретатор не поддерживается из HDInsight 4,0. Более того, поскольку `%sh` интерпретатор создает потенциальные проблемы безопасности, такие как доступ к кэйтабс с помощью команд оболочки, он также удаляется из кластеров HDInsight 3,6 ESP. Это означает `%sh` , что интерпретатор недоступен при нажатии кнопки **создать новую заметку** или в пользовательском интерфейсе интерпретатора по умолчанию.

Пользователи привилегированного домена могут использовать этот `Shiro.ini` файл для управления доступом к пользовательскому интерфейсу интерпретатора. Только эти пользователи могут создавать новые `%sh` интерпретаторы и устанавливать разрешения для каждого нового `%sh` интерпретатора. Чтобы управлять доступом с помощью `shiro.ini` файла, выполните следующие действия.

1. Определите новую роль, используя существующее имя группы домена. В следующем примере `adminGroupName` — это группа привилегированных пользователей в AAD. Не используйте в имени группы специальные символы и пробелы. Символы после `=` предоставляют разрешения для этой роли. `*` означает, что группа имеет полные разрешения.

    ```
    [roles]
    adminGroupName = *
    ```

2. Добавьте новую роль для доступа к интерпретаторам Zeppelin. В следующем примере всем пользователям в предоставляется `adminGroupName` доступ к интерпретаторам Zeppelin и могут создаваться новые интерпретаторы. Можно разместить несколько ролей между квадратными скобками в `roles[]` , разделяя их запятыми. Затем пользователи, имеющие необходимые разрешения, могут получить доступ к интерпретаторам Zeppelin.

    ```
    [urls]
    /api/interpreter/** = authc, roles[adminGroupName]
    ```

## <a name="livy-session-management"></a>Управление сеансом Livy

Первый абзац кода в записной книжке Zeppelin создает новый сеанс Livy в кластере. Этот сеанс является общим для всех Zeppelin записных книжек, которые вы создадите позднее. Если по какой-либо причине сеанс Livy будет прерван, задания не будут выполняться из записной книжки Zeppelin.

В этом случае перед началом выполнения заданий из записной книжки Zeppelin необходимо выполнить следующие действия.  

1. Перезапустите интерпретатор Livy из записной книжки Zeppelin. Для этого откройте параметры интерпретатора, выбрав имя вошедшего в систему пользователя в правом верхнем углу, а затем выберите **интерпретатор**.

    ![Запуск интерпретатора](./media/apache-spark-zeppelin-notebook/zeppelin-launch-interpreter.png "Выходные данные Hive")

2. Прокрутите до **livy2**, а затем выберите **перезапустить**.

    ![Перезапуск интерпретатора Livy](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-restart-interpreter.png "Перезапуск интерпретатора Zeppelin")

3. Запустите ячейку кода из имеющейся записной книжки Zeppelin. Этот код создает новый сеанс Livy в кластере HDInsight.

## <a name="general-information"></a>Общие сведения

### <a name="validate-service"></a>Проверка службы

Чтобы проверить службу из Ambari, перейдите в `https://CLUSTERNAME.azurehdinsight.net/#/main/services/ZEPPELIN/summary` папку, где имя_кластера — имя кластера.

Чтобы проверить службу из командной строки, подключитесь к головному узлу по протоколу SSH. Переключите пользователя в Zeppelin с помощью команды `sudo su zeppelin` . Команды состояния:

|Команда |Описание |
|---|---|
|`/usr/hdp/current/zeppelin-server/bin/zeppelin-daemon.sh status`|Состояние службы.|
|`/usr/hdp/current/zeppelin-server/bin/zeppelin-daemon.sh --version`|Версия службы.|
|`ps -aux | grep zeppelin`|Определение PID.|

### <a name="log-locations"></a>Расположения журналов

|Служба |Путь |
|---|---|
|Zeppelin — сервер|/уср/хдп/куррент/зеппелин-сервер/|
|Журналы сервера|/вар/лог/зеппелин|
|Интерпретатор конфигурации, `Shiro` , site.xml, log4j|/УСР/ХДП/куррент/Зеппелин-сервер/конф или/ЕТК/Зеппелин/конф|
|Каталог PID|/вар/рун/зеппелин|

### <a name="enable-debug-logging"></a>Включение ведения журнала отладки

1. Перейдите в `https://CLUSTERNAME.azurehdinsight.net/#/main/services/ZEPPELIN/summary` папку, где имя_кластера — имя кластера.

1. Перейдите в раздел **конфигурации**  >  **Дополнительно Zeppelin-log4j-Properties**  >  **log4j_properties_content**.

1. Измените `log4j.appender.dailyfile.Threshold = INFO` на `log4j.appender.dailyfile.Threshold = DEBUG` .

1. Добавьте `log4j.logger.org.apache.zeppelin.realm=DEBUG`.

1. Сохраните изменения и перезапустите службу.

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)
* [Ядра, доступные для Jupyter Notebook в кластере Apache Spark для HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Установка записной книжки Jupyter на компьютере и ее подключение к кластеру Apache Spark в Azure HDInsight (предварительная версия)](apache-spark-jupyter-notebook-install-locally.md)

---
title: 'Azure Toolkit for IntelliJ: приложение Spark — HDInsight'
description: Сведения о разработке приложений Spark на языке Scala и их отправке в кластер HDInsight Spark с помощью набора средств Azure для IntelliJ.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 04/14/2020
ms.openlocfilehash: ced066623d23c6894745374ace812454c52f3109
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103420023"
---
# <a name="use-azure-toolkit-for-intellij-to-create-apache-spark-applications-for-hdinsight-cluster"></a>Использование Azure Toolkit for IntelliJ для создания приложений Apache Spark для кластера HDInsight

В этой статье показано, как разрабатывать Apache Spark приложения в Azure HDInsight с помощью подключаемого модуля **набора средств Azure** для интегрированной среды разработки IntelliJ. [Azure HDInsight](../hdinsight-overview.md) — это управляемая служба аналитики с открытым кодом в облаке. Служба позволяет использовать платформы с открытым кодом, такие как Hadoop, Apache Spark, Apache Hive и Apache Kafka.

Подключаемый модуль **набора средств Azure** можно использовать несколькими способами.

* Разработка и отправка приложения Scala Spark в кластер HDInsight Spark.
* получать доступ к ресурсам кластера Azure HDInsight Spark;
* разрабатывать и запускать приложения Scala Spark в локальной среде.

Вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * Использование подключаемого модуля Azure Toolkit for IntelliJ
> * Разработка приложений Apache Spark
> * Отправка приложения в кластер Azure HDInsight

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md).

* [Комплект разработчика Oracle Java](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).  В этой статье используется Java версии 8.0.202.

* Среда IntelliJ IDEA. В этой статье используется среда [IntelliJ IDEA версии  2018.3.4](https://www.jetbrains.com/idea/download/).

* Azure Toolkit for IntelliJ.  Дополнительные сведения см. в статье [Установка набора средств Azure для IntelliJ](/azure/developer/java/toolkit-for-intellij/).

## <a name="install-scala-plugin-for-intellij-idea"></a>Установка подключаемого модуля Scala для IntelliJ IDEA

Шаги для установки подключаемого модуля Scala:

1. Откройте IntelliJ IDEA.

2. На экране приветствия выберите **Configure** (Настройка) > **Plugins** (Подключаемые модули), чтобы открыть окно **подключаемых модулей**.

    ![IntelliJная идея включает подключаемый модуль Scala](./media/apache-spark-intellij-tool-plugin/enable-scala-plugin1.png)

3. Выберите **Install** (Установить) в области подключаемого модуля Scala в новом окне.  

    ![IntelliJная идея устанавливает подключаемый модуль Scala](./media/apache-spark-intellij-tool-plugin/install-scala-plugin.png)

4. После успешной установки подключаемого модуля необходимо перезапустить интегрированную среду разработки.

## <a name="create-a-spark-scala-application-for-an-hdinsight-spark-cluster"></a>Создание приложения Spark Scala для кластера HDInsight Spark

1. Запустите IntelliJ IDEA и выберите **Create New Project** (Создать проект), чтобы открыть окно **New Project** (Новый проект).

2. На левой панели щелкните **Azure Spark/HDInsight**.

3. В главном окне выберите **Spark Project (Scala)** (Проект Spark (Scala)).

4. В раскрывающемся списке **средство построения** выберите один из следующих вариантов.
   * **Maven** для поддержки мастера создания проекта Scala.
   * **SBT** для управления зависимостями и создания проекта Scala.

     ![Диалоговое окно нового проекта в IntelliJ IDEA](./media/apache-spark-intellij-tool-plugin/create-hdi-scala-app.png)

5. Выберите **Далее**.

6. В окне **New Project** (Новый проект) укажите следующую информацию:  

    |  Свойство   | Описание   |  
    | ----- | ----- |  
    |Имя проекта| Введите имя.  Для этой статьи используется `myApp`.|  
    |Project&nbsp;location (Расположение проекта)| Введите расположение для сохранения проекта.|
    |Project SDK (Пакет SDK проекта)| Это поле может быть пустым при первом использовании идеи.  Выберите **New...** (Создать...) и перейдите к JDK.|
    |Версия Spark|Мастер создания интегрирует правильную версию пакетов SDK для Spark и Scala. Если используется версия кластера Spark более ранняя, чем 2.0, выберите **Spark 1.x**. В противном случае выберите **Spark 2.x**. В этом примере используется **Spark 2.3.0 (Scala 2.11.8)** .|

    ![Выбор пакета SDK для Apache Spark](./media/apache-spark-intellij-tool-plugin/intellij-new-project.png)

7. Нажмите кнопку **Готово**.  Может пройти несколько минут, прежде чем проект станет доступным.

8. Проект Spark автоматически создает артефакт. Чтобы просмотреть артефакт, выполните указанные ниже действия.

   а. В строке меню выберите **File** (Файл) > **Project Structure...** (Структура проекта…).

   b. В окне **структуры проекта** щелкните **Artifacts** (Артефакты).  

   c. Нажмите **кнопку Отмена**  после просмотра артефакта.

      ![Сведения об артефакте в диалоговом окне](./media/apache-spark-intellij-tool-plugin/default-artifact-dialog.png)

9. Добавьте исходный код приложения, выполнив следующие действия.

    а. В Project перейдите к папке **MyApp**  >  **src**  >  **Main**  >  **Scala**.  

    b. Щелкните правой кнопкой мыши папку **scala**, а затем выберите **New** (Новый) > **Scala Class** (Класс Scala).

   ![Команды для создания класса Scala в проекте](./media/apache-spark-intellij-tool-plugin/hdi-spark-scala-code.png)

   c. В диалоговом окне **Create New Scala Class** (Создание класса Scala) введите имя, в раскрывающемся списке **Kind** (Вид) выберите **Object** (Объект) и нажмите кнопку **ОК**.

     ![Диалоговое окно создания класса Scala](./media/apache-spark-intellij-tool-plugin/hdi-spark-scala-code-object.png)

   d. После этого файл **MyApp.scala** откроется в главном представлении. Замените код по умолчанию на приведенный ниже:  

    ```scala
    import org.apache.spark.SparkConf
    import org.apache.spark.SparkContext

    object myApp{
        def main (arg: Array[String]): Unit = {
        val conf = new SparkConf().setAppName("myApp")
        val sc = new SparkContext(conf)

        val rdd = sc.textFile("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

        //find the rows that have only one digit in the seventh column in the CSV file
        val rdd1 =  rdd.filter(s => s.split(",")(6).length() == 1)

        rdd1.saveAsTextFile("wasbs:///HVACOut")
        }

    }
    ```

    Этот код считывает данные из файла HVAC.csv (доступного для всех кластеров HDInsight Spark), извлекает строки, содержащие только одну цифру в седьмом столбце CSV-файла, и записывает результат в `/HVACOut` в используемом по умолчанию контейнере хранилища для кластера.

## <a name="connect-to-your-hdinsight-cluster"></a>Подключение к кластеру HDInsight

Пользователь может [войти в подписку Azure](#sign-in-to-your-azure-subscription)или [связать кластер HDInsight](#link-a-cluster). Чтобы подключиться к кластеру HDInsight, используйте учетные данные пользователя Ambari/пароль или присоединенные к домену.

### <a name="sign-in-to-your-azure-subscription"></a>Войдите в свою подписку Azure.

1. В строке меню выберите **Представление** > **Окно инструментов** > **Azure Explorer**.

   ![IntelliJ идея показывает Azure Explorer](./media/apache-spark-intellij-tool-plugin/show-azure-explorer1.png)

2. В Azure Explorer щелкните правой кнопкой мыши узел **Azure**, а затем выберите **Войти**.

   ![Щелчок правой кнопкой мыши в Azure Explorer для IntelliJ IDEA](./media/apache-spark-intellij-tool-plugin/explorer-rightclick-azure.png)

3. В диалоговом окне **Вход в Azure** выберите **Имя пользователя устройства**, а затем — **Войти**.

    ![' IntelliJ идея login устройства входа в Azure](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer2.png)

4. В диалоговом окне **Azure Device Login** (Вход в систему устройства Azure) щелкните **Copy&Open** (Копировать и открыть).

   ![' IntelliJ идея loginа устройства Azure](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer5.png)

5. В интерфейсе браузера вставьте код и нажмите кнопку **Далее**.

   !["Майкрософт введите диалоговое окно кода для HDI"](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer6.png)

6. Введите учетные данные Azure и закройте браузер.

   ![' Microsoft введите диалоговое окно электронной почты для HDI '](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer7.png)

7. После входа в диалоговом окне **Select Subscriptions** (Выбор подписок) будут перечислены все подписки Azure, связанные с указанными учетными данными. Выберите свою подписку, а затем нажмите кнопку **Выбрать**.

    ![Диалоговое окно выбора подписок](./media/apache-spark-intellij-tool-plugin/Select-Subscriptions.png)

8. В **Azure Explorer** разверните **HDInsight**, чтобы просмотреть кластеры HDInsight Spark в своей подписке.

    ![Основное представление IntelliJ IDEA в Azure Explorer](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer3.png)

9. Далее можно развернуть узел имени кластера, чтобы увидеть ресурсы (например, учетные записи хранения), связанные с ним.

    ![Учетные записи хранения Azure Explorer](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer4.png)

### <a name="link-a-cluster"></a>Связывание кластера

Можно связать кластер HDInsight с помощью управляемого имени пользователя Apache Ambari. Аналогичным образом, присоединенный к домену кластер HDInsight можно связать с помощью домена и имени пользователя, например `user1@contoso.com`. Также вы можете связать кластер службы Livy.

1. В строке меню выберите **Представление** > **Окно инструментов** > **Azure Explorer**.

1. В Azure Explorer щелкните правой кнопкой мыши узел **HDInsight**, а затем — **Link A Cluster** (Связать кластер).

   ![Контекстное меню связывания кластера Azure Explorer](./media/apache-spark-intellij-tool-plugin/link-a-cluster-context-menu.png)

1. Доступные параметры в окне **Link A Cluster** (Связывание кластера) зависят от значения, которое вы выбрали из раскрывающегося списка **Link Resource Type** (Связать тип ресурса).  Введите свои значения, а затем нажмите кнопку **ОК**.

    * **Кластер HDInsight**  
  
        |Свойство. |Значение |
        |----|----|
        |Link Resource Type (Связать тип ресурса)|Из раскрывающегося списка выберите **HDInsight Cluster** (Кластер HDInsight).|
        |Cluster Name/URL (Имя или URL-адрес кластера)| Введите имя кластера.|
        |тип проверки подлинности;| Оставьте значение **Basic Authentication** (Обычная аутентификация)|
        |Имя пользователя| Введите имя пользователя кластера. Значение по умолчанию — admin.|
        |Пароль| Введите пароль для этого имени пользователя.|

        ![IntelliJ IDEA диалоговое окно связывания кластера](./media/apache-spark-intellij-tool-plugin/link-hdinsight-cluster-dialog.png)

    * **Livy Service** (Служба Livy)  
  
        |Свойство. |Значение |
        |----|----|
        |Link Resource Type (Связать тип ресурса)|Из раскрывающегося списка выберите **Livy Service** (Служба Livy).|
        |Livy Endpoint (Конечная точка Livy)| Введите конечную точку Livy|
        |Имя кластера,| Введите имя кластера.|
        |Yarn Endpoint (Конечная точка Yarn)|Необязательный параметр.|
        |тип проверки подлинности;| Оставьте значение **Basic Authentication** (Обычная аутентификация)|
        |Имя пользователя| Введите имя пользователя кластера. Значение по умолчанию — admin.|
        |Пароль| Введите пароль для этого имени пользователя.|

        ![IntelliJ IDEA диалоговое окно связывания кластера Livy](./media/apache-spark-intellij-tool-plugin/link-livy-cluster-dialog.png)

1. В узле **HDInsight** отобразится связанный кластер.

   ![Связанный кластер1 Azure Explorer](./media/apache-spark-intellij-tool-plugin/hdinsight-linked-cluster.png)

1. В **обозревателе Azure** также можно удалить связь кластера.

   ![Несвязанный кластер Azure Explorer](./media/apache-spark-intellij-tool-plugin/hdi-unlinked-cluster.png)

## <a name="run-a-spark-scala-application-on-an-hdinsight-spark-cluster"></a>Запуск приложения Spark Scala в кластере HDInsight Spark

После создания приложение Scala можно отправить в кластер.

1. В проекте перейдите в **myApp** > **src** > **main** > **scala** > **myApp**.  Щелкните правой кнопкой мыши папку **myApp** и выберите **Submit Spark Application** (Запустить приложение Spark). Этот параметр скорее всего будет расположен в нижней части списка.

      ![Команда Submit Spark Application to HDInsight (Отправить приложение Spark в HDInsight)](./media/apache-spark-intellij-tool-plugin/hdi-submit-spark-app-1.png)

2. В диалоговом окне **Отправка приложения Spark** выберите **1. Spark в HDInsight**.

3. В окне **Edit configuration** (Изменить конфигурацию) укажите следующие значения, а затем щелкните **ОК**.

    |Свойство. |Значение |
    |----|----|
    |Кластеры Spark (только Linux)|Выберите кластер HDInsight Spark, в котором вы хотите запустить приложение.|
    |Select an Artifact to submit (Выбор артефакта для запуска)|Оставьте параметр по умолчанию.|
    |Имя главного класса|значение по умолчанию — имя главного класса из выбранного файла. Класс можно изменить, нажав кнопку с многоточием ( **...** ) и выбрав другой класс.|
    |Job configurations (Конфигурация заданий)|Можно изменить ключи и значения по умолчанию, или. Дополнительные сведения см. в статье [Apache Livy REST API](https://livy.incubator.apache.org/docs/latest/rest-api.html).|
    |Аргументы командной строки|При необходимости можно ввести аргументы для основного класса, разделив их пробелом.|
    |Referenced Jars (Ссылки на JAR-файлы) и Referenced Files (Ссылки на файлы)|можно ввести пути к используемым JAR и файлам, если они есть. Вы можете также просматривать файлы в виртуальной файловой системе Azure, которая сейчас поддерживает только кластер ADLS 2 поколения. Дополнительные сведения см. в [Apache Spark конфигурации](https://spark.apache.org/docs/latest/configuration.html#runtime-environment).  Ознакомьтесь также со статьей [Краткое руководство. Использование Обозревателя службы хранилища Azure для создания большого двоичного объекта в хранилище объекта](../../storage/blobs/storage-quickstart-blobs-storage-explorer.md).|
    |Job Upload Storage (Хранилище обновлений заданий)|Разверните раздел, чтобы отобразить дополнительные параметры.|
    |Тип хранения|Из раскрывающегося списка выберите **Use Azure Blob to upload** (Для передачи использовать большой двоичный объект Azure).|
    |Учетная запись хранения|Введите имя своей учетной записи хранения.|
    |Storage Key (Ключ хранилища)|Введите свой ключ к хранилищу данных.|
    |Контейнер хранилища|Если вы ввели значения в поля **Учетная запись хранения** и **Storage Key** (Ключ хранилища), из раскрывающегося списка выберите контейнер хранилища.|

    ![Диалоговое окно Spark Submission (Отправка в Spark)](./media/apache-spark-intellij-tool-plugin/hdi-submit-spark-app-02.png)

4. Щелкните **SparkJobRun**, чтобы отправить проект в выбранный кластер. На вкладке **Remote Spark Job in Cluster** (Удаленное задание Spark в кластере) внизу показан ход выполнения задания. Чтобы остановить работу приложения, нажмите красную кнопку.

    ![Окно отправки Apache Spark](./media/apache-spark-intellij-tool-plugin/hdi-spark-app-result.png)

## <a name="debug-apache-spark-applications-locally-or-remotely-on-an-hdinsight-cluster"></a>Локальная и удаленная отладка приложений Apache Spark в кластере HDInsight

Мы также рекомендуем еще один способ отправки приложения Spark в кластер. Он заключается в задании параметров **конфигураций запуска и отладки** в интегрированной среде разработки. См. раздел [отладка Apache Spark приложений локально или удаленно в кластере HDInsight с Azure Toolkit for IntelliJ по протоколу SSH](apache-spark-intellij-tool-debug-remotely-through-ssh.md).

## <a name="access-and-manage-hdinsight-spark-clusters-by-using-azure-toolkit-for-intellij"></a>Доступ к кластерам HDInsight Spark и управление ими с помощью набора средств Azure для IntelliJ

С помощью Azure Toolkit for IntelliJ можно выполнять различные операции.  Большинство операций запускается из **Azure Explorer**.  В строке меню выберите **Представление** > **Окно инструментов** > **Azure Explorer**.

### <a name="access-the-job-view"></a>Доступ к представлению задания

1. В Azure Explorer перейдите к   >  \<Your Cluster>  >  **заданиям** HDInsight.

    ![Узел представления задания IntelliJ Azure Explorer](./media/apache-spark-intellij-tool-plugin/intellij-job-view-node.png)

2. В области справа на вкладке **Spark Job View** (Просмотр заданий Spark) отображаются все приложения, запускаемые в кластере. Выберите имя приложения, дополнительные сведения о котором вы хотите просмотреть.

    ![Представление задания Spark, сведения о приложении](./media/apache-spark-intellij-tool-plugin/intellij-view-job-logs.png)

3. Чтобы отобразить базовые сведения о выполняющемся задании, наведите указатель мыши на граф задания. Чтобы просмотреть этапы графа и сведения, создаваемые каждым заданием, выберите узел на графе задания.

    ![Представление задания Spark, сведения об этапе задания](./media/apache-spark-intellij-tool-plugin/Job-graph-stage-info.png)

4. Выберите вкладку **Журнал**, чтобы просмотреть часто используемые журналы, включая *Driver Stderr*, *Driver Stdout* и *Directory Info*.

    ![Представление задания Spark, журнал сведений](./media/apache-spark-intellij-tool-plugin/intellij-job-log-info.png)

5. Вы можете просматривать пользовательский интерфейс журнала Spark и пользовательский интерфейс YARN (на уровне приложения). Выберите ссылку в верхней части окна.

### <a name="access-the-spark-history-server"></a>Доступ к серверу журнала Spark

1. В Azure Explorer разверните **HDInsight**, щелкните имя кластера Spark правой кнопкой мыши и выберите **Open Spark History UI** (Открыть пользовательский интерфейс журнала Spark).  
2. При появлении запроса введите учетные данные администратора для кластера, указанные при настройке кластера.

3. На панели мониторинга сервера журнала Spark вы сможете найти приложение, выполнение которого только что было завершено, по его имени. В приведенном выше коде имя приложения было указано с помощью `val conf = new SparkConf().setAppName("myApp")`. Ваше приложение Spark имеет имя **MyApp**.

### <a name="start-the-ambari-portal"></a>Запуск портала Ambari

1. В Azure Explorer разверните **HDInsight**, щелкните имя кластера Spark правой кнопкой мыши и выберите **Open Cluster Management Portal (Ambari)** (Открыть портал управления кластерами (Ambari)).  

2. При появлении запроса введите учетные данные администратора для кластера. Вы указали эти учетные данные во время установки кластера.

### <a name="manage-azure-subscriptions"></a>Управление подписками Azure

По умолчанию набор средств Azure для IntelliJ содержит список кластеров Spark из всех ваших подписок Azure. При необходимости можно указать подписки, к которым необходимо получить доступ.  

1. В Azure Explorer щелкните правой кнопкой мыши корневой узел **Azure**, а затем щелкните **Выбрать подписки**.  

2. В окне **Выбрать подписки** снимите флажки напротив подписок, доступ к которым вам не требуется, и выберите **Закрыть**.

## <a name="spark-console"></a>Консоль Spark

Вы можете запустить локальную консоль Spark (Scala) или консоль интерактивного сеанса Spark Livy (Scala).

### <a name="spark-local-consolescala"></a>Локальная консоль Spark (Scala)

Убедитесь в том, что есть необходимый файл WINUTILS.EXE.

1. В строке меню выберите **Run (Запуск)**  > **Edit Configurations...** (Изменить конфигурации).

2. В левой панели окна **Run/Debug Configurations** (Конфигурации запуска или отладки) перейдите к **Apache Spark on HDInsight** (Apache Spark в HDInsight) > **[Spark on HDInsight] myApp** (myApp [Spark в HDInsight]).

3. В главном окне выберите **`Locally Run`** вкладку.

4. Укажите следующие значения и нажмите кнопку **ОК**:

    |Свойство |Значение |
    |----|----|
    |Главный класс задания|значение по умолчанию — имя главного класса из выбранного файла. Класс можно изменить, нажав кнопку с многоточием ( **...** ) и выбрав другой класс.|
    |Переменные среды|Проверьте правильность значения HADOOP_HOME.|
    |Расположение файла WINUTILS.exe|Проверьте правильность пути.|

    ![Настройка конфигурации в локальной консоли](./media/apache-spark-intellij-tool-plugin/console-set-configuration.png)

5. В проекте перейдите в **myApp** > **src** > **main** > **scala** > **myApp**.  

6. В строке меню выберите **Tools (Сервис)**  > **Spark Console (Консоль Spark)**  > **Run Spark Local Console(Scala)** (Запустить локальную консоль Spark (Scala)).

7. После этого могут появиться два диалоговых окна с запросом на автоматическое исправление зависимостей. Если нужно исправить их, выберите **Auto Fix** (Автоматическое исправление).

    ![IntelliJ IDEA Spark автоматическое исправление dialog1](./media/apache-spark-intellij-tool-plugin/intellij-console-autofix1.png)

    ![IntelliJ IDEA Spark автоматическое исправление dialog2](./media/apache-spark-intellij-tool-plugin/intellij-console-autofix2.png)

8. Консоль должна выглядеть примерно так, как показано ниже. В окне консоли введите `sc.appName` и нажмите клавиши CTRL+ВВОД.  Отобразится результат. Чтобы завершить локальную консоль, нажмите кнопку с красной кнопкой.

    ![Результат локальной консоли IntelliJ IDEA](./media/apache-spark-intellij-tool-plugin/local-console-result.png)

### <a name="spark-livy-interactive-session-consolescala"></a>Консоль интерактивного сеанса Spark Livy (Scala)

1. В строке меню выберите **Run (Запуск)**  > **Edit Configurations...** (Изменить конфигурации).

2. В левой панели окна **Run/Debug Configurations** (Конфигурации запуска или отладки) перейдите к **Apache Spark on HDInsight** (Apache Spark в HDInsight) > **[Spark on HDInsight] myApp** (myApp [Spark в HDInsight]).

3. В главном окне выберите **`Remotely Run in Cluster`** вкладку.

4. Укажите следующие значения и нажмите кнопку **ОК**:

    |Свойство |Значение |
    |----|----|
    |Кластеры Spark (только Linux)|Выберите кластер HDInsight Spark, в котором вы хотите запустить приложение.|
    |Имя главного класса|значение по умолчанию — имя главного класса из выбранного файла. Класс можно изменить, нажав кнопку с многоточием ( **...** ) и выбрав другой класс.|

    ![Настройка конфигурации в интерактивной консоли](./media/apache-spark-intellij-tool-plugin/interactive-console-configuration.png)

5. В проекте перейдите в **myApp** > **src** > **main** > **scala** > **myApp**.  

6. В строке меню выберите **Tools (Сервис)**  > **Spark Console (Консоль Spark)**  > **Run Spark Livy Interactive Session Console(Scala)** (Запустить консоль интерактивного сеанса Spark Livy (Scala)).

7. Консоль должна выглядеть примерно так, как показано ниже. В окне консоли введите `sc.appName` и нажмите клавиши CTRL+ВВОД.  Отобразится результат. Чтобы завершить локальную консоль, нажмите кнопку с красной кнопкой.

    ![Результат интерактивной консоли IntelliJ IDEA](./media/apache-spark-intellij-tool-plugin/interactive-console-result.png)

### <a name="send-selection-to-spark-console"></a>Отправка выделенного фрагмента в консоль Spark

Удобно предвидеть результат сценария, отправив код в локальную консоль или консоль интерактивного сеанса Livy (Scala). Выделите код в файле Scala, а затем в контекстном меню выберите пункт **Send Selection To Spark Console** (Отправить выделенный фрагмент в консоль Spark). Выбранный код будет отправлен на консоль. Результат отобразится в консоли после кода. Консоль проверит наличие ошибок.  

   ![Отправка выделенного фрагмента в консоль Spark](./media/apache-spark-intellij-tool-plugin/send-selection-to-console.png)

## <a name="integrate-with-hdinsight-identity-broker-hib"></a>Интеграция с брокером удостоверений HDInsight (HIB)

### <a name="connect-to-your-hdinsight-esp-cluster-with-id-broker-hib"></a>Подключение к кластеру HDInsight ESP с помощью ID Broker (ХИБ)

Вы можете выполнить обычные действия для входа в подписку Azure, чтобы подключиться к кластеру HDInsight ESP с ИД брокером (ХИБ). После входа вы увидите список кластеров в Azure Explorer. Указания см. в разделе [Подключение к кластеру HDInsight](#connect-to-your-hdinsight-cluster).

### <a name="run-a-spark-scala-application-on-an-hdinsight-esp-cluster-with-id-broker-hib"></a>Запуск приложения Spark Scala в кластере HDInsight ESP с ИД брокером (ХИБ)

Вы можете выполнить обычные действия, чтобы отправить задание в кластер HDInsight ESP с ИДЕНТИФИКАТОРом брокера (ХИБ). Указания см. в разделе [Запуск приложения Spark Scala в кластере HDInsight Spark](#run-a-spark-scala-application-on-an-hdinsight-spark-cluster).

Мы отправим необходимые файлы в папку с именем с учетной записью для входа, и в файле конфигурации можно увидеть путь передачи.

   ![путь отправки в конфигурации](./media/apache-spark-intellij-tool-plugin/upload-path-in-the-configuration.png)

### <a name="spark-console-on-an-hdinsight-esp-cluster-with-id-broker-hib"></a>Консоль Spark в кластере HDInsight ESP с ИДЕНТИФИКАТОРом брокера (ХИБ)

Можно запустить локальную консоль Spark (Scala) или запустить консоль интерактивного сеанса Spark Livy (Scala) в кластере HDInsight ESP с ИДЕНТИФИКАТОРом брокера (ХИБ). Дополнительные инструкции см. в разделе [Консоль Spark](#spark-console).

   > [!NOTE]  
   > Для кластера HDInsight ESP с брокером удостоверений (HIB) [связь кластера](#link-a-cluster) и [удаленная отладка приложений Apache Spark](#debug-apache-spark-applications-locally-or-remotely-on-an-hdinsight-cluster) в настоящее время не поддерживается.

## <a name="reader-only-role"></a>Роль только для чтения

Когда пользователи отправляют задания в кластер с разрешением на роль только для чтения, требуются учетные данные Ambari.

### <a name="link-cluster-from-context-menu"></a>Связывание кластера из контекстного меню

1. Войдите с помощью учетной записи роли только для чтения.

2. В **Azure Explorer** разверните **HDInsight**, чтобы просмотреть кластеры HDInsight в своей подписке. Кластеры с пометкой **Role:Reader**, имеют только разрешение роли только для чтения.

    !["Роль Azure Explorer IntelliJ: читатель"](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer15.png)

3. Щелкните правой кнопкой мыши кластер с разрешением роли только для чтения. Выберите **Link this cluster** (Связать этот кластер) из контекстного меню, чтобы связать кластер. Введите имя пользователя и пароль Ambari.

    ![IntelliJ Azure Explorer связать этот кластер](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer11.png)

4. Если кластер связан успешно, HDInsight будет обновлен.
   Этап кластера станет связанным.
  
    ![IntelliJ Azure Explorer связанное диалоговое окно](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer8.png)

### <a name="link-cluster-by-expanding-jobs-node"></a>Связывание кластера путем развертывания узла "Задания"

1. Щелкните узел **Задания** и появится всплывающее окно **Cluster Job Access Denied** (Доступ к заданию кластера запрещен).

2. Щелкните **Link this cluster** (Связать этот кластер), чтобы связать кластер.

    ![диалоговое окно отказа в доступе для задания кластера](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer9.png)

### <a name="link-cluster-from-rundebug-configurations-window"></a>Связывание кластера из окна Run/Debug Configurations (Конфигурации запуска и отладки)

1. Создайте конфигурацию HDInsight. Выберите **Remotely Run in Cluster** (Удаленный запуск в кластере).

2. Выберите кластер, у которого есть разрешение роли только для чтения для **Spark clusters(Linux only)** (Кластеры Spark (только Linux)). Отображается предупреждающее сообщение. Можно щелкнуть **связать этот кластер** с кластером.

   ![Создание конфигурации запуска/отладки IntelliJ IDEA](./media/apache-spark-intellij-tool-plugin/create-configuration.png)

### <a name="view-storage-accounts"></a>Просмотр учетных записей хранения

* Для кластеров с разрешением роли только для чтения щелкните узел **Учетные записи хранения** и появится всплывающее окно **Storage Access Denied** (Доступ к хранилищу запрещен). Выберите **Открыть обозреватель службы хранилища Azure**, чтобы открыть Обозреватель службы хранилища.

   !["IntelliJ идея в доступе к хранилищу запрещен"](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer14.png)

   ![Доступ к хранилищу IntelliJ IDEA кнопка запрета доступа](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer10.png)

* Для связанных кластеров щелкните узел **Учетные записи хранения** и появится всплывающее окно **Storage Access Denied** (Доступ к хранилищу запрещен). Выберите **Открыть обозреватель службы хранилища Azure**, чтобы открыть Обозреватель службы хранилища.

   ![' IntelliJ идея Storage Access Denied2 '](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer13.png)

   ![IntelliJ IDEA Кнопка запрета доступа к хранилищу2](./media/apache-spark-intellij-tool-plugin/intellij-view-explorer12.png)

## <a name="convert-existing-intellij-idea-applications-to-use-azure-toolkit-for-intellij"></a>Преобразование имеющихся приложений IntelliJ IDEA для использования набора средств Azure для IntelliJ

Имеющиеся приложения Spark Scala, созданные в IntelliJ IDEA, можно преобразовать и сделать совместимыми с набором средств Azure для IntelliJ. Затем вы сможете использовать подключаемый модуль для отправки приложений в кластер HDInsight Spark.

1. Для существующего приложения Spark Scala, созданного с помощью IntelliJ идеи, откройте соответствующий `.iml` файл.

2. На корневом уровне — это элемент **module** , такой как следующий текст:

    ```xml
    <module org.jetbrains.idea.maven.project.MavenProjectsManager.isMavenModule="true" type="JAVA_MODULE" version="4">
    ```

   Измените элемент, добавив `UniqueKey="HDInsightTool"` таким образом, чтобы элемент **module** выглядел как следующий текст:

    ```xml
    <module org.jetbrains.idea.maven.project.MavenProjectsManager.isMavenModule="true" type="JAVA_MODULE" version="4" UniqueKey="HDInsightTool">
    ```

3. Сохраните изменения. Ваше приложение станет совместимым с набором средств Azure для IntelliJ. Это можно проверить, правой кнопкой мыши щелкнув имя проекта в структуре проекта. Во всплывающем меню должен появиться пункт **Submit Spark Application to HDInsight** (Отправить приложение Spark в HDInsight).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите созданный кластер, сделав следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В поле **Поиск** в верхней части страницы введите **HDInsight**.

1. Выберите **Кластеры HDInsight** в разделе **Службы**.

1. В появившемся списке кластеров HDInsight щелкните **...** рядом с кластером, созданным для этой статьи.

1. Выберите команду **Удалить**. Выберите **Да**.

![портал Azure удаляет кластер HDInsight](./media/apache-spark-intellij-tool-plugin/hdinsight-azure-portal-delete-cluster.png "Удаление кластера HDInsight")

## <a name="next-steps"></a>Следующие шаги

В этой статье вы узнали, как использовать подключаемый модуль Azure Toolkit for IntelliJ для разработки Apache Spark приложений, написанных на [Scala](https://www.scala-lang.org/). Затем передавали их в кластер HDInsight Spark непосредственно из интегрированной среды разработки (IDE) IntelliJ. Теперь переходите к следующей статье, в которой объясняется, как перенести зарегистрированные в Apache Spark данные в средство бизнес-аналитики, например в Power BI.

> [!div class="nextstepaction"]
> [Анализ данных Apache Spark с помощью Power BI](apache-spark-use-bi-tools.md)

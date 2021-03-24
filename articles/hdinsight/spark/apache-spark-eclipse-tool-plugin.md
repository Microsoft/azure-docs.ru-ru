---
title: 'Azure Toolkit for Eclipse: создание приложений Scala для HDInsight Spark'
description: Использование средств HDInsight из набора средств Azure для Eclipse для разработки приложений Spark на языке Scala и их отправки в кластер HDInsight Spark непосредственно из интегрированной среды разработки Eclipse.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/13/2019
ms.openlocfilehash: e4f19afc0edf0f43b64548ae1c93ed5ff8dfa8a1
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104866579"
---
# <a name="use-azure-toolkit-for-eclipse-to-create-apache-spark-applications-for-an-hdinsight-cluster"></a>Создание приложений Apache Spark для кластера HDInsight с помощью Azure Toolkit for Eclipse

Использование средств HDInsight из набора Azure Toolkit for [Eclipse](https://www.eclipse.org/) для разработки приложений [Apache Spark](https://spark.apache.org/) на языке [Scala](https://www.scala-lang.org/) и их отправки в кластер Azure HDInsight Spark непосредственно из интегрированной среды разработки Eclipse. Подключаемый модуль средств HDInsight можно использовать по-разному.

* Для разработки и отправки приложений Scala Spark в кластер HDInsight Spark.
* Для доступа к ресурсам кластера Azure HDInsight Spark.
* Для разработки и локального запуска приложений Scala Spark.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md).

* [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* [Интегрированная среда разработки Eclipse](https://www.eclipse.org/downloads/). В этой статье используется интегрированная среда разработки Eclipse для разработчиков Java.

## <a name="install-required-plug-ins"></a>Установить необходимые подключаемые модули

### <a name="install-azure-toolkit-for-eclipse"></a>Установка Azure Toolkit for Eclipse

Инструкции по установке см. в статье [Установка набора средств Azure для Eclipse](/azure/developer/java/toolkit-for-eclipse/installation).

### <a name="install-the-scala-plug-in"></a>Установка подключаемого модуля Scala

При открытии Eclipse средства HDInsight автоматически определяют, установлен ли подключаемый модуль Scala. Щелкните **ОК**, чтобы продолжить, и следуйте инструкциям по установке подключаемого модуля из Eclipse Marketplace. После завершения установки перезапустите интегрированную среду разработки.

:::image type="content" source="./media/apache-spark-eclipse-tool-plugin/auto-installation-scala1.png" alt-text="Автоматическая установка подключаемого модуля Scala" border="true":::

### <a name="confirm-plug-ins"></a>Подтвердить подключаемые модули

1. Перейдите в **раздел Help**  >  **Eclipse Marketplace.**..

1. Откройте вкладку **Установленные**.

1. Вы должны увидеть как минимум:
    * Azure Toolkit for Eclipse \<version> .
    * Интегрированная среда разработки Scala \<version> .

## <a name="sign-in-to-your-azure-subscription"></a>Войдите в свою подписку Azure.

1. Запустите интегрированную среду разработки Eclipse.

1. Перейдите к **окну**  >   **Показать**  >  **другие...**  >  **Войти..**.

1. В диалоговом окне **Показать представление** перейдите в **Azure**  >  **Azure Explorer** и нажмите кнопку **Открыть**.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer1.png" alt-text="Apache Spark Показать представление Eclipse" border="true":::

1. В **Azure Explorer** щелкните правой кнопкой мыши узел **Azure** и выберите **Вход**.

1. В диалоговом окне **Вход в Azure** выберите метод проверки подлинности, нажмите кнопку **войти** и завершите процесс входа.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer2.png" alt-text="Apache Sparkная Eclipse подписывание Azure" border="true":::

1. После входа в диалоговом окне " **подписки" будут** перечислены все подписки Azure, связанные с этими учетными данными. Нажмите кнопку " **выбрать** ", чтобы закрыть диалоговое окно.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/Select-Subscriptions.png" alt-text="Диалоговое окно выбора подписок" border="true":::

1. В **Azure Explorer** перейдите в **Azure**  >   **hdinsight** , чтобы просмотреть кластеры hdinsight Spark в вашей подписке.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer3.png" alt-text="Кластеры HDInsight Spark в Azure Explorer3" border="true":::

1. Далее можно развернуть узел имени кластера, чтобы увидеть ресурсы (например, учетные записи хранения), связанные с ним.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer4.png" alt-text="Развертывание имени кластера для просмотра ресурсов" border="true":::

## <a name="link-a-cluster"></a>Связывание кластера

Можно связать обычный кластер с помощью управляемого имени пользователя Ambari. Аналогичным образом, присоединенный к домену кластер HDInsight можно связать с помощью домена и имени пользователя, например `user1@contoso.com`.

1. В **Azure Explorer** щелкните правой кнопкой мыши **HDInsight** и выберите **связать кластер**.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/link-a-cluster-context-menu.png" alt-text="Меню кластера ссылок Azure Explorer" border="true":::

1. Введите **имя кластера**, **имя пользователя** и **пароль**, а затем нажмите кнопку **ОК**. При необходимости введите учетную запись хранения, ключ к хранилищу данных, а затем выберите в представлении в виде дерева контейнер хранилища для обозревателя хранилищ.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/link-cluster-dialog1.png" alt-text="Диалоговое окно &quot;связать новый кластер HDInsight&quot;" border="true":::

   > [!NOTE]  
   > Если кластер зарегистрирован в подписке Azure и связан, используется ключ к хранилищу данных, имя пользователя и пароль для связывания.
   > :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/storage-explorer-in-Eclipse.png" alt-text="Учетные записи хранения Azure Explorer" border="true":::
   >
   > Для пользователя с клавиатурой, когда текущий фокус находится на **ключе хранилища**, необходимо использовать **сочетание клавиш CTRL + TAB** , чтобы сосредоточиться на следующем поле в диалоговом окне.

1. Связанный кластер можно увидеть в **HDInsight**. Теперь в этот связанный кластер можно отправить приложение.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/hdinsight-linked-cluster.png" alt-text="Связанный кластер HDi в Azure Explorer" border="true":::

1. В **обозревателе Azure** также можно удалить связь кластера.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/hdi-unlinked-cluster.png" alt-text="Несвязанный кластер Azure Explorer" border="true":::

## <a name="set-up-a-spark-scala-project-for-an-hdinsight-spark-cluster"></a>Настройка проекта Spark Scala для кластера HDInsight Spark

1. В рабочей области IDE Eclipse выберите **файл**  >  **создать**  >  **проект...**.

1. В мастере **создания проектов** выберите **hdinsight Project**  >  **Spark в hdinsight (Scala)**. Выберите **Далее**.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/create-hdi-scala-app-2.png" alt-text="Выбор проекта Spark в HDInsight (Scala)" border="true":::

1. В диалоговом окне **New HDInsight Scala Project** (Новый проект Scala HDInsight) введите следующие значения, после чего нажмите кнопку **Next** (Далее).
   * Введите имя проекта.
   * Убедитесь, что в поле **JRE** параметр **Use an execution environment JRE** (Использовать среду выполнения JRE) имеет значение **JavaSE-1.7** или более позднюю версию.
   * В области **библиотеки Spark** вы можете выбрать вариант **Use Maven to configure Spark SDK** (Использовать Maven для настройки пакета SDK для Spark).  Наше средство интегрирует правильную версию пакета SDK для Spark и пакета SDK для Scala. Можно также выбрать параметр **Добавить пакет SDK для Spark вручную** , скачать и добавить пакет SDK для Spark вручную.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/create-hdi-scala-app-3.png" alt-text="Диалоговое окно нового проекта Scala HDInsight" border="true":::

1. В следующем диалоговом окне просмотрите сведения и нажмите кнопку **Готово**.

## <a name="create-a-scala-application-for-an-hdinsight-spark-cluster"></a>Создание приложения Scala для кластера HDInsight Spark

1. В **обозревателе пакетов** разверните проект, созданный ранее. Щелкните правой кнопкой мыши элемент **src**, выберите **создать**  >  **другой...**.

1. В диалоговом окне **Выбор мастера** выберите **Scala мастера**  >  **Scala объект**. Выберите **Далее**.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/create-scala-project1.png" alt-text="Выбор мастера Создание объекта Scala" border="true":::

1. В диалоговом окне **Create New File** (Создание файла) введите имя объекта и нажмите кнопку **Finish** (Готово). Откроется текстовый редактор.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/create-scala-project2.png" alt-text="Мастер создания файлов — создать новый файл" border="true":::

1. В текстовом редакторе Замените текущее содержимое следующим кодом:

    ```scala
    import org.apache.spark.SparkConf
    import org.apache.spark.SparkContext

    object MyClusterApp{
        def main (arg: Array[String]): Unit = {
        val conf = new SparkConf().setAppName("MyClusterApp")
        val sc = new SparkContext(conf)

        val rdd = sc.textFile("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

        //find the rows that have only one digit in the seventh column in the CSV
        val rdd1 =  rdd.filter(s => s.split(",")(6).length() == 1)

        rdd1.saveAsTextFile("wasbs:///HVACOut")
        }
    }
    ```

1. Запустите приложение в кластере HDInsight Spark.

   а. В обозревателе пакетов щелкните имя проекта правой кнопкой мыши и выберите пункт **Submit Spark Application to HDInsight** (Отправить приложение Spark в HDInsight).

   b. В диалоговом окне **отправки Spark** укажите следующие значения, а затем выберите **Отправить**:

   * В поле **Cluster Name**(Имя кластера) выберите кластер HDInsight Spark, в котором вы хотите запустить приложение.
   * Выберите артефакт из проекта Eclipse или с жесткого диска. Значение по умолчанию зависит от элемента, который вы щелкнете правой кнопкой мыши в обозревателе пакетов.
   * В раскрывающемся списке **Main class name** (Имя класса main) в мастере отправки отображаются имена всех объектов из вашего проекта. Выберите или введите имя любого из объектов, который требуется запустить. Если вы выбрали артефакт с жесткого диска, необходимо ввести имя класса main вручную. 
   * Так как код приложения в этом примере не требует аргументов командной строки, ссылок на JAR или файлов, оставшиеся текстовые поля можно оставить пустыми.

     :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/create-scala-project3.png" alt-text="Диалоговое окно &quot;Отправка Apache Spark&quot;" border="true":::

1. На вкладке **Spark Submission** (Отправка в Spark) начнет отображаться ход выполнения. Приложение можно остановить, нажав красную кнопку в окне **Spark Submission** (Отправка в Spark). Можно также просмотреть журналы для данного запуска приложения, щелкнув значок глобуса (обозначен на рисунке синей рамкой).

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/create-scala-project4.png" alt-text="Окно отправки Apache Spark" border="true":::

## <a name="access-and-manage-hdinsight-spark-clusters-by-using-hdinsight-tools-in-azure-toolkit-for-eclipse"></a>Доступ к кластерам HDInsight Spark и управление ими с помощью средств HDInsight в наборе средств Azure для Eclipse

С помощью средств HDInsight можно выполнять различные операции, включая доступ к выходным данным заданий.

### <a name="access-the-job-view"></a>Доступ к представлению задания

1. В **Azure Explorer** разверните **HDInsight**, затем имя кластера Spark, а затем выберите **задания**.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-job-view-node.png" alt-text="Узел представления заданий Eclipse в Azure Explorer" border="true":::

1. Перейдите на вкладку **Задания**. Если версия Java ниже, чем **1.8**, средства HDInsight автоматически отправят напоминание об установке подключаемого модуля **E(fx)clipse**. Щелкните **ОК**, чтобы продолжить, и следуйте инструкциям мастера для установки подключаемого модуля из Eclipse Marketplace. По завершении перезапустите Eclipse.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/auto-install-efxclipse.png" alt-text="Установка отсутствующего Clip подключаемого модуля E (FX)" border="true":::

1. Откройте представление задания из узла **Задания**. В области справа на вкладке **Spark Job View** (Просмотр заданий Spark) отображаются все приложения, запускаемые в кластере. Выберите имя приложения, дополнительные сведения о котором вы хотите просмотреть.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-job-logs.png" alt-text="Сведения о журналах заданий представления Apache Eclipse" border="true":::

   Затем можно использовать любое из следующих действий:

   * Наведите указатель мыши на граф задания. На нем отображаются основные сведения о выполняемом задании. Щелкните граф задания, и вы увидите его этапы и сведения, создаваемые каждым заданием.

     :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/Job-graph-stage-info.png" alt-text="Сведения о стадии графа задания Apache Spark" border="true":::

   * Выберите вкладку **Журнал**, чтобы просмотреть часто используемые журналы, включая **Driver Stderr**, **Driver Stdout** и **Directory Info**.

     :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-job-log-info.png" alt-text="Сведения о журнале заданий Apache Spark Eclipse" border="true":::

   * Откройте пользовательский интерфейс журнала Spark и пользовательский интерфейс Apache Hadoop YARN (на уровне приложения), щелкнув ссылки в верхней части окна.

### <a name="access-the-storage-container-for-the-cluster"></a>Доступ к контейнеру хранилища для кластера

1. В Azure Explorer разверните корневой узел **HDInsight**, чтобы увидеть список доступных кластеров HDInsight Spark.

1. Разверните имя кластера, чтобы увидеть учетную запись хранилища и контейнер хранилища по умолчанию для кластера.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer5.png" alt-text="Учетная запись хранения и контейнер по умолчанию" border="true":::

1. Выберите имя связанного с кластером контейнера хранилища. В области справа дважды щелкните папку **HVACOut**. Откройте один из файлов **part-** для просмотра выходных данных приложения.

### <a name="access-the-spark-history-server"></a>Доступ к серверу журнала Spark

1. В Azure Explorer щелкните имя кластера Spark правой кнопкой мыши и выберите пункт **Open Spark History UI** (Открыть пользовательский интерфейс журнала Spark). При появлении запроса введите учетные данные администратора для кластера. Вы указали их при подготовке кластера.

1. На панели мониторинга сервера журнала Spark вы сможете найти приложение, выполнение которого только что было завершено, по его имени. В приведенном выше коде имя приложения было указано с помощью `val conf = new SparkConf().setAppName("MyClusterApp")`. Следовательно, приложение Spark называлось **MyClusterApp**.

### <a name="start-the-apache-ambari-portal"></a>Запуск портала Apache Ambari

1. В Azure Explorer щелкните имя кластера Spark правой кнопкой мыши и выберите пункт **Open Cluster Management Portal (Ambari)** (Открыть портал управления кластерами (Ambari)).

1. При появлении запроса введите учетные данные администратора для кластера. Вы указали их при подготовке кластера.

### <a name="manage-azure-subscriptions"></a>Управление подписками Azure

По умолчанию средство HDInsight в наборе средств Azure для Eclipse содержит список кластеров Spark из всех ваших подписок Azure. При необходимости можно указать подписки, кластеры из которых вас интересуют.

1. В Azure Explorer щелкните правой кнопкой мыши корневой узел **Azure**, а затем выберите **Управление подписками**.

1. В диалоговом окне снимите флажки напротив подписок, доступ к которым вам не требуется, и нажмите кнопку **Закрыть**. Если вы хотите выйти из своей подписки Azure, выберите **Выйти**.

## <a name="run-a-spark-scala-application-locally"></a>Запуск приложения Spark Scala на локальном компьютере

Средства HDInsight в наборе средств Azure для Eclipse позволяют запускать приложения Spark Scala локально на рабочей станции. Как правило, такие приложения не требуют доступа к ресурсам кластера, таким как контейнер хранилища, и могут запускаться и тестироваться локально.

### <a name="prerequisite"></a>Предварительное требование

При запуске локального приложения Spark Scala на компьютере Windows может возникнуть исключение, как описано в [Spark-2356](https://issues.apache.org/jira/browse/SPARK-2356). Это исключение возникает, так как в Windows отсутствует файл **WinUtils.exe**.

Чтобы устранить эту ошибку, необходимо [Winutils.exe](https://github.com/steveloughran/winutils) в расположение, например **C:\WinUtils\bin**, а затем добавить переменную среды **HADOOP_HOME** и присвоить переменной значение **C\WinUtils**.

### <a name="run-a-local-spark-scala-application"></a>Запуск локального приложения Spark Scala

1. Запустите Eclipse и создайте новый проект. В диалоговом окне **New Project** (Новый проект) установите параметры, как на снимке экрана ниже, а затем нажмите кнопку **Next** (Далее).

1. В мастере **создания проектов** выберите **hdinsight проект**  >  **Spark в hdinsight локальный запуск пример (Scala)**. Выберите **Далее**.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/hdi-spark-app-local-run.png" alt-text="Создать проект выбор диалогового окна мастера" border="true":::

1. Чтобы предоставить сведения о проекте, выполните шаги 3–6, как описано в разделе [Настройка проекта Spark Scala для кластера HDInsight Spark](#set-up-a-spark-scala-project-for-an-hdinsight-spark-cluster).

1. Шаблон добавляет пример кода (**LogQuery**) в папку **src**, который можно запустить локально на компьютере.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/local-scala-application.png" alt-text="Расположение локального приложения Scala LogQuery" border="true":::

1. Щелкните правой кнопкой мыши **LogQuery. Scala** и выберите **Запуск от имени**  >  **1 приложения Scala**. На вкладке **Console** (Консоль) отобразятся выходные данные следующего вида.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/hdi-spark-app-local-run-result.png" alt-text="Результат локального запуска приложения Spark" border="true":::

## <a name="reader-only-role"></a>Роль только для чтения

Когда пользователи отправляют задания в кластер с разрешением на роль только для чтения, требуются учетные данные Ambari.

### <a name="link-cluster-from-context-menu"></a>Связывание кластера из контекстного меню

1. Войдите с помощью учетной записи роли только для чтения.

2. В **Azure Explorer** разверните **HDInsight**, чтобы просмотреть кластеры HDInsight в своей подписке. Кластеры с пометкой **Role:Reader**, имеют только разрешение роли только для чтения.

    :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer6.png" alt-text="Кластеры HDInsight Spark в Azure Explorer роль читатель" border="true":::

3. Щелкните правой кнопкой мыши кластер с разрешением роли только для чтения. Выберите **Link this cluster** (Связать этот кластер) из контекстного меню, чтобы связать кластер. Введите имя пользователя и пароль Ambari.

    :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer7.png" alt-text="Ссылки на кластеры HDInsight Spark в Azure Explorer" border="true":::

4. Если кластер связан успешно, HDInsight будет обновлен.
   Этап кластера станет связанным.
  
    :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer8.png" alt-text="Связывание кластеров HDInsight Spark в Azure Explorer" border="true":::

### <a name="link-cluster-by-expanding-jobs-node"></a>Связывание кластера путем развертывания узла "Задания"

1. Щелкните узел **Задания** и появится всплывающее окно **Cluster Job Access Denied** (Доступ к заданию кластера запрещен).

2. Щелкните **Link this cluster** (Связать этот кластер), чтобы связать кластер.

    :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer9.png" alt-text="Кластеры HDInsight Spark в Azure Explorer9" border="true":::

### <a name="link-cluster-from-spark-submission-window"></a>Связывание кластера с окном отправки Spark

1. Создайте проект HDInsight.

2. Щелкните пакет правой кнопкой мыши. Затем выберите **отправить приложение Spark в HDInsight**.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer11.png" alt-text="Отправка кластеров HDInsight Spark в Azure Explorer" border="true":::

3. Выберите кластер с разрешениями роли только для чтения для **имени кластера**. Отображается предупреждающее сообщение. Можно щелкнуть **связать этот кластер** с кластером.

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer15.png" alt-text="Создание кластеров HDInsight Spark в Azure Explorer ссылка" border="true":::

### <a name="view-storage-accounts"></a>Просмотр учетных записей хранения

* Для кластеров с разрешением роли только для чтения щелкните узел **Учетные записи хранения** и появится всплывающее окно **Storage Access Denied** (Доступ к хранилищу запрещен).

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer13.png" alt-text="Кластеры HDInsight Spark в хранилище Azure Explorer" border="true":::

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer12.png" alt-text="Кластеры HDInsight Spark в Azure Explorer запрещены" border="true":::

* Для связанных кластеров щелкните узел **Учетные записи хранения** и появится всплывающее окно **Storage Access Denied** (Доступ к хранилищу запрещен).

   :::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-view-explorer14.png" alt-text="Кластеры HDInsight Spark в Azure Explorer denied2" border="true":::

## <a name="known-problems"></a>Известные проблемы

При использовании **связывания кластера** рекомендуется предоставить учетные данные хранилища.

:::image type="content" source="./media/apache-spark-eclipse-tool-plugin/link-cluster-with-storage-credential-eclipse.png" alt-text="Связывание кластера с Eclipse учетных данных хранилища" border="true":::

Существует два режима отправки заданий. Если учетные данные хранилища указаны, для отправки задания будет использоваться пакетный режим. В противном случае будет использоваться интерактивный режим. Если кластер занят, может появиться приведенная ниже ошибка.

:::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-interactive-cluster-busy-upload.png " alt-text="Eclipse получает сообщение об ошибке, если кластер занят" border="true":::

:::image type="content" source="./media/apache-spark-eclipse-tool-plugin/eclipse-interactive-cluster-busy-submit.png " alt-text="Ошибка получения Eclipse при занятости кластера Yarn" border="true":::

## <a name="see-also"></a>См. также раздел

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)

### <a name="scenarios"></a>Сценарии

* [Использование Apache Spark со средствами бизнес-аналитики. Выполнение интерактивного анализа данных с использованием Spark в HDInsight с помощью средств бизнес-аналитики](apache-spark-use-bi-tools.md)
* [Apache Spark и Машинное обучение. Анализ температуры в здании на основе данных системы кондиционирования с помощью Spark в HDInsight](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark и Машинное обучение. Прогнозирование результатов проверки пищевых продуктов с помощью Spark в HDInsight](apache-spark-machine-learning-mllib-ipython.md)
* [Анализ журналов веб-сайтов с помощью Apache Spark в HDInsight](apache-spark-custom-library-website-log-analysis.md)

### <a name="creating-and-running-applications"></a>Создание и запуск приложений

* [Создание автономного приложения с использованием Scala](apache-spark-create-standalone-application.md)
* [Удаленный запуск заданий с помощью Apache Livy в кластере Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Инструменты и расширения

* [Создание приложений Spark для кластера HDInsight с помощью набора средств Azure для IntelliJ](apache-spark-intellij-tool-plugin.md)
* [Удаленная отладка приложений Spark в HDInsight через VPN с помощью Azure Toolkit for IntelliJ](./apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Удаленная или локальная отладка приложений Spark в кластере HDInsight с помощью Azure Toolkit for IntelliJ через SSH](./apache-spark-intellij-tool-debug-remotely-through-ssh.md)
* [Использование записных книжек Zeppelin с кластером Apache Spark в Azure HDInsight](apache-spark-zeppelin-notebook.md)
* [Ядра, доступные для Jupyter Notebook в кластере Apache Spark для HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Использование внешних пакетов с записными книжками Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
* [Установка записной книжки Jupyter на компьютере и ее подключение к кластеру Apache Spark в Azure HDInsight (предварительная версия)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="managing-resources"></a>Управление ресурсами

* [Управление ресурсами кластера Apache Spark в Azure HDInsight](apache-spark-resource-manager.md)
* [Отслеживание и отладка заданий в кластере Apache Spark в HDInsight на платформе Linux](apache-spark-job-debugging.md)
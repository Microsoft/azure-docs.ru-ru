---
title: Apache Hadoop & Data Lake средств Visual Studio — Azure HDInsight
description: Узнайте, как установить и использовать средства Data Lake для Visual Studio. Используйте средство для подключения к кластерам Apache Hadoop в Azure HDInsight, а затем выполните запросы Hive.
keywords: средства hadoop, запрос hive, visual studio, visual studio hadoop
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017,seodec18
ms.topic: how-to
ms.date: 04/14/2020
ms.openlocfilehash: 8d8e9784ea21bf5f2b6902e3d93c5c09c1ec5670
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944559"
---
# <a name="use-data-lake-tools-for-visual-studio-to-connect-to-azure-hdinsight-and-run-apache-hive-queries"></a>Подключение к Azure HDInsight и выполнение запросов Apache Hive с помощью средств Data Lake для Visual Studio

Узнайте, как использовать Microsoft Azure Data Lake и средства Stream Analytics для Visual Studio (Data Lake Tools). Используйте средство для подключения к [кластерам Apache Hadoop в Azure HDInsight](apache-hadoop-introduction.md) и отправки запросов Hive.  

Дополнительные сведения об использовании HDInsight см. в статье [Приступая к работе с hdinsight](apache-hadoop-linux-tutorial-get-started.md).  

Дополнительные сведения о подключении к Apache Storm см. в статье [Разработка топологий C# для Apache Storm с помощью средств Data Lake](../storm/apache-storm-develop-csharp-visual-studio-topology.md).

Средства Data Lake для Visual Studio можно использовать для доступа к Azure Data Lake Analytics и HDInsight. Дополнительные сведения см. в статье [Разработка скриптов U-SQL с помощью средств Data Lake для Visual Studio](../../data-lake-analytics/data-lake-analytics-data-lake-tools-get-started.md).

## <a name="prerequisites"></a>Предварительные условия

Чтобы завершить работу с этой статьей и использовать средства Data Lake для Visual Studio, вам потребуются следующие компоненты:

* Кластер Azure HDInsight. Сведения о создании этого кластера см. в статье [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](apache-hadoop-linux-tutorial-get-started.md). Чтобы выполнять интерактивные запросы Apache Hive, вам потребуется кластер с [интерактивными запросами HDInsight](../interactive-query/apache-interactive-query-get-started.md).  

* [Visual Studio](https://visualstudio.microsoft.com/downloads/). [Выпуск Visual Studio Community](https://visualstudio.microsoft.com/vs/community/) предоставляется бесплатно. Приведенные здесь инструкции предназначены для [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/).

## <a name="install-data-lake-tools-for-visual-studio"></a>Установка инструментов Data Lake для Visual Studio  

Выполните соответствующие инструкции, чтобы установить средства Data Lake для вашей версии Visual Studio:

* Для Visual Studio 2017 или Visual Studio 2019:

    Во время установки Visual Studio убедитесь, что вы включили рабочую нагрузку **разработки Azure** , рабочую нагрузку " **Хранение и обработка данных** ".  

    Для существующих установок Visual Studio перейдите в строку меню интегрированной среды разработки и выберите **инструменты**  >  **получить инструменты и компоненты** , чтобы открыть Visual Studio Installer. На вкладке **рабочие нагрузки** выберите по крайней мере рабочую нагрузку **разработки Azure** (в разделе **веб-& Cloud**). Или выберите рабочую нагрузку **Хранение и обработка данных** (в разделе **другие наборы инструментов**).

  ![Выбор рабочей нагрузки, Visual Studio Installer](./media/apache-hadoop-visual-studio-tools-get-started/vs-installation.png)

* Для Visual Studio 2015:

    [Загрузка средств Data Lake](https://www.microsoft.com/download/details.aspx?id=49504). Выберите версию средств Data Lake, которая соответствует вашей версии Visual Studio.

## <a name="update-data-lake-tools-for-visual-studio"></a>Обновление средств Data Lake для Visual Studio  

Далее Убедитесь, что вы обновляете Data Lake средства до последней версии.

1. Запустите Visual Studio.

2. В окне **Пуск** выберите **Продолжить без кода**.

3. В строке меню интегрированной среды разработки Visual Studio выберите **расширения**  >  **Управление расширениями**.

4. В диалоговом окне **Управление расширениями** разверните узел **обновления** .

5. Если список доступных обновлений включает **Azure Data Lake и средства анализа Streaming**, выберите его. Затем выберите соответствующую кнопку **обновления** . После того как откроется и исчезнет диалоговое окно **Загрузка и установка** , Visual Studio добавит в расписание обновления расширение **средств анализа Azure Data Lake и Stream** Analytics.

6. Закройте все окна Visual Studio. Откроется диалоговое окно **установщик VSIX** .

7. Выберите **Лицензия** , чтобы прочитать условия лицензии, а затем нажмите кнопку **Закрыть** , чтобы вернуться в диалоговое окно **установщик VSIX** .

8. Нажмите кнопку **Изменить**. Начинается установка обновления расширения. Через некоторое время диалоговое окно изменится, чтобы убедиться, что изменения выполнены. Нажмите кнопку **Закрыть**, а затем перезапустите Visual Studio, чтобы завершить установку.

> [!NOTE]  
> Чтобы подключиться к кластерам Interactive Query и выполнять интерактивные запросы Hive, можно использовать средства Data Lake только версии 2.3.0.0 или более поздней.

## <a name="connect-to-azure-subscriptions"></a>Подключение к подпискам Azure

Вы можете использовать средства Data Lake для Visual Studio для подключения к кластерам HDInsight, выполнения некоторых основных операций управления и выполнения запросов Hive.

> [!NOTE]  
> Сведения о подключении к универсальному кластеру Hadoop см. [в статье Создание и отправка запросов Hive с помощью Visual Studio](/archive/blogs/xiaoyong/how-to-write-and-submit-hive-queries-using-visual-studio).

### <a name="connect-to-an-azure-subscription"></a>Подключение к подписке Azure

Чтобы подключиться к подписке Azure, сделайте следующее:

1. Запустите Visual Studio.

2. В окне **Пуск** выберите **Продолжить без кода**.

3. В строке меню интегрированной среды разработки выберите **Просмотреть**  >  **Обозреватель сервера**.

4. В **Обозреватель сервера** щелкните правой кнопкой мыши **Azure**, выберите **подключиться к Microsoft Azure подписке** и завершите процесс проверки подлинности. В **Обозреватель сервера** разверните **Azure**  >  **HDInsight** , чтобы просмотреть список существующих кластеров HDInsight.

5. Если у вас нет кластеров, создайте его с помощью портал Azure, Azure PowerShell или пакета SDK для HDInsight. Дополнительные сведения см. [в разделе Настройка кластеров в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

   ![Список кластеров HDInsight, обозреватель сервера, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-server-explorer.png)

6. Разверните кластер HDInsight. Кластер содержит узлы для **баз данных Hive**. Кроме того, учетная запись хранения по умолчанию, все дополнительные связанные учетные записи хранения и **журнал службы Hadoop**. Развертывание объектов можно продолжить.

После подключения к подписке Azure можно будет выполнить следующие задачи.

### <a name="connect-to-azure-from-visual-studio"></a>Подключение к Azure из Visual Studio

Чтобы подключиться к порталу Azure из Visual Studio, сделайте следующее:

1. В **Обозреватель сервера** разверните узел **Azure**  >  **HDInsight** и выберите кластер.

2. Щелкните правой кнопкой мыши кластер HDInsight и выберите пункт **Управление кластером в портал Azure**.

### <a name="offer-questions-and-feedback-from-visual-studio"></a>Предложить вопросы и отзывы из Visual Studio

Чтобы задать вопросы и отправить отзыв из Visual Studio, сделайте следующее:

1. В обозреватель сервера выберите **Azure**  >  **HDInsight**.

2. Щелкните правой кнопкой мыши **HDInsight** и выберите **форум MSDN** , чтобы задать вопросы, или **отправьте отзыв** о предоставлении отзыва.

## <a name="link-to-or-edit-a-cluster"></a>Связывание или изменение кластера

> [!NOTE]
> В настоящее время единственный тип кластера HDInsight, с которым можно связаться, — это тип Hive.

Чтобы связать кластер HDInsight, выполните следующие действия.

1. Щелкните правой кнопкой мыши **HDInsight** и выберите **связать кластер hdinsight** , чтобы открыть диалоговое окно **связывание кластера hdinsight** .

2. Введите **URL-адрес подключения** в форме `https://CLUSTERNAME.azurehdinsight.net` . **Имя кластера** автоматически заполняется частью имени кластера URL-адреса при переходе к другому полю. Затем введите **имя пользователя** и **пароль** и нажмите кнопку **Далее**.

    ![Связывание кластера, HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-link-cluster-dialog.png)

3. Нажмите кнопку **Готово**. Если связывание с кластером прошло успешно, кластер будет отображаться в узле **HDInsight** .

Чтобы обновить связанный кластер, щелкните правой кнопкой мыши кластер и выберите команду **изменить**. Затем можно обновить сведения о кластере.

![Изменение связанного кластера, HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-link-cluster-update.png)

## <a name="explore-linked-resources"></a>Анализ связанных ресурсов

В обозревателе сервера отобразятся учетная запись хранения по умолчанию и все связанные учетные записи хранения. Если развернуть учетную запись хранения по умолчанию, отобразятся контейнеры в учетной записи хранения. Учетная запись хранения по умолчанию и контейнер по умолчанию отмечены.

![Средства Data Lake для связанных ресурсов Visual Studio в обозреватель сервера](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-linked-resources.png)

Щелкните контейнер правой кнопкой мыши и выберите пункт **Просмотреть контейнер** , чтобы просмотреть содержимое контейнера. После открытия контейнера можно использовать кнопки на панели инструментов для **обновления** списка содержимого, **отправки большого двоичного объекта**, **удаления выбранных больших** двоичных объектов, **открытия большого двоичного объекта** и скачивания выбранных больших двоичных объектов (**Сохранить как**).

![Операции со списком контейнеров и BLOB-объектами, кластер HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-blob-operations.png)

## <a name="run-interactive-apache-hive-queries"></a>Выполнение интерактивных запросов Apache Hive

[Apache Hive](https://hive.apache.org) — это инфраструктура хранилища данных, встроенная в Hadoop. Hive используется, чтобы обобщать, запрашивать и анализировать данные. Средства Data Lake для Visual Studio поддерживают выполнение запросов Hive из Visual Studio. Дополнительные сведения о Hive см. [в статье что такое Apache Hive и HiveQL в Azure HDInsight?](hdinsight-use-hive.md).

[Интерактивный запрос в Azure HDInsight](../interactive-query/apache-interactive-query-get-started.md) использует [Hive на LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP) в Apache Hive 2,1. Интерактивный запрос обеспечивает взаимодействие с сложными запросами, основанными на хранилище данных, к большим хранящимся наборам DataSet. Выполнение запросов Hive в интерактивном запросе выполняется гораздо быстрее, чем традиционные задания пакетной службы Hive. 

> [!NOTE]  
> Выполнение интерактивных запросов Hive поддерживается только при подключении к кластеру [интерактивных запросов HDInsight](../interactive-query/apache-interactive-query-get-started.md).

Можно также использовать средства Data Lake для Visual Studio, чтобы увидеть, что находится внутри задания Hive. Средства Data Lake для Visual Studio собирают и обрабатывают журналы YARN определенных заданий Hive.

В **Обозреватель сервера** выберите **Azure**  >  **HDInsight** и выберите кластер.  Этот узел является отправной точкой в **Обозреватель сервера** для следующих разделов.

### <a name="view-hivesampletable"></a>Просмотр таблицы hivesampletable

Для всех кластеров HDInsight существует пример таблицы Hive по умолчанию с именем `hivesampletable` .  

В кластере выберите **куст базы данных**  >  **hivesampletable по умолчанию**  >  .

* Чтобы просмотреть `hivesampletable` схему, сделайте следующее:

    Разверните **hivesampletable**. Отображаются имена и типы данных `hivesampletable` столбцов.

* Для просмотра `hivesampletable` данных выполните следующие действия.

    Щелкните правой кнопкой мыши **hivesampletable** и выберите **просмотреть первые 100 строк**. Список результатов 100 появится в окне **Таблица Hive: hivesampletable** . Это действие эквивалентно выполнению следующего запроса Hive с помощью драйвера Hive ODBC:

    `SELECT * FROM hivesampletable LIMIT 100`

    Количество строк можно настроить путем изменения **числа строк**. в раскрывающемся списке можно выбрать строки 50, 100, 200 или 1000.

### <a name="create-hive-tables"></a>Создание таблиц Hive

Таблицу Hive можно создать с помощью графического интерфейса пользователя (GUI) или запросов Hive. Сведения об использовании запросов Hive см. в разделе [Создание и выполнение запросов Hive](#create-and-run-hive-queries).

1. В кластере выберите **база данных Hive**  >  **по умолчанию**.

2. Щелкните правой кнопкой мыши **по умолчанию** и выберите команду **создать таблицу**.

3. Настройка таблицы.

4. Нажмите кнопку **создать таблицу** , чтобы отправить задание, которое создает новую таблицу Hive.

    ![Создание окна таблицы, Hive, кластера HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-create-hive-table.png)

### <a name="create-and-run-hive-queries"></a>Создание и выполнение запросов Hive

Есть два способа создания и выполнения запросов Hive.

* Создание ad-hoc-запросов
* Создание приложения Hive

#### <a name="create-an-ad-hoc-query"></a>Создание специального запроса

Чтобы создать и запустить специальный запрос:

1. Щелкните правой кнопкой мыши кластер, в котором нужно выполнить запрос, и выберите команду **написать запрос Hive**.  

2. Введите запрос Hive.

    Редактор Hive поддерживает технологию IntelliSense. Средства Data Lake для Visual Studio поддерживают загрузку удаленных метаданных при редактировании скрипта Hive. Например, если ввести `SELECT * FROM` , IntelliSense выводит список всех предлагаемых имен таблиц. Если указано имя таблицы, IntelliSense выведет список имен столбцов. Эти инструменты поддерживают почти все инструкции, подзапросы и встроенные определяемые пользователем функции Hive DML.

    ![IntelliSense, пример 1, нерегламентированный запрос Hive, кластер HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-intellisense-table-names.png)

    ![IntelliSense, пример 2, нерегламентированный запрос Hive, кластер HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-intellisense-column-names.png)

    > [!NOTE]  
    > IntelliSense предлагает только метаданные кластеров, выбранных на панели инструментов HDInsight.

    Ниже приведен пример запроса, который можно использовать.

    ```sql
    SELECT devicemodel, COUNT(devicemodel) AS deviceCount
    FROM hivesampletable
    GROUP BY devicemodel
    ORDER BY devicemodel
    ```

3. Выберите режим выполнения:

    * **Интерактивный**  

        В первом раскрывающемся списке выберите **Interactive (интерактивный**) и щелкните **Execute (выполнить**).

        ![Интерактивный режим, нерегламентированный запрос Hive, кластер HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-query-execute.png)  

    * **Пакетная служба**  

        В первом раскрывающемся списке выберите **пакет** и нажмите кнопку **Отправить**. Или щелкните значок раскрывающегося списка рядом с пунктом **Отправить** и выберите **Дополнительно**.

        ![Пакетный режим, нерегламентированный запрос Hive, кластер HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-query-batch.png)

        При выборе варианта дополнительная отправка появляется диалоговое окно **Отправка скрипта** . Настройте **имя задания**, **аргументы**, **дополнительные конфигурации** и **Каталог состояния** для скрипта.

        ![Диалоговое окно "Отправка скрипта", нерегламентированный запрос Hive, кластер HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-submit-jobs-advanced.png)

      > [!NOTE]  
      > Пакеты нельзя отправлять в кластеры интерактивных запросов.  Необходимо использовать интерактивный режим.

#### <a name="create-a-hive-application"></a>Создание приложения Hive

Для создания и запуска решения Hive сделайте следующее:

1. В строке меню выберите **файл**  >  **создать**  >  **проект**.

2. В окне **Создание нового проекта** выберите поле поиска и введите **Hive**. Затем выберите **приложение Hive** и нажмите кнопку **Далее**.

3. В окне **Настройка нового проекта** введите **имя проекта**, выберите или создайте **Расположение** проекта, а затем щелкните **создать**.

    ![Новое приложение Hive, Настройка окна нового проекта, HDInsight Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-new-hive-project.png)

4. В **обозревателе решений** дважды щелкните файл **Script.hql**, чтобы открыть его.

### <a name="view-job-summary-and-output"></a>Просмотр сводки и выходных данных задания

Сводка по заданию незначительно различается в **пакете** и в **интерактивном** режиме.

![Окна сводки по заданиям Hive, пакетный и интерактивный режимы, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-job-summary.png)

Используйте значок **обновления** , чтобы обновить состояние до тех пор, пока состояние задания не изменится на **завершено**.  

* Для сведений о задании из **пакетного** режима выберите ссылки внизу, чтобы просмотреть **запрос задания**, **выходные данные задания** или **Журнал заданий**, а также **Просмотреть журналы Yarn**.

* Сведения о задании из **интерактивного** режима см. в разделе **выходные** и **HiveServer2 выходные** области.

    ![Выходные данные интерактивных заданий Hive, кластер HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-job-details.png)

### <a name="view-job-graph"></a>Просмотр графа задания

В настоящее время графы заданий отображаются только для заданий Hive, использующих Tez в качестве подсистемы выполнения.  Сведения о включении Tez см. [в статье что такое Apache Hive и HiveQL в Azure HDInsight?](hdinsight-use-hive.md).  См. также [использование Apache Tez вместо сокращения Map](../hdinsight-hadoop-optimize-hive-query.md#use-apache-tez-instead-of-map-reduce).  

Чтобы просмотреть все операторы внутри вершины, дважды щелкните вершины графа задания. Можно выбрать также определенный оператор, чтобы увидеть о нем дополнительные сведения.

Даже если в качестве подсистемы выполнения указан Tez, граф задания может не отображаться, если приложение Tez не запускается.  Эта ситуация может возникать из-за того, что задание не содержит инструкций DML. Или, так как инструкции DML могут возвращаться без запуска приложения Tez. Например, `SELECT * FROM table1` не запускает приложение Tez.

![График задания Apache Hive, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-fast-path-hive-execution.png)

### <a name="view-task-execution-detail"></a>Просмотр сведений о выполнении задачи

На графе задания можно выбрать **сведения о выполнении задачи** для получения структурированных и наглядных сведений о заданиях Hive. Вы также можете получить дополнительные сведения о задании. При появлении проблем с производительностью представление можно использовать для получения дополнительных сведений о проблеме. Например, вы можете получить сведения о том, как работает каждая задача, а также подробные сведения о каждой задаче (чтение и запись данных, расписание, время начала и окончания работы). Эти наглядные сведения помогут вам изменить конфигурацию задания или архитектуру системы.

![Окно "представление выполнения задач", Data Lake Инструменты Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-task-execution-view.png)

### <a name="view-hive-jobs"></a>Просмотр заданий Hive

Для заданий Hive можно просмотреть запросы задания, выходные данные задания, журнал задания и журналы YARN.

В последнем выпуске средств можно увидеть, что находится внутри заданий Hive, собирая и отображая журналы Yarn. Журнал YARN может помочь исследовать проблемы производительности. Дополнительные сведения о сборе Yarn журналов в HDInsight см. в статье [доступ к журналам приложений Apache Hadoop Yarn](../hdinsight-hadoop-access-yarn-app-logs-linux.md).

Для просмотра заданий Hive сделайте следующее:

1. Щелкните правой кнопкой мыши кластер HDInsight и выберите **Просмотреть задания**.

    ![Просмотр заданий, Apache Hive, кластера HDInsight, Visual Studio](./media/apache-hadoop-visual-studio-tools-get-started/hdinsight-visual-studio-tools-view-hive-jobs.png)

    Появится список заданий Hive, выполняющихся в кластере.  

2. Выберите задание. В окне **Сводка по заданию Hive** выберите одну из следующих ссылок:
    - **Запрос задания**
    - **Выходные данные задания**
    - **Журнал заданий**  
    - **Журнал Yarn**

## <a name="run-apache-pig-scripts"></a>Запуск скриптов Apache Pig

1. В строке меню выберите **файл**  >  **создать**  >  **проект**.

2. В **начальном** окне выберите поле поиска и введите **Pig**. Затем выберите **приложение Pig** и нажмите кнопку **Далее**.

3. В окне **Настройка нового проекта** введите **имя проекта** и выберите или создайте **Расположение** для проекта. Щелкните **Создать**.

4. В панели **Обозреватель решений** IDE дважды щелкните **script. Pig** , чтобы открыть скрипт.

## <a name="feedback-and-known-issues"></a>Отзывы и известные проблемы

* Проблема, при которой результаты, начинающиеся со значений Null, не отображались, исправлена. Если эта проблема мешает вам работать, обратитесь в нашу службу поддержки.

* Скрипт HQL, создаваемый Visual Studio, кодируется в зависимости от настройки локальной области пользователя. Скрипт выполняется неправильно, если вы загружаете его в кластер как двоичный файл.

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как подключаться к кластерам HDInsight из Visual Studio, используя пакет средств Data Lake для Visual Studio, и выполнять запросы Hive. 

* [Run Apache Hive queries using the Data Lake tools for Visual Studio](apache-hadoop-use-hive-visual-studio.md) (Выполнение запросов Apache Hive с использованием средств Data Lake для Visual Studio)
* [Обзор Apache Hive и HiveQL в Azure HDInsight](hdinsight-use-hive.md)
* [Создание кластера Apache Hadoop — шаблон](apache-hadoop-linux-tutorial-get-started.md)
* [Отправка заданий Apache Hadoop в HDInsight](submit-apache-hadoop-jobs-programmatically.md)
* [Анализ данных Twitter с помощью Apache Hive и Apache Hadoop в HDInsight](../hdinsight-analyze-twitter-data-linux.md)
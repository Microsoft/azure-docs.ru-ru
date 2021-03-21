---
title: 'Azure Toolkit for IntelliJ: Отладка приложений Spark с помощью SSH-HDInsight'
description: Пошаговые инструкции по использованию инструментов HDInsight из набора средств Azure для IntelliJ для удаленной отладки приложений на кластерах HDInsight через SSH.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 12/23/2019
ms.openlocfilehash: ccd642578c8c35ac6b5f23397788ad1e7f83a1f5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98942615"
---
# <a name="debug-apache-spark-applications-on-an-hdinsight-cluster-with-azure-toolkit-for-intellij-through-ssh"></a>Отладка Apache Spark приложений в кластере HDInsight с Azure Toolkit for IntelliJ через SSH

В этой статье содержатся пошаговые инструкции по использованию средств HDInsight в [Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij) для удаленной отладки приложений в кластере HDInsight. Для отладки проекта можно также просмотреть видео [Debug HDInsight Spark applications with Azure Toolkit for IntelliJ](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ) (Отладка приложений HDInsight Spark с помощью набора средств Azure для IntelliJ).

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Создание кластера Apache Spark в HDInsight с помощью шаблона](../spark/apache-spark-jupyter-spark-sql-use-portal.md).

* Для пользователей Windows: при запуске локального приложения Spark Scala на компьютере Windows может возникнуть исключение, как описано в [Spark-2356](https://issues.apache.org/jira/browse/SPARK-2356). Это исключение возникает, так как в Windows отсутствует файл WinUtils.exe.

    Чтобы устранить эту ошибку, скачайте [Winutils.exe](https://github.com/steveloughran/winutils) в расположение, например **C:\WinUtils\bin**. После этого добавьте переменную среды **HADOOP_HOME** и присвойте ей значение **C\WinUtils**.

* [IntelliJная идея](https://www.jetbrains.com/idea/download/#section=windows) (выпуск Community Edition предоставляется бесплатно).

* [Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/installation).

* [Подключаемый модуль Scala для IntelliJ](../spark/apache-spark-intellij-tool-plugin.md#install-scala-plugin-for-intellij-idea).

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="create-a-spark-scala-application"></a>Создание приложения Spark Scala

1. Запустите IntelliJ IDEA и выберите **Create New Project** (Создать проект), чтобы открыть окно **New Project** (Новый проект).

1. На панели слева выберите **Azure Spark/HDInsight**.

1. Выберите **проект Spark с примерами (Scala)** в главном окне.

1. Из раскрывающегося списка **Build tool** (Инструмент сборки) выберите один из следующих вариантов:

    * **Maven** для поддержки мастера создания проекта Scala.
    * **SBT** для управления зависимостями и создания проекта Scala.

     ![IntelliJ создать новый проект Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-create-projectfor-debug-remotely.png)

1. Выберите **Далее**.

1. В следующем окне **нового проекта** укажите следующие сведения:

    |Свойство. |Описание |
    |---|---|
    |Имя проекта|Введите имя. В этом пошаговом руководстве используются `myApp` .|
    |Расположение проекта|Введите необходимое расположение для сохранения проекта.|
    |Project SDK (Пакет SDK проекта)|Если поле пусто, выберите **создать...** и перейдите к своему JDK.|
    |Версия Spark|Мастер создания интегрирует правильную версию пакетов SDK для Spark и Scala. Если используется версия кластера Spark более ранняя, чем 2.0, выберите **Spark 1.x**. В противном случае выберите **Spark 2. x.**. В этом примере используется **Spark 2.3.0 (Scala 2.11.8)** .|

   ![IntelliJ новый проект выберите версию Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-new-project.png)

1. Нажмите кнопку **Готово**. Может пройти несколько минут, прежде чем проект станет доступным. Просмотрите ход выполнения в нижнем правом углу.

1. Разверните проект и перейдите в раздел **src**  >  **Main**  >  **Scala**  >  **Sample**. Дважды щелкните **SparkCore_WasbIOTest**.

## <a name="perform-local-run"></a>Выполнение локального запуска

1. В скрипте **SparkCore_WasbIOTest** щелкните правой кнопкой мыши редактор скриптов и выберите пункт **выполнить "SparkCore_WasbIOTest"** , чтобы выполнить локальный запуск.

1. После завершения локального выполнения можно просмотреть выходной файл, который будет сохранен в **текущем обозревателе проектов**  >  **__по умолчанию__**.

    ![Результат локального запуска проекта IntelliJ](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/spark-local-run-result.png)

1. С помощью наших средств при локальном запуске и локальной отладке по умолчанию настраивается конфигурацию локального запуска. Откройте конфигурацию **[Spark в hdinsight] XXX** в правом верхнем углу, вы увидите **[Spark в hdinsight] XXX** , уже созданную в разделе **Apache Spark в HDInsight**. Откройте вкладку **Locally Run** (Локальный запуск).

    ![Локальное выполнение конфигураций запуска и отладки в Intellij](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/local-run-configuration.png)

    - [Переменные среды.](#prerequisites) Если вы уже задали переменную среды **HADOOP_HOME** для **C:\WinUtils**, она определяется автоматически и добавлять ее вручную не нужно.
    - [Расположение WinUtils.exe.](#prerequisites) Если вы не задали переменную среды, найдите расположение, нажав соответствующую кнопку.
    - Просто выберите один из вариантов (в MacOS и Linux в этом нет необходимости).

1. Кроме того, вы можете вручную задать конфигурацию перед локальным запуском и локальной отладкой. На предыдущем снимке экрана выберите знак "плюс" ( **+** ). Затем выберите параметр **Apache Spark в HDInsight** . Укажите **имя** и **имя класса Main**, чтобы сохранить их, а затем нажмите кнопку локального запуска.

## <a name="perform-local-debugging"></a>Локальная отладка

1. Откройте скрипт **SparkCore_wasbloTest** и задайте точки останова.

1. Щелкните правой кнопкой мыши редактор скриптов и выберите пункт **Отладка "[Spark в HDInsight] XXX"** , чтобы выполнить локальную отладку.

## <a name="perform-remote-run"></a>Удаленный запуск

1. Перейдите к разделу **выполнение**  >  **изменения конфигураций..**.. В этом меню можно создавать или изменять конфигурации для удаленной отладки.

1. В диалоговом окне **Run/Debug Configurations** (Конфигурации выполнения и отладки) щелкните знак "плюс" (**+**). Затем выберите параметр **Apache Spark в HDInsight** .

   ![IntelliJ добавить новую конфигурацию](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-add-new-Configuration.png)

1. Перейдите в режим **удаленного запуска на вкладке кластер** . Введите данные для **имени**, **кластера Spark** и **имени класса Main**. Затем щелкните **Расширенная конфигурация (удаленная отладка)**. Наши средства поддерживают отладку с **исполнителями**. Для параметра **numExectors** по умолчанию устанавливается значение 5. Лучше не указывать значение больше 3.

   ![IntelliJ запуск конфигураций отладки](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-run-debug-configurations.png)

1. В части **Расширенная конфигурация (удаленная отладка)** выберите **включить удаленную отладку Spark**. Введите имя пользователя SSH, затем введите пароль либо воспользуйтесь файлом закрытого ключа. Для выполнения удаленной отладки необходимо настроить ее. Чтобы использовать удаленный запуск, не нужно настраивать отладку.

   ![Расширенная конфигурация IntelliJ включение удаленной отладки Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-enable-spark-remote-debug.png)

1. Теперь конфигурация сохранена под именем, которое вы указали. Чтобы просмотреть сведения о конфигурации, выберите имя конфигурации. Чтобы внести изменения, выберите **Edit Configurations** (Изменить конфигурации).

1. Завершив настройку конфигурации, можно запустить проект на удаленном кластере или выполнить удаленную отладку.

   ![Кнопка удаленного запуска IntelliJ отладки удаленного задания Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/perform-remote-run-button.png)

1. Нажмите кнопку **Отключить**, чтобы журналы отправки не отображались на левой панели. Тем не менее она все еще выполняется в серверной части.

   ![Результат удаленного запуска IntelliJ отладки удаленного задания Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/spark-remote-run-result.png)

## <a name="perform-remote-debugging"></a>Удаленная отладка

1. Настройте точки останова, а затем щелкните значок **Remote debug** (Удаленная отладка). Отличие удаленной отладки заключается в том, что требуется настроить имя пользователя и пароль SSH.

   ![Значок отладки задания удаленной отладки Spark IntelliJ](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debug-icon.png)

1. Когда выполнение программы достигнет точки останова, в области **Debugger** (Отладчик) вы увидите вкладку **Driver** (Драйвер) и две вкладки **Executor** (Исполнитель). Щелкните значок **Resume Program** (Возобновить выполнение программы), чтобы продолжить выполнение кода, который затем достигнет следующей точки останова. Откройте соответствующую вкладку **Исполнитель**, чтобы найти целевой исполнитель для отладки. Вы можете просмотреть журналы выполнения на соответствующей вкладке **Консоль**.

   ![Вкладка "Отладка IntelliJ отладки заданий удаленного задания Spark"](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debugger-tab.png)

### <a name="perform-remote-debugging-and-bug-fixing"></a>Удаленная отладка и исправление ошибок

1. Настройте две точки останова и щелкните значок **Debug** (Отладка), чтобы приступить к удаленной отладке.

1. Выполнение кода будет приостановлено в первой точке останова, а в области **Variables** (Переменные) отобразятся сведения о параметре и переменной.

1. Щелкните значок **Resume Program** (Возобновить выполнение программы), чтобы продолжить. Выполнение кода будет приостановлено во второй точке останова. Как и ожидалось, возникнет исключение.

   ![Ошибка при вызове IntelliJ отладки для удаленного задания Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-throw-error.png)

1. Снова щелкните значок **Resume Program** (Возобновить выполнение программы). В окне **HDInsight Spark Submission** (Отправка HDInsight Spark) будет указано, что во время выполнения задания произошла ошибка.

   ![Отправка сообщения об ошибке IntelliJ отладки удаленного задания Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-error-submission.png)

1. Чтобы динамически обновить значение переменной с помощью функции отладки IntelliJ, еще раз щелкните значок **Debug** (Отладка). Снова появится область **Variables** (Переменные).

1. Щелкните правой кнопкой мыши целевой объект на вкладке **Debug** (Отладка), а затем выберите **Set Value** (Задать значение). Затем введите новое значение переменной. Нажмите клавишу **ВВОД**, чтобы сохранить значение.

   ![Значение набора заданий IntelliJ Debug Remote Spark](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-set-value1.png)

1. Щелкните значок **Resume Program** (Возобновить выполнение программы), чтобы продолжить выполнение программы. На этот раз исключение не возникает. Проект выполняется успешно без исключений.

   ![IntelliJ Отладка удаленного задания Spark без исключения](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debug-without-exception.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)

### <a name="demo"></a>Демонстрация

* Создание проекта Scala (видео): [создание приложений Scala Apache Spark](https://channel9.msdn.com/Series/AzureDataLake/Create-Spark-Applications-with-the-Azure-Toolkit-for-IntelliJ)
* Удаленная отладка (видео): [Use Azure Toolkit for IntelliJ to debug Spark applications remotely on an HDInsight cluster](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ) (Удаленная отладка приложений Apache Spark в кластере HDInsight с помощью Azure Toolkit for IntelliJ)

### <a name="scenarios"></a>Сценарии

* [Руководство. Анализ данных Apache Spark с использованием Power BI в HDInsight](apache-spark-use-bi-tools.md)
* [Apache Spark и Машинное обучение. Анализ температуры в здании на основе данных системы кондиционирования с помощью Spark в HDInsight](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark и Машинное обучение. Прогнозирование результатов проверки пищевых продуктов с помощью Spark в HDInsight](apache-spark-machine-learning-mllib-ipython.md)
* [Анализ журналов веб-сайтов с помощью Apache Spark в HDInsight](./apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>Создание и запуск приложений

* [Создание автономного приложения с использованием Scala](./apache-spark-create-standalone-application.md)
* [Удаленный запуск заданий с помощью Apache Livy в кластере Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Инструменты и расширения

* [Создание приложений Apache Spark для кластера HDInsight с помощью Azure Toolkit for IntelliJ](apache-spark-intellij-tool-plugin.md)
* [Удаленная отладка приложений Spark в HDInsight через VPN с помощью Azure Toolkit for IntelliJ](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Использование средств HDInsight в Azure Toolkit for Eclipse для создания приложений Apache Spark](./apache-spark-eclipse-tool-plugin.md)
* [Использование записных книжек Zeppelin с кластером Apache Spark в Azure HDInsight](apache-spark-zeppelin-notebook.md)
* [Ядра, доступные для Jupyter Notebook в кластере Apache Spark для HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Использование внешних пакетов с записными книжками Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
* [Установка записной книжки Jupyter на компьютере и ее подключение к кластеру Apache Spark в Azure HDInsight (предварительная версия)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>Управление ресурсами

* [Управление ресурсами кластера Apache Spark в Azure HDInsight](apache-spark-resource-manager.md)
* [Отслеживание и отладка заданий в кластере Apache Spark в HDInsight на платформе Linux](apache-spark-job-debugging.md)
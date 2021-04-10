---
title: Учебник по Azure Toolkit for IntelliJ (для приложения Spark)
description: Руководство по использованию Azure Toolkit for IntelliJ для разработки приложений Spark на языке Scala и их отправке в бессерверный пул Apache Spark.
services: synapse-analytics
author: jejiang
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 04/15/2020
ms.openlocfilehash: a4512a8f75d59f617058a50846d7578a90b9d062
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101675816"
---
# <a name="tutorial-create-an-apache-spark-application-with-intellij-using-a-synapse-workspace"></a>Руководство по созданию приложения Apache Spark с помощью IntelliJ и рабочей области Synapse

В этом руководстве показано, как с помощью подключаемого модуля Azure Toolkit for IntelliJ разрабатывать приложения Apache Spark на языке [Scala](https://www.scala-lang.org/) и отправлять их в бессерверный пул Apache Spark напрямую из интегрированной среды разработки (IDE) IntelliJ. С помощью подключаемого модуля можно выполнять следующие действия:

- разрабатывать и отправлять приложения Spark на Scala в пул Spark.
- получать доступ к ресурсам пулов Spark.
- разрабатывать и запускать приложения Scala Spark в локальной среде.

В этом руководстве описано следующее:
> [!div class="checklist"]
>
> - Использование подключаемого модуля Azure Toolkit for IntelliJ
> - Разработка приложений Apache Spark
> - Отправка приложения в пулы Spark

## <a name="prerequisites"></a>Предварительные требования

- [Версия IntelliJ IDEA для сообщества](https://www.jetbrains.com/idea/download/download-thanks.html?platform=windows&code=IIC);
- подключаемый модуль Azure Toolkit 3.27.0-2019.2 — [установите из репозитория подключаемого модуля IntelliJ](/java/azure/intellij/azure-toolkit-for-intellij-installation);
- [JDK (версия 1.8)](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html);
- подключаемый модуль Scala — [установите из репозитория подключаемого модуля IntelliJ](../../hdinsight/spark/apache-spark-intellij-tool-plugin.md#install-scala-plugin-for-intellij-idea).
- Приведенное ниже предварительное требование предназначено только для пользователей Windows.

  При запуске локального приложения Spark Scala на компьютере с Windows может возникнуть исключение, описанное в статье о [SPARK-2356](https://issues.apache.org/jira/browse/SPARK-2356). Это исключение возникает, так как в Windows отсутствует файл WinUtils.exe.
  Чтобы устранить эту ошибку, [скачайте этот исполняемый файл WinUtils](http://public-repo-1.hortonworks.com/hdp-win-alpha/winutils.exe), например, в папку **C:\WinUtils\bin**. После этого добавьте переменную среды **HADOOP_HOME** и присвойте ей значение **C\WinUtils**.

## <a name="create-a-spark-scala-application-for-a-spark-pool"></a>Создание приложения Spark Scala для пула Spark

1. Запустите IntelliJ IDEA и выберите **Create New Project** (Создать проект), чтобы открыть окно **New Project** (Новый проект).
2. На панели слева выберите **Azure Spark/HDInsight**.
3. В главном окне выберите **Spark Project with Samples(Scala)** (Проект Spark с примерами (Scala)).
4. Из раскрывающегося списка **Build tool** (Инструмент сборки) выберите один из следующих вариантов:

   - **Maven** для поддержки мастера создания проекта Scala.
   - **SBT** для управления зависимостями и создания проекта Scala.

    ![Диалоговое окно нового проекта в IntelliJ IDEA](./media/intellij-tool-synapse/create-synapse-application01.png)

5. Выберите **Далее**.
6. В окне **New Project** (Новый проект) укажите следующую информацию:

    | Свойство | Описание |
    | ----- | ----- |
    |Имя проекта| Введите имя. В этом учебнике используется `myApp`.|
    |Project&nbsp;location (Расположение проекта)| Введите необходимое расположение для сохранения проекта.|
    |Project SDK (Пакет SDK проекта)| Это поле может быть пустым при первом использовании IDEA. Выберите **New...** (Создать...) и перейдите к JDK.|
    |Версия Spark|Мастер создания интегрирует правильную версию пакетов SDK для Spark и Scala. Synapse поддерживает только **Spark 2.4.0**.|
    |||

    ![Выбор пакета SDK для Apache Spark](./media/intellij-tool-synapse/create-synapse-application02.png)

7. Нажмите кнопку **Готово**. Может пройти несколько минут, прежде чем проект станет доступным.
8. Проект Spark автоматически создает артефакт. Чтобы просмотреть артефакт, выполните указанные ниже действия.

   а. В строке меню выберите **File** (Файл) > **Project Structure...** (Структура проекта…).

   b. В окне **структуры проекта** щелкните **Artifacts** (Артефакты).

   c. После просмотра артефакта щелкните **Cancel** (Отменить).

    ![Сведения об артефакте в диалоговом окне](./media/intellij-tool-synapse/default-artifact-dialog.png)

9. Найдите раздел **LogQuery** в каталоге **myApp** > **src** > **main** > **scala**> **sample**> **LogQuery**. В этом учебнике для запуска используется **LogQuery**.

   ![Команды для создания класса Scala в проекте](./media/intellij-tool-synapse/select-run-file.png)

## <a name="connect-to-your-spark-pools"></a>Подключение к пулам Spark

Войдите в подписку Azure, чтобы подключиться к пулам Spark.

### <a name="sign-in-to-your-azure-subscription"></a>Войдите в свою подписку Azure.

1. В строке меню выберите **Представление** > **Окно инструментов** > **Azure Explorer**.

   ![В IntelliJ IDEA отображается Аzure Еxplorer](./media/intellij-tool-synapse/show-azure-explorer1.png)

2. В Azure Explorer щелкните правой кнопкой мыши узел **Azure**, а затем выберите **Войти**.

   ![Щелчок правой кнопкой мыши в Azure Explorer для IntelliJ IDEA](./media/intellij-tool-synapse/explorer-rightclick-azure.png)

3. В диалоговом окне **Вход в Azure** выберите **Имя пользователя устройства**, а затем — **Войти**.

    ![Вход в Azure для IntelliJ IDEA](./media/intellij-tool-synapse/intellij-view-explorer2.png)

4. В диалоговом окне **Azure Device Login** (Вход в систему устройства Azure) щелкните **Copy&Open** (Копировать и открыть).

   ![Вход устройства Azure для IntelliJ IDEA](./media/intellij-tool-synapse/intellij-view-explorer5.png)

5. В интерфейсе браузера вставьте код и нажмите кнопку **Далее**.

   ![Ввод кода в диалоговом окне HDI на сайте Microsoft](./media/intellij-tool-synapse/intellij-view-explorer6.png)

6. Введите учетные данные Azure и закройте браузер.

   ![Ввод электронной почты в диалоговом окне HDI на сайте Microsoft](./media/intellij-tool-synapse/intellij-view-explorer7.png)

7. После входа в диалоговом окне **Select Subscriptions** (Выбор подписок) будут перечислены все подписки Azure, связанные с указанными учетными данными. Выберите свою подписку и нажмите кнопку **Select** (Выбрать).

    ![Диалоговое окно выбора подписок](./media/intellij-tool-synapse/Select-Subscriptions.png)

8. В **Azure Explorer** разверните узел **Apache Spark on Synapse** (Apache Spark в Synapse), чтобы просмотреть рабочие области в своей подписке.

    ![Основное представление IntelliJ IDEA в Azure Explorer](./media/intellij-tool-synapse/azure-explorer-workspace.png)

9. Для просмотра пулов Spark можно дополнительно развернуть рабочую область.

    ![Учетные записи хранения Azure Explorer](./media/intellij-tool-synapse/azure-explorer-pool.png)

## <a name="remote-run-a-spark-scala-application-on-a-spark-pool"></a>Удаленный запуск приложения Spark Scala в пуле Spark

После создания приложения Scala его можно запустить удаленно.

1. Откройте окно **Run/Debug Configurations** (Конфигурации запуска и отладки), щелкнув значок.

    ![Команда 1 Submit Spark Application to HDInsight (Отправить приложение Spark в HDInsight)](./media/intellij-tool-synapse/open-configuration-window.png)

2. В диалоговом окне **Run/Debug Configurations** (Конфигурации запуска и отладки) щелкните **+** , а затем выберите **Apache Spark on Synapse** (Apache Spark в Synapse).

    ![Команда 2 Submit Spark Application to HDInsight (Отправить приложение Spark в HDInsight)](./media/intellij-tool-synapse/create-synapse-configuration02.png)

3. В окне **Run/Debug Configurations** (Конфигурации запуска и отладки) укажите следующие значения, а затем нажмите кнопку **ОК**.

    |Свойство |Значение |
    |----|----|
    |Пулы Spark|Выберите пулы Spark, в которых хотите запустить приложение.|
    |Select an Artifact to submit (Выбор артефакта для запуска)|Оставьте параметр по умолчанию.|
    |Имя главного класса|значение по умолчанию — имя главного класса из выбранного файла. Класс можно изменить, нажав значок-многоточие ( **...** ) и выбрав другой класс.|
    |Job configurations (Конфигурация заданий)|Вы можете изменить ключи или значения по умолчанию. Дополнительные сведения см. в статье [Apache Livy REST API](http://livy.incubator.apache.org./docs/latest/rest-api.html).|
    |Аргументы командной строки|При необходимости можно ввести аргументы для основного класса, разделив их пробелом.|
    |Referenced Jars (Ссылки на JAR-файлы) и Referenced Files (Ссылки на файлы)|можно ввести пути к используемым JAR и файлам, если они есть. Вы можете также просматривать файлы в виртуальной файловой системе Azure, которая сейчас поддерживает только кластер ADLS 2 поколения. Дополнительные сведения см. здесь: [Конфигурация Apache Spark]https://spark.apache.org/docs/2.4.5/configuration.html#runtime-environment) и [Краткое руководство. Создание большого двоичного объекта с помощью Обозревателя службы хранилища Azure](../../storage/blobs/storage-quickstart-blobs-storage-explorer.md).|
    |Job Upload Storage (Хранилище обновлений заданий)|Разверните раздел, чтобы отобразить дополнительные параметры.|
    |Тип хранения|В раскрывающемся списке выберите **Use Azure Blob to upload** (Для отправки использовать BLOB-объект Azure) или **Use cluster default storage account to upload** (Для отправки использовать учетную запись хранения по умолчанию для кластера).|
    |Учетная запись хранения|Введите имя своей учетной записи хранения.|
    |Storage Key (Ключ хранилища)|Введите свой ключ к хранилищу данных.|
    |Контейнер хранилища|Если вы ввели значения в поля **Учетная запись хранения** и **Storage Key** (Ключ хранилища), из раскрывающегося списка выберите контейнер хранилища.|

    ![Диалоговое окно 1 Spark Submission (Отправка в Spark)](./media/intellij-tool-synapse/create-synapse-configuration03.png)

4. Щелкните значок **SparkJobRun**, чтобы отправить проект в выбранный пул Spark. На вкладке **Remote Spark Job in Cluster** (Удаленное задание Spark в кластере) внизу показан ход выполнения задания. Чтобы остановить работу приложения, нажмите красную кнопку.

    ![Окно отправки Apache Spark](./media/intellij-tool-synapse/remotely-run-synapse.png)

    ![Диалоговое окно 2 Spark Submission (Отправка в Spark)](./media/intellij-tool-synapse/remotely-run-result.png)

## <a name="local-rundebug-apache-spark-applications"></a>Локальные запуск и отладка приложений Apache Spark

Чтобы настроить локальные запуск и отладку для задания Apache Spark, следуйте приведенным ниже указаниям.

### <a name="scenario-1-do-local-run"></a>Сценарий 1. Локальный запуск

1. В диалоговом окне **Run/Debug Configurations** (Конфигурации выполнения и отладки) щелкните значок плюса ( **+** ). Затем выберите вариант **Apache Spark on Synapse** (Apache Spark в Synapse). Введите сведения в полях **Name** (Имя) и **Main class name** (Имя основного класса).

    ![Локальное выполнение конфигураций запуска и отладки в IntelliJ (1)](./media/intellij-tool-synapse/local-run-synapse.png)

    - Переменные среды и расположение файла WinUtils.exe используются только для Windows.
    - Переменные среды. Если системная переменная среды настроена ранее, она может быть обнаружена автоматически и ее не нужно добавлять вручную.
    - [Расположение WinUtils.exe](http://public-repo-1.hortonworks.com/hdp-win-alpha/winutils.exe). Чтобы указать расположение WinUtils, щелкните значок папки справа.

2. Затем нажмите кнопку локального выполнения.

    ![Локальное выполнение конфигураций запуска и отладки в IntelliJ (2)](./media/intellij-tool-synapse/local-run-synapse01.png)

3. После завершения локального выполнения, если скрипт содержит выходные данные, можно проверить выходной файл в папке **data** > **__default__**.

    ![Результат локального запуска проекта IntelliJ (1)](./media/intellij-tool-synapse/spark-local-run-result.png)

### <a name="scenario-2-do-local-debugging"></a>Сценарий 2. Локальная отладка

1. Откройте скрипт **LogQuery** и установите точки останова.
2. Щелкните значок **Local debug** (Локальная отладка), чтобы выполнить локальную отладку.

    ![Результат локального запуска проекта IntelliJ (2)](./media/intellij-tool-synapse/local-debug-synapse.png)

## <a name="access-and-manage-synapse-workspace"></a>Доступ к рабочей области Synapse и управление ею

В Azure Explorer можно выполнять различные операции в Azure Toolkit для IntelliJ. В строке меню выберите **Представление** > **Окно инструментов** > **Azure Explorer**.

### <a name="launch-workspace"></a>Запуск рабочей области

1. В Azure Explorer перейдите в раздел **Apache Spark on Synapse** (Apache Spark в Synapse), а затем разверните его.

    ![Основное представление IntelliJ IDEA в Azure Explorer](./media/intellij-tool-synapse/azure-explorer-workspace.png)

2. Щелкните правой кнопкой мыши рабочую область, а затем выберите команду **Запуск рабочей области**, после чего откроется веб-сайт.

    ![Представление задания Spark, сведения о приложении (1)](./media/intellij-tool-synapse/launch-workspace-synapse.png)

    ![Представление задания Spark, сведения о приложении (2)](./media/intellij-tool-synapse/launch-workspace-result.png)

## <a name="spark-console"></a>Консоль Spark

Вы можете запустить локальную консоль Spark (Scala) или консоль интерактивного сеанса Spark Livy (Scala).

### <a name="spark-local-console-scala"></a>Локальная консоль Spark (Scala)

Убедитесь в том, что есть необходимый файл WINUTILS.EXE.

1. В строке меню выберите **Run (Запуск)**  > **Edit Configurations...** (Изменить конфигурации).
2. В левой области окна **Run/Debug Configurations** (Конфигурации запуска и отладки) выберите элементы **Apache Spark on Synapse** (Apache Spark в Synapse) >  **[Spark on Synapse] myApp** (myApp [Spark в Synapse]).
3. В главном окне выберите вкладку **Locally Run** (Локальный запуск).
4. Укажите следующие значения и нажмите кнопку **ОК**:

    |Свойство |Значение |
    |----|----|
    |Переменные среды|Проверьте правильность значения HADOOP_HOME.|
    |Расположение файла WINUTILS.exe|Проверьте правильность пути.|

    ![Настройка конфигурации в локальной консоли](./media/intellij-tool-synapse/local-console-synapse01.png)

5. В проекте перейдите в **myApp** > **src** > **main** > **scala** > **myApp**.
6. В строке меню выберите **Tools (Средства)**  > **Spark Console (Консоль Spark)**  > **Run Spark Local Console(Scala)** (Запустить локальную консоль Spark (Scala)).
7. После этого могут появиться два диалоговых окна с запросом на автоматическое исправление зависимостей. Если нужно исправить их, выберите **Auto Fix** (Автоматическое исправление).

    ![IntelliJ IDEA Spark автоматическое исправление dialog1](./media/intellij-tool-synapse/intellij-console-autofix1.png)

    ![IntelliJ IDEA Spark автоматическое исправление dialog2](./media/intellij-tool-synapse/intellij-console-autofix2.png)

8. Консоль должна выглядеть примерно так, как показано ниже. В окне консоли введите `sc.appName` и нажмите клавиши CTRL+ВВОД. Отобразится результат. Чтобы закрыть локальную консоль, нажмите красную кнопку.

    ![Результат локальной консоли IntelliJ IDEA](./media/intellij-tool-synapse/local-console-result.png)

### <a name="spark-livy-interactive-session-console-scala"></a>Консоль интерактивного сеанса Spark Livy (Scala)

Она поддерживается только в IntelliJ 2018.2 и 2018.3.

1. В строке меню выберите **Run (Запуск)**  > **Edit Configurations...** (Изменить конфигурации).

2. В левой области окна **Run/Debug Configurations** (Конфигурации запуска и отладки) выберите элементы **Apache Spark on Synapse** (Apache Spark в Synapse) >  **[Spark on Synapse] myApp** (myApp [Spark в Synapse]).

3. В главном окне выберите вкладку **Remotely Run in Cluster** (Удаленный запуск в кластере).

4. Укажите следующие значения и нажмите кнопку **ОК**:

    |Свойство |Значение |
    |----|----|
    |Имя главного класса| Выберите имя класса main.| 
    |Пулы Spark|Выберите пулы Spark, в которых хотите запустить приложение.|
    ||

    ![Настройка конфигурации в интерактивной консоли](./media/intellij-tool-synapse/interactive-console-configuration.png)

5. В проекте перейдите в **myApp** > **src** > **main** > **scala** > **myApp**.

6. В строке меню выберите **Tools (Средства)**  > **Spark Console (Консоль Spark)**  > **Run Spark Livy Interactive Session Console(Scala)** (Запустить консоль интерактивного сеанса Spark Livy (Scala)).
7. Консоль должна выглядеть примерно так, как показано ниже. В окне консоли введите `sc.appName` и нажмите клавиши CTRL+ВВОД. Отобразится результат. Чтобы закрыть локальную консоль, нажмите красную кнопку.

    ![Результат интерактивной консоли IntelliJ IDEA](./media/intellij-tool-synapse/interactive-console-result.png)

### <a name="send-selection-to-spark-console"></a>Отправка выбранного фрагмента кода в консоль Spark

Вы можете увидеть результат выполнения скрипта, отправив код в локальную консоль или консоль интерактивного сеанса Livy (Scala). Для этого выделите код в файле Scala, а затем в контекстном меню выберите команду **Send Selection To Spark Console** (Отправить выделенный фрагмент в консоль Spark). Выделенный код будет отправлен в консоль и выполнен. Результат отобразится в консоли после кода. Консоль проверит наличие ошибок.

   ![Отправка выбранного фрагмента кода в консоль Spark](./media/intellij-tool-synapse/send-selection-to-console.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Azure Synapse Analytics](../overview-what-is.md)
- [Создание пула Apache Spark для рабочей области Azure Synapse Analytics](../../synapse-analytics/quickstart-create-apache-spark-pool-studio.md)
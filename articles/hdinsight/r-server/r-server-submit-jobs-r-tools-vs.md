---
title: Отправка заданий из расширения "Инструменты R для Visual Studio" в Azure HDInsight
description: Отправка заданий R с локального компьютера Visual Studio в кластер HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 06/19/2019
ms.openlocfilehash: ee984de22052076618728fbacfc31b73c18c073a
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864692"
---
# <a name="submit-jobs-from-r-tools-for-visual-studio"></a>Отправка заданий из расширения "Инструменты R для Visual Studio"

[Инструменты R для Visual Studio](https://marketplace.visualstudio.com/items?itemName=MikhailArkhipov007.RTVS2019) (RTVS) — это бесплатное расширение с открытым исходным кодом для выпусков Community (бесплатно), Professional и Enterprise как [Visual Studio 2017](https://www.visualstudio.com/downloads/), так и [Visual Studio 2015 с обновлением 3](https://go.microsoft.com/fwlink/?LinkId=691129) или выше. RTVS недоступен для [Visual Studio 2019](/visualstudio/porting/port-migrate-and-upgrade-visual-studio-projects?preserve-view=true&view=vs-2019).

Расширение RTVS улучшает ваш рабочий процесс R, предлагая такие инструменты, как [Интерактивное окно R](/visualstudio/rtvs/interactive-repl) (REPL), технология IntelliSense (завершение кода), [визуализация в виде графиков](/visualstudio/rtvs/visualizing-data) через библиотеки R, такие как ggplot2 и ggviz, [отладка кода R](/visualstudio/rtvs/debugging) и другие.

## <a name="set-up-your-environment"></a>Настройка среды

1. Установите [Инструменты R для Visual Studio](/visualstudio/rtvs/installing-r-tools-for-visual-studio).

    :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/install-r-tools-for-vs.png" alt-text="Установка RTVS в Visual Studio 2017" border="true":::

2. Выберите рабочую нагрузку *Приложения для обработки и анализа данных и аналитические приложения*, а затем выберите пункты **Поддержка языка R**, **Поддержка среды выполнения для разработки R** и **Microsoft R Client**.

3. Для проверки подлинности по SSH необходимы открытые и закрытые ключи.
   <!-- {TODO tbd, no such file yet}[use SSH with HDInsight](hdinsight-hadoop-linux-use-ssh-windows.md) -->

4. Установите на компьютер [ML Server](/previous-versions/machine-learning-server/install/r-server-install-windows). ML Server предоставляет [`RevoScaleR`](/machine-learning-server/r-reference/revoscaler/revoscaler) функции и `RxSpark` .

5. Установите [PuTTY](https://www.putty.org/), чтобы предоставить контекст вычисления для выполнения функций `RevoScaleR` из локального клиента к кластеру HDInsight.

6. Вы можете применить параметры обработки и анализа данных к среде Visual Studio, которая предоставляет новый макет рабочего пространства для инструментов R.
   1. Чтобы сохранить текущие параметры Visual Studio, используйте команду **Инструменты > Импорт и экспорт параметров**, а затем выберите **Экспортировать выбранные параметры среды** и укажите имя файла. Чтобы восстановить эти параметры, используйте ту же команду и выберите **Импортировать выбранные параметры среды**.

   2. Перейдите в пункт меню **Инструменты R**, а затем выберите **Параметры обработки и анализа данных...**

       :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/data-science-settings.png" alt-text="Visual Studio Параметры обработки и анализа данных" border="true":::

      > [!NOTE]  
      > С помощью подхода из шага 1 вы также можете сохранить и восстановить персонализированный макет обработки и анализа данных вместо того, чтобы повторять команду **Параметры обработки и анализа данных**.

## <a name="execute-local-r-methods"></a>Выполнение локальных методов R

1. Создайте кластер HDInsight для Служб машинного обучения.
2. Установите [расширение RTVS](/visualstudio/rtvs/installation).
3. Загрузите [ZIP-файл с примерами](https://github.com/Microsoft/RTVS-docs/archive/master.zip).
4. Откройте `examples/Examples.sln` для запуска решения в Visual Studio.
5. Откройте файл `1-Getting Started with R.R` в папке решений `A first look at R`.
6. Начиная сверху файла, нажимайте клавиши CTRL+ENTER, чтобы по очереди отправлять каждую строку в интерактивное окно R. Для некоторых строк это может занять определенное время, так как код в них устанавливает новые пакеты.
    * Кроме того, вы можете выбрать все строки в файле R (CTRL+A) и выполнить их все (CTRL+ENTER) либо на панели инструментов выбрать значок выполнения в интерактивном режиме.

        :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/execute-interactive1.png" alt-text="Интерактивное выполнение Visual Studio" border="true":::

7. Когда все строки в сценарии будут выполнены, у вас должен быть следующий результат:

    :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/visual-studio-workspace.png" alt-text="Инструменты R рабочей области Visual Studio" border="true":::

## <a name="submit-jobs-to-an-hdinsight-ml-services-cluster"></a>Отправка заданий в кластер HDInsight R для Служб машинного обучения

С помощью Microsoft ML Server или Microsoft R Client с компьютера Windows, где установлена программа PuTTY, вы можете создать контекст вычисления, который будет запускать распределенные функции `RevoScaleR` из вашего локального клиента к вашему кластеру HDInsight. Используйте `RxSpark`, чтобы создать контекст вычисления, указав свое имя пользователя, граничный узел кластера Apache Hadoop, ключи SSH и т. д.

1. В качестве адреса узла служб ML в HDInsight используется `CLUSTERNAME-ed-ssh.azurehdinsight.net` `CLUSTERNAME` имя кластера служб ml.

1. Вставьте следующий код в интерактивное окно R в Visual Studio, изменив значения переменных настройки в соответствии с вашей средой.

    ```R
    # Setup variables that connect the compute context to your HDInsight cluster
    mySshHostname <- 'r-cluster-ed-ssh.azurehdinsight.net ' # HDI secure shell hostname
    mySshUsername <- 'sshuser' # HDI SSH username
    mySshClientDir <- "C:\\Program Files (x86)\\PuTTY"
    mySshSwitches <- '-i C:\\Users\\azureuser\\r.ppk' # Path to your private ssh key
    myHdfsShareDir <- paste("/user/RevoShare", mySshUsername, sep = "/")
    myShareDir <- paste("/var/RevoShare", mySshUsername, sep = "/")
    mySshProfileScript <- "/usr/lib64/microsoft-r/3.3/hadoop/RevoHadoopEnvVars.site"

    # Create the Spark Cluster compute context
    mySparkCluster <- RxSpark(
          sshUsername = mySshUsername,
          sshHostname = mySshHostname,
          sshSwitches = mySshSwitches,
          sshProfileScript = mySshProfileScript,
          consoleOutput = TRUE,
          hdfsShareDir = myHdfsShareDir,
          shareDir = myShareDir,
          sshClientDir = mySshClientDir
    )

    # Set the current compute context as the Spark compute context defined above
    rxSetComputeContext(mySparkCluster)
    ```

   :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/apache-spark-context.png" alt-text="Установка контекста Apache Spark" border="true":::

1. В интерактивном окне R выполните следующие команды:

    ```R
    rxHadoopCommand("version") # should return version information
    rxHadoopMakeDir("/user/RevoShare/newUser") # creates a new folder in your storage account
    rxHadoopCopy("/example/data/people.json", "/user/RevoShare/newUser") # copies file to new folder
    ```

    Должен отобразиться результат, аналогичный приведенному ниже:

    :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/successful-rx-commands.png" alt-text="Успешное выполнение команды rx" border="true":::
а
1. Убедитесь, что команда `rxHadoopCopy` успешно скопировала файл `people.json` из папки данных для примера в недавно созданную папку `/user/RevoShare/newUser`:

    1. На панели кластера HDInsight для Служб машинного обучения в Azure в меню слева выберите **Учетные записи хранения**.

        :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/hdinsight-storage-accounts.png" alt-text="Учетные записи хранения Azure HDInsight" border="true":::

    2. Выберите учетную запись хранения по умолчанию для своего кластера, записав имя контейнера и каталога.

    3. На панели учетных записей хранения в меню слева выберите **Контейнеры**.

        :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/hdi-storage-containers.png" alt-text="Контейнеры хранилища Azure HDInsight" border="true":::

    4. Выберите имя контейнера вашего кластера, перейдите в папку **пользователя** (возможно, внизу списка нужно будет нажать *Загрузить еще*), выберите *RevoShare*, а затем — **newUser**. Файл `people.json` должен появиться в папке `newUser`.

        :::image type="content" source="./media/r-server-submit-jobs-r-tools-vs/hdinsight-copied-file.png" alt-text="Расположение папки &quot;скопированные файлы HDInsight&quot;" border="true":::

1. После этого остановите выполнение текущего контекста Apache Spark. Одновременное выполнение нескольких контекстов не поддерживается.

    ```R
    rxStopEngine(mySparkCluster)
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Параметры контекста вычислений для Служб машинного обучения в HDInsight](r-server-compute-contexts.md)
* Пример прогнозирования задержки рейсов см. в статье [Совместное использование ScaleR и SparkR в HDInsight](../hdinsight-hadoop-r-scaler-sparkr.md).
---
title: Руководство. Использование R в контексте вычислений Spark в Azure HDInsight
description: Руководство. Начало работы с R и Spark в кластере служб машинного обучения Azure HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 06/21/2019
ms.openlocfilehash: 07c004fa5f03c53ed0778613a0c16a0ba2aff11f
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104869503"
---
# <a name="tutorial-use-r-in-a-spark-compute-context-in-azure-hdinsight"></a>Руководство. Использование R в контексте вычислений Spark в Azure HDInsight

В этом руководстве содержится пошаговое описание использования функций R на платформе Apache Spark, запущенной в кластере служб машинного обучения Azure HDInsight.

В этом руководстве описано следующее.

> [!div class="checklist"]
> * Скачивание демонстрационных данных в локальное хранилище
> * Копирование данных в хранилище по умолчанию
> * Настройка набора данных
> * Создание источников данных
> * Создание контекста вычислений для Spark
> * Обеспечение соответствия линейной модели
> * Использование составных XDF-файлов
> * Преобразование XDF-файлов в CSV-файлы

## <a name="prerequisites"></a>Предварительные условия

* Кластер служб машинного обучения Azure HDInsight Перейдите на сайт [Create Linux-based clusters in HDInsight by using the Azure portal](../hdinsight-hadoop-create-linux-clusters-portal.md) (Создание кластеров под управлением Linux в HDInsight с помощью портала Azure) и для параметра **Тип кластера** выберите **Службы машинного обучения ML Services**.

## <a name="connect-to-rstudio-server"></a>Подключение к RStudio Server

RStudio Server запускается на граничном узле кластера. Перейдите на следующий сайт (где *CLUSTERNAME* в URL-адресе — это имя созданного вами кластера служб машинного обучения HDInsight):

```
https://CLUSTERNAME.azurehdinsight.net/rstudio/
```

При первом входе вы пройдете аутентификацию дважды. При первом запросе на аутентификацию укажите имя пользователя и пароль администратора кластера. Значением по умолчанию является *admin*. При втором запросе на аутентификацию укажите имя пользователя и пароль SSH. Значением по умолчанию является *sshuser*. При последующих операциях входа вам будет необходимо указывать лишь данные для входа SSH.

## <a name="download-the-sample-data-to-local-storage"></a>Скачивание демонстрационных данных в локальное хранилище

*Набор данных о расписании авиакомпании за 2012 год* состоит из 12 файлов с разделителями-запятыми, содержащих сведения о прибытии и отправлении всех коммерческих авиарейсов в США за 2012 год. Этот набор данных содержит более 6 млн наблюдений.

1. Инициализируйте несколько переменных среды. В консоли RStudio Server введите следующий код:

    ```R
    bigDataDirRoot <- "/tutorial/data" # root directory on cluster default storage
    localDir <- "/tmp/AirOnTimeCSV2012" # directory on edge node
    remoteDir <- "https://packages.revolutionanalytics.com/datasets/AirOnTimeCSV2012" # location of data
    ```

1. В правой панели выберите вкладку **Среда**. Переменные будут отображены в разделе **Значения**.

    :::image type="content" source="./media/ml-services-tutorial-spark-compute/hdinsight-rstudio-image.png" alt-text="Веб-консоль R Studio в HDInsight" border="true":::

1. Создайте локальный каталог и скачайте пример данных. Введите приведенный ниже код в RStudio.

    ```R
    # Create local directory
    dir.create(localDir)
    
    # Download data to the tmp folder(local)
    download.file(file.path(remoteDir, "airOT201201.csv"), file.path(localDir, "airOT201201.csv"))
    download.file(file.path(remoteDir, "airOT201202.csv"), file.path(localDir, "airOT201202.csv"))
    download.file(file.path(remoteDir, "airOT201203.csv"), file.path(localDir, "airOT201203.csv"))
    download.file(file.path(remoteDir, "airOT201204.csv"), file.path(localDir, "airOT201204.csv"))
    download.file(file.path(remoteDir, "airOT201205.csv"), file.path(localDir, "airOT201205.csv"))
    download.file(file.path(remoteDir, "airOT201206.csv"), file.path(localDir, "airOT201206.csv"))
    download.file(file.path(remoteDir, "airOT201207.csv"), file.path(localDir, "airOT201207.csv"))
    download.file(file.path(remoteDir, "airOT201208.csv"), file.path(localDir, "airOT201208.csv"))
    download.file(file.path(remoteDir, "airOT201209.csv"), file.path(localDir, "airOT201209.csv"))
    download.file(file.path(remoteDir, "airOT201210.csv"), file.path(localDir, "airOT201210.csv"))
    download.file(file.path(remoteDir, "airOT201211.csv"), file.path(localDir, "airOT201211.csv"))
    download.file(file.path(remoteDir, "airOT201212.csv"), file.path(localDir, "airOT201212.csv"))
    ```

    Скачивание займет около 9.5 с половиной минут.

## <a name="copy-the-data-to-default-storage"></a>Копирование данных в хранилище по умолчанию

Расположение распределенной файловой системы Hadoop указано с помощью переменной `airDataDir`. Введите приведенный ниже код в RStudio.

```R
# Set directory in bigDataDirRoot to load the data into
airDataDir <- file.path(bigDataDirRoot,"AirOnTimeCSV2012")

# Create directory (default storage)
rxHadoopMakeDir(airDataDir)

# Copy data from local storage to default storage
rxHadoopCopyFromLocal(localDir, bigDataDirRoot)
    
# Optional. Verify files
rxHadoopListFiles(airDataDir)
```

Выполнение этого шага займет около 10 секунд.

## <a name="set-up-a-dataset"></a>Настройка набора данных

1. Создайте объект файловой системы, в котором используются значения по умолчанию. Введите приведенный ниже код в RStudio.

    ```R
    # Define the HDFS (WASB) file system
    hdfsFS <- RxHdfsFileSystem()
    ```

1. Поскольку исходные CSV-файлы имеют довольно громоздкие имена переменных, используется список *colInfo*, чтобы они лучше поддавались управлению. Введите приведенный ниже код в RStudio.

    ```R
    airlineColInfo <- list(
         MONTH = list(newName = "Month", type = "integer"),
        DAY_OF_WEEK = list(newName = "DayOfWeek", type = "factor",
            levels = as.character(1:7),
            newLevels = c("Mon", "Tues", "Wed", "Thur", "Fri", "Sat",
                          "Sun")),
        UNIQUE_CARRIER = list(newName = "UniqueCarrier", type =
                                "factor"),
        ORIGIN = list(newName = "Origin", type = "factor"),
        DEST = list(newName = "Dest", type = "factor"),
        CRS_DEP_TIME = list(newName = "CRSDepTime", type = "integer"),
        DEP_TIME = list(newName = "DepTime", type = "integer"),
        DEP_DELAY = list(newName = "DepDelay", type = "integer"),
        DEP_DELAY_NEW = list(newName = "DepDelayMinutes", type =
                             "integer"),
        DEP_DEL15 = list(newName = "DepDel15", type = "logical"),
        DEP_DELAY_GROUP = list(newName = "DepDelayGroups", type =
                               "factor",
           levels = as.character(-2:12),
           newLevels = c("< -15", "-15 to -1","0 to 14", "15 to 29",
                         "30 to 44", "45 to 59", "60 to 74",
                         "75 to 89", "90 to 104", "105 to 119",
                         "120 to 134", "135 to 149", "150 to 164",
                         "165 to 179", ">= 180")),
        ARR_DELAY = list(newName = "ArrDelay", type = "integer"),
        ARR_DELAY_NEW = list(newName = "ArrDelayMinutes", type =
                             "integer"),  
        ARR_DEL15 = list(newName = "ArrDel15", type = "logical"),
        AIR_TIME = list(newName = "AirTime", type =  "integer"),
        DISTANCE = list(newName = "Distance", type = "integer"),
        DISTANCE_GROUP = list(newName = "DistanceGroup", type =
                             "factor",
         levels = as.character(1:11),
         newLevels = c("< 250", "250-499", "500-749", "750-999",
             "1000-1249", "1250-1499", "1500-1749", "1750-1999",
             "2000-2249", "2250-2499", ">= 2500")))
    
    varNames <- names(airlineColInfo)
    ```

## <a name="create-data-sources"></a>Создание источников данных

В контексте вычислений Spark можно создать источники данных, используя следующие функции:

|Компонент | Описание |
|---------|-------------|
|`RxTextData` | Текстовый источник данных с разделителями-запятыми. |
|`RxXdfData` | Данные в формате файла данных XDF. В RevoScaleR формат файла XDF изменяется для Hadoop, чтобы хранить данные в составном наборе файлов, а не в одном файле. |
|`RxHiveData` | Создает объект источника данных Hive.|
|`RxParquetData` | Создает объект источника данных Parquet.|
|`RxOrcData` | Создает объект источника данных Orc.|

Создайте объект [RxTextData](/machine-learning-server/r-reference/revoscaler/rxtextdata) с использованием файлов, скопированных в HDFS. Введите приведенный ниже код в RStudio.

```R
airDS <- RxTextData( airDataDir,
                        colInfo = airlineColInfo,
                        varsToKeep = varNames,
                        fileSystem = hdfsFS ) 
```

## <a name="create-a-compute-context-for-spark"></a>Создание контекста вычислений для Spark

Для загрузки данных и выполнения анализа на рабочих узлах для контекста вычислений в скрипте необходимо задать [RxSpark](/machine-learning-server/r-reference/revoscaler/rxspark). В этом контексте функции R автоматически распределяют рабочую нагрузку между всеми рабочими узлами с помощью невстроенного требования для управления заданиями или очередью. Контекст вычислений Spark устанавливается через `RxSpark` или `rxSparkConnect()`, а также использует `rxSparkDisconnect()` для возврата в локальный контекст вычислений. Введите приведенный ниже код в RStudio.

```R
# Define the Spark compute context
mySparkCluster <- RxSpark()

# Set the compute context
rxSetComputeContext(mySparkCluster)
```

## <a name="fit-a-linear-model"></a>Обеспечение соответствия линейной модели

1. Используйте функцию [rxLinMod](/machine-learning-server/r-reference/revoscaler/rxlinmod) для обеспечения соответствия линейной модели с использованием вашего источника данных `airDS`. Введите приведенный ниже код в RStudio.

    ```R
    system.time(
         delayArr <- rxLinMod(ArrDelay ~ DayOfWeek, data = airDS,
              cube = TRUE)
    )
    ```
    
    Выполнение этого шага должно занять от 2 до 3 минут.

1. Просмотрите результаты. Введите приведенный ниже код в RStudio.

    ```R
    summary(delayArr)
    ```

    Вы увидите следующие результаты:

    ```output
    Call:
    rxLinMod(formula = ArrDelay ~ DayOfWeek, data = airDS, cube = TRUE)
    
    Cube Linear Regression Results for: ArrDelay ~ DayOfWeek
    Data: airDataXdf (RxXdfData Data Source)
    File name: /tutorial/data/AirOnTimeCSV2012
    Dependent variable(s): ArrDelay
    Total independent variables: 7 
    Number of valid observations: 6005381
    Number of missing observations: 91381 
     
    Coefficients:
                   Estimate Std. Error t value Pr(>|t|)     | Counts
    DayOfWeek=Mon   3.54210    0.03736   94.80 2.22e-16 *** | 901592
    DayOfWeek=Tues  1.80696    0.03835   47.12 2.22e-16 *** | 855805
    DayOfWeek=Wed   2.19424    0.03807   57.64 2.22e-16 *** | 868505
    DayOfWeek=Thur  4.65502    0.03757  123.90 2.22e-16 *** | 891674
    DayOfWeek=Fri   5.64402    0.03747  150.62 2.22e-16 *** | 896495
    DayOfWeek=Sat   0.91008    0.04144   21.96 2.22e-16 *** | 732944
    DayOfWeek=Sun   2.82780    0.03829   73.84 2.22e-16 *** | 858366
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
    
    Residual standard error: 35.48 on 6005374 degrees of freedom
    Multiple R-squared: 0.001827 (as if intercept included)
    Adjusted R-squared: 0.001826 
    F-statistic:  1832 on 6 and 6005374 DF,  p-value: < 2.2e-16 
    Condition number: 1 
    ```

    Результаты указывают, что вы обработали все данные (6 млн наблюдений) с помощью CSV-файлов в указанном каталоге. Так как указано значение `cube = TRUE`, у вас есть предполагаемый коэффициент для каждого дня недели (а не отсекаемого отрезка).

## <a name="use-composite-xdf-files"></a>Использование составных XDF-файлов

Как вы уже видели, вы можете анализировать CSV-файлы напрямую с помощью R в Hadoop. Но анализ можно выполнить быстрее, если хранить данные в более эффективном формате. Формат XDF-файла R эффективен, но немного изменен для HDFS таким образом, что отдельные файлы остаются в пределах одного блока HDFS. (Размер блока HDFS зависит от установки, но обычно составляет 64 или 128 МБ.) 

При использовании [rxImport](/machine-learning-server/r-reference/revoscaler/rximport) в Hadoop для создания набора составных XDF-файлов необходимо указать источник данных `RxTextData`, такой как `AirDS`, можно указать в качестве inData, а источник данных `RxXdfData` с параметром fileSystem, для которого задано значение файловой системы HDFS, — в качестве аргумента outFile. Объект `RxXdfData` можно затем использовать в качестве аргумента данных в последующем анализе R.

1. Определите объект `RxXdfData`. Введите приведенный ниже код в RStudio.

    ```R
    airDataXdfDir <- file.path(bigDataDirRoot,"AirOnTimeXDF2012")
    
    airDataXdf <- RxXdfData( airDataXdfDir,
                            fileSystem = hdfsFS )
    ```

1. Задайте для размера блока значение в 250 000 строк и укажите, что необходимо считывать все данные. Введите приведенный ниже код в RStudio.

    ```R
    blockSize <- 250000
    numRowsToRead = -1
    ```

1. Импортируйте данные с помощью `rxImport`. Введите приведенный ниже код в RStudio.

    ```R
    rxImport(inData = airDS,
             outFile = airDataXdf,
             rowsPerRead = blockSize,
             overwrite = TRUE,
             numRows = numRowsToRead )
    ```
    
    Выполнение этого шага должно занять несколько минут.

1. Переоцените такую же линейную модель, используя новый, более быстрый источник данных. Введите приведенный ниже код в RStudio.

    ```R
    system.time(
         delayArr <- rxLinMod(ArrDelay ~ DayOfWeek, data = airDataXdf,
              cube = TRUE)
    )
    ```
    
    Выполнение шага займет не больше минуты.

1. Просмотрите результаты. Результаты должны быть такими же, как в CSV-файлах. Введите приведенный ниже код в RStudio.

    ```R
    summary(delayArr)
    ```

## <a name="convert-xdf-to-csv"></a>Преобразование XDF-файлов в CSV-файлы

### <a name="in-a-spark-context"></a>В контексте Spark

Если же вы преобразовали CSV-файл в XDF-файл, чтобы получить эффективность при выполнении анализа, но теперь хотите преобразовать данные обратно в CSV-формат, это можно сделать с помощью [rxDataStep](/machine-learning-server/r-reference/revoscaler/rxdatastep).

Чтобы создать папку CSV-файлов, сначала создайте объект `RxTextData`, используя имя каталога в качестве аргумента файла. Этот объект представляет папку, в которой создаются CSV-файлы. Этот каталог создается при запуске `rxDataStep`. Затем укажите на этот объект `RxTextData` в аргументе `outFile``rxDataStep`. Каждому созданному CSV-файлу будет присвоено имя на основе имени каталога, за которым следует число.

Предположим, что после выполнения логистической регрессии и прогнозирования в системе HDFS из составного XDF-файла `airDataXdf` нужно записать папку CSV-файлов таким образом, чтобы новые CSV-файлы содержали прогнозируемые значения и остатки. Введите приведенный ниже код в RStudio.

```R
airDataCsvDir <- file.path(bigDataDirRoot,"AirDataCSV2012")
airDataCsvDS <- RxTextData(airDataCsvDir,fileSystem=hdfsFS)
rxDataStep(inData=airDataXdf, outFile=airDataCsvDS)
```

Выполнение этого шага должно занять около 2.5 минут.

`rxDataStep` записал один CSV-файл для каждого XDFD-файла во входном составном XDF-файле. Это поведение по умолчанию для записи CSV-файлов из составных XDF-файлов в файловую систему HDFS, если для контекста вычислений задано значение `RxSpark`.

### <a name="in-a-local-context"></a>В локальном контексте

Кроме того, контекст вычисления можно изменить обратно на `local` после выполнения анализа и использования преимуществ двух аргументов в `RxTextData`, обеспечивающих немного больший контроль при записи CSV-файлов в HDFS: `createFileSet` и `rowsPerOutFile`. Если для параметра `createFileSet` задано значение `TRUE`, папка CSV-файлов записывается в указанный каталог. Если для параметра `createFileSet` задано значение `FALSE`, записывается один CSV-файл. Для второго аргумента, `rowsPerOutFile`, можно задать целое число, чтобы указать, сколько строк необходимо записывать в каждый CSV-файл, когда для параметра `createFileSet` задано значение `TRUE`.

Введите приведенный ниже код в RStudio.

```R
rxSetComputeContext("local")
airDataCsvRowsDir <- file.path(bigDataDirRoot,"AirDataCSVRows2012")
airDataCsvRowsDS <- RxTextData(airDataCsvRowsDir, fileSystem=hdfsFS, createFileSet=TRUE, rowsPerOutFile=1000000)
rxDataStep(inData=airDataXdf, outFile=airDataCsvRowsDS)
```

Выполнение этого шага должно занять около 10 минут.

При использовании контекста вычислений `RxSpark` для параметра `createFileSet` по умолчанию используется значение `TRUE`, и параметр `rowsPerOutFile` не действует. Поэтому если вы хотите создать один CSV-файл или настроить количество строк в файле, выполните `rxDataStep` в контексте вычислений `local` (данные могут по-прежнему находиться в HDFS).

## <a name="final-steps"></a>Заключительные действия

1. Очистите данные. Введите приведенный ниже код в RStudio.

    ```R
    rxHadoopRemoveDir(airDataDir)
    rxHadoopRemoveDir(airDataXdfDir)
    rxHadoopRemoveDir(airDataCsvDir)
    rxHadoopRemoveDir(airDataCsvRowsDir)
    rxHadoopRemoveDir(bigDataDirRoot)
    ```

1. Остановите удаленное приложение Spark. Введите приведенный ниже код в RStudio.

    ```R
    rxStopEngine(mySparkCluster)
    ```

1. Закройте сеанс R. Введите приведенный ниже код в RStudio.

    ```R
    quit()
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этим руководством кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Поскольку стоимость кластера во много раз превышает стоимость хранилища, экономически целесообразно удалять неиспользуемые кластеры.

Инструкции по удалению кластера см. в статье [Delete an HDInsight cluster using your browser, PowerShell, or the Azure CLI](../hdinsight-delete-cluster.md) (Удаление кластера HDInsight с помощью браузера, PowerShell или Azure CLI).

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как использовать функции R в Apache Spark, запущенном в кластере служб машинного обучения в HDInsight. Дополнительные сведения см. в следующих статьях:

* [Compute context options for an Azure HDInsight Machine Learning services cluster](r-server-compute-contexts.md) (Параметры контекста вычислений для кластера служб машинного обучения Azure HDInsight)
* [RevoScaleR Functions for Spark on Hadoop](/machine-learning-server/r-reference/revoscaler/revoscaler-hadoop-functions) (Функции RevoScaleR для Spark в Hadoop)
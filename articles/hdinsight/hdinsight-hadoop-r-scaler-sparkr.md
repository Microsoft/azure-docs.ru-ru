---
title: Использование ScaleR и SparkR с Azure HDInsight
description: Использование Scale и Spark для обработки данных и разработки моделей с помощью служб ML в Azure HDInsight
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/26/2019
ms.openlocfilehash: 95fcca289b0776cc19464b13eb7d243ca4f8d5ed
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945504"
---
# <a name="combine-scaler-and-sparkr-in-hdinsight"></a>Совместное использование ScaleR и SparkR в HDInsight

В этой статье описано, как спрогнозировать задержки в прибытиях авиарейсов с помощью модели логистической регрессии **ScaleR**. В примере используются данные по задержкам авиарейсов и погоде, объединенные с помощью **SparkR**.

Несмотря на то, что оба пакета работают в подсистеме выполнения Spark Apache Hadoop, они блокируют обмен данными в памяти, так как каждый из них требует собственных соответствующих сеансов Spark. Это будет исправлено в последующих версиях ML Server, а пока временным решением является поддержание неперекрывающихся сеансов Spark и обмен данными с помощью промежуточных файлов. В приведенных в этой статье инструкциях показано, что обеспечить выполнение этих требований достаточно просто.

Этот пример изначально озвучили в докладе Марио Инчиоза (Mario Inchiosa) и Рони Берда (Roni Burd) на конференции Strata 2016 Дополнительные сведения см. в вебинаре [Building a Scalable Data Science Platform with R](https://channel9.msdn.com/blogs/Cloud-and-Enterprise-Premium/Building-A-Scalable-Data-Science-Platform-with-R-and-Hadoop) (Создание масштабируемой платформы анализа и обработки данных на основе R).

Этот код изначально был написан для ML Server под управлением Spark в кластере HDInsight в Azure. Однако понятие совместного использования SparkR и ScaleR в одном скрипте в равной степени применимо к локальным средам.

Здесь предполагается средний уровень знания языка R и библиотеки [ScaleR](/machine-learning-server/r/concept-what-is-revoscaler) для ML Server. Вы пойдете в [Spark](https://spark.apache.org/docs/2.1.0/sparkr.html) во время прохода по этому сценарию.

## <a name="the-airline-and-weather-datasets"></a>Наборы данных об авиакомпаниях и погоде

Данные о полетах взяты из [архива правительства США](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236). Он также доступен в виде ZIP-файла [AirOnTimeCSV.zip](https://packages.revolutionanalytics.com/datasets/AirOnTime87to12/AirOnTimeCSV.zip).

Данные о погоде можно скачать как ZIP-файлы в необработанном виде по месяцам из [репозитория Национального управления океанических и атмосферных исследований](https://www.ncdc.noaa.gov/orders/qclcd/). Для этого примера загрузите данные за период с мая 2007 г. по декабрь 2012 г. Используйте почасовые файлы данных и файл `YYYYMMMstation.txt` внутри каждого из ZIP-архивов.

## <a name="setting-up-the-spark-environment"></a>Настройка среды Spark

Чтобы настроить среду Spark, используйте следующий код:

```
workDir        <- '~'  
myNameNode     <- 'default' 
myPort         <- 0
inputDataDir   <- 'wasb://hdfs@myAzureAccount.blob.core.windows.net'
hdfsFS         <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

# create a persistent Spark session to reduce startup times 
#   (remember to stop it later!)
 
sparkCC        <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort, persistentRun=TRUE)

# create working directories 

rxHadoopMakeDir('/user')
rxHadoopMakeDir('user/RevoShare')
rxHadoopMakeDir('user/RevoShare/remoteuser')

(dataDir <- '/share')
rxHadoopMakeDir(dataDir)
rxHadoopListFiles(dataDir) 

setwd(workDir)
getwd()

# version of rxRoc that runs in a local CC 
rxRoc <- function(...){
  rxSetComputeContext(RxLocalSeq())
  roc <- RevoScaleR::rxRoc(...)
  rxSetComputeContext(sparkCC)
  return(roc)
}

logmsg <- function(msg) { cat(format(Sys.time(), "%Y-%m-%d %H:%M:%S"),':',msg,'\n') } 
t0 <- proc.time() 

#..start 

logmsg('Start') 
(trackers <- system("mapred job -list-active-trackers", intern = TRUE))
logmsg(paste('Number of task nodes=',length(trackers)))
```

Далее добавьте `Spark_Home` к пути поиска пакетов R. Это позволит использовать SparkR и инициализировать сеанс SparkR:

```
#..setup for use of SparkR  

logmsg('Initialize SparkR') 

.libPaths(c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib"), .libPaths()))
library(SparkR)

sparkEnvir <- list(spark.executor.instances = '10',
                   spark.yarn.executor.memoryOverhead = '8000')

sc <- sparkR.init(
  sparkEnvir = sparkEnvir,
  sparkPackages = "com.databricks:spark-csv_2.10:1.3.0"
)

sqlContext <- sparkRSQL.init(sc)
```

## <a name="preparing-the-weather-data"></a>Подготовка данных о погоде

Чтобы подготовить данные по погоде, подставьте их в столбцы, необходимые для моделирования: 

- "Visibility"
- "DryBulbCelsius"
- "DewPointCelsius"
- "RelativeHumidity"
- "WindSpeed"
- "Altimeter"

Затем добавьте код аэропорта, связанного с метеостанцией, и преобразуйте значения местного времени в формат UTC. 

Начните с создания файла для сопоставления сведений о метеостанции (WBAN) с кодом аэропорта. Следующий код считывает каждый час необработанных файлов данных погоды, подмножества в нужных столбцах, объединяет файл сопоставления станции погоды, корректирует дату и время измерения до времени в формате UTC, а затем записывает новую версию файла:

```
# Look up AirportID and Timezone for WBAN (weather station ID) and adjust time

adjustTime <- function(dataList)
{
  dataset0 <- as.data.frame(dataList)
  
  dataset1 <- base::merge(dataset0, wbanToAirIDAndTZDF1, by = "WBAN")

  if(nrow(dataset1) == 0) {
    dataset1 <- data.frame(
      Visibility = numeric(0),
      DryBulbCelsius = numeric(0),
      DewPointCelsius = numeric(0),
      RelativeHumidity = numeric(0),
      WindSpeed = numeric(0),
      Altimeter = numeric(0),
      AdjustedYear = numeric(0),
      AdjustedMonth = numeric(0),
      AdjustedDay = integer(0),
      AdjustedHour = integer(0),
      AirportID = integer(0)
    )
    
    return(dataset1)
  }
  
  Year <- as.integer(substr(dataset1$Date, 1, 4))
  Month <- as.integer(substr(dataset1$Date, 5, 6))
  Day <- as.integer(substr(dataset1$Date, 7, 8))
  
  Time <- dataset1$Time
  Hour <- ceiling(Time/100)
  
  Timezone <- as.integer(dataset1$TimeZone)
  
  adjustdate = as.POSIXlt(sprintf("%4d-%02d-%02d %02d:00:00", Year, Month, Day, Hour), tz = "UTC") + Timezone * 3600

  AdjustedYear = as.POSIXlt(adjustdate)$year + 1900
  AdjustedMonth = as.POSIXlt(adjustdate)$mon + 1
  AdjustedDay   = as.POSIXlt(adjustdate)$mday
  AdjustedHour  = as.POSIXlt(adjustdate)$hour
  
  AirportID = dataset1$AirportID
  Weather = dataset1[,c("Visibility", "DryBulbCelsius", "DewPointCelsius", "RelativeHumidity", "WindSpeed", "Altimeter")]
  
  data.set = data.frame(cbind(AdjustedYear, AdjustedMonth, AdjustedDay, AdjustedHour, AirportID, Weather))
  
  return(data.set)
}

wbanToAirIDAndTZDF <- read.csv("wban-to-airport-id-tz.csv")

colInfo <- list(
  WBAN = list(type="integer"),
  Date = list(type="character"),
  Time = list(type="integer"),
  Visibility = list(type="numeric"),
  DryBulbCelsius = list(type="numeric"),
  DewPointCelsius = list(type="numeric"),
  RelativeHumidity = list(type="numeric"),
  WindSpeed = list(type="numeric"),
  Altimeter = list(type="numeric")
)

weatherDF <- RxTextData(file.path(inputDataDir, "WeatherRaw"), colInfo = colInfo)

weatherDF1 <- RxTextData(file.path(inputDataDir, "Weather"), colInfo = colInfo,
                filesystem=hdfsFS)

rxSetComputeContext("localpar")
rxDataStep(weatherDF, outFile = weatherDF1, rowsPerRead = 50000, overwrite = T,
           transformFunc = adjustTime,
           transformObjects = list(wbanToAirIDAndTZDF1 = wbanToAirIDAndTZDF))
```

## <a name="importing-the-airline-and-weather-data-to-spark-dataframes"></a>Импорт данных авиакомпаний и погодных данных в таблицы данных Spark

Теперь мы используем функцию SparkR [read.df()](https://spark.apache.org/docs/latest/api/R/read.df.html) для импорта данных по погоде и авиакомпаниям в кадры данных Spark. Эта функция, как и многие другие методы Spark, выполняется ленивым образом, то есть они ставятся в очередь на выполнение, но не выполняются до необходимости.

```
airPath     <- file.path(inputDataDir, "AirOnTime08to12CSV")
weatherPath <- file.path(inputDataDir, "Weather") # pre-processed weather data
rxHadoopListFiles(airPath) 
rxHadoopListFiles(weatherPath) 

# create a SparkR DataFrame for the airline data

logmsg('create a SparkR DataFrame for the airline data') 
# use inferSchema = "false" for more robust parsing
airDF <- read.df(sqlContext, airPath, source = "com.databricks.spark.csv", 
                 header = "true", inferSchema = "false")

# Create a SparkR DataFrame for the weather data

logmsg('create a SparkR DataFrame for the weather data') 
weatherDF <- read.df(sqlContext, weatherPath, source = "com.databricks.spark.csv", 
                     header = "true", inferSchema = "true")
```

## <a name="data-cleansing-and-transformation"></a>Фильтрация и преобразование данных

Далее мы выполним очистку данных авиакомпании, которые мы импортировали для переименования столбцов. Мы оставим только необходимые переменные и округлим время запланированного отправления до ближайшего часа, чтобы объединить с последними данными по погоде перед вылетом.

```
logmsg('clean the airline data') 
airDF <- rename(airDF,
                ArrDel15 = airDF$ARR_DEL15,
                Year = airDF$YEAR,
                Month = airDF$MONTH,
                DayofMonth = airDF$DAY_OF_MONTH,
                DayOfWeek = airDF$DAY_OF_WEEK,
                Carrier = airDF$UNIQUE_CARRIER,
                OriginAirportID = airDF$ORIGIN_AIRPORT_ID,
                DestAirportID = airDF$DEST_AIRPORT_ID,
                CRSDepTime = airDF$CRS_DEP_TIME,
                CRSArrTime =  airDF$CRS_ARR_TIME
)

# Select desired columns from the flight data. 
varsToKeep <- c("ArrDel15", "Year", "Month", "DayofMonth", "DayOfWeek", "Carrier", "OriginAirportID", "DestAirportID", "CRSDepTime", "CRSArrTime")
airDF <- select(airDF, varsToKeep)

# Apply schema
coltypes(airDF) <- c("character", "integer", "integer", "integer", "integer", "character", "integer", "integer", "integer", "integer")

# Round down scheduled departure time to full hour.
airDF$CRSDepTime <- floor(airDF$CRSDepTime / 100)
```

Теперь выполним аналогичные операции с данными по погоде:

```
# Average weather readings by hour
logmsg('clean the weather data') 
weatherDF <- agg(groupBy(weatherDF, "AdjustedYear", "AdjustedMonth", "AdjustedDay", "AdjustedHour", "AirportID"), Visibility="avg",
                  DryBulbCelsius="avg", DewPointCelsius="avg", RelativeHumidity="avg", WindSpeed="avg", Altimeter="avg"
                  )

weatherDF <- rename(weatherDF,
                    Visibility = weatherDF$'avg(Visibility)',
                    DryBulbCelsius = weatherDF$'avg(DryBulbCelsius)',
                    DewPointCelsius = weatherDF$'avg(DewPointCelsius)',
                    RelativeHumidity = weatherDF$'avg(RelativeHumidity)',
                    WindSpeed = weatherDF$'avg(WindSpeed)',
                    Altimeter = weatherDF$'avg(Altimeter)'
)
```

## <a name="joining-the-weather-and-airline-data"></a>Объединение данных о погоде с данными об авиакомпаниях

Теперь воспользуемся функцией SparkR [join()](https://spark.apache.org/docs/latest/api/R/join.html), чтобы выполнить левое внешнее соединение данных по погоде с данными по авиакомпаниям на основе идентификатора аэропорта отправления (AirportID) и значения даты и времени (datetime). Внешнее соединение позволяет нам хранить все записи данных авиакомпании, даже если нет соответствующих данных о погоде. После соединения удалим некоторые избыточные столбцы и переименуем оставшиеся, чтобы удалить входящий префикс кадра данных, возникший при соединении.

```
logmsg('Join airline data with weather at Origin Airport')
joinedDF <- SparkR::join(
  airDF,
  weatherDF,
  airDF$OriginAirportID == weatherDF$AirportID &
    airDF$Year == weatherDF$AdjustedYear &
    airDF$Month == weatherDF$AdjustedMonth &
    airDF$DayofMonth == weatherDF$AdjustedDay &
    airDF$CRSDepTime == weatherDF$AdjustedHour,
  joinType = "left_outer"
)

# Remove redundant columns
vars <- names(joinedDF)
varsToDrop <- c('AdjustedYear', 'AdjustedMonth', 'AdjustedDay', 'AdjustedHour', 'AirportID')
varsToKeep <- vars[!(vars %in% varsToDrop)]
joinedDF1 <- select(joinedDF, varsToKeep)

joinedDF2 <- rename(joinedDF1,
                    VisibilityOrigin = joinedDF1$Visibility,
                    DryBulbCelsiusOrigin = joinedDF1$DryBulbCelsius,
                    DewPointCelsiusOrigin = joinedDF1$DewPointCelsius,
                    RelativeHumidityOrigin = joinedDF1$RelativeHumidity,
                    WindSpeedOrigin = joinedDF1$WindSpeed,
                    AltimeterOrigin = joinedDF1$Altimeter
)
```

Аналогичным образом объединим данные по погоде и авиакомпаниям на основе идентификатора аэропорта прибытия (AirportID) и значения даты и времени (datetime).

```
logmsg('Join airline data with weather at Destination Airport')
joinedDF3 <- SparkR::join(
  joinedDF2,
  weatherDF,
  airDF$DestAirportID == weatherDF$AirportID &
    airDF$Year == weatherDF$AdjustedYear &
    airDF$Month == weatherDF$AdjustedMonth &
    airDF$DayofMonth == weatherDF$AdjustedDay &
    airDF$CRSDepTime == weatherDF$AdjustedHour,
  joinType = "left_outer"
)

# Remove redundant columns
vars <- names(joinedDF3)
varsToDrop <- c('AdjustedYear', 'AdjustedMonth', 'AdjustedDay', 'AdjustedHour', 'AirportID')
varsToKeep <- vars[!(vars %in% varsToDrop)]
joinedDF4 <- select(joinedDF3, varsToKeep)

joinedDF5 <- rename(joinedDF4,
                    VisibilityDest = joinedDF4$Visibility,
                    DryBulbCelsiusDest = joinedDF4$DryBulbCelsius,
                    DewPointCelsiusDest = joinedDF4$DewPointCelsius,
                    RelativeHumidityDest = joinedDF4$RelativeHumidity,
                    WindSpeedDest = joinedDF4$WindSpeed,
                    AltimeterDest = joinedDF4$Altimeter
                    )
```

## <a name="save-results-to-csv-for-exchange-with-scaler"></a>Сохранение результатов в CSV-файл для обмена со ScaleR

На этом операции соединения с помощью SparkR завершены. Сохраним данные из окончательного кадра данных Spark joinedDF5 в CSV-файл в качестве входных данных для SparkR и завершим сеанс SparkR. Мы явно указываем SparkR сохранить итоговый CSV-файл в 80 отдельных секциях, чтобы обеспечить достаточный параллелизм при обработке в ScaleR.

```
logmsg('output the joined data from Spark to CSV') 
joinedDF5 <- repartition(joinedDF5, 80) # write.df below will produce this many CSVs

# write result to directory of CSVs
write.df(joinedDF5, file.path(dataDir, "joined5Csv"), "com.databricks.spark.csv", "overwrite", header = "true")

# We can shut down the SparkR Spark context now
sparkR.stop()

# remove non-data files
rxHadoopRemove(file.path(dataDir, "joined5Csv/_SUCCESS"))
```

## <a name="import-to-xdf-for-use-by-scaler"></a>Импорт в XDF для использования в ScaleR

Мы могли бы использовать CSV-файл с объединенными данными по авиакомпаниям и погоде "как есть", чтобы создать модель с помощью текстового источника данных ScaleR. Но сначала импортируйте его в Xdf-, так как он более эффективен при выполнении нескольких операций с набором данных:

```
logmsg('Import the CSV to compressed, binary XDF format') 

# set the Spark compute context for ML Services 
rxSetComputeContext(sparkCC)
rxGetComputeContext()

colInfo <- list(
  ArrDel15 = list(type="numeric"),
  Year = list(type="factor"),
  Month = list(type="factor"),
  DayofMonth = list(type="factor"),
  DayOfWeek = list(type="factor"),
  Carrier = list(type="factor"),
  OriginAirportID = list(type="factor"),
  DestAirportID = list(type="factor"),
  RelativeHumidityOrigin = list(type="numeric"),
  AltimeterOrigin = list(type="numeric"),
  DryBulbCelsiusOrigin = list(type="numeric"),
  WindSpeedOrigin = list(type="numeric"),
  VisibilityOrigin = list(type="numeric"),
  DewPointCelsiusOrigin = list(type="numeric"),
  RelativeHumidityDest = list(type="numeric"),
  AltimeterDest = list(type="numeric"),
  DryBulbCelsiusDest = list(type="numeric"),
  WindSpeedDest = list(type="numeric"),
  VisibilityDest = list(type="numeric"),
  DewPointCelsiusDest = list(type="numeric")
)

joinedDF5Txt <- RxTextData(file.path(dataDir, "joined5Csv"),
                           colInfo = colInfo, fileSystem = hdfsFS)
rxGetInfo(joinedDF5Txt) 

destData <- RxXdfData(file.path(dataDir, "joined5XDF"), fileSystem = hdfsFS)

rxImport(inData = joinedDF5Txt, destData, overwrite = TRUE)

rxGetInfo(destData, getVarInfo = T)

# File name: /user/RevoShare/dev/delayDataLarge/joined5XDF 
# Number of composite data files: 80 
# Number of observations: 148619655 
# Number of variables: 22 
# Number of blocks: 320 
# Compression type: zlib 
# Variable information: 
#   Var 1: ArrDel15, Type: numeric, Low/High: (0.0000, 1.0000)
# Var 2: Year
# 26 factor levels: 1987 1988 1989 1990 1991 ... 2008 2009 2010 2011 2012
# Var 3: Month
# 12 factor levels: 10 11 12 1 2 ... 5 6 7 8 9
# Var 4: DayofMonth
# 31 factor levels: 1 3 4 5 7 ... 29 30 2 18 31
# Var 5: DayOfWeek
# 7 factor levels: 4 6 7 1 3 2 5
# Var 6: Carrier
# 30 factor levels: PI UA US AA DL ... HA F9 YV 9E VX
# Var 7: OriginAirportID
# 374 factor levels: 15249 12264 11042 15412 13930 ... 13341 10559 14314 11711 10558
# Var 8: DestAirportID
# 378 factor levels: 13303 14492 10721 11057 13198 ... 14802 11711 11931 12899 10559
# Var 9: CRSDepTime, Type: integer, Low/High: (0, 24)
# Var 10: CRSArrTime, Type: integer, Low/High: (0, 2400)
# Var 11: RelativeHumidityOrigin, Type: numeric, Low/High: (0.0000, 100.0000)
# Var 12: AltimeterOrigin, Type: numeric, Low/High: (28.1700, 31.1600)
# Var 13: DryBulbCelsiusOrigin, Type: numeric, Low/High: (-46.1000, 47.8000)
# Var 14: WindSpeedOrigin, Type: numeric, Low/High: (0.0000, 81.0000)
# Var 15: VisibilityOrigin, Type: numeric, Low/High: (0.0000, 90.0000)
# Var 16: DewPointCelsiusOrigin, Type: numeric, Low/High: (-41.7000, 29.0000)
# Var 17: RelativeHumidityDest, Type: numeric, Low/High: (0.0000, 100.0000)
# Var 18: AltimeterDest, Type: numeric, Low/High: (28.1700, 31.1600)
# Var 19: DryBulbCelsiusDest, Type: numeric, Low/High: (-46.1000, 53.9000)
# Var 20: WindSpeedDest, Type: numeric, Low/High: (0.0000, 136.0000)
# Var 21: VisibilityDest, Type: numeric, Low/High: (0.0000, 88.0000)
# Var 22: DewPointCelsiusDest, Type: numeric, Low/High: (-43.0000, 29.0000)

finalData <- RxXdfData(file.path(dataDir, "joined5XDF"), fileSystem = hdfsFS)

```

## <a name="splitting-data-for-training-and-test"></a>Разделение данных на наборы для обучения и тестирования

Воспользуемся rxDataStep для отделения данных за 2012 год в целях тестирования, оставив остальные данные для обучения.

```
# split out the training data

logmsg('split out training data as all data except year 2012')
trainDS <- RxXdfData( file.path(dataDir, "finalDataTrain" ),fileSystem = hdfsFS)

rxDataStep( inData = finalData, outFile = trainDS,
            rowSelection = ( Year != 2012 ), overwrite = T )

# split out the testing data

logmsg('split out the test data for year 2012') 
testDS <- RxXdfData( file.path(dataDir, "finalDataTest" ), fileSystem = hdfsFS)

rxDataStep( inData = finalData, outFile = testDS,
            rowSelection = ( Year == 2012 ), overwrite = T )

rxGetInfo(trainDS)
rxGetInfo(testDS)
```

## <a name="train-and-test-a-logistic-regression-model"></a>Обучение и тестирование модели логистической регрессии

Теперь мы готовы к созданию модели. Чтобы увидеть влияние данных о погоде на задержку во время прибытия, мы используем подпрограммы для логистической регрессии. С ее помощью мы смоделируем, зависит ли задержка прибытия более чем на 15 минут от погоды в аэропортах отправления и прибытия.

```
logmsg('train a logistic regression model for Arrival Delay > 15 minutes') 
formula <- as.formula(ArrDel15 ~ Year + Month + DayofMonth + DayOfWeek + Carrier +
                     OriginAirportID + DestAirportID + CRSDepTime + CRSArrTime + 
                     RelativeHumidityOrigin + AltimeterOrigin + DryBulbCelsiusOrigin +
                     WindSpeedOrigin + VisibilityOrigin + DewPointCelsiusOrigin + 
                     RelativeHumidityDest + AltimeterDest + DryBulbCelsiusDest +
                     WindSpeedDest + VisibilityDest + DewPointCelsiusDest
                   )

# Use the scalable rxLogit() function but set max iterations to 3 for the purposes of 
# this exercise 

logitModel <- rxLogit(formula, data = trainDS, maxIterations = 3)

base::summary(logitModel)
```

Теперь давайте посмотрим, как это делается на тестовых данных, сделав некоторые прогнозы и взглянув на ROC и AUC.

```
# Predict over test data (Logistic Regression).

logmsg('predict over the test data') 
logitPredict <- RxXdfData(file.path(dataDir, "logitPredict"), fileSystem = hdfsFS)

# Use the scalable rxPredict() function

rxPredict(logitModel, data = testDS, outData = logitPredict,
          extraVarsToWrite = c("ArrDel15"), 
          type = 'response', overwrite = TRUE)

# Calculate ROC and Area Under the Curve (AUC).

logmsg('calculate the roc and auc') 
logitRoc <- rxRoc("ArrDel15", "ArrDel15_Pred", logitPredict)
logitAuc <- rxAuc(logitRoc)
head(logitAuc)
logitAuc

plot(logitRoc)
```

## <a name="scoring-elsewhere"></a>Альтернативное применение модели

Эту модель можно также использовать для оценки данных на другой платформе Путем сохранения в файл RDS, а затем передачи и импорта этой службы RDS в целевую среду оценки, например Microsoft SQL Server R Services. Важно убедиться, что уровни коэффициента данных, которые должны оцениваться, соответствуют показателям, на которых была построена модель. Это можно сделать, извлекая и сохранив сведения о столбцах, связанных с данными моделирования, с помощью функции средства масштабирования `rxCreateColInfo()` , а затем применяя эти сведения о столбцах к источнику входных данных для прогноза. В следующем примере кода мы сохраняем несколько строк тестового набора данных и извлекаем и используем сведения о столбцах из этого примера в скрипте прогнозирования:

```
# save the model and a sample of the test dataset 

logmsg('save serialized version of the model and a sample of the test data')
rxSetComputeContext('localpar') 
saveRDS(logitModel, file = "logitModel.rds")
testDF <- head(testDS, 1000)  
saveRDS(testDF    , file = "testDF.rds"    )
list.files()

rxHadoopListFiles(file.path(inputDataDir,''))
rxHadoopListFiles(dataDir)

# stop the spark engine 
rxStopEngine(sparkCC) 

logmsg('Done.')
elapsed <- (proc.time() - t0)[3]
logmsg(paste('Elapsed time=',sprintf('%6.2f',elapsed),'(sec)\n\n'))
```

## <a name="summary"></a>Итоги

В этой статье мы рассмотрели, как можно сочетать использование Spark для манипулирования данными с помощью масштабируемости для разработки моделей в Hadoop Spark. Этот сценарий требует создания нескольких сеансов Spark, использования одновременно только одного сеанса и обмена данными посредством CSV-файлов. При всей своей простоте этот процесс станет еще проще в предстоящем выпуске Служб машинного обучения, в котором SparkR и ScaleR смогут совместно использовать сеансы и кадры данных Spark.

## <a name="next-steps-and-more-information"></a>Дальнейшие действия и дополнительные сведения

- Дополнительные сведения об использовании ML Server в Apache Spark см. в разделе [руководство по началу работы](/machine-learning-server/r/how-to-revoscaler-spark).

- Сведения о службах ML в HDInsight см. в статье [Обзор служб ML в hdinsight](r-server/r-server-overview.md).

Дополнительные сведения об использовании SparkR см. в следующих статьях:

- [Документ Apache Spark](https://spark.apache.org/docs/2.1.0/sparkr.html).

- [Обзор Spark](/azure/databricks/spark/latest/sparkr/overview)

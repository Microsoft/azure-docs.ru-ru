---
title: 'Выполнение скрипта R: Справочник по модулям'
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать модуль «выполнение скрипта R» в конструкторе Машинное обучение Azure для выполнения пользовательского кода R.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 12/17/2020
ms.openlocfilehash: bdd7fd8e19bf2de6d0b3c6b2edd4515771fae237
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98119031"
---
# <a name="execute-r-script-module"></a>Выполнить модуль R Script

В этой статье описывается, как использовать модуль «выполнение скрипта R» для выполнения кода R в конвейере конструктора Машинное обучение Azure.

С помощью R можно выполнять задачи, которые не поддерживаются существующими модулями, например: 
- создание настраиваемых преобразований данных;
- Использование собственных метрик для вычисления прогнозов
- Создание моделей с использованием алгоритмов, которые не реализованы как автономные модули в конструкторе

## <a name="r-version-support"></a>Поддержка версии R

Машинное обучение Azure конструктор использует распределение R по CRAN (всеобъемлющую сеть с архивом R). Текущая используемая версия — CRAN 3.5.1.

## <a name="supported-r-packages"></a>Поддерживаемые пакеты R

Среда R предустановлена с более чем 100 пакетами. Полный список см. в разделе с [предварительно установленными пакетами R](#preinstalled-r-packages).

Можно также добавить следующий код в любой модуль выполнить сценарий R, чтобы просмотреть установленные пакеты.

```R
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  dataframe1 <- data.frame(installed.packages())
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```
> [!NOTE]
> Если ваш конвейер содержит несколько модулей выполнения сценариев R, для которых требуются пакеты, отсутствующие в предварительно установленном списке, установите пакеты в каждом модуле. 

## <a name="installing-r-packages"></a>Установка пакетов R
Чтобы установить дополнительные пакеты R, используйте `install.packages()` метод. Пакеты устанавливаются для каждого модуля выполнить сценарий R. Они не являются общими для других модулей «выполнение сценария R».

> [!NOTE]
> **Не** рекомендуется устанавливать пакет R из пакета сценариев. Рекомендуется устанавливать пакеты непосредственно в редакторе скриптов.
> При установке пакетов укажите репозиторий CRAN, например `install.packages("zoo",repos = "http://cran.us.r-project.org")` .

> [!WARNING]
> Модуль скрипта R екскуте не поддерживает установку пакетов, для которых требуется компиляция в собственном коде, например `qdap` пакет, требующий Java и Package, для `drc` которых требуется C++. Это происходит потому, что этот модуль выполняется в предварительно установленной среде с разрешениями, не являющимися администраторами.
> Не устанавливайте пакеты, предварительно созданные на платформе или для Windows, так как модули конструктора работают на Ubuntu. Чтобы проверить, является ли пакет предварительно созданным в Windows, можно зайти на [Cran](https://cran.r-project.org/) и выполнить поиск по пакету, скачать один двоичный файл в соответствии с вашей операционной системой, а затем проверить **сборку:** Part в файле **описания** . Ниже приведен пример: :::image type="content" source="media/module/r-package-description.png" alt-text="Описание пакета R" lightbox="media/module/r-package-page.png"::: .

В этом примере показано, как установить Zoo:
```R
# R version: 3.5.1
# The script MUST contain a function named azureml_main,
# which is the entry point for this module.

# Note that functions dependent on the X11 library,
# such as "View," are not supported because the X11 library
# is not preinstalled.

# The entry point function MUST have two input arguments.
# If the input port is not connected, the corresponding
# dataframe argument will be null.
#   Param<dataframe1>: a R DataFrame
#   Param<dataframe2>: a R DataFrame
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  
  if(!require(zoo)) install.packages("zoo",repos = "http://cran.us.r-project.org")
  library(zoo)
  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```
 > [!NOTE]
 > Перед установкой пакета проверьте, существует ли он уже, чтобы не повторять установку. Повторные установки могут привести к истечению времени ожидания запросов веб-службы.     

## <a name="access-to-registered-dataset"></a>Доступ к зарегистрированному набору данных

Чтобы получить доступ к [зарегистрированным наборам данных](../how-to-create-register-datasets.md) в рабочей области, можно обратиться к следующему примеру кода:

```R
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  run = get_current_run()
  ws = run$experiment$workspace
  dataset = azureml$core$dataset$Dataset$get_by_name(ws, "YOUR DATASET NAME")
  dataframe2 <- dataset$to_pandas_dataframe()
  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```

## <a name="uploading-files"></a>Отправка файлов
Модуль выполнить сценарий R поддерживает передачу файлов с помощью пакета SDK для Машинное обучение Azure R.

В следующем примере показано, как передать файл изображения в инструкцию EXECUTE R Script:
```R
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")

  # Generate a jpeg graph
  img_file_name <- "rect.jpg"
  jpeg(file=img_file_name)
  example(rect)
  dev.off()

  upload_files_to_run(names = list(file.path("graphic", img_file_name)), paths=list(img_file_name))


  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
```

После завершения выполнения конвейера можно просмотреть изображение на правой панели модуля.

> [!div class="mx-imgBorder"]
> ![Предварительный просмотр отправленного изображения](media/module/upload-image-in-r-script.png)

## <a name="how-to-configure-execute-r-script"></a>Настройка выполнения сценария R

Модуль выполнить сценарий R содержит пример кода в качестве отправной точки.

![Схема входных данных для модуля R](media/module/execute-r-script.png)

Наборы данных, хранящиеся в конструкторе, автоматически преобразуются в кадр с данными R при загрузке с этим модулем.

1.  Добавьте модуль **выполнить сценарий R** в конвейер.  

1. Подключите все входные данные, необходимые для сценария. Входные значения необязательны и могут включать данные и дополнительный код R.

    * **DataSet1**: ссылка на первый вход в качестве `dataframe1` . Входной набор данных должен быть отформатирован в виде файла CSV, TSV или ARFF. Или можно подключить набор данных Машинное обучение Azure.

    * **DataSet2**: ссылка на второй вход в качестве `dataframe2` . Этот набор данных также должен быть отформатирован в формате CSV, TSV или ARFF или в виде набора данных Машинное обучение Azure.

    * **Пакет скриптов**. третий вход принимает ZIP-файлы. ZIP-файл может содержать несколько файлов и несколько типов файлов.

1. В текстовом поле **Скрипт r** введите или вставьте допустимый скрипт R.

    > [!NOTE]
    > Будьте внимательны при написании сценария. Убедитесь в отсутствии синтаксических ошибок, таких как использование необъявленных переменных или неимпортированных модулей или функций. Обратите особое внимание на список предварительно установленных пакетов в конце этой статьи. Чтобы использовать пакеты, которых нет в списке, установите их в скрипт. Например, `install.packages("zoo",repos = "http://cran.us.r-project.org")`.
    
    Чтобы помочь вам приступить к работе, в текстовом поле « **Скрипт R** » указывается образец кода, который можно изменить или заменить.
    
    ```R
    # R version: 3.5.1
    # The script MUST contain a function named azureml_main,
    # which is the entry point for this module.

    # Note that functions dependent on the X11 library,
    # such as "View," are not supported because the X11 library
    # is not preinstalled.
    
    # The entry point function MUST have two input arguments.
    # If the input port is not connected, the corresponding
    # dataframe argument will be null.
    #   Param<dataframe1>: a R DataFrame
    #   Param<dataframe2>: a R DataFrame
    azureml_main <- function(dataframe1, dataframe2){
    print("R script run.")

    # If a .zip file is connected to the third input port, it's
    # unzipped under "./Script Bundle". This directory is added
    # to sys.path.

    # Return datasets as a Named List
    return(list(dataset1=dataframe1, dataset2=dataframe2))
    }
    ```

    Функция точки входа должна иметь входные аргументы `Param<dataframe1>` и `Param<dataframe2>` , даже если эти аргументы не используются в функции.

    > [!NOTE]
    > Данные, передаваемые в модуль выполнить сценарий R, указываются как `dataframe1` и `dataframe2` , что отличается от машинное обучение Azure конструктора (ссылка конструктора как `dataset1` , `dataset2` ). Убедитесь, что в скрипте указаны правильные ссылки на входные данные.  
 
    > [!NOTE]
    > Существующий код R может потребовать незначительных изменений для выполнения в конвейере конструктора. Например, входные данные, которые вы предоставляете в формате CSV, должны быть явно преобразованы в набор данных, прежде чем их можно будет использовать в коде. Типы данных и столбцов, используемые в языке R, также отличаются некоторыми способами от типов данных и столбцов, используемых в конструкторе.

1. Если размер скрипта превышает 16 КБ, используйте порт **пакета сценариев** , чтобы избежать ошибок, таких как *Командная строка, превышает ограничение в 16597 символов*. 
    
    1. Упакуйте скрипт и другие настраиваемые ресурсы в ZIP-файл.
    1. Отправьте ZIP-файл в качестве **файлового набора данных** в студию. 
    1. Перетащите модуль DataSet из списка *наборы* данных на панели слева модуль на странице Создание конструктора. 
    1. Подключите модуль набора данных к порту **пакета скрипта** модуля **выполнение скрипта R** .
    
    Ниже приведен пример кода для использования скрипта в пакете скрипта:

    ```R
    azureml_main <- function(dataframe1, dataframe2){
    # Source the custom R script: my_script.R
    source("./Script Bundle/my_script.R")

    # Use the function that defined in my_script.R
    dataframe1 <- my_func(dataframe1)

    sample <- readLines("./Script Bundle/my_sample.txt")
    return (list(dataset1=dataframe1, dataset2=data.frame("Sample"=sample)))
    }
    ```

1.  Для параметра **случайное начальное** значение введите значение, которое будет использоваться в среде R в качестве значения случайного начального номера. Действие этого параметра эквивалентно вызову `set.seed(value)` в коде R.  

1. Отправьте конвейер.  

## <a name="results"></a>Результаты

Модуль выполнения R Script modules может возвращать несколько выходов, но они должны быть представлены в виде кадров данных R. Конструктор автоматически преобразует данные в наборы данных для обеспечения совместимости с другими модулями.

Стандартные сообщения и ошибки из R возвращаются в журнал модуля.

Если необходимо распечатать результаты в скрипте R, результаты печати можно найти на **70_driver_log** на вкладке **выходные данные и журналы** на правой панели окна модуля.

## <a name="sample-scripts"></a>Примеры сценариев

Существует множество способов расширения конвейера с помощью пользовательских скриптов R. В этом разделе приведен пример кода для распространенных задач.


### <a name="add-an-r-script-as-an-input"></a>Добавление скрипта R в качестве входных данных

Модуль выполнить сценарий R поддерживает произвольные файлы R script в качестве входных данных. Чтобы использовать их, необходимо передать их в рабочую область как часть ZIP-файла.

1. Чтобы отправить zip-файл, содержащий код R, в рабочую область, перейдите на страницу ресурсов **наборы данных** . Выберите **создать набор данных**, а затем выберите **из локального файла** и тип **файлового** набора данных.  

1. Убедитесь, что сжатый файл отображается в разделе **Мои наборы** данных в категории **наборы данных** в левом дереве модулей.

1.  Подключите набор данных к порту ввода **пакета сценариев** .

1. Все файлы в ZIP-файле доступны во время выполнения конвейера. 

    Если файл пакета скрипта содержит структуру каталогов, структура сохраняется. Но необходимо изменить код так, чтобы в нем был добавлен путь к каталогу Directory **./script** .

### <a name="process-data"></a>Обработка данных

В следующем примере показано, как масштабировать и нормализовать входные данные.

```R
# R version: 3.5.1
# The script MUST contain a function named azureml_main,
# which is the entry point for this module.

# Note that functions dependent on the X11 library,
# such as "View," are not supported because the X11 library
# is not preinstalled.

# The entry point function MUST have two input arguments.
# If the input port is not connected, the corresponding
# dataframe argument will be null.
#   Param<dataframe1>: a R DataFrame
#   Param<dataframe2>: a R DataFrame
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  # If a .zip file is connected to the third input port, it's
  # unzipped under "./Script Bundle". This directory is added
  # to sys.path.
  series <- dataframe1$width
  # Find the maximum and minimum values of the width column in dataframe1
  max_v <- max(series)
  min_v <- min(series)
  # Calculate the scale and bias
  scale <- max_v - min_v
  bias <- min_v / dis
  # Apply min-max normalizing
  dataframe1$width <- dataframe1$width / scale - bias
  dataframe2$width <- dataframe2$width / scale - bias
  # Return datasets as a Named List
  return(list(dataset1=dataframe1, dataset2=dataframe2))
}
 ```

### <a name="read-a-zip-file-as-input"></a>Чтение ZIP-файла в качестве входных данных

В этом примере показано, как использовать набор данных в ZIP-файле в качестве входных данных для модуля «выполнение скрипта R».

1. Создайте файл данных в формате CSV и назовите его **mydatafile.csv**.
1. Создайте ZIP-файл и добавьте CSV-файл в архив.
1. Отправьте сжатый ZIP-файл в рабочую область Машинное обучение Azure. 
1. Подключите результирующий набор данных к **скриптбундле** входному модулю **выполнения скрипта R** .
1. Используйте следующий код для чтения CSV-данных из ZIP-файла.

```R
azureml_main <- function(dataframe1, dataframe2){
  print("R script run.")
  mydataset<-read.csv("./Script Bundle/mydatafile.csv",encoding="UTF-8");  
  # Return datasets as a Named List
  return(list(dataset1=mydataset, dataset2=dataframe2))
}
```

### <a name="replicate-rows"></a>Репликация строк

В этом примере показано, как реплицировать положительные записи в наборе данных для балансировки образца:

```R
azureml_main <- function(dataframe1, dataframe2){
  data.set <- dataframe1[dataframe1[,1]==-1,]  
  # positions of the positive samples
  pos <- dataframe1[dataframe1[,1]==1,]
  # replicate the positive samples to balance the sample  
  for (i in 1:20) data.set <- rbind(data.set,pos)  
  row.names(data.set) <- NULL
  # Return datasets as a Named List
  return(list(dataset1=data.set, dataset2=dataframe2))
}
```

### <a name="pass-r-objects-between-execute-r-script-modules"></a>Передача объектов R между модулями выполнения скрипта R

Вы можете передавать объекты R между экземплярами модуля Выполнение скрипта R с помощью внутреннего механизма сериализации. В этом примере предполагается, что необходимо переместить объект R с именем `A` между двумя модулями выполнение скрипта r.

1. Добавьте в конвейер первый модуль **выполнить сценарий R** . Затем введите следующий код в текстовом поле **Скрипт R** , чтобы создать сериализованный объект `A` в виде столбца в таблице выходных данных модуля:  
  
    ```R
    azureml_main <- function(dataframe1, dataframe2){
      print("R script run.")
      # some codes generated A
      
      serialized <- as.integer(serialize(A,NULL))  
      data.set <- data.frame(serialized,stringsAsFactors=FALSE)

      return(list(dataset1=data.set, dataset2=dataframe2))
    }
    ```

    Явное преобразование в целочисленный тип выполняется потому, что функция сериализации выводит данные в `Raw` формате R, который конструктор не поддерживает.

1. Добавьте второй экземпляр модуля **выполнить сценарий R** и подключите его к порту вывода предыдущего модуля.

1. Введите следующий код в текстовом поле **Скрипт R** , чтобы извлечь объект `A` из таблицы входных данных. 

    ```R
    azureml_main <- function(dataframe1, dataframe2){
      print("R script run.")
      A <- unserialize(as.raw(dataframe1$serialized))  
      # Return datasets as a Named List
      return(list(dataset1=dataframe1, dataset2=dataframe2))
    }
    ```

## <a name="preinstalled-r-packages"></a>Предварительно установленные пакеты R

В настоящее время доступны следующие предварительно установленные пакеты R:

| Пакет      | Версия    | 
|--------------|------------| 
| аскпасс      | 1,1        | 
| assertthat   | 0.2.1      | 
| backports    | 1.1.4      | 
| base         | 3.5.1      | 
| base64enc    | 0.1–3      | 
| BH           | 1.69.0-1   | 
| bindr        | 0.1.1      | 
| bindrcpp     | 0.2.2      | 
| bitops       | 1.0–6      | 
| boot         | 1.3 – 22     | 
| broom        | 0.5.2      | 
| callr        | 3.2.0      | 
| знак вставки        | 6.0 – 84     | 
| caTools      | 1.17.1.2   | 
| cellranger   | 1.1.0      | 
| class        | 7.3 – 15     | 
| cli          | 1.1.0      | 
| Коллекция        | 0.6.0      | 
| cluster      | 2.0.7-1    | 
| codetools    | 0,2-16     | 
| colorspace   | 1.4–1      | 
| compiler     | 3.5.1      | 
| crayon       | 1.3.4      | 
| curl         | 3.3        | 
| data.table   | 1.12.2     | 
| наборы данных     | 3.5.1      | 
| DBI          | 1.0.0      | 
| dbplyr       | 1.4.1      | 
| digest       | 0.6.19     | 
| dplyr        | 0.7.6      | 
| e1071        | 1.7-2      | 
| evaluate     | 0,14       | 
| fansi        | 0.4.0      | 
| forcats      | 0.3.0      | 
| foreach      | 1.4.4      | 
| foreign      | 0,8 — 71     | 
| зависимости           | 1.3.1      | 
| gdata        | 2.18.0     | 
| универсальные шаблоны     | 0.0.2      | 
| ggplot2      | 3.2.0      | 
| glmnet       | 2.0 — 18     | 
| glue         | 1.3.1      | 
| gower        | 0.2.1      | 
| gplots       | 3.0.1.1    | 
| графика     | 3.5.1      | 
| grDevices    | 3.5.1      | 
| Сетка         | 3.5.1      | 
| gtable       | 0.3.0      | 
| gtools       | 3.8.1      | 
| haven        | 2.1.0      | 
| highr        | 0,8        | 
| hms          | 0.4.2      | 
| htmltools    | 0.3.6      | 
| httr         | 1.4.0      | 
| ipred        | 0.9-9      | 
| iterators    | 1.0.10     | 
| jsonlite     | 1.6        | 
| KernSmooth   | 2.23–15    | 
| knitr        | 1.23       | 
| labeling     | 0.3        | 
| lattice      | 0,20 — 38    | 
| lava         | 1.6.5      | 
| lazyeval     | 0.2.2      | 
| lubridate    | 1.7.4      | 
| magrittr     | 1.5        | 
| markdown     | 1          | 
| MASS         | 7.3 — 51.4   | 
| Матрица       | 1.2 – 17     | 
| methods      | 3.5.1      | 
| mgcv         | 1,8 – 28     | 
| mime         | 0,7        | 
| ModelMetrics | 1.2.2      | 
| modelr       | 0.1.4      | 
| munsell      | 0.5.0      | 
| nlme         | 3.1 – 140    | 
| nnet         | 7.3–12     | 
| numDeriv     | 2016.8-1.1 | 
| openssl      | 1.4        | 
| parallel     | 3.5.1      | 
| pillar       | 1.4.1      | 
| pkgconfig    | 2.0.2      | 
| plogr        | 0.2.0      | 
| plyr         | 1.8.4      | 
| prettyunits  | 1.0.2      | 
| processx     | 3.3.1      | 
| prodlim      | 2018.04.18 | 
| progress     | 1.2.2      | 
| ps           | 1.3.0      | 
| purrr        | 0.3.2      | 
| quadprog     | 1.5-7      | 
| quantmod     | 0.4–15     | 
| R6           | 2.4.0      | 
| randomForest | 4.6-14     | 
| RColorBrewer | 1.1–2      | 
| Rcpp         | 1.0.1      | 
| RcppRoll     | 0.3.0      | 
| readr        | 1.3.1      | 
| readxl       | 1.3.1      | 
| recipes      | 0.1.5      | 
| rematch      | 1.0.1      | 
| reprex       | 0.3.0      | 
| reshape2     | 1.4.3      | 
| reticulate   | 1.12       | 
| rlang        | 0.4.0      | 
| rmarkdown    | 1.13       | 
| ROCR         | 1.0–7      | 
| rpart        | 4.1 – 15     | 
| rstudioapi   | 0.1        | 
| rvest        | 0.3.4      | 
| scales       | 1.0.0      | 
| selectr      | 0.4–1      | 
| spatial      | 7.3–11     | 
| splines      | 3.5.1      | 
| SQUAREM      | 2017.10-1  | 
| stats        | 3.5.1      | 
| stats4       | 3.5.1      | 
| stringi      | 1.4.3      | 
| stringr      | 1.3.1      | 
| survival     | 2.44-1.1   | 
| sys          | 3.2        | 
| tcltk        | 3.5.1      | 
| tibble       | 2.1.3      | 
| tidyr        | 0.8.3      | 
| tidyselect   | 0.2.5      | 
| tidyverse    | 1.2.1      | 
| timeDate     | 3043.102   | 
| тинитекс      | 0.13       | 
| средства        | 3.5.1      | 
| tseries      | 0,10-47    | 
| TTR          | 0.23-4     | 
| utf8         | 1.1.4      | 
| utils        | 3.5.1      | 
| vctrs        | 0.1.0      | 
| viridisLite  | 0.3.0      | 
| whisker      | 0.3–2      | 
| withr        | 2.1.2      | 
| ксфун         | 0,8        | 
| xml2         | 1.2.0      | 
| xts          | 0,11-2     | 
| yaml         | 2.2.0      | 
| zeallot      | 0.1.0      | 
| zoo          | 1.8–6      | 

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [набором доступных модулей](module-reference.md) в службе Машинного обучения Azure.
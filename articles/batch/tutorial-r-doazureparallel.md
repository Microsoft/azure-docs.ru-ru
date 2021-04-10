---
title: Руководство. Параллельное моделирование на R с помощью пакетной службы Azure
description: Руководство. Пошаговые инструкции по запуску финансового моделирования методом Монте-Карло в пакетной службе Azure с помощью пакета doAzureParallel на языке R
ms.devlang: r
ms.topic: tutorial
ms.date: 10/08/2020
ms.custom: mvc
ROBOTS: NOINDEX
ms.openlocfilehash: 7e74a3b02be35b40e1a92bef269a3aed0b2313e4
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103491781"
---
# <a name="tutorial-run-a-parallel-r-simulation-with-azure-batch"></a>Руководство по Параллельное моделирование на R с помощью пакетной службы Azure

Выполняйте масштабные параллельные рабочие нагрузки R с помощью [doAzureParallel](https://www.github.com/Azure/doAzureParallel), упрощенного пакета R, который позволяет использовать пакетную службу Azure напрямую из сеанса R. Пакет doAzureParallel создан на основе популярного пакета R [foreach](https://cran.r-project.org/web/packages/foreach/index.html). doAzureParallel принимает каждую итерацию цикла foreach и передает ее в качестве задачи пакетной службы Azure.

В этом руководстве показано, как развернуть пул пакетной службы и выполнить параллельное задание R в пакетной службе Azure непосредственно из RStudio. Вы узнаете, как выполнять следующие задачи:


> [!div class="checklist"]
> * Установка пакета doAzureParallel и настройка доступа к учетным записям пакетной службы и другим учетным записям хранения.
> * Создание пула пакетной службы в качестве параллельного сервера для сеанса R.
> * Запуск примера параллельного моделирования в пуле.

## <a name="prerequisites"></a>Предварительные требования

* Установленный дистрибутив [R](https://www.r-project.org/), такой как [Microsoft R Open](https://mran.microsoft.com/open). Используйте R версии 3.3.1 или более поздней.

* [RStudio](https://www.rstudio.com/) (коммерческий выпуск или [RStudio Desktop](https://www.rstudio.com/products/rstudio/#Desktop) с открытым кодом).

* Учетная запись пакетной службы Azure и службы хранилища Azure. Чтобы создать эти учетные записи, см. примеры быстрого начала работы с пакетной службой с помощью [портала Azure](quick-create-portal.md) или [Azure CLI](quick-create-cli.md).

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]
## <a name="install-doazureparallel"></a>Установка doAzureParallel

В консоли RStudio установите [пакет doAzureParallel с GitHub](https://www.github.com/Azure/doAzureParallel). Следующие команды скачивают и устанавливают пакет и его зависимости в рамках текущего сеанса R:

```R
# Install the devtools package
install.packages("devtools")

# Install rAzureBatch package
devtools::install_github("Azure/rAzureBatch")

# Install the doAzureParallel package
devtools::install_github("Azure/doAzureParallel")

# Load the doAzureParallel library
library(doAzureParallel)
```
Установка может занять несколько минут.

Чтобы настроить doAzureParallel с ранее полученными учетными данными учетной записи, создайте в рабочем каталоге файл конфигурации *credentials.json*:

```R
generateCredentialsConfig("credentials.json")
```

Укажите в этом файле имена и ключи учетной записи хранения и учетной записи пакетной службы. Не изменяйте параметр `githubAuthenticationToken`.

По завершении файл учетных данных выглядит следующим образом:

```json
{
  "batchAccount": {
    "name": "mybatchaccount",
    "key": "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==",
    "url": "https://mybatchaccount.mybatchregion.batch.azure.com"
  },
  "storageAccount": {
    "name": "mystorageaccount",
    "key": "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ=="
  },
  "githubAuthenticationToken": ""
}
```

Сохраните файл. Выполните следующую команду, чтобы задать учетные данные для текущего сеанса R:

```R
setCredentials("credentials.json")
```

## <a name="create-a-batch-pool"></a>Создание пула пакетной службы

doAzureParallel содержит функцию для создания пула пакетной службы Azure (кластер), выполняющего параллельные задания R. На узлах запускается [виртуальная машина для обработки и анализа данных Azure](../machine-learning/data-science-virtual-machine/overview.md) на базе Ubuntu. Microsoft R Open и популярные пакеты R предварительно установлены в этом образе. Вы можете просмотреть или настроить определенные параметры кластера, например число и размер узлов.

Для создания файла конфигурации JSON кластера в рабочем каталоге:

```R
generateClusterConfig("cluster.json")
```

Откройте файл, чтобы просмотреть конфигурацию по умолчанию с 3 выделенными узлами и 3 [низкоприоритетными](batch-low-pri-vms.md) узлами. Эти параметры используются в качестве примеров, с которыми можно поэкспериментировать или которые можно изменить. Выделенные узлы зарезервированы для пула. Низкоприоритетные узлы предлагаются по сниженной цене с учетом избыточных ресурсов виртуальной машины в Azure. Эти узлы становятся недоступны, если в Azure недостаточно ресурсов.

Для работы с этим руководством измените конфигурацию следующим образом:

* Увеличьте значение `taskSlotsPerNode` до *2*, чтобы воспользоваться преимуществами обоих ядер на каждом узле.
* Задайте для `dedicatedNodes` значение *0*, чтобы воспользоваться низкоприоритетными виртуальными машинами, доступными для пакетной службы. Задайте для свойства `min` параметра `lowPriorityNodes` значение *5*, а для свойства `max` значение *10*. При необходимости вы можете выбрать меньшие значения.

Оставьте значения по умолчанию для остальных параметров и сохраните файл. Экран будет выглядеть примерно так:

```json
{
  "name": "myPoolName",
  "vmSize": "Standard_D2_v2",
  "taskSlotsPerNode": 2,
  "poolSize": {
    "dedicatedNodes": {
      "min": 0,
      "max": 0
    },
    "lowPriorityNodes": {
      "min": 5,
      "max": 10
    },
    "autoscaleFormula": "QUEUE"
  },
  "containerImage": "rocker/tidyverse:latest",
  "rPackages": {
    "cran": [],
    "github": [],
    "bioconductor": []
  },
  "commandLine": []
}
```

Создайте кластер. В пакетной службе сразу же создается пул, но для выделения и запуска вычислительных узлов понадобится несколько минут. Как только кластер станет доступным, зарегистрируйте его как параллельный сервер для сеанса R.

```R
# Create your cluster if it does not exist; this takes a few minutes
cluster <- makeCluster("cluster.json")

# Register your parallel backend
registerDoAzureParallel(cluster)

# Check that the nodes are running
getDoParWorkers()
```

В выходных данных показано число рабочих процессов выполнения для doAzureParallel. Это количество узлов, умноженное на значение `taskSlotsPerNode`. Если вы изменили конфигурацию кластера, как описано ранее, вы получите число *10*.

## <a name="run-a-parallel-simulation"></a>Запуск параллельного моделирования

После создания кластера вы готовы к выполнению цикла foreach на зарегистрированном параллельном сервере (пул пакетной службы Azure). Например, запустите финансовое моделирование методом Монте-Карло, сначала локально с использованием стандартного цикла foreach, а затем выполнив этот цикл в пакетной службе. Этот пример представляет собой упрощенную версию прогнозирования курса акций путем моделирования множества различных результатов, которые можно получить в течение 5 лет.

Предположим, что курс акций корпорации Contoso каждый день в среднем увеличивается в 1,001 раз по сравнению с начальным курсом, но имеет волатильность 0,01 (стандартное отклонение). С учетом начального курса в 100 долларов определите курс акций Contoso через 5 лет, используя моделирование цен методом Монте-Карло.

Параметры для моделирования методом Монте-Карло:

```R
mean_change = 1.001
volatility = 0.01
opening_price = 100
```

Для моделирования курса при закрытии биржи определите следующую функцию:

```R
getClosingPrice <- function() {
  days <- 1825 # ~ 5 years
  movement <- rnorm(days, mean=mean_change, sd=volatility)
  path <- cumprod(c(opening_price, movement))
  closingPrice <- path[days]
  return(closingPrice)
}
```

Сначала выполните 10 000 локальных симуляций, используя стандартный цикл foreach с ключевым словом `%do%`:

```R
start_s <- Sys.time()
# Run 10,000 simulations in series
closingPrices_s <- foreach(i = 1:10, .combine='c') %do% {
  replicate(1000, getClosingPrice())
}
end_s <- Sys.time()
```


Отобразите курс при закрытии биржи на гистограмме, чтобы показать распределение результатов:

```R
hist(closingPrices_s)
```

Результат аналогичен приведенному ниже:

![Снимок экрана: гистограмма цен закрытия](media/tutorial-r-doazureparallel/closing-prices-local.png)
  
Локальное моделирование завершится в течение нескольких секунд или менее:

```R
difftime(end_s, start_s)
```

Предполагаемое время получения результатов для 10 миллионов локальных симуляций локально, определенное методом линейной аппроксимации, составляет около 30 минут:

```R
1000 * difftime(end_s, start_s, unit = "min")
```


Теперь запустите код, используя цикл `foreach` с ключевым словом `%dopar%`, чтобы сравнить время, необходимое для выполнения 10 миллионов симуляций в Azure. Для параллелизации моделирования с использованием пакетной службы выполните 100 итераций 100 000 симуляций:

```R
# Optimize runtime. Chunking allows running multiple iterations on a single R instance.
opt <- list(chunkSize = 10)
start_p <- Sys.time()
closingPrices_p <- foreach(i = 1:100, .combine='c', .options.azure = opt) %dopar% {
  replicate(100000, getClosingPrice())
}
end_p <- Sys.time()
```

При моделировании задачи распределяются по узлам в пуле пакетной службы. Действия можно увидеть на тепловой карте пула на портале Azure. Последовательно выберите **Учетные записи пакетной службы** > *myBatchAccount*. Щелкните **Пулы** > *myPoolName*.

![Тепловая карта пула, в котором выполняются параллельные задачи R](media/tutorial-r-doazureparallel/pool.png)

Через несколько минут моделирование завершается. Пакет автоматически объединяет результаты и извлекает их из узлов. После этого результаты можно использовать во время сеанса R.

```R
hist(closingPrices_p)
```

Результат аналогичен приведенному ниже:

![Распределение курса при закрытии биржи](media/tutorial-r-doazureparallel/closing-prices.png)

Сколько времени выполнялось параллельное моделирование?

```R
difftime(end_p, start_p, unit = "min")
```

Вы увидите, что при моделировании в пуле пакетной службы обеспечивается значительное повышение производительности по сравнению с ожидаемым временем моделирования в локальной среде.

## <a name="clean-up-resources"></a>Очистка ресурсов

Задание автоматически удаляется после завершения. Если кластер больше не нужен, вызовите функцию `stopCluster` в пакете doAzureParallel для его удаления:

```R
stopCluster(cluster)
```

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> Установка пакета doAzureParallel и настройка доступа к учетным записям пакетной службы и другим учетным записям хранения.
> * Создание пула пакетной службы в качестве параллельного сервера для сеанса R.
> * Запуск примера параллельного моделирования в пуле.


Дополнительные сведения о doAzureParallel см. в документации и примерах на сайте GitHub.

> [!div class="nextstepaction"]
> [Пакет doAzureParallel](https://github.com/Azure/doAzureParallel/)

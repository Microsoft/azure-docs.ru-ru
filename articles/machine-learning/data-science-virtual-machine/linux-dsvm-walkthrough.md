---
title: Знакомство с Linux
titleSuffix: Azure Data Science Virtual Machine
description: Узнайте, как выполнять ряд распространенных задач обработки и анализа данных с помощью Виртуальной машины для обработки и анализа данных с Linux.
services: machine-learning
ms.service: data-science-vm
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 09/17/2020
ms.openlocfilehash: 42136d0d58dbc318aab0e111fcef46f80751ca88
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100517677"
---
# <a name="data-science-with-an-ubuntu-data-science-virtual-machine-in-azure"></a>Обработка и анализ данных с помощью виртуальной машины Ubuntu для обработки и анализа данных в Azure

В этом пошаговом руководстве показано, как выполнить несколько общих задач обработки и анализа данных с помощью виртуальной машины Ubuntu для обработки и анализа данных (DSVM). Ubuntu DSVM — это образ виртуальной машины, доступный в Azure и предустановленный с набором средств, которые обычно используются для анализа данных и машинного обучения. Ключевые компоненты программного обеспечения подготавливаются [к работе при подготовке виртуальной машины Ubuntu для обработки и анализа данных](./dsvm-ubuntu-intro.md). Образ DSVM позволяет за считаные минуты приступить к обработке и анализу данных без установки и настройки всех инструментов по отдельности. Вы можете легко масштабировать DSVM по мере необходимости и останавливать ее, когда она не используется. DSVM — это гибкий и экономичный ресурс.

В этом пошаговом руководстве мы анализируем набор данных [spambase](https://archive.ics.uci.edu/ml/datasets/spambase). Spambase — это набор сообщений электронной почты, которые отмечены как спам или обычная почта (не спам). Spambase также содержит ряд статистических данных по содержимому сообщений электронной почты. К статистике мы вернемся в этом пошаговом руководстве несколько позже.

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать DSVM с Linux, вам потребуются перечисленные ниже компоненты.

* **Подписка Azure**. Сведения о том, как получить подписку Azure, см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/).

* [**Виртуальная машина Ubuntu**](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804)для обработки и анализа данных. Сведения о подготовке виртуальной машины см. [в статье Подготовка виртуальной машины Ubuntu для обработки и анализа данных](./release-notes.md).
* Клиент [**X2Go**](https://wiki.x2go.org/doku.php), установленный на компьютере, и открытый сеанс XFCE. Дополнительные сведения см. в статье по [установке и настройке клиента X2Go](dsvm-ubuntu-intro.md#x2go).
* Для более плавной прокрутки в веб-браузере Firefox в DSVM переключите флаг `gfx.xrender.enabled` в режим `about:config`. [Подробнее](https://www.reddit.com/r/firefox/comments/4nfmvp/ff_47_unbearable_slow_over_remote_x11/). Рекомендуется также задать для параметра `mousewheel.enable_pixel_scrolling` значение `False`. [Подробнее](https://support.mozilla.org/questions/981140).

## <a name="download-the-spambase-dataset"></a>Скачивание набора данных spambase

Набор данных [spambase](https://archive.ics.uci.edu/ml/datasets/spambase) относительно небольшой. Он содержит 4601 пример. Благодаря умеренным требованиям к ресурсам такой размер набора данных прекрасно подходит для демонстрации некоторых основных функций DSVM.

> [!NOTE]
> Это пошаговое руководство было создано с использованием DSVM с Linux размером D2 v2 (версия Ubuntu 18.04). Для выполнения процедур, продемонстрированных в этом пошаговом руководстве, можно использовать DSVM этого размера.

Если требуется больше места для хранения, можно создать дополнительные диски и присоединить их к DSVM. Эти диски используют постоянное хранилище Azure, поэтому данные сохраняются, даже если сервер повторно подготовлен из-за изменения размера или завершения работы. Чтобы добавить диск и присоединить его к DSVM, следуйте инструкциям в статье [Добавление диска к виртуальной машине Linux](../../virtual-machines/linux/add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Эти инструкции предназначены для интерфейса Azure CLI, который уже установлен на DSVM. Их можно выполнять полностью из самой виртуальной машины DSVM. Добавить ресурсы хранения можно также с помощью [Файлов Azure](../../storage/files/storage-how-to-use-files-linux.md).

Чтобы скачать данные, откройте окно терминала и выполните следующую команду:

```bash
wget --no-check-certificate https://archive.ics.uci.edu/ml/machine-learning-databases/spambase/spambase.data
```

Скачанный файл не содержит строку заголовка. Давайте создадим другой файл, у которого есть заголовок. Для этого выполните следующую команду:

```bash
echo 'word_freq_make, word_freq_address, word_freq_all, word_freq_3d,word_freq_our, word_freq_over, word_freq_remove, word_freq_internet,word_freq_order, word_freq_mail, word_freq_receive, word_freq_will,word_freq_people, word_freq_report, word_freq_addresses, word_freq_free,word_freq_business, word_freq_email, word_freq_you, word_freq_credit,word_freq_your, word_freq_font, word_freq_000, word_freq_money,word_freq_hp, word_freq_hpl, word_freq_george, word_freq_650, word_freq_lab,word_freq_labs, word_freq_telnet, word_freq_857, word_freq_data,word_freq_415, word_freq_85, word_freq_technology, word_freq_1999,word_freq_parts, word_freq_pm, word_freq_direct, word_freq_cs, word_freq_meeting,word_freq_original, word_freq_project, word_freq_re, word_freq_edu,word_freq_table, word_freq_conference, char_freq_semicolon, char_freq_leftParen,char_freq_leftBracket, char_freq_exclamation, char_freq_dollar, char_freq_pound, capital_run_length_average,capital_run_length_longest, capital_run_length_total, spam' > headers
```

Затем объедините два файла вместе.

```bash
cat spambase.data >> headers
mv headers spambaseHeaders.data
```

Набор данных содержит несколько видов статистики для каждого сообщения электронной почты.

* Столбцы наподобие **word\_freq\__WORD_** указывают частоту упоминания в сообщении слова, соответствующего *WORD*, в процентах. Например, если выражение **word\_freq\_make** равно **1**, значит 1 % всех слов в сообщении приходится на *make*.
* Столбцы наподобие **char\_freq\__CHAR_** указывают процент всех символов в сообщении, которые соответствуют *CHAR*.
* **capital\_run\_length\_longest** — самая длинная последовательность прописных букв.
* **capital\_run\_length\_average** — средняя длина всех последовательностей прописных букв.
* **capital\_run\_length\_total** — общая длина всех последовательностей прописных букв.
* **spam** указывает, были ли сообщения признаны нежелательной почтой (1 — нежелательная почта, 0 —обычная почта).

## <a name="explore-the-dataset-by-using-r-open"></a>Просмотр набора данных с помощью Open R

Давайте изучим данные и выполним некоторые основные задачи машинного обучения с использованием R. Компонент [Microsoft Open R](https://mran.revolutionanalytics.com/open/) предварительно установлен на DSVM. Многопоточные математические библиотеки в этой предварительно установленной версии R отличаются более высокой производительностью, чем однопоточные версии. Open R предусматривает возможность воспроизведения за счет использования моментального снимка репозитория пакетов CRAN.

Чтобы получить копии примеров кода, используемых в этом пошаговом руководстве, клонируйте репозиторий Azure-Machine-Learning-Data-Science. На DSVM предварительно установлена система Git. В программе командной строки Git выполните следующую команду:

```bash
git clone https://github.com/Azure/Azure-MachineLearning-DataScience.git
```

Откройте окно терминала и запустите новый сеанс R в интерактивной консоли R. Вы также можете использовать приложение RStudio, предварительно установленное на DSVM.

Чтобы импортировать данные и настроить среду, выполните следующую команду:

```R
data <- read.csv("spambaseHeaders.data")
set.seed(123)
```

Чтобы просмотреть сводные статистические данные по каждому столбцу, выполните следующую команду:

```R
summary(data)
```

Чтобы получить другое представление данных, выполните следующую команду:

```R
str(data)
```

В этом представлении отображается тип каждой переменной и первые несколько значений в наборе данных.

Столбец **spam** распознан как целое число, но это на самом деле категориальная переменная (или факторная). Чтобы задать тип данных в этом столбце, выполните следующую команду:

```R
data$spam <- as.factor(data$spam)
```

Чтобы выполнить исследовательский анализ, используйте пакет [ggplot2](https://ggplot2.tidyverse.org/). Это популярная библиотека построения для R, предварительно установленная на DSVM. На основе приведенных ранее сводных данных мы получили статистические данные по частоте использования восклицательного знака. Давайте построим графики частоты, используя следующие команды:

```R
library(ggplot2)
ggplot(data) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Так как нулевой показатель искажает график, давайте его исключим.

```R
email_with_exclamation = data[data$char_freq_exclamation > 0, ]
ggplot(email_with_exclamation) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Выше показателя 1 наблюдается необычная плотность. Давайте взглянем только на эти данные.

```R
ggplot(data[data$char_freq_exclamation > 1, ]) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Затем разобьем их на наборы с обычной и нежелательной почтой.

```R
ggplot(data[data$char_freq_exclamation > 1, ], aes(x=char_freq_exclamation)) +
geom_density(lty=3) +
geom_density(aes(fill=spam, colour=spam), alpha=0.55) +
xlab("spam") +
ggtitle("Distribution of spam \nby frequency of !") +
labs(fill="spam", y="Density")
```

Эти примеры помогут создать похожие графики и исследовать данные в других столбцах.

## <a name="train-and-test-a-machine-learning-model"></a>Обучение и тестирование модели машинного обучения

Давайте обучим несколько моделей машинного обучения для классификации сообщений электронной почты в наборе данных как обычную и нежелательную почту. В этом разделе мы рассмотрим, как обучить модель дерева принятия решений и модель случайного леса. Затем мы протестируем точность прогнозов.

> [!NOTE]
> Пакет *rpart* (рекурсивное секционирование и деревья регрессии), используемый в следующем коде, уже установлен на DSVM.

Сначала разделим набор данных на две части: для обучения и тестирования.

```R
rnd <- runif(dim(data)[1])
trainSet = subset(data, rnd <= 0.7)
testSet = subset(data, rnd > 0.7)
```

А затем создадим дерево принятия решений для классификации сообщений электронной почты.

```R
require(rpart)
model.rpart <- rpart(spam ~ ., method = "class", data = trainSet)
plot(model.rpart)
text(model.rpart)
```

Ниже приведен результат:

![Схема созданного дерева принятия решений](./media/linux-dsvm-walkthrough/decision-tree.png)

Чтобы определить, насколько хорошо эта модель работает с обучающим набором данных, используйте следующий код:

```R
trainSetPred <- predict(model.rpart, newdata = trainSet, type = "class")
t <- table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

Чтобы определить, насколько хорошо она работает с тестовым набором данных:

```R
testSetPred <- predict(model.rpart, newdata = testSet, type = "class")
t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

Теперь давайте испробуем модель случайного леса. Случайные леса обучают множество деревьев принятия решений и выводят класс, который представляет собой режим классификаций из всех отдельных деревьев принятия решений. Это более эффективный метод машинного обучения, так как он исправляет тенденцию к переобучению обучающего набора данных, которая возникает при использовании модели дерева принятия решений.

```R
require(randomForest)
trainVars <- setdiff(colnames(data), 'spam')
model.rf <- randomForest(x=trainSet[, trainVars], y=trainSet$spam)

trainSetPred <- predict(model.rf, newdata = trainSet[, trainVars], type = "class")
table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)

testSetPred <- predict(model.rf, newdata = testSet[, trainVars], type = "class")
t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

<a name="deep-learning"></a>

## <a name="deep-learning-tutorials-and-walkthroughs"></a>Учебники и пошаговые инструкции по глубокому обучению

В дополнение к примерам на основе платформы также предоставляется набор исчерпывающих пошаговых руководств. Эти пошаговые руководства помогут вам быстро приступить к разработке приложений для глубокого обучения в таких областях, как распознавание изображений, текста и языка.

- [Запуск нейронных сетей на разных платформах](https://github.com/ilkarman/DeepLearningFrameworks). В этом комплексном руководстве показано, как переносить код с одной платформы на другую. В нем также рассказывается, как сравнивать эффективность модели и среды выполнения на различных платформах. 

- [Руководство по созданию комплексного решения по обнаружению продуктов на изображениях](https://github.com/Azure/cortana-intelligence-product-detection-from-images). Обнаружение изображений — это метод, который позволяет находить и классифицировать объекты на изображениях. Эта технология может быть очень полезна во многих сферах бизнеса в реальной жизни. Например, розничные продавцы могут использовать этот метод для определения того, какой продукт клиент взял с полки. Эти сведения, в свою очередь, помогают управлять запасами продукции. 

- [Глубокое обучение для звуковых данных](/archive/blogs/machinelearning/hearing-ai-getting-started-with-deep-learning-for-audio-on-azure). В этом учебнике показано, как обучить модель глубокого обучения для обнаружения звуковых событий в [наборе данных городских звуков](https://urbansounddataset.weebly.com/). В этом учебнике приводятся общие сведения о работе со звуковыми данными.

- [Классификация текстовых документов](https://github.com/anargyri/lstm_han). В этом пошаговом руководстве показано, как создать и обучить две разные архитектуры нейронных сетей: сеть, ориентированную на иерархию, и сеть долгой краткосрочной памяти (LSTM). Эти нейронные сети выполняют классификацию текстовых документов с помощью API Keras для глубокого обучения. Keras — это интерфейс для трех самых популярных платформ машинного обучения: Microsoft Cognitive Toolkit, TensorFlow и Theano.

## <a name="other-tools"></a>Другие средства

В остальных разделах показано, как использовать некоторые из средств, установленных в DSVM с Linux. Мы обсудим следующие средства:

* XGBoost;
* Python
* JupyterHub;
* Rattle;
* PostgreSQL и SQuirreL SQL;
* Azure Synapse Analytics (ранее — Хранилище данных SQL)

### <a name="xgboost"></a>XGBoost;

[XGBoost](https://xgboost.readthedocs.org/en/latest/) предоставляет быструю и точную реализацию усиленного дерева.

```R
require(xgboost)
data <- read.csv("spambaseHeaders.data")
set.seed(123)

rnd <- runif(dim(data)[1])
trainSet = subset(data, rnd <= 0.7)
testSet = subset(data, rnd > 0.7)

bst <- xgboost(data = data.matrix(trainSet[,0:57]), label = trainSet$spam, nthread = 2, nrounds = 2, objective = "binary:logistic")

pred <- predict(bst, data.matrix(testSet[, 0:57]))
accuracy <- 1.0 - mean(as.numeric(pred > 0.5) != testSet$spam)
print(paste("test accuracy = ", accuracy))
```

Этот средство можно вызвать из командной строки или Python.

### <a name="python"></a>Python

Для разработки на языке Python на DSVM установлены дистрибутивы Anaconda Python версий 3.5 и 2.7.

> [!NOTE]
> Дистрибутив Anaconda включает [Conda](https://conda.pydata.org/docs/index.html). Conda можно использовать для создания пользовательских сред Python, в которых могу быть установлены разные версии или пакеты.

Давайте считаем часть набора данных spambase и классифицируем сообщения, используя метод опорных векторов в Scikit-learn.

```Python
import pandas
from sklearn import svm
data = pandas.read_csv("spambaseHeaders.data", sep = ',\s*')
X = data.ix[:, 0:57]
y = data.ix[:, 57]
clf = svm.SVC()
clf.fit(X, y)
```

Для создания прогнозов используем следующую команду:

```Python
clf.predict(X.ix[0:20, :])
```

Чтобы научиться публиковать конечные точки Машинного обучения Azure, давайте создадим более простую модель. Мы будем использовать три переменные, которые использовались при публикации модели R ранее.

```Python
X = data[["char_freq_dollar", "word_freq_remove", "word_freq_hp"]]
y = data.ix[:, 57]
clf = svm.SVC()
clf.fit(X, y)
```

### <a name="jupyterhub"></a>JupyterHub

Дистрибутив Anaconda на DSVM для обработки и анализа данных поставляется с Jupyter Notebook, кроссплатформенной средой для совместного использования кода Python, R или Julia и анализа. Доступ к Jupyter Notebook можно получить через JupyterHub. Войдите в систему, используя имя пользователя и пароль локального Linux в https:// \<DSVM DNS name or IP address\> : 8000/. Все файлы конфигурации JupyterHub находятся в каталоге /etc/jupyterhub.

> [!NOTE]
> Чтобы использовать диспетчер пакетов Python (с помощью команды `pip`) из Jupyter Notebook в текущем ядре, используйте в ячейке кода такую команду:
>
>   ```Python
>    import sys
>    ! {sys.executable} -m pip install numpy -y
>   ```
> 
> Чтобы использовать установщик Conda (с помощью команды `conda`) из Jupyter Notebook в текущем ядре, используйте в ячейке кода такую команду:
>
>   ```Python
>    import sys
>    ! {sys.prefix}/bin/conda install --yes --prefix {sys.prefix} numpy
>   ```

На DSVM уже установлено несколько примеров записных книжек.

* Примеры записных книжек Python:
  * [IntroToJupyterPython.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroToJupyterPython.ipynb)
* Пример записной книжки R:
  * [IntroTutorialinR](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroTutorialinR.ipynb) 

> [!NOTE]
> На DSVM с Linux можно также использовать из командной строки язык Julia.

### <a name="rattle"></a>Rattle;

[Rattle](https://cran.r-project.org/web/packages/rattle/index.html) (*R* *A* nalytical *T* ool *T* o *L* earn *E* asily — аналитическое средство R для легкого обучения) — это графическое средство R для интеллектуального анализа данных. Rattle содержит интуитивно понятный интерфейс, упрощающий загрузку, изучение и преобразование данных, а также создание и анализ моделей. Возможности этого средства описаны в [пошаговом руководстве по графическому пользовательскому интерфейсу для интеллектуального анализа данных для R в Rattle](https://journal.r-project.org/archive/2009-2/RJournal_2009-2_Williams.pdf).

Установите и запустите Rattle с помощью следующих команд:

```R
if(!require("rattle")) install.packages("rattle")
require(rattle)
rattle()
```

> [!NOTE]
> Вам не нужно устанавливать Rattle на DSVM. Однако при открытии Rattle вам может быть предложено установить дополнительные пакеты.

В Rattle используется интерфейс на основе вкладок. Большинство вкладок соответствует шагам [процесса обработки и анализа данных группы](../team-data-science-process/index.yml), например загрузке данных и их изучению. Процесс обработки и анализа данных выполняется слева направо, перемещаясь по вкладкам. Последняя вкладка содержит журнал команд R, выполняемых Rattle.

Чтобы загрузить и настроить набор данных:

1. Чтобы загрузить файл, выберите вкладку **Data** (Данные).
1. Щелкните селектор рядом с полем **Filename** (Имя файла) и выберите **spambaseHeaders.data**.
1. Чтобы загрузить файл, Щелкните **Выполнить**. Должна появиться сводка по каждому столбцу, в том числе сведения о распознанном типе данных (входные данные, целевой объект или другой тип переменной) и количестве уникальных значений.
1. Rattle правильно распознал тип данных столбца **spam** как целевой объект. Выберите столбец **spam**, а затем задайте для параметра **Target Data Type** (Целевой тип данных) значение **Categoric** (Категориальный).

Чтобы изучить данные:

1. Выберите вкладку **Explore** (Изучение).
1. Установите флажок **Summary** > **Execute** (Сводка > Выполнить), чтобы просмотреть сведения о типах переменных и получить сводные статистические данные.
1. Чтобы просмотреть другие статистические данные о каждой переменной, установите другие флажки, например **Describe** (Описание) или **Basics** (Основные сведения).

Для создания аналитических графиков можно также использовать вкладку **Explore** (Просмотр). Чтобы построить гистограмму данных:

1. Установите флажок **Distributions**(Дистрибутивы).
1. Для **word_freq_remove** и **word_freq_you** выберите **Histogram** (Гистограмма).
1. Нажмите кнопку **Выполнить**. В окне графиков должны отобразиться оба графика частоты, глядя на которые видно, что слово _you_ встречается в сообщениях гораздо чаще, чем слово _remove_.

Графики **корреляции** не менее интересны. Чтобы создать график, выполните следующие действия.

1. Для параметра **Type** (Тип) выберите значение **Correlation** (Корреляция).
1. Нажмите кнопку **Выполнить**.
1. Rattle рекомендует использовать не более 40 переменных. Нажмите кнопку **Yes** (Да), чтобы просмотреть график.

В результате можно получить интересные корреляции: например, _technology_ (технологии) тесно коррелирует с _HP_ и _labs_ (задания). Данные также тесно коррелируют с _650_, так как это код региона пользователей, предоставивших набор данных.

Числовые значения для корреляции между словами доступны в окне **Explore** (Изучение). Интересно, что _technology_ (технология) отрицательно коррелирует с _your_ (ваш) и _money_ (деньги).

В Rattle также можно преобразовать набор данных для решения некоторых стандартных задач. Например, можно изменить масштаб признаков, добавить отсутствующие значения, обработать выбросы и удалить переменные или наблюдения с отсутствующими данными. Rattle также может определить правила взаимосвязей между наблюдениями и (или) переменными. Эти вкладки не рассматриваются в этом вводном пошаговом руководстве.

Rattle также может выполнять анализ кластера. Давайте исключим некоторые признаки, чтобы облегчить чтение выходных данных. На вкладке **Data** (Данные) выберите **Ignore** (Пропустить) рядом с каждой переменной, за исключением следующих 10 элементов:

* word_freq_hp;
* word_freq_technology;
* word_freq_george;
* word_freq_remove;
* word_freq_your;
* word_freq_dollar;
* word_freq_money;
* capital_run_length_longest;
* word_freq_business;
* spam

Вернитесь на вкладку **Cluster** (Кластер). Выберите **KMeans** (Метод K-средних), а затем задайте для параметра **Number of clusters** (число кластеров) значение **4**. Нажмите кнопку **Выполнить**. Результаты появятся в окне вывода. В одном кластере часто встречаются слова _george_ и _hp_. Скорее всего, это деловое сообщение электронной почты.

Чтобы создать базовую модель дерева принятия решений, выполните следующие действия.

1. Перейдите на вкладку **Model** (Модель).
1. Для параметра **Type** (Тип) выберите значение **Tree** (Дерево).
1. Нажмите кнопку **Execute** (Выполнить), чтобы отобразить дерево в виде текста в окне вывода.
1. Нажмите кнопку **Draw** (Рисовать), чтобы просмотреть графическое представление. Дерево принятия решений похоже на дерево, полученное ранее при использовании rpart.

В Rattle есть удобная возможность для выполнения нескольких методов машинного обучения и их быстрой оценки. Чтоб воспользоваться ею, выполните следующие действия.

1. Для параметра **Type** (Тип) выберите значение **All** (Все).
1. Нажмите кнопку **Выполнить**.
1. Когда Rattle закончит работу, можно выбрать любое значение параметра **Type** (Тип), например **SVM**, и просмотреть результаты.
1. Можно также сравнить производительность моделей с использованием набора для проверки на вкладке **Evaluate** (Оценка). Например, если выбрать **матрицу ошибок** , отобразится матрица неточностей, общий показатель ошибок и ошибка усредненного класса для каждой модели в наборе для проверки. Вы также можете построить кривые ROC, запустить анализ чувствительности и выполнить другие оценки модели.

Завершив создание моделей, выберите вкладку **Log** (Журнал), чтобы просмотреть код R, выполняемый Rattle во время сеанса. Чтобы сохранить его, нажмите кнопку **Export** (Экспорт).

> [!NOTE]
> В текущем выпуске Rattle присутствует ошибка. Чтобы изменить скрипт или использовать его для повтора этих шагов позже, необходимо вставить символ **#** перед фразой *Export this log...* (Экспорт этого журнала) в тексте журнала.

### <a name="postgresql-and-squirrel-sql"></a>PostgreSQL и SQuirreL SQL

На ВМ для обработки и анализа данных установлен компонент PostgreSQL. PostgreSQL — продвинутая реляционная база данных с открытым исходным кодом. В этом разделе показано, как загрузить наш набор данных spambase в PostgreSQL, а затем отправить запрос на предоставление содержащихся в нем данных.

Перед загрузкой данных необходимо разрешить выполнять проверку подлинности с использованием пароля на узле localhost. В окне командной строки выполните следующую команду.

```Bash
sudo gedit /var/lib/pgsql/data/pg_hba.conf
```

В нижней части файла конфигурации расположены несколько строк, описывающие разрешенные подключения.

```
# "local" is only for Unix domain socket connections:
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```

В строке **IPv4 local connections** замените **ident** на **md5**, чтобы можно было выполнить вход с помощью имени пользователя и пароля.

```
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
```

Перезапустите службу PostgreSQL.

```Bash
sudo systemctl restart postgresql
```

Чтобы запустить *psql* (интерактивный терминал для PostgreSQL) от имени встроенного пользователя Postgres, воспользуйтесь следующей командой:

```Bash
sudo -u postgres psql
```

Создайте новую учетную запись пользователя, используя имя пользователя учетной записи Linux, которую вы указывали при входе. Создайте пароль.

```Bash
CREATE USER <username> WITH CREATEDB;
CREATE DATABASE <username>;
ALTER USER <username> password '<password>';
\quit
```

Войдите в psql.

```Bash
psql
```

Импортируйте данные в новую базу данных.

```SQL
CREATE DATABASE spam;
\c spam
CREATE TABLE data (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer);
\copy data FROM /home/<username>/spambase.data DELIMITER ',' CSV;
\quit
```

Теперь давайте изучим данные и выполним некоторые запросы с помощью SQuirreL SQL — графического средства, позволяющего взаимодействовать с базами данных через драйвер JDBC.

Чтобы начать работу, запустите SQuirreL SQL из меню **Applications** (Приложения). Чтобы настроить драйвер, сделайте следующее:

1. Выберите **Windows** > **View Drivers** (Windows > Просмотр драйверов).
1. Щелкните **PostgreSQL** правой кнопкой мыши и выберите **Modify Driver** (Изменить драйвер).
1. Выберите **Extra Class Path** > **Add** (Путь к дополнительным классам > Добавить).
1. В поле **File Name** (Имя файла) введите **/usr/share/java/jdbcdrivers/postgresql-9.4.1208.jre6.jar**.
1. Щелкните **Open**(Открыть).
1. Выберите **List Drivers** (Список драйверов). В поле **Class Name** (Имя класса) выберите **org.postgresql.Driver**, а затем нажмите кнопку **ОК**.

Чтобы установить подключение к локальному серверу:

1. Выберите **Windows** > **View Aliases** (Windows > Просмотр псевдонимов).
1. Чтобы создать псевдоним, нажмите кнопку **+** . В поле имени нового псевдонима введите **Spam database** (База данных нежелательной почты). 
1. Для параметра **Driver** (Драйвер) выберите значение **PostgreSQL**.
1. В качестве URL-адреса укажите **jdbc:postgresql://localhost/spam**.
1. Введите имя пользователя и пароль.
1. Щелкните **ОК**.
1. Чтобы открыть окно **Подключение**, дважды щелкните псевдоним **База данных нежелательной почты**.
1. Выберите **Подключиться**.

Чтобы выполнить запросы:

1. Выберите вкладку **SQL** .
1. В поле запроса в верхней части вкладки **SQL** введите базовый запрос, например `SELECT * from data;`.
1. Нажмите клавиши CTRL+ВВОД, чтобы выполнить запрос. По умолчанию SQuirreL SQL возвращает первые 100 строк из запроса.

Для изучения этих данных можно выполнить множество других запросов. Например, как отличается частота упоминания слова *make* в нежелательной и обычной почте?

```SQL
SELECT avg(word_freq_make), spam from data group by spam;
```

Или каковы характеристики сообщений электронной почты, в которых часто встречается *3d*?

```SQL
SELECT * from data order by word_freq_3d desc;
```

Большинство сообщений электронной почты с большим количеством *3d*, очевидно, являются нежелательными. Эти сведения могут оказаться полезными при создании прогнозной модели для классификации сообщений электронной почты.

Если вы хотите выполнять задачи машинного обучения с данными, хранящимися в базе данных PostgreSQL, рекомендуем воспользоваться [MADlib](https://madlib.incubator.apache.org/).

### <a name="azure-synapse-analytics-formerly-sql-dw"></a>Azure Synapse Analytics (ранее — Хранилище данных SQL)

Azure синапсе Analytics — это облачная масштабируемая база данных, которая может обрабатывать большие объемы данных, как реляционные, так и нереляционные. Дополнительные сведения см. в статье [что такое Azure синапсе Analytics?](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

Чтобы подключиться к хранилищу данных и создать таблицу, выполните следующую команду из командной строки:

```Bash
sqlcmd -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -I
```

Воспользуйтесь следующей командой в командной строке sqlcmd:

```SQL
CREATE TABLE spam (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer) WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
GO
```

Скопируйте данные с помощью программы bcp.

```bash
bcp spam in spambaseHeaders.data -q -c -t  ',' -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -F 1 -r "\r\n"
```

> [!NOTE]
> Скачанный файл содержит символы конца строки в стиле Windows. Программа bcp ждет завершения строк в стиле Unix. Используйте флаг -r, чтобы сообщить об этом bcp.

Затем выполните запрос с помощью программы sqlcmd.

```sql
select top 10 spam, char_freq_dollar from spam;
GO
```

Запрос можно также выполнить с помощью SQuirreL SQL. Выполните действия, аналогичные PostgreSQL, с помощью драйвера JDBC SQL Server. Драйвер JDBC находится в папке /usr/share/java/jdbcdrivers/sqljdbc42.jar.
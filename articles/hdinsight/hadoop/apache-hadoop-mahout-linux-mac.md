---
title: Создание рекомендаций с помощью Apache Mahout и Azure HDInsight
description: Узнайте, как использовать библиотеку машинного обучения Apache Mahout для создания списка рекомендуемых фильмов с помощью HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/14/2020
ms.openlocfilehash: c31ffaf094801bdd49e5800bd338a15d8b8315f6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946501"
---
# <a name="generate-recommendations-using-apache-mahout-in-azure-hdinsight"></a>Создание рекомендаций с помощью Apache Mahout и Azure HDInsight

Узнайте, как использовать библиотеку машинного обучения [Apache Mahout](https://mahout.apache.org) для создания списка рекомендуемых к просмотру фильмов с помощью Azure HDInsight.

Mahout — это библиотека [машинного обучения](https://en.wikipedia.org/wiki/Machine_learning) для Apache Hadoop. Mahout содержит алгоритмы для обработки данных, такие как фильтрация, классификация и кластеризация. В этой статье описано, как создать список рекомендуемых фильмов на основе фильмов, уже просмотренных вашими друзьями, используя подсистему рекомендаций.

Дополнительные сведения о версии Mahout, входящей в состав кластера HDInsight, см. в статье [Что представляют собой компоненты и версии Apache Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)

## <a name="prerequisites"></a>Предварительные требования

Кластер Apache Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](./apache-hadoop-linux-tutorial-get-started.md).

## <a name="understanding-recommendations"></a>Общие сведения о рекомендациях

Одной из функций, предоставляемой Mahout, является подсистема рекомендаций. Она принимает данные в формате `userID`, `itemId` и `prefValue` (предпочтения для элемента). Затем Mahout может провести анализ совместной встречаемости, чтобы определить: *пользователи с предпочтениями к одному элементу также имеют их и к определенным другим элементам*. После этого Mahout определяет пользователей со сходными предпочтениями элементов, что можно использовать для создания рекомендаций.

Ниже приведен простой пример рабочего процесса с использованием данных о фильме.

* **Совместная встречаемость**: Глебу, Ольге и Семену нравятся фильмы *Звездные войны*, *Империя наносит ответный удар* и *Возвращение джедая*. Mahout определяет, что пользователям, которым нравится любой из этих фильмов, также нравятся другие два.

* **Совместная встречаемость**: Семену и Ольге также нравятся фильмы *Призрачная угроза*, *Атака клонов* и *Месть ситхов*. Mahout определяет, что пользователям, которым нравятся предыдущие три фильма, также нравятся и эти три.

* **Рекомендации на основе подобия**: так как Глебу понравились первые три фильма, Mahout будет отбирать фильмы, которые нравились другим пользователям со схожими предпочтениями, но которые Глеб еще не смотрел (по положительным отзывам и рейтингу). В этом случае Mahout рекомендует посмотреть фильмы *Призрачная угроза*, *Атака клонов* и *Месть ситхов*.

### <a name="understanding-the-data"></a>Основные сведения о данных

Очень кстати, что компания [GroupLens Research](https://grouplens.org/datasets/movielens/) предоставляет данные о рейтинге фильмов в формате, совместимом с Mahout. Эти данные можно найти в хранилище по умолчанию вашего кластера по адресу `/HdiSamples/HdiSamples/MahoutMovieData`.

Вы увидите два файла, `moviedb.txt` и `user-ratings.txt`. Файл `user-ratings.txt` используется во время анализа. `moviedb.txt` используется для предоставления понятного описания при просмотре результатов.

Данные, содержащиеся в файле `user-ratings.txt`, имеют структуру `userID`, `movieID`, `userRating` и `timestamp`, благодаря чему можно понять, насколько высоко каждый пользователь оценил фильм. Вот пример данных:

```output
    196    242    3    881250949
    186    302    3    891717742
    22     377    1    878887116
    244    51     2    880606923
    166    346    1    886397596
```

## <a name="run-the-analysis"></a>Выполнение анализа

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Чтобы запустить задание рекомендации, выполните следующую команду:

    ```bash
    mahout recommenditembased -s SIMILARITY_COOCCURRENCE -i /HdiSamples/HdiSamples/MahoutMovieData/user-ratings.txt -o /example/data/mahoutout --tempDir /temp/mahouttemp
    ```

> [!NOTE]  
> Выполнение задания может занять несколько минут; может выполняться несколько заданий MapReduce.

## <a name="view-the-output"></a>Просмотр результатов

1. По завершении задания воспользуйтесь следующей командой, чтобы просмотреть результат:

    ```bash
    hdfs dfs -text /example/data/mahoutout/part-r-00000
    ```

    Результат выглядит следующим образом.

    ```output
    1    [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
    2    [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
    3    [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
    4    [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]
    ```

    Первый столбец — `userID`. Значения, хранящиеся в скобках «[» и «]», — это `movieId`:`recommendationScore`.

2. Выходные данные, а также файл moviedb.txt, можно использовать, чтобы отобразить более понятные сведения о рекомендациях. Сначала необходимо скопировать файлы локально с помощью следующих команд.

    ```bash
    hdfs dfs -get /example/data/mahoutout/part-r-00000 recommendations.txt
    hdfs dfs -get /HdiSamples/HdiSamples/MahoutMovieData/* .
    ```

    Эта команда копирует выходные данные в файл **recommendations.txt** в текущем каталоге. В него также будут скопированы и файлы данных фильмов.

3. Чтобы создать сценарий Python, выполняющий поиск названий фильмов для данных в полученных рекомендациях, используйте следующую команду:

    ```bash
    nano show_recommendations.py
    ```

    Когда откроется редактор, добавьте в файл следующее содержимое:

   ```python
   #!/usr/bin/env python

   import sys

   if len(sys.argv) != 5:
        print "Arguments: userId userDataFilename movieFilename recommendationFilename"
        sys.exit(1)

   userId, userDataFilename, movieFilename, recommendationFilename = sys.argv[1:]

   print "Reading Movies Descriptions"
   movieFile = open(movieFilename)
   movieById = {}
   for line in movieFile:
       tokens = line.split("|")
       movieById[tokens[0]] = tokens[1:]
   movieFile.close()

   print "Reading Rated Movies"
   userDataFile = open(userDataFilename)
   ratedMovieIds = []
   for line in userDataFile:
       tokens = line.split("\t")
       if tokens[0] == userId:
           ratedMovieIds.append((tokens[1],tokens[2]))
   userDataFile.close()

   print "Reading Recommendations"
   recommendationFile = open(recommendationFilename)
   recommendations = []
   for line in recommendationFile:
       tokens = line.split("\t")
       if tokens[0] == userId:
           movieIdAndScores = tokens[1].strip("[]\n").split(",")
           recommendations = [ movieIdAndScore.split(":") for movieIdAndScore in movieIdAndScores ]
           break
   recommendationFile.close()

   print "Rated Movies"
   print "------------------------"
   for movieId, rating in ratedMovieIds:
       print "%s, rating=%s" % (movieById[movieId][0], rating)
   print "------------------------"

   print "Recommended Movies"
   print "------------------------"
   for movieId, score in recommendations:
       print "%s, score=%s" % (movieById[movieId][0], score)
   print "------------------------"
   ```

    Нажмите **CTRL+X**, **Y**, и, наконец, **ВВОД**, чтобы сохранить данные.

4. Запустите скрипт Python. Следующая команда предполагает, что вы находитесь в каталоге со всеми скачанными файлами.

    ```bash
    python show_recommendations.py 4 user-ratings.txt moviedb.txt recommendations.txt
    ```

    Она рассматривает рекомендации, созданные для пользователя ID 4.

   * Файл **user-ratings.txt** используется для получения оцененных фильмов.

   * Файл **moviedb.txt** используется для извлечения названий фильмов.

   * Файл **recommendations.txt** используется для получения рекомендованных фильмов для этого пользователя.

     Результат этой команды аналогичен следующему:

        ```output
        Seven Years in Tibet (1997), score=5.0
        Indiana Jones and the Last Crusade (1989), score=5.0
        Jaws (1975), score=5.0
        Sense and Sensibility (1995), score=5.0
        Independence Day (ID4) (1996), score=5.0
        My Best Friend's Wedding (1997), score=5.0
        Jerry Maguire (1996), score=5.0
        Scream 2 (1997), score=5.0
        Time to Kill, A (1996), score=5.0
        ```

## <a name="delete-temporary-data"></a>Удаление временных данных

Задания Mahout не удаляют временные данные, созданные во время выполнения задания. В примере задания указан параметр `--tempDir` , чтобы выделить временные файлы в отдельный каталог для простоты последующего их удаления. Чтобы удалить временные файлы, используйте следующую команду:

```bash
hdfs dfs -rm -f -r /temp/mahouttemp
```

> [!WARNING]  
> Если вы хотите выполнить команду еще раз, необходимо удалить выходной каталог. Используйте следующую команду для удаления этого каталога:
>
> `hdfs dfs -rm -f -r /example/data/mahoutout`

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как использовать Mahout, откройте для себя другие способы работы с данными в HDInsight.

* [Обзор Apache Hive и HiveQL в Azure HDInsight](hdinsight-use-hive.md)
* [Использование MapReduce с HDInsight](hdinsight-use-mapreduce.md)

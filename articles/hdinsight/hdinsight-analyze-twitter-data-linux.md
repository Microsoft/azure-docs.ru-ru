---
title: Анализ данных Twitter с помощью Apache Hive в Azure HDInsight
description: Сведения о преобразовании необработанных данных Twitter в поддерживающую поиск таблицу Hive с помощью Apache Hive и Apache Hadoop в HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.date: 12/16/2019
ms.openlocfilehash: 74a18be37608b2ac0c8a64655fa833753111706a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98942780"
---
# <a name="analyze-twitter-data-using-apache-hive-and-apache-hadoop-on-hdinsight"></a>Анализ данных Twitter с помощью Apache Hive и Apache Hadoop в HDInsight

Узнайте, как использовать [Apache Hive](https://hive.apache.org/) для обработки данных Twitter. Результатом является список пользователей Twitter, отправивших большинство твитов, которые содержат определенное слово.

> [!IMPORTANT]  
> Действия, описанные в этом документе, были протестированы в HDInsight 3.6.

## <a name="get-the-data"></a>Получение данных

Twitter позволяет получать данные для каждого твита в виде документа JSON с помощью интерфейса REST API. [OAuth](https://oauth.net) .

### <a name="create-a-twitter-application"></a>Создание приложения Twitter

1. В веб-браузере Войдите в [https://developer.twitter.com/apps/](https://developer.twitter.com/apps/) . Щелкните ссылку **Регистрация сейчас** , если у вас нет учетной записи Twitter.

2. Выберите **создать новое приложение**.

3. Введите **Имя**, **Описание**, **Веб-сайт**. В поле **Веб-сайт** можно использовать URL-адрес. В следующей таблице приведены некоторые примеры значений:

   | Поле | Значение |
   |--- |--- |
   | Имя |MyHDInsightApp |
   | Описание |MyHDInsightApp |
   | Веб-сайт |`https://www.myhdinsightapp.com` |

4. Выберите **Да, я принимаю**, а затем выберите **создать приложение Twitter**.

5. Перейдите на вкладку **разрешения** . Разрешение по умолчанию доступно **только для чтения**.

6. Откройте вкладку **Ключи и токены доступа** .

7. Выберите **создать мой маркер доступа**.

8. Выберите **проверить OAuth** в правом верхнем углу страницы.

9. Запишите **ключ клиента**, **Секрет клиента**, **Маркер доступа** и **Секрет маркера доступа**.

### <a name="download-tweets"></a>Скачивание твитов

Следующий код Python скачивает 10 000 твитов из Twitter и сохраняет их в файл с именем **tweets.txt**.

> [!NOTE]  
> Так как Python уже установлен, в кластере HDInsight выполняются следующие действия.

1. С помощью команды [ssh command](./hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Используйте следующие команды для установки [Tweepy](https://www.tweepy.org/), индикатора [выполнения](https://pypi.python.org/pypi/progressbar/2.2)и других необходимых пакетов:

   ```bash
   sudo apt install python-dev libffi-dev libssl-dev
   sudo apt remove python-openssl
   python -m pip install virtualenv
   mkdir gettweets
   cd gettweets
   virtualenv gettweets
   source gettweets/bin/activate
   pip install tweepy progressbar pyOpenSSL requests[security]
   ```

1. Создайте файл **gettweets.py** с помощью следующей команды:

   ```bash
   nano gettweets.py
   ```

1. Измените приведенный ниже код, заменив `Your consumer secret` ,, `Your consumer key` `Your access token` и `Your access token secret` соответствующими данными из приложения Twitter. Затем вставьте измененный код в качестве содержимого файла **gettweets.py** .

   ```python
   #!/usr/bin/python

   from tweepy import Stream, OAuthHandler
   from tweepy.streaming import StreamListener
   from progressbar import ProgressBar, Percentage, Bar
   import json
   import sys

   #Twitter app information
   consumer_secret='Your consumer secret'
   consumer_key='Your consumer key'
   access_token='Your access token'
   access_token_secret='Your access token secret'

   #The number of tweets we want to get
   max_tweets=100

   #Create the listener class that receives and saves tweets
   class listener(StreamListener):
       #On init, set the counter to zero and create a progress bar
       def __init__(self, api=None):
           self.num_tweets = 0
           self.pbar = ProgressBar(widgets=[Percentage(), Bar()], maxval=max_tweets).start()

       #When data is received, do this
       def on_data(self, data):
           #Append the tweet to the 'tweets.txt' file
           with open('tweets.txt', 'a') as tweet_file:
               tweet_file.write(data)
               #Increment the number of tweets
               self.num_tweets += 1
               #Check to see if we have hit max_tweets and exit if so
               if self.num_tweets >= max_tweets:
                   self.pbar.finish()
                   sys.exit(0)
               else:
                   #increment the progress bar
                   self.pbar.update(self.num_tweets)
           return True

       #Handle any errors that may occur
       def on_error(self, status):
           print status

   #Get the OAuth token
   auth = OAuthHandler(consumer_key, consumer_secret)
   auth.set_access_token(access_token, access_token_secret)
   #Use the listener class for stream processing
   twitterStream = Stream(auth, listener())
   #Filter for these topics
   twitterStream.filter(track=["azure","cloud","hdinsight"])
   ```

    > [!TIP]  
    > Настройте фильтр раздела в последней строке для отслеживания популярных ключевых слов. Использование ключевых слов, популярных во время выполнения сценария, позволяет быстрее собирать данные.

1. Нажмите клавиши **Ctrl + X**, а затем **Y** (Да) для сохранения файла.

1. Чтобы запустить файл и скачать твиты, выполните следующую команду:

    ```bash
    python gettweets.py
    ```

    Отобразится индикатор хода выполнения. Он дойдет до 100 % по мере скачивания твитов.

   > [!NOTE]  
   > Если индикатор хода выполнения перемещается очень медленно, то следует изменить фильтр, чтобы отслеживать популярные темы. Если в фильтре содержится много твитов, можно быстро получить нужные твиты 100.

### <a name="upload-the-data"></a>Передача данных

Для отправки данных в хранилище HDInsight используйте следующие команды:

```bash
hdfs dfs -mkdir -p /tutorials/twitter/data
hdfs dfs -put tweets.txt /tutorials/twitter/data/tweets.txt
```

Эти команды сохраняют данные в расположении, к которому могут обращаться все узлы в кластере.

## <a name="run-the-hiveql-job"></a>Выполнение задания HiveQL

1. Используйте следующую команду, чтобы создать файл, содержащий инструкции [HiveQL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual) :

   ```bash
   nano twitter.hql
   ```

    В качестве содержимого файла добавьте следующий текст:

   ```hiveql
   set hive.exec.dynamic.partition = true;
   set hive.exec.dynamic.partition.mode = nonstrict;
   -- Drop table, if it exists
   DROP TABLE tweets_raw;
   -- Create it, pointing toward the tweets logged from Twitter
   CREATE EXTERNAL TABLE tweets_raw (
       json_response STRING
   )
   STORED AS TEXTFILE LOCATION '/tutorials/twitter/data';
   -- Drop and recreate the destination table
   DROP TABLE tweets;
   CREATE TABLE tweets
   (
       id BIGINT,
       created_at STRING,
       created_at_date STRING,
       created_at_year STRING,
       created_at_month STRING,
       created_at_day STRING,
       created_at_time STRING,
       in_reply_to_user_id_str STRING,
       text STRING,
       contributors STRING,
       retweeted STRING,
       truncated STRING,
       coordinates STRING,
       source STRING,
       retweet_count INT,
       url STRING,
       hashtags array<STRING>,
       user_mentions array<STRING>,
       first_hashtag STRING,
       first_user_mention STRING,
       screen_name STRING,
       name STRING,
       followers_count INT,
       listed_count INT,
       friends_count INT,
       lang STRING,
       user_location STRING,
       time_zone STRING,
       profile_image_url STRING,
       json_response STRING
   );
   -- Select tweets from the imported data, parse the JSON,
   -- and insert into the tweets table
   FROM tweets_raw
   INSERT OVERWRITE TABLE tweets
   SELECT
       cast(get_json_object(json_response, '$.id_str') as BIGINT),
       get_json_object(json_response, '$.created_at'),
       concat(substr (get_json_object(json_response, '$.created_at'),1,10),' ',
       substr (get_json_object(json_response, '$.created_at'),27,4)),
       substr (get_json_object(json_response, '$.created_at'),27,4),
       case substr (get_json_object(json_response,    '$.created_at'),5,3)
           when "Jan" then "01"
           when "Feb" then "02"
           when "Mar" then "03"
           when "Apr" then "04"
           when "May" then "05"
           when "Jun" then "06"
           when "Jul" then "07"
           when "Aug" then "08"
           when "Sep" then "09"
           when "Oct" then "10"
           when "Nov" then "11"
           when "Dec" then "12" end,
       substr (get_json_object(json_response, '$.created_at'),9,2),
       substr (get_json_object(json_response, '$.created_at'),12,8),
       get_json_object(json_response, '$.in_reply_to_user_id_str'),
       get_json_object(json_response, '$.text'),
       get_json_object(json_response, '$.contributors'),
       get_json_object(json_response, '$.retweeted'),
       get_json_object(json_response, '$.truncated'),
       get_json_object(json_response, '$.coordinates'),
       get_json_object(json_response, '$.source'),
       cast (get_json_object(json_response, '$.retweet_count') as INT),
       get_json_object(json_response, '$.entities.display_url'),
       array(
           trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[1].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[2].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[3].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[4].text')))),
       array(
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[1].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[2].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[3].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[4].screen_name')))),
       trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
       trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
       get_json_object(json_response, '$.user.screen_name'),
       get_json_object(json_response, '$.user.name'),
       cast (get_json_object(json_response, '$.user.followers_count') as INT),
       cast (get_json_object(json_response, '$.user.listed_count') as INT),
       cast (get_json_object(json_response, '$.user.friends_count') as INT),
       get_json_object(json_response, '$.user.lang'),
       get_json_object(json_response, '$.user.location'),
       get_json_object(json_response, '$.user.time_zone'),
       get_json_object(json_response, '$.user.profile_image_url'),
       json_response
   WHERE (length(json_response) > 500);
   ```

1. Нажмите клавиши **Ctrl + X**, а затем **Y** (Да) для сохранения файла.

1. Выполните приведенную ниже команду, чтобы запустить код HiveQL в файле:

   ```bash
   beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -i twitter.hql
   ```

    Эта команда запускает файл **twitter.hql**. После выполнения запроса отобразится строка `jdbc:hive2//localhost:10001/>`.

1. Используйте следующий запрос в командной строке Beeline, чтобы убедиться, что данные импортированы:

   ```hiveql
   SELECT name, screen_name, count(1) as cc
   FROM tweets
   WHERE text like "%Azure%"
   GROUP BY name,screen_name
   ORDER BY cc DESC LIMIT 10;
   ```

    Этот запрос возвращает не более 10 твитов, содержащих слово **Azure** в тексте сообщения.

    > [!NOTE]  
    > Если вы изменили фильтр в сценарии `gettweets.py`, замените **Azure** одним из используемых фильтров.

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали, как преобразовать неструктурированный набор данных JSON в структурированную [Apache Hive](https://hive.apache.org/) таблицу. Дополнительные сведения о Hive в HDInsight см. в следующих документах:

* [Приступая к работе с HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Анализ данных о задержке рейсов с помощью HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
---
title: Доступ к журналам приложений Apache Hadoop YARN в Azure HDInsight
description: Узнайте, как получить доступ к журналам приложений YARN в кластере HDInsight (Apache Hadoop) под управлением Linux с помощью как командной строки, так и веб-браузера.
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: 9558e6c5ddd58b1d5fd70da03187caef50d1275d
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104865576"
---
# <a name="access-apache-hadoop-yarn-application-logs-on-linux-based-hdinsight"></a>Доступ к журналам приложений Apache Hadoop YARN в HDInsight под управлением Linux

Узнайте, как получить доступ к журналам приложений [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) (Yet Another Resource Negotiator) в кластере Apache Hadoop в Azure HDInsight.

## <a name="what-is-apache-yarn"></a>Что такое Apache YARN?

YARN поддерживает несколько моделей программирования (один из них Apache Hadoop MapReduce) путем разделения управления ресурсами от планирования и мониторинга приложений. YARN использует глобальную *`ResourceManager`* (RM), для каждого рабочего узла *NodeManagers* (узлов) и для каждого приложения *диспетчеров* (AMs). Диспетчер приложений согласовывает ресурсы (ЦП, память, диск, сеть), необходимые для работы приложения, с диспетчером ресурсов. Диспетчер ресурсов совместно с диспетчером узлов предоставляют эти ресурсы в виде *контейнеров*. Диспетчер приложений отвечает за отслеживание хода выполнения контейнеров, назначаемых ему диспетчером ресурсов. Приложению может потребоваться много контейнеров в зависимости от характера приложения.

Для выполнения приложения может потребоваться несколько *попыток*. Если выполнение приложения завершается ошибкой, его можно повторить как новую попытку. Каждая попытка выполняется в контейнере. В смысле, контейнер предоставляет контекст для базовой единицы работы, выполненной приложением YARN. Вся работа, выполненная в контексте контейнера, выполняется на единственном рабочем узле, на котором был предоставлен контейнер. Дополнительные сведения см. в статье [Hadoop: написание приложений YARN](https://hadoop.apache.org/docs/r2.7.4/hadoop-yarn/hadoop-yarn-site/WritingYarnApplications.html)или [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) .

Чтобы масштабировать кластер для поддержки большей пропускной способности обработки, можно использовать автоматическое [масштабирование](hdinsight-autoscale-clusters.md) или [масштабировать кластеры вручную с помощью нескольких разных языков](hdinsight-scaling-best-practices.md#utilities-to-scale-clusters).

## <a name="yarn-timeline-server"></a>Сервер временной шкалы YARN

[Apache Hadoop YARN Timeline Server](https://hadoop.apache.org/docs/r2.7.3/hadoop-yarn/hadoop-yarn-site/TimelineServer.html) предоставляет общие сведения о приложениях, завершивших работу.

YARN Timeline Server содержит следующие типы данных:

* уникальный идентификатор приложения;
* имя пользователя, запустившего приложение;
* информация о попытках завершить приложение;
* информация о контейнерах, которые использовались во время каждой из попыток.

## <a name="yarn-applications-and-logs"></a>Приложения и журналы YARN

Журналы приложений (и соответствующие журналы контейнеров) крайне важны для отладки проблемных приложений Hadoop. YARN предоставляет удобную платформу для сбора, агрегирования и хранения журналов приложений с помощью объединения журналов.

Функция объединения журналов делает доступ к журналам приложений более детерминированным, так как она объединяет журналы со всех контейнеров на рабочем узле и хранит их как один сводный файл журнала на рабочем узле в файловой системе по умолчанию после завершения приложения. Ваше приложение может использовать сотни или тысячи контейнеров, но журналы для всех контейнеров, выполненных на одном рабочем узле, всегда объединяются в один файл. Таким образом, на каждый рабочий узел, используемый приложением, входит только один журнал. Объединение журналов включено по умолчанию на кластерах HDInsight версии 3.0 и более поздних версий. Объединенные журналы находятся в хранилище по умолчанию для кластера. В качестве пути HDFS к журналам используется следующий путь:

```
/app-logs/<user>/logs/<applicationId>
```

В пути `user` — это имя пользователя, запустившего приложение. `applicationId` является уникальным идентификатором, назначенным YARN RM приложению.

Агрегированные журналы не могут быть прочитаны напрямую, так как они написаны в TFile двоичном формате, индексируемом по контейнеру. Используйте журналы YARN `ResourceManager` или средства CLI для просмотра этих журналов в виде обычного текста для интересующих вас приложений или контейнеров.

## <a name="yarn-logs-in-an-esp-cluster"></a>Yarn журналов в кластере ESP

К пользовательской в Ambari необходимо добавить две конфигурации `mapred-site` .

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net`, где `CLUSTERNAME` — это имя вашего кластера.

1. В пользовательском интерфейсе Ambari перейдите к **MapReduce2**  >  **configs**  >  **Advanced**  >  **Custom mapred-site**.

1. Добавьте *один* из следующих наборов свойств:

    **Набор 1**

    ```
    mapred.acls.enabled=true
    mapreduce.job.acl-view-job=*
    ```

    **Набор 2**

    ```
    mapreduce.job.acl-view-job=<user1>,<user2>,<user3>
    ```

1. Сохраните изменения и перезапустите все затронутые службы.

## <a name="yarn-cli-tools"></a>Средства CLI для YARN

1. С помощью команды [ssh command](./hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Получите список всех идентификаторов текущих выполняемых приложений Yarn с помощью следующей команды:

    ```bash
    yarn top
    ```

    Запишите идентификатор приложения из `APPLICATIONID` столбца, журналы которого должны быть скачаны.

    ```output
    YARN top - 18:00:07, up 19d, 0:14, 0 active users, queue(s): root
    NodeManager(s): 4 total, 4 active, 0 unhealthy, 0 decommissioned, 0 lost, 0 rebooted
    Queue(s) Applications: 2 running, 10 submitted, 0 pending, 8 completed, 0 killed, 0 failed
    Queue(s) Mem(GB): 97 available, 3 allocated, 0 pending, 0 reserved
    Queue(s) VCores: 58 available, 2 allocated, 0 pending, 0 reserved
    Queue(s) Containers: 2 allocated, 0 pending, 0 reserved
    
                      APPLICATIONID USER             TYPE      QUEUE   #CONT  #RCONT  VCORES RVCORES     MEM    RMEM  VCORESECS    MEMSECS %PROGR       TIME NAME
     application_1490377567345_0007 hive            spark  thriftsvr       1       0       1       0      1G      0G    1628407    2442611  10.00   18:20:20 Thrift JDBC/ODBC Server
     application_1490377567345_0006 hive            spark  thriftsvr       1       0       1       0      1G      0G    1628430    2442645  10.00   18:20:20 Thrift JDBC/ODBC Server
    ```

1. Эти журналы можно отобразить в виде обычного текста, запустив одну из указанных ниже команд.

    ```bash
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application>
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application> -containerId <containerId> -nodeAddress <worker-node-address>
    ```

    При выполнении этих команд необходимо указать параметры &lt;applicationId>, &lt;user-who-started-the-application>, &lt;containerId> и &lt;worker-node-address>.

### <a name="other-sample-commands"></a>Другие примеры команд

1. Скачайте журналы контейнеров Yarn для всех главных приложений с помощью приведенной ниже команды. На этом шаге будет создан файл журнала с именем `amlogs.txt` в текстовом формате.

    ```bash
    yarn logs -applicationId <application_id> -am ALL > amlogs.txt
    ```

1. Скачайте журналы только для последнего основного контейнера приложения Yarn с помощью следующей команды:

    ```bash
    yarn logs -applicationId <application_id> -am -1 > latestamlogs.txt
    ```

1. Скачайте журналы для первых двух основных контейнеров приложения Yarn с помощью следующей команды:

    ```bash
    yarn logs -applicationId <application_id> -am 1,2 > first2amlogs.txt
    ```

1. Скачайте все журналы контейнеров приложения Yarn с помощью следующей команды:

    ```bash
    yarn logs -applicationId <application_id> > logs.txt
    ```

1. Скачайте журнал контейнера Yarn для определенного контейнера с помощью следующей команды:

    ```bash
    yarn logs -applicationId <application_id> -containerId <container_id> > containerlogs.txt
    ```

## <a name="yarn-resourcemanager-ui"></a>`ResourceManager`Пользовательский интерфейс YARN

`ResourceManager`Пользовательский интерфейс YARN работает в кластере головного узла. Доступ к нему осуществляется через веб-интерфейс Ambari. Чтобы просмотреть журналы YARN, выполните следующие действия:

1. В браузере перейдите по адресу `https://CLUSTERNAME.azurehdinsight.net`. Параметр CLUSTERNAME нужно заменить именем кластера HDInsight.

2. В списке служб слева выберите **YARN**.

    :::image type="content" source="./media/hdinsight-hadoop-access-yarn-app-logs-linux/yarn-service-selected.png" alt-text="Выбрана служба Apache Ambari Yarn":::

3. В раскрывающемся списке **быстрые ссылки** выберите один из головных узлов кластера, а затем выберите **`ResourceManager Log`** .

    :::image type="content" source="./media/hdinsight-hadoop-access-yarn-app-logs-linux/hdi-yarn-quick-links.png" alt-text="Быстрые ссылки Apache Ambari Yarn":::

    Вы увидите список ссылок на журналы YARN.

## <a name="next-steps"></a>Дальнейшие действия

* [Архитектура Apache Hadoop в HDInsight](hdinsight-hadoop-architecture.md)
* [Устранение неполадок рабочих нагрузок Apache Hadoop YARN с помощью Azure HDInsight](hdinsight-troubleshoot-yarn.md)

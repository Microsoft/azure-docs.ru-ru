---
title: Использование Apache Hadoop Hive с Curl в HDInsight — Azure
description: Узнайте, как удаленно отправлять задания Apache Pig в Azure HDInsight с помощью функции фигурного обучения.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/06/2020
ms.openlocfilehash: 124661c57f779b9f8a639debcc093ed15e717694
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946458"
---
# <a name="run-apache-hive-queries-with-apache-hadoop-in-hdinsight-using-rest"></a>Выполнение запросов Apache Hive в Apache Hadoop в HDInsight с использованием REST

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

Узнайте, как с помощью REST API WebHCat выполнять запросы Apache Hive с Apache Hadoop в кластере Azure HDInsight.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](./apache-hadoop-linux-tutorial-get-started.md).

* Клиент REST. В этом документе используется [командлет Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest) [в Windows PowerShell и](https://curl.haxx.se/) функция [Bash](/windows/wsl/install-win10).

* При использовании Bash также потребуется JQ, процессор командной строки JSON.  См. раздел [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).

## <a name="base-uri-for-rest-api"></a>Базовый URI для API-интерфейса RESTful

Базовый универсальный код ресурса (URI) для REST API в HDInsight — `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME` , где `CLUSTERNAME` — это имя кластера.  В именах кластеров в URI **учитывается регистр**.  Хотя имя кластера в части URI () в полном доменном имени ( `CLUSTERNAME.azurehdinsight.net` ) не учитывает регистр, другие вхождения в URI учитывают регистр.

## <a name="authentication"></a>Аутентификация

При использовании Curl или любых других средств связи REST с WebHCat нужно выполнять аутентификацию запросов с помощью пароля и имени пользователя администратора кластера HDInsight. REST API защищен с помощью [обычной проверки подлинности](https://en.wikipedia.org/wiki/Basic_access_authentication). Чтобы обеспечить безопасную отправку учетных данных на сервер, все запросы следует отправлять с помощью протокола HTTPS.

### <a name="setup-preserve-credentials"></a>Установка (сохранение учетных данных)

Сохраните свои учетные данные, чтобы избежать их повторного ввода для каждого примера.  Имя кластера будет сохранено на отдельном шаге.

**A. bash**  
Измените приведенный ниже сценарий, заменив его `PASSWORD` фактическим паролем.  Затем введите команду.

```bash
export password='PASSWORD'
```  

**Б. PowerShell** Выполните приведенный ниже код и введите свои учетные данные во всплывающем окне:

```powershell
$creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
```

### <a name="identify-correctly-cased-cluster-name"></a>Выявление правильного имени кластера с учетом регистра

Фактический регистр имени кластера может отличаться от ожидаемого, в зависимости от способа создания кластера.  В этих шагах будет показан фактический регистр, а затем сохранен в переменной для всех последующих примеров.

Измените приведенные ниже сценарии, чтобы они заменили `CLUSTERNAME` имя кластера. Затем введите команду. (Имя кластера для FQDN не учитывает регистр.)

```bash
export clusterName=$(curl -u admin:$password -sS -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
echo $clusterName
```  

```powershell
# Identify properly cased cluster name
$resp = Invoke-WebRequest -Uri "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" `
    -Credential $creds -UseBasicParsing
$clusterName = (ConvertFrom-Json $resp.Content).items.Clusters.cluster_name;

# Show cluster name
$clusterName
```

## <a name="run-a-hive-query"></a>Выполнение запроса Hive

1. Чтобы убедиться, что можно подключиться к кластеру HDInsight, используйте одну из следующих команд:

    ```bash
    curl -u admin:$password -G https://$clusterName.azurehdinsight.net/templeton/v1/status
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/status" `
       -Credential $creds `
       -UseBasicParsing
    $resp.Content
    ```

    Вы должны получить ответ, аналогичный показанному ниже тексту.

    ```json
    {"status":"ok","version":"v1"}
    ```

    Ниже приведены параметры, используемые в этой команде:

    * `-u` — имя пользователя и пароль, используемый для проверки подлинности запроса.
    * `-G` — указывает, что этот запрос является операцией GET.

1. Начало URL-адреса `https://$CLUSTERNAME.azurehdinsight.net/templeton/v1` одинаковое для всех запросов. Путь `/status` указывает, что по запросу серверу должно быть возвращено состояние WebHCat (другое название — Templeton). Используя следующую команду, можно также запросить версию Hive:

    ```bash
    curl -u admin:$password -G https://$clusterName.azurehdinsight.net/templeton/v1/version/hive
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/version/hive" `
       -Credential $creds `
       -UseBasicParsing
    $resp.Content
    ```

    Этот запрос возвращает ответ, аналогичный показанному ниже тексту.

    ```json
    {"module":"hive","version":"1.2.1000.2.6.5.3008-11"}
    ```

1. Используйте следующую команду, чтобы создать таблицу **log4jLogs**.

    ```bash
    jobid=$(curl -s -u admin:$password -d user.name=admin -d execute="DROP+TABLE+log4jLogs;CREATE+EXTERNAL+TABLE+log4jLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+ROW+FORMAT+DELIMITED+FIELDS+TERMINATED+BY+' '+STORED+AS+TEXTFILE+LOCATION+'/example/data/';SELECT+t4+AS+sev,COUNT(*)+AS+count+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log'+GROUP+BY+t4;" -d statusdir="/example/rest" https://$clusterName.azurehdinsight.net/templeton/v1/hive | jq -r .id)
    echo $jobid
    ```

    ```powershell
    $reqParams = @{"user.name"="admin";"execute"="DROP TABLE log4jLogs;CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED BY ' ' STORED AS TEXTFILE LOCATION '/example/data/;SELECT t4 AS sev,COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;";"statusdir"="/example/rest"}
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/hive" `
       -Credential $creds `
       -Body $reqParams `
       -Method POST `
       -UseBasicParsing
    $jobID = (ConvertFrom-Json $resp.Content).id
    $jobID
    ```

    Этот запрос использует метод POST, который отправляет данные как часть запроса в REST API. Следующие значения данных отправляются с запросом:

     * `user.name` — пользователь, выполняющий команду.
     * `execute` — операторы HiveQL, которые необходимо выполнить.
     * `statusdir` — каталог, в который будет записано состояние этого задания.

   Эти операторы выполняют следующие действия:

   * `DROP TABLE` — Если таблица уже существует, она удаляется.
   * `CREATE EXTERNAL TABLE` — создает "внешнюю" таблицу в Hive. Внешние таблицы хранят только определение таблицы в Hive. Данные остаются в исходном расположении.

     > [!NOTE]  
     > Внешние таблицы следует использовать, если исходные данные должны обновляться с использованием внешних источников. Например, процессом автоматизированной передачи данных или другой операцией MapReduce.
     >
     > Удаление внешней таблицы **не** приводит к удалению данных, будет удалено только определение таблицы.

   * `ROW FORMAT` — настройка форматирования данных. Поля всех журналов разделены пробелами.
   * `STORED AS TEXTFILE LOCATION` — Где хранятся данные (каталог example/Data) и хранятся в виде текста.
   * `SELECT` — Выбирает количество строк, в которых столбец **T4** содержит значение **[Error]**. Эта инструкция возвращает значение **3**, так как данное значение содержат три строки.

     > [!NOTE]  
     > Обратите внимание, что при использовании Curl пробелы между операторами HiveQL заменяются знаком `+`. Заключенные в кавычки значения, содержащие пробелы в качестве разделителя, заменять на `+`не нужно.

      Эта команда возвращает идентификатор задания, который может использоваться для проверки состояния задания.

1. Чтобы проверить состояние задания, используйте следующую команду.

    ```bash
    curl -u admin:$password -d user.name=admin -G https://$clusterName.azurehdinsight.net/templeton/v1/jobs/$jobid | jq .status.state
    ```

    ```powershell
    $reqParams=@{"user.name"="admin"}
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/jobs/$jobID" `
       -Credential $creds `
       -Body $reqParams `
       -UseBasicParsing
    # ConvertFrom-JSON can't handle duplicate names with different case
    # So change one to prevent the error
    $fixDup=$resp.Content.Replace("jobID","job_ID")
    (ConvertFrom-Json $fixDup).status.state
    ```

    Если задание завершено, оно будет в состоянии **SUCCEEDED** (Успешно).

1. После изменения состояния задания на **SUCCEEDED** результаты задания можно получить из хранилища больших двоичных объектов Azure. Параметр `statusdir`, передаваемый с помощью запроса, содержит расположение выходного файла. В данном случае это `/example/rest`. Этот адрес задает каталог `example/curl` для сохранения выходных данных, который размещен в хранилище по умолчанию для кластера.

    Вы можете вывести список этих файлов и скачать их с помощью [интерфейса командной строки Azure](/cli/azure/install-azure-cli). Дополнительные сведения об использовании Azure CLI со службой хранилища Azure см. в документе [Использование Azure CLI со службой хранилища Azure](../../storage/blobs/storage-quickstart-blobs-cli.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительная информация о других способах работы с Hadoop в HDInsight.

* [Использование Apache Hive с Apache Hadoop в HDInsight](hdinsight-use-hive.md)
* [Использование MapReduce в Apache Hadoop в HDInsight](hdinsight-use-mapreduce.md)

Дополнительную информацию о REST API, используемом в этом документе, см. в [справочнике по WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference).
---
title: Использование MapReduce и Curl с Apache Hadoop в HDInsight — Azure
description: Узнайте, как удаленно запускать задания MapReduce с помощью Curl при использовании Apache Hadoop в HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/13/2020
ms.openlocfilehash: e90dc2c7220caf5bd72b7086adc275934652e150
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98939695"
---
# <a name="run-mapreduce-jobs-with-apache-hadoop-on-hdinsight-using-rest"></a>Запуск заданий MapReduce в среде Apache Hadoop, размещенной в HDInsight, с помощью REST

Узнайте, как использовать Apache Hive WebHCat REST API для выполнения заданий MapReduce на Apache Hadoop кластере HDInsight. Curl используется для демонстрации возможностей взаимодействия с HDInsight с помощью необработанных HTTP-запросов для выполнения заданий MapReduce, их мониторинга и получения их результатов.

> [!NOTE]  
> Если вы уже знакомы с использованием серверов Hadoop на платформе Linux, но не знакомы с HDInsight, ознакомьтесь с документом [Сведения об использовании HDInsight в Linux](../hdinsight-hadoop-linux-information.md).

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Hadoop в HDInsight. См. [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](../hdinsight-hadoop-create-linux-clusters-portal.md).

Любое из следующих:
  * Windows PowerShell или,
  * [Перелистывание](https://curl.haxx.se/) с [JQ](https://stedolan.github.io/jq/)

## <a name="run-a-mapreduce-job"></a>Выполнение задания MapReduce

> [!NOTE]  
> При использовании Curl или любых других средств связи REST с WebHCat нужно проводить аутентификацию запросов с помощью пароля и имени пользователя администратора кластера HDInsight. Имя кластера необходимо использовать в составе универсального кода ресурса (URI), используемого для отправки запросов на сервер.
>
> API-интерфейс REST защищается с помощью [обычной проверки подлинности доступа](https://en.wikipedia.org/wiki/Basic_access_authentication). Чтобы обеспечить безопасную отправку учетных данных на сервер, все запросы следует отправлять с помощью протокола HTTPS.

### <a name="curl"></a>Curl

1. Для простоты использования задайте переменные ниже. Этот пример основан на среде Windows, исправлять ее в соответствии с потребностями среды.

    ```cmd
    set CLUSTERNAME=
    set PASSWORD=
    ```

1. Используйте следующую команду в командной строке, чтобы проверить возможность подключения к кластеру HDInsight:

    ```bash
    curl -u admin:%PASSWORD% -G https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/status
    ```

    Ниже приведены параметры, используемые в этой команде:

   * **-u**: имя пользователя и пароль, используемые для аутентификации запроса.
   * **-G**: указывает, что это запрос GET.

   Начало URI `https://CLUSTERNAME.azurehdinsight.net/templeton/v1` одинаковое для всех запросов.

    Вы должны получить ответ, аналогичный показанному ниже коду JSON.

    ```json
    {"version":"v1","status":"ok"}
    ```

1. Чтобы отправить задание MapReduce, используйте следующую команду. При необходимости измените путь на **JQ** .

    ```cmd
    curl -u admin:%PASSWORD% -d user.name=admin ^
    -d jar=/example/jars/hadoop-mapreduce-examples.jar ^
    -d class=wordcount -d arg=/example/data/gutenberg/davinci.txt -d arg=/example/data/output ^
    https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/mapreduce/jar | ^
    C:\HDI\jq-win64.exe .id
    ```

    Конец универсального кода ресурса (/mapreduce/jar) сообщает WebHCat, что этот запрос запускает задание MapReduce из класса в JAR-файле. Ниже приведены параметры, используемые в этой команде:

   * **-d**: `-G` не используется, поэтому запрос по умолчанию использует метод POST. `-d` задает значения данных, отправляемые в запросе.
     * **user.name**— пользователь, выполняющий команду.
     * **jar**: расположение JAR-файла, содержащего класс для запуска.
     * **class**: класс, содержащий логику MapReduce.
     * **arg**: аргументы, передаваемые в задание MapReduce. В данном случае это входной текстовый файл и каталог, который используется для вывода.

    Эта команда должна возвращать идентификатор задания, который можно использовать для проверки состояния задания: `job_1415651640909_0026` .

1. Чтобы проверить состояние задания, используйте следующую команду. Замените значение на `JOBID` **фактическое** значение, возвращенное на предыдущем шаге. При необходимости измените расположение **JQ** .

    ```cmd
    set JOBID=job_1415651640909_0026

    curl -G -u admin:%PASSWORD% -d user.name=admin https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/jobs/%JOBID% | ^
    C:\HDI\jq-win64.exe .status.state
    ```

### <a name="powershell"></a>PowerShell

1. Для простоты использования задайте переменные ниже. Замените `CLUSTERNAME` фактическим именем кластера. Выполните команду и введите пароль для входа в кластер при появлении запроса.

    ```powershell
    $clusterName="CLUSTERNAME"
    $creds = Get-Credential -UserName admin -Message "Enter the cluster login password"
    ```

1. Используйте следующую команду, чтобы убедиться, что вы можете подключиться к кластеру HDInsight:

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clustername.azurehdinsight.net/templeton/v1/status" `
        -Credential $creds `
        -UseBasicParsing
    $resp.Content
    ```

    Вы должны получить ответ, аналогичный показанному ниже коду JSON.

    ```json
    {"version":"v1","status":"ok"}
    ```

1. Чтобы отправить задание MapReduce, используйте следующую команду:

    ```powershell
    $reqParams = @{}
    $reqParams."user.name" = "admin"
    $reqParams.jar = "/example/jars/hadoop-mapreduce-examples.jar"
    $reqParams.class = "wordcount"
    $reqParams.arg = @()
    $reqParams.arg += "/example/data/gutenberg/davinci.txt"
    $reqparams.arg += "/example/data/output"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/mapreduce/jar" `
       -Credential $creds `
       -Body $reqParams `
       -Method POST `
       -UseBasicParsing
    $jobID = (ConvertFrom-Json $resp.Content).id
    $jobID
    ```

    Конец универсального кода ресурса (/mapreduce/jar) сообщает WebHCat, что этот запрос запускает задание MapReduce из класса в JAR-файле. Ниже приведены параметры, используемые в этой команде:

    * **user.name**— пользователь, выполняющий команду.
    * **jar**: расположение JAR-файла, содержащего класс для запуска.
    * **class**: класс, содержащий логику MapReduce.
    * **arg**: аргументы, передаваемые в задание MapReduce. В данном случае это входной текстовый файл и каталог, который используется для вывода.

   Эта команда должна возвращать идентификатор задания, который можно использовать для проверки состояния задания: `job_1415651640909_0026` .

1. Чтобы проверить состояние задания, используйте следующую команду.

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

### <a name="both-methods"></a>Оба метода

1. Если задание завершено, то возвращается состояние `SUCCEEDED`.

1. После изменения состояния задания на `SUCCEEDED` результаты задания можно получить из хранилища BLOB-объектов Azure. Параметр `statusdir`, передаваемый в запросе, содержит расположение выходного файла. В данном случае это `/example/curl`. Этот адрес задает каталог `/example/curl` для сохранения выходных данных задания, который размещен в хранилище по умолчанию для кластера.

Вы можете вывести список этих файлов и скачать их с помощью [интерфейса командной строки Azure](/cli/azure/install-azure-cli). Дополнительные сведения об использовании Azure CLI для работы с хранилищем BLOB-объектов Azure см. в разделе [Краткое руководство. Создание, скачивание и перечисление больших двоичных объектов с помощью Azure CLI](../../storage/blobs/storage-quickstart-blobs-cli.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительная информация о других способах работы с Hadoop в HDInsight:

* [Использование MapReduce в Apache Hadoop в HDInsight](hdinsight-use-mapreduce.md)
* [Использование Apache Hive с Apache Hadoop в HDInsight](hdinsight-use-hive.md)

Дополнительные сведения об интерфейсе REST, используемом в этой статье, см. в [справочнике по WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference).

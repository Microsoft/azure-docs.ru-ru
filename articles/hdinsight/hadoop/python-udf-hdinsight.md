---
title: Использование определяемых пользователем функций Python с Apache Hive и Apache Pig в Azure HDInsight
description: Узнайте, как использовать определяемые пользователем функции Python стека технологий Apache Hadoop на базе Azure — Apache Hive и Apache Pig в HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.date: 11/15/2019
ms.custom: H1Hack27Feb2017,hdinsightactive, devx-track-python
ms.openlocfilehash: 593b809813f949cd1d0bcc17e1d1b7255ea19130
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944277"
---
# <a name="use-python-user-defined-functions-udf-with-apache-hive-and-apache-pig-in-hdinsight"></a>Использование определяемых пользователем функций Python с Apache Hive и Apache Pig в HDInsight

Узнайте, как использовать определяемые пользователем функции (UDF) Python с Apache Hive и Apache Pig в Apache Hadoop на кластерах Azure HDInsight.

## <a name="python-on-hdinsight"></a><a name="python"></a>Python в HDInsight

По умолчанию на кластерах HDInsight 3.0 и более поздних версиях установлен Python версии 2.7. Apache Hive можно использовать с этой версией Python для потоковой обработки. При этом для передачи данных между Hive и определяемой пользователем функцией используется STDOUT и STDIN.

В состав HDInsight также входят Jython, который представляет собой реализацию Python, написанную на Java. Jython выполняется непосредственно на виртуальная машина Java и не использует потоковую передачу. Jython является рекомендуемым интерпретатором Python при использовании Python с Pig.

## <a name="prerequisites"></a>Предварительные условия

* **Кластер Hadoop в HDInsight**. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](apache-hadoop-linux-tutorial-get-started.md).
* **Клиент SSH**. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).
* [Схема универсального кода ресурса (URI)](../hdinsight-hadoop-linux-information.md#URI-and-scheme) для основного хранилища кластеров. Это может быть `wasb://` для службы хранилища Azure, `abfs://` для Azure Data Lake Storage 2-го поколения или adl://для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure включено безопасное перемещение, URI будет wasbs://.  См. также сведения о [безопасной передаче](../../storage/common/storage-require-secure-transfer.md).
* **Возможное изменение конфигурации хранилища.**  Если используется тип учетной записи хранения, см. раздел [Конфигурация хранилища](#storage-configuration) `BlobStorage` .
* Необязательный параметр.  Если планируется использовать PowerShell, необходимо установить [модуль AZ](/powershell/azure/new-azureps-module-az) .

> [!NOTE]  
> Учетная запись хранения, используемая в этой статье, была включена в службу хранилища Azure с включенной [безопасной](../../storage/common/storage-require-secure-transfer.md) поддержкой и поэтому `wasbs` используется во всей статье.

## <a name="storage-configuration"></a>Конфигурация хранилища

Если используемая учетная запись хранения имеет тип или, никаких действий не требуется `Storage (general purpose v1)` `StorageV2 (general purpose v2)` .  Процесс, описанный в этой статье, будет создавать выходные данные по меньшей мере `/tezstaging` .  Конфигурация Hadoop по умолчанию будет содержать `/tezstaging` в `fs.azure.page.blob.dir` переменной конфигурации `core-site.xml` для службы `HDFS` .  Эта конфигурация приведет к тому, что выходные данные в каталог будут страничными BLOB-объектами, которые не поддерживаются для типа учетной записи хранения `BlobStorage` .  Для использования в `BlobStorage` этой статье удалите `/tezstaging` из `fs.azure.page.blob.dir` переменной конфигурации.  Доступ к конфигурации можно получить из [пользовательского интерфейса Ambari](../hdinsight-hadoop-manage-ambari.md).  В противном случае вы получите сообщение об ошибке: `Page blob is not supported for this account type.`

> [!WARNING]  
> Шаги в этом документе основаны на следующих предположениях:  
>
> * Вы создаете скрипты Python в локальной среде разработки.
> * Вы отправляете скрипты в HDInsight с помощью `scp` команды или предоставленного скрипта PowerShell.
>
> Если вы хотите использовать [Azure Cloud Shell (bash)](../../cloud-shell/overview.md) для работы с HDInsight, необходимо выполнить следующие действия.
>
> * Создать скрипты в среде Cloud Shell.
> * Использовать `scp` для отправки файлов из Cloud Shell в HDInsight.
> * Использовать `ssh` из Cloud Shell для подключения к HDInsight и выполнения примеров.

## <a name="apache-hive-udf"></a><a name="hivepython"></a>Определяемая пользователем функция Apache Hive

Скрипт Python можно использовать в качестве определяемой пользователем функции из Hive через HiveQL с помощью инструкции `TRANSFORM`. Например, следующий запрос HiveQL вызывает файл `hiveudf.py`, хранящийся в учетной записи хранения Azure по умолчанию для кластера.

```hiveql
add file wasbs:///hiveudf.py;

SELECT TRANSFORM (clientid, devicemake, devicemodel)
    USING 'python hiveudf.py' AS
    (clientid string, phoneLabel string, phoneHash string)
FROM hivesampletable
ORDER BY clientid LIMIT 50;
```

Вот что делает данный пример:

1. Инструкция `add file` в начале файла добавляет файл `hiveudf.py` в распределенный кэш, и он становится доступен всем узлам кластера.
2. Инструкция `SELECT TRANSFORM ... USING` выбирает данные из `hivesampletable`. Она также передает параметры clientid, devicemake и devicemodel в скрипт `hiveudf.py`.
3. Предложение `AS` описывает поля, возвращаемые из `hiveudf.py`.

<a name="streamingpy"></a>

### <a name="create-file"></a>Создать файл

В среде разработки создайте текстовый файл с именем `hiveudf.py`. Используйте следующий код в качестве содержимого файла:

```python
#!/usr/bin/env python
import sys
import string
import hashlib

while True:
    line = sys.stdin.readline()
    if not line:
        break

    line = string.strip(line, "\n ")
    clientid, devicemake, devicemodel = string.split(line, "\t")
    phone_label = devicemake + ' ' + devicemodel
    print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])
```

Сценарий выполняет следующие действия:

1. Считывает строку данных из STDIN.
2. Стоящий в конце знак новой строки удаляется с помощью `string.strip(line, "\n ")`.
3. При обработке потока в одной строке будут содержаться все значения, разделенные символом табуляции. Поэтому можно использовать `string.split(line, "\t")` для разделения входящих данных при каждой табуляции, возвращая лишь поля.
4. По завершении обработки результат должен быть записан в поток STDOUT в виде одной строки, с разделенными символами табуляции полями. Например, `print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])`.
5. Цикл `while` повторяется до тех пор, пока считывается `line`.

Выходные данные скрипта представляют собой объединенные входные значения для `devicemake` и `devicemodel`, а также хэш для объединенного значения.

### <a name="upload-file-shell"></a>Передача файла (оболочка)

В приведенных ниже командах замените `sshuser` фактическим именем пользователя, если оно отличается.  Замените `mycluster` фактическим именем кластера.  Убедитесь, что в рабочем каталоге находится файл.

1. Используйте `scp` для копирования файлов в кластер HDInsight. Измените и введите следующую команду:

    ```cmd
    scp hiveudf.py sshuser@mycluster-ssh.azurehdinsight.net:
    ```

2. Используйте SSH, чтобы подключиться к кластеру.  Измените и введите следующую команду:

    ```cmd
    ssh sshuser@mycluster-ssh.azurehdinsight.net
    ```

3. В сеансе SSH добавьте ранее загруженные файлы Python в хранилище кластера.

    ```bash
    hdfs dfs -put hiveudf.py /hiveudf.py
    ```

### <a name="use-hive-udf-shell"></a>Использование UDF Hive (Shell)

1. Чтобы подключиться к Hive, используйте следующую команду в открытом сеансе SSH:

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
    ```

    Эта команда запускает клиент Beeline.

2. Введите следующий запрос `0: jdbc:hive2://headnodehost:10001/>` в командной строке:

   ```hive
   add file wasbs:///hiveudf.py;
   SELECT TRANSFORM (clientid, devicemake, devicemodel)
       USING 'python hiveudf.py' AS
       (clientid string, phoneLabel string, phoneHash string)
   FROM hivesampletable
   ORDER BY clientid LIMIT 50;
   ```

3. После ввода последней строки запустится задание. По завершении задания эта команда возвращает выходные данные следующего вида:

    ```output
    100041    RIM 9650    d476f3687700442549a83fac4560c51c
    100041    RIM 9650    d476f3687700442549a83fac4560c51c
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
    100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
    ```

4. Чтобы выйти из Beeline, введите следующую команду:

    ```hive
    !q
    ```

### <a name="upload-file-powershell"></a>Передача файла (PowerShell)

PowerShell также можно использовать для удаленного запуска запросов на использование Hive. Убедитесь, что рабочий каталог находится в `hiveudf.py` расположении.  Используйте следующий сценарий PowerShell для выполнения запроса Hive, использующего `hiveudf.py` скрипт:

```PowerShell
# Login to your Azure subscription
# Is there an active Azure subscription?
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Revise file path as needed
$pathToStreamingFile = ".\hiveudf.py"

# Get cluster info
$clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
$clusterInfo = Get-AzHDInsightCluster -ClusterName $clusterName
$resourceGroup = $clusterInfo.ResourceGroup
$storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
$container=$clusterInfo.DefaultStorageContainer
$storageAccountKey=(Get-AzStorageAccountKey `
   -ResourceGroupName $resourceGroup `
   -Name $storageAccountName)[0].Value

# Create an Azure Storage context
$context = New-AzStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey $storageAccountKey

# Upload local files to an Azure Storage blob
Set-AzStorageBlobContent `
    -File $pathToStreamingFile `
    -Blob "hiveudf.py" `
    -Container $container `
    -Context $context
```

> [!NOTE]  
> Дополнительные сведения об отправке файлов см. в статье [Отправка данных для заданий Apache Hadoop в HDInsight](../hdinsight-upload-data.md).

#### <a name="use-hive-udf"></a>Использование UDF Hive

```PowerShell
# Script should stop on failures
$ErrorActionPreference = "Stop"

# Login to your Azure subscription
# Is there an active Azure subscription?
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Get cluster info
$clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
$creds=Get-Credential -UserName "admin" -Message "Enter the login for the cluster"

$HiveQuery = "add file wasbs:///hiveudf.py;" +
                "SELECT TRANSFORM (clientid, devicemake, devicemodel) " +
                "USING 'python hiveudf.py' AS " +
                "(clientid string, phoneLabel string, phoneHash string) " +
                "FROM hivesampletable " +
                "ORDER BY clientid LIMIT 50;"

# Create Hive job object
$jobDefinition = New-AzHDInsightHiveJobDefinition `
    -Query $HiveQuery

# For status bar updates
$activity="Hive query"

# Progress bar (optional)
Write-Progress -Activity $activity -Status "Starting query..."

# Start defined Azure HDInsight job on specified cluster.
$job = Start-AzHDInsightJob `
    -ClusterName $clusterName `
    -JobDefinition $jobDefinition `
    -HttpCredential $creds

# Progress bar (optional)
Write-Progress -Activity $activity -Status "Waiting on query to complete..."

# Wait for completion or failure of specified job
Wait-AzHDInsightJob `
    -JobId $job.JobId `
    -ClusterName $clusterName `
    -HttpCredential $creds

# Uncomment the following to see stderr output
<#
Get-AzHDInsightJobOutput `
   -Clustername $clusterName `
   -JobId $job.JobId `
   -HttpCredential $creds `
   -DisplayOutputType StandardError
#>

# Progress bar (optional)
Write-Progress -Activity $activity -Status "Retrieving output..."

# Gets the log output
Get-AzHDInsightJobOutput `
    -Clustername $clusterName `
    -JobId $job.JobId `
    -HttpCredential $creds
```

Результат выполнения задания **Hive** должен выглядеть аналогично следующему примеру:

```output
100041    RIM 9650    d476f3687700442549a83fac4560c51c
100041    RIM 9650    d476f3687700442549a83fac4560c51c
100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
100042    Apple iPhone 4.2.x    375ad9a0ddc4351536804f1d5d0ea9b9
```

## <a name="apache-pig-udf"></a><a name="pigpython"></a>Определяемая пользователем функция Apache Pig

Скрипт Python можно использовать в виде определяемой пользователем функции из Pig с использованием инструкции `GENERATE`. Вы можете запустить скрипт с помощью Jython или CPython.

* Jython работает на виртуальной машине Java и изначально может вызываться из Pig.
* CPython является внешним процессом, поэтому данные из Pig на JVM отправляются в скрипт, выполняющийся в процессе Python. Выходные данные скрипта Python отправляются обратно в Pig.

Чтобы указать интерпретатор Python, используйте `register` при указании ссылки на скрипт Python. Следующие примеры регистрируют скрипты с Pig в качестве `myfuncs`:

* **Чтобы использовать Jython,** сделайте следующее: `register '/path/to/pigudf.py' using jython as myfuncs;`
* **Чтобы использовать язык C Python**: `register '/path/to/pigudf.py' using streaming_python as myfuncs;`

> [!IMPORTANT]  
> При использовании Jython путь к файлу pig_jython может быть локальным путем или WASBS://. Но при использовании CPython необходимо указать ссылку на файл в локальной файловой системе узла, который используется для отправки задания Pig.

После регистрации язык Pig Latin будет одинаковым для обоих примеров:

```pig
LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
LOG = FILTER LOGS by LINE is not null;
DETAILS = FOREACH LOG GENERATE myfuncs.create_structure(LINE);
DUMP DETAILS;
```

Вот что делает данный пример:

1. Первая строка загружает образец файла данных `sample.log` в `LOGS`. Она также определяет каждую запись как массив символов `chararray`.
2. Следующая строка отфильтровывает все пустые значения, сохраняя результат работы в `LOG`.
3. Затем выполняется итерация по записям в `LOG` и используется инструкция `GENERATE` для вызова метода `create_structure`, содержащегося в скрипте Python или Jython, загруженном как `myfuncs`. `LINE` используется для передачи текущей записи в функцию.
4. Наконец, выходные данные сбрасываются в поток STDOUT командой `DUMP`. После завершения операции эта команда выведет результат.

### <a name="create-file"></a>Создать файл

В среде разработки создайте текстовый файл с именем `pigudf.py`. Используйте следующий код в качестве содержимого файла:

<a name="streamingpy"></a>

```python
# Uncomment the following if using C Python
#from pig_util import outputSchema


@outputSchema("log: {(date:chararray, time:chararray, classname:chararray, level:chararray, detail:chararray)}")
def create_structure(input):
    if (input.startswith('java.lang.Exception')):
        input = input[21:len(input)] + ' - java.lang.Exception'
    date, time, classname, level, detail = input.split(' ', 4)
    return date, time, classname, level, detail
```

В примере Pig Latin `LINE` входные данные определяются как массив символов CharArray, так как для входных данных не существует соответствующей схемы. Скрипт Python выполняет преобразование данных в согласованную схему на выходе.

1. Инструкция `@outputSchema` задает формат данных, в котором они возвращаются в Pig. В данном случае это **data bag**, являющийся типом данных Pig. Корзина содержит следующие поля, все они имеют тип "Массив строк" (строки):

   * date — дата создания записи журнала;
   * date — время создания записи журнала;
   * classname — имя класса, для которого создана запись;
   * level — уровень журналирования;
   * detail — подробная информация о записи журнала.

2. Затем `def create_structure(input)` определяет функцию, в которую Pig отправляет строковые элементы.

3. Данные для примера, `sample.log`, в основном соответствуют схеме даты, времени, имени класса, уровня и подробной информации. Однако он содержит несколько строк, начинающихся с `*java.lang.Exception*`. Эти строки должны быть изменены в соответствии со схемой. Инструкция `if` проверяет на наличие таких строк, затем обрабатывает входные данные, переставляя строку `*java.lang.Exception*` в конец, формируя данные в соответствии с ожидаемой схемой.

4. Затем команда `split` используется для разделения данных по первым четырем символам пробела. Выходным данным присваиваются значения `date`, `time`, `classname`, `level` и `detail`.

5. И результаты возвращаются в Pig.

Когда данные возвращаются в Pig, они имеют согласованную схему, определенную инструкцией `@outputSchema`.

### <a name="upload-file-shell"></a>Передача файла (оболочка)

В приведенных ниже командах замените `sshuser` фактическим именем пользователя, если оно отличается.  Замените `mycluster` фактическим именем кластера.  Убедитесь, что в рабочем каталоге находится файл.

1. Используйте `scp` для копирования файлов в кластер HDInsight. Измените и введите следующую команду:

    ```cmd
    scp pigudf.py sshuser@mycluster-ssh.azurehdinsight.net:
    ```

2. Используйте SSH, чтобы подключиться к кластеру.  Измените и введите следующую команду:

    ```cmd
    ssh sshuser@mycluster-ssh.azurehdinsight.net
    ```

3. В сеансе SSH добавьте ранее загруженные файлы Python в хранилище кластера.

    ```bash
    hdfs dfs -put pigudf.py /pigudf.py
    ```

### <a name="use-pig-udf-shell"></a>Использование UDF Pig (Shell)

1. Чтобы подключиться к Pig, используйте следующую команду в открытом сеансе SSH:

    ```bash
    pig
    ```

2. В окне запроса `grunt>` введите следующие операторы:

   ```pig
   Register wasbs:///pigudf.py using jython as myfuncs;
   LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
   LOG = FILTER LOGS by LINE is not null;
   DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
   DUMP DETAILS;
   ```

3. После ввода указанной строки должно запуститься задание. По завершении задания эта команда возвращает выходные данные следующего вида:

    ```output
    ((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
    ((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
    ((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
    ((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
    ((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))
    ```

4. Используйте `quit` для выхода из оболочки Grunt, а затем следующую команду для изменения файла pigudf.py в локальной файловой системе:

    ```bash
    nano pigudf.py
    ```

5. Войдите в редактор и раскомментируйте следующую строку, удалив символ `#` в начале строки.

    ```bash
    #from pig_util import outputSchema
    ```

    Эта строка изменяет сценарий Python для работы с C Python вместо Jython. Закончив вносить изменения, нажмите клавиши **CTRL+X**, чтобы выйти из редактора. Выберите **Y**, а затем **введите** , чтобы сохранить изменения.

6. Используйте команду `pig` , чтобы снова запустить оболочку. При появлении запроса `grunt>` введите следующие инструкции, чтобы запустить сценарий Python с помощью интерпретатора CPython.

   ```pig
   Register 'pigudf.py' using streaming_python as myfuncs;
   LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
   LOG = FILTER LOGS by LINE is not null;
   DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
   DUMP DETAILS;
   ```

    Когда это задание будет выполнено, вы увидите такой же результат, как при запуске сценария с помощью Jython.

### <a name="upload-file-powershell"></a>Передача файла (PowerShell)

PowerShell также можно использовать для удаленного запуска запросов на использование Hive. Убедитесь, что рабочий каталог находится в `pigudf.py` расположении.  Используйте следующий сценарий PowerShell для выполнения запроса Hive, использующего `pigudf.py` скрипт:

```PowerShell
# Login to your Azure subscription
# Is there an active Azure subscription?
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"

# Revise file path as needed
$pathToJythonFile = ".\pigudf.py"


# Get cluster info
$clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
$clusterInfo = Get-AzHDInsightCluster -ClusterName $clusterName
$resourceGroup = $clusterInfo.ResourceGroup
$storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
$container=$clusterInfo.DefaultStorageContainer
$storageAccountKey=(Get-AzStorageAccountKey `
   -ResourceGroupName $resourceGroup `
   -Name $storageAccountName)[0].Value

# Create an Azure Storage context
$context = New-AzStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey $storageAccountKey

# Upload local files to an Azure Storage blob
Set-AzStorageBlobContent `
    -File $pathToJythonFile `
    -Blob "pigudf.py" `
    -Container $container `
    -Context $context
```

### <a name="use-pig-udf-powershell"></a>Использование UDF Pig (PowerShell)

> [!NOTE]  
> При удаленной отправке задания с помощью PowerShell нельзя использовать CPython в качестве интерпретатора.

PowerShell также можно использовать для запуска заданий Pig Latin. Чтобы выполнить задание Pig Latin, использующее `pigudf.py` скрипт, используйте следующий сценарий PowerShell:

```PowerShell
# Script should stop on failures
$ErrorActionPreference = "Stop"

# Login to your Azure subscription
# Is there an active Azure subscription?
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# Get cluster info
$clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
$creds=Get-Credential -UserName "admin" -Message "Enter the login for the cluster"


$PigQuery = "Register wasbs:///pigudf.py using jython as myfuncs;" +
            "LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);" +
            "LOG = FILTER LOGS by LINE is not null;" +
            "DETAILS = foreach LOG generate myfuncs.create_structure(LINE);" +
            "DUMP DETAILS;"

# Create Pig job object
$jobDefinition = New-AzHDInsightPigJobDefinition -Query $PigQuery

# For status bar updates
$activity="Pig job"

# Progress bar (optional)
Write-Progress -Activity $activity -Status "Starting job..."

# Start defined Azure HDInsight job on specified cluster.
$job = Start-AzHDInsightJob `
    -ClusterName $clusterName `
    -JobDefinition $jobDefinition `
    -HttpCredential $creds

# Progress bar (optional)
Write-Progress -Activity $activity -Status "Waiting for the Pig job to complete..."

# Wait for completion or failure of specified job
Wait-AzHDInsightJob `
    -Job $job.JobId `
    -ClusterName $clusterName `
    -HttpCredential $creds

# Uncomment the following to see stderr output
<#
Get-AzHDInsightJobOutput `
    -Clustername $clusterName `
    -JobId $job.JobId `
    -HttpCredential $creds `
    -DisplayOutputType StandardError
#>

# Progress bar (optional)
Write-Progress -Activity $activity "Retrieving output..."

# Gets the log output
Get-AzHDInsightJobOutput `
    -Clustername $clusterName `
    -JobId $job.JobId `
    -HttpCredential $creds
```

Результат выполнения задания **Pig** должен выглядеть аналогично следующим данным:

```output
((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))
```

## <a name="troubleshooting"></a><a name="troubleshooting"></a>Устранение неполадок

### <a name="errors-when-running-jobs"></a>Ошибки при выполнении заданий

При выполнении задания hive может возникнуть ошибка, аналогичная приведенной ниже:

```output
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: [Error 20001]: An error occurred while reading or writing to your custom script. It may have crashed with an error.
```

Эта проблема может быть вызвана символами окончания строк в файле Python. Многие редакторы Windows по умолчанию используют символы CRLF, но в приложениях Linux обычно ожидается использование символа LF.

Вы можете использовать следующие команды PowerShell для удаления символов CR перед передачей файла в HDInsight:

[!code-powershell[main](../../../powershell_scripts/hdinsight/run-python-udf/run-python-udf.ps1?range=148-150)]

### <a name="powershell-scripts"></a>Сценарии PowerShell

Оба примера скриптов PowerShell, используемых для запуска примеров, содержат закомментированную строку, которая отображает вывод ошибок для задания. Если ожидаемые выходные данные для задания не отображаются, раскомментируйте следующую строку и проверьте, указывает ли сведения об ошибке на проблему.

[!code-powershell[main](../../../powershell_scripts/hdinsight/run-python-udf/run-python-udf.ps1?range=135-139)]

Сведения об ошибках (STDERR) и результат выполнения задания (STDOUT) также записываются в хранилище HDInsight.

| Для данного задания... | Смотрите эти файлы в контейнере |
| --- | --- |
| Hive |/HivePython/stderr<p>/HivePython/stdout |
| Pig, |/PigPython/stderr<p>/PigPython/stdout |

## <a name="next-steps"></a><a name="next"></a>Следующие шаги

Если вам нужно загрузить модули Python, которые не поставляются по умолчанию, см. статью [How to deploy a Python module to Windows Azure HDInsight](/archive/blogs/benjguin/how-to-deploy-a-python-module-to-windows-azure-hdinsight) (Как развернуть модуль Python в Windows Azure HDInsight).

Сведения о других способах использования Pig и Hive и дополнительную информацию об использовании MapReduce см. в следующих документах:

* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hdinsight-use-hive.md)
* [Использование MapReduce в Hadoop в HDInsight](hdinsight-use-mapreduce.md)
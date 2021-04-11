---
title: Создайте свою первую фабрику данных (REST)
description: В этом руководстве вы создадите образец конвейера фабрики данных Azure с помощью REST API фабрики данных.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: tutorial
ms.date: 11/01/2017
ms.openlocfilehash: eb9ce65985521fb3f82032f04aec62538a157639
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104785468"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-data-factory-rest-api"></a>Руководство. Создание первой фабрики данных Azure с помощью REST API фабрики данных
> [!div class="op_single_selector"]
> * [Обзор и предварительные требования](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Шаблон Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)
>
>


> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию службы "Фабрика данных", ознакомьтесь с [кратким руководством по созданию фабрики данных с помощью службы "Фабрика данных Azure"](../quickstart-create-data-factory-rest-api.md).

Из этой статьи вы узнаете, как создать первую фабрику данных Azure с помощью REST API фабрики данных. Чтобы выполнить приведенные здесь инструкции с помощью других средств или пакетов SDK, выберите в раскрывающемся списке один из доступных вариантов.

В этом руководстве конвейеру доступно одно действие — **действие Hive HDInsight**. Это действие запускает сценарий Hive в кластере HDInsight Azure, который преобразует входные данные в выходные. Конвейер запускается раз в месяц по расписанию. Время начала и окончания запуска также указаны.

> [!NOTE]
> В этой статье рассматриваются не все REST API. Полную документацию по командлетам фабрики данных см. в [справочнике по REST API фабрики данных](/rest/api/datafactory/).
> 
> Конвейер может содержать сразу несколько действий. Два действия можно объединить в цепочку (выполнить одно действие вслед за другим), настроив выходной набор данных одного действия как входной набор данных другого действия. Дополнительные сведения см. в разделе [Несколько действий в конвейере](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).


## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Прочтите [обзорную статью](data-factory-build-your-first-pipeline.md) и выполните **предварительные требования** .
* Установите на компьютер программу [curl](https://curl.haxx.se/dlwiz/) . Она будет использоваться с командами REST для создания фабрики данных.
* Следуя инструкциям в [этой статье](../../active-directory/develop/howto-create-service-principal-portal.md) , выполните следующее:
  1. Создайте веб-приложение с именем **ADFGetStartedApp** в Azure Active Directory.
  2. Получите **идентификатор клиента** и **секретный ключ**.
  3. Получите значение для **tenant_id**.
  4. Назначьте приложение **ADFGetStartedApp** роли **участника фабрики данных**.
* Установите [Azure PowerShell](/powershell/azure/).
* Откройте **PowerShell** и выполните приведенные ниже команды. Не закрывайте Azure PowerShell, пока выполняются описанные в учебнике инструкции. Если закрыть и снова открыть это окно, то придется вновь выполнять эти команды.
  1. Выполните командлет **Connect-AzAccount** и введите имя пользователя и пароль, которые используются для входа на портал Azure.
  2. Выполните командлет **Get-AzSubscription**, чтобы просмотреть все подписки для этой учетной записи.
  3. Выполните команду **Get-AzSubscription-SubscriptionName NameOfAzureSubscription | Set-AzContext**, чтобы выбрать подписку, с которой вы собираетесь работать. Замените **NameOfAzureSubscription** именем своей подписки Azure.
* Создайте группу ресурсов Azure с именем **ADFTutorialResourceGroup** , выполнив следующую команду в PowerShell:

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

   Некоторые действия, описанные в этом учебнике, предполагают, что вы используете группу ресурсов с именем ADFTutorialResourceGroup. Если вы используете другую группу ресурсов, укажите ее имя вместо ADFTutorialResourceGroup.

## <a name="create-json-definitions"></a>Создание определений JSON
В папке, где находится файл curl.exe, создайте следующие JSON-файлы.

### <a name="datafactoryjson"></a>datafactory.json
> [!IMPORTANT]
> Имя должно быть глобально уникальным, поэтому может потребоваться добавить к ADFCopyTutorialDF префикс или суффикс.
>
>

```JSON
{
    "name": "FirstDataFactoryREST",
    "location": "WestUS"
}
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> Замените значения **accountname** и **accountkey** на имя вашей учетной записи хранения Azure и ее ключ. Сведения о том, как получить ключ доступа к хранилищу, см. в статье [Управление ключами доступа к хранилищу](../../storage/common/storage-account-keys-manage.md).
>
>

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="hdinsightondemandlinkedservicejson"></a>hdinsightondemandlinkedservice.json

```JSON
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

В следующей таблице приведены описания свойств JSON, используемых в этом фрагменте кода.

| Свойство | Описание |
|:--- |:--- |
| clusterSize |Размер кластера HDInsight. |
| timeToLive |Указывает, сколько времени может простаивать кластер HDInsight, прежде чем он будет удален. |
| linkedServiceName |Указывает имя учетной записи хранения, в которой будут храниться журналы, создаваемые HDInsight. |

Обратите внимание на следующие моменты.

* С помощью приведенного выше JSON-файла фабрика данных создает кластер HDInsight **под управлением Linux**. Дополнительные сведения см. в разделе [Связанная служба Azure HDInsight по запросу](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service).
* Вместо кластера HDInsight по запросу можно использовать **собственный кластер HDInsight**. См. сведения о [связанной службе Azure HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service).
* Кластер HDInsight создает **контейнер по умолчанию** в хранилище BLOB-объектов, указанном в коде JSON (**linkedServiceName**). При удалении кластера HDInsight этот контейнер не удаляется. В этом весь замысел. Если используется связанная служба HDInsight по запросу, кластер HDInsight создается всякий раз, когда обрабатывается срез данных (если не используется динамический кластер **timeToLive**), после чего кластер удаляется.

    По мере обработки новых срезов количество контейнеров в хранилище BLOB-объектов будет увеличиваться. Если эти контейнеры не используются для устранения неполадок с заданиями, удалите их — это позволит сократить расходы на хранение. Имена контейнеров указаны в формате adf **имя_фабрики_данных**-**имя_связанной_службы**-метка_даты_и_времени. Для удаления контейнеров в хранилище BLOB-объектов Azure используйте такие инструменты, как [Обозреватель службы хранилища Microsoft Azure](https://storageexplorer.com/).

Дополнительные сведения см. в разделе [Связанная служба Azure HDInsight по запросу](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service).

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

Приведенный выше JSON-файл определяет набор данных с именем **AzureBlobInput**, представляющий входные данные для действия в конвейере. Кроме того, файл указывает, что входные данные размещаются в контейнере больших двоичных объектов **adfgetstarted** и в папке **inputdata**.

В следующей таблице приведены описания свойств JSON, используемых в этом фрагменте кода.

| Свойство | Описание |
|:--- |:--- |
| type |Для свойства типа задано значение AzureBlob, так как данные хранятся в хранилище BLOB-объектов Azure. |
| linkedServiceName |Ссылается на созданную ранее службу StorageLinkedService. |
| fileName |Это необязательное свойство. Если это свойство не указано, выбираются все файлы из папки folderPath. В этом случае обрабатывается только файл input.log. |
| type |Файлы журнала представлены в текстовом формате, поэтому мы используем значение TextFormat. |
| columnDelimiter |Столбцы в файлах журнала разделяются запятыми (,). |
| frequency/interval |Для свойства frequency задано значение Month, а для свойства interval — значение 1. Это означает, что срезы входных данных доступны ежемесячно. |
| external |Это свойство имеет значение true, если входные данные не создаются службой фабрики данных. |

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "adfgetstarted/partitioneddata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        }
    }
}
```

JSON-файл определяет набор данных с именем **AzureBlobOutput**, представляющий выходные данные для действия в конвейере. Кроме того, файл указывает, что результаты хранятся в контейнере больших двоичных объектов **adfgetstarted** и в папке **partitioneddata**. В разделе **availability** указывается частота, с которой будет создаваться выходной набор данных (ежемесячно).

### <a name="pipelinejson"></a>pipeline.json
> [!IMPORTANT]
> Замените свойство **storageaccountname** именем вашей учетной записи хранения Azure.
>
>

```JSON
{
    "name": "MyFirstPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [{
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                "scriptLinkedService": "AzureStorageLinkedService",
                "defines": {
                    "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                    "partitionedtable": "wasb://adfgetstarted@<storageaccountname>t.blob.core.windows.net/partitioneddata"
                }
            },
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "policy": {
                "concurrency": 1,
                "retry": 3
            },
            "scheduler": {
                "frequency": "Month",
                "interval": 1
            },
            "name": "RunSampleHiveActivity",
            "linkedServiceName": "HDInsightOnDemandLinkedService"
        }],
        "start": "2017-07-10T00:00:00Z",
        "end": "2017-07-11T00:00:00Z",
        "isPaused": false
    }
}
```

Этот фрагмент JSON-файла создает конвейер из одного действия, использующего Hive для обработки данных в кластере HDInsight.

Файл **partitionweblogs.hql** скрипта Hive хранится в учетной записи хранения Azure (указывается с помощью свойства scriptLinkedService с именем **StorageLinkedService**) в папке **script** в контейнере **adfgetstarted**.

Раздел **defines** указывает параметров среды выполнения, которые будут переданы в сценарий Hive в качестве значений конфигурации Hive (например, ${hiveconf:inputtable}, ${hiveconf:partitionedtable}).

Активный период конвейера задается с помощью свойств **start** и **end**.

В действии JSON укажите, что скрипт Hive будет выполняться в среде вычислений, указанной в свойстве **linkedServiceName**, — **HDInsightOnDemandLinkedService**.

> [!NOTE]
> Сведения о свойствах JSON, используемых в предыдущем примере, см. в разделе "Конвейер JSON" статьи [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md).
>
>

## <a name="set-global-variables"></a>Настройка глобальных переменных
Выполните следующие команды в Azure PowerShell, подставив собственные значения.

> [!IMPORTANT]
> Инструкции по получению значений для параметров client_id, client_secret, tenant и subscription_id см. в разделе [Предварительные требования](#prerequisites).
>
>

```powershell
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
$adf = "FirstDataFactoryREST"
```


## <a name="authenticate-with-aad"></a>Проверка подлинности с помощью AAD

```powershell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken)
```


## <a name="create-data-factory"></a>Создание фабрики данных
На этом шаге вы создадите фабрику данных Azure с именем **FirstDataFactoryREST**. Фабрика данных может иметь один или несколько конвейеров. Конвейер может содержать одно или несколько действий. Это может быть, например, действие копирования данных из исходного хранилища в целевое или действие HDInsight Hive для выполнения скрипта Hive, преобразующего данные. Чтобы создать фабрику данных, выполните следующие команды:

1. Назначьте команду переменной с именем **cmd**.

    Указываемое здесь имя фабрики данных (ADFCopyTutorialDF) должно соответствовать имени, указанному в **datafactory.json**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@datafactory.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/FirstDataFactoryREST?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания фабрики данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.

    ```powershell
    Write-Host $results
    ```

Обратите внимание на следующие моменты.

* Имя фабрики данных Azure должно быть глобально уникальным. Если в результатах отобразится сообщение об ошибке **Имя FirstDataFactoryREST фабрики данных недоступно**, сделайте следующее:
  1. Измените имя (например, на ваше_имя_FirstDataFactoryREST) в файле **datafactory.json** . Ознакомьтесь с разделом [Фабрика данных — правила именования](data-factory-naming-rules.md) , чтобы узнать о правилах именования артефактов фабрики данных.
  2. В первой команде, где переменной **$cmd** присваивается значение, замените FirstDataFactoryREST на новое имя и выполните команду.
  3. Выполните следующие две команды, чтобы вызвать REST API для создания фабрики данных и вывода результатов операции.
* Чтобы создать экземпляры фабрики данных, вы должны быть администратором или участником подписки Azure.
* В будущем имя фабрики данных может быть зарегистрировано в качестве DNS-имени и, следовательно, стать отображаемым.
* Если возникнет ошибка, сделайте следующее. **This subscription is not registered to use namespace Microsoft.DataFactory** (Подписка не зарегистрирована для использования пространства имен Microsoft.DataFactory), выполните одно из следующих действий и повторите попытку публикации:

  * Чтобы зарегистрировать поставщик фабрики данных Azure, выполните следующую команду в Azure PowerShell:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

      Чтобы убедиться, что поставщик фабрики данных зарегистрирован, выполните следующую команду:
    ```powershell
    Get-AzResourceProvider
    ```
  * Войдите на [портал Azure](https://portal.azure.com) с использованием подписки Azure и откройте колонку фабрики данных или создайте на портале фабрику данных. Поставщик будет зарегистрирован автоматически.

Прежде чем создавать конвейер, необходимо создать несколько сущностей фабрики данных. Сначала создайте связанные службы, чтобы связать хранилища данных и вычисления со своим хранилищем данных, и определите входные и выходные наборы данных, которые будут представлять данные в связанных хранилищах.

## <a name="create-linked-services"></a>Создание связанных служб
На этом шаге вы свяжете учетную запись службы хранилища Azure и используемый по запросу кластер Azure HDInsight с фабрикой данных. В этом примере учетная запись хранения Azure содержит входные и выходные данные для конвейера. Для выполнения скрипта Hive, указанного в действии конвейера, в этом примере используется связанная служба HDInsight.

### <a name="create-azure-storage-linked-service"></a>Создание связанной службы хранения Azure
На этом шаге вы свяжете учетную запись хранения Azure с фабрикой данных. В целях данного руководства используйте одну и ту же учетную запись хранения Azure для хранения входных и выходных данных и файла скрипта HQL.

1. Назначьте команду переменной с именем **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data “@azurestoragelinkedservice.json” https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания связанной службы в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.

    ```powershell
    Write-Host $results
    ```

### <a name="create-azure-hdinsight-linked-service"></a>Создание связанной службы Azure HDInsight
На этом шаге вы свяжете используемый по запросу кластер HDInsight с фабрикой данных. Кластер HDInsight автоматически создается в среде выполнения и удаляется после завершения обработки и простоя в течение указанного времени. Вместо используемого по запросу кластера HDInsight можно использовать собственный кластер HDInsight. Дополнительные сведения см. в статье [Связанные службы вычислений](data-factory-compute-linked-services.md).

1. Назначьте команду переменной с именем **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@hdinsightondemandlinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/hdinsightondemandlinkedservice?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания связанной службы в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.

    ```powershell
    Write-Host $results
    ```

## <a name="create-datasets"></a>Создание наборов данных
На этом шаге вы создадите наборы данных, которые представляют входные и выходные данные для обработки Hive. Эти наборы данных ссылаются на службу **StorageLinkedService** , созданную ранее в ходе работы с этим руководством. Точки связанной службы указывают на учетную запись хранения Azure, а наборы данных указывают контейнер, папку и имя файла в хранилище, в котором содержатся входные и выходные данные.

### <a name="create-input-dataset"></a>Создание входного набора данных
Теперь создайте входной набор данных, представляющий входные данные, которые хранятся в хранилище BLOB-объектов Azure.

1. Назначьте команду переменной с именем **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@inputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobInput?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания набора данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.

    ```powershell
    Write-Host $results
    ```

### <a name="create-output-dataset"></a>Создание выходного набора данных
Теперь создайте выходной набор данных, представляющий выходные данные, которые хранятся в хранилище BLOB-объектов Azure.

1. Назначьте команду переменной с именем **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobOutput?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания набора данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.

    ```powershell
    Write-Host $results
    ```

## <a name="create-pipeline"></a>Создание конвейера
На этом шаге вы создадите свой первый конвейер с действием **HDInsightHive** . Срез входных данных создается ежемесячно (frequency: Month, interval: 1), срез выходных данных создается ежемесячно, свойство scheduler для действия также указывается ежемесячно. Параметры выходного набора данных (outputs) и планировщика действия (scheduler) должны совпадать. В настоящее время расписание активируется с помощью выходного набора данных, поэтому его необходимо создать, даже если действие не создает никаких выходных данных. Если действие не принимает никаких входных данных, входной набор данных можно не создавать.

Убедитесь, что файл **input.log** отображается в папке **adfgetstarted/inputdata** в хранилище BLOB-объектов Azure, и выполните следующую команду, чтобы развернуть конвейер. Так как время в свойствах **start** и **end** задано в прошлом, а для свойства **isPaused** задано значение false, конвейер (действие в конвейере) запускается сразу после развертывания.

1. Назначьте команду переменной с именем **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@pipeline.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datapipelines/MyFirstPipeline?api-version=2015-10-01};
    ```
2. Выполните команду с использованием командлета **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Просмотрите результаты. После успешного создания набора данных в **результатах** появится JSON-файл. В противном случае отобразится сообщение об ошибке.

    ```powershell
    Write-Host $results
    ```
4. Поздравляем! Вы создали свой первый конвейер с помощью Azure PowerShell!

## <a name="monitor-pipeline"></a>Отслеживание конвейера
На этом шаге используется REST API фабрики данных, чтобы отслеживать срезы, созданные конвейером.

```powershell
$ds ="AzureBlobOutput"

$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=1970-01-01T00%3a00%3a00.0000000Z"&"end=2016-08-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};

$results2 = Invoke-Command -scriptblock $cmd;

IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

> [!IMPORTANT]
> Создание используемого по требованию кластера HDInsight обычно занимает некоторое время (около 20 минут). Таким образом, конвейер обработает срез **примерно через 30 минут** .
>
>

Запускайте командлет Invoke-Command, пока не увидите срез в состоянии **Готово** или **Сбой**. Когда срез перейдет в состояние "Готово", проверьте выходные данные в папке **partitioneddata** контейнера **adfgetstarted** в хранилище BLOB-объектов.  Создание кластера HDInsight по требованию занимает некоторое время.

![выходные данные](./media/data-factory-build-your-first-pipeline-using-rest-api/three-ouptut-files.png)

> [!IMPORTANT]
> В случае успешной обработки среза входной файл удаляется. Если вы хотите повторно обработать срез или еще раз выполнить инструкции из руководства, передайте входной файл (input.log) в папку inputdata в контейнере adfgetstarted.
>
>

Для отслеживания срезов и устранения возникших проблем можно использовать портал Azure. Дополнительные сведения см. в разделе [Отслеживание конвейера](data-factory-monitor-manage-pipelines.md).

## <a name="summary"></a>Сводка
Следуя инструкциям из этого руководства, вы создали фабрику данных Azure для обработки данных путем выполнения сценария Hive в кластере Hadoop HDInsight. Вы использовали редактор фабрики данных на портале Azure для выполнения следующих действий:

1. создание **фабрики данных Azure**;
2. создание двух **связанных служб**.
   1. **Служба хранилища Azure** — связанная служба для связывания хранилища BLOB-объектов Azure, которое содержит входные и выходные файлы, с фабрикой данных.
   2. **Azure HDInsight** — связанная служба по запросу для связывания кластера HDInsight Hadoop с фабрикой данных. Фабрика данных Azure своевременно создает кластер HDInsight Hadoop для обработки входных данных и генерирования выходных данных.
3. Создание двух **наборов данных**, которые описывают входные и выходные данные для действия HDInsight Hive в конвейере.
4. Создание **конвейера** с действием **HDInsight Hive**.

## <a name="next-steps"></a>Дальнейшие действия
В этой статье описывается создание конвейера с помощью действия преобразования (действие HDInsight), которое по требованию выполняет сценарий Hive в кластере Azure HDInsight. Сведения о том, как копировать данные из хранилища BLOB-объектов Azure в SQL Azure с помощью действия копирования, см. в статье [Копирование данных из хранилища BLOB-объектов Azure в базу данных SQL с помощью фабрики данных](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="see-also"></a>См. также:
| Раздел | Description |
|:--- |:--- |
| [Справочник по REST API фабрики данных](/rest/api/datafactory/) |См. полную документацию по командлетам фабрики данных. |
| [Конвейеры](data-factory-create-pipelines.md) |Эта статья поможет вам понять сущность конвейеров и действий в фабрике данных Azure, а также научиться с их помощью создавать комплексные рабочие процессы, управляемые данными, для конкретных бизнес-сценариев. |
| [Наборы данных](data-factory-create-datasets.md) |Эта статья поможет вам понять, что такое наборы данных в фабрике данных Azure. |
| [Планирование и выполнение](data-factory-scheduling-and-execution.md) |Здесь объясняются аспекты планирования и исполнения в модели приложений фабрики данных. |
| [Мониторинг конвейеров фабрики данных Azure и управление ими с помощью нового приложения по мониторингу и управлению](data-factory-monitor-manage-app.md) |В этой статье описывается мониторинг и отладка конвейеров, а также управление ими с помощью приложения мониторинга и управления. |

---
title: Преобразование данных в виртуальной сети Azure с помощью Hive
description: С помощью Azure PowerShell вы создадите конвейер Фабрики данных, который преобразует данные, используя действие Hive в кластере HDInsight, находящемся в виртуальной сети Azure (VNet).
ms.service: data-factory
author: nabhishek
ms.author: abnarain
ms.topic: tutorial
ms.custom: seo-dt-2019
ms.date: 01/22/2018
ms.openlocfilehash: 85f51fd52ce3224b37c27cea6c49a8a386fbea2c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100377777"
---
# <a name="transform-data-in-azure-virtual-network-using-hive-activity-in-azure-data-factory"></a>Преобразование данных в виртуальной сети Azure с помощью действия Hive в фабрике данных Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве с помощью Azure PowerShell вы создадите конвейер Фабрики данных, который преобразует данные, используя действие Hive в кластере HDInsight, находящемся в виртуальной сети Azure (VNet). В этом руководстве вы выполните следующие шаги:

> [!div class="checklist"]
> * Создали фабрику данных. 
> * Создали и настроили среду выполнения интеграции с самостоятельным размещением.
> * Создали и развернули эти связанные службы.
> * Создали и развернули конвейер, который содержит действие Hive.
> * Запуск конвейера.
> * Мониторинг конвейера 
> * Проверили выходные данные. 

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Учетная запись хранения Azure.** Создайте скрипт Hive и отправьте его в хранилище Azure. Выходные данные скрипта Hive хранятся в этой учетной записи хранения. В этом примере кластер HDInsight использует эту учетную запись хранения Azure в качестве основного хранилища. 
- **Виртуальная сеть Azure.** Если у вас нет виртуальной сети Azure, создайте ее, выполнив [эти инструкции](../virtual-network/quick-create-portal.md). В этом примере HDInsight находится в виртуальной сети Azure. Ниже приведен образец конфигурации виртуальной сети Azure. 

    ![Создание виртуальной сети](media/tutorial-transform-data-using-hive-in-vnet/create-virtual-network.png)
- **Кластер HDInsight.** Создайте кластер HDInsight и присоедините его к виртуальной сети, созданной на предыдущем шаге, следуя указаниям в статье [Расширение возможностей HDInsight с помощью виртуальной сети Azure](../hdinsight/hdinsight-plan-virtual-network-deployment.md). Ниже приведен образец конфигурации HDInsight в виртуальной сети. 

    ![HDInsight в виртуальной сети](media/tutorial-transform-data-using-hive-in-vnet/hdinsight-in-vnet-configuration.png)
- **Azure PowerShell**. Следуйте инструкциям по [установке и настройке Azure PowerShell](/powershell/azure/install-Az-ps).

### <a name="upload-hive-script-to-your-blob-storage-account"></a>Отправка скрипта Hive в вашу учетную запись хранилища BLOB-объектов

1. Создайте файл Hive SQL с именем **hivescript.hql** со следующим содержимым:

   ```sql
   DROP TABLE IF EXISTS HiveSampleOut; 
   CREATE EXTERNAL TABLE HiveSampleOut (clientid string, market string, devicemodel string, state string)
   ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' 
   STORED AS TEXTFILE LOCATION '${hiveconf:Output}';

   INSERT OVERWRITE TABLE HiveSampleOut
   Select 
       clientid,
       market,
       devicemodel,
       state
   FROM hivesampletable
   ```
2. В хранилище BLOB-объектов Azure создайте контейнер с именем **adftutorial**, если он не существует.
3. Создайте папку с именем **hivescripts**.
4. Отправьте файл **hivescript.hql** во вложенную папку **hivescripts**.

  

## <a name="create-a-data-factory"></a>Создание фабрики данных


1. Укажите имя группы ресурсов. В этом руководстве вы создадите группу ресурсов. Но при необходимости можно использовать существующую группу ресурсов. 

    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup" 
    ```
2. Укажите имя фабрики данных. Оно должно быть глобально уникальным.

    ```powershell
    $dataFactoryName = "MyDataFactory09142017"
    ```
3. Укажите имя конвейера. 

    ```powershell
    $pipelineName = "MyHivePipeline" # 
    ```
4. Укажите имя для локальной среды выполнения интеграции. Локальная среда выполнения интеграции будет необходима, когда фабрике данных потребуется доступ к ресурсам (например, к базе данных SQL Azure) в VNet. 
    ```powershell
    $selfHostedIntegrationRuntimeName = "MySelfHostedIR09142017" 
    ```
2. Запустите **PowerShell**. Не закрывайте Azure PowerShell, пока выполняются описанные в этом кратком руководстве инструкции. Если закрыть и снова открыть это окно, то придется вновь выполнять эти команды. Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (служба хранилища Azure, база данных SQL Azure и т. д.) и вычисления (HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.

    Выполните следующую команду и введите имя пользователя и пароль, которые используются для входа на портал Azure.
        
    ```powershell
    Connect-AzAccount
    ```        
    Чтобы просмотреть все подписки для этой учетной записи, выполните следующую команду:

    ```powershell
    Get-AzSubscription
    ```
    Выполните следующую команду, чтобы выбрать подписку, с которой вы собираетесь работать. Замените значение **SubscriptionId** на идентификатор подписки Azure:

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"    
    ```  
3. Создайте группу ресурсов: Создайте группу ресурсов ADFTutorialResourceGroup, если она не существует в вашей подписке. 

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location "East Us" 
    ```
4. Создайте фабрику данных. 

    ```powershell
     $df = Set-AzDataFactoryV2 -Location EastUS -Name $dataFactoryName -ResourceGroupName $resourceGroupName
    ```

    Чтобы просмотреть выходные данные, выполните следующую команду: 

    ```powershell
    $df
    ```

## <a name="create-self-hosted-ir"></a>Создание локальной среды IR
В этом разделе вы создадите локальную среду выполнения интеграции и свяжите ее с виртуальной машиной Azure в той же виртуальной сети Azure, где находится ваш кластер HDInsight.

1. Создайте локальную среду выполнения интеграции. Используйте уникальное имя в случае, если существует другая среда выполнения интеграции с тем же именем.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $selfHostedIntegrationRuntimeName -Type SelfHosted
   ```
    Эта команда выполняет логическую регистрацию локальной среды выполнения интеграции. 
2. Чтобы получить ключи проверки подлинности для регистрации локальной среды выполнения интеграции, используйте PowerShell. Скопируйте один из ключей для регистрации локальной среды выполнения интеграции.

   ```powershell
   Get-AzDataFactoryV2IntegrationRuntimeKey -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $selfHostedIntegrationRuntimeName | ConvertTo-Json
   ```

   Пример выходных данных: 

   ```powershell
   {
       "AuthKey1":  "IR@0000000000000000000000000000000000000=",
       "AuthKey2":  "IR@0000000000000000000000000000000000000="
   }
   ```
    Запишите значение **AuthKey1** без кавычек. 
3. Создайте виртуальную машину Azure и присоедините ее к той же виртуальной сети, которая содержит кластер HDInsight. Дополнительные сведения см. в разделе [Создание виртуальных машин](../virtual-network/quick-create-portal.md#create-virtual-machines). Присоедините их к виртуальной сети Azure. 
4. На виртуальной машине Azure скачайте [локальную среду выполнения интеграции](https://www.microsoft.com/download/details.aspx?id=39717). Используйте ключ проверки подлинности, полученный на предыдущем шаге, чтобы вручную зарегистрировать локальную среду выполнения интеграции. 

   ![Регистрация среды выполнения интеграции](media/tutorial-transform-data-using-hive-in-vnet/register-integration-runtime.png)

   Когда локальная среда выполнения интеграции будет успешно зарегистрирована, вы увидите следующее сообщение: Сообщение об ![успешной регистрации](media/tutorial-transform-data-using-hive-in-vnet/registered-successfully.png)

   Когда узел будет подключен к облачной службе, отобразится следующая страница: Страница ![Node is connected](media/tutorial-transform-data-using-hive-in-vnet/node-is-connected.png) (Узел подключен)

## <a name="author-linked-services"></a>Создание связанных служб

Создайте и разверните две связанные службы в этом разделе:
- Связанную службу хранилища Azure, которая связывает учетную запись хранения Azure с фабрикой данных. Это основное хранилище, которое использует кластер HDInsight. В этом случае мы также используем эту учетную запись хранения Azure для сохранения скрипта Hive и его выходных данных.
- Связанную службу HDInsight. Фабрика данных Azure отправляет скрипт Hive в этот кластер HDInsight для выполнения.

### <a name="azure-storage-linked-service"></a>Связанная служба хранения Azure

Создайте файл JSON, используя предпочитаемый редактор, скопируйте следующее определение JSON связанной службы хранилища Azure и сохраните файл как **MyStorageLinkedService.json**.

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=<storageAccountName>;AccountKey=<storageAccountKey>"
      },
      "connectVia": {
        "referenceName": "MySelfhostedIR",
        "type": "IntegrationRuntimeReference"
      }  
    }
}
```

Замените значения **&lt;accountname&gt; и &lt;accountkey&gt;** на имя вашей учетной записи хранения Azure и ее ключ.

### <a name="hdinsight-linked-service"></a>Связанная служба HDInsight

Создайте файл JSON, используя предпочитаемый редактор, скопируйте следующее определение JSON связанной службы HDInsight Azure и сохраните файл как **MyHDInsightLinkedService.json**.

```
{
  "name": "MyHDInsightLinkedService",
  "properties": {     
      "type": "HDInsight",
      "typeProperties": {
          "clusterUri": "https://<clustername>.azurehdinsight.net",
          "userName": "<username>",
          "password": {
            "value": "<password>",
            "type": "SecureString"
          },
          "linkedServiceName": {
            "referenceName": "MyStorageLinkedService",
            "type": "LinkedServiceReference"
          }
      },
      "connectVia": {
        "referenceName": "MySelfhostedIR",
        "type": "IntegrationRuntimeReference"
      }
  }
}
```

Обновите значения следующих свойств в определении связанных служб:

- **userName**. Имя пользователя для входа в кластер, которое вы указали при создании кластера. 
- **password**. Пароль пользователя.
- **clusterUri**. Укажите URL-адрес кластера HDInsight в формате `https://<clustername>.azurehdinsight.net`.  В этой статье предполагается, что у вас есть доступ к кластеру через Интернет. Например, вы можете подключиться к кластеру по адресу `https://clustername.azurehdinsight.net`. Этот адрес использует общедоступный шлюз, который будет недоступен, если вы использовали группы безопасности сети или определяемые пользователем маршруты для ограничения доступа через Интернет. Чтобы фабрика данных отправляла задания в кластер HDInsight в виртуальной сети Azure, необходимо настроить виртуальную сеть Azure таким образом, чтобы URL-адрес был разрешен в частный IP-адрес шлюза, используемого HDInsight.

  1. На портале Azure откройте виртуальную сеть, в которой находится HDInsight. Откройте сетевой интерфейс, используя имя, которое начинается с `nic-gateway-0`. Запишите частный IP-адрес. Например, 10.6.0.15. 
  2. Если ваша виртуальная сеть Azure имеет DNS-сервер, обновите запись DNS, чтобы URL-адрес кластера HDInsight `https://<clustername>.azurehdinsight.net` можно было разрешить в `10.6.0.15`. Рекомендуем использовать этот подход. Если у вас нет DNS-сервера в виртуальной сети Azure, вы можете применить временное решение: отредактировать файл hosts (C:\Windows\System32\drivers\etc) всех виртуальных машин, зарегистрированных в качестве узлов локальной среды выполнения интеграции, добавив следующую запись: 
  
        `10.6.0.15 myHDIClusterName.azurehdinsight.net`

## <a name="create-linked-services"></a>Создание связанных служб
В PowerShell перейдите в папку, где были созданы JSON-файлы, и выполните следующую команду, чтобы развернуть связанные службы. 

1. В PowerShell перейдите в папку, где были созданы JSON-файлы.
2. Чтобы создать связанную службу хранилища Azure, выполните следующую команду. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyStorageLinkedService" -File "MyStorageLinkedService.json"
    ```
3. Чтобы создать связанную службу Azure HDInsight, выполните следующую команду. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyHDInsightLinkedService" -File "MyHDInsightLinkedService.json"
    ```

## <a name="author-a-pipeline"></a>Создание конвейера
На этом этапе создайте конвейер с действием Hive. Это действие выполняет скрипт Hive для получения данных из примера таблицы и сохранения их по пути, который вы определили. Создайте JSON-файл в любом редакторе, скопируйте следующее определение JSON определения конвейера и сохраните его как **MyHivePipeline.json**.


```json
{
  "name": "MyHivePipeline",
  "properties": {
    "activities": [
      {
        "name": "MyHiveActivity",
        "type": "HDInsightHive",
        "linkedServiceName": {
            "referenceName": "MyHDILinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
          "scriptPath": "adftutorial\\hivescripts\\hivescript.hql",
          "getDebugInfo": "Failure",
          "defines": {           
            "Output": "wasb://<Container>@<StorageAccount>.blob.core.windows.net/outputfolder/"
          },
          "scriptLinkedService": {
            "referenceName": "MyStorageLinkedService",
            "type": "LinkedServiceReference"
          }
        }
      }
    ]
  }
}

```

Обратите внимание на следующие моменты.

- **scriptPath** указывает путь к скрипту Hive в учетной записи хранения Azure, используемой для MyStorageLinkedService. Путь учитывает регистр.
- **Выходные данные** выступают в качестве аргумента, используемого в скрипте Hive. Используйте формат `wasb://<Container>@<StorageAccount>.blob.core.windows.net/outputfolder/`, который должен указывать на существующую папку в службе хранилища Azure. Путь учитывает регистр. 

Перейдите в папку, где были созданы файлы JSON, и выполните следующую команду, чтобы развернуть конвейер: 


```powershell
Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name $pipelineName -File "MyHivePipeline.json"
```

## <a name="start-the-pipeline"></a>Запуск конвейера 

1. Запуск конвейера. Эта команда также запишет идентификатор выполнения конвейера для будущего мониторинга.

    ```powershell
    $runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName $pipelineName
   ```
2. Запустите следующий скрипт, чтобы проверять состояние выполнения, пока оно не завершится.

    ```powershell
    while ($True) {
        $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

        if(!$result) {
            Write-Host "Waiting for pipeline to start..." -foregroundcolor "Yellow"
        }
        elseif (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
            Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
        }
        else {
            Write-Host "Pipeline '"$pipelineName"' run finished. Result:" -foregroundcolor "Yellow"
            $result
            break
        }
        ($result | Format-List | Out-String)
        Start-Sleep -Seconds 15
    }
    
    Write-Host "Activity `Output` section:" -foregroundcolor "Yellow"
    $result.Output -join "`r`n"

    Write-Host "Activity `Error` section:" -foregroundcolor "Yellow"
    $result.Error -join "`r`n"
    ```

   Вот результат примера выполнения:

    ```json
    Pipeline run status: In Progress
    
    ResourceGroupName : ADFV2SampleRG2
    DataFactoryName   : SampleV2DataFactory2
    ActivityName      : MyHiveActivity
    PipelineRunId     : 000000000-0000-0000-000000000000000000
    PipelineName      : MyHivePipeline
    Input             : {getDebugInfo, scriptPath, scriptLinkedService, defines}
    Output            :
    LinkedServiceName :
    ActivityRunStart  : 9/18/2017 6:58:13 AM
    ActivityRunEnd    :
    DurationInMs      :
    Status            : InProgress
    Error             :
    
    Pipeline ' MyHivePipeline' run finished. Result:
    
    ResourceGroupName : ADFV2SampleRG2
    DataFactoryName   : SampleV2DataFactory2
    ActivityName      : MyHiveActivity
    PipelineRunId     : 0000000-0000-0000-0000-000000000000
    PipelineName      : MyHivePipeline
    Input             : {getDebugInfo, scriptPath, scriptLinkedService, defines}
    Output            : {logLocation, clusterInUse, jobId, ExecutionProgress...}
    LinkedServiceName :
    ActivityRunStart  : 9/18/2017 6:58:13 AM
    ActivityRunEnd    : 9/18/2017 6:59:16 AM
    DurationInMs      : 63636
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    Activity Output section:
    "logLocation": "wasbs://adfjobs@adfv2samplestor.blob.core.windows.net/HiveQueryJobs/000000000-0000-47c3-9b28-1cdc7f3f2ba2/18_09_2017_06_58_18_023/Status"
    "clusterInUse": "https://adfv2HivePrivate.azurehdinsight.net"
    "jobId": "job_1505387997356_0024"
    "ExecutionProgress": "Succeeded"
    "effectiveIntegrationRuntime": "MySelfhostedIR"
    Activity Error section:
    "errorCode": ""
    "message": ""
    "failureType": ""
    "target": "MyHiveActivity"
    ```
4. В папке `outputfolder` найдите новый файл, созданный в результате выполнения запроса Hive. Он должен выглядеть следующим образом: 

   ```
   8 en-US SCH-i500 California
   23 en-US Incredible Pennsylvania
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   212 en-US SCH-i500 New York
   246 en-US SCH-i500 District Of Columbia
   246 en-US SCH-i500 District Of Columbia
   ```

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве вы выполнили следующие шаги: 

> [!div class="checklist"]
> * Создали фабрику данных. 
> * Создали и настроили среду выполнения интеграции с самостоятельным размещением.
> * Создали и развернули эти связанные службы.
> * Создали и развернули конвейер, который содержит действие Hive.
> * Запуск конвейера.
> * Мониторинг конвейера 
> * Проверили выходные данные. 

Перейдите к следующему руководству, чтобы узнать о преобразовании данных с помощью кластера Spark в Azure:

> [!div class="nextstepaction"]
>[Branching and chaining activities in a Data Factory pipeline](tutorial-control-flow.md) (Ветвление и создание цепного потока управления фабрики данных)
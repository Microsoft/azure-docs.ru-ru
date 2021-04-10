---
title: Копирование данных в хранилище BLOB-объектов с помощью службы "Фабрика данных Azure"
description: Создание фабрики данных Azure с помощью PowerShell для копирования данных между расположениями в Хранилище BLOB-объектов Azure.
author: linda33wj
ms.service: data-factory
ms.devlang: powershell
ms.topic: quickstart
ms.date: 04/10/2020
ms.author: jingwang
ms.openlocfilehash: 9f419d89a9757a11055781335cbf98e9eb651548
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100372728"
---
# <a name="quickstart-create-an-azure-data-factory-using-powershell"></a>Краткое руководство. Создание фабрики данных Azure с помощью PowerShell

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Текущая версия](quickstart-create-data-factory-powershell.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом кратком руководстве показано, как создать фабрику данных Azure с помощью PowerShell. Конвейер, который вы создадите в этой фабрике данных, **копирует** данные из одной папки в другую в хранилище BLOB-объектов Azure. Инструкции по **преобразованию** данных с помощью Фабрики данных Azure см. в статье [Преобразование данных с помощью действия Spark в фабрике данных Azure](transform-data-using-spark.md).

> [!NOTE]
> Эта статья не содержит подробный обзор службы фабрики данных. Общие сведения о службе фабрики данных Azure см. в статье [Введение в фабрику данных Azure](introduction.md).

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)]

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы установить модули Azure PowerShell, выполните инструкции из статьи [Установка и настройка Azure PowerShell](/powershell/azure/install-Az-ps).

>[!WARNING]
>Если вы не используете последние версии PowerShell и модуля Фабрики данных, во время выполнения команд могут возникнуть ошибки десериализации. 

#### <a name="log-in-to-powershell"></a>Вход в PowerShell

1. Запустите **PowerShell** на компьютере. Не закрывайте PowerShell, пока выполняются описанные в этом кратком руководстве инструкции. Если закрыть и снова открыть это окно, то придется вновь выполнять эти команды.

2. Выполните следующую команду и введите те же имя пользователя Azure и пароль, которые используются для входа на портал Azure:

    ```powershell
    Connect-AzAccount
    ```

3. Чтобы просмотреть все подписки для этой учетной записи, выполните следующую команду:

    ```powershell
    Get-AzSubscription
    ```

4. Если с вашей учетной записью связаны несколько подписок, выполните следующую команду, чтобы выбрать нужную подписку. Замените значение **SubscriptionId** на идентификатор подписки Azure:

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"
    ```

## <a name="create-a-data-factory"></a>Создание фабрики данных

1. Определите переменную для имени группы ресурсов, которую в дальнейшем можно будет использовать в командах PowerShell. Скопируйте текст следующей команды в PowerShell, укажите имя [группы ресурсов Azure](../azure-resource-manager/management/overview.md) в двойных кавычках, а затем выполните команду. Например: `"ADFQuickStartRG"`.

     ```powershell
    $resourceGroupName = "ADFQuickStartRG";
    ```

    Если группа ресурсов уже существует, вы можете не перезаписывать ее. Назначьте переменной `$ResourceGroupName` другое значение и еще раз выполните команду.

2. Чтобы создать группу ресурсов Azure, выполните следующую команду:

    ```powershell
    $ResGrp = New-AzResourceGroup $resourceGroupName -location 'East US'
    ```

    Если группа ресурсов уже существует, вы можете не перезаписывать ее. Назначьте переменной `$ResourceGroupName` другое значение и еще раз выполните команду.

3. Определите переменную для имени фабрики данных. 

    > [!IMPORTANT]
    >  Измените имя фабрики данных, чтобы оно было глобально уникальным. Например, укажите ADFTutorialFactorySP1127.

    ```powershell
    $dataFactoryName = "ADFQuickStartFactory";
    ```

4. Чтобы создать фабрику данных, выполните следующий командлет **Set-AzDataFactoryV2**, используя свойства Location и ResourceGroupName из переменной $ResGrp.

    ```powershell
    $DataFactory = Set-AzDataFactoryV2 -ResourceGroupName $ResGrp.ResourceGroupName `
        -Location $ResGrp.Location -Name $dataFactoryName
    ```

Обратите внимание на следующие моменты.

* Имя фабрики данных Azure должно быть глобально уникальным. Если появляется следующая ошибка, измените имя и повторите попытку.

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```

* Чтобы создать экземпляры фабрики данных, нужно назначить учетной записи пользователя, используемой для входа в Azure, роль **участника**, **владельца** либо **администратора** подписки Azure.

* Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (служба хранилища Azure, база данных SQL Azure и т. д.) и вычисления (HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.


## <a name="create-a-linked-service"></a>Создание связанной службы

Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В этом кратком руководстве мы создадим службу, связанную со службой хранилища Azure, которая используется и как хранилище-источник, и как хранилище-приемник. Связанная служба содержит сведения о подключении, используемые фабрикой данных для подключения к ней в среде выполнения.

>[!TIP]
>В этом кратком руководстве аутентификация в хранилище данных выполняется с помощью *ключа учетной записи*, но при необходимости можно выбрать другие поддерживаемые инструменты проверки подлинности, такие как *универсальный код ресурса SAS*,*субъект-служба* и *управляемое удостоверение*. Дополнительные сведения см. в соответствующих разделах [этой статьи](./connector-azure-blob-storage.md#linked-service-properties).
>Чтобы безопасно хранить секреты для хранилищ данных, также рекомендуется использовать Azure Key Vault. Дополнительные сведения см. в [этой статье](./store-credentials-in-key-vault.md).

1. Создайте JSON-файл **AzureStorageLinkedService.json** в папке **C:\ADFv2QuickStartPSH** со следующим содержимым: (Если папки ADFv2QuickStartPSH нет, создайте ее.)

    > [!IMPORTANT]
    > Перед сохранением файла замените значения &lt;accountname&gt; и &lt;accountkey&gt; на имя вашей учетной записи хранения Azure и ее ключ.

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "annotations": [],
            "type": "AzureBlobStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>;EndpointSuffix=core.windows.net"
            }
        }
    }
    ```

    Если вы используете Блокнот, в диалоговом окне **Сохранить как** в поле **Тип файла** выберите пункт **Все файлы**. Иначе к файлу может добавиться расширение `.txt`. Например, `AzureStorageLinkedService.json.txt`. Если вы создали файл в проводнике до того, как открыли его в Блокноте, расширение `.txt` может не отображаться, так как по умолчанию установлен параметр **Скрывать расширения для зарегистрированных типов файлов**. Удалите расширение `.txt`, прежде чем перейти к следующему шагу.

2. В **PowerShell** перейдите в папку **ADFv2QuickStartPSH**.

    ```powershell
    Set-Location 'C:\ADFv2QuickStartPSH'
    ```

3. Выполните командлет **Set-AzDataFactoryV2LinkedService**, чтобы создать связанную службу. **AzureStorageLinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName -Name "AzureStorageLinkedService" `
        -DefinitionFile ".\AzureStorageLinkedService.json"
    ```

    Пример выходных данных:

    ```console
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobStorageLinkedService
    ```

## <a name="create-datasets"></a>Создание наборов данных

В ходе этой процедуры вы создадите два набора данных, **InputDataset** и **OutputDataset**. Эти наборы данных имеют тип **Binary**. Они будут ссылаться на связанную службу хранилища Azure, созданную в предыдущем разделе.
Входной набор данных представляет исходные данные в папке входных данных. В определении входного набора данных укажите контейнер больших двоичных объектов (**adftutorial**), папку (**input**) и файл (**emp.txt**), определяющие расположение исходных данных.
Выходной набор данных представляет данные, которые копируются в место назначения. В определении выходного набора данных укажите контейнер больших двоичных объектов (**adftutorial**), папку (**output**) и файл, определяющие расположение копируемых данных. 
1. Создайте файл JSON с именем **InputDataset.json** в папке **C:\ADFv2QuickStartPSH** со следующим содержимым:

    ```json
    {
        "name": "InputDataset",
        "properties": {
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            },
            "annotations": [],
            "type": "Binary",
            "typeProperties": {
                "location": {
                    "type": "AzureBlobStorageLocation",
                    "fileName": "emp.txt",
                    "folderPath": "input",
                    "container": "adftutorial"
                }
            }
        }
    }
    ```

2. Чтобы создать набор данных **InputDataset**, выполните командлет **Set-AzDataFactoryV2Dataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName -Name "InputDataset" `
        -DefinitionFile ".\InputDataset.json"
    ```

    Пример выходных данных:

    ```console
    DatasetName       : InputDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.BinaryDataset
    ```

3. Повторите эти шаги, чтобы создать выходной набор данных. Создайте файл JSON с именем **OutputDataset.json** в папке **C:\ADFv2QuickStartPSH** со следующим содержимым:

    ```json
    {
        "name": "OutputDataset",
        "properties": {
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            },
            "annotations": [],
            "type": "Binary",
            "typeProperties": {
                "location": {
                    "type": "AzureBlobStorageLocation",
                    "folderPath": "output",
                    "container": "adftutorial"
                }
            }
        }
    }
    ```

4. Выполните командлет **Set-AzDataFactoryV2Dataset**, чтобы создать набор данных **OutDataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName -Name "OutputDataset" `
        -DefinitionFile ".\OutputDataset.json"
    ```

    Пример выходных данных:

    ```console
    DatasetName       : OutputDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.BinaryDataset
    ```
## <a name="create-a-pipeline"></a>Создание конвейера

На этом этапе вы создадите конвейер с действием копирования, которое использует входной и выходной наборы данных. При помощи действия копирования данные копируются из файла, указанного в параметрах входного набора данных, в файл, указанный в параметрах выходного набора данных.  

1. Создайте файл JSON с именем **Adfv2QuickStartPipeline.json** в папке **C:\ADFv2QuickStartPSH** со следующим содержимым:

    ```json
    {
        "name": "Adfv2QuickStartPipeline",
        "properties": {
            "activities": [
                {
                    "name": "CopyFromBlobToBlob",
                    "type": "Copy",
                    "dependsOn": [],
                    "policy": {
                        "timeout": "7.00:00:00",
                        "retry": 0,
                        "retryIntervalInSeconds": 30,
                        "secureOutput": false,
                        "secureInput": false
                    },
                    "userProperties": [],
                    "typeProperties": {
                        "source": {
                            "type": "BinarySource",
                            "storeSettings": {
                                "type": "AzureBlobStorageReadSettings",
                                "recursive": true
                            }
                        },
                        "sink": {
                            "type": "BinarySink",
                            "storeSettings": {
                                "type": "AzureBlobStorageWriteSettings"
                            }
                        },
                        "enableStaging": false
                    },
                    "inputs": [
                        {
                            "referenceName": "InputDataset",
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "OutputDataset",
                            "type": "DatasetReference"
                        }
                    ]
                }
            ],
            "annotations": []
        }
    }
    ```

2. Чтобы создать конвейер **Adfv2QuickStartPipeline**, выполните командлет **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    $DFPipeLine = Set-AzDataFactoryV2Pipeline `
        -DataFactoryName $DataFactory.DataFactoryName `
        -ResourceGroupName $ResGrp.ResourceGroupName `
        -Name "Adfv2QuickStartPipeline" `
        -DefinitionFile ".\Adfv2QuickStartPipeline.json"
    ```

## <a name="create-a-pipeline-run"></a>Создание конвейера

На этом шаге вы создадите конвейер.

Выполните командлет **Invoke-AzDataFactoryV2Pipeline**, чтобы создать конвейер. Командлет позволяет получить идентификатор выполнения конвейера для дальнейшего мониторинга.

  ```powershell
$RunId = Invoke-AzDataFactoryV2Pipeline `
    -DataFactoryName $DataFactory.DataFactoryName `
    -ResourceGroupName $ResGrp.ResourceGroupName `
    -PipelineName $DFPipeLine.Name 
```

## <a name="monitor-the-pipeline-run"></a>Мониторинг конвейера

1. Запустите приведенный ниже скрипт PowerShell, чтобы проверять состояние выполнения, пока не закончится копирование данных. Скопируйте приведенный ниже скрипт в окно PowerShell и нажмите клавишу ВВОД.

    ```powershell
    while ($True) {
        $Run = Get-AzDataFactoryV2PipelineRun `
            -ResourceGroupName $ResGrp.ResourceGroupName `
            -DataFactoryName $DataFactory.DataFactoryName `
            -PipelineRunId $RunId

        if ($Run) {
            if ( ($Run.Status -ne "InProgress") -and ($Run.Status -ne "Queued") ) {
                Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
                $Run
                break
            }
            Write-Output ("Pipeline is running...status: " + $Run.Status)
        }

        Start-Sleep -Seconds 10
    }
    ```

    Вот результат примера выполнения:

    ```console
    Pipeline is running...status: InProgress
    Pipeline run finished. The status is:  Succeeded
    
    ResourceGroupName : ADFQuickStartRG
    DataFactoryName   : ADFQuickStartFactory
    RunId             : 00000000-0000-0000-0000-0000000000000
    PipelineName      : Adfv2QuickStartPipeline
    LastUpdated       : 8/27/2019 7:23:07 AM
    Parameters        : {}
    RunStart          : 8/27/2019 7:22:56 AM
    RunEnd            : 8/27/2019 7:23:07 AM
    DurationInMs      : 11324
    Status            : Succeeded
    Message           : 
    ```
2. Запустите следующий скрипт, извлекающий сведения о выполнении действия копирования, например размер записанных и прочитанных данных.

    ```powershell
    Write-Output "Activity run details:"
    $Result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineRunId $RunId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $Result

    Write-Output "Activity 'Output' section:"
    $Result.Output -join "`r`n"

    Write-Output "Activity 'Error' section:"
    $Result.Error -join "`r`n"
    ```
3. Убедитесь, что это выходные данные аналогичные следующему примеру результатов выполнения действия:

    ```console
    ResourceGroupName : ADFQuickStartRG
    DataFactoryName   : ADFQuickStartFactory
    ActivityRunId     : 00000000-0000-0000-0000-000000000000
    ActivityName      : CopyFromBlobToBlob
    PipelineRunId     : 00000000-0000-0000-0000-000000000000
    PipelineName      : Adfv2QuickStartPipeline
    Input             : {source, sink, enableStaging}
    Output            : {dataRead, dataWritten, filesRead, filesWritten...}
    LinkedServiceName :
    ActivityRunStart  : 8/27/2019 7:22:58 AM
    ActivityRunEnd    : 8/27/2019 7:23:05 AM
    DurationInMs      : 6828
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    Activity 'Output' section:
    "dataRead": 20
    "dataWritten": 20
    "filesRead": 1
    "filesWritten": 1
    "sourcePeakConnections": 1
    "sinkPeakConnections": 1
    "copyDuration": 4
    "throughput": 0.01
    "errors": []
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (Central US)"
    "usedDataIntegrationUnits": 4
    "usedParallelCopies": 1
    "executionDetails": [
      {
        "source": {
          "type": "AzureBlobStorage"
        },
        "sink": {
          "type": "AzureBlobStorage"
        },
        "status": "Succeeded",
        "start": "2019-08-27T07:22:59.1045645Z",
        "duration": 4,
        "usedDataIntegrationUnits": 4,
        "usedParallelCopies": 1,
        "detailedDurations": {
          "queuingDuration": 3,
          "transferDuration": 1
        }
      }
    ]
    
    Activity 'Error' section:
    "errorCode": ""
    "message": ""
    "failureType": ""
    "target": "CopyFromBlobToBlob"
    ```

[!INCLUDE [data-factory-quickstart-verify-output-cleanup.md](../../includes/data-factory-quickstart-verify-output-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом примере конвейер копирует данные из одного расположения в другое в хранилище BLOB-объектов Azure. Перейдите к [руководствам](tutorial-copy-data-dot-net.md), чтобы узнать об использовании фабрики данных в различных сценариях.

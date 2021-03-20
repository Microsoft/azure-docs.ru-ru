---
title: Отслеживание фабрики данных Azure с помощью программных средств
description: Узнайте, как отслеживать конвейер в фабрике данных с помощью различных пакетов средств разработки программного обеспечения (пакетов SDK).
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/16/2018
author: dcstwh
ms.author: weetok
ms.custom: devx-track-python
ms.openlocfilehash: 6c913c7c623c77baea0c575d06d2c44709af43fa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101740449"
---
# <a name="programmatically-monitor-an-azure-data-factory"></a>Отслеживание фабрики данных Azure с помощью программных средств

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этой статье описывается как отслеживать конвейер в фабрике данных с помощью различных пакетов средств разработки программного обеспечения (пакетов SDK). 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="data-range"></a>Срок хранения данных

В фабрике данных данные запуска конвейеров сохраняются только в течение 45 дней. Если вы выполняете программный запрос на сведения о запуске для конвейера фабрики данных (например, с помощью команды PowerShell `Get-AzDataFactoryV2PipelineRun`), максимальные значения дат для необязательных параметров `LastUpdatedAfter` и `LastUpdatedBefore` не указываются. Но если вы запрашиваете данные за прошлый год, например, вы не получите ошибку, но только данные о выполнении конвейера за последние 45 дней.

Если вы хотите, чтобы данные о выполнении конвейера настроились более 45 дней, Настройте собственное ведение журнала диагностики с помощью [Azure Monitor](monitor-using-azure-monitor.md).

## <a name="pipeline-run-information"></a>Сведения о выполнении конвейера

Сведения о свойствах выполнения конвейера см. в [справочнике по API пипелинерун](/rest/api/datafactory/pipelineruns/get#pipelinerun). Состояние выполнения конвейера отличается от состояния жизненного цикла, возможные значения состояния выполнения перечислены ниже.

* Поставлено в очередь
* InProgress
* Выполнено
* Сбой
* Идет отмена
* Отменено

## <a name="net"></a>.NET
Полный пошаговый анализ создания и мониторинга конвейера с помощью пакета SDK для .NET см. в статье [Создание фабрики данных и конвейера с помощью .NET](quickstart-create-data-factory-dot-net.md).

1. Добавьте следующий код, чтобы постоянно проверять состояние выполнения конвейера до завершения копирования данных.

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, runResponse.RunId);
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress" || pipelineRun.Status == "Queued")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

2. Добавьте следующий код, извлекающий сведения о выполнении действия копирования, например размер записанных и прочитанных данных.

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");
   
    List<ActivityRun> activityRuns = client.ActivityRuns.ListByPipelineRun(
    resourceGroup, dataFactoryName, runResponse.RunId, DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)).ToList(); 
    if (pipelineRun.Status == "Succeeded")
        Console.WriteLine(activityRuns.First().Output);
    else
        Console.WriteLine(activityRuns.First().Error);
    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

Полная документация по пакету SDK для .NET приведена в [справочнике по пакету SDK для .NET для фабрики данных](/dotnet/api/microsoft.azure.management.datafactory).

## <a name="python"></a>Python
Полное пошаговое руководство по созданию и мониторингу конвейера с помощью пакета SDK для Python см. в статье [Создание фабрики данных и конвейера с помощью Python](quickstart-create-data-factory-python.md).

Для отслеживания работы конвейера добавьте следующий код.

```python
# Monitor the pipeline run
time.sleep(30)
pipeline_run = adf_client.pipeline_runs.get(
    rg_name, df_name, run_response.run_id)
print("\n\tPipeline run status: {}".format(pipeline_run.status))
activity_runs_paged = list(adf_client.activity_runs.list_by_pipeline_run(
    rg_name, df_name, pipeline_run.run_id, datetime.now() - timedelta(1),  datetime.now() + timedelta(1)))
print_activity_run_details(activity_runs_paged[0])
```

Полная документация по пакету SDK для Python приведена в [справочнике по пакету SDK для Python для фабрики данных](/python/api/overview/azure/datafactory).

## <a name="rest-api"></a>REST API
Полный пошаговый анализ создания и мониторинга конвейера с помощью REST API см. в статье [Создание фабрики данных и конвейера с помощью REST API](quickstart-create-data-factory-rest-api.md).
 
1. Запустите следующий скрипт, чтобы проверять состояние выполнения, пока не закончится копирование данных.

    ```powershell
    $request = "https://management.azure.com/subscriptions/${subsId}/resourceGroups/${resourceGroup}/providers/Microsoft.DataFactory/factories/${dataFactoryName}/pipelineruns/${runId}?api-version=${apiVersion}"
    while ($True) {
        $response = Invoke-RestMethod -Method GET -Uri $request -Header $authHeader
        Write-Host  "Pipeline run status: " $response.Status -foregroundcolor "Yellow"

        if ( ($response.Status -eq "InProgress") -or ($response.Status -eq "Queued") ) {
            Start-Sleep -Seconds 15
        }
        else {
            $response | ConvertTo-Json
            break
        }
    }
    ```
2. Запустите следующий скрипт, извлекающий сведения о выполнении действия копирования, например размер записанных и прочитанных данных.

    ```powershell
    $request = "https://management.azure.com/subscriptions/${subsId}/resourceGroups/${resourceGroup}/providers/Microsoft.DataFactory/factories/${dataFactoryName}/pipelineruns/${runId}/activityruns?api-version=${apiVersion}&startTime="+(Get-Date).ToString('yyyy-MM-dd')+"&endTime="+(Get-Date).AddDays(1).ToString('yyyy-MM-dd')+"&pipelineName=Adfv2QuickStartPipeline"
    $response = Invoke-RestMethod -Method GET -Uri $request -Header $authHeader
    $response | ConvertTo-Json
    ```

Полная документация по REST API приведена в [справочнике по REST API фабрики данных](/rest/api/datafactory/).

## <a name="powershell"></a>PowerShell
Полный пошаговый анализ создания и мониторинга конвейера с помощью PowerShell см. в статье [Создание фабрики данных и конвейера с помощью PowerShell](quickstart-create-data-factory-powershell.md).

1. Запустите следующий скрипт, чтобы проверять состояние выполнения, пока не закончится копирование данных.

    ```powershell
    while ($True) {
        $run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

        if ($run) {
            if ( ($run.Status -ne "InProgress") -and ($run.Status -ne "Queued") ) {
                Write-Output ("Pipeline run finished. The status is: " +  $run.Status)
                $run
                break
            }
            Write-Output ("Pipeline is running...status: " + $run.Status)
        }

        Start-Sleep -Seconds 30
    }
    ```
2. Запустите следующий скрипт, извлекающий сведения о выполнении действия копирования, например размер записанных и прочитанных данных.

    ```powershell
    Write-Host "Activity run details:" -foregroundcolor "Yellow"
    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $result
    
    Write-Host "Activity 'Output' section:" -foregroundcolor "Yellow"
    $result.Output -join "`r`n"
    
    Write-Host "\nActivity 'Error' section:" -foregroundcolor "Yellow"
    $result.Error -join "`r`n"
    ```

Полная документация по командлетам PowerShell приведена в [справочнике по командлетам PowerShell для фабрики данных](/powershell/module/az.datafactory).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об использовании Azure Monitor для отслеживания конвейеров фабрики данных см. в статье [Monitor data factories using Azure Monitor](monitor-using-azure-monitor.md) (Отслеживание фабрик данных с помощью Azure Monitor).
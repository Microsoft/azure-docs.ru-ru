---
title: Запуск пакета служб SSIS с действием хранимой процедуры — Azure
description: Из этой статьи вы узнаете, как в конвейере Фабрики данных Azure запустить пакет SQL Server Integration Services (SSIS) с помощью действия хранимой процедуры.
author: swinarko
ms.service: data-factory
ms.devlang: powershell
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: sawinark
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6260606fe56d4dfc6bac93e04e726b5fd3298777
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100391513"
---
# <a name="run-an-ssis-package-with-the-stored-procedure-activity-in-azure-data-factory"></a>Запуск в Фабрике данных Azure пакета SQL Server Integration Services с помощью действия хранимой процедуры

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этой статье описывается, как запустить пакет SQL Server Integration Services (SSIS) в конвейере Фабрики данных Azure с помощью действия хранимой процедуры. 

## <a name="prerequisites"></a>Предварительные условия

### <a name="azure-sql-database"></a>База данных SQL Azure 
В этом руководстве для размещения каталога служб SSIS используется база данных SQL Azure. Вы также можете использовать Управляемый экземпляр Azure SQL.

## <a name="create-an-azure-ssis-integration-runtime"></a>Создание среды выполнения интеграции Azure SSIS.
Создайте среду выполнения интеграции Azure SSIS, если у вас ее нет. Для этого выполните пошаговую инструкцию в статье [Развертывание пакетов служб интеграции SQL Server (SSIS) в Azure](./tutorial-deploy-ssis-packages-azure.md).

## <a name="data-factory-ui-azure-portal"></a>Пользовательский интерфейс фабрики данных (портал Azure)
В этом разделе для создания конвейера фабрики данных с действием хранимой процедуры, которая вызывает пакет SSIS, используется пользовательский интерфейс фабрики данных.

### <a name="create-a-data-factory"></a>Создание фабрики данных
Сначала нужно создать фабрику данных с помощью портала Azure. 

1. Запустите веб-браузер **Microsoft Edge** или **Google Chrome**. Сейчас только эти браузеры поддерживают пользовательский интерфейс фабрики данных.
2. Перейдите на [портал Azure](https://portal.azure.com). 
3. В меню слева щелкните **Создать**, выберите **Данные+аналитика** и щелкните **Фабрика данных**. 
   
   ![Создать -> Фабрика данных](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory-menu.png)
2. На странице **Новая фабрика данных** введите **ADFTutorialDataFactory** в поле **Имя**. 
      
     ![Страница "Новая фабрика данных"](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory.png)
 
   Имя фабрики данных Azure должно быть **глобально уникальным**. Если вы увидите следующую ошибку для поля имени, введите другое имя фабрики данных (например, ваше_имя_ADFTutorialBulkCopyDF). Ознакомьтесь со статьей [Фабрика данных Azure — правила именования](naming-rules.md), чтобы узнать правила именования для артефактов службы "Фабрика данных".
  
     ![Ошибка, связанная с недоступностью имени](./media/how-to-invoke-ssis-package-stored-procedure-activity/name-not-available-error.png)
3. Выберите **подписку** Azure, в рамках которой вы хотите создать фабрику данных. 
4. Для **группы ресурсов** выполните одно из следующих действий.
     
   - Выберите **Использовать существующую** и укажите существующую группу ресурсов в раскрывающемся списке. 
   - Выберите **Создать новую** и укажите имя группы ресурсов.   
         
     Сведения о группах ресурсов см. в статье, где описывается [использование групп ресурсов для управления ресурсами Azure](../azure-resource-manager/management/overview.md).  
4. Укажите **V2** при выборе **версии**.
5. Укажите **расположение** фабрики данных. В этом раскрывающемся списке отображаются только сведения о расположениях, поддерживаемых службой "Фабрика данных". Хранилища данных (служба хранилища Azure, служба "База данных SQL Azure" и т. д.) и вычислительные ресурсы (HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.
6. Кроме того, установите флажок **Закрепить на панели мониторинга**.     
7. Нажмите кнопку **Создать**.
8. На панели мониторинга вы увидите приведенный ниже элемент с состоянием **Deploying data factory** (Развертывание фабрики данных). 

     ![Элемент Deploying data factory (Развертывание фабрики данных)](media//how-to-invoke-ssis-package-stored-procedure-activity/deploying-data-factory.png)
9. Когда завершится создание, откроется страница **Фабрика данных**, как показано на рисунке ниже.
   
     ![Домашняя страница фабрики данных](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-home-page.png)
10. Щелкните плитку **Создание и мониторинг**, чтобы открыть на отдельной вкладке приложение пользовательского интерфейса службы "Фабрика данных Azure". 

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>Создание конвейера с действием хранимой процедуры
На этом этапе вы создадите конвейер, используя пользовательский интерфейс фабрики данных. Вы добавите действие хранимой процедуры в конвейер и настроите его для запуска пакета SSIS с помощью хранимой процедуры sp_executesql. 

1. На странице Начало работы щелкните **создать конвейер**. 

    ![Страница "Начало работы"](./media/how-to-invoke-ssis-package-stored-procedure-activity/get-started-page.png)
2. На панели **Действия** разверните элемент **Общие** и перетащите действие **Хранимая процедура** в область конструктора конвейера. 

    ![Перетаскивание действия "Хранимая процедура"](./media/how-to-invoke-ssis-package-stored-procedure-activity/drag-drop-sproc-activity.png)
3. В окне свойств действия хранимой процедуры перейдите на вкладку **Учетная запись SQL** и нажмите кнопку **+ Создать**. Вы создаете подключение к базе данных в базе данных SQL Azure, в которой размещен каталог служб SSIS (база данных SSIDB необходимо запустить). 
   
    ![Кнопка создания связанной службы](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-linked-service-button.png)
4. В окне **New Linked Service** (Новая связанная служба) выполните следующие действия: 

    1. В поле **Тип** выберите **База данных SQL Azure**.
    2. Выберите среду выполнения интеграции Azure **по умолчанию** для подключения к базе данных SQL Azure, в которой размещена база данных `SSISDB`.
    3. В поле **Имя сервера** выберите базу данных SQL Azure, в которой размещена база данных SSISDB.
    4. В поле **Имя базы данных** выберите **SSISDB**.
    5. В поле **Имя пользователя** введите имя пользователя, у которого есть доступ к базе данных.
    6. В поле **Пароль** введите пароль для этого пользователя. 
    7. Проверьте подключение к базе данных, нажав кнопку **Проверить соединение**.
    8. Сохраните связанную службу, нажав кнопку **Сохранить**. 

        ![Снимок экрана, на котором показан процесс добавления новой связанной службы.](./media/how-to-invoke-ssis-package-stored-procedure-activity/azure-sql-database-linked-service-settings.png)
5. В окне Свойства перейдите на вкладку **хранимая процедура** на вкладке **учетная запись SQL** и выполните следующие действия. 

    1. Выберите **Изменить**. 
    2. В поле **имя хранимой процедуры** введите `sp_executesql` . 
    3. Нажмите кнопку **+ Создать** в разделе **Параметры хранимой процедуры**. 
    4. В поле для **имени** параметра введите **stmt**. 
    5. В поле для **типа** параметра введите **String**. 
    6. В поле для **значения** параметра введите следующий SQL-запрос.

        В SQL-запросе укажите правильные значения для параметров **folder_name**, **project_name** и **package_name**. 

        ```sql
        DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER name in SSIS Catalog>', @project_name=N'<PROJECT name in SSIS Catalog>', @package_name=N'<PACKAGE name>.dtsx', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END
        ```

        ![Связанная служба "База данных SQL Azure"](./media/how-to-invoke-ssis-package-stored-procedure-activity/stored-procedure-settings.png)
6. Чтобы проверить конфигурацию конвейера, щелкните **Проверка** на панели инструментов. Чтобы закрыть **отчет о проверке конвейера**, нажмите кнопку **>>** .

    ![Проверка конвейера](./media/how-to-invoke-ssis-package-stored-procedure-activity/validate-pipeline.png)
7. Опубликуйте конвейер в фабрике данных, нажав кнопку **Опубликовать все**. 

    ![Публикация](./media/how-to-invoke-ssis-package-stored-procedure-activity/publish-all-button.png)    

### <a name="run-and-monitor-the-pipeline"></a>Запуск и мониторинг конвейера
В этом разделе вы активируете выполнение конвейера, а затем будете отслеживать его. 

1. Чтобы запустить конвейер, щелкните **триггер** на панели инструментов и выберите **Активировать сейчас**. 

    ![Trigger Now (Активировать сейчас)](media/how-to-invoke-ssis-package-stored-procedure-activity/trigger-now.png)

2. На странице **Запуск конвейера** нажмите кнопку **Готово**. 
3. Перейдите на вкладку **Мониторинг** слева. На ней отображается выполнение конвейера и его состояние вместе с другой информацией (например, время начала выполнения). Чтобы обновить это представление, щелкните **Refresh** (Обновить).

    ![Запуски конвейера](./media/how-to-invoke-ssis-package-stored-procedure-activity/pipeline-runs.png)

3. Щелкните ссылку **View Activity Runs** (Просмотр выполнений действий) в столбце **Actions** (Действия). Вы видите только одно выполнение действия, так как конвейер содержит только действие (действие хранимой процедуры).

    ![Выполнение действия](./media/how-to-invoke-ssis-package-stored-procedure-activity/activity-runs.png)

4. Можно выполнить следующий **запрос** к базе данных SSISDB в базе данных SQL, чтобы убедиться, что пакет выполнен. 

    ```sql
    select * from catalog.executions
    ```

    ![Проверка выполнения пакета](./media/how-to-invoke-ssis-package-stored-procedure-activity/verify-package-executions.png)


> [!NOTE]
> Вы также можете создать запланированный триггер для запуска конвейера по расписанию (ежечасно, ежедневно и т. д.). Пример см. в разделе [Запуск конвейера по расписанию](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule).

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

В этом разделе с помощью Azure PowerShell вы создадите конвейер фабрики данных с действием хранимой процедуры, которое вызывает пакет SSIS. 

Чтобы установить модули Azure PowerShell, выполните инструкции из статьи [Установка и настройка Azure PowerShell](/powershell/azure/install-az-ps). 

### <a name="create-a-data-factory"></a>Создание фабрики данных
Вы можете использовать ту же фабрику данных, в которой есть среда выполнения интеграции Azure SSIS, или создать отдельную. В следующей процедуре представлены шаги для создания фабрики данных. Вы создадите конвейер с действием хранимой процедуры в фабрике данных. Действие хранимой процедуры выполняет хранимую процедуру в базе данных SSISDB для запуска вашего пакета SSIS. 

1. Определите переменную для имени группы ресурсов, которую в дальнейшем можно будет использовать в командах PowerShell. Скопируйте текст следующей команды в PowerShell, укажите имя [группы ресурсов Azure](../azure-resource-manager/management/overview.md) в двойных кавычках, а затем выполните команду. Например: `"adfrg"`. 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    Если группа ресурсов уже существует, вы можете не перезаписывать ее. Назначьте переменной `$ResourceGroupName` другое значение и еще раз выполните команду.
2. Чтобы создать группу ресурсов Azure, выполните следующую команду: 

    ```powershell
    $ResGrp = New-AzResourceGroup $resourceGroupName -location 'eastus'
    ``` 
    Если группа ресурсов уже существует, вы можете не перезаписывать ее. Назначьте переменной `$ResourceGroupName` другое значение и еще раз выполните команду. 
3. Определите переменную для имени фабрики данных. 

    > [!IMPORTANT]
    >  Измените имя фабрики данных, чтобы оно было глобально уникальным. 

    ```powershell
    $DataFactoryName = "ADFTutorialFactory";
    ```

5. Чтобы создать фабрику данных, выполните следующий командлет **Set-AzDataFactoryV2**, используя свойства Location и ResourceGroupName из переменной $ResGrp. 
    
    ```powershell       
    $DataFactory = Set-AzDataFactoryV2 -ResourceGroupName $ResGrp.ResourceGroupName -Location $ResGrp.Location -Name $dataFactoryName 
    ```

Обратите внимание на следующие моменты.

* Имя фабрики данных Azure должно быть глобально уникальным. Если появляется следующая ошибка, измените имя и повторите попытку.

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```
* Чтобы создать экземпляры фабрики данных, нужно назначить учетной записи пользователя, используемой для входа в Azure, роль **участника**, **владельца** либо **администратора** подписки Azure.
* Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (служба хранилища Azure, база данных SQL Azure и т. д.) и вычисления (HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.

### <a name="create-an-azure-sql-database-linked-service"></a>Создание связанной службы Базы данных SQL Azure
Создайте связанную службу, чтобы связать базу данных, в которой размещен каталог служб SSIS, с фабрикой данных. Фабрика данных использует информацию связанной службы для подключения к базе данных SSISDB и выполняет хранимую процедуру для запуска пакета SSIS. 

1. В папке **C:\ADF\RunSSISPackage** создайте файл JSON с именем **AzureSqlDatabaseLinkedService.json** и следующим содержимым: 

    > [!IMPORTANT]
    > Перед сохранением файла замените &lt;servername&gt;, &lt;username&gt; и &lt;password&gt; значениями своей базы данных SQL Azure.

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=SSISDB;User ID=<username>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. В **Azure PowerShell** перейдите в папку **C:\ADF\RunSSISPackage**.

3. Выполните командлет **Set-AzDataFactoryV2LinkedService**, чтобы создать связанную службу. **AzureSqlDatabaseLinkedService**. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "AzureSqlDatabaseLinkedService" -File ".\AzureSqlDatabaseLinkedService.json"
    ```

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>Создание конвейера с действием хранимой процедуры 
На этом этапе создается конвейер с действием хранимой процедуры. Это действие вызывает хранимую процедуру sp_executesql для запуска пакета SSIS. 

1. Создайте файл JSON с именем **RunSSISPackagePipeline.json** в папке **C:\ADF\RunSSISPackage** со следующим содержимым:

    > [!IMPORTANT]
    > Прежде чем сохранять файл, замените заполнители &lt;folder name&gt;, &lt;project name&gt;, &lt;package name&gt; именами папки, проекта и пакета в каталоге SSIS соответственно. 

    ```json
    {
        "name": "RunSSISPackagePipeline",
        "properties": {
            "activities": [
                {
                    "name": "My SProc Activity",
                    "description":"Runs an SSIS package",
                    "type": "SqlServerStoredProcedure",
                    "linkedServiceName": {
                        "referenceName": "AzureSqlDatabaseLinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "typeProperties": {
                        "storedProcedureName": "sp_executesql",
                        "storedProcedureParameters": {
                            "stmt": {
                                "value": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER NAME>', @project_name=N'<PROJECT NAME>', @package_name=N'<PACKAGE NAME>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                            }
                        }
                    }
                }
            ]
        }
    }
    ```

2. Чтобы создать конвейер: **RunSSISPackagePipeline**, выполните командлет **Set-AzDataFactoryV2Pipeline** .

    ```powershell
    $DFPipeLine = Set-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "RunSSISPackagePipeline" -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

    Пример выходных данных:

    ```
    PipelineName      : Adfv2QuickStartPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopyFromBlobToBlob}
    Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="create-a-pipeline-run"></a>Создание конвейера
Используйте командлет **Invoke-AzDataFactoryV2Pipeline** для запуска конвейера. Командлет позволяет получить идентификатор выполнения конвейера для дальнейшего мониторинга.

```powershell
$RunId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineName $DFPipeLine.Name
```

### <a name="monitor-the-pipeline-run"></a>Мониторинг конвейера

Запустите приведенный ниже скрипт PowerShell, чтобы проверять состояние выполнения, пока не закончится копирование данных. Скопируйте приведенный ниже скрипт в окно PowerShell и нажмите клавишу ВВОД. 

```powershell
while ($True) {
    $Run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -PipelineRunId $RunId

    if ($Run) {
        if ($run.Status -ne 'InProgress') {
            Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
            $Run
            break
        }
        Write-Output  "Pipeline is running...status: InProgress"
    }

    Start-Sleep -Seconds 10
}   
```

### <a name="create-a-trigger"></a>Создание триггера
На предыдущем этапе вы выполняли вызов конвейера по запросу. Вы также можете создать триггер расписания для запуска конвейера по расписанию (ежечасно, ежедневно и т. д.).

1. Создайте файл JSON с именем **MyTrigger.json** в папке **C:\ADF\RunSSISPackage** со следующим содержимым: 

    ```json
    {
        "properties": {
            "name": "MyTrigger",
            "type": "ScheduleTrigger",
            "typeProperties": {
                "recurrence": {
                    "frequency": "Hour",
                    "interval": 1,
                    "startTime": "2017-12-07T00:00:00-08:00",
                    "endTime": "2017-12-08T00:00:00-08:00"
                }
            },
            "pipelines": [{
                    "pipelineReference": {
                        "type": "PipelineReference",
                        "referenceName": "RunSSISPackagePipeline"
                    },
                    "parameters": {}
                }
            ]
        }
    }    
    ```
2. В **Azure PowerShell** перейдите в папку **C:\ADF\RunSSISPackage**.
3. Выполните командлет **Set-AzDataFactoryV2Trigger** , который создает триггер. 

    ```powershell
    Set-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" -DefinitionFile ".\MyTrigger.json"
    ```
4. По умолчанию триггер находится в остановленном состоянии. Запустите триггер, выполнив командлет **Start-AzDataFactoryV2Trigger** . 

    ```powershell
    Start-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" 
    ```
5. Убедитесь, что триггер запущен, выполнив командлет **Get-AzDataFactoryV2Trigger** . 

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"     
    ```    
6. Через час выполните следующую команду. Например, если текущее время 15:25 (UTC), запустите команду в 16:00 (UTC). 
    
    ```powershell
    Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-06" -TriggerRunStartedBefore "2017-12-09"
    ```

    Можно выполнить следующий запрос к базе данных SSISDB в базе данных SQL, чтобы убедиться, что пакет выполнен. 

    ```sql
    select * from catalog.executions
    ```


## <a name="next-steps"></a>Дальнейшие действия
Вы также можете отслеживать конвейер с помощью портала Azure. Пошаговые инструкции см. в разделе [Мониторинг конвейера](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).
---
title: Настройка конвейера CI/CD для Azure Data Lake Analytics
description: Узнайте, как настроить непрерывную интеграцию и непрерывное развертывание для Azure Data Lake Analytics.
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 09/14/2018
ms.openlocfilehash: 95b638b85e0746d2995488f2a28a5fb2512b1063
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96015270"
---
# <a name="how-to-set-up-a-cicd-pipeline-for-azure-data-lake-analytics"></a>Настройка конвейера CI/CD для Azure Data Lake Analytics  

Из этой статьи вы узнаете, как настроить конвейер непрерывной интеграции и развертывания (далее — конвейер CI/CD) для заданий U-SQL и баз данных U-SQL.  

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="use-cicd-for-u-sql-jobs"></a>CI/CD для заданий U-SQL

Средства Azure Data Lake для Visual Studio предоставляют тип проекта U-SQL, который позволяет организовывать скрипты U-SQL. Использование проектов U-SQL для управления кодом U-SQL упрощает выполнение сценариев CI/CD.

## <a name="build-a-u-sql-project"></a>Создание проекта U-SQL

Проект U-SQL можно создать с помощью Microsoft Build Engine (MSBuild), передав соответствующие параметры. Выполните действия, описанные в этой статье, чтобы настроить процесс сборки для проекта U-SQL.

### <a name="project-migration"></a>Миграция проекта

Прежде чем настраивать задачу сборки для проекта U-SQL, убедитесь, что вы используете последнюю версию проекта U-SQL. Откройте файл проекта U-SQL в редакторе и убедитесь, что у вас есть следующие элементы импорта:

```xml
<!-- check for SDK Build target in current path then in USQLSDKPath-->
<Import Project="UsqlSDKBuild.targets" Condition="Exists('UsqlSDKBuild.targets')" />
<Import Project="$(USQLSDKPath)\UsqlSDKBuild.targets" Condition="!Exists('UsqlSDKBuild.targets') And '$(USQLSDKPath)' != '' And Exists('$(USQLSDKPath)\UsqlSDKBuild.targets')" />
```

В противном случае вы можете перенести проект двумя способами:

- Вариант 1. Замените старый элемент импорта приведенным выше.
- Вариант 2. Откройте старый проект в Средствах Azure Data Lake для Visual Studio. Используйте версию новее 2.3.3000.0. Старый шаблон проекта будет автоматически обновлен до последней версии. В новых проектах, созданных с помощью версий новее 2.3.3000.0, используется новый шаблон.

### <a name="get-nuget"></a>Получение пакета NuGet

MSBuild не предоставляет встроенной поддержки для проектов U-SQL. Чтобы реализовать такую возможность, включите ссылку на решение в пакет NuGet [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), который добавит необходимую языковую службу.

Чтобы добавить ссылку на пакет NuGet, в обозревателе решений Visual Studio щелкните решение правой кнопкой мыши и выберите **Управление пакетами NuGet**. Также можно добавить в папку решения файл с именем `packages.config`, включив в него следующее содержимое:

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180620" targetFramework="net452" />
</packages>
```

### <a name="manage-u-sql-database-references"></a>Управление ссылками на базу данных U-SQL

Скрипты U-SQL в проекте U-SQL могут содержать инструкции запросов для объектов базы данных U-SQL. В этом случае нужно до сборки проекта U-SQL указать соответствующий проект базы данных U-SQL, который включает определение объекта. Например, если вы создаете запрос к таблице U-SQL или ссылаетесь на сборку.

См. дополнительные сведения о [проектах базы данных U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md).

>[!NOTE]
> Инструкция DROP может вызвать случайное удаление. Чтобы включить инструкцию DROP, необходимо явно указать аргументы MSBuild. **Алловдропстатемент** ВКЛЮЧИТ операцию Drop, не связанную с данными, например DROP ASSEMBLY и функцию Drop с табличным значением. **Алловдатадропстатемент** ВКЛЮЧИТ операцию Drop, связанную с данными, например Drop Table и Drop Schema. Необходимо включить Алловдропстатемент перед использованием Алловдатадропстатемент.
>

### <a name="build-a-u-sql-project-with-the-msbuild-command-line"></a>Создание проекта U-SQL из командной строки MSBuild

Прежде всего выполните миграцию проекта и получите пакет NuGet. Затем вызовите стандартную командную строку MSBuild, указав следующие дополнительные аргументы для сборки проекта U-SQL:

```console
msbuild USQLBuild.usqlproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime;USQLTargetType=SyntaxCheck;DataRoot=datarootfolder;/p:EnableDeployment=true
```

Ниже приведены определения и значения аргументов.

- **Усклсдкпас = \<U-SQL Nuget package> \буилд\рунтиме**. Этот параметр ссылается на путь установки пакета NuGet для языковой службы U-SQL.
- **USQLTargetType=Merge или SyntaxCheck**:

  - **Слияние**. Режим слияния компилирует файлы с выделенным кодом. Примеры — файлы **CS**, **PY**, и **R**. В этом режиме полученная библиотека пользовательского кода включается в скрипт U-SQL. Этот режим используется для двоичных файлов DLL, кода Python или R.

  - **SyntaxCheck**. Режим SyntaxCheck сначала объединяет файлы с выделенным кодом в скрипт U-SQL. Затем он компилирует скрипт U-SQL для проверки кода.

- **Корневой каталог \<DataRoot path> =**. DataRoot используется только в режиме SyntaxCheck. Создавая скрипт в режиме SyntaxCheck, MSBuild проверяет включаемые в этот скрипт ссылки на объекты базы данных. Перед сборкой настройте на компьютере сборки локальную среду так, чтобы она содержала в папке DataRoot все объекты, на которые ссылается база данных U-SQL. Вы также можете управлять этими зависимостями базы данных, используя [ссылки на проект базы данных U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project). MSBuild проверяет только ссылки на объекты базы данных, но не файлы.

- **EnableDeployment=true** или **false**. EnableDeployment указывает, разрешено ли развертывать указанные базы данных U-SQL во время процесса сборки. Если вы ссылаетесь на проект базы данных U-SQL и используете объекты базы данных в скрипте U-SQL, установите для этого параметра значение **true**.

### <a name="continuous-integration-through-azure-pipelines"></a>Непрерывная интеграция с помощью Azure Pipelines

Кроме командной строки вы можете также использовать задачу Visual Studio Build или MSBuild для сборки проектов U-SQL в Azure Pipelines. Чтобы настроить конвейер сборки, обязательно добавьте в него две задачи: задачу восстановления NuGet и задачу MSBuild.

![Задача MSBuild для проекта U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png)

1. Добавьте задачу восстановления NuGet, чтобы получить пакет NuGet со ссылкой на решение, который включает `Azure.DataLake.USQL.SDK`, чтобы предоставить возможность MSBuild найти целевые объекты языка U-SQL. Задайте для параметра **Расширенный**  >  **Каталог назначения** значение, `$(Build.SourcesDirectory)/packages` Если вы хотите использовать пример аргументов MSBuild непосредственно на шаге 2.

   ![Задача восстановления NuGet для проекта U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2. Задайте аргументы MSBuild в средствах сборки Visual Studio или в задаче MSBuild, как показано в следующем примере. Вы также можете определить переменные для этих аргументов в конвейере сборки Azure Pipelines.

   ![Определение переменных MSBuild CI/CD для проекта U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables.png)

   ```console
   /p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime /p:USQLTargetType=SyntaxCheck /p:DataRoot=$(Build.SourcesDirectory) /p:EnableDeployment=true
   ```

### <a name="u-sql-project-build-output"></a>Выходные данные сборки проекта U-SQL

После запуска сборки все скрипты проекта U-SQL компилируются и сохраняются в ZIP-файл с именем `USQLProjectName.usqlpack`. Структура папок в проекте хранится в выходных данных сборки в формате ZIP.

> [!NOTE]
> Файлы кода программной части для каждого скрипта U-SQL будут объединены как встроенный оператор в выходные данные построения скрипта.

## <a name="test-u-sql-scripts"></a>Тестирование скриптов U-SQL

Azure Data Lake предоставляет тестовые проекты для скриптов U-SQL и кода C# (UDO/UDAG/UDF).

- Узнайте, как [добавить тестовые случаи для скриптов U-SQL и расширенного кода C#](data-lake-analytics-cicd-test.md#test-u-sql-scripts)

- См. дополнительные сведения о [запуске тестовых случаев Azure Pipelines](data-lake-analytics-cicd-test.md#run-test-cases-in-azure-devops).

## <a name="deploy-a-u-sql-job"></a>Развертывание задания U-SQL

Проверив код в ходе сборки и тестирования, вы можете отправить задания U-SQL непосредственно из Azure Pipelines с помощью задания Azure PowerShell. Вы также можете развернуть скрипт в Azure Data Lake Storage или хранилище BLOB-объектов, а затем [выполнить запланированные задания в Фабрике данных Azure](../data-factory/transform-data-using-data-lake-analytics.md).

### <a name="submit-u-sql-jobs-through-azure-pipelines"></a>Отправка заданий U-SQL через Azure Pipelines

Выходные данные сборки проекта U-SQL — это ZIP-файл с именем **USQLProjectName.usqlpack**. ZIP-файл содержит все скрипты U-SQL в проекте. Вы можете использовать [задачи Azure PowerShell](/azure/devops/pipelines/tasks/deploy/azure-powershell) в Pipelines со следующим образцом скрипта PowerShell для отправки заданий U-SQL непосредственно из Azure Pipelines.

```powershell
<#
   This script can be used to submit U-SQL Jobs with given U-SQL project build output(.usqlpack file).
   This will unzip the U-SQL project build output, and submit all scripts one-by-one.

   Note: the code behind file for each U-SQL script will be merged into the built U-SQL script in build output.

   Example :
      USQLJobSubmission.ps1 -ADLAAccountName "myadlaaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\" -DegreeOfParallelism 2
#>

param(
   [Parameter(Mandatory=$true)][string]$ADLAAccountName, # ADLA account name to submit U-SQL jobs
   [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
   [Parameter(Mandatory=$false)][string]$DegreeOfParallelism = 1
)

function Unzip($USQLPackfile, $UnzipOutput)
{
   $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
   Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
   Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

## Get U-SQL scripts in U-SQL project build output(.usqlpack file)
Function GetUsqlFiles()
{

   $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

   $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

   foreach ($USQLPackfile in $USQLPackfiles)
   {
      Unzip $USQLPackfile $UnzipOutput
   }

   $USQLFiles = Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue

   return $USQLFiles
}

## Submit U-SQL scripts to ADLA account one-by-one
Function SubmitAnalyticsJob()
{
   $usqlFiles = GetUsqlFiles

   Write-Output "$($usqlFiles.Count) jobs to be submitted..."

   # Submit each usql script and wait for completion before moving ahead.
   foreach ($usqlFile in $usqlFiles)
   {
      $scriptName = "[Release].[$([System.IO.Path]::GetFileNameWithoutExtension($usqlFile.fullname))]"

      Write-Output "Submitting job for '{$usqlFile}'"

      $jobToSubmit = Submit-AzDataLakeAnalyticsJob -Account $ADLAAccountName -Name $scriptName -ScriptPath $usqlFile -DegreeOfParallelism $DegreeOfParallelism

      LogJobInformation $jobToSubmit

      Write-Output "Waiting for job to complete. Job ID:'{$($jobToSubmit.JobId)}', Name: '$($jobToSubmit.Name)' "
      $jobResult = Wait-AzDataLakeAnalyticsJob -Account $ADLAAccountName -JobId $jobToSubmit.JobId  
      LogJobInformation $jobResult
   }
}

Function LogJobInformation($jobInfo)
{
   Write-Output "************************************************************************"
   Write-Output ([string]::Format("Job Id: {0}", $(DefaultIfNull $jobInfo.JobId)))
   Write-Output ([string]::Format("Job Name: {0}", $(DefaultIfNull $jobInfo.Name)))
   Write-Output ([string]::Format("Job State: {0}", $(DefaultIfNull $jobInfo.State)))
   Write-Output ([string]::Format("Job Started at: {0}", $(DefaultIfNull  $jobInfo.StartTime)))
   Write-Output ([string]::Format("Job Ended at: {0}", $(DefaultIfNull  $jobInfo.EndTime)))
   Write-Output ([string]::Format("Job Result: {0}", $(DefaultIfNull $jobInfo.Result)))
   Write-Output "************************************************************************"
}

Function DefaultIfNull($item)
{
   if ($item -ne $null)
   {
      return $item
   }
   return ""
}

Function Main()
{
   Write-Output ([string]::Format("ADLA account: {0}", $ADLAAccountName))
   Write-Output ([string]::Format("Root folde for usqlpack: {0}", $ArtifactsRoot))
   Write-Output ([string]::Format("AU count: {0}", $DegreeOfParallelism))

   Write-Output "Starting USQL script deployment..."

   SubmitAnalyticsJob

   Write-Output "Finished deployment..."
}

Main
```

>[!NOTE]
> Команды: `Submit-AzDataLakeAnalyticsJob` и `Wait-AzDataLakeAnalyticsJob` являются Azure PowerShell командлетами для Azure Data Lake Analytics в Azure Resource Manager Framework. Вы нужны рабочую станцию с установленными Azure PowerShell. Для получения дополнительных команд и примеров можно обратиться к [списку команд](/powershell/module/Az.DataLakeAnalytics) .
>

### <a name="deploy-u-sql-jobs-through-azure-data-factory"></a>Планирование заданий U-SQL в Фабрике данных Azure

Вы можете отправлять задания U-SQL непосредственно из Azure Pipelines. Вы также можете передать скрипты сборки в Azure Data Lake Storage или хранилище BLOB-объектов, а затем [выполнить запланированные задания в Фабрике данных Azure](../data-factory/transform-data-using-data-lake-analytics.md).

Используйте [задачи Azure PowerShell](/azure/devops/pipelines/tasks/deploy/azure-powershell) в Azure Pipelines со следующим образцом скрипта PowerShell для отправки скриптов U-SQL в ученую запись Azure Data Lake Store:

```powershell
<#
   This script can be used to upload U-SQL files to ADLS with given U-SQL project build output(.usqlpack file).
   This will unzip the U-SQL project build output, and upload all scripts to ADLS one-by-one.

   Example :
      FileUpload.ps1 -ADLSName "myadlsaccount" -ArtifactsRoot "C:\USQLProject\bin\debug\"
#>

param(
   [Parameter(Mandatory=$true)][string]$ADLSName, # ADLS account name to upload U-SQL scripts
   [Parameter(Mandatory=$true)][string]$ArtifactsRoot, # Root folder of U-SQL project build output
   [Parameter(Mandatory=$false)][string]$DestinationFolder = "USQLScriptSource" # Destination folder in ADLS
)

Function UploadResources()
{
   Write-Host "************************************************************************"
   Write-Host "Uploading files to $ADLSName"
   Write-Host "***********************************************************************"

   $usqlScripts = GetUsqlFiles

   $files = @(get-childitem $usqlScripts -recurse)
   foreach($file in $files)
   {
      Write-Host "Uploading file: $($file.Name)"
      Import-AzDataLakeStoreItem -AccountName $ADLSName -Path $file.FullName -Destination "/$(Join-Path $DestinationFolder $file)" -Force
   }
}

function Unzip($USQLPackfile, $UnzipOutput)
{
   $USQLPackfileZip = Rename-Item -Path $USQLPackfile -NewName $([System.IO.Path]::ChangeExtension($USQLPackfile, ".zip")) -Force -PassThru
   Expand-Archive -Path $USQLPackfileZip -DestinationPath $UnzipOutput -Force
   Rename-Item -Path $USQLPackfileZip -NewName $([System.IO.Path]::ChangeExtension($USQLPackfileZip, ".usqlpack")) -Force
}

Function GetUsqlFiles()
{

   $USQLPackfiles = Get-ChildItem -Path $ArtifactsRoot -Include *.usqlpack -File -Recurse -ErrorAction SilentlyContinue

   $UnzipOutput = Join-Path $ArtifactsRoot -ChildPath "UnzipUSQLScripts"

   foreach ($USQLPackfile in $USQLPackfiles)
   {
      Unzip $USQLPackfile $UnzipOutput
   }

   return Get-ChildItem -Path $UnzipOutput -Include *.usql -File -Recurse -ErrorAction SilentlyContinue
}

UploadResources
```

## <a name="cicd-for-a-u-sql-database"></a>CI/CD для базы данных U-SQL

Средства Azure Data Lake для Visual Studio предоставляют шаблон проекта базы данных U-SQL, который поможет вам выполнять разработку, развертывание и администрирование баз данных U-SQL. См. дополнительные сведения о [проекте базы данных U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md).

## <a name="build-u-sql-database-project"></a>Создание проекта базы данных U-SQL

### <a name="get-the-nuget-package"></a>Получение пакета NuGet

MSBuild не имеет встроенной поддержки для проектов базы данных U-SQL. Чтобы получить эту возможность, включите ссылку на решение в пакет NuGet [Microsoft.Azure.DataLake.USQL.SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), который добавляет необходимую языковую службу.

Чтобы добавить ссылку на пакет NuGet, в обозревателе решений Visual Studio щелкните решение правой кнопкой мыши. Выберите **Управление пакетами NuGet**. Затем выполните поиск нужного пакета NuGet и установите его. Также вы можете добавить в папку решения файл с именем **packages.config** и поместить в него следующее содержимое:

```xml
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.DataLake.USQL.SDK" version="1.3.180615" targetFramework="net452" />
</packages>
```

### <a name="build-u-sql-a-database-project-with-the-msbuild-command-line"></a>Сборка проекта базы данных U-SQL из командной строки MSBuild

Чтобы скомпилировать проект базы данных U-SQL, вызовите стандартную командную строку MSBuild и передайте ссылку на пакет U-SQL SDK для NuGet в качестве дополнительного аргумента. См. следующий пример. 

```console
msbuild DatabaseProject.usqldbproj /p:USQLSDKPath=packages\Microsoft.Azure.DataLake.USQL.SDK.1.3.180615\build\runtime
```

Аргумент `USQLSDKPath=<U-SQL Nuget package>\build\runtime` ссылается на путь установки пакета NuGet для языковой службы U-SQL.

### <a name="continuous-integration-with-azure-pipelines"></a>Непрерывная интеграция с Azure Pipelines

Кроме командной строки вы можете использовать задачу Visual Studio Build или MSBuild для сборки проектов U-SQL в Azure Pipelines. Чтобы настроить задачу сборки, обязательно добавьте в конвейер сборки две задачи: задачу восстановления NuGet и задачу MSBuild.

   ![Задача MSBuild CI/CD для проекта U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-task.png)

1. Добавьте задачу восстановления NuGet, чтобы получить пакет NuGet со ссылкой на решение, который включает `Azure.DataLake.USQL.SDK`, чтобы MSBuild смог найти целевые объекты языка U-SQL. Задайте для параметра **Расширенный**  >  **Каталог назначения** значение, `$(Build.SourcesDirectory)/packages` Если вы хотите использовать пример аргументов MSBuild непосредственно на шаге 2.

   ![Задача NuGet CI/CD для проекта U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-nuget-task.png)

2. Задайте аргументы MSBuild в средствах сборки Visual Studio или в задаче MSBuild, как показано в следующем примере. Вы также можете определить переменные для этих аргументов в конвейере сборки Azure Pipelines.

   ![Определение переменных MSBuild CI/CD для проекта базы данных U-SQL](./media/data-lake-analytics-cicd-overview/data-lake-analytics-set-vsts-msbuild-variables-database-project.png)

   ```console
   /p:USQLSDKPath=$(Build.SourcesDirectory)/packages/Microsoft.Azure.DataLake.USQL.SDK.1.3.180615/build/runtime
   ```

### <a name="u-sql-database-project-build-output"></a>Выходные данные сборки проекта базы данных U-SQL

Выходные данные сборки для проекта базы данных U-SQL представляют собой пакет развертывания базы данных U-SQL с суффиксом `.usqldbpack`. Пакет `.usqldbpack` представляет собой ZIP-файл, который собирает все инструкции DDL в единый скрипт U-SQL в папке DDL. Во временной папке собраны все **DLL-файлы** и другие дополнительные файлы для сборки.

## <a name="test-table-valued-functions-and-stored-procedures"></a>Тестирование функций с табличным значением и хранимых процедур

Добавление тестовых случаев для функций с табличным значением и хранимых процедур сейчас не поддерживается. Но вы можете создать проект U-SQL со скриптами U-SQL, которые вызывают эти функции, а затем написать тестовые случаи для них. Чтобы настроить тестовые случаи для функций с табличным значением и хранимых процедур, определенных в проекте базы данных U-SQL, выполните следующие шаги.

1. Создайте проект U-SQL для тестовых целей и напишите скрипты U-SQL, которые вызывают функции с табличным значением и хранимые процедуры.

1. Добавьте ссылку на базу данных в этот проект U-SQL. Чтобы получить определение хранимой процедуры и функций с табличным значением, создайте ссылку на проект базы данных, который содержит инструкцию DDL. См. дополнительные сведения о [создании ссылок на базы данных](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project).

1. Добавьте тестовые случаи для скриптов U-SQL, которые вызывают функции с табличным значением и хранимые процедуры. Узнайте, как [добавлять тестовые случаи для скриптов U-SQL](data-lake-analytics-cicd-test.md#test-u-sql-scripts).

## <a name="deploy-u-sql-database-through-azure-pipelines"></a>Развертывание U-SQL database в Azure Pipelines

`PackageDeploymentTool.exe` предоставляет программный интерфейс и интерфейс командной строки, которые помогают развернуть пакеты развертывания базы данных U-SQL (**.usqldbpack**). Этот пакет SDK включен в [пакет SDK NuGet для U-SQL](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), размещенный в файле **build/runtime/PackageDeploymentTool.exe**. С помощью `PackageDeploymentTool.exe` можно развертывать базы данных U-SQL в Azure Data Lake Analytics и локальных учетных записях.

> [!NOTE]
>
> Поддержка командной строки PowerShell и задач выпуска Azure Pipelines для развертывания баз данных U-SQL будет реализована позже.
>

Чтобы настроить задачу развертывания базы данных в Azure Pipelines, выполните следующие шаги:

1. Добавьте в конвейер сборки или выпуска задачу скрипта PowerShell,которая выполняет следующий скрипт PowerShell. Эта задача позволяет получить зависимости пакета Azure SDK для `PackageDeploymentTool.exe` и `PackageDeploymentTool.exe`. Можно настроить параметры **-AzureSDK** и **-DBDeploymentTool** для загрузки зависимостей и средства развертывания в определенные папки. На шаге 2 передайте путь **-AzureSDK** в `PackageDeploymentTool.exe` как параметр **-AzureSDKPath**.

   ```powershell
   <#
      This script is used for getting dependencies and SDKs for U-SQL database deployment.
      PowerShell command line support for deploying U-SQL database package(.usqldbpack file) will come soon.

      Example :
          GetUSQLDBDeploymentSDK.ps1 -AzureSDK "AzureSDKFolderPath" -DBDeploymentTool "DBDeploymentToolFolderPath"
   #>

   param (
      [string]$AzureSDK = "AzureSDK", # Folder to cache Azure SDK dependencies
      [string]$DBDeploymentTool = "DBDeploymentTool", # Folder to cache U-SQL database deployment tool
      [string]$workingfolder = "" # Folder to execute these command lines
   )

   if ([string]::IsNullOrEmpty($workingfolder))
   {
      $scriptpath = $MyInvocation.MyCommand.Path
      $workingfolder = Split-Path $scriptpath
   }
   cd $workingfolder

   echo "workingfolder=$workingfolder, outputfolder=$outputfolder"
   echo "Downloading required packages..."

   iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Analytics/3.5.1-preview -outf Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.Management.DataLake.Store/2.4.1-preview -outf Microsoft.Azure.Management.DataLake.Store.2.4.1-preview.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.IdentityModel.Clients.ActiveDirectory/2.28.3 -outf Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime/2.3.11 -outf Microsoft.Rest.ClientRuntime.2.3.11.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure/3.3.7 -outf Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Rest.ClientRuntime.Azure.Authentication/2.3.3 -outf Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip
   iwr https://www.nuget.org/api/v2/package/Newtonsoft.Json/6.0.8 -outf Newtonsoft.Json.6.0.8.zip
   iwr https://www.nuget.org/api/v2/package/Microsoft.Azure.DataLake.USQL.SDK/ -outf USQLSDK.zip

   echo "Extracting packages..."

   Expand-Archive Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview -Force
   Expand-Archive Microsoft.Azure.Management.DataLake.Store.2.4.1-preview.zip -DestinationPath Microsoft.Azure.Management.DataLake.Store.2.4.1-preview -Force
   Expand-Archive Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3.zip -DestinationPath Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3 -Force
   Expand-Archive Microsoft.Rest.ClientRuntime.2.3.11.zip -DestinationPath Microsoft.Rest.ClientRuntime.2.3.11 -Force
   Expand-Archive Microsoft.Rest.ClientRuntime.Azure.3.3.7.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.3.3.7 -Force
   Expand-Archive Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3.zip -DestinationPath Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3 -Force
   Expand-Archive Newtonsoft.Json.6.0.8.zip -DestinationPath Newtonsoft.Json.6.0.8 -Force
   Expand-Archive USQLSDK.zip -DestinationPath USQLSDK -Force

   echo "Copy required DLLs to output folder..."

   mkdir $AzureSDK -Force
   mkdir $DBDeploymentTool -Force
   copy Microsoft.Azure.Management.DataLake.Analytics.3.5.1-preview\lib\net452\*.dll $AzureSDK
   copy Microsoft.Azure.Management.DataLake.Store.2.4.1-preview\lib\net452\*.dll $AzureSDK
   copy Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\*.dll $AzureSDK
   copy Microsoft.Rest.ClientRuntime.2.3.11\lib\net452\*.dll $AzureSDK
   copy Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\*.dll $AzureSDK
   copy Microsoft.Rest.ClientRuntime.Azure.Authentication.2.3.3\lib\net452\*.dll $AzureSDK
   copy Newtonsoft.Json.6.0.8\lib\net45\*.dll $AzureSDK
   copy USQLSDK\build\runtime\*.* $DBDeploymentTool
   ```

2. Добавьте **задачу командной строки** в конвейер сборки или выпуска и создайте для него скрипт, вызвав `PackageDeploymentTool.exe`. Файл `PackageDeploymentTool.exe` расположен в папке, заданной параметром **$DBDeploymentTool**. Пример скрипта: 

   - Разверните базу данных U-SQL в локальной среде.

      ```cmd
      PackageDeploymentTool.exe deploylocal -Package <package path> -Database <database name> -DataRoot <data root path>
      ```

   - Используйте режим интерактивной аутентификации, чтобы развернуть базу данных U-SQL в учетной записи Azure Data Lake Analytics.

      ```cmd
      PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -AzureSDKPath <azure sdk path> -Interactive
      ```

   - Используйте аутентификацию **secret**, чтобы развернуть базу данных U-SQL в учетной записи Azure Data Lake Analytics.

      ```cmd
      PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete>
      ```

   - Используйте аутентификацию **certFile**, чтобы развернуть базу данных U-SQL в учетной записи Azure Data Lake Analytics.

      ```cmd
      PackageDeploymentTool.exe deploycluster -Package <package path> -Database <database name> -Account <account name> -ResourceGroup <resource group name> -SubscriptionId <subscript id> -Tenant <tenant name> -ClientId <client id> -Secrete <secrete> -CertFile <certFile>
      ```

### <a name="packagedeploymenttoolexe-parameter-descriptions"></a>Описание параметров файла PackageDeploymentTool.exe.

#### <a name="common-parameters"></a>Общие параметры

| Параметр | Описание | Значение по умолчанию | Обязательно |
|---------|-----------|-------------|--------|
|Пакет|Путь к развертываемому пакету развертывания базы данных U-SQL.|null|true|
|База данных|Имя развертываемой или создаваемой базы данных.|master|false|
|LogFile|Путь к файлу для ведения журнала. По умолчанию используется стандартный выход (консоль).|null|false|
|LogLevel|Уровень ведения журнала: подробный, обычный, предупреждения, ошибки.|LogLevel.Normal|false|

#### <a name="parameter-for-local-deployment"></a>Параметр для локального развертывания

|Параметр|Описание|Значение по умолчанию|Обязательно|
|---------|-----------|-------------|--------|
|DataRoot|Локальный путь к корневой папке данных.|null|true|

#### <a name="parameters-for-azure-data-lake-analytics-deployment"></a>Параметры для развертывания Azure Data Lake Analytics

|Параметр|Описание|Значение по умолчанию|Обязательно|
|---------|-----------|-------------|--------|
|Учетная запись|Определяет имя учетной записи Azure Data Lake Analytics, в которую выполняется развертывание.|null|true|
|ResourceGroup|Имя группы ресурсов Azure для учетной записи Azure Data Lake Analytics.|null|true|
|SubscriptionId|Идентификатор подписки Azure для учетной записи Azure Data Lake Analytics.|null|true|
|Клиент|Имя клиента, то есть имя домена Azure Active Directory (Azure AD). Это значение вы найдете на странице управления подпиской на портале Azure.|null|true|
|AzureSDKPath|Путь для поиска зависимых сборок в пакете Azure SDK.|null|true|
|Интерактивно|Определяет, нужно ли использовать интерактивный режим для аутентификации.|false|false|
|ClientId|Идентификатор приложения в Azure AD, используемый для неинтерактивной аутентификации.|null|Обязательно, если не используется интерактивная аутентификация.|
|Секрет|Секрет или пароль для неинтерактивной аутентификации. Следует использовать только в доверенной и защищенной среде.|null|Обязательно для неинтерактивной аутентификации (иначе используется SecreteFile).|
|SecreteFile|В этом файле хранится секрет или пароль для неинтерактивной аутентификации. Следите за тем, чтобы права на чтение этого файла были только у текущего пользователя.|null|Обязательно для неинтерактивной аутентификации (иначе используется Secrete).|
|CertFile|В этом файле хранится сертификат X.509 для неинтерактивной аутентификации. По умолчанию используется аутентификация с помощью секрета клиента.|null|false|
| JobPrefix | Префикс для развертывания базы данных в задании DDL U-SQL. | Deploy_ + DateTime.Now | false |

## <a name="next-steps"></a>Дальнейшие действия

- [Тестирование кода Azure Data Lake Analytics](data-lake-analytics-cicd-test.md).
- [Выполните скрипт U-SQL на локальном компьютере](data-lake-analytics-data-lake-tools-local-run.md).
- [Использование проекта базы данных U-SQL в разработке базы данных U-SQL для Azure Data Lake](data-lake-analytics-data-lake-tools-develop-usql-database.md).
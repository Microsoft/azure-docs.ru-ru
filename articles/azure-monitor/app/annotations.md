---
title: Заметки о выпуске для Application Insights | Документация Майкрософт
description: Добавление маркеров развертывания или сборки для диаграмм обозревателя метрик в Application Insights.
ms.topic: conceptual
ms.date: 08/14/2020
ms.openlocfilehash: 776efd56aaa523d1c2621c51cba0446a42bb7411
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103461918"
---
# <a name="annotations-on-metric-charts-in-application-insights"></a>Заметки к диаграммам метрик в Application Insights

Заметки показывают, где развертывается новая сборка или другие важные события. Заметки позволяют легко определить, влияют ли ваши изменения на производительность приложения. Они могут автоматически создаваться системой сборки [Azure pipelines](/azure/devops/pipelines/tasks/) . Заметки можно также создавать, чтобы помечать какие-либо события, используя PowerShell.

## <a name="release-annotations-with-azure-pipelines-build"></a>Заметки о выпуске с Azure Pipelines сборкой

Заметки о выпуске — это функция облачной Azure Pipelines службы Azure DevOps.

### <a name="install-the-annotations-extension-one-time"></a>Установка расширения заметок (однократно)

Чтобы иметь возможность создавать заметки о выпуске, необходимо установить одно из многих расширений Azure DevOps, доступных в Visual Studio Marketplace.

1. Войдите в проект [Azure DevOps](https://azure.microsoft.com/services/devops/) .
   
1. На странице [расширение заметок к Выпуску](https://marketplace.visualstudio.com/items/ms-appinsights.appinsightsreleaseannotations) Visual Studio Marketplace выберите свою организацию Azure DevOps и нажмите кнопку **установить** , чтобы добавить расширение в организацию Azure DevOps.
   
   ![Выберите организацию Azure DevOps и нажмите кнопку установить.](./media/annotations/1-install.png)
   
Это расширение необходимо установить только один раз для вашей организации Azure DevOps. Теперь можно настроить заметки к выпуску для любого проекта в Организации.

### <a name="configure-release-annotations"></a>Настройка заметок к выпуску

Создайте отдельный ключ API для каждого из шаблонов выпуска Azure Pipelines.

1. Войдите в [портал Azure](https://portal.azure.com) и откройте ресурс Application Insights, который наблюдает за приложением. Если у вас ее нет, [Создайте новый ресурс Application Insights](./app-insights-overview.md).
   
1. Откройте вкладку **доступ к API** и скопируйте **идентификатор Application Insights**.
   
   ![В разделе доступ через API скопируйте идентификатор приложения.](./media/annotations/2-app-id.png)

1. В отдельном окне браузера откройте или создайте шаблон выпуска, который управляет развертываниями Azure Pipelines.
   
1. Выберите **Добавить задачу**, а затем в меню выберите задачу **заметки о выпуске Application Insights** .
   
   ![Выберите Добавить задачу и выберите Application Insights заметки о выпуске.](./media/annotations/3-add-task.png)

   > [!NOTE]
   > Сейчас задача заметки о выпуске поддерживает только агенты на основе Windows. Он не будет выполняться в Linux, macOS или других типах агентов.
   
1. В поле **Application ID (идентификатор приложения**) вставьте идентификатор Application Insights, скопированный на вкладке **доступ к API** .
   
   ![Вставьте идентификатор Application Insights](./media/annotations/4-paste-app-id.png)
   
1. Вернитесь в окно Application Insights **доступа к API** и выберите **создать ключ API**. 
   
   ![На вкладке доступ через API выберите Создать ключ API.](./media/annotations/5-create-api-key.png)
   
1. В окне **Создание ключа API** введите описание, выберите **записать заметки** и нажмите кнопку **создать ключ**. Скопируйте новый ключ.
   
   ![В окне Создание ключа API введите описание, выберите записать заметки и нажмите кнопку Создать ключ.](./media/annotations/6-create-api-key.png)
   
1. В окне шаблон выпуска на вкладке **переменные** выберите **Добавить** , чтобы создать определение переменной для нового ключа API.

1. В разделе **имя** введите `ApiKey` и в поле **значение** вставьте ключ API, скопированный на вкладке **доступ к API** .
   
   ![На вкладке DevOps переменные Azure выберите Добавить, назовите переменную ApiKey и вставьте ключ API в поле значение.](./media/annotations/7-paste-api-key.png)
   
1. Выберите **сохранить** в главном окне шаблона выпуска, чтобы сохранить шаблон.


   > [!NOTE]
   > Ограничения для ключей API описаны в документации по [ограничениям скорости REST API](https://dev.applicationinsights.io/documentation/Authorization/Rate-limits).

## <a name="view-annotations"></a>Просмотр заметок


   > [!NOTE]
   > Заметки о выпуске в настоящее время недоступны на панели метрик Application Insights

Теперь при использовании шаблона выпуска для развертывания нового выпуска заметка отправляется в Application Insights. Заметки можно просмотреть в следующих расположениях:

На панели **Использование** вы также можете вручную создавать заметки о выпуске:

![Снимок экрана линейчатой диаграммы с количеством посещений пользователей, отображаемых за определенный период времени. Заметки о выпуске отображаются в виде зеленых флажков над диаграммой, указывающей на момент возникновения выпуска.](./media/annotations/usage-pane.png)

В любом запросе книги на основе журнала, где визуализация отображает время на оси x.

![Снимок экрана: панель "книги" с запросом на основе журнала временных рядов с отображаемыми заметками](./media/annotations/workbooks-annotations.png)

Чтобы включить заметки в книге, перейдите в раздел **Дополнительные параметры** и выберите пункт **Показывать заметки**.

![Снимок экрана меню дополнительных параметров с словами показывать заметки, выделенные с галочкой рядом с параметром, чтобы включить его.](./media/annotations/workbook-show-annotations.png)

Выберите любой маркер заметки, чтобы открыть сведения о выпуске, включая запрашивающей стороны, ветвь системы управления версиями, конвейер выпуска и среду.

## <a name="create-custom-annotations-from-powershell"></a>Создание настраиваемых заметок в PowerShell
Вы можете использовать сценарий PowerShell Креатерелеасеаннотатион из GitHub для создания заметок из любого процесса, не используя Azure DevOps.

1. Создайте локальную копию CreateReleaseAnnotation.ps1:

    ```powershell
    
    # Copyright (c) Microsoft Corporation. All rights reserved. 
    # Licensed under the MIT License. See License.txt in the project root for license information. 
    
    # Sample usage .\CreateReleaseAnnotation.ps1 -applicationId "<appId>" -apiKey "<apiKey>" -releaseFilePath "<path to .exe with file version>" -releaseProperties @{"ReleaseDescription"="Release with annotation";"TriggerBy"="John Doe"}
    param(
        [parameter(Mandatory = $true)][string]$applicationId,
        [parameter(Mandatory = $true)][string]$apiKey,
        [parameter(Mandatory = $true)][string]$releaseFilePath,
        [parameter(Mandatory = $false)]$releaseProperties
    )
    
    $releaseName = (Get-Item $releaseFilePath).VersionInfo.FileVersion
    Write-Host "Creating release annotation $releaseName in ApplicationInsights" -ForegroundColor Cyan
    
    # background info on how fwlink works: After you submit a web request, many sites redirect through a series of intermediate pages before you finally land on the destination page.
    # So when calling Invoke-WebRequest, the result it returns comes from the final page in any redirect sequence. Hence, I set MaximumRedirection to 0, as this prevents the call to 
    # be redirected. By doing this, we get a response with status code 302, which indicates that there is a redirection link from the response body. We grab this redirection link and 
    # construct the url to make a release annotation.
    # Here's how this logic is going to works
    # 1. Client send http request, such as:  http://go.microsoft.com/fwlink/?LinkId=625115
    # 2. FWLink get the request and find out the destination URL for it, such as:  http://www.bing.com
    # 3. FWLink generate a new http response with status code “302” and with destination URL “http://www.bing.com”. Send it back to Client.
    # 4. Client, such as a powershell script, knows that status code “302” means redirection to new a location, and the target location is “http://www.bing.com”
    function GetRequestUrlFromFwLink($fwLink)
    {
        $request = Invoke-WebRequest -Uri $fwLink -MaximumRedirection 0 -UseBasicParsing -ErrorAction Ignore
        if ($request.StatusCode -eq "302") {
            return $request.Headers.Location
        }
        
        return $null
    }
    
    function CreateAnnotation($grpEnv)
    {
        $retries = 1
        $success = $false
        while (!$success -and $retries -lt 6) {
            $location = "$grpEnv/applications/$applicationId/Annotations?api-version=2015-11"
                
            Write-Host "Invoke a web request for $location to create a new release annotation. Attempting $retries"
            set-variable -Name createResultStatus -Force -Scope Local -Value $null
            set-variable -Name createResultStatusDescription -Force -Scope Local -Value $null
            set-variable -Name result -Force -Scope Local
    
            try {
                $result = Invoke-WebRequest -Uri $location -Method Put -Body $bodyJson -Headers $headers -ContentType "application/json; charset=utf-8" -UseBasicParsing
            } catch {
                if ($_.Exception){
                    if($_.Exception.Response) {
                        $createResultStatus = $_.Exception.Response.StatusCode.value__
                        $createResultStatusDescription = $_.Exception.Response.StatusDescription
                    }
                    else {
                        $createResultStatus = "Exception"
                        $createResultStatusDescription = $_.Exception.Message
                    }
                }
            }
    
            if ($result -eq $null) {
                if ($createResultStatus -eq $null) {
                    $createResultStatus = "Unknown"
                }
                if ($createResultStatusDescription -eq $null) {
                    $createResultStatusDescription = "Unknown"
                }
            }
            else {
                    $success = $true                     
            }
    
            if ($createResultStatus -eq 409 -or $createResultStatus -eq 404 -or $createResultStatus -eq 401) # no retry when conflict or unauthorized or not found
            {
                break
            }
    
            $retries = $retries + 1
            sleep 1
        }
    
        $createResultStatus
        $createResultStatusDescription
        return
    }
    
    # Need powershell version 3 or greater for script to run
    $minimumPowershellMajorVersion = 3
    if ($PSVersionTable.PSVersion.Major -lt $minimumPowershellMajorVersion) {
       Write-Host "Need powershell version $minimumPowershellMajorVersion or greater to create release annotation"
       return
    }
    
    $currentTime = (Get-Date).ToUniversalTime()
    $annotationDate = $currentTime.ToString("MMddyyyy_HHmmss")
    set-variable -Name requestBody -Force -Scope Script
    $requestBody = @{}
    $requestBody.Id = [GUID]::NewGuid()
    $requestBody.AnnotationName = $releaseName
    $requestBody.EventTime = $currentTime.GetDateTimeFormats("s")[0] # GetDateTimeFormats returns an array
    $requestBody.Category = "Deployment"
    
    if ($releaseProperties -eq $null) {
        $properties = @{}
    } else {
        $properties = $releaseProperties    
    }
    $properties.Add("ReleaseName", $releaseName)
    
    $requestBody.Properties = ConvertTo-Json($properties) -Compress
    
    $bodyJson = [System.Text.Encoding]::UTF8.GetBytes(($requestBody | ConvertTo-Json))
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add("X-AIAPIKEY", $apiKey)
    
    set-variable -Name createAnnotationResult1 -Force -Scope Local -Value $null
    set-variable -Name createAnnotationResultDescription -Force -Scope Local -Value ""
    
    # get redirect link from fwlink
    $requestUrl = GetRequestUrlFromFwLink("http://go.microsoft.com/fwlink/?prd=11901&pver=1.0&sbp=Application%20Insights&plcid=0x409&clcid=0x409&ar=Annotations&sar=Create%20Annotation")
    if ($requestUrl -eq $null) {
        $output = "Failed to find the redirect link to create a release annotation"
        throw $output
    }
    
    $createAnnotationResult1, $createAnnotationResultDescription = CreateAnnotation($requestUrl)
    if ($createAnnotationResult1) 
    {
         $output = "Failed to create an annotation with Id: {0}. Error {1}, Description: {2}." -f $requestBody.Id, $createAnnotationResult1, $createAnnotationResultDescription
         throw $output
    }
    
    $str = "Release annotation created. Id: {0}." -f $requestBody.Id
    Write-Host $str -ForegroundColor Green
    
    ```
   
1. Выполните действия, описанные в предыдущей процедуре, чтобы получить идентификатор Application Insights и создать ключ API на вкладке **доступ к api** Application Insights.
   
1. Вызовите сценарий PowerShell с помощью следующего кода, заменив заполнители в угловых скобках значениями. `-releaseProperties`Являются необязательными. 
   
   ```powershell
   
        .\CreateReleaseAnnotation.ps1 `
         -applicationId "<applicationId>" `
         -apiKey "<apiKey>" `
         -releaseName "<releaseName>" `
         -releaseProperties @{
             "ReleaseDescription"="<a description>";
             "TriggerBy"="<Your name>" }
   ```

Можно изменить скрипт, например, чтобы создать заметки в прошлом.

## <a name="next-steps"></a>Следующие шаги

* [Создание рабочих элементов](./diagnostic-search.md#create-work-item)
* [Автоматизация с помощью PowerShell](./powershell.md)


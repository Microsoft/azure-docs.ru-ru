---
title: DevOps для конвейера приема данных
titleSuffix: Azure Machine Learning
description: Узнайте, как применять DevOpsные методики для создания конвейера приема данных для подготовки данных с помощью фабрики данных Azure и Azure Databricks.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.author: iefedore
author: eedorenko
manager: davete
ms.reviewer: larryfr
ms.date: 06/23/2020
ms.openlocfilehash: e8a8b952d917db3a7eefd2e0371d41287c5be944
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102612479"
---
# <a name="devops-for-a-data-ingestion-pipeline"></a>DevOps для конвейера приема данных

В большинстве случаев решение приема данных представляет собой композицию скриптов, вызовов служб и конвейера, управляющего всеми действиями. Из этой статьи вы узнаете, как применять DevOpsные методики к жизненному циклу разработки общего конвейера приема данных, который готовит данные для обучения модели машинного обучения. Конвейер создается с использованием следующих служб Azure:

* __Фабрика данных Azure__: считывает необработанные данные и осуществляет подготовку данных.
* __Azure Databricks__: запускает записную книжку Python, которая преобразует данные.
* __Azure pipelines__: автоматизируйте процесс непрерывной интеграции и разработки.

## <a name="data-ingestion-pipeline-workflow"></a>Рабочий процесс конвейера приема данных

Конвейер приема данных реализует следующий рабочий процесс:

1. Необработанные данные считываются в конвейер фабрики данных Azure (ADF).
1. Конвейер ADF отправляет данные в кластер Azure Databricks, который запускает записную книжку Python для преобразования данных.
1. Данные хранятся в контейнере больших двоичных объектов, где их можно использовать Машинное обучение Azure для обучения модели.

![Рабочий процесс конвейера приема данных](media/how-to-cicd-data-ingestion/data-ingestion-pipeline.png)

## <a name="continuous-integration-and-delivery-overview"></a>Обзор непрерывной интеграции и доставки

Как и в случае многих программных решений, над ним работает группа (например, инженеры по работе с данными). Они совместно работают и совместно используют одни и те же ресурсы Azure, такие как фабрика данных Azure, Azure Databricks и учетные записи хранения Azure. Коллекция этих ресурсов является средой разработки. Специалисты по данным вносят вклад в одну базу исходного кода.

Система непрерывной интеграции и доставки автоматизирует процесс сборки, тестирования и доставки (развертывания) решения. Процесс непрерывной интеграции (CI) выполняет следующие задачи:

* Сборка кода
* Проверка с помощью тестов качества кода
* Выполняет модульные тесты
* Создает артефакты, такие как протестированный код и шаблоны Azure Resource Manager

Процесс непрерывной поставки (CD) развертывает артефакты в нижестоящих средах.

![схема приема данных cicd](media/how-to-cicd-data-ingestion/cicd-data-ingestion.png)

В этой статье показано, как автоматизировать процессы CI и CD с помощью [Azure pipelines](https://azure.microsoft.com/services/devops/pipelines/).

## <a name="source-control-management"></a>Управление системой управления версиями

Управление системой управления версиями требуется для контроля изменений и обеспечения совместной работы между членами команды.
Например, код будет храниться в репозитории Azure DevOps, GitHub или GitLab. Рабочий процесс совместной работы основан на модели ветвления. Например, [гитфлов](https://datasift.github.io/gitflow/IntroducingGitFlow.html).

### <a name="python-notebook-source-code"></a>Исходный код записной книжки Python

Специалисты по работе с данными работают с исходным кодом для записной книжки Python локально в интегрированной среде разработки (например, [Visual Studio Code](https://code.visualstudio.com)) или непосредственно в рабочей области "кирпичи данных". После внесения изменений в код они объединяются в репозиторий после применения политики ветвления.

> [!TIP] 
> Мы рекомендуем хранить код в `.py` файлах, а не в `.ipynb` Jupyter Notebook формате. Он повышает удобочитаемость кода и включает автоматическую проверку качества кода в процессе непрерывной интеграции.

### <a name="azure-data-factory-source-code"></a>Исходный код фабрики данных Azure

Исходный код конвейеров фабрики данных Azure — это коллекция JSON-файлов, созданных рабочей областью фабрики данных Azure. Обычно инженеры по работе с данными работают с визуальным конструктором в рабочей области фабрики данных Azure, а не непосредственно с файлами исходного кода. 

Сведения о настройке рабочей области для использования репозитория системы управления версиями см. в разделе [Author by Azure Repos интеграция с Git](../data-factory/source-control.md#author-with-azure-repos-git-integration).   

## <a name="continuous-integration-ci"></a>Непрерывная интеграция (CI)

Основная цель процесса непрерывной интеграции состоит в том, чтобы собрать совместную работу группы из исходного кода и подготовить ее к развертыванию в нижестоящих средах. Как и в случае с управлением исходным кодом, этот процесс отличается для записных книжек Python и конвейеров фабрики данных Azure. 

### <a name="python-notebook-ci"></a>Элемент конфигурации записной книжки Python

Процесс CI для записных книжек Python получает код из ветви совместной работы (например, ***master** _ или _ *_Разработка_* *) и выполняет следующие действия:
* Linting кода
* Модульное тестирование
* Сохранение кода как артефакта

В следующем фрагменте кода показано, как реализовать эти шаги в конвейере Azure DevOps ***YAML*** :

```yaml
steps:
- script: |
   flake8 --output-file=$(Build.BinariesDirectory)/lint-testresults.xml --format junit-xml  
  workingDirectory: '$(Build.SourcesDirectory)'
  displayName: 'Run flake8 (code style analysis)'  
  
- script: |
   python -m pytest --junitxml=$(Build.BinariesDirectory)/unit-testresults.xml $(Build.SourcesDirectory)
  displayName: 'Run unit tests'

- task: PublishTestResults@2
  condition: succeededOrFailed()
  inputs:
    testResultsFiles: '$(Build.BinariesDirectory)/*-testresults.xml'
    testRunTitle: 'Linting & Unit tests'
    failTaskOnFailedTests: true
  displayName: 'Publish linting and unit test results'

- publish: $(Build.SourcesDirectory)
    artifact: di-notebooks
```

Конвейер использует [flake8](https://pypi.org/project/flake8/) для работы с кодом Python linting. Он выполняет модульные тесты, определенные в исходном коде, и публикует linting и результаты тестов, чтобы они были доступны на экране выполнения конвейера Azure:

![модульные тесты linting](media/how-to-cicd-data-ingestion/linting-unit-tests.png)

Если linting и модульное тестирование выполнены успешно, конвейер скопирует исходный код в репозиторий артефактов, который будет использоваться последующими шагами развертывания.

### <a name="azure-data-factory-ci"></a>CI фабрики данных Azure

Процесс непрерывной интеграции для конвейера фабрики данных Azure является узким местом для конвейера приема данных. Непрерывная интеграция отсутствует. Развертываемый артефакт фабрики данных Azure — это коллекция шаблонов Azure Resource Manager. Единственный способ создать эти шаблоны — нажать кнопку ***опубликовать*** в рабочей области фабрика данных Azure.

1. Специалисты по работе с данными объединяют исходный код из их компонентов в ветвь совместной работы, например ***master** _ или _ *_разработать_* *. 
1. Пользователь с предоставленными разрешениями нажимает кнопку ***Publish (опубликовать*** ), чтобы создать Azure Resource Manager шаблоны из исходного кода в ветви совместной работы. 
1. Рабочая область проверяет конвейеры (с учетом linting и модульного тестирования), создает шаблоны Azure Resource Manager (Подумайте о построении) и сохраняет созданные шаблоны в техническом ветви ***adf_publish*** в том же репозитории кода (с учетом артефактов публикации). Эта ветвь создается автоматически рабочей областью фабрики данных Azure. 

Дополнительные сведения об этом процессе см. [в статье непрерывная интеграция и доставка в фабрике данных Azure](../data-factory/continuous-integration-deployment.md).

Важно убедиться, что созданные шаблоны Azure Resource Manager не зависят от среды. Это означает, что все значения, которые могут различаться в разных средах, являются параметризованным. Фабрика данных Azure достаточно интеллектуальна для предоставления большинства таких значений в качестве параметров. Например, в следующем шаблоне свойства подключения к рабочей области Машинное обучение Azure предоставляются в виде параметров:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "factoryName": {
            "value": "devops-ds-adf"
        },
        "AzureMLService_servicePrincipalKey": {
            "value": ""
        },
        "AzureMLService_properties_typeProperties_subscriptionId": {
            "value": "0fe1c235-5cfa-4152-17d7-5dff45a8d4ba"
        },
        "AzureMLService_properties_typeProperties_resourceGroupName": {
            "value": "devops-ds-rg"
        },
        "AzureMLService_properties_typeProperties_servicePrincipalId": {
            "value": "6e35e589-3b22-4edb-89d0-2ab7fc08d488"
        },
        "AzureMLService_properties_typeProperties_tenant": {
            "value": "72f988bf-86f1-41af-912b-2d7cd611db47"
        }
    }
}
```

Тем не менее вам может потребоваться предоставить пользовательские свойства, которые не обрабатываются рабочей областью фабрики данных Azure по умолчанию. В сценарии этой статьи конвейер фабрики данных Azure вызывает записную книжку Python, обрабатывающую данные. Записная книжка принимает параметр с именем входного файла данных.

```Python
import pandas as pd
import numpy as np

data_file_name = getArgument("data_file_name")
data = pd.read_csv(data_file_name)

labels = np.array(data['target'])
...
```

Это имя отличается для сред ***dev** _, _*_QA_*_, _*_UAT_*_ и _*_производственных_*_ . В сложном конвейере с несколькими действиями может быть несколько пользовательских свойств. Рекомендуется объединить все эти значения в одном месте и определить их как конвейеры _ *_переменные_* *:

![На снимке экрана показана Записная книжка, которая называется Препаредата и M L. Выполните конвейер с именем M L Execute конвейер в верхней части со вкладкой переменные, выбранной ниже, с возможностью добавления новых переменных, каждая с именем, типом и значением по умолчанию.](media/how-to-cicd-data-ingestion/adf-variables.png)

Действия конвейера могут ссылаться на переменные конвейера при их фактическом использовании:

![На снимке экрана показана Записная книжка с именем Препаредата и M L. Выполните конвейер с именем M L Execute конвейер в верхней части с вкладкой Settings (параметры), выбранной ниже.](media/how-to-cicd-data-ingestion/adf-notebook-parameters.png)

В рабочей области фабрики данных Azure ***не*** предоставляются переменные конвейера по умолчанию в качестве параметров шаблонов Azure Resource Manager. В рабочей области используется [шаблон параметризации по умолчанию](../data-factory/continuous-integration-deployment.md#default-parameterization-template) , определяющий, какие свойства конвейера должны быть предоставлены как параметры шаблона Azure Resource Manager. Чтобы добавить в список переменные конвейера, обновите `"Microsoft.DataFactory/factories/pipelines"` раздел [шаблона параметризации по умолчанию](../data-factory/continuous-integration-deployment.md#default-parameterization-template) , используя следующий фрагмент кода, и поместите файл result JSON в корень исходной папки:

```json
"Microsoft.DataFactory/factories/pipelines": {
        "properties": {
            "variables": {
                "*": {
                    "defaultValue": "="
                }
            }
        }
    }
```

При этом Рабочая область фабрики данных Azure будет добавлять переменные в список параметров при нажатии кнопки " ***опубликовать*** ":

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "factoryName": {
            "value": "devops-ds-adf"
        },
        ...
        "data-ingestion-pipeline_properties_variables_data_file_name_defaultValue": {
            "value": "driver_prediction_train.csv"
        }        
    }
}
```

Значения в файле JSON являются значениями по умолчанию, настроенными в определении конвейера. Они должны быть переопределены значениями целевой среды при развертывании шаблона Azure Resource Manager.

## <a name="continuous-delivery-cd"></a>Непрерывная поставка (CD)

Процесс непрерывной поставки принимает артефакты и развертывает их в первую целевую среду. Это гарантирует, что решение работает, выполняя тесты. В случае успешного выполнения он переходит к следующей среде. 

Конвейер Azure на компакт-диске состоит из нескольких этапов, представляющих собой среды. На каждом этапе содержатся [развертывания](/azure/devops/pipelines/process/deployment-jobs) и [задания](/azure/devops/pipelines/process/phases?tabs=yaml) , выполняющие следующие действия.

* Развертывание записной книжки Python в Azure Databricks рабочей области
* Развертывание конвейера фабрики данных Azure 
* Запуск конвейера
* Проверка результата приема данных

Этапы конвейера можно настроить с помощью [утверждений](/azure/devops/pipelines/process/approvals?tabs=check-pass) и [шлюзов](/azure/devops/pipelines/release/approvals/gates) , которые обеспечивают дополнительный контроль над тем, как процесс развертывания проходит через цепочку сред.

### <a name="deploy-a-python-notebook"></a>Развертывание записной книжки Python

В следующем фрагменте кода определяется [развертывание](/azure/devops/pipelines/process/deployment-jobs) конвейера Azure, которое копирует записную книжку Python в кластер кирпичей.

```yaml
- stage: 'Deploy_to_QA'
  displayName: 'Deploy to QA'
  variables:
  - group: devops-ds-qa-vg
  jobs:
  - deployment: "Deploy_to_Databricks"
    displayName: 'Deploy to Databricks'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: UsePythonVersion@0
              inputs:
                versionSpec: '3.x'
                addToPath: true
                architecture: 'x64'
              displayName: 'Use Python3'

            - task: configuredatabricks@0
              inputs:
                url: '$(DATABRICKS_URL)'
                token: '$(DATABRICKS_TOKEN)'
              displayName: 'Configure Databricks CLI'    

            - task: deploynotebooks@0
              inputs:
                notebooksFolderPath: '$(Pipeline.Workspace)/di-notebooks'
                workspaceFolder: '/Shared/devops-ds'
              displayName: 'Deploy (copy) data processing notebook to the Databricks cluster'       
```            

Артефакты, созданные CI, автоматически копируются в агент развертывания и доступны в `$(Pipeline.Workspace)` папке. В этом случае задача развертывания ссылается на `di-notebooks` артефакт, содержащий записную книжку Python. В этом [развертывании](/azure/devops/pipelines/process/deployment-jobs) для копирования файлов записной книжки в рабочую область "модули обработки" используется [расширение Azure DevOps для модуля](https://marketplace.visualstudio.com/items?itemName=riserrad.azdo-databricks) обработки.

`Deploy_to_QA`Этап содержит ссылку на `devops-ds-qa-vg` группу переменных, определенную в проекте Azure DevOps. Действия на этом этапе относятся к переменным из этой группы переменных (например, `$(DATABRICKS_URL)` и `$(DATABRICKS_TOKEN)` ). Идея состоит в том, что следующий этап (например, `Deploy_to_UAT` ) будет действовать с теми же именами переменных, которые определены в отдельной группе переменных с областью действия UAT.

### <a name="deploy-an-azure-data-factory-pipeline"></a>Развертывание конвейера фабрики данных Azure

Развертываемый артефакт фабрики данных Azure является шаблоном Azure Resource Manager. Он будет развертываться вместе с задачей ***развертывания группы ресурсов Azure*** , как показано в следующем фрагменте кода:

```yaml
  - deployment: "Deploy_to_ADF"
    displayName: 'Deploy to ADF'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: AzureResourceGroupDeployment@2
              displayName: 'Deploy ADF resources'
              inputs:
                azureSubscription: $(AZURE_RM_CONNECTION)
                resourceGroupName: $(RESOURCE_GROUP)
                location: $(LOCATION)
                csmFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateForFactory.json'
                csmParametersFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateParametersForFactory.json'
                overrideParameters: -data-ingestion-pipeline_properties_variables_data_file_name_defaultValue "$(DATA_FILE_NAME)"
```
Значение параметра filename файла данных берется из `$(DATA_FILE_NAME)` переменной, определенной в группе переменных этапа контроля качества. Аналогичным образом все параметры, определенные в ***ARMTemplateForFactory.js*** , можно переопределить. Если это не так, используются значения по умолчанию.

### <a name="run-the-pipeline-and-check-the-data-ingestion-result"></a>Запуск конвейера и проверка результата приема данных

Следующий шаг — убедиться, что развернутое решение работает. Следующее определение задания выполняет конвейер фабрики данных Azure с помощью [скрипта PowerShell](https://github.com/microsoft/DataOps/tree/master/adf/utils) и выполняет записную книжку Python в кластере Azure Databricks. Записная книжка проверяет, правильно ли были приняты данные, и проверяет файл данных результатов с `$(bin_FILE_NAME)` именем.

```yaml
  - job: "Integration_test_job"
    displayName: "Integration test job"
    dependsOn: [Deploy_to_Databricks, Deploy_to_ADF]
    pool:
      vmImage: 'ubuntu-latest'
    timeoutInMinutes: 0
    steps:
    - task: AzurePowerShell@4
      displayName: 'Execute ADF Pipeline'
      inputs:
        azureSubscription: $(AZURE_RM_CONNECTION)
        ScriptPath: '$(Build.SourcesDirectory)/adf/utils/Invoke-ADFPipeline.ps1'
        ScriptArguments: '-ResourceGroupName $(RESOURCE_GROUP) -DataFactoryName $(DATA_FACTORY_NAME) -PipelineName $(PIPELINE_NAME)'
        azurePowerShellVersion: LatestVersion
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true
        architecture: 'x64'
      displayName: 'Use Python3'

    - task: configuredatabricks@0
      inputs:
        url: '$(DATABRICKS_URL)'
        token: '$(DATABRICKS_TOKEN)'
      displayName: 'Configure Databricks CLI'    

    - task: executenotebook@0
      inputs:
        notebookPath: '/Shared/devops-ds/test-data-ingestion'
        existingClusterId: '$(DATABRICKS_CLUSTER_ID)'
        executionParams: '{"bin_file_name":"$(bin_FILE_NAME)"}'
      displayName: 'Test data ingestion'

    - task: waitexecution@0
      displayName: 'Wait until the testing is done'
```

Последняя задача в задании проверяет результат выполнения записной книжки. Если она возвращает ошибку, она устанавливает для состояния выполнения конвейера значение Failed.

## <a name="putting-pieces-together"></a>Совместное размещение элементов

Полный конвейер Azure CI/CD состоит из следующих этапов.
* CI
* Развертывание в QA
    * Развертывание в модулях и развертывание в ADF
    * Интеграционный тест

Он содержит несколько этапов ***развертывание** _, равное количеству требуемых целевых сред. Каждая стадия _ *_deploy_** содержит два [развертывания](/azure/devops/pipelines/process/deployment-jobs) , которые выполняются параллельно, и [Задание](/azure/devops/pipelines/process/phases?tabs=yaml) , которое запускается после развертывания для тестирования решения в среде.

Пример реализации конвейера собираются в следующем фрагменте кода ***YAML*** :

```yaml
variables:
- group: devops-ds-vg

stages:
- stage: 'CI'
  displayName: 'CI'
  jobs:
  - job: "CI_Job"
    displayName: "CI Job"
    pool:
      vmImage: 'ubuntu-latest'
    timeoutInMinutes: 0
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true
        architecture: 'x64'
      displayName: 'Use Python3'
    - script: pip install --upgrade flake8 flake8_formatter_junit_xml
      displayName: 'Install flake8'
    - checkout: self
    - script: |
       flake8 --output-file=$(Build.BinariesDirectory)/lint-testresults.xml --format junit-xml  
    workingDirectory: '$(Build.SourcesDirectory)'
    displayName: 'Run flake8 (code style analysis)'  
    - script: |
       python -m pytest --junitxml=$(Build.BinariesDirectory)/unit-testresults.xml $(Build.SourcesDirectory)
    displayName: 'Run unit tests'
    - task: PublishTestResults@2
    condition: succeededOrFailed()
    inputs:
        testResultsFiles: '$(Build.BinariesDirectory)/*-testresults.xml'
        testRunTitle: 'Linting & Unit tests'
        failTaskOnFailedTests: true
    displayName: 'Publish linting and unit test results'    

    # The CI stage produces two artifacts (notebooks and ADF pipelines).
    # The pipelines Azure Resource Manager templates are stored in a technical branch "adf_publish"
    - publish: $(Build.SourcesDirectory)/$(Build.Repository.Name)/code/dataingestion
      artifact: di-notebooks
    - checkout: git://${{variables['System.TeamProject']}}@adf_publish    
    - publish: $(Build.SourcesDirectory)/$(Build.Repository.Name)/devops-ds-adf
      artifact: adf-pipelines

- stage: 'Deploy_to_QA'
  displayName: 'Deploy to QA'
  variables:
  - group: devops-ds-qa-vg
  jobs:
  - deployment: "Deploy_to_Databricks"
    displayName: 'Deploy to Databricks'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: UsePythonVersion@0
              inputs:
                versionSpec: '3.x'
                addToPath: true
                architecture: 'x64'
              displayName: 'Use Python3'

            - task: configuredatabricks@0
              inputs:
                url: '$(DATABRICKS_URL)'
                token: '$(DATABRICKS_TOKEN)'
              displayName: 'Configure Databricks CLI'    

            - task: deploynotebooks@0
              inputs:
                notebooksFolderPath: '$(Pipeline.Workspace)/di-notebooks'
                workspaceFolder: '/Shared/devops-ds'
              displayName: 'Deploy (copy) data processing notebook to the Databricks cluster'             
  - deployment: "Deploy_to_ADF"
    displayName: 'Deploy to ADF'
    timeoutInMinutes: 0
    environment: qa
    strategy:
      runOnce:
        deploy:
          steps:
            - task: AzureResourceGroupDeployment@2
              displayName: 'Deploy ADF resources'
              inputs:
                azureSubscription: $(AZURE_RM_CONNECTION)
                resourceGroupName: $(RESOURCE_GROUP)
                location: $(LOCATION)
                csmFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateForFactory.json'
                csmParametersFile: '$(Pipeline.Workspace)/adf-pipelines/ARMTemplateParametersForFactory.json'
                overrideParameters: -data-ingestion-pipeline_properties_variables_data_file_name_defaultValue "$(DATA_FILE_NAME)"
  - job: "Integration_test_job"
    displayName: "Integration test job"
    dependsOn: [Deploy_to_Databricks, Deploy_to_ADF]
    pool:
      vmImage: 'ubuntu-latest'
    timeoutInMinutes: 0
    steps:
    - task: AzurePowerShell@4
      displayName: 'Execute ADF Pipeline'
      inputs:
        azureSubscription: $(AZURE_RM_CONNECTION)
        ScriptPath: '$(Build.SourcesDirectory)/adf/utils/Invoke-ADFPipeline.ps1'
        ScriptArguments: '-ResourceGroupName $(RESOURCE_GROUP) -DataFactoryName $(DATA_FACTORY_NAME) -PipelineName $(PIPELINE_NAME)'
        azurePowerShellVersion: LatestVersion
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true
        architecture: 'x64'
      displayName: 'Use Python3'

    - task: configuredatabricks@0
      inputs:
        url: '$(DATABRICKS_URL)'
        token: '$(DATABRICKS_TOKEN)'
      displayName: 'Configure Databricks CLI'    

    - task: executenotebook@0
      inputs:
        notebookPath: '/Shared/devops-ds/test-data-ingestion'
        existingClusterId: '$(DATABRICKS_CLUSTER_ID)'
        executionParams: '{"bin_file_name":"$(bin_FILE_NAME)"}'
      displayName: 'Test data ingestion'

    - task: waitexecution@0
      displayName: 'Wait until the testing is done'                

```

## <a name="next-steps"></a>Дальнейшие действия

* [Система управления версиями в Фабрике данных Azure](../data-factory/source-control.md)
* [Непрерывные интеграция и поставка в Фабрике данных Azure](../data-factory/continuous-integration-deployment.md)
* [DevOps для Azure Databricks](https://marketplace.visualstudio.com/items?itemName=riserrad.azdo-databricks)
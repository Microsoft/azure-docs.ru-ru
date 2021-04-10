---
title: Руководство по CI/CD на виртуальных машинах Azure с использованием Azure Pipelines
description: Узнайте, как настроить непрерывную интеграцию (CI) и непрерывное развертывание (CD) приложения Node.js на виртуальных машинах Azure с помощью конвейера Azure на основе YAML.
author: ushan
tags: azure-devops-pipelines
ms.assetid: ''
ms.service: virtual-machines
ms.collection: linux
ms.topic: tutorial
ms.tgt_pltfrm: azure-pipelines
ms.workload: infrastructure
ms.date: 1/3/2020
ms.author: ushan
ms.custom: devops, devx-track-js
ms.openlocfilehash: 49282bf6cbc7c24b75fbe3f1bbe68bd1fac62ae3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102552496"
---
# <a name="tutorial-deploy-your-app-to-linux-virtual-machines-in-azure-using-azure-devops-services-and-azure-pipelines"></a>Руководство по Развертывание приложения на виртуальных машинах Linux в Azure с помощью Azure DevOps Services и Azure Pipelines

Непрерывная интеграция (CI) и непрерывное развертывание (CD) — это конвейер, с помощью которого можно выполнить сборку, выпустить и развернуть свой код после каждой фиксации. Этот документ описывает, как настроить конвейер CI/CD для Azure Pipelines, чтобы выполнять развертывания на нескольких компьютера.

Azure Pipelines предоставляет комплексный и полнофункциональный набор инструментов для автоматизации CI/CD. С его помощью вы можете выполнять развертывания на виртуальных машинах локально и в любом облаке.

В этом руководстве описано, как настроить конвейер CI/CD на основе YAML, который развертывает приложение в [среде](/azure/devops/pipelines/process/environments) Azure Pipelines с виртуальными машинами Linux в качестве ресурсов, каждый из которых выступает в качестве веб-сервера для запуска приложения.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Получение примера приложения.
> * Создание в Azure Pipelines конвейера CI на основе YAML для сборки примера приложения.
> * Создание среды Azure Pipelines для виртуальных машин Azure.
> * Создание конвейера CD в Azure Pipelines.
> * Выполнение развертывания вручную и с помощью непрерывной интеграции.

## <a name="before-you-begin"></a>Перед началом

* Войдите в организацию Azure DevOps Services ( **https://dev.azure.com/** ). 
  Вы можете получить [бесплатную организацию Azure DevOps Services](https://go.microsoft.com/fwlink/?LinkId=307137&clcid=0x409&wt.mc_id=o~msft~vscom~home-vsts-hero~27308&campaign=o~msft~vscom~home-vsts-hero~27308).

  > [!NOTE]
  > Дополнительные сведения см. в статье [о подключении к Azure DevOps Services](/azure/devops/organizations/projects/connect-to-projects).

*  Для целевого развертывания понадобится виртуальная машина Linux.  Для получения дополнительной информации см. статью [Создание виртуальных машин Linux и управление ими с помощью Azure CLI](./tutorial-manage-vm.md).

*  Откройте входящий порт 80 для виртуальной машины. Дополнительные сведения см. в статье [Создание групп безопасности сети с помощью портала Azure](../../virtual-network/tutorial-filter-network-traffic.md).

## <a name="get-your-sample-app-code"></a>Получение примера кода приложения

Если у вас уже есть приложение в GitHub, которое вы готовы развернуть, попробуйте создать конвейер для этого кода.

Но если вы только начинаете работу, лучше использовать наш пример кода. В этом случае создайте вилку этого репозитория в GitHub:

#### <a name="java"></a>[Java](#tab/java)

```
https://github.com/spring-projects/spring-petclinic
```

> [!NOTE]
> Petclinic — это приложение [Java Spring Boot](https://spring.io/guides/gs/spring-boot), созданное с помощью [Maven](https://spring.io/guides/gs/maven/).

#### <a name="javascript"></a>[JavaScript](#tab/java-script)

```
https://github.com/azure-devops/fabrikam-node
```

> [!NOTE]
> Это приложение Node.js создано с помощью [Yeoman](https://yeoman.io/learning/index.html). Оно использует Express, Bower и Grunt. У него также есть некоторые пакеты npm в качестве зависимостей.
> Кроме того, пример содержит сценарий, который устанавливает Nginx и развертывает приложение. Он выполняется на виртуальных машинах. В частности, сценарий выполняет следующее:
> 1. Устанавливает Node, Nginx и PM2.
> 2. Настраивает Nginx и PM2.
> 3. Запускает приложения Node.

* * * 

## <a name="prerequisites-for-the-linux-vm"></a>Предварительные требования для виртуальных машин Linux

Упомянутые выше примеры приложений были протестированы на Ubuntu 16.04. Мы рекомендуем использовать для этого краткого руководства эту версию виртуальной машины Linux.
Выполните описанные ниже дополнительные действия с учетом стека среды выполнения, который вы используете для приложения.

#### <a name="java"></a>[Java](#tab/java)

- Для развертывания приложений на основе Java Spring Boot и Spring Cloud создайте в Azure виртуальную машину Linux, используя [этот шаблон](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004), который включает среду выполнения на основе OpenJDK с полной поддержкой.
- Для развертывания сервлетов Java на сервере Tomcat создайте виртуальную машину Linux, используя Java 8 и [этот шаблон](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004), и [настройте Tomcat 9.x в качестве службы](https://tomcat.apache.org/tomcat-9.0-doc/setup.html).
- Для развертывания приложения на базе Java EE используйте шаблон Azure, чтобы создать [виртуальную машину Linux с Java и WebSphere 9.x](https://azuremarketplace.microsoft.com/marketplace/apps/midvision.websphere-application-server-nde-90), [виртуальную машину Linux с Java и WebLogic 12.x](https://azuremarketplace.microsoft.com/marketplace/apps/oracle.20191009-arm-oraclelinux-wls-admin) или [виртуальную машину Linux с Java и WildFly/JBoss 14](https://azuremarketplace.microsoft.com/marketplace/apps/azul.azul-zulu13-ubuntu-2004). 


#### <a name="javascript"></a>[JavaScript](#tab/java-script)

Чтобы установить приложение JavaScript или Node.js, вам потребуется виртуальная машина Linux с веб-сервером Nginx.
Если у вас нет виртуальной машины Linux с Nginx, [создайте ее в Azure](./quick-create-cli.md).

* * * 

## <a name="create-an-azure-pipelines-environment-with-azure-virtual-machines"></a>Создание среды Azure Pipelines с виртуальными машинами Azure

Виртуальные машины можно добавлять в качестве ресурсов в [средах](/azure/devops/pipelines/process/environments) и назначать целевыми объектами для развертываний на нескольких компьютерах. Представления журнала развертываний в среде позволяют отслеживать передачу данных с виртуальной машины в конвейер и затем на фиксацию.

Среду можно создать в узле **Окружения** в разделе **Конвейеры**.
1.  Войдите в свою организацию Azure DevOps и откройте нужный проект.
2.  В этом проекте перейдите к странице **Pipelines** (Конвейеры). Теперь выберите **Environments** (Среды) и щелкните **Create Environment** (Создать среду). Введите **имя** среды (обязательно) и **описание** (при желании).
3.  Выберите **Virtual Machines** (Виртуальные машины) в качестве **ресурса** для добавления в среду и щелкните **Next** (Далее).
4.  Выберите операционную систему (Windows или Linux) и **скопируйте скрипт PS для регистрации**. 
5.  Теперь запустите скопированный скрипт из командной строки PowerShell с правами администратора на каждой из целевых виртуальных машин, которые будут зарегистрированы в этой среде.
    > [!NOTE]
    > - В скрипт добавляется личный маркер доступа пользователя, вошедшего в систему. Срок его действия истекает в тот же день, после чего скопированный скрипт становится непригодным для использования.
    > - Если на виртуальной машине запущен какой-либо агент, укажите уникальное имя агента в соответствующем поле для регистрации в среде.
6.  После регистрации виртуальная машина будет отображаться в виде ресурса среды на вкладке с ресурсами для выбранной среды.

    ![VMcreation](media/tutorial-deploy-vms-azure-pipelines/vm-creation.png)

7.  Чтобы добавить дополнительные виртуальные машины, вы можете просмотреть этот скрипт и снова скопировать его, щелкнув Add resource (Добавить ресурс) и выбрав Virtual Machines (Виртуальные машины) в качестве ресурса. Этот скрипт будет одинаковым для всех виртуальных машин, которые вы добавляете в ту же среду. 
8.  Каждый компьютер взаимодействует с Azure Pipelines для координации развертывания приложения.

    ![VMresource_view](media/tutorial-deploy-vms-azure-pipelines/vm-resourceview.png)

9. Вы можете добавить в виртуальную машину теги, включив их в интерактивный скрипт PowerShell для регистрации или добавляя и удаляя их в представлении ресурсов, щелкнув три точки в конце строки для любого ресурса виртуальной машины.

   Назначенные теги позволяют ограничить развертывание конкретными виртуальными машинами при использовании этой среды в задании развертывания. Длина тегов не может превышать 256 символов, но ограничений на количество используемых тегов нет.

   ![VMtags](media/tutorial-deploy-vms-azure-pipelines/vm-tags.png)

* * * 

## <a name="define-your-ci-build-pipeline"></a>Определение конвейера сборки CI

Вам потребуется конвейер сборки непрерывной интеграции (CI), который публикует ваше веб-приложение, а также скрипт развертывания, который можно выполнять локально на сервере Ubuntu. Настройте конвейер сборки CI для выбранной среды выполнения. 

1. Войдите в свою организацию Azure DevOps и откройте нужный проект.

1. В этом проекте перейдите к странице **Pipelines** (Конвейеры). Выберите действие создания конвейера сборки.

1. Пройдите через шаги мастера, выбрав **GitHub** в качестве расположения исходного кода.

1. Возможно, вам придется выполнить вход в GitHub. Для этого введите учетные данные GitHub.

1. Когда появится список репозиториев, выберите репозиторий с нужным примером приложения.

1. Azure Pipelines проанализирует этот репозиторий и предложит подходящий шаблон конвейера.

#### <a name="java"></a>[Java](#tab/java)

Выберите **базовый** шаблон и скопируйте приведенный ниже фрагмент кода YAML, который компилирует проект Java и выполняет тесты с помощью Apache Maven.

```YAML
jobs:
- job: Build
  displayName: Build Maven Project
  steps:
  - task: Maven@3
    displayName: 'Maven Package'
    inputs:
      mavenPomFile: 'pom.xml'
  - task: CopyFiles@2
    displayName: 'Copy Files to artifact staging directory'
    inputs:
      SourceFolder: '$(System.DefaultWorkingDirectory)'
      Contents: '**/target/*.?(war|jar)'
      TargetFolder: $(Build.ArtifactStagingDirectory)
  - upload: $(Build.ArtifactStagingDirectory)
    artifact: drop
```

См. сведения о том, как [создать приложение Java с помощью Maven](/azure/devops/pipelines/ecosystems/java).

#### <a name="javascript"></a>[JavaScript](#tab/java-script)

Выберите **базовый** шаблон и скопируйте приведенный ниже фрагмент кода YAML, который компилирует общий проект Node.js с помощью npm.

```YAML
- stage: Build
  displayName: Build stage
  jobs:  
  - job: Build
    displayName: Build
    - task: NodeTool@0
      inputs:
        versionSpec: '10.x'
      displayName: 'Install Node.js'
    - script: |
        npm install
        npm run build --if-present
        npm run test --if-present
      displayName: 'npm install, build and test'
    - task: ArchiveFiles@2
      displayName: 'Archive files'
      inputs:
        rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
        includeRootFolder: false
        archiveType: zip
        archiveFile: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
        replaceExistingArchive: true
    - upload: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
      artifact: drop
```

См. сведения о том, как [создать приложение Node.js с помощью gulp](/azure/devops/pipelines/ecosystems/javascript).

- Изучите работу конвейера. Убедитесь, что все стандартные входные данные подходят для вашего кода.

- Щелкните **Save and run** (Сохранить и запустить), затем выберите **Commit directly to the main branch** (Зафиксировать непосредственно в главной ветви) и снова щелкните **Save and run** (Сохранить и запустить).

- Будет выполнен новый запуск. Дождитесь, пока он завершится.

* * * 

## <a name="define-cd-steps-to-deploy-to-the-linux-vm"></a>Определение шагов CD для развертывания на виртуальной машине Linux

1. Измените файл YAML для приведенного выше конвейера, добавив в него [задание развертывания](/azure/devops/pipelines/process/deployment-jobs) с использованием ссылки на среду и ресурсы виртуальной машины, которые вы подготовили ранее, и следующий синтаксис YAML:

   ```YAML
   jobs:  
   - deployment: VMDeploy
     displayName: web
     environment:
       name:  <environment name>
       resourceType: VirtualMachine
       tags: web
   ```
2. Вы можете выбрать из среды определенные наборы виртуальных машин, которые будут получать развертывание, указав **теги**, определенные в среде для каждой виртуальной машины.
См. [полную схему YAML для задания развертывания](/azure/devops/pipelines/yaml-schema?tabs=schema#deployment-job).

3. В качестве стратегии развертывания вы можете выбрать `runOnce` или `rolling`. 

   Простейшая стратегия развертывания `runOnce` содержит обработчики жизненного цикла `preDeploy` `deploy`, `routeTraffic` и `postRouteTraffic`, которые выполняются один раз. После этого выполняется `on:` `success` или `on:` `failure`.

   Ниже приведен пример фрагмента YAML для `runOnce`.
   ```YAML
   jobs:
   - deployment: VMDeploy
     displayName: web
     pool:
       vmImage: 'Ubuntu-16.04'
     environment:
       name: <environment name>
       resourceType: VirtualMachine
     strategy:
         runOnce:
           deploy:
             steps:
             - script: echo my first deployment
   ```

4. Ниже приведен пример фрагмента YAML, который позволяет определить стратегию поочередного обновления виртуальных машин (по пять целевых объектов в каждой итерации). `maxParallel` определяет количество целевых объектов, для которых развертывание может выполняться параллельно. При выборе учитывается абсолютное количество или доля целевых объектов, которые в любое время должны оставаться доступными (кроме целевых объектов, для которых выполняется развертывание). При этом также учитываются условия, определяющие выполнение и невыполнение развертывания.

   ```YAML
   jobs: 
   - deployment: VMDeploy
     displayName: web
     environment:
       name: <environment name>
       resourceType: VirtualMachine
     strategy:
         rolling:
           maxParallel: 5  #for percentages, mention as x%
           preDeploy:
             steps:
             - download: current
               artifact: drop
             - script: echo initialize, cleanup, backup, install certs
           deploy:
             steps:
             - task: Bash@3
               inputs:
                 targetType: 'inline'
                 script: |
                   # Modify deployment script based on the app type
                   echo "Starting deployment script run"
                   sudo java -jar '$(Pipeline.Workspace)/drop/**/target/*.jar'
           routeTraffic:
             steps:
             - script: echo routing traffic
           postRouteTraffic:
             steps:
             - script: echo health check post-route traffic
           on:
             failure:
               steps:
               - script: echo Restore from backup! This is on failure
             success:
               steps:
               - script: echo Notify! This is on success
   ```

   При каждом выполнении этого задания создается журнал развертывания в среде `<environment name>`, в которой вы создали и зарегистрировали виртуальные машины.

## <a name="run-your-pipeline-and-get-traceability-views-in-environment"></a>Выполнение конвейера и получение представлений для отслеживания в среде
Представление развертываний для среды обеспечивает возможность отслеживания всех фиксаций и рабочих элементов, а также историю развертываний для нескольких конвейеров для каждой среды или ресурса.

![VMDeployments_view](media/tutorial-deploy-vms-azure-pipelines/vm-deployments.png)
  
![VMjobs_view](media/tutorial-deploy-vms-azure-pipelines/vm-jobsview.png)

## <a name="next-steps"></a>Дальнейшие действия
- Вы можете приступить к [настройке конвейера](/azure/devops/pipelines/customize-pipeline), который только что создали.
- См. сведения о [других доступных операциях с конвейерами YAML](/azure/devops/pipelines/yaml-schema).
- Чтобы узнать, как развернуть стек LAMP (Linux, Apache, MySQL и PHP), перейдите к следующему руководству.

> [!div class="nextstepaction"]
> [Развертывание стека LAMP](tutorial-lamp-stack.md)

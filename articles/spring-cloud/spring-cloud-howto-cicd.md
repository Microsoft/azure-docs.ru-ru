---
title: CI/CD для Azure Веснного облака
description: CI/CD для Azure Веснного облака
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 331ef39facb9f7cf8f069f2a238be325f53de2d0
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102618039"
---
# <a name="cicd-for-azure-spring-cloud"></a>CI/CD для Azure Веснного облака

Средства непрерывной интеграции и непрерывной поставки позволяют быстро развертывать обновления для существующих приложений с минимальными усилиями и рисками. Azure DevOps помогает организовывать и контролировать эти ключевые задания. Сейчас Azure Веснного облака не предлагает конкретный подключаемый модуль Azure DevOps.  Однако вы можете интегрировать приложения с пружинным облаком с DevOps, используя [задачу Azure CLI](/azure/devops/pipelines/tasks/deploy/azure-cli).

В этой статье показано, как использовать задачу Azure CLI с Azure "Весна Cloud" для интеграции с Azure DevOps.

## <a name="create-an-azure-resource-manager-service-connection"></a>Создание подключения к службе Azure Resource Manager

Ознакомьтесь с [этой статьей](/azure/devops/pipelines/library/connect-to-azure) , чтобы узнать, как создать подключение службы Azure Resource Manager к проекту Azure DevOps. Не забудьте выбрать ту же подписку, которую вы используете для своего экземпляра облачной службы Azure весны.

## <a name="azure-cli-task-templates"></a>Azure CLI шаблонов задач
::: zone pivot="programming-language-csharp"
### <a name="deploy-artifacts"></a>Развертывание артефактов

Вы можете создавать и развертывать проекты с помощью серии `tasks` . Этот фрагмент кода определяет переменные, задачу .NET Core для сборки приложения и задачу Azure CLI для развертывания *ZIP* -файла.

```yaml
variables:
  workingDirectory: './steeltoe-sample'
  planetMainEntry: 'Microsoft.Azure.SpringCloud.Sample.PlanetWeatherProvider.dll'
  solarMainEntry: 'Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.dll'
  planetAppName: 'planet-weather-provider'
  solarAppName: 'solar-system-weather'
  serviceName: '<your service name>'
  resourceGroupName: '<your resource group name>'

steps:
# Restore, build, publish and package the zipped planet app
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: false
    arguments: '--configuration Release'
    zipAfterPublish: false
    modifyOutputPath: false
    workingDirectory: $(workingDirectory)

# Configure Azure CLI and install spring-cloud extension
- task: AzureCLI@1
  inputs:
    azureSubscription: '<your subscription>'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az extension add --name spring-cloud --y
      az configure --defaults group=${{ variables.resourceGroupName }}
      az configure --defaults spring-cloud=${{ variables.serviceName }}
      az spring-cloud app deploy -n ${{ variables.planetAppName }} --runtime-version NetCore_31 --main-entry ${{ variables.planetMainEntry }} --artifact-path ./${{ variables.planetAppName }}/publish-deploy-planet.zip
      az spring-cloud app deploy -n ${{ variables.solarAppName }} --runtime-version NetCore_31 --main-entry ${{ variables.solarMainEntry }} --artifact-path ./${{ variables.solarAppName }}/publish-deploy-solar.zip
      az spring-cloud app update -n ${{ variables.solarAppName }} --assign-endpoint
      az spring-cloud app show -n ${{ variables.solarAppName }} -o table
    workingDirectory: '${{ variables.workingDirectory }}/src'
```

::: zone-end
::: zone pivot="programming-language-java"
### <a name="deploy-artifacts"></a>Развертывание артефактов

Вы можете создавать и развертывать проекты с помощью серии `tasks` . Сначала этот фрагмент кода определяет задачу Maven для создания приложения, а затем — вторую задачу, которая развертывает JAR-файл с помощью расширения Azure "Весна Cloud" Azure CLI.

```yaml
steps:
- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
- task: AzureCLI@1
  inputs:
    azureSubscription: <your service connection name>
    scriptLocation: inlineScript
    inlineScript: |
      az extension add -y --name spring-cloud
      az spring-cloud app deploy --resource-group <your-resource-group> --service <your-spring-cloud-service> --name <app-name> --jar-path ./target/your-result-jar.jar
      # deploy other app
```

### <a name="deploy-from-source"></a>Развертывание из источника

Развертывание можно выполнить непосредственно в Azure без отдельного этапа сборки.

```yaml
- task: AzureCLI@1
  inputs:
    azureSubscription: <your service connection name>
    scriptLocation: inlineScript
    inlineScript: |
      az extension add -y --name spring-cloud
      az spring-cloud app deploy --resource-group <your-resource-group> --service <your-spring-cloud-service> --name <app-name>

      # or if it is a multi-module project
      az spring-cloud app deploy --resource-group <your-resource-group> --service <your-spring-cloud-service> --name <app-name> --target-module relative/path/to/module
```
::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

* [Краткое руководство. Развертывание первого приложения Azure Spring Cloud](spring-cloud-quickstart.md)
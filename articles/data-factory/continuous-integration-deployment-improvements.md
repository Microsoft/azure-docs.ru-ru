---
title: Автоматическая публикация для непрерывной интеграции и доставки
description: Узнайте, как опубликовать для непрерывной интеграции и доставки автоматически.
ms.service: data-factory
author: nabhishek
ms.author: abnarain
ms.reviewer: jburchel
ms.topic: conceptual
ms.date: 02/02/2021
ms.openlocfilehash: b2c48fcc11feaec3efc0acab283609181b92a3dc
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104780469"
---
# <a name="automated-publishing-for-continuous-integration-and-delivery"></a>Автоматическая публикация для непрерывной интеграции и доставки

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

## <a name="overview"></a>Обзор

Непрерывная интеграция — это способ автоматического тестирования каждого изменения, внесенного в базу кода. Как можно раньше, непрерывная доставка проходит проверку, которая происходит во время непрерывной интеграции и передает изменения в промежуточную или рабочую систему.

В фабрике данных Azure непрерывная интеграция и непрерывная поставка (CI/CD) — это перемещение конвейеров фабрики данных из одной среды, например для разработки, тестирования и производства, в другую. Фабрика данных использует [шаблоны Azure Resource Manager (шаблоны ARM)](../azure-resource-manager/templates/overview.md) для хранения конфигурации различных сущностей фабрики данных, таких как конвейеры, наборы данных и потоки.

Существуют два рекомендуемых метода перемещения фабрики данных в другую среду.

- Автоматическое развертывание с использованием интеграции фабрики данных с [Azure pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines).
- Ручная загрузка шаблона ARM с помощью интеграции с интерфейсом пользователя фабрики данных с Azure Resource Manager.

Дополнительные сведения см. [в статье непрерывная интеграция и доставка в фабрике данных Azure](continuous-integration-deployment.md).

В этой статье рассматриваются улучшения непрерывного развертывания и функция автоматической публикации для CI/CD.

## <a name="continuous-deployment-improvements"></a>Усовершенствования непрерывного развертывания

Функция автоматической публикации принимает функции шаблона **проверить все** и **экспортировать ARM** из фабрики данных и делает логику доступной через общедоступный пакет NPM [@microsoft/azure-data-factory-utilities](https://www.npmjs.com/package/@microsoft/azure-data-factory-utilities) . По этой причине можно запустить эти действия программными средствами вместо того, чтобы переходить к пользовательскому интерфейсу фабрики данных и выбрать кнопку вручную. Благодаря этой возможности конвейер CI/CD будет работать с непрерывной интеграцией справедливо.

### <a name="current-cicd-flow"></a>Текущий поток CI/CD

1. Каждый пользователь вносит изменения в свои частные ветви.
1. Отправка в главную базу не разрешена. Пользователи должны создать запрос на вытягивание для внесения изменений.
1. Пользователи должны загрузить пользовательский интерфейс фабрики данных и выбрать **опубликовать** , чтобы развернуть изменения в фабрике данных и создать шаблоны ARM в ветви Publish.
1. Конвейер выпуска DevOps настроен для создания нового выпуска и развертывания шаблона ARM каждый раз, когда новое изменение передается в ветвь Publish.

![Схема, показывающая текущий поток CI/CD.](media/continuous-integration-deployment-improvements/current-ci-cd-flow.png)

### <a name="manual-step"></a>Этот этап выполняется вручную

В текущем потоке CI/CD взаимодействие с пользователем является посредником для создания шаблона ARM. В результате пользователь должен перейти в пользовательский интерфейс фабрики данных и вручную выбрать **опубликовать** , чтобы запустить создание шаблона ARM и удалить его в ветви Publish.

### <a name="the-new-cicd-flow"></a>Новый поток CI/CD

1. Каждый пользователь вносит изменения в свои частные ветви.
1. Отправка в главную базу не разрешена. Пользователи должны создать запрос на вытягивание для внесения изменений.
1. Сборка конвейера Azure DevOps запускается каждый раз при внесении новой фиксации в главную базу. Он проверяет ресурсы и создает шаблон ARM в качестве артефакта, если проверка прошла.
1. Конвейер выпуска DevOps настроен для создания нового выпуска и развертывания шаблона ARM каждый раз, когда будет доступна новая сборка.

![Схема, на которой показан новый поток CI/CD.](media/continuous-integration-deployment-improvements/new-ci-cd-flow.png)

### <a name="what-changed"></a>Что изменилось?

- Теперь у нас есть процесс сборки, использующий конвейер сборки DevOps.
- В конвейере сборки используется пакет Адфутилитиес NPM, который проверит все ресурсы и создаст шаблоны ARM. Эти шаблоны могут быть одиночными и связанными.
- Конвейер сборки отвечает за проверку ресурсов фабрики данных и создание шаблона ARM вместо пользовательского интерфейса фабрики данных (кнопка "**опубликовать** ").
- Определение выпуска DevOps теперь будет использовать этот новый конвейер сборки вместо артефакта Git.

> [!NOTE]
> Можно продолжать использовать существующий механизм, который является `adf_publish` ветвью, или можно использовать новый поток. Поддерживаются оба варианта.

## <a name="package-overview"></a>Основные сведения о пакете

В пакете сейчас доступны две команды:

- Экспорт шаблона ARM
- Проверить

### <a name="export-arm-template"></a>Экспорт шаблона ARM

Выполните команду `npm run start export <rootFolder> <factoryId> [outputFolder]` , чтобы экспортировать шаблон ARM, используя ресурсы указанной папки. Эта команда также выполняет проверку перед созданием шаблона ARM. Приведем пример:

```
npm run start export C:\DataFactories\DevDataFactory /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.DataFactory/factories/DevDataFactory ArmTemplateOutput
```

- `RootFolder` — Это обязательное поле, представляющее место расположения ресурсов фабрики данных.
- `FactoryId` является обязательным полем, представляющим идентификатор ресурса фабрики данных в формате `/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.DataFactory/factories/<dfName>` .
- `OutputFolder` Необязательный параметр, указывающий относительный путь для сохранения созданного шаблона ARM.
 
> [!NOTE]
> Созданный шаблон ARM не опубликован в динамической версии фабрики. Развертывание должно выполняться с помощью конвейера CI/CD.
 
### <a name="validate"></a>Проверить

Выполните команду `npm run start validate <rootFolder> <factoryId>` , чтобы проверить все ресурсы заданной папки. Приведем пример:

```
npm run start validate C:\DataFactories\DevDataFactory /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.DataFactory/factories/DevDataFactory
```

- `RootFolder` — Это обязательное поле, представляющее место расположения ресурсов фабрики данных.
- `FactoryId` является обязательным полем, представляющим идентификатор ресурса фабрики данных в формате `/subscriptions/<subId>/resourceGroups/<rgName>/providers/Microsoft.DataFactory/factories/<dfName>` .

## <a name="create-an-azure-pipeline"></a>Создание конвейера Azure

Хотя пакеты NPM могут использоваться различными способами, одним из основных преимуществ является использование [конвейера Azure](https://nam06.safelinks.protection.outlook.com/?url=https:%2F%2Fdocs.microsoft.com%2F%2Fazure%2Fdevops%2Fpipelines%2Fget-started%2Fwhat-is-azure-pipelines%3Fview%3Dazure-devops%23:~:text%3DAzure%2520Pipelines%2520is%2520a%2520cloud%2Cit%2520available%2520to%2520other%2520users.%26text%3DAzure%2520Pipelines%2520combines%2520continuous%2520integration%2Cship%2520it%2520to%2520any%2520target.&data=04%7C01%7Cabnarain%40microsoft.com%7C5f064c3d5b7049db540708d89564b0bc%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C1%7C637423607000268277%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C1000&sdata=jo%2BkIvSBiz6f%2B7kmgqDN27TUWc6YoDanOxL9oraAbmA%3D&reserved=0). При каждом слиянии в ветви совместной работы можно активировать конвейер, который сначала проверяет весь код, а затем экспортирует шаблон ARM в [артефакт сборки](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2F%2Fazure%2Fdevops%2Fpipelines%2Fartifacts%2Fbuild-artifacts%3Fview%3Dazure-devops%26tabs%3Dyaml%23how-do-i-consume-artifacts&data=04%7C01%7Cabnarain%40microsoft.com%7C5f064c3d5b7049db540708d89564b0bc%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C1%7C637423607000278113%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C1000&sdata=dN3t%2BF%2Fzbec4F28hJqigGANvvedQoQ6npzegTAwTp1A%3D&reserved=0) , который может использоваться конвейером выпуска. Отличие от текущего процесса CI/CD заключается в том, что вы *указываете конвейер выпуска на этот артефакт вместо существующей `adf_publish` ветви*.

Чтобы приступить к работе, выполните эти действия:

1.  Откройте проект Azure DevOps и перейдите в раздел **конвейеры**. Выберите **Новый конвейер**.

    ![Снимок экрана, на котором показана кнопка "создать конвейер".](media/continuous-integration-deployment-improvements/new-pipeline.png)
    
1.  Выберите репозиторий, в котором вы хотите сохранить сценарий YAML конвейера. Рекомендуется сохранить его в папке Build в том же репозитории ресурсов фабрики данных. Убедитесь, что в репозитории имеется *package.js* файла, содержащего имя пакета, как показано в следующем примере:

    ```json
    {
        "scripts":{
            "build":"node node_modules/@microsoft/azure-data-factory-utilities/lib/index"
        },
        "dependencies":{
            "@microsoft/azure-data-factory-utilities":"^0.1.3"
        }
    } 
    ```
    
1.  Выберите **начальный конвейер**. Если вы отправили или объединили файл YAML, как показано в следующем примере, можно также указать непосредственно на нем и изменить его.

    ![Снимок экрана, показывающий начальный конвейер.](media/continuous-integration-deployment-improvements/starter-pipeline.png)

    ```yaml
    # Sample YAML file to validate and export an ARM template into a build artifact
    # Requires a package.json file located in the target repository
    
    trigger:
    - main #collaboration branch
    
    pool:
      vmImage: 'ubuntu-latest'
    
    steps:
    
    # Installs Node and the npm packages saved in your package.json file in the build
    
    - task: NodeTool@0
      inputs:
        versionSpec: '10.x'
      displayName: 'Install Node.js'
    
    - task: Npm@1
      inputs:
        command: 'install'
        verbose: true
      displayName: 'Install npm package'
    
    # Validates all of the Data Factory resources in the repository. You'll get the same validation errors as when "Validate All" is selected.
    # Enter the appropriate subscription and name for the source factory.
    
    - task: Npm@1
      inputs:
        command: 'custom'
        customCommand: 'run build validate $(Build.Repository.LocalPath) /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.DataFactory/factories/yourFactoryName'
      displayName: 'Validate'
    
    # Validate and then generate the ARM template into the destination folder, which is the same as selecting "Publish" from the UX.
    # The ARM template generated isn't published to the live version of the factory. Deployment should be done by using a CI/CD pipeline. 
    
    - task: Npm@1
      inputs:
        command: 'custom'
        customCommand: 'run build export $(Build.Repository.LocalPath) /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.DataFactory/factories/yourFactoryName "ArmTemplate"'
      displayName: 'Validate and Generate ARM template'
    
    # Publish the artifact to be used as a source for a release pipeline.
    
    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.Repository.LocalPath)/ArmTemplate'
        artifact: 'ArmTemplates'
        publishLocation: 'pipeline'
    ```

1.  Введите код YAML. В качестве отправной точки рекомендуется использовать файл YAML.
1.  Сохраните и запустите. Если вы использовали YAML, он активируется каждый раз при обновлении основной ветви.

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения о непрерывной интеграции и доставке в фабрике данных:

- [Непрерывная интеграция и доставка в фабрике данных Azure](continuous-integration-deployment.md).

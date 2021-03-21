---
title: CI/CD с Azure Pipelines и шаблонами
description: Описание настройки непрерывной интеграции в Azure Pipelines с помощью шаблонов Azure Resource Manager. В нем показано, как использовать сценарий PowerShell или копировать файлы в промежуточное расположение и выполнять развертывание из него.
ms.topic: conceptual
ms.date: 03/09/2021
ms.openlocfilehash: 4a2f1f15de0abd802f3dce138b2cea33e52e3dfc
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102561948"
---
# <a name="integrate-arm-templates-with-azure-pipelines"></a>Интеграция шаблонов Resource Manager с Azure Pipelines

Вы можете интегрировать шаблоны Azure Resource Manager (шаблоны ARM) с Azure Pipelines для непрерывной интеграции и непрерывного развертывания (CI/CD). В этой статье вы узнаете о двух более сложных способах развертывания шаблонов с помощью Azure Pipelines.

## <a name="select-your-option"></a>Выбор параметра

Прежде чем продолжить работу с этой статьей, рассмотрим различные варианты развертывания шаблона ARM из конвейера.

* **Используйте задачу развертывания шаблона ARM**. Этот параметр является самым простым. Этот подход работает, если вы хотите развернуть шаблон непосредственно из репозитория. Этот вариант не рассматривается в этой статье, а рассматривается в руководстве по [непрерывной интеграции шаблонов ARM с Azure pipelines](deployment-tutorial-pipeline.md). В нем показано, как использовать [задачу развертывания шаблона ARM](https://github.com/microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceManagerTemplateDeploymentV3/README.md) для развертывания шаблона из репозитория GitHub.

* **Добавьте задачу, запускающую скрипт Azure PowerShell**. Этот вариант обеспечивает согласованность в течение всего жизненного цикла разработки, поскольку можно использовать тот же сценарий, который использовался при выполнении локальных тестов. Сценарий развертывает шаблон, но также может выполнять другие операции, такие как получение значений для использования в качестве параметров. Этот параметр показан в этой статье. См. раздел [Azure PowerShell Task](#azure-powershell-task).

   Visual Studio предоставляет [проект группы ресурсов Azure](create-visual-studio-deployment-project.md) , включающий сценарий PowerShell. Скрипт помещает артефакты из проекта в учетную запись хранения, к которой диспетчер ресурсов имеет доступ. Артефакты — это элементы в проекте, такие как связанные шаблоны, скрипты и двоичные файлы приложения. Если вы хотите продолжить использовать скрипт из проекта, используйте задачу «скрипт PowerShell», описанную в этой статье.

* **Добавьте задачи для копирования и развертывания задач**. Этот параметр предлагает удобную альтернативу скрипту проекта. В конвейере настраиваются две задачи. Одна задача перемещает артефакты в доступное расположение. Другая задача развертывает шаблон из этого расположения. Этот параметр показан в этой статье. См. раздел [задачи копирования и развертывания](#copy-and-deploy-tasks).

## <a name="prepare-your-project"></a>Подготовка проекта

В этой статье предполагается, что ваш шаблон ARM и организация Azure DevOps готовы к созданию конвейера. Ниже показано, как обеспечить готовность к работе.

* У вас есть организация Azure DevOps. Если у вас ее нет, [создайте подписку бесплатно](/azure/devops/pipelines/get-started/pipelines-sign-up). Если у вашей команды уже есть организация Azure DevOps, убедитесь, что вы являетесь администратором проекта Azure DevOps, который вы хотите использовать.

* Вы настроили [Подключение службы](/azure/devops/pipelines/library/connect-to-azure) к подписке Azure. Задачи в конвейере выполняются под удостоверением субъекта-службы. Действия по созданию подключения см. в разделе [Создание проекта DevOps](deployment-tutorial-pipeline.md#create-a-devops-project).

* У вас есть [шаблон ARM](quickstart-create-templates-use-visual-studio-code.md) , определяющий инфраструктуру для проекта.

## <a name="create-pipeline"></a>Создание конвейера

1. Если вы ранее не добавили конвейер, необходимо создать новый конвейер. В Организации Azure DevOps выберите **конвейеры** и **Новый конвейер**.

   ![Добавить новый конвейер](./media/add-template-to-azure-pipelines/new-pipeline.png)

1. Укажите место хранения кода. На следующем рисунке показано, как выбрать **Azure Repos Git**.

   ![Выбор источника кода](./media/add-template-to-azure-pipelines/select-source.png)

1. В этом источнике выберите репозиторий с кодом для проекта.

   ![Выбор репозитория](./media/add-template-to-azure-pipelines/select-repo.png)

1. Выберите тип создаваемого конвейера. Можно выбрать **начальный конвейер**.

   ![Выбор конвейера](./media/add-template-to-azure-pipelines/select-pipeline.png)

Вы можете добавить задачу Azure PowerShell или скопировать файл и развернуть задачи.

## <a name="azure-powershell-task"></a>Задача Azure PowerShell

В этом разделе показано, как настроить непрерывное развертывание с помощью одной задачи, запускающей скрипт PowerShell в проекте. Если вам нужен скрипт PowerShell, который развертывает шаблон, см. статью [Deploy-AzTemplate.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzTemplate.ps1) или [Deploy-AzureResourceGroup.ps1](https://github.com/Azure/azure-quickstart-templates/blob/master/Deploy-AzureResourceGroup.ps1).

Следующий файл YAML создает [задачу Azure PowerShell](/azure/devops/pipelines/tasks/deploy/azure-powershell):

```yml
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzurePowerShell@5
  inputs:
    azureSubscription: 'script-connection'
    ScriptType: 'FilePath'
    ScriptPath: './Deploy-AzTemplate.ps1'
    ScriptArguments: -Location 'centralus' -ResourceGroupName 'demogroup' -TemplateFile templates\mainTemplate.json
    azurePowerShellVersion: 'LatestVersion'
```

Если для задачи задано значение `AzurePowerShell@5` , конвейер использует [модуль AZ](/powershell/azure/new-azureps-module-az). Если вы используете модуль AzureRM в скрипте, задайте для задачи значение `AzurePowerShell@3` .

```yaml
steps:
- task: AzurePowerShell@3
```

Для `azureSubscription` Укажите имя созданного подключения к службе.

```yaml
inputs:
    azureSubscription: '<your-connection-name>'
```

Для `scriptPath` укажите относительный путь от файла конвейера к скрипту. Чтобы увидеть путь, можно просмотреть репозиторий.

```yaml
ScriptPath: '<your-relative-path>/<script-file-name>.ps1'
```

В `ScriptArguments` введите любые параметры, необходимые для скрипта. В следующем примере показаны некоторые параметры для скрипта, но необходимо настроить параметры для скрипта.

```yaml
ScriptArguments: -Location 'centralus' -ResourceGroupName 'demogroup' -TemplateFile templates\mainTemplate.json
```

При нажатии на кнопку **сохранить** конвейер сборки запускается автоматически. Вернитесь к сводке для конвейера сборки и просмотрите состояние.

![Просмотр результатов](./media/add-template-to-azure-pipelines/view-results.png)

Вы можете выбрать выполняющийся в данный момент конвейер для просмотра сведений о задачах. По завершении вы увидите результаты для каждого шага.

## <a name="copy-and-deploy-tasks"></a>Задачи копирования и развертывания

В этом разделе показано, как настроить непрерывное развертывание с помощью двух задач. Первая задача перемещает артефакты в учетную запись хранения, а вторая выполняет развертывание шаблона.

Чтобы скопировать файлы в учетную запись хранения, субъекту-службе для подключения к службе должен быть назначен участник данных BLOB-объекта хранилища или роль владельца данных BLOB-объекта хранилища. Дополнительные сведения см. в статье Начало [работы с AzCopy](../../storage/common/storage-use-azcopy-v10.md).

В следующем YAML показана [задача копирования файлов Azure](/azure/devops/pipelines/tasks/deploy/azure-file-copy).

```yml
trigger:
- master

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureFileCopy@4
  inputs:
    SourcePath: 'templates'
    azureSubscription: 'copy-connection'
    Destination: 'AzureBlob'
    storage: 'demostorage'
    ContainerName: 'projecttemplates'
  name: AzureFileCopy
```

Существует несколько частей этой задачи для пересмотра вашей среды. `SourcePath`Указывает расположение артефактов относительно файла конвейера.

```yaml
SourcePath: '<path-to-artifacts>'
```

Для `azureSubscription` Укажите имя созданного подключения к службе.

```yaml
azureSubscription: '<your-connection-name>'
```

Для хранилища и имени контейнера укажите имена учетной записи хранения и контейнера, которые будут использоваться для хранения артефактов. Учетная запись хранения должна существовать.

```yaml
storage: '<your-storage-account-name>'
ContainerName: '<container-name>'
```

После создания задачи «копирование файла» можно приступить к добавлению задачи для развертывания промежуточного шаблона.

В следующем YAML показана [задача развертывания шаблона Azure Resource Manager](https://github.com/microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceManagerTemplateDeploymentV3/README.md).

```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'copy-connection'
    subscriptionId: '00000000-0000-0000-0000-000000000000'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'demogroup'
    location: 'West US'
    templateLocation: 'URL of the file'
    csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
    csmParametersFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.parameters.json$(AzureFileCopy.StorageContainerSasToken)'
    deploymentMode: 'Incremental'
    deploymentName: 'deploy1'
```

Существует несколько частей этой задачи для более подробного рассмотрения.

- `deploymentScope`: Выберите область развертывания из параметров: `Management Group` , `Subscription` и `Resource Group` . Дополнительные сведения об областях см. в разделе [Области развертывания](deploy-rest.md#deployment-scope).

- `azureResourceManagerConnection`: Укажите имя созданного подключения к службе.

- `subscriptionId`: Укажите идентификатор целевой подписки. Это свойство применяется только к области развертывания группы ресурсов и области развертывания подписки.

- `resourceGroupName` и `location` : укажите имя и расположение группы ресурсов, в которую требуется выполнить развертывание. Задача создает группу ресурсов, если она не существует.

   ```yml
   resourceGroupName: '<resource-group-name>'
   location: '<location>'
   ```

- `csmFileLink`: Укажите ссылку на промежуточный шаблон. При задании значения используйте переменные, возвращаемые задачей «копирование файлов». В следующем примере создается ссылка на шаблон с именем mainTemplate.jsв. Папка с именем **Templates** включается в том случае, когда задача копирования файлов скопировала файл. В конвейере укажите путь к шаблону и имя шаблона.

   ```yml
   csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
   ```

Ваш конвейер выглядит следующим образом:

```yml
trigger:
- master

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureFileCopy@4
  inputs:
    SourcePath: 'templates'
    azureSubscription: 'copy-connection'
    Destination: 'AzureBlob'
    storage: 'demostorage'
    ContainerName: 'projecttemplates'
  name: AzureFileCopy
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'copy-connection'
    subscriptionId: '00000000-0000-0000-0000-000000000000'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'demogroup'
    location: 'West US'
    templateLocation: 'URL of the file'
    csmFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.json$(AzureFileCopy.StorageContainerSasToken)'
    csmParametersFileLink: '$(AzureFileCopy.StorageContainerUri)templates/mainTemplate.parameters.json$(AzureFileCopy.StorageContainerSasToken)'
    deploymentMode: 'Incremental'
    deploymentName: 'deploy1'
```

При нажатии на кнопку **сохранить** конвейер сборки запускается автоматически. Вернитесь к сводке для конвейера сборки и просмотрите состояние.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об использовании операции "что если" в конвейере см. [в разделе тестовые шаблоны ARM с What-If в конвейере](https://4bes.nl/2021/03/06/test-arm-templates-with-what-if/).
* Дополнительные сведения об использовании шаблонов ARM с действиями GitHub см. в статье [Развертывание шаблонов Azure Resource Manager с помощью действий GitHub](deploy-github-actions.md).

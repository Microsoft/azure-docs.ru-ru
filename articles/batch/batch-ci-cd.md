---
title: Использование Azure Pipelines для создания и развертывания решений HPC
description: Узнайте, как развернуть конвейер сборки или выпуска для приложения HPC, работающего в пакетной службе Azure.
author: chrisreddington
ms.author: chredd
ms.date: 03/04/2021
ms.topic: how-to
ms.openlocfilehash: 7170044af58a508ff5a43751cc376f8b8d498444
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102435551"
---
# <a name="use-azure-pipelines-to-build-and-deploy-hpc-solutions"></a>Использование Azure Pipelines для создания и развертывания решений HPC

Средства, предоставляемые Azure DevOps, могут преобразовываться в автоматизированное создание и тестирование решений для высокопроизводительных вычислений (HPC). [Azure pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines) предоставляет ряд современных процессов непрерывной интеграции (CI) и непрерывного развертывания (CD) для создания, развертывания, тестирования и мониторинга программного обеспечения. Эти процессы ускоряют доставку программного обеспечения, позволяя сосредоточиться на коде, а не на поддержке инфраструктуры и операций.

В этой статье объясняется, как настроить процессы CI/CD с помощью [Azure pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines) для решений HPC, развернутых в пакетной службе Azure.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения действий, описанных в этой статье, требуется [Организация Azure DevOps](/azure/devops/organizations/accounts/create-organization). Вам также потребуется [создать проект в Azure DevOps](/azure/devops/organizations/projects/create-project).

Перед началом работы рекомендуется иметь базовое представление об [управлении исходным кодом](/azure/devops/user-guide/source-control) и [Azure Resource Manager синтаксисе шаблона](../azure-resource-manager/templates/template-syntax.md) .

## <a name="create-an-azure-pipeline"></a>Создание конвейера Azure

В этом примере вы создадите конвейер сборки и выпуска для развертывания инфраструктуры пакетной службы Azure и выпуска пакета приложения. Если код разрабатывается локально, поток развертывания в целом выглядит так:

![Схема, показывающая поток развертывания в конвейере,](media/batch-ci-cd/DeploymentFlow.png)

В этом примере используется несколько шаблонов Azure Resource Manager и существующих двоичных файлов. Вы можете скопировать эти примеры в репозиторий и отправить их в Azure DevOps.

### <a name="understand-the-azure-resource-manager-templates"></a>Общие сведения о шаблонах Azure Resource Manager

В этом примере используется несколько шаблонов Azure Resource Manager для развертывания решения. Три шаблона возможностей (аналогично единицам или модулям) используются для реализации определенного фрагмента функциональности. Затем шаблон комплексного решения (deployment.json) используется для развертывания этих базовых шаблонов возможностей. Эта [Связанная структура шаблона ](../azure-resource-manager/templates/deployment-tutorial-linked-template.md) позволяет отдельно тестировать каждый шаблон возможности и повторно использовать его в разных решениях.

![Схема, показывающая структуру связанных шаблонов с помощью шаблонов Azure Resource Manager.](media/batch-ci-cd/ARMTemplateHierarchy.png)

Этот шаблон определяет учетную запись хранения Azure, которая необходима для развертывания приложения в учетной записи пакетной службы. Подробные сведения см. в справочнике по [шаблонам диспетчер ресурсов для типов ресурсов Microsoft. Storage](/azure/templates/microsoft.storage/allversions).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[parameters('accountName')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "apiVersion": "2018-02-01",
            "location": "[resourceGroup().location]",
            "properties": {}
        }
    ],
    "outputs": {
        "blobEndpoint": {
          "type": "string",
          "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))).primaryEndpoints.blob]"
        },
        "resourceId": {
          "type": "string",
          "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))]"
        }
    }
}
```

Следующий шаблон определяет [учетную запись пакетной службы Azure](accounts.md). Учетная запись пакетной службы выступает в качестве платформы для выполнения многочисленных приложений в [пулах](nodes-and-pools.md#pools). Дополнительные сведения см. в справочнике по [шаблонам диспетчер ресурсов для типов ресурсов Microsoft.BatCH](/azure/templates/microsoft.batch/allversions).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
            }
        },
        "storageAccountId": {
           "type": "string",
           "metadata": {
                "description": "ID of the Azure Storage Account"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "[parameters('batchAccountName')]",
            "type": "Microsoft.Batch/batchAccounts",
            "apiVersion": "2017-09-01",
            "location": "[resourceGroup().location]",
            "properties": {
              "poolAllocationMode": "BatchService",
              "autoStorage": {
                  "storageAccountId": "[parameters('storageAccountId')]"
              }
            }
          }
    ],
    "outputs": {}
}
```

Следующий шаблон создает пул пакетов в учетной записи пакетной службы. Дополнительные сведения см. в справочнике по [шаблонам диспетчер ресурсов для типов ресурсов Microsoft.BatCH](/azure/templates/microsoft.batch/allversions).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
           }
        },
        "batchAccountPoolName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account Pool"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "name": "[concat(parameters('batchAccountName'),'/', parameters('batchAccountPoolName'))]",
            "type": "Microsoft.Batch/batchAccounts/pools",
            "apiVersion": "2017-09-01",
            "properties": {
                "deploymentConfiguration": {
                    "virtualMachineConfiguration": {
                        "imageReference": {
                            "publisher": "Canonical",
                            "offer": "UbuntuServer",
                            "sku": "18.04-LTS",
                            "version": "latest"
                        },
                        "nodeAgentSkuId": "batch.node.ubuntu 18.04"
                    }
                },
                "vmSize": "Standard_D1_v2"
            }
          }
    ],
    "outputs": {}
}
```

Окончательный шаблон выступает в качестве Orchestrator и выполняет развертывание трех базовых шаблонов возможностей.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "templateContainerUri": {
           "type": "string",
           "metadata": {
                "description": "URI of the Blob Storage Container containing the Azure Resource Manager templates"
            }
        },
        "templateContainerSasToken": {
           "type": "string",
           "metadata": {
                "description": "The SAS token of the container containing the Azure Resource Manager templates"
            }
        },
        "applicationStorageAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
            }
         },
        "batchAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account"
            }
         },
         "batchAccountPoolName": {
             "type": "string",
             "metadata": {
                  "description": "Name of the Azure Batch Account Pool"
              }
          }
    },
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "storageAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/storageAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "accountName": {"value": "[parameters('applicationStorageAccountName')]"}
                }
            }
        },  
        {
            "apiVersion": "2017-05-10",
            "name": "batchAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "storageAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "storageAccountId": {"value": "[reference('storageAccountDeployment').outputs.resourceId.value]"}
                }
            }
        },
        {
            "apiVersion": "2017-05-10",
            "name": "poolDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "batchAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccountPool.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "batchAccountPoolName": {"value": "[parameters('batchAccountPoolName')]"}
                }
            }
        }
    ],
    "outputs": {}
}
```

### <a name="understand-the-hpc-solution"></a>Общие сведения о решении HPC

Как отмечалось ранее, в этом примере используются несколько шаблонов Azure Resource Manager и существующих двоичных файлов. Вы можете скопировать эти примеры в репозиторий и отправить их в Azure DevOps.

Для этого решения в качестве пакета приложения используется FFmpeg. Вы можете [скачать пакет FFmpeg](https://github.com/GyanD/codexffmpeg/releases/tag/4.3.1-2020-11-08) , если он еще не создан.

![Снимок экрана структуры репозитория.](media/batch-ci-cd/git-repository.jpg)

Этот репозиторий состоит из четырех основных разделов:

- Папка **ARM-Templates** , содержащая шаблоны Azure Resource Manager
- Папка **HPC-Application** , содержащая Windows 64-разрядная версия [FFmpeg 4.3.1](https://github.com/GyanD/codexffmpeg/releases/tag/4.3.1-2020-11-08).
- Папка **конвейеров** , содержащая файл YAML, определяющий процесс построения конвейера.
- Необязательно. папка **Client-Application** , которая является копией [файла .NET пакетной службы Azure с помощью примера FFmpeg](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) . Это приложение не требуется для этой статьи.


> [!NOTE]
> Это лишь один из примеров структуры для базы кода. Такой подход служит для демонстрации того, что код приложения, инфраструктуры и конвейера хранится в одном репозитории.

Теперь, когда исходный код настроен, можно начать первую сборку.

## <a name="continuous-integration"></a>Непрерывная интеграция

[Azure Pipelines](/azure/devops/pipelines/get-started/) в составе служб Azure DevOps Services помогает реализовать конвейер сборки, тестирования и развертывания для приложений.

На этом этапе конвейера обычно выполняются тесты для проверки кода и сборки соответствующих компонентов программного обеспечения. Число и типы тестов, а также дополнительные задачи зависят от общей стратегии сборки и выпуска.

## <a name="prepare-the-hpc-application"></a>Подготовка приложения HPC

В этом разделе вы будете работать с папкой **HPC-Application** . Эта папка содержит программное обеспечение (FFmpeg), которое будет выполняться в учетной записи пакетной службы Azure.

1. Перейдите к разделу "Сборки" Azure Pipelines в вашей организации Azure DevOps. Создайте **конвейер**.

    ![Снимок экрана: Новый конвейер.](media/batch-ci-cd/new-build-pipeline.jpg)

1. Создать конвейер сборки можно двумя способами:

    а. [Используйте визуальный конструктор](/azure/devops/pipelines/get-started-designer). Для этого выберите "использовать визуальный конструктор" на **новой странице конвейера** .

    b. [Используйте сборки YAML](/azure/devops/pipelines/get-started-yaml). Вы можете создать новый конвейер YAML, щелкнув параметр Azure Repos или GitHub на **новой странице конвейера** . Кроме того, можно сохранить приведенный ниже пример в системе управления версиями и ссылаться на существующий файл YAML, выбрав визуальный конструктор, а затем используя шаблон YAML.

    ```yml
    # To publish an application into Azure Batch, we need to
    # first zip the file, and then publish an artifact, so that
    # we can take the necessary steps in our release pipeline.
    steps:
    # First, we Zip up the files required in the Batch Account
    # For this instance, those are the ffmpeg files
    - task: ArchiveFiles@2
      displayName: 'Archive applications'
      inputs:
        rootFolderOrFile: hpc-application
        includeRootFolder: false
        archiveFile: '$(Build.ArtifactStagingDirectory)/package/$(Build.BuildId).zip'
    # Publish that zip file, so that we can use it as part
    # of our Release Pipeline later
    - task: PublishPipelineArtifact@0
      inputs:
        artifactName: 'hpc-application'
        targetPath: '$(Build.ArtifactStagingDirectory)/package'
    ```

1. Когда сборка будет настроена требуемым образом, выберите команду **Сохранить и поместить в очередь**. Если включена непрерывная интеграция (в разделе **Триггеры**), сборка будет автоматически запускаться при внесении в репозиторий новой фиксации, соответствующей условиям в сборке.

    ![Снимок экрана существующего конвейера сборки.](media/batch-ci-cd/existing-build-pipeline.jpg)

1. Чтобы наблюдать за процессом сборки в Azure DevOps, перейдите к разделу **Сборка** в Azure Pipelines. Выберите соответствующую сборку из определения сборки.

    ![Снимок экрана с динамическими выходными данными из сборки в Azure DevOps.](media/batch-ci-cd/Build-1.jpg)

> [!NOTE]
> Если для выполнения решения HPC используется клиентское приложение, необходимо создать отдельное определение сборки для этого приложения. Ряд руководств можно найти в документации по [Azure Pipelines](/azure/devops/pipelines/get-started/index).

## <a name="continuous-deployment"></a>Непрерывное развертывание

Azure Pipelines также используется для развертывания приложения и базовой инфраструктуры. [Конвейеры выпуска](/azure/devops/pipelines/release) обеспечивают непрерывное развертывание и автоматизируют процесс выпуска.

### <a name="deploy-your-application-and-underlying-infrastructure"></a>Развертывание приложения и базовой инфраструктуры

Существует ряд действий, связанных с развертыванием инфраструктуры. Так как в этом решении используются [связанные шаблоны](../azure-resource-manager/templates/linked-templates.md), эти шаблоны должны быть доступны из общедоступной конечной точки (HTTP или HTTPS). Это может быть репозиторий в GitHub, учетная запись хранилища BLOB-объектов Azure или другое место хранения. Безопасность переданных артефактов шаблона может обеспечиваться за счет хранения в частном режиме и доступа к ним с помощью некоторого маркера подписанного URL-адрес (SAS).

В приведенном ниже примере показано, как развернуть инфраструктуру с шаблонами из большого двоичного объекта службы хранилища Azure.

1. Создайте **новое определение выпуска**, а затем выберите пустое определение. Переименуйте созданную среду, чтобы она была важна для вашего конвейера.

    ![Снимок экрана начального конвейера выпуска.](media/batch-ci-cd/Release-0.jpg)

1. Создайте зависимость от конвейера сборки, чтобы получить выходные данные для приложения HPC.

    > [!NOTE]
    > Запишите **Псевдоним источника**, так как это потребуется при создании задач в определении выпуска.

    ![Снимок экрана, на котором показана ссылка артефакта на Хпкаппликатионпаккаже в соответствующем конвейере сборки.](media/batch-ci-cd/Release-1.jpg)

1. Создайте ссылку на другой артефакт, на этот раз репозиторий Azure. Это необходимо для доступа к шаблонам Resource Manager, хранящимся в репозитории. Так как шаблоны Resource Manager не требуют компиляции, их не нужно отправлять через конвейер сборки.

    > [!NOTE]
    > Опять же, обратите внимание на **Псевдоним источника**, так как это потребуется позже.

    ![Снимок экрана, на котором показана ссылка артефакта на Azure Repos.](media/batch-ci-cd/Release-2.jpg)

1. Перейдите к разделу **variables**. Вам потребуется создать несколько переменных в конвейере, чтобы не нужно было повторно вводить одни и те же данные в несколько задач. В этом примере используются следующие переменные:

   - **аппликатионсторажеаккаунтнаме**: имя учетной записи хранения, в которой хранятся двоичные файлы приложения HPC.
   - **батчаккаунтаппликатионнаме**: имя приложения в учетной записи пакетной службы.
   - **батчаккаунтнаме**: имя учетной записи пакетной службы.
   - **batchAccountPoolName**: имя пула виртуальных машин, выполняющих обработку.
   - **батчаппликатионид**: уникальный идентификатор для приложения пакетной службы.
   - **батчаппликатионверсион**: Семантическая версия приложения пакетной службы (то есть двоичные файлы FFmpeg)
   - **Расположение**: расположение для развертываемых ресурсов Azure
   - **resourceGroupName**: имя создаваемой группы ресурсов и место, где будут развернуты ресурсы
   - **storageAccountName**: имя учетной записи хранения, в которой содержатся связанные шаблоны диспетчер ресурсов

   ![Снимок экрана, показывающий переменные, заданные для выпуска Azure Pipelines.](media/batch-ci-cd/Release-4.jpg)

1. Перейдите к задачам среды разработки. На приведенном ниже снимке экрана представлено шесть задач. Эти задачи скачивают ZIP-файлы ffmpeg, развертывают учетную запись хранения для размещения вложенных шаблонов Resource Manager, копируют эти шаблоны Resource Manager в учетную запись хранения, развертывают учетную запись пакетной службы и необходимые зависимости, создают приложение в учетной записи пакетной службы Azure и передают пакет приложения в учетную запись пакетной службы Azure.

    ![Снимок экрана, показывающий задачи, используемые для освобождения приложения HPC в пакетной службе Azure.](media/batch-ci-cd/Release-3.jpg)

1. Добавьте задачу **Скачать артефакт конвейера (предварительная версия)** и задайте указанные ниже свойства.
    - **Отображаемое имя**: скачивание ApplicationPackage в агент
    - **Имя скачиваемого артефакта**: hpc-application
    - **Путь для скачивания**: $(System.DefaultWorkingDirectory)

1. Создайте учетную запись хранения для хранения шаблонов Azure Resource Manager. Можно использовать существующую учетную запись хранения из решения, но для поддержки этого автономного примера и изоляции содержимого необходимо создать выделенную учетную запись хранения.

    Добавьте задачу **Развертывание группы ресурсов Azure** и задайте указанные ниже свойства.
    - **Отображаемое имя:** Развертывание учетной записи хранения для шаблонов диспетчер ресурсов
    - **Подписка Azure:** Выберите подходящую подписку Azure
    - **Action** (Действие). Создание или изменение группы ресурсов
    - **Группа ресурсов**: $(resourceGroupName)
    - **Расположение**: $(location)
    - **Шаблон**: $(System.ArtifactsDirectory)/ **{псевдоним_источника_артефактов_репозитория_Azure}** /arm-templates/storageAccount.json
    - **Переопределение параметров шаблона**: -accountName $(storageAccountName)

1. Передайте артефакты из системы управления версиями в учетную запись хранения с помощью Azure Pipelines. В рамках этой Azure Pipelinesной задачи URI контейнера учетной записи хранения и маркер SAS могут быть выведены в переменную в Azure Pipelines, что позволяет повторно использовать их в рамках этой фазы агента.

    Добавьте задачу **Копирование файлов Azure** и задайте указанные ниже свойства.
    - **Источник**: $(System.ArtifactsDirectory)/ **{псевдоним_источника_артефактов_репозитория_Azure}** /arm-templates/
    - **Тип подключения Azure**: Azure Resource Manager
    - **Подписка Azure:** Выберите подходящую подписку Azure
    - **Тип назначения**: Большой двоичный объект Azure
    - **Учетная запись хранения Диспетчера ресурсов**: $(storageAccountName)
    - **Имя контейнера**: templates
    - **Код URI контейнера хранилища**: templateContainerUri
    - **Токен SAS контейнера хранилища**: templateContainerSasToken

1. Разверните шаблон оркестратора. Этот шаблон включает параметры для URI контейнера учетной записи хранения и маркер SAS. Переменные, необходимые в шаблоне диспетчер ресурсов, хранятся в разделе Variables определения выпуска или были заданы из другой Azure Pipelines задачи (например, в составе задачи копирования больших двоичных объектов Azure).

    Добавьте задачу **Развертывание группы ресурсов Azure** и задайте указанные ниже свойства.
    - **Отображаемое имя**: развертывание пакетной службы Azure
    - **Подписка Azure:** Выберите подходящую подписку Azure
    - **Action** (Действие). Создание или изменение группы ресурсов
    - **Группа ресурсов**: $(resourceGroupName)
    - **Расположение**: $(location)
    - **Шаблон**: $(System.ArtifactsDirectory)/ **{псевдоним_источника_артефактов_репозитория_Azure}** /arm-templates/deployment.json
    - **Переопределение параметров шаблона**: `-templateContainerUri $(templateContainerUri) -templateContainerSasToken $(templateContainerSasToken) -batchAccountName $(batchAccountName) -batchAccountPoolName $(batchAccountPoolName) -applicationStorageAccountName $(applicationStorageAccountName)`

   Распространенной практикой является использование задач Azure Key Vault. Если для субъекта-службы, подключенного к подписке Azure, установлены соответствующие политики доступа, он может скачать секреты из Azure Key Vault и использовать их в качестве переменных в конвейере. Имя секрета будет задано со связанным значением. Например, в определении выпуска на секрет sshPassword можно ссылаться с помощью $(sshPassword).

1. На следующих шагах вызывается Azure CLI. Первый используется для создания приложения в пакетной службе Azure и отправки связанных пакетов.

    Добавьте задачу **Azure CLI** и задайте указанные ниже свойства.
    - **Отображаемое имя:** Создание приложения в учетной записи пакетной службы Azure
    - **Подписка Azure:** Выберите подходящую подписку Azure
    - **Расположение скрипта**: встроенный скрипт
    - **Встроенный скрипт**: `az batch application create --application-id $(batchApplicationId) --name $(batchAccountName) --resource-group $(resourceGroupName)`

1. Второй шаг используется для отправки связанных пакетов в приложение (в данном случае файлы FFmpeg).

    Добавьте задачу **Azure CLI** и задайте указанные ниже свойства.
    - **Отображаемое имя:** Отправка пакета в учетную запись пакетной службы Azure
    - **Подписка Azure:** Выберите подходящую подписку Azure
    - **Расположение скрипта**: встроенный скрипт
    - **Встроенный скрипт**: `az batch application package create --application-id $(batchApplicationId)  --name $(batchAccountName)  --resource-group $(resourceGroupName) --version $(batchApplicationVersion) --package-file=$(System.DefaultWorkingDirectory)/$(Release.Artifacts.{YourBuildArtifactSourceAlias}.BuildId).zip`

    > [!NOTE]
    > Номер версии пакета приложения присваивается переменной. Это позволяет перезаписывать предыдущие версии пакета и позволяет вручную управлять номером версии пакета, отправленного в пакетную службу Azure.

1. Создайте выпуск, выбрав **Выпуск > Создать выпуск**. После активации выберите ссылку на новый выпуск, чтобы просмотреть его состояние.

1. Просмотрите выходные данные агента, нажав кнопку **журналы** под средой.

    ![Снимок экрана, показывающий состояние выпуска.](media/batch-ci-cd/Release-5.jpg)

## <a name="test-the-environment"></a>Тестирование среды

После настройки среды убедитесь в том, что приведенные ниже тесты выполняются успешно.

Подключитесь к новой учетной записи пакетной службы Azure с помощью Azure CLI из командной строки PowerShell.

- Войдите в свою учетную запись Azure с помощью команды `az login` и следуйте инструкциям по проверке подлинности.
- Теперь выполните проверку подлинности учетной записи пакетной службы: `az batch account login -g <resourceGroup> -n <batchAccount>`

#### <a name="list-the-available-applications"></a>Получение списка доступных приложений

```azurecli
az batch application list -g <resourcegroup> -n <batchaccountname>
```

#### <a name="check-the-pool-is-valid"></a>Проверка допустимости пула

```azurecli
az batch pool list
```

Обратите внимание на значение `currentDedicatedNodes` в выходных данных команды. Это значение будет скорректировано в следующем тесте.

#### <a name="resize-the-pool"></a>Изменение размера пула

Измените размер пула так, чтобы в нем было достаточно вычислительных узлов для тестирования заданий и задач. Выполните проверку текущего состояния до завершения изменения размера и получения нужного количества узлов с помощью команды pool list.

```azurecli
az batch pool resize --pool-id <poolname> --target-dedicated-nodes 4
```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с этими учебниками, чтобы узнать, как взаимодействовать с учетной записью пакетной службы с помощью простого приложения.

- [Запуск параллельной рабочей нагрузки с помощью пакетной службы Azure с использованием API Python](tutorial-parallel-python.md)
- [Запуск параллельной рабочей нагрузки с помощью пакетной службы Azure с использованием .NET API](tutorial-parallel-dotnet.md)

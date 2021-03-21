---
title: Настройка среды разработки для скриптов развертывания в шаблонах | Документация Майкрософт
description: Настройка среды разработки для скриптов развертывания в шаблонах Azure Resource Manager (шаблоны ARM).
services: azure-resource-manager
author: mumian
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 12/14/2020
ms.author: jgao
ms.openlocfilehash: b2e1ffb3cbd513766945864e33589c46284bf942
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102200943"
---
# <a name="configure-development-environment-for-deployment-scripts-in-arm-templates"></a>Настройка среды разработки для скриптов развертывания в шаблонах ARM

Узнайте, как создать среду разработки для разработки и тестирования сценариев развертывания шаблонов ARM с помощью образа скрипта развертывания. Можно либо создать [экземпляр контейнера Azure](../../container-instances/container-instances-overview.md) , либо использовать [DOCKER](https://docs.docker.com/get-docker/). Оба варианта описаны в этой статье.

## <a name="prerequisites"></a>Предварительные требования

### <a name="azure-powershell-container"></a>Контейнер Azure PowerShell

Если у вас нет скрипта развертывания Azure PowerShell, можно создать файл *hello.ps1* , используя следующее содержимое:

```powershell
param([string] $name)
$output = 'Hello {0}' -f $name
Write-Output $output
$DeploymentScriptOutputs = @{}
$DeploymentScriptOutputs['text'] = $output
```

### <a name="azure-cli-container"></a>Контейнер Azure CLI

Для Azure CLI образа контейнера можно создать файл *Hello.sh* , используя следующее содержимое:

```bash
firstname=$1
lastname=$2
output="{\"name\":{\"displayName\":\"$firstname $lastname\",\"firstName\":\"$firstname\",\"lastName\":\"$lastname\"}}"
echo -n "Hello "
echo $output | jq -r '.name.displayName'
```

> [!NOTE]
> При запуске скрипта развертывания Azure CLI переменная среды, именуемая, `AZ_SCRIPTS_OUTPUT_PATH` хранит расположение выходного файла скрипта. Переменная среды недоступна в контейнере среды разработки. Дополнительные сведения о работе с выходными данными Azure CLI см. в разделе [Работа с выходами из скрипта CLI](deployment-script-template.md#work-with-outputs-from-cli-script).

## <a name="use-azure-powershell-container-instance"></a>Использовать Azure PowerShell экземпляр контейнера

Для создания сценариев на компьютере необходимо создать учетную запись хранения и подключить учетную запись хранения к экземпляру контейнера. Чтобы можно было передать скрипт в учетную запись хранения и запустить скрипт в экземпляре контейнера.

> [!NOTE]
> Учетная запись хранения, созданная для тестирования сценария, не является той же учетной записью хранения, которую использует служба скриптов развертывания для выполнения скрипта. Служба скриптов развертывания создает уникальное имя в виде общей папки при каждом выполнении.

### <a name="create-an-azure-powershell-container-instance"></a>Создание Azure PowerShell экземпляра контейнера

Следующий шаблон Azure Resource Manager (шаблон ARM) создает экземпляр контейнера и файловый ресурс, а затем подключает общую папку к образу контейнера.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "type": "string",
      "metadata": {
        "description": "Specify a project name that is used for generating resource names."
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azuredeploymentscripts-powershell:az5.2",
      "metadata": {
        "description": "Specify the container image."
      }
    },
    "mountPath": {
      "type": "string",
      "defaultValue": "/mnt/azscripts/azscriptinput",
      "metadata": {
        "description": "Specify the mount path."
      }
    }
  },
  "variables": {
    "storageAccountName": "[tolower(concat(parameters('projectName'), 'store'))]",
    "fileShareName": "[concat(parameters('projectName'), 'share')]",
    "containerGroupName": "[concat(parameters('projectName'), 'cg')]",
    "containerName": "[concat(parameters('projectName'), 'container')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    },
    {
      "type": "Microsoft.Storage/storageAccounts/fileServices/shares",
      "apiVersion": "2019-06-01",
      "name": "[concat(variables('storageAccountName'), '/default/', variables('fileShareName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "name": "[variables('containerGroupName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ],
      "properties": {
        "containers": [
          {
            "name": "[variables('containerName')]",
            "properties": {
              "image": "[parameters('containerImage')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "protocol": "TCP",
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "[parameters('mountPath')]"
                }
              ],
              "command": [
                "/bin/sh",
                "-c",
                "pwsh -c 'Start-Sleep -Seconds 1800'"
              ]
            }
          }
        ],
        "osType": "Linux",
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
              "readOnly": false,
              "shareName": "[variables('fileShareName')]",
              "storageAccountName": "[variables('storageAccountName')]",
              "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
            }
          }
        ]
      }
    }
  ]
}
```

По умолчанию для пути подключения используется значение `/mnt/azscripts/azscriptinput` . Это путь в экземпляре контейнера, в котором он подключен к общей папке.

Образ контейнера по умолчанию, указанный в шаблоне, — **MCR.Microsoft.com/azuredeploymentscripts-PowerShell:az5.2**. См. список всех [поддерживаемых версий Azure PowerShell](https://mcr.microsoft.com/v2/azuredeploymentscripts-powershell/tags/list).

Шаблон приостанавливает экземпляр контейнера через 1 800 секунд. У вас 30 минут до того, как экземпляр контейнера переходит в состояние завершения и сеанс завершается.

Чтобы развернуть шаблон, выполните следующую команду:

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Location $location -name $resourceGroupName
New-AzResourceGroupDeployment -resourceGroupName $resourceGroupName -TemplateFile $templatefile -projectName $projectName
```

### <a name="upload-the-deployment-script"></a>Отправка скрипта развертывания

Отправьте скрипт развертывания в учетную запись хранения. Ниже приведен пример сценария PowerShell.

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name that you used earlier"
$fileName = Read-Host -Prompt "Enter the deployment script file name with the path"

$resourceGroupName = "${projectName}rg"
$storageAccountName = "${projectName}store"
$fileShareName = "${projectName}share"

$context = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName).Context
Set-AzStorageFileContent -Context $context -ShareName $fileShareName -Source $fileName -Force
```

Вы также можете передать файл с помощью портал Azure или Azure CLI.

### <a name="test-the-deployment-script"></a>Тестирование скрипта развертывания

1. В портал Azure откройте группу ресурсов, в которой развернут экземпляр контейнера, и учетную запись хранения.
2. Откройте группу контейнеров. Имя группы контейнеров по умолчанию — имя проекта, к которому добавляется *CG*. Экземпляр контейнера находится в состоянии **выполнения** .
3. В меню ресурс выберите **контейнеры**. Имя экземпляра контейнера — это имя проекта, к которому добавляется *контейнер*.

    ![Снимок экрана: экземпляр контейнера Connect скрипта развертывания в портал Azure.](./media/deployment-script-template-configure-dev/deployment-script-container-instance-connect.png)

4. Выберите **подключить** и нажмите кнопку **подключить**. Если не удается подключиться к экземпляру контейнера, перезапустите группу контейнеров и повторите попытку.
5. В области консоли выполните следующие команды:

    ```console
    cd /mnt/azscripts/azscriptinput
    ls
    pwsh ./hello.ps1 "John Dole"
    ```

    Выходные данные — **Hello Джон дружащим**.

    ![Снимок экрана сценария развертывания "подключить тестовый вывод экземпляра контейнера" в консоли.](./media/deployment-script-template-configure-dev/deployment-script-container-instance-test.png)

## <a name="use-an-azure-cli-container-instance"></a>Использование экземпляра контейнера Azure CLI

Чтобы создать скрипты на компьютере, создайте учетную запись хранения и подключите учетную запись хранения к экземпляру контейнера. Затем можно отправить скрипт в учетную запись хранения и запустить скрипт в экземпляре контейнера.

> [!NOTE]
> Учетная запись хранения, созданная для тестирования сценария, не совпадает с учетной записью хранения, используемой службой скриптов развертывания для выполнения скрипта. Служба скриптов развертывания создает уникальное имя в качестве общей папки при каждом выполнении.

### <a name="create-an-azure-cli-container-instance"></a>Создание Azure CLI экземпляра контейнера

Следующий шаблон ARM создает экземпляр контейнера и общую папку, а затем подключает общую папку к образу контейнера:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "type": "string",
      "metadata": {
        "description": "Specify a project name that is used for generating resource names."
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azure-cli:2.9.1",
      "metadata": {
        "description": "Specify the container image."
      }
    },
    "mountPath": {
      "type": "string",
      "defaultValue": "/mnt/azscripts/azscriptinput",
      "metadata": {
        "description": "Specify the mount path."
      }
    }
  },
  "variables": {
    "storageAccountName": "[tolower(concat(parameters('projectName'), 'store'))]",
    "fileShareName": "[concat(parameters('projectName'), 'share')]",
    "containerGroupName": "[concat(parameters('projectName'), 'cg')]",
    "containerName": "[concat(parameters('projectName'), 'container')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    },
    {
      "type": "Microsoft.Storage/storageAccounts/fileServices/shares",
      "apiVersion": "2019-06-01",
      "name": "[concat(variables('storageAccountName'), '/default/', variables('fileShareName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "name": "[variables('containerGroupName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ],
      "properties": {
        "containers": [
          {
            "name": "[variables('containerName')]",
            "properties": {
              "image": "[parameters('containerImage')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "protocol": "TCP",
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "[parameters('mountPath')]"
                }
              ],
              "command": [
                "/bin/bash",
                "-c",
                "echo hello; sleep 1800"
              ]
            }
          }
        ],
        "osType": "Linux",
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
              "readOnly": false,
              "shareName": "[variables('fileShareName')]",
              "storageAccountName": "[variables('storageAccountName')]",
              "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
            }
          }
        ]
      }
    }
  ]
}
```

По умолчанию для пути подключения используется значение `/mnt/azscripts/azscriptinput` . Это путь в экземпляре контейнера, в котором он подключен к общей папке.

Образ контейнера по умолчанию, указанный в шаблоне, — **MCR.Microsoft.com/Azure-CLI:2.9.1**. См. список [поддерживаемых версий Azure CLI](https://mcr.microsoft.com/v2/azure-cli/tags/list).

> [!IMPORTANT]
> Скрипт развертывания использует доступные образы CLI из реестра контейнеров Майкрософт (мкр). Сертификация образа CLI для сценария развертывания занимает около одного месяца. Не используйте версии CLI, выпущенные менее 30 дней назад. Чтобы найти даты выпуска образов, ознакомьтесь с [заметками о выпуске Azure CLI](/cli/azure/release-notes-azure-cli). Если используется неподдерживаемая версия, в сообщении об ошибке выводится список поддерживаемых версий.

Шаблон приостанавливает экземпляр контейнера через 1 800 секунд. У вас 30 минут до того, как экземпляр контейнера перейдет в состояние терминала, и сеанс завершится.

Чтобы развернуть шаблон, выполните следующую команду:

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Location $location -name $resourceGroupName
New-AzResourceGroupDeployment -resourceGroupName $resourceGroupName -TemplateFile $templatefile -projectName $projectName
```

### <a name="upload-the-deployment-script"></a>Отправка скрипта развертывания

Отправьте скрипт развертывания в учетную запись хранения. Ниже приведен пример PowerShell.

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name that you used earlier"
$fileName = Read-Host -Prompt "Enter the deployment script file name with the path"

$resourceGroupName = "${projectName}rg"
$storageAccountName = "${projectName}store"
$fileShareName = "${projectName}share"

$context = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName).Context
Set-AzStorageFileContent -Context $context -ShareName $fileShareName -Source $fileName -Force
```

Вы также можете передать файл с помощью портал Azure или Azure CLI.

### <a name="test-the-deployment-script"></a>Тестирование скрипта развертывания

1. В портал Azure откройте группу ресурсов, в которой развернут экземпляр контейнера, и учетную запись хранения.
1. Откройте группу контейнеров. Имя группы контейнеров по умолчанию — имя проекта, к которому добавляется *CG*. Экземпляр контейнера отображается в состоянии **выполняется** .
1. В меню ресурс выберите **контейнеры**. Имя экземпляра контейнера — это имя проекта, к которому добавляется *контейнер*.

    ![экземпляр контейнера подключения скрипта развертывания](./media/deployment-script-template-configure-dev/deployment-script-container-instance-connect.png)

1. Выберите **подключить** и нажмите кнопку **подключить**. Если не удается подключиться к экземпляру контейнера, перезапустите группу контейнеров и повторите попытку.
1. В области консоли выполните следующие команды:

    ```console
    cd /mnt/azscripts/azscriptinput
    ls
    ./hello.sh John Dole
    ```

    Выходные данные — **Hello Джон дружащим**.

    ![тест экземпляра контейнера скрипта развертывания](./media/deployment-script-template-configure-dev/deployment-script-container-instance-test-cli.png)

## <a name="use-docker"></a>Использование DOCKER

Вы можете использовать предварительно настроенный образ контейнера DOCKER в качестве среды разработки скриптов развертывания. Сведения об установке DOCKER см. в статье [Получение DOCKER](https://docs.docker.com/get-docker/).
Также необходимо настроить общий доступ к файлам, чтобы подключить каталог, содержащий скрипты развертывания, в контейнер DOCKER.

1. Извлеките образ контейнера сценария развертывания на локальный компьютер.

    ```command
    docker pull mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    В примере используется версия PowerShell 4.3.0.

    Чтобы извлечь образ CLI из мкр, выполните следующие действия.

    ```command
    docker pull mcr.microsoft.com/azure-cli:2.0.80
    ```

    В этом примере используется версия CLI 2.0.80. Сценарий развертывания использует образы контейнеров CLI по умолчанию, доступные [здесь](https://hub.docker.com/_/microsoft-azure-cli).

1. Локальное выполнение образа DOCKER.

    ```command
    docker run -v <host drive letter>:/<host directory name>:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    Замените **&lt;host driver letter>** и **&lt;host directory name>** , указав существующую папку на общем диске. Она будет сопоставлена с папкой _/data_ в контейнере. Например, для сопоставлений _д:\доккер_:

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    **-it** означает поддержание активности образа контейнера.

    Пример для CLI.

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azure-cli:2.0.80
    ```

1. На следующем снимке экрана показано, как запустить сценарий PowerShell, учитывая, что на общем диске имеется *helloworld.ps1* файл.

    ![Диск Docker сценария развертывания шаблона Resource Manager в командной строке](./media/deployment-script-template/resource-manager-deployment-script-docker-cmd.png)

После успешного тестирования скрипта его можно использовать в качестве скрипта развертывания в шаблонах.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как использовать сценарии развертывания. Теперь вы можете изучить учебник по сценариям развертывания:

> [!div class="nextstepaction"]
> [Учебник. Использование скриптов развертывания в шаблонах ARM](./template-tutorial-deployment-script.md)

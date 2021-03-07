---
title: Включение Azure Monitor на устройстве с графическим процессором Azure Stacks Pro
description: Описание включения Azure Monitor на устройстве с графическим процессором Azure Stack.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: c2f66895fccd14dcffd8c5570f1d5f46933090aa
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102439193"
---
# <a name="enable-azure-monitor-on-your-azure-stack-edge-pro-gpu-device"></a>Включение Azure Monitor на устройстве Azure Stackного графического процессора Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Мониторинг контейнеров на устройстве с Azure Stack ребра Pro очень важен, если вы используете несколько приложений для вычислений. Azure Monitor позволяет получать журналы контейнеров, а также метрики памяти и процессора из кластера Kubernetes, работающего на вашем устройстве.

В этой статье описываются шаги, необходимые для включения Azure Monitor на устройстве и сбора журналов контейнеров в Log Analytics рабочей области. Хранилище метрик Azure Monitor в настоящее время не поддерживается на устройстве с графическим процессором Azure Stack. 


## <a name="prerequisites"></a>Предварительные требования

Прежде чем начать, вам потребуется:

- Устройство Azure Stack пограничной Pro. Убедитесь, что устройство активировано в соответствии с инструкциями в разделе [учебник. Активация устройства](azure-stack-edge-gpu-deploy-activate.md).
- Вы завершили **настройку шага вычислений** в соответствии с [руководством: Настройка вычислений на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-configure-compute.md) на устройстве. Устройство должно иметь связанный ресурс центра Интернета вещей, устройство IoT и устройство IoT Edge.


## <a name="create-log-analytics-workspace"></a>Создание рабочей области Log Analytics

Чтобы создать рабочую область log Analytics, выполните следующие действия. Рабочая область log Analytics — это логическая единица хранения, в которой собираются и сохраняются данные журнала.

1. В портал Azure выберите **+ создать ресурс** и выполните поиск по **log Analytics рабочей области** , а затем щелкните **создать**. 
1. В **рабочей области создание log Analytics** настройте следующие параметры. Примите остаток в качестве значения по умолчанию.

    1. На вкладке **Основные сведения** укажите подписку, группу ресурсов, имя и регион для рабочей области. 

        ![Вкладка "Основные сведения" для рабочей области Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-basics-1.png)  

    1. На вкладке **ценовая категория** примите **план оплаты по мере** использования по умолчанию.

        ![Вкладка "цены" для рабочей области Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-pricing-1.png) 

    1. На вкладке " **Проверка и создание** " Проверьте сведения о рабочей области и нажмите кнопку " **создать**".

        ![Обзор и создание для Log Analytics рабочей области](media/azure-stack-edge-gpu-enable-azure-monitor/create-log-analytics-workspace-review-create-1.png)

Дополнительные сведения см. в статье [создание log Analytics рабочей области с помощью портал Azure](../azure-monitor/logs/quick-create-workspace.md).



## <a name="enable-container-insights"></a>Включить аналитику контейнера

Чтобы включить аналитику контейнера в рабочей области, выполните следующие действия. 

1. Выполните подробные инструкции по [добавлению решения Azure Monitor Containers](../azure-monitor/containers/container-insights-hybrid-setup.md#how-to-add-the-azure-monitor-containers-solution). Используйте следующий файл шаблона `containerSolution.json` :

    ```yml
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace Resource ID"
            }
        },
        "workspaceRegion": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace region"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
            "apiVersion": "2017-05-10",
            "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
            "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "2015-11-01-preview",
                            "type": "Microsoft.OperationsManagement/solutions",
                            "location": "[parameters('workspaceRegion')]",
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "properties": {
                                "workspaceResourceId": "[parameters('workspaceResourceId')]"
                            },
                            "plan": {
                                "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                                "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                                "promotionCode": "",
                                "publisher": "Microsoft"
                            }
                        }
                    ]
                },
                "parameters": {}
            }
            }
        ]
    }
    ```

1. Получите идентификатор и расположение ресурса. Перейдите к `Your Log Analytics workspace > General > Properties`. Скопируйте следующие сведения:

    - **идентификатор ресурса**, который является полным идентификатором ресурса Azure log Analytics рабочей области Azure. 
    - **Расположение**— регион Azure.

    ![Свойства рабочей области Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-properties-1.png) 

1. Используйте следующий файл параметров `containerSolutionParams.json` . Замените `workspaceResourceId` идентификатором ресурса и `workspaceRegion` расположением, скопированным на предыдущем шаге.

    ```yaml
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        "workspaceResourceId": {
            "value": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourcegroups/myaserg/providers/microsoft.operationalinsights/workspaces/myaseloganalyticsws"
        },
        "workspaceRegion": {
        "value": "westus"
        }
        }
    }
    ```
    
    Ниже приведен пример выходных данных Log Analytics рабочей области с включенной аналитикой контейнеров:
    
    ```powershell
    Requesting a Cloud Shell.Succeeded.
    Connecting terminal...
    MOTD: Switch to Bash from PowerShell: bash
    VERBOSE: Authenticating to Azure ...
    VERBOSE: Building your Azure drive ...
    
    PS /home/alpa> az account set -s fa68082f-8ff7-4a25-95c7-ce9da541242f
    PS /home/alpa> ls
    clouddrive  containerSolution.json
    PS /home/alpa> ls
    clouddrive  containerSolution.json  containerSolutionParams.json
    PS /home/alpa> az deployment group create --resource-group myaserg --name Testdeployment1 --template-file containerSolution.json --parameters containerSolutionParams.json
    {- Finished ..
        "id": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourceGroups/myaserg/providers/Microsoft.Resources/deployments/Testdeployment1",
        "location": null,
        "name": "Testdeployment1",
        "properties": {
        "correlationId": "3a9045fe-2de0-428c-b17b-057508a8c575",
        "debugSetting": null,
        "dependencies": [],
        "duration": "PT11.1588316S",
        "error": null,
        "mode": "Incremental",
        "onErrorDeployment": null,
        "outputResources": [
            {
            "id": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourceGroups/myaserg/providers/Microsoft.OperationsManagement/solutions/ContainerInsights(myaseloganalyticsws)",
            "resourceGroup": "myaserg"
            }
        ],
        "outputs": null,
        "parameters": {
            "workspaceRegion": {
            "type": "String",
            "value": "westus"
            },
            "workspaceResourceId": {
            "type": "String",
            "value": "/subscriptions/fa68082f-8ff7-4a25-95c7-ce9da541242f/resourcegroups/myaserg/providers/microsoft.operationalinsights/workspaces/myaseloganalyticsws"
            }
        },
        "parametersLink": null,
        "providers": [
            {
            "id": null,
            "namespace": "Microsoft.Resources",
            "registrationPolicy": null,
            "registrationState": null,
            "resourceTypes": [
                {
                "aliases": null,
                "apiProfiles": null,
                "apiVersions": null,
                "capabilities": null,
                "defaultApiVersion": null,
                "locations": [
                    null
                ],
                "properties": null,
                "resourceType": "deployments"
                }
            ]
            }
        ],
        "provisioningState": "Succeeded",
        "templateHash": "10500027184662969395",
        "templateLink": null,
        "timestamp": "2020-11-06T22:09:56.908983+00:00",
        "validatedResources": null
        },
        "resourceGroup": "myaserg",
        "tags": null,
        "type": "Microsoft.Resources/deployments"
    }
    PS /home/alpa>
    ```

## <a name="configure-azure-monitor-on-your-device"></a>Настройка Azure Monitor на устройстве

1. Перейдите к созданному ресурсу Log Analytics и скопируйте **идентификатор рабочей области** и **первичный ключ** (ключ рабочей области).

    ![Управление агентами в рабочей области Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-agents-management-1.png)

    Сохраните эти сведения, так как они будут использоваться на более позднем этапе.

1. [Подключитесь к интерфейсу PowerShell устройства](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface). 
    
1. Используйте идентификатор рабочей области log Analytics и ключ рабочей области с помощью следующего командлета:

    `Set-HcsKubernetesAzureMonitorConfiguration -WorkspaceId <> -WorkspaceKey <>`

1. После включения Azure Monitor Вы увидите журналы в рабочей области Log Analytics. Чтобы просмотреть состояние кластера Kubernetes, развернутого на устройстве, перейдите в раздел **Azure Monitor > insights > контейнеры**. Для параметра среда выберите **все**. 

    ![Метрики в рабочей области Log Analytics](media/azure-stack-edge-gpu-enable-azure-monitor/log-analytics-workspace-metrics-1.png)

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [отслеживать рабочие нагрузки Kubernetes с помощью панели мониторинга Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).
- Узнайте, как [управлять уведомлениями о событиях устройств](azure-stack-edge-gpu-manage-device-event-alert-notifications.md). 
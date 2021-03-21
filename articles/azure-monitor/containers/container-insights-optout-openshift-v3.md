---
title: Как отключить мониторинг кластера Azure Red Hat OpenShift v3 | Документация Майкрософт
description: В этой статье описывается, как можно отключить мониторинг кластера Azure Red Hat OpenShift с помощью Container Insights.
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: 6a278898942e40d3347791d31a3c38341df01f7c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101731822"
---
# <a name="how-to-stop-monitoring-your-azure-red-hat-openshift-v3-cluster"></a>Как отключить мониторинг кластера Azure Red Hat OpenShift v3

>[!IMPORTANT]
> Azure Red Hat OpenShift 3,11 будет снят с учета по 2022 июня.
>
> По состоянию на Октябрь 2020 вы больше не сможете создавать новые кластеры 3,11.
> Существующие кластеры 3,11 продолжат работать до 2022 июня, но больше не будут поддерживаться после этой даты.
>
> Следуйте указаниям этого руководством, чтобы [создать кластер Azure Red Hat OpenShift 4](../../openshift/tutorial-create-cluster.md).
> Если у вас есть определенные вопросы, [свяжитесь с нами](mailto:aro-feedback@microsoft.com).

После включения мониторинга кластера Azure Red Hat OpenShift версии 3. x вы можете отключить мониторинг кластера с помощью Container Insights, если вы решите, что вы больше не хотите его отслеживать. В этой статье показано, как это сделать с помощью предоставленного шаблона Azure Resource Manager.  

## <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

Предоставлены два шаблона Azure Resource Manager, которые позволяют последовательно и многократно удалять ресурсы решения в вашей группе ресурсов. Один из них — шаблон JSON, указывающий конфигурацию для отключения мониторинга, а другая содержит значения параметров, которые вы настраиваете для указания идентификатора ресурса кластера OpenShift и региона Azure, в котором развернут кластер.

Если вы не знакомы с концепцией развертывания ресурсов с помощью шаблона, ознакомьтесь со статьями:
* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)
* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../../azure-resource-manager/templates/deploy-cli.md)

Если вы решили использовать Azure CLI, необходимо сначала установить интерфейс командной строки и использовать его локально. Необходимо запустить Azure CLI версии 2.0.65 или более поздней. Для определения версии выполните `az --version`. Если вам необходимо установить или обновить Azure CLI, ознакомьтесь со статьей [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

### <a name="create-template"></a>Создание шаблона

1. Скопируйте и вставьте в него следующий синтаксис JSON:

    ```json
    {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
         "aroResourceId": {
           "type": "string",
           "metadata": {
             "description": "ARO Cluster Resource ID"
          }
        },
        "aroResourceLocation": {
          "type": "string",
          "metadata": {
            "description": "Location of the aro cluster resource e.g. westcentralus"
          }
        }
      },
      "resources": [
        {
           "name": "[split(parameters('aroResourceId'),'/')[8]]",
           "type": "Microsoft.ContainerService/openShiftManagedClusters",
           "location": "[parameters('aroResourceLocation')]",
           "apiVersion": "2019-09-30-preview",
           "properties": {
             "mode": "Incremental",
             "id": "[parameters('aroResourceId')]",
             "monitorProfile": {
               "workspaceResourceID": null,
               "enabled": false
            }
          }
        }
      ]
    }
    ```

2. Сохраните этот файл как **OptOutTemplate.json** в локальной папке.

3. Вставьте в него следующий синтаксис JSON:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aroResourceId": {
          "value": "/subscriptions/<subscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.ContainerService/openShiftManagedClusters/<clusterName>"
        },
        "aroResourceLocation": {
          "value": "<azure region of the cluster e.g. westcentralus>"
        }
      }
    }
    ```

4. Измените значения **ароресаурцеид** и **ароресаурцелокатион** , используя значения кластера OpenShift, которые можно найти на странице **свойств** выбранного кластера.

    ![Страница свойств контейнера](media/container-insights-optout-openshift/cluster-properties-page.png)

5. Сохраните этот файл как **OptOutParam.json** в локальной папке.

6. Теперь вы можете развернуть этот шаблон.

### <a name="remove-the-solution-using-azure-cli"></a>Удаление решения с помощью Azure CLI

Выполните следующую команду с Azure CLI в Linux, чтобы удалить решение и очистить конфигурацию в кластере.

```azurecli
az login   
az account set --subscription "Subscription Name"
az deployment group create --resource-group <ResourceGroupName> --template-file ./OptOutTemplate.json --parameters @./OptOutParam.json  
```

Изменение конфигурации может занять несколько минут. По завершении выполнения вы увидите сообщение, подобное приведенному ниже, с возвращенным результатом:

```output
ProvisioningState       : Succeeded
```

### <a name="remove-the-solution-using-powershell"></a>Удаление решения с помощью PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Выполните следующие команды PowerShell в папке, содержащей шаблон, чтобы удалить решение и очистить конфигурацию из кластера.    

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName <yourSubscriptionName>
New-AzResourceGroupDeployment -Name opt-out -ResourceGroupName <ResourceGroupName> -TemplateFile .\OptOutTemplate.json -TemplateParameterFile .\OptOutParam.json
```

Изменение конфигурации может занять несколько минут. По завершении выполнения вы увидите сообщение, подобное приведенному ниже, с возвращенным результатом:

```output
ProvisioningState       : Succeeded
```

## <a name="next-steps"></a>Дальнейшие действия

Если рабочая область была создана только для мониторинга кластера и больше не требуется, ее необходимо удалить вручную. Если вы не знакомы с удалением рабочей области, см. статью [удаление log Analytics рабочей области Azure](../logs/delete-workspace.md).
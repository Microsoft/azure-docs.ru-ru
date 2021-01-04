---
title: Как остановить мониторинг кластера Службы Azure Kubernetes | Документация Майкрософт
description: В этой статье описывается, как остановить мониторинг кластера Azure AKS с помощью Azure Monitor для контейнеров.
ms.topic: conceptual
ms.date: 08/19/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 99c4ecb6c8b77f1576f25816ba486ed262249c32
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695710"
---
# <a name="how-to-stop-monitoring-your-azure-kubernetes-service-aks-with-azure-monitor-for-containers"></a>Как остановить мониторинг кластера Службы Azure Kubernetes (AKS) с помощью Azure Monitor для контейнеров

Если после включения мониторинга вашего кластера AKS вы решите, что больше не хотите его отслеживать, вы можете остановить мониторинг. В этой статье показано, как это сделать с помощью Azure CLI или предоставленных шаблонов Azure Resource Manager.  


## <a name="azure-cli"></a>Azure CLI

Используйте команду [​az aks disable-addons](/cli/azure/aks?view=azure-cli-latest#az-aks-disable-addons), чтобы отключить Azure Monitor для контейнеров. Команда удаляет агент из узлов кластера, не удаляет решение или данные, которые уже собраны и хранятся в ресурсе Azure Monitor.  

```azurecli
az aks disable-addons -a monitoring -n MyExistingManagedCluster -g MyExistingManagedClusterRG
```

Сведения о повторном включении мониторинга кластера см. в разделе [Включение мониторинга с помощью Azure CLI](container-insights-enable-new-cluster.md#enable-using-azure-cli).

## <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

Предоставлены два шаблона Azure Resource Manager, которые позволяют последовательно и многократно удалять ресурсы решения в вашей группе ресурсов. Один шаблон JSON задает конфигурацию прекращения мониторинга, а другой содержит значения параметров, которые можно настроить, чтобы указать идентификатор ресурса кластера AKS и группу ресурсов, в которой он развернут.

Если вы не знакомы с концепцией развертывания ресурсов с помощью шаблона, ознакомьтесь со статьями:
* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)
* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../../azure-resource-manager/templates/deploy-cli.md)

>[!NOTE]
>Шаблон необходимо развернуть в той же группе ресурсов кластера. Если вы опустите какие-либо другие свойства или надстройки при использовании этого шаблона, это может привести к удалению их из кластера. Например, *енаблербак* для политик RBAC Kubernetes, реализованных в кластере, или *аксресаурцетагвалуес* , если для кластера AKS указаны теги.  
>

Если вы решили использовать Azure CLI, необходимо сначала установить интерфейс командной строки и использовать его локально. Необходимо запустить Azure CLI версии 2.0.27 или более поздней. Для определения версии выполните `az --version`. Если вам необходимо установить или обновить Azure CLI, ознакомьтесь со статьей [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

### <a name="create-template"></a>Создание шаблона

1. Скопируйте и вставьте в него следующий синтаксис JSON:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aksResourceId": {
           "type": "string",
           "metadata": {
             "description": "AKS Cluster Resource ID"
           }
       },
      "aksResourceLocation": {
        "type": "string",
        "metadata": {
           "description": "Location of the AKS resource e.g. \"East US\""
         }
       },
    "aksResourceTagValues": {
      "type": "object",
      "metadata": {
        "description": "Existing all tags on AKS Cluster Resource"
        }
      }
     },
    "resources": [
      {
        "name": "[split(parameters('aksResourceId'),'/')[8]]",
        "type": "Microsoft.ContainerService/managedClusters",
        "location": "[parameters('aksResourceLocation')]",
        "tags": "[parameters('aksResourceTagValues')]",
        "apiVersion": "2018-03-31",
        "properties": {
          "mode": "Incremental",
          "id": "[parameters('aksResourceId')]",
          "addonProfiles": {
            "omsagent": {
              "enabled": false,
              "config": null
            }
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
        "aksResourceId": {
          "value": "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroup>/providers/Microsoft.ContainerService/managedClusters/<ResourceName>"
        },
        "aksResourceLocation": {
          "value": "<aksClusterRegion>"
        },
        "aksResourceTagValues": {
          "value": {
            "<existing-tag-name1>": "<existing-tag-value1>",
            "<existing-tag-name2>": "<existing-tag-value2>",
            "<existing-tag-nameN>": "<existing-tag-valueN>"
          }
        }
      }
    }
    ```

4. Замените значения **aksResourceId** и **aksResourceLocation** значениями кластера AKS, которые можно найти на странице **свойств** выбранного кластера.

    ![Страница свойств контейнера](media/container-insights-optout/container-properties-page.png)

    На странице **Свойства** также скопируйте **идентификатор ресурса рабочей области**. Это значение потребуется, если позже вы решите удалить рабочую область Log Analytics, что выходит за рамки этого процесса.

    Измените значения для **аксресаурцетагвалуес** , чтобы они совпадали с существующими значениями тегов, указанными для кластера AKS.

5. Сохраните этот файл как **OptOutParam.json** в локальной папке.

6. Теперь вы можете развернуть этот шаблон.

### <a name="remove-the-solution-using-azure-cli"></a>Удаление решения с помощью Azure CLI

Выполните следующую команду с помощью Azure CLI в Linux, чтобы удалить решение и очистить конфигурацию в кластере AKS.

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

Выполните следующие команды PowerShell в папке с шаблоном, чтобы удалить решение и очистить конфигурацию в кластере AKS.    

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

Если рабочая область была создана только для мониторинга кластера и больше не требуется, ее необходимо удалить вручную. Если вам неизвестно, как удалить рабочую область, прочитайте раздел [Удаление рабочей области Azure Log Analytics на портале Azure](../platform/delete-workspace.md). Не забывайте о том, что **идентификатор ресурса рабочей области** , скопированный ранее на шаге 4, вам понадобится.

---
title: Журнал развертывания
description: Сведения о просмотре операций развертывания Azure Resource Manager с помощью портала, PowerShell, Azure CLI и REST API.
tags: top-support-issue
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 00e0c51244d5c191d4c9f05f689b90ece81ec5a4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91284624"
---
# <a name="view-deployment-history-with-azure-resource-manager"></a>Просмотр журнала развертываний с помощью Azure Resource Manager

Azure Resource Manager позволяет просматривать журнал развертывания. Вы можете изучить определенные операции в прошлых развертываниях и просмотреть данные о развернутых ресурсах. Этот журнал содержит сведения об ошибках.

Журнал развертывания для группы ресурсов ограничен 800 развертываниями. Как только вы приближаетесь к ограничению, развертывания автоматически удаляются из журнала. См. статью [Автоматическое удаление из журнала развертывания](deployment-history-deletions.md).

Сведения об устранении некоторых ошибок развертывания см. в статье об [устранении распространенных ошибок при развертывании ресурсов в Azure с помощью Azure Resource Manager](common-deployment-errors.md).

## <a name="get-deployments-and-correlation-id"></a>Получение сведений о развертываниях и идентификатора корреляции

Сведения о развертывании можно получить с помощью портала Azure, PowerShell, Azure CLI или REST API. Каждое развертывание имеет идентификатор корреляции, который используется для отслеживания связанных событий. При [создании запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md)служба поддержки может запросить идентификатор корреляции. Поддержка использует идентификатор корреляции для идентификации операций для неудачного развертывания.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Выберите группу ресурсов, которую нужно изучить.

1. Щелкните ссылку в разделе **Развертывания**.

   ![Выбор журнала развертывания](./media/deployment-history/select-deployment-history.png)

1. Выберите нужное развертывание из журнала развертывания.

   ![Выбор развертывания](./media/deployment-history/select-details.png)

1. Отобразится сводка по развертыванию, где есть и идентификатор корреляции.

    ![Сводка по развертыванию](./media/deployment-history/show-correlation-id.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Список всех развертываний для группы ресурсов можно получить с помощью команды [Get-AzResourceGroupDeployment](/powershell/module/az.resources/Get-AzResourceGroupDeployment).

```azurepowershell-interactive
Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup
```

Чтобы получить сведения о конкретном развертывании из группы ресурсов, добавьте параметр **DeploymentName**.

```azurepowershell-interactive
Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName ExampleDeployment
```

Чтобы получить идентификатор корреляции, используйте:

```azurepowershell-interactive
(Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName ExampleDeployment).CorrelationId
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Список всех развертываний для группы ресурсов можно получить с помощью команды [az deployment group list](/cli/azure/group/deployment#az-deployment-group-list).

```azurecli-interactive
az deployment group list --resource-group ExampleGroup
```

Чтобы получить сведения о конкретном развертывании, используйте [az deployment group show](/cli/azure/group/deployment#az-deployment-group-show).

```azurecli-interactive
az deployment group show --resource-group ExampleGroup --name ExampleDeployment
```

Чтобы получить идентификатор корреляции, используйте:

```azurecli-interactive
az deployment group show --resource-group ExampleGroup --name ExampleDeployment --query properties.correlationId
```

# <a name="http"></a>[HTTP](#tab/http)

Список всех развертываний для группы ресурсов можно получить с помощью следующей операции. Номер последней версии API, который нужно указать в запросе, можно узнать в статье [Развертывания: вывод списка по группе ресурсов](/rest/api/resources/deployments/listbyresourcegroup).

```
GET https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/?api-version={api-version}
```

Чтобы получить сведения о конкретном развертывании, выполните следующую операцию. Номер последней версии API, который нужно указать в запросе, можно узнать в статье [Развертывания: получение](/rest/api/resources/deployments/get).

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
```

В ответе будет указан идентификатор корреляции.

```json
{
 ...
 "properties": {
   "mode": "Incremental",
   "provisioningState": "Failed",
   "timestamp": "2019-11-26T14:18:36.4518358Z",
   "duration": "PT26.2091817S",
   "correlationId": "47ff4228-bf2e-4ee5-a008-0b07da681230",
   ...
 }
}
```

---

## <a name="get-deployment-operations-and-error-message"></a>Получение сведений об операциях развертывания и сообщений об ошибке

Каждое развертывание может включать несколько операций. Чтобы больше узнать о развертывании, изучите его операции. При сбое развертывания в его операции будут включено сообщение об ошибке.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. На странице сводных данных по развертыванию выберите **Сведения об операции**.

    ![Выберите сведения об операции](./media/deployment-history/get-operation-details.png)

1. Вы увидите подробные сведения об этом шаге развертывания. Если возникла ошибка, сведения о ней будут включены в это сообщение.

    ![Просмотр сведений об операции](./media/deployment-history/see-operation-details.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы просмотреть сведения об операциях развертывания в группе ресурсов, используйте команду [Get-AzResourceGroupDeploymentOperation](/powershell/module/az.resources/get-azdeploymentoperation).

```azurepowershell-interactive
Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName ExampleDeploy
```

Для просмотра сведений о невыполненных операциях примените фильтр по состоянию **Failed** (Сбой).

```azurepowershell-interactive
(Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName ExampleDeploy).Properties | Where-Object ProvisioningState -eq Failed
```

Чтобы получить сообщение о состоянии операций, завершившихся сбоем, используйте следующую команду:

```azurepowershell-interactive
((Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName ExampleDeploy ).Properties | Where-Object ProvisioningState -eq Failed).StatusMessage.error
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы просмотреть сведения об операциях для развертывания в группе ресурсов, используйте команду [az deployment operation group list](/cli/azure/deployment/operation/group#az-deployment-operation-group-list). Для этого нужно иметь Azure CLI 2.6.0 или более поздней версии.

```azurecli-interactive
az deployment operation group list --resource-group ExampleGroup --name ExampleDeployment
```

Для просмотра сведений о невыполненных операциях примените фильтр по состоянию **Failed** (Сбой).

```azurecli-interactive
az deployment operation group list --resource-group ExampleGroup --name ExampleDeploy --query "[?properties.provisioningState=='Failed']"
```

Чтобы получить сообщение о состоянии операций, завершившихся сбоем, используйте следующую команду:

```azurecli-interactive
az deployment operation group list --resource-group ExampleGroup --name ExampleDeploy --query "[?properties.provisioningState=='Failed'].properties.statusMessage.error"
```

# <a name="http"></a>[HTTP](#tab/http)

Для получения сведений об операциях развертывания выполните приведенную ниже операцию. Номер последней версии API, который нужно указать в запросе, можно узнать в статье [Операции развертывания: список](/rest/api/resources/deploymentoperations/list).

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}
```

Ответ содержит сообщение об ошибке.

```json
{
  "value": [
    {
      "id": "/subscriptions/xxxx/resourceGroups/examplegroup/providers/Microsoft.Resources/deployments/exampledeploy/operations/13EFD9907103D640",
      "operationId": "13EFD9907103D640",
      "properties": {
        "provisioningOperation": "Create",
        "provisioningState": "Failed",
        "timestamp": "2019-11-26T14:18:36.3177613Z",
        "duration": "PT21.0580179S",
        "trackingId": "9d3cdac4-54f8-486c-94bd-10c20867b8bc",
        "serviceRequestId": "01a9d0fe-896b-4c94-a30f-60b70a8f1ad9",
        "statusCode": "BadRequest",
        "statusMessage": {
          "error": {
            "code": "InvalidAccountType",
            "message": "The AccountType Standard_LRS1 is invalid. For more information, see - https://aka.ms/storageaccountskus"
          }
        },
        "targetResource": {
          "id": "/subscriptions/xxxx/resourceGroups/examplegroup/providers/Microsoft.Storage/storageAccounts/storageq2czadzfgizc2",
          "resourceType": "Microsoft.Storage/storageAccounts",
          "resourceName": "storageq2czadzfgizc2"
        }
      }
    },
    ...
  ]
}
```

---

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об устранении некоторых ошибок развертывания см. в статье об [устранении распространенных ошибок при развертывании ресурсов в Azure с помощью Azure Resource Manager](common-deployment-errors.md).
* Сведения о том, как управление развертываниями осуществляется в журнале, см. в разделе [Автоматическое удаление из журнала развертывания](deployment-history-deletions.md).
* Чтобы проверить развернутую службу перед ее выполнением, ознакомьтесь со статьей [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](deploy-powershell.md).


---
title: Масштабирование управляемого кластера Service Fabric (предварительная версия)
description: В этом учебнике описано, как масштабировать тип узла управляемого кластера Service Fabric.
ms.topic: tutorial
ms.date: 09/28/2020
ms.openlocfilehash: 769bcb339b2cc1419c7a3d92d0f08130029a9f95
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98785436"
---
# <a name="tutorial-scale-out-a-service-fabric-managed-cluster-preview"></a>Руководство по масштабированию управляемого кластера Service Fabric (предварительная версия)

В этой серии учебников мы обсудим следующие темы:

> [!div class="checklist"]
> * [Развертывание управляемого кластера Service Fabric.](tutorial-managed-cluster-deploy.md)
> * Масштабирование управляемого кластера Service Fabric.
> * [Добавление и удаление типов узлов в управляемом кластере Service Fabric](tutorial-managed-cluster-add-remove-node-type.md).
> * [Развертывание приложения в управляемом кластере Service Fabric](tutorial-managed-cluster-deploy-app.md).

В этой части серии учебников описывается:

> [!div class="checklist"]
> * Масштабирование узла управляемого кластера Service Fabric.

## <a name="prerequisites"></a>Предварительные требования

* [Управляемый кластер Service Fabric (см. статью *Развертывание управляемого кластера*](tutorial-managed-cluster-deploy.md)).
* [Azure PowerShell версии 4.7.0](/powershell/azure/release-notes-azureps#azservicefabric) или более поздней (см. статью [*Установка Azure PowerShell*](/powershell/azure/install-az-ps)).

## <a name="scale-a-service-fabric-managed-cluster"></a>Масштабирование управляемого кластера Service Fabric
Измените число экземпляров, чтобы увеличить или уменьшить количество узлов в типе узла, который необходимо масштабировать. Имена типов узлов можно найти в шаблоне Azure Resource Manager (шаблон ARM) из развертывания кластера или в Service Fabric Explorer.  

> [!NOTE]
> Если тип узла является основным, вы не сможете использовать следующие 3 узла для кластера SKU "Базовый" и 5 узлов для кластера SKU "Стандартный".

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "mysfcluster"
$nodeTypeName = "FE"
$instanceCount = "7"

Set-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -name $nodeTypeName -InstanceCount $instanceCount -Verbose
```

Кластер начнет обновляться автоматически, и через несколько минут вы увидите дополнительные узлы.

## <a name="next-steps"></a>Дальнейшие действия

На этом шаге мы узнали, как масштабировать тип узла в управляемом кластере Service Fabric. Дополнительные сведения о добавлении и удалении типов узлов см. в статье

> [!div class="nextstepaction"]
> [Добавление и удаление типов узлов в управляемом кластере Service Fabric](tutorial-managed-cluster-add-remove-node-type.md).

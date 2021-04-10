---
title: Добавление и удаление типов узлов в управляемом кластере Service Fabric (предварительная версия)
description: В этом учебнике описано, как добавлять и удалять типы узлов управляемого кластера Service Fabric.
ms.topic: tutorial
ms.date: 09/28/2020
ms.openlocfilehash: bb33512652677fc4e46d8ba3668dca985bbcfe01
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98791245"
---
# <a name="tutorial-add-and-remove-node-types-from-a-service-fabric-managed-cluster-preview"></a>Руководство по добавлению и удалению типов узлов в управляемом кластере Service Fabric (предварительная версия)

В этой серии учебников мы обсудим следующие темы:

> [!div class="checklist"]
> * [Развертывание управляемого кластера Service Fabric](tutorial-managed-cluster-deploy.md).
> * [Масштабирование управляемого кластера Service Fabric](tutorial-managed-cluster-scale.md).
> * Добавление и удаление узлов в управляемом кластере Service Fabric.
> * [Развертывание приложения в управляемом кластере Service Fabric](tutorial-managed-cluster-deploy-app.md).

В этой части серии учебников описывается:

> [!div class="checklist"]
> * Добавление типа узла в управляемый кластер Service Fabric.
> * Удаление типа узла из управляемого кластера Service Fabric.

## <a name="prerequisites"></a>Предварительные требования

* [Управляемый кластер Service Fabric (см. статью *Развертывание управляемого кластера*](tutorial-managed-cluster-deploy.md)).
* [Azure PowerShell версии 4.7.0](/powershell/azure/release-notes-azureps#azservicefabric) или более поздней (см. статью [*Установка Azure PowerShell*](/powershell/azure/install-az-ps)).

## <a name="add-a-node-type-to-a-service-fabric-managed-cluster"></a>Добавление типа узла в управляемый кластер Service Fabric

Вы можете добавить тип узла в управляемый кластер Service Fabric с помощью шаблона Azure Resource Manager, PowerShell или CLI. Проходя этот учебник, мы добавим тип узла с помощью Azure PowerShell.

Чтобы создать новый тип узла, необходимо определить три свойства:

* **Имя типа узла**. Имя, уникальное для всех существующих типов узлов в кластере.
* **Число экземпляров**. Начальное число узлов нового типа узла.
* **Размер виртуальной машины**. Номер SKU виртуальной машины для узлов. Если значение не задано, по умолчанию используется значение *Standard_D2*.

> [!NOTE]
> Если добавляемый тип узла является первым или единственным типом узла в кластере, необходимо использовать свойство "Первичный".

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "mysfcluster"
$nodeTypeName = "nt2"
$vmSize = "Standard_D2_v2"

New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeTypeName -InstanceCount 3 -vmSize $vmSize
```

## <a name="remove-a-node-type-from-a-service-fabric-managed-cluster"></a>Удаление типа узла из управляемого кластера Service Fabric

Чтобы удалить тип узла из управляемого кластера Service Fabric, необходимо использовать PowerShell или CLI. Проходя этот учебник, мы научимся удалять тип узла с помощью Azure PowerShell.

> [!NOTE]
> Вы не сможете удалить тип первичного узла, если он является единственным типом первичного узла в кластере.  

Чтобы удалить тип узла:

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "myCluster"
$nodeTypeName = "nt2"

Remove-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName  -Name $nodeTypeName
```

## <a name="next-steps"></a>Дальнейшие действия

 Выполняя действия, указанные в этом разделе, мы добавили и удалили типы узлов. Дополнительные сведения см. в статье:

> [!div class="nextstepaction"]
> [Развертывание приложения в управляемом кластере Service Fabric](tutorial-managed-cluster-deploy-app.md)

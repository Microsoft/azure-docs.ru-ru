---
title: Руководство. Удаление приложения, работающего в Сетке Azure Service Fabric
description: Из этого руководства вы узнаете, как удалить приложение, работающее в Сетке Service Fabric, вместе с ресурсами.
author: georgewallace
ms.topic: tutorial
ms.date: 01/11/2019
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: d449343fd00ff958470a71ecb3a37d585d7ff8ed
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99626616"
---
# <a name="tutorial-remove-an-application-and-resources"></a>Руководство. Удаление приложения и ресурсов

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Это руководство представляет собой четвертую часть цикла. Вы узнаете, как удалить работающее приложение, которое было [ранее развернуто в Сетке Service Fabric](service-fabric-mesh-tutorial-template-deploy-app.md). 

В четвертой части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Удаление приложения, работающего в Сетке Service Fabric
> * Удаление ресурсов приложения

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * [Развертывание приложения в Сетке Service Fabric с помощью шаблона](service-fabric-mesh-tutorial-template-deploy-app.md)
> * [Масштабирование приложения, работающего в Сетке Service Fabric](service-fabric-mesh-tutorial-template-scale-services.md)
> * [Обновление приложения, работающего в Сетке Service Fabric](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * Удаление приложения

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет Azure подписки до начала работы, можно [создать бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Откройте [Azure Cloud Shell](service-fabric-mesh-howto-setup-cli.md) или [установите Azure CLI и интерфейс командной строки Сетки Azure Service Fabric на локальный компьютер](service-fabric-mesh-howto-setup-cli.md#install-the-azure-service-fabric-mesh-cli).

## <a name="delete-the-resource-group-and-all-the-resources"></a>Удаление группы ресурсов и ее содержимого

Удалите все созданные ресурсы, если они больше не нужны. Ранее вы [создали группу ресурсов](service-fabric-mesh-tutorial-template-deploy-app.md#create-a-container-registry) для размещения экземпляра реестра контейнеров Azure и ресурсов приложения Сетки Service Fabric.  Удалив эту группу ресурсов, вы удалите все связанные с нею ресурсы.

```azurecli
az group delete --resource-group myResourceGroup
```

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="individually-delete-the-resources"></a>Удаление ресурсов по отдельности
Экземпляр ACR, приложение Сетки Service Fabric и сетевые ресурсы можно удалить по отдельности.

Удаление экземпляра ACR:

```azurecli
az acr delete --resource-group myResourceGroup --name myContainerRegistry
```

Удаление приложения Сетки Service Fabric:

```azurecli
az mesh app delete --resource-group myResourceGroup --name todolistapp
```

Удаление сети:
```azurecli
az mesh network delete --resource-group myResourceGroup --name todolistappNetwork
```

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Удаление приложения, работающего в Сетке Service Fabric
> * Удаление ресурсов приложения

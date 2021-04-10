---
title: Руководство по удалению ресурсов службы "Сетка Azure Service Fabric"
description: Сведения об удалении ресурсов в Сетке Azure Service Fabric таким образом, чтобы с вас не взималась плата за неиспользуемые ресурсы.
author: georgewallace
ms.topic: tutorial
ms.date: 09/18/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: 4d78e1e27d70b147bb52dff13675e63b79335d62
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99625470"
---
# <a name="tutorial-remove-azure-resources"></a>Руководство по удалению ресурсов Azure

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Это руководство является пятой частью серии руководств. Оно содержит сведения об удалении приложения и его ресурсов таким образом, чтобы не оплачивать их.

Из этого руководства вы узнаете, как выполнить следующие задачи:
> [!div class="checklist"]
> * Удаление ресурсов, используемых приложением, таким образом, чтобы за них не взималась плата.

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * [Создание приложения Сетки Service Fabric в Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [Отладка приложения Сетки Service Fabric, выполняющегося в локальном кластере разработки](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [Развертывание приложения Сетки Service Fabric](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * [Обновление приложения Сетки Service Fabric](service-fabric-mesh-tutorial-upgrade.md)
> * Удаление ресурсов Сетки Service Fabric

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если вы еще не развернули приложение списка дел, следуйте инструкциям в статье [Руководство. Развертывание приложения Сетки Service Fabric](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Это последняя часть руководства. Закончив работу с созданными ресурсами, удалите их, чтобы с вас не взималась плата за неиспользуемые ресурсы. Это очень важно, так как Сетка является бессерверной службой, плата за которую взимается на ежесекундной основе. Дополнительные сведения о ценах на Сетку см. на странице цен: https://aka.ms/sfmeshpricing.

Одним из удобств, предоставляемых Azure, является то, что в случае создания ресурсов, связанных с определенной группой ресурсов, при удалении этой группы удаляются и все связанные с ней ресурсы. Таким образом, нет необходимости удалять их по очереди.

Так как созданная группа ресурсов предназначена для создания приложения списка задач, можно безопасно удалить эту группу ресурсов, что приведет к удалению всех связанных ресурсов.

```azurecli
az group delete --resource-group sfmeshTutorial1RG
```

```powershell
Remove-AzureRmResourceGroup -Name sfmeshTutorial1RG
```

Кроме того, можно удалить группу ресурсов **sfmeshTutorial1RG**[с портала](../azure-resource-manager/management/manage-resource-groups-portal.md#delete-resource-groups). 

## <a name="next-steps"></a>Дальнейшие действия

После завершения публикации приложения Сетки Service Fabric в Azure попробуйте следующее.

* Чтобы увидеть еще один пример связи между службами, изучите [пример приложения для голосования](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/votingapp).
* Узнайте больше о модели ресурсов Service Fabric из раздела [Introduction to Service Fabric Resource Model](service-fabric-mesh-service-fabric-resources.md) (Общие сведения о модели ресурсов Service Fabric).
* Узнайте больше о службе "Сетка Service Fabric" из раздела [Что такое Сетка Service Fabric?](service-fabric-mesh-overview.md)
* Ознакомьтесь с [Обзором Azure Cloud Shell](../cloud-shell/overview.md)

---
title: Добавление виртуальной машины с помощью общего образа в Azure DevTest Labs | Документация Майкрософт
description: Узнайте, как добавить виртуальную машину с помощью образа из коллекции Shared Image Gallery в Azure DevTest Labs
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 9421a1e21be9446b0e59328bd9a3730b57655274
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85483862"
---
# <a name="add-a-vm-using-an-image-from-the-attached-shared-image-gallery"></a>Добавление виртуальной машины с помощью образа из коллекции подключенных общих образов
Azure DevTest Labs позволяет подключить коллекцию общих образов к лаборатории, а затем использовать образы в галерее в качестве базовых для виртуальных машин, создаваемых в лаборатории. Сведения о присоединении коллекции общих образов к лаборатории см. в разделе [Настройка коллекции общих образов](configure-shared-image-gallery.md). В этой статье показано, как добавить виртуальную машину в лабораторию с помощью образа из коллекции совместно используемых образов в качестве основы. 

## <a name="azure-portal"></a>Портал Azure
В этом разделе вы узнаете, как использовать портал Azure для добавления виртуальной машины в лабораторию на основе образа из коллекции подключенных общих образов. Этот раздел не предоставляет подробных пошаговых инструкций по созданию виртуальной машины с помощью портал Azure. Дополнительные сведения см. [в разделе Создание виртуальной машины портал Azure](devtest-lab-add-vm.md). В нем показаны только шаги, в которых вы выбрали образ из коллекции подключенных общих образов, и выберите версию образа, которую вы хотите использовать. 

При добавлении виртуальной машины в лабораторию вы можете выбрать образ из присоединенной коллекции общих образов в качестве базового образа: 

![Выберите общий образ для базового](./media/add-vm-use-shared-image/select-shared-image-for-base.png)

Затем на вкладке **Дополнительные параметры** на странице **Создание ресурса лаборатории** можно выбрать версию образа, которую нужно использовать в качестве базового образа:

![Выберите версию образа](./media/add-vm-use-shared-image/select-version-shared-image.png)

После создания виртуальной машины можно переключиться на использование другой версии образа. 

## <a name="resource-manager-template"></a>Шаблон Resource Manager
Если вы используете шаблон Azure Resource Manager для создания виртуальной машины с помощью общего образа коллекции образов, укажите значение для параметра **шаредимажеид** в разделе **свойств** . См. следующий пример. 

```json
"resources": [
{
    ...
    "properties": {
         "sharedImageId": "/subscriptions/111111111-1111-1111-1111-111111111111/resourcegroups/mydtlrg/providers/microsoft.devtestlab/labs/mydtllab/sharedgalleries/spsig/sharedimages/myimagefromgallery",
        "sharedImageVersion": "1.0.1",
        ...
    }
}
],
```

Полный пример шаблона диспетчер ресурсов см. в статье [Создание виртуальной машины с помощью общего образа из коллекции образов](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates/101-dtl-create-vm-username-pwd-sharedimage) в репозитории GitHub. 

## <a name="rest-api"></a>REST API

1. Сначала необходимо получить идентификатор образа в коллекции общих образов. Один из способов — вывести все образы в присоединенной коллекции общих образов с помощью следующей команды GET. 

    ```rest
    GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries/{name}/sharedimages?api-version= 2018-10-15-preview
    ```
2. Вызовите метод размещения на виртуальных машинах, передав идентификатор общего образа, полученного по предыдущему вызову `properties.SharedImageId` .

## <a name="next-steps"></a>Дальнейшие действия
Чтобы узнать, как подключить коллекцию общих образов к лаборатории и настроить ее, см. раздел [Configure Shared Image Gallery](configure-shared-image-gallery.md).
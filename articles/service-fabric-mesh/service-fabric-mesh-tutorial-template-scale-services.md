---
title: Руководство. Масштабирование приложения, работающего в Сетке Azure Service Fabric
description: Из этого руководства вы узнаете, как масштабировать службы в приложении, работающем в Сетке Service Fabric.
author: georgewallace
ms.topic: tutorial
ms.date: 01/11/2019
ms.author: gwallace
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: 02dc5d43a23c572d441da2bbb7386885bf66ece7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99625388"
---
# <a name="tutorial-scale-an-application-running-in-service-fabric-mesh"></a>Руководство по Масштабирование приложения, работающего в Сетке Service Fabric

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Это руководство представляет собой вторую часть цикла. Узнайте, как вручную изменять число экземпляров службы приложения, которое было [развернуто в Сетке Service Fabric](service-fabric-mesh-tutorial-template-deploy-app.md). По завершении у вас будет интерфейсная служба с тремя работающими экземплярами и служба данных с двумя запущенными экземплярами.

Из второй части цикла вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Настройка нужного числа экземпляров службы.
> * Выполнение обновления.

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * [Развертывание приложения в Сетке Service Fabric с помощью шаблона](service-fabric-mesh-tutorial-template-deploy-app.md)
> * Масштабирование приложения, работающего в Сетке Service Fabric
> * [Обновление приложения, работающего в Сетке Service Fabric](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * [Удаление приложения](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет Azure подписки до начала работы, можно [создать бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* [Установите Azure CLI и интерфейса командной строки Сетки Azure Service Fabric локально](service-fabric-mesh-howto-setup-cli.md#install-the-azure-service-fabric-mesh-cli).

## <a name="manually-scale-your-services-in-or-out"></a>Масштабирование служб вручную

Одним из основных преимуществ развертывания приложений в Сетке Service Fabric является возможность легко масштабировать службы. Это можно использовать для обработки изменяющихся объемов нагрузки в службах или для повышения уровня доступности.

В этом руководстве используется пример To Do List, который был [развернут ранее](service-fabric-mesh-tutorial-template-deploy-app.md) и должен быть запущен. Приложение содержит две службы: WebFrontEnd и ToDoService. Каждая служба изначально была развернута с одной репликой.  Чтобы просмотреть количество работающих реплик службы WebFrontEnd, используйте следующую команду.

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp --query "replicaCount"
```

Чтобы просмотреть количество работающих реплик службы ToDoService, используйте следующую команду.

```azurecli
az mesh service show --resource-group myResourceGroup --name ToDoService --app-name todolistapp --query "replicaCount"
```

В шаблоне развертывания ресурса приложения для каждой службы определено свойство *replicaCount*, с помощью которого можно настроить число развертываний этой службы. Приложение может состоять из нескольких служб с уникальными номерами *replicaCount*, развертывание которых и управление которыми осуществляется вместе. Чтобы изменить количество реплик службы, в шаблоне развертывания или файле параметров измените значение *replicaCount* для каждой службы, которую необходимо масштабировать.  Затем обновите приложение.

### <a name="modify-the-deployment-template-parameters"></a>Изменение параметров шаблона развертывания

Если в шаблоне есть значения, которые изменятся после развертывания приложения или которые вы хотели бы изменять для разных развертываний (если планируете повторно использовать этот шаблон), рекомендуется параметризировать эти значения.

Ранее приложение было развернуто с помощью файла [шаблона развертывания mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json) и файла [параметров mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json).

Откройте файл [параметров mesh_rp.windows.parameter.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json) локально и задайте для *frontEndReplicaCount* значение 3, а для *serviceReplicaCount* — значение 2.

```json
      "frontEndReplicaCount":{
        "value": "3"
      },
      "serviceReplicaCount":{
        "value": "2"
      }
```

Сохраните изменения в файле параметров.  Параметры *frontEndReplicaCount* и *serviceReplicaCount* объявлены в разделе *parameters*[шаблона развертывания mesh_rp.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json).

```json
"frontEndReplicaCount":{
      "defaultValue": "1",
      "type": "string"
    },
    "serviceReplicaCount":{
      "defaultValue": "1",
      "type": "string"
    }
```

Свойство *replicaCount* службы WebFrontEnd ссылается на параметр *frontEndReplicaCount*, а свойство *replicaCount* службы ToDoService — на параметр *serviceReplicaCount*.

```json
    "services": [
    {
    "name": "WebFrontEnd",
    "properties": {
        "description": "WebFrontEnd description.",
        "osType": "Windows",
        "codePackages": [
        {
            ...
        }
        ],
        "replicaCount": "[parameters('frontEndReplicaCount')]",
        "networkRefs": [
        {
            "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'todolistappNetwork')]"
        }
        ]
    }
    },
    {
    "name": "ToDoService",
    "properties": {
        "description": "ToDoService description.",
        "osType": "Windows",
        "codePackages": [
        {
            ...
        }
        ],
        "replicaCount": "[parameters('serviceReplicaCount')]",
        "networkRefs": [
        {
            "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'todolistappNetwork')]"
        }
        ]
    }
    }
],
```

После изменения шаблона обновите приложение.

### <a name="upgrade-your-application"></a>Обновление приложения.

Когда приложение запущено, его можно обновить, повторно развернув шаблон и обновленный файл параметров.

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

Будет запущено последовательное обновление приложения, и через несколько минут вы увидите, что экземпляров служб стало больше.  Чтобы просмотреть количество работающих реплик службы WebFrontEnd, используйте следующую команду.

```azurecli
az mesh service show --resource-group myResourceGroup --name WebFrontEnd --app-name todolistapp --query "replicaCount"
```

Чтобы просмотреть количество работающих реплик службы ToDoService, используйте следующую команду.

```azurecli
az mesh service show --resource-group myResourceGroup --name ToDoService --app-name todolistapp --query "replicaCount"
```

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Настройка нужного числа экземпляров службы.
> * Выполнение обновления.

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Обновление приложения, работающего в Сетке Service Fabric](service-fabric-mesh-tutorial-template-upgrade-app.md)

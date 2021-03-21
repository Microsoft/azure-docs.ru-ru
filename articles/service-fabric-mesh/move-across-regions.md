---
title: Перемещение Service Fabric приложения сетки в другой регион
description: Вы можете переместить Service Fabric ресурсы сетки, развернув копию текущего шаблона в новом регионе Azure.
author: erikadoyle
ms.author: edoyle
ms.topic: how-to
ms.date: 01/14/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 1b59d482b8b88e37da2d61636ff3f254a46ba5c2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99626093"
---
# <a name="move-a-service-fabric-mesh-application-to-another-azure-region"></a>Перемещение Service Fabric приложения сетки в другой регион Azure

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

В этой статье описывается, как переместить приложение сетки Service Fabric и его ресурсы в другой регион Azure. Вы можете переместить ресурсы в другой регион по ряду причин. Например, в ответ на сбои, чтобы получить доступ к функциям или службам, доступным только в конкретных регионах, в соответствии с требованиями к внутренней политике и управлению или в соответствии с требованиями к планированию емкости.

 [Service Fabricная сетка не поддерживает](../azure-resource-manager/management/region-move-support.md#microsoftservicefabricmesh) возможность прямого перемещения ресурсов между регионами Azure. Тем не менее ресурсы можно перемещать косвенно, развертывая копию текущего шаблона Azure Resource Manager в новом целевом регионе, а затем перенаправляя входящий трафик и зависимости на только что созданное Service Fabricное приложение сетки.

## <a name="prerequisites"></a>Предварительные условия

* Входной контроллер (например, [шлюз приложений](../application-gateway/index.yml)), служащий посредником для маршрутизации трафика между клиентами и приложением Service Fabricной сетки
* Доступность Service Fabric сети (Предварительная версия) в целевом регионе Azure ( `westus` , `eastus` или `westeurope` )

## <a name="prepare"></a>Подготовка.

1. Сделайте "моментальный снимок" текущего состояния приложения Service Fabricной сетки, экспортировав шаблон и параметры Azure Resource Manager из последнего развертывания. Для этого выполните действия, описанные в разделе [Экспорт шаблона после развертывания](../azure-resource-manager/templates/export-template-portal.md#export-template-after-deployment) с помощью портал Azure. Можно также использовать [Azure CLI](../azure-resource-manager/management/manage-resource-groups-cli.md#export-resource-groups-to-templates), [Azure PowerShell](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates)или [REST API](/rest/api/resources/resourcegroups/exporttemplate).

2. Если это применимо, [экспортируйте другие ресурсы в той же группе ресурсов](../azure-resource-manager/templates/export-template-portal.md#export-template-from-a-resource-group) для повторного развертывания в целевом регионе.

3. Проверьте (и при необходимости измените) экспортированный шаблон, чтобы убедиться, что существующие значения свойств являются теми, которые вы хотите использовать в целевом регионе. Новый `location` (регион Azure) — это параметр, который будет предоставляться при повторном развертывании.

## <a name="move"></a>Переместить

1. Создайте новую группу ресурсов (или используйте существующую) в целевом регионе.

2. После экспорта шаблона выполните действия, описанные в разделе [развертывание ресурсов из пользовательского шаблона](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template) с помощью портал Azure. Можно также использовать [Azure CLI](../azure-resource-manager/templates/deploy-cli.md), [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)или [REST API](../azure-resource-manager/templates/deploy-rest.md).

3. Рекомендации по перемещению связанных ресурсов, таких как [учетные записи хранения Azure](../storage/common/storage-account-move.md), см. в руководстве по отдельным службам, перечисленным в разделе [Перемещение ресурсов Azure между регионами](../azure-resource-manager/management/move-region.md).

## <a name="verify"></a>Проверка

1. После завершения развертывания проверьте конечные точки приложения, чтобы проверить функциональные возможности приложения.

2. Вы также можете проверить состояние приложения, проверив состояние приложения ([AZ сеток](/cli/azure/ext/mesh/mesh/app#ext-mesh-az-mesh-app-show)Application View) и просмотр журналов приложений и ([AZ сетки кода-пакетных журналов](/cli/azure/ext/mesh/mesh/code-package-log)) с помощью [интерфейса командной строки Azure Service Fabric сеток](./service-fabric-mesh-quickstart-deploy-container.md#set-up-service-fabric-mesh-cli).

## <a name="commit"></a>Commit

После того как вы подтвердили эквивалентную функциональность приложения Service Fabricной сетки в целевом регионе, настройте входной контроллер (например, [шлюз приложений](../application-gateway/redirect-overview.md)), чтобы перенаправить трафик в новое приложение.

## <a name="clean-up-source-resources"></a>Очистка исходных ресурсов

Чтобы завершить перемещение Service Fabricного приложения сетки, [удалите исходное приложение и (или) родительскую группу ресурсов](../azure-resource-manager/management/delete-resource-group.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Перемещение ресурсов Azure между регионами](../azure-resource-manager/management/move-region.md)
* [Поддержка перемещения ресурсов Azure в разных регионах](../azure-resource-manager/management/region-move-support.md)
* [Перемещение ресурсов в новую группу ресурсов или подписку](../azure-resource-manager/management/move-resource-group-and-subscription.md)
* [Поддержка операций перемещения для ресурсов](../azure-resource-manager/management/move-support-resources.md
)

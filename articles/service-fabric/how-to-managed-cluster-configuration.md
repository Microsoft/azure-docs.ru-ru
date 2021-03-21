---
title: Настройка управляемого кластера Service Fabric (Предварительная версия)
description: Узнайте, как настроить управляемый кластер Service Fabric для автоматического обновления ОС, правил NSG и многого другого.
ms.topic: how-to
ms.date: 02/15/2021
ms.openlocfilehash: 44b1b949fe314231cb44f190c31b53903e47a904
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101732638"
---
# <a name="service-fabric-managed-cluster-preview-configuration-options"></a>Параметры конфигурации управляемого кластера (Предварительная версия) Service Fabric

Помимо выбора [номера SKU управляемого кластера Service Fabric](overview-managed-cluster.md#service-fabric-managed-cluster-skus) при создании кластера существует ряд других способов его настройки. В текущей предварительной версии можно:

* Настройка [параметров сети](how-to-managed-cluster-networking.md) для кластера
* Добавление [расширения масштабируемого набора виртуальных машин](how-to-managed-cluster-vmss-extension.md) в тип узла
* Настройка [управляемого удостоверения](how-to-managed-identity-managed-cluster-virtual-machine-scale-sets.md) для типов узлов
* Включение [автоматического обновления ОС](how-to-managed-cluster-configuration.md#enable-automatic-os-image-upgrades) для узлов
* Включение [шифрования дисков ОС и данных](how-to-enable-managed-cluster-disk-encryption.md) на узлах

## <a name="enable-automatic-os-image-upgrades"></a>Включить автоматическое обновление образа ОС

Вы можете включить автоматическое обновление образа ОС для виртуальных машин, на которых выполняются управляемые узлы кластера. Хотя ресурсы масштабируемого набора виртуальных машин управляются от вашего имени с помощью Service Fabric управляемых кластеров, вы можете включить автоматическое обновление образов ОС для узлов кластера. Как и в [классическом Service Fabric](service-fabric-best-practices-infrastructure-as-code.md#azure-virtual-machine-operating-system-automatic-upgrade-configuration) кластерах, управляемые узлы кластера по умолчанию не обновляются, чтобы предотвратить непредвиденные перерывы в работе кластера.

Включение автоматического обновления ОС.

* Используйте `2021-01-01-preview` (или более позднюю) версию ресурсов *Microsoft. ServiceFabric/Манажедклустерс* и *Microsoft. ServiceFabric/манажедклустерс/NodeType* .
* Присвойте свойству кластера `enableAutoOSUpgrade` *значение true* .
* Присвойте свойству ресурса кластера nodeType значение `vmImageVersion` *Latest* .

Пример:

```json
    {
      "apiVersion": "2021-01-01-preview",
      "type": "Microsoft.ServiceFabric/managedclusters",
      ...
      "properties": {
        ...
        "enableAutoOSUpgrade": true
      },
    },
    {
      "apiVersion": "2021-01-01-preview",
      "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
       ...
      "properties": {
        ...
        "vmImageVersion": "latest",
        ...
      }
    }
}

```

После включения Service Fabric начнет запрашивать и отслеживать версии образа ОС в управляемом кластере. Если доступна новая версия ОС, типы узлов кластера (масштабируемые наборы виртуальных машин) будут обновлены по одному за раз. Service Fabric обновления среды выполнения выполняются только после подтверждения того, что обновления образа ОС узла кластера не выполняются.

В случае сбоя обновления Service Fabric повторит попытку через 24 часа, не более трех повторных попыток. Как и классические (неуправляемые) Service Fabric обновления, неработоспособные приложения или узлы могут блокировать обновление образа ОС.

Дополнительные сведения об обновлении образов см. в статье [Автоматическое обновление образа ОС с помощью масштабируемых наборов виртуальных машин Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md).

## <a name="next-steps"></a>Дальнейшие действия

[Общие сведения об управляемых кластерах Service Fabric](overview-managed-cluster.md)

[Шаблоны кластера Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)

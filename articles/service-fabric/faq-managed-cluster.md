---
title: Распространенные вопросы об управляемых кластерах Service Fabric
description: Часто задаваемые вопросы о Service Fabric управляемых кластерах, включая возможности, варианты использования и распространенные сценарии.
ms.topic: troubleshooting
ms.author: pepogors
author: peterpogorski
ms.date: 02/15/2021
ms.custom: references_regions
ms.openlocfilehash: aa77896ba88d0ffd0a6f94a84603b5f4a1803357
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100633093"
---
# <a name="service-fabric-managed-clusters-frequently-asked-questions"></a>Распространенные вопросы об управляемых кластерах Service Fabric

Ниже приведены некоторые часто задаваемые вопросы и ответы на Service Fabric управляемых кластеров (Предварительная версия).

## <a name="general"></a>Общие сведения

### <a name="what-are-service-fabric-managed-clusters"></a>Что такое Service Fabric управляемых кластеров?

Service Fabric управляемые кластеры — это эволюция модели ресурсов кластеров Service Fabric, призванная упростить развертывание кластеров и управление ими. Service Fabric управляемый кластер использует Azure Resource Managerную модель инкапсуляции, чтобы пользователю было достаточно определить и развернуть только один ресурс кластера по сравнению с множеством независимых ресурсов, которые они должны развертывать сегодня (масштабируемый набор виртуальных машин, Load Balancer, IP-адрес и др.).

### <a name="what-regions-are-supported-in-the-preview"></a>Какие регионы поддерживаются в предварительной версии?

Поддерживаемые регионы для предварительной версии Service Fabric управляемых кластеров: `centraluseuap` , `eastus2euap` , `eastasia` ,, `northeurope` `westcentralus` и `eastus2` .

### <a name="can-i-do-an-in-place-migration-of-my-existing-service-fabric-cluster-to-a-managed-cluster-resource"></a>Можно ли выполнить перенос существующего кластера Service Fabric в управляемый кластерный ресурс на месте?

Нет. В настоящее время потребуется создать новый ресурс кластера Service Fabric, чтобы использовать новый тип ресурса кластера, управляемого Service Fabric.

### <a name="is-there-an-additional-cost-for-service-fabric-managed-clusters"></a>Существуют ли дополнительные затраты на Service Fabric управляемых кластеров?

Нет. За Service Fabric управляемый кластер не взимается дополнительная плата, кроме затрат на базовые ресурсы вычислений, хранения и сетевых ресурсов, необходимых для кластера.

### <a name="is-there-a-new-sla-introduced-by-the-service-fabric-managed-cluster-resource"></a>Есть ли новое соглашение об уровне обслуживания, представленное Service Fabric ресурсом управляемого кластера?

Соглашение об уровне обслуживания не меняется от текущей модели ресурсов Service Fabric.

### <a name="what-is-the-difference-between-a-basic-and-standard-sku-cluster"></a>В чем разница между базовым и кластером SKU уровня "Стандартный"?

Базовый кластер SKU означает, что большинство конфигураций предоставляется поставщиком ресурсов Service Fabric. Кластеры уровня "базовый" предназначены для тестирования и подготовки в рабочей среде. Кластер уровня SKU "Стандартный" позволяет пользователям настраивать кластер в соответствии с потребностями. Дополнительные сведения см. в разделе [Service Fabric номера SKU управляемого кластера](./overview-managed-cluster.md#service-fabric-managed-cluster-skus).

## <a name="cluster-deployment-and-management"></a>Развертывание кластера и управление им

### <a name="i-run-custom-script-extensions-on-my-virtual-machine-scale-set-can-i-continue-to-do-that-with-a-managed-service-fabric-resource"></a>Я запускаю пользовательские расширения сценариев в масштабируемом наборе виртуальных машин, можно ли продолжить это с управляемым ресурсом Service Fabric?

Да, можно указать расширения виртуальной машины в типах управляемых узлов кластера. Дополнительные сведения см. в разделе [Добавление расширения масштабируемого набора в тип узла управляемого кластера Service Fabric](how-to-managed-cluster-vmss-extension.md).

### <a name="i-want-to-have-an-internal-only-load-balancer-is-that-possible"></a>Я хочу получить подсистему балансировки нагрузки только для внутреннего использования?

Нет. В настоящее время нет возможности использовать подсистему балансировки нагрузки только для внутреннего использования. Мы рекомендуем блокировать правила группы безопасности сети (NSG), чтобы заблокировать любой нежелательный входящий и исходящий трафик.

### <a name="can-i-autoscale-my-cluster"></a>Можно ли автоматически масштабировать кластер?

Автомасштабирование в настоящее время недоступно в предварительной версии.

### <a name="can-i-deploy-my-cluster-across-availability-zones"></a>Можно ли развернуть кластер в зонах доступности?

Кластеры зон с перекрестной доступностью в настоящее время недоступны в предварительной версии.

### <a name="can-i-select-between-automatic-and-manual-upgrades-for-my-cluster-runtime"></a>Можно ли выбрать между автоматическим и ручным обновлением среду выполнения кластера?

В предварительной версии все обновления среды выполнения будут выполняться автоматически.

## <a name="applications"></a>Приложения

### <a name="is-there-a-local-development-experience-for-service-fabric-managed-clusters"></a>Существует ли локальная среда разработки для Service Fabric управляемых кластеров?

Локальная среда разработки остается неизменной по отношению к существующим Service Fabric кластерам. Дополнительные сведения см. в разделе [Настройка среды разработки](./service-fabric-get-started.md) для получения дополнительных сведений о локальном интерфейсе разработки.

### <a name="can-i-deploy-my-applications-as-an-azure-resource-manager-resource"></a>Можно ли развернуть приложения как ресурс Azure Resource Manager?

Да. Добавлена поддержка для развертывания приложений в качестве ресурса Azure Resource Manager (в дополнение к развертыванию с помощью PowerShell и интерфейса командной строки). Чтобы приступить к работе, см. статью [развертывание Service Fabric управляемого кластера приложения с помощью шаблона ARM](how-to-managed-cluster-app-deployment-template.md).

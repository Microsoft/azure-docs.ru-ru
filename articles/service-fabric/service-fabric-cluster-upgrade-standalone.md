---
title: Обновление автономного кластера Azure Service Fabric
description: Сведения о том, как обновить версию или конфигурацию изолированного кластера Azure Service Fabric.  T
ms.topic: conceptual
ms.date: 11/12/2018
ms.openlocfilehash: 519a9163d16fda2dd9fcf49cf22fe4ad4a272e09
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86260979"
---
# <a name="upgrading-and-updating-a-service-fabric-standalone-cluster"></a>Обновление изолированного кластера Service Fabric

Для любой современной системы разработка с учетом возможности модернизации является неотъемлемой составляющей успеха продукта. Изолированный кластер Azure Service Fabric — это ресурс, владельцем которого вы являетесь. В этой статье приводятся сведения об обновлении.

## <a name="controlling-the-fabric-version-that-runs-on-your-cluster"></a>Управление версиями Service Fabric в кластере
Убедитесь, что кластер всегда работает под управлением [поддерживаемой версии Service Fabric](service-fabric-versions.md). Когда корпорация Майкрософт объявляет о выпуске новой версии Service Fabric, для предыдущей версии определяется срок завершения жизненного цикла. Этот срок составляет по меньшей мере 60 дней с даты объявления. О доступности новых выпусков сообщается в [блоге группы разработчиков Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric), после чего вы можете их использовать.

Вы можете настроить для кластера автоматическое обновление Service Fabric по мере выпуска новых версий корпорацией Майкрософт или вручную выбрать поддерживаемую версию, в которой должен работать ваш кластер. Дополнительные сведения см. в статье [Обновление автономного кластера Azure Service Fabric в Windows Server](service-fabric-cluster-upgrade-windows-server.md).

## <a name="customize-configuration-settings"></a>Настройка параметров конфигурации

В файле *ClusterConfig.json* можно настроить множество разных [параметров конфигурации](service-fabric-cluster-manifest.md), например уровень надежности кластера и свойства узла.  Дополнительные сведения см. в статье, посвященной [обновлению конфигурации изолированного кластера](service-fabric-cluster-config-upgrade-windows-server.md).  Кроме того, можно настроить множество других сложных параметров.  Дополнительные сведения см. в статье [Настройка параметров кластера Service Fabric](service-fabric-cluster-fabric-settings.md).

## <a name="define-node-properties"></a>Определение свойств узла
Иногда необходимо обеспечить выполнение конкретных рабочих нагрузок только на узлах определенных типов в кластере. Например, для одних рабочих нагрузок могут требоваться графические процессоры или накопители SSD, а для других — нет. Для каждого из типов узлов в кластере можно добавить пользовательские свойства. Ограничения размещения представляют собой операторы, привязанные к отдельным службам и включающие одно или несколько свойств узла. Ограничения размещения определяют, где должны запускаться службы.

Сведения об использовании ограничений на размещение, свойств узла и способах их определения см. [здесь](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).
 

## <a name="add-capacity-metrics"></a>Добавление метрик емкости
Для каждого типа узла можно добавить настраиваемые метрики емкости, которые будут использоваться в приложениях для передачи данных о нагрузке. Дополнительные сведения об использовании метрик емкости для отчетах о нагрузке см. в документах по диспетчеру кластерных ресурсов Service Fabric на страницах [Описание кластера Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md) и [Управление потреблением ресурсов и нагрузкой в Service Fabric с помощью метрик](service-fabric-cluster-resource-manager-metrics.md).

## <a name="patch-the-os-in-the-cluster-nodes"></a>Применение исправлений для операционной системы на узлах кластера
Приложение для управления исправлениями — это приложение Service Fabric, которое позволяет автоматизировать установку исправлений операционной системы в кластере Service Fabric и избегать простоев. [Приложение для управления исправлениями для Windows](service-fabric-patch-orchestration-application.md) можно развернуть в кластере, чтобы установить исправления контролируемым образом без простоев служб. 


## <a name="next-steps"></a>Дальнейшие действия
* Узнайте, как настроить некоторые [Параметры структуры кластера Service Fabric](service-fabric-cluster-fabric-settings.md) .
* Узнайте, как выполнять [масштабирование кластера](service-fabric-cluster-scale-in-out.md)
* Дополнительные сведения об [обновлении приложений](service-fabric-application-upgrade.md)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG

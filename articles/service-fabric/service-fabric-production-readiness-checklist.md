---
title: Контрольный список готовности рабочей среды Azure Service Fabric
description: Подготовка приложения Service Fabric и рабочей среде кластера с помощью рекомендаций.
ms.topic: conceptual
ms.date: 6/05/2019
ms.openlocfilehash: a03df40a8ce213c5de9ed7017d47713c4de3449d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87835873"
---
# <a name="production-readiness-checklist"></a>Контрольный список готовности рабочей среды

Готовы ли ваше приложение и кластер к приему рабочего трафика? Запуск и тестирование приложения и кластера не обязательно означает, что они готовы к перемещению в рабочую среду. Обеспечьте бесперебойную работу приложения и кластера, выполнив следующий контрольный список. Мы настоятельно рекомендуем выполнить все его пункты. Разумеется, вы можете использовать альтернативные решения для какого-либо пункта (например, собственные инфраструктуры диагностики).


## <a name="prerequisites-for-production"></a>Предварительные требования для рабочей среды
1. Рекомендации по работе с Azure Service Fabric: [проектирование приложений](./service-fabric-best-practices-applications.md), [Безопасность](./service-fabric-best-practices-security.md), работа в [сети](./service-fabric-best-practices-networking.md), [планирование и масштабирование ресурсов](./service-fabric-best-practices-capacity-scaling.md), [инфраструктура как код](./service-fabric-best-practices-infrastructure-as-code.md), [мониторинг и диагностика](./service-fabric-best-practices-monitoring.md). 
1. [Настройте параметры FabricTransport](./service-fabric-reliable-actors-fabrictransportsettings.md) , если вы используете модель программирования Reliable Actors и требует безопасного взаимодействия между службами.
1. Для кластеров с более чем 20 ядрами или 10 узлами создайте выделенный тип основного узла для системных служб. Добавьте [ограничения размещения](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md), чтобы зарезервировать тип основного узла для системных служб.
1. Используйте номер SKU D2v2 или выше для типа основного узла. Рекомендуется выбрать номер SKU с емкостью диска по меньшей мере 50 ГБ.
1. Рабочие кластеры должны быть [защищены](service-fabric-cluster-security.md). Примером настройки безопасного кластера может послужить этот [шаблон кластера](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/7-VM-Windows-3-NodeTypes-Secure-NSG). Используйте общие имена сертификатов и старайтесь не использовать самозаверяющие сертификаты.
1. Добавьте [ограничения ресурсов для контейнеров и служб](service-fabric-resource-governance.md) таким образом, чтобы они не занимали более 75 % ресурсов узла. 
1. Изучите и задайте [уровень устойчивости](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster). Уровень устойчивости Silver или выше рекомендуется использовать для типов узлов, на которых выполняются рабочие нагрузки с отслеживанием состояния.
1. Изучите и выберите [уровень надежности](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) типа узла. Рекомендуется использовать уровень надежности Silver или выше.
1. Протестируйте загрузку и масштабирование своих рабочих нагрузок, чтобы определить [требования к емкости](service-fabric-cluster-capacity.md) кластера. 
1. Службы и приложения отслеживаются, также создаются и сохраняются журналы приложений. При этом могут создаваться оповещения. Например, см. статью [Добавление журналов в приложение Service Fabric](service-fabric-how-to-diagnostics-log.md) и [Отслеживание контейнеров с помощью Azure Monitor журналов](service-fabric-diagnostics-oms-containers.md).
1. Кластер отслеживается с помощью предупреждений (например, с [Azure Monitor журналами](service-fabric-diagnostics-event-analysis-oms.md)). 
1. Базовая инфраструктура масштабируемого набора виртуальных машин отслеживается с помощью предупреждений (например, с [журналами Azure Monitor](service-fabric-diagnostics-oms-agent.md).
1. В кластере всегда есть [основные и дополнительные сертификаты](service-fabric-cluster-security-update-certs-azure.md) (то есть вы не будете заблокированы).
1. Используйте разные кластеры для среды разработки, промежуточной среды и рабочей среды. 
1. [Обновления приложений](service-fabric-application-upgrade.md) и [обновления кластера](service-fabric-tutorial-upgrade-cluster.md) следует сначала проверить на кластерах разработки и промежуточных кластерах. 
1. Отключите автоматические обновления на рабочих кластерах и включите их на кластерах разработки и промежуточных кластерах (при необходимости можно выполнить откат). 
1. Установите целевую точку восстановления (RPO) для службы, а также настройте [процесс аварийного восстановления](service-fabric-disaster-recovery.md) и протестируйте его.
1. Запланируйте [масштабирование](service-fabric-cluster-scaling.md) кластера вручную или программным способом.
1. Запланируйте [установку исправлений](service-fabric-patch-orchestration-application.md) на узлах кластера. 
1. Установите конвейер CI/CD, чтобы последние внесенные изменения постоянно проверялись. Например, можно использовать [Azure DevOps](service-fabric-tutorial-deploy-app-with-cicd-vsts.md) или [Jenkins](/azure/developer/jenkins/deploy-to-service-fabric-cluster).
1. Выполните нагрузочное тестирование кластеров разработки и промежуточных кластеров с помощью [службы анализа сбоев](service-fabric-testability-overview.md) и вызовите управляемый [хаос](service-fabric-controlled-chaos.md). 
1. Запланируйте [масштабирование](service-fabric-concepts-scalability.md) приложений. 


Если вы используете модель программирования Reliable Services или Reliable Actors в Service Fabric, необходимо выполнить следующие пункты.
1. Обновляйте приложения во время локальной разработки, чтобы убедиться, что код службы учитывает маркер отмены в методе `RunAsync` и закрывает пользовательские прослушиватели связи.
1. Избегайте [типичных проблем](service-fabric-work-with-reliable-collections.md) при использовании Reliable Collections.
1. Отслеживайте счетчики производительности памяти среды CLR .NET при выполнении нагрузочных тестов. Кроме того, проверяйте наличие высокой частоты сбора мусора или неконтролируемого увеличения размера кучи.
1. Храните автономные резервные копии [Reliable Services и Reliable Actors](service-fabric-reliable-services-backup-restore.md) и проверьте процесс восстановления.
1. Количество экземпляров виртуальной машины основного NodeType в идеале должно быть равным минимальному значению для уровня надежности кластеров; условия, при которых уместно превышать минимальный уровень: временно при вертикальном масштабировании номера SKU масштабируемого набора виртуальных машин основных NodeType.

## <a name="optional-best-practices"></a>Дополнительные рекомендации

Хотя выше перечислены достаточные предварительные требования для переноса в рабочую среду, следует также рассмотреть следующие пункты.
1. Подключите [модель работоспособности Service Fabric](service-fabric-health-introduction.md), чтобы расширить встроенные возможности оценки работоспособности и создания отчетов.
1. Разверните пользовательскую службу наблюдения, которая отслеживает приложение и сообщает о [загрузке](service-fabric-cluster-resource-manager-metrics.md) для [балансировки ресурсов](service-fabric-cluster-resource-manager-balancing.md). 


## <a name="next-steps"></a>Дальнейшие действия
* [Руководство. Развертывание кластера Service Fabric на платформе Windows в виртуальной сети Azure](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* [Руководство. Развертывание кластера Service Fabric на платформе Linux в виртуальной сети Azure](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
* Дополнительные сведения о [жизненном цикле приложения](service-fabric-application-lifecycle.md) Service Fabric.
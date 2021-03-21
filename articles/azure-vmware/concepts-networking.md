---
title: Основные понятия — сетевое взаимодействие
description: Узнайте о ключевых аспектах и вариантах использования сети и взаимосвязи в решении VMware для Azure.
ms.topic: conceptual
ms.date: 03/11/2021
ms.openlocfilehash: 4c964151c49e2fea56031dd24bacf4655753a18d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103491815"
---
# <a name="azure-vmware-solution-networking-and-interconnectivity-concepts"></a>Сети решения Azure VMware и основные понятия взаимодействия

[!INCLUDE [avs-networking-description](includes/azure-vmware-solution-networking-description.md)]

Существует два способа взаимодействия в частном облаке решения Azure VMware.

- [**Простое взаимодействие только с Azure**](#azure-virtual-network-interconnectivity) позволяет управлять частным облаком и использовать его только с одной виртуальной сетью в Azure. Эта реализация лучше всего подходит для оценок и реализаций решений VMware для Azure, которые не нуждаются в доступе из локальных сред.

- [**Возможность взаимодействия между локальными и частным облакоми**](#on-premises-interconnectivity) расширяет базовую реализацию только Azure, включая взаимодействие между локальными и частными облаками решения Azure VMware.
 
В этой статье рассматриваются основные понятия, которые устанавливают сетевые подключения и взаимосвязь, включая требования и ограничения. Эта статья содержит сведения, необходимые для настройки сети для работы с решением VMware для Azure.

## <a name="azure-vmware-solution-private-cloud-use-cases"></a>Варианты использования частного облака решения Azure VMware

Ниже перечислены варианты использования частных облаков решений Azure VMware.
- Новые рабочие нагрузки виртуальных машин VMware в облаке
- Передача рабочей нагрузки виртуальной машины в облако (только для решения VMware из локальной среды в Azure)
- Миграция рабочей нагрузки виртуальной машины в облако (только для решения VMware из локальной среды в Azure)
- Аварийное восстановление (решение Azure VMware для Azure VMware или локальное решение для Azure VMware)
- Использование служб Azure

> [!TIP]
> Все варианты использования службы решения Azure VMware включены с подключением между локальными и частными облаками.

## <a name="azure-virtual-network-interconnectivity"></a>Взаимодействие виртуальной сети Azure

Вы можете соединить виртуальную сеть Azure с реализацией частного облака решения VMware для Azure. Вы можете управлять частным облаком решения Azure VMware, использовать рабочие нагрузки в частном облаке и получать доступ к другим службам Azure.

На схеме ниже показана базовая сетевая связь, установленная во время развертывания частного облака. В нем показаны логические сетевые подключения между виртуальной сетью в Azure и частным облаком. Это подключение устанавливается через сервер ExpressRoute, который входит в состав службы решения Azure VMware. Взаимодействие выполняет следующие основные варианты использования:

- Входящий доступ к vCenter Server и НСКС-T Manager, доступный на виртуальных машинах в подписке Azure.
- Исходящий доступ с виртуальных машин в частном облаке к службам Azure.
- Входящий доступ к рабочим нагрузкам, выполняемым в частном облаке.


:::image type="content" source="media/concepts/adjacency-overview-drawing-single.png" alt-text="Подключение базовой виртуальной сети к частному облаку" border="false":::

## <a name="on-premises-interconnectivity"></a>Локальное взаимодействие

В полностью взаимосвязанных сценариях вы можете получить доступ к решению VMware для Azure из виртуальных сетей Azure и из локальной среды. Эта реализация является расширением базовой реализации, описанной в предыдущем разделе. Для подключения из локальной среды к частному облаку решения Azure VMware в Azure требуется канал ExpressRoute.

На схеме ниже показано взаимодействие между локальными и частным облаком, которое позволяет использовать следующие варианты использования:

- Горячий/холодный vCenter vMotion между локальной средой и решением VMware для Azure.
- С локального компьютера на Azure VMware решение для управления частным облаком.

:::image type="content" source="media/concepts/adjacency-overview-drawing-double.png" alt-text="Виртуальная сеть и локальное полное подключение к частному облаку" border="false":::

Для полного взаимодействия с частным облаком необходимо включить ExpressRoute Global Reach а затем запросить ключ авторизации и идентификатор частного пиринга для Global Reach в портал Azure. Ключ авторизации и идентификатор пиринга используются для установления Global Reach между каналом ExpressRoute в подписке и каналом ExpressRoute для частного облака. После связывания две цепи ExpressRoute направляют сетевой трафик между локальными средами в частное облако. Дополнительные сведения о процедурах см. в [руководстве по созданию Global REACH пиринга ExpressRoute в частном облаке](tutorial-expressroute-global-reach-private-cloud.md).

## <a name="limitations"></a>Ограничения
[!INCLUDE [azure-vmware-solutions-limits](includes/azure-vmware-solutions-limits.md)]

## <a name="next-steps"></a>Следующие шаги 

Теперь, когда вы узнали о сети решения Azure VMware и концепции взаимодействия, вы можете узнать о следующих возможностях:

- [Основные понятия хранилища решений VMware для Azure](concepts-storage.md).
- [Концепции Azure VMware для идентификации решений](concepts-identity.md).
- [Как включить ресурс решения Azure VMware](enable-azure-vmware-solution.md).

<!-- LINKS - external -->
[enable Global Reach]: ../expressroute/expressroute-howto-set-global-reach.md

<!-- LINKS - internal -->
[concepts-upgrades]: ./concepts-upgrades.md
[concepts-storage]: ./concepts-storage.md

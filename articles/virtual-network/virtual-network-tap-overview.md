---
title: Обзор TAP виртуальной сети Azure | Документы Майкрософт
description: Подробная информация о TAP виртуальной сети. TAP виртуальной сети предоставляет вам более подробную копию сетевого трафика виртуальной машины, который можно передавать в сборщик пакетов.
services: virtual-network
documentationcenter: na
author: karthikananth
manager: ganesr
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/14/2019
ms.author: kaanan
ms.openlocfilehash: 6160dd09edc57f2f52306d4dad0dde413fff0616
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102617188"
---
# <a name="virtual-network-tap"></a>TAP виртуальной сети
> [!IMPORTANT]
> В настоящее время обзор TAP для виртуальной сети находится во всех регионах Azure. Вы можете отправить нам по электронной почте по адресу <azurevnettap@microsoft.com> своей подписки, и мы будем уведомлять о будущих обновлениях предварительной версии. В предварительной конфигурации вы можете использовать решения на основе агента или NVA, которые предоставляют функции видимости КАСАНИя и сети с помощью наших [решений партнеров пакета пакетов](#virtual-network-tap-partner-solutions) , доступных в [предложениях Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances%3Ball&search=Network%20Traffic&filters=partners).

TAP (точка доступа к терминалу) виртуальной сети Azure позволяет непрерывно передавать сетевой трафик виртуальной машины в сборщик сетевых пакетов или средство аналитики. Средство сбора данных или анализа предоставляется партнером по [сетевым виртуальным устройствам](https://azure.microsoft.com/solutions/network-appliances/) . Список партнерских решений, работающих с TAP виртуальной сети, см. в разделе о [партнерских решениях](#virtual-network-tap-partner-solutions).
Ниже показано, как работает TAP виртуальной сети. Конфигурацию TAP можно добавить в [сетевой интерфейс](virtual-network-network-interface.md), присоединенный к виртуальной машине, развернутой в вашей виртуальной сети. Местом назначения станет IP-адрес виртуальной сети в той же виртуальной сети, что и отслеживаемый сетевой интерфейс или [одноранговая виртуальная сеть](virtual-network-peering-overview.md). Для обеспечения высокой доступности решение сборщика для TAP виртуальной сети можно развернуть за внутренней подсистемой балансировки нагрузки Azure.
![Как работает TAP виртуальной сети](./media/virtual-network-tap/architecture.png)

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к созданию КАСАНИя виртуальной сети, необходимо получить сообщение с подтверждением, зарегистрированное в предварительной версии, и создать одну или несколько виртуальных машин, созданных с помощью [Azure Resource Manager](../azure-resource-manager/management/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) модели развертывания, и партнерское решение для агрегирования трафика TAP в том же регионе Azure. Если в вашей виртуальной сети нет ни одного из партнерских решений, разверните одно из них, как указано в разделе о [партнерских решениях](#virtual-network-tap-partner-solutions). Один и тот же ресурс TAP виртуальной сети позволяет объединять трафик из разных сетевых интерфейсов в одной и той же или разных подписках. Если отслеживаемые сетевые интерфейсы находятся в разных подписках, обе подписки должны быть связаны с одним клиентом Azure Active Directory. Кроме того, отслеживаемые сетевые интерфейсы и конечная точка назначения для объединения трафика TAP могут находиться в одноранговых виртуальных сетях в одном и том же регионе. Если вы используете эту модель развертывания, перед настройкой TAP виртуальной сети убедитесь, что [пиринг виртуальных сетей](virtual-network-peering-overview.md) включен.

## <a name="permissions"></a>Разрешения

Учетные записи, используемые для применения конфигурации TAP к сетевым интерфейсам, должны быть назначены роли [участник сети](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) или [пользовательской роли](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json), которой назначены необходимые действия, перечисленные в следующей таблице:

| Действие | Имя |
|---|---|
| Microsoft.Network/virtualNetworkTaps/* | Требуется для создания, обновления, чтения и удаления ресурса TAP виртуальной сети |
| Microsoft.Network/networkInterfaces/read | Требуется для чтения ресурса сетевого интерфейса, в котором будет настраиваться TAP |
| Microsoft.Network/tapConfigurations/* | Требуется для создания, обновления, чтения и удаления конфигурации TAP в сетевом интерфейсе |


## <a name="virtual-network-tap-partner-solutions"></a>Партнерские решения TAP виртуальной сети

### <a name="network-packet-brokers"></a>Брокеры сетевых пакетов

- [Гигавуе Cloud Suite для Azure](https://www.gigamon.com/solutions/cloud/public-cloud/gigavue-cloud-suite-azure.html)
- [Ixia CloudLens](https://www.ixiacom.com/cloudlens/cloudlens-azure)
- [Nubeva Prisms](https://www.nubeva.com/azurevtap)
- [Big Switch Big Monitoring Fabric](https://www.bigswitch.com/products/big-monitoring-fabric/public-cloud/microsoft-azure)

### <a name="security-analytics-networkapplication-performance-management"></a>Аналитика безопасности и управления производительностью сети или приложений

- [Безопасность в спящем режиме](https://awakesecurity.com/technology-partners/microsoft-azure/)
- [Стеалсватчное облако Cisco](https://blogs.cisco.com/security/cisco-stealthwatch-cloud-and-microsoft-azure-reliable-cloud-infrastructure-meets-comprehensive-cloud-security)
- [дарктраце](https://www.darktrace.com/en/azure/)
- [ExtraHop Reveal(x)](https://www.extrahop.com/partners/tech-partners/microsoft/)
- [Fidelis Cybersecurity](https://www.fidelissecurity.com/technology-partners/microsoft-azure )
- [Flowmon](https://www.flowmon.com/blog/azure-vtap)
- [Нетфорт Лангуардиан](https://www.netfort.com/languardian/solutions/visibility-in-azure-network-tap/)
- [Netscout vSTREAM]( https://www.netscout.com/marketplace-azure)
- [Безопасность с неименованием](https://nonamesecurity.com/)
- [Riverbed Стилцентрал Аппреспонсе]( https://www.riverbed.com/products/steelcentral/steelcentral-appresponse-11.html)
- [Платформа RSA NetWitness®](https://www.rsa.com/content/dam/en/solution-brief/rsa-netwitness-platform-overview-for-federal-agencies.pdf)
- [Vectra Cognito](https://vectra.ai/microsoftazure)



## <a name="next-steps"></a>Следующие шаги

- Дополнительные сведения о [создании TAP виртуальной сети](tutorial-tap-virtual-network-cli.md).

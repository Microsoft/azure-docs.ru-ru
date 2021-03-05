---
title: Виртуальная сеть для служб Azure
titlesuffix: Azure Virtual Network
description: Узнайте, как развернуть выделенные службы Azure в виртуальной сети и узнать о возможностях, предоставляемых этими развертываниями.
services: virtual-network
documentationcenter: na
author: mohnader
ms.service: virtual-network
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/06/2020
ms.author: kumud
ms.openlocfilehash: 79b6fa1043b1bb8add6b6beb5fd38312ee12ea75
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102210280"
---
# <a name="deploy-dedicated-azure-services-into-virtual-networks"></a>Развертывание выделенных служб Azure в виртуальных сетях

При развертывании выделенных служб Azure в [виртуальной сети](virtual-networks-overview.md) вы можете взаимодействовать с ресурсами служб в частном порядке через частные IP-адреса.

![Службы, развернутые в виртуальной сети](./media/virtual-network-for-azure-services/deploy-service-into-vnet.png)

Развертывание служб в виртуальной сети обеспечивает следующие возможности:

- Ресурсы в виртуальной сети могут взаимодействовать друг с другом в частном порядке через частные IP-адреса, например непосредственно передавать данные между службой HDInsight и сервером SQL Server, запущенным на виртуальной машине в виртуальной сети.
- Локальные ресурсы могут обращаться к ресурсам в виртуальной сети с помощью частных IP-адресов через [VPN-подключение типа "сеть —сеть" (VPN-шлюз)](../vpn-gateway/design.md?toc=%2fazure%2fvirtual-network%2ftoc.json#s2smulti) или [канал ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Виртуальные сети могут создавать [пиринговую связь](virtual-network-peering-overview.md), что позволяет ресурсам в виртуальной сети взаимодействовать друг с другом через частные IP-адреса.
- Экземпляры службы в виртуальной сети обычно полностью управляются службой Azure. Сюда входит отслеживание работоспособности ресурсов и масштабирование с учетом нагрузки.
- Экземпляры служб развертываются в подсети в виртуальной сети. Входящий и исходящий доступ к подсети должен быть открыт посредством [групп безопасности сети](./network-security-groups-overview.md#network-security-groups) согласно рекомендациям для служб.
- Некоторые службы также накладывают ограничения на подсеть, в которой они развернуты, ограничивая применение политик, маршруты или сочетания виртуальных машин и ресурсов служб в одной подсети. Проверяйте ограничения по каждой службе, так как со временем они могут меняться. Примерами таких служб являются Azure NetApp Files, Выделенное устройство HSM, Экземпляры контейнеров Azure, Служба приложений. 
- При необходимости для служб можно использовать [делегированную подсеть](virtual-network-manage-subnet.md#add-a-subnet), чтобы явно указать, что в этой подсети может размещаться определенная служба. Делегирование подсети прямо разрешает службам создавать в подсети необходимые для них ресурсы.
- См. пример ответа REST API в [виртуальной сети с делегированной подсетью](/rest/api/virtualnetwork/virtualnetworks/get#get-virtual-network-with-a-delegated-subnet). Полный список служб, использующих модель делегированной подсети, можно получить с помощью API [Доступные делегирования](/rest/api/virtualnetwork/availabledelegations/list).

### <a name="services-that-can-be-deployed-into-a-virtual-network"></a>Службы, которые можно развернуть в виртуальной сети

|Категория|Служба| Выделенная<sup>1</sup> подсеть
|-|-|-|
| Службы вычислений | Виртуальные машины: [Linux](/previous-versions/azure/virtual-machines/linux/infrastructure-example?toc=%2fazure%2fvirtual-network%2ftoc.json) или [Windows](/previous-versions/azure/virtual-machines/windows/infrastructure-example?toc=%2fazure%2fvirtual-network%2ftoc.json) <br/>[Масштабируемые наборы виртуальных машин](../virtual-machine-scale-sets/virtual-machine-scale-sets-mvss-existing-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Облачная служба](/previous-versions/azure/reference/jj156091(v=azure.100)): только виртуальная сеть (классическая)<br/> [Пакетная служба Azure](../batch/nodes-and-pools.md?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual-network-vnet-and-firewall-configuration)| Нет <br/> Нет <br/> Нет <br/> Нет<sup>2</sup>
| Сеть | [Шлюз приложений — WAF](../application-gateway/application-gateway-ilb-arm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[VPN-шлюз](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Брандмауэр Azure](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)  <br/> [Бастион Azure](../bastion/bastion-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Виртуальные сетевые модули](/windows-server/networking/sdn/manage/use-network-virtual-appliances-on-a-vn)| Да <br/> Да <br/> Да <br/> Да <br/> Нет
|Данные|[Кэш Redis](../azure-cache-for-redis/cache-how-to-premium-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Управляемый экземпляр SQL Azure](../azure-sql/managed-instance/connectivity-architecture-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)| Да <br/> Да <br/> 
|Analytics | [Azure HDInsight](../hdinsight/hdinsight-plan-virtual-network-deployment.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks?toc=%2fazure%2fvirtual-network%2ftoc.json) |Нет<sup>2</sup> <br/> Нет<sup>2</sup> <br/> 
| Идентификация | [Доменные службы Azure Active Directory](../active-directory-domain-services/tutorial-create-instance.md?toc=%2fazure%2fvirtual-network%2ftoc.json) |Нет <br/>
| Контейнеры | [Служба Azure Kubernetes (AKS)](../aks/concepts-network.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Экземпляр контейнера Azure (ACI)](https://www.aka.ms/acivnet)<br/>[Обработчик службы контейнеров Azure](https://github.com/Azure/acs-engine) с [подключаемым модулем](https://github.com/Azure/acs-engine/tree/master/examples/vnet) CNI для виртуальной сети Azure<br/>[Функции Azure](../azure-functions/functions-networking-options.md#virtual-network-integration) |Нет<sup>2</sup><br/> Да <br/> Нет <br/> Да
| Интернет | [Управление API](../api-management/api-management-using-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Веб-приложения](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Среда службы приложений](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>|Да <br/> Да <br/> Да <br/> Да
| Размещенные* | [Выделенное устройство HSM Azure](../dedicated-hsm/index.yml?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure NetApp Files](../azure-netapp-files/azure-netapp-files-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>|Да <br/> Да <br/>
| Azure Spring Cloud | [Развертывание в виртуальной сети Azure (внедрение VNet)](../spring-cloud/spring-cloud-tutorial-deploy-in-azure-virtual-network.md)<br/>| Да <br/>
| | |

<sup>1</sup> "Выделенная" означает, что в этой подсети можно развертывать только ресурсы, относящиеся к службе, и не может использоваться вместе с виртуальными машинами или Масштабируемыми наборами виртуальных машин клиента. <br/> 
<sup>2</sup> Рекомендуется организовать работу этих служб в выделенной подсети, но это не обязательное требование, накладываемое службой.

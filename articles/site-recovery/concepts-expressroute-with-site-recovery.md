---
title: Об использовании ExpressRoute с Azure Site Recovery
description: Сведения об использовании Azure ExpressRoute и службы Azure Site Recovery для аварийного восстановления и миграции.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/13/2019
ms.author: mayg
ms.openlocfilehash: 99fa8d4cf8f48d0fe72da36baef20c83add438c0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94330263"
---
# <a name="azure-expressroute-with-azure-site-recovery"></a>Использование Azure ExpressRoute и Azure Site Recovery

Microsoft Azure ExpressRoute позволяет переносить локальные сети в облако Microsoft по частному подключению, которое обеспечивается поставщиком услуг подключения. ExpressRoute позволяет устанавливать подключения к облачным службам Майкрософт, таким как Microsoft Azure, Microsoft 365 и Dynamics 365.

Эта статья содержит сведения об использовании ExpressRoute и Azure Site Recovery для аварийного восстановления и миграции.

## <a name="expressroute-circuits"></a>Каналы ExpressRoute

Канал ExpressRoute представляет собой логическое подключение вашей локальной сети к облачным службам Майкрософт через поставщика услуг подключения. Таких каналов ExpressRoute может быть несколько. Каждый канал может находиться в том же или в других регионах и подключаться к среде через различных поставщиков услуг связи. Дополнительные сведения о каналах ExpressRoute см. [здесь](../expressroute/expressroute-circuit-peerings.md).

С каналом ExpressRoute связано несколько доменов маршрутизации. Дополнительные сведения о доменах маршрутизации ExpressRoute и их сравнение см. [здесь](../expressroute/expressroute-circuit-peerings.md#peeringcompare).

## <a name="on-premises-to-azure-replication-with-expressroute"></a>Репликация из локальной среды в Azure с помощью ExpressRoute

Azure Site Recovery позволяет осуществлять аварийное восстановление и миграцию в Azure для локальных [виртуальных машин Hyper-V](hyper-v-azure-architecture.md), [виртуальных машин VMware](vmware-azure-architecture.md) и [физических серверов](physical-azure-architecture.md). Для всех сценариев репликации из локальной среды в Azure соответствующие данные отправляются и сохраняются в учетной записи хранения Azure. Во время репликации вам не требуется вносить плату за виртуальные машины. При выполнении отработки отказа в Azure служба Site Recovery автоматически создаст виртуальные машины IaaS Azure.

Site Recovery реплицирует данные в учетную запись хранения Azure или управляемый диск реплики в целевом регионе Azure через общедоступную конечную точку. Для использования ExpressRoute для Site Recovery трафика репликации можно использовать [пиринг Майкрософт](../expressroute/expressroute-circuit-peerings.md#microsoftpeering) или существующий [общедоступный пиринг](../expressroute/about-public-peering.md) (не рекомендуется для новых операций создания). Пиринг Майкрософт является рекомендуемым доменом маршрутизации для репликации. Обратите внимание, что репликация поддерживается через частный пиринг, только если [для хранилища включены закрытые конечные точки](hybrid-how-to-enable-replication-private-endpoints.md).

Убедитесь, что также выполнены [требования к сети](vmware-azure-configuration-server-requirements.md#network-requirements) для сервера конфигурации. Для оркестрации репликации в Site Recovery требуется возможность подключения сервера конфигурации к определенным URL-адресам. Для такого подключения невозможно использовать ExpressRoute. 

Если вы используете локальный прокси-сервер и хотите использовать ExpressRoute для трафика репликации, необходимо настроить список обхода прокси-сервера на сервере конфигурации и серверах обработки. Выполните инструкции, описанные ниже.

- Скачайте средство [PsExec отсюда, чтобы получить](/sysinternals/downloads/psexec) доступ к контексту пользователя системы.
- Откройте Internet Explorer в контексте системного пользователя, выполнив следующую командную строку: PsExec-s-i "%Програмфилес%\интернет Explorer\iexplore.exe"
- Добавление параметров прокси-сервера в IE
- В списке обхода добавьте URL-адрес службы хранилища Azure *. blob.core.windows.net.

Это обеспечит передачу только трафика репликации через ExpressRoute, в то время как обмен данными может проходить через прокси-сервер.

После отработки отказа виртуальных машин или серверов в виртуальную сеть Azure к ним можно получить доступ с помощью [частного пиринга](../expressroute/expressroute-circuit-peerings.md#privatepeering). 

На следующей схеме показан комбинированный сценарий: ![репликация из локальной среды в Azure с помощью ExpressRoute](./media/concepts-expressroute-with-site-recovery/site-recovery-with-expressroute.png)

## <a name="azure-to-azure-replication-with-expressroute"></a>Репликация из Azure в Azure с помощью ExpressRoute

Azure Site Recovery позволяет выполнять аварийное восстановление [виртуальных машин Azure](azure-to-azure-architecture.md). В зависимости от того, используют ли виртуальные машины Azure [управляемые диски Azure](../virtual-machines/managed-disks-overview.md), данные репликации отправляются в учетную запись хранения Azure или на реплику управляемого диска в целевом регионе Azure. Конечные точки репликации являются открытыми, но трафик репликации для виртуальных машин Azure по умолчанию не выходит в Интернет, независимо от того, в каком исходном регионе Azure создана виртуальная сеть. Вы можете переопределить системный маршрут Azure по умолчанию для префикса адреса 0.0.0.0/0, указав [настраиваемый маршрут](../virtual-network/virtual-networks-udr-overview.md#custom-routes), и перенаправить трафик виртуальных машин на локальный сетевой виртуальный модуль (NVA), но такая конфигурация не рекомендуется для репликации Site Recovery. При использовании настраиваемых маршрутов рекомендуется создать в виртуальной сети для хранилища [конечную точку службы для виртуальной сети](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage), чтобы трафик репликации не покидал границ Azure.

По умолчанию ExpressRoute для репликации при аварийном восстановлении виртуальной машины Azure не требуется. После отработки отказа виртуальных машин в целевой регион Azure вы можете обращаться к ним через [частный пиринг](../expressroute/expressroute-circuit-peerings.md#privatepeering). Обратите внимание, что цены на перенос данных применяются независимо от режима репликации данных в регионах Azure.

Если вы уже используете ExpressRoute для подключения из локального центра обработки данных к виртуальным машинам Azure в исходном регионе, можно запланировать переподключение ExpressRoute в целевом регионе отработки отказа. Вы можете использовать тот же канал ExpressRoute для подключения к целевому региону через новое подключение к виртуальной сети или использовать отдельный канал ExpressRoute и подключение для аварийного восстановления. Различные возможные сценарии описаны [здесь](azure-vm-disaster-recovery-with-expressroute.md#fail-over-azure-vms-when-using-expressroute).

Репликацию виртуальных машин Azure можно выполнить в любой регион Azure, входящий в тот же географический кластер, как описано [здесь](../site-recovery/azure-to-azure-support-matrix.md#region-support). Если выбранный целевой регион Azure находится не в том же геополитическом регионе, что и исходный регион, вам потребуется ExpressRoute ценовой категории "Премиум". Дополнительные сведения см. в разделе [Расположения ExpressRoute](../expressroute/expressroute-locations.md) и на странице [цен на ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [каналах ExpressRoute](../expressroute/expressroute-circuit-peerings.md).
- Дополнительные сведения о [доменах маршрутизации ExpressRoute](../expressroute/expressroute-circuit-peerings.md#peeringcompare).
- Дополнительные сведения о [расположениях ExpressRoute](../expressroute/expressroute-locations.md).
- Узнайте больше об аварийном восстановлении [виртуальных машин Azure с помощью ExpressRoute](azure-vm-disaster-recovery-with-expressroute.md).
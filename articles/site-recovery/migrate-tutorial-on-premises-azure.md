---
title: Перенос локальных компьютеров с помощью Миграции Azure
description: В этой статье описан общий процесс переноса локальных компьютеров в Azure и приводятся рекомендации по использованию Миграции Azure.
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/27/2020
ms.openlocfilehash: 5f5a0f65bfbde5ef8513780933cc570a4b5737df
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581133"
---
# <a name="migrate-on-premises-machines-to-azure"></a>Миграция локальных компьютеров в Azure

В этой статье описаны параметры для миграции локальных компьютеров в Azure. 

## <a name="migrate-with-azure-migrate"></a>Миграция с помощью средства "Миграция Azure"

Мы рекомендуем перенести компьютеры в Azure с помощью службы [Миграция Azure](../migrate/migrate-services-overview.md). Миграция Azure специально предназначена для переноса серверов. Миграция Azure — это центр управления для обнаружения, оценки и переноса локальных компьютеров в Azure.

Сведения о выполнении миграции с помощью средства "Миграция Azure" приведены по следующим ссылкам:

- [Узнайте о вариантах миграции](../migrate/server-migrate-overview.md) для виртуальных машин VMware, а затем перенесите виртуальные машины в Azure с помощью миграции [без агентов](../migrate/tutorial-migrate-vmware.md) или [миграции на основе агентов](../migrate/tutorial-migrate-vmware-agent.md).
- [Перенос виртуальных машин Hyper-V](../migrate/tutorial-migrate-hyper-v.md) в Azure.
- Перенос [физических серверов или других виртуальных машин](../migrate/tutorial-migrate-physical-virtual-machines.md), в том числе [экземпляров AWS](../migrate/tutorial-migrate-aws-virtual-machines.md) в Azure.

## <a name="migrate-with-site-recovery"></a>Миграция с помощью Site Recovery
Site Recovery следует использовать только для аварийного восстановления, а не для миграции.

Если вы уже используете Azure Site Recovery и хотите использовать его для миграции, выполните те же действия, что и при аварийном восстановлении.

- Виртуальные машины VMware: [подготовьте Azure](tutorial-prepare-azure.md) и [VMware](vmware-azure-tutorial-prepare-on-premises.md), запустите [репликацию компьютеров](vmware-azure-tutorial.md), [проверьте](tutorial-dr-drill-azure.md), что все работает, и [выполните отработку отказа](vmware-azure-tutorial-failover-failback.md).
- Виртуальные машины Hyper-V: [подготовьте Azure](tutorial-prepare-azure-for-hyperv.md) и [Hyper-V](hyper-v-prepare-on-premises-tutorial.md), запустите [репликацию компьютеров](hyper-v-azure-tutorial.md), [проверьте](tutorial-dr-drill-azure.md), что все работает, и [выполните отработку отказа](hyper-v-azure-failover-failback-tutorial.md).
- Физические серверы: [следуйте указаниям пошагового руководства](physical-azure-disaster-recovery.md), чтобы подготовить Azure и компьютеры к аварийному восстановлению и настроить репликацию.

> [!NOTE]
> При выполнении отработки отказа для аварийного восстановления в качестве последнего шага вы примените отработку отказа. При миграции локальных компьютеров параметр **Фиксация** не учитывается. Вместо этого следует выбрать параметр **Завершение миграции**. 

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Ознакомьтесь с часто задаваемыми вопросами](../migrate/resources-faq.md) о Миграции Azure.

  

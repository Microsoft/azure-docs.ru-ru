---
title: Руководство. Добавление сегмента сети NSX-T в Решении Azure VMware
description: Узнайте, как создать сегмент сети NSX-T, который будет использоваться для виртуальных машин в vCenter.
ms.topic: tutorial
ms.date: 03/13/2021
ms.openlocfilehash: 9125e552f9641a2d26b9584b66a4447f9c152161
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103462135"
---
# <a name="tutorial-add-a-network-segment-in-azure-vmware-solution"></a>Руководство по Добавление сегмента сети в Решении Azure VMware 

Виртуальные машины, созданные в vCenter, помещаются в сегменты сети, созданные в NSX-T, и они видимы в vCenter.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Переход в NSX-T Manager для добавления сегментов сети.
> * Добавление сегмента сети.
> * Просмотр нового сегмента сети в vCenter.

## <a name="prerequisites"></a>Предварительные требования

Частное облако Решения Azure VMware с доступом к интерфейсам vCenter и NSX-T Manager. Дополнительные сведения см. в статье о [конфигурации сети](tutorial-configure-networking.md).

## <a name="add-a-network-segment"></a>Добавление сегмента сети

[!INCLUDE [add-network-segment-steps](includes/add-network-segment-steps.md)]

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы создали сегмент сети NSX-T, который будет использоваться для виртуальных машин в vCenter. 

Теперь вы можете выполнить следующие действия: 

- [Создание и администрирование DHCP для Решения Azure VMware](manage-dhcp.md)
- [Создание библиотеки содержимого для развертывания виртуальных машин в Решении Azure VMware](deploy-vm-content-library.md) 
- [Настройка пиринга между локальной средой и частным облаком](tutorial-expressroute-global-reach-private-cloud.md)


<!-- LINKS - external-->

<!-- LINKS - internal -->

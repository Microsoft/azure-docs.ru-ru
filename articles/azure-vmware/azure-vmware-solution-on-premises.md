---
title: Подключение Решения Azure VMware к локальной среде
description: Узнайте, как подключить Решение Azure VMware к локальной среде.
ms.topic: tutorial
ms.date: 03/13/2021
ms.openlocfilehash: 0b26dc4756cb37544c2b2f8c5a75df0ac1a9d629
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103491798"
---
# <a name="connect-azure-vmware-solution-to-your-on-premises-environment"></a>Подключение Решения Azure VMware к локальной среде

В этой статье вы продолжите использовать [сведения, собранные при планировании](production-ready-deployment-steps.md), чтобы подключить Решение Azure VMware к локальной среде.

Прежде чем приступать к подключению Решения Azure VMware к локальной среде, необходимо иметь следующее:

- цепь ExpressRoute из локальной среды в Azure;
- блок неперекрывающихся сетевых адресов CIDR /29 для пиринга ExpressRoute Global Reach, который вы определили как часть [этапа планирования](production-ready-deployment-steps.md).

>[!NOTE]
> Вы можете подключиться через VPN, но этот метод не рассматривается в этом кратком руководстве.

## <a name="establish-an-expressroute-global-reach-connection"></a>Установка подключения ExpressRoute Global Reach

Чтобы установить локальное подключение к частному облаку Решения Azure VMware с помощью ExpressRoute Global Reach, следуйте указаниям в [этом учебнике](tutorial-expressroute-global-reach-private-cloud.md).

На схеме ниже показано, как установить подключение.

:::image type="content" source="media/pre-deployment/azure-vmware-solution-on-premises-diagram.png" alt-text="Схема локального сетевого подключения ExpressRoute Global Reach." lightbox="media/pre-deployment/azure-vmware-solution-on-premises-diagram.png" border="false":::

## <a name="verify-on-premises-network-connectivity"></a>Проверка локального сетевого подключения

Теперь вы должны увидеть в **локальном пограничном маршрутизаторе**, в котором подключается ExpressRoute, сегменты сети NSX-T и сегменты управления Решением VMware Azure.

>[!IMPORTANT]
>У всех разная среда, и некоторым нужно будет разрешить, чтобы эти маршруты распространялись обратно в локальную сеть.  

В некоторых средах брандмауэры защищают цепь ExpressRoute.  Если брандмауэры отсутствуют и удаление маршрутов не выполняется, проверьте связь с сервером vCenter Решения Azure VMware или [виртуальной машиной в сегменте NSX-T](deploy-azure-vmware-solution.md#add-a-vm-on-the-nsx-t-network-segment) из локальной среды. Кроме того, с виртуальной машины в сегменте NSX-T можно получить доступ к ресурсам в локальной среде.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующему разделу для развертывания и настройки VMware HCX.

> [!div class="nextstepaction"]
> [Развертывание и настройка VMware HCX](tutorial-deploy-vmware-hcx.md)
---
title: Создание узла бастиона с помощью Azure PowerShell | Документация Майкрософт
description: В этой статье вы узнаете, как создать узел бастиона для Azure.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 10/14/2020
ms.author: cherylmc
ms.openlocfilehash: ba2716613a0e950cbae5c65add410ac8a8b38955
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92077732"
---
# <a name="create-an-azure-bastion-host-using-azure-powershell"></a>Создание узла-бастиона Azure с помощью Azure PowerShell

В этой статье показано, как создать узел бастиона для Azure с помощью PowerShell. Когда вы подготавливаете службу Azure бастиона в виртуальной сети, работа по протоколу RDP и SSH доступна для всех виртуальных машин в той же виртуальной сети. Бастион Azure развертывается на уровне виртуальной сети, а не подписки, учетной записи или виртуальной машины.

При необходимости можно создать узел бастиона для Azure с помощью [портал Azure](./tutorial-create-host-portal.md).

## <a name="prerequisites"></a>Предварительные условия

Убедитесь в том, что у вас уже есть подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial).

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="create-a-bastion-host"></a><a name="createhost"></a>Создание узла-бастиона

Этот раздел поможет вам создать новый ресурс Azure бастиона с помощью Azure PowerShell.

1. Создайте виртуальную сеть и подсеть бастиона для Azure. Необходимо создать подсеть бастиона для Azure, используя значение Name **азуребастионсубнет**. По этому значению Azure сможет определить, в какую подсеть следует развертывать ресурсы типа "Бастион". Она отличается от подсети шлюза. Необходимо использовать подсеть как минимум/27 или более крупную подсеть (/27,/26 и т. д.). Создайте **азуребастионсубнет** без таблиц или делегирований маршрутов. Если вы используете группы безопасности сети в **азуребастионсубнет**, см. статью [Работа с группы безопасности сети](bastion-nsg.md) .

   ```azurepowershell-interactive
   $subnetName = "AzureBastionSubnet"
   $subnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24
   $vnet = New-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myBastionRG" -Location "westeurope" -AddressPrefix 10.0.0.0/16 -Subnet $subnet
   ```

2. Создайте общедоступный IP-адрес для Бастиона Azure. Общедоступные IP-адреса — это общедоступный IP-адрес ресурса бастиона, к которому будут обращаться протоколы RDP/SSH (через порт 443). Общедоступный IP-адрес должен находиться в том же регионе, что и создаваемый ресурс типа "Бастион".

   ```azurepowershell-interactive
   $publicip = New-AzPublicIpAddress -ResourceGroupName "myBastionRG" -name "myPublicIP" -location "westeurope" -AllocationMethod Static -Sku Standard
   ```

3. Создайте новый ресурс Azure бастиона в Азуребастионсубнет виртуальной сети. Создание и развертывание ресурса бастиона занимает около 5 минут.

   ```azurepowershell-interactive
   $bastion = New-AzBastion -ResourceGroupName "myBastionRG" -Name "myBastion" -PublicIpAddress $publicip -VirtualNetwork $vnet
   ```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в статье [вопросы и ответы по бастиона](bastion-faq.md) .
* Сведения об использовании групп безопасности сети с подсетью Бастиона Azure см. в статье [Работа с группами безопасности сети](bastion-nsg.md).

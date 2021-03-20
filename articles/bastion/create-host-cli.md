---
title: Создание узла бастиона с помощью Azure CLI | Azure бастиона
description: В этой статье вы узнаете, как создать и удалить узел бастиона.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 10/14/2020
ms.author: cherylmc
ms.openlocfilehash: e4f0a3ea85fa479826fe9e666df2b18f8dba7ce3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92079128"
---
# <a name="create-an-azure-bastion-host-using-azure-cli"></a>Создание узла-бастиона Azure с помощью Azure CLI

В этой статье показано, как создать узел бастиона для Azure с помощью Azure CLI. Когда вы подготавливаете службу Azure бастиона в виртуальной сети, работа по протоколу RDP и SSH доступна для всех виртуальных машин в той же виртуальной сети. Бастион Azure развертывается на уровне виртуальной сети, а не подписки, учетной записи или виртуальной машины.

При необходимости можно создать узел бастиона для Azure с помощью [портал Azure](./tutorial-create-host-portal.md)или [Azure PowerShell](bastion-create-host-powershell.md).

## <a name="prerequisites"></a>Предварительные условия

Убедитесь в том, что у вас уже есть подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial).

[!INCLUDE [Cloud Shell CLI](../../includes/vpn-gateway-cloud-shell-cli.md)]

## <a name="create-a-bastion-host"></a><a name="createhost"></a>Создание узла-бастиона

Этот раздел поможет вам создать новый ресурс Azure бастиона с помощью Azure CLI.

> [!NOTE]
> Как показано в примерах, используйте `--location` параметр `--resource-group` для каждой команды, чтобы убедиться, что ресурсы развертываются вместе.

1. Создайте виртуальную сеть и подсеть бастиона для Azure. Необходимо создать подсеть бастиона для Azure, используя значение Name **азуребастионсубнет**. По этому значению Azure сможет определить, в какую подсеть следует развертывать ресурсы типа "Бастион". Она отличается от подсети шлюза. Необходимо использовать подсеть как минимум/27 или более крупную подсеть (/27,/26 и т. д.). Создайте **азуребастионсубнет** без таблиц или делегирований маршрутов. Если вы используете группы безопасности сети в **азуребастионсубнет**, см. статью [Работа с группы безопасности сети](bastion-nsg.md) .

   ```azurecli-interactive
   az network vnet create --resource-group MyResourceGroup --name MyVnet --address-prefix 10.0.0.0/16 --subnet-name AzureBastionSubnet --subnet-prefix 10.0.0.0/24 --location northeurope
   ```

2. Создайте общедоступный IP-адрес для Бастиона Azure. Общедоступные IP-адреса — это общедоступный IP-адрес ресурса бастиона, к которому будут обращаться протоколы RDP/SSH (через порт 443). Общедоступный IP-адрес должен находиться в том же регионе, что и создаваемый ресурс типа "Бастион".

   ```azurecli-interactive
   az network public-ip create --resource-group MyResourceGroup --name MyIp --sku Standard --location northeurope
   ```

3. Создайте новый ресурс Azure бастиона в Азуребастионсубнет виртуальной сети. Создание и развертывание ресурса бастиона занимает около 5 минут.

   ```azurecli-interactive
   az network bastion create --name MyBastion --public-ip-address MyIp --resource-group MyResourceGroup --vnet-name MyVnet --location northeurope
   ```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в статье [вопросы и ответы по бастиона](bastion-faq.md) .
* Сведения об использовании групп безопасности сети с подсетью Бастиона Azure см. в статье [Работа с группами безопасности сети](bastion-nsg.md).

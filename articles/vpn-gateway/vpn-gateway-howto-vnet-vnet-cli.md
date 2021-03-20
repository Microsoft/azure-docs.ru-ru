---
title: 'Подключение виртуальной сети к виртуальной сети с помощью подключения между виртуальными сетями: Azure CLI'
description: Установка подключения "виртуальная сеть — виртуальная сеть" между виртуальными сетями с помощью Azure CLI.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.custom: devx-track-azurecli
ms.openlocfilehash: b9502f3fbd50aad756e15daa4db1badda2abf9ab
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94660072"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-azure-cli"></a>Настройка подключения VPN-шлюза между виртуальными сетями с помощью Azure CLI

Эта статья поможет установить подключение "виртуальная сеть — виртуальная сеть" между виртуальными сетями. Виртуальные сети могут относиться к одному или разным регионам и к одной или разным подпискам. При подключении виртуальных сетей из разных подписок подписки не обязательно должны быть связаны с одним и тем же клиентом Active Directory.

Приведенные в этой статье инструкции относятся к модели развертывания с помощью Resource Manager и выполняются с помощью Azure CLI. Эту конфигурацию также можно создать с помощью разных средств или моделей развертывания, выбрав вариант из следующего списка:

> [!div class="op_single_selector"]
> * [Портал Azure](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Портал Azure (классический)](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Подключение с использованием разных моделей развертывания — портал Azure](vpn-gateway-connect-different-deployment-models-portal.md)
> * [Подключение с использованием разных моделей развертывания — PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about-connecting-vnets"></a><a name="about"></a>Сведения о подключении виртуальных сетей

Подключить виртуальные сети можно несколькими способами. В следующих разделах описаны различные способы подключения виртуальных сетей.

### <a name="vnet-to-vnet"></a>Подключение типа "виртуальная сеть — виртуальная сеть"

Чтобы легко подключать виртуальные сети, рекомендуем настроить подключение "виртуальная сеть — виртуальная сеть". Подключение "виртуальная сеть — виртуальная сеть" между виртуальными сетями похоже на создание подключения "сеть — сеть" (IPsec) к локальному расположению. В обоих типах подключений применяется VPN-шлюз для создания защищенного туннеля, использующего IPsec/IKE. При обмене данными оба типа подключений работают одинаково. Различие между этими типами заключается в способе настройки шлюза локальной сети. При создании подключения "виртуальная сеть — виртуальная сеть" диапазон адресов для шлюза локальной сети не отображается. Он создается и заполняется автоматически. Если вы обновите диапазон адресов для одной виртуальной сети, другая виртуальная сеть автоматически определит маршрут к обновленному диапазону адресов. Как правило, намного проще и быстрее создать подключение "виртуальная сеть — виртуальная сеть", чем подключение "сеть — сеть" между виртуальными сетями.

### <a name="connecting-vnets-using-site-to-site-ipsec-steps"></a>Подключение виртуальных сетей по типу "сеть — сеть" (IPsec)

При работе со сложной конфигурацией сети можно подключить виртуальные сети с помощью подключения [сеть — сеть](vpn-gateway-howto-site-to-site-resource-manager-cli.md), а не "виртуальная сеть — виртуальная сеть". Используя подключение "сеть — сеть", вы вручную создаете и настраиваете локальные сетевые шлюзы. Шлюз локальной сети для каждой виртуальной сети воспринимает другую виртуальную сеть как локальный сайт. Это позволит вам указать дополнительный диапазон адресов для шлюза локальной сети, чтобы маршрутизировать трафик. Если диапазон адресов для виртуальной сети изменится, вам потребуется вручную обновить соответствующий шлюз локальной сети, чтобы отразить эти изменения. Он не обновляется автоматически.

### <a name="vnet-peering"></a>Пиринговая связь между виртуальными сетями

Вы можете подключить виртуальные сети с помощью пиринга виртуальных сетей. При пиринге виртуальных сетей не используется VPN-шлюз и применяются другие ограничения. Кроме того, [цены на пиринг виртуальных сетей](https://azure.microsoft.com/pricing/details/virtual-network) рассчитываются не так, как [цены на VPN-шлюз "виртуальная сеть — виртуальная сеть"](https://azure.microsoft.com/pricing/details/vpn-gateway). Дополнительную информацию см. в статье [Пиринговая связь между виртуальными сетями](../virtual-network/virtual-network-peering-overview.md).

## <a name="why-create-a-vnet-to-vnet-connection"></a><a name="why"></a>Зачем создавать подключение "виртуальная сеть — виртуальная сеть"?

Возможно, вам потребуется подключить виртуальные сети с помощью подключения "виртуальная сеть — виртуальная сеть" по следующим причинам.

* **Межрегиональная географическая избыточность и географическое присутствие**

  * Вы можете настроить собственную георепликацию или синхронизацию с безопасной связью без прохождения трафика через конечные точки с выходом в Интернет.
  * С помощью Azure Load Balancer и диспетчера трафика можно настроить высокодоступную рабочую нагрузку с геоизбыточностью в нескольких регионах Azure. Одним из важных примеров является настройка SQL Always On с группами доступности, распределенными между несколькими регионами Azure.
* **Региональные многоуровневые приложения с изоляцией или административной границей**

  * В одном регионе можно настроить многоуровневые приложения с несколькими виртуальными сетями, которые связаны друг с другом из-за требований к изоляции или административных требований.

Подключение типа "виртуальная сеть — виртуальная сеть" можно комбинировать с многосайтовыми конфигурациями. Это позволяет настраивать топологии сети, совмещающие распределенные подключения с подключениями между виртуальными сетями.

## <a name="which-vnet-to-vnet-steps-should-i-use"></a><a name="steps"></a>Какие действия по созданию подключения "виртуальная сеть — виртуальная сеть" следует использовать?

В этой статье приведены два разных набора действий по созданию подключения "виртуальная сеть — виртуальная сеть". Один предназначен для [виртуальных сетей в одной подписке](#samesub), а другой — для [виртуальных сетей в разных подписках](#difsub). 

Для этого вы можете объединить конфигурации или выбрать ту, с которой предпочитаете работать. Во всех конфигурациях используется подключение "виртуальная сеть — виртуальная сеть". Сетевой трафик передается между виртуальными сетями, которые напрямую подключены друг к другу. В этом руководстве трафик не перенаправляется из TestVNet4 в TestVNet5.

* [Виртуальных сетей, находящиеся в одной подписке:](#samesub) В действиях, описанных в этой конфигурации, используются TestVNet1 и TestVNet4.

  ![Схема, в которой показаны действия V NET-V для поточностей с V, которые находятся в одной подписке.](./media/vpn-gateway-howto-vnet-vnet-cli/v2vrmps.png)

* [Виртуальных сетей, которые находятся в разных подписках:](#difsub) В действиях, описанных в этой конфигурации, используются TestVNet1 и TestVNet5.

  ![Схема подключения между виртуальными сетями](./media/vpn-gateway-howto-vnet-vnet-cli/v2vdiffsub.png)


## <a name="connect-vnets-that-are-in-the-same-subscription"></a><a name="samesub"></a>Подключение виртуальных сетей из одной подписки

### <a name="before-you-begin"></a>Перед началом

Перед началом работы установите последнюю версию команд интерфейса командной строки (версию 2.0 или более позднюю). Сведения об установке команд CLI см. в руководстве по [установке Azure](/cli/azure/install-azure-cli).

### <a name="plan-your-ip-address-ranges"></a><a name="Plan"></a>Планирование диапазонов IP-адресов

Далее мы создадим две виртуальные сети с соответствующими конфигурациями и подсетями шлюзов. Затем мы создадим VPN-подключение между двумя виртуальными сетями. В конфигурации сети важно задать диапазоны IP-адресов. Имейте в виду, необходимо убедиться в том, что ни один из диапазонов виртуальных сетей или диапазонов локальных сетей никак не перекрываются. В этих примерах мы не включаем DNS-сервер. Сведения о разрешении имен виртуальных машин см. в [этой статье](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

В примерах мы используем следующие значения:

**Значения для TestVNet1:**

* Имя виртуальной сети: TestVNet1
* Группа ресурсов: TestRG1
* Расположение: восточная часть США
* TestVNet1: 10.11.0.0/16 и 10.12.0.0/16
* FrontEnd: 10.11.0.0/24
* BackEnd: 10.12.0.0/24
* GatewaySubnet: 10.12.255.0/27
* Имя шлюза: VNet1GW
* Общедоступный IP-адрес: VNet1GWIP
* Тип VPN: RouteBased
* Подключение (1–4): VNet1toVNet4
* Подключение (1–5): VNet1toVNet5 (для виртуальных сетей в разных подписках)

**Значения для TestVNet4:**

* Имя виртуальной сети: TestVNet4
* TestVNet2: 10.41.0.0/16 и 10.42.0.0/16
* FrontEnd: 10.41.0.0/24
* BackEnd: 10.42.0.0/24
* GatewaySubnet: 10.42.255.0/27
* Группа ресурсов: TestRG4
* Расположение: западная часть США
* Имя шлюза: VNet4GW
* Общедоступный IP-адрес: VNet4GWIP
* Тип VPN: RouteBased
* Подключение: VNet4toVNet1

### <a name="step-1---connect-to-your-subscription"></a><a name="Connect"></a>Шаг 1. Подключение к подписке

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-numbers-include.md)]

### <a name="step-2---create-and-configure-testvnet1"></a><a name="TestVNet1"></a>Шаг 2. Создание и настройка TestVNet1

1. Создайте группу ресурсов.

   ```azurecli
   az group create -n TestRG1  -l eastus
   ```
2. Создайте виртуальную сеть TestVNet1 и ее подсети. Пример ниже создает виртуальная сеть TestVNet1 и подсеть FrontEnd.

   ```azurecli
   az network vnet create -n TestVNet1 -g TestRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24
   ```
3. Создайте дополнительное адресное пространство для внутренней подсети. Обратите внимание, что на этом шаге мы указываем созданное ранее адресное пространство, а также дополнительное адресное пространство, которое нужно добавить. Это связано с тем, что команда [az network vnet update](/cli/azure/network/vnet) перезаписывает предыдущие параметры. При использовании этой команды обязательно укажите все префиксы адресов.

   ```azurecli
   az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestRG1
   ```
4. Создайте внутреннюю подсеть.
  
   ```azurecli
   az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestRG1 --address-prefix 10.12.0.0/24 
   ```
5. Создайте подсеть шлюза. Обратите внимание, что подсети шлюза присвоено имя GatewaySubnet. Это обязательное имя. В этом примере в подсети шлюза используется длина префикса /27. Хотя можно создать подсеть шлюза размером /29, рекомендуется создать подсеть большего размера, включающую несколько адресов, выбрав по крайней мере значение /28 или /27. Таким образом, у вас будет достаточно адресов, чтобы добавить дополнительные конфигурации в будущем.

   ```azurecli 
   az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestRG1 --address-prefix 10.12.255.0/27
   ```
6. Запросите выделение общедоступного IP-адреса для шлюза, который будет создан для виртуальной сети. Учтите, что параметр AllocationMethod является динамическим. Указать необходимый IP-адрес нельзя. Он выделяется для шлюза динамически.

   ```azurecli
   az network public-ip create -n VNet1GWIP -g TestRG1 --allocation-method Dynamic
   ```
7. Создайте шлюз для виртуальной сети TestVNet1. Для подключений типа VNet-to-VNet необходимо использовать тип VPN RouteBased. При выполнении этой команды с использованием параметра --no-wait вы не увидите ответа или выходных данных. Этот параметр позволяет создать шлюз в фоновом режиме. Это не означает, что создание VPN-шлюза завершается немедленно. Этот процесс часто занимает 45 минут и более, в зависимости от используемого номера SKU-шлюза.

   ```azurecli
   az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address VNet1GWIP -g TestRG1 --vnet TestVNet1 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-3---create-and-configure-testvnet4"></a><a name="TestVNet4"></a>Шаг 3. Создание и настройка TestVNet4

1. Создайте группу ресурсов.

   ```azurecli
   az group create -n TestRG4  -l westus
   ```
2. Создайте TestVNet4.

   ```azurecli
   az network vnet create -n TestVNet4 -g TestRG4 --address-prefix 10.41.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.41.0.0/24
   ```

3. Создайте дополнительную подсеть для TestVNet4.

   ```azurecli
   az network vnet update -n TestVNet4 --address-prefixes 10.41.0.0/16 10.42.0.0/16 -g TestRG4 
   az network vnet subnet create --vnet-name TestVNet4 -n BackEnd -g TestRG4 --address-prefix 10.42.0.0/24 
   ```
4. Создайте подсеть шлюза.

   ```azurecli
   az network vnet subnet create --vnet-name TestVNet4 -n GatewaySubnet -g TestRG4 --address-prefix 10.42.255.0/27
   ```
5. Запросите общедоступный IP-адрес.

   ```azurecli
   az network public-ip create -n VNet4GWIP -g TestRG4 --allocation-method Dynamic
   ```
6. Создайте шлюз виртуальной сети TestVNet4.

   ```azurecli
   az network vnet-gateway create -n VNet4GW -l westus --public-ip-address VNet4GWIP -g TestRG4 --vnet TestVNet4 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-4---create-the-connections"></a><a name="createconnect"></a>Шаг 4. Создание подключений

Теперь у вас есть две виртуальные сети с VPN-шлюзами. Далее необходимо установить подключения VPN-шлюза между шлюзами виртуальной сети. Если вы использовали пример выше, шлюзы виртуальной сети находятся в разных группах ресурсов. В этом случае при установке подключения необходимо определить и задать идентификаторы ресурсов для каждого шлюза. Если виртуальные сети находятся в той же группе ресурсов, используйте [второй набор инструкций](#samerg), так как указывать идентификаторы ресурсов не нужно.

### <a name="to-connect-vnets-that-reside-in-different-resource-groups"></a><a name="diffrg"></a>Подключение виртуальных сетей, расположенных в разных группах ресурсов

1. Получите идентификатор ресурса VNet1GW из выходных данных следующей команды:

   ```azurecli
   az network vnet-gateway show -n VNet1GW -g TestRG1
   ```

   В выходных данных найдите строку "id":. Значения в кавычках понадобятся при создании подключения в следующем разделе. Скопируйте эти значения в текстовый редактор, например Блокнот, чтобы их можно было быстро вставить при создании подключения.

   Выходные данные примера:

   ```
   "activeActive": false, 
   "bgpSettings": { 
    "asn": 65515, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
   }, 
   "enableBgp": false, 
   "etag": "W/\"ecb42bc5-c176-44e1-802f-b0ce2962ac04\"", 
   "gatewayDefaultSite": null, 
   "gatewayType": "Vpn", 
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW", 
   "ipConfigurations":
   ```

   Скопируйте значения в кавычках после **"id":**.

   ```
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
   ```

2. Получите идентификатор ресурса VNet4GW и скопируйте эти значения в текстовый редактор.

   ```azurecli
   az network vnet-gateway show -n VNet4GW -g TestRG4
   ```

3. Создайте подключение между TestVNet1 и TestVNet4. На этом шаге вы создадите подключение между TestVNet1 и TestVNet4. В этих примерах вы увидите ссылки на общий ключ. Можно использовать собственные значения для общего ключа. Важно, чтобы общий ключ в обоих подключениях был одинаковым. Создание подключения занимает некоторое время.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW 
   ```
4. Создайте подключение между TestVNet4 и TestVNet1. Этот шаг аналогичен приведенному выше, за исключением того, что создается подключение из сети TestVNet4 в сеть TestVNet1. Убедитесь, что общие ключи совпадают. Установка подключения занимает несколько минут.

   ```azurecli
   az network vpn-connection create -n VNet4ToVNet1 -g TestRG4 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW -l westus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1G
   ```
5. Проверьте подключения. Дополнительные сведения см. в разделе [Проверка подключений](#verify).

### <a name="to-connect-vnets-that-reside-in-the-same-resource-group"></a><a name="samerg"></a>Подключение виртуальных сетей, расположенных в той же группе ресурсов

1. Создайте подключение между TestVNet1 и TestVNet4. На этом шаге вы создадите подключение между TestVNet1 и TestVNet4. Обратите внимание, что группы ресурсов в этих примерах совпадают. В этих примерах вы также увидите ссылки на общий ключ. Вы можете использовать собственные значения общего ключа, но они должны совпадать для обоих подключений. Создание подключения занимает некоторое время.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet4GW
   ```
2. Создайте подключение между TestVNet4 и TestVNet1. Этот шаг аналогичен приведенному выше, за исключением того, что создается подключение из сети TestVNet4 в сеть TestVNet1. Убедитесь, что общие ключи совпадают. Установка подключения занимает несколько минут.

   ```azurecli
   az network vpn-connection create -n VNet4ToVNet1 -g TestRG1 --vnet-gateway1 VNet4GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet1GW
   ```
3. Проверьте подключения. Дополнительные сведения см. в разделе [Проверка подключений](#verify).

## <a name="connect-vnets-that-are-in-different-subscriptions"></a><a name="difsub"></a>Подключение виртуальных сетей из разных подписок

В этом сценарии мы создадим подключение между TestVNet1 и TestVNet5. Виртуальные сети находятся в разных подписках. Подписки не обязательно должны быть связаны с одним и тем же клиентом Active Directory. В этой конфигурации добавляется дополнительное подключение между виртуальными сетями для подключения TestVNet1 к TestVNet5.

### <a name="step-5---create-and-configure-testvnet1"></a><a name="TestVNet1diff"></a>Шаг 5. Создание и настройка TestVNet1

Перед выполнением этих инструкций следует выполнить действия в предыдущих разделах. Чтобы создать и настроить TestVNet1 и VPN-шлюз для TestVNet1, необходимо выполнить [Шаг 1](#Connect) и [Шаг 2](#TestVNet1) . В этой конфигурации не нужно создавать TestVNet4 из предыдущего раздела. Но если вы создадите эту виртуальную сеть, она не помешает при выполнении этих шагов. Выполнив шаг 1 и 2, перейдите к шагу 6.

### <a name="step-6---verify-the-ip-address-ranges"></a><a name="verifyranges"></a>Шаг 6. Проверка диапазонов IP-адресов

При создании дополнительных подключений важно проследить, чтобы пространство IP-адресов новой виртуальной сети не пересекалось с диапазонами других виртуальных сетей или диапазонами других локальных сетевых шлюзов. В этом упражнении используйте следующие значения для сети TestVNet5.

**Значения для TestVNet5:**

* Имя виртуальной сети: TestVNet5
* Группа ресурсов: TestRG5
* Расположение: Восточная Япония
* TestVNet5: 10.51.0.0/16 и 10.52.0.0/16
* FrontEnd: 10.51.0.0/24
* BackEnd: 10.52.0.0/24
* GatewaySubnet: 10.52.255.0.0/27
* Имя шлюза: VNet5GW
* Общедоступный IP-адрес: VNet5GWIP
* Тип VPN: RouteBased
* Подключение: VNet5toVNet1
* Тип подключения: VNet2VNet

### <a name="step-7---create-and-configure-testvnet5"></a><a name="TestVNet5"></a>Шаг 7. Создание и настройка TestVNet5

Это действие необходимо выполнить в контексте новой подписки — подписки 5. Эту часть может выполнить администратор в другой организации, которой принадлежит подписка. Для переключения между подписками используйте `az account list --all` для перечисления подписок, доступных для вашей учетной записи, а затем используйте `az account set --subscription <subscriptionID>` для переключения на подписку, которую вы хотите использовать.

1. Проверьте подключение к подписке 5, а затем создайте группу ресурсов.

   ```azurecli
   az group create -n TestRG5  -l japaneast
   ```
2. Создайте TestVNet5.

   ```azurecli
   az network vnet create -n TestVNet5 -g TestRG5 --address-prefix 10.51.0.0/16 -l japaneast --subnet-name FrontEnd --subnet-prefix 10.51.0.0/24
   ```

3. Добавьте подсети.

   ```azurecli
   az network vnet update -n TestVNet5 --address-prefixes 10.51.0.0/16 10.52.0.0/16 -g TestRG5
   az network vnet subnet create --vnet-name TestVNet5 -n BackEnd -g TestRG5 --address-prefix 10.52.0.0/24
   ```

4. Добавьте подсеть шлюза.

   ```azurecli
   az network vnet subnet create --vnet-name TestVNet5 -n GatewaySubnet -g TestRG5 --address-prefix 10.52.255.0/27
   ```

5. Запросите общедоступный IP-адрес.

   ```azurecli
   az network public-ip create -n VNet5GWIP -g TestRG5 --allocation-method Dynamic
   ```
6. Создание шлюза TestVNet5

   ```azurecli
   az network vnet-gateway create -n VNet5GW -l japaneast --public-ip-address VNet5GWIP -g TestRG5 --vnet TestVNet5 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
   ```

### <a name="step-8---create-the-connections"></a><a name="connections5"></a>Шаг 8. Создание подключений

Так как шлюзы находятся в разных подписках, мы разделили этот шаг на два сеанса интерфейса командной строки, обозначенные **[Подписка 1]** и **[Подписка 5]**. Для переключения между подписками используйте `az account list --all` для перечисления подписок, доступных для вашей учетной записи, а затем используйте `az account set --subscription <subscriptionID>` для переключения на подписку, которую вы хотите использовать.

1. **[Подписка 1].** Войдите в систему и подключитесь к подписке 1. Чтобы получить имя и идентификатор шлюза из выходных данных, выполните следующую команду:

   ```azurecli
   az network vnet-gateway show -n VNet1GW -g TestRG1
   ```

   Скопируйте значение "id":. Отправьте идентификатор и имя шлюза виртуальной сети (VNet1GW) администратору подписки 5 по электронной почте или другим способом.

   Выходные данные примера:

   ```
   "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
   ```

2. **[Подписка 5].** Войдите в систему и подключитесь к подписке 5. Чтобы получить имя и идентификатор шлюза из выходных данных, выполните следующую команду:

   ```azurecli
   az network vnet-gateway show -n VNet5GW -g TestRG5
   ```

   Скопируйте значение "id":. Отправьте идентификатор и имя шлюза виртуальной сети (VNet5GW) администратору подписки 1 по электронной почте или другим способом.

3. **[Подписка 1].** На этом шаге вы создадите подключение между TestVNet1 и TestVNet5. Вы можете использовать собственные значения общего ключа, но они должны совпадать для обоих подключений. Создание подключения может занять некоторое время. Обязательно подключитесь к подписке 1.

   ```azurecli
   az network vpn-connection create -n VNet1ToVNet5 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW
   ```

4. **[Подписка 5].** Этот шаг аналогичен приведенному выше, за исключением того, что создается подключение из сети TestVNet5 в сеть TestVNet1. Убедитесь, что общие ключи совпадают и что вы подключились к подписке 5.

   ```azurecli
   az network vpn-connection create -n VNet5ToVNet1 -g TestRG5 --vnet-gateway1 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW -l japaneast --shared-key "eeffgg" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
   ```

## <a name="verify-the-connections"></a><a name="verify"></a>Проверка подключений
[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>Часто задаваемые вопросы о подключениях типа "виртуальная сеть — виртуальная сеть"
[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Установив подключение, можно добавить виртуальные машины в виртуальные сети. Дополнительные сведения см. в [документации по виртуальным машинам](../index.yml).
* Сведения о BGP см. в статьях [Обзор использования BGP с VPN-шлюзами Azure](vpn-gateway-bgp-overview.md) и [Настройка BGP на VPN-шлюзах Azure с помощью Azure Resource Manager и PowerShell](vpn-gateway-bgp-resource-manager-ps.md).
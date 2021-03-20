---
title: Использование внутренней службы DNS для разрешения имен виртуальных машин с помощью Azure CLI
description: Как создать виртуальные сетевые карты и использовать внутренние DNS для разрешения имен виртуальных машин в Azure с помощью Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/16/2017
ms.author: cynthn
ms.openlocfilehash: 3d68ac7aa9927e62011c58b17139d7232ce4a10c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98200759"
---
# <a name="create-virtual-network-interface-cards-and-use-internal-dns-for-vm-name-resolution-on-azure"></a>Создание виртуальных сетевых карт и использование внутренних DNS-имен для разрешения имен виртуальных машин в Azure

В этой статье показано, как задать статические внутренние DNS-имена для виртуальных машин Linux, использующих виртуальные сетевые карты и DNS-имена меток, с помощью Azure CLI. Статические имена DNS используются для постоянных инфраструктурных служб, таких как сервер сборки Jenkins, который используется для этого поддержания документа, или сервер Git.

Для этого необходимы следующие компоненты:

* [Учетная запись Azure;](https://azure.microsoft.com/pricing/free-trial/)
* [файлы открытого и закрытого ключа SSH](mac-create-ssh-keys.md).

## <a name="quick-commands"></a>Быстрые команды
Если вам необходимо быстро выполнить задачу, в следующем разделе описаны нужные команды. Более подробные сведения и контекст для каждого шага можно найти в оставшейся части документа, начиная с [этой статьи](#detailed-walkthrough). Чтобы выполнить эти действия, нужно установить последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войти в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index).

Предварительные требования: группа ресурсов, виртуальная сеть и подсеть, группа безопасности сети с разрешенными входящими подключениями SSH.

### <a name="create-a-virtual-network-interface-card-with-a-static-internal-dns-name"></a>Создание виртуальной сетевой карты со статическим внутренним DNS-именем
Создайте виртуальную сетевую карту с помощью команды [az network nic create](/cli/azure/network/nic). Флаг `--internal-dns-name` интерфейса командной строки используется для установки метки DNS, которая обеспечивает статическое DNS-имя для виртуальной сетевой карты. Следующий пример создает виртуальную сетевую карту `myNic`, подключает ее к виртуальной сети `myVnet` и создает внутреннюю запись DNS-имени `jenkins`.

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --internal-dns-name jenkins
```

### <a name="deploy-a-vm-and-connect-the-vnic"></a>Развертывание виртуальной машины и подключение виртуальной сетевой карты
Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). Флаг `--nics` позволяет подключить виртуальную сетевую карту к виртуальной машине при развертывании в Azure. Следующий пример создает виртуальную машину `myVM`, использующую Управляемые диски Azure, и подключает к ней виртуальную сетевую карту `myNic`, созданную на предыдущем шаге.

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="detailed-walkthrough"></a>Подробное пошаговое руководство

Для полной непрерывной интеграции и развертывания инфраструктуры в Azure определенные серверы должны быть статическими или долговременными серверами. Рекомендуется, чтобы ресурсы Azure, такие как виртуальные сети и группы безопасности сети, были статическими, хранились в течение продолжительного времени и редко развертывались. После развертывания виртуальной сети ее можно повторно использовать при новых развертываниях. Инфраструктура при этом не пострадает. Позже в эту виртуальную сеть можно будет добавить сервер репозитория Git или сервер автоматизации Jenkins, обеспечивающий непрерывные интеграцию и развертывание для сред разработки или тестовых сред.  

Внутренние имена DNS можно сопоставлять только внутри виртуальной сети Azure. Поскольку DNS-имена являются внутренними, они не сопоставляются с адресами в Интернете, обеспечивая дополнительную безопасность инфраструктуры.

В следующих примерах замените имена параметров собственными значениями. Используемые имена параметров: `myResourceGroup`, `myNic` и `myVM`.

## <a name="create-the-resource-group"></a>Создание группы ресурсов
Сначала создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В следующем примере создается группа ресурсов с именем `myResourceGroup` в расположении `westus`:

```azurecli
az group create --name myResourceGroup --location westus
```

## <a name="create-the-virtual-network"></a>Создание виртуальной сети

Следующий шаг — создание виртуальной сети Azure для запуска виртуальных машин. Для этого пошагового руководства виртуальная сеть содержит одну подсеть. Дополнительные сведения о виртуальных сетях Azure см. в разделе о [создании виртуальной сети](../../virtual-network/manage-virtual-network.md#create-a-virtual-network). 

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet). В следующем примере создается виртуальная сеть `myVnet` и подсеть `mySubnet`.

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

## <a name="create-the-network-security-group"></a>Создание группы безопасности сети
На уровне сети группы безопасности сети Azure эквивалентны брандмауэру. Дополнительные сведения о группах безопасности сети см. в статье [Создание групп безопасности сети с помощью интерфейса командной строки Azure](../../virtual-network/tutorial-filter-network-traffic-cli.md). 

Создайте группу безопасности сети с помощью команды [az network nsg create](/cli/azure/network/nsg). В следующем примере создается группа безопасности сети с именем `myNetworkSecurityGroup`.

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

## <a name="add-an-inbound-rule-to-allow-ssh"></a>Добавление правила входящего трафика, разрешающего подключения SSH
Добавьте в группу безопасности сети правило входящего трафика с помощью команды [az network nsg rule create](/cli/azure/network/nsg/rule). В следующем примере создается правило `myRuleAllowSSH`.

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myRuleAllowSSH \
    --protocol tcp \
    --direction inbound \
    --priority 1000 \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 22 \
    --access allow
```

## <a name="associate-the-subnet-with-the-network-security-group"></a>Связывание подсети с группой безопасности сети
Для связывания подсети с группой безопасности сети выполните команду [az network vnet subnet update](/cli/azure/network/vnet/subnet). В следующем примере подсеть `mySubnet` связывается с группой безопасности сети `myNetworkSecurityGroup`.

```azurecli
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnet \
    --network-security-group myNetworkSecurityGroup
```


## <a name="create-the-virtual-network-interface-card-and-static-dns-names"></a>Создание виртуальной сетевой карты и статических DNS-имен
Платформа Azure является очень гибкой, но чтобы использовать DNS-имена для разрешения имен виртуальных машин, необходимо создать виртуальные сетевые карты с метками DNS. Виртуальные сетевые карты имеют большое значение, так как их можно повторно использовать, подключая к разным виртуальным машинам на протяжении жизненного цикла инфраструктуры. Это позволяет использовать виртуальную сетевую карту как статический ресурс, тогда как виртуальные машины могут быть временными. С помощью меток DNS на виртуальных сетевых картах можно обеспечить простое разрешение имен из других виртуальных машин в виртуальной сети. Использование сопоставляемых имен позволяет другим виртуальным машинам получить доступ к серверу автоматизации по DNS-имени `Jenkins` или серверу Git в качестве `gitrepo`.  

Создайте виртуальную сетевую карту с помощью команды [az network nic create](/cli/azure/network/nic). Следующий пример создает виртуальную сетевую карту `myNic`, подключает ее к виртуальной сети `myVnet``myVnet` и создает внутреннюю запись DNS-имени `jenkins`.

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --internal-dns-name jenkins
```

## <a name="deploy-the-vm-into-the-virtual-network-infrastructure"></a>Развертывание виртуальной машины в инфраструктуре виртуальной сети
Теперь у нас есть виртуальная сеть, подсеть и группа безопасности сети, выступающая в качестве брандмауэра для защиты подсети путем блокирования всего входящего трафика, кроме трафика, поступающего через порт 22 по протоколу SSH, а также виртуальная сетевая карта. Теперь вы можете развернуть виртуальную машину в этой созданной сетевой инфраструктуре.

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). Следующий пример создает виртуальную машину `myVM`, использующую Управляемые диски Azure, и подключает к ней виртуальную сетевую карту `myNic`, созданную на предыдущем шаге.

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

Используя флаги командной строки для вызова существующих ресурсов, мы укажем среде Azure развернуть виртуальную машину в существующей сети. После развертывания виртуальную сеть и подсеть можно оставить в качестве статических или постоянных ресурсов в регионе Azure для повторного развертывания.  

## <a name="next-steps"></a>Дальнейшие действия
* [Создание полной среды Linux с помощью интерфейса командной строки Azure](create-cli-complete.md)
* [Создание виртуальной машины Linux в Azure с помощью шаблонов](create-ssh-secured-vm-from-template.md)

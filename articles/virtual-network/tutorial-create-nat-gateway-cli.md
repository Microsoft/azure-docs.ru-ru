---
title: Руководство по Создание шлюза NAT — Azure CLI
titlesuffix: Azure Virtual Network NAT
description: Сведения о начале работы по созданию шлюза NAT с помощью Azure CLI.
author: asudbring
ms.author: allensu
ms.service: virtual-network
ms.subservice: nat
ms.topic: tutorial
ms.date: 03/10/2021
ms.custom: template-tutorial, devx-track-azurecli
ms.openlocfilehash: 60436b8d4a0f338f4ece59ad4cd11c14c9e4c352
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762647"
---
# <a name="tutorial-create-a-nat-gateway-using-the-azure-cli"></a>Учебник. Создание шлюза NAT с помощью Azure CLI

В этом руководстве показано, как использовать услугу преобразования сетевых адресов (NAT) в виртуальной сети Azure. Вы узнаете, как создать шлюз NAT для обеспечения исходящего подключения в виртуальных машинах в Azure. 

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создайте виртуальную сеть.
> * Создайте виртуальную машину.
> * Создание шлюза NAT и связывание его с виртуальной сетью.
> * Подключение к виртуальной машине и проверка IP-адреса NAT.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим кратким руководством требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

В следующем примере создается группа ресурсов с именем **myResourceGroupNAT** в расположении **eastus2**:

```azurecli-interactive
  az group create \
    --name myResourceGroupNAT \
    --location eastus2
```

## <a name="create-the-nat-gateway"></a>Создание шлюза NAT

Мы создадим в этом разделе шлюз NAT и вспомогательные ресурсы.

### <a name="create-public-ip-address"></a>Создание общедоступного IP-адреса

Для доступа к Интернету требуется один или несколько общедоступных IP-адресов для шлюза NAT. Создайте общедоступный IP-адрес с именем **myPublicIP** в группе **myResourceGroupNAT** с помощью команды [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create). 

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myPublicIP \
    --sku standard \
    --allocation static
```

### <a name="create-nat-gateway-resource"></a>Создание ресурса шлюза NAT

Создайте глобальный шлюз Azure NAT с помощью команды [az network nat gateway create](/cli/azure/network/nat#az_network_nat_gateway_create). В результате ее выполнения будет создан ресурс шлюза с именем **myNATgateway**, использующий общедоступный IP-адрес **myPublicIP**. Установите время ожидания простоя на 10 минут.  

```azurecli-interactive
  az network nat gateway create \
    --resource-group myResourceGroupNAT \
    --name myNATgateway \
    --public-ip-addresses myPublicIP \
    --idle-timeout 10       
  ```

### <a name="create-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с именем **myVnet** с подсетью **mySubnet**, выполнив команду [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create), в группе ресурсов **myResourceGroup**. Диапазон IP-адресов для виртуальной сети: **10.1.0.0/16**. Диапазон адресов для подсети в виртуальной сети: **10.1.0.0/24**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupNAT \
    --location eastus2 \
    --name myVnet \
    --address-prefix 10.1.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 10.1.0.0/24
```

### <a name="create-bastion-host"></a>Создание узла-бастиона

Создайте узел Бастиона Azure с именем **myBastionHost**, чтобы получить доступ к виртуальной машине. 

Используйте команду [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create), чтобы создать подсеть Бастиона Azure.

```azurecli-interactive
az network vnet subnet create \
    --resource-group myResourceGroupNAT \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

Создайте общедоступный IP-адреса для узла бастиона с помощью команды [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create). 

```azurecli-interactive
az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myBastionIP \
    --sku Standard
```

Воспользуйтесь командой [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create), чтобы создать узел бастиона. 

```azurecli-interactive
az network bastion create \
    --resource-group myResourceGroupNAT \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus2
```

### <a name="configure-nat-service-for-source-subnet"></a>Настройка NAT для исходной подсети

Мы настроим исходную подсеть **mySubnet** в виртуальной сети **myVnet**, чтобы использовать конкретный ресурс шлюза NAT **myNATgateway** в команде [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update). Эта команда активирует NAT в указанной подсети.

```azurecli-interactive
  az network vnet subnet update \
    --resource-group myResourceGroupNAT \
    --vnet-name myVnet \
    --name mySubnet \
    --nat-gateway myNATgateway
```

Шлюз NAT теперь используется для всего исходящего трафика для назначений в Интернете.  Нет необходимости настраивать UDR.


## <a name="virtual-machine"></a>Виртуальная машина

С помощью инструкций из этого раздела вы создадите виртуальную машину для тестирования шлюза NAT и проверки общедоступного IP-адреса исходящего подключения.

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm#az_vm_create).

```azurecli-interactive
az vm create \
    --name myVM \
    --resource-group myResourceGroupNAT  \
    --admin-username azureuser \
    --image win2019datacenter \
    --public-ip-address "" \
    --subnet mySubnet \
    --vnet-name myVNet
```

Прежде чем перейти к следующему разделу, дождитесь завершения создания виртуальной машины.

## <a name="test-nat-gateway"></a>Тестирование шлюза NAT

Используя сведения из этого раздела, мы протестируем шлюз NAT. Сначала мы определим общедоступный IP-адрес шлюза NAT. Затем мы подключимся к тестовой виртуальной машине и проверим исходящее подключение через шлюз NAT.
    
1. Войдите на [портал Azure](https://portal.azure.com)

1. Найдите общедоступный IP-адрес для шлюза NAT на экране **обзора**. В меню слева щелкните **Все службы**, выберите **Все ресурсы**, а затем — **myPublicIP**.

2. Запишите общедоступный IP-адрес:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/find-public-ip.png" alt-text="Определение общедоступного IP-адреса шлюза NAT" border="true":::

3. В меню слева щелкните **Все службы**, выберите **Все ресурсы**, а затем в списке ресурсов выберите виртуальную машину **myVM**, расположенную в группе ресурсов **myResourceGroupNAT**.

4. На странице **Обзор** выберите **Подключиться** и **Бастион**.

5. Нажмите синюю кнопку **Использовать Бастион**.

6. Введите имя пользователя и пароль, введенные в процессе создания виртуальной машины.

7. На виртуальной машине **myTestVM** откройте браузер **Internet Explorer**.

8. В адресной строке введите **https://whatsmyip.com** .

9. Убедитесь, что отображаемый IP-адрес соответствует адресу шлюза NAT, записанному на предыдущем этапе:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/my-ip.png" alt-text="Внешний исходящий IP-адрес в Internet Explorer" border="true":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь продолжать использовать это приложение, удалите виртуальную сеть, виртуальную машину и шлюз NAT, выполнив следующие действия:

```azurecli-interactive 
  az group delete \
    --name myResourceGroupNAT
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о NAT виртуальных сетей см. в следующей статье:
> [!div class="nextstepaction"]
> [Общие сведения о NAT виртуальных сетей](nat-overview.md)

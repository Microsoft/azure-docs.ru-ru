---
title: Краткое руководство. Создание частной конечной точки Azure с помощью Azure CLI
description: Из этого краткого руководства вы узнаете, как создать частную конечную точку с помощью Azure CLI.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 11/07/2020
ms.author: allensu
ms.openlocfilehash: 5088b4e50899a2643488103ba29a7e36a7f256ea
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778361"
---
# <a name="quickstart-create-a-private-endpoint-using-azure-cli"></a>Краткое руководство. Создание частной конечной точки с помощью Azure CLI

Чтобы начать работу с Приватным каналом Azure, воспользуйтесь частной конечной точкой для безопасного подключения к веб-приложению Azure.

Из этого краткого руководства вы узнаете, как создать частную конечную точку для веб-приложения Azure и развернуть виртуальную машину для тестирования частного подключения.  

Частные конечные точки можно создавать для служб Azure разных типов, например Azure SQL и службы хранилища Azure.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* Веб-приложение Azure с уровнем **Премиум версии 2** или планом для службы приложений более высокого уровня, развернутое в подписке Azure.  
    * Дополнительные сведения и пример см. в статье [кратком руководстве по созданию веб-приложения ASP.NET Core в Azure](../app-service/quickstart-dotnetcore.md). 
    * Подробные инструкции по созданию веб-приложения и конечной точки см. в [учебнике по подключению к веб-приложению с помощью частной конечной точки Azure](tutorial-private-endpoint-webapp-portal.md).
* Войдите на портал Azure и убедитесь, что ваша подписка активна, выполнив команду `az login`.
* Проверьте версию Azure CLI в терминале или в командном окне, выполнив команду `az --version`. Сведения о последней версии см. в [заметках о выпуске](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Если у вас нет последней версии, обновите установку, следуя указаниям в [руководстве по установке операционной системы или платформы](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create):

* с именем **CreatePrivateEndpointQS-rg**; 
* в расположении **eastus**.

```azurecli-interactive
az group create \
    --name CreatePrivateEndpointQS-rg \
    --location eastus
```

## <a name="create-a-virtual-network-and-bastion-host"></a>Создание виртуальной сети и узла бастиона

В этом разделе вы создадите виртуальную сеть, подсеть и узел бастиона. 

Для безопасного подключения к виртуальной машине с целью тестирования частной конечной точки будет использоваться узел бастиона.

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create):

* с именем **myVNet**;
* с префиксом адреса **10.0.0.0/16**;
* с именем подсети **myBackendSubnet**;
* с префиксом подсети **10.0.0.0/24**;
* в группе ресурсов **CreatePrivateEndpointQS-rg**;
* расположении **eastus**.

```azurecli-interactive
az network vnet create \
    --resource-group CreatePrivateEndpointQS-rg\
    --location eastus \
    --name myVNet \
    --address-prefixes 10.0.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.0.0.0/24
```

Измените настройки подсети, чтобы отключить сетевые политики для частной конечной точки, с помощью команды [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update).

```azurecli-interactive
az network vnet subnet update \
    --name myBackendSubnet \
    --resource-group CreatePrivateEndpointQS-rg \
    --vnet-name myVNet \
    --disable-private-endpoint-network-policies true
```

Чтобы создать общедоступный IP-адрес узла-бастиона, воспользуйтесь командой [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create):

* создайте стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**;
* в **CreatePrivateEndpointQS-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myBastionIP \
    --sku Standard
```

Используйте команду [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create), чтобы создать подсеть бастиона:

* с именем **AzureBastionSubnet**;
* с префиксом адреса **10.0.1.0/24**;
* виртуальная сеть **myVNet**;
* в группе ресурсов **CreatePrivateEndpointQS-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.0.1.0/24
```

Используйте команду [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) для создания узла-бастиона:

* с именем **myBastionHost**;
* в **CreatePrivateEndpointQS-rg**;
* связанного с общедоступным IP-адресом **myBastionIP**;
* связанного с виртуальной сетью **myVNet**;
* в расположении **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Развертывание узла-бастиона может занять несколько минут.

## <a name="create-test-virtual-machine"></a>Создание тестовой виртуальной машины

В этом разделе вы создадите виртуальную машину, которая будет использоваться для тестирования частной конечной точки.

Создайте виртуальную машину с помощью команды  [az vm create](/cli/azure/vm#az_vm_create). При появлении запроса укажите пароль в качестве учетных данных для входа на виртуальную машину:

* с именем **myVM**;
* в **CreatePrivateEndpointQS-rg**;
* в сети **myVNet**;
* подсеть **myBackendSubnet**;
* с образом сервера **Win2019Datacenter**.

```azurecli-interactive
az vm create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myVM \
    --image Win2019Datacenter \
    --public-ip-address "" \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --admin-username azureuser
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-private-endpoint"></a>Создание частной конечной точки

Используя сведения из этого раздела, вы создадите частную конечную точку.

Используйте команду [az webapp list](/cli/azure/webapp#az_webapp_list), чтобы разместить идентификатор ресурса созданного ранее веб-приложения в переменную оболочки.

Создайте конечную точку и подключение с помощью команды [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create):

* с именем **myPrivateEndpoint**;
* в группе ресурсов **CreatePrivateEndpointQS-rg**;
* виртуальная сеть **myVNet**;
* подсеть **myBackendSubnet**;
* с именем подключения **myConnection**;
* с веб-приложением **\<webapp-resource-group-name>** .

```azurecli-interactive
id=$(az webapp list \
    --resource-group <webapp-resource-group-name> \
    --query '[].[id]' \
    --output tsv)

az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group CreatePrivateEndpointQS-rg \
    --vnet-name myVNet --subnet myBackendSubnet \
    --private-connection-resource-id $id \
    --group-id sites \
    --connection-name myConnection  
```

## <a name="configure-the-private-dns-zone"></a>Настройка частной зоны DNS

Используя сведения из этого раздела, вы создадите и настроите частную зону DNS с помощью [az network private-dns zone create](/cli/azure/network/private-dns/zone#ext_privatedns_az_network_private_dns_zone_create).  

Воспользуйтесь командой [az network private-dns link vnet create](/cli/azure/network/private-dns/link/vnet#ext_privatedns_az_network_private_dns_link_vnet_create), чтобы создать в виртуальной сети ссылку на зону DNS.

Вы создадите группу зоны DNS с помощью команды [az network private-endpoint dns-zone-group create](/cli/azure/network/private-endpoint/dns-zone-group#az_network_private_endpoint_dns_zone_group_create):

* с именем зоны **privatelink.azurewebsites.net**;
* виртуальная сеть **myVNet**;
* в группе ресурсов **CreatePrivateEndpointQS-rg**;
* с именем ссылки на DNS **myDNSLink**;
* связанную с **myPrivateEndpoint**;
* с именем группы зон **MyZoneGroup**.

```azurecli-interactive
az network private-dns zone create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name "privatelink.azurewebsites.net"

az network private-dns link vnet create \
    --resource-group CreatePrivateEndpointQS-rg \
    --zone-name "privatelink.azurewebsites.net" \
    --name MyDNSLink \
    --virtual-network myVNet \
    --registration-enabled false

az network private-endpoint dns-zone-group create \
   --resource-group CreatePrivateEndpointQS-rg \
   --endpoint-name myPrivateEndpoint \
   --name MyZoneGroup \
   --private-dns-zone "privatelink.azurewebsites.net" \
   --zone-name webapp
```

## <a name="test-connectivity-to-private-endpoint"></a>Проверка подключения к частной конечной точке

В этом разделе описано, как использовать виртуальную машину, созданную на предыдущем шаге, для подключения к серверу SQL через частную конечную точку.

1. Войдите на [портал Azure](https://portal.azure.com) 
 
2. В области навигации слева выберите **Группы ресурсов**.

3. Выберите **CreatePrivateEndpointQS-rg**.

4. Выберите **myVM**.

5. На обзорной странице для **myVM** выберите **Подключиться**, а затем — **Бастион**.

6. Нажмите синюю кнопку **Использовать Бастион**.

7. Введите имя пользователя и пароль, указанные при создании виртуальной машины.

8. После подключения откройте на сервере Windows PowerShell.

9. Введите `nslookup <your-webapp-name>.azurewebsites.net`. Замените **\<your-webapp-name>** именем веб-приложения, созданного на предыдущих шагах.  Вы получите сообщение, аналогичное показанному ниже:

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mywebapp8675.privatelink.azurewebsites.net
    Address:  10.0.0.5
    Aliases:  mywebapp8675.azurewebsites.net
    ```

    Для имени веб-приложения возвращается частный IP-адрес **10.0.0.5**.  Этот адрес находится в подсети виртуальной сети, созданной ранее.

10. В окне подключения бастиона к **myVM** откройте Internet Explorer.

11. Введите URL-адрес веб-приложения **https://\<your-webapp-name>.azurewebsites.net**.

12. Если приложение не развернуто, вы получите страницу веб-приложения по умолчанию.

    :::image type="content" source="./media/create-private-endpoint-portal/web-app-default-page.png" alt-text="Страница веб-приложения по умолчанию." border="true":::

13. Закройте подключение к **myVM**.

## <a name="clean-up-resources"></a>Очистка ресурсов 
По завершении использования частной конечной точки и виртуальной машины удалите группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [az group delete](/cli/azure/group#az_group_delete):

```azurecli-interactive
az group delete \
    --name CreatePrivateEndpointQS-rg
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать:

* виртуальную сеть и узел-бастион;
* виртуальную машину;
* частную конечную точку для веб-приложения Azure.

Вы использовали виртуальную машину для безопасного тестирования подключения к веб-приложению через частную конечную точку.

Дополнительные сведения о службах, поддерживающих частную конечную точку, см. в следующей статье:
> [!div class="nextstepaction"]
> [Доступность Приватного канала](private-link-overview.md#availability)

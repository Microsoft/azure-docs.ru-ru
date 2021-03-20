---
title: Частная ссылка — Azure CLI — база данных Azure для MySQL
description: Узнайте, как настроить закрытую ссылку для базы данных Azure для MySQL с Azure CLI
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 01/09/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: da6a2b97d2656d56fa2aa0e7259fba433bd7b81e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95998589"
---
# <a name="create-and-manage-private-link-for-azure-database-for-mysql-using-cli"></a>Создание и управление частной связью для базы данных Azure для MySQL с помощью интерфейса командной строки

Частная конечная точка — ключевой компонент для построения частной ссылки в Azure. Это позволяет ресурсам Azure, таким как виртуальные машины (VM), обмениваться данными в частном порядке с ресурсами частной ссылки. В этой статье вы узнаете, как использовать Azure CLI для создания виртуальной машины в виртуальной сети Azure и сервера базы данных Azure для MySQL с помощью частной конечной точки Azure.

> [!NOTE]
> Функция закрытых ссылок доступна только для серверов базы данных Azure для MySQL в общего назначения или в ценовой категории, оптимизированные для памяти. Убедитесь, что сервер базы данных находится в одной из этих ценовых категорий.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Перед созданием любого ресурса необходимо создать группу ресурсов, которая будет содержать эту виртуальную сеть. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В этом примере создается группа ресурсов с именем *myResourceGroup* в расположении *westeurope* :

```azurecli-interactive
az group create --name myResourceGroup --location westeurope
```

## <a name="create-a-virtual-network"></a>Создайте виртуальную сеть
Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet). В этом примере создается виртуальная сеть по умолчанию с именем *myVirtualNetwork* с подсетью *mySubnet*.

```azurecli-interactive
az network vnet create \
 --name myVirtualNetwork \
 --resource-group myResourceGroup \
 --subnet-name mySubnet
```

## <a name="disable-subnet-private-endpoint-policies"></a>Отключение политик подсети частной конечной точки 
Azure развертывает ресурсы в подсеть в виртуальной сети, поэтому необходимо создать или обновить подсеть, чтобы отключить [политики сети](../private-link/disable-private-endpoint-network-policy.md)частной конечной точки. Обновите конфигурацию подсети *mySubnet* с помощью команды [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update).

```azurecli-interactive
az network vnet subnet update \
 --name mySubnet \
 --resource-group myResourceGroup \
 --vnet-name myVirtualNetwork \
 --disable-private-endpoint-network-policies true
```
## <a name="create-the-vm"></a>Создание виртуальной машины 
Создайте виртуальную машину с помощью команды az vm create. При появлении запроса укажите пароль в качестве учетных данных для входа на виртуальную машину. В этом примере создается виртуальная машина с именем *myVM*. 
```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image Win2019Datacenter
```
Запишите общедоступный IP-адрес виртуальной машины. Этот адрес используется на следующем шаге, чтобы подключиться к виртуальной машине из Интернета.

## <a name="create-an-azure-database-for-mysql-server"></a>Создайте сервер базы данных Azure для MySQL. 
Создайте базу данных Azure для MySQL с помощью команды AZ MySQL Server Create. Помните, что имя сервера MySQL должно быть уникальным в пределах Azure, поэтому замените значение заполнителя в квадратных скобках своим уникальным значением: 

```azurecli-interactive
# Create a server in the resource group 
az mysql server create \
--name mydemoserver \
--resource-group myResourcegroup \
--location westeurope \
--admin-user mylogin \
--admin-password <server_admin_password> \
--sku-name GP_Gen5_2
```

> [!NOTE]
> Иногда база данных Azure для MySQL и подсеть виртуальной сети относятся к разным подпискам. В этих случаях необходимо обеспечить следующую конфигурацию:
> - Убедитесь, что в обеих подписках зарегистрирован поставщик ресурсов **Microsoft. дбформискл** . Дополнительные сведения см. в разделе [resource-manager-registration][resource-manager-portal]

## <a name="create-the-private-endpoint"></a>Создание частной конечной точки 
Создайте частную конечную точку для сервера MySQL в виртуальной сети. 

```azurecli-interactive
az network private-endpoint create \  
    --name myPrivateEndpoint \  
    --resource-group myResourceGroup \  
    --vnet-name myVirtualNetwork  \  
    --subnet mySubnet \  
    --private-connection-resource-id $(az resource show -g myResourcegroup -n mydemoserver --resource-type "Microsoft.DBforMySQL/servers" --query "id" -o tsv) \    
    --group-id mysqlServer \  
    --connection-name myConnection  
 ```

## <a name="configure-the-private-dns-zone"></a>Настройка частной зоны DNS 
Создайте зону Частная зона DNS для домена сервера MySQL и создайте связь связи с виртуальной сетью. 
```azurecli-interactive
az network private-dns zone create --resource-group myResourceGroup \ 
   --name  "privatelink.mysql.database.azure.com" 
az network private-dns link vnet create --resource-group myResourceGroup \ 
   --zone-name  "privatelink.mysql.database.azure.com"\ 
   --name MyDNSLink \ 
   --virtual-network myVirtualNetwork \ 
   --registration-enabled false 

#Query for the network interface ID  
networkInterfaceId=$(az network private-endpoint show --name myPrivateEndpoint --resource-group myResourceGroup --query 'networkInterfaces[0].id' -o tsv)
 
 
az resource show --ids $networkInterfaceId --api-version 2019-04-01 -o json 
# Copy the content for privateIPAddress and FQDN matching the Azure database for MySQL name 
 
 
#Create DNS records 
az network private-dns record-set a create --name myserver --zone-name privatelink.mysql.database.azure.com --resource-group myResourceGroup  
az network private-dns record-set a add-record --record-set-name myserver --zone-name privatelink.mysql.database.azure.com --resource-group myResourceGroup -a <Private IP Address>
```

> [!NOTE] 
> Полное доменное имя в параметре DNS клиента не разрешается в настроенный частный IP-адрес. Вам потребуется настроить зону DNS для настроенного FQDN, как показано [ниже](../dns/dns-operations-recordsets-portal.md).

## <a name="connect-to-a-vm-from-the-internet"></a>Подключение к виртуальной машине из Интернета

Подключитесь к виртуальной машине *myVm* из Интернета, выполнив следующие действия.

1. На портале в строке поиска введите *myVm*.

1. Нажмите кнопку **Подключиться**. После нажатия кнопки **Подключиться** откроется окно **Connect to virtual machine** (Подключение к виртуальной машине).

1. Щелкните **Скачать RDP-файл**. Azure создаст и скачает на ваш компьютер файл протокола удаленного рабочего стола (*RDP*).

1. Откройте файл *downloaded.rdp*.

    1. При появлении запроса выберите **Подключиться**.

    1. Введите имя пользователя и пароль, указанные при создании виртуальной машины.

        > [!NOTE]
        > Возможно, потребуется выбрать **More choices** > **Use a different account** (Дополнительные варианты > Использовать другую учетную запись), чтобы указать учетные данные, введенные при создании виртуальной машины.

1. Щелкните **ОК**.

1. При входе в систему может появиться предупреждение о сертификате. В таком случае выберите **Да** или **Продолжить**.

1. Когда появится рабочий стол виртуальной машины, сверните его, чтобы вернуться на локальный рабочий стол.  

## <a name="access-the-mysql-server-privately-from-the-vm"></a>Доступ к серверу MySQL в частном порядке с виртуальной машины

1. На удаленном рабочем столе  *myVm* откройте PowerShell.

2. Введите  `nslookup mydemomysqlserver.privatelink.mysql.database.azure.com`. 

    Должно появиться сообщение следующего вида:
    ```azurepowershell
    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    mydemomysqlserver.privatelink.mysql.database.azure.com
    Address:  10.1.3.4
    ```

3. Проверьте подключение к частному каналу для сервера MySQL с помощью любого доступного клиента. В примере ниже я использовал [MySQL Workbench](https://dev.mysql.com/doc/workbench/en/wb-installing-windows.html) для выполнения этой операции.


4. В окне **новое подключение** введите или выберите следующие сведения:

    | Параметр | Значение |
    | ------- | ----- |
    | Имя подключения| Выберите нужное имя подключения.|
    | Hostname (Имя узла) | Выбор *mydemoserver.privatelink.MySQL.Database.Azure.com* |
    | Имя пользователя | Введите имя пользователя, *username@servername* которое предоставляется при создании сервера MySQL. |
    | Пароль | Введите пароль, предоставленный при создании сервера MySQL. |
    ||

5. Нажмите кнопку "Подключиться".

6. Просмотр баз данных из левого меню.

7. При необходимости Создание или запрос сведений из базы данных MySQL.

8. Закройте подключение к удаленному рабочему столу myVm.

## <a name="clean-up-resources"></a>Очистка ресурсов 
Чтобы удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполните команду "az group delete". 

```azurecli-interactive
az group delete --name myResourceGroup --yes 
```

## <a name="next-steps"></a>Дальнейшие действия
- Подробнее о том [, что такое частная конечная точка Azure](../private-link/private-endpoint-overview.md)

<!-- Link references, to text, Within this same GitHub repo. -->
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
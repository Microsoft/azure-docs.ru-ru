---
title: Частная ссылка — Azure CLI — база данных Azure для PostgreSQL — один сервер
description: Узнайте, как настроить частную ссылку для базы данных Azure для PostgreSQL-Single Server из Azure CLI
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 01/09/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: b8aaebdd37f835201ef549e3f97e0c0b657e4fe9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96020132"
---
# <a name="create-and-manage-private-link-for-azure-database-for-postgresql---single-server-using-cli"></a>Создание и управление частной связью для базы данных Azure для PostgreSQL — одиночный сервер с помощью интерфейса командной строки

Частная конечная точка — ключевой компонент для построения частной ссылки в Azure. Это позволяет ресурсам Azure, таким как виртуальные машины (VM), обмениваться данными в частном порядке с ресурсами частной ссылки. В этой статье вы узнаете, как использовать Azure CLI для создания виртуальной машины в виртуальной сети Azure и базы данных Azure для PostgreSQL с помощью частной конечной точки Azure.

> [!NOTE]
> Функция "Частная связь" доступна только для серверов базы данных Azure для PostgreSQL в общего назначения или ценовых категориях, оптимизированных для памяти. Убедитесь, что сервер базы данных находится в одной из этих ценовых категорий.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:

- [сервер и база данных Azure для PostgreSQL](quickstart-create-server-database-azure-cli.md);

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать Azure CLI локально, для выполнения инструкций из этого руководства вам потребуется использовать Azure CLI 2.0.28 или более поздней версии. Выполните команду `az --version`, чтобы узнать установленную версию. Сведения об установке или обновлении Azure CLI см. в [этой статье](/cli/azure/install-azure-cli).

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

## <a name="create-an-azure-database-for-postgresql---single-server"></a>Создание базы данных Azure для PostgreSQL — одиночный сервер 
Создайте базу данных Azure для PostgreSQL с помощью команды AZ postgres Server Create. Помните, что имя сервера PostgreSQL должно быть уникальным в Azure, поэтому замените значение заполнителя собственными уникальными значениями, которые использовались ранее: 

```azurecli-interactive
# Create a server in the resource group 
az postgres server create \
--name mydemoserver \
--resource-group myresourcegroup \
--location westeurope \
--admin-user mylogin \
--admin-password <server_admin_password> \
--sku-name GP_Gen5_2
```

## <a name="create-the-private-endpoint"></a>Создание частной конечной точки 
Создайте частную конечную точку для сервера PostgreSQL в виртуальной сети. 

```azurecli-interactive
az network private-endpoint create \  
    --name myPrivateEndpoint \  
    --resource-group myResourceGroup \  
    --vnet-name myVirtualNetwork  \  
    --subnet mySubnet \  
    --private-connection-resource-id $(az resource show -g myResourcegroup -n mydemoserver --resource-type "Microsoft.DBforPostgreSQL/servers" --query "id" -o tsv) \    
    --group-id postgresqlServer \  
    --connection-name myConnection  
 ```

## <a name="configure-the-private-dns-zone"></a>Настройка частной зоны DNS 
Создайте Частная зона DNS зону для домена сервера PostgreSQL и создайте связь связи с виртуальной сетью. 

```azurecli-interactive
az network private-dns zone create --resource-group myResourceGroup \ 
   --name  "privatelink.postgres.database.azure.com" 
az network private-dns link vnet create --resource-group myResourceGroup \ 
   --zone-name  "privatelink.postgres.database.azure.com"\ 
   --name MyDNSLink \ 
   --virtual-network myVirtualNetwork \ 
   --registration-enabled false 

#Query for the network interface ID  
networkInterfaceId=$(az network private-endpoint show --name myPrivateEndpoint --resource-group myResourceGroup --query 'networkInterfaces[0].id' -o tsv)
 
 
az resource show --ids $networkInterfaceId --api-version 2019-04-01 -o json 
# Copy the content for privateIPAddress and FQDN matching the Azure database for PostgreSQL name 
 
 
#Create DNS records 
az network private-dns record-set a create --name myserver --zone-name privatelink.postgres.database.azure.com --resource-group myResourceGroup  
az network private-dns record-set a add-record --record-set-name myserver --zone-name privatelink.postgres.database.azure.com --resource-group myResourceGroup -a <Private IP Address>
```

> [!NOTE] 
> Полное доменное имя в параметре DNS клиента не разрешается в настроенный частный IP-адрес. Вам потребуется настроить зону DNS для настроенного FQDN, как показано [ниже](../dns/dns-operations-recordsets-portal.md).

> [!NOTE]
> Иногда База данных Azure для PostgreSQL и подсеть виртуальной сети относятся к разным подпискам. В этих случаях необходимо обеспечить следующую конфигурацию:
> - Убедитесь, что в обеих подписках зарегистрирован поставщик ресурсов **Microsoft. дбфорпостгрескл** . Дополнительные сведения см. в разделе [поставщики ресурсов](../azure-resource-manager/management/resource-providers-and-types.md).

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

## <a name="access-the-postgresql-server-privately-from-the-vm"></a>Доступ к серверу PostgreSQL в частном порядке с виртуальной машины

1. Откройте PowerShell на удаленном рабочем столе *myVm*.

2. Введите  `nslookup mydemopostgresserver.privatelink.postgres.database.azure.com`. 

   Должно появиться сообщение следующего вида:

   ```azurepowershell
   Server:  UnKnown
   Address:  168.63.129.16
   Non-authoritative answer:
   Name:    mydemopostgresserver.privatelink.postgres.database.azure.com
   Address:  10.1.3.4
   ```

3. Проверьте подключение к частной ссылке для сервера PostgreSQL, используя любой доступный клиент. В следующем примере для выполнения операции используется [Azure Data Studio](/sql/azure-data-studio/download?view=sql-server-ver15&preserve-view=true) .

4. В окне **новое подключение** введите или выберите следующие сведения:

   | Параметр | Значение |
   | ------- | ----- |
   | Тип сервера| Выберите **PostgreSQL**.|
   | Имя сервера| Выбор *mydemopostgresserver.privatelink.postgres.Database.Azure.com* |
   | Имя пользователя | Введите имя пользователя, username@servername которое предоставляется во время создания сервера PostgreSQL. |
   |Пароль |Введите пароль, указанный при создании сервера PostgreSQL. |
   |SSL|Выберите **обязательный**.|
   ||

5. Нажмите кнопку "Подключиться".

6. Просмотр баз данных из левого меню.

7. При необходимости Создание или запрос информации с сервера postgreSQL.

8. Закройте подключение к удаленному рабочему столу myVm.

## <a name="clean-up-resources"></a>Очистка ресурсов 
Чтобы удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы, выполните команду "az group delete". 

```azurecli-interactive
az group delete --name myResourceGroup --yes 
```

## <a name="next-steps"></a>Дальнейшие действия
- Подробнее о том [, что такое частная конечная точка Azure](../private-link/private-endpoint-overview.md)

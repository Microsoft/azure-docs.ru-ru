---
title: Руководство. Подключение к серверу SQL Azure с использованием частной конечной точки Azure (Azure CLI)
description: Из этого учебника вы узнаете, как создать сервер SQL Azure с частной конечной точкой с помощью Azure CLI
services: private-link
author: asudbring
ms.service: private-link
ms.topic: tutorial
ms.date: 11/03/2020
ms.author: allensu
ms.custom: fasttrack-edit, devx-track-azurecli
ms.openlocfilehash: a8fafeaaf974893c9a1a71115912f2a7b019ddd9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771827"
---
# <a name="tutorial-connect-to-an-azure-sql-server-using-an-azure-private-endpoint---azure-cli"></a>Руководство. Подключение к серверу SQL Azure с использованием частной конечной точки Azure (Azure CLI)

Частная конечная точка Azure — ключевой компонент для построения Приватного канала в Azure. Это позволяет ресурсам Azure, таким как виртуальные машины (ВМ), обмениваться данными с ресурсами Приватного канала в частном порядке.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание виртуальной сети и узла бастиона.
> * Создайте виртуальную машину.
> * Создание Azure SQL Server и частной конечной точки.
> * Проверка подключения к частной конечной точке SQL Server.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* Войдите на портал Azure и убедитесь, что ваша подписка активна, выполнив команду `az login`.
* Проверьте версию Azure CLI в терминале или в командном окне, выполнив команду `az --version`. Сведения о последней версии см. в [заметках о выпуске](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Если у вас нет последней версии, обновите установку, следуя указаниям в [руководстве по установке операционной системы или платформы](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create):

* с именем **CreateSQLEndpointTutorial-rg**; 
* в расположении **eastus**.

```azurecli-interactive
az group create \
    --name CreateSQLEndpointTutorial-rg \
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
* в группе ресурсов **CreateSQLEndpointTutorial-rg**;
* расположении **eastus**.

```azurecli-interactive
az network vnet create \
    --resource-group CreateSQLEndpointTutorial-rg\
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
    --resource-group CreateSQLEndpointTutorial-rg \
    --vnet-name myVNet \
    --disable-private-endpoint-network-policies true
```

Чтобы создать общедоступный IP-адрес узла-бастиона, воспользуйтесь командой [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create):

* создайте стандартный избыточный между зонами общедоступный IP-адрес с именем **myPublicIP**;
* в **CreateSQLEndpointTutorial-rg**;

```azurecli-interactive
az network public-ip create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myBastionIP \
    --sku Standard
```

Используйте команду [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create), чтобы создать подсеть бастиона:

* с именем **AzureBastionSubnet**;
* с префиксом адреса **10.0.1.0/24**;
* виртуальная сеть **myVNet**;
* в группе ресурсов **CreateSQLEndpointTutorial-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.0.1.0/24
```

Используйте команду [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) для создания узла-бастиона:

* с именем **myBastionHost**;
* в **CreateSQLEndpointTutorial-rg**;
* связанного с общедоступным IP-адресом **myBastionIP**;
* связанного с виртуальной сетью **myVNet**;
* в расположении **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreateSQLEndpointTutorial-rg \
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
* в **CreateSQLEndpointTutorial-rg**;
* в сети **myVNet**;
* подсеть **myBackendSubnet**;
* с образом сервера **Win2019Datacenter**.

```azurecli-interactive
az vm create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myVM \
    --image Win2019Datacenter \
    --public-ip-address "" \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --admin-username azureuser
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-an-azure-sql-server"></a>Создание сервера SQL Azure

Используя сведения из этого раздела, вы создадите сервер и базу данных SQL.

Создайте сервер SQL с помощью команды [az sql server create](/cli/azure/sql/server#az_sql_server_create):

* заменив **\<sql-server-name>** уникальным именем сервера;
* заменив **\<your-password>** собственным паролем;
* в **CreateSQLEndpointTutorial-rg**;
* в регионе **eastus**.

```azurecli-interactive
az sql server create \
    --name <sql-server-name> \
    --resource-group CreateSQLEndpointTutorial-rg \
    --location eastus \
    --admin-user sqladmin \
    --admin-password <your-password>
```

Создайте базу данных с помощью команды [az sql db create](/cli/azure/sql/db#az_sql_db_create):

* с именем **mydatabase**;
* в **CreateSQLEndpointTutorial-rg**;
* заменив **\<sql-server-name>** уникальным именем сервера.

```azurecli-interactive
az sql db create \
    --resource-group CreateSQLEndpointTutorial-rg  \
    --server <sql-server-name> \
    --name myDataBase \
    --sample-name AdventureWorksLT
```

## <a name="create-private-endpoint"></a>Создание частной конечной точки

Используя сведения из этого раздела, вы создадите частную конечную точку.

Используйте команду [az sql server list](/cli/azure/sql/server#az_sql_server_list), чтобы разместить идентификатор ресурса сервера SQL в переменную оболочки.

Создайте конечную точку и подключение с помощью команды [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create):

* с именем **myPrivateEndpoint**;
* в группе ресурсов **CreateSQLEndpointTutorial-rg**;
* виртуальная сеть **myVNet**;
* подсеть **myBackendSubnet**;
* с именем подключения **myConnection**.

```azurecli-interactive
id=$(az sql server list \
    --resource-group CreateSQLEndpointTutorial-rg \
    --query '[].[id]' \
    --output tsv)

az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group CreateSQLEndpointTutorial-rg \
    --vnet-name myVNet --subnet myBackendSubnet \
    --private-connection-resource-id $id \
    --group-ids sqlServer \
    --connection-name myConnection  
```

## <a name="configure-the-private-dns-zone"></a>Настройка частной зоны DNS

Используя сведения из этого раздела, вы создадите и настроите частную зону DNS с помощью [az network private-dns zone create](/cli/azure/network/private-dns/zone#ext_privatedns_az_network_private_dns_zone_create).  

Воспользуйтесь командой [az network private-dns link vnet create](/cli/azure/network/private-dns/link/vnet#ext_privatedns_az_network_private_dns_link_vnet_create), чтобы создать в виртуальной сети ссылку на зону DNS.

Вы создадите группу зоны DNS с помощью команды [az network private-endpoint dns-zone-group create](/cli/azure/network/private-endpoint/dns-zone-group#az_network_private_endpoint_dns_zone_group_create):

* с именем зоны **privatelink.database.windows.net**;
* виртуальная сеть **myVNet**;
* в группе ресурсов **CreateSQLEndpointTutorial-rg**;
* с именем ссылки на DNS **myDNSLink**;
* связанную с **myPrivateEndpoint**;
* с именем группы зон **MyZoneGroup**.

```azurecli-interactive
az network private-dns zone create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name "privatelink.database.windows.net"

az network private-dns link vnet create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --zone-name "privatelink.database.windows.net" \
    --name MyDNSLink \
    --virtual-network myVNet \
    --registration-enabled false

az network private-endpoint dns-zone-group create \
   --resource-group CreateSQLEndpointTutorial-rg \
   --endpoint-name myPrivateEndpoint \
   --name MyZoneGroup \
   --private-dns-zone "privatelink.database.windows.net" \
   --zone-name sql
```

## <a name="test-connectivity-to-private-endpoint"></a>Проверка подключения к частной конечной точке

В этом разделе описано, как использовать виртуальную машину, созданную на предыдущем шаге, для подключения к серверу SQL через частную конечную точку.

1. Войдите на [портал Azure](https://portal.azure.com) 
 
2. В области навигации слева выберите **Группы ресурсов**.

3. Выберите группу ресурсов **CreateSQLEndpointTutorial-rg**.

4. Выберите **myVM**.

5. На обзорной странице для **myVM** выберите **Подключиться**, а затем — **Бастион**.

6. Нажмите синюю кнопку **Использовать Бастион**.

7. Введите имя пользователя и пароль, указанные при создании виртуальной машины.

8. После подключения откройте на сервере Windows PowerShell.

9. Введите `nslookup <sqlserver-name>.database.windows.net`. Замените **\<sqlserver-name>** именем сервера SQL, созданного на предыдущих шагах.  Вы получите сообщение, аналогичное показанному ниже:

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mysqlserver8675.privatelink.database.windows.net
    Address:  10.0.0.5
    Aliases:  mysqlserver8675.database.windows.net
    ```

    Для имени сервера SQL возвращен частный IP-адрес **10.0.0.5**.  Этот адрес находится в подсети виртуальной сети, созданной ранее.


10. Установите [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?preserve-view=true&view=sql-server-2017) в **myVM**.

11. Откройте **SQL Server Management Studio**.

12. В окне **Подключение к серверу** введите или выберите следующую информацию:

    | Параметр | Значение |
    | ------- | ----- |
    | Тип сервера | Выберите **Ядро СУБД**.|
    | Имя сервера | Введите **\<sql-server-name>.database.windows.net** |
    | Проверка подлинности | Выберите **Проверка подлинности SQL Server**. |
    | Имя пользователя | Введите имя пользователя, указанное при создании сервера |
    | Пароль | Введите пароль, указанный при создании сервера |
    | Запоминание пароля | Выберите **Да**. |

13. Выберите **Подключиться**.

14. Просмотрите базы данных в меню слева.

15. (Дополнительно) Создание или запрос информации из базы данных **mysqldatabase**.

16. Закройте подключение бастиона к **myVM**. 

## <a name="clean-up-resources"></a>Очистка ресурсов 

После завершения работы с частной конечной точкой,SQL server и виртуальными машинами удалите группу ресурсов и все содержащиеся в ней ресурсы: 

```azurecli-interactive
az group delete \
    --name CreateSQLEndpointTutorial-rg
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы создали:

* виртуальную сеть и узел-бастион;
* Виртуальная машина.
* Azure SQL Server с частной конечной точкой.

Вы использовали виртуальную машину для безопасного тестирования подключения к SQL Server через частную конечную точку.

В качестве следующего шага вы также можете ознакомиться со сценарием архитектуры **веб-приложения с частным подключением к базе данных SQL Azure**, когда веб-приложение за пределами виртуальной сети подключается к частной конечной точке базы данных.
> [!div class="nextstepaction"]
> [Веб-приложение с частным подключением к базе данных SQL Azure](/azure/architecture/example-scenario/private-web-app/private-web-app)
---
title: Управление конечными точками виртуальной сети — Azure CLI — база данных Azure для MariaDB
description: В этой статье описывается, как создать конечные точки службы виртуальной сети и правила Базы данных Azure для MariaDB и управлять ими с помощью командной строки Azure CLI.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 43d1b7700395bd06960737eae4f318d61aa03717
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98665095"
---
# <a name="create-and-manage-azure-database-for-mariadb-vnet-service-endpoints-using-azure-cli"></a>Создание конечных точек службы виртуальной сети Базы данных Azure для MariaDB и управление ими с помощью Azure CLI

Правила и конечные точки служб виртуальной сети позволяют расширить частное адресное пространство виртуальной сети, чтобы охватить сервер Базы данных Azure для MariaDB. С помощью удобных команд интерфейса командной строки Azure (CLI) можно создавать, обновлять, удалять, выводить списки и просматривать правила и конечные точки службы виртуальной сети для управления сервером. Общие сведения о конечных точках службы виртуальной сети Базы данных Azure для MariaDB, включая ограничения, см. в [этой статье](concepts-data-access-security-vnet.md). Конечные точки службы виртуальной сети доступны во всех поддерживаемых регионах Базы данных Azure для MariaDB.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Вам потребуются [сервер Базы данных Azure для MariaDB и база данных](quickstart-create-mariadb-server-database-using-azure-cli.md).

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

> [!NOTE]
> Поддержка конечных точек службы виртуальной сети предназначена только для серверов общего назначения и серверов, оптимизированных для операций в памяти.

## <a name="configure-vnet-service-endpoints"></a>Настройка конечных точек служб виртуальной сети

Для настройки виртуальных сетей используется команда [az network vnet](/cli/azure/network/vnet).

Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Выберите конкретный идентификатор подписки вашей учетной записи, выполнив команду [az account set](/cli/azure/account#az-account-set). Подставьте свойство **id** из выходных данных **az login** для вашей подписки в заполнитель для идентификатора подписки.

- Учетная запись должна предоставлять необходимые разрешения для создания виртуальной сети и конечной точки службы.

Пользователь с правами на запись в виртуальной сети может настроить конечные точки служб в виртуальных сетях независимо друг от друга.

Для защиты ресурсов служб Azure в виртуальной сети пользователь должен иметь разрешение "Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/" для добавляемых подсетей. Это разрешение по умолчанию включено во встроенные роли администраторов служб и может быть изменено при создании настраиваемых ролей.

Узнайте больше о [встроенных ролях](../role-based-access-control/built-in-roles.md) и назначении разрешений, определенных для [настраиваемых ролей](../role-based-access-control/custom-roles.md).

Виртуальные сети и ресурсы служб Azure могут находиться в одной или разных подписках. Если виртуальные сети и ресурсы служб Azure находятся в разных подписках, ресурсы должны быть размещены в одном клиенте Active Directory (AD). Убедитесь, что у обеих подписок есть зарегистрированный поставщик ресурсов **Microsoft.Sql**. Дополнительные сведения см. в разделе [resource-manager-registration][resource-manager-portal]

> [!IMPORTANT]
> Прежде чем настраивать конечные точки службы, настоятельно рекомендуется прочитать эту статью о конфигурациях конечной точки службы и рекомендациях: **Конечная точка службы для виртуальной сети.** [Конечная точка службы для виртуальной сети](../virtual-network/virtual-network-service-endpoints-overview.md) — это подсеть, значения свойств которой включают в себя одно формальное имя типа службы Azure или несколько. Конечные точки службы виртуальной сети используют имя типа службы **Microsoft.Sql**, которое относится к службе Azure, которая называется "База данных SQL". Этот тег службы также применяется к Базе данных SQL Azure, Базе данных Azure для MariaDB, PostgreSQL и MySQL. Важно отметить, что при применении тега службы **Microsoft.Sql** к конечной точке службы виртуальной сети настраивается трафик конечной точки службы для всех служб баз данных Azure, в том числе для Базы данных SQL Azure, Базы данных Azure для MariaDB, Базы данных Azure для PostgreSQL и Базы данных Azure для MySQL в подсети.

### <a name="sample-script"></a>Пример скрипта

Этот пример скрипта используется для создания сервера Базы данных Azure для MariaDB, виртуальной сети, конечной точки службы виртуальной сети и обеспечения безопасности при передачи данных с сервера в подсеть с помощью правила виртуальной сети. В этом примере скрипта измените имя и пароль администратора. Замените идентификатор подписки, используемый в команде `az account set --subscription`, собственным.

```azurecli-interactive
# To find the name of an Azure region in the CLI run this command: az account list-locations
# Substitute  <subscription id> with your identifier
az account set --subscription <subscription id>

# Create a resource group
az group create \
--name myresourcegroup \
--location westus

# Create a MariaDB server in the resource group
# Name of a server maps to DNS name and is thus required to be globally unique in Azure.
# Substitute the <server_admin_password> with your own value.
az mariadb server create \
--name mydemoserver \
--resource-group myresourcegroup \
--location westus \
--admin-user mylogin \
--admin-password <server_admin_password> \
--sku-name GP_Gen5_2

# Get available service endpoints for Azure region output is JSON
# Use the command below to get the list of services supported for endpoints, for an Azure region, say "westus".
az network vnet list-endpoint-services \
-l westus

# Add Azure SQL service endpoint to a subnet *mySubnet* while creating the virtual network *myVNet* output is JSON
az network vnet create \
-g myresourcegroup \
-n myVNet \
--address-prefixes 10.0.0.0/16 \
-l westus

# Creates the service endpoint
az network vnet subnet create \
-g myresourcegroup \
-n mySubnet \
--vnet-name myVNet \
--address-prefix 10.0.1.0/24 \
--service-endpoints Microsoft.SQL

# View service endpoints configured on a subnet
az network vnet subnet show \
-g myresourcegroup \
-n mySubnet \
--vnet-name myVNet

# Create a VNet rule on the sever to secure it to the subnet Note: resource group (-g) parameter is where the database exists. VNet resource group if different should be specified using subnet id (URI) instead of subnet, VNet pair.
az mariadb server vnet-rule create \
-n myRule \
-g myresourcegroup \
-s mydemoserver \
--vnet-name myVNet \
--subnet mySubnet
```

<!-- 
In this sample script, change the highlighted lines to customize the admin username and password. Replace the SubscriptionID used in the `az account set --subscription` command with your own subscription identifier.
[!code-azurecli-interactive[main](../../cli_scripts/mariadb/create-mysql-server-vnet/create-mysql-server.sh?highlight=5,20 "Create an Azure Database for MariaDB, VNet, VNet service endpoint, and VNet rule.")]
-->

## <a name="clean-up-deployment"></a>Очистка развертывания
После выполнения примера сценария можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```azurecli-interactive
az group delete --name myresourcegroup
```


<!--
[!code-azurecli-interactive[main](../../cli_scripts/mysql/create-mysql-server-vnet/delete-mysql.sh "Delete the resource group.")]
-->

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
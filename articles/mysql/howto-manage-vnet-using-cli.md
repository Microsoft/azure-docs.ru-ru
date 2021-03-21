---
title: Управление конечными точками виртуальной сети. Azure CLI — база данных Azure для MySQL
description: В этой статье описывается, как создать конечные точки службы виртуальной сети и правила базы данных Azure для MySQL и управлять ими с помощью командной строки Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 237b9bc4d7ac6366a67accb31fdf3c80c778b5d6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94636764"
---
# <a name="create-and-manage-azure-database-for-mysql-vnet-service-endpoints-using-azure-cli"></a>Создание конечных точек службы виртуальной сети базы данных Azure для MySQL и управление ими с помощью Azure CLI
Правила и конечные точки служб виртуальной сети расширяют частное адресное пространство виртуальной сети на сервер базы данных Azure для MySQL. С помощью удобных команд интерфейса командной строки Azure (CLI) можно создавать, обновлять, удалять, выводить списки и просматривать правила и конечные точки службы виртуальной сети для управления сервером. Общие сведения о конечных точках службы виртуальной сети базы данных Azure для MySQL, включая ограничения, см. в [этой статье](concepts-data-access-and-security-vnet.md). Конечные точки службы виртуальной сети доступны во всех поддерживаемых регионах Базы данных Azure для MySQL.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Вам потребуется [база данных Azure для сервера MySQL и база данных](quickstart-create-mysql-server-database-using-azure-cli.md).
 
- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

> [!NOTE]
> Поддержка конечных точек службы виртуальной сети предназначена только для серверов общего назначения и серверов, оптимизированных для операций в памяти.
> В случае пиринга между виртуальными сетями, если трафик проходит через общий шлюз виртуальной сети с конечными точками и должен попадать в кэширующий узел, создайте правило ACL или виртуальной сети, чтобы разрешить Виртуальным машинам Azure в шлюзе виртуальной сети доступ к Базе данных Azure для сервера MySQL.

## <a name="configure-vnet-service-endpoints-for-azure-database-for-mysql"></a>Настройка конечных точек службы виртуальной сети для базы данных Azure для MySQL
Для настройки виртуальных сетей используется команда [az network vnet](/cli/azure/network/vnet).

Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Выберите конкретный идентификатор подписки вашей учетной записи, выполнив команду [az account set](/cli/azure/account#az-account-set). Замените свойство **ID** из выходных данных **AZ login** для вашей подписки на заполнитель идентификатора подписки.

- Учетная запись должна предоставлять необходимые разрешения для создания виртуальной сети и конечной точки службы.

Пользователь с правами на запись в виртуальной сети может настроить конечные точки служб в виртуальных сетях независимо друг от друга.

Для защиты ресурсов служб Azure в виртуальной сети пользователь должен иметь разрешение "Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/" для добавляемых подсетей. Это разрешение по умолчанию включено во встроенные роли администраторов служб и может быть изменено при создании настраиваемых ролей.

Узнайте больше о [встроенных ролях](../role-based-access-control/built-in-roles.md) и назначении разрешений, определенных для [настраиваемых ролей](../role-based-access-control/custom-roles.md).

Виртуальные сети и ресурсы служб Azure могут находиться в одной или разных подписках. Если виртуальные сети и ресурсы служб Azure находятся в разных подписках, ресурсы должны быть размещены в одном клиенте Active Directory (AD). Убедитесь, что у обеих подписок есть зарегистрированный поставщик ресурсов **Microsoft.Sql**. Дополнительные сведения см. в разделе [resource-manager-registration][resource-manager-portal]

> [!IMPORTANT]
> Прежде чем выполнять сценарий ниже и настраивать конечные точки службы, советуем прочитать эту статью о конфигурациях конечной точки службы и рекомендациях: **Конечная точка службы для виртуальной сети.** [Конечная точка службы для виртуальной сети](../virtual-network/virtual-network-service-endpoints-overview.md) — это подсеть, значения свойств которой включают в себя одно формальное имя типа службы Azure или несколько. Конечные точки службы виртуальной сети используют имя типа службы **Microsoft.Sql**, которое относится к службе Azure, которая называется "База данных SQL". Этот тег службы также применяется к службам "База данных SQL Azure", "База данных Azure для PostgreSQL" и "База данных Azure для MySQL". Важно отметить, что при применении тега службы **Microsoft.Sql** к конечной точке службы виртуальной сети она настроит трафик конечной точки службы для всех служб базы данных Azure, в том числе служб "База данных SQL Azure", "База данных Azure для PostgreSQL" и "База данных Azure для MySQL" в подсети. 
> 

### <a name="sample-script-to-create-an-azure-database-for-mysql-database-create-a-vnet-vnet-service-endpoint-and-secure-the-server-to-the-subnet-with-a-vnet-rule"></a>Пример сценария для создания базы данных службы "База данных Azure для MySQL", виртуальной сети, конечной точки службы виртуальной сети и обеспечения безопасности для сервера подсети с помощью правила виртуальной сети
В этом примере скрипта измените выделенные строки, чтобы настроить имя и пароль администратора. Замените идентификатор подписки, используемый в команде `az account set --subscription`, собственным.
[!code-azurecli-interactive[main](../../cli_scripts/mysql/create-mysql-server-vnet/create-mysql-server.sh?highlight=5,20 "Create an Azure Database for MySQL, VNet, VNet service endpoint, and VNet rule.")]

## <a name="clean-up-deployment"></a>Очистка развертывания
После выполнения примера сценария можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.
[!code-azurecli-interactive[main](../../cli_scripts/mysql/create-mysql-server-vnet/delete-mysql.sh "Delete the resource group.")]

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md

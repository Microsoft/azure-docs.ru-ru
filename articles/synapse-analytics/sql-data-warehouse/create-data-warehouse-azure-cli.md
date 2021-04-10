---
title: Краткое руководство. Создание пула SQL Synapse с помощью Azure CLI
description: Сведения о том, как быстро создать пул SQL Synapse с правилом брандмауэра на уровне сервера с помощью Azure CLI.
services: synapse-analytics
author: gaursa
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: gaursa
ms.custom: azure-synapse
ms.openlocfilehash: c17bfc06e76bad7730707530001ebeccea539c37
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104602270"
---
# <a name="quickstart-create-a-synapse-sql-pool-with-azure-cli"></a>Краткое руководство. Создание пула SQL Synapse с помощью Azure CLI

Сведения о создании пула SQL Synapse (хранилище данных) в Azure Synapse Analytics с помощью Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="getting-started"></a>Начало работы

Используйте эти команды, чтобы войти в Azure и настроить группу ресурсов.

1. Если вы используете локальную установку, выполните команду [az login](/cli/azure/reference-index#az_login), чтобы войти в Azure.

   ```azurecli
   az login
   ```

1. При необходимости используйте команду [az account set](/cli/azure/account#az_account_set), чтобы выбрать подписку:

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Выполните команду [az group create](/cli/azure/group#az_group_create), чтобы создать группу ресурсов.

   ```azurecli
   az group create --name myResourceGroup --location WestEurope
   ```

1. Создайте [логический сервер SQL](../../azure-sql/database/logical-servers.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) с помощью команды [az sql server create](/cli/azure/sql/server#az_sql_server_create):

   ```azurecli
   az sql server create --resource-group myResourceGroup --name mysqlserver \
      --admin-user ServerAdmin --admin-password ChangeYourAdminPassword1
   ```

   Сервер содержит группу баз данных, которыми можно управлять как группой.

## <a name="configure-a-server-level-firewall-rule"></a>Настройка правила брандмауэра на уровне сервера

Создайте [правила брандмауэра на уровне сервера](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Правило брандмауэра на уровне сервера позволяет внешним приложениям, таким как SQL Server Management Studio или программе SQLCMD, подключаться к пулу SQL через брандмауэр службы пула SQL.

Создайте правило брандмауэра с помощью команды [az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_create).

```azurecli
az sql server firewall-rule create --resource-group myResourceGroup --name AllowSome \
   --server mysqlserver --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

В этом примере брандмауэр открыт только для других ресурсов Azure. Чтобы включить возможность внешнего подключения, измените IP-адрес на соответствующий адрес своей среды. Чтобы открыть все IP-адреса, используйте 0.0.0.0 как начальный IP-адрес, а 255.255.255.255 — как конечный.

> [!NOTE]
> Конечные точки SQL взаимодействуют через порт 1433. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 1433 может быть запрещен сетевым брандмауэром. В таком случае вы не сможете подключиться к серверу. Для этого ваш ИТ-отдел должен открыть порт 1433.
>

## <a name="create-and-manage-your-sql-pool"></a>Создание пула SQL и управление им

Создайте пул SQL. В этом примере в качестве цели службы используется DW100c, что является бюджетной начальной точкой для пула SQL.

> [!NOTE]
> Вам потребуется созданная ранее рабочая область. Дополнительные сведения см. в [кратком руководстве Создание рабочей области Azure Synapse с помощью Azure CLI](../quickstart-create-workspace-cli.md).

Для создания пула SQL используйте команду [az synapse sql pool create](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_create):

```azurecli
az synapse sql pool create --resource-group myResourceGroup --name mySampleDataWarehouse \
   --performance-level "DW1000c" --workspace-name testsynapseworkspace
```

Дополнительные сведения о параметрах см. в статье о командлете [az synapse sql pool](/cli/azure/ext/synapse/synapse/sql/pool).

Вы можете просмотреть пулы SQL с помощью команды [az synapse sql pool list](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_list):

```azurecli
az synapse sql pool list --resource-group myResourceGroup --workspace-name testsynapseworkspace
```

Чтобы обновить существующий пул, используйте команду [az synapse sql pool update](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_update):

```azurecli
az synapse sql pool update --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Чтобы приостановить пул, используйте команду [az synapse sql pool pause](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_pause):

```azurecli
az synapse sql pool pause --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Чтобы запустить приостановленный пул, используйте команду [az synapse sql pool resume](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_resume):

```azurecli
az synapse sql pool resume --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Чтобы удалить существующий пул SQL, используйте команду [az synapse sql pool delete](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_delete):

```azurecli
az synapse sql pool delete --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Другие краткие руководства в этой серии созданы на основе этого документа.

> [!TIP]
> Если вы собираетесь продолжать работу с ними, не удаляйте ресурсы, которые вы создали при работе с этим руководством. Если вы не планируете продолжать работу, воспользуйтесь командой [az group delete](/cli/azure/group#az_group_delete), чтобы удалить все ресурсы, созданные при прохождении этого краткого руководства.
>

```azurecli
az group delete --ResourceGroupName MyResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Вы создали пул SQL, создали правило брандмауэра и подключились к этому пулу SQL. Чтобы узнать больше, перейдите к статье о [загрузке данных в пул SQL](./load-data-from-azure-blob-storage-using-copy.md).
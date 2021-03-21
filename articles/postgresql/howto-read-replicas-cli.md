---
title: Управление репликами чтения — Azure CLI, REST API, база данных Azure для PostgreSQL — один сервер
description: Узнайте, как управлять репликами чтения в базе данных Azure для PostgreSQL-Single Server из Azure CLI и REST API
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 12/17/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7e74a58a14bdcc2a6fe1e9f86305aae415c6abf7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97674520"
---
# <a name="create-and-manage-read-replicas-from-the-azure-cli-rest-api"></a>Создание реплик чтения и управление ими из Azure CLI REST API

Из этой статьи вы узнаете, как создавать реплики чтения и управлять ими в базе данных Azure для PostgreSQL с помощью Azure CLI и REST API. Дополнительные сведения о репликах чтения см. в [этой статье](concepts-read-replicas.md).

## <a name="azure-replication-support"></a>Поддержка репликации Azure
[Считывание реплик](concepts-read-replicas.md) и [логического декодирования](concepts-logical.md) зависит от postgres журнала упреждающего записи (WAL) для получения сведений. Для этих двух функций требуется разный уровень ведения журнала от postgres. Для логического декодирования требуется более высокий уровень ведения журнала, чем считывание реплик.

Чтобы настроить правильный уровень ведения журнала, используйте параметр поддержки репликации Azure. Поддержка репликации в Azure имеет три параметра:

* **Off** — накладывает наименьшую информацию в Wal. Этот параметр недоступен на большинстве серверов базы данных Azure для PostgreSQL.  
* **Реплика** — более подробная, чем **Off**. Это минимальный уровень ведения журнала, необходимый для работы [реплик чтения](concepts-read-replicas.md) . Этот параметр используется по умолчанию на большинстве серверов.
* **Логический** — более подробный, чем **реплика**. Это минимальный уровень ведения журнала для работы логического декодирования. Реплики чтения также работают с этим параметром.


> [!NOTE]
> При развертывании реплик чтения для устойчивых основных рабочих нагрузок с большим количеством операций записи, задержка репликации может продолжать расти и никогда не сможет быть перехватываться с базой данных-источником. Это также может увеличить использование хранилища на основном сайте, так как файлы WAL не удаляются до тех пор, пока они не будут получены в реплике.

## <a name="azure-cli"></a>Azure CLI
Вы можете создавать реплики чтения и управлять ими с помощью Azure CLI.

### <a name="prerequisites"></a>Предварительные условия

- [Установите Azure CLI 2.0](/cli/azure/install-azure-cli).
- [Сервер базы данных Azure для PostgreSQL](quickstart-create-server-up-azure-cli.md) , который будет основным сервером.


### <a name="prepare-the-primary-server"></a>Подготовка сервера источника

1. Проверьте значение сервера источника `azure.replication_support` . Для работы реплики чтения должна быть как минимум РЕПЛИКа.

   ```azurecli-interactive
   az postgres server configuration show --resource-group myresourcegroup --server-name mydemoserver --name azure.replication_support
   ```

2. Если не `azure.replication_support` является как минимум репликой, задайте ее. 

   ```azurecli-interactive
   az postgres server configuration set --resource-group myresourcegroup --server-name mydemoserver --name azure.replication_support --value REPLICA
   ```

3. Перезапустите сервер, чтобы применить изменение.

   ```azurecli-interactive
   az postgres server restart --name mydemoserver --resource-group myresourcegroup
   ```

### <a name="create-a-read-replica"></a>Создание реплики чтения

Команда [AZ postgres Server Replica Create](/cli/azure/postgres/server/replica#az-postgres-server-replica-create) требует наличия следующих параметров:

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| resource-group | myresourcegroup |  Группа ресурсов, в которой будет создан сервер реплики.  |
| name | mydemoserver — реплика | Имя нового сервера реплики, который создается. |
| source-server | mydemoserver | Имя или идентификатор ресурса существующего первичного сервера, с которого выполняется репликация. Используйте идентификатор ресурса, если требуется, чтобы группы ресурсов реплик и Master были разными. |

В следующем примере CLI реплика создается в том же регионе, что и Главная.

```azurecli-interactive
az postgres server replica create --name mydemoserver-replica --source-server mydemoserver --resource-group myresourcegroup
```

Чтобы создать реплику чтения между регионами, используйте `--location` параметр. Приведенный ниже пример интерфейса командной строки создает реплику в западной части США.

```azurecli-interactive
az postgres server replica create --name mydemoserver-replica --source-server mydemoserver --resource-group myresourcegroup --location westus
```

> [!NOTE]
> Дополнительные сведения о том, в каких регионах можно создать реплику, см. в статье [об основных понятиях реплики чтения](concepts-read-replicas.md). 

Если параметр не задан `azure.replication_support` для **реплики** на общего назначения или на сервере-источнике, оптимизированном для памяти, и сервер перезагружен, появится сообщение об ошибке. Перед созданием реплики выполните эти два действия.

> [!IMPORTANT]
> Ознакомьтесь с [разделом рекомендации в обзоре реплики чтения](concepts-read-replicas.md#considerations).
>
> Перед обновлением параметра сервера-источника до нового значения измените значение параметра реплики на равное или большее. Это действие помогает реплике справиться с изменениями, внесенными в главную базу.

### <a name="list-replicas"></a>Список реплик
Список реплик сервера-источника можно просмотреть с помощью команды [AZ postgres Server Replica List](/cli/azure/postgres/server/replica#az-postgres-server-replica-list) .

```azurecli-interactive
az postgres server replica list --server-name mydemoserver --resource-group myresourcegroup 
```

### <a name="stop-replication-to-a-replica-server"></a>Остановка репликации на сервер-реплику
Вы можете отключить репликацию между сервером-источником и репликой чтения, выполнив команду [AZ postgres Server Replica](/cli/azure/postgres/server/replica#az-postgres-server-replica-stop) .

После того как репликация будет прервана на сервере-источнике и реплике чтения, ее нельзя будет отменить. Реплика чтения становится отдельным сервером, который поддерживает операции чтения и записи. Это сервер нельзя снова преобразовать в реплику.

```azurecli-interactive
az postgres server replica stop --name mydemoserver-replica --resource-group myresourcegroup 
```

### <a name="delete-a-primary-or-replica-server"></a>Удаление основного или сервер-реплики
Чтобы удалить основной или сервер-реплику, используйте команду [AZ postgres Server DELETE](/cli/azure/postgres/server#az-postgres-server-delete) .

При удалении сервера-источника репликация на все реплики чтения останавливается. Реплики чтения становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

```azurecli-interactive
az postgres server delete --name myserver --resource-group myresourcegroup
```

## <a name="rest-api"></a>REST API
Вы можете создавать реплики чтения и управлять ими с помощью [REST API Azure](/rest/api/azure/).

### <a name="prepare-the-primary-server"></a>Подготовка сервера источника

1. Проверьте значение сервера источника `azure.replication_support` . Для работы реплики чтения должна быть как минимум РЕПЛИКа.

   ```http
   GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/configurations/azure.replication_support?api-version=2017-12-01
   ```

2. Если не `azure.replication_support` является как минимум репликой, задайте ее.

   ```http
   PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/configurations/azure.replication_support?api-version=2017-12-01
   ```

   ```JSON
   {
    "properties": {
    "value": "replica"
    }
   }
   ```

2. [Перезапустите сервер](/rest/api/postgresql/servers/restart) , чтобы применить изменение.

   ```http
   POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/restart?api-version=2017-12-01
   ```

### <a name="create-a-read-replica"></a>Создание реплики чтения
Вы можете создать реплику чтения с помощью [API создания](/rest/api/postgresql/servers/create):

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{replicaName}?api-version=2017-12-01
```

```json
{
  "location": "southeastasia",
  "properties": {
    "createMode": "Replica",
    "sourceServerId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}"
  }
}
```

> [!NOTE]
> Дополнительные сведения о том, в каких регионах можно создать реплику, см. в статье [об основных понятиях реплики чтения](concepts-read-replicas.md). 

Если параметр не задан `azure.replication_support` для **реплики** на общего назначения или на сервере-источнике, оптимизированном для памяти, и сервер перезагружен, появится сообщение об ошибке. Перед созданием реплики выполните эти два действия.

Реплика создается с использованием тех же параметров вычислений и хранилища, что и у главного сервера. После создания реплики некоторые параметры могут быть изменены независимо от сервера-источника: поколение вычислений, виртуальных ядер, хранилище и срок хранения резервной копии. Изменить также можно ценовую категорию (за исключением уровня "Базовый").


> [!IMPORTANT]
> Перед обновлением параметра сервера-источника до нового значения измените значение параметра реплики на равное или большее. Это действие помогает реплике справиться с изменениями, внесенными в главную базу.

### <a name="list-replicas"></a>Список реплик
Список реплик сервера-источника можно просмотреть с помощью [API списка реплик](/rest/api/postgresql/replicas/listbyserver).

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/Replicas?api-version=2017-12-01
```

### <a name="stop-replication-to-a-replica-server"></a>Остановка репликации на сервер-реплику
Вы можете отключить репликацию между сервером-источником и репликой чтения с помощью [API обновления](/rest/api/postgresql/servers/update).

После того как репликация будет прервана на сервере-источнике и реплике чтения, ее нельзя будет отменить. Реплика чтения становится отдельным сервером, который поддерживает операции чтения и записи. Это сервер нельзя снова преобразовать в реплику.

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{replicaServerName}?api-version=2017-12-01
```

```json
{
  "properties": {
    "replicationRole":"None"  
   }
}
```

### <a name="delete-a-primary-or-replica-server"></a>Удаление основного или сервер-реплики
Чтобы удалить основной или сервер-реплику, используйте [API удаления](/rest/api/postgresql/servers/delete):

При удалении сервера-источника репликация на все реплики чтения останавливается. Реплики чтения становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{serverName}?api-version=2017-12-01
```

## <a name="next-steps"></a>Дальнейшие действия
* См. дополнительные сведения о [репликах чтения в Базе данных Azure для PostgreSQL](concepts-read-replicas.md).
* Дополнительные сведения см. в статье [Создание реплик чтения и управление ими с помощью портала Azure](howto-read-replicas-portal.md).
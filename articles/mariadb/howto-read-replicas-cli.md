---
title: Управление репликами чтения — Azure CLI, REST API — база данных Azure для MariaDB
description: В этой статье описывается, как настроить и управлять репликами чтения в базе данных Azure для MariaDB с помощью Azure CLI и REST API.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 6/10/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 70da1e9c70bf80737065362c68781652dd9ab6e5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98664959"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mariadb-using-the-azure-cli-and-rest-api"></a>Как создавать реплики чтения и управлять ими в базе данных Azure для MariaDB с помощью Azure CLI и REST API

В этой статье вы узнаете, как создавать реплики чтения и управлять ими в службе "база данных Azure для MariaDB" с помощью Azure CLI и REST API.

## <a name="azure-cli"></a>Azure CLI
Вы можете создавать реплики чтения и управлять ими с помощью Azure CLI.

### <a name="prerequisites"></a>Предварительные условия

- [Установите Azure CLI 2.0](/cli/azure/install-azure-cli).
- [Сервер базы данных Azure для MariaDB](quickstart-create-mariadb-server-database-using-azure-portal.md) , который будет использоваться в качестве исходного сервера. 

> [!IMPORTANT]
> Функция чтения реплики доступна только для серверов базы данных Azure для MariaDB в общего назначения или ценовой категории, оптимизированные для памяти. Убедитесь, что исходный сервер находится в одной из этих ценовых категорий.

### <a name="create-a-read-replica"></a>Создание реплики чтения

> [!IMPORTANT]
> При создании реплики для источника, не имеющего существующих реплик, источник сначала перезапускается для подготовки к репликации. Это необходимо учесть, то есть выполнять такие операции в период низкой нагрузки.

Чтобы создать сервер реплики чтения, выполните следующую команду:

```azurecli-interactive
az mariadb server replica create --name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup
```

Для команды `az mariadb server replica create` обязательны указанные ниже параметры.

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Группа ресурсов, в которой будет создан сервер реплики.  |
| name | mydemoreplicaserver | Имя нового сервера реплики, который создается. |
| source-server | mydemoserver | Имя или идентификатор существующего исходного сервера, с которого выполняется репликация. |

Чтобы создать реплику чтения между регионами, используйте `--location` параметр. 

Приведенный ниже пример интерфейса командной строки создает реплику в западной части США.

```azurecli-interactive
az mariadb server replica create --name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup --location westus
```

> [!NOTE]
> Дополнительные сведения о том, в каких регионах можно создать реплику, см. в статье [об основных понятиях реплики чтения](concepts-read-replicas.md). 

> [!NOTE]
> Реплики чтения создаются с той же конфигурацией сервера, что и у главного сервера. Вы можете изменить созданную конфигурацию сервера-реплики. Рекомендуется, чтобы конфигурация сервера реплики хранилась в значении, превышающем значение источника, чтобы реплика могла поддерживать базу данных master.

### <a name="list-replicas-for-a-source-server"></a>Вывод списка реплик для исходного сервера

Чтобы просмотреть все реплики для данного исходного сервера, выполните следующую команду: 

```azurecli-interactive
az mariadb server replica list --server-name mydemoserver --resource-group myresourcegroup
```

Для команды `az mariadb server replica list` обязательны указанные ниже параметры.

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Группа ресурсов, в которой будет создан сервер реплики.  |
| server-name | mydemoserver | Имя или идентификатор исходного сервера. |

### <a name="stop-replication-to-a-replica-server"></a>Остановка репликации на сервер-реплику

> [!IMPORTANT]
> Остановка репликации на сервер является необратимой операцией. После остановки репликации между источником и репликой отменить ее нельзя. Сервер-реплика становится автономным и начинает поддерживает операции чтения и записи. Это сервер нельзя снова преобразовать в реплику.

Репликацию на сервер реплики чтения можно остановить, выполнив следующую команду:

```azurecli-interactive
az mariadb server replica stop --name mydemoreplicaserver --resource-group myresourcegroup
```

Для команды `az mariadb server replica stop` обязательны указанные ниже параметры.

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Группа ресурсов, в которой находится сервер реплики.  |
| name | mydemoreplicaserver | Имя сервера реплики для остановки репликации. |

### <a name="delete-a-replica-server"></a>Удаление сервера-реплики

Для удаления сервера реплики чтения можно выполнить команду **[AZ MariaDB Server DELETE](/cli/azure/mariadb/server)** .

```azurecli-interactive
az mariadb server delete --resource-group myresourcegroup --name mydemoreplicaserver
```

### <a name="delete-a-source-server"></a>Удаление исходного сервера

> [!IMPORTANT]
> Удаление исходного сервера приводит к остановке репликации на все серверы-реплики и удалению самого исходного сервера. Серверы-реплики становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

Чтобы удалить исходный сервер, можно выполнить команду **[AZ MariaDB Server DELETE](/cli/azure/mariadb/server)** .

```azurecli-interactive
az mariadb server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="rest-api"></a>REST API
Вы можете создавать реплики чтения и управлять ими с помощью [REST API Azure](/rest/api/azure/).

### <a name="create-a-read-replica"></a>Создание реплики чтения
Вы можете создать реплику чтения с помощью [API создания](/rest/api/mariadb/servers/create):

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMariaDB/servers/{replicaName}?api-version=2017-12-01
```

```json
{
  "location": "southeastasia",
  "properties": {
    "createMode": "Replica",
    "sourceServerId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMariaDB/servers/{masterServerName}"
  }
}
```

> [!NOTE]
> Дополнительные сведения о том, в каких регионах можно создать реплику, см. в статье [об основных понятиях реплики чтения](concepts-read-replicas.md). 

Если параметр не задан `azure.replication_support` для **реплики** на общего назначения или на исходном сервере, оптимизированном для памяти, и сервер перезагружен, появится сообщение об ошибке. Перед созданием реплики выполните эти два действия.

Реплика создается с использованием тех же параметров вычислений и хранилища, что и у главного сервера. После создания реплики некоторые параметры могут быть изменены независимо от исходного сервера: поколение вычислений, виртуальных ядер, хранилище и срок хранения резервной копии. Изменить также можно ценовую категорию (за исключением уровня "Базовый").


> [!IMPORTANT]
> Перед обновлением параметра сервера источника до нового значения измените значение параметра реплики на равное или большее. Это действие помогает реплике справиться с изменениями, внесенными в главную базу.

### <a name="list-replicas"></a>Список реплик
Список реплик исходного сервера можно просмотреть с помощью [API списка реплик](/rest/api/mariadb/replicas/listbyserver).

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMariaDB/servers/{masterServerName}/Replicas?api-version=2017-12-01
```

### <a name="stop-replication-to-a-replica-server"></a>Остановка репликации на сервер-реплику
Вы можете отключить репликацию между исходным сервером и репликой чтения с помощью [API обновления](/rest/api/mariadb/servers/update).

После того как репликация на исходный сервер и реплика чтения будет прервана, отменить ее невозможно. Реплика чтения становится отдельным сервером, который поддерживает операции чтения и записи. Это сервер нельзя снова преобразовать в реплику.

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMariaDB/servers/{masterServerName}?api-version=2017-12-01
```

```json
{
  "properties": {
    "replicationRole":"None"  
   }
}
```

### <a name="delete-a-source-or-replica-server"></a>Удаление сервера источника или реплики
Чтобы удалить сервер-источник или реплику, используйте [API удаления](/rest/api/mariadb/servers/delete):

При удалении исходного сервера репликация на все реплики чтения останавливается. Реплики чтения становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMariaDB/servers/{serverName}?api-version=2017-12-01
```


## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [репликах чтения](concepts-read-replicas.md)
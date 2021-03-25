---
title: Управление репликами чтения в базе данных Azure для гибкого сервера MySQL с помощью Azure CLI.
description: Узнайте, как настроить и управлять репликами чтения в базе данных Azure для гибкого сервера MySQL с помощью Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/23/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: bc95cd3ab471826538a551687c38d1422e4b7163
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105108661"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-flexible-server-using-the-azure-cli"></a>Как создавать реплики чтения и управлять ими в базе данных Azure для гибкого сервера MySQL с помощью Azure CLI

> [!IMPORTANT]
> Чтение реплик в базе данных Azure для MySQL — гибкий сервер находится на этапе предварительной версии.

В этой статье вы узнаете, как создавать реплики чтения и управлять ими в базе данных Azure для гибкого сервера MySQL с помощью Azure CLI. Дополнительные сведения о репликах чтения см. в [этой статье](concepts-read-replicas.md).

> [!Note]
> Реплика не поддерживается на сервере с включенной высокой доступностью. 

## <a name="azure-cli"></a>Azure CLI
Вы можете создавать реплики чтения и управлять ими с помощью Azure CLI.

### <a name="prerequisites"></a>Предварительные требования

- [Установите Azure CLI 2.0](/cli/azure/install-azure-cli).
- [Гибкий сервер базы данных Azure для MySQL](quickstart-create-server-cli.md) , который будет использоваться в качестве исходного сервера.

### <a name="create-a-read-replica"></a>Создание реплики чтения

> [!IMPORTANT]
> При создании реплики для источника, не имеющего существующих реплик, источник сначала перезапускается для подготовки к репликации. Это необходимо учесть, то есть выполнять такие операции в период низкой нагрузки.

Чтобы создать сервер реплики чтения, выполните следующую команду:

```azurecli-interactive
az mysql flexible-server replica create --replica-name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup
``` 

> [!NOTE]
> Реплики чтения создаются с той же конфигурацией сервера, что и источник. Вы можете изменить созданную конфигурацию сервера-реплики. Сервер реплики всегда создается в той же группе ресурсов, что и исходный сервер, и ту же самую подписку. Если вы хотите создать сервер реплики в другой группе ресурсов или другой подписке, можно [переместить сервер реплики](../../azure-resource-manager/management/move-resource-group-and-subscription.md) после его создания. Рекомендуется, чтобы конфигурация сервера реплики хранилась в значениях, равных или превышающих источник, чтобы гарантировать, что реплика сможет справиться с источником.


### <a name="list-replicas-for-a-source-server"></a>Вывод списка реплик для исходного сервера

Чтобы просмотреть все реплики для данного исходного сервера, выполните следующую команду: 

```azurecli-interactive
az mysql flexible-server replica list --server-name mydemoserver --resource-group myresourcegroup
```

### <a name="stop-replication-to-a-replica-server"></a>Остановка репликации на сервер-реплику

> [!IMPORTANT]
> Остановка репликации на сервер является необратимой операцией. После остановки репликации между источником и репликой отменить ее нельзя. Сервер-реплика становится автономным и начинает поддерживает операции чтения и записи. Это сервер нельзя снова преобразовать в реплику.

Репликацию на сервер реплики чтения можно остановить, выполнив следующую команду:

```azurecli-interactive
az mysql flexible-server replica stop-replication --replica-name mydemoreplicaserver --resource-group myresourcegroup
```

### <a name="delete-a-replica-server"></a>Удаление сервера-реплики

Сервер реплики чтения можно удалить, выполнив команду **[az mysql server delete](/cli/azure/mysql/server)**.

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoreplicaserver
```

### <a name="delete-a-source-server"></a>Удаление исходного сервера

> [!IMPORTANT]
> Удаление исходного сервера приводит к остановке репликации на все серверы-реплики и удалению самого исходного сервера. Серверы-реплики становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

Чтобы удалить исходный сервер, можно выполнить команду **[AZ MySQL гибкий-Server DELETE](/cli/azure/mysql/flexible-server)** .

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [репликах чтения](concepts-read-replicas.md)
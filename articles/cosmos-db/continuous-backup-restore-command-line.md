---
title: Используйте Azure CLI, чтобы настроить непрерывное резервное копирование и восстановление на момент времени в Azure Cosmos DB.
description: Узнайте, как подготавливать учетную запись с непрерывным резервным копированием и восстановлением данных с помощью Azure CLI.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 02/01/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 9ea71dae746ac423e7b17b6235b4d5cd3e143cd7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100377335"
---
# <a name="configure-and-manage-continuous-backup-and-point-in-time-restore-preview---using-azure-cli"></a>Настройка непрерывного резервного копирования и восстановление на момент времени (Предварительная версия) и управление ими с помощью Azure CLI
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> Функция восстановления на момент времени (режим непрерывного резервного копирования) для Azure Cosmos DB в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Функция восстановления на момент времени (Предварительная версия) Azure Cosmos DB позволяет восстановиться после случайного изменения в контейнере, восстановления удаленной учетной записи, базы данных или контейнера или восстановления в любом регионе (где существовали резервные копии). Режим непрерывного резервного копирования позволяет выполнять восстановление на любой момент времени в течение последних 30 дней.

В этой статье описывается, как подготавливать учетную запись с непрерывным резервным копированием и восстановлением данных с помощью Azure CLI.

## <a name="install-azure-cli"></a><a id="install"></a>Установка Azure CLI

1. Установите последнюю версию Azure CLI

   * Установите последнюю версию [Azure CLI](/cli/azure/install-azure-cli) или версию выше 2.17.1.
   * Если вы уже установили интерфейс командной строки, выполните `az upgrade` команду для обновления до последней версии. Эта команда будет работать только с CLI версии выше 2,11. Если у вас более ранняя версия, используйте ссылку выше, чтобы установить последнюю версию.

1. Установите `cosmosdb-preview` расширение CLI.

   * Команды восстановления на момент времени доступны в разделе `cosmosdb-preview` расширение.
   * Это расширение можно установить, выполнив следующую команду: `az extension add --name cosmosdb-preview`
   * Это расширение можно удалить, выполнив следующую команду: `az extension remove --name cosmosdb-preview`

1. Вход и выбор подписки

   * Войдите в учетную запись Azure с помощью `az login` команды.
   * Выберите нужную подписку с помощью `az account set -s <subscriptionguid>` команды.

## <a name="provision-a-sql-api-account-with-continuous-backup"></a><a id="provision-sql-api"></a>Предоставление учетной записи API SQL с непрерывным резервным копированием

Чтобы подготовить учетную запись API SQL с непрерывной резервной копией, `--backup-policy-type Continuous` необходимо передать дополнительный аргумент вместе с обычной командой подготовки. Следующая команда представляет собой пример учетной записи для записи в одном регионе `pitracct2` с политикой непрерывного резервного копирования, созданной в регионе " *Западная часть США* " в разделе *myrg* Resource Group:

```azurecli-interactive

az cosmosdb create \
  --name pitracct2 \
  --resource-group myrg \
  --backup-policy-type Continuous \
  --default-consistency-level Session \
  --locations regionName="West US"

```

## <a name="provision-an-azure-cosmos-db-api-for-mongodb-account-with-continuous-backup"></a><a id="provision-mongo-api"></a>Предоставление Azure Cosmos DB API для учетной записи MongoDB с непрерывным резервным копированием

Следующая команда показывает пример учетной записи для записи в один регион `pitracct3` с именем с политикой непрерывного резервного копирования, создавшей регион " *Западная часть США* " в группе ресурсов *myrg* :

```azurecli-interactive

az cosmosdb create \
  --name pitracct3 \
  --kind MongoDB \
  --resource-group myrg \
  --server-version "3.6" \
  --backup-policy-type Continuous \
  --default-consistency-level Session \
  --locations regionName="West US"

```

## <a name="trigger-a-restore-operation-with-cli"></a><a id="trigger-restore"></a>Активация операции восстановления с помощью интерфейса командной строки

Самый простой способ запустить восстановление — выполнить команду Restore с именем целевой учетной записи, исходной учетной записью, расположением, группой ресурсов, меткой времени (в формате UTC) и, при необходимости, именем базы данных и контейнера. Ниже приведены некоторые примеры активации операции восстановления.

1. Создайте новую учетную запись Azure Cosmos DB, восстановив ее из существующей учетной записи.

   ```azurecli-interactive

   az cosmosdb restore \
    --target-database-account-name MyRestoredCosmosDBDatabaseAccount \
    --account-name MySourceAccount \
    --restore-timestamp 2020-07-13T16:03:41+0000 \
    --resource-group MyResourceGroup \
    --location "West US"

   ```

2. Создание новой учетной записи Azure Cosmos DB путем восстановления из существующей учетной записи базы данных только выбранных баз данных и контейнеров.

   ```azurecli-interactive

   az cosmosdb restore \
    --resource-group MyResourceGroup \
    --target-database-account-name MyRestoredCosmosDBDatabaseAccount \
    --account-name MySourceAccount \
    --restore-timestamp 2020-07-13T16:03:41+0000 \
    --location "West US" \
    --databases-to-restore name=MyDB1 collections=collection1 collection2 \
    --databases-to-restore name=MyDB2 collections=collection3 collection4

   ```

## <a name="enumerate-restorable-resources-for-sql-api"></a><a id="enumerate-sql-api"></a>Перечисление ресурсов restorable для API SQL

Приведенные ниже команды перечисления помогают найти ресурсы, доступные для восстановления с различными метками времени. Кроме того, они предоставляют канал ключевых событий в ресурсах учетной записи restorable, базы данных и контейнера.

**Список всех учетных записей, которые могут быть восстановлены в текущей подписке**

Выполните следующую команду CLI, чтобы вывести список всех учетных записей, которые можно восстановить в текущей подписке.

```azurecli-interactive
az cosmosdb restorable-database-account list --account-name "pitrbb"
```

Ответ включает все учетные записи базы данных (как активных, так и удаленных), которые можно восстановить, и регионы, из которых они могут быть восстановлены.

```json
{
    "accountName": "pitrbb",
    "apiType": "Sql",
    "creationTime": "2021-01-08T23:34:11.095870+00:00",
    "deletionTime": null,
    "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.DocumentDB/locations/West US/restorableDatabaseAccounts/7133a59a-d1c0-4645-a699-6e296d6ac865",
    "identity": null,
    "location": "West US",
    "name": "7133a59a-d1c0-4645-a699-6e296d6ac865",
    "restorableLocations": [
      {
        "creationTime": "2021-01-08T23:34:11.095870+00:00",
        "deletionTime": null,
        "locationName": "West US",
        "regionalDatabaseAccountInstanceId": "f02df26b-c0ec-4829-8bef-3482d36e6230"
      }
    ],
    "tags": null,
    "type": "Microsoft.DocumentDB/locations/restorableDatabaseAccounts"
  }
```

Так же, как `CreationTime` и `DeletionTime` для учетной записи, существует `CreationTime` или `DeletionTime` для региона. Эти значения позволяют выбрать правильный регион и допустимый диапазон времени для восстановления в этом регионе.

**Вывод списка всех версий баз данных в активной учетной записи базы данных**

Перечисление всех версий баз данных позволяет выбрать правильную базу данных в сценарии, в котором фактическое время существования базы данных неизвестно.

Выполните следующую команду CLI, чтобы вывести список всех версий баз данных. Эта команда работает только с учетными записями Live. `instanceId` `location` Параметры и извлекаются из `name` `location` свойств и в ответе `az cosmosdb restorable-database-account list` команды. Атрибут instanceId также является свойством восстанавливаемой учетной записи базы данных-источника:

```azurecli-interactive
az cosmosdb sql restorable-database list \
  --instance-id "7133a59a-d1c0-4645-a699-6e296d6ac865" \
  --location "West US"
```

Теперь выходные данные команды показывают, когда была создана и удалена база данных.

```json
[
  {
    "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.DocumentDB/locations/West US/restorableDatabaseAccounts/7133a59a-d1c0-4645-a699-6e296d6ac865/restorableSqlDatabases/40e93dbd-2abe-4356-a31a-35567b777220",
    ..
    "name": "40e93dbd-2abe-4356-a31a-35567b777220",
    "resource": {
      "database": {
        "id": "db1"
      },
      "eventTimestamp": "2021-01-08T23:27:25Z",
      "operationType": "Create",
      "ownerId": "db1",
      "ownerResourceId": "YuZAAA=="
    },
    ..
  },
  {
    "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.DocumentDB/locations/West US/restorableDatabaseAccounts/7133a59a-d1c0-4645-a699-6e296d6ac865/restorableSqlDatabases/243c38cb-5c41-4931-8cfb-5948881a40ea",
    ..
    "name": "243c38cb-5c41-4931-8cfb-5948881a40ea",
    "resource": {
      "database": {
        "id": "spdb1"
      },
      "eventTimestamp": "2021-01-08T23:25:25Z",
      "operationType": "Create",
      "ownerId": "spdb1",
      "ownerResourceId": "OIQ1AA=="
    },
   ..
  }
]
```

**Вывод списка всех версий контейнеров SQL базы данных в действующей учетной записи базы данных**

Используйте следующую команду, чтобы получить список всех версий контейнеров SQL. Эта команда работает только с учетными записями Live. `databaseRid`Параметр — это `ResourceId` база данных, которую необходимо восстановить. Это значение `ownerResourceid` атрибута, найденного в ответе `az cosmosdb sql restorable-database list` команды.

```azurecli-interactive
az cosmosdb sql restorable-container list \
    --instance-id "7133a59a-d1c0-4645-a699-6e296d6ac865" \
    --database-rid "OIQ1AA==" \
    --location "West US"
```

Выходные данные команды показывают список операций, выполняемых со всеми контейнерами в этой базе данных:

```json
[
  {
    ...
    
      "eventTimestamp": "2021-01-08T23:25:29Z",
      "operationType": "Replace",
      "ownerId": "procol3",
      "ownerResourceId": "OIQ1APZ7U18="
...
  },
  {
    ...
      "eventTimestamp": "2021-01-08T23:25:26Z",
      "operationType": "Create",
      "ownerId": "procol3",
      "ownerResourceId": "OIQ1APZ7U18="
  },
]
```

**Поиск баз данных или контейнеров, которые могут быть восстановлены по любой заданной метке времени**

Используйте следующую команду, чтобы получить список баз данных или контейнеров, которые можно восстановить по любой заданной метке времени. Эта команда работает только с учетными записями Live.

```azurecli-interactive

az cosmosdb sql restorable-resource list \
  --instance-id "7133a59a-d1c0-4645-a699-6e296d6ac865" \
  --location "West US" \
  --restore-location "West US" \  
  --restore-timestamp "2021-01-10 T01:00:00+0000"

```

```json
[
  {
    "collectionNames": [
      "procol1",
      "procol2"
    ],
    "databaseName": "db1"
  },
  {
    "collectionNames": [
      "procol3",
       "spcol1"
    ],
    "databaseName": "spdb1"
  }
]
```

## <a name="enumerate-restorable-resources-for-mongodb-api-account"></a><a id="enumerate-mongodb-api"></a>Перечисление ресурсов restorable для учетной записи API MongoDB

Приведенные ниже команды перечисления помогают найти ресурсы, доступные для восстановления с различными метками времени. Кроме того, они предоставляют канал ключевых событий в ресурсах учетной записи restorable, базы данных и контейнера. Как и в случае с API SQL, можно использовать `az cosmosdb` команду, но с `mongodb` параметром AS вместо `sql` . Эти команды работают только для учетных записей Live.

**Вывод списка всех версий баз данных MongoDB в действующей учетной записи базы данных**

```azurecli-interactive
az cosmosdb mongodb restorable-database list \
    --instance-id "7133a59a-d1c0-4645-a699-6e296d6ac865" \
    --location "West US"
```

**Вывод списка всех версий коллекций MongoDB базы данных в активной учетной записи базы данных**

```azurecli-interactive
az cosmosdb mongodb restorable-collection list \
    --instance-id "<InstanceID>" \
    --database-rid "AoQ13r==" \
    --location "West US"
```

**Перечисление всех ресурсов учетной записи базы данных MongoDB, доступных для восстановления в заданной метке времени и регионе**

```azurecli-interactive
az cosmosdb mongodb restorable-resource list \
    --instance-id "7133a59a-d1c0-4645-a699-6e296d6ac865" \
    --location "West US" \
    --restore-location "West US" \
    --restore-timestamp "2020-07-20T16:09:53+0000"
```

## <a name="next-steps"></a>Дальнейшие действия

* Настройка непрерывного резервного копирования и управление им с помощью [портал Azure](continuous-backup-restore-portal.md), [PowerShell](continuous-backup-restore-powershell.md)или [Azure Resource Manager](continuous-backup-restore-template.md).
* [Модель ресурсов режима непрерывного резервного копирования](continuous-backup-restore-resource-model.md)
* [Управление разрешениями](continuous-backup-restore-permissions.md) , необходимыми для восстановления данных в режиме непрерывной архивации.

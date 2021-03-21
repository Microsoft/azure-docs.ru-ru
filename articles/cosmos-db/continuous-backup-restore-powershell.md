---
title: Используйте Azure PowerShell, чтобы настроить непрерывное резервное копирование и восстановление на момент времени в Azure Cosmos DB.
description: Узнайте, как подготавливать учетную запись с непрерывным резервным копированием и восстановлением данных с помощью Azure PowerShell.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 02/01/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 5261075a82eaefd91cbedd2dd2fe08cb1e0a20b4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100381840"
---
# <a name="configure-and-manage-continuous-backup-and-point-in-time-restore-preview---using-azure-powershell"></a>Настройка непрерывного резервного копирования и восстановление на момент времени (Предварительная версия) и управление ими с помощью Azure PowerShell
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> Функция восстановления на момент времени (режим непрерывного резервного копирования) для Azure Cosmos DB в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Функция восстановления на момент времени (Предварительная версия) Azure Cosmos DB позволяет восстановиться после случайного изменения в контейнере, восстановления удаленной учетной записи, базы данных или контейнера или восстановления в любом регионе (где существовали резервные копии). Режим непрерывного резервного копирования позволяет выполнять восстановление на любой момент времени в течение последних 30 дней.

В этой статье описывается, как подготавливать учетную запись с непрерывным резервным копированием и восстановлением данных с помощью Azure PowerShell.

## <a name="install-azure-powershell"></a><a id="install-powershell"></a>Установка Azure PowerShell

1. Выполните следующую команду из Azure PowerShell, чтобы установить `Az.CosmosDB` модуль предварительной версии, который содержит команды, связанные с восстановлением до точки во времени:

   ```azurepowershell
   Install-Module -Name Az.CosmosDB -AllowPrerelease
   ```

1. После установки модулей Войдите в Azure с помощью

   ```azurepowershell
   Connect-AzAccount
   ```

1. Выберите определенную подписку с помощью следующей команды:

   ```azurepowershell
   Select-AzSubscription -Subscription <SubscriptionName>
   ```

## <a name="provision-a-sql-api-account-with-continuous-backup"></a><a id="provision-sql-api"></a>Предоставление учетной записи API SQL с непрерывным резервным копированием

Чтобы подготовить учетную запись с непрерывным резервным копированием, добавьте аргумент `-BackupPolicyType Continuous` вместе с командой обычной подготовки.

Следующий командлет — это пример учетной записи для записи в одном регионе `pitracct2` с политикой непрерывного резервного копирования, созданной в регионе " *Западная часть США* " в разделе *myrg* Resource Group:

```azurepowershell

New-AzCosmosDBAccount `
  -ResourceGroupName "myrg" `
  -Location "West US" `
  -BackupPolicyType Continuous `
  -Name "pitracct2" `
  -ApiKind "Sql"
      
```

## <a name="provision-a-mongodb-api-account-with-continuous-backup"></a><a id="provision-mongodb-api"></a>Подготавливает учетную запись API MongoDB с непрерывным резервным копированием

Следующий командлет — это пример непрерывной учетной записи резервного копирования *pitracct2* , созданной в регионе " *Западная часть США* " в группе ресурсов *myrg* :

```azurepowershell

New-AzCosmosDBAccount `
  -ResourceGroupName "myrg" `
  -Location "West US" `
  -BackupPolicyType Continuous `
  -Name "pitracct3" `
  -ApiKind "MongoDB" `
  -ServerVersion "3.6"

```

## <a name="trigger-a-restore-operation"></a><a id="trigger-restore"></a>Активация операции восстановления

Следующий командлет является примером активации операции восстановления с помощью команды Restore с использованием целевой учетной записи, исходной учетной записи, расположения, группы ресурсов и метки времени.

```azurepowershell

Restore-AzCosmosDBAccount `
  -TargetResourceGroupName <resourceGroupName> `
  -TargetDatabaseAccountName <restored-account-name> `
  -SourceDatabaseAccountName <sourceDatabaseAccountName> `
  -RestoreTimestampInUtc <UTC time> `
  -Location <Azure Region Name>

```

**Пример 1.** Восстановление всей учетной записи:

```azurepowershell

Restore-AzCosmosDBAccount `
  -TargetResourceGroupName "rg" `
  -TargetDatabaseAccountName "pitrbb-ps-1" `
  -SourceDatabaseAccountName "source-sql" `
  -RestoreTimestampInUtc "2021-01-05T22:06:00" `
  -Location "West US"

```

**Пример 2:** Восстановление конкретных коллекций и баз данных. В этом примере восстанавливается коллекция myCol1, myCol2 из myDB1 и вся база данных myDB2, которая включает в себя все контейнеры.

```azurepowershell
$datatabaseToRestore1 = New-AzCosmosDBDatabaseToRestore -DatabaseName "myDB1" -CollectionName "myCol1", "myCol2"
$datatabaseToRestore2 = New-AzCosmosDBDatabaseToRestore -DatabaseName "myDB2"

Restore-AzCosmosDBAccount `
  -TargetResourceGroupName "rg" `
  -TargetDatabaseAccountName "pitrbb-ps-1" `
  -SourceDatabaseAccountName "source-sql" `
  -RestoreTimestampInUtc "2021-01-05T22:06:00" `
  -DatabasesToRestore $datatabaseToRestore1, $datatabaseToRestore2 `
  -Location "West US"

```

## <a name="enumerate-restorable-resources-for-sql-api"></a><a id="enumerate-sql-api"></a>Перечисление ресурсов restorable для API SQL

Командлеты перечисления помогают найти ресурсы, доступные для восстановления в различных отметках времени. Кроме того, они предоставляют канал ключевых событий в ресурсах учетной записи restorable, базы данных и контейнера.

**Список всех учетных записей, которые могут быть восстановлены в текущей подписке**

Выполните `Get-AzCosmosDBRestorableDatabaseAccount` команду PowerShell, чтобы вывести список всех учетных записей, которые можно восстановить в текущей подписке.

Ответ включает все учетные записи базы данных (как активных, так и удаленных), которые можно восстановить, и регионы, из которых они могут быть восстановлены.

```console
{
    "accountName": "sampleaccount",
    "apiType": "Sql",
    "creationTime": "2020-08-08T01:04:52.070190+00:00",
    "deletionTime": null,
    "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.DocumentDB/locations/West US/restorableDatabaseAccounts/23e99a35-cd36-4df4-9614-f767a03b9995",
    "identity": null,
    "location": "West US",
    "name": "23e99a35-cd36-4df4-9614-f767a03b9995",
    "restorableLocations": [
      {
        "creationTime": "2020-08-08T01:04:52.945185+00:00",
        "deletionTime": null,
        "locationName": "West US",
        "regionalDatabaseAccountInstanceId": "30701557-ecf8-43ce-8810-2c8be01dccf9"
      },
      {
        "creationTime": "2020-08-08T01:15:43.507795+00:00",
        "deletionTime": null,
        "locationName": "East US",
        "regionalDatabaseAccountInstanceId": "8283b088-b67d-4975-bfbe-0705e3e7a599"
      }
    ],
    "tags": null,
    "type": "Microsoft.DocumentDB/locations/restorableDatabaseAccounts"
  },
```

Так же, как `CreationTime` и `DeletionTime` для учетной записи, существует `CreationTime` или `DeletionTime` для региона. Эти значения позволяют выбрать правильный регион и допустимый диапазон времени для восстановления в этом регионе.

**Вывод списка всех версий баз данных SQL в действующей учетной записи базы данных**

Перечисление всех версий баз данных позволяет выбрать правильную базу данных в сценарии, в котором фактическое время существования базы данных неизвестно.

Выполните следующую команду PowerShell, чтобы вывести список всех версий баз данных. Эта команда работает только с учетными записями Live. `DatabaseAccountInstanceId` `LocationName` Параметры и извлекаются из `name` `location` свойств и в ответе `Get-AzCosmosDBRestorableDatabaseAccount` командлета. `DatabaseAccountInstanceId`Атрибут ссылается на `instanceId` свойство восстанавливаемой учетной записи базы данных источника:


```azurepowershell

Get-AzCosmosdbSqlRestorableDatabase `
  -LocationName "East US" `
  -DatabaseAccountInstanceId <DatabaseAccountInstanceId>

```

**Вывод списка всех версий контейнеров SQL базы данных в активной учетной записи базы данных.**

Используйте следующую команду, чтобы получить список всех версий контейнеров SQL. Эта команда работает только с учетными записями Live. `DatabaseRid`Параметр — это `ResourceId` база данных, которую необходимо восстановить. Это значение `ownerResourceid` атрибута, найденного в ответе `Get-AzCosmosdbSqlRestorableDatabase` командлета. Ответ также содержит список операций, выполняемых со всеми контейнерами в этой базе данных.

```azurepowershell

Get-AzCosmosdbSqlRestorableContainer `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -DatabaseRid "AoQ13r==" `
  -LocationName "West US"

```

**Поиск баз данных или контейнеров, которые могут быть восстановлены по любой заданной метке времени**

Используйте следующую команду, чтобы получить список баз данных или контейнеров, которые можно восстановить по любой заданной метке времени. Эта команда работает только с учетными записями Live.

```azurepowershell

Get-AzCosmosdbSqlRestorableResource `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -LocationName "West US" `
  -RestoreLocation "East US" `
  -RestoreTimestamp "2020-07-20T16:09:53+0000"

```

## <a name="enumerate-restorable-resources-for-mongodb"></a><a id="enumerate-mongodb-api"></a>Перечисление ресурсов restorable для MongoDB

Приведенные ниже команды перечисления помогают найти ресурсы, доступные для восстановления с различными метками времени. Кроме того, они предоставляют канал ключевых событий в ресурсах учетной записи restorable, базы данных и контейнера. Эти команды работают только для учетных записей Live, и они похожи на команды API SQL, но `MongoDB` в имени команды вместо `sql` .

**Вывод списка всех версий баз данных MongoDB в действующей учетной записи базы данных**

```azurepowershell

Get-AzCosmosdbMongoDBRestorableDatabase `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -LocationName "West US"

```

**Вывод списка всех версий коллекций MongoDB базы данных в активной учетной записи базы данных**

```azurepowershell

Get-AzCosmosdbMongoDBRestorableCollection `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -DatabaseRid "AoQ13r==" `
  -LocationName "West US"
```

**Перечисление всех ресурсов учетной записи базы данных MongoDB, доступных для восстановления в заданной метке времени и регионе**

```azurepowershell

Get-AzCosmosdbMongoDBRestorableResource `
  -DatabaseAccountInstanceId "d056a4f8-044a-436f-80c8-cd3edbc94c68" `
  -LocationName "West US" `
  -RestoreLocation "West US" `
  -RestoreTimestamp "2020-07-20T16:09:53+0000"
```

## <a name="next-steps"></a>Дальнейшие действия

* Настройка непрерывного резервного копирования и управление им с помощью [Azure CLI](continuous-backup-restore-command-line.md), [Диспетчер ресурсов](continuous-backup-restore-template.md)или [портал Azure](continuous-backup-restore-portal.md).
* [Модель ресурсов режима непрерывного резервного копирования](continuous-backup-restore-resource-model.md)
* [Управление разрешениями](continuous-backup-restore-permissions.md) , необходимыми для восстановления данных в режиме непрерывной архивации.

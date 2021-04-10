---
title: 'REST API: синхронизация между несколькими базами данных'
description: Работа с примером скрипта REST API для синхронизации нескольких баз данных.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: REST API
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: d3ff8114c11b224a0bdbb0bd2d0e5686a7e57b55
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105565917"
---
# <a name="use-rest-api-to-sync-data-between-multiple-databases"></a>Использование REST API для синхронизации данных между несколькими базами данных 

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Этот пример использует REST API для Синхронизации данных SQL, чтобы синхронизировать данные между несколькими базами данных.

Общие сведения о синхронизации данных SQL см. в статье [Синхронизация данных в нескольких облачных и локальных базах данных с помощью синхронизации данных SQL в Azure](../sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> Служба "Синхронизация данных SQL" пока не поддерживает Управляемый экземпляр SQL Azure.

## <a name="create-sync-group"></a>Создание группы синхронизации

Используйте шаблон [создания или обновления](/rest/api/sql/syncgroups/createorupdate), чтобы создать группу синхронизации.
 
При создании группы синхронизации не передавайте в нее схему синхронизации (таблицы и столбцы) или значение masterSyncMemberName, так как на этот момент группа синхронизации еще не имеет сведений о таблицах и столбцах.

Пример запроса для создания группы синхронизации: 

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser"
  }
}
```

Пример ответа для создания группы синхронизации:

Код состояния: 200.
```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```

Код состояния: 201.
```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```

## <a name="create-sync-member"></a>Создание участника синхронизации

Используйте шаблон [создания или обновления](/rest/api/sql/syncmembers/createorupdate), чтобы создать участник синхронизации.

Пример запроса для создания участника синхронизации:

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  }
}
```
Пример ответа при создании участника синхронизации:

Код состояния: 200.
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

Код состояния: 201.
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

## <a name="refresh-schema"></a>Обновление схемы

После успешного создания группы синхронизации обновите схему, используя следующие шаблоны.

Шаблон [обновления схемы центральной базы данных](/rest/api/sql/syncgroups/refreshhubschema) позволяет обновить схему для центральной базы данных. 

Пример запроса для обновления схему центральной базы данных: 

```http
POST https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/refreshHubSchema?api-version=2015-05-01-preview
```

Пример ответа для обновления схемы центральной базы данных: 

Код состояния: 200.

Код состояния: 202.

Шаблон [списка схем центральной базы данных](/rest/api/sql/syncgroups/listhubschemas) позволяет вывести список схем для центральной базы данных. 

Шаблон [обновления схемы участника](/rest/api/sql/syncmembers/refreshmemberschema) позволяет обновить схему базы данных участника. 

Шаблон [списка схем участника](/rest/api/sql/syncmembers/listmemberschemas) позволяет вывести список схем для базы данных участника. 

Переходите к следующему шагу только после успешного обновления схемы. 

## <a name="update-sync-group"></a>Обновление группы синхронизации 

Используйте шаблон [создания или обновления](/rest/api/sql/syncgroups/createorupdate), чтобы обновить группу синхронизации.

Для обновления группы синхронизации укажите схему синхронизации. Укажите новую схему и значение masterSyncMemberName, которое определяет имя для хранения нужной схемы. 

Пример запроса для обновления группы синхронизации: 

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser"
  }
}
```

Пример ответа для обновления группы синхронизации: 

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```

```json
{
  "properties": {
    "interval": -1,
    "lastSyncTime": "0001-01-01T08:00:00Z",
    "conflictResolutionPolicy": "HubWin",
    "syncDatabaseId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328",
    "hubDatabaseUserName": "hubUser",
    "syncState": "NotReady"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-3521/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187",
  "name": "syncgroupcrud-3187",
  "type": "Microsoft.Sql/servers/databases/syncGroups"
}
```
## <a name="update-sync-member"></a>Обновление участника синхронизации

Используйте шаблон [создания или обновления](/rest/api/sql/syncmembers/createorupdate), чтобы обновить участник синхронизации.

Пример запроса для обновления участника синхронизации: 

```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879?api-version=2015-05-01-preview
```

```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  }
}
```

Пример ответа для обновления участника синхронизации: 

Код состояния: 200.
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

Код состояния: 201.
```json
{
  "properties": {
    "databaseType": "AzureSqlDatabase",
    "serverName": "syncgroupcrud-3379.database.windows.net",
    "databaseName": "syncgroupcrud-7421",
    "userName": "myUser",
    "syncDirection": "Bidirectional",
    "syncState": "UnProvisioned"
  },
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/syncMembers/syncgroupcrud-4879",
  "name": "syncgroupcrud-4879",
  "type": "Microsoft.Sql/servers/databases/syncGroups/syncMembers"
}
```

## <a name="trigger-sync"></a>Активация синхронизации

Используйте шаблон [активации синхронизации](/rest/api/sql/syncgroups/triggersync) для активации операции синхронизации.

Пример запроса для активации операции синхронизации: 

```http
POST https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/syncgroupcrud-65440/providers/Microsoft.Sql/servers/syncgroupcrud-8475/databases/syncgroupcrud-4328/syncGroups/syncgroupcrud-3187/triggerSync?api-version=2015-05-01-preview
```

Пример ответа для активации операции синхронизации: 

Код состояния: 200.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев PowerShell для Базы данных SQL Azure можно найти в разделе [Примеры Azure PowerShell для Базы данных SQL Azure](../powershell-script-content-guide.md).

Дополнительные сведения о синхронизации данных SQL см. в следующих материалах:

- Обзор. [Синхронизация данных в нескольких облачных и локальных базах данных с помощью синхронизации данных SQL в Azure](../sql-data-sync-data-sql-server-sql-database.md)
- Настройка синхронизации данных
    - С помощью портала Azure: [Руководство по настройке функции синхронизации данных SQL между Базой данных SQL Azure и SQL Server](../sql-data-sync-sql-server-configure.md)
    - С помощью PowerShell: [Использование PowerShell для синхронизации между базой данных в службе "База данных SQL Azure" и SQL Server](sql-data-sync-sync-data-between-azure-onprem.md)
- Агент синхронизации данных: [Агент синхронизации данных для синхронизации данных SQL в Azure](../sql-data-sync-agent-overview.md)
- Рекомендации: [Рекомендации по синхронизации данных SQL в Azure](../sql-data-sync-best-practices.md)
- Мониторинг: [Мониторинг синхронизации данных SQL с помощью журналов Azure Monitor](../monitor-tune-overview.md)
- Устранение неполадок: [Устранение неполадок с синхронизацией данных SQL в Azure](../sql-data-sync-troubleshoot.md)
- Обновление схемы синхронизации
    - С помощью Transact-SQL: [Автоматическая репликация изменений схемы при синхронизации данных SQL в Azure](../sql-data-sync-update-sync-schema.md)
    - С помощью PowerShell: [Использование PowerShell для обновления схемы синхронизации в существующей группе синхронизации](update-sync-schema-in-sync-group.md)

Дополнительные сведения о Базе данных SQL см. в разделах:

- [Функции службы базы данных SQL Azure](../sql-database-paas-overview.md)
- [Управление жизненным циклом базы данных](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))
---
title: Часто задаваемые вопросы о функции восстановления на момент времени Azure Cosmos DB.
description: В этой статье перечислены часто задаваемые вопросы о функции восстановления на момент времени Azure Cosmos DB, которая достигается с помощью режима непрерывной архивации.
author: kanshiG
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 02/01/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 1cf94964f420f7a7d4fc0f6ba0b77813b3e75787
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100393230"
---
# <a name="frequently-asked-questions-on-the-azure-cosmos-db-point-in-time-restore-feature-preview"></a>Часто задаваемые вопросы о функции восстановления Azure Cosmos DB на момент времени (Предварительная версия)
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

> [!IMPORTANT]
> Функция восстановления на момент времени (режим непрерывного резервного копирования) для Azure Cosmos DB в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этой статье перечислены часто задаваемые вопросы о функции восстановления Azure Cosmos DB на момент времени (Предварительная версия), которая достигается с помощью режима непрерывной архивации.

## <a name="how-much-time-does-it-takes-to-restore"></a>Сколько времени требуется для восстановления?
Длительность восстановления зависит от размера данных.

### <a name="can-i-submit-the-restore-time-in-local-time"></a>Можно ли отправить время восстановления по местному времени?
Восстановление может не происходить в зависимости от того, существовали ли в это время ключевые ресурсы, такие как базы данных или контейнеры. Проверить можно, введя время и просмотрев выбранную базу данных или контейнер на определенный момент времени. Если не существует ресурсов для восстановления, процесс восстановления не работает.

### <a name="how-can-i-track-if-an-account-is-being-restored"></a>Как можно отвести себя о восстановлении учетной записи?
После отправки команды RESTORE и перехода на ту же страницу после завершения операции в строке состояния отобразится сообщение об успешном восстановлении учетной записи. Можно также выполнить поиск восстановленной учетной записи и [отвести состояние восстанавливаемой учетной записи](continuous-backup-restore-portal.md#track-restore-status). Пока выполняется восстановление, состояние учетной записи будет *создаваться* после завершения операции восстановления состояние учетной записи изменится на "в *сети*".

Аналогично PowerShell и CLI можно отслеживать ход выполнения операции восстановления, выполняя `az cosmosdb show` команду следующим образом:

```azurecli-interactive
az cosmosdb show --name "accountName" --resource-group "resourceGroup"
```

ProvisioningState отображается, *Если* учетная запись находится в режиме «в сети».

```json
{
"virtualNetworkRules": [],
"writeLocations" : [
{
    "documentEndpoint": "https://<accountname>.documents.azure.com:443/", 
    "failoverpriority": 0,
    "id": "accountName" ,
    "isZoneRedundant" : false, 
    "locationName": "West US 2", 
    "provisioningState": "Succeeded"
}
]
}
```

### <a name="how-can-i-find-out-whether-an-account-was-restored-from-another-account"></a>Как узнать, восстановлена ли учетная запись из другой учетной записи?
Выполните `az cosmosdb show` команду в выходных данных, чтобы увидеть, что свойство имеет значение `createMode` . Значение, если задано **Восстановление**. Это означает, что учетная запись была восстановлена из другой учетной записи. `restoreParameters`Свойство имеет дополнительные сведения `restoreSource` , например, с идентификатором исходной учетной записи. Последним идентификатором GUID в `restoreSource` параметре является InstanceId исходной учетной записи.

Например, в следующих выходных данных идентификатор экземпляра исходной учетной записи — *7b4bb-f6a0-430E-ade1-638d781830cc* .

```json
"restoreParameters": {
   "databasesToRestore" : [],
   "restoreMode": "PointInTime",
   "restoreSource": "/subscriptions/2a5b-f6a0-430e-ade1-638d781830dd/providers/Microsoft.DocumentDB/locations/westus/restorableDatabaseAccounts/7b4bb-f6a0-430e-ade1-638d781830cc",
   "restoreTimestampInUtc": "2020-06-11T22:05:09Z"
}
```

### <a name="what-happens-when-i-restore-a-shared-throughput-database-or-a-container-within-a-shared-throughput-database"></a>Что происходит при восстановлении общей пропускной способности базы данных или контейнера в общей пропускной способности?
Восстанавливается вся база данных с общей пропускной способностью. Нельзя выбрать подмножество контейнеров в базе данных с общей пропускной способностью для восстановления.

### <a name="what-is-the-use-of-instanceid-in-the-account-definition"></a>Каково использование InstanceID в определении учетной записи?
В любой момент времени свойство учетной записи Azure Cosmos DB `accountName` является глобально уникальным, пока оно активно. Однако после удаления учетной записи можно создать другую учетную запись с тем же именем, поэтому "accountName" больше не сможет опознать экземпляр учетной записи. 

ID или `instanceId` является свойством экземпляра учетной записи и используется для устранения неоднозначности в нескольких учетных записях (активных и удаленных), если они имеют одно и то же имя для восстановления. Идентификатор экземпляра можно получить, выполнив `Get-AzCosmosDBRestorableDatabaseAccount`  `az cosmosdb restorable-database-account` команды или. Значение атрибута Name обозначает "InstanceID".

## <a name="next-steps"></a>Дальнейшие действия

* Что такое режим [непрерывного резервного копирования](continuous-backup-restore-introduction.md) ?
* Настройка непрерывного резервного копирования и управление им с помощью [портал Azure](continuous-backup-restore-portal.md), [PowerShell](continuous-backup-restore-powershell.md), [интерфейса командной строки](continuous-backup-restore-command-line.md)или [Azure Resource Manager](continuous-backup-restore-template.md).
* [Управление разрешениями](continuous-backup-restore-permissions.md) , необходимыми для восстановления данных в режиме непрерывной архивации.
* [Модель ресурсов режима непрерывного резервного копирования](continuous-backup-restore-resource-model.md)
---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 03/27/2019
ms.author: tamram
ms.openlocfilehash: 9a60c624b181a1efd2f6deebd349daa82214a8a4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "67184962"
---
<!--created by Robin Shahan to go in the articles for table storage w/powershell.
    There is one for Azure Table Storage and one for Azure Cosmos DB Table API -->

## <a name="managing-table-entities"></a>Управление сущностями таблицы

Теперь, когда имеется таблица, рассмотрим управление сущностями или строками в таблице. 

Сущности могут иметь до 255 свойств, включая три свойства системы: **PartitionKey**, **RowKey** и **timestamp**. Вы несете ответственность за вставку и обновление значений **PartitionKey** и **RowKey**. Сервер управляет значением **метки времени**, которое нельзя изменить. Вместе **PartitionKey** и **RowKey** однозначно идентифицируют каждую сущность в пределах таблицы.

* **PartitionKey**— определяет раздел, в котором хранится сущность.
* **RowKey**— уникально определяет сущность в разделе.

Можно определить до 252 свойств для сущности. 

### <a name="add-table-entities"></a>Добавление сущностей таблицы

Добавление сущностей в таблицу с помощью **Add-азтаблеров**. В этих примерах используются ключи секций со значениями `partition1` и `partition2` , а также ключи строк, равные аббревиатурам состояний. Свойства в каждой сущности — `username` и `userid` . 

```powershell
$partitionKey1 = "partition1"
$partitionKey2 = "partition2"

# add four rows 
Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey1 `
    -rowKey ("CA") -property @{"username"="Chris";"userid"=1}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey2 `
    -rowKey ("NM") -property @{"username"="Jessie";"userid"=2}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey1 `
    -rowKey ("WA") -property @{"username"="Christine";"userid"=3}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey2 `
    -rowKey ("TX") -property @{"username"="Steven";"userid"=4}
```

### <a name="query-the-table-entities"></a>Запрос сущностей таблицы

Запросы к сущностям в таблице можно выполнять с помощью команды **Get-азтаблеров** .

> [!NOTE]
> Командлеты **Get-азуресторажетаблеровалл**, **Get-азуресторажетаблеровбипартитионкэй**, **Get-азуресторажетаблеровбиколумннаме** и **Get-AzureStorageTableRowByCustomFilter** являются устаревшими и будут удалены в будущих обновлениях версий.

#### <a name="retrieve-all-entities"></a>Получение всех сущностей

```powershell
Get-AzTableRow -table $cloudTable | ft
```

Эта команда выдает результаты наподобие следующих:

| userid | username | partition | rowkey |
|----|---------|---------------|----|
| 1 | Chris | partition1 | Целостности и доступности |
| 3 | Christine | partition1 | WA |
| 2 | Jessie | partition2 | NM |
| 4 | Steven | partition2 | TX |

#### <a name="retrieve-entities-for-a-specific-partition"></a>Получение сущностей для определенного раздела

```powershell
Get-AzTableRow -table $cloudTable -partitionKey $partitionKey1 | ft
```

Результаты выглядят как следующая таблица.

| userid | username | partition | rowkey |
|----|---------|---------------|----|
| 1 | Chris | partition1 | Целостности и доступности |
| 3 | Christine | partition1 | WA |

#### <a name="retrieve-entities-for-a-specific-value-in-a-specific-column"></a>Получение сущностей для конкретного значения в определенном столбце

```powershell
Get-AzTableRow -table $cloudTable `
    -columnName "username" `
    -value "Chris" `
    -operator Equal
```

Этот запрос получает одну запись.

|поле|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | Целостности и доступности |

#### <a name="retrieve-entities-using-a-custom-filter"></a>Получение сущностей с использованием пользовательского фильтра 

```powershell
Get-AzTableRow `
    -table $cloudTable `
    -customFilter "(userid eq 1)"
```

Этот запрос получает одну запись.

|поле|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | Целостности и доступности |

### <a name="updating-entities"></a>Обновление сущностей 

Для обновления сущностей нужно выполнить три шага. Сначала получите сущность, которую необходимо изменить. Затем внесите изменения В-третьих, зафиксируйте это изменение с помощью **обновления азтаблеров**.

Измените сущность с именем пользователя = "Jessie" на = "Jessie2". В этом примере также показан другой способ создания пользовательского фильтра с помощью типов .NET.

```powershell
# Create a filter and get the entity to be updated.
[string]$filter = `
    [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("username",`
    [Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,"Jessie")
$user = Get-AzTableRow `
    -table $cloudTable `
    -customFilter $filter

# Change the entity.
$user.username = "Jessie2"

# To commit the change, pipe the updated record into the update cmdlet.
$user | Update-AzTableRow -table $cloudTable

# To see the new record, query the table.
Get-AzTableRow -table $cloudTable `
    -customFilter "(username eq 'Jessie2')"
```

В результатах показана запись Jessie2.

|поле|value|
|----|----|
| userid | 2 |
| username | Jessie2 |
| PartitionKey | partition2 |
| RowKey      | NM |

### <a name="deleting-table-entities"></a>Удаление сущностей таблицы

Вы можете удалить одну или все сущности в таблице.

#### <a name="deleting-one-entity"></a>Удаление одной сущности

Чтобы удалить одну сущность, получите ссылку на эту сущность и передайте ее в командлет **Remove-азтаблеров**.

```powershell
# Set filter.
[string]$filter = `
  [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("username",`
  [Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,"Jessie2")

# Retrieve entity to be deleted, then pipe it into the remove cmdlet.
$userToDelete = Get-AzTableRow `
    -table $cloudTable `
    -customFilter $filter
$userToDelete | Remove-AzTableRow -table $cloudTable

# Retrieve entities from table and see that Jessie2 has been deleted.
Get-AzTableRow -table $cloudTable | ft
```

#### <a name="delete-all-entities-in-the-table"></a>Удаление всех сущностей в таблице

Чтобы удалить все сущности в таблице, получите их и передайте результаты в командлет "remove". 

```powershell
# Get all rows and pipe the result into the remove cmdlet.
Get-AzTableRow `
    -table $cloudTable | Remove-AzTableRow -table $cloudTable 

# List entities in the table (there won't be any).
Get-AzTableRow -table $cloudTable | ft
```

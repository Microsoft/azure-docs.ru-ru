---
title: Выполнение операций в хранилище таблиц Azure с помощью PowerShell | Документация Майкрософт
description: Узнайте, как выполнять общие задачи, такие как создание запросов, удаление данных из учетной записи хранилища таблиц Azure с помощью PowerShell.
author: roygara
ms.service: storage
ms.topic: article
ms.date: 04/05/2019
ms.author: rogarana
ms.subservice: tables
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9a7502d48095fe18b983c1971d5145f51e766c95
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93306914"
---
# <a name="perform-azure-table-storage-operations-with-azure-powershell"></a>Выполнение операций в хранилище таблиц Azure с помощью Azure PowerShell 
[!INCLUDE [storage-table-cosmos-db-tip-include](../../../includes/storage-table-cosmos-db-langsoon-tip-include.md)]

Хранилище таблиц Azure — хранилище данных NoSQL, которое можно использовать для хранения и запросов огромных наборов структурированных нереляционных данных. Основными компонентами службы являются таблицы, сущности и свойства. Таблица представляет собой коллекцию сущностей. Сущность — это набор свойств. Каждая сущность может иметь до 252 свойств. Все они являются парами "имя — значение". В этой статье предполагается, что вы уже знакомы с понятиями хранилища таблиц Azure. Дополнительные сведения см. в статьях [Understanding the Table Service Data Model](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model) (Общие сведения о модели данных службы таблиц) и [Приступая к работе с хранилищем таблиц Azure с помощью .NET](../../cosmos-db/tutorial-develop-table-dotnet.md).

В этом практическом руководстве рассматриваются распространенные операции в хранилище таблиц Azure. Вы узнаете, как выполнять следующие задачи: 

> [!div class="checklist"]
> * Создание таблицы
> * извлечение таблицы;
> * Добавление сущностей таблицы
> * запрос к таблице;
> * удаление сущностей таблицы.
> * Удаление таблицы

В этом практическом руководстве показано, как создать учетную запись службы хранилища Azure в новой группе ресурсов, чтобы вы могли легко удалить ее при необходимости. Вы можете использовать существующую учетную запись хранения.

В примерах требуется AZ PowerShell modules `Az.Storage (1.1.0 or greater)` и `Az.Resources (1.2.0 or greater)` . В окне PowerShell выполните `Get-Module -ListAvailable Az*`, чтобы найти версию. Если версия не отображается или нужно установить обновление, ознакомьтесь со статьей [Install and configure Azure PowerShell](/powershell/azure/install-az-ps) (Установка и настройка Azure PowerShell).

> [!IMPORTANT]
> Для использования этой функции Azure от PowerShell требуется установленный модуль `Az`. Текущая версия `AzTable` несовместима с предыдущим модулем AzureRM.
> При необходимости выполните [последние инструкции по установке AZ Module](/powershell/azure/install-az-ps) .

После установки или обновления Azure PowerShell необходимо установить модуль **азтабле**, который содержит команды для управления сущностями. Чтобы установить этот модуль, запустите PowerShell от имени администратора и используйте команду **Install-Module**.

> [!IMPORTANT]
> Для совместимости имен модулей мы все еще публикуем тот же модуль под старым именем `AzureRmStorageTables` в коллекция PowerShell. Этот документ будет ссылаться только на новое имя.

```powershell
Install-Module AzTable
```

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью команды `Add-AzAccount` и следуйте инструкциям на экране.

```powershell
Add-AzAccount
```

## <a name="retrieve-list-of-locations"></a>Получение списка расположений

Если вы не знаете, какое расположение нужно использовать, можно получить список доступных расположений. Получив список, найдите расположение, которое нужно использовать. В этих примерах используется **eastus**. Сохраните это значение в переменной **location** для использования в будущем.

```powershell
Get-AzLocation | select Location
$location = "eastus"
```

## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). 

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Сохраните имя группы ресурсов в переменной для будущего использования. В этом примере создается группа ресурсов с именем *pshtablesrg* в регионе *eastus*.

```powershell
$resourceGroup = "pshtablesrg"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

## <a name="create-storage-account"></a>Создание учетной записи хранения

Создайте стандартную учетную запись хранения общего назначения с локально избыточным хранилищем (LRS) с помощью команды [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount). Обязательно укажите уникальное имя учетной записи хранения. Затем получите контекст, представляющий учетную запись хранения. При работе с учетной записью хранения можно ссылаться на контекст, а не повторять предоставление учетных данных.

```powershell
$storageAccountName = "pshtablestorage"
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind Storage

$ctx = $storageAccount.Context
```

## <a name="create-a-new-table"></a>Создание таблицы

Чтобы создать таблицу, используйте командлет [New-азсторажетабле](/powershell/module/az.storage/New-AzStorageTable) . В этом примере таблица называется `pshtesttable`.

```powershell
$tableName = "pshtesttable"
New-AzStorageTable –Name $tableName –Context $ctx
```

## <a name="retrieve-a-list-of-tables-in-the-storage-account"></a>Извлечение списка таблиц, содержащихся в учетной записи хранения

Получите список таблиц в учетной записи хранения с помощью команды [Get-азсторажетабле](/powershell/module/azure.storage/Get-AzureStorageTable).

```powershell
Get-AzStorageTable –Context $ctx | select Name
```

## <a name="retrieve-a-reference-to-a-specific-table"></a>Получение ссылки на определенную таблицу

Чтобы выполнить операции с таблицей, необходима ссылка на конкретную таблицу. Получите ссылку с помощью команды [Get-азсторажетабле](/powershell/module/azure.storage/Get-AzureStorageTable).

```powershell
$storageTable = Get-AzStorageTable –Name $tableName –Context $ctx
```

## <a name="reference-cloudtable-property-of-a-specific-table"></a>Ссылка на свойство CloudTable конкретной таблицы

> [!IMPORTANT]
> Использование CloudTable является обязательным при работе с модулем PowerShell **азтабле** . Чтобы получить ссылку на этот объект, вызовите команду **Get-азсторажетабле** . Эта команда также создает таблицу, если она еще не существует.

Для выполнения операций с таблицей с помощью **азтабле** требуется ссылка на свойство CloudTable определенной таблицы.

```powershell
$cloudTable = (Get-AzStorageTable –Name $tableName –Context $ctx).CloudTable
```

[!INCLUDE [storage-table-entities-powershell-include](../../../includes/storage-table-entities-powershell-include.md)]

## <a name="delete-a-table"></a>Удаление таблицы

Чтобы удалить таблицу, используйте [Remove-азсторажетабле](/powershell/module/az.storage/Remove-AzStorageTable). Этот командлет удаляет таблицу, включая все ее данные.

```powershell
Remove-AzStorageTable –Name $tableName –Context $ctx

# Retrieve the list of tables to verify the table has been removed.
Get-AzStorageTable –Context $Ctx | select Name
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы создали группу ресурсов и учетную запись хранения в начале этого практического руководства, можно удалить все ресурсы, созданные в этом упражнении, путем удаления группы ресурсов. Эта команда удаляет все ресурсы, которые содержатся в группе, и саму группу ресурсов.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

В этом практическом руководстве описаны распространенные операции в хранилище таблиц Azure, выполняемые с помощью PowerShell, в частности: 

> [!div class="checklist"]
> * Создание таблицы
> * извлечение таблицы;
> * Добавление сущностей таблицы
> * запрос к таблице;
> * удаление сущностей таблицы.
> * Удаление таблицы

Дополнительные сведения см. в следующих статьях:

* [Командлеты PowerShell для службы хранилища](/powershell/module/az.storage#storage)

* [Работа с таблицами Azure из PowerShell-Азурермсторажетабле/Азтабле PS v 2.0](https://paulomarquesc.github.io/working-with-azure-storage-tables-from-powershell)

* [Обозреватель хранилищ Microsoft Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.
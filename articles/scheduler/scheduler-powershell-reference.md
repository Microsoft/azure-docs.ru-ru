---
title: Справочник по командлетам PowerShell
description: Сведения о командлетах PowerShell для планировщика Azure
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: article
ms.date: 08/18/2016
ms.openlocfilehash: ccc9f709348d961e49bced00946658a6997837c0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92368117"
---
# <a name="powershell-cmdlets-reference-for-azure-scheduler"></a>Справочник по командлетам PowerShell для планировщика Azure

> [!IMPORTANT]
> Поддержка планировщика Azure [прекращается](../logic-apps/logic-apps-overview.md). Вместо него будет использоваться [Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Чтобы продолжить работу с заданиями, настроенными в планировщике, выполните [миграцию на Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) как можно скорее. 
>
> Планировщик больше не доступен на портале Azure, но [REST API](/rest/api/scheduler) и [командлеты PowerShell Планировщика Azure](scheduler-powershell-reference.md) можно использовать и дальше для управления заданиями и коллекциями заданий.

Чтобы писать сценарии для создания заданий планировщика и коллекций заданий и управления ими, можно использовать командлеты PowerShell. В этой статье перечислены основные командлеты PowerShell для планировщика Azure со ссылками на соответствующие справочные статьи. Сведения об установке Azure PowerShell для подписки Azure см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/). Дополнительные сведения о [командлетах Azure Resource Manager](/powershell/azure/)см. в разделе [использование Azure PowerShell с Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

| Командлет | Описание |
|--------|-------------|
| [Disable-Азсчедулержобколлектион](/powershell/module/azurerm.scheduler/disable-azurermschedulerjobcollection) |Отключает коллекцию заданий. |
| [Enable-AzureRmSchedulerJobCollection](/powershell/module/azurerm.scheduler/enable-azurermschedulerjobcollection) |Включает коллекцию заданий. |
| [Get-Азсчедулержоб](/powershell/module/azurerm.scheduler/get-azurermschedulerjob) |Получает задания планировщика. |
| [Get-Азсчедулержобколлектион](/powershell/module/azurerm.scheduler/get-azurermschedulerjobcollection) |Получает коллекции заданий. |
| [Get-Азсчедулержобхистори](/powershell/module/azurerm.scheduler/get-azurermschedulerjobhistory) |Получает журнал заданий. |
| [New-Азсчедулерхттпжоб](/powershell/module/azurerm.scheduler/new-azurermschedulerhttpjob) |Создает задание HTTP. |
| [New-Азсчедулержобколлектион](/powershell/module/azurerm.scheduler/new-azurermschedulerjobcollection) |Создает коллекцию заданий. |
| [New-Азсчедулерсервицебускуеуежоб](/powershell/module/azurerm.scheduler/new-azurermschedulerservicebusqueuejob) | Создает задание очереди служебной шины. |
| [New-Азсчедулерсервицебустопикжоб](/powershell/module/azurerm.scheduler/new-azurermschedulerservicebustopicjob) |Создает задание раздела служебной шины. |
| [New-Азсчедулерсторажекуеуежоб](/powershell/module/azurerm.scheduler/new-azurermschedulerstoragequeuejob) |Создает задание очереди хранилища. |
| [Remove-Азсчедулержоб](/powershell/module/azurerm.scheduler/remove-azurermschedulerjob) |Удаляет задание планировщика. |
| [Remove-Азсчедулержобколлектион](/powershell/module/azurerm.scheduler/remove-azurermschedulerjobcollection) |Удаляет коллекцию заданий. |
| [Set-Азсчедулерхттпжоб](/powershell/module/azurerm.scheduler/set-azurermschedulerhttpjob) |Изменяет задание HTTP планировщика. |
| [Set-Азсчедулержобколлектион](/powershell/module/azurerm.scheduler/set-azurermschedulerjobcollection) |Изменяет коллекцию заданий. |
| [Set-Азсчедулерсервицебускуеуежоб](/powershell/module/azurerm.scheduler/set-azurermschedulerservicebusqueuejob) |Изменяет задание очереди служебной шины. |
| [Set-Азсчедулерсервицебустопикжоб](/powershell/module/azurerm.scheduler/set-azurermschedulerservicebustopicjob) |Изменяет задание раздела служебной шины. |
| [Set-Азсчедулерсторажекуеуежоб](/powershell/module/azurerm.scheduler/set-azurermschedulerstoragequeuejob) |Изменяет задание очереди хранилища. |
||| 

Для получения дополнительных сведений можно выполнить любой из этих командлетов: 

```text
Get-Help <cmdlet name> -Detailed
Get-Help <cmdlet name> -Examples
Get-Help <cmdlet name> -Full
```

## <a name="next-steps"></a>Дальнейшие действия

* [Основные понятия, терминология и иерархия сущностей планировщика Azure](scheduler-concepts-terms.md)
* [Ограничения, значения по умолчанию и коды ошибок планировщика Azure](scheduler-limits-defaults-errors.md)
* [Справочник по API REST планировщика Azure](/rest/api/scheduler)
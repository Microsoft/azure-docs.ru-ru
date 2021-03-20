---
title: Управление службами Azure Analysis Services с помощью PowerShell | Документация Майкрософт
description: Описание командлетов PowerShell для Azure Analysis Services, используемых для распространенных административных задач, таких как создание серверов, приостановка операций или изменение уровня обслуживания.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 28f414c5eaaea7b987f2c3694cb8fc73b70838e9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92018786"
---
# <a name="manage-azure-analysis-services-with-powershell"></a>Управление службами Azure Analysis Services с помощью PowerShell

В этой статье описаны командлеты PowerShell, используемые для выполнения задач управления базами данных и сервером служб Azure Analysis Services. 

Для выполнения таких задач управления ресурсами сервера, как создание или удаление сервера, приостановка или возобновление работы сервера или изменение уровня обслуживания (уровня служб), используются командлеты Azure Analysis Services. Для выполнения других задач управления базами данных, таких как добавление или удаление участников роли, обработка или секционирование, используются командлеты, включенные в том же модуле SqlServer, что и в службах SQL Server Analysis Services.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="permissions"></a>Разрешения

Для большинства задач PowerShell требуется, чтобы у пользователя были привилегии администратора на сервере служб Analysis Services, которым он управляет. Запланированные задачи PowerShell являются автоматическими операциями. У учетной записи или субъекта-службы, запускающих планировщик, должны быть права администратора на сервере служб Analysis Services. 

Для операций сервера, использующих командлеты Azure PowerShell, ваша учетная запись или планировщик учетной записи также должен принадлежать к роли владельца для ресурса в [контроле доступа на основе ролей Azure (Azure RBAC)](../role-based-access-control/overview.md). 

## <a name="resource-and-server-operations"></a>Операции с ресурсами и серверами 

Установка модуля: [Az.AnalysisServices](https://www.powershellgallery.com/packages/Az.AnalysisServices)   
Документация: [справочные материалы по Az.AnalysisServices](/powershell/module/az.analysisservices)

## <a name="database-operations"></a>Операции с базой данных

Для операций с базами данных служб Azure Analysis Services используется тот же модуль SqlServer, что и для служб SQL Server Analysis Services. Однако для служб Azure Analysis Services поддерживаются не все командлеты. 

Модуль SqlServer предоставляет командлеты для конкретных задач управления базой данных, а также командлет общего назначения Invoke-ASCmd, который принимает запрос TMSL или сценарий. Для служб Azure Analysis Services поддерживаются следующие командлеты из модуля SqlServer.

Установка модуля: [SqlServer](https://www.powershellgallery.com/packages/SqlServer)   
Документация: [справочные материалы по SqlServer](/powershell/module/sqlserver)

### <a name="supported-cmdlets"></a>Поддерживаемые командлеты

|Командлет|Описание|
|------------|-----------------| 
|[Add-RoleMember](/powershell/module/sqlserver/Add-RoleMember)|Добавление участника в роль базы данных.| 
|[Backup-ASDatabase](/powershell/module/sqlserver/backup-asdatabase)|Архивация базы данных Analysis Services.|  
|[Remove-RoleMember](/powershell/module/sqlserver/remove-rolemember)|Удаление участника из роли базы данных.|   
|[Invoke-ASCmd](/powershell/module/sqlserver/invoke-ascmd)|Выполнение сценария TMSL.|
|[Invoke-ProcessASDatabase](/powershell/module/sqlserver/invoke-processasdatabase)|Обработка базы данных.|  
|[Invoke-ProcessPartition](/powershell/module/sqlserver/invoke-processpartition)|Обработка секции.| 
|[Invoke-ProcessTable](/powershell/module/sqlserver/invoke-processtable)|Обработка таблицы.|  
|[Merge-Partition](/powershell/module/sqlserver/merge-partition)|Объединение секции.|  
|[Restore-ASDatabase](/powershell/module/sqlserver/restore-asdatabase)|Восстановление базы данных Analysis Services.| 
  

## <a name="related-information"></a>Дополнительные сведения

* [SQL Server PowerShell](/sql/powershell/sql-server-powershell)      
* [Скачивание модуля PowerShell SQL Server](/sql/ssms/download-sql-server-ps-module)   
* [Скачивание SSMS](/sql/ssms/download-sql-server-management-studio-ssms)   
* [Модуль SqlServer в коллекции PowerShell](https://www.powershellgallery.com/packages/SqlServer)    
* [Программирование табличных моделей с уровнем совместимости 1200 и выше](/analysis-services/tabular-model-programming-compatibility-level-1200/tabular-model-programming-for-compatibility-level-1200)
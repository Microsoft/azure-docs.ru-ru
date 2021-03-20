---
title: sys.sp_cleanup_data_retention (Transact-SQL) — Azure SQL ребро
description: Дополнительные сведения об использовании sys.sp_cleanup_data_retention (Transact-SQL) в Azure SQL ребр
keywords: sys.sp_cleanup_data_retention (Transact-SQL), SQL ребро
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 9c0a6700a476d4f7f875af5373e3c99bc4e25af2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90938638"
---
# <a name="syssp_cleanup_data_retention-transact-sql"></a>sys.sp_cleanup_data_retention (Transact-SQL)

**Применимо к:** SQL Azure, граница

Выполняет очистку устаревших записей из таблиц, для которых включены политики хранения данных. Дополнительные сведения см. в разделе [хранение данных](data-retention-overview.md). 

## <a name="syntax"></a>Синтаксис 

```sql
sys.sp_cleanup_data_retention   
    { [@schema_name = ] 'schema_name' },  
    { [@table_name = ] 'table_name' },   
    [ [@rowcount =] rowcount OUTPUT ]    

```

## <a name="arguments"></a>Аргументы  
`[ @schema_name = ] schema_name`    
 Имя схемы-владельца для таблицы, для которой необходимо выполнить очистку. *schema_name* является обязательным параметром типа **sysname**.
  
`[ @table_name = ] 'table_name'`    
 Имя таблицы, для которой необходимо выполнить операцию очистки. *table_name* является обязательным параметром типа **sysname**.

## <a name="output-parameter"></a>Выходной параметр  

`[ @rowcount = ] rowcount OUTPUT`   
 ROWCOUNT — Необязательный выходной параметр, представляющий число записей, очищенных из таблицы. *количество строк* равно int.
  
## <a name="permissions"></a>Разрешения    
 Требуются db_owner разрешения.

## <a name="next-steps"></a>Дальнейшие действия
- [Хранение данных и автоматическая очистка данных](data-retention-overview.md)
- [Управление историческими данными с помощью политики хранения](data-retention-cleanup.md) 
- [Включение и отключение хранения данных](data-retention-enable-disable.md)

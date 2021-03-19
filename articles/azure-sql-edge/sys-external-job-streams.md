---
title: sys.external_job_streams (Transact-SQL) — Azure SQL ребро
description: Дополнительные сведения об использовании sys.external_job_streams в Azure SQL ребр
keywords: sys.external_job_streams, SQL для пограничных вычислений
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 35010d3aba7f6d5ee3185291c917ff7726ba8bd7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90900364"
---
# <a name="sysexternal_job_streams-transact-sql"></a>sys.external_job_streams (Transact-SQL)

Возвращает строку для каждого входного или выходного объекта внешнего потока, сопоставленного с заданием внешней потоковой передачи.

|Имя столбца|Тип данных|Описание|  
|-----------------|---------------|-----------------|
|**job_id**|**int**| Идентификационный номер для объекта задания потоковой передачи. Этот столбец сопоставляется со столбцом object_id для sys.external_streaming_jobs.|
|**stream_id**|**int**| Идентификационный номер объекта потока. Этот столбец сопоставляется со столбцом object_id для sys.external_streams. |
|**is_input**|**bit**| Значение 1, если объект потока использует входные данные для задания потоковой передачи. В противном случае — 0.|
|**is_output**|**bit**| Значение 1, если объект потока использует выходные данные для задания потоковой передачи. В противном случае — 0.|

## <a name="example"></a>Пример

Это представление каталога используется с представлениями каталога `sys.external_streams` и `sys.external_streaming_jobs`. Результат выполнения запроса показан ниже.

```sql
Select
    sj.Name as Job_Name,
    sj.Create_date as Job_Create_date,
    sj.modify_date as Job_Modify_date,
    sj.statement as Stream_Job_Query,
    Input_Stream_Name =
        Case js.is_input
            when 1 then s.Name
            else null
        END,
    output_Stream_Name =
        case js.is_output
            when 1 then s.Name
            else null
        END,
    s.location as Stream_Location
from sys.external_job_streams js
inner join sys.external_streams s on s.object_id = js.stream_id
inner join sys.external_streaming_jobs sj on sj.object_id = js.job_id
```

## <a name="permissions"></a>Разрешения

Видимость метаданных в представлениях каталогов ограничивается защищаемыми объектами, которыми пользователь владеет или на которые ему были предоставлены разрешения. Дополнительные сведения см. в разделе [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>См. также раздел

- [Представления каталога (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Системные представления (Transact-SQL)](/sql/t-sql/language-reference/)

---
title: Использование циклов T-SQL
description: Советы по использованию циклов T-SQL, замене курсоров и разработке связанных решений с помощью синапсе SQL в Azure синапсе Analytics.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 0d83e1305a851bf6bafb6c4c79f5caf73f8e44b6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98120892"
---
# <a name="use-t-sql-loops-with-synapse-sql-in-azure-synapse-analytics"></a>Использование циклов T-SQL с синапсе SQL в Azure синапсе Analytics

В этой статье приводятся ключевые советы по использованию циклов T-SQL, замене курсоров и разработке связанных решений с помощью синапсе SQL.

## <a name="purpose-of-while-loops"></a>Назначение циклов WHILE

Синапсе SQL поддерживает цикл [while](/sql/t-sql/language-elements/while-transact-sql?preserve-view=true&view=sql-server-ver15) для многократного выполнения блоков инструкций. Цикл WHILE продолжается, пока не будут выполнены указанные условия или пока код не прервет цикл с помощью ключевого слова BREAK. 

Циклы в синапсе SQL полезны для замены курсоров, определенных в коде SQL. К счастью, почти все курсоры, записанные в коде SQL, относятся к разряду перемотки и доступности только для чтения. Таким образом, циклы WHILE — отличная альтернатива замене курсоров.

## <a name="replace-cursors-in-synapse-sql"></a>Замена курсоров в синапсе SQL

Прежде чем углубляться в, следует принять во внимание следующий вопрос: "можно ли переписывать этот курсор для использования операций на основе наборов?" Во многих случаях ответ имеет значение Да и часто является лучшим подходом. Операция, основанная на множестве, часто выполняется быстрее, чем методом итеративного, построчного по строкам.

Курсоры быстрого перемотки только на чтение легко заменяются циклической конструкцией. Ниже приведен простой пример кода. Приведенный в примере код обновляет статистику для каждой таблицы в базе данных. Перебор таблиц в цикле позволяет выполнить каждую команду в последовательности.

Для начала создайте временную таблицу с уникальным номером строки, обозначающим отдельные операторы:

```sql
CREATE TABLE #tbl
WITH
( DISTRIBUTION = ROUND_ROBIN
)
AS
SELECT  ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) AS Sequence
,       [name]
,       'UPDATE STATISTICS '+QUOTENAME([name]) AS sql_code
FROM    sys.tables
;
```

Во вторых, инициализируйте переменные, необходимые для выполнения цикла:

```sql
DECLARE @nbr_statements INT = (SELECT COUNT(*) FROM #tbl)
,       @i INT = 1
;
```

Теперь запустите цикл последовательного выполнения операторов:

```sql
WHILE   @i <= @nbr_statements
BEGIN
    DECLARE @sql_code NVARCHAR(4000) = (SELECT sql_code FROM #tbl WHERE Sequence = @i);
    EXEC    sp_executesql @sql_code;
    SET     @i +=1;
END
```

И, наконец, удалите временную таблицу, созданной на первом этапе:

```sql
DROP TABLE #tbl;
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные советы по разработке приведены в [обзоре разработки](develop-overview.md).
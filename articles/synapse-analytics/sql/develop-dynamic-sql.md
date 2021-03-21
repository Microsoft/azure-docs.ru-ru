---
title: Использование динамического SQL в синапсе SQL
description: Советы по использованию динамического SQL в синапсе SQL.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: 86d5028a09a805142f7632f93530f8a54965d82f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101675037"
---
# <a name="dynamic-sql-in-synapse-sql"></a>Динамический SQL в синапсе SQL

В этой статье вы найдете советы по использованию динамического SQL и разработке решений с помощью синапсе SQL.

## <a name="dynamic-sql-example"></a>Пример динамического SQL

При разработке кода приложения может потребоваться использовать динамический SQL, чтобы обеспечить гибкие, универсальные и модульные решения.

> [!NOTE]
> В настоящее время выделенный пул SQL не поддерживает типы данных больших двоичных объектов. Это может ограничивать размер строк, так как к двоичным относятся типы данных nvarchar(max) и varchar(max). Если вы использовали эти типы в коде приложения при создании очень больших строк, необходимо будет разбить код на фрагменты и вместо этого использовать инструкцию EXEC.

Вот простой пример:

```sql
DECLARE @sql_fragment1 VARCHAR(8000)=' SELECT name '
,       @sql_fragment2 VARCHAR(8000)=' FROM sys.system_views '
,       @sql_fragment3 VARCHAR(8000)=' WHERE name like ''%table%''';

EXEC( @sql_fragment1 + @sql_fragment2 + @sql_fragment3);
```

Если строка короткая, то можно использовать [sp_executesql](/sql/relational-databases/system-stored-procedures/sp-executesql-transact-sql?view=azure-sqldw-latest&preserve-view=true) как обычно.

> [!NOTE]
> Инструкции, выполняемые как динамические SQL, будут по-прежнему подвергаться всем правилам проверки T-SQL.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные советы по разработке приведены в [обзоре разработки](develop-overview.md).

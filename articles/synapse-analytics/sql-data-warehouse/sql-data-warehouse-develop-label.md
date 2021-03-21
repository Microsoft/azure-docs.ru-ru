---
title: Использование меток для инструментирования запросов
description: Советы по использованию меток для инструментирования запросов для выделенных пулов SQL в Azure синапсе Analytics.
services: synapse-analytics
author: MSTehrani
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: emtehran
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 7958caf71658dcdcbf31bac84697931e9049452f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96462751"
---
# <a name="using-labels-to-instrument-queries-for-dedicated-sql-pools-in-azure-synapse-analytics"></a>Использование меток для инструментирования запросов для выделенных пулов SQL в Azure синапсе Analytics

В эту статью включены советы по разработке решений с помощью меток для инструментирования запросов в выделенных пулах SQL.

## <a name="what-are-labels"></a>Что такое метки

Выделенный пул SQL поддерживает концепцию, называемую метками запросов. Прежде чем углубляться в это понятие, рассмотрим пример.

```sql
SELECT *
FROM sys.tables
OPTION (LABEL = 'My Query Label')
;
```

Последняя строка добавляет к запросу тег "Метка моего запроса". Этот тег полезен, так как метка может выполнять запросы через динамические административные представления.

Использование меток в запросах позволяет находить проблемные запросы и помогает определять ход выполнения посредством извлечения, преобразования и загрузки.

Хорошо продуманный способ именования упрощает работу с метками. Например, запуск метки с помощью проекта, процедуры, инструкции или комментария однозначно определяет запрос для всего кода в системе управления версиями.

Следующий запрос использует динамическое административное представление для поиска по метке:

```sql
SELECT  *
FROM    sys.dm_pdw_exec_requests r
WHERE   r.[label] = 'My Query Label'
;
```

> [!NOTE]
> Слово label в запросе необходимо заключать в квадратные скобки или двойные кавычки. Label — зарезервированное слово. Его использование без разделителя приводит к ошибке.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные советы по разработке приведены в [обзоре разработки](sql-data-warehouse-overview-develop.md).

---
title: Безопасность на уровне столбцов для выделенного пула SQL
description: С помощью функции безопасности на уровне столбцов можно управлять доступом к столбцам таблицы в базе данных на основе контекста выполнения пользователя или членства в группе. Это упрощает проектирование и реализацию системы безопасности в приложении и позволяет устанавливать ограничения на доступ к столбцам.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/19/2020
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019
tags: azure-synapse
ms.openlocfilehash: fb34051f7d4b24190806dde939c8cc6d9c2a4896
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98679953"
---
# <a name="column-level-security"></a>Защита на уровне столбцов

Column-Level безопасность позволяет клиентам управлять доступом к столбцам таблицы на основе контекста выполнения или членства пользователя в группе.

> [!VIDEO https://www.youtube.com/embed/OU_ESg0g8r8]
Так как это видео было опубликовано как [безопасность на уровне строк](/sql/relational-databases/security/row-level-security?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) стало доступным для выделенного пула SQL в Azure синапсе.

Безопасность на уровне столбцов упрощает проектирование и реализацию безопасности в приложении, позволяя ограничивать доступ к столбцам для защиты конфиденциальных данных. Например, можно предоставить конкретным пользователям доступ только к определенным столбцам таблицы, имеющей отношение к их отделу. Логика ограничения находится на уровне базы данных, а не на отдалении от данных на другом уровне приложения. База данных применяет ограничения доступа при каждой попытке получения доступа к данным независимо от уровня. Это ограничение делает систему безопасности более надежной и устойчивой за счет уменьшения ее контактной зоны. Кроме того, безопасность на уровне столбцов также позволяет отказаться от ввода представлений для фильтрации столбцов с целью наложения на пользователей ограничений доступа.

Безопасность на уровне столбцов можно реализовать с помощью инструкции T-SQL [Grant](/sql/t-sql/statements/grant-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) . При использовании этого механизма поддерживается проверка подлинности SQL и Azure Active Directory (Azure AD).

![На схеме показана схематическое таблица с первым столбцом, который направляется закрытым замком, и его ячейками — оранжевым цветом, тогда как остальные столбцы — это белые ячейки.](./media/column-level-security/cls.png)

## <a name="syntax"></a>Синтаксис

```syntaxsql
GRANT <permission> [ ,...n ] ON
    [ OBJECT :: ][ schema_name ]. object_name [ ( column [ ,...n ] ) ]
    TO <database_principal> [ ,...n ]
    [ WITH GRANT OPTION ]
    [ AS <database_principal> ]
<permission> ::=
    SELECT
  | UPDATE
<database_principal> ::=
      Database_user
    | Database_role
    | Database_user_mapped_to_Windows_User
    | Database_user_mapped_to_Windows_Group
```

## <a name="example"></a>Пример

В следующем примере показано, как ограничить доступ пользователя `TestUser` к столбцу `SSN` таблицы `Membership`:

Создайте таблицу `Membership` со столбцом SSN для хранения номеров социального страхования:

```sql
CREATE TABLE Membership
  (MemberID int IDENTITY,
   FirstName varchar(100) NULL,
   SSN char(9) NOT NULL,
   LastName varchar(100) NOT NULL,
   Phone varchar(12) NULL,
   Email varchar(100) NULL);
```

Предоставьте пользователю `TestUser` доступ ко всем столбцам, за исключением столбца SSN, который содержит конфиденциальные данные:

```sql
GRANT SELECT ON Membership(MemberID, FirstName, LastName, Phone, Email) TO TestUser;
```

Запросы, выполняемые от имени `TestUser`, завершатся ошибкой, если они содержат столбец SSN:

```sql
SELECT * FROM Membership;

-- Msg 230, Level 14, State 1, Line 12
-- The SELECT permission was denied on the column 'SSN' of the object 'Membership', database 'CLS_TestDW', schema 'dbo'.
```

## <a name="use-cases"></a>Варианты использования

Ниже приведены некоторые примеры использования безопасности на уровне столбцов в современном мире.

- Фирма финансовых услуг позволяет только менеджерам по учетным записям иметь доступ к номерам социального страхования (SSN), номерам телефонов и другим личным данным клиента.
- В медицинской организации доступ к конфиденциальным медицинским записям имеют только врачи и медсестры. Сотрудники отдела выставления счетов не могут просматривать эти данные.

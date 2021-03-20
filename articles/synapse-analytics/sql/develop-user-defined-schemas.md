---
title: Определяемые пользователем схемы в синапсе SQL
description: В следующих разделах вы найдете различные советы по использованию определяемых пользователем схем T-SQL для разработки решений с возможностью синапсе SQL Azure синапсе Analytics.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 6920d6cd190927747a69fc4dc0f0791278665abe
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96451700"
---
# <a name="user-defined-schemas-within-synapse-sql"></a>Определяемые пользователем схемы в синапсе SQL

В следующих разделах вы найдете различные советы по использованию определяемых пользователем схем T-SQL для разработки решений в синапсе SQL.

## <a name="schemas-for-application-boundaries"></a>Схемы для границ приложений

В традиционной архитектуре аналитики часто используются отдельные базы данных для создания границ приложений на основе рабочей нагрузки, домена или безопасности. Например, традиционная инфраструктура SQL Server Analytics может включать в себя промежуточную базу данных, базу данных аналитики и базы данных киоска данных. В этой топологии каждая база данных действует как Рабочая нагрузка и граница безопасности в архитектуре.

Вместо этого синапсе SQL выполняет всю рабочую нагрузку аналитики в пределах одной базы данных. Межбазовые подключения к базам данных не разрешены. Синапсе SQL ждет, что все таблицы, используемые хранилищем, хранятся в одной базе данных.

> [!NOTE]
> Выделенные пулы SQL не поддерживают перекрестные запросы к базам данных любого типа. Поэтому необходимо будет пересмотреть реализации аналитики, использующие этот шаблон. Бессерверный пул SQL поддерживает запросы между базами данных.

## <a name="user-defined-schema-recommendations"></a>Рекомендации для определяемых пользователем схем

Включены рекомендации по консолидации рабочих нагрузок, безопасности, домена и функциональных границ с помощью определяемых пользователем схем.

- Используйте одну базу данных для выполнения всей рабочей нагрузки аналитики.
- Объедините существующую среду аналитики для использования одной базы данных.
- Используйте **определяемые пользователем схемы** , чтобы создать границы, ранее реализуемые с помощью баз данных.

Если определяемые пользователем схемы еще не использовались ранее, то у вас есть чистый планшет. Используйте старое имя базы данных в качестве базиса для определяемых пользователем схем в базе данных SQL синапсе.

Если схемы уже используются, есть несколько вариантов:

- Удалить прежние имена схем и создать новые.
- Не используйте устаревшие имена схем, предваряя прежнее имя схемы до имени таблицы.
- Сохраняйте устаревшие имена схем, реализовав представления таблицы в дополнительной схеме, которая повторно создает старую структуру схемы.

> [!NOTE]
> При первой проверке вариант 3 может показаться наиболее привлекательным выбором. Представления доступны только для чтения в синапсе SQL. Любое изменение данных или таблицы потребуется выполнять в базовой таблице. Кроме того, вариант 3 добавляет в систему уровень представлений. Если вы уже используете представления в архитектуре, вам может потребоваться сделать это еще лучше.
> 
> 

### <a name="examples"></a>Примеры

Реализуйте определяемые пользователем схемы на основе имен баз данных.

```sql
CREATE SCHEMA [stg]; -- stg previously database name for staging database
GO
CREATE SCHEMA [edw]; -- edw previously database name for the analytics
GO
CREATE TABLE [stg].[customer] -- create staging tables in the stg schema
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[customer] -- create analytics tables in the edw schema
(       CustKey BIGINT NOT NULL
,       ...
);
```

Используйте устаревшие имена схем, предваряя их именами таблиц. Используйте схемы для создания границы рабочих нагрузок.

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- edw defines the analytics boundary
GO
CREATE TABLE [stg].[dim_customer] --pre-pend the old schema name to the table and create in the staging boundary
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[dim_customer] --pre-pend the old schema name to the table and create in the analytics boundary
(       CustKey BIGINT NOT NULL
,       ...
);
```

Сохраняйте устаревшие имена схем с помощью представлений.

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- stg defines the analytics boundary
GO
CREATE SCHEMA [dim]; -- edw defines the legacy schema name boundary
GO
CREATE TABLE [stg].[customer] -- create the base staging tables in the staging boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE TABLE [edw].[customer] -- create the base analytics tables in the analytics boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE VIEW [dim].[customer] -- create a view in the legacy schema name boundary for presentation consistency purposes only
AS
SELECT  CustKey
,       ...
FROM    [edw].customer
;
```

> [!NOTE]
> Любое изменение стратегии схемы требует проверки модели безопасности для базы данных. Во многих случаях вы можете упростить модель безопасности, назначив разрешения на уровне схемы.

Если требуются более детализированные разрешения, можно использовать роли базы данных. Дополнительные сведения о ролях базы данных см. в статье [Управление ролями и пользователями базы данных](../../analysis-services/analysis-services-database-users.md) .

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные советы по разработке приведены в [обзоре разработки Synapse SQL](develop-overview.md).

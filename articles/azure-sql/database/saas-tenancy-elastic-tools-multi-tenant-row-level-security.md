---
title: Приложения с несколькими клиентами с помощью инструментов RLS и эластичных баз данных
description: Средства эластичных баз данных в сочетании с безопасностью на уровне строк позволяют создать приложение с высокой масштабируемостью уровня данных.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: VanMSFT
ms.author: vanto
ms.reviewer: sstein
ms.date: 12/18/2018
ms.openlocfilehash: 6d753a90f2a4cb19c9f3933d007fb3d378af6d81
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92793217"
---
# <a name="multi-tenant-applications-with-elastic-database-tools-and-row-level-security"></a>Мультитенантные приложения со средствами эластичных баз данных и безопасностью на уровне строк
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Используемые вместе [средства эластичных баз данных](elastic-scale-get-started.md) и функция [безопасности на уровне строк (RLS)][rls] обеспечивают масштабирование уровня данных для мультитенантного приложения с помощью Базы данных SQL Azure. Вместе эти технологии позволяют создавать приложения с высокой степенью масштабирования уровня данных. Уровень данных поддерживает мультитенантные сегменты и использует **ADO.NET SqlClient** или **Entity Framework**. Дополнительные сведения см. в статье [Мультитенантные шаблоны клиента в Базе данных SaaS](./saas-tenancy-app-design-patterns.md).

- **Средства эластичных баз данных** позволяют разработчикам развернуть уровень данных, используя стандартные методы сегментирования, набор библиотек .NET и шаблоны служб Azure. Управление сегментами с использованием [клиентской библиотеки эластичных баз данных][s-d-elastic-database-client-library] помогает автоматизировать и упростить многие инфраструктурные задачи, которые обычно связаны с сегментированием.
- Функция **безопасности на уровне строк** позволяет безопасно хранить данные нескольких клиентов в одной базе данных. Политики безопасности на уровне строк автоматически исключают из результатов запроса все строки, не принадлежащие клиенту. Централизованная логика фильтрации в базе данных упрощает обслуживание и минимизирует риск снижения безопасности. Без нее пришлось бы полагаться на реализацию безопасности в клиентском коде, что более рискованно.

При совместном использовании этих функций приложение может хранить данные для нескольких клиентов в одной сегментированной базе данных. Несколько клиентов использую одну базу данных, и каждому из них это обойдется дешевле. При этом вы можете в том же приложении за отдельную плату предоставлять клиентам уровня "Премиум" выделенные и изолированные сегменты базы данных. Одним из преимуществ изоляции одного клиента являются более строгие гарантии производительности. В базе данных одного клиента не возникает конкуренции за ресурсы с другими клиентами.

Идея заключается в том, чтобы с помощью API-интерфейсов [маршрутизации на основе данных](elastic-scale-data-dependent-routing.md) из клиентской библиотеки эластичной базы данных организовать автоматическое подключение каждого клиента к правильному сегменту базы данных. Только один сегмент содержит определенное значение идентификатора клиента. Идентификатор клиента является *ключом сегментирования*. В пределах подключенной базы данных применяется политика безопасности на уровне строк. Она предоставляет конкретному клиенту доступ только к тем строкам данных, которые содержат его идентификатор клиента.

> [!NOTE]
> Этот идентификатор может состоять из нескольких столбцов. Для простоты в этой статье мы предполагаем, что используется идентификатор клиента из одного столбца.

![Архитектура приложений для ведения блогов][1]

## <a name="download-the-sample-project"></a>Загрузка примера проекта

### <a name="prerequisites"></a>Предварительные условия

- Используйте Visual Studio 2012 или более поздней версии.
- Создание трех баз данных в базе данных SQL Azure
- Скачайте пример проекта: [Elastic DB Tools for Azure SQL — Multi-Tenant Shards](https://go.microsoft.com/?linkid=9888163)
  - Заполните информацию по своим базам данных в начале файла **Program.cs**

Этот проект расширяет проект, описанный в [Elastic DB Tools for Azure SQL — Entity Framework Integration](elastic-scale-use-entity-framework-applications-visual-studio.md) , за счет добавления поддержки для баз данных с мультитенантными сегментами. При сборке этого проекта создается простое консольное приложение для создания блогов и записей в них. Проект уже содержит четыре клиента и две мультитенантные базы данных сегмента. Эта конфигурация отражена на схеме выше.

Выполните сборку и запуск приложения. Приложение откроет диспетчер карт сегментов из средств эластичных баз данных и выполнит следующие проверки:

1. С помощью Entity Framework и LINQ, создайте новый блог, а затем отобразите все блоги для каждого клиента.
2. С помощью ADO.NET SqlClient отобразите все блоги для клиента.
3. Попробуйте вставить блог неправильного клиента, чтобы проверить, возникнет ли ошибка.

Обратите внимание, что поскольку RLS пока не включена в базах данных сегментов, каждая из этих проверок выдает ошибки: клиенты видят блоги, которые им не принадлежат, а приложение не запрещается вставить блог неправильному клиенту. В оставшейся части этой статьи описано, как решить эти проблемы, применив изоляцию клиентов с помощью RLS. Необходимы два действия.

1. **Уровень приложений.** Измените код приложения, чтобы после создания подключения в SESSION\_CONTEXT всегда хранилось текущее значение идентификатора клиента. В нашем примере идентификатор клиента уже хранится таким образом.
2. **Уровень данных.** Создайте в каждой базе данных сегмента политику безопасности на уровне строк, чтобы фильтровать строки по значению идентификатора клиента из SESSION\_CONTEXT. Политику необходимо создать для каждой базы данных сегмента, иначе строки в мультитенантных сегментах не будут фильтроваться.

## <a name="1-application-tier-set-tenantid-in-the-session_context"></a>1. уровень приложения: задайте значение TenantId в контексте сеанса. \_

Прежде всего, подключитесь к базе данных сегмента, используя API-интерфейсы маршрутизации на основе данных из клиентской библиотеки эластичной базы данных. Теперь приложение должно сообщить базе данных, для какого идентификатора клиента создано это подключение. Политика безопасности на уровне строк использует этот идентификатор клиента, чтобы исключать все строки, принадлежащие другим клиентам. Сохраните текущий идентификатор клиента в [SESSION\_CONTEXT](/sql/t-sql/functions/session-context-transact-sql) для созданного подключения.

Вместо SESSION\_CONTEXT можно также использовать [CONTEXT\_INFO](/sql/t-sql/functions/context-info-transact-sql). Но мы рекомендуем применять именно SESSION\_CONTEXT. SESSION\_CONTEXT проще использовать, так как при этом по умолчанию возвращается значение NULL, а также поддерживаются пары "ключ — значение".

### <a name="entity-framework"></a>Entity Framework

Для приложений, использующих Entity Framework, проще всего задать SESSION\_CONTEXT в рамках переопределения ElasticScaleContext, как описано в разделе о [маршрутизации на основе данных с использованием EF DbContext](elastic-scale-use-entity-framework-applications-visual-studio.md#data-dependent-routing-using-ef-dbcontext). Создайте и выполните команду SqlCommand, которая сохраняет идентификатор клиента в качестве значения shardingKey в SESSION\_CONTEXT текущего подключения. Теперь мы вернем подключение, установленное с применением маршрутизации на основе данных. Таким образом, вам нужен всего один сегмент кода, чтобы задать нужное значение в SESSION\_CONTEXT.

```csharp
// ElasticScaleContext.cs
// Constructor for data-dependent routing.
// This call opens a validated connection that is routed to the
// proper shard by the shard map manager.
// Note that the base class constructor call fails for an open connection
// if migrations need to be done and SQL credentials are used.
// This is the reason for the separation of constructors.
// ...
public ElasticScaleContext(ShardMap shardMap, T shardingKey, string connectionStr)
    : base(
        OpenDDRConnection(shardMap, shardingKey, connectionStr),
        true)  // contextOwnsConnection
{
}

public static SqlConnection OpenDDRConnection(
    ShardMap shardMap,
    T shardingKey,
    string connectionStr)
{
    // No initialization.
    Database.SetInitializer<ElasticScaleContext<T>>(null);

    // Ask shard map to broker a validated connection for the given key.
    SqlConnection conn = null;
    try
    {
        conn = shardMap.OpenConnectionForKey(
            shardingKey,
            connectionStr,
            ConnectionOptions.Validate);

        // Set TenantId in SESSION_CONTEXT to shardingKey
        // to enable Row-Level Security filtering.
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText =
            @"exec sp_set_session_context
                @key=N'TenantId', @value=@shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", shardingKey);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }
        throw;
    }
}
// ...
```

Теперь при каждом вызове ElasticScaleContext в SESSION\_CONTEXT автоматически сохраняется нужное значение идентификатора клиента.

```csharp
// Program.cs
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (var db = new ElasticScaleContext<int>(
        sharding.ShardMap, tenantId, connStrBldr.ConnectionString))
    {
        var query = from b in db.Blogs
                    orderby b.Name
                    select b;

        Console.WriteLine("All blogs for TenantId {0}:", tenantId);
        foreach (var item in query)
        {
            Console.WriteLine(item.Name);
        }
    }
});
```

### <a name="adonet-sqlclient"></a>ADO.NET SqlClient

Для приложений, которые используют ADO.NET SqlClient, создайте функцию-оболочку для метода ShardMap.OpenConnectionForKey. Эта оболочка должна автоматически сохранять значение идентификатора клиента в SESSION\_CONTEXT, прежде чем возвращать подключение. Чтобы в SESSION\_CONTEXT всегда содержалось нужное значение, открывайте подключения только с помощью этой функции-оболочки.

```csharp
// Program.cs
// Wrapper function for ShardMap.OpenConnectionForKey() that
// automatically sets SESSION_CONTEXT with the correct
// tenantId before returning a connection.
// As a best practice, you should only open connections using this method
// to ensure that SESSION_CONTEXT is always set before executing a query.
// ...
public static SqlConnection OpenConnectionForTenant(
    ShardMap shardMap, int tenantId, string connectionStr)
{
    SqlConnection conn = null;
    try
    {
        // Ask shard map to broker a validated connection for the given key.
        conn = shardMap.OpenConnectionForKey(
            tenantId, connectionStr, ConnectionOptions.Validate);

        // Set TenantId in SESSION_CONTEXT to shardingKey
        // to enable Row-Level Security filtering.
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText =
            @"exec sp_set_session_context
                @key=N'TenantId', @value=@shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", tenantId);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }
        throw;
    }
}

// ...

// Example query via ADO.NET SqlClient.
// If row-level security is enabled, only Tenant 4's blogs are listed.
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (SqlConnection conn = OpenConnectionForTenant(
        sharding.ShardMap, tenantId4, connStrBldr.ConnectionString))
    {
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText = @"SELECT * FROM Blogs";

        Console.WriteLine(@"--
All blogs for TenantId {0} (using ADO.NET SqlClient):", tenantId4);

        SqlDataReader reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine("{0}", reader["Name"]);
        }
    }
});

```

## <a name="2-data-tier-create-row-level-security-policy"></a>2. уровень данных: Создание политики безопасности на уровне строк

### <a name="create-a-security-policy-to-filter-the-rows-each-tenant-can-access"></a>Создание политики безопасности для фильтрации строк, доступных для всех клиентов

Теперь перед любым запросом приложение сохраняет текущее значение идентификатора клиента в SESSION\_CONTEXT, что позволяет политике безопасности на уровне строк исключать при обработке запроса все строки с другими значениями идентификатора клиента.

Политика безопасности на уровне строк реализуется в инструкциях Transact-SQL. Определяемая пользователем функция описывает логику доступа, а политика безопасности связывает эту функцию с любым количеством таблиц. В нашем проекте реализовано следующее:

1. Функция проверяет, что к базе данных подключено приложение и что значение TenantId из SESSION\_CONTEXT соответствует значению TenantId для определенной строки.
    - Здесь важно, что подключено именно наше приложение, а не другой пользователь SQL.

2. Предикат FILTER позволяет применять запросы SELECT, UPDATE и DELETE ко всем строкам, которые удовлетворяют условиям фильтрования по идентификатору клиента.
    - Предикат BLOCK запрещает создавать (INSERT) или обновлять (UPDATE) строки, не соответствующие этому фильтру.
    - Если значение SESSION\_CONTEXT не задано, функция возвращает значение NULL и запрещает просмотр или добавление любых строк.

Чтобы включить безопасность на уровне строк для всех сегментов, выполните следующий запрос T-SQL с помощью Visual Studio (SSDT), SSMS или скрипта PowerShell, который включен в этот проект. Если же вы используете [задания обработки эластичных баз данных](./elastic-jobs-overview.md), можно автоматизировать выполнение инструкции T-SQL для всех сегментов.

```sql
CREATE SCHEMA rls; -- Separate schema to organize RLS objects.
GO

CREATE FUNCTION rls.fn_tenantAccessPredicate(@TenantId int)
    RETURNS TABLE
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_accessResult
        -- Use the user in your application’s connection string.
        -- Here we use 'dbo' only for demo purposes!
        WHERE DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo')
        AND CAST(SESSION_CONTEXT(N'TenantId') AS int) = @TenantId;
GO

CREATE SECURITY POLICY rls.tenantAccessPolicy
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Blogs,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Blogs,
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Posts,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Posts;
GO
```

> [!TIP]
> В сложных проектах предикаты придется добавлять в несколько сотен таблиц, что становится утомительной задачей. Но есть вспомогательная хранимая процедура, которая автоматически создает политику безопасности и добавляет предикат ко всем таблицам в схеме. См. запись блога [о применении безопасности на уровне строк ко всем таблицам с помощью вспомогательного скрипта](https://techcommunity.microsoft.com/t5/sql-server/apply-row-level-security-to-all-tables-helper-script/ba-p/384360).

Теперь если вы снова запустите пример приложения, клиенты увидят только те строки, которые им принадлежат. Кроме того, приложение не может вставлять строки, принадлежащие другим клиентам, кроме подключенного в текущий момент к базе данных сегмента. Не сможет оно и изменить идентификатор клиента в любой строке, даже доступной для просмотра. При попытке выполнения любого из этих действий создается исключение DbUpdateException.

Если позже вы добавите новую таблицу, измените политику безопасности с помощью запроса ALTER, добавив в нее предикаты FILTER и BLOCK для новой таблицы.

```sql
ALTER SECURITY POLICY rls.tenantAccessPolicy
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.MyNewTable,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.MyNewTable;
GO
```

### <a name="add-default-constraints-to-automatically-populate-tenantid-for-inserts"></a>Добавление ограничений по умолчанию на автоматическое заполнение TenantId для команд INSERT

Для каждой таблицы можно установить стандартное ограничение, чтобы вставка строк столбец сопровождалась автоматическим заполнением идентификатора клиента соответствующим значением из SESSION\_CONTEXT. См. пример ниже.

```sql
-- Create default constraints to auto-populate TenantId with the
-- value of SESSION_CONTEXT for inserts.
ALTER TABLE Blogs
    ADD CONSTRAINT df_TenantId_Blogs
    DEFAULT CAST(SESSION_CONTEXT(N'TenantId') AS int) FOR TenantId;
GO

ALTER TABLE Posts
    ADD CONSTRAINT df_TenantId_Posts
    DEFAULT CAST(SESSION_CONTEXT(N'TenantId') AS int) FOR TenantId;
GO
```

Теперь приложению не требуется указывать TenantId при вставке строк:

```csharp
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (var db = new ElasticScaleContext<int>(
        sharding.ShardMap, tenantId, connStrBldr.ConnectionString))
    {
        // The default constraint sets TenantId automatically!
        var blog = new Blog { Name = name };
        db.Blogs.Add(blog);
        db.SaveChanges();
    }
});
```

> [!NOTE]
> Если вы используете ограничения по умолчанию для проекта Entity Framework, мы рекомендуем *НЕ* включать столбец идентификатора клиента (TenantId) в модель данных EF. Дело в том, что запросы Entity Framework автоматически поставляют значения по умолчанию, которые переопределят стандартное ограничение на основе SESSION\_CONTEXT, созданное в инструкции T-SQL.
> Чтобы использовать ограничения по умолчанию, например в примере проекта, следует удалить TenantId из файла DataClasses.cs (и запустить Add-Migration в консоли диспетчера пакетов) и воспользоваться T-SQL, чтобы убедиться, что это поле существует только в таблицах базы данных. В этом случае EF не подставляет неправильные значения по умолчанию при вставке данных.

### <a name="optional-enable-a-superuser-to-access-all-rows"></a>(Необязательно) Предоставление *суперпользователю* доступа ко всем строкам

Для некоторых приложений будет удобно создать *суперпользователя* с доступом ко всем строкам. Роль суперпользователя позволит составлять отчеты по всем клиентам, размещенным во всех сегментах. Также эта роль удобна для операций слияния и разделения сегментов, при которых строки разных клиентов перемещаются между базами данных.

Для этого создайте пользователя SQL (в нашем примере это `superuser`) в каждой базе данных сегмента. Затем измените политику безопасности, внеся в нее новую функцию предиката, которая разрешит новому пользователю доступ ко всем строкам. Пример такой функции приводится ниже.

```sql
-- New predicate function that adds superuser logic.
CREATE FUNCTION rls.fn_tenantAccessPredicateWithSuperUser(@TenantId int)
    RETURNS TABLE
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_accessResult
        WHERE
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo') -- Replace 'dbo'.
            AND CAST(SESSION_CONTEXT(N'TenantId') AS int) = @TenantId
        )
        OR
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('superuser')
        );
GO

-- Atomically swap in the new predicate function on each table.
ALTER SECURITY POLICY rls.tenantAccessPolicy
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Blogs,
    ALTER BLOCK  PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Blogs,
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Posts,
    ALTER BLOCK  PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Posts;
GO
```


### <a name="maintenance"></a>Обслуживание

- **Добавление новых сегментов.** Выполните скрипт T-SQL, чтобы включить политику RLS для всех новых сегментов, иначе запросы в таких сегментах фильтроваться не будут.
- **Добавление новых таблиц.** Добавляйте предикаты FILTER и BLOCK в политику безопасности на всех сегментах при создании любой новой таблицы. Без этого запросы к такой таблице фильтроваться не будут. Добавление предикатов можно автоматизировать с помощью триггера DDL, как это описано в записи об [автоматическом применении безопасности на уровне строк для вновь созданных таблиц](https://techcommunity.microsoft.com/t5/SQL-Server/Apply-Row-Level-Security-automatically-to-newly-created-tables/ba-p/384393).

## <a name="summary"></a>Итоги

Средства эластичных баз данных и безопасность на уровне строк могут использоваться совместно для масштабирования уровня данных приложения благодаря поддержке мультитенантных и однотенантных сегментов. Мультитенантные сегменты повышают эффективность хранения данных. Вы заметите это, если у вас есть большое количество клиентов, у каждого из которых есть только несколько строк данных. Сегменты одного клиента пригодятся для особо важных клиентов с более строгими требованиями к производительности базы данных и изоляции. Дополнительные сведения см. в [руководстве по обеспечению безопасности на уровне строк][rls].

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Что такое пул эластичных БД Azure?](elastic-pool-overview.md)
- [Развертывание с помощью Базы данных SQL Azure](elastic-scale-introduction.md)
- [Шаблоны разработки для мультитенантных приложений SaaS с использованием Базы данных Azure SQL](./saas-tenancy-app-design-patterns.md)
- [Authentication in multitenant apps, using Azure AD and OpenID Connect](/azure/architecture/multitenant-identity/authenticate)
- [Сведения о приложении Tailspin Surveys](/azure/architecture/multitenant-identity/tailspin)

## <a name="questions-and-feature-requests"></a>Вопросы и запросы на функции

Для получения вопросов свяжитесь с нами на [странице Microsoft Q&A вопрос для базы данных SQL](/answers/topics/azure-sql-database.html). Запросы на добавление новых функций оставляйте на [форуме по Базы данных SQL для обратной связи](https://feedback.azure.com/forums/217321-sql-database/).

<!--Image references-->
[1]: ./media/saas-tenancy-elastic-tools-multi-tenant-row-level-security/blogging-app.png
<!--anchors-->
[rls]: /sql/relational-databases/security/row-level-security
[s-d-elastic-database-client-library]:elastic-database-client-library.md
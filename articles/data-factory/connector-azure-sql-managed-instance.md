---
title: Копирование и преобразование данных в Управляемый экземпляр SQL Azure
description: Узнайте, как копировать и преобразовывать данные в Управляемый экземпляр Azure SQL с помощью фабрики данных Azure.
ms.service: data-factory
ms.topic: conceptual
ms.author: jingwang
author: linda33wj
ms.custom: seo-lt-2019
ms.date: 03/17/2021
ms.openlocfilehash: eae085a73e8f43813aa3f02fa910c7931f10f36c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104597423"
---
# <a name="copy-and-transform-data-in-azure-sql-managed-instance-by-using-azure-data-factory"></a>Копирование и преобразование данных в Управляемый экземпляр Azure SQL с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные в Управляемый экземпляр Azure SQL и обратно и использовать поток данных для преобразования данных в Управляемый экземпляр SQL Azure. Дополнительные сведения о Фабрике данных Azure см. во [вводной статье](introduction.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель SQL Управляемый экземпляр поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием матрицы [поддерживаемых источников и приемников](copy-activity-overview.md).
- [Поток данных для сопоставления](concepts-data-flow-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Действие получения метаданных в Фабрике данных Azure](control-flow-get-metadata-activity.md)

Для действия копирования этот соединитель базы данных SQL Azure поддерживает следующие функции:

- Копирование данных с помощью проверки подлинности SQL и Azure Active Directory (Azure AD) маркера приложения с помощью субъекта-службы или управляемых удостоверений для ресурсов Azure.
- В качестве источника, получая данные с помощью SQL-запроса или хранимой процедуры. Можно также выбрать параллельное копирование из источника SQL MI. Дополнительные сведения см. в разделе [параллельная копия из SQL MI](#parallel-copy-from-sql-mi) .
- В качестве приемника автоматически создает целевую таблицу, если она не существует на основе исходной схемы. Добавление данных в таблицу или вызов хранимой процедуры с пользовательской логикой во время копирования.

>[!NOTE]
> Сейчас [Always ENCRYPTED](/sql/relational-databases/security/encryption/always-encrypted-database-engine) SQL управляемый экземпляр не поддерживается этим соединителем. Для решения этой проблемы можно использовать [универсальный соединитель ODBC](connector-odbc.md) и драйвер SQL Server ODBC через локальную среду выполнения интеграции. Дополнительные сведения см. в разделе [Always encrypted](#using-always-encrypted) . 

## <a name="prerequisites"></a>Предварительные требования

Чтобы получить доступ к [общедоступной конечной точке](../azure-sql/managed-instance/public-endpoint-overview.md)SQL управляемый экземпляр, вы можете использовать управляемую среду выполнения интеграции Azure в фабрике данных Azure. Убедитесь, что включена общедоступная конечная точка, а также разрешите трафик общедоступной конечной точки в группе безопасности сети, чтобы фабрика данных Azure могла подключаться к базе данных. Дополнительные сведения см. в [этом руководстве](../azure-sql/managed-instance/public-endpoint-configure.md).

Чтобы получить доступ к закрытой конечной точке SQL Управляемый экземпляр, настройте локальную [среду выполнения интеграции](create-self-hosted-integration-runtime.md) , которая может обращаться к базе данных. При подготовке локальной среды выполнения интеграции в той же виртуальной сети, что и управляемый экземпляр, убедитесь, что компьютер среды выполнения интеграции находится в другой подсети, чем управляемый экземпляр. При подготовке локальной среды выполнения интеграции в виртуальной сети, отличной от той, в которой находится управляемый экземпляр, можно использовать пиринг виртуальной сети или виртуальную сеть для подключения к виртуальной сети. Дополнительные сведения см. в статье [Подключение приложения к Управляемому экземпляру SQL](../azure-sql/managed-instance/connect-application-instance.md).

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных Azure, относящихся к соединителю SQL Управляемый экземпляр.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы SQL Управляемый экземпляр поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства Type необходимо задать значение **азуресклми**. | Да |
| connectionString |Это свойство указывает сведения **ConnectionString** , необходимые для подключения к SQL управляемый экземпляр с использованием проверки подлинности SQL. Дополнительные сведения см. в следующих примерах. <br/>По умолчанию используется порт 1433. Если вы используете SQL Управляемый экземпляр с общедоступной конечной точкой, явно укажите порт 3342.<br> Вы также можете добавить пароль в Azure Key Vault. Если это проверка подлинности SQL, вытяните `password` конфигурацию из строки подключения. Дополнительные сведения см. в примере JSON, который следует за таблицей, и [Храните учетные данные в Azure Key Vault](store-credentials-in-key-vault.md). |Да |
| servicePrincipalId | Укажите идентификатора клиента приложения. | Да, при использовании проверки подлинности Azure AD с субъектом-службой |
| servicePrincipalKey | Укажите ключ приложения. Пометьте это поле как **SecureString** , чтобы безопасно хранить его в фабрике данных Azure, или [сослаться на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да, при использовании проверки подлинности Azure AD с субъектом-службой |
| tenant | Укажите сведения о клиенте, например имя домена или идентификатор клиента, в котором находится приложение. Эти сведения можно получить, наведя указатель мыши на правый верхний угол страницы портала Azure. | Да, при использовании проверки подлинности Azure AD с субъектом-службой |
| азуреклаудтипе | Для проверки подлинности субъекта-службы укажите тип облачной среды Azure, в которой зарегистрировано приложение Azure AD. <br/> Допустимые значения: **азурепублик**, **AzureChina**, **AzureUsGovernment** и **азурежермани**. По умолчанию используется облачная среда фабрики данных. | Нет |
| connectVia | Это [среда выполнения интеграции](concepts-integration-runtime.md) для подключения к хранилищу данных. Вы можете использовать локальную среду выполнения интеграции или среду выполнения интеграции Azure, если управляемый экземпляр имеет общедоступную конечную точку и позволяет фабрике данных Azure получить к ней доступ. Если не указано другое, используется среда выполнения интеграции Azure по умолчанию. |Да |

В разделах ниже описываются требования и приводятся примеры JSON для разных типов проверки подлинности.

- [Проверка подлинности SQL](#sql-authentication).
- [Проверка подлинности маркера приложения Azure AD: субъект-служба](#service-principal-authentication)
- [Проверка подлинности маркера приложения Azure AD: управляемые удостоверения для ресурсов Azure](#managed-identity)

### <a name="sql-authentication"></a>Проверка подлинности SQL

**Пример 1. Использование проверки подлинности SQL**

```json
{
    "name": "AzureSqlMILinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример 2. Использование проверки подлинности SQL с паролем в Azure Key Vault**

```json
{
    "name": "AzureSqlMILinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;",
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="service-principal-authentication"></a>Проверка подлинности субъекта-службы

Чтобы использовать проверку подлинности по маркеру приложения Azure AD на основе субъекта-службы, выполните следующие действия:

1. Выполните действия по [подготовке администратора Azure Active Directory для управляемый экземпляр](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

2. [Создайте приложение Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) на портале Azure. Запишите имя приложения и следующие значения, которые используются для определения связанной службы:

    - Идентификатор приложения
    - Ключ приложения
    - Tenant ID

3. [Создайте имена входа](/sql/t-sql/statements/create-login-transact-sql) для управляемого удостоверения фабрики данных Azure. В SQL Server Management Studio (SSMS) подключитесь к управляемому экземпляру с помощью учетной записи SQL Server, которая является **sysadmin**. Запустите следующий код T-SQL для базы данных **master**:

    ```sql
    CREATE LOGIN [your application name] FROM EXTERNAL PROVIDER
    ```

4. [Создание пользователей автономной базы данных](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities) для управляемого удостоверения в фабрике данных Azure. Подключитесь к базе данных, откуда или куда вы хотите скопировать данные, запустите следующий код T-SQL: 
  
    ```sql
    CREATE USER [your application name] FROM EXTERNAL PROVIDER
    ```

5. Предоставьте управляемому удостоверению фабрики данных необходимые разрешения, как обычно для пользователей SQL и других. Выполните следующий код. Дополнительные параметры см. в [этом документе](/sql/t-sql/statements/alter-role-transact-sql).

    ```sql
    ALTER ROLE [role name e.g. db_owner] ADD MEMBER [your application name]
    ```

6. Настройка связанной службы SQL Управляемый экземпляр в фабрике данных Azure.

**Пример. Использование проверки подлинности субъекта-службы**

```json
{
    "name": "AzureSqlDbLinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Управляемые удостоверения для аутентификации ресурсов Azure

Фабрика данных может быть связана с [управляемым удостоверением ресурсов Azure](data-factory-service-identity.md), которое представляет отдельную фабрику данных. Это управляемое удостоверение можно использовать для проверки подлинности SQL Управляемый экземпляр. С помощью этого удостоверения назначенная фабрика может обращаться к данным и копировать их из вашей базы данных или в нее.

Чтобы использовать аутентификацию управляемого удостоверения, выполните следующие действия.

1. Выполните действия по [подготовке администратора Azure Active Directory для управляемый экземпляр](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

2. [Создайте имена входа](/sql/t-sql/statements/create-login-transact-sql) для управляемого удостоверения фабрики данных Azure. В SQL Server Management Studio (SSMS) подключитесь к управляемому экземпляру с помощью учетной записи SQL Server, которая является **sysadmin**. Запустите следующий код T-SQL для базы данных **master**:

    ```sql
    CREATE LOGIN [your Data Factory name] FROM EXTERNAL PROVIDER
    ```

3. [Создание пользователей автономной базы данных](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities) для управляемого удостоверения в фабрике данных Azure. Подключитесь к базе данных, откуда или куда вы хотите скопировать данные, запустите следующий код T-SQL: 
  
    ```sql
    CREATE USER [your Data Factory name] FROM EXTERNAL PROVIDER
    ```

4. Предоставьте управляемому удостоверению фабрики данных необходимые разрешения, как обычно для пользователей SQL и других. Выполните следующий код. Дополнительные параметры см. в [этом документе](/sql/t-sql/statements/alter-role-transact-sql).

    ```sql
    ALTER ROLE [role name e.g. db_owner] ADD MEMBER [your Data Factory name]
    ```

5. Настройка связанной службы SQL Управляемый экземпляр в фабрике данных Azure.

**Пример. Использование проверки подлинности управляемого удостоверения**

```json
{
    "name": "AzureSqlDbLinkedService",
    "properties": {
        "type": "AzureSqlMI",
        "typeProperties": {
            "connectionString": "Data Source=<hostname,port>;Initial Catalog=<databasename>;"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о наборах данных. В этом разделе содержится список свойств, поддерживаемых набором данных SQL Управляемый экземпляр.

Чтобы скопировать данные в SQL Управляемый экземпляр и обратно, поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **азуресклмитабле**. | Да |
| схема | Имя схемы. |"Нет" для источника, "Да" для приемника  |
| table | Имя таблицы или представления. |"Нет" для источника, "Да" для приемника  |
| tableName | Имя таблицы или представления со схемой. Это свойство поддерживается только для обеспечения обратной совместимости. Для новой рабочей нагрузки используйте `schema` и `table`. | "Нет" для источника, "Да" для приемника |

**Пример**

```json
{
    "name": "AzureSqlMIDataset",
    "properties":
    {
        "type": "AzureSqlMITable",
        "linkedServiceName": {
            "referenceName": "<SQL Managed Instance linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "schema": "<schema_name>",
            "table": "<table_name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, доступных для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником и приемником SQL Управляемый экземпляр.

### <a name="sql-managed-instance-as-a-source"></a>Управляемый экземпляр SQL в качестве источника

>[!TIP]
>Чтобы эффективно загружать данные из SQL MI с помощью секционирования данных, Узнайте больше от [параллельного копирования из SQL MI](#parallel-copy-from-sql-mi).

Чтобы скопировать данные из Управляемый экземпляр SQL, в разделе Источник действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **склмисаурце**. | Да |
| sqlReaderQuery |Это свойство применяет пользовательский SQL-запрос для чтения данных. Например, `select * from MyTable`. |нет |
| sqlReaderStoredProcedureName |Это свойство содержит имя хранимой процедуры, которая считывает данные из исходной таблицы. Последней инструкцией SQL должна быть инструкция SELECT в хранимой процедуре. |нет |
| storedProcedureParameters |Это параметры для хранимой процедуры.<br/>Допустимые значения: пары имен или значений. Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. |нет |
| isolationLevel | Задает режим блокировки транзакций для источника данных SQL. Допустимые значения: **ReadCommitted**, **READUNCOMMITTED**, **RepeatableRead**, **Serializable**, **snapshot**. Если значение не указано, используется уровень изоляции базы данных по умолчанию. Дополнительные сведения см. в [этом документе](/dotnet/api/system.data.isolationlevel). | нет |
| partitionOptions | Задает параметры секционирования данных, используемые для загрузки данных из SQL MI. <br>Допустимые значения: **None** (по умолчанию), **фисикалпартитионсофтабле** и **динамикранже**.<br>Если параметр секции включен (то есть не `None` ), степень параллелизма для параллельной загрузки данных из SQL MI управляется [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) параметром действия копирования. | нет |
| partitionSettings | Позволяет указать группу параметров для секционирования данных. <br>Применяется, если параметр Partition не имеет значение `None` . | Нет |
| ***В разделе `partitionSettings` :*** | | |
| partitionColumnName | Укажите имя исходного столбца **в целочисленном или в типе даты/времени** ( `int` , `smallint` , `bigint` , `date` , `smalldatetime` , `datetime` , `datetime2` или `datetimeoffset` ), которое будет использоваться секционированием по диапазонам для параллельного копирования. Если значение не указано, то индекс или первичный ключ таблицы будет обнаружен и использован в качестве столбца секционирования.<br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, присоединитесь к  `?AdfDynamicRangePartitionCondition ` предложению WHERE. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-sql-mi) . | нет |
| partitionUpperBound | Максимальное значение столбца секционирования для разделения диапазона секций. Это значение используется для выбора шага секционирования, а не для фильтрации строк в таблице. Все строки в таблице или результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.  <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-sql-mi) . | нет |
| partitionLowerBound | Минимальное значение столбца секционирования для разделения диапазона секций. Это значение используется для выбора шага секционирования, а не для фильтрации строк в таблице. Все строки в таблице или результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.<br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-sql-mi) . | Нет |

**Обратите внимание на следующие моменты.**

- Если для **склмисаурце** указан **sqlReaderQuery** , действие копирования выполняет этот запрос к источнику управляемый экземпляр SQL для получения данных. Есть и другой вариант: создать хранимую процедуру, указав ее имя в **sqlReaderStoredProcedureName** и параметры в **storedProcedureParameters**, если она принимает параметры.
- При использовании хранимой процедуры в источнике для получения данных обратите внимание на то, что хранимая процедура разработана так, чтобы она возвращала другую схему при передаче другого значения параметра, может возникнуть сбой или можно увидеть непредвиденный результат при импорте схемы из пользовательского интерфейса или при копировании данных в базу данных SQL с помощью автоматического создания таблиц.

**Пример. Использование SQL-запроса**

```json
"activities":[
    {
        "name": "CopyFromAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Managed Instance input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlMISource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Пример. использование хранимой процедуры**

```json
"activities":[
    {
        "name": "CopyFromAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Managed Instance input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlMISource",
                "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
                "storedProcedureParameters": {
                    "stringData": { "value": "str3" },
                    "identifier": { "value": "$$Text.Format('{0:yyyy}', <datetime parameter>)", "type": "Int"}
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Определение хранимой процедуры**

```sql
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
    select *
    from dbo.UnitTestSrcTable
    where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sql-managed-instance-as-a-sink"></a>SQL Управляемый экземпляр в качестве приемника

> [!TIP]
> Дополнительные сведения о поддерживаемых поведениях записи, конфигурациях и рекомендациях от [рекомендаций по загрузке данных в SQL управляемый экземпляр](#best-practice-for-loading-data-into-sql-managed-instance).

Чтобы скопировать данные в SQL Управляемый экземпляр, в разделе приемника действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type приемника действия копирования должно иметь значение **склмисинк**. | Да |
| preCopyScript |Это свойство задает SQL-запрос для выполнения действия копирования перед записью данных в SQL Управляемый экземпляр. Он вызывается однократно при каждом запуске копирования. Это свойство можно использовать для очистки предварительно загруженных данных. |нет |
| tableOption | Указывает, следует ли [автоматически создавать таблицу приемника, если она](copy-activity-overview.md#auto-create-sink-tables) не существует на основе исходной схемы. Автоматическое создание таблицы не поддерживается, если приемник указывает хранимую процедуру. Допустимые значения: `none` (по умолчанию), `autoCreate`. |нет |
| sqlWriterStoredProcedureName | Имя хранимой процедуры, в которой определяется, как применить исходные данные в целевой таблице. <br/>Эта хранимая процедура будет *вызываться для каждого пакета*. Для операций, выполняемых только один раз и не имеющих никаких действий с исходными данными, например Delete или TRUNCATE, используйте `preCopyScript` свойство.<br>См. пример из [вызова хранимой процедуры из приемника SQL](#invoke-a-stored-procedure-from-a-sql-sink). | Нет |
| сторедпроцедуретаблетипепараметернаме |Имя параметра табличного типа, указанного в хранимой процедуре.  |Нет |
| sqlWriterTableType |Имя типа таблицы, используемое в хранимой процедуре. Действие копирования предоставляет доступ к перемещаемым данным во временной таблице с указанным здесь типом. Это позволяет при выполнении хранимой процедуры объединить копируемые и существующие данные. |нет |
| storedProcedureParameters |Параметры для хранимой процедуры.<br/>Допустимые значения: пары "имя — значение". Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. | Нет |
| writeBatchSize |Число строк, вставляемых в таблицу SQL для *каждого пакета*.<br/>Допустимые значения: целое число (количество строк). По умолчанию фабрика данных Azure динамически определяет соответствующий размер пакета в зависимости от размера строки.  |Нет |
| writeBatchTimeout |Это свойство определяет время ожидания при выполнении операции пакетной вставки, по истечении которого она считается не выполненной.<br/>Допустимые значения — для интервала времени. Например, "00:30:00", то есть 30 минут. |Нет |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

**Пример 1. Добавление данных**

```json
"activities":[
    {
        "name": "CopyToAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Managed Instance output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlMISink",
                "tableOption": "autoCreate",
                "writeBatchSize": 100000
            }
        }
    }
]
```

**Пример 2. вызов хранимой процедуры во время копирования**

Дополнительные сведения см. в статье [вызов хранимой процедуры из приемника SQL MI](#invoke-a-stored-procedure-from-a-sql-sink).

```json
"activities":[
    {
        "name": "CopyToAzureSqlMI",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Managed Instance output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlMISink",
                "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
                "storedProcedureTableTypeParameterName": "MyTable",
                "sqlWriterTableType": "MyTableType",
                "storedProcedureParameters": {
                    "identifier": { "value": "1", "type": "Int" },
                    "stringData": { "value": "str1" }
                }
            }
        }
    }
]
```

## <a name="parallel-copy-from-sql-mi"></a>Параллельная копия из SQL MI

Соединитель Azure SQL Управляемый экземпляр в действии копирования предоставляет встроенное секционирование данных для параллельного копирования данных. Параметры секционирования данных можно найти на вкладке **источник** действия копирования.

![Снимок экрана с параметрами секционирования](./media/connector-sql-server/connector-sql-partition-options.png)

При включении секционированной копии действие копирования выполняет параллельные запросы к источнику SQL MI для загрузки данных по секциям. Степень параллелизма определяется с помощью параметра [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) для действия копирования. Например, если установлено значение `parallelCopies` 4, фабрика данных одновременно создает и выполняет четыре запроса на основе указанного параметра секции и параметров, и каждый запрос извлекает часть данных из SQL MI.

Рекомендуется включить параллельное копирование с секционированием данных, особенно при загрузке большого объема данных из SQL MI. Ниже приведены рекомендуемые конфигурации для разных сценариев. При копировании данных в хранилище данных, основанное на файлах, рекомендуется записывать в папку несколько файлов (указывать только имя папки). в этом случае производительность лучше, чем при записи в один файл.

| Сценарий                                                     | Предлагаемые параметры                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Полная загрузка из большой таблицы с физическими секциями.        | **Параметр секции**: физические разделы таблицы. <br><br/>Во время выполнения фабрика данных автоматически обнаруживает физические секции и копирует данные по секциям. <br><br/>Чтобы проверить, имеет ли таблица физическую секцию, можно обратиться к [этому запросу](#sample-query-to-check-physical-partition). |
| Полная загрузка из большой таблицы без физических секций, в то время как со столбцом типа Integer или DateTime для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Столбец секционирования** (необязательно): укажите столбец, используемый для секционирования данных. Если не указано, используется индекс или столбец первичного ключа.<br/>**Верхняя граница секции** и **Нижняя граница секции** (необязательно): укажите, нужно ли определить шаг секционирования. Это не предназначено для фильтрации строк в таблице, все строки в таблице будут секционированы и скопированы. Если значение не указано, действие копирования автоматически определяет значения.<br><br>Например, если столбец секции "ID" имеет диапазон значений от 1 до 100, а нижняя граница равна 20, а верхняя граница — 80, то при параллельном копировании как 4 фабрика данных извлекает данные по 4 секциям в диапазоне <= 20, [21, 50], [51, 80] и >= 81 соответственно. |
| Загрузка большого объема данных с помощью пользовательского запроса без физических секций, в то время как со столбцом типа Integer или Date/DateTime для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Запрос**: `SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец для секционирования данных.<br>**Верхняя граница секции** и **Нижняя граница секции** (необязательно): укажите, нужно ли определить шаг секционирования. Это не предназначено для фильтрации строк в таблице, все строки в результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.<br><br>Во время выполнения фабрика данных заменяет `?AdfRangePartitionColumnName` фактическое имя столбца и диапазоны значений для каждой секции и отправляет в SQL MI. <br>Например, если столбец секции "ID" имеет диапазон значений от 1 до 100, а нижняя граница равна 20, а верхняя граница — 80, то при параллельном копировании как 4 фабрика данных извлекает данные по 4 секциям в диапазоне <= 20, [21, 50], [51, 80] и >= 81 соответственно. <br><br>Ниже приведены дополнительные примеры запросов для различных сценариев.<br> 1. запросите всю таблицу: <br>`SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition`<br> 2. запрос из таблицы с выделением столбцов и дополнительными фильтрами WHERE-Order: <br>`SELECT <column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 3. запрос с вложенными запросами: <br>`SELECT <column_list> FROM (<your_sub_query>) AS T WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 4. запрос с Секцией во вложенном запросе: <br>`SELECT <column_list> FROM (SELECT <your_sub_query_column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition) AS T`
|

Рекомендации по загрузке данных с параметром секции:

1. Чтобы избежать неравномерного распределения данных, выберите отличительный столбец в качестве столбца секционирования (например, первичный ключ или уникальный ключ). 
2. Если таблица имеет встроенную секцию, используйте параметр секционирования "физические разделы таблицы", чтобы повысить производительность.    
3. Если для копирования данных используется Azure Integration Runtime, можно задать больший размер "[единицы интеграции данных (Диу)](copy-activity-performance-features.md#data-integration-units)" (>4), чтобы использовать больше вычислительных ресурсов. Проверьте применимые сценарии.
4. "[Степень параллелизма копирования](copy-activity-performance-features.md#parallel-copy)" — Управление номерами секций, установка этого числа слишком большого размера, что значительно вредит производительности, рекомендуется задать это число как (Диу или число размещенных в нем IR-узлов) * (от 2 до 4).

**Пример. полная загрузка из большой таблицы с физическими секциями**

```json
"source": {
    "type": "SqlMISource",
    "partitionOption": "PhysicalPartitionsOfTable"
}
```

**Пример: запрос с секционированием по динамическому диапазону**

```json
"source": {
    "type": "SqlMISource",
    "query": "SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column (optional) to decide the partition stride, not as data filter>",
        "partitionLowerBound": "<lower_value_of_partition_column (optional) to decide the partition stride, not as data filter>"
    }
}
```

### <a name="sample-query-to-check-physical-partition"></a>Пример запроса для проверки физической секции

```sql
SELECT DISTINCT s.name AS SchemaName, t.name AS TableName, pf.name AS PartitionFunctionName, c.name AS ColumnName, iif(pf.name is null, 'no', 'yes') AS HasPartition
FROM sys.tables AS t
LEFT JOIN sys.objects AS o ON t.object_id = o.object_id
LEFT JOIN sys.schemas AS s ON o.schema_id = s.schema_id
LEFT JOIN sys.indexes AS i ON t.object_id = i.object_id 
LEFT JOIN sys.index_columns AS ic ON ic.partition_ordinal > 0 AND ic.index_id = i.index_id AND ic.object_id = t.object_id 
LEFT JOIN sys.columns AS c ON c.object_id = ic.object_id AND c.column_id = ic.column_id 
LEFT JOIN sys.partition_schemes ps ON i.data_space_id = ps.data_space_id 
LEFT JOIN sys.partition_functions pf ON pf.function_id = ps.function_id 
WHERE s.name='[your schema]' AND t.name = '[your table name]'
```

Если таблица имеет физическую секцию, вы увидите «Хаспартитион» как «Да», как показано ниже.

![Результат SQL запроса](./media/connector-azure-sql-database/sql-query-result.png)

## <a name="best-practice-for-loading-data-into-sql-managed-instance"></a>Рекомендации по загрузке данных в SQL Управляемый экземпляр

При копировании данных в SQL Управляемый экземпляр может потребоваться другое поведение при записи.

- [Append](#append-data): мои исходные данные имеют только новые записи.
- [Upsert](#upsert-data): исходные данные имеют как вставки, так и обновления.
- [Перезаписать](#overwrite-the-entire-table): я хочу каждый раз загружать всю таблицу измерения.
- [Запись с помощью пользовательской логики](#write-data-with-custom-logic): требуется дополнительные операции перед окончательным вставкой в целевую таблицу. 

Сведения о настройке в фабрике данных Azure и рекомендациях см. в соответствующих разделах.

### <a name="append-data"></a>Добавление данных.

Добавление данных — это поведение по умолчанию для соединителя приемника SQL Управляемый экземпляр. Фабрика данных Azure обеспечивает эффективную запись в таблицу с помощью операций групповой вставки. Источник и приемник можно настроить соответствующим образом в действии копирования.

### <a name="upsert-data"></a>Операция upsert с данными

**Вариант 1.** При наличии большого объема данных для копирования можно выполнить загрузку всех записей в промежуточную таблицу с помощью действия копирования, а затем запустить действие хранимой процедуры, чтобы применить инструкцию [Merge](/sql/t-sql/statements/merge-transact-sql) или INSERT или Update в одном снимке. 

В настоящее время действие копирования не поддерживает загрузку данных во временную таблицу базы данных. Существует более сложный способ настройки с помощью сочетания нескольких действий. см. статью [Оптимизация Upsert для базы данных SQL](https://github.com/scoriani/azuresqlbulkupsert). Ниже показан пример использования постоянной таблицы в качестве промежуточного.

Например, в фабрике данных Azure можно создать конвейер с **действием копирования** , связанным с **действием хранимой процедуры**. Первый копирует данные из исходного хранилища в промежуточную таблицу Azure SQL Управляемый экземпляр, например **упсертстагингтабле**, как имя таблицы в наборе данных. Затем второй вызовет хранимую процедуру для слияния исходных данных из промежуточной таблицы в целевую таблицу и очистки промежуточной таблицы.

![Upsert](./media/connector-azure-sql-database/azure-sql-database-upsert.png)

В базе данных Определите хранимую процедуру с помощью логики слияния, как показано в следующем примере, на который указывает действие из предыдущего действия хранимой процедуры. Предположим, что целевым объектом является **маркетинговая** таблица с тремя столбцами: **ProfileID**, **State** и **Category**. Выполните Upsert на основе столбца **ProfileID** .

```sql
CREATE PROCEDURE [dbo].[spMergeData]
AS
BEGIN
    MERGE TargetTable AS target
    USING UpsertStagingTable AS source
    ON (target.[ProfileID] = source.[ProfileID])
    WHEN MATCHED THEN
        UPDATE SET State = source.State
    WHEN NOT matched THEN
        INSERT ([ProfileID], [State], [Category])
      VALUES (source.ProfileID, source.State, source.Category);
    
    TRUNCATE TABLE UpsertStagingTable
END
```

**Вариант 2.** Можно выбрать [вызов хранимой процедуры в рамках действия копирования](#invoke-a-stored-procedure-from-a-sql-sink). Этот подход запускает каждый пакет (в соответствии со `writeBatchSize` свойством) в исходной таблице, а не использует инструкцию INSERT в качестве метода по умолчанию в действии копирования.

### <a name="overwrite-the-entire-table"></a>Перезаписать всю таблицу

Свойство **preCopyScript** можно настроить в приемнике действия копирования. В этом случае для каждого выполняемого действия копирования фабрика данных Azure сначала запускает сценарий. Затем выполняется копирование для вставки данных. Например, чтобы перезаписать всю таблицу последними данными, укажите скрипт, чтобы сначала удалить все записи, прежде чем выполнять пакетную загрузку новых данных из источника.

### <a name="write-data-with-custom-logic"></a>Запись данных с помощью пользовательской логики

Действия по записи данных с помощью пользовательской логики похожи на процедуры, описанные в разделе [данных Upsert](#upsert-data) . Если необходимо применить дополнительные операции перед окончательной вставкой исходных данных в целевую таблицу, можно загрузить в промежуточную таблицу, затем вызвать действие хранимой процедуры или вызвать хранимую процедуру в приемнике действия копирования, чтобы применить данные.

## <a name="invoke-a-stored-procedure-from-a-sql-sink"></a><a name="invoke-a-stored-procedure-from-a-sql-sink"></a> Вызов хранимой процедуры из приемника SQL

При копировании данных в Управляемый экземпляр SQL можно также настроить и вызвать пользовательскую хранимую процедуру с дополнительными параметрами для каждого пакета исходной таблицы. Функция хранимой процедуры использует возможности [возвращающих табличное значение параметров](/dotnet/framework/data/adonet/sql/table-valued-parameters).

Вы можете использовать хранимую процедуру, когда встроенные механизмы копирования не подходят. Примером может быть применение дополнительной обработки перед окончательной вставкой исходных данных в целевую таблицу. Некоторые дополнительные примеры обработки позволяют объединить столбцы, найти дополнительные значения и вставить их в несколько таблиц.

В следующем примере показано, как использовать хранимую процедуру для выполнения операции Upsert в таблице базы данных SQL Server. Предположим, что у входных данных и **маркетинговой** таблицы приемника есть три столбца: **ProfileID**, **State** и **Category**. Выполните Upsert на основе столбца **ProfileID** и примените его только к определенной категории с названием «Product а».

1. В базе данных определите тип таблицы с тем же именем, что и **и sqlwritertabletype**. Для типа таблицы укажите ту же схему, которая возвращается для входных данных.

    ```sql
    CREATE TYPE [dbo].[MarketingType] AS TABLE(
        [ProfileID] [varchar](256) NOT NULL,
        [State] [varchar](256) NOT NULL,
        [Category] [varchar](256) NOT NULL
    )
    ```

2. В базе данных Определите хранимую процедуру с тем же именем, что и **sqlWriterStoredProcedureName**. Она обрабатывает входные данные из указанного источника и выполняет их слияние в выходную таблицу. Имя параметра типа таблицы в хранимой процедуре совпадает с именем **TableName** , определенным в наборе данных.

    ```sql
    CREATE PROCEDURE spOverwriteMarketing @Marketing [dbo].[MarketingType] READONLY, @category varchar(256)
    AS
    BEGIN
    MERGE [dbo].[Marketing] AS target
    USING @Marketing AS source
    ON (target.ProfileID = source.ProfileID and target.Category = @category)
    WHEN MATCHED THEN
        UPDATE SET State = source.State
    WHEN NOT MATCHED THEN
        INSERT (ProfileID, State, Category)
        VALUES (source.ProfileID, source.State, source.Category);
    END
    ```

3. В фабрике данных Azure Определите раздел **приемника SQL MI** в действии копирования следующим образом:

    ```json
    "sink": {
        "type": "SqlMISink",
        "sqlWriterStoredProcedureName": "spOverwriteMarketing",
        "storedProcedureTableTypeParameterName": "Marketing",
        "sqlWriterTableType": "MarketingType",
        "storedProcedureParameters": {
            "category": {
                "value": "ProductA"
            }
        }
    }
    ```

## <a name="mapping-data-flow-properties"></a>Свойства потока данных для сопоставления

При преобразовании данных в потоке сопоставления данных можно выполнять чтение и запись в таблицы из Управляемый экземпляр Azure SQL. Дополнительные сведения см. в описаниях [преобразования источника](data-flow-source.md) и [преобразования приемника](data-flow-sink.md) в разделе, посвященном потокам данных для сопоставления.

> [!NOTE]
> Соединитель Azure SQL Управляемый экземпляр в настоящее время используется в качестве общедоступной предварительной версии. Вы можете подключиться к общедоступной конечной точке SQL Управляемый экземпляр, но еще не к частной конечной точке.

### <a name="source-transformation"></a>Преобразование источника

В таблице ниже перечислены свойства, поддерживаемые источником Управляемый экземпляр Azure SQL. Эти свойства можно изменить на вкладке **Параметры источника** .

| Имя | Описание | Обязательно | Допустимые значения | Свойство сценария потока данных |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Таблица | Если выбрать вариант Таблица как входные данные, поток данных извлекает все данные из таблицы, указанной в наборе данных. | Нет | - |- |
| Запрос | При выборе запроса в качестве входных данных укажите SQL-запрос для выборки данных из источника, переопределяющий любую таблицу, указанную в наборе данных. Использование запросов — отличный способ сократить количество строк для тестирования или поиска.<br><br>Предложение **ORDER BY** не поддерживается, но можно задать полную инструкцию SELECT FROM. Кроме того, можно использовать табличные функции, определяемые пользователем. **SELECT * FROM удфжетдата ()** — это определяемая пользователем функция в SQL, которая возвращает таблицу, которую можно использовать в потоке данных.<br>Пример запроса: `Select * from MyTable where customerId > 1000 and customerId < 2000`| нет | Строка | query |
| Размер пакета | Укажите размер пакета для фрагментирования больших данных в операции чтения. | Нет | Целое число | batchSize |
| Уровень изоляции | Выберите один из следующих уровней изоляции:<br>— Зафиксировано чтение<br>-READ UNCOMMITTED (по умолчанию)<br>— Повторяемое чтение<br>— Сериализуемый<br>-None (пропускать уровень изоляции) | Нет | <small>READ_COMMITTED<br/>READ_UNCOMMITTED<br/>REPEATABLE_READ<br/>SERIALIZABLE<br/>NONE</small> |isolationLevel |

#### <a name="azure-sql-managed-instance-source-script-example"></a>Пример скрипта источника Управляемый экземпляр Azure SQL

При использовании Управляемый экземпляр Azure SQL в качестве типа источника связанный сценарий потока данных будет следующим:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    isolationLevel: 'READ_UNCOMMITTED',
    query: 'select * from MYTABLE',
    format: 'query') ~> SQLMISource
```

### <a name="sink-transformation"></a>Преобразование приемника

В таблице ниже перечислены свойства, поддерживаемые приемником Управляемый экземпляр Azure SQL. Эти свойства можно изменить на вкладке **параметры приемника** .

| Имя | Описание | Обязательно | Допустимые значения | Свойство сценария потока данных |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Update - метод | Укажите, какие операции разрешены в назначении базы данных. По умолчанию разрешены только операции вставки.<br>Для обновления, Upsert или удаления строк необходимо [Преобразование «изменение строки](data-flow-alter-row.md) » для добавления тегов в строки для этих действий. | Да | `true` или `false` | удаляемые <br/>Insertable <br/>обновляемые <br/>упсертабле |
| Ключевые столбцы | Для обновлений, операции Upsert и DELETE необходимо задать ключевые столбцы, чтобы определить, какая строка будет изменена.<br>Имя столбца, которое вы выбираете в качестве ключа, будет использоваться как часть последующего обновления, Upsert, DELETE. Поэтому необходимо выбрать столбец, существующий в сопоставлении приемника. | Нет | Array | ключи |
| Пропустить запись ключевых столбцов | Если вы не хотите записывать значение в ключевой столбец, выберите "пропустить запись ключевых столбцов". | Нет | `true` или `false` | скипкэйвритес |
| Действие таблицы |Определяет, следует ли повторно создавать или удалять все строки целевой таблицы перед записью.<br>- **Нет**: никаких действий над таблицей не выполняется.<br>- **Повторное создание**: таблица будет удалена и создана повторно. Это действие необходимо, если новая таблица создается динамически.<br>- **Усечение**: все строки из целевой таблицы будут удалены. | Нет | `true` или `false` | повторно создать<br/>truncate |
| Размер пакета | Укажите, сколько строк записывается в каждом пакете. Более крупные размеры пакетов улучшают сжатие и оптимизацию памяти, но при кэшировании данных возникает риск нехватки памяти. | Нет | Целое число | batchSize |
| Сценарии SQL pre и POST | Укажите многострочные скрипты SQL, которые будут выполняться до (Предварительная обработка) и после чего (после обработки) записываются в базу данных приемника. | нет | Строка | пресклс<br>постсклс |

#### <a name="azure-sql-managed-instance-sink-script-example"></a>Пример скрипта приемника Управляемый экземпляр Azure SQL

При использовании Управляемый экземпляр Azure SQL в качестве типа приемника связанный сценарий потока данных:

```
IncomingStream sink(allowSchemaDrift: true,
    validateSchema: false,
    deletable:false,
    insertable:true,
    updateable:true,
    upsertable:true,
    keys:['keyColumn'],
    format: 'table',
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> SQLMISink
```

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Свойства действия GetMetadata

Подробные сведения об этих свойствах см. в разделе [Действие GetMetadata](control-flow-get-metadata-activity.md) 

## <a name="data-type-mapping-for-sql-managed-instance"></a>Сопоставление типов данных для SQL Управляемый экземпляр

При копировании данных в SQL Управляемый экземпляр и из него с помощью действия копирования из типов данных SQL Управляемый экземпляр в промежуточные типы данных фабрики данных Azure используются следующие сопоставления. Дополнительные сведения о том, как действие копирования сопоставляет схему и типы данных источника и приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип данных SQL Управляемый экземпляр | Промежуточный тип данных Фабрики данных Azure |
|:--- |:--- |
| BIGINT |Int64 |
| binary |Byte[] |
| bit |Логическое |
| char |String, Char[] |
| Дата |Дата и время |
| Datetime |Дата и время |
| datetime2 |Дата и время |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| Изображение |Byte[] |
| INT |Int32 |
| money |Decimal |
| nchar |String, Char[] |
| ntext |String, Char[] |
| NUMERIC |Decimal |
| nvarchar |String, Char[] |
| real |Один |
| rowversion |Byte[] |
| smalldatetime |Дата и время |
| smallint |Int16 |
| smallmoney |Decimal |
| sql_variant |Объект |
| текст |String, Char[] |
| time |TimeSpan |
| TIMESTAMP |Byte[] |
| tinyint |Int16 |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| Xml |Строка |

>[!NOTE]
> Для типов данных, которые сопоставляются с промежуточным типом Decimal, в настоящее время действие копирования поддерживает точность до 28. Если для ваших данных требуется точность больше 28, попробуйте преобразовать их в строковые данные в SQL-запросе.

## <a name="using-always-encrypted"></a>Использование Always Encrypted

При копировании данных в Azure SQL Управляемый экземпляр и обратно с [Always encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)используйте [универсальный соединитель ODBC](connector-odbc.md) и SQL Server драйвер ODBC через локально размещенную Integration Runtime. Соединитель Azure SQL Управляемый экземпляр не поддерживает Always Encrypted сейчас. 

В частности:

1. Настройте автономную Integration Runtime, если у вас ее нет. Дополнительные сведения см. в статье [Создание и настройка локальной среды выполнения интеграции](create-self-hosted-integration-runtime.md).

2. Скачайте 64-разрядный драйвер ODBC для SQL Server [отсюда](/sql/connect/odbc/download-odbc-driver-for-sql-server)и установите на компьютер Integration Runtime. Дополнительные сведения о том, как работает этот драйвер, см. в статье [использование Always encrypted с драйвером ODBC для SQL Server](/sql/connect/odbc/using-always-encrypted-with-the-odbc-driver#using-the-azure-key-vault-provider).

3. Создание связанной службы с типом ODBC для подключения к базе данных SQL см. следующие примеры:

    - Чтобы использовать **проверку подлинности SQL**: Укажите строку подключения ODBC, как показано ниже, и выберите **Обычная** проверка подлинности, чтобы задать имя пользователя и пароль.

        ```
        Driver={ODBC Driver 17 for SQL Server};Server=<serverName>;Database=<databaseName>;ColumnEncryption=Enabled;KeyStoreAuthentication=KeyVaultClientSecret;KeyStorePrincipalId=<servicePrincipalKey>;KeyStoreSecret=<servicePrincipalKey>
        ```

    - При запуске автономного Integration Runtime на виртуальной машине Azure вы можете использовать **проверку подлинности управляемого удостоверения** с помощью удостоверения виртуальной машины Azure: 

        1. Выполните те же [Предварительные требования](#managed-identity) , чтобы создать пользователя базы данных для управляемого удостоверения и предоставить соответствующую роль в базе данных.
        2. В поле связанная служба укажите строку подключения ODBC, как показано ниже, и выберите **Анонимная** проверка подлинности, так как сама строка подключения указывает `Authentication=ActiveDirectoryMsi` .

        ```
        Driver={ODBC Driver 17 for SQL Server};Server=<serverName>;Database=<databaseName>;ColumnEncryption=Enabled;KeyStoreAuthentication=KeyVaultClientSecret;KeyStorePrincipalId=<servicePrincipalKey>;KeyStoreSecret=<servicePrincipalKey>; Authentication=ActiveDirectoryMsi;
        ```

4. Создайте набор данных и действие копирования с соответствующим типом ODBC. Дополнительные сведения см. в статье о [соединителе ODBC](connector-odbc.md) .

## <a name="next-steps"></a>Дальнейшие действия
Список хранилищ данных, поддерживаемых действием копирования в фабрике данных Azure в качестве источников и приемников, см. в разделе [Supported Data Stores (поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats)).
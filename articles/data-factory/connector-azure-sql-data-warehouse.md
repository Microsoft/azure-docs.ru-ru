---
title: Копирование и преобразование данных в Azure Synapse Analytics
description: Узнайте, как копировать данные в Azure Synapse Analytics и обратно, а также преобразовывать данные в Azure Synapse Analytics с помощью Фабрики данных.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/17/2021
ms.openlocfilehash: 9c843ededd1fa863cc5eb4dc0db3a6da3478466d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104597527"
---
# <a name="copy-and-transform-data-in-azure-synapse-analytics-by-using-azure-data-factory"></a>Копирование и преобразование данных в Azure синапсе Analytics с помощью фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
>
> - [Версия 1](v1/data-factory-azure-sql-data-warehouse-connector.md)
> - [Текущая версия](connector-azure-sql-data-warehouse.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как копировать данные в Azure Synapse Analytics и обратно с помощью действия копирования в Фабрике данных Azure, а также как преобразовывать данные в Azure Data Lake Storage 2-го поколения с помощью Потока данных. Дополнительные сведения о Фабрике данных Azure см. во [вводной статье](introduction.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель Azure Synapse Analytics предназначен для поддержки следующих действий:

- [действия копирования](copy-activity-overview.md) с таблицей [поддерживаемой матрицы источника/приемника](copy-activity-overview.md);
- [потока данных для сопоставления](concepts-data-flow-overview.md);
- [Действие поиска](control-flow-lookup-activity.md)
- [Действие получения метаданных в Фабрике данных Azure](control-flow-get-metadata-activity.md)

Для действия копирования этот соединитель Azure Synapse Analytics поддерживает следующие функции:

- копирование данных с использованием аутентификации SQL и аутентификации на основе маркеров приложения Azure Active Directory (Azure AD) с субъектом-службой или управляемыми удостоверениями ресурсов Azure;
- извлечение данных с использованием SQL-запроса или хранимой процедуры (в качестве источника); Вы также можете выбрать параллельное копирование из источника Azure синапсе Analytics. Дополнительные сведения см. в разделе [Параллельное копирование из Azure синапсе Analytics](#parallel-copy-from-azure-synapse-analytics) .
- В качестве приемника Загрузите данные с помощью [polybase](#use-polybase-to-load-data-into-azure-synapse-analytics) или [инструкций Copy или инструкции](#use-copy-statement) INSERT. Для повышения производительности копирования рекомендуется использовать Polybase или инструкцию COPY. Соединитель также поддерживает автоматическое создание целевой таблицы, если она не существует на основе исходной схемы.

> [!IMPORTANT]
> При копировании данных с помощью Integration Runtime фабрики данных Azure настройте [правило брандмауэра на уровне сервера](../azure-sql/database/firewall-configure.md) , чтобы службы Azure могли получить доступ к [ЛОГИЧЕСКОМу серверу SQL](../azure-sql/database/logical-servers.md)Server.
> При копировании данных с помощью локальной среды выполнения интеграции настройте брандмауэр таким образом, чтобы он разрешал соответствующий диапазон IP-адресов. В этот диапазон входит IP-адрес компьютера, который используется для подключения к Azure Synapse Analytics.

## <a name="get-started"></a>Начало работы

> [!TIP]
> Для достижения максимальной производительности используйте инструкцию Polybase или COPY для загрузки данных в Azure синапсе Analytics. [Использование polybase для загрузки данных в Azure синапсе Analytics](#use-polybase-to-load-data-into-azure-synapse-analytics) и [Использование инструкции Copy для загрузки данных в разделы Azure синапсе Analytics](#use-copy-statement) содержат подробные сведения. Пошаговое руководство и пример использования см. в статье [Загрузка 1 ТБ в Azure Synapse Analytics в течение 15 минут с помощью Фабрики данных Azure](load-azure-sql-data-warehouse.md).

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

В следующих разделах содержатся описания свойств, которые определяют сущности Фабрики данных, относящиеся к соединителю Azure Synapse Analytics.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Azure Synapse Analytics поддерживаются следующие свойства.

| Свойство            | Описание                                                  | Обязательно                                                     |
| :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| type                | Для свойства type необходимо задать значение **AzureSqlDW**.             | Да                                                          |
| connectionString    | В свойстве **connectionString** указываются сведения, необходимые для подключения к экземпляру Azure Synapse Analytics. <br/>Пометьте это поле как SecureString, чтобы безопасно хранить его в Фабрике данных. Вы также можете поместить ключ субъекта-службы и пароль в Azure Key Vault и в случае аутентификации SQL получить конфигурацию `password` из строки подключения. Ознакомьтесь с примером JSON под таблицей и с подробными сведениями в статье [Хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md). | Да                                                          |
| servicePrincipalId  | Укажите идентификатора клиента приложения.                         | Значение Yes при использовании проверки подлинности Azure AD на основе субъекта-службы. |
| servicePrincipalKey | Укажите ключ приложения. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Значение Yes при использовании проверки подлинности Azure AD на основе субъекта-службы. |
| tenant              | Укажите сведения о клиенте (доменное имя или идентификатор клиента), в котором находится приложение. Его можно получить, наведя указатель мыши на правый верхний угол страницы портала Azure. | Значение Yes при использовании проверки подлинности Azure AD на основе субъекта-службы. |
| азуреклаудтипе | Для проверки подлинности субъекта-службы укажите тип облачной среды Azure, в которой зарегистрировано приложение Azure AD. <br/> Допустимые значения: `AzurePublic` , `AzureChina` , `AzureUsGovernment` и `AzureGermany` . По умолчанию используется облачная среда фабрики данных. | Нет |
| connectVia          | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Вы можете использовать среду выполнения интеграции Azure или локальную среду IR (если хранилище данных расположено в частной сети). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. | нет                                                           |

В разделах ниже описываются требования и приводятся примеры JSON для разных типов проверки подлинности.

- [Проверка подлинности SQL](#sql-authentication).
- Аутентификация по маркеру приложения Azure AD. [Субъект-служба](#service-principal-authentication)
- Аутентификация по маркеру приложения Azure AD. [Управляемые удостоверения для ресурсов Azure](#managed-identity)

>[!TIP]
>При создании связанной службы для пула SQL без **сервера** Azure синапсе в пользовательском интерфейсе выберите "Ввод вручную" вместо обзора из подписки.

>[!TIP]
>Если вы получили ошибку с кодом ошибки UserErrorFailedToConnectToSqlServer и сообщение типа "Предел сеанса для базы данных — XXX, и он был достигнут", добавьте `Pooling=false` в строку подключения и повторите попытку.

### <a name="sql-authentication"></a>Проверка подлинности SQL

#### <a name="linked-service-example-that-uses-sql-authentication"></a>Пример использования проверки подлинности SQL в связанной службе

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пароль в Azure Key Vault**.

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30",
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

1. **[Создайте приложение Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal)** на портале Azure. Запишите имя приложения и следующие значения, которые используются для определения связанной службы:

   - Идентификатор приложения
   - Ключ приложения
   - Tenant ID

2. **[Предоставьте администратору Azure Active Directory](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-database)** для сервера в портал Azure, если вы еще этого не сделали. Администратор Azure AD может быть пользователем Azure AD или группой Azure AD. Если вы предоставляете группе с управляемым удостоверением роль администратора, пропустите шаги 3 и 4. Администратор будет иметь полный доступ к базе данных.

3. **[Создайте пользователей автономной базы данных](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities)** для субъекта-службы. Подключитесь к хранилищу данных, из которого или в которое требуется скопировать данные с помощью таких средств, как среда SSMS, используя удостоверение Azure AD, у которого есть хотя бы разрешение ALTER ANY USER. Выполните следующий код T-SQL:
  
    ```sql
    CREATE USER [your application name] FROM EXTERNAL PROVIDER;
    ```

4. **Предоставьте субъекту-службе необходимые разрешения** точно так же, как вы предоставляете разрешения пользователям SQL или другим пользователям. Выполните следующий код или посмотрите другие варианты [здесь](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql). Если вы хотите загружать данные с помощью PolyBase, изучите [необходимые разрешения базы данных](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your application name];
    ```

5. **Настройте связанную службу Azure Synapse Analytics** в Фабрике данных Azure.

#### <a name="linked-service-example-that-uses-service-principal-authentication"></a>Пример использования аутентификации на основе субъекта-службы в связанной службе

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30",
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

Фабрика данных может быть связана с [управляемым удостоверением ресурсов Azure](data-factory-service-identity.md), которое представляет определенную фабрику. Это управляемое удостоверение можно использовать для проверки подлинности Azure Synapse Analytics. Назначенная фабрика может получить доступ и скопировать данные из вашего хранилища данных или в него, используя этот идентификатор.

Для использования проверки подлинности по управляемому удостоверению выполните следующие действия.

1. **[Предоставьте администратору Azure Active Directory](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-database)** для сервера на портал Azure, если вы еще этого не сделали. Администратор Azure AD может быть пользователем Azure AD или группой Azure AD. Если вы предоставляете группе с управляемым удостоверением роль администратора, пропустите шаги 3 и 4. Администратор будет иметь полный доступ к базе данных.

2. **[Создайте пользователей автономной базы данных](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities)** для управляемого удостоверения Фабрики данных. Подключитесь к хранилищу данных, из которого или в которое требуется скопировать данные с помощью таких средств, как среда SSMS, используя удостоверение Azure AD, у которого есть хотя бы разрешение ALTER ANY USER. Выполните следующую инструкцию T-SQL.
  
    ```sql
    CREATE USER [your Data Factory name] FROM EXTERNAL PROVIDER;
    ```

3. **Предоставьте управляемому удостоверению Фабрики данных необходимые разрешения**, как это обычно делается для пользователей SQL и других пользователей. Выполните следующий код или посмотрите другие варианты [здесь](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql). Если вы хотите загружать данные с помощью PolyBase, изучите [необходимые разрешения базы данных](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your Data Factory name];
    ```

4. **Настройте связанную службу Azure Synapse Analytics** в Фабрике данных Azure.

**Пример**.

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md).

Для набора данных Azure Synapse Analytics поддерживаются следующие свойства.

| Свойство  | Описание                                                  | Обязательно                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | Свойство **type** для набора данных должно иметь значение: **AzureSqlDWTable**. | Да                         |
| схема | Имя схемы. |"Нет" для источника, "Да" для приемника  |
| table | Имя таблицы или представления. |"Нет" для источника, "Да" для приемника  |
| tableName | Имя таблицы или представления со схемой. Это свойство поддерживается только для обеспечения обратной совместимости. Для новой рабочей нагрузки используйте `schema` и `table`. | "Нет" для источника, "Да" для приемника |

### <a name="dataset-properties-example"></a>Пример свойств набора данных

```json
{
    "name": "AzureSQLDWDataset",
    "properties":
    {
        "type": "AzureSqlDWTable",
        "linkedServiceName": {
            "referenceName": "<Azure Synapse Analytics linked service name>",
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

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых Azure Synapse Analytics как источником и как приемником.

### <a name="azure-synapse-analytics-as-the-source"></a>Azure Synapse Analytics как источник

>[!TIP]
>Чтобы эффективно загружать данные из Azure синапсе Analytics с помощью секционирования данных, Узнайте больше от [параллельного копирования из Azure синапсе Analytics](#parallel-copy-from-azure-synapse-analytics).

Чтобы скопировать данные из Azure Synapse Analytics, задайте для свойства **type** в источнике действия копирования значение **SqlDWSource**. В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство                     | Описание                                                  | Обязательно |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | Свойство **type** источника действия копирования должно иметь значение **SqlDWSource**. | Да      |
| sqlReaderQuery               | Используйте пользовательский SQL-запрос для чтения данных. Например, `select * from MyTable`. | нет       |
| sqlReaderStoredProcedureName | Имя хранимой процедуры, которая считывает данные из исходной таблицы. Последней инструкцией SQL должна быть инструкция SELECT в хранимой процедуре. | нет       |
| storedProcedureParameters    | Параметры для хранимой процедуры.<br/>Допустимые значения: пары имен или значений. Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. | нет       |
| isolationLevel | Задает режим блокировки транзакций для источника данных SQL. Допустимые значения: **ReadCommitted**, **READUNCOMMITTED**, **RepeatableRead**, **Serializable**, **snapshot**. Если значение не указано, используется уровень изоляции базы данных по умолчанию. Дополнительные сведения см. в разделе [System. Data. IsolationLevel](/dotnet/api/system.data.isolationlevel). | Нет |
| partitionOptions | Указывает параметры секционирования данных, используемые для загрузки данных из Azure синапсе Analytics. <br>Допустимые значения: **None** (по умолчанию), **фисикалпартитионсофтабле** и **динамикранже**.<br>Если параметр секции включен (то есть не `None` ), степень параллелизма для параллельной загрузки данных из аналитики Azure синапсе определяется [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) параметром действия копирования. | нет |
| partitionSettings | Позволяет указать группу параметров для секционирования данных. <br>Применяется, если параметр Partition не имеет значение `None` . | Нет |
| ***В разделе `partitionSettings` :*** | | |
| partitionColumnName | Укажите имя исходного столбца **в целочисленном или в типе даты/времени** ( `int` , `smallint` , `bigint` , `date` , `smalldatetime` , `datetime` , `datetime2` или `datetimeoffset` ), которое будет использоваться секционированием по диапазонам для параллельного копирования. Если значение не указано, индекс или первичный ключ таблицы автоматически обнаруживается и используется в качестве столбца секционирования.<br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, присоединитесь к  `?AdfDynamicRangePartitionCondition ` предложению WHERE. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-azure-synapse-analytics) . | нет |
| partitionUpperBound | Максимальное значение столбца секционирования для разделения диапазона секций. Это значение используется для выбора шага секционирования, а не для фильтрации строк в таблице. Все строки в таблице или результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.  <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-azure-synapse-analytics) . | нет |
| partitionLowerBound | Минимальное значение столбца секционирования для разделения диапазона секций. Это значение используется для выбора шага секционирования, а не для фильтрации строк в таблице. Все строки в таблице или результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.<br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-azure-synapse-analytics) . | Нет |

**Обратите внимание на следующий момент:**

- При использовании хранимой процедуры в источнике для получения данных обратите внимание на то, что хранимая процедура разработана так, чтобы она возвращала другую схему при передаче другого значения параметра, может возникнуть сбой или можно увидеть непредвиденный результат при импорте схемы из пользовательского интерфейса или при копировании данных в базу данных SQL с помощью автоматического создания таблиц.

#### <a name="example-using-sql-query"></a>Пример. Использование SQL-запроса

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure Synapse Analytics input dataset name>",
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
                "type": "SqlDWSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="example-using-stored-procedure"></a>Пример. Использование хранимой процедуры

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure Synapse Analytics input dataset name>",
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
                "type": "SqlDWSource",
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

#### <a name="sample-stored-procedure"></a>Пример хранимой процедуры:

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

### <a name="azure-synapse-analytics-as-sink"></a><a name="azure-sql-data-warehouse-as-sink"></a> Azure Synapse Analytics как приемник

Фабрика данных Azure поддерживает три способа загрузки данных в Azure синапсе Analytics.

![Параметры копирования приемника Azure синапсе Analytics](./media/connector-azure-sql-data-warehouse/sql-dw-sink-copy-options.png)

- [Использование PolyBase](#use-polybase-to-load-data-into-azure-synapse-analytics)
- [Использовать инструкцию COPY](#use-copy-statement)
- Использование массовой вставки

Самый быстрый и масштабируемый способ загрузки данных — с помощью [polybase](/sql/relational-databases/polybase/polybase-guide) или [инструкции Copy](/sql/t-sql/statements/copy-into-transact-sql).

Чтобы скопировать данные в Azure синапсе Analytics, задайте для параметра тип приемника в действии копирования значение **SqlDWSink**. В разделе **sink** действия копирования поддерживаются следующие свойства:

| Свойство          | Описание                                                  | Обязательно                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| type              | Свойство **type** приемника действия копирования должно иметь значение **SqlDWSink**. | Да                                           |
| allowPolyBase     | Указывает, следует ли использовать Polybase для загрузки данных в Azure синапсе Analytics. Свойства `allowCopyCommand` и `allowPolyBase` не могут одновременно иметь значение true. <br/><br/>Ограничения и сведения см. в разделе [Загрузка данных в Azure синапсе Analytics с помощью polybase](#use-polybase-to-load-data-into-azure-synapse-analytics) .<br/><br/>Допустимые значения: **true** и **false** (по умолчанию). | Нет.<br/>Применяется при использовании PolyBase.     |
| polyBaseSettings  | Группа свойств, которые можно задать, если свойство `allowPolybase` имеет значение **true**. | Нет.<br/>Применяется при использовании PolyBase. |
| allowCopyCommand | Указывает, следует ли использовать [инструкцию Copy](/sql/t-sql/statements/copy-into-transact-sql) для загрузки данных в Azure синапсе Analytics. Свойства `allowCopyCommand` и `allowPolyBase` не могут одновременно иметь значение true. <br/><br/>Ограничения и сведения см. [в разделе Использование инструкции Copy для загрузки данных в Azure синапсе Analytics](#use-copy-statement) .<br/><br/>Допустимые значения: **true** и **false** (по умолчанию). | Нет.<br>Применяется при использовании инструкции COPY. |
| copyCommandSettings | Группа свойств, которые можно задать, если свойство `allowCopyCommand` имеет значение TRUE. | Нет.<br/>Применяется при использовании инструкции COPY. |
| writeBatchSize    | Число строк для вставки в таблицу SQL **в одном пакете**.<br/><br/>Допустимое значение: **целое число** (количество строк). По умолчанию Фабрика данных динамически определяет соответствующий размер пакета в зависимости от размера строки. | Нет.<br/>Применяется при использовании массовой вставки.     |
| writeBatchTimeout | Время ожидания до выполнения операции пакетной вставки, пока не закончится срок ее действия.<br/><br/>Допустимое значение — **timespan**. Пример "00:30:00" (30 минут). | Нет.<br/>Применяется при использовании массовой вставки.        |
| preCopyScript     | Укажите SQL-запрос для выполнения действия копирования, прежде чем записывать данные в Azure синапсе Analytics при каждом запуске. Это свойство используется для очистки предварительно загруженных данных. | нет                                            |
| tableOption | Указывает, следует ли [автоматически создавать таблицу приемника, если она](copy-activity-overview.md#auto-create-sink-tables) не существует на основе исходной схемы. Допустимые значения: `none` (по умолчанию), `autoCreate`. |нет |
| disableMetricsCollection | Фабрика данных собирает такие метрики, как Azure синапсе Analytics DWU для оптимизации производительности копирования и рекомендации, которые предоставляют дополнительный доступ к базе данных master. Если вас не устраивает такое поведение, укажите `true`, чтобы отключить его. | Нет (значение по умолчанию — `false`) |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

#### <a name="azure-synapse-analytics-sink-example"></a>Пример приемника Azure синапсе Analytics

```json
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true,
    "polyBaseSettings":
    {
        "rejectType": "percentage",
        "rejectValue": 10.0,
        "rejectSampleValue": 100,
        "useTypeDefault": true
    }
}
```

## <a name="parallel-copy-from-azure-synapse-analytics"></a>Параллельная копия из Azure синапсе Analytics

Соединитель Azure синапсе Analytics в действии копирования предоставляет встроенное секционирование данных для параллельного копирования данных. Параметры секционирования данных можно найти на вкладке **источник** действия копирования.

![Снимок экрана с параметрами секционирования](./media/connector-sql-server/connector-sql-partition-options.png)

При включении секционированной копии действие копирования выполняет параллельные запросы к источнику Azure синапсе Analytics для загрузки данных по секциям. Степень параллелизма определяется с помощью параметра [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) для действия копирования. Например, если задано значение `parallelCopies` 4, фабрика данных одновременно создает и выполняет четыре запроса на основе указанного параметра секции и параметров, и каждый запрос извлекает часть данных из аналитики Azure синапсе.

Рекомендуется включить параллельное копирование с секционированием данных, особенно при загрузке большого объема данных из Azure синапсе Analytics. Ниже приведены рекомендуемые конфигурации для разных сценариев. При копировании данных в хранилище данных, основанное на файлах, рекомендуется записывать в папку несколько файлов (указывать только имя папки). в этом случае производительность лучше, чем при записи в один файл.

| Сценарий                                                     | Предлагаемые параметры                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Полная загрузка из большой таблицы с физическими секциями.        | **Параметр секции**: физические разделы таблицы. <br><br/>Во время выполнения фабрика данных автоматически обнаруживает физические секции и копирует данные по секциям. <br><br/>Чтобы проверить, имеет ли таблица физическую секцию, можно обратиться к [этому запросу](#sample-query-to-check-physical-partition). |
| Полная загрузка из большой таблицы без физических секций, в то время как со столбцом типа Integer или DateTime для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Столбец секционирования** (необязательно): укажите столбец, используемый для секционирования данных. Если не указано, используется индекс или столбец первичного ключа.<br/>**Верхняя граница секции** и **Нижняя граница секции** (необязательно): укажите, нужно ли определить шаг секционирования. Это не предназначено для фильтрации строк в таблице, все строки в таблице будут секционированы и скопированы. Если значение не указано, действие копирования автоматически определяет значения.<br><br>Например, если столбец секции "ID" имеет диапазон значений от 1 до 100, а нижняя граница равна 20, а верхняя граница — 80, то при параллельном копировании как 4 фабрика данных извлекает данные по 4 секциям в диапазоне <= 20, [21, 50], [51, 80] и >= 81 соответственно. |
| Загрузка большого объема данных с помощью пользовательского запроса без физических секций, в то время как со столбцом типа Integer или Date/DateTime для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Запрос**: `SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец для секционирования данных.<br>**Верхняя граница секции** и **Нижняя граница секции** (необязательно): укажите, нужно ли определить шаг секционирования. Это не предназначено для фильтрации строк в таблице, все строки в результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.<br><br>Во время выполнения фабрика данных заменяет `?AdfRangePartitionColumnName` фактические имена столбцов и диапазоны значений для каждой секции и отправляет их в Azure синапсе Analytics. <br>Например, если столбец секции "ID" имеет диапазон значений от 1 до 100, а нижняя граница равна 20, а верхняя граница — 80, то при параллельном копировании как 4 фабрика данных извлекает данные по 4 секциям в диапазоне <= 20, [21, 50], [51, 80] и >= 81 соответственно. <br><br>Ниже приведены дополнительные примеры запросов для различных сценариев.<br> 1. запросите всю таблицу: <br>`SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition`<br> 2. запрос из таблицы с выделением столбцов и дополнительными фильтрами WHERE-Order: <br>`SELECT <column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 3. запрос с вложенными запросами: <br>`SELECT <column_list> FROM (<your_sub_query>) AS T WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 4. запрос с Секцией во вложенном запросе: <br>`SELECT <column_list> FROM (SELECT <your_sub_query_column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition) AS T`
|

Рекомендации по загрузке данных с параметром секции:

1. Чтобы избежать неравномерного распределения данных, выберите отличительный столбец в качестве столбца секционирования (например, первичный ключ или уникальный ключ). 
2. Если таблица имеет встроенную секцию, используйте параметр секционирования "физические разделы таблицы", чтобы повысить производительность.
3. Если для копирования данных используется Azure Integration Runtime, можно задать больший размер "[единицы интеграции данных (Диу)](copy-activity-performance-features.md#data-integration-units)" (>4), чтобы использовать больше вычислительных ресурсов. Проверьте применимые сценарии.
4. "[Степень параллелизма копирования](copy-activity-performance-features.md#parallel-copy)" — Управление номерами секций, установка этого числа слишком большого размера, что значительно вредит производительности, рекомендуется задать это число как (Диу или число размещенных в нем IR-узлов) * (от 2 до 4).
5. Примечание. Azure синапсе Analytics может выполнять не более 32 запросов одновременно, в то время как значение «степень параллелизма копирования» слишком велико, может привести к возникновению проблемы регулирования синапсе.

**Пример. полная загрузка из большой таблицы с физическими секциями**

```json
"source": {
    "type": "SqlDWSource",
    "partitionOption": "PhysicalPartitionsOfTable"
}
```

**Пример: запрос с секционированием по динамическому диапазону**

```json
"source": {
    "type": "SqlDWSource",
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
SELECT DISTINCT s.name AS SchemaName, t.name AS TableName, c.name AS ColumnName, CASE WHEN c.name IS NULL THEN 'no' ELSE 'yes' END AS HasPartition
FROM sys.tables AS t
LEFT JOIN sys.objects AS o ON t.object_id = o.object_id
LEFT JOIN sys.schemas AS s ON o.schema_id = s.schema_id
LEFT JOIN sys.indexes AS i ON t.object_id = i.object_id
LEFT JOIN sys.index_columns AS ic ON ic.partition_ordinal > 0 AND ic.index_id = i.index_id AND ic.object_id = t.object_id
LEFT JOIN sys.columns AS c ON c.object_id = ic.object_id AND c.column_id = ic.column_id
LEFT JOIN sys.types AS y ON c.system_type_id = y.system_type_id
WHERE s.name='[your schema]' AND t.name = '[your table name]'
```

Если таблица имеет физическую секцию, вы увидите "Хаспартитион" как "Yes".

## <a name="use-polybase-to-load-data-into-azure-synapse-analytics"></a>Загрузка данных в Azure синапсе Analytics с помощью Polybase

Применение [PolyBase](/sql/relational-databases/polybase/polybase-guide) — это эффективный способ загрузки большого объема данных в Azure Synapse Analytics с высокой пропускной способностью. Используя PolyBase вместо стандартного механизма BULKINSERT, можно значительно увеличить пропускную способность. Пошаговое руководство и пример использования см. в статье [Загрузка 1 ТБ в Azure Synapse Analytics](v1/data-factory-load-sql-data-warehouse.md).

- Если исходные данные находятся в **большом двоичном объекте Azure, Azure Data Lake Storage 1-го поколения или Azure Data Lake Storage 2-го поколения**, а **Формат является совместимым с polybase**, можно использовать действие копирования для непосредственного вызова Polybase, чтобы позволить Azure синапсе Analytics получать данные из источника. Дополнительные сведения см. в разделе **[Прямое копирование с помощью PolyBase](#direct-copy-by-using-polybase)** .
- Если хранилище и формат исходных данных изначально не поддерживаются PolyBase, то можно использовать функцию **[промежуточного копирования с помощью PolyBase](#staged-copy-by-using-polybase)** . Промежуточное копирование также обеспечивает лучшую пропускную способность. Он автоматически преобразует данные в формат, совместимый с Polybase, сохраняет данные в хранилище BLOB-объектов Azure, а затем вызывает Polybase для загрузки данных в Azure синапсе Analytics.

> [!TIP]
> Дополнительные сведения см. в разделе [Рекомендации по использованию PolyBase](#best-practices-for-using-polybase). При использовании Polybase с Azure Integration Runtime эффективными [единицами интеграции данных (Диу)](copy-activity-performance-features.md#data-integration-units) для прямого или промежуточного хранения в синапсе всегда является 2. Настройка Диу не влияет на производительность, так как загрузка данных из хранилища осуществляется ядром синапсе.

В разделе `polyBaseSettings` действия копирования поддерживаются следующие параметры PolyBase.

| Свойство          | Описание                                                  | Обязательно                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| rejectValue       | Указывает количество или процент строк, которые могут быть отклонены, прежде чем запрос завершится с ошибкой.<br/><br/>Дополнительные сведения о параметрах отклонения PolyBase см. в подразделе "Аргументы" раздела [CREATE EXTERNAL TABLE (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql). <br/><br/>Допустимые значения: 0 (по умолчанию), 1, 2 и. т. д. | нет                                            |
| rejectType        | Указывает, является ли параметр **rejectValue** литеральным или процентным.<br/><br/>Допустимые значения: **Значение** (по умолчанию) и **Процент**. | нет                                            |
| rejectSampleValue | Определяет количество строк, которое PolyBase следует получить до повторного вычисления процента отклоненных строк.<br/><br/>Допустимые значения: 1, 2, … | Да, если **rejectType** имеет значение **percentage**. |
| useTypeDefault    | Указывает способ обработки отсутствующих значений в текстовых файлах с разделителями, когда PolyBase извлекает данные из текстового файла.<br/><br/>Дополнительные сведения об этом свойстве см. в подразделе "Аргументы" раздела [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql).<br/><br/>Допустимые значения: **true** и **false** (по умолчанию).<br><br> | нет                                            |

### <a name="direct-copy-by-using-polybase"></a>Прямое копирование с помощью PolyBase

Azure синапсе Analytics Polybase напрямую поддерживает большой двоичный объект Azure, Azure Data Lake Storage 1-го поколения и Azure Data Lake Storage 2-го поколения. Если ваши исходные данные соответствуют условиям, описанным в этом разделе, используйте PolyBase, чтобы выполнять копирование напрямую из исходного хранилища данных в Azure Synapse Analytics. В противном случае см. сведения в разделе [Промежуточное копирование с помощью PolyBase](#staged-copy-by-using-polybase).

> [!TIP]
> Чтобы эффективно копировать данные в Azure синапсе Analytics, Узнайте больше от [фабрики данных Azure, что упрощает и упрощает анализ данных при использовании Data Lake Store с Azure синапсе Analytics](/archive/blogs/azuredatalake/azure-data-factory-makes-it-even-easier-and-convenient-to-uncover-insights-from-data-when-using-data-lake-store-with-sql-data-warehouse).

Если требования не выполняются, фабрика данных Azure проверяет параметры и автоматически возвращается к механизму перемещения данных BULKINSERT.

1. **Связанная служба источника** имеет следующие типы и методы проверки подлинности.

    | Поддерживаемые типы хранилища данных источника                             | Поддерживаемые типы проверки подлинности источника                        |
    | :----------------------------------------------------------- | :---------------------------------------------------------- |
    | [Хранилище BLOB-объектов Azure](connector-azure-blob-storage.md)                | Проверка подлинности на основе ключа учетной записи, проверка подлинности по управляемому удостоверению |
    | [Хранилище Azure Data Lake Gen1](connector-azure-data-lake-store.md) | Проверка подлинности субъекта-службы                            |
    | [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md) | Проверка подлинности на основе ключа учетной записи, проверка подлинности по управляемому удостоверению |

    >[!IMPORTANT]
    >- При использовании аутентификации управляемого удостоверения для связанной службы хранилища изучите необходимые конфигурации для [большого двоичного объекта Azure](connector-azure-blob-storage.md#managed-identity) и [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#managed-identity) соответственно.
    >- Если в службе хранилища Azure настроена конечная точка службы виртуальной сети, необходимо использовать управляемую проверку подлинности с включенным параметром "разрешить использование доверенной службы Майкрософт" в учетной записи хранения. см. статью [влияние использования конечных точек службы виртуальной сети в службе хранилища Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage)

2. **Формат исходных данных** — **Parquet**, **ORC** или **текстовый файл с разделителями** со следующими конфигурациями.

   1. Путь к папке не содержит фильтр с подстановочными знаками.
   2. Имя файла пустое или указывает на единственный файл. При указании имени файла в действии копирования можно использовать только подстановочные знаки `*` или `*.*`.
   3. Параметр `rowDelimiter` имеет **значение по умолчанию**, **\n**, **\r\n** или **\r**.
   4. В параметре `nullValue` оставляется значение по умолчанию или задается **пустая строка** (""), а в параметре `treatEmptyAsNull` оставляется значение по умолчанию или задается значение true.
   5. В параметре `encodingName` оставляется значение по умолчанию или задается значение **utf-8**.
   6. Параметры `quoteChar`, `escapeChar` и `skipLineCount` не задаются. Поддержка PolyBase пропускает строку заголовка, которую в файле определения приложения можно настроить как `firstRowAsHeader`.
   7. `compression` не может иметь **Сжатие**, **``GZip``** или **Deflate**.

3. Если источником является папка, параметр `recursive` в действии копирования должен иметь значение true.

4. `wildcardFolderPath` , `wildcardFilename` , `modifiedDateTimeStart` , `modifiedDateTimeEnd` , `prefix` , и `enablePartitionDiscovery` `additionalColumns` не указаны.

>[!NOTE]
>Обратите внимание, что если источником является папка, то PolyBase извлекает файлы из этой папки и всех вложенных в нее папок, но не извлекает данные из файлов, имена которых начинаются с подчеркивания (_) или точки (.) (см. [описание аргумента LOCATION здесь](/sql/t-sql/statements/create-external-table-transact-sql#arguments-2)).

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "ParquetDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ParquetSource",
                "storeSettings":{
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            }
        }
    }
]
```

### <a name="staged-copy-by-using-polybase"></a>Промежуточное копирование с помощью PolyBase

Если исходные данные не совместимы с Polybase, включите копирование данных с помощью промежуточного хранилища больших двоичных объектов Azure или Azure Data Lake Storage 2-го поколения (это не может быть хранилище Azure класса Premium). В таком случае Фабрика данных Azure автоматически преобразует данные, чтобы они соответствовали требованиям к формату данных PolyBase. Затем он вызывает Polybase для загрузки данных в Azure синапсе Analytics. Наконец, он очищает временные данные из хранилища. Дополнительные сведения о копировании данных с помощью промежуточного хранения см. в разделе [поэтапное копирование](copy-activity-performance-features.md#staged-copy) .

Чтобы использовать эту функцию, создайте [связанную службу хранилища BLOB-объектов Azure](connector-azure-blob-storage.md#linked-service-properties) или [связанную службу Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#linked-service-properties) с **ключом учетной записи или проверкой подлинности управляемого удостоверения** , которая ссылается на учетную запись хранения Azure в качестве промежуточного хранилища.

>[!IMPORTANT]
>- При использовании проверки подлинности с управляемым удостоверением для промежуточной связанной службы изучите необходимые конфигурации для [большого двоичного объекта Azure](connector-azure-blob-storage.md#managed-identity) и [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#managed-identity) соответственно.
>- Если в промежуточной службе хранилища Azure настроена конечная точка службы виртуальной сети, необходимо использовать управляемую аутентификацию с проверкой подлинности с включенным параметром "разрешить использование доверенной службы Майкрософт" в учетной записи хранения. см. статью [влияние использования конечных точек службы виртуальной сети](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage) 

>[!IMPORTANT]
>Если в промежуточной службе хранилища Azure настроена управляемая частная конечная точка с включенным брандмауэром хранилища, необходимо использовать управляемую аутентификацию с проверкой подлинности и предоставить разрешения читателя данных BLOB-объектов хранилища для SQL Server синапсе, чтобы обеспечить доступ к промежуточным файлам во время загрузки Polybase.

```json
"activities":[
    {
        "name": "CopyFromSQLServerToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "SQLServerDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingStorage",
                    "type": "LinkedServiceReference"
                }
            }
        }
    }
]
```

### <a name="best-practices-for-using-polybase"></a>Советы и рекомендации по использованию PolyBase

В следующих разделах приводятся рекомендации в дополнение к рекомендациям, упомянутым в статье рекомендации [по использованию Azure синапсе Analytics](../synapse-analytics/sql/best-practices-dedicated-sql-pool.md).

#### <a name="required-database-permission"></a>Необходимые разрешения базы данных

Чтобы использовать Polybase, пользователь, загружающий данные в Azure синапсе Analytics, должен иметь [разрешение CONTROL](/sql/relational-databases/security/permissions-database-engine) на целевую базу данных. Это разрешение можно получить, добавив этого пользователя как участника роли **db_owner**. Сведения о том, как это сделать, см. в [обзоре Azure синапсе Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-manage-security.md#authorization).

#### <a name="row-size-and-data-type-limits"></a>Ограничение размера строки и типа данных

Загрузки PolyBase ограничены записями размером менее 1 МБ. PolyBase нельзя использовать для загрузки в типы данных VARCHR(MAX), NVARCHAR(MAX) или VARBINARY(MAX). Дополнительные сведения см. в статье [ограничения емкости службы аналитики Azure синапсе](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md#loads).

Если исходные данные имеют записи размером более 1 МБ, можно попробовать вертикально разбить исходные таблицы на несколько небольших. Убедитесь, что максимальный размер каждой записи не превышает предел. Затем эти небольшие таблицы можно загрузить с помощью PolyBase и слить вместе в Azure Synapse Analytics.

Кроме того, для данных с такими широкими столбцами можно не использовать Polybase, а загружать их с помощью файла определения приложения, отключив параметр "allow PolyBase".

#### <a name="azure-synapse-analytics-resource-class"></a>Класс ресурсов Azure синапсе Analytics

Чтобы добиться максимальной пропускной способности, назначьте пользователю больший класс ресурсов, который загружает данные в Azure синапсе Analytics через Polybase.

#### <a name="polybase-troubleshooting"></a>Устранение неполадок c PolyBase

#### <a name="loading-to-decimal-column"></a>Загрузка в столбец с типом данных Decimal

Если исходные данные имеют текстовый формат или другие совместимые с Polybase хранилища (с использованием промежуточного копирования и Polybase) и содержат пустое значение для загрузки в десятичный столбец аналитики Azure синапсе, может возникнуть следующая ошибка:

```output
ErrorCode=FailedDbOperation, ......HadoopSqlException: Error converting data type VARCHAR to DECIMAL.....Detailed Message=Empty string can't be converted to DECIMAL.....
```

Решение заключается в том, чтобы снять флажок "**Использовать тип по умолчанию**" (т. е. установить для этого параметра значение false) в разделе "Параметры PolyBase" приемника действия копирования. "[USE_TYPE_DEFAULT](/sql/t-sql/statements/create-external-file-format-transact-sql#arguments)" — собственная конфигурация PolyBase, которая указывает способ обработки отсутствующих значений в текстовых файлах с разделителями, когда PolyBase извлекает данные из текстового файла.

#### <a name="check-the-tablename-property-in-azure-synapse-analytics"></a>Проверка свойства tableName в Azure синапсе Analytics

В следующей таблице приведены примеры того, как указать свойство **tableName** в наборе данных JSON. В ней показаны несколько сочетаний имен схем и таблиц.

| Схема базы данных | Имя таблицы | Свойство **tableName** в JSON               |
| --------- | ---------- | ----------------------------------------- |
| dbo       | MyTable    | MyTable или dbo.MyTable либо [dbo].[MyTable] |
| dbo1      | MyTable    | dbo1.MyTable или [dbo1].[MyTable]          |
| dbo       | My.Table   | [My.Table] или [dbo].[My.Table]            |
| dbo1      | My.Table   | [dbo1].[My.Table]                         |

Если вы видите следующую ошибку, это может указывать на неправильное значение свойства **tableName**. Правильные значения для свойства **tableName** в JSON см. в таблице выше.

```output
Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider
```

#### <a name="columns-with-default-values"></a>Столбцы со значениями по умолчанию

Сейчас для PolyBase в фабрике данных требуется такое же количество столбцов, как и в целевой таблице. Примером может служить таблица с четырьмя столбцами, и один из них определен со значением по умолчанию. Входные данные по-прежнему должны иметь четыре столбца. Если входной набор данных будет содержать три столбца, мы получим ошибку, похожую на следующую:

```output
All columns of the table must be specified in the INSERT BULK statement.
```

Значение NULL рассматривается как вариант значения по умолчанию. Если столбец допускает значение NULL, входные данные большого двоичного объекта для этого столбца могут быть пустыми. Но они не могут отсутствовать во входном наборе данных. Для отсутствующих значений PolyBase будет вставлять в Azure Synapse Analytics значения NULL.

#### <a name="external-file-access-failed"></a>Сбой доступа к внешним файлам

Если вы получаете следующую ошибку, убедитесь, что вы используете проверку подлинности с управляемым удостоверением и получили разрешения на чтение данных BLOB-объекта хранилища для управляемого удостоверения рабочей области Azure синапсе.

```output
Job failed due to reason: at Sink '[SinkName]': shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: External file access failed due to internal error: 'Error occurred while accessing HDFS: Java exception raised on call to HdfsBridge_IsDirExist. Java exception message:\r\nHdfsBridge::isDirExist 
```

Дополнительные сведения см. [в разделе Предоставление разрешений управляемому удостоверению после создания рабочей области](../synapse-analytics/security/how-to-grant-workspace-managed-identity-permissions.md#grant-permissions-to-managed-identity-after-workspace-creation).

## <a name="use-copy-statement-to-load-data-into-azure-synapse-analytics"></a><a name="use-copy-statement"></a> Использование инструкции COPY для загрузки данных в Azure синапсе Analytics

[Инструкция копирования](/sql/t-sql/statements/copy-into-transact-sql) Azure синапсе Analytics напрямую поддерживает загрузку данных из **большого двоичного объекта Azure и Azure Data Lake Storage 2-го поколения**. Если исходные данные соответствуют критериям, описанным в этом разделе, можно выбрать использование инструкции COPY в ADF для загрузки данных в Azure синапсе Analytics. Фабрика данных Azure проверяет параметры и не выполняет действие копирования, если условия не выполнены.

>[!NOTE]
>В настоящее время Фабрика данных поддерживает копирование только из источников, совместимых с инструкцией COPY, которые приведены ниже.

>[!TIP]
>При использовании инструкции COPY с Azure Integration Runtime в качестве эффективных [единиц интеграции данных (Диу)](copy-activity-performance-features.md#data-integration-units) всегда используется 2. Настройка Диу не влияет на производительность, так как загрузка данных из хранилища осуществляется ядром синапсе.

При использовании инструкции COPY поддерживается следующая конфигурация.

1. **Связанная служба и формат источника** могут иметь следующие типы и методы проверки подлинности.

    | Поддерживаемые типы хранилища данных источника                             | Поддерживаемые форматы           | Поддерживаемые типы проверки подлинности источника                         |
    | :----------------------------------------------------------- | -------------------------- | :----------------------------------------------------------- |
    | [Хранилище BLOB-объектов Azure](connector-azure-blob-storage.md)                | [Текст с разделителями](format-delimited-text.md)             | Проверка подлинности на основе ключа учетной записи, проверка подлинности с помощью подписанного URL-адреса, проверка подлинности на основе субъекта-службы, проверка подлинности по управляемому удостоверению |
    | &nbsp;                                                       | [Parquet](format-parquet.md)                    | Проверка подлинности на основе ключа учетной записи, проверка подлинности с помощью подписанного URL-адреса |
    | &nbsp;                                                       | [ORC](format-orc.md)                        | Проверка подлинности на основе ключа учетной записи, проверка подлинности с помощью подписанного URL-адреса |
    | [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md) | [Текст с разделителями](format-delimited-text.md)<br/>[Parquet](format-parquet.md)<br/>[ORC](format-orc.md) | Проверка подлинности на основе ключа учетной записи, проверка подлинности на основе субъекта-службы, проверка подлинности по управляемому удостоверению |

    >[!IMPORTANT]
    >- При использовании аутентификации управляемого удостоверения для связанной службы хранилища изучите необходимые конфигурации для [большого двоичного объекта Azure](connector-azure-blob-storage.md#managed-identity) и [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#managed-identity) соответственно.
    >- Если в службе хранилища Azure настроена конечная точка службы виртуальной сети, необходимо использовать управляемую проверку подлинности с включенным параметром "разрешить использование доверенной службы Майкрософт" в учетной записи хранения. см. статью [влияние использования конечных точек службы виртуальной сети в службе хранилища Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage)

2. Параметры формата могут быть следующие.

   1. Для **Parquet**: `compression` не может иметь **Сжатие**, **привязку** или **``GZip``** .
   2. Для **ORC**: в параметре `compression` может быть задано **без сжатия**, **```zlib```** или **Snappy**.
   3. Для **текста с разделителями**:
      1. В параметре `rowDelimiter` может быть явно задан **один символ** или " **\r\n**", значение по умолчанию не поддерживается.
      2. В параметре `nullValue` может быть оставлено значение по умолчанию или задана **пустая строка** ("").
      3. В параметре `encodingName` может быть оставлено значение по умолчанию или задано значение **utf-8 или utf-16**.
      4. Параметр `escapeChar` должен совпадать с параметром `quoteChar` и не быть пустым.
      5. В параметре `skipLineCount` может быть оставлено значение по умолчанию или задано значение 0.
      6. `compression` не может быть **сжатием** или **``GZip``** .

3. Если источником является папка, параметр `recursive` в действии копирования должен иметь значение true, а в параметре `wildcardFilename` должен быть задан подстановочный знак `*`. 

4. `wildcardFolderPath` , `wildcardFilename` (кроме `*` ), `modifiedDateTimeStart` ,, `modifiedDateTimeEnd` и `prefix` `enablePartitionDiscovery` `additionalColumns` не указаны.

В разделе `allowCopyCommand` действия копирования поддерживаются следующие параметры инструкции COPY.

| Свойство          | Описание                                                  | Обязательно                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| defaultValues | Задает значения по умолчанию для каждого целевого столбца в Azure синапсе Analytics.  Значения по умолчанию из этого свойства переопределяют ограничение DEFAULT, заданное в хранилище данных, а столбец идентификаторов не может иметь значение по умолчанию. | нет |
| additionalOptions | Дополнительные параметры, которые будут переданы в инструкцию COPY в Azure синапсе Analytics непосредственно в предложении WITH [инструкции Copy](/sql/t-sql/statements/copy-into-transact-sql). Значение необходимо заключать в кавычки для соответствия требованиям инструкции COPY. | нет |

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaCOPY",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "ParquetDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ParquetSource",
                "storeSettings":{
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowCopyCommand": true,
                "copyCommandSettings": {
                    "defaultValues": [
                        {
                            "columnName": "col_string",
                            "defaultValue": "DefaultStringValue"
                        }
                    ],
                    "additionalOptions": {
                        "MAXERRORS": "10000",
                        "DATEFORMAT": "'ymd'"
                    }
                }
            },
            "enableSkipIncompatibleRow": true
        }
    }
]
```

## <a name="mapping-data-flow-properties"></a>Свойства потока данных для сопоставления

При преобразовании данных в потоке данных для сопоставления можно выполнять операции чтения и записи в таблицах из Azure Synapse Analytics. Дополнительные сведения см. в описаниях [преобразования источника](data-flow-source.md) и [преобразования приемника](data-flow-sink.md) в разделе, посвященном потокам данных для сопоставления.

### <a name="source-transformation"></a>Преобразование источника

Параметры, относящиеся к Azure Synapse Analytics, находятся на вкладке **Параметры источника** преобразования источника.

**Входные данные** Укажите, следует ли указывать источник в таблице (эквивалент ```Select * from <table-name>``` ) или введите пользовательский SQL-запрос.

**Включить промежуточное хранение** Настоятельно рекомендуется использовать этот параметр в рабочих нагрузках с источниками Azure синапсе Analytics. При выполнении [действия потока данных](control-flow-execute-data-flow-activity.md) с источниками Azure синапсе Analytics из КОНВЕЙЕРа ADF запросит учетную запись хранения промежуточного расположения и будет использовать ее для промежуточной загрузки данных. Это самый быстрый механизм загрузки данных из Azure синапсе Analytics.

- При использовании аутентификации управляемого удостоверения для связанной службы хранилища изучите необходимые конфигурации для [большого двоичного объекта Azure](connector-azure-blob-storage.md#managed-identity) и [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#managed-identity) соответственно.
- Если в службе хранилища Azure настроена конечная точка службы виртуальной сети, необходимо использовать управляемую проверку подлинности с включенным параметром "разрешить использование доверенной службы Майкрософт" в учетной записи хранения. см. статью [влияние использования конечных точек службы виртуальной сети в службе хранилища Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage)
- Если в качестве источника используется пул SQL без **сервера** Azure синапсе, включение промежуточного хранения не поддерживается.

**Запрос.** Если в поле входных данных вы выбрали запрос, введите здесь SQL-запрос для источника. Этот параметр переопределяет любую таблицу, выбранную в наборе данных. Предложения **Order By** здесь не поддерживаются, но можно задать полную инструкцию SELECT FROM. Кроме того, можно использовать табличные функции, определяемые пользователем. Пример использования такой функции в SQL — инструкция **select * from udfGetData()** , возвращающая таблицу. Этот запрос создаст исходную таблицу, которую можно использовать в потоке данных. Использование запросов — также отличный способ сокращения количества строк для тестирования или поиска.

Пример SQL: ```Select * from MyTable where customerId > 1000 and customerId < 2000```

**Размер пакета.** Введите размер пакета для разделения больших наборов данных на блоки для чтения. В потоках данных ADF будет использовать этот параметр для установки кэширования Spark по столбцам. Это поле параметра, в котором будут использоваться параметры Spark по умолчанию, если они не заданы.

**Уровень изоляции**. Значение по умолчанию для источников SQL в потоке данных для сопоставления — Read Uncommitted (чтение незафиксированных данных). Здесь можно изменить уровень изоляции на одно из следующих значений.

- Read Committed (чтение зафиксированных данных)
- Read Uncommitted (чтение незафиксированных данных)
- Повторяющаяся операция чтения
- Упорядочиваемый уровень изоляции
- None (игнорировать уровень изоляции)

![Уровень изоляции](media/data-flow/isolationlevel.png)

### <a name="sink-transformation"></a>Преобразование приемника

Параметры, относящиеся к Azure Synapse Analytics, находятся на вкладке **Параметры**  преобразования приемника.

**Метод обновления**. Определяет, какие операции разрешены в назначении базы данных. По умолчанию разрешены только операции вставки. Для выполнения обновления (update), обновления или вставки (upsert) или удаления (delete) строк требуется преобразование alter-row строк, отмеченных для этих действий. Для выполнения обновления (update), обновления или вставки (upsert) или удаления (delete) должен быть установлен ключевой столбец (или столбцы), позволяющий определить строки для изменения.

**Действие таблицы**. Определяет, следует ли повторно создавать или удалять все строки в целевой таблице перед записью.

- None (нет): никакие действия над таблицей не выполняются.
- Recreate (создать повторно): таблица будет удалена и создана повторно. Это действие необходимо, если новая таблица создается динамически.
- Truncate (усечь): все строки из целевой таблицы будут удалены.

**Включить промежуточное хранение:** Это позволяет загружать пулы SQL Azure синапсе Analytics с помощью команды Copy и рекомендуется для большинства приемников Синпасе. Промежуточное хранилище настраивается в [действии "выполнение потока данных](control-flow-execute-data-flow-activity.md)". 

- При использовании аутентификации управляемого удостоверения для связанной службы хранилища изучите необходимые конфигурации для [большого двоичного объекта Azure](connector-azure-blob-storage.md#managed-identity) и [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#managed-identity) соответственно.
- Если в службе хранилища Azure настроена конечная точка службы виртуальной сети, необходимо использовать управляемую проверку подлинности с включенным параметром "разрешить использование доверенной службы Майкрософт" в учетной записи хранения. см. статью [влияние использования конечных точек службы виртуальной сети в службе хранилища Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage)

**Размер пакета.** Определяет, сколько строк записывается в каждую группу. Более крупные размеры пакетов улучшают сжатие и оптимизацию памяти, но при кэшировании данных возникает риск нехватки памяти.

**Скрипты SQL предобработки и постобработки**. Введите многострочные скрипты SQL, которые будут выполняться до (предобработка) и после (постобработка) записи данных в базу данных-приемник.

![Скрипты SQL предобработки и постобработки](media/data-flow/prepost1.png "Скрипты обработки SQL")

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Свойства действия GetMetadata

Подробные сведения об этих свойствах см. в разделе [Действие GetMetadata](control-flow-get-metadata-activity.md)

## <a name="data-type-mapping-for-azure-synapse-analytics"></a>Сопоставление типов данных для Azure Synapse Analytics

При копировании данных в Azure Synapse Analytics и обратно используются следующие сопоставления между типами данных Azure Synapse Analytics и промежуточными типами данных Фабрики данных Azure. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

>[!TIP]
>См. статью [табличные типы данных в Azure синапсе Analytics](../synapse-analytics/sql/develop-tables-data-types.md) в статье Поддерживаемые типы данных Azure синапсе Analytics и обходные пути для неподдерживаемых.

| Тип данных Azure Synapse Analytics    | Тип промежуточных данных фабрики данных |
| :------------------------------------ | :----------------------------- |
| BIGINT                                | Int64                          |
| binary                                | Byte[]                         |
| bit                                   | Логическое                        |
| char                                  | String, Char[]                 |
| Дата                                  | Дата и время                       |
| Datetime                              | Дата и время                       |
| datetime2                             | Дата и время                       |
| Datetimeoffset                        | DateTimeOffset                 |
| Decimal                               | Decimal                        |
| FILESTREAM attribute (varbinary(max)) | Byte[]                         |
| Float                                 | Double                         |
| Изображение                                 | Byte[]                         |
| INT                                   | Int32                          |
| money                                 | Decimal                        |
| nchar                                 | String, Char[]                 |
| NUMERIC                               | Decimal                        |
| nvarchar                              | String, Char[]                 |
| real                                  | Один                         |
| rowversion                            | Byte[]                         |
| smalldatetime                         | Дата и время                       |
| smallint                              | Int16                          |
| smallmoney                            | Decimal                        |
| time                                  | TimeSpan                       |
| tinyint                               | Byte                           |
| UNIQUEIDENTIFIER                      | Guid                           |
| varbinary                             | Byte[]                         |
| varchar                               | String, Char[]                 |

## <a name="next-steps"></a>Дальнейшие действия

В таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.

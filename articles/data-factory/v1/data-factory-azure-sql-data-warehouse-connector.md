---
title: Копирование данных в Azure синапсе Analytics и обратно
description: Узнайте, как копировать данные в Azure синапсе Analytics и обратно с помощью фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: aa364ec434db980bf226008537ca928628fcac1b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100392091"
---
# <a name="copy-data-to-and-from-azure-synapse-analytics-using-azure-data-factory"></a>Копирование данных в Azure синапсе Analytics и обратно с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-azure-sql-data-warehouse-connector.md)
> * [Версия 2 (текущая)](../connector-azure-sql-data-warehouse.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию службы фабрики данных, см. статью [Azure синапсе Analytics Connector в версии 2](../connector-azure-sql-data-warehouse.md).

В этой статье объясняется, как с помощью действия копирования в фабрике данных Azure перемещать данные в Azure синапсе Analytics и обратно. Она основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , в которой представлен общий обзор перемещения данных с помощью действия копирования.

> [!TIP]
> Для обеспечения лучшей производительности загружайте данные в Azure Synapse Analytics с помощью PolyBase. Дополнительные сведения см. в разделе [Загрузка данных в Azure Synapse Analytics с помощью PolyBase](#use-polybase-to-load-data-into-azure-synapse-analytics). Пошаговое руководство и пример использования см. в статье [Загрузка 1 ТБ в Azure Synapse Analytics в течение 15 минут с помощью Фабрики данных Azure](data-factory-load-sql-data-warehouse.md).

## <a name="supported-scenarios"></a>Поддерживаемые сценарии
Данные **из Azure синапсе Analytics** можно скопировать в следующие хранилища данных:

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

Вы можете скопировать данные из следующих хранилищ данных **в Azure синапсе Analytics**:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!TIP]
> Если при копировании данных из SQL Server или базы данных SQL Azure в Azure синапсе Analytics таблица не существует в целевом хранилище, фабрика данных может автоматически создать таблицу в Azure синапсе Analytics, используя схему таблицы в исходном хранилище данных. Дополнительные сведения см. в разделе [Автоматическое создание таблицы](#auto-table-creation).

## <a name="supported-authentication-type"></a>Поддерживаемый тип проверки подлинности
Соединитель Azure синапсе Analytics поддерживает обычную проверку подлинности.

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, которое перемещает данные в Azure синапсе Analytics и обратно с помощью различных инструментов и интерфейсов API.

Самый простой способ создать конвейер, копирующий данные в Azure синапсе Analytics или из него, — использовать мастер копирования данных. Краткое пошаговое руководство по созданию конвейера с помощью мастера копирования данных см. в статье [учебник. Загрузка данных в Azure синапсе Analytics с помощью фабрики данных](../load-azure-sql-data-warehouse.md) .

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **фабрику данных**. Фабрика данных может содержать один или несколько конвейеров. 
2. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных. Например, при копировании данных из хранилища BLOB-объектов Azure в Azure синапсе Analytics вы создадите две связанные службы, связывающие учетную запись хранения Azure и Azure синапсе Analytics с фабрикой данных. Сведения о свойствах связанной службы, относящихся к Azure синапсе Analytics, см. в разделе [Свойства связанной службы](#linked-service-properties) . 
3. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. В примере, упомянутом на последнем этапе, создайте набор данных, чтобы указать контейнер BLOB-объектов и папку, содержащую входные данные. Кроме того, вы создаете другой набор данных, чтобы указать таблицу в Azure синапсе Analytics, содержащую данные, скопированные из хранилища BLOB-объектов. Сведения о свойствах набора данных, относящихся к Azure синапсе Analytics, см. в разделе [Свойства набора данных](#dataset-properties) .
4. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. В примере выше BlobSource используется как источник, а SqlDWSink — как приемник для действия копирования. Аналогично, если вы копируете из Azure синапсе Analytics в хранилище BLOB-объектов Azure, вы используете SqlDWSource и BlobSink в действии копирования. Сведения о свойствах действия копирования, которые относятся к Azure синапсе Analytics, см. в разделе [Свойства действия копирования](#copy-activity-properties) . Для получения сведений о том, как использовать хранилище данных в качестве источника или приемника, щелкните ссылку в предыдущем разделе об источнике данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Примеры с определениями JSON для сущностей фабрики данных, которые используются для копирования данных в Azure синапсе Analytics и обратно, см. в разделе [примеры JSON](#json-examples-for-copying-data-to-and-from-azure-synapse-analytics) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, относящихся к Azure синапсе Analytics.

## <a name="linked-service-properties"></a>Свойства связанной службы
В следующей таблице содержится описание элементов JSON, относящихся к связанной службе Azure синапсе Analytics.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type |Для свойства Type необходимо задать значение **AzureSqlDW** . |Да |
| connectionString |Укажите сведения, необходимые для подключения к экземпляру Azure синапсе Analytics для свойства connectionString. Поддерживается только обычная проверка подлинности. |Да |

> [!IMPORTANT]
> [Брандмауэр Базы данных SQL Azure](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure) необходимо настроить для [разрешения службам Azure доступа к серверу](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure). Кроме того, при копировании данных в Azure синапсе Analytics из-за пределов Azure, включая из локальных источников данных с помощью шлюза фабрики данных, настройте соответствующий диапазон IP-адресов для компьютера, отправляющего данные в Azure синапсе Analytics.

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных (SQL Azure, большие двоичные объекты Azure, таблицы Azure и т. д.).

Раздел typeProperties во всех типах наборов данных разный. В нем содержатся сведения о расположении данных в хранилище данных. Раздел **typeProperties** набора данных типа **AzureSqlDWTable** имеет следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| tableName |Имя таблицы или представления в базе данных Azure синапсе Analytics, на которую ссылается связанная служба. |Да |

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства (включая имя, описание, входные и выходные таблицы, политику и т. д.) доступны для всех типов действий.

> [!NOTE]
> Действие копирования принимает только один набор входных данных и возвращает только один набор выходных.

В свою очередь свойства, доступные в разделе typeProperties действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

### <a name="sqldwsource"></a>SqlDWSource
Когда источник относится к типу **SqlDWSource**, в разделе **typeProperties** доступны следующие свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| sqlReaderQuery |Используйте пользовательский запрос для чтения данных. |Строка запроса SQL. Например, select * from MyTable. |нет |
| sqlReaderStoredProcedureName |Имя хранимой процедуры, которая считывает данные из исходной таблицы. |Имя хранимой процедуры. Последней инструкцией SQL должна быть инструкция SELECT в хранимой процедуре. |нет |
| storedProcedureParameters |Параметры для хранимой процедуры. |Пары имен и значений. Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. |нет |

Если для SqlDWSource указан **sqlReaderQuery** , действие копирования выполняет этот запрос к источнику аналитики Azure синапсе для получения данных.

Кроме того, можно создать хранимую процедуру, указав **sqlReaderStoredProcedureName** и **storedProcedureParameters** (если хранимая процедура принимает параметры).

Если не указать ни sqlReaderQuery, ни sqlReaderStoredProcedureName, то для создания запроса, который будет выполняться в Azure синапсе Analytics, будут использоваться столбцы, определенные в разделе структура JSON-файла набора данных. Например, `select column1, column2 from mytable`. Если у определения набора данных нет структуры, выбираются все столбцы из таблицы.

#### <a name="sqldwsource-example"></a>Пример SqlDWSource

```JSON
"source": {
    "type": "SqlDWSource",
    "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
    "storedProcedureParameters": {
        "stringData": { "value": "str3" },
        "identifier": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
    }
}
```
**Определение хранимой процедуры:**

```SQL
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

### <a name="sqldwsink"></a>SqlDWSink
**SqlDWSink** поддерживает указанные ниже свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| sqlWriterCleanupScript |Укажите запрос на выполнение действия копирования, позволяющий убедиться в том, что данные конкретного среза очищены. Дополнительные сведения см. в [разделе о повторяемости](#repeatability-during-copy). |Инструкция запроса. |Нет |
| allowPolyBase |Указывает, следует ли использовать PolyBase (если применимо) вместо механизма BULKINSERT. <br/><br/> **Использование Polybase — рекомендуемый способ загрузки данных в Azure синапсе Analytics.** Ограничения и сведения см. в разделе [Загрузка данных в Azure синапсе Analytics с помощью polybase](#use-polybase-to-load-data-into-azure-synapse-analytics) . |True <br/>False (по умолчанию) |Нет |
| polyBaseSettings |Группа свойств, которые можно задать, если свойство **allowPolybase** имеет значение **true**. |&nbsp; |Нет |
| rejectValue |Указывает количество или процент строк, которые могут быть отклонены, прежде чем запрос завершится с ошибкой. <br/><br/>Дополнительные сведения о параметрах отклонения Polybase см. в разделе **arguments** статьи [CREATE External Table (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql) . |0 (по умолчанию), 1, 2, ... |нет |
| rejectType |Указывает тип значения, заданного для параметра rejectValue: литеральное значение или процент. |Value (по умолчанию), Percentage |нет |
| rejectSampleValue |Определяет количество строк, которое PolyBase следует получить до повторного вычисления процента отклоненных строк. |1, 2, … |Да, если **rejectType** имеет значение **percentage**. |
| useTypeDefault |Указывает способ обработки отсутствующих значений в текстовых файлах с разделителями, когда PolyBase извлекает данные из текстового файла.<br/><br/>Дополнительные сведения об этом свойстве см. в подразделе "Аргументы" раздела [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql). |True, False (по умолчанию) |Нет |
| writeBatchSize |Вставляет данные в таблицу SQL, когда размер буфера достигает значения writeBatchSize. |Целое число (количество строк) |Нет (значение по умолчанию — 10 000). |
| writeBatchTimeout |Время ожидания до выполнения операции пакетной вставки, пока не завершится срок ее действия. |Интервал времени<br/><br/> Пример "00:30:00" (30 минут). |Нет |

#### <a name="sqldwsink-example"></a>Пример SqlDWSink

```JSON
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true
}
```

## <a name="use-polybase-to-load-data-into-azure-synapse-analytics"></a>Загрузка данных в Azure синапсе Analytics с помощью Polybase
Использование **[polybase](/sql/relational-databases/polybase/polybase-guide)** — это эффективный способ загрузки большого объема данных в Azure синапсе Analytics с высокой пропускной способностью. Используя PolyBase вместо стандартного механизма BULKINSERT, можно значительно увеличить пропускную способность. Подробное сравнение см. в разделе [Базовые показатели производительности](data-factory-copy-activity-performance.md#performance-reference). Пошаговое руководство и пример использования см. в статье [Загрузка 1 ТБ в Azure Synapse Analytics в течение 15 минут с помощью Фабрики данных Azure](data-factory-load-sql-data-warehouse.md).

* Если исходные данные находятся в хранилище **BLOB-объектов Azure или Azure Data Lake Store**, а формат совместим с polybase, вы можете напрямую скопировать их в Azure синапсе Analytics с помощью polybase. Дополнительные сведения см. в разделе **[Прямое копирование с помощью PolyBase](#direct-copy-using-polybase)**.
* Если хранилище и формат исходных данных изначально не поддерживаются PolyBase, то можно использовать функцию **[промежуточного копирования с помощью PolyBase](#staged-copy-using-polybase)**. Она также обеспечивает лучшую пропускную способность за счет автоматического преобразования данных в формат, совместимый с PolyBase, и хранения данных в хранилище BLOB-объектов Azure. Затем данные загружаются в Azure синапсе Analytics.

Присвойте `allowPolyBase` свойству **значение true** , как показано в следующем примере, чтобы фабрика данных Azure использовала polybase для копирования данных в Azure синапсе Analytics. Если присвоить allowPolyBase значение true, то можно указать определенные свойства PolyBase, используя группу свойств `polyBaseSettings`. В разделе [SqlDWSink](#sqldwsink) представлены дополнительные сведения о свойствах, которые можно использовать с polyBaseSettings.

```JSON
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

### <a name="direct-copy-using-polybase"></a>Прямое копирование с помощью PolyBase
Azure синапсе Analytics Polybase напрямую поддерживает большой двоичный объект Azure и Azure Data Lake Store (с помощью субъекта-службы) в качестве источника и с конкретными требованиями к формату файлов. Если исходные данные соответствуют критериям, описанным в этом разделе, можно напрямую копировать данные из исходного хранилища данных в Azure синапсе Analytics с помощью Polybase. В противном случае можно использовать [промежуточное копирование с помощью PolyBase](#staged-copy-using-polybase).

> [!TIP]
> Чтобы эффективно копировать данные из Data Lake Store в Azure синапсе Analytics, изучите дополнительные сведения из [фабрики данных Azure. это еще проще и удобно для анализа данных при использовании Data Lake Store с Azure синапсе Analytics](/archive/blogs/azuredatalake/azure-data-factory-makes-it-even-easier-and-convenient-to-uncover-insights-from-data-when-using-data-lake-store-with-sql-data-warehouse).

Если требования не выполняются, фабрика данных Azure проверяет параметры и автоматически возвращается к механизму перемещения данных BULKINSERT.

1. **Связанная служба источника** имеет тип: **AzureStorage** или **AzureDataLakeStore с проверкой подлинности субъекта-службы**.
2. Тип **входного набора данных** — **AzureBlob** или **AzureDataLakeStore**, тип формата в свойствах `type` — **OrcFormat**, **ParquetFormat** или **TextFormat** со следующими конфигурациями.

   1. Параметр `rowDelimiter` должен иметь значение **\n**.
   2. Параметр `nullValue` должен быть **пустой строке** (""), или параметру `treatEmptyAsNull` присваивается значение **true**.
   3. Параметру `encodingName` присваивается значение **utf-8**, которое является значением **по умолчанию**.
   4. Параметры `escapeChar`, `quoteChar`, `firstRowAsHeader` и `skipLineCount` не указываются.
   5. Параметр `compression` может иметь значение **no compression**, **GZip** или **Deflate**.

      ```JSON
      "typeProperties": {
       "folderPath": "<blobpath>",
       "format": {
           "type": "TextFormat",
           "columnDelimiter": "<any delimiter>",
           "rowDelimiter": "\n",
           "nullValue": "",
           "encodingName": "utf-8"
       },
       "compression": {
           "type": "GZip",
           "level": "Optimal"
       }
      },
      ```

3. В разделе **BlobSource** или **AzureDataLakeStore** для действия копирования в конвейере отсутствует параметр `skipHeaderLineCount`.
4. В разделе **SqlDWSink** для действия копирования в конвейере отсутствует параметр `sliceIdentifierColumnName`. PolyBase гарантирует, что за один цикл выполнения либо все данные будут обновлены, либо ничего не будет обновлено. Чтобы обеспечить **воспроизводимость**, используйте свойство `sqlWriterCleanupScript`).
5. В соответствующем действии копирования не используется `columnMapping`.

### <a name="staged-copy-using-polybase"></a>промежуточное копирование с помощью PolyBase
Если исходные данные не соответствуют критериям, приведенным в предыдущем разделе, можно включить копирование данных через промежуточное хранилище BLOB-объектов Azure (не может быть хранилищем класса Premium). В этом случае фабрика данных Azure автоматически выполняет преобразования данных в соответствии с требованиями Polybase к формату данных, а затем использует Polybase для загрузки данных в Azure синапсе Analytics и во время последней очистки временных данных из хранилища BLOB-объектов. Подробные сведения о том, как работает копирование данных через промежуточное хранилище BLOB-объектов Azure, см. в разделе [Промежуточное копирование](data-factory-copy-activity-performance.md#staged-copy).

> [!NOTE]
> При копировании данных из локального хранилища данных в Azure синапсе Analytics с помощью Polybase и промежуточного хранения, если версия Управление данными шлюза ниже 2,4, JRE (среда выполнения Java) требуется на компьютере шлюза, который используется для преобразования исходных данных в правильный формат. Чтобы избежать такой зависимости, мы рекомендуем обновить шлюз до последней версии.
>

Чтобы использовать эту функцию, создайте [связанную службу службы хранилища Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service), относящуюся к учетной записи хранения Azure, которая содержит это промежуточное хранилище BLOB-объектов. Затем укажите свойства `enableStaging` и `stagingSettings` для действия копирования, как показано в следующем коде:

```json
"activities":[
{
    "name": "Sample copy activity from SQL Server to Azure Synapse Analytics via PolyBase",
    "type": "Copy",
    "inputs": [{ "name": "OnpremisesSQLServerInput" }],
    "outputs": [{ "name": "AzureSQLDWOutput" }],
    "typeProperties": {
        "source": {
            "type": "SqlSource",
        },
        "sink": {
            "type": "SqlDwSink",
            "allowPolyBase": true
        },
        "enableStaging": true,
        "stagingSettings": {
            "linkedServiceName": "MyStagingBlob"
        }
    }
}
]
```

## <a name="best-practices-when-using-polybase"></a>Рекомендации по использованию PolyBase
В следующих разделах приводятся дополнительные рекомендации по тем, которые упоминаются в рекомендациях по работе с [Azure синапсе Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-best-practices.md).

### <a name="required-database-permission"></a>Необходимые разрешения базы данных
Для использования Polybase требуется, чтобы пользователь, который используется для загрузки данных в Azure синапсе Analytics, получил [разрешение CONTROL](/sql/relational-databases/security/permissions-database-engine) на целевую базу данных. Это разрешение можно получить, добавив этого пользователя как участника роли db_owner. Дополнительные сведения см. в [этом разделе](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-manage-security.md#authorization).

### <a name="row-size-and-data-type-limitation"></a>Ограничение размера строки и типа данных
Нагрузки PolyBase ограничиваются загрузкой строк меньше **1 МБ** и не позволяют загружать данные типа VARCHAR(MAX), NVARCHAR(MAX) или VARBINARY(MAX). Дополнительные сведения см. [здесь](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md#loads).

Если источник данных содержит строки больше 1 МБ, вы можете разделить исходные таблицы по вертикали на несколько небольших таблиц, для каждой из которых максимальный размер строки не будет превышать это ограничение. Затем можно загрузить небольшие таблицы с помощью Polybase и объединить их в Azure синапсе Analytics.

### <a name="azure-synapse-analytics-resource-class"></a>Класс ресурсов Azure синапсе Analytics
Для достижения наилучшей пропускной способности рекомендуется назначить пользователю более крупного класса ресурсов, который будет использоваться для загрузки данных в Azure синапсе Analytics через Polybase. Узнайте, как это сделать, ознакомившись с [примером изменения класса ресурсов пользователя](../../synapse-analytics/sql-data-warehouse/resource-classes-for-workload-management.md).

### <a name="tablename-in-azure-synapse-analytics"></a>tableName в Azure синапсе Analytics
В следующей таблице показаны правильные значения свойства **tableName** в наборе данных JSON для разных сочетаний имен таблиц и схем.

| Схема базы данных | Имя таблицы | Свойство tableName в JSON |
| --- | --- | --- |
| dbo |MyTable |MyTable или dbo.MyTable либо [dbo].[MyTable] |
| dbo1 |MyTable |dbo1.MyTable или [dbo1].[MyTable] |
| dbo |My.Table |[My.Table] или [dbo].[My.Table] |
| dbo1 |My.Table |[dbo1].[My.Table] |

Если вы увидите приведенное ниже сообщение об ошибке, это может указывать на неправильное значение свойства tableName. Правильные значения для свойства tableName в JSON см. в таблице выше.

```
Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider
```

### <a name="columns-with-default-values"></a>Столбцы со значениями по умолчанию
Сейчас для PolyBase в фабрике данных требуется такое же количество столбцов, как и в целевой таблице. Предположим, у вас есть таблица с четырьмя столбцами и один из них определен со значением по умолчанию. Входные данные по-прежнему должны содержать четыре столбца. Если входной набор данных будет содержать 3 столбца, мы получим ошибку, похожую на следующую.

```
All columns of the table must be specified in the INSERT BULK statement.
```
Значение NULL рассматривается как вариант значения по умолчанию. Если столбец допускает значения NULL, входные значения (в большом двоичном объекте) для этого столбца могут быть пустыми (но не могут отсутствовать во входном наборе данных). Polybase вставляет для них значение NULL в Azure синапсе Analytics.

## <a name="auto-table-creation"></a>Автоматическое создание таблицы
Если вы используете мастер копирования для копирования данных из SQL Server или базы данных SQL Azure в Azure синапсе Analytics, а таблица, соответствующая исходной таблице, не существует в целевом хранилище, фабрика данных может автоматически создать таблицу в хранилище данных с помощью схемы исходной таблицы.

Фабрика данных создает таблицу в целевом хранилище с тем же именем, что и у таблицы в исходном хранилище данных. Типы данных для столбцов выбираются на основании приведенного ниже сопоставления типов. При необходимости выполняется преобразование типов, чтобы устранить несовместимость между исходным и целевым хранилищами. Также используется распределение таблиц методом циклического перебора.

| Тип столбца исходной базы данных SQL | Тип столбца целевой аналитики Azure синапсе (ограничение размера) |
| --- | --- |
| Int | Int |
| BigInt | BigInt |
| SmallInt | SmallInt |
| TinyInt | TinyInt |
| bit | bit |
| Decimal | Decimal |
| Числовой | Decimal |
| Float | Float |
| Money | Money |
| Real | Real |
| SmallMoney | SmallMoney |
| Двоичные данные | Двоичные данные |
| Varbinary | VarBinary (до 8000) |
| Дата | Дата |
| Дата и время | Дата/время |
| datetime2 | datetime2 |
| время; | время; |
| DateTimeOffset | DateTimeOffset |
| SmallDateTime | SmallDateTime |
| Текст | Varchar (до 8000) |
| NText | NVarChar (до 4000) |
| Образ — | VarBinary (до 8000) |
| UniqueIdentifier | UniqueIdentifier |
| Char | Char |
| NCHAR | NCHAR |
| VarChar | Varchar (до 8000) |
| NVarChar | NVarChar (до 4000) |
| Xml | Varchar (до 8000) |

[!INCLUDE [data-factory-type-repeatability-for-sql-sources](../../../includes/data-factory-type-repeatability-for-sql-sources.md)]

## <a name="type-mapping-for-azure-synapse-analytics"></a>Сопоставление типов для Azure синапсе Analytics
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md), во время копирования типы источников автоматически преобразовываются в типы приемников. Такое преобразование выполняется в два этапа.

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

При перемещении данных в & из Azure синапсе Analytics следующие сопоставления используются из типа SQL в тип .NET и наоборот.

Сопоставление аналогично [SQL Serverному сопоставлению типов данных для ADO.NET](/dotnet/framework/data/adonet/sql-server-data-type-mappings).

| Тип ядра СУБД SQL Server | Тип платформы .NET Framework |
| --- | --- |
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
| sql_variant |Object * |
| текст |String, Char[] |
| time |TimeSpan |
| TIMESTAMP |Byte[] |
| tinyint |Byte |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| xml |Xml |

Также можно сопоставить столбцы из набора данных, используемого в качестве источника, со столбцами из приемника в определении действия копирования. Дополнительные сведения см. в статье [Сопоставление столбцов исходного набора данных со столбцами целевого набора данных](data-factory-map-columns.md).

## <a name="json-examples-for-copying-data-to-and-from-azure-synapse-analytics"></a>Примеры JSON для копирования данных в Azure синапсе Analytics и обратно
Следующие примеры содержат образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В них показано, как копировать данные в Azure синапсе Analytics и хранилище BLOB-объектов Azure и обратно. Тем не менее данные можно копировать **непосредственно** из любых источников в любой указанный [здесь](data-factory-data-movement-activities.md#supported-data-stores-and-formats) приемник. Это делается с помощью действия копирования в фабрике данных Azure.

### <a name="example-copy-data-from-azure-synapse-analytics-to-azure-blob"></a>Пример. копирование данных из Azure синапсе Analytics в BLOB-объект Azure
Образец определяет следующие сущности фабрики данных.

1. Связанная служба типа [AzureSqlDW](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [AzureSqlDWTable](#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Конвейер [pipeline](data-factory-create-pipelines.md) с действием копирования, в котором используются [SqlDWSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В этом примере данные временных рядов (ежечасные, ежедневные и т. д.) из таблицы в базе данных Azure синапсе Analytics копируются в большой двоичный объект каждый час. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба Azure синапсе Analytics:**

```JSON
{
  "name": "AzureSqlDWLinkedService",
  "properties": {
    "type": "AzureSqlDW",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Связанная служба хранилища BLOB-объектов Azure**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Входной набор данных Azure синапсе Analytics:**

В примере предполагается, что вы создали таблицу MyTable в Azure синапсе Analytics, которая содержит столбец с именем "timestampcolumn" для данных временных рядов.

Если параметру external присвоить значение true, то фабрика данных воспримет этот набор данных как внешний и созданный не в результате какого-либо действия в этой фабрике данных.

```JSON
{
  "name": "AzureSqlDWInput",
  "properties": {
    "type": "AzureSqlDWTable",
    "linkedServiceName": "AzureSqlDWLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
**Выходной набор данных большого двоичного объекта Azure:**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Действие копирования в конвейере с источником SqlDWSource и приемником BlobSink**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **SqlDWSource**, а для типа **sink** — значение **BlobSink**. SQL-запрос, указанный для свойства **SqlReaderQuery** , выбирает для копирования данные за последний час.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[
      {
        "name": "AzureSQLDWtoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureSqlDWInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlDWSource",
            "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
> [!NOTE]
> В приведенном примере для свойства SqlDWSource указано **sqlReaderQuery** . Действие копирования выполняет этот запрос к источнику Azure синапсе Analytics для получения данных.
>
> Кроме того, можно создать хранимую процедуру, указав **sqlReaderStoredProcedureName** и **storedProcedureParameters** (если хранимая процедура принимает параметры).
>
> Если не указать ни sqlReaderQuery, ни sqlReaderStoredProcedureName, то столбцы, определенные в разделе Structure JSON набора данных, будут использоваться для создания запроса (выберите Столбец1, Столбец2 из MyTable) для выполнения в Azure синапсе Analytics. Если у определения набора данных нет структуры, выбираются все столбцы из таблицы.
>
>

### <a name="example-copy-data-from-azure-blob-to-azure-synapse-analytics"></a>Пример. копирование данных из большого двоичного объекта Azure в Azure синапсе Analytics
Образец определяет следующие сущности фабрики данных.

1. Связанная служба типа [AzureSqlDW](#linked-service-properties).
2. Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Входной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. Выходной [набор данных](data-factory-create-datasets.md) типа [AzureSqlDWTable](#dataset-properties).
5. Конвейер [pipeline](data-factory-create-pipelines.md) с действием копирования, которое использует [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) и [SqlDWSink](#copy-activity-properties).

В этом примере данные временных рядов (ежечасно, ежедневно и т. д.) копируются из большого двоичного объекта Azure в таблицу в базе данных Azure синапсе Analytics каждый час. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба Azure синапсе Analytics:**

```JSON
{
  "name": "AzureSqlDWLinkedService",
  "properties": {
    "type": "AzureSqlDW",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Связанная служба хранилища BLOB-объектов Azure**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Входной набор данных большого двоичного объекта Azure:**

Данные берутся из нового BLOB-объекта каждый час (frequency: hour, interval: 1). Путь к папке с BLOB-объектом и имя файла вычисляются динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц и день начала, а в имени файла — час начала. параметр "External": "true" информирует службу фабрики данных о том, что эта таблица является внешней по отношению к фабрике данных и не создается действием в фабрике данных.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
**Выходной набор данных Azure синапсе Analytics:**

В примере данные копируются в таблицу с именем MyTable в Azure синапсе Analytics. Создайте таблицу в Azure синапсе Analytics с тем же количеством столбцов, которое будет содержать CSV-файл большого двоичного объекта. Новые строки добавляются в таблицу каждый час.

```JSON
{
  "name": "AzureSqlDWOutput",
  "properties": {
    "type": "AzureSqlDWTable",
    "linkedServiceName": "AzureSqlDWLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**Действие копирования в конвейере с источником BlobSource и приемником SqlDWSink**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **BlobSource**, а для типа **sink** — значение **SqlDWSink**.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoSQLDW",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlDWOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource",
            "blobColumnSeparators": ","
          },
          "sink": {
            "type": "SqlDWSink",
            "allowPolyBase": true
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
Пошаговое руководство см. в статье [Загрузка 1 ТБ в Azure синапсе Analytics в течение 15 минут с фабрикой данных Azure](data-factory-load-sql-data-warehouse.md) и [Загрузка данных с помощью фабрики данных](../load-azure-sql-data-warehouse.md) Azure в документации по Azure синапсе Analytics.

## <a name="performance-and-tuning"></a>Производительность и настройка
Ознакомьтесь со статьей [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в которой описываются ключевые факторы, влияющие на производительность перемещения данных (действие копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.
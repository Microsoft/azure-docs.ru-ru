---
title: Запрос Azure Cosmos DB данных с помощью бессерверного пула SQL в ссылке Azure синапсе
description: В этой статье вы узнаете, как запрашивать Azure Cosmos DB с помощью бессерверного пула SQL в ссылке Azure синапсе.
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 03/02/2021
ms.author: jovanpop
ms.reviewer: jrasnick
ms.custom: cosmos-db
ms.openlocfilehash: 10262b168b91370956c9559ba688c72213ba7618
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870999"
---
# <a name="query-azure-cosmos-db-data-with-a-serverless-sql-pool-in-azure-synapse-link"></a>Запрос Azure Cosmos DB данных с помощью несвязанного с сервером пула SQL в Azure синапсе Link

Серверный пул SQL позволяет анализировать данные в контейнерах Azure Cosmos DB, которые включены с помощью [ссылки Azure синапсе](../../cosmos-db/synapse-link.md) практически в режиме реального времени, не влияя на производительность транзакционных рабочих нагрузок. Он предлагает знакомый синтаксис T-SQL для запроса данных из [аналитического хранилища](../../cosmos-db/analytical-store-introduction.md) и интегрированного подключения к широкому спектру средств бизнес-АНАЛИТИКИ (BI) и специальных запросов с помощью интерфейса T-SQL.

Для запросов Azure Cosmos DB Полная контактная зона [выбора](/sql/t-sql/queries/select-transact-sql?view=azure-sqldw-latest&preserve-view=true) поддерживается с помощью функции [OPENROWSET](develop-openrowset.md) , которая включает большинство [функций и операторов SQL](overview-features.md). Кроме того, можно сохранять результаты запроса, считывающего данные из Azure Cosmos DB вместе с данными в хранилище BLOB-объектов Azure или Azure Data Lake Storage с помощью инструкции [CREATE External Table как SELECT](develop-tables-cetas.md#cetas-in-serverless-sql-pool) (CETAS). В настоящее время вы не можете хранить результаты запросов пула SQL Server, чтобы Azure Cosmos DB с помощью CETAS.

В этой статье вы узнаете, как написать запрос с несвязанным серверным пулом SQL, который будет запрашивать данные из Azure Cosmos DB контейнеров, которые включены с помощью ссылки Azure синапсе. Затем вы можете узнать больше о создании бессерверных представлений пула SQL в Azure Cosmos DB контейнерах и подключении их к Power BI моделям в [этом руководстве](./tutorial-data-analyst.md). В этом руководстве используется контейнер с [четко определенной схемой Azure Cosmos DB](../../cosmos-db/analytical-store-introduction.md#schema-representation).

## <a name="overview"></a>Обзор

Бессерверный пул SQL позволяет запрашивать Azure Cosmos DB аналитического хранилища с помощью `OPENROWSET` функции. 
- `OPENROWSET` со встроенным ключом. Этот синтаксис можно использовать для запроса коллекций Azure Cosmos DB без необходимости подготовки учетных данных.
- `OPENROWSET` указанные учетные данные, содержащие ключ учетной записи Cosmos DB. Этот синтаксис можно использовать для создания представлений в Azure Cosmos DB коллекциях.

### <a name="openrowset-with-key"></a>[OPENROWSET с ключом](#tab/openrowset-key)

Для поддержки запросов и анализа данных в Azure Cosmos DBном аналитическом хранилище серверный пул SQL использует следующий `OPENROWSET` синтаксис:

```sql
OPENROWSET( 
       'CosmosDB',
       '<Azure Cosmos DB connection string>',
       <Container name>
    )  [ < with clause > ] AS alias
```

Строка подключения Azure Cosmos DB указывает имя учетной записи Azure Cosmos DB, имя базы данных, главный ключ учетной записи базы данных и необязательное имя региона для `OPENROWSET` функции.

Строка подключения имеет следующий формат:
```sql
'account=<database account name>;database=<database name>;region=<region name>;key=<database account master key>'
```

Имя контейнера Azure Cosmos DB указывается без кавычек в `OPENROWSET` синтаксисе. Если имя контейнера содержит специальные символы, например тире (-), имя должно быть заключено в квадратные скобки ( `[]` ) в `OPENROWSET` синтаксисе.

### <a name="openrowset-with-credential"></a>[OPENROWSET с учетными данными](#tab/openrowset-credential)

Можно использовать `OPENROWSET` синтаксис, который ссылается на учетные данные:

```sql
OPENROWSET( 
       PROVIDER = 'CosmosDB',
       CONNECTION = '<Azure Cosmos DB connection string without account key>',
       OBJECT = '<Container name>',
       [ CREDENTIAL | SERVER_CREDENTIAL ] = '<credential name>'
    )  [ < with clause > ] AS alias
```

Строка подключения Azure Cosmos DB не содержит ключ в этом случае. Строка подключения имеет следующий формат:
```sql
'account=<database account name>;database=<database name>;region=<region name>'
```

Главный ключ учетной записи базы данных помещается в учетные данные уровня сервера или учетные данные области базы данных. 

---

> [!IMPORTANT]
> Убедитесь, что используются некоторые параметры сортировки базы данных UTF-8, например, `Latin1_General_100_CI_AS_SC_UTF8` поскольку строковые значения в Azure Cosmos DB аналитическом хранилище кодируются как текст в кодировке UTF-8.
> Несоответствие кодировки текста в файле и параметров сортировки может привести к непредвиденным ошибкам преобразования текста.
> Параметры сортировки по умолчанию для текущей базы данных можно легко изменить с помощью инструкции T-SQL `alter database current collate Latin1_General_100_CI_AI_SC_UTF8` .

> [!NOTE]
> Несерверный пул SQL не поддерживает запросы к Azure Cosmos DB хранилищу транзакций.

## <a name="sample-dataset"></a>Пример набора данных

Примеры в этой статье основаны на данных из [Европейского центра для предотвращения болезни и контроля (ЕКДК) ковид-19](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) и [ковид-19 Open Research DataSet (шнур-19), ДОИ: 10.5281/зенодо. 3715505](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/).

На этих страницах можно просмотреть лицензию и структуру данных. Вы также можете загрузить демонстрационные данные для наборов данных [ЕКДК](https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.json) и [кабель-19](https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json) .

Для работы с этой статьей, посвященной запросам Azure Cosmos DB данных с помощью несерверного пула SQL, убедитесь, что созданы следующие ресурсы:

* Azure Cosmos DB учетной записи базы данных, для которой [включена ссылка Azure синапсе](../../cosmos-db/configure-synapse-link.md).
* База данных Azure Cosmos DB с именем `covid` .
* Два Azure Cosmos DB контейнеров с именами `Ecdc` и `Cord19` загружены с предыдущими примерами наборов данных.

Для целей тестирования можно использовать следующую строку подключения: `Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==` . Обратите внимание, что это подключение не гарантирует производительность, так как эта учетная запись может находиться в удаленном регионе по сравнению с конечной точкой SQL синапсе.

## <a name="explore-azure-cosmos-db-data-with-automatic-schema-inference"></a>Просмотр Azure Cosmos DB данных с помощью автоматического вывода схемы

Самый простой способ исследовать данные в Azure Cosmos DB — использовать функцию автоматического вывода схемы. Пропустив `WITH` предложение из `OPENROWSET` инструкции, можно указать несерверному пулу SQL автоматическое обнаружение (выводить) схему аналитического хранилища контейнера Azure Cosmos DB.

### <a name="openrowset-with-key"></a>[OPENROWSET с ключом](#tab/openrowset-key)

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Ecdc) as documents
```

### <a name="openrowset-with-credential"></a>[OPENROWSET с учетными данными](#tab/openrowset-credential)

```sql
/*  Setup - create server-level or database scoped credential with Azure Cosmos DB account key:
    CREATE CREDENTIAL MyCosmosDbAccountCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
*/
SELECT TOP 10 *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

---

В предыдущем примере мы предзадали бессерверному пулу SQL подключаться к `covid` базе данных в учетной записи Azure Cosmos DB, `MyCosmosDbAccount` прошедшей проверку подлинности с помощью Azure Cosmos DB ключа (фиктивного в предыдущем примере). Затем мы обратились к `Ecdc` аналитическому хранилищу контейнера в `West US 2` регионе. Поскольку нет проекции конкретных свойств, `OPENROWSET` функция возвратит все свойства из Azure Cosmos DB элементов.

Предполагая, что элементы в контейнере Azure Cosmos DB имеют `date_rep` `cases` свойства, и, `geo_id` Результаты этого запроса показаны в следующей таблице.

| date_rep | cases | geo_id |
| --- | --- | --- |
| 2020-08-13 | 254 | Сервер отчетов |
| 2020-08-12 | 235 | Сервер отчетов |
| 2020-08-11 | 163 | Сервер отчетов |

Если необходимо исследовать данные из другого контейнера в той же Azure Cosmos DB базе данных, можно использовать одну и ту же строку подключения и ссылаться на требуемый контейнер в качестве третьего параметра:

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19) as cord19
```

## <a name="explicitly-specify-schema"></a>Явное указание схемы

Хотя функция автоматического вывода схемы в `OPENROWSET` предоставляет простой и простой в использовании куериенце, для бизнес-сценариев может потребоваться явно указать схему для получения релевантных свойств, предназначенных только для чтения, из Azure Cosmos DB данных.

`OPENROWSET`Функция позволяет явно указать свойства, которые требуется считать из данных в контейнере, и указать их типы данных.

Предположим, что мы импортировали некоторые данные из [набора данных ЕКДК ковид](https://azure.microsoft.com/services/open-datasets/catalog/ecdc-covid-19-cases/) со следующей структурой в Azure Cosmos DB:

```json
{"date_rep":"2020-08-13","cases":254,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-12","cases":235,"countries_and_territories":"Serbia","geo_id":"RS"}
{"date_rep":"2020-08-11","cases":163,"countries_and_territories":"Serbia","geo_id":"RS"}
```

Эти плоские документы JSON в Azure Cosmos DB могут быть представлены в виде набора строк и столбцов в синапсе SQL. `OPENROWSET`Функция позволяет указать подмножество свойств, которые необходимо считать, и точные типы столбцов в `WITH` предложении:

### <a name="openrowset-with-key"></a>[OPENROWSET с ключом](#tab/openrowset-key)
```sql
SELECT TOP 10 *
FROM OPENROWSET(
      'CosmosDB',
      'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Ecdc
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```
### <a name="openrowset-with-credential"></a>[OPENROWSET с учетными данными](#tab/openrowset-credential)
```sql
/*  Setup - create server-level or database scoped credential with Azure Cosmos DB account key:
    CREATE CREDENTIAL MyCosmosDbAccountCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
*/
SELECT TOP 10 *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```
---
Результат этого запроса может выглядеть, как в следующей таблице:

| date_rep | cases | geo_id |
| --- | --- | --- |
| 2020-08-13 | 254 | Сервер отчетов |
| 2020-08-12 | 235 | Сервер отчетов |
| 2020-08-11 | 163 | Сервер отчетов |

Дополнительные сведения о типах SQL, которые следует использовать для Azure Cosmos DB значений, см. в разделе [правила сопоставления типов SQL](#azure-cosmos-db-to-sql-type-mappings) в конце статьи.

## <a name="create-view"></a>Создать представление

Создание представлений в базах данных master или по умолчанию не рекомендуется или не поддерживается. Поэтому необходимо создать пользовательскую базу данных для представлений.

Определив схему, можно подготовить представление поверх данных Azure Cosmos DB. Ключ учетной записи Azure Cosmos DB следует размещать в отдельных учетных данных и ссылаться на эти учетные данные из `OPENROWSET` функции. Не оставляйте ключ учетной записи в определении представления.

```sql
CREATE CREDENTIAL MyCosmosDbAccountCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 's5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==';
GO
CREATE OR ALTER VIEW Ecdc
AS SELECT *
FROM OPENROWSET(
      PROVIDER = 'CosmosDB',
      CONNECTION = 'Account=synapselink-cosmosdb-sqlsample;Database=covid',
      OBJECT = 'Ecdc',
      SERVER_CREDENTIAL = 'MyCosmosDbAccountCredential'
    ) with ( date_rep varchar(20), cases bigint, geo_id varchar(6) ) as rows
```

Не используйте `OPENROWSET` без явно определенных схем, так как это может повлиять на производительность. Убедитесь, что используются наименьшие возможные размеры столбцов (например, VARCHAR (100) вместо VARCHAR (8000) по умолчанию. Необходимо использовать некоторые параметры сортировки UTF-8 в качестве параметров сортировки базы данных по умолчанию или задать их как явные параметры сортировки столбцов, чтобы избежать [проблем с преобразованием UTF-8](../troubleshoot/reading-utf8-text.md). Параметры сортировки `Latin1_General_100_BIN2_UTF8` обеспечивают лучшую производительность при Yu фильтрации данных с помощью некоторых строковых столбцов.

## <a name="query-nested-objects-and-arrays"></a>Запрос вложенных объектов и массивов

С помощью Azure Cosmos DB можно представить более сложные модели данных, составляя их как вложенные объекты или массивы. Функция автосинхронизации ссылки Azure синапсе для Azure Cosmos DB управляет представлением схемы в аналитическом хранилище, которое включает в себя обработку вложенных типов данных, позволяющих выполнять расширенные запросы из несерверного пула SQL.

Например, набор данных " [кабель-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) " содержит документы JSON, которые следуют этой структуре:

```json
{
    "paper_id": <str>,                   # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": <array of objects>    # list of author dicts, in order
        ...
     }
     ...
}
```

Вложенные объекты и массивы в Azure Cosmos DB представлены в результатах запроса в виде строк JSON, когда `OPENROWSET` функция считывает их. При использовании предложения можно указать пути к вложенным значениям в объектах `WITH` .

```sql
SELECT TOP 10 *
FROM OPENROWSET( 
       'CosmosDB',
       'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19)
WITH (  paper_id    varchar(8000),
        title        varchar(1000) '$.metadata.title',
        metadata     varchar(max),
        authors      varchar(max) '$.metadata.authors'
) AS docs;
```

Результат этого запроса может выглядеть, как в следующей таблице:

| paper_id | title | метаданные | authors |
| --- | --- | --- |
| bb11206963e831f... | Дополнительная информация — епидеми... | `{"title":"Supplementary Informati…` | `[{"first":"Julien","last":"Mélade","suffix":"","af…`| 
| bb1206963e831f1... | Использование Конвалесцент Сера в защищенной-E... | `{"title":"The Use of Convalescent…` | `[{"first":"Antonio","last":"Lavazza","suffix":"", …` |
| bb378eca9aac649... | Тилосема ескулентум (Марама) и B... | `{"title":"Tylosema esculentum (Ma…` | `[{"first":"Walter","last":"Chingwaru","suffix":"",…` | 

Узнайте больше о том, как анализировать [сложные типы данных в ссылках Azure синапсе](../how-to-analyze-complex-schema.md) и [вложенных структурах в бессерверном пуле SQL](query-parquet-nested-types.md).

> [!IMPORTANT]
> Если в тексте, например вместо, отображаются непредвиденные символы `MÃƒÂ©lade` `Mélade` , параметры сортировки базы данных не задаются в параметрах сортировки [UTF-8](/sql/relational-databases/collations/collation-and-unicode-support#utf8) .
> [Измените параметры сортировки базы данных](/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) на параметры сортировки UTF-8 с помощью инструкции SQL, такой как `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8` .

## <a name="flatten-nested-arrays"></a>Сведение вложенных массивов

Azure Cosmos DB данные могут иметь вложенные подмассивы, такие как массив автора, из набора данных " [шнур-19](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) ":

```json
{
    "paper_id": <str>,                      # 40-character sha1 of the PDF
    "metadata": {
        "title": <str>,
        "authors": [                        # list of author dicts, in order
            {
                "first": <str>,
                "middle": <list of str>,
                "last": <str>,
                "suffix": <str>,
                "affiliation": <dict>,
                "email": <str>
            },
            ...
        ],
        ...
}
```

В некоторых случаях может потребоваться "присоединить" к свойствам верхнего элемента (метаданных) со всеми элементами массива (авторы). Бессерверный пул SQL позволяет выполнить сведение вложенных структур, применяя `OPENJSON` функцию к вложенному массиву:

```sql
SELECT
    *
FROM
    OPENROWSET(
      'CosmosDB',
      'Account=synapselink-cosmosdb-sqlsample;Database=covid;Key=s5zarR2pT0JWH9k8roipnWxUYBegOuFGjJpSjGlR36y86cW0GQ6RaaG8kGjsRAQoWMw1QKTkkX8HQtFpJjC8Hg==',
       Cord19
    ) WITH ( title varchar(1000) '$.metadata.title',
             authors varchar(max) '$.metadata.authors' ) AS docs
      CROSS APPLY OPENJSON ( authors )
                  WITH (
                       first varchar(50),
                       last varchar(50),
                       affiliation nvarchar(max) as json
                  ) AS a
```

Результат этого запроса может выглядеть, как в следующей таблице:

| title | authors | first | last | принадлежность |
| --- | --- | --- | --- | --- |
| Дополнительная информация — епидеми... |   `[{"first":"Julien","last":"Mélade","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | Жюльен | мéладе | `   {"laboratory":"Centre de Recher…` |
Дополнительная информация — епидеми... | `[{"first":"Nicolas","last":"4#","suffix":"","affiliation":{"laboratory":"","institution":"U…` | николас | четырех # |`{"laboratory":"","institution":"U…` | 
| Дополнительная информация — епидеми... |   `[{"first":"Beza","last":"Ramazindrazana","suffix":"","affiliation":{"laboratory":"Centre de Recher…` | беза | рамазиндразана | `{"laboratory":"Centre de Recher…` |
| Дополнительная информация — епидеми... |   `[{"first":"Olivier","last":"Flores","suffix":"","affiliation":{"laboratory":"UMR C53 CIRAD, …` | оливиер | флорес |`{"laboratory":"UMR C53 CIRAD, …` |     

> [!IMPORTANT]
> Если в тексте, например вместо, отображаются непредвиденные символы `MÃƒÂ©lade` `Mélade` , параметры сортировки базы данных не задаются в параметрах сортировки [UTF-8](/sql/relational-databases/collations/collation-and-unicode-support#utf8) . [Измените параметры сортировки базы данных](/sql/relational-databases/collations/set-or-change-the-database-collation#to-change-the-database-collation) на параметры сортировки UTF-8 с помощью инструкции SQL, такой как `ALTER DATABASE MyLdw COLLATE LATIN1_GENERAL_100_CI_AS_SC_UTF8` .

## <a name="azure-cosmos-db-to-sql-type-mappings"></a>Сопоставления типов SQL Azure Cosmos DB

Хотя Azure Cosmos DB хранилище транзакций не зависит от схемы, аналитическое хранилище схематизированных для оптимизации производительности аналитических запросов. С помощью функции автосинхронизации ссылки Azure синапсе Azure Cosmos DB управляет представлением схемы в аналитическом хранилище из поля, которое включает обработку вложенных типов данных. Поскольку серверный пул SQL запрашивает аналитическое хранилище, важно понимать, как сопоставлять типы входных данных Azure Cosmos DB с типами данных SQL.

Учетные записи Azure Cosmos DB API SQL (Core) поддерживают типы свойств JSON чисел, строк, логических значений, null, вложенных объектов или массивов. Необходимо выбрать типы SQL, соответствующие этим типам JSON, если вы используете `WITH` предложение в `OPENROWSET` . В следующей таблице показаны типы столбцов SQL, которые следует использовать для различных типов свойств в Azure Cosmos DB.

| Тип свойства Azure Cosmos DB | Тип столбца SQL |
| --- | --- |
| Логическое значение | bit |
| Целое число | BIGINT |
| Decimal | FLOAT |
| Строка | varchar (параметры сортировки базы данных UTF-8) |
| Дата и время (строка в формате ISO) | varchar (30) |
| Дата и время (метка времени UNIX) | BIGINT |
| NULL | `any SQL type` 
| Вложенный объект или массив | varchar (max) (параметры сортировки базы данных UTF-8), сериализованный как текст JSON |

## <a name="full-fidelity-schema"></a>Схема полной точности

Azure Cosmos DB полная схема точности записывает как значения, так и их наиболее подходящие типы для каждого свойства в контейнере. `OPENROWSET`Функция в контейнере с полной схемой точности предоставляет как тип, так и фактическое значение в каждой ячейке. Предположим, что следующий запрос считывает элементы из контейнера с полной схемой точности:

```sql
SELECT *
FROM OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) as rows
```

Результат этого запроса будет возвращать типы и значения, отформатированные как текст JSON:

| date_rep | cases | geo_id |
| --- | --- | --- |
| {"Date": "2020-08-13"} | {"Int32": "254"} | {"строка": "RS"} |
| {"Date": "2020-08-12"} | {"Int32": "235"}| {"строка": "RS"} |
| {"Date": "2020-08-11"} | {"Int32": "316"} | {"строка": "RS"} |
| {"Date": "2020-08-10"} | {"Int32": "281"} | {"строка": "RS"} |
| {"Date": "2020-08-09"} | {"Int32": "295"} | {"строка": "RS"} |
| {"строка": "2020/08/08"} | {"Int32": "312"} | {"строка": "RS"} |
| {"Date": "2020-08-07"} | {"float64": "339.0"} | {"строка": "RS"} |

Для каждого значения можно увидеть тип, определенный в Azure Cosmos DB элементе контейнера. Большинство значений `date_rep` свойства содержат `date` значения, но некоторые из них неправильно хранятся в виде строк в Azure Cosmos DB. Схема с полной точностью вернет как правильно введенные `date` значения, так и неверно отформатированные `string` значения.
Количество вариантов — это информация, сохраненная в виде `int32` значения, но есть одно значение, которое указывается в виде десятичного числа. Это значение имеет `float64` тип. При наличии некоторых значений, превышающих максимальное `int32` число, они будут храниться как `int64` тип. Все `geo_id` значения в этом примере хранятся в виде `string` типов.

> [!IMPORTANT]
> `OPENROWSET`Функция без `WITH` предложения предоставляет оба значения с ожидаемыми типами и значениями с неправильно введенными типами. Эта функция предназначена для просмотра данных, а не для создания отчетов. Не следует анализировать значения JSON, возвращаемые этой функцией, для построения отчетов. Используйте явное [предложение WITH](#query-items-with-full-fidelity-schema) для создания отчетов. Необходимо очистить значения, имеющие неверные типы в Azure Cosmos DB контейнере, чтобы применить исправления в полном аналитическом хранилище.

Если вам нужно запросить Azure Cosmos DB учетных записей типа API Mongo DB, см. Дополнительные сведения о полном представлении схемы точности в аналитическом хранилище и именах расширенных свойств, которые будут использоваться в [Azure Cosmos DB аналитических данных](../../cosmos-db/analytical-store-introduction.md#analytical-schema).

### <a name="query-items-with-full-fidelity-schema"></a>Запрос элементов с полной схемой точности

При запросе полной схемы точности необходимо явно указать тип SQL и ожидаемый тип свойства Azure Cosmos DB в `WITH` предложении.

В следующем примере предполагается, что `string` является правильным типом `geo_id` Свойства и `int32` является правильным типом для `cases` Свойства:

```sql
SELECT geo_id, cases = SUM(cases)
FROM OPENROWSET(
      'CosmosDB'
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) WITH ( geo_id VARCHAR(50) '$.geo_id.string',
             cases INT '$.cases.int32'
    ) as rows
GROUP BY geo_id
```

Значения для `geo_id` и `cases` , имеющие другие типы, будут возвращены как `NULL` значения. Этот запрос будет ссылаться только на объект `cases` с указанным типом в выражении ( `cases.int32` ).

Если имеются значения с другими типами ( `cases.int64` , `cases.float64` ), которые не могут быть очищены в контейнере Azure Cosmos DB, необходимо явно ссылаться на них в `WITH` предложении и объединить результаты. Следующий запрос объединяет `int32` `int64` и, и `float64` хранится в `cases` столбце:

```sql
SELECT geo_id, cases = SUM(cases_int) + SUM(cases_bigint) + SUM(cases_float)
FROM OPENROWSET(
      'CosmosDB',
      'account=MyCosmosDbAccount;database=covid;region=westus2;key=C0Sm0sDbKey==',
       Ecdc
    ) WITH ( geo_id VARCHAR(50) '$.geo_id.string', 
             cases_int INT '$.cases.int32',
             cases_bigint BIGINT '$.cases.int64',
             cases_float FLOAT '$.cases.float64'
    ) as rows
GROUP BY geo_id
```

В этом примере количество вариантов сохраняется как `int32` `int64` значения, или `float64` . Чтобы вычислить количество вариантов по странам, необходимо извлечь все значения.

## <a name="known-issues"></a>Известные проблемы

- Серверный пул SQL возвратит предупреждение во время компиляции, если `OPENROWSET` Параметры сортировки столбца не имеют кодировки UTF-8. Можно легко изменить параметры сортировки по умолчанию для всех `OPENROWSET` функций, выполняемых в текущей базе данных, с помощью инструкции T-SQL `alter database current collate Latin1_General_100_CI_AS_SC_UTF8` .

Возможные ошибки и действия по устранению неполадок перечислены в следующей таблице.

| Ошибка | Первопричина |
| --- | --- |
| Синтаксические ошибки:<br/> — Неправильный синтаксис рядом с `Openrowset`<br/> - `...` не является распознаваемым `BULK OPENROWSET` параметром поставщика.<br/> — Неправильный синтаксис рядом с `...` | Возможные основные причины:<br/> — Не использовать CosmosDB в качестве первого параметра.<br/> — Использование строкового литерала вместо идентификатора в третьем параметре.<br/> — Не указывает третий параметр (имя контейнера). |
| Ошибка в строке подключения CosmosDB. | — Учетная запись, база данных или ключ не указаны. <br/> — Есть несколько параметров в строке подключения, которая не распознана.<br/> — Точка с запятой ( `;` ) помещается в конец строки соединения. |
| Сбой разрешения пути CosmosDB с ошибкой "неправильное имя учетной записи" или "неправильное имя базы данных". | Не удается найти указанное имя учетной записи, имя базы данных или контейнер, либо не было включено аналитическое хранилище для указанной коллекции.|
| Сбой разрешения пути CosmosDB с ошибкой "неверное значение секрета" или "секрет имеет значение null или является пустым". | Ключ учетной записи не является допустимым или отсутствует. |
| Столбец `column name` типа `type name` несовместим с внешним типом данных `type name` . | Указанный тип столбца в `WITH` предложении не соответствует типу в контейнере Azure Cosmos DB. Попробуйте изменить тип столбца, как описано в разделе [Azure Cosmos DB к сопоставлениям типов SQL](#azure-cosmos-db-to-sql-type-mappings), или используйте `VARCHAR` тип. |
| Столбец содержит `NULL` значения во всех ячейках. | Возможно, в предложении есть неправильное имя столбца или выражение пути `WITH` . Имя столбца (или выражение пути после типа столбца) в `WITH` предложении должно соответствовать имени некоторого свойства в коллекции Azure Cosmos DB. При сравнении учитывается *регистр*. Например, `productCode` и `ProductCode` являются разными свойствами. |

Вы можете сообщить о предложениях и проблемах на [странице отзывов о Azure синапсе Analytics](https://feedback.azure.com/forums/307516-azure-synapse-analytics?category_id=387862).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- [Использование Power BI и бессерверного пула SQL с ссылкой Azure синапсе](../../cosmos-db/synapse-link-power-bi.md)
- [Создание и использование представлений в бессерверном пуле SQL](create-use-views.md)
- [Руководство по созданию бессерверных представлений пула SQL с Azure Cosmos DB и их подключение к Power BIным моделям через DirectQuery](./tutorial-data-analyst.md)

---
title: Доступ к файлам хранилища в бессерверном пуле SQL
description: В этой статье описывается, как отправлять запросы к файлам хранилища с помощью бессерверного пула SQL в Azure Synapse Analytics.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/19/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 679df881e928825496a03b2fef7c9374d61593f7
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103199989"
---
# <a name="access-external-storage-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Доступ к внешнему хранилищу с помощью бессерверного пула SQL в Azure Synapse Analytics

В этой статье описывается, как пользователи могут считывать данные из файлов, хранящихся в службе хранилища Azure в бессерверном пуле SQL. Пользователи могут обращаться к хранилищу с помощью следующих средств:

- [OPENROWSET](develop-openrowset.md) — функция, которая позволяет выполнять специальные запросы к файлам в службе хранилища Azure.
- [Внешняя таблица](develop-tables-external-tables.md) — предопределенная структура данных, созданная на основе набора внешних файлов.

Пользователь может использовать [разные методы аутентификации](develop-storage-files-storage-access-control.md), такие как сквозная аутентификация Azure AD (по умолчанию для участников Azure AD) и аутентификация SAS (по умолчанию для участников SQL).

## <a name="query-files-using-openrowset"></a>Запрашивание файлов с помощью OPENROWSET

С помощью функции OPENROWSET пользователи могут запрашивать внешние файлы в службе хранилища Azure, если у них есть доступ к хранилищу. Пользователь, подключенный к бессерверному пулу SQL, должен использовать следующий запрос для чтения содержимого файлов в службе хранилища Azure:

```sql
SELECT * FROM
 OPENROWSET(BULK 'https://<storage_account>.dfs.core.windows.net/<container>/<path>/*.parquet', format= 'parquet') as rows
```

Пользователь может получить доступ к хранилищу на основе следующих правил доступа:

- Пользователь Azure AD — `OPENROWSET` будет использовать удостоверение Azure AD вызывающей стороны для обращения к службе хранилища Azure или хранилищу с анонимным доступом.
- Пользователь SQL — `OPENROWSET` будет обращаться к хранилищу с помощью анонимного доступа или выполнять олицетворение с помощью маркера SAS или управляемого удостоверения рабочей области.

### <a name="impersonation"></a>[Олицетворение](#tab/impersonation)

Субъекты SQL также могут использовать OPENROWSET для непосредственного запрашивания файлов, защищенных маркерами SAS или управляемым удостоверением рабочей области. Если пользователь SQL выполняет эту функцию, опытный пользователь с разрешением `ALTER ANY CREDENTIAL` должен создать учетные данные уровня сервера, соответствующие URL-адресу в функции (с использованием имени хранилища и контейнера) и предоставить разрешения REFERENCES для этих учетных данных вызывающему объекту функции OPENROWSET.

```sql
EXECUTE AS somepoweruser

CREATE CREDENTIAL [https://<storage_account>.dfs.core.windows.net/<container>]
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 'sas token';

GRANT REFERENCES ON CREDENTIAL::[https://<storage_account>.dfs.core.windows.net/<container>] TO sqluser
```

Если учетные данные уровня сервера, которые соответствуют URL-адресу, отсутствуют или у пользователя SQL нет разрешения REFERENCES для этих учетных данных, будет возвращена ошибка. Субъекты SQL не поддерживают олицетворение с помощью некоторых удостоверений Azure AD.

### <a name="direct-access"></a>[Прямой доступ](#tab/direct-access)

Дополнительная настройка для предоставления пользователям Azure AD доступа к файлам с помощью удостоверений не требуется.
Получить доступ к службе хранилища Azure, разрешающей анонимный доступ, может любой пользователь (дополнительная настройка не требуется).

---

> [!NOTE]
> Эта версия OPENROWSET предназначена для быстрого и удобного просмотра данных с использованием аутентификации по умолчанию. Чтобы работать с олицетворением или управляемым удостоверением, используйте OPENROWSET с DATASOURCE, как описано в следующем разделе.

## <a name="query-data-sources-using-openrowset"></a>Обращение к источникам данных с помощью OPENROWSET

С помощью функции OPENROWSET пользователь может запрашивать файлы, размещенные во внешнем источнике данных:

```sql
SELECT * FROM
 OPENROWSET(BULK 'file/path/*.parquet',
 DATASOURCE = MyAzureInvoices,
 FORMAT= 'parquet') as rows
```

У пользователя, выполняющего этот запрос, должен быть доступ к файлам. Если пользователи не могут напрямую обращаться к файлам с помощью [удостоверений Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) или [анонимного доступа](develop-storage-files-storage-access-control.md?tabs=public-access), их нужно олицетворить с помощью [маркера SAS](develop-storage-files-storage-access-control.md?tabs=shared-access-signature) или [управляемого удостоверения рабочей области](develop-storage-files-storage-access-control.md?tabs=managed-identity).

### <a name="impersonation"></a>[Олицетворение](#tab/impersonation)

`DATABASE SCOPED CREDENTIAL` определяют способ доступа к файлам в указанном источнике данных (сейчас это SAS и управляемое удостоверение). Опытным пользователям с разрешением `CONTROL DATABASE` нужно создать `DATABASE SCOPED CREDENTIAL` для обращения к хранилищу и `EXTERNAL DATA SOURCE` с определением используемых URL-адреса источника данных и учетных данных:

```sql
EXECUTE AS somepoweruser;

-- Create MASTER KEY if it doesn't exists in database
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'some very strong password';

CREATE DATABASE SCOPED CREDENTIAL AccessAzureInvoices
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
 SECRET = '******srt=sco&amp;sp=rwac&amp;se=2017-02-01T00:55:34Z&amp;st=201********' ;

CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>/' ,
 CREDENTIAL = AccessAzureInvoices) ;
```

Вызывающий объект должен иметь одно из следующих разрешений для выполнения функции OPENROWSET:

- Одно из разрешений на выполнение функции OPENROWSET:
  - `ADMINISTER BULK OPERATIONS` позволяет выполнить вход для выполнения функции OPENROWSET.
  - `ADMINISTER DATABASE BULK OPERATIONS` позволяет пользователю базы данных выполнить функцию OPENROWSET.
- `REFERENCES DATABASE SCOPED CREDENTIAL` для учетных данных, на которые указывает ссылка в `EXTERNAL DATA SOURCE`.

### <a name="direct-access"></a>[Прямой доступ](#tab/direct-access)

Пользователь может создать внешний источник данных без учетных данных со ссылкой на хранилище с открытым доступом ИЛИ использовать сквозную аутентификацию Azure AD.

```sql
CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>') ;
```
---
## <a name="external-table"></a>EXTERNAL TABLE

Пользователь с разрешениями на чтение таблицы может обращаться к внешним файлам с помощью внешней таблицы (EXTERNAL TABLE), созданной на основе набора папок и файлов службы хранилища Azure.

Пользователь с [разрешениями на создание внешней таблицы](/sql/t-sql/statements/create-external-table-transact-sql?preserve-view=true&view=sql-server-ver15#permissions) (например, CREATE TABLE и ALTER ANY CREDENTIAL или REFERENCES DATABASE SCOPED CREDENTIAL), может использовать следующий скрипт для создания таблицы на основе источника данных службы хранилища Azure:

```sql
CREATE EXTERNAL TABLE [dbo].[DimProductexternal]
( ProductKey int, ProductLabel nvarchar, ProductName nvarchar )
WITH
(
LOCATION='/DimProduct/year=*/month=*' ,
DATA_SOURCE = AzureDataLakeStore ,
FILE_FORMAT = TextFileFormat
) ;
```

Пользователь, считывающий данные из этой таблицы, должен иметь доступ к файлам. Если пользователи не могут напрямую обращаться к файлам с помощью [удостоверений Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) или [анонимного доступа](develop-storage-files-storage-access-control.md?tabs=public-access), их нужно олицетворить с помощью [маркера SAS](develop-storage-files-storage-access-control.md?tabs=shared-access-signature) или [управляемого удостоверения рабочей области](develop-storage-files-storage-access-control.md?tabs=managed-identity).

### <a name="impersonation"></a>[Олицетворение](#tab/impersonation)

Учетные данные уровня базы данных определяют способ доступа к файлам в указанном источнике данных. Пользователям с разрешением CONTROL DATABASE нужно создать учетные данные уровня базы данных (DATABASE SCOPED CREDENTIAL) для обращения к хранилищу и внешнему источнику данных (EXTERNAL DATA SOURCE) с определением используемых URL-адреса источника данных и учетных данных:

```sql
EXECUTE AS somepoweruser;

-- Create MASTER KEY if it doesn't exists in database
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'some very strong password';

CREATE DATABASE SCOPED CREDENTIAL cred
 WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
 SECRET = '******srt=sco&sp=rwac&se=2017-02-01T00:55:34Z&st=201********' ;

CREATE EXTERNAL DATA SOURCE AzureDataLakeStore
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>' ,
 CREDENTIAL = cred
 ) ;
```

### <a name="direct-access"></a>[Прямой доступ](#tab/direct-access)

Пользователь может создать внешний источник данных без учетных данных со ссылкой на хранилище с открытым доступом ИЛИ использовать сквозную аутентификацию Azure AD.

```sql
CREATE EXTERNAL DATA SOURCE MyAzureInvoices
 WITH ( LOCATION = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>') ;
```
---

### <a name="read-external-files-with-external-table"></a>Считывание внешних файлов с помощью EXTERNAL TABLE

Структура EXTERNAL TABLE позволяет считывать данные из файлов, на которые указывает источник данных, с помощью стандартной инструкции SQL SELECT:

```sql
SELECT *
FROM dbo.DimProductsExternal
```

Для чтения данных вызывающий объект должен иметь следующие разрешения:
- Разрешение `SELECT` ДЛЯ внешней таблицы.
- Разрешение `REFERENCES DATABASE SCOPED CREDENTIAL`, если у `DATA SOURCE` есть `CREDENTIAL`.

## <a name="permissions"></a>Разрешения

В следующей таблице перечислены необходимые разрешения для перечисленных выше операций.

| Запрос | Необходимые разрешения|
| --- | --- |
| OPENROWSET(BULK) без источника данных | `ADMINISTER BULK OPERATIONS`, `ADMINISTER DATABASE BULK OPERATIONS` или имя для входа SQL должны содержать REFERENCES CREDENTIAL::\<URL> для хранилища, защищенного с помощью SAS |
| OPENROWSET(BULK) с источником данных без учетных данных | `ADMINISTER BULK OPERATIONS` или `ADMINISTER DATABASE BULK OPERATIONS` |
| OPENROWSET(BULK) с источником данных с учетными данными | `REFERENCES DATABASE SCOPED CREDENTIAL` и либо `ADMINISTER BULK OPERATIONS`, либо `ADMINISTER DATABASE BULK OPERATIONS` |
| CREATE EXTERNAL DATA SOURCE | `ALTER ANY EXTERNAL DATA SOURCE` и `REFERENCES DATABASE SCOPED CREDENTIAL` |
| CREATE EXTERNAL TABLE | `CREATE TABLE`, `ALTER ANY SCHEMA`, `ALTER ANY EXTERNAL FILE FORMAT` и `ALTER ANY EXTERNAL DATA SOURCE` |
| SELECT FROM EXTERNAL TABLE | `SELECT TABLE` и `REFERENCES DATABASE SCOPED CREDENTIAL` |
| CETAS | Для создания таблиц: `CREATE TABLE`, `ALTER ANY SCHEMA`, `ALTER ANY DATA SOURCE` и `ALTER ANY EXTERNAL FILE FORMAT`; для чтения данных: `ADMINISTER BULK OPERATIONS`, `REFERENCES CREDENTIAL` или `SELECT TABLE` для каждых таблицы, представления или функции в запросе, а также разрешение на чтение и запись в хранилище. |

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы готовы для работы со следующими руководствами:

- [Запрашивание данных в хранилище](query-data-storage.md)

- [Запрашивание CSV-файлов](query-single-csv-file.md)

- [Запрашивание файлов Parquet](query-parquet-files.md)

- [Запрашивание JSON-файлов](query-json-files.md)

- [Запрашивание папок и нескольких файлов](query-folders-multiple-csv-files.md)

- [Использование метаданных файла в запросах](query-specific-files.md)

- [Запрашивание вложенных типов](query-parquet-nested-types.md)

- [Создание и использование представлений](create-use-views.md)

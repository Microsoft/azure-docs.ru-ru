---
title: 'Устранение неполадок: чтение текста UTF-8 из файлов CSV или PARQUET с помощью бессерверного пула SQL'
description: Чтение текста UTF-8 из файлов CSV или PARQUET с помощью бессерверного пула SQL в Azure синапсе Analytics
author: julieMSFT
ms.author: jrasnick
ms.topic: troubleshooting
ms.service: synapse-analytics
ms.subservice: sql
ms.date: 12/03/2020
ms.openlocfilehash: 70ce3c82790db0296d5359b5db2e6a323306c309
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96576423"
---
# <a name="troubleshoot-reading-utf-8-text-from-csv-or-parquet-files-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Устранение неполадок при чтении текста UTF-8 из файлов CSV или Parquet с помощью бессерверного пула SQL в Azure синапсе Analytics

В этой статье содержатся инструкции по устранению неполадок при чтении текста UTF-8 из файлов CSV или Parquet с помощью бессерверного пула SQL в Azure синапсе Analytics.

При чтении текста UTF-8 из файла CSV или PARQUET с помощью бессерверного пула SQL некоторые специальные символы, такие как u и ц, неверно преобразуются, если запрос возвращает столбцы типа VARCHAR с параметрами сортировки, отличными от UTF8. Это известная проблема в SQL Server и Azure SQL. Параметры сортировки, отличные от UTF8, используются по умолчанию в синапсе SQL, поэтому запросы клиентов будут затронуты. Клиенты, использующие стандартные английские символы и некоторые подмножества символов национальных алфавитов, могут не заметить ошибки преобразования. Неверное преобразование более подробно описано в [статье Использование параметров сортировки UTF-8 для чтения текста UTF-8 в бессерверном пуле SQL](https://techcommunity.microsoft.com/t5/azure-synapse-analytics/always-use-utf-8-collations-to-read-utf-8-text-in-serverless-sql/ba-p/1883633) .

## <a name="workaround"></a>Обходной путь

Чтобы решить эту проблему, всегда используйте параметры сортировки UTF-8 при чтении текста UTF-8 из файлов CSV или PARQUET.

- Во многих случаях необходимо просто задать параметры сортировки UTF8 в базе данных (операция с метаданными).

   ```sql
   alter database MyDB
         COLLATE Latin1_General_100_BIN2_UTF8;
   ```

- Можно явно определить параметры сортировки для столбца VARCHAR в OPENROWSET или внешней таблице:

   ```sql
   select geo_id, cases = sum(cases)
   from openrowset(
           bulk 'latest/ecdc_cases.parquet', data_source = 'covid', format = 'parquet'
       ) with ( cases int,
                geo_id VARCHAR(6) COLLATE Latin1_General_100_BIN2_UTF8 ) as rows
   group by geo_id
   ```
 
- Если параметры сортировки UTF8 не заданы для внешних таблиц, считывающих данные UTF8, необходимо повторно создать затронутые внешние таблицы и задать параметры сортировки UTF8 для столбцов VARCHAR (операция с метаданными).


## <a name="next-steps"></a>Дальнейшие действия

* [Запрос файлов Parquet с помощью синапсе SQL](../sql/query-parquet-files.md)
* [Запрос CSV-файлов с помощью синапсе SQL](../sql/query-single-csv-file.md)
* [CETAS в SQL Synapse](../sql/develop-tables-cetas.md)
* [Краткое руководство. Использование бессерверного пула SQL](../quickstart-sql-on-demand.md)

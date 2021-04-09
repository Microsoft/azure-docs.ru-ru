---
title: Самопомощь при работе с бессерверным пулом SQL
description: В этом разделе содержатся сведения, которые могут помочь в устранении проблем с бессерверным пулом SQL.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: f94134b2a06155b1b1f390175578e501a840038b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101669419"
---
# <a name="self-help-for-serverless-sql-pool"></a>Самопомощь при использовании бессерверного пула SQL

Эта статья содержит информацию об устранении наиболее частых проблем с бессерверным пулом SQL в Azure Synapse Analytics.

## <a name="serverless-sql-pool-is-grayed-out-in-synapse-studio"></a>Бессерверный пул SQL неактивен в Synapse Studio

Если в Synapse Studio не удается установить подключение к бессерверному пулу SQL, вы заметите, что бессерверный пул SQL неактивен или отображается статус "Отключено". Как правило, эта проблема возникает в одном из случаев, указанных ниже.

1) Ваша сеть не позволяет связаться с серверной частью Azure Synapse. Чаще всего оказывается, что заблокирован порт 1443. Чтобы включить бессерверный пул SQL, разблокируйте этот порт. Дополнительные сведения о других проблемах, которые могут помешать правильной работе пула SQL, см. в [полном руководстве по устранению неполадок](../troubleshoot/troubleshoot-synapse-studio.md).
2) У вас нет разрешений на вход в бессерверный пул SQL. Чтобы получить доступ, один из администраторов рабочей области Azure Synapse должен добавить вас в роль администратора рабочей области или администратора SQL. [Дополнительные сведения см. в полном руководстве по контролю доступа](../security/synapse-workspace-access-control-overview.md).

## <a name="query-fails-because-file-cannot-be-opened"></a>Сбой запроса из-за невозможности открыть файл

Если происходит сбой запроса с сообщением File cannot be opened because it does not exist or it is used by another process (Не удается открыть файл, так как он не существует или используется другим процессом) и вы уверены, что оба файла существуют и не используются другим процессом, это означает, что бессерверный пул SQL не может получить доступ к файлу. Эта проблема обычно возникает из-за того, что у удостоверения Azure Active Directory нет прав доступа к файлу. По умолчанию бессерверный пул SQL пытается получить доступ к файлу, используя удостоверение Azure Active Directory. Чтобы устранить эту проблему, необходимо получить соответствующие права доступа к файлу. Проще всего предоставить себе роль "Участник для данных BLOB-объектов хранилища" в учетной записи хранения, к которой вы пытаетесь выполнить запрос. 
- [Дополнительные сведения см. в полном руководстве по контролю доступа Azure Active Directory к хранилищу](../../storage/common/storage-auth-aad-rbac-portal.md). 
- См. статью [Управление доступом к учетной записи хранения в бессерверном пуле SQL в Azure Synapse Analytics](develop-storage-files-storage-access-control.md).

## <a name="query-fails-because-it-cannot-be-executed-due-to-current-resource-constraints"></a>Сбой выполнения запроса из-за текущих ограничений ресурсов 

Если сбой запроса завершается сообщением This query cannot be executed due to current resource constraints (Этот запрос нельзя выполнить из-за текущих ограничений ресурсов), это означает, что бессерверный пул SQL не может выполнить его в этот момент из-за ограничений, применяемых к ресурсам. 

- Убедитесь, что используются типы данных допустимого размера. Кроме того, укажите схему для файлов Parquet для столбцов строк, так как по умолчанию у них будет тип VARCHAR(8000). 

- Если запрос предназначен для CSV-файлов, [создайте статистику](develop-tables-statistics.md#statistics-in-serverless-sql-pool). 

- Ознакомьтесь с [рекомендациями по повышению производительности для бессерверного пула SQL](best-practices-sql-on-demand.md), чтобы оптимизировать создание запросов.  

## <a name="create-statement-is-not-supported-in-master-database"></a>Инструкция CREATE STATEMENT не поддерживается в базе данных master

Если запрос завершается сообщением об ошибке:

> "Не удалось выполнить запрос. Ошибка: СОЗДАНИЕ ВНЕШЕНЕЙ ТАБЛИЦЫ/ИСТОЧНИКА ДАННЫХ/УЧЕТНЫХ ДАННЫХ ДЛЯ БАЗЫ ДАННЫХ/ФОРМАТА ФАЙЛА не поддерживается в базе данных master." 

Это означает, что база данных master в бессерверном пуле SQL не поддерживает создание таких ресурсов:
  - Внешние таблицы
  - Внешние источники данных
  - Учетные данные для базы данных
  - Форматы внешних файлов

Решение.

  1. Создание пользовательской базы данных:

```sql
CREATE DATABASE <DATABASE_NAME>
```

  2. Выполните инструкцию CREATE в контексте <DATABASE_NAME>, работа которого ранее завершилась сбоем для базы данных master. 
  
  Пример создания формата внешнего файла:
    
```sql
USE <DATABASE_NAME>
CREATE EXTERNAL FILE FORMAT [SynapseParquetFormat] 
WITH ( FORMAT_TYPE = PARQUET)
```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь со следующими статьями, чтобы узнать больше об использовании бессерверного пула SQL:

- [Запрашивание одного CSV-файла](query-single-csv-file.md)

- [Запрашивание папок и нескольких CSV-файлов](query-folders-multiple-csv-files.md)

- [Запрашивание конкретных файлов](query-specific-files.md)

- [Запрашивание файлов Parquet](query-parquet-files.md)

- [Запрашивание вложенных типов Parquet](query-parquet-nested-types.md)

- [Запрашивание JSON-файлов](query-json-files.md)

- [Создание и использование представлений](create-use-views.md)

- [Создание и использование внешних таблиц](create-use-external-tables.md)

- [Сохранение результатов запроса в хранилище](create-external-table-as-select.md)

---
title: Подключение и запрос — гибкие серверные PostgreSQL
description: Ссылки на краткие руководства, в которых показано, как подключиться к базе данных Azure для гибкого сервера PostgreSQL и выполнить запросы.
services: postgresql
ms.service: postgresql
ms.topic: how-to
author: mksuni
ms.author: sumuth
ms.date: 12/08/2020
ms.openlocfilehash: ee3b1f7db8bdafb1233b32579e032e8c864c37a9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97364584"
---
# <a name="connect-and-query-overview-for-azure-database-for-postgresql--flexible-server"></a>Общие сведения о подключении и запросах для базы данных Azure для PostgreSQL — гибкого сервера

В следующем документе приведены ссылки на примеры, демонстрирующие подключение и выполнение запросов к серверу базы данных Azure для PostgreSQL. В этом руководством также содержатся рекомендации и расширения TLS, которые можно использовать для подключения к серверу на поддерживаемых языках.

>[!IMPORTANT]
> База данных Azure для гибкого сервера PostgreSQL доступна в **предварительной версии**.

## <a name="quickstarts"></a>Краткие руководства

| Краткое руководство | Описание |
|---|---|
|[PgAdmin](https://www.pgadmin.org/)|PgAdmin можно использовать для подключения к серверу, что упрощает создание, Обслуживание и использование объектов базы данных.|
|[psql в Azure Cloud Shell](./quickstart-create-server-cli.md#connect-using-postgresql-command-line-client)|В этой статье показано, как запустить [**psql**](https://www.postgresql.org/docs/current/static/app-psql.html) в [Azure Cloud Shell](../../cloud-shell/overview.md) для подключения к серверу, а затем выполнить инструкции для запроса, вставки, обновления и удаления данных в базе данных. Вы можете запустить **psql** , если он установлен в среде разработки.|
|[Python](connect-python.md)|В этом кратком руководстве показано, как использовать Python для подключения к базе данных и использования работы с объектами базы данных для запроса данных. |
|[Django со службой приложений](tutorial-django-app-service-postgres.md)|В этом руководстве показано, как использовать Ruby для создания программы для подключения к базе данных и использования работы с объектами базы данных для запроса данных.|

## <a name="tls-considerations-for-database-connectivity"></a>Рекомендации по использованию протокола TLS для подключения к базе данных

Протокол TLS используется всеми драйверами, предоставляемыми или поддерживаемыми корпорацией Майкрософт для подключения к базам данных Azure для PostgreSQL. Специальная настройка не требуется, но необходимо применить TLS 1,2 для вновь созданных серверов. Мы рекомендуем использовать TLS 1,0 и 1,1, а затем обновить версию TLS для серверов. См. раздел [Настройка TLS](how-to-connect-tls-ssl.md) .

## <a name="postgresql-extensions"></a>Расширения PostgreSQL

PostgreSQL предоставляет возможность расширить функциональность базы данных с помощью расширений. Расширения позволяют объединить несколько связанных объектов SQL в один пакет, чтобы загружать и удалять их из базы данных одной командой. После загрузки в базу данных расширения действуют как встроенные функции.

- [Расширения postgres 12](./concepts-extensions.md#postgres-12-extensions)
- [Расширения postgres 11](./concepts-extensions.md#postgres-11-extensions)
- [дблинк и postgres_fdw](./concepts-extensions.md#dblink-and-postgres_fdw)
- [pg_prewarm](./concepts-extensions.md#pg_prewarm)
- [pg_stat_statements](./concepts-extensions.md#pg_stat_statements)

Дополнительные сведения см. [в разделе Использование расширений PostgreSQL на гибком сервере](concepts-extensions.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Перенос данных с помощью дампа и восстановления](../howto-migrate-using-dump-and-restore.md)
- [Перенос данных с помощью импорта и экспорта](../howto-migrate-using-export-and-import.md)

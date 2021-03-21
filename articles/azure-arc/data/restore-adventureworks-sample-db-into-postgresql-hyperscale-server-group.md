---
title: Импорт образца базы данных AdventureWorks в службу Arc Azure с включенной PostgreSQL
description: Восстановление образца базы данных AdventureWorks в службе "Дуга Azure" с поддержкой PostgreSQL Scale
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: a9efa17fb782d5a913493907b66973272e4e0356
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102441794"
---
# <a name="import-the-adventureworks-sample-database-to-azure-arc-enabled-postgresql-hyperscale"></a>Импорт образца базы данных AdventureWorks в службу Arc Azure с включенной PostgreSQL

[AdventureWorks](/sql/samples/adventureworks-install-configure) — это образец базы данных, содержащей базу данных OLTP, которая используется в учебниках, и примеры. Он предоставляется и обслуживается корпорацией Майкрософт в составе [репозитория SQL Server Samples GitHub](https://github.com/microsoft/sql-server-samples/tree/master/samples/databases).

Проект с открытым исходным кодом конвертирует базу данных AdventureWorks, чтобы обеспечить совместимость с PostgreSQLом масштабирования Azure с поддержкой ARC.
- [Исходный проект](https://github.com/lorint/AdventureWorks-for-Postgres)
- [Подпишитесь на проект, который предварительно преобразует CSV-файлы для совместимости с PostgreSQL](https://github.com/NorfolkDataSci/adventure-works-postgres)

В этом документе описывается простой процесс получения образца базы данных AdventureWorks, импортированного в группу серверов PostgreSQL.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="download-the-adventureworks-backup-file"></a>Скачивание файла резервной копии AdventureWorks

Скачайте файл AdventureWorks. SQL в контейнер группы серверов PostgreSQL. В этом примере мы будем использовать `kubectl exec` команду для удаленного выполнения команды в контейнере "PostgreSQL" группы серверов, чтобы скачать файл в контейнер. Этот файл можно скачать из любого расположения, доступного в `curl` . Этот же метод следует использовать при наличии других файлов резервных копий базы данных, которые необходимо извлечь в контейнер PostgreSQL. После того как он находится в контейнере "PostgreSQL" группы серверов, можно легко создать базу данных, схему и заполнить данные.

Выполните команду, подобную следующей, чтобы загрузить файлы, заменяя значения имени Pod и имени пространства имен перед запуском:

> [!NOTE]
>  Чтобы скачать файл из GitHub, вашему контейнеру потребуется подключение к Интернету через 443.

> [!NOTE]
>  Используйте имя Pod узла координатора группы серверов postgres Scale. Его имя — <server group name> c-0 (например, postgres01c-0, где c означает узел координатора).  Если вы не знаете имя Pod, выполните команду `kubectl get pod`

```console
kubectl exec <PostgreSQL pod name> -n <namespace name> -c postgres  -- /bin/bash -c "cd /tmp && curl -k -O https://raw.githubusercontent.com/microsoft/azure_arc/main/azure_arc_data_jumpstart/aks/arm_template/postgres_hs/AdventureWorks.sql"

#Example:
#kubectl exec postgres02-0 -n arc -c postgres -- /bin/bash -c "cd /tmp && curl -k -O https://raw.githubusercontent.com/microsoft/azure_arc/main/azure_arc_data_jumpstart/aks/arm_template/postgres_hs/AdventureWorks.sql"
```

## <a name="step-2-import-the-adventureworks-database"></a>Шаг 2. Импорт базы данных AdventureWorks

Аналогичным образом можно выполнить команду kubectl Exec, чтобы использовать средство CLI PSQL, включенное в контейнеры PostgreSQL масштабирования серверов, чтобы создать и загрузить базу данных.

Выполните команду, подобную следующей, чтобы создать пустую базу данных, сначала подставив значение имени Pod и имени пространства имен, прежде чем запускать его.

```console
kubectl exec <PostgreSQL pod name> -n <namespace name> -c postgres -- psql --username postgres -c 'CREATE DATABASE "adventureworks";'

#Example
#kubectl exec postgres02-0 -n arc -c postgres -- psql --username postgres -c 'CREATE DATABASE "adventureworks";'
```

Затем выполните команду, подобную следующей, чтобы импортировать базу данных, заменив значения имени Pod и имени пространства имен, прежде чем запускать его.

```console
kubectl exec <PostgreSQL pod name> -n <namespace name> -c postgres -- psql --username postgres -d adventureworks -f /tmp/AdventureWorks.sql

#Example
#kubectl exec postgres02-0 -n arc -c postgres -- psql --username postgres -d adventureworks -f /tmp/AdventureWorks.sql
```


> **Примечание. Вы не увидите настолько значительных преимуществ в производительности, пока не PostgreSQL в службе Arc Azure, пока не будет выполнено масштабирование, и вы будете сегментированы и распределены данные и таблицы между рабочими узлами PostgreSQLной группы серверов. См. раздел [предложенные дальнейшие действия](#suggested-next-steps).**

## <a name="suggested-next-steps"></a>Предлагаемые дальнейшие действия
- Ознакомьтесь с основными понятиями и руководствами по использованию службы "база данных Azure для PostgreSQL", чтобы распределить данные между несколькими узлами PostgreSQL, а также получить преимущества от всех возможностей службы "база данных Azure для PostgreSQL". :
    * [Узлы и таблицы](../../postgresql/concepts-hyperscale-nodes.md)
    * [Определение типа приложения](../../postgresql/concepts-hyperscale-app-type.md)
    * [Выбор столбца распределения](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Совместное размещение таблиц](../../postgresql/concepts-hyperscale-colocation.md)
    * [Распространение и изменение таблиц](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Проектирование базы данных с несколькими клиентами](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Разработка панели мониторинга для аналитики в режиме реального времени](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

   > \* В приведенных выше документах пропустите разделы **Вход в портал Azure**& **создать базу данных Azure для PostgreSQL-Scale (Цитус)**. Реализуйте оставшиеся шаги в развертывании Azure ARC. Эти разделы относятся к службе "база данных Azure для PostgreSQL" (Цитус), предлагаемой в качестве службы PaaS в облаке Azure, но другие части документов напрямую применимы к вашей геомасштабированию Azure с включенной PostgreSQL.

- [Масштабирование группы серверов с Гипермасштабированием Базы данных Azure для PostgreSQL](scale-out-postgresql-hyperscale-server-group.md)

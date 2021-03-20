---
title: Задания эластичной базы данных (предварительная версия)
description: Настройка заданий обработки эластичных баз данных (Предварительная версия) для выполнения скриптов Transact-SQL (T-SQL) в наборе из одной или нескольких баз данных в базе данных SQL Azure
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: srinia
ms.author: srinia
ms.reviewer: sstein
ms.date: 12/18/2018
ms.openlocfilehash: f9a026ed47d662b80ef01e505bfbcf8f32d20b04
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92792180"
---
# <a name="create-configure-and-manage-elastic-jobs-preview"></a>Создание, Настройка и управление заданиями обработки эластичных БД (Предварительная версия)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этой статье вы узнаете, как создавать, настраивать задания обработки эластичных баз данных и управлять ими.

Если вы не использовали задания обработки эластичных баз данных, изучите статью [Automate management tasks using database jobs](job-automation-overview.md) (Автоматизация управления задачами с помощью заданий базы данных).

## <a name="create-and-configure-the-agent"></a>Создание и настройка агента

1. Создание или указание пустой базы данных S0 или выше. Она будет использоваться в качестве *базы данных задания* при создании агента задания обработки эластичных баз данных.
2. Создайте агент заданий обработки эластичных баз данных на [портале](https://portal.azure.com/#create/Microsoft.SQLElasticJobAgent) или с помощью [PowerShell](elastic-jobs-powershell-create.md#create-the-elastic-job-agent).

   ![Создание агента задания обработки эластичных баз данных](./media/elastic-jobs-overview/create-elastic-job-agent.png)

## <a name="create-run-and-manage-jobs"></a>Создание, запуск заданий и управление ими

1. Создание учетных данных для выполнения заданий в *базе данных заданий* с помощью [PowerShell](elastic-jobs-powershell-create.md) или [T-SQL](elastic-jobs-tsql-create-manage.md#create-a-credential-for-job-execution).
2. Определите целевую группу (базы данных, для которой требуется выполнить задание) с помощью [PowerShell](elastic-jobs-powershell-create.md) или [T-SQL](elastic-jobs-tsql-create-manage.md#create-a-target-group-servers).
3. Создайте учетные данные агента заданий в каждой базе данных, где будет запущено задание [(добавьте пользователя (или роль) в каждую базу данных в группе)](logins-create-manage.md). Пример см. в [руководстве по PowerShell](elastic-jobs-powershell-create.md).
4. Создайте задание с помощью [PowerShell](elastic-jobs-powershell-create.md) или [T-SQL](elastic-jobs-tsql-create-manage.md#deploy-new-schema-to-many-databases).
5. Добавьте шаги задания с помощью [PowerShell](elastic-jobs-powershell-create.md) или [T-SQL](elastic-jobs-tsql-create-manage.md#deploy-new-schema-to-many-databases).
6. Запустите задание с помощью [PowerShell](elastic-jobs-powershell-create.md#run-the-job) или [T-SQL](elastic-jobs-tsql-create-manage.md#begin-unplanned-execution-of-a-job).
7. Отслеживайте состояние выполнения заданий с помощью портала, [PowerShell](elastic-jobs-powershell-create.md#monitor-status-of-job-executions) или [T-SQL](elastic-jobs-tsql-create-manage.md#monitor-job-execution-status).

   ![Портал](./media/elastic-jobs-overview/elastic-job-executions-overview.png)

## <a name="credentials-for-running-jobs"></a>Учетные данные для выполнения заданий

Задания используют [учетные данные для базы данных](/sql/t-sql/statements/create-database-scoped-credential-transact-sql), чтобы подключаться к базам данных, указанным целевой группой при выполнении. Если целевая группа содержит серверы или пулы, эти учетные данные с привязкой к базе данных используются для подключения к базе данных master для перечисления доступных баз данных.

Настройка правильных учетных данных для запуска задания может быть немного запутанной, поэтому учитывайте следующие моменты:

- Учетные данные области базы данных должны быть созданы в *базе данных заданий*.
- **Все целевые базы данных должны иметь вход с [достаточными разрешениями](/sql/relational-databases/security/permissions-database-engine) для успешного завершения задания** (`jobuser` на диаграмме ниже).
- Учетные данные будут повторно использоваться в заданиях, а пароли учетных данных будут зашифрованы и защищены от пользователей, имеющих доступ только для чтения объектов заданий.

Рассмотрев следующее изображение, вы сможете ознакомиться с правильными учетными данными и настроить их. **Не забудьте создать пользователя в каждой базе данных (все *целевые базы данных пользователя*), где задание должно быть запущено**.

![Учетные данные заданий обработки эластичных баз данных](./media/elastic-jobs-overview/job-credentials.png)

## <a name="security-best-practices"></a>Лучшие методики обеспечения безопасности

Несколько рекомендаций по оптимальной работе с заданиями обработки эластичных баз данных

- Предоставляйте доступ к API-интерфейсам только доверенным лицам.
- Учетные данные должны иметь только те права, которые необходимы для выполнения шагов задания. Дополнительные сведения см. в разделе [авторизация и разрешения](/dotnet/framework/data/adonet/sql/authorization-and-permissions-in-sql-server).
- При использовании члена целевой группы сервера или пула настоятельно рекомендуется создать отдельные учетные данные с правами в базе данных master для просмотра или перечисления баз данных, используемых для расширения списков серверов или пулов до выполнения задания.

## <a name="agent-performance-capacity-and-limitations"></a>Производительность, емкость и ограничения агента

В заданиях обработки эластичных баз данных используются минимальные вычислительные ресурсы при ожидании завершения длительных заданий.

В зависимости от размера целевой группы баз данных и требуемого времени выполнения задания (количество одновременных рабочих ролей) агенту требуется разные объемы вычислений и производительности *базы данных заданий* (чем больше целевых объектов и заданий, тем выше требуемый объем вычислений).

Эта предварительная версия ограничена 100 одновременными заданиями.

### <a name="prevent-jobs-from-reducing-target-database-performance"></a>Предотвращение сокращения производительности заданий целевой базы данных

Чтобы ресурсы не перегружались при работе с базами данных в эластичном пуле SQL, задания можно настроить для ограничения количества баз данных, в которых может одновременно работать задание.

Задайте количество одновременных баз данных, на которых выполняется задание, задав `sp_add_jobstep` параметр хранимой процедуры `@max_parallelism` в T-SQL или `Add-AzSqlElasticJobStep -MaxParallelism` в PowerShell.

## <a name="best-practices-for-creating-jobs"></a>Рекомендации по созданию заданий

### <a name="idempotent-scripts"></a>Идемпотентные сценарии
Скрипты задания T-SQL должны быть [идемпотентными](https://en.wikipedia.org/wiki/Idempotence). **Идемпотентный** означает, что, если сценарий выполнился успешно и запустился снова, его результат останется неизменным. Сценарий может завершиться ошибкой из-за временных проблем в сети. В этом случае задание будет автоматически пытаться выполнить сценарий указанное количество раз, после чего попытки будут прекращены. Идемпотентный сценарий дает неизменный результат даже после двух успешных запусков (или более).

Прежде чем создавать объект, рекомендуется сначала проверить, не существует ли он уже.


```sql
IF NOT EXISTS (some_object)
    -- Create the object
    -- If it exists, drop the object before recreating it.
```

Аналогичным образом сценарий должен успешно завершаться, выполняя логическую проверку на существование различных условий и их обработку.



## <a name="next-steps"></a>Дальнейшие действия

- [Создание заданий обработки эластичных баз данных и управление ими с помощью PowerShell](elastic-jobs-powershell-create.md)
- [Use Transact-SQL (T-SQL) to create and manage Elastic Database Jobs](elastic-jobs-tsql-create-manage.md) (Создание заданий обработки эластичных БД и управление ими с использованием Transact-SQL (T-SQL))
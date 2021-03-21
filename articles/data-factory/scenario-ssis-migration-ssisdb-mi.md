---
title: Миграция служб SSIS с помощью Управляемый экземпляр Azure SQL в качестве назначения рабочей нагрузки базы данных
description: Миграция служб SSIS с помощью Управляемый экземпляр Azure SQL в качестве назначения рабочей нагрузки базы данных.
author: chugugrace
ms.author: chugu
ms.service: data-factory
ms.topic: conceptual
ms.date: 9/12/2019
ms.openlocfilehash: 3d2bc60f8ba7120f8d962500c06be50e905c11a5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373595"
---
# <a name="ssis-migration-with-azure-sql-managed-instance-as-the-database-workload-destination"></a>Миграция служб SSIS с помощью Управляемый экземпляр Azure SQL в качестве назначения рабочей нагрузки базы данных

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

При переносе рабочих нагрузок базы данных из SQL Server экземпляра в Управляемый экземпляр SQL Azure необходимо ознакомиться со [службой миграции данных Azure](../dms/dms-overview.md)(DMS) и [сетевыми топологиями для миграции SQL управляемый экземпляр с помощью DMS](../dms/resource-network-topologies.md).

Эта статья посвящена миграции пакетов служб интеграции SQL Server (SSIS), хранящихся в каталоге SSIS (SSISDB), и агент SQL Server заданий, планирующих выполнение пакетов служб SSIS.

## <a name="migrate-ssis-catalog-ssisdb"></a>Перенос каталога служб SSIS (SSISDB)

Миграцию SSISDB можно выполнить с помощью DMS, как описано в статье [Миграция пакетов служб SSIS в SQL управляемый экземпляр](../dms/how-to-migrate-ssis-packages-managed-instance.md).

## <a name="ssis-jobs-to-sql-managed-instance-agent"></a>Задания служб SSIS для агента SQL Управляемый экземпляр

SQL Управляемый экземпляр имеет собственный планировщик первого класса, так же как агент SQL Server локально.  [Пакеты служб SSIS можно запускать с помощью агента Azure SQL управляемый экземпляр](how-to-invoke-ssis-package-managed-instance-agent.md).

Так как средство миграции для заданий служб SSIS пока недоступно, их необходимо перенести из локальной агент SQL Server в агент SQL Управляемый экземпляр с помощью сценариев или вручную.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Фабрика данных Azure](./introduction.md).
- [Среда выполнения интеграции Azure SSIS](./create-azure-ssis-integration-runtime.md)
- [Azure Database Migration Service](../dms/dms-overview.md)
- [Сетевые топологии для миграции SQL Управляемый экземпляр с помощью DMS](../dms/resource-network-topologies.md)
- [Миграция пакетов служб SSIS в Управляемый экземпляр SQL](../dms/how-to-migrate-ssis-packages-managed-instance.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Подключение к каталогу SSIS (SSISDB) в Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [Запуск пакетов служб SSIS, развернутых в Azure](/sql/integration-services/lift-shift/ssis-azure-run-packages)
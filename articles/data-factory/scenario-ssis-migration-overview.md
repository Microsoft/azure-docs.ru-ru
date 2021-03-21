---
title: Миграция локальных рабочих нагрузок SQL Server Integration Services (SSIS) в службы SSIS в фабрике данных Azure (ADF)
description: Перенос локальных рабочих нагрузок служб SSIS в службы SSIS в ADF.
author: chugugrace
ms.author: chugu
ms.service: data-factory
ms.topic: conceptual
ms.date: 9/3/2019
ms.openlocfilehash: 78c488302a874319f79a143e4657d161fe849855
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373612"
---
# <a name="migrate-on-premises-ssis-workloads-to-ssis-in-adf"></a>Миграция локальных рабочих нагрузок служб SSIS в службы SSIS в ADF

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

## <a name="overview"></a>Обзор

При переносе рабочих нагрузок базы данных из локальной SQL Server в службы баз данных Azure, а именно в базу данных SQL Azure или Azure SQL Управляемый экземпляр, ваши рабочие нагрузки ETL на SQL Server Integration Services (SSIS) должны также быть перенесены.

Azure-SSIS Integration Runtime (IR) в фабрике данных Azure (ADF) поддерживает выполнение пакетов служб SSIS. После подготовки Azure-SSIS IR можно использовать привычные средства, такие как SQL Server Data Tools (SSDT)/SQL Server Management Studio (SSMS), и служебные программы командной строки, такие как дтинсталл/dtutil/dtexec, для развертывания и запуска пакетов в Azure. Дополнительные сведения см. в разделе [Общие сведения о точности и сдвиге Azure SSIS](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview).

В этой статье описывается процесс переноса рабочих нагрузок ETL из локальных служб SSIS в службы SSIS в ADF. Процесс миграции состоит из двух этапов: **оценка** и **миграция**.

## <a name="assessment"></a>Оценка

Для создания полного плана миграции можно определить проблемы с исходными пакетами служб SSIS, которые препятствуют успешной миграции.

Помощник по миграции данных (DMA) представляет собой бесплатное скачиваемое средство для этой цели, которое можно установить и запустить локально. Проект оценки DMA типа **Integration Services** можно создать для оценки пакетов служб SSIS в пакетах и определения проблем совместимости, представленных в следующих категориях.

- Блокирование миграции: проблемы совместимости, которые блокируют запуск пакетов источника миграции на Azure-SSIS IR. DMA предоставляет рекомендации по решению этих проблем.

- Информативные проблемы: частично поддерживаемые или устаревшие функции, используемые в исходных пакетах. DMA предоставляет полный набор рекомендаций, альтернативных подходов, доступных в Azure, и устраняет действия по устранению проблемы.

### <a name="four-storage-types-for-ssis-packages"></a>Четыре типа хранилища для пакетов служб SSIS

- Каталог служб SSIS (SSISDB). Появилось в SQL Server 2012 и содержит набор хранимых процедур, представлений и функций, возвращающих табличное значение, используемых для работы с проектами и пакетами служб SSIS.
- Файловая система.
- SQL Server системная база данных (MSDB).
- Хранилище пакетов служб SSIS. Слой управления пакетами поверх двух подтипов:
  - MSDB — это системная база данных в SQL Server, используемая для хранения пакетов служб SSIS.
  - Управляемая файловая система, которая является определенной папкой в SQL Server пути установки, используемого для хранения пакетов служб SSIS.

Сейчас DMA поддерживает пакетную оценку пакетов, хранящихся в **файловой системе**, **хранилище пакетов** и **каталоге служб SSIS** , с момента **DMA версии 5.0**.

Получите [DMA](/sql/dma/dma-overview)и [выполните оценку пакета с его помощью](/sql/dma/dma-assess-ssis).

## <a name="migration"></a>Миграция

В зависимости от [типов хранилища](#four-storage-types-for-ssis-packages) исходных пакетов SSIS и назначения миграции рабочих нагрузок базы данных действия по переносу  **пакетов служб SSIS** и **Агент SQL Server заданий** , планирующих выполнение пакетов служб SSIS, могут отличаться. Имеется два сценария.

- [**Управляемый экземпляр SQL Azure** в качестве назначения рабочей нагрузки базы данных](#azure-sql-managed-instance-as-database-workload-destination)
- [**База данных SQL Azure** в качестве назначения рабочей нагрузки базы данных](#azure-sql-database-as-database-workload-destination)

Это также практичный способ использования [средств DevOps служб SSIS](/sql/integration-services/devops/ssis-devops-overview)для повторного развертывания пакетного пакета в назначении миграции.  

### <a name="azure-sql-managed-instance-as-database-workload-destination"></a>**Управляемый экземпляр SQL Azure** в качестве назначения рабочей нагрузки базы данных

| **Тип хранилища пакета** |Пакетная миграция пакетов служб SSIS|Пакетная миграция заданий служб SSIS|
|-|-|-|
|SSISDB|[Перенос **SSISDB**](scenario-ssis-migration-ssisdb-mi.md)|<li>[Миграция заданий служб SSIS в агент Azure SQL Управляемый экземпляр](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li>Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Файловая система|Повторно разверните их в файловые ресурсы или файлы Azure через дтинсталл, dtutil или ручную копию или для доступа к файловым системам через виртуальную сеть или локальную среду IR. Дополнительные сведения см. в разделе [программа dtutil](/sql/integration-services/dtutil-utility).|<li>[Миграция заданий служб SSIS в агент Azure SQL Управляемый экземпляр](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li> Миграция с помощью [мастера миграции заданий служб SSIS в SSMS](how-to-migrate-ssis-job-ssms.md) <li>Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|SQL Server (MSDB)|Экспортируйте их в файловые системы, файловые ресурсы или файлы Azure с помощью SSMS или dtutil. Дополнительные сведения см. в разделе [Экспорт пакетов служб SSIS](/sql/integration-services/service/package-management-ssis-service#import-and-export-packages).|Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Хранилище пакетов|Экспортируйте их в хранилище пакетов с помощью SSMS/dtutil или повторно разверните их в хранилище пакетов с помощью дтинсталл/dtutil или копии вручную. Дополнительные сведения см. в статье [Управление пакетами с помощью Azure-SSIS Integration Runtime хранилища пакетов](azure-ssis-integration-runtime-package-store.md).|<li>[Миграция заданий служб SSIS в агент Azure SQL Управляемый экземпляр](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li> Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|

### <a name="azure-sql-database-as-database-workload-destination"></a>**База данных SQL Azure** в качестве назначения рабочей нагрузки базы данных

| **Тип хранилища пакета** |Пакетная миграция пакетов служб SSIS|Как выполнить пакетную миграцию заданий|
|-|-|-|
|SSISDB|Повторное развертывание в Azure-SSISDB с помощью SSDT/SSMS. Дополнительные сведения см. [в разделе Развертывание пакетов служб SSIS в Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial).|Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Файловая система|Повторно разверните их в файловые ресурсы или файлы Azure через дтинсталл, dtutil или ручную копию или для доступа к файловым системам через виртуальную сеть или локальную среду IR. Дополнительные сведения см. в разделе [программа dtutil](/sql/integration-services/dtutil-utility).|<li> Миграция с помощью [мастера миграции заданий служб SSIS в SSMS](how-to-migrate-ssis-job-ssms.md) <li> Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|SQL Server (MSDB)|Экспортируйте их в файловые системы, файловые ресурсы или файлы Azure с помощью SSMS или dtutil. Дополнительные сведения см. в разделе [Экспорт пакетов служб SSIS](/sql/integration-services/service/package-management-ssis-service#import-and-export-packages).|Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Хранилище пакетов|Экспортируйте их в файловые системы, файловые ресурсы или файлы Azure с помощью SSMS/dtutil или повторно разверните их в файловые ресурсы или файлы Azure с помощью дтинсталл/dtutil/вручную и не задерживайте их в файловых системах, чтобы получить доступ через виртуальную сеть или локальную среду IR. Дополнительные сведения см. в разделе Программа dtutil. Дополнительные сведения см. в разделе [программа dtutil](/sql/integration-services/dtutil-utility).|Преобразуйте их в конвейеры и действия/триггеры ADF с помощью сценариев, портала SSMS или ADF. Дополнительные сведения см. в разделе [функция планирования SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Фабрика данных Azure](./introduction.md).
- [Помощник по переносу баз данных](/sql/dma/dma-overview)
- [Перемещение рабочих нагрузок служб SSIS в облако](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview)
- [Инструменты DevOps Integration Services](/sql/integration-services/devops/ssis-devops-overview)
- [Перенос пакетов Integration Services в Управляемый экземпляр SQL Azure](../dms/how-to-migrate-ssis-packages-managed-instance.md)
- [Повторное развертывание пакетов в базе данных SQL Azure](../dms/how-to-migrate-ssis-packages.md)

- [Доступ к локальным данным из Azure-SSIS Integration Runtime](https://techcommunity.microsoft.com/t5/sql-server-integration-services/vnet-or-no-vnet-secure-data-access-from-ssis-in-azure-data/ba-p/1062056)
- [Настройка среды выполнения интеграции Azure Integration Services](how-to-configure-azure-ssis-ir-custom-setup.md)
- [Access data stores and file shares with Windows authentication from SSIS packages in Azure](ssis-azure-connect-with-windows-auth.md) (Доступ к хранилищам данных и общим папкам с помощью проверки подлинности Windows из пакетов служб SSIS в Azure)
- [Использовать аутентификацию управляемого удостоверения](/sql/integration-services/connection-manager/azure-storage-connection-manager#managed-identities-for-azure-resources-authentication)
- [Использование хранилища ключей Azure](store-credentials-in-key-vault.md)
- [Настройка среды выполнения интеграции Azure-SSIS для высокой производительности](configure-azure-ssis-integration-runtime-performance.md);
- [Запуск и остановка Azure-SSIS Integration Runtime по расписанию](how-to-schedule-azure-ssis-integration-runtime.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Проверка пакетов SSIS, развертываемых в Azure](/sql/integration-services/lift-shift/ssis-azure-validate-packages)
- [Запуск пакетов служб SSIS, развернутых в Azure](/sql/integration-services/lift-shift/ssis-azure-run-packages)
- [Мониторинг Azure-SSIS Integration Runtime](./monitor-integration-runtime.md#azure-ssis-integration-runtime)
- [Планирование выполнения пакетов служб SSIS в Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages)
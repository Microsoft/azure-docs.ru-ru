---
title: Состояние сценария миграции базы данных
titleSuffix: Azure Database Migration Service
description: Сведения о состоянии сценариев миграции, поддерживаемых Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: troubleshooting
ms.date: 07/08/2020
ms.openlocfilehash: d51de526205e8ef663dcfd682ec5858e4bc2f645
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102183987"
---
# <a name="status-of-migration-scenarios-supported-by-azure-database-migration-service"></a>Состояние сценариев миграции, поддерживаемых Azure Database Migration Service

Azure Database Migration Service предназначен для поддержки различных сценариев миграции (пар «источник-назначение») как для автономных (одноразовых), так и для оперативных (непрерывной синхронизации) миграций. Покрытие сценария, обеспечиваемое Azure Database Migration Service, расширяется со временем. Регулярно добавляются новые сценарии. В этой статье описываются сценарии миграции, которые в настоящее время поддерживаются Azure Database Migration Service и состояние (частный предварительный просмотр, общедоступная Предварительная версия или общая доступность) для каждого сценария.

## <a name="offline-versus-online-migrations"></a>Автономная миграция и миграция с подключением к сети

С помощью Azure Database Migration Service можно выполнить автономную или оперативную миграцию. При использовании *автономной* миграции простой приложения начинается в то же время, что и миграция. Чтобы ограничить время простоя до времени, необходимого для перехода в новую среду после завершения миграции, используйте *оперативную* миграцию. Рекомендуется протестировать автономную миграцию, чтобы определить, приемлем ли простой. Если нет, выполните оперативную миграцию.

## <a name="migration-scenario-status"></a>Состояние сценария миграции

Состояние сценариев миграции, поддерживаемых Azure Database Migration Service, зависит от времени. Как правило, сценарии впервые выпускаются в **закрытой предварительной версии**. Для участия в закрытой предварительной версии клиенты должны отправить предварительного утверждения через [сайт DMS Preview](https://aka.ms/dms-preview). После закрытой предварительной версии состояние сценария изменится на **общедоступная Предварительная версия**. Azure Database Migration Service пользователи могут испытать сценарии миграции в общедоступной предварительной версии непосредственно из пользовательского интерфейса. Регистрация не требуется.  Однако сценарии миграции в общедоступной предварительной версии могут быть недоступны во всех регионах и могут быть подвергнуты дополнительным изменениям перед окончательным выпуском. После выхода общедоступной предварительной версии состояние сценария изменится на **обычное доступность**. Общедоступная версия — это окончательное состояние выпуска, и эта функциональность завершена и доступна всем пользователям.

## <a name="migration-scenario-support"></a>Поддержка сценария миграции

В следующих таблицах показано, какие сценарии миграции поддерживаются при использовании Azure Database Migration Service.

> [!NOTE]
> Если сценарий, указанный ниже, не отображается в пользовательском интерфейсе, обратитесь к нему за дополнительными сведениями к псевдониму " [запрос на миграцию базы данных Azure](mailto:AskAzureDatabaseMigrations@service.microsoft.com) ".

> [!IMPORTANT]
> Чтобы просмотреть все сценарии, которые в настоящее время поддерживаются Azure Database Migration Service в закрытой предварительной версии, см. [сайт DMS Preview](https://aka.ms/dms-preview).

### <a name="offline-one-time-migration-support"></a>Поддержка автономной (однократной) миграции

В следующей таблице показана поддержка автономных миграций в службе Azure Database Migration Service.

| целевого объекта  | Источник | Поддержка | Состояние |
| ------------- | ------------- |:-------------:|:-------------:|
| **БД SQL Azure** | SQL Server | ✔ | GA |
|   | RDS SQL | X |  |
|   | Oracle; | X |  |
| **Управляемый экземпляр Базы данных SQL Azure** | SQL Server | ✔ | GA |
|   | RDS SQL | X |  |
|   | Oracle; | X |   |
| **Виртуальная машина Azure SQL** | SQL Server | ✔ | GA |
|   | Oracle; | X |   |
| **Azure Cosmos DB** | MongoDB | ✔ | GA |
| **База данных Azure для MySQL** | MySQL | X |   |
|   | RDS MySQL | X |   |
| **База данных Azure для PostgreSQL — один сервер** | PostgreSQL | X |
|  | RDS PostgreSQL | X |   |
| **База данных Azure для PostgreSQL — масштабирование (Цитус)** | PostgreSQL | X |
|  | RDS PostgreSQL | X |   |

### <a name="online-continuous-sync-migration-support"></a>Поддержка миграции с подключением к сети (непрерывная синхронизация)

В следующей таблице показана поддержка миграций с подключением к сети в службе Azure Database Migration Service.

| целевого объекта  | Источник | Поддержка | Состояние |
| ------------- | ------------- |:-------------:|:-------------:|
| **БД SQL Azure** | SQL Server | X | GA |
|   | RDS SQL | X | GA |
|   | Oracle; | X |  |
| **Управляемый экземпляр Базы данных SQL Azure** | SQL Server | ✔ | GA |
|   | RDS SQL | X | GA |
|   | Oracle; | X |  |
| **Виртуальная машина Azure SQL** | SQL Server | X |   |
|   | Oracle;  | X |  |
| **Azure Cosmos DB** | MongoDB | ✔ | GA |
| **База данных Azure для MySQL** | MySQL | ✔ | GA |
|   | RDS MySQL | ✔ | GA |
| **База данных Azure для PostgreSQL — один сервер** | PostgreSQL | ✔ | GA |
|   | База данных Azure для PostgreSQL — один сервер | ✔ | GA |
|   | RDS PostgreSQL | ✔ | GA |
|   | Oracle; | ✔ | Общедоступная предварительная версия |
| **База данных Azure для PostgreSQL — масштабирование (Цитус)** | PostgreSQL | ✔ | GA |
|   | RDS PostgreSQL | ✔ | GA |


## <a name="next-steps"></a>Дальнейшие действия

Общие сведения о доступности Azure Database Migration Service и региональных стандартов см. в статье [что такое Azure Database Migration Service](dms-overview.md).

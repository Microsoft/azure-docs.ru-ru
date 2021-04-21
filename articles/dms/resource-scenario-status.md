---
title: Состояние сценария миграции базы данных
titleSuffix: Azure Database Migration Service
description: Узнайте о состоянии сценариев миграции, поддерживаемых службой Azure Database Migration Service.
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
ms.openlocfilehash: fc2ab86a318086750b11780a7802aa3591065264
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103463516"
---
# <a name="status-of-migration-scenarios-supported-by-azure-database-migration-service"></a>Состояние сценариев миграции, поддерживаемых службой Azure Database Migration Service.

Azure Database Migration Service предназначена для поддержки различных сценариев миграции (пар исходных и целевых объектов): автономной (однократной) и с подключением к сети (непрерывная синхронизация). Сценарии использования, предоставляемые Azure Database Migration Service, расширяются со временем. Регулярно добавляются новые сценарии. В этой статье перечислены сценарии миграции, которые в настоящее время поддерживаются службой Azure Database Migration Service, и состояние (закрытая предварительная версия, общедоступная предварительная версия, общедоступная версия) каждого сценария.

## <a name="offline-versus-online-migrations"></a>Автономная миграция и миграция с подключением к сети

С помощью Azure Database Migration Service эту операцию можно выполнять автономно или с подключением к сети. При использовании *автономной* миграции простой приложения начинается в то же время, что и миграция. Чтобы ограничить время простоя временем, затрачиваемым на переключение на новую среду после завершения миграции, можно использовать миграцию *с подключением к сети*. Рекомендуется выполнить тестирование миграции в автономном режиме, чтобы определить, является ли простой допустимым, и если нет, то выполнить миграцию в сетевое развертывание.

## <a name="migration-scenario-status"></a>Состояние сценария миграции

Состояние сценариев миграции, поддерживаемых службой Azure Database Migration Service, меняется со временем. Как правило, сначала сценарии выпускаются в **закрытой предварительной версии**. Для использования закрытой предварительной версии клиенты должны отправить заявку через [сайт предварительных версий DMS](https://aka.ms/dms-preview). Когда этап закрытой предварительной версии завершается, состояние сценария изменяется на **Общедоступная предварительная версия**. Пользователи Azure Database Migration Service могут опробовать сценарии миграции в общедоступной предварительной версии непосредственно из пользовательского интерфейса. Регистрация не требуется.  Тем не менее сценарии миграции в общедоступной предварительной версии могут быть доступны не во всех регионах и могут меняться перед окончательным выпуском. После выхода общедоступной предварительной версии состояние сценария изменится на **общая доступность**. Общедоступная версия — это окончательное состояние выпуска, и этот функционал завершен и доступен всем пользователям.

## <a name="migration-scenario-support"></a>Поддержка сценария миграции

В следующих таблицах показано, какие сценарии миграции поддерживаются при использовании Azure Database Migration Service.

> [!NOTE]
> Если сценарий, указанный ниже как поддерживаемый, не появляется в пользовательском интерфейсе, обратитесь к [команде Azure Database Migration](mailto:AskAzureDatabaseMigrations@service.microsoft.com), чтобы получить дополнительные сведения.

> [!IMPORTANT]
> Чтобы просмотреть все сценарии, которые в настоящее время поддерживаются Azure Database Migration Service в закрытой предварительной версии, см. [сайт предварительных версий DMS](https://aka.ms/dms-preview).

### <a name="offline-one-time-migration-support"></a>Поддержка автономной (однократной) миграции

В следующей таблице показана поддержка автономных миграций в службе Azure Database Migration Service.

| Целевой объект  | Источник | Поддержка | Состояние |
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
| **База данных Azure PostgreSQL — один сервер** | PostgreSQL | X |
|  | RDS PostgreSQL | X |   |
| **База данных Azure для PostgreSQL — Гипермасштабирование (Citus)** | PostgreSQL | X |
|  | RDS PostgreSQL | X |   |

### <a name="online-continuous-sync-migration-support"></a>Поддержка миграции с подключением к сети (непрерывная синхронизация)

В следующей таблице показана поддержка миграций с подключением к сети в службе Azure Database Migration Service.

| Целевой объект  | Источник | Поддержка | Состояние |
| ------------- | ------------- |:-------------:|:-------------:|
| **БД SQL Azure** | SQL Server | X |  |
|   | RDS SQL | X |  |
|   | Oracle; | X |  |
| **Управляемый экземпляр Базы данных SQL Azure** | SQL Server | ✔ | GA |
|   | RDS SQL | X |  |
|   | Oracle; | X |  |
| **Виртуальная машина Azure SQL** | SQL Server | X |   |
|   | Oracle;  | X |  |
| **Azure Cosmos DB** | MongoDB | ✔ | GA |
| **База данных Azure для MySQL** | MySQL | ✔ | GA |
|   | RDS MySQL | ✔ | GA |
| **База данных Azure PostgreSQL — один сервер** | PostgreSQL | ✔ | GA |
|   | База данных Azure PostgreSQL — один сервер | ✔ | GA |
|   | RDS PostgreSQL | ✔ | GA |
|   | Oracle; | ✔ | Общедоступная предварительная версия (является нерекомендуемой после 1 мая 2021 г.) |
| **База данных Azure для PostgreSQL — Гипермасштабирование (Citus)** | PostgreSQL | ✔ | GA |
|   | RDS PostgreSQL | ✔ | GA |

> [!IMPORTANT]
> Сценарий миграции с Oracle на Базу данных Azure для PostgreSQL (сейчас на этапе предварительной версии) станет недоступен после 1 мая 2021 г. Мы продолжим предоставлять поддержку через альтернативные средства (например, Ora2pg) и обеспечивать оптимальный способ миграции с Oracle на PostgreSQL. Рекомендации по миграции см. в [руководстве по миграции с Oracle на Базу данных Azure для PostgreSQL](https://aka.ms/OracletoPGguide).


## <a name="next-steps"></a>Дальнейшие действия

Общие сведения о службе Azure Database Migration Service и доступности по регионам см. в статье [Что такое Azure Database Migration Service](dms-overview.md).

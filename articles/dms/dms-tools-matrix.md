---
title: Матрица Azure Database Migration Service инструментов
description: Дополнительные сведения о службах и средствах, доступных для переноса баз данных и для поддержки различных этапов процесса миграции.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: reference
ms.date: 03/03/2020
ms.openlocfilehash: 258874ffdbc471305b76f7d52406d2f45e708829
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99575743"
---
# <a name="services-and-tools-available-for-data-migration-scenarios"></a>Доступные службы и средства для сценариев переноса данных

Эта статья содержит матрицу Майкрософт, а также сторонние службы и средства, доступные для помощи в различных сценариях перемещения баз данных и данных, а также специальных задачах.

В следующих таблицах приведены службы и средства, которые можно использовать для планирования успешного переноса данных и выполнения различных этапов.

> [!NOTE]
> В следующих таблицах сторонние средства отмечены звездочкой (*).

## <a name="business-justification-phase"></a>Этап коммерческого обоснования

| Источник | Назначение | Обнаружение /<br/>Inventory (Товары) | Целевое средство и рекомендация<br/>рекомендация. | Совокупная стоимость владения/рентабельность инвестиций и<br/>Бизнес-ситуация |
| --- | --- | --- | --- | --- |
| SQL Server | БД SQL Azure | [MAP Toolkit](/previous-versions//bb977556(v=technet.10))<br/>[Миграция Azure](https://azure.microsoft.com/services/azure-migrate/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
 SQL Server | Управляемый экземпляр Базы данных SQL Azure | [MAP Toolkit](/previous-versions//bb977556(v=technet.10))<br/>[Миграция Azure](https://azure.microsoft.com/services/azure-migrate/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| SQL Server | Виртуальная машина с SQL Azure | [MAP Toolkit](/previous-versions//bb977556(v=technet.10))<br/>[Миграция Azure](https://azure.microsoft.com/services/azure-migrate/)<br/>[Cloudamize](https://www.cloudamize.com/) | [Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| SQL Server | Azure Synapse Analytics | [MAP Toolkit](/previous-versions//bb977556(v=technet.10))<br/>[Миграция Azure](https://azure.microsoft.com/services/azure-migrate/)<br/>[Cloudamize](https://www.cloudamize.com/) |  | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| RDS SQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  | [DMA](/sql/dma/dma-overview?view=sql-server-2017) | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| Oracle; | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [MAP Toolkit](/previous-versions//bb977556(v=technet.10))<br/>[Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[MigVisor*](https://www.migvisor.com/) |  |
| Oracle; | Azure Synapse Analytics | [MAP Toolkit](/previous-versions//bb977556(v=technet.10))<br/>[Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Oracle; | База данных Azure для PostgreSQL —<br/>Отдельный сервер | [MAP Toolkit](/previous-versions//bb977556(v=technet.10))<br/>[Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) |  |  |
| MongoDB | Cosmos DB | [Cloudamize](https://www.cloudamize.com/) | [Cloudamize](https://www.cloudamize.com/) |  |
| Cassandra | Cosmos DB |  |  |  |
| MySQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/) | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| MySQL | База данных Azure для MySQL | [Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) |  | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| RDS MySQL | База данных Azure для MySQL |  |  | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер | [Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) |  | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| RDS PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер |  |  | [Калькулятор совокупной стоимости владения](https://azure.microsoft.com/pricing/tco/calculator/) |
| DB2 | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Access | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Sybase — SAP ASE | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Миграция Azure](https://azure.microsoft.com/services/azure-migrate/) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Sybase — SAP IQ | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |  |  |
| | | | | |

## <a name="pre-migration-phase"></a>Этап подготовки к миграции

| Источник | Назначение | Доступ к данным приложений<br/>Оценка слоя | База данных<br/>Оценка | Производительность<br/>Оценка |
| --- | --- | --- | --- | --- |
| SQL Server | БД SQL Azure | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [Канал DMA](/sql/dma/dma-overview?view=sql-server-2017) | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DEA](https://www.microsoft.com/download/details.aspx?id=54090)<br/>[Cloudamize](https://www.cloudamize.com/) |
| SQL Server | Управляемый экземпляр Базы данных SQL Azure | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [Канал DMA](/sql/dma/dma-overview?view=sql-server-2017) | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DEA](https://www.microsoft.com/download/details.aspx?id=54090)<br/>[Cloudamize](https://www.cloudamize.com/) |
| SQL Server | Виртуальная машина с SQL Azure | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [Канал DMA](/sql/dma/dma-overview?view=sql-server-2017) | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DEA](https://www.microsoft.com/download/details.aspx?id=54090)<br/>[Cloudamize](https://www.cloudamize.com/) |
| SQL Server | Azure Synapse Analytics | [дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit) |  |  |
| RDS SQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [Канал DMA](/sql/dma/dma-overview?view=sql-server-2017) | [DMA](/sql/dma/dma-overview?view=sql-server-2017) | [DEA](https://www.microsoft.com/download/details.aspx?id=54090) |
| Oracle; | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Oracle; | Azure Synapse Analytics | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Oracle; | База данных Azure для PostgreSQL —<br/>Отдельный сервер |  | [Ora2Pg*](http://ora2pg.darold.net/start.html) |  |
| MongoDB | Cosmos DB |  | [Cloudamize](https://www.cloudamize.com/) | [Cloudamize](https://www.cloudamize.com/) |
| Cassandra | Cosmos DB |  |  |  |
| MySQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/) |  |
| MySQL | База данных Azure для MySQL |  |  |  |
| RDS MySQL | База данных Azure для MySQL |  |  |  |
| PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер |  |  |  |
| RDS PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер |  |  |  |
| DB2 | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Access | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Sybase — SAP ASE | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Дамт](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)  /  [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |  |
| Sybase — SAP IQ | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  | |  |
| | | | | |

## <a name="migration-phase"></a>Этап миграции

| Источник | Назначение | схема | Данные<br/>Работа | Данные<br/>Собрания |
| --- | --- | --- | --- | --- |
| SQL Server | БД SQL Azure | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[Cloudamize](https://www.cloudamize.com/) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| SQL Server | Управляемый экземпляр Базы данных SQL Azure | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| SQL Server | Виртуальная машина с SQL Azure | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[DMS](https://azure.microsoft.com/services/database-migration/)<br>[Cloudamize](https://www.cloudamize.com/) | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[DMS](https://azure.microsoft.com/services/database-migration/)<br>[Cloudamize](https://www.cloudamize.com/) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| SQL Server | Azure Synapse Analytics |  |  |  |
| RDS SQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [DMA](/sql/dma/dma-overview?view=sql-server-2017) | [DMA](/sql/dma/dma-overview?view=sql-server-2017)<br/>[DMS](https://azure.microsoft.com/services/database-migration/) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| Oracle; | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[SharePlex*](https://www.quest.com/products/shareplex/)<br/>[Испирер *](https://www.ispirer.com/solutions) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[SharePlex*](https://www.quest.com/products/shareplex/)<br/>[Испирер *](https://www.ispirer.com/solutions) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[SharePlex*](https://www.quest.com/products/shareplex/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| Oracle; | Azure Synapse Analytics | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[SharePlex*](https://www.quest.com/products/shareplex/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| Oracle; | База данных Azure для PostgreSQL —<br/>Отдельный сервер | [Испирер *](https://www.ispirer.com/solutions) | [Испирер *](https://www.ispirer.com/solutions) | [DMS](https://azure.microsoft.com/services/database-migration/) |
| MongoDB | Cosmos DB | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/)<br/>[Imanis Data*](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/talena-inc.talena-solution-template?tab=Overview) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/)<br/>[Imanis Data*](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/talena-inc.talena-solution-template?tab=Overview) | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Cloudamize](https://www.cloudamize.com/)<br/>[Imanis Data*](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/talena-inc.talena-solution-template?tab=Overview)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| Cassandra | Cosmos DB | [Imanis Data*](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/talena-inc.talena-solution-template?tab=Overview) | [Imanis Data*](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/talena-inc.talena-solution-template?tab=Overview) | [Imanis Data*](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/talena-inc.talena-solution-template?tab=Overview) |
| MySQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| MySQL | База данных Azure для MySQL | [mysqldump*](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html) |  | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| RDS MySQL | База данных Azure для MySQL | [mysqldump*](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html) |  | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер | [pg_dump*](https://www.postgresql.org/docs/11/static/app-pgdump.html) |  | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| RDS PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер | [pg_dump*](https://www.postgresql.org/docs/11/static/app-pgdump.html) |  | [DMS](https://azure.microsoft.com/services/database-migration/)<br/>[Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| DB2 | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| Access | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017) |
| Sybase — SAP ASE | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [SSMA](/sql/ssma/sql-server-migration-assistant?view=sql-server-2017)<br/>[Испирер *](https://www.ispirer.com/solutions) | [Attunity](https://www.attunity.com/products/replicate/)<br/>[Striim*](https://www.striim.com/partners/striim-for-microsoft-azure/) |
| Sybase — SAP IQ | База данных SQL Azure, Управляемый экземпляр, виртуальная машина | [Испирер *](https://www.ispirer.com/solutions) | [Испирер *](https://www.ispirer.com/solutions) | |
| | | | | |

## <a name="post-migration-phase"></a>Этап после миграции

| Источник | Назначение | Оптимизация |
| --- | --- | --- |
| SQL Server | БД SQL Azure | [Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) |
| SQL Server | Управляемый экземпляр Базы данных SQL Azure | [Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) |
| SQL Server | Виртуальная машина с SQL Azure | [Cloud Atlas*](https://www.unifycloud.com/cloud-migration-tool/)<br/>[Cloudamize](https://www.cloudamize.com/) |
| SQL Server | Azure Synapse Analytics |  |
| RDS SQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |
| Oracle; | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |
| Oracle; | Azure Synapse Analytics |  |
| Oracle; | База данных Azure для PostgreSQL —<br/>Отдельный сервер |  |
| MongoDB | Cosmos DB | [Cloudamize](https://www.cloudamize.com/) |
| Cassandra | Cosmos DB |  |
| MySQL | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |
| MySQL | База данных Azure для MySQL |  |
| RDS MySQL | База данных Azure для MySQL |  |
| PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер |  |
| RDS PostgreSQL | База данных Azure для PostgreSQL —<br/>Отдельный сервер |  |
| DB2 | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |
| Access | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |
| Sybase — SAP ASE | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |
| Sybase — SAP IQ | База данных SQL Azure, Управляемый экземпляр, виртуальная машина |  |
| | | |

## <a name="next-steps"></a>Дальнейшие действия

Общие сведения о Azure Database Migration Service см. в статье [что такое Azure Database Migration Service](dms-overview.md).
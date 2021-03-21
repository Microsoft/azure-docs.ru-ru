---
title: Что такое служба Azure Database Migration Service?
description: Обзор службы Azure Database Migration Service, которая обеспечивает непрерывную миграцию из множества источников баз данных на платформы данных Azure.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.topic: overview
ms.date: 02/20/2020
ms.openlocfilehash: 328c29afee3752ecb11b83f22d67f20aa3a2c93e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94963017"
---
# <a name="what-is-azure-database-migration-service"></a>Что такое служба Azure Database Migration Service?

Azure Database Migration Service — это полностью управляемая служба, которая выполняет непрерывную миграцию из множества источников баз данных на платформы данных Azure с минимальным временем простоя (миграции в подключенном режиме).

## <a name="migrate-databases-to-azure-with-familiar-tools"></a>Миграция баз данных в Azure с помощью привычных средств

В Azure Database Migration Service реализованы некоторые возможности, которые можно найти в других наших средствах и службах. Она предоставляет пользователям комплексное решение с высокой доступностью. Служба использует [Помощник по миграции данных](/sql/dma/dma-overview) для создания экспертных отчетов с рекомендациями по изменениям, которые необходимы для переноса данных. Все необходимые исправления выполняются пользователями. Когда вы будете готовы приступить к переносу, Azure Database Migration Service выполнит все необходимые действия. Запустив перенос, вы можете больше не думать об этом процессе. Все будет сделано автоматически в соответствии с рекомендациями корпорации Майкрософт. 

> [!NOTE]
> Чтобы выполнить сетевую миграцию с помощью Azure Database Migration Service, требуется создать экземпляр ценовой категории "Премиум".

## <a name="regional-availability"></a>Доступность по регионам

Актуальные сведения о доступности Azure Database Migration Service в регионах см. на странице [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=database-migration).

## <a name="pricing"></a>Цены

Актуальные сведения о ценах на Azure Database Migration Service см. [на этой странице с ценами](https://azure.microsoft.com/pricing/details/database-migration/).

## <a name="next-steps"></a>Дальнейшие действия

* [Состояние сценариев миграции, поддерживаемых службой Azure Database Migration Service.](resource-scenario-status.md)
* [Создание экземпляра службы Database Migration Service с помощью портала Azure.](quickstart-create-data-migration-service-portal.md)
* [Миграция с SQL Server в Базу данных SQL Azure](tutorial-sql-server-to-azure-sql.md).
* [Предварительные требования для использования службы Azure Database Migration Service.](pre-reqs.md)
* [Часто задаваемые вопросы о службе Azure Database Migration Service.](faq.md)
* [Доступные службы и средства для сценариев переноса данных](dms-tools-matrix.md)
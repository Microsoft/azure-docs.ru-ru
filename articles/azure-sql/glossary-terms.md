---
title: Глоссарий терминов
titleSuffix: Azure SQL
description: Глоссарий терминов для работы с базой данных SQL Azure, Azure SQL Управляемый экземпляр и SQL на виртуальной машине Azure.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=4
ms.devlang: ''
ms.topic: reference
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/09/2020
ms.openlocfilehash: 8bd1e312463cf89ae30d54bc4a32a497fc7a9a78
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98249658"
---
# <a name="azure-sql-database-glossary-of-terms"></a>Глоссарий терминов для Базы данных SQL Azure
[!INCLUDE[appliesto-asf](includes/appliesto-asf.md)]

## <a name="azure-sql-database"></a>База данных SQL Azure

|Контекст|Термин|Дополнительные сведения|
|:---|:---|:---|
|Служба Azure|База данных SQL Azure или База данных SQL|[База данных SQL Azure](database/sql-database-paas-overview.md)|
|Модель приобретения|Модель приобретения на основе единиц DTU|[Модель приобретения на основе DTU](database/service-tiers-dtu.md)|
||Модель приобретения на основе виртуальных ядер|[Модель приобретения на основе виртуальных ядер](database/service-tiers-vcore.md)|
|Параметр развертывания |Отдельная база данных|[Отдельные базы данных](database/single-database-overview.md)|
||Эластичный пул|[Эластичный пул](database/elastic-pool-overview.md)|
|Уровень служб|Базовый, Стандартный, премиум, общего назначения, масштабирование, критически важный для бизнеса|Сведения об уровнях служб в модели Виртуальное ядро см. в разделе [уровни служб базы данных SQL](database/service-tiers-vcore.md#service-tiers). Сведения об уровнях служб в модели DTU см. в разделе [модель DTU](database/service-tiers-dtu.md#compare-the-dtu-based-service-tiers).|
|Уровень вычислений|Бессерверные вычисления|[Бессерверные вычисления](database/service-tiers-vcore.md#compute-tiers)
||Подготовленные вычисления|[Подготовленные вычисления](database/service-tiers-vcore.md#compute-tiers)
|Поколение вычислительных ресурсов|Серия го поколения, M-Series, серия fsv2, серии DC|[Поколения оборудования](database/service-tiers-vcore.md#hardware-generations)
|Серверная сущность| Сервер |[Логические серверы SQL Server](database/logical-servers.md)|
|Тип ресурса|Виртуальное ядро|Ядро ЦП, предоставляемое вычислительному ресурсу для отдельной базы данных, эластичного пула. |
||Размер вычислений и объем хранилища|Размер вычислений — это максимальный объем ресурсов ЦП, памяти и других устройств, не связанных с хранилищем, доступных для отдельной базы данных или эластичного пула.  Размер хранилища — это максимальный объем хранилища, доступный для отдельной базы данных или эластичного пула. Параметры изменения размера в модели Виртуальное ядро см. в разделе [Виртуальное ядро Single databases](database/resource-limits-vcore-single-databases.md)and [Виртуальное ядро эластичные пулы](database/resource-limits-vcore-elastic-pools.md).  (.. /манажед-инстанце/ресаурце-лимитс.МД).  Сведения о параметрах изменения размера в модели DTU см. в разделе [DTU Single databases](database/resource-limits-dtu-single-databases.md) and [DTU эластичные пулы](database/resource-limits-dtu-elastic-pools.md).

## <a name="azure-sql-managed-instance"></a>Управляемый экземпляр SQL Azure

|Контекст|Термин|Дополнительные сведения|
|:---|:---|:---|
|Служба Azure|Управляемый экземпляр SQL Azure|[Управляемый экземпляр SQL](managed-instance/sql-managed-instance-paas-overview.md)|
|Модель приобретения|Модель приобретения на основе виртуальных ядер|[Модель приобретения на основе виртуальных ядер](database/service-tiers-vcore.md)|
|Параметр развертывания |Один экземпляр|[Один экземпляр](managed-instance/sql-managed-instance-paas-overview.md)|
||Пул экземпляров (Предварительная версия)|[Пулы экземпляров](managed-instance/instance-pools-overview.md)|
|Уровень служб|Общего назначения, критически важный для бизнеса|[Уровни служб SQL Управляемый экземпляр](managed-instance/sql-managed-instance-paas-overview.md#service-tiers)|
|Уровень вычислений|Подготовленные вычисления|[Подготовленные вычисления](database/service-tiers-vcore.md#compute-tiers)|
|Поколение вычислительных ресурсов|5-го поколения|[Поколения оборудования](database/service-tiers-vcore.md#hardware-generations)
|Серверная сущность|Управляемый экземпляр или экземпляр| Н/д, так как SQL Управляемый экземпляр сам является сервером |
|Тип ресурса|Виртуальное ядро|Ядро ЦП, предоставляемое вычислительному ресурсу для Управляемый экземпляр SQL.|
||Размер вычислений и объем хранилища|Размер вычислений — это максимальный объем ресурсов ЦП, памяти и других устройств, не связанных с хранилищем, для SQL Управляемый экземпляр.  Размер хранилища — это максимальный объем хранилища, доступный для Управляемый экземпляр SQL.  Для параметров изменения размера, [управляемых экземпляров SQL](managed-instance/resource-limits.md). |

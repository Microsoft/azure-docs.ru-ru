---
title: Преимущество гибридного использования Azure
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Используйте существующие лицензии SQL Server для базы данных SQL Azure и скидок Управляемый экземпляр SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=4
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake
ms.date: 02/16/2021
ms.openlocfilehash: f7a37e761e37e295bbb92e442b1813ebded2a7cd
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104955284"
---
# <a name="azure-hybrid-benefit---azure-sql-database--sql-managed-instance"></a>Преимущество гибридного использования Azure — база данных SQL Azure & SQL Управляемый экземпляр
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

В подготовленной среде вычислений для модели приобретения на основе виртуальное ядро вы можете обмениваться существующими лицензиями по тарифам со скидкой в базе данных SQL Azure и Azure SQL Управляемый экземпляр с помощью [преимущество гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-benefit/). Это преимущество Azure позволяет экономить до 30% или даже более высокого уровня в базе данных SQL & Управляемый экземпляр SQL с помощью лицензий на SQL Server с Software Assurance. На [преимущество гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) странице имеется калькулятор, помогающий определить экономию.  Обратите внимание, что Преимущество гибридного использования Azure не относится к серверу базы данных SQL Azure.

> [!NOTE]
> Переход на Преимущество гибридного использования Azure не требует времени простоя.

![структура цен на виртуальное ядро](./media/azure-hybrid-benefit/pricing.png)

## <a name="choose-a-license-model"></a>Выбор модели лицензии

С помощью Преимущество гибридного использования Azure вы можете платить только за базовую инфраструктуру Azure, используя существующую лицензию SQL Server для самого SQL Server ядра СУБД (цены на базовое вычисление) или заплатить как за базовую инфраструктуру, так и за лицензию SQL Server (цены, включаемые в лицензию).

Вы можете выбрать или изменить модель лицензирования в портал Azure: 
- Для новых баз данных во время создания выберите **настроить базу данных** на вкладке " **основы** " и выберите параметр для экономии денег.
- Для существующих баз данных выберите **настроить** в меню **Параметры** и выберите параметр для экономии денег.

Можно также настроить новую или существующую базу данных с помощью одного из следующих API-интерфейсов:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы задать или обновить тип лицензии с помощью PowerShell, выполните следующие действия.

- [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)
- [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)
- [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance)
- [Set-AzSqlInstance](/powershell/module/az.sql/set-azsqlinstance)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы задать или обновить тип лицензии с помощью Azure CLI:

- [az sql db create](/cli/azure/sql/db#az-sql-db-create)
- [az sql mi create](/cli/azure/sql/mi#az-sql-mi-create)
- [az sql mi update](/cli/azure/sql/mi#az-sql-mi-update)

# <a name="rest-api"></a>[REST API](#tab/rest)

Чтобы задать или обновить тип лицензии с помощью REST API:

- [Базы данных: создание или обновление](/rest/api/sql/databases/createorupdate)
- [Базы данных: обновление](/rest/api/sql/databases/update)
- [Управляемые экземпляры. Создание или обновление](/rest/api/sql/managedinstances/createorupdate)
- [Управляемые экземпляры. Обновление](/rest/api/sql/managedinstances/update)

* * *


### <a name="azure-hybrid-benefit-questions"></a>Вопросы о Преимуществе гибридного использования Azure

#### <a name="are-there-dual-use-rights-with-azure-hybrid-benefit-for-sql-server"></a>Включает ли программа "Преимущество гибридного использования Azure для SQL Server" права двойного использования?

Права двойного использования лицензии предусматривают 180 дней для обеспечения беспрепятственного выполнения перемещений. После этого 180-дневного периода можно использовать только лицензию SQL Server в облаке в базе данных SQL. У вас больше нет прав на двойное использование в локальной среде и в облаке.

#### <a name="how-does-azure-hybrid-benefit-for-sql-server-differ-from-license-mobility"></a>Каким образом программа "Преимущество гибридного использования Azure для SQL Server" отличается от перемещения лицензий?

Мы предлагаем преимущества перемещения лицензий для SQL Server клиентов с помощью программы Software Assurance. Это позволяет переназначить свои лицензии общим серверам партнера. Это преимущество можно использовать в Azure IaaS и AWS EC2.

Программа "Преимущество гибридного использования Azure для SQL Server" отличается от программы перемещения лицензий в двух ключевых аспектах.

- Она предоставляет экономические преимущества перемещения высоко виртуализированных рабочих нагрузок в Azure. Клиенты SQL Server Enterprise Edition могут получить четыре ядра в Azure в общего назначения SKU для каждого ядра, который они владеют локально для приложений с высокой степенью виртуализации. Перемещение лицензий не позволяет использовать специальные преимущества для перемещения виртуализированных рабочих нагрузок в облако.
- Он обеспечивает назначение PaaS в Azure (SQL Управляемый экземпляр), которое строго совместимо с SQL Server.

#### <a name="what-are-the-specific-rights-of-the-azure-hybrid-benefit-for-sql-server"></a>Что представляют собой права программы "Преимущество гибридного использования Azure для SQL Server"?

Клиенты базы данных SQL и SQL Управляемый экземпляр имеют следующие права, связанные с Преимущество гибридного использования Azure для SQL Server:

|Лицензионное соглашение|Что Преимущество гибридного использования Azure для SQL Server получить?|
|---|---|
|Основные клиенты SQL Server Enterprise Edition с Software Assurance|<li>Можно ли платить по базовому тарифу на масштабирование, общего назначения или критически важный для бизнеса SKU</li><br><li>1 ядро в локальной среде = 4 ядра в номере SKU для масштабирования</li><br><li>1 ядро в локальной среде = 4 ядра в номере SKU общего назначения</li><br><li>1 ядро в локальной среде = 1 ядро в номере SKU "Критически важный для бизнеса"</li>|
|Основные клиенты SQL Server Standard Edition с Software Assurance|<li>Можно ли платить по базовому тарифу на масштабирование, общего назначения или критически важный для бизнеса SKU</li><br><li>1 ядро в локальной среде = 1 ядро в ЕДИНИЦах масштабирования</li><br><li>1 ядро в локальной среде = 1 ядро в номере SKU общего назначения</li><br><li>4 ядро в локальной среде = 1 ядро в критически важный для бизнеса SKU</li>|
|||


## <a name="next-steps"></a>Дальнейшие действия

- Сведения о выборе варианта развертывания Azure SQL см. в разделе [Выбор правильного развертывания в SQL Azure](azure-sql-iaas-vs-paas-what-is-overview.md).
- Сравнение функций базы данных SQL и Управляемый экземпляр SQL см. в разделе [SQL database & sql управляемый экземпляр Features](database/features-comparison.md).

---
title: Действия по оптимизации после миграции с помощью API Azure Cosmos DB для MongoDB
description: Этот документ содержит методики оптимизации после миграции от MongoDB к APi Azure Cosmos DB для Mongo DB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 02/14/2021
ms.author: chrande
ms.openlocfilehash: ce0facaba43e9cc8e37b19fbd3dfc840067d278f
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102560401"
---
# <a name="post-migration-optimization-steps-when-using-azure-cosmos-dbs-api-for-mongodb"></a>Действия по оптимизации после миграции при использовании API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Для управления хранимыми данными, перенесенными из базы данных MongoDB в API Azure Cosmos DB для MongoDB, можно подключиться к Azure Cosmos DB. В этом руководстве описаны действия, которые могут потребоваться после миграции. Инструкции по миграции см. в [руководстве по миграции MongoDB в Azure Cosmos DB API для MongoDB](../dms/tutorial-mongodb-cosmos-db.md) .

Из этого руководства вы узнаете, как выполнить следующие задачи:

- [Подключение приложения](#connect-your-application)
- [Оптимизация политики индексирования](#optimize-the-indexing-policy)
- [Настройка глобального распределения для API Azure Cosmos DB для MongoDB](#globally-distribute-your-data)
- [Задать уровень согласованности](#set-consistency-level)

> [!NOTE]
> Единственным обязательным шагом после переноса на уровне приложения является изменение строки подключения в приложении так, чтобы она указывала на новую учетную запись Azure Cosmos DB. Все остальные шаги миграции рекомендуются для оптимизации.
>

## <a name="connect-your-application"></a>Подключение приложения

1. В новом окне Войдите в [портал Azure](https://www.portal.azure.com/)
2. В области слева в [портал Azure](https://www.portal.azure.com/)откройте меню **все ресурсы** и найдите учетную запись Azure Cosmos DB, в которую были перенесены данные.
3. Откройте колонку **строка подключения** . На правой панели содержатся все сведения, необходимые для успешного подключения к учетной записи.
4. Используйте сведения о подключении в конфигурации приложения (или в других соответствующих местах), чтобы отразить Azure Cosmos DB API для подключения к MongoDB в приложении.
:::image type="content" source="./media/mongodb-post-migration/connection-string.png" alt-text="На снимке экрана показаны параметры для строки подключения.":::

Дополнительные сведения см. на странице [Подключение приложения MongoDB к Azure Cosmos DB](connect-mongodb-account.md) .

## <a name="optimize-the-indexing-policy"></a>Оптимизация политики индексирования

По умолчанию все поля данных автоматически индексируются во время переноса данных в Azure Cosmos DB. Во многих случаях эта политика индексирования по умолчанию приемлема. Как правило, удаление индексов оптимизирует запросы на запись и наличие политики индексирования по умолчанию (т. е. Автоматическое индексирование) оптимизирует запросы на чтение.

Дополнительные сведения об индексировании см. в разделах [индексирование данных в API Azure Cosmos DB для MongoDB](mongodb-indexing.md) , а также [индексирование в Azure Cosmos DB](index-overview.md) статьях.

## <a name="globally-distribute-your-data"></a>Глобально распространять данные

Служба Azure Cosmos DB доступна во всех [регионах Azure](https://azure.microsoft.com/regions/#services) по всему миру. После выбора уровня согласованности по умолчанию для учетной записи Azure Cosmos DB можно связать один или несколько регионов Azure (в зависимости от потребностей глобального распределения). Для обеспечения высокой доступности и непрерывности бизнес-процессов мы всегда рекомендуем запускать по крайней мере в двух регионах. Вы можете ознакомиться с советами по [оптимизации затрат на развертывание в нескольких регионах в Azure Cosmos DB](optimize-cost-regions.md).

Чтобы глобально распространить данные, см. статью [глобальное распространение данных в API Azure Cosmos DB для MongoDB](tutorial-global-distribution-mongodb.md).

## <a name="set-consistency-level"></a>Задать уровень согласованности

Azure Cosmos DB предлагает 5 четко определенных [уровней согласованности](consistency-levels.md). Для чтения о сопоставлении уровней согласованности MongoDB и Azure Cosmos DB, считывании [уровней согласованности и Azure Cosmos DBных API](./consistency-levels.md). Уровень согласованности по умолчанию — уровень согласованности сеанса. Изменение уровня согласованности является необязательным, и его можно оптимизировать для приложения. Чтобы изменить уровень согласованности с помощью портал Azure, выполните следующие действия.

1. Перейдите в колонку **согласованности по умолчанию** в разделе Параметры.
2. Выберите [уровень согласованности](consistency-levels.md)

Большинство пользователей оставляют свой уровень согласованности при настройке согласованности сеансов по умолчанию. Однако существуют [компромиссы доступности и производительности для различных уровней согласованности](./consistency-levels.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Подключение приложения MongoDB к Azure Cosmos DB](connect-mongodb-account.md)
* [Подключение к учетной записи Azure Cosmos DB с помощью Studio 3T](mongodb-mongochef.md)
* [Глобальное распространение операций чтения с помощью API Azure Cosmos DB для MongoDB](mongodb-readpreference.md)
* [Завершение срока действия данных с помощью API Azure Cosmos DB для MongoDB](mongodb-time-to-live.md)
* [Уровни согласованности в Azure Cosmos DB](consistency-levels.md)
* [Индексирование в Azure Cosmos DB](index-overview.md)
* [Единицы запросов в Azure Cosmos DB](request-units.md)
---
title: Сопоставление уровней согласованности для Azure Cosmos DB API для MongoDB
description: Сопоставление уровней согласованности для Azure Cosmos DB API для MongoDB.
author: sivethe
ms.author: sivethe
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: conceptual
ms.date: 10/12/2020
ms.reviewer: sngun
ms.openlocfilehash: e8ac3e376c8d67e82def3a57910707c6b1433912
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93333160"
---
# <a name="consistency-levels-for-azure-cosmos-db-and-the-api-for-mongodb"></a>Уровни согласованности для Azure Cosmos DB и API для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В отличие от Azure Cosmos DB, машинный MongoDB не предоставляет точно определенные гарантии согласованности. Вместо этого Native MongoDB позволяет пользователям настраивать следующие гарантии согласованности: запись, проблема чтения и директива «Master» — для направления операций чтения к первичным или вторичным репликам для достижения требуемого уровня согласованности.

При использовании API Azure Cosmos DB для MongoDB драйвер MongoDB считает регион записи первичной репликой, а все остальные регионы — репликой чтения. Вы можете выбрать регион, связанный с вашей учетной записью Azure Cosmos в качестве первичной реплики.

> [!NOTE]
> По умолчанию для Azure Cosmos DB используется модель согласованности Session. Session — это клиентская модель согласованности, которая изначально не поддерживается ни в Cassandra, ни в MongoDB. Дополнительные сведения о выборе модели согласованности см. в разделе [уровни согласованности в Azure Cosmos DB](consistency-levels.md)

При использовании API Azure Cosmos DB для MongoDB:

* Проблема записи сопоставлена с уровнем согласованности по умолчанию, настроенным в учетной записи Azure Cosmos.

* Azure Cosmos DB динамически сопоставляет проблему чтения, указанную драйвером клиента MongoDB, с одним из Azure Cosmos DBных уровней согласованности, динамически настроенных для запроса на чтение.  

* Вы можете добавить заметки к определенному региону, связанному с вашей учетной записью Azure Cosmos, как "основной", сделав этот регион первым доступным для записи регионом. 

## <a name="mapping-consistency-levels"></a>Сопоставление уровней согласованности

В следующей таблице показано, как проблемы с записью и чтением в машинном MongoDB сопоставляются с уровнями согласованности Azure Cosmos при использовании API Azure Cosmos DB для MongoDB:

:::image type="content" source="./media/consistency-levels-across-apis/consistency-model-mapping-mongodb.png" alt-text="Сопоставление модели согласованности MongoDB" lightbox= "./media/consistency-levels-across-apis/consistency-model-mapping-mongodb.png":::

Если ваша учетная запись Azure Cosmos настроена с уровнем согласованности, отличным от строгой согласованности, можно выяснить вероятность того, что клиенты могут получить надежные и согласованные операции чтения для рабочих нагрузок, просмотрев метрику *Probabilistically ограниченного устаревания* (PBS). Эта метрика предоставляется на портале Azure. Дополнительные сведения см. в разделе [Мониторинг метрики вероятностного ограниченного устаревания (PBS)](how-to-manage-consistency.md#monitor-probabilistically-bounded-staleness-pbs-metric).

Вероятностное ограниченное устаревание показывает степень итоговой согласованности. Эта метрика позволяет понять, насколько часто можно добиться более высокой согласованности, чем уровень согласованности, настроенный в вашей учетной записи Azure Cosmos. Другими словами, вы можете увидеть вероятность (в миллисекундах) выполнения строго согласованных операций чтения для комбинации регионов записи и чтения.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о глобальных уровнях распределения и согласованности для Azure Cosmos DB:

* [Общие сведения о глобальном распределении](distribute-data-globally.md)
* [Обзор уровня согласованности](consistency-levels.md)
* [Высокая доступность](high-availability.md)

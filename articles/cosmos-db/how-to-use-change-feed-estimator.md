---
title: Использование средства оценки канала изменений в Azure Cosmos DB
description: Узнайте, как использовать средство оценки канала изменений для анализа хода выполнения обработчика канала изменений
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 08/15/2019
ms.author: maquaran
ms.custom: devx-track-csharp
ms.openlocfilehash: a44557d15f437317c2b5fa659ab8d4ca3c208edf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339847"
---
# <a name="use-the-change-feed-estimator"></a>Использование средства оценки канала изменений
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описывается, как можно отслеживать ход выполнения экземпляров [обработчика канала изменений](./change-feed-processor.md), считывающих канал изменений.

## <a name="why-is-monitoring-progress-important"></a>Почему процесс отслеживания хода выполнения важен?

Обработчик канала изменений действует как указатель, который перемещается по [каналу изменений](./change-feed.md) и доставляет изменения в реализацию делегата. 

Экземпляр развертывания обработчика канала изменений может обрабатывать изменения с определенной скоростью на основе доступных ресурсов, таких как ЦП, память, сеть и т. д.

Если эта скорость ниже, чем скорость, с которой происходят изменения в контейнере Azure Cosmos, обработчик начнет отставать.

Определение этого сценария помогает понять, нужно ли масштабировать развертывание обработчика канала изменений.

## <a name="implement-the-change-feed-estimator"></a>Реализация средства оценки канала изменений

Как и [обработчик канала изменений](./change-feed-processor.md), средство оценки канала изменений работает как модель передачи. Средство оценки будет измерять разницу между последним обработанным элементом (определяемым состоянием контейнера аренд) и последним изменением в контейнере и передавать это значение делегату. Интервал, с которым производится измерение, также можно настроить по умолчанию на 5 секунд.

Например, если обработчик канала изменений определен таким образом:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartProcessorEstimator)]

Правильным способом инициализации средства оценки для измерения этого обработчика будет использование `GetChangeFeedEstimatorBuilder` следующим образом:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartEstimator)]

Здесь и обработчик, и средство оценки имеют одно значение `leaseContainer` и одно и то же имя.

Два других параметра — это делегат, который получит число, представляющее, **сколько изменений ожидают чтения** обработчиком, и интервал времени, в течение которого вы хотите, чтобы это измерение было выполнено.

Пример делегата, который получает оценку:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=EstimationDelegate)]

Вы можете отправить эту оценку в решение для мониторинга и использовать ее, чтобы понять, как ход выполнения меняется с течением времени.

> [!NOTE]
> Средство оценки канала изменений не нужно развертывать как часть обработчика канала изменений. Оно также не должно быть частью того же проекта. Оно может быть независимым и работать в совершенно ином экземпляре. Просто необходимо использовать одно и то же имя и конфигурацию аренды.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Azure Cosmos DB](sql-api-sdk-dotnet.md)
* [Примеры использования в GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [Дополнительные примеры на GitHub](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)

## <a name="next-steps"></a>Дальнейшие действия

Вы можете продолжить знакомство с обработчиком канала изменений, перейдя к следующим статьям:

* [Обработчик канала изменений в Azure Cosmos DB](change-feed-processor.md)
* [Настройка времени запуска обработчика канала изменений](./change-feed-processor.md#starting-time)
---
title: Устранение неполадок с частотой запросов Azure Cosmos DB слишком большими исключениями
description: Узнайте, как диагностировать и исправить частоту запросов слишком больших исключений.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: c3a692a8df93931bd8fd7982e69fe2f9118444c2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94411343"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-rate-too-large-exceptions"></a>Диагностика и устранение неполадок с частотой запросов Azure Cosmos DB слишком большими исключениями
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Сообщение об ошибке "частота запросов слишком велика" или код ошибки 429 указывает на регулирование запросов.

## <a name="troubleshooting-steps"></a>Действия по устранению неполадок
В следующем разделе содержатся известные причины и решения для слишком большого количества запросов.

### <a name="check-the-metrics"></a>Проверка метрик
Проверьте [Azure Cosmos DBное наблюдение](monitor-cosmos-db.md) , чтобы увидеть число исключений 429.

#### <a name="cause"></a>Причина.
Используемая пропускная способность (единицы запросов в секунду) превысила [подготовленную пропускную способность](set-throughput.md). Пакет SDK автоматически повторяет запросы на основе указанной политики повтора. Если эта ошибка возникает часто, попробуйте увеличить пропускную способность коллекции. Проверьте метрики портала, чтобы узнать, не возникают ли ошибки 429. Проверьте ключ раздела, чтобы обеспечить [равномерное распределение хранилища и объема запросов](partitioning-overview.md).

#### <a name="solution"></a>Решение.
1. Используйте [портал или пакет SDK](set-throughput.md) для увеличения подготовленной пропускной способности.
1. Переключите базу данных или контейнер на [Автомасштабирование](provision-throughput-autoscale.md).

## <a name="next-steps"></a>Дальнейшие действия
* [Диагностика и устранение неполадок](troubleshoot-dot-net-sdk.md) при использовании пакета SDK для Azure Cosmos DB .NET.
* Ознакомьтесь с рекомендациями по производительности для [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) и [.NET v2](performance-tips.md).
* [Диагностика и устранение неполадок](troubleshoot-java-sdk-v4-sql.md) при использовании пакета SDK для Java версии 4 Azure Cosmos DB.
* Сведения о рекомендациях по производительности для [пакета SDK для Java v4](performance-tips-java-sdk-v4-sql.md).
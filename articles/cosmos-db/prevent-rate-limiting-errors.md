---
title: Предотвращение ошибок, ограничивающих частоту, для Azure Cosmos DB API для операций MongoDB.
description: Узнайте, как запретить Azure Cosmos DB API для операций ограничения частоты обращения с помощью функции SSR (Повтор на стороне сервера).
author: gahl-levy
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 01/13/2021
ms.author: gahllevy
ms.openlocfilehash: e1ccf55d38a9a3a5a1d0a3622c90dd7b51e5e477
ms.sourcegitcommit: d49bd223e44ade094264b4c58f7192a57729bada
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/02/2021
ms.locfileid: "99258496"
---
# <a name="prevent-rate-limiting-errors-for-azure-cosmos-db-api-for-mongodb-operations"></a>Предотвращение ошибок, ограничивающих частоту, для Azure Cosmos DB API для операций MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Azure Cosmos DB API для операций MongoDB может завершиться с ошибкой ограничения скорости (16500/429), если они превышают ограничение пропускной способности коллекции (RUs). 

Можно включить функцию повтора на стороне сервера (SSR) и позволить серверу автоматически повторять эти операции. Повторные запросы выполняются после небольшой задержки для всех коллекций в вашей учетной записи. Эта функция является удобной альтернативой для обработки ошибок ограничения частоты в клиентском приложении.

## <a name="use-the-azure-portal"></a>Использование портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Перейдите к Azure Cosmos DB API для учетной записи MongoDB.

1. Перейдите в область **компоненты** под разделом **Параметры** .

1. Выберите пункт **повторить попытку на стороне сервера**.

1. Нажмите кнопку **включить** , чтобы включить эту функцию для всех коллекций в вашей учетной записи.

:::image type="content" source="./media/prevent-rate-limiting-errors/portal-features-server-side-retry.png" alt-text="Снимок экрана с функцией повтора на стороне сервера для Azure Cosmos DB API для MongoDB":::

## <a name="use-the-azure-cli"></a>Использование Azure CLI

1. Убедитесь, что SSR уже включен для вашей учетной записи:
```bash
az cosmosdb show --name accountname --resource-group resourcegroupname
```
2. **Включить** SSR для всех коллекций в учетной записи базы данных. Чтобы это изменение вступило в силу, может потребоваться до 15 мин.
```bash
az cosmosdb update --name accountname --resource-group resourcegroupname --capabilities EnableMongo DisableRateLimitingResponses
```
Следующая команда **отключит SSR для** всех коллекций в учетной записи базы данных. Чтобы это изменение вступило в силу, может потребоваться до 15 мин.
```bash
az cosmosdb update --name accountname --resource-group resourcegroupname --capabilities EnableMongo DisableRateLimitingResponses
```

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы
* Как повторяются запросы?
    * Повторные запросы выполняются постоянно (повторно и снова) до тех пор, пока не будет достигнуто время ожидания в 60 секунд. Если время ожидания истекло, клиент получит [исключение ексцеедедтимелимит (50)](mongodb-troubleshoot.md).
*  Как можно отслеживать последствия SSR?
    *  В области метрики Cosmos DB можно просмотреть ошибки ограничения частоты (429s), которые повторяются на стороне сервера. Следует помнить, что эти ошибки не переходят на клиент, когда SSR включен, так как они обрабатываются и повторяются на стороне сервера. 
    *  В журналах [Cosmos DB ресурсов](cosmosdb-monitor-resource-logs.md)можно искать записи журнала, содержащие "естиматедделайфромрателимитингинмиллисекондс".
*  Будет ли SSR влиять на уровень согласованности?
    *  SSR не влияет на согласованность запроса. Запросы выполняются повторно на сервере, если их частота ограничена (ошибка 429). 
*  Влияет ли SSR на любой тип ошибок, которые может получить клиент?
    *  Нет, SSR влияет только на ошибки ограничения частоты (429s), пытаясь повторить их на стороне сервера. Эта функция предотвращает обработку ошибок ограничения частоты в клиентском приложении. Все [остальные ошибки](mongodb-troubleshoot.md) будут отправлены клиенту. 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об устранении распространенных ошибок см. в этой статье:

* [Устранение распространенных неполадок в API Azure Cosmos DB для MongoDB](mongodb-troubleshoot.md)

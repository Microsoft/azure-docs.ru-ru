---
title: Устранение неполадок Azure Cosmos DB запрещенных исключений
description: Узнайте, как диагностировать и исправить запрещенные исключения.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 02/05/2021
ms.author: maquaran
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 0854e5d2c323df695460908c45714673756328c2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99971914"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-forbidden-exceptions"></a>Диагностика и устранение неполадок Azure Cosmos DB запрещенных исключений
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Код состояния HTTP 403 представляет выполнение запроса запрещено.

## <a name="firewall-blocking-requests"></a>Блокирующие запросы брандмауэра
В этом сценарии обычно отображаются ошибки, подобные приведенным ниже.

```
Request originated from client IP {...} through public internet. This is blocked by your Cosmos DB account firewall settings.
```

```
Request is blocked. Please check your authorization token and Cosmos DB account firewall settings
```

### <a name="solution"></a>Решение
Убедитесь, что текущие [Параметры брандмауэра](how-to-configure-firewall.md) указаны правильно и содержат IP-адреса или сети, из которых вы пытаетесь подключиться.
Если вы недавно обновили их, помните, что для применения изменений может потребоваться до **15 минут**.

## <a name="next-steps"></a>Дальнейшие действия
* Настройте [брандмауэр IP-адресов](how-to-configure-firewall.md).
* Настройка доступа из [виртуальных сетей](how-to-configure-vnet-service-endpoint.md).
* Настройка доступа из [частных конечных точек](how-to-configure-private-endpoints.md).

---
title: Устранение неполадок в Azure Cosmos DB недоступности службы
description: Узнайте, как диагностировать и исправлять исключения, недоступные для службы Azure Cosmos DB.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 08/06/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: d19d30c03412ba7212211b30646acb50c3f55ece
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93340035"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-service-unavailable-exceptions"></a>Диагностика и устранение неполадок, связанных с Azure Cosmos DB недоступности службы
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Пакету SDK не удалось подключиться к Azure Cosmos DB.

## <a name="troubleshooting-steps"></a>Действия по устранению неполадок
Следующий список содержит известные причины и решения для исключений, недоступных для службы.

### <a name="the-required-ports-are-being-blocked"></a>Требуемые порты блокируются
Убедитесь, что включены все [необходимые порты](sql-sdk-connection-modes.md#service-port-ranges) .

### <a name="client-side-transient-connectivity-issues"></a>Временные проблемы подключения на стороне клиента
Исключения, недоступные для службы, могут быть вызваны временными проблемами подключения, вызывающими тайм-ауты. Как правило, трассировка стека, связанная с этим сценарием, будет содержать `TransportException` ошибку. Пример:

```C#
TransportException: A client transport error occurred: The request timed out while waiting for a server response. 
(Time: xxx, activity ID: xxx, error code: ReceiveTimeout [0x0010], base error: HRESULT 0x80131500
```

Чтобы устранить [неполадки, выполните действия по устранению неполадок с запросом времени ожидания](troubleshoot-dot-net-sdk-request-timeout.md#troubleshooting-steps) .

### <a name="service-outage"></a>Простой службы
Проверьте [состояние Azure](https://status.azure.com/status) , чтобы узнать, существует ли текущая ошибка.


## <a name="next-steps"></a>Дальнейшие действия
* [Диагностика и устранение неполадок](troubleshoot-dot-net-sdk.md) при использовании пакета SDK для Azure Cosmos DB .NET.
* Ознакомьтесь с рекомендациями по производительности для [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) и [.NET v2](performance-tips.md).
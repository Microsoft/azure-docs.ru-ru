---
title: Устранение неполадок с Azure Cosmos DB недоступности службы в пакете SDK для Java v4
description: Узнайте, как диагностировать и исправлять исключения, недоступные для Azure Cosmos DB службы, с помощью пакета SDK для Java v4.
author: kushagrathapar
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 10/28/2020
ms.author: kuthapar
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 03c899307b00010ca87b279ed720b92946d3673c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93340096"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-java-v4-sdk-service-unavailable-exceptions"></a>Диагностика и устранение неполадок Azure Cosmos DB служба пакета SDK для Java версии 4 недоступна
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
Пакету SDK для Java v4 не удалось подключиться к Azure Cosmos DB.

## <a name="troubleshooting-steps"></a>Действия по устранению неполадок
Следующий список содержит известные причины и решения для исключений, недоступных для службы.

### <a name="the-required-ports-are-being-blocked"></a>Требуемые порты блокируются
Убедитесь, что включены все [необходимые порты](sql-sdk-connection-modes.md#service-port-ranges) .

### <a name="client-side-transient-connectivity-issues"></a>Временные проблемы подключения на стороне клиента
Исключения, недоступные для службы, могут быть вызваны временными проблемами подключения, вызывающими тайм-ауты. Как правило, трассировка стека, связанная с этим сценарием, будет содержать `ServiceUnavailableException` ошибку с диагностическими сведениями. Пример:

```java
Exception in thread "main" ServiceUnavailableException{userAgent=azsdk-java-cosmos/4.6.0 Linux/4.15.0-1096-azure JRE/11.0.8, error=null, resourceAddress='null', requestUri='null', statusCode=503, message=Service is currently unavailable, please retry after a while. If this problem persists please contact support.: Message: "" {"diagnostics"}
```

Чтобы устранить [неполадки, выполните действия по устранению неполадок с запросом времени ожидания](troubleshoot-request-timeout-java-sdk-v4-sql.md#troubleshooting-steps) .

### <a name="service-outage"></a>Простой службы
Проверьте [состояние Azure](https://status.azure.com/status) , чтобы узнать, существует ли текущая ошибка.


## <a name="next-steps"></a>Дальнейшие действия
* [Диагностика и устранение неполадок](troubleshoot-java-sdk-v4-sql.md) при использовании пакета SDK для Java версии 4 Azure Cosmos DB.
* Сведения о рекомендациях по производительности для [пакета SDK для Java v4](performance-tips-java-sdk-v4-sql.md).
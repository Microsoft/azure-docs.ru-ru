---
title: Руководство по обработке ошибок и повторным попыткам в функциях Azure
description: Узнайте, как управлять ошибками и событиями повтора в функциях Azure со ссылками на определенные ошибки привязки.
author: craigshoemaker
ms.topic: conceptual
ms.date: 10/01/2020
ms.author: cshoe
ms.openlocfilehash: e52babd5b0612b4590f1422a941b25ba30c3f169
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93284455"
---
# <a name="azure-functions-error-handling-and-retries"></a>Обработка ошибок и повторных попыток в функциях Azure

Обработка ошибок в функциях Azure важна, чтобы избежать потери данных, пропущенных событий и мониторинга работоспособности приложения.

В этой статье описываются общие стратегии обработки ошибок и ссылки на ошибки привязки.

## <a name="handling-errors"></a>Обработка ошибок

[!INCLUDE [bindings errors intro](../../includes/functions-bindings-errors-retries.md)]

## <a name="binding-error-codes"></a>Коды ошибок привязки

При интеграции со службами Azure ошибки могут исходить из API-интерфейсов базовых служб. Сведения об ошибках, связанных с привязками, доступны в разделе **исключения и коды возврата** в следующих статьях:

+ [Azure Cosmos DB](functions-bindings-cosmosdb.md#exceptions-and-return-codes)

+ [Хранилище BLOB-объектов](functions-bindings-storage-blob-output.md#exceptions-and-return-codes)

+ [Центры событий](functions-bindings-event-hubs-output.md#exceptions-and-return-codes)

+ [Центры Интернета вещей](functions-bindings-event-iot-output.md#exceptions-and-return-codes)

+ [Центры уведомлений](functions-bindings-notification-hubs.md#exceptions-and-return-codes)

+ [Хранилище очередей](functions-bindings-storage-queue-output.md#exceptions-and-return-codes)

+ [Служебная шина](functions-bindings-service-bus-output.md#exceptions-and-return-codes)

+ [Хранилище таблиц](functions-bindings-storage-table-output.md#exceptions-and-return-codes)

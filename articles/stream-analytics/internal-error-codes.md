---
title: Устранение неполадок с использованием кодов ошибок в Azure Stream Analytics
description: Устранить неполадки в работе Azure Stream Analytics помогут коды внутренних ошибок.
ms.author: sidram
author: sidramadoss
ms.topic: troubleshooting
ms.date: 05/07/2020
ms.service: stream-analytics
ms.openlocfilehash: ddea74997850ff080f6c82d55d1be3cfb851deda
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98019386"
---
# <a name="azure-stream-analytics-internal-error-codes"></a>Коды внутренних ошибок в Azure Stream Analytics

Журналы действий и журналы ресурсов можно использовать для отладки и устранения ошибок при выполнении заданий в Azure Stream Analytics. В этой статье приводится описание всех кодов внутренних ошибок. Внутренними называются общие ошибки, которые возникают на платформе Stream Analytics, когда она не может определить, произошла внутренняя ошибка доступности или ошибка в системе.

## <a name="cosmosdboutputbatchsizetoolarge"></a>CosmosDBOutputBatchSizeTooLarge

* **Причина.** Слишком большой размер пакета, используемого для записи в Cosmos DB. 
* **Рекомендация**. Повторите попытку с пакетом меньшего размера.

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок с входящими подключениями](stream-analytics-troubleshoot-input.md)
* [Устранение неполадок с выходными данными в Azure Stream Analytics](stream-analytics-troubleshoot-output.md)
* [Устранение неполадок с запросами в Azure Stream Analytics](stream-analytics-troubleshoot-query.md)
* [Устранение неполадок в Azure Stream Analytics с помощью журналов ресурсов](stream-analytics-job-diagnostic-logs.md)
* [Ошибки данных в Azure Stream Analytics](data-errors.md)
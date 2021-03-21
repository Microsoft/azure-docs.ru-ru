---
title: Коды ошибок данных — Azure Stream Analytics
description: Устранить неполадки с Azure Stream Analytics помогут коды ошибок данных.
ms.author: sidram
author: sidramadoss
ms.topic: troubleshooting
ms.date: 05/07/2020
ms.service: stream-analytics
ms.openlocfilehash: 33e617c8c3589d76d649dd2ea2236a4247b12500
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98016496"
---
# <a name="azure-stream-analytics-data-error-codes"></a>Коды ошибок данных Azure Stream Analytics

Журналы действий и журналы ресурсов можно использовать для отладки и устранения ошибок при выполнении заданий в Azure Stream Analytics. В этой статье приводится описание всех кодов ошибок данных. Ошибки данных возникают при наличии в потоке недопустимых данных, таких как непредвиденная схема записи.

## <a name="inputdeserializationerror"></a>InputDeserializationError

* **Причина.** Произошла ошибка при десериализации входных данных.

## <a name="inputeventtimestampnotfound"></a>InputEventTimestampNotFound

* **Причина.** Stream Analytics не может получить метку времени для ресурса. 

## <a name="inputeventtimestampbyovervaluenotfound"></a>InputEventTimestampByOverValueNotFound

* **Причина.** Stream Analytics не может получить значение `TIMESTAMP BY OVER COLUMN`.

## <a name="inputeventlatebeyondthreshold"></a>InputEventLateBeyondThreshold

* **Причина.** Значение времени отправки события превышает заданное ограничение.

## <a name="inputeventearlybeyondthreshold"></a>InputEventEarlyBeyondThreshold

* **Причина.** Значение времени поступления события ввода меньше порогового значения метки времени приложения события ввода.

## <a name="azurefunctionmessagesizeexceeded"></a>AzureFunctionMessageSizeExceeded

* **Причина.** Объем сообщений, выводимых в Функции Azure, превышает предельный размер.

## <a name="eventhuboutputrecordexceedssizelimit"></a>EventHubOutputRecordExceedsSizeLimit

* **Причина.** Превышен максимальный размер выходной записи при записи в концентратор событий.

## <a name="cosmosdboutputinvalidid"></a>CosmosDBOutputInvalidId

* **Причина.** Недопустимое значение или тип определенного столбца.
* **Рекомендация**. Добавьте уникальные непустые строки, длина которых не превышает 255 символов.

## <a name="cosmosdboutputinvalididcharacter"></a>CosmosDBOutputInvalidIdCharacter

* **Причина.** Идентификатор документа выходной записи содержит недопустимый символ.

## <a name="cosmosdboutputmissingid"></a>CosmosDBOutputMissingId

* **Причина.** Выходная запись не содержит столбец \[id], используемый как свойство первичного ключа.

## <a name="cosmosdboutputmissingidcolumn"></a>CosmosDBOutputMissingIdColumn

* **Причина.** Выходная запись не содержит свойство идентификатора документа. 
* **Рекомендация**. Убедитесь, что в выходных данных запроса присутствует столбец с уникальной непустой строкой, длина которого не превышает 255 символов.

## <a name="cosmosdboutputmissingpartitionkey"></a>CosmosDBOutputMissingPartitionKey

* **Причина.** Выходная запись не содержит столбец, используемый как свойство ключа секции.

## <a name="cosmosdboutputsinglerecordtoolarge"></a>CosmosDBOutputSingleRecordTooLarge

* **Причина.** Слишком большой объем данных записи в Cosmos DB.

## <a name="sqldatabaseoutputdataerror"></a>SQLDatabaseOutputDataError

* **Причина.** Stream Analytics не может записать события в Базу данных SQL из-за ошибок в данных.

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок с входящими подключениями](stream-analytics-troubleshoot-input.md)
* [Устранение неполадок с выходными данными в Azure Stream Analytics](stream-analytics-troubleshoot-output.md)
* [Устранение неполадок с запросами в Azure Stream Analytics](stream-analytics-troubleshoot-query.md)
* [Устранение неполадок в Azure Stream Analytics с помощью журналов ресурсов](stream-analytics-job-diagnostic-logs.md)
* [Ошибки данных в Azure Stream Analytics](data-errors.md)

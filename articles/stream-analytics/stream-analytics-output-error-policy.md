---
title: Политики обработки ошибок вывода в Azure Stream Analytics
description: Узнайте о том, какие политики обработки ошибок вывода доступны в Azure Stream Analytics.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/04/2018
ms.custom: seodec18
ms.openlocfilehash: 19d762a55127af34e84185b11518aa6584acb5bd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98012416"
---
# <a name="azure-stream-analytics-output-error-policy"></a>Политика обработки ошибок вывода в Azure Stream Analytics
В этой статье описываются политики обработки ошибок вывода, которые могут быть настроены в Azure Stream Analytics.

Политики обработки ошибок вывода применяются только к ошибкам преобразования данных, возникающим, когда выходное событие, созданное заданием Stream Analytics, не соответствует схеме целевого приемника. Это политику можно настроить, выбрав параметр **Повторить** или **Удалить**. На портале Azure в задании Stream Analytics перейдите к разделу **Настройка** и щелкните **Политика обработки ошибок**, чтобы сделать выбор.

![Расположение политики обработки ошибок вывода в Azure Stream Analytics](./media/stream-analytics-output-error-policy/stream-analytics-error-policy-locate.png)


## <a name="retry"></a>Повторить попытку
Когда происходит ошибка, Azure Stream Analytics бесконечно повторяет попытку записи события, пока запись не будет выполнена. Время ожидания для повторных попыток не задано. В конечном итоге обработку всех последующих событий блокирует событие, которое Azure Stream Analytics безуспешно пытается записать. Это политика обработки ошибок вывода по умолчанию.

## <a name="drop"></a>Drop
Azure Stream Analytics будет удалять все выходные события, вызывающее ошибку преобразования данных. Восстановить удаленные события для повторной обработки позже невозможно.


Для всех временных ошибок (например, сбоев сети) выполняется повторная попытка независимо от конфигурации политики обработки ошибок вывода.


## <a name="next-steps"></a>Дальнейшие действия
[Руководство по устранению неполадок для Azure Stream Analytics](./stream-analytics-troubleshoot-query.md)
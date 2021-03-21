---
title: API длительных операций Azure Maps
description: Узнайте о длительной асинхронной фоновой обработке в Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: f5fb7c8059c8b98e8ec514a4159e96f48db7b1ad
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96906205"
---
# <a name="creator-preview-long-running-operation-api"></a>API Long-Running операции создателя (Предварительная версия)

> [!IMPORTANT]
> Службы Создателя Azure Maps в настоящее время предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Некоторые API в Azure Maps используют [шаблон асинхронного Request-Reply](/azure/architecture/patterns/async-request-reply). Этот шаблон позволяет Azure Maps предоставлять высокодоступные и быстро реагирующие службы. В этой статье описывается характерная для Azure Maps реализация длительной асинхронной фоновой обработки.

## <a name="submitting-a-request"></a>Отправка запроса

Клиентское приложение запускает длительную операцию с помощью синхронного вызова к API HTTP. Как правило, такой вызов поступает в виде HTTP-запроса POST. При успешном создании асинхронной рабочей нагрузки API возвращает код состояния HTTP `202`, сообщающий о том, что запрос принят. Этот ответ содержит заголовок `Location`, указывающий на конечную точку, которую клиент может опрашивать для проверки состояния длительной операции.

### <a name="example-of-a-success-response"></a>Пример ответа, сообщающего об успешном принятии и обработке запроса

```HTTP
Status: 202 Accepted
Location: https://atlas.microsoft.com/service/operations/{operationId}

```

Если вызов не проходит проверку, API вместо этого возвращает ответ HTTP `400`, сообщающий о недопустимом запросе. Текст ответа содержит для клиента дополнительные сведения о том, почему запрос является недопустимым.

### <a name="monitoring-the-operation-status"></a>Мониторинг состояния операции

Чтобы проверить состояние длительной операции, можно опросить конечную точку расположения, указанную в заголовке принятого ответа. Текст ответа из запроса о состоянии операции всегда будет содержать свойства `status` и `createdDateTime`. Свойство `status` представляет текущее состояние длительной операции. Возможные значения включают `"NotStarted"`, `"Running"`, `"Succeeded"` и `"Failed"`. Свойство `createdDateTime` представляет время выполнения первоначального запроса для запуска длительной операции. Если состояние имеет значение `"NotStarted"` или `"Running"`, в ответе также будет указан заголовок `Retry-After`. Заголовок `Retry-After`, представляющий время в секундах, позволяет определить, когда следует выполнить следующий опрашивающий вызов к API состояния операции.

### <a name="example-of-running-a-status-response"></a>Пример ответа с информацией о состоянии

```HTTP
Status: 200 OK
Retry-After: 30
{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Running"
}
```

## <a name="handling-operation-completion"></a>Обработка завершения операций

После завершения длительной операции состояние ответа будет `"Succeeded"` или `"Failed"`. Если новый ресурс был создан в ходе длительной операции, ответ с информацией об успешном принятии и обработке запроса будет содержать код состояния HTTP `201 Created`. Ответ также будет содержать заголовок `Location`, указывающий на метаданные ресурса. Если новый ресурс не был создан, ответ будет содержать код состояния HTTP `200 OK`. Хотя в случае ошибки ответ также будет содержать код состояния `200 OK`, в тексте также будет присутствовать свойство `error`. Данные об ошибках соответствуют спецификации ошибок OData.

### <a name="example-of-success-response"></a>Пример ответа с информацией об успешном выполнении

```HTTP
Status: 201 Created
Location: "https://atlas.microsoft.com/tileset/{tileset-id}"
 {
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Succeeded",
    "resourceLocation": "https://atlas.microsoft.com/tileset/{tileset-id}"
}
```

### <a name="example-of-failure-response"></a>Пример ответа с информацией об ошибке при принятии и обработке запроса

```HTTP
Status: 200 OK

{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Failed",
    "error": {
        "code": "InvalidFeature",
        "message": "The provided feature is invalid.",
        "details": {
            "code": "NoGeometry",
            "message": "No geometry was provided with the feature."
        }
    }
}
```
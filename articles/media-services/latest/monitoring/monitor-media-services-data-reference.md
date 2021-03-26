---
title: Справочник по мониторингу данных служб мультимедиа
description: Важные справочные материалы, необходимые при мониторинге служб мультимедиа
author: IngridAtMicrosoft
ms.author: inhenkel
manager: femila
ms.topic: reference
ms.service: media-services
ms.custom: subject-monitoring
ms.date: 03/17/2021
ms.openlocfilehash: 66fce608515d16c5418ddd18e00319a3cbf088f7
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105609094"
---
# <a name="monitoring-media-services-data-reference"></a>Справочник по мониторингу данных служб мультимедиа

В этой статье рассматриваются данные, полезные для мониторинга служб мультимедиа. Дополнительные сведения обо всех метриках платформы, поддерживаемых в Azure Monitor, см. в статье [Поддерживаемые метрики в Azure Monitor](../../../azure-monitor/essentials/metrics-supported.md).

## <a name="media-services-metrics"></a>Метрики служб мультимедиа

Метрики собираются через регулярные промежутки времени независимо от того, изменяется ли значение. Метрики удобно использовать для создания оповещений, так можно часто делать их выборку и быстро создавать оповещения с использованием относительно простой логики.

Службы мультимедиа поддерживают метрики мониторинга для следующих ресурсов:

* Учетная запись
* Конечная точка потоковой передачи

### <a name="account"></a>Учетная запись

Вы можете отслеживать следующие метрики учетной записи.

|Имя метрики|Отображаемое имя|Описание|
|---|---|---|
|AssetCount|Число ресурсов|Активы в вашей учетной записи.|
|AssetQuota|Квота ресурсов|Квота ресурса в вашей учетной записи.|
|AssetQuotaUsedPercentage|Процент используемой квоты ресурсов|Процент использования квоты актива.|
|ContentKeyPolicyCount|Число политик ключей содержимого|Политики ключей содержимого в вашей учетной записи.|
|ContentKeyPolicyQuota|Квота политик ключей содержимого|Политики ключей содержимого. квота для учетной записи.|
|ContentKeyPolicyQuotaUsedPercentage|Процент использования квоты политик ключей содержимого|Процентная доля квоты политики ключей содержимого, которая уже использовалась.|
|StreamingPolicyCount|Число политик потоковой передачи|Политики потоковой передачи в учетной записи.|
|StreamingPolicyQuota|Квота политик потоковой передачи|Квота потоковой передачи политик в вашей учетной записи.|
|StreamingPolicyQuotaUsedPercentage|Процент использования квоты политик потоковой передачи|Процент уже использованной квоты политики потоковой передачи.|

Также следует проверить [квоты и ограничения учетной записи](../limits-quotas-constraints.md).

### <a name="streaming-endpoint"></a>Конечная точка потоковой передачи

Поддерживаются следующие метрики [конечных точек потоковой передачи](/rest/api/media/streamingendpoints) служб мультимедиа:

|Имя метрики|Отображаемое имя|Описание|
|---|---|---|
|Requests|Requests|Предоставляет общее число HTTP-запросов, обслуживаемых конечной точкой потоковой передачи.|
|Исходящие|Исходящие|Всего байтов исходящего трафика в минуту на конечную точку потоковой передачи.|
|SuccessE2ELatency|Сквозная задержка успешного запроса|Период времени, в течение которого конечная точка потоковой передачи получает запрос на момент отправки последнего байта ответа.|
|Загрузка ЦП| | Загрузка ЦП для конечных точек потоковой передачи уровня "Премиум". Эти данные недоступны для стандартных конечных точек потоковой передачи. |
|Пропускная способность для исходящих данных | | Пропускная способность исходящего трафика в битах в секунду.|

## <a name="metric-dimensions"></a>Измерения метрик

Дополнительные сведения об измерениях метрик см. в разделе [Многомерные метрики](../../../azure-monitor/essentials/data-platform-metrics.md#multi-dimensional-metrics).

<!--**PLACEHOLDER** for dimensions table.-->

## <a name="resource-logs"></a>Журналы ресурсов

## <a name="media-services-diagnostic-logs"></a>Журналы диагностики служб мультимедиа

Журналы диагностики предоставляют обширные и часто встречающиеся данные о работе ресурсов Azure. Дополнительные сведения см. в статье получение [и использование данных журнала из ресурсов Azure](../../../azure-monitor/essentials/platform-logs-overview.md).

Службы мультимедиа поддерживают следующие журналы диагностики:

* Доставка ключей

### <a name="key-delivery"></a>Доставка ключей

|Имя|Описание|
|---|---|
|Запрос службы доставки ключей|Журналы, отображающие сведения о запросе службы доставки ключей. Дополнительные сведения см. в разделе [схемы](monitor-media-services-data-reference.md).|

## <a name="schemas"></a>Схемы

Подробное описание схемы журналов диагностики верхнего уровня см. в статье [Поддерживаемые службы, схемы и категории для журналов диагностики Azure](../../../azure-monitor/essentials/resource-logs-schema.md).

## <a name="key-delivery-log-schema-properties"></a>Свойства схемы журнала доставки ключей

Эти свойства относятся к схеме журнала доставки ключей.

|Имя|Описание|
|---|---|
|keyId|ИДЕНТИФИКАТОР запрошенного ключа.|
|keyType|Может принимать одно из следующих значений: "Clear" (без шифрования), "FairPlay", "PlayReady" или "Widevine".|
|policyName|Azure Resource Manager имя политики.|
|tokenType|Тип токена.|
|statusMessage|Сообщение состояния.|

### <a name="example"></a>Пример

Свойства схемы запросов на доставку ключей.

```json
{
    "time": "2019-01-11T17:59:10.4908614Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Succeeded",
    "resultSignature": "OK",
    "durationMs": 315,
    "identity": {
        "authorization": {
            "issuer": "http://testacs",
            "audience": "urn:test"
        },
        "claims": {
            "urn:microsoft:azure:mediaservices:contentkeyidentifier": "3321e646-78d0-4896-84ec-c7b98eddfca5",
            "iss": "http://testacs",
            "aud": "urn:test",
            "exp": "1547233138"
        }
    },
    "level": "Informational",
    "location": "uswestcentral",
    "properties": {
        "requestId": "b0243468-d8e5-4edf-a48b-d408e1661050",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "JWT",
        "statusMessage": "OK"
    }
} 
```

```json
 {
    "time": "2019-01-11T17:59:33.4676382Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Failed",
    "resultSignature": "Unauthorized",
    "durationMs": 2,
    "level": "Error",
    "location": "uswestcentral",
    "properties": {
        "requestId": "875af030-b77c-416b-b7e1-58f23ebec182",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "None",
        "statusMessage": "No token present in authorization header or URL."
    }
} 
```

>[!NOTE]
> Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [monitoring-next-steps](../includes/monitoring-next-steps.md)]

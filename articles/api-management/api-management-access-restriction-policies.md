---
title: Политики ограничения доступа в службе управления API Azure | Документация Майкрософт
description: Сведения о политиках ограничения, доступных для использования в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
ms.assetid: 034febe3-465f-4840-9fc6-c448ef520b0f
ms.service: api-management
ms.topic: article
ms.date: 02/26/2021
ms.author: apimpm
ms.openlocfilehash: 882d96271b6976db1ffc0dde181d5699c5cc27de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101688252"
---
# <a name="api-management-access-restriction-policies"></a>Политики ограничения доступа в службе управления API

В этой статье рассматриваются приведенные ниже политики управления API. Дополнительные сведения о добавлении и настройке политик см. в статье о [политиках в управлении API](./api-management-policies.md).

## <a name="access-restriction-policies"></a><a name="AccessRestrictionPolicies"></a> Политики ограничения доступа

-   [Check HTTP-заголовок](#CheckHTTPHeader) — обеспечивает принудительное существование и (или) значения заголовка HTTP.
-   [Ограничение частоты вызовов по подписке](#LimitCallRate) — предотвращает пики использования API, ограничивая частоту вызовов для каждой подписки.
-   [Ограничение частоты вызовов по ключу](#LimitCallRateByKey) — предотвращает пики использования API, ограничивая частоту вызовов по ключу.
-   [Ограничение IP-адресов вызывающих объектов](#RestrictCallerIPs) – фильтрует (разрешает или запрещает) вызовы с конкретных IP-адресов и/или диапазонов адресов.
-   [Задание квоты использования по подписке](#SetUsageQuota) — позволяет принудительно устанавливать возобновляемую или действующую в течение срока службы квоту на число вызовов и (или) квоту пропускной способности для каждой подписки.
-   [Задание квоты использования по ключу](#SetUsageQuotaByKey) — позволяет принудительно устанавливать возобновляемую или действующую в течение срока службы квоту на число вызовов и (или) квоту пропускной способности для каждого ключа.
-   [Проверка JWT](#ValidateJWT) – обеспечивает принудительное задание и проверку JWT, извлеченного из заданного заголовка HTTP или параметра запроса.

> [!TIP]
> Политики ограничений доступа можно использовать в разных областях для разных целей. Например, можно защитить весь API с помощью проверки подлинности AAD, применив `validate-jwt` политику на уровне API, или применить ее на уровне операций API и использовать `claims` для более детального управления.

## <a name="check-http-header"></a><a name="CheckHTTPHeader"></a> Проверить заголовок HTTP

Используйте политику `check-header`, чтобы запрос содержал заданный заголовок HTTP. При необходимости можно проверить, содержит ли заголовок определенное значение, или проверить диапазон допустимых значений. При сбое проверки политика завершает обработку запроса, после чего возвращает код состояния HTTP и сообщение об ошибке, указанное в политике.

### <a name="policy-statement"></a>Правило политики

```xml
<check-header name="header name" failed-check-httpcode="code" failed-check-error-message="message" ignore-case="true">
    <value>Value1</value>
    <value>Value2</value>
</check-header>
```

### <a name="example"></a>Пример

```xml
<check-header name="Authorization" failed-check-httpcode="401" failed-check-error-message="Not authorized" ignore-case="false">
    <value>f6dc69a089844cf6b2019bae6d36fac8</value>
</check-header>
```

### <a name="elements"></a>Элементы

| Название         | Описание                                                                                                                                   | Обязательно |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| check-header | Корневой элемент.                                                                                                                                 | Да      |
| value        | Допустимое значение заголовка HTTP. Если указано несколько элементов value и одно из значений совпадает, проверка считается успешной. | Нет       |

### <a name="attributes"></a>Атрибуты

| Имя                       | Описание                                                                                                                                                            | Обязательно | По умолчанию |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| failed-check-error-message | Сообщение об ошибке, возвращаемое в тексте ответа HTTP, если заголовок не существует или имеет недопустимое значение. Это сообщение должно содержать правильно экранированные специальные символы. | Да      | Н/Д     |
| failed-check-httpcode      | Код состояния HTTP, который возвращается, если заголовок не существует или имеет недопустимое значение.                                                                                        | Да      | Н/Д     |
| header-name                | Имя заголовка HTTP для проверки.                                                                                                                                  | Да      | Н/Д     |
| ignore-case                | Можно задать значение true или false. Если задано значение true и значение заголовка сравнивается с набором допустимых значений, регистр игнорируется.                                    | Да      | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound.

-   **Области политики:** все области.

## <a name="limit-call-rate-by-subscription"></a><a name="LimitCallRate"></a> Ограничить частоту вызовов по подписке

Политика `rate-limit` предотвращает пики использования API для каждой подписки, ограничивая частоту вызовов до указанного числа за определенный период времени. При превышении частоты вызовов вызывающий объект получает `429 Too Many Requests` код состояния ответа.

> [!IMPORTANT]
> Эту политику можно использовать для каждого документа политики только один раз.
>
> Ни в одном из атрибутов этой политики не могут использоваться [выражения политики](api-management-policy-expressions.md).

> [!CAUTION]
> Из-за распределенного характера архитектуры регулирования ограничение частоты никогда не является полностью точным. Разница между настроенными и реальным количеством допустимых запросов зависит от объема запроса и скорости, задержки серверной части и других факторов.

> [!NOTE]
> Сведения о разнице между ограничениями скорости и квотами [см. в разделе пределы скорости и квоты.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Правило политики

```xml
<rate-limit calls="number" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" 
        retry-after-header-name="header name" 
        retry-after-variable-name="policy expression variable name"
        remaining-calls-header-name="header name"  
        remaining-calls-variable-name="policy expression variable name"
        total-calls-header-name="header name"/>
    </api>
</rate-limit>
```

### <a name="example"></a>Пример

В следующем примере ограничение скорости для каждой подписки составляет 20 вызовов в течение 90 секунд. После каждого выполнения политики оставшиеся вызовы, разрешенные за период времени, сохраняются в переменной `remainingCallsPerSubscription` .

```xml
<policies>
    <inbound>
        <base />
        <rate-limit calls="20" renewal-period="90" remaining-calls-variable-name="remainingCallsPerSubscription"/>
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Элементы

| Название       | Описание                                                                                                                                                                                                                                                                                              | Обязательно |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| rate-limit | Корневой элемент.                                                                                                                                                                                                                                                                                            | Да      |
| api        | Добавьте один или несколько этих элементов, чтобы установить ограничение частоты вызовов для API в пределах продукта. Ограничения частоты вызовов продукта и API применяются раздельно. Ссылаться на API можно с помощью `name` или `id`. Если указаны оба атрибута, `id` будет использоваться, а `name` — игнорироваться.                    | Нет       |
| Операция  | Добавьте один или несколько этих элементов, чтобы применить ограничение частоты вызовов для операций в API. Ограничения частоты вызовов продукта, API и операции применяются раздельно. Ссылаться на операцию можно с помощью `name` или `id`. Если указаны оба атрибута, `id` будет использоваться, а `name` — игнорироваться. | Нет       |

### <a name="attributes"></a>Атрибуты

| Имя           | Описание                                                                                           | Обязательно | По умолчанию |
| -------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| name           | Имя API, для которого применяется ограничение частоты.                                                | Да      | Н/Д     |
| calls          | Максимальное общее число вызовов, разрешенное за интервал времени, указанный в параметре `renewal-period` . | Да      | Н/Д     |
| renewal-period | Длина скользящего окна в секундах, в течение которого количество разрешенных запросов должно быть не больше значения, указанного в параметре `calls` .                                              | Да      | Н/Д     |
| Retry-After-Header-Name    | Имя заголовка ответа, значение которого является рекомендуемым интервалом повтора в секундах после превышения указанного числа вызовов. |  Нет | Н/Д  |
| Retry-After-переменная-name    | Имя переменной выражения политики, в которой хранится рекомендуемый интервал повтора в секундах после превышения указанного числа вызовов. |  Нет | Н/Д  |
| остальные-Calls-Header-Name    | Имя заголовка ответа, значение которого после каждого выполнения политики равно количеству оставшихся вызовов, допустимым для интервала времени, указанного в `renewal-period` . |  Нет | Н/Д  |
| Оставшаяся — вызовы-переменная-имя    | Имя переменной выражения политики, которая после каждого выполнения политики хранит количество оставшихся вызовов, допустимых для интервала времени, указанного в `renewal-period` . |  Нет | Н/Д  |
| Total-Calls-Header-Name    | Имя заголовка ответа, значение которого равно значению, указанному в параметре `calls` . |  Нет | Н/Д  |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.

-   **Области политики:** продукт, API, операция

## <a name="limit-call-rate-by-key"></a><a name="LimitCallRateByKey"></a> Ограничение частоты вызовов по ключу

> [!IMPORTANT]
> Эта функция недоступна в ценовой категории **Потребление** управления API.

Политика `rate-limit-by-key` предотвращает пики использования API для каждого ключа, ограничивая частоту вызовов до указанного числа за определенный период времени. Ключ может содержать произвольное строковое значение и обычно указывается с помощью выражения политики. Чтобы указать, какие запросы следует учитывать для ограничения, можно добавить дополнительное условие увеличения. При превышении этой частоты вызовов вызывающий объект получает `429 Too Many Requests` код состояния ответа.

Дополнительные сведения и примеры этой политики см. в статье [Расширенное регулирование запросов с помощью управления API](./api-management-sample-flexible-throttling.md).

> [!CAUTION]
> Из-за распределенного характера архитектуры регулирования ограничение частоты никогда не является полностью точным. Разница между настроенными и реальным количеством допустимых запросов зависит от объема запроса и скорости, задержки серверной части и других факторов.

> [!NOTE]
> Сведения о разнице между ограничениями скорости и квотами [см. в разделе пределы скорости и квоты.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Правило политики

```xml
<rate-limit-by-key calls="number"
                   renewal-period="seconds"
                   increment-condition="condition"
                   counter-key="key value" 
                   retry-after-header-name="header name" retry-after-variable-name="policy expression variable name"
                   remaining-calls-header-name="header name"  remaining-calls-variable-name="policy expression variable name"
                   total-calls-header-name="header name"/> 

```

### <a name="example"></a>Пример

В следующем примере ограничение скорости в 10 вызовов в 60 секунд определяется IP-адресом вызывающей стороны. После каждого выполнения политики оставшиеся вызовы, разрешенные за период времени, сохраняются в переменной `remainingCallsPerIP` .

```xml
<policies>
    <inbound>
        <base />
        <rate-limit-by-key  calls="10"
              renewal-period="60"
              increment-condition="@(context.Response.StatusCode == 200)"
              counter-key="@(context.Request.IpAddress)"
              remaining-calls-variable-name="remainingCallsPerIP"/>
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Элементы

| Название              | Описание   | Обязательно |
| ----------------- | ------------- | -------- |
| ставка-ограничение по ключу | Корневой элемент. | Да      |

### <a name="attributes"></a>Атрибуты

| Имя                | Описание                                                                                           | Обязательно | По умолчанию |
| ------------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| calls               | Максимальное общее число вызовов, разрешенное в течение периода времени, указанного в `renewal-period`. | Да      | Н/Д     |
| counter-key         | Ключ, используемый для политики ограничения частоты.                                                             | Да      | Н/Д     |
| increment-condition | Логическое выражение, указывающее, должен ли запрос учитываться в направлении ( `true` ).        | Нет       | Н/Д     |
| renewal-period      | Длина скользящего окна в секундах, в течение которого количество разрешенных запросов должно быть не больше значения, указанного в параметре `calls` .                                           | Да      | Н/Д     |
| Retry-After-Header-Name    | Имя заголовка ответа, значение которого является рекомендуемым интервалом повтора в секундах после превышения указанного числа вызовов. |  Нет | Н/Д  |
| Retry-After-переменная-name    | Имя переменной выражения политики, в которой хранится рекомендуемый интервал повтора в секундах после превышения указанного числа вызовов. |  Нет | Н/Д  |
| остальные-Calls-Header-Name    | Имя заголовка ответа, значение которого после каждого выполнения политики равно количеству оставшихся вызовов, допустимым для интервала времени, указанного в `renewal-period` . |  Нет | Н/Д  |
| Оставшаяся — вызовы-переменная-имя    | Имя переменной выражения политики, которая после каждого выполнения политики хранит количество оставшихся вызовов, допустимых для интервала времени, указанного в `renewal-period` . |  Нет | Н/Д  |
| Total-Calls-Header-Name    | Имя заголовка ответа, значение которого равно значению, указанному в параметре `calls` . |  Нет | Н/Д  |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.

-   **Области политики:** все области.

## <a name="restrict-caller-ips"></a><a name="RestrictCallerIPs"></a> Ограничение IP-адресов вызывающих объектов

Политика `ip-filter` фильтрует (разрешает и запрещает) вызовы с конкретных IP-адресов и (или) диапазонов адресов.

### <a name="policy-statement"></a>Правило политики

```xml
<ip-filter action="allow | forbid">
    <address>address</address>
    <address-range from="address" to="address" />
</ip-filter>
```

### <a name="example"></a>Пример

В следующем примере политика разрешает только запросы, поступающие с одного IP-адреса или диапазона IP-адресов.

```xml
<ip-filter action="allow">
    <address>13.66.201.169</address>
    <address-range from="13.66.140.128" to="13.66.140.143" />
</ip-filter>
```

### <a name="elements"></a>Элементы

| Название                                      | Описание                                         | Обязательно                                                       |
| ----------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------- |
| ip-filter                                 | Корневой элемент.                                       | Да                                                            |
| address                                   | Указывает один IP-адрес для фильтрации.   | По крайней мере один элемент `address` или `address-range` является обязательным. |
| address-range from="address" to="address" | Указывает диапазон IP-адресов для фильтрации. | По крайней мере один элемент `address` или `address-range` является обязательным. |

### <a name="attributes"></a>Атрибуты

| Имя                                      | Описание                                                                                 | Обязательно                                           | По умолчанию |
| ----------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------- | ------- |
| address-range from="address" to="address" | Диапазон IP-адресов, для которого действует разрешение или запрет доступа.                                        | Обязательный атрибут, если используется элемент `address-range`. | Н/Д     |
| ip-filter action="allow &#124; forbid"    | Указывает, должны ли быть разрешены или запрещены вызовы для указанных IP-адресов и диапазонов. | Да                                                | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.
-   **Области политики:** все области.

## <a name="set-usage-quota-by-subscription"></a><a name="SetUsageQuota"></a> Задать квоты использования по подписке

Политика `quota` принудительно устанавливает возобновляемую или действующую в течение срока службы квоту на число вызовов и (или) квоту пропускной способности для каждой подписки.

> [!IMPORTANT]
> Эту политику можно использовать для каждого документа политики только один раз.
>
> Ни в одном из атрибутов этой политики не могут использоваться [выражения политики](api-management-policy-expressions.md).

> [!NOTE]
> Сведения о разнице между ограничениями скорости и квотами [см. в разделе пределы скорости и квоты.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Правило политики

```xml
<quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" />
    </api>
</quota>
```

### <a name="example"></a>Пример

```xml
<policies>
    <inbound>
        <base />
        <quota calls="10000" bandwidth="40000" renewal-period="3600" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Элементы

| Название      | Описание                                                                                                                                                                                                                                                                                  | Обязательно |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| quota     | Корневой элемент.                                                                                                                                                                                                                                                                                | Да      |
| api       | Добавьте один или несколько этих элементов, чтобы применить квоту вызовов для API в рамках продукта. Квоты для продукта и вызовов API применяются раздельно. Ссылаться на API можно с помощью `name` или `id`. Если указаны оба атрибута, `id` будет использоваться, а `name` — игнорироваться.                    | Нет       |
| Операция | Добавьте один или несколько этих элементов, чтобы применить квоту вызова для операций в API. Квоты для продукта, API и вызовов операций применяются раздельно. Ссылаться на операцию можно с помощью `name` или `id`. Если указаны оба атрибута, `id` будет использоваться, а `name` — игнорироваться. | Нет      |

### <a name="attributes"></a>Атрибуты

| Имя           | Описание                                                                                               | Обязательно                                                         | По умолчанию |
| -------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| name           | Имя API или операции, к которой применяется квота.                                             | Да                                                              | Н/Д     |
| bandwidth      | Максимальное общее число килобайтов, разрешенное в течение периода времени, указанного в `renewal-period`. | Необходимо указать атрибут `calls`, `bandwidth` или оба вместе. | Н/Д     |
| calls          | Максимальное общее число вызовов, разрешенное в течение периода времени, указанного в `renewal-period`.     | Необходимо указать атрибут `calls`, `bandwidth` или оба вместе. | Н/Д     |
| renewal-period | Период времени (в секундах), по окончании которого сбрасывается квота. Если для него задано значение `0` "бесконечно". | Да                                                              | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.
-   **Области политики:** product.

## <a name="set-usage-quota-by-key"></a><a name="SetUsageQuotaByKey"></a> Задание квоты использования по ключу

> [!IMPORTANT]
> Эта функция недоступна в ценовой категории **Потребление** управления API.

Политика `quota-by-key` принудительно устанавливает возобновляемую или действующую в течение срока службы квоту на число вызовов и (или) квоту пропускной способности для каждого ключа. Ключ может содержать произвольное строковое значение и обычно указывается с помощью выражения политики. Чтобы указать, какие запросы следует учитывать в квоте, можно добавить дополнительное условие увеличения. Если несколько политик будут увеличивать одно и то же значение ключа, оно увеличивается только один раз за запрос. При превышении частоты вызовов вызывающий объект получает `403 Forbidden` код состояния ответа.

Дополнительные сведения и примеры этой политики см. в статье [Расширенное регулирование запросов с помощью управления API](./api-management-sample-flexible-throttling.md).

> [!NOTE]
> Сведения о разнице между ограничениями скорости и квотами [см. в разделе пределы скорости и квоты.](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas)

### <a name="policy-statement"></a>Правило политики

```xml
<quota-by-key calls="number"
              bandwidth="kilobytes"
              renewal-period="seconds"
              increment-condition="condition"
              counter-key="key value" />

```

### <a name="example"></a>Пример

В следующем примере квота содержит ключ, состоящий из IP-адреса вызывающего объекта.

```xml
<policies>
    <inbound>
        <base />
        <quota-by-key calls="10000" bandwidth="40000" renewal-period="3600"
                      increment-condition="@(context.Response.StatusCode >= 200 && context.Response.StatusCode < 400)"
                      counter-key="@(context.Request.IpAddress)" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Элементы

| Название  | Описание   | Обязательно |
| ----- | ------------- | -------- |
| quota | Корневой элемент. | Да      |

### <a name="attributes"></a>Атрибуты

| Имя                | Описание                                                                                               | Обязательно                                                         | По умолчанию |
| ------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| bandwidth           | Максимальное общее число килобайтов, разрешенное в течение периода времени, указанного в `renewal-period`. | Необходимо указать атрибут `calls`, `bandwidth` или оба вместе. | Н/Д     |
| calls               | Максимальное общее число вызовов, разрешенное в течение периода времени, указанного в `renewal-period`.     | Необходимо указать атрибут `calls`, `bandwidth` или оба вместе. | Н/Д     |
| counter-key         | Ключ, используемый для политики квоты.                                                                      | Да                                                              | Н/Д     |
| increment-condition | Логическое выражение, указывающее, следует ли учитывать запрос для квоты (`true`).             | Нет                                                               | Н/Д     |
| renewal-period      | Период времени (в секундах), по окончании которого сбрасывается квота. Если для него задано значение `0` "бесконечно".                                                   | Да                                                              | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.
-   **Области политики:** все области.

## <a name="validate-jwt"></a><a name="ValidateJWT"></a> Проверка JWT

`validate-jwt`Политика применяет существование и допустимость веб-токена JSON (JWT), извлеченного из указанного заголовка HTTP или указанного параметра запроса.

> [!IMPORTANT]
> Для политики `validate-jwt` требуется, чтобы зарегистрированное утверждение `exp` было включено в маркер JWT, только если для атрибута `require-expiration-time` не задано значение `false`.
> Политика `validate-jwt` поддерживает алгоритмы подписывания HS256 и RS256. Для HS256 необходимо, чтобы ключ содержался внутри политики в кодировке Base64. Для RS256 ключ может предоставляться либо через конечную точку конфигурации с открытым ИДЕНТИФИКАТОРом, либо с помощью идентификатора отправленного сертификата, содержащего пару открытого ключа или модуля-экспоненты.
> `validate-jwt`Политика поддерживает маркеры, зашифрованные с помощью симметричных ключей, с использованием следующих алгоритмов шифрования: A128CBC-HS256, A192CBC-HS384, A256CBC-HS512.

### <a name="policy-statement"></a>Правило политики

```xml
<validate-jwt
    header-name="name of http header containing the token (use query-parameter-name attribute if the token is passed in the URL)"
    failed-validation-httpcode="http status code to return on failure"
    failed-validation-error-message="error message to return on failure"
    token-value="expression returning JWT token as a string"
    require-expiration-time="true|false"
    require-scheme="scheme"
    require-signed-tokens="true|false"
    clock-skew="allowed clock skew in seconds"
    output-token-variable-name="name of a variable to receive a JWT object representing successfully validated token">
  <openid-config url="full URL of the configuration endpoint, e.g. https://login.constoso.com/openid-configuration" />
  <issuer-signing-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </issuer-signing-keys>
  <decryption-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </decryption-keys>
  <audiences>
    <audience>audience string</audience>
    <!-- if there are multiple possible audiences, then add additional audience elements -->
  </audiences>
  <issuers>
    <issuer>issuer string</issuer>
    <!-- if there are multiple possible issuers, then add additional issuer elements -->
  </issuers>
  <required-claims>
    <claim name="name of the claim as it appears in the token" match="all|any" separator="separator character in a multi-valued claim">
      <value>claim value as it is expected to appear in the token</value>
      <!-- if there is more than one allowed values, then add additional value elements -->
    </claim>
    <!-- if there are multiple possible allowed values, then add additional value elements -->
  </required-claims>
</validate-jwt>

```

### <a name="examples"></a>Примеры

#### <a name="simple-token-validation"></a>Простая проверка маркера

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key>  <!-- signing key specified as a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>  <!-- audience is set to API Management host name -->
    </audiences>
    <issuers>
        <issuer>http://contoso.com/</issuer>
    </issuers>
</validate-jwt>
```

#### <a name="token-validation-with-rsa-certificate"></a>Проверка маркера с помощью сертификата RSA

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <issuer-signing-keys>
        <key certificate-id="my-rsa-cert" />  <!-- signing key specified as certificate ID, enclosed in double-quotes -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>  <!-- audience is set to API Management host name -->
    </audiences>
    <issuers>
        <issuer>http://contoso.com/</issuer>
    </issuers>
</validate-jwt>
```

#### <a name="azure-active-directory-token-validation"></a>Проверка маркеров Azure Active Directory

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration" />
    <audiences>
        <audience>25eef6e4-c905-4a07-8eb4-0d08d5df8b3f</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="azure-active-directory-b2c-token-validation"></a>Проверка токена Azure Active Directory B2C

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/b2c_1_signin/v2.0/.well-known/openid-configuration" />
    <audiences>
        <audience>d313c4e4-de5f-4197-9470-e509a2f0b806</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="authorize-access-to-operations-based-on-token-claims"></a>Авторизация доступа к операциям на основе утверждений маркера

В этом примере показано, как использовать политику [проверки JWT](api-management-access-restriction-policies.md#ValidateJWT) для авторизации доступа к операциям на основе значения утверждений токена.

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer" output-token-variable-name="jwt">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key> <!-- signing key is stored in a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>
    </audiences>
    <issuers>
        <issuer>contoso.com</issuer>
    </issuers>
    <required-claims>
        <claim name="group" match="any">
            <value>finance</value>
            <value>logistics</value>
        </claim>
    </required-claims>
</validate-jwt>
<choose>
    <when condition="@(context.Request.Method == "POST" && !((Jwt)context.Variables["jwt"]).Claims["group"].Contains("finance"))">
        <return-response>
            <set-status code="403" reason="Forbidden" />
        </return-response>
    </when>
</choose>
```

### <a name="elements"></a>Элементы

| Элемент             | Описание                                                                                                                                                                                                                                                                                                                                           | Обязательно |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-jwt        | Корневой элемент.                                                                                                                                                                                                                                                                                                                                         | Да      |
| audiences           | Содержит список допустимых утверждений audience, которые могут присутствовать в маркере. При наличии нескольких значений audience каждое значение проверяется либо до их исчерпания (в этом случае проверка будет не пройдена), либо до обнаружения подходящего значения. Необходимо указать по крайней мере один элемент audience.                                                                     | Нет       |
| issuer-signing-keys | Список ключей безопасности в кодировке Base64, используемых для проверки подписанных маркеров. Если имеется несколько ключей безопасности, то каждый из них будет выполняться до тех пор, пока все они не будут исчерпаны (в этом случае проверка завершится ошибкой) или одна из них завершится успешно (полезно для смены токена). Ключи могут содержать дополнительный атрибут `id`, используемый для сопоставления с утверждением `kid`. <br/><br/>В качестве альтернативы можно указать ключ подписывания издателя, используя:<br/><br/> - `certificate-id` в формате `<key certificate-id="mycertificate" />` , чтобы указать идентификатор сущности сертификата, [отправленной](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-certificate-entity#Add) в Управление API<br/>-Модуль RSA `n` и `e` пара Экспонент в формате `<key n="<modulus>" e="<exponent>" />` для указания параметров RSA в формате base64url-Encoded               | Нет       |
| decryption-keys     | Список ключей в кодировке Base64 для расшифровки маркеров. Если имеется несколько ключей безопасности, то каждый из них будет пытаться пройти до тех пор, пока не будут исчерпаны все ключи (в этом случае проверка завершится ошибкой) или ключ завершится успешно. Ключи могут содержать дополнительный атрибут `id`, используемый для сопоставления с утверждением `kid`.<br/><br/>В качестве альтернативы можно указать ключ расшифровки, используя:<br/><br/> - `certificate-id` в формате `<key certificate-id="mycertificate" />` , чтобы указать идентификатор сущности сертификата, [отправленной](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-certificate-entity#Add) в Управление API                                                 | Нет       |
| issuers             | Список допустимых субъектов-служб, выдавших маркер. При наличии нескольких значений элемента issuer каждое значение проверяется либо до их исчерпания (в этом случае проверка будет не пройдена), либо до обнаружения подходящего значения.                                                                                                                                         | Нет       |
| openid-config       | Элемент, используемый для указания соответствующей конечной точки конфигурации Open ID, из которой можно получить ключи подписывания и элемент issuer.                                                                                                                                                                                                                        | Нет       |
| required-claims     | Содержит список утверждений, которые должны содержаться в маркере, который будет считаться допустимым. Если для атрибута `match` задано значение `all`, каждое значение утверждения в политике должно присутствовать в маркере для успешного завершения проверки. Если для атрибута `match` задано значение `any`, в маркере должно присутствовать по крайней мере одно утверждение для успешного завершения проверки. | Нет       |

### <a name="attributes"></a>Атрибуты

| Имя                            | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                            | Обязательно                                                                         | По умолчанию                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| clock-skew                      | Интервал времени. Настройка максимальной ожидаемой разницы во времени между системными часами поставщика маркеров и экземпляра управления API.                                                                                                                                                                                                                                                                                                               | Нет                                                                               | 0 секунд                                                                         |
| failed-validation-error-message | Сообщение об ошибке, которое возвращается в текст HTTP-ответа, если JWT не прошел проверку. Это сообщение должно содержать правильно экранированные специальные символы.                                                                                                                                                                                                                                                                                                 | Нет                                                                               | Сообщение об ошибке по умолчанию зависит от проблемы проверки, например "JWT отсутствует". |
| failed-validation-httpcode      | Код состояния HTTP, который возвращается, если JWT не прошел проверку.                                                                                                                                                                                                                                                                                                                                                                                         | Нет                                                                               | 401                                                                               |
| header-name                     | Имя заголовка НТТР, содержащего маркер.                                                                                                                                                                                                                                                                                                                                                                                                         | `header-name` `query-parameter-name` Необходимо указать один из или `token-value` . | Н/Д                                                                               |
| query-parameter-name            | Имя параметра запроса, содержащего маркер безопасности.                                                                                                                                                                                                                                                                                                                                                                                                     | `header-name` `query-parameter-name` Необходимо указать один из или `token-value` . | Н/Д                                                                               |
| значение токена                     | Выражение, возвращающее строку, содержащую токен JWT                                                                                                                                                                                                                                                                                                                                                                                                     | `header-name` `query-parameter-name` Необходимо указать один из или `token-value` . | Н/Д                                                                               |
| идентификатор                              | Атрибут `id` в элементе `key` позволяет указать строку, которая будет сопоставлена с утверждением `kid` в маркере (при наличии), чтобы найти подходящий ключ для проверки подписи.                                                                                                                                                                                                                                           | Нет                                                                               | Н/Д                                                                               |
| match                           | Атрибут `match` в элементе `claim` указывает, должно ли присутствовать каждое значение утверждения политики в маркере для успешного завершения проверки. Возможны следующие значения:<br /><br /> - `all` — каждое значение утверждения в политике должно присутствовать в маркере для успешного завершения проверки.<br /><br /> - `any` — в маркере должно присутствовать по крайней мере одно значение утверждения для успешного завершения проверки.                                                       | Нет                                                                               | все                                                                               |
| require-expiration-time         | Логическое. Указывает, требуется ли утверждение истечения срока действия для маркера.                                                                                                                                                                                                                                                                                                                                                                               | Нет                                                                               | true                                                                              |
| require-scheme                  | Имя схемы маркеров, например "Bearer". Когда задан этот атрибут, политики обеспечивают присутствие указанной схемы в значении заголовка Authorization.                                                                                                                                                                                                                                                                                    | Нет                                                                               | Н/Д                                                                               |
| require-signed-tokens           | Логическое. Указывает, должен ли быть подписан маркер.                                                                                                                                                                                                                                                                                                                                                                                           | Нет                                                                               | true                                                                              |
| separator                       | Строка. Указывает разделитель (например, ",") для извлечения набора значений из многозначного утверждения.                                                                                                                                                                                                                                                                                                                                          | Нет                                                                               | Н/Д                                                                               |
| url                             | URL-адрес конечной точки конфигурации Open ID, по которому можно получить метаданные конфигурации Open ID. Ответ должен соответствовать спецификациям, как определено в URL-адресе :`https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata`. Для Azure Active Directory используйте URL-адрес `https://login.microsoftonline.com/{tenant-name}/.well-known/openid-configuration`, подставив необходимое имя клиента каталога, например `contoso.onmicrosoft.com`. | Да                                                                              | Н/Д                                                                               |
| Output-Token-переменная-имя      | Строка. Имя переменной контекста, которая будет принимать значение токена как объект типа [`Jwt`](api-management-policy-expressions.md) после успешной проверки токена                                                                                                                                                                                                                                                                                     | Нет                                                                               | Н/Д                                                                               |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.
-   **Области политики:** все области.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о работе с политиками см. в следующих статьях:

-   [Политики в управлении API](api-management-howto-policies.md)
-   [Преобразование API-интерфейсов](transform-api.md).
-   Полный перечень операторов политик и их параметров см. в [справочнике по политикам](./api-management-policies.md).
-   [Примеры политик](./policy-reference.md).

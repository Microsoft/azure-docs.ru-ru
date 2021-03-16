---
title: Расширенные политики службы управления API Azure | Документация Майкрософт
description: Сведения о расширенных политиках, доступных для использования в службе управления API Azure. См. примеры и Просмотр дополнительных доступных ресурсов.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/13/2020
ms.author: apimpm
ms.openlocfilehash: 03529fd3c0231617c477f4f16773039a02386683
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103562490"
---
# <a name="api-management-advanced-policies"></a>Расширенные политики в службе управления API

В этой статье рассматриваются приведенные ниже политики управления API. Дополнительные сведения о добавлении и настройке политик см. в статье о [политиках в управлении API](./api-management-policies.md).

## <a name="advanced-policies"></a><a name="AdvancedPolicies"></a> Расширенные политики

-   [Управление потоками](api-management-advanced-policies.md#choose) — условно применяет правила политики на основе результатов вычисления логических [выражений](api-management-policy-expressions.md).
-   [Перенаправляющий запрос](#ForwardRequest) — перенаправляет запрос в серверную службу.
-   [Ограничения параллелизма](#LimitConcurrency) не позволяет, чтобы заключенные политики одновременно выполнялись запросами, количество которых выше указанного.
-   [Регистрация в концентраторе событий](#log-to-eventhub) — отправляет сообщения в концентратор событий, который указан сущностью Logger.
-   [Макетирование ответа](#mock-response) — прекращает выполнение конвейера и возвращает макетированный ответ непосредственно вызывающему объекту.
-   [Повторить](#Retry) — повторяет выполнение инструкций встраиваемой политики, если не выполнено условие и до тех пор пока оно не будет выполнено. Выполнение будет повторяться через определенные промежутки времени и до указанного количества повторных попыток.
-   [Возврат ответа](#ReturnResponse) — прекращает выполнение конвейера и возвращает указанный ответ непосредственно вызывающему объекту.
-   [Отправка одностороннего запроса](#SendOneWayRequest) — отправляет запрос на указанный URL-адрес и не ожидает ответа.
-   [Отправка запроса](#SendRequest) — отправляет запрос на указанный URL-адрес.
-   [Установка прокси-сервера HTTP](#SetHttpProxy) — позволяет маршрутизировать перенаправленные запросы через прокси-сервер HTTP.
-   [Установка метода запроса](#SetRequestMethod) — позволяет изменить метод HTTP для запроса.
-   [Установка кода состояния](#SetStatus) — меняет код состояния HTTP на указанное значение.
-   [Задание переменной](api-management-advanced-policies.md#set-variable) — сохраняет значение в именованной переменной [контекста](api-management-policy-expressions.md#ContextVariables) для последующего использования.
-   [Трассировка](#Trace) . Добавляет пользовательские трассировки в выходные данные [инспектора API](./api-management-howto-api-inspector.md) , Application Insights телеметрии и журналы ресурсов.
-   [Ожидание](#Wait) — ожидание вложенных [запросов на отправку](api-management-advanced-policies.md#SendRequest), [Получение значения из кэша](api-management-caching-policies.md#GetFromCacheByKey)или политики [потока управления](api-management-advanced-policies.md#choose) для завершения перед продолжением.

## <a name="control-flow"></a><a name="choose"></a> Поток управления

Политика `choose` применяет правила встраиваемой политики на основании результата вычисления логических выражений, подобных if-then-else или конструкции switch в языке программирования.

### <a name="policy-statement"></a><a name="ChoosePolicyStatement"></a> Правило политики

```xml
<choose>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <when condition="Boolean expression | Boolean constant">
        <!— one or more policy statements to be applied if the above condition is true  -->
    </when>
    <otherwise>
        <!— one or more policy statements to be applied if none of the above conditions are true  -->
</otherwise>
</choose>
```

Политика потока управления должна содержать по крайней мере один элемент `<when/>`. Элемент `<otherwise/>` является необязательным. Условия в элементах `<when/>` вычисляются в порядке их расположения в политике. Будут применены правила политики, включенные в первом элементе `<when/>` с атрибутом условия `true`. Политики, включенные в элементе `<otherwise/>` (если он имеется), будут применены, если все атрибуты условий элемента `<when/>` имеют значение `false`.

### <a name="examples"></a>Примеры

#### <a name="example"></a><a name="ChooseExample"></a> Пример

В следующем примере демонстрируются политика [задания переменной](api-management-advanced-policies.md#set-variable) и две политики потока управления.

Политика задания переменной находится в разделе "входящие" и определяет создание `isMobile` логической [контекстной](api-management-policy-expressions.md#ContextVariables) переменной, которой присваивается значение true, если `User-Agent` заголовок запроса содержит текст `iPad` или `iPhone`.

Первая политика управления потоками также находится в разделе inbound и условно применяет одну из двух политик [установки параметра строки запроса](api-management-transformation-policies.md#SetQueryStringParameter) в зависимости от значения переменной контекста `isMobile`.

Вторая политика управления потоками находится в разделе outbound и условно применяет политику [преобразования XML в JSON](api-management-transformation-policies.md#ConvertXMLtoJSON), если переменная `isMobile` имеет значение `true`.

```xml
<policies>
    <inbound>
        <set-variable name="isMobile" value="@(context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPad") || context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPhone"))" />
        <base />
        <choose>
            <when condition="@(context.Variables.GetValueOrDefault<bool>("isMobile"))">
                <set-query-parameter name="mobile" exists-action="override">
                    <value>true</value>
                </set-query-parameter>
            </when>
            <otherwise>
                <set-query-parameter name="mobile" exists-action="override">
                    <value>false</value>
                </set-query-parameter>
            </otherwise>
        </choose>
    </inbound>
    <outbound>
        <base />
        <choose>
            <when condition="@(context.Variables.GetValueOrDefault<bool>("isMobile"))">
                <xml-to-json kind="direct" apply="always" consider-accept-header="false"/>
            </when>
        </choose>
    </outbound>
</policies>
```

#### <a name="example"></a>Пример

В этом примере показано, как выполнить фильтрацию содержимого путем удаления элементов данных из ответа, полученного из внутренней службы при использовании продукта `Starter`. Пример настройки и использования этой политики см. в видео [Cloud Cover Episode 177: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Cloud Cover, эпизод 177: возможности управления API с Владом Виноградским) с отметки времени 34:30. Начните с 31:50, чтобы увидеть обзор [темного API-интерфейса небесного прогноза,](https://developer.forecast.io/) используемого для этой демонстрации.

```xml
<!-- Copy this snippet into the outbound section to remove a number of data elements from the response received from the backend service based on the name of the api product -->
<choose>
  <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
    <set-body>@{
        var response = context.Response.Body.As<JObject>();
        foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
          response.Property (key).Remove ();
        }
        return response.ToString();
      }
    </set-body>
  </when>
</choose>
```

### <a name="elements"></a>Элементы

| Элемент   | Описание                                                                                                                                                                                                                                                               | Обязательный |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| choose    | Корневой элемент.                                                                                                                                                                                                                                                             | Да      |
| when      | Условие для применения в частях `if` или `ifelse` политики `choose`. Если политика `choose` имеет несколько разделов `when`, они вычисляются последовательно. Когда `condition` элемента when принимает значение `true`, никакие последующие условия `when` не вычисляются. | Да      |
| otherwise | Содержит фрагмент кода политики, который используется в том случае, если ни одно из условий `when` не принимает значение `true`.                                                                                                                                                                               | Нет       |

### <a name="attributes"></a>Атрибуты

| Атрибут                                              | Описание                                                                                               | Обязательный |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- | -------- |
| condition="Логическое выражение &#124; Логическая константа" | Логическое выражение или константа, которую необходимо вычислить, когда вычисляется правило политики, содержащей `when`. | Да      |

### <a name="usage"></a><a name="ChooseUsage"></a> Загрузка

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="forward-request"></a><a name="ForwardRequest"></a> Пересылка запроса

Политика `forward-request` перенаправляет входящий запрос во внутреннюю службу, указанную в [контексте](api-management-policy-expressions.md#ContextVariables) запроса. URL-адрес внутренней службы указывается в [параметрах](./import-and-publish.md) API и может быть изменен с помощью политики [настройки внутренней службы](api-management-transformation-policies.md) .

> [!NOTE]
> Удаление этой политики приводит к тому, что запрос не перенаправляется во внутреннюю службу и политики в разделе outbound вычисляются сразу после успешного завершения обработки политик в разделе inbound.

### <a name="policy-statement"></a>Правило политики

```xml
<forward-request timeout="time in seconds" follow-redirects="false | true" buffer-request-body="false | true" buffer-response="true | false" fail-on-error-status-code="false | true"/>
```

### <a name="examples"></a>Примеры

#### <a name="example"></a>Пример

Следующая политика уровня API перенаправляет все запросы API в серверную службу с интервалом времени ожидания 60 секунд.

```xml
<!-- api level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <forward-request timeout="60"/>
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>Пример

Эта политика уровня операций использует элемент `base`, чтобы наследовать внутреннюю политику от родительской области уровня API.

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <base/>
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>Пример

Эта политика уровня операций явным образом перенаправляет все запросы во внутреннюю службу с временем ожидания 120 секунд и не наследует родительскую политику уровня API. Если серверная служба реагирует на код состояния ошибки от 400 до 599 включительно, то будет запущен раздел [On-Error](api-management-error-handling-policies.md) .

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <forward-request timeout="120" fail-on-error-status-code="true" />
        <!-- effective policy. note the absence of <base/> -->
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

#### <a name="example"></a>Пример

Эта политика уровня операций не перенаправляет запросы во внутреннюю службу.

```xml
<!-- operation level -->
<policies>
    <inbound>
        <base/>
    </inbound>
    <backend>
        <!-- no forwarding to backend -->
    </backend>
    <outbound>
        <base/>
    </outbound>
</policies>

```

### <a name="elements"></a>Элементы

| Элемент         | Описание   | Обязательный |
| --------------- | ------------- | -------- |
| forward-request | Корневой элемент. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут                                     | Описание                                                                                                                                                                                                                                                                                                    | Обязательный | По умолчанию |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| timeout="целое число"                             | Время ожидания (в секундах), в течение которого заголовки HTTP-ответа будут возвращены внутренней службой до возникновения ошибки времени ожидания. Минимальное значение — 0 секунд. Значения, превышающие 240 секунд, могут не учитываться, так как Базовая сетевая инфраструктура может удалить неактивные подключения после этого времени. | Нет       | None    |
| следовать-перенаправления = "false &#124; true"          | Указывает, что происходит с перенаправлениями от внутренней службы: шлюз выполняет их или они возвращаются вызывающему объекту.                                                                                                                                                                                                    | Нет       | false   |
| buffer-Request-Body = "false &#124; true"       | Если задано значение "true", запрос буферизован и будет повторно использован при [повторной попытке](api-management-advanced-policies.md#Retry).                                                                                                                                                                                               | Нет       | false   |
| buffer-Response = "false &#124; true" | Влияет на обработку фрагментированных откликов. Если задано значение "false", каждый блок, полученный от серверной части, немедленно возвращается вызывающему. Если задано значение "true", блоки помещаются в буфер (8 КБ, если не обнаружен конец потока) и только затем возвращается вызывающему объекту. | Нет | true |
| Ошибка-On-Error-Status-code = "false &#124; true" | Если задано значение true Triggers [On-Error](api-management-error-handling-policies.md) , то для кодов ответов в диапазоне от 400 до 599 включительно.                                                                                                                                                                      | Нет       | false   |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** backend.
-   **Области политики:** все области.

## <a name="limit-concurrency"></a><a name="LimitConcurrency"></a> Ограничение параллелизма

Политика `limit-concurrency` не позволяет, чтобы заключенные политики одновременно выполнялись запросами, количество которых выше указанного. После превышения этого числа новые запросы будут завершаться сбоем с кодом состояния "429 Too Many Requests" (429 — слишком много запросов).

### <a name="policy-statement"></a><a name="LimitConcurrencyStatement"></a> Правило политики

```xml
<limit-concurrency key="expression" max-count="number">
        <!— nested policy statements -->
</limit-concurrency>
```

### <a name="examples"></a>Примеры

#### <a name="example"></a>Пример

Ниже приведен пример, как ограничить число запросов, пересылаемых в серверную часть, на основе значения переменной контекста.

```xml
<policies>
  <inbound>…</inbound>
  <backend>
    <limit-concurrency key="@((string)context.Variables["connectionId"])" max-count="3">
      <forward-request timeout="120"/>
    </limit-concurrency>
  </backend>
  <outbound>…</outbound>
</policies>
```

### <a name="elements"></a>Элементы

| Элемент           | Описание   | Обязательный |
| ----------------- | ------------- | -------- |
| limit-concurrency | Корневой элемент. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут | Описание                                                                                        | Обязательный | По умолчанию |
| --------- | -------------------------------------------------------------------------------------------------- | -------- | ------- |
| ключ       | Строка. Допустимое выражение. Задает область параллелизма. Используется несколькими политиками. | Да      | Н/Д     |
| max-count | Целое число. Указывает максимальное количество запросов для ввода политики.           | Да      | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="log-to-event-hub"></a><a name="log-to-eventhub"></a> Регистрация в концентраторе событий

Политика `log-to-eventhub` отправляет сообщения в определенном формате концентратору событий, который указан сущностью Logger. Как и предполагает ее имя, эта политика используется для сохранения контекстных сведений выбранного запроса или ответа для сетевого или автономного анализа.

> [!NOTE]
> Пошаговое руководство по настройке концентратора событий и ведению журнала событий см. в статье [Как регистрировать события службы управления API в Центрах событий Azure](./api-management-howto-log-event-hubs.md).

### <a name="policy-statement"></a>Правило политики

```xml
<log-to-eventhub logger-id="id of the logger entity" partition-id="index of the partition where messages are sent" partition-key="value used for partition assignment">
  Expression returning a string to be logged
</log-to-eventhub>

```

### <a name="example"></a>Пример

Любую строку можно использовать как значение для регистрации в Центрах событий. В этом примере в журнал концентратора событий, зарегистрированный с идентификатором, заносятся Дата и время, имя службы развертывания, идентификатор запроса, IP-адрес и имя операции для всех входящих вызовов. `contoso-logger`

```xml
<policies>
  <inbound>
    <log-to-eventhub logger-id ='contoso-logger'>
      @( string.Join(",", DateTime.UtcNow, context.Deployment.ServiceName, context.RequestId, context.Request.IpAddress, context.Operation.Name) )
    </log-to-eventhub>
  </inbound>
  <outbound>
  </outbound>
</policies>
```

### <a name="elements"></a>Элементы

| Элемент         | Описание                                                                     | Обязательный |
| --------------- | ------------------------------------------------------------------------------- | -------- |
| log-to-eventhub | Корневой элемент. Значение этого элемента — это строка, которая будет зарегистрирована в концентраторе событий. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут     | Описание                                                               | Обязательный                                                             |
| ------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| logger-id     | Идентификатор средства ведения журнала, зарегистрированного в службе управления API.         | Да                                                                  |
| partition-id  | Указывает индекс раздела, куда должны отправляться сообщения.             | Необязательный элемент. Этот атрибут не может использоваться, если используется `partition-key`. |
| partition-key | Указывает значение, используемое для назначения секции при отправке сообщений. | Необязательный элемент. Этот атрибут не может использоваться, если используется `partition-id`.  |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="mock-response"></a><a name="mock-response"></a> Макетирование ответа

Политика `mock-response`, как следует из имени, используется для макетирования интерфейсов API и операций. Она прекращает выполнение конвейера и возвращает макетированный ответ вызывающему объекту. Эта политика всегда пытается вернуть ответы наивысшего качества. При возможности, она предпочитает примеры содержимого ответа. Она создает примеры ответов из схем, если схемы указаны, а примеры — нет. Если не найдены ни примеры, ни схемы, возвращаются ответы без содержимого.

### <a name="policy-statement"></a>Правило политики

```xml
<mock-response status-code="code" content-type="media type"/>

```

### <a name="examples"></a>Примеры

```xml
<!-- Returns 200 OK status code. Content is based on an example or schema, if provided for this
status code. First found content type is used. If no example or schema is found, the content is empty. -->
<mock-response/>

<!-- Returns 200 OK status code. Content is based on an example or schema, if provided for this
status code and media type. If no example or schema found, the content is empty. -->
<mock-response status-code='200' content-type='application/json'/>
```

### <a name="elements"></a>Элементы

| Элемент       | Описание   | Обязательный |
| ------------- | ------------- | -------- |
| mock-response | Корневой элемент. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут    | Описание                                                                                           | Обязательный | По умолчанию |
| ------------ | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| status-code  | Указывает код состояния ответа и позволяет выбрать соответствующий пример или схему.                 | нет       | 200     |
| content-type | Указывает значение заголовка ответа `Content-Type` и позволяет выбрать соответствующий пример или схему. | Нет       | None    |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, on-error.

-   **Области политики:** все области.

## <a name="retry"></a><a name="Retry"></a> Повторите

`retry`Политика выполняет дочерние политики один раз, а затем повторяет их выполнение до тех пор, пока `condition` `false` не будет `count` исчерпана повторная попытка.

### <a name="policy-statement"></a>Правило политики

```xml

<retry
    condition="boolean expression or literal"
    count="number of retry attempts"
    interval="retry interval in seconds"
    max-interval="maximum retry interval in seconds"
    delta="retry interval delta in seconds"
    first-fast-retry="boolean expression or literal">
        <!-- One or more child policies. No restrictions -->
</retry>

```

### <a name="example"></a>Пример

В следующем примере перенаправление запроса повторяется до 10 раз, следуя экспоненциальному алгоритму повторения. Так как `first-fast-retry` имеет значение false, все попытки повтора выполняются в соответствии с экспоненциальным алгоритмом повторения.

```xml

<retry
    condition="@(context.Response.StatusCode == 500)"
    count="10"
    interval="10"
    max-interval="100"
    delta="10"
    first-fast-retry="false">
        <forward-request buffer-request-body="true" />
</retry>

```

### <a name="elements"></a>Элементы

| Элемент | Описание                                                         | Обязательный |
| ------- | ------------------------------------------------------------------- | -------- |
| retry   | Корневой элемент. Может содержать любые другие политики в качестве дочерних элементов. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут        | Описание                                                                                                                                           | Обязательный | По умолчанию |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| condition        | Логический литерал или [выражение](api-management-policy-expressions.md), указывающее, что нужно сделать: прекратить повторные попытки (`false`) или продолжить (`true`).      | Да      | Н/Д     |
| count            | Положительное число, определяющее максимальное число повторных попыток.                                                                                | Да      | Н/Д     |
| interval         | Положительное значение в секундах, определяющее интервал ожидания между повторными попытками.                                                                 | Да      | Н/Д     |
| max-interval     | Положительное значение в секундах, определяющее максимальный интервал ожидания между повторными попытками. Оно используется для реализации экспоненциального алгоритма повторения. | Нет       | Н/Д     |
| delta            | Положительное значение в секундах, определяющее увеличение интервала ожидания. Используется для реализации линейного и экспоненциального алгоритмов повторения.             | Нет       | Н/Д     |
| first-fast-retry | Если задано значение `true` , первая повторная попытка выполняется немедленно.                                                                                  | Нет       | `false` |

> [!NOTE]
> Если указан только параметр `interval`, попытки выполняются с **фиксированным** интервалом.
> Если указаны параметры `interval` и `delta`, применяется **линейный** алгоритм повторения, где время ожидания между повторениями вычисляется согласно формуле `interval + (count - 1)*delta`.
> Если указаны параметры `interval`, `max-interval` и `delta`, применяется **экспоненциальный** алгоритм повторения, где время ожидания между повторениями растет экспоненциально от значения `interval` до значения `max-interval` согласно формуле `min(interval + (2^count - 1) * random(delta * 0.8, delta * 1.2), max-interval)`.

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes). Обратите внимание, что эта политика наследует ограничения использования дочерних политик.

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="return-response"></a><a name="ReturnResponse"></a> Возврат ответа

Политика `return-response` прерывает выполнение конвейера и возвращает вызывающему объекту ответ по умолчанию или настраиваемый ответ. По умолчанию возвращается код `200 OK` без текста. Можно указать настраиваемый ответ с помощью переменной контекста или правил политики. Если указаны оба, ответ, содержащийся в переменной контекста, изменяется правилами политики перед возвращением вызывающему объекту.

### <a name="policy-statement"></a>Правило политики

```xml
<return-response response-variable-name="existing context variable">
  <set-header/>
  <set-body/>
  <set-status/>
</return-response>

```

### <a name="example"></a>Пример

```xml
<return-response>
   <set-status code="401" reason="Unauthorized"/>
   <set-header name="WWW-Authenticate" exists-action="override">
      <value>Bearer error="invalid_token"</value>
   </set-header>
</return-response>

```

### <a name="elements"></a>Элементы

| Элемент         | Описание                                                                               | Обязательный |
| --------------- | ----------------------------------------------------------------------------------------- | -------- |
| return-response | Корневой элемент.                                                                             | Да      |
| set-header      | Правило политики [set-header](api-management-transformation-policies.md#SetHTTPheader). | Нет       |
| set-body        | Правило политики [set-body](api-management-transformation-policies.md#SetBody).         | Нет       |
| set-status      | Правило политики [set-status](api-management-advanced-policies.md#SetStatus).           | Нет       |

### <a name="attributes"></a>Атрибуты

| Атрибут              | Описание                                                                                                                                                                          | Обязательный  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| response-variable-name | Имя переменной контекста, на которую ссылается, например, вышестоящая политика [send-request](api-management-advanced-policies.md#SendRequest) и которая содержит объект `Response`. | Необязательный элемент. |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="send-one-way-request"></a><a name="SendOneWayRequest"></a> Отправка одностороннего запроса

Политика `send-one-way-request` отправляет запрос на указанный URL-адрес и не ожидает ответа.

### <a name="policy-statement"></a>Правило политики

```xml
<send-one-way-request mode="new | copy">
  <url>...</url>
  <method>...</method>
  <header name="" exists-action="override | skip | append | delete">...</header>
  <body>...</body>
  <authentication-certificate thumbprint="thumbprint" />
</send-one-way-request>

```

### <a name="example"></a>Пример

В этом примере политики демонстрируется использование политики `send-one-way-request` для отправки сообщения в чат Slack, если код HTTP-ответа больше или равен 500. Дополнительные сведения об этом примере см. в статье [Использование внешних служб из службы управления API Azure](./api-management-sample-send-request.md).

```xml
<choose>
    <when condition="@(context.Response.StatusCode >= 500)">
      <send-one-way-request mode="new">
        <set-url>https://hooks.slack.com/services/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</set-url>
        <set-method>POST</set-method>
        <set-body>@{
                return new JObject(
                        new JProperty("username","APIM Alert"),
                        new JProperty("icon_emoji", ":ghost:"),
                        new JProperty("text", String.Format("{0} {1}\nHost: {2}\n{3} {4}\n User: {5}",
                                                context.Request.Method,
                                                context.Request.Url.Path + context.Request.Url.QueryString,
                                                context.Request.Url.Host,
                                                context.Response.StatusCode,
                                                context.Response.StatusReason,
                                                context.User.Email
                                                ))
                        ).ToString();
            }</set-body>
      </send-one-way-request>
    </when>
</choose>

```

### <a name="elements"></a>Элементы

| Элемент                    | Описание                                                                                                 | Обязательный                        |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------- |
| send-one-way-request       | Корневой элемент.                                                                                               | Да                             |
| url                        | URL-адрес запроса.                                                                                     | Нет, если mode=copy. В противном случае да. |
| method                     | Метод HTTP, используемый для запроса.                                                                            | Нет, если mode=copy. В противном случае да. |
| заголовок                     | Заголовок запроса. Используйте несколько элементов заголовка для нескольких заголовков запросов.                                  | Нет                              |
| текст                       | Текст запроса.                                                                                           | Нет                              |
| authentication-certificate | [Сертификат, используемый для проверки подлинности клиента.](api-management-authentication-policies.md#ClientCertificate) | Нет                              |

### <a name="attributes"></a>Атрибуты

| Атрибут     | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Обязательный | По умолчанию  |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------- |
| mode="строка" | Определяет, является ли это новым запросом или копией текущего запроса. В исходящем режиме mode=copy не инициализирует текст запроса.                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Нет       | Оператор new      |
| name          | Указывает имя заголовка, которое должно быть установлено.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Да      | Н/Д      |
| exists-action | Указывает действие, которое должно быть выполнено, когда заголовок уже задан. Атрибут должен иметь одно из следующих значений:<br /><br /> -override — заменяет значение существующего заголовка.<br />-Skip — не заменяет существующее значение заголовка.<br />-Append — добавляет значение к существующему значению заголовка.<br />-Delete — удаляет заголовок из запроса.<br /><br /> Если установлено значение `override`, перечисление нескольких записей с одним и тем же именем будет приводить к тому, что заголовок будет устанавливаться в соответствии со всеми записями (будут перечисляться несколько раз). В результате будут установлены только перечисленные значения. | Нет       | override |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="send-request"></a><a name="SendRequest"></a> Отправка запроса

Политика `send-request` отправляет запрос по указанному URL-адресу с задержкой, не превышающей заданное значение времени ожидания.

### <a name="policy-statement"></a>Правило политики

```xml
<send-request mode="new|copy" response-variable-name="" timeout="60 sec" ignore-error
="false|true">
  <set-url>...</set-url>
  <set-method>...</set-method>
  <set-header name="" exists-action="override|skip|append|delete">...</set-header>
  <set-body>...</set-body>
  <authentication-certificate thumbprint="thumbprint" />
</send-request>

```

### <a name="example"></a>Пример

В этом примере показан один из способов проверки маркера ссылки на сервере авторизации. Дополнительные сведения об этом примере см. в статье [Использование внешних служб из службы управления API Azure](./api-management-sample-send-request.md).

```xml
<inbound>
  <!-- Extract Token from Authorization header parameter -->
  <set-variable name="token" value="@(context.Request.Headers.GetValueOrDefault("Authorization","scheme param").Split(' ').Last())" />

  <!-- Send request to Token Server to validate token (see RFC 7662) -->
  <send-request mode="new" response-variable-name="tokenstate" timeout="20" ignore-error="true">
    <set-url>https://microsoft-apiappec990ad4c76641c6aea22f566efc5a4e.azurewebsites.net/introspection</set-url>
    <set-method>POST</set-method>
    <set-header name="Authorization" exists-action="override">
      <value>basic dXNlcm5hbWU6cGFzc3dvcmQ=</value>
    </set-header>
    <set-header name="Content-Type" exists-action="override">
      <value>application/x-www-form-urlencoded</value>
    </set-header>
    <set-body>@($"token={(string)context.Variables["token"]}")</set-body>
  </send-request>

  <choose>
        <!-- Check active property in response -->
        <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
            <!-- Return 401 Unauthorized with http-problem payload -->
            <return-response>
                <set-status code="401" reason="Unauthorized" />
                <set-header name="WWW-Authenticate" exists-action="override">
                    <value>Bearer error="invalid_token"</value>
                </set-header>
            </return-response>
        </when>
    </choose>
  <base />
</inbound>

```

### <a name="elements"></a>Элементы

| Элемент                    | Описание                                                                                                 | Обязательный                        |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------- |
| send-request               | Корневой элемент.                                                                                               | Да                             |
| url                        | URL-адрес запроса.                                                                                     | Нет, если mode=copy. В противном случае да. |
| method                     | Метод HTTP, используемый для запроса.                                                                            | Нет, если mode=copy. В противном случае да. |
| заголовок                     | Заголовок запроса. Используйте несколько элементов заголовка для нескольких заголовков запросов.                                  | Нет                              |
| текст                       | Текст запроса.                                                                                           | Нет                              |
| authentication-certificate | [Сертификат, используемый для проверки подлинности клиента.](api-management-authentication-policies.md#ClientCertificate) | Нет                              |

### <a name="attributes"></a>Атрибуты

| Атрибут                       | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Обязательный | По умолчанию  |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------- |
| mode="строка"                   | Определяет, является ли это новым запросом или копией текущего запроса. В исходящем режиме mode=copy не инициализирует текст запроса.                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Нет       | Оператор new      |
| response-variable-name="строка" | Имя переменной контекста, которая примет объект response. Если переменная не существует, она будет создана после успешного выполнения политики и станет доступна через [`context.Variable`](api-management-policy-expressions.md#ContextVariables) коллекцию.                                                                                                                                                                                                                                                                                                                          | Да      | Н/Д      |
| timeout="целое число"               | Интервал времени ожидания в секундах до того, как вызов, адресованный URL-адресу, завершится сбоем.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Нет       | 60       |
| ignore-error                    | Если получено значение true и запрос завершился ошибкой:<br /><br /> -Если параметр Response-Variable-Name был указан, он будет содержать значение null.<br />— Если Response-Variable-Name не задано, context. Запрос не будет обновлен.                                                                                                                                                                                                                                                                                                                                                                                   | Нет       | false    |
| name                            | Указывает имя заголовка, которое должно быть установлено.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Да      | Н/Д      |
| exists-action                   | Указывает действие, которое должно быть выполнено, когда заголовок уже задан. Атрибут должен иметь одно из следующих значений:<br /><br /> -override — заменяет значение существующего заголовка.<br />-Skip — не заменяет существующее значение заголовка.<br />-Append — добавляет значение к существующему значению заголовка.<br />-Delete — удаляет заголовок из запроса.<br /><br /> Если установлено значение `override`, перечисление нескольких записей с одним и тем же именем будет приводить к тому, что заголовок будет устанавливаться в соответствии со всеми записями (будут перечисляться несколько раз). В результате будут установлены только перечисленные значения. | Нет       | override |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="set-http-proxy"></a><a name="SetHttpProxy"></a> Установка прокси-сервера HTTP

Политика `proxy` позволяет маршрутизировать запросы, перенаправленные в серверную часть, через прокси-сервер HTTP. Между шлюзом и прокси-сервером поддерживается только протокол HTTP (не HTTPS). Используются только базовая проверка подлинности и проверка подлинности NTLM.

### <a name="policy-statement"></a>Правило политики

```xml
<proxy url="http://hostname-or-ip:port" username="username" password="password" />

```

### <a name="example"></a>Пример

Обратите внимание на использование [свойства](api-management-howto-properties.md) в качестве значений имени пользователя и пароля, чтобы избегать хранения конфиденциальных сведений в документе политики.

```xml
<proxy url="http://192.168.1.1:8080" username={{username}} password={{password}} />

```

### <a name="elements"></a>Элементы

| Элемент | Описание  | Обязательный |
| ------- | ------------ | -------- |
| proxy   | Корневой элемент | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут         | Описание                                            | Обязательный | По умолчанию |
| ----------------- | ------------------------------------------------------ | -------- | ------- |
| url="string"      | URL-адрес прокси-сервера в виде http://host:port.             | Да      | Н/Д     |
| username="string" | Имя пользователя, которое следует использовать для проверки подлинности на прокси-сервере. | Нет       | Н/Д     |
| password="string" | Пароль, который следует использовать для проверки подлинности на прокси-сервере. | Нет       | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.

-   **Области политики:** все области.

## <a name="set-request-method"></a><a name="SetRequestMethod"></a> Задание метода запроса

Политика `set-method` позволяет изменить метод HTTP-запроса для запроса.

### <a name="policy-statement"></a>Правило политики

```xml
<set-method>METHOD</set-method>

```

### <a name="example"></a>Пример

В этом примере политики демонстрируется использование политики `set-method` для отправки сообщения в чат Slack, если код HTTP-ответа больше или равен 500. Дополнительные сведения об этом примере см. в статье [Использование внешних служб из службы управления API Azure](./api-management-sample-send-request.md).

```xml
<choose>
    <when condition="@(context.Response.StatusCode >= 500)">
      <send-one-way-request mode="new">
        <set-url>https://hooks.slack.com/services/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</set-url>
        <set-method>POST</set-method>
        <set-body>@{
                return new JObject(
                        new JProperty("username","APIM Alert"),
                        new JProperty("icon_emoji", ":ghost:"),
                        new JProperty("text", String.Format("{0} {1}\nHost: {2}\n{3} {4}\n User: {5}",
                                                context.Request.Method,
                                                context.Request.Url.Path + context.Request.Url.QueryString,
                                                context.Request.Url.Host,
                                                context.Response.StatusCode,
                                                context.Response.StatusReason,
                                                context.User.Email
                                                ))
                        ).ToString();
            }</set-body>
      </send-one-way-request>
    </when>
</choose>

```

### <a name="elements"></a>Элементы

| Элемент    | Описание                                                       | Обязательный |
| ---------- | ----------------------------------------------------------------- | -------- |
| set-method | Корневой элемент. Значение элемента задает метод HTTP. | Да      |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, on-error.

-   **Области политики:** все области.

## <a name="set-status-code"></a><a name="SetStatus"></a> Задание кода состояния

Политика `set-status` меняет код состояния HTTP на указанное значение.

### <a name="policy-statement"></a>Правило политики

```xml
<set-status code="" reason=""/>

```

### <a name="example"></a>Пример

В этом примере показано, как вернуть ответ 401, если маркер проверки подлинности является недопустимым. Дополнительные сведения см. в статье [Использование внешних служб из службы управления API Azure](./api-management-sample-send-request.md).

```xml
<choose>
  <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
    <return-response response-variable-name="existing response variable">
      <set-status code="401" reason="Unauthorized" />
      <set-header name="WWW-Authenticate" exists-action="override">
        <value>Bearer error="invalid_token"</value>
      </set-header>
    </return-response>
  </when>
</choose>

```

### <a name="elements"></a>Элементы

| Элемент    | Описание   | Обязательный |
| ---------- | ------------- | -------- |
| set-status | Корневой элемент. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут       | Описание                                                | Обязательный | По умолчанию |
| --------------- | ---------------------------------------------------------- | -------- | ------- |
| code="целое число"  | Код состояния HTTP для возврата.                            | Да      | Н/Д     |
| reason="строка" | Описание причины для возврата кода состояния. | Да      | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** outbound, backend, on-error.
-   **Области политики:** все области.

## <a name="set-variable"></a><a name="set-variable"></a> Задать переменную

Политика `set-variable` объявляет [переменную](api-management-policy-expressions.md#ContextVariables) контекста и присваивает ей значение, заданное с помощью [выражение](api-management-policy-expressions.md) или строкового литерала. Если выражение содержит литерал, он будет преобразован в строку и будет иметь тип значения `System.String`.

### <a name="policy-statement"></a><a name="set-variablePolicyStatement"></a> Правило политики

```xml
<set-variable name="variable name" value="Expression | String literal" />
```

### <a name="example"></a><a name="set-variableExample"></a> Пример

В следующем примере демонстрируется политика задания переменной в разделе inbound. Эта политика задания переменной определяет создание `isMobile` логической [контекстной](api-management-policy-expressions.md#ContextVariables) переменной, которая принимает значение true, если заголовок запроса `User-Agent` содержит текст `iPad` или `iPhone`.

```xml
<set-variable name="IsMobile" value="@(context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPad") || context.Request.Headers.GetValueOrDefault("User-Agent","").Contains("iPhone"))" />
```

### <a name="elements"></a>Элементы

| Элемент      | Описание   | Обязательный |
| ------------ | ------------- | -------- |
| set-variable | Корневой элемент. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут | Описание                                                              | Обязательно |
| --------- | ------------------------------------------------------------------------ | -------- |
| name      | Имя переменной.                                                | Да      |
| Значение     | Значение переменной. Это может быть выражение или литеральное значение. | Да      |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.
-   **Области политики:** все области.

### <a name="allowed-types"></a><a name="set-variableAllowedTypes"></a> Допустимые типы

Выражения, используемые в политике `set-variable`, должны возвращать один из следующих основных типов.

-   System.Boolean
-   System.SByte
-   System.Byte
-   System.UInt16
-   System.UInt32
-   System.UInt64
-   System.Int16
-   System.Int32
-   System.Int64
-   System.Decimal
-   System.Single
-   System.Double
-   System.Guid
-   System.String
-   System.Char
-   System.DateTime
-   System.TimeSpan
-   System.Byte?
-   System.UInt16?
-   System.UInt32?
-   System.UInt64?
-   System.Int16?
-   System.Int32?
-   System.Int64?
-   System.Decimal?
-   System.Single?
-   System.Double?
-   System.Guid?
-   System.String?
-   System.Char?
-   System.DateTime?

## <a name="trace"></a><a name="Trace"></a> Трассировки

`trace`Политика добавляет пользовательскую трассировку в выходные данные инспектора API, Application Insights телеметрии и (или) журналы ресурсов.

-   Политика добавляет пользовательскую трассировку в выходные данные [инспектора API](./api-management-howto-api-inspector.md) при активации трассировки, т. е. `Ocp-Apim-Trace` заголовок запроса представлен и имеет значение true, а `Ocp-Apim-Subscription-Key` заголовок запроса имеется и содержит допустимый ключ, позволяющий отслеживать.
-   Политика создает данные телеметрии [трассировки](../azure-monitor/app/data-model-trace-telemetry.md) в Application Insights, когда [Application Insights интеграция](./api-management-howto-app-insights.md) включена и значение, `severity` указанное в политике, равно или больше, чем `verbosity` указано в параметре диагностики.
-   Политика добавляет свойство в запись журнала при включении [журналов ресурсов](./api-management-howto-use-azure-monitor.md#activity-logs) , а уровень серьезности, указанный в политике, равен уровню детализации, указанному в параметре диагностики, или выше него.

### <a name="policy-statement"></a>Правило политики

```xml

<trace source="arbitrary string literal" severity="verbose|information|error">
    <message>String literal or expressions</message>
    <metadata name="string literal or expressions" value="string literal or expressions"/>
</trace>

```

### <a name="example"></a><a name="traceExample"></a> Пример

```xml
<trace source="PetStore API" severity="verbose">
    <message>@((string)context.Variables["clientConnectionID"])</message>
    <metadata name="Operation Name" value="New-Order"/>
</trace>
```

### <a name="elements"></a>Элементы

| Элемент  | Описание                                                                                                                                          | Обязательный |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| трассировка    | Корневой элемент.                                                                                                                                        | Да      |
| message  | Строка или выражение для записи.                                                                                                                 | Да      |
| метаданные | Добавляет пользовательское свойство в данные телеметрии [трассировки](../azure-monitor/app/data-model-trace-telemetry.md) Application Insights. | Нет       |

### <a name="attributes"></a>Атрибуты

| Атрибут | Описание                                                                                                               | Обязательный | По умолчанию |
| --------- | ------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| source    | Строковый литерал, понятный средству просмотра трассировки и указывающий источник сообщения.                                   | Да      | Н/Д     |
| severity  | Задает степень серьезности трассировки. Допустимые значения: `verbose` , `information` , `error` (от нижнего к верхнему). | Нет       | Подробный |
| name      | Имя свойства.                                                                                                     | Да      | Н/Д     |
| Значение     | Значение свойства.                                                                                                    | Да      | Н/Д     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend, on-error.

-   **Области политики:** все области.

## <a name="wait"></a><a name="Wait"></a> Ожидания

Политика `wait` параллельно выполняет свои непосредственные дочерние политики и ожидает выполнения всех или одной из них, прежде чем будет завершена. Политика ожидания может иметь в качестве непосредственных дочерних следующие политики: [Отправка запроса](api-management-advanced-policies.md#SendRequest), [Получение значения из кэша](api-management-caching-policies.md#GetFromCacheByKey) и [Управление потоками](api-management-advanced-policies.md#choose).

### <a name="policy-statement"></a>Правило политики

```xml
<wait for="all|any">
  <!--Wait policy can contain send-request, cache-lookup-value,
        and choose policies as child elements -->
</wait>

```

### <a name="example"></a>Пример

В следующем примере две политики `choose` являются непосредственными дочерними политиками политики `wait`. Каждая из этих политик `choose` выполняется в параллельном режиме. Каждая политика `choose` пытается извлечь кэшированное значение. В случае промаха кэша для получения значения вызывается внутренняя служба. В этом примере политика `wait` не завершается, пока не завершатся все ее непосредственные дочерние политики, так как атрибут `for` имеет значение `all`. В этом примере переменные контекста (`execute-branch-one`, `value-one`, `execute-branch-two` и `value-two`) объявляются вне области этого примера политики.

```xml
<wait for="all">
  <choose>
    <when condition="@((bool)context.Variables["execute-branch-one="])">
      <cache-lookup-value key="key-one" variable-name="value-one" />
      <choose>
        <when condition="@(!context.Variables.ContainsKey("value-one="))">
          <send-request mode="new" response-variable-name="value-one">
            <set-url>https://backend-one</set-url>
            <set-method>GET</set-method>
          </send-request>
        </when>
      </choose>
    </when>
  </choose>
  <choose>
    <when condition="@((bool)context.Variables["execute-branch-two="])">
      <cache-lookup-value key="key-two" variable-name="value-two" />
      <choose>
        <when condition="@(!context.Variables.ContainsKey("value-two="))">
          <send-request mode="new" response-variable-name="value-two">
            <set-url>https://backend-two</set-url>
            <set-method>GET</set-method>
          </send-request>
        </when>
      </choose>
    </when>
  </choose>
</wait>

```

### <a name="elements"></a>Элементы

| Элемент | Описание                                                                                                   | Обязательный |
| ------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| wait    | Корневой элемент. Может содержать в качестве дочерних элементов только политики `send-request`, `cache-lookup-value` и `choose`. | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут | Описание                                                                                                                                                                                                                                                                                                                                                                                                            | Обязательный | По умолчанию |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| для       | Определяет, ожидает ли политика `wait` завершения всех непосредственных дочерних политик или только одной. Допустимыми значениями являются:<br /><br /> —- `all`: ожидание завершения всех непосредственных дочерних политик;<br />-Any — ожидание завершения любой ненепосредственной дочерней политики. После завершения первой непосредственной дочерней политики политика `wait` завершается и прерывает выполнение других непосредственных дочерних политик. | Нет       | все     |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound, outbound, backend.
-   **Области политики:** все области.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о работе с политиками см. в следующих статьях:

-   [Политики в управлении API](api-management-howto-policies.md)
-   [Выражения политики](api-management-policy-expressions.md)
-   Полный перечень операторов политик и их параметров см. в [справочнике по политикам](./api-management-policies.md).
-   [Примеры политик](./policy-reference.md).

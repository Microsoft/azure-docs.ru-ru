---
title: Политики интеграции ДАПР управления API Azure | Документация Майкрософт
description: Узнайте о политиках управления API Azure для взаимодействия с расширениями микрослужб ДАПР.
author: vladvino
ms.author: vlvinogr
ms.date: 02/18/2021
ms.topic: article
ms.service: api-management
ms.openlocfilehash: 051bf4398555f318f613c66d58ec65be1d30e215
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101646815"
---
# <a name="api-management-dapr-integration-policies"></a>Политики интеграции ДАПР управления API

В этом разделе содержится справочник по политикам управления API интеграции ДАПР. ДАПР — это Переносимая среда выполнения для создания приложений на основе микрослужб с отслеживанием состояния и без них с любым языком или платформой. Он кодифицирует общие шаблоны микрослужб, такие как обнаружение службы и вызов с помощью логики повторных попыток сборки, публикации и подписки с семантикой доставки по крайней мере один раз или подключаемые ресурсы привязки для упрощения компоновки с помощью внешних служб. Дополнительные сведения и инструкции о том, как приступить к работе с ДАПР, см. на странице [DAPR.IO](https://dapr.io) . Дополнительные сведения о добавлении и настройке политик см. в статье о [политиках в управлении API](api-management-howto-policies.md).

> [!CAUTION]
> Политики, упоминаемые в этом разделе, доступны в общедоступной предварительной версии и подвержены [дополнительным условиям использования для предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!IMPORTANT]
> Политики, упоминаемые в этом разделе, работают только в [автономной версии шлюза управления API](self-hosted-gateway-overview.md) с включенной поддержкой ДАПР.

## <a name="enable-dapr-support-in-the-self-hosted-gateway"></a>Включение поддержки ДАПР в шлюзе, размещенном на собственном сервере

Чтобы включить поддержку ДАПР в шлюзе, расположенном на собственном сервере, добавьте [заметки ДАПР](https://github.com/dapr/docs/blob/master/README.md) ниже в [шаблон развертывания Kubernetes](how-to-deploy-self-hosted-gateway-kubernetes.md) , заменив имя app-Name на нужное имя. Полное пошаговое руководство по настройке и использованию управления API с ДАПР доступно [здесь](https://aka.ms/apim/dapr/walkthru).
```yml
template:
    metadata:
      labels:
        app: app-name
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "app-name"
```


## <a name="distributed-application-runtime-dapr-integration-policies"></a>Политики интеграции Distributed Application Runtime (Dapr)

-  [Отправить запрос в службу](api-management-dapr-policies.md#invoke): использует среду выполнения ДАПР для выявления и надежной связи с микрослужбой ДАПР. Дополнительные сведения о вызове службы в ДАПР см. в описании в этом файле [readme](https://github.com/dapr/docs/blob/master/README.md#service-invocation) .
-  [Отправить сообщение в Pub/подразделы](api-management-dapr-policies.md#pubsub): использует среду выполнения ДАПР для публикации сообщения в раздел публикации и подписки. Дополнительные сведения о публикации и подписывание сообщений в ДАПР см. в описании в файле [сведений](https://github.com/dapr/docs/blob/master/README.md) .
-  [Выходная привязка триггера](api-management-dapr-policies.md#bind): использует среду выполнения ДАПР для вызова внешней системы через выходную привязку. Дополнительные сведения о привязках в ДАПР см. в описании в этом файле [readme](https://github.com/dapr/docs/blob/master/README.md) .

## <a name="send-request-to-a-service"></a><a name="invoke"></a> Отправить запрос службе

Эта политика задает целевой URL-адрес для текущего запроса на `http://localhost:3500/v1.0/invoke/{app-id}[.{ns-name}]/method/{method-name}` замену параметров шаблона значениями, указанными в инструкции политики.

Политика предполагает, что ДАПР выполняется в контейнере расширения в том же Pod, что и шлюз. После получения запроса среда выполнения ДАПР выполняет обнаружение служб и фактический вызов, включая возможное преобразование протокола между HTTP и gRPC, повторные попытки, распределенную трассировку и обработку ошибок.

### <a name="policy-statement"></a>Правило политики

```xml
<set-backend-service backend-id="dapr" dapr-app-id="app-id" dapr-method="method-name" dapr-namespace="ns-name" />
```

### <a name="examples"></a>Примеры

#### <a name="example"></a>Пример

В следующем примере демонстрируется вызов метода с именем "Back" в микрослужбе с именем "echo". `set-backend-service`Политика задает URL-адрес назначения `http://localhost:3500/v1.0/invoke/echo.echo-app/method/back` . `forward-request`Политика отправляет запрос в среду выполнения ДАПР, которая доставляет его микрослужбе.

Эта `forward-request` Политика показана здесь для ясности. Политика обычно является "унаследованной" из глобальной области с помощью `base` ключевого слова.

```xml
<policies>
    <inbound>
        <base />
        <set-backend-service backend-id="dapr" dapr-app-id="echo" dapr-method="back" dapr-namespace="echo-app" />
    </inbound>
    <backend>
        <forward-request />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

### <a name="elements"></a>Элементы

| Элемент             | Описание  | Обязательно |
|---------------------|--------------|----------|
| set-backend-service | Корневой элемент | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут        | Описание                     | Обязательно | По умолчанию |
|------------------|---------------------------------|----------|---------|
| backend-id       | Необходимо задать значение "ДАПР"           | Да      | Н/Д     |
| ДАПР-App-ID      | Имя целевой микрослужбы. Используется для формирования параметра [AppID](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) в ДАПР.| Да | Н/Д |
| ДАПР-метод      | Имя метода или URL-адреса для вызова в целевой микрослужбе. Сопоставляется с параметром [имени метода](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) в ДАПР.| Да | Н/Д |
| ДАПР — пространство имен   | Имя пространства имен, в котором размещена целевая микрослужба. Используется для формирования параметра [AppID](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) в ДАПР.| Нет | Н/Д |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

- **Разделы политики:** inbound.
- **Области политики:** все области.

## <a name="send-message-to-pubsub-topic"></a><a name="pubsub"></a> Отправить сообщение в Pub/подраздел

Эта политика указывает шлюзу управления API на отправку сообщения в раздел публикации и подписки ДАПР. Политика выполняет эту задачу, создавая запрос HTTP POST для `http://localhost:3500/v1.0/publish/{{pubsub-name}}/{{topic}}` замены параметров шаблона и добавления содержимого, указанного в инструкции политики.

Политика предполагает, что среда выполнения ДАПР выполняется в контейнере расширения в том же Pod, что и шлюз. Среда выполнения ДАПР реализует семантику публикации и подтипа.

### <a name="policy-statement"></a>Правило политики

```xml
<publish-to-dapr pubsub-name="pubsub-name" topic="topic-name" ignore-error="false|true" response-variable-name="resp-var-name" timeout="in seconds" template="Liquid" content-type="application/json">
    <!-- message content -->
</publish-to-dapr>
```

### <a name="examples"></a>Примеры

#### <a name="example"></a>Пример

В следующем примере показано, как отправить текст текущего запроса в [раздел](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md#url-parameters) "New" [компонента](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md#url-parameters)Pub/подраздела "Orders". Ответ, полученный от среды выполнения ДАПР, хранится в записи "ДАПР-Response" коллекции Variables в объекте [контекста](api-management-policy-expressions.md#ContextVariables) .

Если среда выполнения ДАПР не может определить целевой раздел, например, и выдает ошибку, то активируется раздел "On-Error". Ответ, полученный от среды выполнения ДАПР, возвращается непосредственно вызывающему объекту. В противном случае возвращается ответ по умолчанию `200 OK` .

Раздел "Серверная часть" пуст, а запрос не передается серверной части.

```xml
<policies>
     <inbound>
        <base />
        <publish-to-dapr
           pubsub-name="orders"
               topic="new"
               response-variable-name="dapr-response">
            @(context.Request.Body.As<string>())
        </publish-to-dapr>
    </inbound>
    <backend>
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
        <return-response response-variable-name="pubsub-response" />
    </on-error>
</policies>
```

### <a name="elements"></a>Элементы

| Элемент             | Описание  | Обязательно |
|---------------------|--------------|----------|
| Публикация в ДАПР     | Корневой элемент | Да      |

### <a name="attributes"></a>Атрибуты

| Атрибут        | Описание                     | Обязательно | По умолчанию |
|------------------|---------------------------------|----------|---------|
| pubsub — имя      | Имя целевого компонента PubSub. Сопоставляется с параметром [пубсубнаме](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md) в ДАПР. Если он отсутствует, значение атрибута __раздела__ должно быть в виде `pubsub-name/topic-name` .    | Нет       | None    |
| Раздел            | Имя раздела. Сопоставляется с параметром [раздела](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md) в ДАПР.               | Да      | Н/Д     |
| ignore-error     | Если задано значение, `true` то политика не активирует раздел ["On-Error"](api-management-error-handling-policies.md) при получении ошибки из среды выполнения ДАПР | Нет | `false` |
| response-variable-name | Имя записи коллекции [Variables](api-management-policy-expressions.md#ContextVariables) , используемой для хранения ответа от среды выполнения ДАПР | Нет | None |
| timeout | Время (в секундах) ожидания ответа среды выполнения ДАПР. Может находиться в диапазоне от 1 до 240 секунд. | Нет | 5 |
| шаблон | Обработчик шаблонов, используемый для преобразования содержимого сообщения. "Жидкость" — единственное поддерживаемое значение. | Нет | None |
| content-type | Тип содержимого сообщения. Единственное поддерживаемое значение — "Application/JSON". | Нет | None |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

- **Разделы политики:** inbound, outbound, on-error.
- **Области политики:** все области.

## <a name="trigger-output-binding"></a><a name="bind"></a> Выходная привязка триггера

Эта политика указывает шлюзу управления API на срабатывание исходящей [привязки](https://github.com/dapr/docs/blob/master/README.md)ДАПР. Политика выполняет эту задачу, создавая запрос HTTP POST для `http://localhost:3500/v1.0/bindings/{{bind-name}}` замены параметра шаблона и добавления содержимого, указанного в инструкции политики.

Политика предполагает, что среда выполнения ДАПР выполняется в контейнере расширения в том же Pod, что и шлюз. Среда выполнения ДАПР отвечает за вызов внешнего ресурса, представленного привязкой.

### <a name="policy-statement"></a>Правило политики

```xml
<invoke-dapr-binding name="bind-name" operation="op-name" ignore-error="false|true" response-variable-name="resp-var-name" timeout="in seconds" template="Liquid" content-type="application/json">
    <metadata>
        <item key="item-name"><!-- item-value --></item>
    </metadata>
    <data>
        <!-- message content -->
    </data>
</invoke-dapr-binding>
```

### <a name="examples"></a>Примеры

#### <a name="example"></a>Пример

В следующем примере демонстрируется запуск привязки исходящей связи с именем "внешние системы" с именем операции "Create", метаданные, состоящие из двух элементов "ключ-значение" с именами "Source" и "Client-IP", и текст, поступающий из исходного запроса. Ответ, полученный от среды выполнения ДАПР, записывается в запись "Bind-Response" коллекции Variables в объекте [контекста](api-management-policy-expressions.md#ContextVariables) .

Если среда выполнения ДАПР по какой-либо причине завершается с ошибкой, вызывается раздел "On-Error", и ответ, полученный от среды выполнения ДАПР, возвращается непосредственно вызывающему объекту. В противном случае возвращается ответ по умолчанию `200 OK` .

Раздел "Серверная часть" пуст, а запрос не передается серверной части.

```xml
<policies>
     <inbound>
        <base />
        <invoke-dapr-binding
                      name="external-system"
                      operation="create"
                      response-variable-name="bind-response">
            <metadata>
                <item key="source">api-management</item>
                <item key="client-ip">@( context.Request.IpAddress )</item>
            </metadata>
            <data>
                @( context.Request.Body.As<string>() )
            </data>
        </invoke-dapr-binding>
    </inbound>
    <backend>
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
        <return-response response-variable-name="bind-response" />
    </on-error>
</policies>
```

### <a name="elements"></a>Элементы

| Элемент             | Описание  | Обязательно |
|---------------------|--------------|----------|
| Invoke-ДАПР — привязка | Корневой элемент | Да      |
| метаданные            | Привязка конкретных метаданных в виде пар «ключ-значение». Сопоставляется со свойством [метаданных](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) в ДАПР. | Нет |
| .            | Содержимое сообщения. Сопоставляется со свойством [Data](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) в ДАПР. | Нет |


### <a name="attributes"></a>Атрибуты

| Атрибут        | Описание                     | Обязательно | По умолчанию |
|------------------|---------------------------------|----------|---------|
| name            | Имя целевой привязки. Должно совпадать с именем привязок, [определенных](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#bindings-structure) в ДАПР.           | Да      | Н/Д     |
| Операция       | Имя целевой операции (только для привязки). Сопоставляется со свойством [Operation](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) в ДАПР. | Нет | None |
| ignore-error     | Если задано значение, `true` то политика не активирует раздел ["On-Error"](api-management-error-handling-policies.md) при получении ошибки из среды выполнения ДАПР | Нет | `false` |
| response-variable-name | Имя записи коллекции [Variables](api-management-policy-expressions.md#ContextVariables) , используемой для хранения ответа от среды выполнения ДАПР | Нет | None |
| timeout | Время (в секундах) ожидания ответа среды выполнения ДАПР. Может находиться в диапазоне от 1 до 240 секунд. | Нет | 5 |
| шаблон | Обработчик шаблонов, используемый для преобразования содержимого сообщения. "Жидкость" — единственное поддерживаемое значение. | Нет | None |
| content-type | Тип содержимого сообщения. Единственное поддерживаемое значение — "Application/JSON". | Нет | None |

### <a name="usage"></a>Использование

Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

- **Разделы политики:** inbound, outbound, on-error.
- **Области политики:** все области.

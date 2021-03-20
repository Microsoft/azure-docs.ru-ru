---
title: Использование службы управления API для создания HTTP-запросов
description: Узнайте, как использовать политики запросов и ответов в службе управления API для вызова внешних служб из API.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 4539c0fa-21ef-4b1c-a1d4-d89a38c242fa
ms.service: api-management
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/15/2016
ms.author: apimpm
ms.openlocfilehash: 2f4bd040d7e5858fd561444f56dbce7b3f940d9a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92742397"
---
# <a name="using-external-services-from-the-azure-api-management-service"></a>Использование внешних служб из службы управления API Azure
Политики, доступные в службе управления API Azure, позволяют выполнять множество полезных задач исключительно на основе входящих запросов, исходящих ответов и сведений о базовой конфигурации. Однако возможность взаимодействия с внешними службами управления из политик управления API предоставляет гораздо больше преимуществ.

Вы уже имеете представление о взаимодействии со [службой концентратора событий Azure для ведения журнала, мониторинга и анализа](api-management-log-to-eventhub-sample.md). В этой статье вы узнаете о политиках, которые позволяют работать с любой внешней HTTP-службой. Эти политики можно использовать для запуска удаленных событий или для получения данных, которые определенным образом применяются для обработки исходного запроса и ответа.

## <a name="send-one-way-request"></a>Отправка одностороннего запроса
Возможно, самым простым вариантом внешнего взаимодействия является автономный запрос, который позволяет уведомлять внешнюю службу о каком-либо важном событии. Политику потока управления `choose` можно использовать для обнаружения любых интересующих вас условий.  Если условие выполняется, можно сделать внешний HTTP-запрос, используя политику [отправки одностороннего запроса](./api-management-advanced-policies.md#SendOneWayRequest). Это может быть запрос к системе обмена сообщениями, например Hipchat или Slack, или к почтовому API, например SendGrid или MailChimp, или для обращения в службу поддержки критических инцидентов, например PagerDuty. Все эти системы обмена сообщениями обладают простыми API-интерфейсами HTTP, которые могут быть вызваны.

### <a name="alerting-with-slack"></a>Оповещения с использованием Slack
В следующем примере демонстрируется отправка сообщения в чат Slack, если код состояния HTTP-ответа больше или равен 500. Ошибка в диапазоне 500 указывает на наличие проблемы с API серверной части, которую клиент API не может устранить самостоятельно. Обычно требуется вмешательство со стороны службы управления API.  

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

В системе Slack используется понятие входящих веб-привязок. При настройке входящей веб-привязки Slack создает специальный URL-адрес, который служит для выполнения простого запроса POST и передачи сообщения в канал Slack. Создаваемый текст JSON основан на формате, определенном системой Slack.

![Веб-привязка Slack](./media/api-management-sample-send-request/api-management-slack-webhook.png)

### <a name="is-fire-and-forget-good-enough"></a>Достаточно ли хорош автономный запрос?
Использование автономного запроса связано с некоторыми компромиссами. Если по какой-либо причине запрос завершается ошибкой, сообщение о сбое не выводится. В этой конкретной ситуации сложности, связанные с необходимостью наличия вторичной системы сообщений о сбоях и дополнительными издержками при ожидании ответа, являются неоправданными. В ситуациях, где проверка ответа имеет важное значение, лучшим вариантом будет политика [запроса на отправку](./api-management-advanced-policies.md#SendRequest) .

## <a name="send-request"></a>запроса на отправку
Политика `send-request` позволяет использовать внешнюю службу для выполнения сложных действий по обработке и возврату данных в службу управления API для дальнейшей обработки политики.

### <a name="authorizing-reference-tokens"></a>Авторизация маркеров ссылок
Основная задача API управления состоит в защите серверных ресурсов. Если сервер авторизации, используемый API-интерфейсом, создает [маркеры JWT](https://jwt.io/) в рамках потока OAuth2, как это делается в [Azure Active Directory](../active-directory/hybrid/whatis-hybrid-identity.md), с помощью политики `validate-jwt` можно проверить допустимость маркера. Некоторые серверы авторизации создают так называемые [маркеры ссылок](https://leastprivilege.com/2015/11/25/reference-tokens-and-introspection/), для проверки которых требуется обратный вызов к серверу авторизации.

### <a name="standardized-introspection"></a>Стандартизованный самоанализ
Раньше на сервере авторизации не было стандартизированного способа проверки маркера ссылки. Однако группа IETF опубликовала недавно предложенный стандарт [RFC 7662](https://tools.ietf.org/html/rfc7662) , определяющий метод проверки допустимости маркера сервером ресурсов.

### <a name="extracting-the-token"></a>Извлечение маркера
Первым действием является извлечение маркера из заголовка авторизации. Значение заголовка должно быть отформатировано с помощью схемы авторизации `Bearer` — согласно [RFC 6750](https://tools.ietf.org/html/rfc6750#section-2.1) сначала указывается одиночный пробел, за которым следует маркер авторизации. К сожалению, существуют случаи, когда схема авторизации опускается. Чтобы учесть этот момент во время анализа, служба управления API разделяет значение заголовка и в возвращенном массиве строк выбирает последнюю строку. Это возможное решение позволяет справиться с неправильно отформатированными заголовками авторизации.

```xml
<set-variable name="token" value="@(context.Request.Headers.GetValueOrDefault("Authorization","scheme param").Split(' ').Last())" />
```

### <a name="making-the-validation-request"></a>Выполнение запроса на проверку
После того как служба управления API получила маркер авторизации, она может выполнить запрос на проверку маркера. Согласно RFC 7662 этот процесс называется самоанализом и HTML-форму требуется отправить в ресурс самоанализа с помощью `POST`. HTML-форма должна содержать по крайней мере пару "ключ-значение" с ключом `token`. Этот запрос к серверу авторизации также должен пройти проверку подлинности, чтобы гарантировать, что вредоносным клиентам не удается найти допустимые маркеры.

```xml
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
```

### <a name="checking-the-response"></a>Проверка ответа
Для предоставления доступа к возвращаемому ответу используется атрибут `response-variable-name` . Имя, определенное в этом свойстве, можно использовать в качестве ключа к словарю `context.Variables` для доступа к объекту `IResponse`.

Из объекта ответа можно извлечь текст, а согласно RFC 7622 служба управления API ожидает ответ в виде объекта JSON, содержащий как минимум свойство с именем `active`, которое является логическим значением. Если `active` имеет значение "true", маркер считается допустимым.

Кроме того, если сервер авторизации не включает поле "Active", чтобы указать, является ли маркер допустимым, используйте такой инструмент, как POST, чтобы определить, какие свойства установлены в допустимом токене. Например, если допустимый ответ маркера содержит свойство с именем "expires_in", проверьте, существует ли это имя свойства в ответе сервера авторизации таким образом:

<при условии = "@ ((Иреспонсе) context. Переменные ["токенстате"]). Body.As <JObject> (). Свойство ("expires_in") = = null) ">

### <a name="reporting-failure"></a>Отчеты об ошибках
Для обнаружения недопустимого маркера используется политика `<choose>`, и если маркер является недопустимым, возвращается ответ 401.

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

Согласно документу [RFC 6750](https://tools.ietf.org/html/rfc6750#section-3), содержащему сведения об использовании маркеров `bearer`, вместе с ответом 401 служба управления API также возвращает заголовок `WWW-Authenticate`. WWW-Authenticate предназначен для предоставления клиенту сведений о создании надлежащим образом авторизованного запроса. Наличие множества разнообразных подходов, действующих в рамках платформы OAuth2, делает передачу всех необходимых данных довольно сложной задачей. К счастью, существуют возможности, позволяющие [клиентам узнать о правильной авторизации запросов к серверу ресурсов](https://tools.ietf.org/html/draft-jones-oauth-discovery-00).

### <a name="final-solution"></a>Окончательное решение
В итоге вы получите следующую политику:

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
              <return-response response-variable-name="existing response variable">
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

Это только один из многочисленных примеров использования политики `send-request` для интеграции полезных внешних служб в процесс выполнения запросов и получения ответов, реализуемый через службу управления API.

## <a name="response-composition"></a>Структура ответа
Политику `send-request` можно использовать для совершенствования основного запроса к серверной системе (как показано в предыдущем примере) или в качестве полной замены вызова серверной части. С помощью этой методики можно легко создавать сложные ресурсы, которые объединяются из нескольких разных систем.

### <a name="building-a-dashboard"></a>Создание панели мониторинга
Иногда требуется возможность предоставлять информацию, которая существует в нескольких серверных системах, для улучшения взаимодействия с панелью мониторинга. Ключевые показатели эффективности поставляются из разных серверных систем, но вы не хотите предоставлять к ним прямой доступ и планируете получать все данные в одном запросе. Возможно, некоторые извлеченные сведения потребуется сначала секционировать, фрагментировать и немного очистить. Возможность кэширования составного ресурса будет полезна для снижения нагрузки серверной части, поскольку вам известно, что пользователи имеют привычку многократно нажимать клавишу F5, чтобы увидеть изменение показателей низкой производительности.    

### <a name="faking-the-resource"></a>Создание ресурса
Первым шагом в создании ресурса панели мониторинга является настройка новой операции на портале Azure. Это операция с заполнителем, используемая для настройки политики построения для создания динамического ресурса.

![Операция панели мониторинга](./media/api-management-sample-send-request/api-management-dashboard-operation.png)

### <a name="making-the-requests"></a>Выполнение запросов
После создания операции можно настроить политику специально для этой операции. 

![Снимок экрана, на котором показан экран области политики.](./media/api-management-sample-send-request/api-management-dashboard-policy.png)

Первым шагом является извлечение всех параметров запроса из входящего запроса, чтобы их можно было переслать в серверную часть. В этом примере на панели мониторинга сведения отображаются на основе определенного периода времени, поэтому здесь присутствуют параметры `fromDate` и `toDate`. С помощью политики `set-variable` можно извлечь сведения из URL-адреса запроса.

```xml
<set-variable name="fromDate" value="@(context.Request.Url.Query["fromDate"].Last())">
<set-variable name="toDate" value="@(context.Request.Url.Query["toDate"].Last())">
```

После получения этой информации можно выполнять запросы ко всем серверным системам. Каждый запрос создает новый URL-адрес со сведениями о параметрах, вызывает соответствующий сервер и сохраняет ответ в переменной контекста.

```xml
<send-request mode="new" response-variable-name="revenuedata" timeout="20" ignore-error="true">
  <set-url>@($"https://accounting.acme.com/salesdata?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>

<send-request mode="new" response-variable-name="materialdata" timeout="20" ignore-error="true">
  <set-url>@($"https://inventory.acme.com/materiallevels?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>

<send-request mode="new" response-variable-name="throughputdata" timeout="20" ignore-error="true">
<set-url>@($"https://production.acme.com/throughput?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>

<send-request mode="new" response-variable-name="accidentdata" timeout="20" ignore-error="true">
<set-url>@($"https://production.acme.com/accidentdata?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>
```

Эти запросы выполняются последовательно, что не очень удобно. 

### <a name="responding"></a>Ответы на запросы
Для формирования составного запроса можно использовать политику [возврата ответа](./api-management-advanced-policies.md#ReturnResponse). Элемент `set-body` может применять выражение для создания нового `JObject` со всеми представлениями компонентов, внедренными в качестве свойств.

```xml
<return-response response-variable-name="existing response variable">
  <set-status code="200" reason="OK" />
  <set-header name="Content-Type" exists-action="override">
    <value>application/json</value>
  </set-header>
  <set-body>
    @(new JObject(new JProperty("revenuedata",((IResponse)context.Variables["revenuedata"]).Body.As<JObject>()),
                  new JProperty("materialdata",((IResponse)context.Variables["materialdata"]).Body.As<JObject>()),
                  new JProperty("throughputdata",((IResponse)context.Variables["throughputdata"]).Body.As<JObject>()),
                  new JProperty("accidentdata",((IResponse)context.Variables["accidentdata"]).Body.As<JObject>())
                  ).ToString())
  </set-body>
</return-response>
```

Законченная политика выглядит следующим образом:

```xml
<policies>
    <inbound>

  <set-variable name="fromDate" value="@(context.Request.Url.Query["fromDate"].Last())">
  <set-variable name="toDate" value="@(context.Request.Url.Query["toDate"].Last())">

    <send-request mode="new" response-variable-name="revenuedata" timeout="20" ignore-error="true">
      <set-url>@($"https://accounting.acme.com/salesdata?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <send-request mode="new" response-variable-name="materialdata" timeout="20" ignore-error="true">
      <set-url>@($"https://inventory.acme.com/materiallevels?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <send-request mode="new" response-variable-name="throughputdata" timeout="20" ignore-error="true">
    <set-url>@($"https://production.acme.com/throughput?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <send-request mode="new" response-variable-name="accidentdata" timeout="20" ignore-error="true">
    <set-url>@($"https://production.acme.com/accidentdata?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <return-response response-variable-name="existing response variable">
      <set-status code="200" reason="OK" />
      <set-header name="Content-Type" exists-action="override">
        <value>application/json</value>
      </set-header>
      <set-body>
        @(new JObject(new JProperty("revenuedata",((IResponse)context.Variables["revenuedata"]).Body.As<JObject>()),
                      new JProperty("materialdata",((IResponse)context.Variables["materialdata"]).Body.As<JObject>()),
                      new JProperty("throughputdata",((IResponse)context.Variables["throughputdata"]).Body.As<JObject>()),
                      new JProperty("accidentdata",((IResponse)context.Variables["accidentdata"]).Body.As<JObject>())
                      ).ToString())
      </set-body>
    </return-response>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
</policies>
```

В конфигурации операции заполнителя можно настроить ресурс панели мониторинга для кэширования в течение как минимум часа. 

## <a name="summary"></a>Итоги
Служба управления API Azure предоставляет гибкие политики, выборочно применяемые к HTTP-трафику, и позволяет формировать серверные службы. Если вы хотите усовершенствовать имеющийся шлюз API за счет функций оповещения, проверки или создать новые сложные ресурсы на основе нескольких серверных служб, `send-request` и связанные политики предоставят вам широкий диапазон новых возможностей.

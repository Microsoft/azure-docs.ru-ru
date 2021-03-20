---
title: Мониторинг API с помощью управления API Azure, концентраторов событий и Моесиф
titleSuffix: Azure API Management
description: Пример приложения, демонстрирующий политику от журнала к концентратору, путем подключения Управления API Azure, Центров событий Azure и Moesif для ведения журнала и мониторинга HTTP-запросов
services: api-management
documentationcenter: ''
author: darrelmiller
manager: erikre
editor: ''
ms.assetid: c528cf6f-5f16-4a06-beea-fa1207541a47
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 01/23/2018
ms.author: apimpm
ms.openlocfilehash: abb9cbb73f8957cec2cb3240bbf186623b9b2ef9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88205507"
---
# <a name="monitor-your-apis-with-azure-api-management-event-hubs-and-moesif"></a>Мониторинг API-интерфейсов с помощью Управления API Azure, Центров событий и Moesif
[Служба управления API](api-management-key-concepts.md) предоставляет множество возможностей для улучшения обработки HTTP-запросов, адресованных API-интерфейсу HTTP. Однако запросы и ответы существуют очень недолго. Созданный запрос проходит через службу управления API на серверный API вашей службы. API обрабатывает запрос и отправляет ответ обратно потребителю API. Служба управления API частично сохраняет важные статистические данные об интерфейсах API и отображает их на панели мониторинга портала Azure, но вся остальная информация теряется безвозвратно.

Вы можете использовать политику [регистрации в концентраторах событий](../event-hubs/event-hubs-about.md), чтобы служба управления API отправляла любые нужные сведения о запросе и ответе в концентратор событий Azure. Существует ряд причин для создания событий из сообщений HTTP, отправляемых в API-интерфейсы. В частности, это происходит при создании журнала аудита обновлений, анализе использования, оповещении об исключениях или интеграции служб сторонних поставщиков.

В этой статье мы покажем, как зафиксировать полное сообщение о запросе и ответе HTTP, как отправить это сообщение в концентратор событий, а затем передать его в сторонние службы, которые выполняют ведение журналов и мониторинг HTTP-запросов.

## <a name="why-send-from-api-management-service"></a>Почему лучше отправлять сообщения из службы управления API?
Конечно, можно создать ПО промежуточного слоя HTTP, которое может подключаться к платформам HTTP API для фиксации HTTP-запросов и ответов и их передачи в системы ведения журналов и мониторинга. Но у этого подхода есть недостатки: ПО промежуточного слоя HTTP должно интегрироваться с серверной частью API и соответствовать платформе, на которой размещен API-интерфейс. Если используется несколько интерфейсов API, ПО промежуточного слоя следует развернуть на каждом из них. Зачастую серверный API-интерфейс невозможно обновить по той или иной причине.

Служба управления API Azure предоставляет вам централизованное решение для интеграции с инфраструктурой ведения журналов, не зависящее от платформы. Также оно обеспечит хорошую масштабируемость, в том числе благодаря возможностям [георепликации](api-management-howto-deploy-multi-region.md) службы управления API Azure.

## <a name="why-send-to-an-azure-event-hub"></a>Почему лучше отправлять сообщения в концентратор событий Azure?
Вполне логичным будет вопрос, почему создаваемая политика должна использовать Центры событий Azure. Есть много различных служб, в которых можно регистрировать запросы. Что мешает отправлять запросы сразу конечному получателю?  Такой вариант возможен. Однако при отправке запросов на регистрацию из службы управления API следует учитывать то, как сообщения из журнала влияют на производительность API. Постепенное увеличение нагрузки можно компенсировать, увеличив количество доступных экземпляров компонентов системы или используя георепликацию. Но короткие пиковые скачки трафика могут привести к задержке запросов, если при увеличении нагрузки замедляется обработка запросов к инфраструктуре ведения журнала.

Центры событий Azure рассчитаны на огромные объемы входящих данных. Их пропускная способность существенно превышает возможности большинства API-процессов по обработке HTTP-запросов. Концентратор событий выполняет роль сложного буфера между службой управления API и инфраструктурой, в которой хранятся и обрабатываются сообщения. Это гарантирует, что ограничения инфраструктуры ведения журналов не повлияют на производительность API.

Данные, переданные в концентратор событий, сохраняются там и ожидают обработки со стороны потребителей концентратора событий. Для концентратора событий не имеет значения, как обрабатывается сообщение, — он просто гарантирует успешную доставку сообщения.

Центры событий могут выполнять потоковую передачу событий сразу нескольким группам потребителей. Это позволяет обрабатывать события различным системам. Благодаря этому можно реализовать множество сценариев интеграции без увеличения задержек на обработку запросов в службе управления API. Службе в любом случае достаточно создать только одно событие.

## <a name="a-policy-to-send-applicationhttp-messages"></a>Политика отправки сообщений приложений и сообщений HTTP
Концентратор событий принимает данные события в виде простой строки с произвольным содержимым. Чтобы сформировать HTTP-запрос и отправить его в Центры событий, следует дополнить строку информацией о запросе или ответе. В таких случаях можно повторно использовать уже существующий формат, чтобы не писать новый код для синтаксического анализа. Сначала мне показалось, что для отправки HTTP-запросов и ответов подойдет формат [HAR](http://www.softwareishard.com/blog/har-12-spec/). Но этот формат оптимизирован для сохранения последовательности HTTP-запросов в формате JSON. Он содержит несколько обязательных элементов, которые создают излишние сложности при передаче HTTP-сообщения по сети.

Еще один вариант — тип носителя `application/http` , описанный в спецификации HTTP [RFC 7230](https://tools.ietf.org/html/rfc7230). Этот тип носителя имеет точно такой же формат, как и обычные HTTP-сообщения, отправляемые по сети, но позволяет разместить сообщение целиком в тексте другого HTTP-запроса. В нашем случае в тексте запроса мы и будем отправлять сообщение в Центры событий. В библиотеках [клиента Microsoft ASP.NET Web API 2.2](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/) уже есть средство синтаксического анализа для этого формата, которое преобразует его в собственные объекты `HttpRequestMessage` и `HttpResponseMessage`.

Чтобы создать такое сообщение, следует воспользоваться [выражениями политики](./api-management-policy-expressions.md) службы управления API Azure, которые используют синтаксис C#. Приведенная ниже политика будет отправлять сообщение с HTTP-запросом в Центры событий Azure.

```xml
<log-to-eventhub logger-id="conferencelogger" partition-id="0">
@{
   var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                               context.Request.Method,
                                               context.Request.Url.Path + context.Request.Url.QueryString);

   var body = context.Request.Body?.As<string>(true);
   if (body != null && body.Length > 1024)
   {
       body = body.Substring(0, 1024);
   }

   var headers = context.Request.Headers
                          .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                          .ToArray<string>();

   var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

   return "request:"   + context.Variables["message-id"] + "\n"
                       + requestLine + headerString + "\r\n" + body;
}
</log-to-eventhub>
```

### <a name="policy-declaration"></a>Объявление политики
Некоторые элементы этого выражения политики следует разъяснить. Политика регистрации в концентраторе событий имеет атрибут logger-id, который обозначает средство ведения журнала, созданное в службе управления API. Подробные сведения о том, как в службе управления API настроить средство ведения журналов в Центрах событий, можно найти в документе [Как регистрировать события в Центрах событий Azure при использовании службы управления API Azure](api-management-howto-log-event-hubs.md). Второй (необязательный) атрибут сообщает Центрам событий, в каком разделе нужно хранить сообщение. Центры событий используют разделы для поддержки масштабируемости. Для работы требуется не меньше двух разделов. Правильная последовательность доставки сообщений гарантируется только в пределах раздела. Если не указать, в каком разделе концентратор событий должен разместить сообщение, он использует их поочередно для равномерного распределения нагрузки. Но это может привести к тому, что сообщения будут обрабатываться не по порядку.

### <a name="partitions"></a>Секции
Чтобы обеспечить правильный порядок доставки сообщений потребителям, не теряя при этом преимущества распределения нагрузки по разделам, я решил отправлять сообщения с HTTP-запросами в один раздел, а сообщения с ответами — в другой. Это обеспечивает равномерное распределение нагрузки и одновременно гарантирует, что все запросы и ответы обрабатываются по порядку. Может случиться так, что ответ будет обработан раньше соответствующего запроса, но это не проблема. У нас есть отдельный механизм сопоставления запросов и ответов, и мы доподлинно знаем, что запросы всегда поступают раньше ответов.

### <a name="http-payloads"></a>Полезные данные HTTP
После создания `requestLine` мы проверяем, нужно ли обрезать текст запроса. Длина текста запроса ограничена 1024 символами. Это значение можно увеличить, но размер одного сообщения для концентратора событий не может превышать 256 КБ, поэтому есть вероятность, что некоторые HTTP-запросы не поместятся в одно сообщение. При ведении журнала и анализе значительный объем информации можно получить уже из строки запроса HTTP и заголовка. Кроме того, многие запросы к интерфейсам API имеют сравнительно короткий текст. Таким образом, потеря полезной информации от усечения больших фрагментов будет пренебрежимо мала по сравнению с экономией затрат на передачу, обработку и хранение полного текста запроса. И еще одно замечание об обработке текста запроса: в метод `As<string>()` необходимо передать значение `true`, так как мы можем читать текст запроса, но его должна прочитать еще и серверная часть API. Если передать в этот метод значение true, текст запроса сохраняется в буфере и может быть прочитан повторно. Об этом особенно важно помнить, если ваш API передает файлы большого объема или долгие опросы. В таких случаях лучше вовсе не читать текст запроса.

### <a name="http-headers"></a>Заголовки HTTP
HTTP-заголовки можно поместить в сообщение в простом формате пар "ключ— значение". Мы решили исключить из обработки некоторые поля с секретной информацией, чтобы избежать утечки учетных данных. Для анализа ключи API и другие учетные данные, скорее всего, не потребуются. Если позднее нам нужно будет проанализировать работу конкретного пользователя или конкретного продукта, мы можем получить нужные данные из объекта `context` и добавить их в сообщение.

### <a name="message-metadata"></a>Метаданные сообщения
В полном тексте сообщения, которое будет отправлено в концентратор событий, первая строка не является частью сообщения `application/http`. В первой строке содержатся дополнительные метаданные. Они указывают, является ли сообщение запросом или ответом, и содержат идентификатор сообщения, по которому можно сопоставить запросы и ответы. Идентификатор сообщения создается с помощью другой политики, которая выглядит следующим образом:

```xml
<set-variable name="message-id" value="@(Guid.NewGuid())" />
```

Мы могли бы создать сообщение с запросом, сохранить его в переменной до получения ответа, а затем отправить запрос и ответ в одном сообщении. Но раздельная отправка запроса и ответа с общим идентификатором для их сопоставления позволяет варьировать размер сообщения. Также мы получаем возможность использовать преимущества нескольких разделов с сохранением порядка сообщений. Кроме того, так мы быстрее увидим поступивший запрос в панели мониторинга журналов. Также возможны ситуации, когда допустимый ответ вообще не отправляется в концентратор событий, например в случае неустранимой ошибки запроса в службе управления API. В этом случае мы по крайней мере имеем в журнале запись о запросе.

Политика отправки HTTP-сообщения с ответом похожа на политику для запроса, поэтому полная конфигурация политики выглядит так:

```xml
<policies>
  <inbound>
      <set-variable name="message-id" value="@(Guid.NewGuid())" />
      <log-to-eventhub logger-id="conferencelogger" partition-id="0">
      @{
          var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                                      context.Request.Method,
                                                      context.Request.Url.Path + context.Request.Url.QueryString);

          var body = context.Request.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Request.Headers
                               .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                               .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                               .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "request:"   + context.Variables["message-id"] + "\n"
                              + requestLine + headerString + "\r\n" + body;
      }
  </log-to-eventhub>
  </inbound>
  <backend>
      <forward-request follow-redirects="true" />
  </backend>
  <outbound>
      <log-to-eventhub logger-id="conferencelogger" partition-id="1">
      @{
          var statusLine = string.Format("HTTP/1.1 {0} {1}\r\n",
                                              context.Response.StatusCode,
                                              context.Response.StatusReason);

          var body = context.Response.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Response.Headers
                                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                                          .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "response:"  + context.Variables["message-id"] + "\n"
                              + statusLine + headerString + "\r\n" + body;
     }
  </log-to-eventhub>
  </outbound>
</policies>
```

Политика `set-variable` создает значение, которое политика `log-to-eventhub` может использовать в разделах `<inbound>` и `<outbound>`.

## <a name="receiving-events-from-event-hubs"></a>Получение событий от Центров событий
События из концентратора событий Azure поступают по [протоколу AMQP](https://www.amqp.org/). Команда служебной шины Microsoft предоставила в открытом доступе клиентские библиотеки, которые упрощают обработку событий. Служба поддерживает два различных подхода: *прямой потребитель* и класс `EventProcessorHost`. Примеры использования этих подходов можно найти в [руководстве по программированию Центров событий](../event-hubs/event-hubs-programming-guide.md). Вкратце рассмотрим различия между ними. `Direct Consumer` дает вам полный контроль, а `EventProcessorHost` берет часть работы на себя, но делает некоторые предположения о том, как вы обрабатываете события.

### <a name="eventprocessorhost"></a>EventProcessorHost
В этом примере мы для простоты используем `EventProcessorHost`, хоть это и не лучший выбор в данном случае. `EventProcessorHost` выполняет самую сложную работу по управлению потоками для конкретного класса обработчика событий. Но в нашем случае мы просто преобразуем сообщение в другой формат и передаем его в другую службу, используя асинхронный метод. У нас нет необходимости обновлять общее состояние, и нет риска возникновения проблем с потоками. В большинстве случаев класс `EventProcessorHost` будет оптимальным вариантом, как минимум самым простым в использовании.

### <a name="ieventprocessor"></a>IEventProcessor
Основной задачей при использовании `EventProcessorHost` является реализация интерфейса `IEventProcessor`, который содержит метод `ProcessEventAsync`. Суть этого метода показана ниже.

```csharp
async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
{

    foreach (EventData eventData in messages)
    {
        _Logger.LogInfo(string.Format("Event received from partition: {0} - {1}", context.Lease.PartitionId,eventData.PartitionKey));

        try
        {
            var httpMessage = HttpMessage.Parse(eventData.GetBodyStream());
            await _MessageContentProcessor.ProcessHttpMessage(httpMessage);
        }
        catch (Exception ex)
        {
            _Logger.LogError(ex.Message);
        }
    }
    ... checkpointing code snipped ...
}
```

Список объектов EventData передается в наш метод, где мы последовательно перебираем элементы этого списка. Содержимое каждого метода преобразуется в объект HttpMessage, и этот объект передается в экземпляр IHttpMessageProcessor.

### <a name="httpmessage"></a>HttpMessage
Экземпляр `HttpMessage` содержит три фрагмента данных:

```csharp
public class HttpMessage
{
    public Guid MessageId { get; set; }
    public bool IsRequest { get; set; }
    public HttpRequestMessage HttpRequestMessage { get; set; }
    public HttpResponseMessage HttpResponseMessage { get; set; }

... parsing code snipped ...

}
```

Экземпляр `HttpMessage` содержит идентификатор GUID `MessageId`, который позволяет связать HTTP-запрос с соответствующим ответом, а также логическое значение, которое сообщает, содержит ли этот объект экземпляр HttpRequestMessage и HttpResponseMessage. Я применил встроенные классы HTTP из `System.Net.Http`, что позволило воспользоваться кодом синтаксического анализа `application/http`, который включен в `System.Net.Http.Formatting`.  

### <a name="ihttpmessageprocessor"></a>IHttpMessageProcessor
Затем экземпляр `HttpMessage` передается в реализацию `IHttpMessageProcessor`, который я создал в качестве интерфейса, чтобы отделить получение и интерпретацию события из концентратора событий Azure от фактической обработки этого события.

## <a name="forwarding-the-http-message"></a>Пересылка сообщения HTTP
Мне показалось интересным использовать в нашем примере отправку HTTP-запроса в [API Analytics Moesif](https://www.moesif.com). Moesif — это облачная служба, которая специализируется на анализе и отладке протокола HTTP. У службы есть бесплатный уровень, поэтому ее легко испытать в работе. Мы сможем в реальном времени отслеживать HTTP-запросы, которые проходят через службу управления API.

Реализация `IHttpMessageProcessor` выглядит следующим образом:

```csharp
public class MoesifHttpMessageProcessor : IHttpMessageProcessor
{
    private readonly string RequestTimeName = "MoRequestTime";
    private MoesifApiClient _MoesifClient;
    private ILogger _Logger;
    private string _SessionTokenKey;
    private string _ApiVersion;
    public MoesifHttpMessageProcessor(ILogger logger)
    {
        var appId = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-APP-ID", EnvironmentVariableTarget.Process);
        _MoesifClient = new MoesifApiClient(appId);
        _SessionTokenKey = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-SESSION-TOKEN", EnvironmentVariableTarget.Process);
        _ApiVersion = Environment.GetEnvironmentVariable("APIMEVENTS-MOESIF-API-VERSION", EnvironmentVariableTarget.Process);
        _Logger = logger;
    }

    public async Task ProcessHttpMessage(HttpMessage message)
    {
        if (message.IsRequest)
        {
            message.HttpRequestMessage.Properties.Add(RequestTimeName, DateTime.UtcNow);
            return;
        }

        EventRequestModel moesifRequest = new EventRequestModel()
        {
            Time = (DateTime) message.HttpRequestMessage.Properties[RequestTimeName],
            Uri = message.HttpRequestMessage.RequestUri.OriginalString,
            Verb = message.HttpRequestMessage.Method.ToString(),
            Headers = ToHeaders(message.HttpRequestMessage.Headers),
            ApiVersion = _ApiVersion,
            IpAddress = null,
            Body = message.HttpRequestMessage.Content != null ? System.Convert.ToBase64String(await message.HttpRequestMessage.Content.ReadAsByteArrayAsync()) : null,
            TransferEncoding = "base64"
        };

        EventResponseModel moesifResponse = new EventResponseModel()
        {
            Time = DateTime.UtcNow,
            Status = (int) message.HttpResponseMessage.StatusCode,
            IpAddress = Environment.MachineName,
            Headers = ToHeaders(message.HttpResponseMessage.Headers),
            Body = message.HttpResponseMessage.Content != null ? System.Convert.ToBase64String(await message.HttpResponseMessage.Content.ReadAsByteArrayAsync()) : null,
            TransferEncoding = "base64"
        };

        Dictionary<string, string> metadata = new Dictionary<string, string>();
        metadata.Add("ApimMessageId", message.MessageId.ToString());

        EventModel moesifEvent = new EventModel()
        {
            Request = moesifRequest,
            Response = moesifResponse,
            SessionToken = _SessionTokenKey != null ? message.HttpRequestMessage.Headers.GetValues(_SessionTokenKey).FirstOrDefault() : null,
            Tags = null,
            UserId = null,
            Metadata = metadata
        };

        Dictionary<string, string> response = await _MoesifClient.Api.CreateEventAsync(moesifEvent);

        _Logger.LogDebug("Message forwarded to Moesif");
    }

    private static Dictionary<string, string> ToHeaders(HttpHeaders headers)
    {
        IEnumerable<KeyValuePair<string, IEnumerable<string>>> enumerable = headers.GetEnumerator().ToEnumerable();
        return enumerable.ToDictionary(p => p.Key, p => p.Value.GetEnumerator()
                                                         .ToEnumerable()
                                                         .ToList()
                                                         .Aggregate((i, j) => i + ", " + j));
    }
}
```

`MoesifHttpMessageProcessor` использует [библиотеку API C# для Moesif](https://www.moesif.com/docs/api?csharp#events), которая упрощает передачу данных событий HTTP в их службу. Для отправки данных HTTP в API сборщика Моесиф требуется учетная запись и идентификатор приложения. Вы получаете идентификатор приложения моесиф, создав учетную запись на [веб-сайте моесиф](https://www.moesif.com) , а затем перейдем к установке приложения в _меню справа_  ->  .

## <a name="complete-sample"></a>Полный пример
[Исходный код](https://github.com/dgilling/ApimEventProcessor) и тесты для этого примера доступны на сайте GitHub. Чтобы запустить этот пример, вам потребуются [служба управления API](get-started-create-service-instance.md), [подключенный к ней концентратор событий](api-management-howto-log-event-hubs.md) и [учетная запись хранения](../storage/common/storage-account-create.md).   

Пример представляет собой простое консольное приложение, которое прослушивает события, поступающие от концентратора событий, преобразует их в объекты Moesif `EventRequestModel` и `EventResponseModel`, а затем пересылает их на API сборщика Moesif.

На следующем анимированном изображении вы увидите запрос к API на Портале разработчика, получение, обработку и пересылку сообщений в консольном приложении, а также запрос и ответ, отображающиеся в потоке события.

![Демонстрация передачи запроса в Runscope](./media/api-management-log-to-eventhub-sample/apim-eventhub-runscope.gif)

## <a name="summary"></a>Итоги
Служба управления API Azure— это идеальное место для фиксации HTTP-трафика, поступающего на API-интерфейсы и в обратном направлении. Центры событий Azure — это высокомасштабируемое недорогое решение, которое собирает информацию о трафике и передает ее в системы дополнительной обработки для регистрации, мониторинга и сложного анализа. Для подключения к сторонним системам мониторинга трафика, например Moesif, потребуется всего несколько десятков строк кода.

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительная информация о Центрах событий Azure
  * [Приступая к работе с Центрами событий Azure](../event-hubs/event-hubs-c-getstarted-send.md)
  * [Прием сообщений через EventProcessorHost](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)
  * [Руководство по программированию Центров событий](../event-hubs/event-hubs-programming-guide.md)
* Дополнительные сведения об интеграции службы управления API и Центров событий
  * [Как регистрировать события в Центрах событий Azure при использовании службы управления API Azure](api-management-howto-log-event-hubs.md)
  * [Справочник по сущности "Средство ведения журнала"](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-logger-entity)
  * [Справочник по политике регистрации в концентраторе событий](./api-management-advanced-policies.md#log-to-eventhub)

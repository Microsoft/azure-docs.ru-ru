---
title: Руководство по регулируемым запросам
description: Сведения о группировании, поочередной отправке, разбиении на страницы и распараллеливании запросов, чтобы избежать регулирования в Azure Resource Graph.
ms.date: 01/27/2021
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: ddd3cf4d411733e831c94039c3bc9aeaf0e95271
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98917713"
---
# <a name="guidance-for-throttled-requests-in-azure-resource-graph"></a>Рекомендации по работе с регулируемыми запросами в Azure Resource Graph

При создании средств для активного программного доступа к данным Azure Resource Graph следует учитывать, как на результаты запросов влияет регулирование. Изменив процесс запроса данных, вы и ваша организация сможете избежать регулирования и получать поток актуальных данных о ресурсах Azure.

В этой статье рассматриваются четыре области и шаблона, связанных с созданием запросов к Azure Resource Graph:

- общие сведения о заголовках регулирования;
- группирование запросов;
- поочередная отправка запросов;
- влияние разбиения на страницы.

## <a name="understand-throttling-headers"></a>Общие сведения о заголовках регулирования

Azure Resource Graph выделяет определенную квоту для каждого пользователя на определенный период времени. Например, пользователь не может отправить более 15 запросов в течение 5 секунд, иначе применяется регулирование. Значение квоты определяется множеством факторов и может изменяться.

В каждый ответ на запрос Azure Resource Graph добавляет следующие два заголовка регулирования:

- `x-ms-user-quota-remaining` (целое число): оставшаяся квота ресурсов для пользователя. Это значение соответствует количеству запросов.
- `x-ms-user-quota-resets-after` (чч:мм:сс): время, оставшееся до сброса квоты потребления этого пользователя.

Если субъект безопасности имеет доступ к более чем 5000 подпискам в [области запросов](./query-language.md#query-scope)клиента или группы управления, ответ ограничен первыми 5000 подписками, а `x-ms-tenant-subscription-limit-hit` заголовок возвращается как `true` .

Чтобы продемонстрировать работу этих заголовков, давайте рассмотрим ответ на запрос со следующими значениями заголовков: `x-ms-user-quota-remaining: 10` и `x-ms-user-quota-resets-after: 00:00:03`.

- Это означает, что за следующие 3 секунды можно без регулирования отправить не более 10 запросов.
- По прошествии этих трех секунд значения `x-ms-user-quota-remaining` и `x-ms-user-quota-resets-after` будут сброшены на `15` и `00:00:05` соответственно.

Пример использования заголовков для _откладывания_ запросов см. в разделе [Запрос в параллельном режиме](#query-in-parallel).

## <a name="grouping-queries"></a>Группирование запросов

Группирование запросов по подпискам, группам ресурсов или конкретному ресурсу работает более эффективно, чем параллельное выполнение. Более крупный запрос часто меньше влияет на квоту, чем множество мелких и конкретных запросов. Мы рекомендуем создавать группы размером не более _300_ элементов.

- Пример плохо оптимизированного подхода

  ```csharp
  // NOT RECOMMENDED
  var header = /* your request header */
  var subscriptionIds = /* A big list of subscriptionIds */

  foreach (var subscriptionId in subscriptionIds)
  {
      var userQueryRequest = new QueryRequest(
          subscriptions: new[] { subscriptionId },
          query: "Resoures | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

  // ...
  }
  ```

- Пример №1 по оптимизации подхода методом группирования

  ```csharp
  // RECOMMENDED
  var header = /* your request header */
  var subscriptionIds = /* A big list of subscriptionIds */

  const int groupSize = 100;
  for (var i = 0; i <= subscriptionIds.Count / groupSize; ++i)
  {
      var currSubscriptionGroup = subscriptionIds.Skip(i * groupSize).Take(groupSize).ToList();
      var userQueryRequest = new QueryRequest(
          subscriptions: currSubscriptionGroup,
          query: "Resources | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

    // ...
  }
  ```

- Пример №2 по оптимизации подхода методом группирования для получения нескольких ресурсов в одном запросе

  ```kusto
  Resources | where id in~ ({resourceIdGroup}) | project name, type
  ```

  ```csharp
  // RECOMMENDED
  var header = /* your request header */
  var resourceIds = /* A big list of resourceIds */

  const int groupSize = 100;
  for (var i = 0; i <= resourceIds.Count / groupSize; ++i)
  {
      var resourceIdGroup = string.Join(",",
          resourceIds.Skip(i * groupSize).Take(groupSize).Select(id => string.Format("'{0}'", id)));
      var userQueryRequest = new QueryRequest(
          subscriptions: subscriptionList,
          query: $"Resources | where id in~ ({resourceIdGroup}) | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

    // ...
  }
  ```

## <a name="staggering-queries"></a>Поочередная отправка запросов

С учетом того, как применяется регулирование, есть смысл отправлять запросы поочередно. Например, лучше отправлять не 60 запросов сразу, а поэтапно в четырех 5-секундных окнах:

- Расписание без применения очередности запросов

  | Число запросов         | 60  | 0    | 0     | 0     |
  |---------------------|-----|------|-------|-------|
  | Интервал времени (с) | 0-5 | 5-10 | 10-15 | 15–20 |

- Расписание с применением очередности запросов

  | Число запросов         | 15  | 15   | 15    | 15    |
  |---------------------|-----|------|-------|-------|
  | Интервал времени (с) | 0-5 | 5-10 | 10-15 | 15–20 |

Ниже приведен пример использования заголовков регулирования при запросе к графу ресурсов Azure.

```csharp
while (/* Need to query more? */)
{
    var userQueryRequest = /* ... */
    // Send post request to Azure Resource Graph
    var azureOperationResponse = await this.resourceGraphClient
        .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
        .ConfigureAwait(false);

    var responseHeaders = azureOperationResponse.response.Headers;
    int remainingQuota = /* read and parse x-ms-user-quota-remaining from responseHeaders */
    TimeSpan resetAfter = /* read and parse x-ms-user-quota-resets-after from responseHeaders */
    if (remainingQuota == 0)
    {
        // Need to wait until new quota is allocated
        await Task.Delay(resetAfter).ConfigureAwait(false);
    }
}
```

### <a name="query-in-parallel"></a>Параллельная отправка запросов

Хотя мы рекомендуем чаще применять группирование, чем распараллеливание, в некоторых случаях группирование выполнить сложно. В таких случаях вы можете отправлять несколько запросов к Azure Resource Graph параллельно. Ниже приведен пример _того, как выполнить_ откладывание на основе заголовков регулирования в таких сценариях:

```csharp
IEnumerable<IEnumerable<string>> queryGroup = /* Groups of queries  */
// Run groups in parallel.
await Task.WhenAll(queryGroup.Select(ExecuteQueries)).ConfigureAwait(false);

async Task ExecuteQueries(IEnumerable<string> queries)
{
    foreach (var query in queries)
    {
        var userQueryRequest = new QueryRequest(
            subscriptions: subscriptionList,
            query: query);
        // Send post request to Azure Resource Graph.
        var azureOperationResponse = await this.resourceGraphClient
            .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
            .ConfigureAwait(false);
        
        var responseHeaders = azureOperationResponse.response.Headers;
        int remainingQuota = /* read and parse x-ms-user-quota-remaining from responseHeaders */
        TimeSpan resetAfter = /* read and parse x-ms-user-quota-resets-after from responseHeaders */
        if (remainingQuota == 0)
        {
            // Delay by a random period to avoid bursting when the quota is reset.
            var delay = (new Random()).Next(1, 5) * resetAfter;
            await Task.Delay(delay).ConfigureAwait(false);
        }
    }
}
```

## <a name="pagination"></a>Разбиение на страницы

Так как Azure Resource Graph возвращает не более 1000 записей в одном ответе на запрос, может потребоваться [разбиение на страницы](./work-with-data.md#paging-results), чтобы полностью получить требуемый набор данных. Но учитывайте, что клиенты Azure Resource Graph могут по-разному обрабатывать разбиение на страницы.

- Пакет SDK для C#

  При использовании пакета SDK ResourceGraph для поддержки разбиения на страницы нужно передавать в запросе маркер пропуска, полученный в ответе на предыдущий запрос. Такой механизм означает, что вам придется самостоятельно объединять результаты из всех вызовов, получающих страницы одного набора. В этом случае каждый запрос с разбиением на страницы расходует одну единицу квоты запросов:

  ```csharp
  var results = new List<object>();
  var queryRequest = new QueryRequest(
      subscriptions: new[] { mySubscriptionId },
      query: "Resources | project id, name, type | top 5000");
  var azureOperationResponse = await this.resourceGraphClient
      .ResourcesWithHttpMessagesAsync(queryRequest, header)
      .ConfigureAwait(false);
  while (!string.Empty(azureOperationResponse.Body.SkipToken))
  {
      queryRequest.SkipToken = azureOperationResponse.Body.SkipToken;
      // Each post call to ResourceGraph consumes one query quota
      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(queryRequest, header)
          .ConfigureAwait(false);
      results.Add(azureOperationResponse.Body.Data.Rows);

      // Inspect throttling headers in query response and delay the next call if needed.
  }
  ```

- Azure CLI или Azure PowerShell

  При использовании Azure CLI или Azure PowerShell запросы к Azure Resource Graph разбиваются на страницы автоматически, чтобы получать не более 5000 записей. Результаты запроса возвращаются единым списком после объединения всех результатов из отдельных вызовов. В этом случае один запрос, к которому применяется разбиение на страницы, может потреблять больше одной единицы квоты запросов (в зависимости от числа записей в его результатах). Например, в следующих примерах один запуск запроса может использовать до пяти квот запросов:

  ```azurecli-interactive
  az graph query -q 'Resources | project id, name, type' --first 5000
  ```

  ```azurepowershell-interactive
  Search-AzGraph -Query 'Resources | project id, name, type' -First 5000
  ```

## <a name="still-get-throttled"></a>Не удалось избавиться от регулирования?

Если даже после применения описанных выше рекомендаций вы сталкиваетесь с регулированием, обратитесь к разработчикам по адресу [resourcegraphsupport@microsoft.com](mailto:resourcegraphsupport@microsoft.com).

Укажите следующие сведения:

- Подробности о вашем варианте использования и бизнес-обоснование для повышения предела регулирования.
- К какому объему ресурсов у вас есть доступ? Сколько из них возвращаются одним запросом?
- Какие типы ресурсов вас интересуют?
- Какой шаблон запросов вы используете? Данные о числе выполненных запросов в секунду и т. п.

## <a name="next-steps"></a>Дальнейшие действия

- См. описание используемого языка в разделе [Запросы для начинающих](../samples/starter.md).
- См. описание расширенных вариантов использования в разделе [Расширенные запросы](../samples/advanced.md).
- Узнайте больше о том, как [изучать ресурсы](explore-resources.md).
---
title: Diagnose and troubleshoot issues when using Azure Cosmos DB .NET SDK (Диагностика и устранение неполадок при использовании пакета SDK Azure Cosmos DB для .NET)
description: Используйте такие функции, как ведение журнала на стороне клиента и другие сторонние средства для выявления, диагностики и устранения Azure Cosmos DB проблем при использовании пакета SDK для .NET.
author: anfeldma-ms
ms.service: cosmos-db
ms.date: 03/05/2021
ms.author: anfeldma
ms.subservice: cosmosdb-sql
ms.topic: troubleshooting
ms.reviewer: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: 1f7548b355353eb77419f4d1760b40ba02eeddda
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102442202"
---
# <a name="diagnose-and-troubleshoot-issues-when-using-azure-cosmos-db-net-sdk"></a>Diagnose and troubleshoot issues when using Azure Cosmos DB .NET SDK (Диагностика и устранение неполадок при использовании пакета SDK Azure Cosmos DB для .NET)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Пакет SDK для Java версии 4](troubleshoot-java-sdk-v4-sql.md)
> * [Пакет SDK для Async Java версии 2](troubleshoot-java-async-sdk.md)
> * [.NET](troubleshoot-dot-net-sdk.md)
> 

В этой статье рассматриваются распространенные проблемы, обходные пути, шаги диагностики и средства при использовании [пакета SDK для .NET](sql-api-sdk-dotnet.md) с учетными ЗАПИСЯМИ SQL API Azure Cosmos DB.
Пакет SDK для .NET предоставляет логическое представление на стороне клиента для доступа к Azure Cosmos DB API SQL. В этой статье описываются средства и подходы, которые помогут вам, если вы столкнетесь с проблемами.

## <a name="checklist-for-troubleshooting-issues"></a>Контрольный список для устранения проблем

Перед перемещением приложения в рабочую среду рассмотрите следующий контрольный список. Использование контрольного списка позволяет предотвратить несколько распространенных проблем, которые могут возникнуть. Кроме того, можно быстро диагностировать причины возникновения проблемы.

*    Используйте последнюю версию [пакета SDK](sql-api-sdk-dotnet-standard.md). Предварительные версии пакетов SDK не следует использовать для рабочей среды. Это предотвратит Устранение известных проблем, которые уже исправлены.
*    Просмотрите [советы по повышению производительности](performance-tips.md) и следуйте рекомендациям. Это поможет предотвратить масштабирование, задержку и другие проблемы с производительностью.
*    Включите ведение журнала пакета SDK, чтобы помочь устранить проблему. Включение ведения журнала может повлиять на производительность, поэтому рекомендуется включать его только при устранении неполадок. Можно включить следующие журналы:
*    [Метрики журнала](./monitor-cosmos-db.md) с помощью портал Azure. Метрики портала отображают данные телеметрии Azure Cosmos DB, что позволяет определить, соответствует ли эта ошибка Azure Cosmos DB или если она находится на стороне клиента.
*    Заносить [строку диагностики](/dotnet/api/microsoft.azure.documents.client.resourceresponsebase.requestdiagnosticsstring) в пакет SDK v2 или [ДИАГНОСТИКУ](/dotnet/api/microsoft.azure.cosmos.responsemessage.diagnostics) в пакете SDK версии 3 из ответов операции Point.
*    Регистрировать [метрики запросов SQL](sql-api-query-metrics.md) из всех ответов на запросы 
*    Следуйте указаниям по настройке [ведения журнала пакета SDK]( https://github.com/Azure/azure-cosmos-dotnet-v2/blob/master/docs/documentdb-sdk_capture_etl.md)

Ознакомьте с разделом [Распространенные проблемы и их обходные решения](#common-issues-workarounds) в этой статье.

Просмотрите [раздел проблем GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/issues) , который активно отслеживается. Вы можете там найти уже готовое обходное решение похожей проблемы. Если вы не нашли решение, подайте ему вопрос GitHub. Можно открыть такт поддержки для срочных проблем.


## <a name="common-issues-and-workarounds"></a><a name="common-issues-workarounds"></a>Распространенные проблемы и их обходные решения

### <a name="general-suggestions"></a>Общие рекомендации
* Если это возможно, запустите приложение в том же регионе Azure, что и учетная запись Azure Cosmos DB. 
* Вы можете столкнуться с проблемами подключения или доступности из-за недостатка ресурсов на клиентском компьютере. Мы рекомендуем отслеживать использование ЦП на узлах, на которых работает клиент Azure Cosmos DB, и масштабировать их, если они работают при высокой нагрузке.

### <a name="check-the-portal-metrics"></a>Проверка метрик портала
Проверка [метрик портала](./monitor-cosmos-db.md) поможет определить, является ли это проблемой на стороне клиента, или возникли проблемы со службой. Например, если метрики содержат высокую частоту запросов с ограниченным количеством записей (код состояния HTTP 429), то есть запрос регулируется, а затем проверяется [слишком большое значение частоты запросов](troubleshoot-request-rate-too-large.md) . 

## <a name="retry-logic"></a>Логика повторных попыток <a id="retry-logics"></a>
При любой ошибке ввода-вывода пакет SDK Cosmos DB попытается повторить неудачную операцию, если в нем поддерживаются повторные попытки. Наличие повторных попыток для любой ошибки является хорошей практикой, но, в частности, требуется обработка или повторная попытка записи. Рекомендуется использовать последний пакет SDK, так как логика повторных попыток постоянно улучшается.

1. Ошибки ввода-вывода при чтении и запросе будут повторяться пакетом SDK, не отображая их конечному пользователю.
2. Операции записи (Create, Upsert, Replace, DELETE) не являются идемпотентными, и поэтому пакет SDK не всегда может без каких-либо ошибок повторить операции записи. Необходимо, чтобы логика приложения пользователя обрабатывала ошибку и повторить попытку.
3. Устранение [проблем с доступностью пакета SDK](troubleshoot-sdk-availability.md) объясняет повторные попытки для учетных записей Cosmos DB в нескольких регионах.

## <a name="common-error-status-codes"></a>Коды распространенных состояний ошибок <a id="error-codes"></a>

| Код состояния | Описание | 
|----------|-------------|
| 400 | Недопустимый запрос (зависит от сообщения об ошибке)| 
| 401 | [Не санкционировано](troubleshoot-unauthorized.md) | 
| 403 | [Forbidden](troubleshoot-forbidden.md) |
| 404 | [Ресурс не найден](troubleshoot-not-found.md) |
| 408 | [Время ожидания запроса истекло](troubleshoot-dot-net-sdk-request-timeout.md) |
| 409 | Сбой конфликта заключается в том, что идентификатор, предоставленный для ресурса в операции записи, взят существующим ресурсом. Используйте другой идентификатор для ресурса, чтобы разрешить эту проблему, так как идентификатор должен быть уникальным в пределах всех документов с тем же значением ключа секции. |
| 410 | Пропали исключения (временный сбой, который не должен нарушать соглашение об уровне обслуживания) |
| 412 | Сбой предварительного условия — это место, в котором в операции указан eTag, отличный от версии, доступной на сервере. Это ошибка оптимистичного параллелизма. Повторите запрос после считывания последней версии ресурса и обновления eTag в запросе.
| 413 | [Сущность запроса слишком велика](concepts-limits.md#per-item-limits) |
| 429 | [Слишком много запросов](troubleshoot-request-rate-too-large.md) |
| 449 | Временная ошибка, которая возникает только при операциях записи и является надежной для повторных попыток |
| 500 | Не удалось выполнить операцию из-за непредвиденной ошибки службы. Обратитесь в службу поддержки. См. статью о [выпуске проблемы с поддержкой Azure](https://aka.ms/azure-support). |
| 503 | [Служба недоступна](troubleshoot-service-unavailable.md) | 

### <a name="azure-snat-pat-port-exhaustion"></a><a name="snat"></a>Нехватка портов SNAT (PAT) Azure

Если приложение развертывается на [виртуальных машинах Azure без общедоступного IP-адреса](../load-balancer/load-balancer-outbound-connections.md), [порты Azure SNAT](../load-balancer/load-balancer-outbound-connections.md#preallocatedports) по умолчанию устанавливают подключения к любой конечной точке за пределами виртуальной машины. Количество разрешенных подключений от виртуальной машины к конечной точке Azure Cosmos DB ограничивается [конфигурацией Azure SNAT](../load-balancer/load-balancer-outbound-connections.md#preallocatedports). Такая ситуация может привести к регулированию подключений, закрытию соединения или приведенным выше [таймаутам запросов](troubleshoot-dot-net-sdk-request-timeout.md).

 Порты Azure SNAT используются только в том случае, если виртуальная машина имеет частный IP-адрес, подключающийся к общедоступному IP-адресу. Существует два обходных решения для предотвращения ограничения Azure SNAT (при условии, что вы уже используете один экземпляр клиента во всем приложении):

* Добавьте конечную точку службы Azure Cosmos DB к подсети виртуальной сети для службы "Виртуальные машины Azure". Дополнительные сведения см. в статье [Конечные точки служб для виртуальной сети Azure](../virtual-network/virtual-network-service-endpoints-overview.md). 

    При включении конечной точки службы запросы больше не отправляются с общедоступного IP-адреса в Azure Cosmos DB. Вместо этого отправляются идентификаторы виртуальной сети и подсети. Это изменение может привести к сбою брандмауэра, если разрешены только общедоступные IP-адреса. Если вы используете брандмауэр, при включении конечной точки службы добавьте подсеть в брандмауэре с помощью [списков управления доступом для виртуальных сетей](/previous-versions/azure/virtual-network/virtual-networks-acl).
* Назначьте [общедоступный IP-адрес виртуальной машине Azure](../load-balancer/troubleshoot-outbound-connection.md#assignilpip).

### <a name="high-network-latency"></a><a name="high-network-latency"></a>Высокая задержка сети
Высокую задержку сети можно определить с помощью [строки диагностики](/dotnet/api/microsoft.azure.documents.client.resourceresponsebase.requestdiagnosticsstring) в пакете SDK v2 или [диагностики](/dotnet/api/microsoft.azure.cosmos.responsemessage.diagnostics#Microsoft_Azure_Cosmos_ResponseMessage_Diagnostics) в пакете SDK v3.

Значение, если [время ожидания](troubleshoot-dot-net-sdk-request-timeout.md) отсутствует, и диагностика отображает отдельные запросы, в которых очевидно большая задержка.

# <a name="v3-sdk"></a>[ПАКЕТ SDK V3](#tab/diagnostics-v3)

Диагностику можно получить из любого `ResponseMessage` , `ItemResponse` , `FeedResponse` или `CosmosException` `Diagnostics` Свойства:

```csharp
ItemResponse<MyItem> response = await container.CreateItemAsync<MyItem>(item);
Console.WriteLine(response.Diagnostics.ToString());
```

Взаимодействие с сетью в диагностике будет таким, например:

```json
{
    "name": "Microsoft.Azure.Documents.ServerStoreModel Transport Request",
    "id": "0e026cca-15d3-4cf6-bb07-48be02e1e82e",
    "component": "Transport",
    "start time": "12: 58: 20: 032",
    "duration in milliseconds": 1638.5957
}
```

Где `duration in milliseconds` будет отображаться задержка.

# <a name="v2-sdk"></a>[ПАКЕТ SDK ДЛЯ V2](#tab/diagnostics-v2)

Диагностика доступна, когда клиент настроен в [прямом режиме](sql-sdk-connection-modes.md), через `RequestDiagnosticsString` свойство:

```csharp
ResourceResponse<Document> response = await client.ReadDocumentAsync(documentLink, new RequestOptions() { PartitionKey = new PartitionKey(partitionKey) });
Console.WriteLine(response.RequestDiagnosticsString);
```

А задержка может быть в разности между `ResponseTime` и `RequestStartTime` :

```bash
RequestStartTime: 2020-03-09T22:44:49.5373624Z, RequestEndTime: 2020-03-09T22:44:49.9279906Z,  Number of regions attempted:1
ResponseTime: 2020-03-09T22:44:49.9279906Z, StoreResult: StorePhysicalAddress: rntbd://..., ...
```
--- 

Эта задержка может быть вызвана несколькими причинами:

* Приложение не работает в том же регионе, что и учетная запись Azure Cosmos DB.
* Конфигурация [PreferredLocations](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.preferredlocations) или [аппликатионрегион](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationregion) неверна и пытается подключиться к другому региону, на котором выполняется приложение.
* В связи с высоким трафиком сетевой интерфейс может быть узким местом. Если приложение выполняется на виртуальных машинах Azure, возможны следующие обходные пути:
    * Рассмотрите возможность использования [виртуальной машины с включенной функцией ускорения сети](../virtual-network/create-vm-accelerated-networking-powershell.md).
    * Включить [ускоренную сеть на существующей виртуальной машине](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms).
    * Рассмотрите возможность использования [виртуальной машины более высокого уровня](../virtual-machines/sizes.md).

### <a name="common-query-issues"></a>Распространенные проблемы с запросами

[Метрики запроса](sql-api-query-metrics.md) помогут определить, где в большинстве случаев тратится запрос. С помощью метрик запроса можно увидеть, какая часть ИТ-операций выполняется на серверной части и на клиенте. Дополнительные сведения об [устранении неполадок с производительностью запросов](troubleshoot-query-performance.md).

* Если серверный запрос возвращается быстро и тратит большое время на клиент, Проверьте нагрузку на компьютер. Скорее всего, недостаточно ресурсов, и пакет SDK ожидает, пока ресурсы будут доступны для обработки ответа.
* Если серверный запрос работает слишком долго, попробуйте [оптимизировать запрос](troubleshoot-query-performance.md) и просмотреть текущую [политику индексирования](index-overview.md) .

    > [!NOTE]
    > Для повышения производительности рекомендуется использовать обработку узлов 64-разрядной версии Windows. Пакет SDK для SQL содержит собственный файл ServiceInterop.dll для локального анализа и оптимизации запросов. ServiceInterop.dll поддерживается только на платформе Windows x64. Для Linux и других неподдерживаемых платформ, где ServiceInterop.dll недоступен, чтобы получить оптимизированный запрос, будет выполнен дополнительный сетевой вызов шлюза.

Если возникла следующая ошибка: `Unable to load DLL 'Microsoft.Azure.Cosmos.ServiceInterop.dll' or one of its dependencies:` и используют Windows, следует выполнить обновление до последней версии Windows.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о рекомендациях по производительности для [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) и [.NET v2](performance-tips.md)
* Узнайте о [пакетах средств разработки Java на основе Reactor](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-pattern-guide.md).

 <!--Anchors-->
[Common issues and workarounds]: #common-issues-workarounds
[Enable client SDK logging]: #logging
[Azure SNAT (PAT) port exhaustion]: #snat
[Production check list]: #production-check-list
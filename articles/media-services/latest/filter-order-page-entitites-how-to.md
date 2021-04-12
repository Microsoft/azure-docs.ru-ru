---
title: Фильтрация, упорядочение и разбиение сущностей на страницы
description: Сведения о фильтрации, упорядочении и разбиении на страницы сущностей Служб мультимедиа Azure версии 3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: overview
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: c2748bb879c016ce7d13e73602e9bc0604738dff
ms.sourcegitcommit: 9f4510cb67e566d8dad9a7908fd8b58ade9da3b7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106121189"
---
# <a name="filtering-ordering-and-paging-of-media-services-entities"></a>Фильтрация, упорядочивание и разбиение по страницам сущностей Служб мультимедиа

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этом разделе обсуждаются параметры запроса OData и поддержка разбиения на страницы, доступные при перечислении сущностей Служб мультимедиа Azure версии 3.

## <a name="considerations"></a>Рекомендации

* Свойства сущностей типа `Datetime` всегда задаются в формате UTC.
* Пробел в строке запроса должен быть закодирован в формате URL перед отправкой запроса.

## <a name="comparison-operators"></a>Операторы сравнения

Для сравнения поля с постоянным значением можно использовать следующие операторы:

Операторы равенства:

- `eq`: Проверяет, является ли поле *равным* постоянному значению.
- `ne`: Проверяет, является ли поле *не равным* постоянному значению.

Операторы диапазона:

- `gt`: Проверяет, *больше ли* поле постоянного значения.
- `lt`: Проверяет, *меньше ли* поле постоянного значения.
- `ge`: Проверяет, *больше или равно* поле постоянному значению.
- `le`: Проверяет, *меньше или равно* поле постоянному значению.

## <a name="filter"></a>Filter

Используйте `$filter`, чтобы указать параметр фильтра OData для поиска только интересующих вас объектов.

В приведенных ниже примерах фильтрация выполняется по значению `alternateId` ресурса.

```
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01&$filter=properties/alternateId%20eq%20'unique identifier'
```

В следующем примере C# выполняется фильтрация по дате создания ресурса:

```csharp
var odataQuery = new ODataQuery<Asset>("properties/created lt 2018-05-11T17:39:08.387Z");
var firstPage = await MediaServicesArmClient.Assets.ListAsync(CustomerResourceGroup, CustomerAccountName, odataQuery);
```

## <a name="order-by"></a>Упорядочить по

Используйте `$orderby`, чтобы отсортировать возвращенные объекты по указанному параметру. Например:  

```
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01$orderby=properties/created%20gt%202018-05-11T17:39:08.387Z
```

Чтобы отсортировать результаты по возрастанию или по убыванию, добавьте в имя поля `asc` или `desc`, разделяя их пробелом. Например: `$orderby properties/created desc`.

## <a name="skip-token"></a>Маркер пропуска

Если ответ запроса содержит много элементов, служба возвращает значение `$skiptoken` (`@odata.nextLink`), которое используется для получения следующей страницы результатов. Используйте его для постраничного просмотра всего набора результатов.

В Службах мультимедиа версии 3 нельзя настроить размер страницы. Размер страницы зависит от типа сущности. Дополнительные сведения см. в следующих разделах.

Если сущности создаются или удаляются во время разбиения коллекции на страницы, изменения отражаются в возвращаемых результатах (если эти изменения находятся в той части коллекции, которая не была скачана).

> [!TIP]
> Всегда используйте `nextLink` для перечисления коллекции, независимо от конкретного размера страницы.
>
> Значение `nextLink` будет представлено только в том случае, если есть более одной страницы сущностей.

Рассмотрим следующий пример, где используется `$skiptoken`. Замените *amstestaccount* именем своей учетной записи и установите для значения *api-version* последнюю версию.

При запросе списка ресурсов следующим образом:

```
GET  https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01 HTTP/1.1
x-ms-client-request-id: dd57fe5d-f3be-4724-8553-4ceb1dbe5aab
Content-Type: application/json; charset=utf-8
```

вы должны увидеть ответ следующего вида.

```
HTTP/1.1 200 OK

{
"value":[
{
"name":"Asset 0","id":"/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaservices/amstestaccount/assets/Asset 0","type":"Microsoft.Media/mediaservices/assets","properties":{
"assetId":"00000000-0000-0000-0000-000000000000","created":"2018-12-11T22:12:44.98Z","lastModified":"2018-12-11T22:15:48.003Z","container":"asset-00000000-0000-0000-0000-0000000000000","storageAccountName":"amsacctname","storageEncryptionFormat":"None"
}
},
// lots more assets
{
"name":"Asset 517","id":"/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaservices/amstestaccount/assets/Asset 517","type":"Microsoft.Media/mediaservices/assets","properties":{
"assetId":"00000000-0000-0000-0000-000000000000","created":"2018-12-11T22:14:08.473Z","lastModified":"2018-12-11T22:19:29.657Z","container":"asset-00000000-0000-0000-0000-000000000000","storageAccountName":"amsacctname","storageEncryptionFormat":"None"
}
}
],"@odata.nextLink":"https:// management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01&$skiptoken=Asset+517"
}
```

Необходимо затем запросить следующую страницу, отправив запрос get.

```
https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/mediaresources/providers/Microsoft.Media/mediaServices/amstestaccount/assets?api-version=2018-07-01&$skiptoken=Asset+517
```

В следующем примере C# показано перечисление всех указателей потока в учетной записи.

```csharp
var firstPage = await MediaServicesArmClient.StreamingLocators.ListAsync(CustomerResourceGroup, CustomerAccountName);

var currentPage = firstPage;
while (currentPage.NextPageLink != null)
{
    currentPage = await MediaServicesArmClient.StreamingLocators.ListNextAsync(currentPage.NextPageLink);
}
```

## <a name="using-logical-operators-to-combine-query-options"></a>Использование логических операторов для объединения параметров запроса

Службы мультимедиа версии 3 поддерживают логические операторы **OR** и **AND**. 

В следующем примере для проверки состояния задания выполняется проверка:

```
https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/qbtest/providers/Microsoft.Media/mediaServices/qbtest/transforms/VideoAnalyzerTransform/jobs?$filter=properties/state%20eq%20Microsoft.Media.JobState'Scheduled'%20or%20properties/state%20eq%20Microsoft.Media.JobState'Processing'&api-version=2018-07-01
```

Тот же запрос создается в C# следующим образом: 

```csharp
var odataQuery = new ODataQuery<Job>("properties/state eq Microsoft.Media.JobState'Scheduled' or properties/state eq Microsoft.Media.JobState'Processing'");
client.Jobs.List(config.ResourceGroup, config.AccountName, VideoAnalyzerTransformName, odataQuery);
```

## <a name="filtering-and-ordering-options-of-entities"></a>Фильтрация и упорядочивание параметров сущностей

В следующей таблице показано, как можно применить параметры фильтрации и упорядочивания к разным сущностям.

|Имя сущности|Имя свойства|Filter|Порядок|
|---|---|---|---|
|[Ресурсы](/rest/api/media/assets/)|name|`eq`, `gt`, `lt`, `ge`, `le`|`asc` и `desc`|
||properties.alternateId |`eq`||
||properties.assetId |`eq`||
||properties.created| `eq`, `gt`, `lt`| `asc` и `desc`|
|[Политики ключа содержимого](/rest/api/media/contentkeypolicies)|name|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` и `desc`|
||properties.created    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` и `desc`|
||properties.description    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`||
||properties.lastModified|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` и `desc`|
||properties.policyId|`eq`, `ne`||
|[Задания](/rest/api/media/jobs)| name  | `eq`            | `asc` и `desc`|
||properties.state        | `eq`, `ne`        |                         |
||properties.created      | `gt`, `ge`, `lt`, `le`| `asc` и `desc`|
||properties.lastModified | `gt`, `ge`, `lt`, `le` | `asc` и `desc`| 
|[Указатели потоковой передачи](/rest/api/media/streaminglocators)|name|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` и `desc`|
||properties.created    |`eq`, `ne`, `ge`, `le`,  `gt`, `lt`|`asc` и `desc`|
||properties.endTime    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` и `desc`|
|[Политики потоковой передачи](/rest/api/media/streamingpolicies)|name|`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` и `desc`|
||properties.created    |`eq`, `ne`, `ge`, `le`, `gt`, `lt`|`asc` и `desc`|
|[Преобразования](/rest/api/media/transforms)| name | `eq`            | `asc` и `desc`|
|| properties.created      | `gt`, `ge`, `lt`, `le`| `asc` и `desc`|
|| properties.lastModified | `gt`, `ge`, `lt`, `le`| `asc` и `desc`|

## <a name="next-steps"></a>Дальнейшие действия

* [Список ресурсов](/rest/api/media/assets/list)
* [Список политик ключей содержимого](/rest/api/media/contentkeypolicies/list)
* [Список заданий](/rest/api/media/jobs/list)
* [Список политик потоковой передачи](/rest/api/media/streamingpolicies/list)
* [Список указателей потоковой передачи](/rest/api/media/streaminglocators/list)
* [Краткое руководство по потоковой передаче видеофайлов — .NET](stream-files-dotnet-quickstart.md)
* [Квоты и ограничения](limits-quotas-constraints-reference.md)

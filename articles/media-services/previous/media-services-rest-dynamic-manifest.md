---
title: Создание фильтров с помощью REST API служб мультимедиа | Документация Майкрософт
description: В этом разделе описывается создание фильтров, с помощью которых клиент может передавать определенные секции потока. Службы мультимедиа REST API создают динамические манифесты для реализации этой выборочной потоковой передачи.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: f7d23daf-7cd2-49c7-a195-ab902912ab3c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.reviewr: cenkdin
ms.openlocfilehash: b9286260d0fead11c7f55eff213c8ad0241a8cba
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103008796"
---
# <a name="creating-filters-with-azure-media-services-rest-api"></a>Создание фильтров с помощью с помощью API REST служб мультимедиа

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-dynamic-manifest.md)
> * [REST](media-services-rest-dynamic-manifest.md)
> 
> 

Начиная с версии 2.17, службы мультимедиа позволяют определять фильтры для активов. Эти фильтры представляют собой правила на стороне сервера. Они позволяют пользователям выполнять следующие действия: воспроизводить только часть видео (не все видео) или выбирать только часть представлений аудио и видео, которые может обрабатывать клиентское устройство (не все представления, связанные с файлом). Такая фильтрация активов достигается с помощью **динамических манифестов**, которые создаются по запросу клиента для потоковой передачи видео на основе указанных фильтров.

Подробные сведения о фильтрах и динамическом манифесте см. в статье [Фильтры и динамические манифесты](media-services-dynamic-manifest-overview.md).

В этой статье показано, как использовать интерфейсы REST API для создания, обновления и удаления фильтров. 

## <a name="types-used-to-create-filters"></a>Типы, используемые для создания фильтров
При создании фильтров используются следующие типы:  

* [Фильтр](/rest/api/media/operations/filter)
* [AssetFilter](/rest/api/media/operations/assetfilter)
* [пресентатионтимеранже](/rest/api/media/operations/presentationtimerange)
* [FilterTrackSelect и FilterTrackPropertyCondition](/rest/api/media/operations/filtertrackselect)

> [!NOTE]
> 
> При доступе к сущностям в службах мультимедиа необходимо задать определенные поля и значения заголовков в HTTP-запросах. Дополнительную информацию см. в статье [Обзор интерфейса REST API служб мультимедиа](media-services-rest-how-to-use.md).

## <a name="connect-to-media-services"></a>Подключение к службам мультимедиа

Сведения о подключении к API AMS см. в разделе [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="create-filters"></a>Создание фильтров
### <a name="create-global-filters"></a>Создание глобальных фильтров
Чтобы создать глобальный фильтр, используйте следующие запросы HTTP:  

#### <a name="http-request"></a>HTTP-запрос
Заголовки запросов

```console
POST https://media.windows.net/API/Filters HTTP/1.1 
DataServiceVersion:3.0 
MaxDataServiceVersion: 3.0 
Content-Type: application/json 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
Host:media.windows.net 
```

Текст запроса 

```console
{  
   "Name":"GlobalFilter",
   "PresentationTimeRange":{  
      "StartTimestamp":"0",
      "EndTimestamp":"9223372036854775807",
      "PresentationWindowDuration":"12000000000",
      "LiveBackoffDuration":"0",
      "Timescale":"10000000"
   },
   "Tracks":[  
      {  
         "PropertyConditions":
              [  
            {  
               "Property":"Type",
               "Value":"audio",
               "Operator":"Equal"
            },
            {  
               "Property":"Bitrate",
               "Value":"0-2147483647",
               "Operator":"Equal"
            }
         ]
      }
   ]
}
```



#### <a name="http-response"></a>HTTP-ответ

```console
HTTP/1.1 201 Created 
```

### <a name="create-local-assetfilters"></a>Создание локальных фильтров активов
Чтобы создать локальный фильтр активов (AssetFilter), используйте следующие запросы HTTP:  

#### <a name="http-request"></a>HTTP-запрос
Заголовки запросов

```console
POST https://media.windows.net/API/AssetFilters HTTP/1.1 
DataServiceVersion: 3.0 
MaxDataServiceVersion: 3.0 
Content-Type: application/json 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
Host: media.windows.net  
```

Текст запроса 

```console
{   
   "Name":"AssetFilter", 
   "ParentAssetId":"nb:cid:UUID:536e555d-1500-80c3-92dc-f1e4fdc6c592", 
   "PresentationTimeRange":{   
      "StartTimestamp":"0", 
      "EndTimestamp":"9223372036854775807", 
      "PresentationWindowDuration":"12000000000", 
      "LiveBackoffDuration":"0", 
      "Timescale":"10000000" 
   }, 
   "Tracks":[   
      {   
         "PropertyConditions": 
              [   
            {   
               "Property":"Type", 
               "Value":"audio", 
               "Operator":"Equal" 
            }, 
            {   
               "Property":"Bitrate", 
               "Value":"0-2147483647", 
               "Operator":"Equal" 
            } 
         ] 
      } 
   ] 
} 
```

#### <a name="http-response"></a>HTTP-ответ

```console
HTTP/1.1 201 Created 
. . . 
```

## <a name="list-filters"></a>Вывод списка фильтров
### <a name="get-all-global-filters-in-the-ams-account"></a>Получение всех глобальных фильтров ( **Filter**) в учетной записи AMS
Для вывода списка фильтров используйте следующие запросы HTTP: 

#### <a name="http-request"></a>HTTP-запрос

```console
GET https://media.windows.net/API/Filters HTTP/1.1 
DataServiceVersion:3.0 
MaxDataServiceVersion: 3.0 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
Host: media.windows.net 
```

### <a name="get-assetfilters-associated-with-an-asset"></a>Получение фильтров активов ( **AssetFilter**), связанных с тем или иным активом
#### <a name="http-request"></a>HTTP-запрос

```console
GET https://media.windows.net/API/Assets('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592')/AssetFilters HTTP/1.1 
DataServiceVersion: 3.0 
MaxDataServiceVersion: 3.0 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
Host: media.windows.net 
```

### <a name="get-an-assetfilter-based-on-its-id"></a>Получение фильтра активов ( **AssetFilter** ) по его идентификатору
#### <a name="http-request"></a>HTTP-запрос

```console
GET https://media.windows.net/API/AssetFilters('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592__%23%23%23__TestFilter') HTTP/1.1 
DataServiceVersion: 3.0 
MaxDataServiceVersion: 3.0 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
x-ms-client-request-id: 00000000
```

## <a name="update-filters"></a>Обновление фильтров
Для обновления фильтра новыми значениями свойства используйте операции PATCH, PUT или MERGE.  Дополнительные сведения об этих операциях см. [здесь](/openspecs/windows_protocols/ms-odata/59d5abd3-7b12-490a-a0e2-9d9324b91893).

При обновлении фильтра может понадобиться до 2 минут на обновление правил конечной точкой потоковой передачи. Если содержимое было обработано с помощью данного фильтра (и кэшировано на прокси-серверах и в кэшах CDN), обновление этого фильтра может привести к сбоям проигрывателя. Очистите кэш после обновления фильтра. Если такой вариант невозможен, рассмотрите возможность использования другого фильтра.  

### <a name="update-global-filters"></a>Обновление глобальных фильтров
Чтобы обновить глобальный фильтр, используйте следующие запросы HTTP: 

#### <a name="http-request"></a>HTTP-запрос
Заголовки запроса: 

```console
MERGE https://media.windows.net/API/Filters('filterName') HTTP/1.1 
DataServiceVersion:3.0 
MaxDataServiceVersion: 3.0 
Content-Type: application/json 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
Host: media.windows.net 
Content-Length: 384
```

Тело запроса: 

```console
{ 
   "Tracks":[   
      {   
         "PropertyConditions": 
         [   
            {   
               "Property":"Type", 
               "Value":"audio", 
               "Operator":"Equal" 
            }, 
            {   
               "Property":"Bitrate", 
               "Value":"0-2147483647", 
               "Operator":"Equal" 
            } 
         ] 
      } 
   ] 
} 
```

### <a name="update-local-assetfilters"></a>Обновление локальных фильтров ресурсов-контейнеров
Чтобы обновить локальный фильтр, используйте следующие запросы HTTP: 

#### <a name="http-request"></a>HTTP-запрос
Заголовки запроса: 

```console
MERGE https://media.windows.net/API/AssetFilters('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592__%23%23%23__TestFilter')  HTTP/1.1 
DataServiceVersion: 3.0 
MaxDataServiceVersion: 3.0 
Content-Type: application/json 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
Host: media.windows.net 
```

Тело запроса: 

```console
{ 
   "Tracks":[   
      {   
         "PropertyConditions": 
         [   
            {   
               "Property":"Type", 
               "Value":"audio", 
               "Operator":"Equal" 
            }, 
            {   
               "Property":"Bitrate", 
               "Value":"0-2147483647", 
               "Operator":"Equal" 
            } 
         ] 
      } 
   ] 
} 
```

## <a name="delete-filters"></a>Удаление фильтров
### <a name="delete-global-filters"></a>Удаление глобальных фильтров
Чтобы удалить глобальный фильтр, используйте следующие запросы HTTP:

#### <a name="http-request"></a>HTTP-запрос

```console
DELETE https://media.windows.net/api/Filters('GlobalFilter') HTTP/1.1 
DataServiceVersion:3.0 
MaxDataServiceVersion: 3.0 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN>  
x-ms-version: 2.19 
Host: media.windows.net 
```

### <a name="delete-local-assetfilters"></a>Удаление локальных фильтров активов
Чтобы удалить локальный фильтр активов, используйте следующие запросы HTTP:

#### <a name="http-request"></a>HTTP-запрос

```console
DELETE https://media.windows.net/API/AssetFilters('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592__%23%23%23__LocalFilter') HTTP/1.1 
DataServiceVersion: 3.0 
MaxDataServiceVersion: 3.0 
Accept: application/json 
Accept-Charset: UTF-8 
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19 
Host: media.windows.net 
```

## <a name="build-streaming-urls-that-use-filters"></a>Построение URL-адресов потоковой передачи с использованием фильтров
Сведения о публикации и доставке ресурсов см. в статье [Доставка содержимого клиентам](media-services-deliver-content-overview.md).

Следующие примеры показывают, как добавлять фильтры к URL-адресам потоковой передачи.

**MPEG DASH** 

`http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf, filter=MyFilter)`

**Apple HTTP Live Streaming (HLS) V4**

`http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl, filter=MyFilter)`

**Apple HTTP Live Streaming (HLS) V3**

`http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3, filter=MyFilter)`

**Smooth Streaming**

`http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(filter=MyFilter)`

    
## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>См. также:
[Обзор динамических манифестов](media-services-dynamic-manifest-overview.md)

---
title: Поиск категорий для API Bing Local Business Search
titleSuffix: Azure Cognitive Services
description: В этой статье вы узнаете, как задавать поиск категорий для конечной точки API Bing Local Business Search.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.topic: conceptual
ms.date: 11/01/2018
ms.author: rosh
ms.openlocfilehash: 19b769d1fff431f95c20e607c17747f2ff483d2f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96487190"
---
# <a name="search-categories-for-the-bing-local-business-search-api"></a>Поиск категорий для API Bing Local Business Search

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

API Bing Local Business Search позволяет искать сущности локального бизнеса в различных категориях с приоритетом в пользу закрытия расположения пользователя. Поисковые запросы можно включить в поиск вместе с  [параметрами](specify-geographic-search.md)`localCircularView` и `localMapView`.


## <a name="toplevel-categories"></a>Категории верхнего уровня 

Следующие типы определяют основные категории поиска.  Больше чем одну категорию можно указать с помощью списка с разделителями запятыми, назначенного параметру `localCategories`.  
- EatDrink 
- SeeDo 
- Shop 
- HotelsAndMotels 
- BanksAndCreditUnions 
- Parking 
- Hospitals 

## <a name="sub-categories"></a>Подкатегории
Подкатегории проходят тот же путь, что и `localCategories`. Подкатегории — это более конкретные категории. Они подчинены в том смысле, что если вы укажете категорию C и одну из ее подкатегорий S в том же списке с разделителями-запятыми, вы получите те же результаты, что указали для C.

### <a name="eat-drink"></a>EatDrink

> Бревериесандбревпубс, Кокктаиллаунжес, Африканрестаурантс, Американрестаурантс, Бажелс, BarbecueRestaurants, Taverns, SportsBars, линейчатые, BarsGrillsAndPubs, BuffetRestaurants | Белгианрестаурантс, Бритишрестаурантс, Каферестаурантс, Кариббеанрестаурантс, Чинесерестаурантс, CoffeeAndTea, delicatessens, DeliveryService, столик, DiscountStores, кольца, FastFood, FrenchRestaurants, FrozenYogurt, GermanRestaurants, Supermarkets, GreekRestaurants, grocers, HawaiianRestaurants, HungarianRestaurants, IceCreamAndFrozenDesserts, IndianRestaurants, ItalianRestaurants, JapaneseRestaurants, Juices, KoreanRestaurants, LiquorStores, MexicanRestaurants, MiddleEasternRestaurants, пицц, PolishRestaurants, PortugueseRestaurants, pretzels, рестораны, RussianAndUkrainianRestaurants, бутерброды, SeafoodRestaurants, SpanishRestaurants, SteakHouseRestaurants

### <a name="see-do"></a>SeeDo

> Амусементпаркс, Аттрактионс, Карнивалс, Касинос, Ландмарксандхисторикалситес, Миниатуреголфкаурсес, MovieTheaters, музеи, парки, SightseeingTours, TouristInformation, зоопарках

### <a name="shop"></a>Shop

> Антикуесторес, Bookstore, Кдандрекордсторес, Чилдренсклосингсторес, Цигарандтобаккошопс, Комикбуксторес, DepartmentStores, DiscountStores, FleaMarketsAndBazaars, FurnitureStores, HomeImprovementStores, JewelryAndWatchesStores, KitchenwareStores, LiquorStores, MallsAndShoppingCenters, MensClothingStores, MusicStores, OutletStores, PetShops, PetSupplyStores, SchoolAndOfficeSupplyStores, ShoeStores, SportingGoodsStores, ToyAndGameStores, VitaminAndSupplementStores, WomensClothingStores


## <a name="examples-of-local-categories-search"></a>Examples of Local Categories search

Следующие примеры результатов GET соответствуют параметру `localCategories`.

`https://api.cognitive.microsoft.com/localbusinesses/v7.0/search?&q=&mkt=en-US&localcategories=HotelsAndMotels`

`https://api.cognitive.microsoft.com/localbusinesses/v7.0/search?&q=&mkt=en-US&localcategories=EatDrink`

`https://api.cognitive.microsoft.com/localbusinesses/v7.0/search?&q=&mkt=en-US&localcategories=Shop`

`https://api.cognitive.microsoft.com/localbusinesses/v7.0/search?&q=&mkt=en-US&localcategories=Hospitals`

Следующий запрос ограничивает число результатов "hospital" для первых трех, возвращаемых API Bing Local Business Search.

`https://api.cognitive.microsoft.com/localbusinesses/v7.0/search?&q=&mkt=en-US&localCategories=Hospitals&count=3&offset=0`

Следующий пример ответа JSON включает три больницы в Сиэтле.

```json
BingAPIs-TraceId: 68AFB51807C6485CAB8AAF20E232EFFF
BingAPIs-SessionId: F89E7B8539B34BF58AAF811485E83B20
X-MSEdge-ClientID: 1C44E64DBFAA6BCA1270EADDBE7D6A22
BingAPIs-Market: en-US
X-MSEdge-Ref: Ref A: 68AFB51807C6485CAB8AAF20E232EFFF Ref B: CO1EDGE0108 Ref C: 2018-10-22T18:52:28Z

{
   "_type": "SearchResponse",
   "queryContext": {
      "originalQuery": ""
   },
   "places": {
      "readLink": "https:\/\/www.bingapis.com\/api\/v7\/places\/search?q=",
      "totalEstimatedMatches": 0,
      "value": [
         {
            "_type": "LocalBusiness",
            "id": "https:\/\/www.bingapis.com\/api\/v7\/#Places.0",
            "name": "Overlake Hospital Medical Center",
            "url": "http:\/\/www.overlakehospital.org\/",
            "entityPresentationInfo": {
               "entityScenario": "ListItem",
               "entityTypeHints": [
                  "Place",
                  "LocalBusiness"
               ]
            },
            "geo": {
               "latitude": 47.6204986572266,
               "longitude": -122.185829162598
            },
            "routablePoint": {
               "latitude": 47.6204986572266,
               "longitude": -122.185668945312
            },
            "address": {
               "streetAddress": "1035 116th Ave NE",
               "addressLocality": "Bellevue",
               "addressRegion": "WA",
               "postalCode": "98004",
               "addressCountry": "US",
               "neighborhood": "",
               "text": "1035 116th Ave NE, Bellevue, WA, 98004"
            },
            "telephone": "(425) 688-5000"
         },
         {
            "_type": "LocalBusiness",
            "id": "https:\/\/www.bingapis.com\/api\/v7\/#Places.1",
            "name": "Virginia Mason University Village Medical Center",
            "url": "https:\/\/virginiamason.org\/Seattle",
            "entityPresentationInfo": {
               "entityScenario": "ListItem",
               "entityTypeHints": [
                  "Place",
                  "LocalBusiness"
               ]
            },
            "geo": {
               "latitude": 47.6095390319824,
               "longitude": -122.327941894531
            },
            "routablePoint": {
               "latitude": 47.6093978881836,
               "longitude": -122.328277587891
            },
            "address": {
               "streetAddress": "1100 9th Ave",
               "addressLocality": "Seattle",
               "addressRegion": "WA",
               "postalCode": "98101",
               "addressCountry": "US",
               "neighborhood": "University District",
               "text": "1100 9th Ave, Seattle, WA, 98101"
            },
            "telephone": "(206) 223-6600"
         },
         {
            "_type": "LocalBusiness",
            "id": "https:\/\/www.bingapis.com\/api\/v7\/#Places.2",
            "name": "Seattle Children’s Hospital",
            "url": "http:\/\/www.seattlechildrens.org\/",
            "entityPresentationInfo": {
               "entityScenario": "ListItem",
               "entityTypeHints": [
                  "Place",
                  "LocalBusiness"
               ]
            },
            "geo": {
               "latitude": 47.6625061035156,
               "longitude": -122.283760070801
            },
            "routablePoint": {
               "latitude": 47.6631507873535,
               "longitude": -122.284614562988
            },
            "address": {
               "streetAddress": "4800 Sand Point Way NE",
               "addressLocality": "Seattle",
               "addressRegion": "WA",
               "postalCode": "98105",
               "addressCountry": "US",
               "neighborhood": "Laurelhurst",
               "text": "4800 Sand Point Way NE, Seattle, WA, 98105"
            },
            "telephone": "(206) 987-2000"
         }
      ],
      "searchAction": {

      }
   }
}
```

## <a name="next-steps"></a>Дальнейшие действия
- [Use geographic boundaries to filter results from the Bing Local Business Search API](specify-geographic-search.md) (Использование географических границ для фильтрации результатов API Bing Local Business Search)
- [Sending and using Bing Local Business Search API queries and responses](local-search-query-response.md) (Отправление и использование запросов и ответов API Bing Local Business Search)
- [Quickstart: Send a query to the Bing Local Business Search API in C#](quickstarts/local-quickstart.md) (Краткое руководство. Отправление запроса API Bing Local Business Search в C#)
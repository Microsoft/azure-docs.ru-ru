---
title: Запрос передачи данных с помощью Microsoft Azure Maps службы Mobility Services (Предварительная версия)
description: Узнайте, как использовать службы Azure Maps Mobility Services (Предварительная версия) для запроса общедоступных транзитных данных, таких как идентификаторы города Metro, транзитных маршрутов, маршрутных путей и путевых маршрутов.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 740080d742f535f868b2ae194b24bebe5ac6ac24
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96906035"
---
# <a name="request-public-transit-data-using-the-azure-maps-mobility-services-preview"></a>Запрос общедоступных транзитных данных с помощью Azure Maps служб Mobility Services (Предварительная версия) 

> [!IMPORTANT]
> Службы Mobility Azure Maps в настоящее время доступны в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


В этой статье показано, как использовать [службы мобильности](/rest/api/maps/mobility) Azure Maps для запроса общедоступных транзитных данных. Транзитные данные включают остановку передачи, сведения о маршруте и оценки времени поездки.

В этой статье вы узнаете, как:

* Получение идентификатора области Metro с помощью [API-интерфейса «получить область Metro](/rest/api/maps/mobility/getmetroareapreview) »
* Запрос, находящихся поблизости, перестает использовать службу [получения ближайших транзитных](/rest/api/maps/mobility/getnearbytransitpreview) услуг.
* Запрос [получения API транзитных маршрутов](/rest/api/maps/mobility/gettransitroutepreview) для планирования маршрута с использованием общедоступного транзита.
* Запрос на перенаправление геометрического маршрута и подробное расписание для маршрута с помощью [API-интерфейса для получения пути](https://aka.ms/https://azure.microsoft.com/services/azure-maps/).

## <a name="prerequisites"></a>Предварительные требования

1. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](./how-to-manage-authentication.md).

В этом руководстве используется приложение [Postman](https://www.postman.com/), но вы можете выбрать другую среду разработки API.

## <a name="get-a-metro-area-id"></a>Получить идентификатор области метро

Чтобы запросить подробную информацию о транзитных агентствах и поддерживаемых типах транзитных складов для определенной области города, вам понадобится эта `metroId` область. [Интерфейс API "получить область метро](/rest/api/maps/mobility/getmetroareapreview) " позволяет запрашивать области метро, в которых доступны Azure Maps службы Mobility Services. Ответ содержит такие сведения `metroId` , как, `metroName` и представление геометрии области Metro в формате геоjson.

Давайте создадим запрос для получения области Metro для Seattle-Tacoma идентификатора области Metro. Чтобы запросить идентификатор для области Metro, выполните следующие действия.

1. Откройте приложение POST и создадим коллекцию для хранения запросов. В верхней части приложения Postman выберите элемент **Создать**. В окне **Create New** (Создание) выберите **Collection** (Коллекция).  Присвойте имя коллекции и нажмите кнопку **Создать**.

2. Чтобы создать запрос, нажмите кнопку **Создать** еще раз. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **Имя запроса** для запроса. Выберите коллекцию, созданную на предыдущем шаге, в качестве расположения для сохранения запроса. Затем нажмите кнопку **Сохранить**.
  
    ![Создание запроса в POST](./media/how-to-request-transit-data/postman-new.png)

3. Выберите метод **получения** HTTP на вкладке Построитель и введите следующий URL-адрес для создания запроса GET. Замените на `{subscription-key}` Azure Maps первичный ключ.

    ```HTTP
    https://atlas.microsoft.com/mobility/metroArea/id/json?subscription-key={subscription-key}&api-version=1.0&query=47.63096,-122.126
    ```

4. После успешного выполнения запроса вы получите следующий ответ:

    ```JSON
    {
        "results": [
            {
                "metroId": 522,
                "metroName": "Seattle–Tacoma–Bellevue, WA",
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 48.5853,
                        "longitude": -124.80934
                    },
                    "btmRightPoint": {
                        "latitude": 46.90534,
                        "longitude": -121.55032
                    }
                },
                "geometry": {
                    "type": "Polygon",
                    "coordinates": [
                        [
                            [
                                -121.99604,
                                47.16147
                            ],
                            [
                                -121.97051,
                                47.17222
                            ],
                            [
                                -121.96308,
                                47.17671
                            ],
                            ...,
                            ...,
                            ...,
                            [
                                -122.01525,
                                47.16008
                            ],
                            [
                                -122.00553,
                                47.15919
                            ],
                            [
                                -121.99604,
                                47.16147
                            ]
                        ]
                    ]
                }
            }
        ]
    }
    ```

## <a name="request-nearby-transit-stops"></a>Запрос нахождения ближайших транзитных точек останавливается

Служба Azure Maps [получить близлежащие транзитные](/rest/api/maps/mobility/getnearbytransitpreview) службы позволяет выполнять поиск по транзитным объектам. API возвращает сведения о транзитном объекте, такие как прекращение общедоступного транзита и передача общих велосипедов по заданному расположению. Далее мы выполним запрос к службе для поиска ближайших общественных транзитных мест в радиусе 300 метров по отношению к заданному расположению.

Чтобы выполнить запрос на [транзитное](/rest/api/maps/mobility/getnearbytransitpreview)подключение, выполните следующие действия:

1. В окне POST щелкните **новый запрос**  |  **получить запрос** и назовите его **Get поблизости**.

2. На вкладке Построитель выберите метод **Get** HTTP, введите следующий URL-адрес запроса для КОНЕЧНОЙ точки API и нажмите кнопку **Отправить**.

    ```HTTP
    https://atlas.microsoft.com/mobility/transit/nearby/json?subscription-key={subscription-key}&api-version=1.0&query=47.63096,-122.126&radius=300&objectType=stop
    ```

3. После успешного запроса структура ответа должна выглядеть следующим образом:

    ```JSON
    {
        "results": [
            {
                "id": "522---2060603",
                "type": "stop",
                "objectDetails": {
                    "stopKey": "71300",
                    "stopName": "NE 24th St & 162nd Ave NE",
                    "stopCode": "71300",
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                },
                "position": {
                    "latitude": 47.631504,
                    "longitude": -122.125275
                },
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 47.632413,
                        "longitude": -122.12659
                    },
                    "btmRightPoint": {
                        "latitude": 47.630594,
                        "longitude": -122.123959
                    }
                }
            },
            {
                "id": "522---2061020",
                "type": "stop",
                "objectDetails": {
                    "stopKey": "68372",
                    "stopName": "NE 24th St & 160th Ave NE",
                    "stopCode": "68372",
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                },
                "position": {
                    "latitude": 47.631409,
                    "longitude": -122.127136
                },
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 47.632318,
                        "longitude": -122.128451
                    },
                    "btmRightPoint": {
                        "latitude": 47.630499,
                        "longitude": -122.12582
                    }
                }
            },
            {
                "id": "522---2060604",
                "type": "stop",
                "objectDetails": {
                    "stopKey": "71310",
                    "stopName": "NE 24th St & 160th Ave NE",
                    "stopCode": "71310",
                    "mainTransitType": "Bus",
                    "mainAgencyId": "522---5872",
                    "mainAgencyName": "Metro Transit"
                },
                "position": {
                    "latitude": 47.631565,
                    "longitude": -122.127808
                },
                "viewport": {
                    "topLeftPoint": {
                        "latitude": 47.632474,
                        "longitude": -122.129124
                    },
                    "btmRightPoint": {
                        "latitude": 47.630655,
                        "longitude": -122.126492
                    }
                }
            }
        ]
    } 
    ```

Если вы внимательно следите за структурой ответа, вы увидите, что она содержит параметры для каждого транзитного объекта. Каждый транзитный объект имеет такие параметры `id` , как, `type` , `stopName` ,, `mainTransitType` `mainAgencyName` и положения объекта в координатах.

В целях обучения мы будем использовать точку `id` останова в качестве источника для нашего маршрута в следующем разделе.  

## <a name="request-a-transit-route"></a>Запрос транзитного маршрута

API Azure Maps [получения транзитных маршрутов](/rest/api/maps/mobility/gettransitroutepreview) позволяет планировать поездки. Он возвращает наилучшие возможные варианты маршрута от источника к назначению. Служба предоставляет различные режимы поездки, в том числе прохождение, поездкам и общедоступные передачи. Далее мы будем выполнять поиск по маршруту с ближайшей шины на место в Сиэтле.

### <a name="get-location-coordinates-for-destination"></a>Получение координат расположения для назначения

Чтобы получить координаты места расположения "игольчатый", мы будем использовать службу "Azure Maps [нечеткий поиск](/rest/api/maps/search/getsearchfuzzy)".

Чтобы выполнить запрос к службе поиска нечетких запросов, выполните следующие действия:

1. В окне POST щелкните **создать запрос**  |  **получить запрос** и назовите его « **Получение координат расположения**».

2. На вкладке Построитель выберите метод **Get** HTTP, введите следующий URL-адрес запроса и нажмите кнопку **Отправить**.

    ```HTTP
    https://atlas.microsoft.com/search/fuzzy/json?subscription-key={subscription-key}&api-version=1.0&query=space needle
    ```

3. Если взглянуть на ответ внимательно, он содержит несколько расположений в результатах поиска в виде стрелки. Каждый результат содержит координаты расположения под **позицией**. Скопируйте `lat` и `lon` под **позицией** первого результата.

   ```JSON
   {
        "summary": {
            "query": "space needle",
            "queryType": "NON_NEAR",
            "queryTime": 35,
            "numResults": 8,
            "offset": 0,
            "totalResults": 11,
            "fuzzyLevel": 1
        },
        "results": [
            {
                "type": "POI",
                "id": "US/POI/p0/6993440",
                "score": 4.67369,
                "info": "search:ta:840539001406144-US",
                "poi": {
                    "name": "Space Needle",
                    "phone": "+(1)-(206)-9052100",
                    "categorySet": [
                        {
                            "id": 7376009
                        }
                    ],
                    "url": "www.spaceneedle.com",
                    "categories": [
                        "important tourist attraction",
                        "tower"
                    ],
                    "classifications": [
                        {
                            "code": "IMPORTANT_TOURIST_ATTRACTION",
                            "names": [
                                {
                                    "nameLocale": "en-US",
                                    "name": "important tourist attraction"
                                },
                                {
                                    "nameLocale": "en-US",
                                    "name": "tower"
                                }
                            ]
                        }
                    ]
                },
                "address": {
                    "streetNumber": "400",
                    "streetName": "Broad St",
                    "municipalitySubdivision": "South Lake Union, Seattle, Lower Queen Anne",
                    "municipality": "Seattle",
                    "countrySecondarySubdivision": "King",
                    "countryTertiarySubdivision": "Seattle",
                    "countrySubdivision": "WA",
                    "postalCode": "98109",
                    "countryCode": "US",
                    "country": "United States",
                    "countryCodeISO3": "USA",
                    "freeformAddress": "400 Broad St, Seattle, WA 98109",
                    "localName": "Seattle",
                    "countrySubdivisionName": "Washington"
                },
                "position": {
                    "lat": 47.62039,
                    "lon": -122.34928
                },
                "viewport": {
                    "topLeftPoint": {
                        "lat": 47.62129,
                        "lon": -122.35061
                    },
                    "btmRightPoint": {
                        "lat": 47.61949,
                        "lon": -122.34795
                    }
                },
                "entryPoints": [
                    {
                        "type": "main",
                        "position": {
                            "lat": 47.61982,
                            "lon": -122.34886
                        }
                    }
                ]
            },
            ...,
            ...,
            ...
        ]
    }
    ```

### <a name="request-route"></a>Маршрут запроса

Чтобы выполнить запрос маршрута, выполните следующие действия:

1. В окне POST щелкните **создать запрос**  |  **получить запрос** и назовите его **получить сведения о маршруте**.

2. На вкладке Построитель выберите метод **Get** HTTP, введите следующий URL-адрес запроса для КОНЕЧНОЙ точки API и нажмите кнопку **Отправить**.

    Мы зазапрашиваем открытые транзитные маршруты для шины, указав `modeType` Параметры и `transitType` . URL-адрес запроса содержит расположения, полученные в предыдущих разделах. Для у `originType` нас теперь есть **стопид**. И для у `destionationType` нас есть **Расположение**.

    См. [список параметров URI](/rest/api/maps/mobility/gettransitroutepreview#uri-parameters) , которые можно использовать в запросе к [API получения транзитных маршрутов](/rest/api/maps/mobility/gettransitroutepreview).
  
    ```HTTP
    https://atlas.microsoft.com/mobility/transit/route/json?subscription-key={subscription-key}&api-version=1.0&originType=stopId&origin=522---2060603&destionationType=position&destination=47.62039,-122.34928&modeType=publicTransit&transitType=bus
    ```

3. После успешного запроса структура ответа должна выглядеть следующим образом:

    ```JSON
    {
        "results": [
            {
                "itineraryId": "cb6b6b6f-5cda-451e-b68d-2e97971dd60c---20190906BBBEC4D2219A436A9D794224978C9BBF:0---522",
                "departureTime": "2019-09-07T01:01:50Z",
                "arrivalTime": "2019-09-07T02:16:33Z",
                "travelTimeInSeconds": 4483,
                "numberOfLegs": 8,
                "legs": [
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:01:50Z",
                        "legEndTime": "2019-09-07T01:01:50Z",
                        "caption": "249"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:01:50Z",
                        "legEndTime": "2019-09-07T01:26:00Z",
                        "caption": "249",
                        "lengthInMeters": 9139
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:26:00Z",
                        "legEndTime": "2019-09-07T01:28:00Z",
                        "caption": "255"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:28:00Z",
                        "legEndTime": "2019-09-07T01:57:21Z",
                        "caption": "255",
                        "lengthInMeters": 13136
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T01:57:22Z",
                        "legEndTime": "2019-09-07T02:01:27Z",
                        "caption": "Denny Way",
                        "lengthInMeters": 308
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T02:01:27Z",
                        "legEndTime": "2019-09-07T02:06:33Z",
                        "caption": "8"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T02:06:33Z",
                        "legEndTime": "2019-09-07T02:12:41Z",
                        "caption": "8",
                        "lengthInMeters": 1060
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T02:12:42Z",
                        "legEndTime": "2019-09-07T02:16:33Z",
                        "lengthInMeters": 251
                    }
                ],
                "itineraryFare": {
                    "price": {
                        "amount": 550,
                        "currencyCode": "USD"
                    },
                    "tickets": [
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        },
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        }
                    ]
                }
            },
            ...,
            {
                "itineraryId": "cb6b6b6f-5cda-451e-b68d-2e97971dd60c---20190906BBBEC4D2219A436A9D794224978C9BBF:2---522",
                "departureTime": "2019-09-07T00:49:32Z",
                "arrivalTime": "2019-09-07T02:20:06Z",
                "travelTimeInSeconds": 5434,
                "numberOfLegs": 10,
                "legs": [
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T00:49:32Z",
                        "legEndTime": "2019-09-07T00:49:32Z",
                        "caption": "226"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T00:49:32Z",
                        "legEndTime": "2019-09-07T01:15:00Z",
                        "caption": "226",
                        "lengthInMeters": 6792
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:15:00Z",
                        "legEndTime": "2019-09-07T01:20:00Z",
                        "caption": "241"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:20:00Z",
                        "legEndTime": "2019-09-07T01:28:00Z",
                        "caption": "241",
                        "lengthInMeters": 3397
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:28:00Z",
                        "legEndTime": "2019-09-07T01:33:00Z",
                        "caption": "550"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T01:33:00Z",
                        "legEndTime": "2019-09-07T01:58:00Z",
                        "caption": "550",
                        "lengthInMeters": 12899
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T01:58:01Z",
                        "legEndTime": "2019-09-07T01:59:21Z",
                        "caption": "4th Avenue South",
                        "lengthInMeters": 99
                    },
                    {
                        "legType": "Wait",
                        "legStartTime": "2019-09-07T01:59:21Z",
                        "legEndTime": "2019-09-07T02:01:00Z",
                        "caption": "33"
                    },
                    {
                        "legType": "Bus",
                        "legStartTime": "2019-09-07T02:01:00Z",
                        "legEndTime": "2019-09-07T02:13:29Z",
                        "caption": "33,24",
                        "lengthInMeters": 2447
                    },
                    {
                        "legType": "Walk",
                        "legStartTime": "2019-09-07T02:13:30Z",
                        "legEndTime": "2019-09-07T02:20:06Z",
                        "lengthInMeters": 457
                    }
                ],
                "itineraryFare": {
                    "price": {
                        "amount": 550,
                        "currencyCode": "USD"
                    },
                    "tickets": [
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        },
                        {
                            "amount": 275,
                            "currencyCode": "USD"
                        }
                    ]
                }
            }
        ]
    }
    ```

4. Если вы следите за осторожностью, в ответе есть несколько маршрутов **шины** . Каждому маршруту присваивается уникальный **идентификатор**, сводка, описывающая каждую часть маршрута, и объект `itineraryFare` , который предоставляет как детализированную, так и общую стоимость билетов шины. Точка маршрута является частью маршрута между двумя вайпоинтсами. Далее мы зазапрашиваем сведения о быстром маршруте, используя `itineraryId` в ответе.

## <a name="request-fastest-route-itinerary"></a>Запрашивать самый быстрый маршрут маршрутов

Служба Azure Maps [получить путевые](/rest/api/maps/mobility/gettransititinerarypreview) маршруты позволяет запрашивать данные для конкретного маршрута, используя **идентификатор** маршрута, возвращенный службой [API получения транзитных маршрутов](/rest/api/maps/mobility/gettransitroutepreview) . Чтобы выполнить запрос, выполните следующие действия:

1. В окне POST щелкните **создать запрос**  |  **получить запрос** и назовите его **получить сведения о транзитном транспорте**.

2. На вкладке Построитель выберите метод **Get** HTTP. Введите следующий URL-адрес запроса для конечной точки API и нажмите кнопку **Отправить**.

    Мы будем присвоить `detailType` параметру значение **Geometry** , чтобы ответ содержал сведения о недоступности для бесплатного транзитного пути и последующего включения навигации для прохода и нахождения велосипеда маршрута.

    ```HTTP
    https://atlas.microsoft.com/mobility/transit/itinerary/json?api-version=1.0&subscription-key={subscription-key}&query={itineraryId}&detailType=geometry
    ```

3. После успешного запроса структура ответа должна выглядеть так, как показано ниже.  Если вы видите ответ JSON, то обратите внимание, что каждая Разрезка шины содержит `legfare` элемент. `legfare`Элемент содержит затраты (в центах) каждого маршрута шины, который приобретается отдельно. В конце ответа вы увидите `itineraryFare` элемент, содержащий стоимость в центе всего маршрута. В этом примере существует четыре маршрута шины `$2.75` . Однако если вы приобретаете один билет для всего маршрута, стоимость — `$5.50` .

    ```JSON
   {
        "departureTime": "2020-07-22T19:54:47Z",
        "arrivalTime": "2020-07-22T21:12:21Z",
        "legs": [
            {
                "legType": "Wait",
                "legStartTime": "2020-07-22T19:54:47Z",
                "legEndTime": "2020-07-22T19:54:47Z",
                "lineGroup": {
                    "lineGroupId": "522---666063",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "226",
                    "caption1": "Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "caption2": "226 Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---2756599",
                    "lineGroupId": "522---666063",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "lineNumber": "226",
                    "lineDestination": "Bellevue Transit Center Crossroads"
                },
                "stops": [
                    {
                        "stopId": "522---2060603",
                        "stopKey": "71300",
                        "stopName": "NE 24th St & 162nd Ave NE",
                        "stopCode": "71300",
                        "position": {
                            "latitude": 47.631504,
                            "longitude": -122.125275
                        },
                        "mainTransitType": "Bus",
                        "mainAgencyId": "522---5872",
                        "mainAgencyName": "Metro Transit"
                    },
                    {
                        "stopId": "522---2062263",
                        "stopKey": "85630",
                        "stopName": "Bellevue Tc",
                        "stopCode": "85630",
                        "position": {
                            "latitude": 47.615591,
                            "longitude": -122.196491
                        },
                        "mainTransitType": "Bus",
                        "mainAgencyId": "522---5872",
                        "mainAgencyName": "Metro Transit"
                    }
                ],
                "waitOnVehicle": false
            },
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T19:54:47Z",
                "legEndTime": "2020-07-22T20:15:00Z",
                "lineGroup": {
                    "lineGroupId": "522---666063",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "226",
                    "caption1": "Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "caption2": "226 Eastgate P&R-Crossroads-Overlake-Bellevue TC",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---2756599",
                    "lineGroupId": "522---666063",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "lineNumber": "226",
                    "lineDestination": "Bellevue Transit Center Crossroads"
                },
                "stops": [
                   ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                       ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "pay"
                        }
                    ]
                }
            },
            ...,
            ...,
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T20:20:00Z",
                "legEndTime": "2020-07-22T20:28:00Z",
                "lineGroup": {
                    "lineGroupId": "522---666071",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "241",
                    "caption1": "Eastgate P&R - Bellevue Transit Center",
                    "caption2": "241 Eastgate P&R - Bellevue Transit Center",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---2756619",
                    "lineGroupId": "522---666071",
                    "direction": "backward",
                    "agencyId": "522---5872",
                    "lineNumber": "241",
                    "lineDestination": "Eastgate P&R Factoria"
                },
                "stops": [
                ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                   ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "transfer"
                        }
                    ]
                }
            },
            ...,
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T20:31:00Z",
                "legEndTime": "2020-07-22T20:54:13Z",
                "lineGroup": {
                    "lineGroupId": "522---312636",
                    "agencyId": "522---854535",
                    "agencyName": "Sound Transit",
                    "lineNumber": "550",
                    "caption1": "Bellevue - Seattle",
                    "caption2": "550 Bellevue - Seattle",
                    "color": "00008B",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---962201",
                    "lineGroupId": "522---312636",
                    "direction": "backward",
                    "agencyId": "522---854535",
                    "lineNumber": "550",
                    "lineDestination": "Seattle"
                },
                "stops": [
                   ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                 ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "pay"
                        }
                    ]
                }
            },
            ...,
            ...,
            {
                "legType": "Bus",
                "legStartTime": "2020-07-22T20:57:00Z",
                "legEndTime": "2020-07-22T21:06:00Z",
                "lineGroup": {
                    "lineGroupId": "522---480518",
                    "agencyId": "522---5872",
                    "agencyName": "Metro Transit",
                    "lineNumber": "13",
                    "caption1": "Seattle Pacific - Downtown Seattle",
                    "caption2": "13 Seattle Pacific - Downtown Seattle",
                    "color": "347E5D",
                    "transitType": "Bus"
                },
                "line": {
                    "lineId": "522---1744932",
                    "lineGroupId": "522---480518",
                    "direction": "forward",
                    "agencyId": "522---5872",
                    "lineNumber": "13",
                    "lineDestination": "Seattle Pacific University Seattle Center W"
                },
                "stops": [
                   ...
                ],
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                      ...
                    ]
                },
                "legFare": {
                    "fares": [
                        {
                            "price": {
                                "amount": 275,
                                "currencyCode": "USD"
                            },
                            "usage": "transfer"
                        }
                    ]
                }
            },
            ...,
        ],
        "itineraryFare": {
            "price": {
                "amount": 550,
                "currencyCode": "USD"
            },
            "tickets": [
                {
                    "amount": 275,
                    "currencyCode": "USD"
                },
                {
                    "amount": 275,
                    "currencyCode": "USD"
                }
            ]
        }
    }

    ```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как запрашивать данные в режиме реального времени с помощью служб Mobility Services (Предварительная версия):

> [!div class="nextstepaction"]
> [Как запросить данные в режиме реального времени](how-to-request-real-time-data.md)

Ознакомьтесь с документацией по API Azure Maps Mobility Services (Предварительная версия)

> [!div class="nextstepaction"]
> [Документация по службам Mobility Services](/rest/api/maps/mobility)
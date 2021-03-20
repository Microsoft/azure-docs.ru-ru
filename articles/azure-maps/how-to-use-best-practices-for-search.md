---
title: Рекомендации по использованию службы поиска Azure Maps | Microsoft Azure Maps
description: Получите и используйте наши рекомендации по использованию службы поиска Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/02/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 11c1938c3c1ccba533f52336fad81ebeaae53b24
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895483"
---
# <a name="best-practices-for-azure-maps-search-service"></a>Рекомендации по использованию службы поиска Azure Maps

Azure Maps [Служба поиска](/rest/api/maps/search) включает API-интерфейсы, которые предлагают различные возможности для поиска адресов, мест, Бизнес-вхождений по имени или категории, а также другие географические сведения. Например,[API нечеткого поиска](/rest/api/maps/search/getsearchfuzzy) позволяет пользователям искать адрес или интересующую точку (достопримечательности).

В этой статье объясняется, как применять различные полезные методики при обращении к данным из службы поиска Azure Maps. Вы узнаете, как:
> [!div class="checklist"]
> * Построение запросов для возврата релевантных соответствий
> * Сужение результатов поиска
> * Сведения о различиях между типами результатов
> * Чтение структуры "Поиск адреса-ответ"

## <a name="prerequisites"></a>Предварительные требования

1. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки.

В этой статье используется [приложение POST](https://www.postman.com/downloads/) для создания вызовов RESTful, но можно выбрать любую среду разработки API.

## <a name="best-practices-to-geocode-addresses"></a>Рекомендации по геокодированию адресов

При поиске полного или частичного адреса с помощью службы поиска Azure Maps программный интерфейс API считывает ключевые слова из вашего поискового запроса. После этого он возвращает координаты долготы и широты нужного адреса. Этот процесс называется *геокодированием*.

Возможность геокодирования в стране (регионе) зависит от доступности данных о дорогах и точности работы службы геокодирования. Дополнительные сведения о возможностях геокодирования Azure Maps в различных странах или регионах см. в разделе [Наличие информации о геокодировании](./geocoding-coverage.md).

### <a name="limit-search-results"></a>Сужение результатов поиска

 API поиска Azure Maps позволяет сужать результаты поиска необходимым образом. Результаты следует сужать, чтобы ваши пользователи видели только релевантные данные.

> [!NOTE]
> API поиска поддерживают больше параметров, чем описано в этой статье.

#### <a name="geobiased-search-results"></a>Географически уточненные результаты поиска

Чтобы уточнить результаты для нужной пользователю географической области, всегда добавляйте как можно больше сведений о расположении. Для этого вам может потребоваться сузить результаты поиска, указав некоторые типы входных данных:

* Задайте параметр `countrySet`. Например, установите для него значение `US,FR`. По умолчанию API выполняет поиск по всему миру, поэтому он может давать нерелевантные результаты. Если в запросе нет параметра `countrySet`, результаты поиска могут оказаться неточными. Например, при поиске города с именем *Бельвью*(Bellevue) возвращаются результаты для США и Франции, так как в обеих этих странах есть город с именем *Бельвью*.

* Для задания ограничивающего прямоугольника можно использовать параметры `btmRight` и `topleft`. Эти параметры ограничивают поиск определенной областью на карте.

* Чтобы влиять на область релевантности результатов, определите параметры координат — `lat` и `lon`. Чтобы задать радиус области поиска, используйте параметр `radius`.


#### <a name="fuzzy-search-parameters"></a>Параметры поиска по нечеткому соответствию

[API-интерфейс Search Fuzzy](/rest/api/maps/search/getsearchfuzzy) Azure Maps рекомендуется использовать, если вы не знаете, какие входные данные введены пользователем в поисковом запросе. Например, входные данные пользователя могут быть адресом или типом интересующей вас точки (достопримечательности), например " *Покупка Mall*". API-интерфейс объединяет поиск точек интереса и геокодирование в канонический *однострочный поиск*: 

* Параметры `minFuzzyLevel` и `maxFuzzyLevel` позволяют выдавать релевантные результаты даже в том случае, если параметры запроса не точно соответствуют необходимой пользователю информации. Чтобы повысить качество поиска и сократить число нерелевантных результатов, задайте для поисковых запросов значения по умолчанию — `minFuzzyLevel=1` и `maxFuzzyLevel=2`. 

    Например, если для параметра `maxFuzzyLevel` задано значение 2, поисковый запрос *рестран* будет соответствовать слову *ресторан*. При необходимости уровни нечеткого соответствия, используемые по умолчанию, можно изменить. 

* Используйте параметр `idxSet`, чтобы задать приоритет для точного набора типов результатов. Чтобы задать приоритет для точного набора результатов, можно отправить разделенный запятыми список индексов. Порядок элементов в списке не имеет значения. Azure Maps поддерживает следующие индексы.

* `Addr` - **Диапазоны адресов**. Адресные точки, которые интерполируются с начала и с конца улицы. Эти точки представляются в виде диапазонов адресов.
* `Geo` - **Географические регионы**. Административно-территориальные единицы страны. Географическим регионом, например, может быть страна, регион, штат или город.
* `PAD` - **Точечные адреса**. Адреса, содержащие название улицы и номер дома. Точечные адреса могут находиться в индексе. Например, *Soquel Dr 2501*. Это самый высокий уровень точности для адресов.  
* `POI` - **Достопримечательности**. Точки на карте, которые считаются интересными или заслуживающими внимания. [API Search Address](/rest/api/maps/search/getsearchaddress) не возвращает расположения достопримечательностей.  
* `Str` - **Улицы**. Улицы на карте.
* `XStr` - **Перекрестки**. Точки пересечения двух улиц.


#### <a name="usage-examples"></a>Примеры использования

* `idxSet=POI` — поиск только достопримечательностей. 

* `idxSet=PAD,Addr` — поиск только адресов. `PAD` указывает точечный адрес, `Addr` указывает диапазон адресов.

### <a name="reverse-geocode-and-filter-for-a-geography-entity-type"></a>Обратное геокодирование и фильтр по типу сущности географических регионов

При выполнении поиска с обратным геокодированием с помощью [API-интерфейса Search Address Reverse](/rest/api/maps/search/getsearchaddressreverse) служба может возвращать полигональные контуры административных областей. Например, может потребоваться получить многоугольник области для города.  Чтобы сузить область поиска до конкретных типов сущности географических регионов, включите в запрос параметр `entityType`. 

Полученный ответ будет содержать идентификатор географического региона и соответствующий тип сущности. Если указать более одной сущности, конечная точка вернет *наименьшую доступную сущность*. Полученный геометрический идентификатор можно использовать для получения геометрического вида географического региона через [службу Search Polygon](/rest/api/maps/search/getsearchpolygon).

#### <a name="sample-request"></a>Пример запроса

```HTTP
https://atlas.microsoft.com/search/address/reverse/json?api-version=1.0&subscription-key={subscription-key}&query=47.6394532,-122.1304551&language=en-US&entityType=Municipality
```

#### <a name="response"></a>Ответ

```JSON
{
    "summary": {
        "queryTime": 14,
        "numResults": 1
    },
    "addresses": [
        {
            "address": {
                "routeNumbers": [],
                "countryCode": "US",
                "countrySubdivision": "WA",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "municipality": "Redmond",
                "country": "United States",
                "countryCodeISO3": "USA",
                "countrySubdivisionName": "Washington"
            },
            "position": "47.639454,-122.130455",
            "dataSources": {
                "geometry": {
                    "id": "00005557-4100-3c00-0000-0000596ae571"
                }
            },
            "entityType": "Municipality"
        }
    ]
}
```

### <a name="set-the-results-language"></a>Задание языка результатов

Чтобы задать язык выдаваемых результатов поиска, используйте параметр `language`. Если для запроса язык не задан, по умолчанию служба поиска использует наиболее распространенный язык в данной стране или данном регионе. Если данные на указанном языке не доступны, используется язык по умолчанию. 

Дополнительные сведения см. в разделе [Поддерживаемые языки в службе Azure Maps](./supported-languages.md).


### <a name="use-predictive-mode-automatic-suggestions"></a>Использование режима прогнозирования (автоматических рекомендаций)

Чтобы найти дополнительные результаты для частичных запросов, задайте для параметра `typeahead` значение `true`. Такой запрос будет интерпретироваться как запрос с частичными входными данными, а поиск перейдет в режим прогнозирования. Если не задать для параметра `typeahead` значение `true`, служба будет предполагать, что ей передана вся необходимая информация.

В указанном ниже примере службе поиска адреса передается запрос *Microso*. Здесь для параметра `typeahead` задается значение `true`. Ответ показывает, что служба поиска интерпретировала этот запрос как частичный. Ответ содержит результаты для автоматически предположенного запроса.

#### <a name="sample-query"></a>Пример запроса

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1.0&typeahead=true&countrySet=US&lat=47.6370891183&lon=-122.123736172&query=Microsoft
```

#### <a name="response"></a>Ответ

```JSON
{
    "summary": {
        "query": "microsoft",
        "queryType": "NON_NEAR",
        "queryTime": 18,
        "numResults": 7,
        "offset": 0,
        "totalResults": 7,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6370891183,
            "lon": -122.123736172
        }
    },
    "results": [
        {
            "type": "Street",
            "id": "US/STR/p0/9438784",
            "score": 2.594099998474121,
            "dist": 314.0590106663596,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
            },
            "position": {
                "lat": 47.63988,
                "lon": -122.12438
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64223,
                    "lon": -122.1256,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63748,
                    "lon": -122.12309,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/1756074",
            "score": 2.592679977416992,
            "dist": 876.0272035824189,
            "address": {
                "streetName": "Microsoft Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Road, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.64032,
                "lon": -122.1344
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64253,
                    "lon": -122.13535,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63816,
                    "lon": -122.13305,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/1470668",
            "score": 2.5290400981903076,
            "dist": 2735.4883918101486,
            "address": {
                "streetName": "Microsoft West Campus Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Bellevue",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98007",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft West Campus Road, Bellevue, WA 98007"
            },
            "position": {
                "lat": 47.65784,
                "lon": -122.14335
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.65785,
                    "lon": -122.14335,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.65784,
                    "lon": -122.14325,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/12812615",
            "score": 2.527509927749634,
            "dist": 2870.9579016916873,
            "address": {
                "streetName": "Microsoft West Campus Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft West Campus Road, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.66034,
                "lon": -122.1404
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.66039,
                    "lon": -122.14325,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.65778,
                    "lon": -122.13749,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/197588",
            "score": 2.4630401134490967,
            "dist": 878.1404663812472,
            "address": {
                "streetName": "157th Avenue Northeast",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "extendedPostalCode": "980525344, 980525398, 980525399",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "157th Avenue Northeast, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.64351,
                "lon": -122.13056
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64473,
                    "lon": -122.13058,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.6425,
                    "lon": -122.13016,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/3033991",
            "score": 2.0754499435424805,
            "dist": 3655467.8844475765,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Yorkmount, Charlotte",
            },
            "position": {
                "lat": 35.14267,
                "lon": -80.91824
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 35.14287,
                    "lon": -80.91839,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 35.14267,
                    "lon": -80.91814,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/8395877",
            "score": 2.0754499435424805,
            "dist": 3655437.0037482483,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Charlotte",
                "municipality": "Charlotte",
                "countrySecondarySubdivision": "Mecklenburg",
                "countryTertiarySubdivision": "Township 1 Charlotte",
                "countrySubdivision": "NC",
                "countrySubdivisionName": "North Carolina",
                "postalCode": "28273",
                "extendedPostalCode": "282738105, 282738106, 282738108, 2827382, 282738200",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Way, Charlotte, NC 28273"
            },
            "position": {
                "lat": 35.14134,
                "lon": -80.9198
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 35.14274,
                    "lon": -80.92159,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 35.14002,
                    "lon": -80.91824,
                    "valid": true
                }
            }
        }
    ]
}
```


### <a name="encode-a-uri-to-handle-special-characters"></a>Кодирование URI-кода для работы со специальными символами 

Чтобы найти адреса перекрестков, необходимо закодировать универсальный код ресурса (URI) для работы со специальными символами в адресе. Рассмотрим следующий пример адреса: *1st Avenue & Union Street, Seattle* (1-я авеню и Юнион-стрит, Сиэтл). В данном случае перед отправкой запроса необходимо закодировать символ амперсанда (`&`). 

Рекомендуется кодировать символьные данные в URI-кодах. В URI-кодах все специальные символы кодируются с помощью знака процента (`%`) и двузначного шестнадцатеричного значения, соответствующего коду данного символа в кодировке UTF-8.

#### <a name="usage-examples"></a>Примеры использования

Начнем с этого адреса:

```
query=1st Avenue & E 111th St, New York
```

Закодируем адрес:

```
query=1st%20Avenue%20%26%20E%20111th%20St%2C%20New%20York
```

Можно использовать следующие методы.

JavaScript или TypeScript:
```javascript
encodeURIComponent(query)
```

C# или Visual Basic:
```csharp
Uri.EscapeDataString(query)
```

Java:
```Java
URLEncoder.encode(query, "UTF-8") 
```

Python:
```Python
import urllib.parse 
urllib.parse.quote(query)
```

C++:
```C++
#include <curl/curl.h>
curl_easy_escape(query)
```

PHP.
```PHP
urlencode(query)
```

Ruby:
```Ruby
CGI::escape(query) 
```

Swift:
```Swift
query.stringByAddingPercentEncodingWithAllowedCharacters(.URLHostAllowedCharacterSet()) 
```

Go:
```Go
import ("net/url") 
url.QueryEscape(query)
```


## <a name="best-practices-for-poi-searching"></a>Рекомендации по поиску точек интереса

При поиске точек интереса результаты можно запрашивать по названию точек интереса. Например, можно искать компанию по ее названию. 

Мы настоятельно рекомендуем использовать параметр `countrySet` для указания стран или регионов, которые должно охватывать ваше приложение. По умолчанию поиск ведется по всему миру. Это слишком широкий поиск, и он может дать ненужные результаты, а также занять много времени.

### <a name="brand-search"></a>Поиск торговых марок

Для улучшения релевантности результатов и данных при поиске точек интереса ответ на поисковый запрос содержит сведения о торговых марках. Эти сведения можно использовать при дальнейшем анализе ответа.

При отправке запроса можно указать разделенный запятыми список торговых марок. Используйте этот список, чтобы сузить результаты до набора конкретных торговых марок путем задания параметра `brandSet`. Порядок элементов в списке не имеет значения. Когда вы указываете несколько списков торговых марок, возвращаемые результаты должны соответствовать по крайней мере одному из ваших списков.

Чтобы изучить поиск торговых марок, давайте создадим запрос на [поиск по категориям точек интереса](/rest/api/maps/search/getsearchpoicategory). В указанном ниже примере мы ищем заправочные станции рядом с кампусом Майкрософт в Редмонде, штат Вашингтон. В ответе содержатся сведения о торговой марке для каждой найденной точки интереса.

#### <a name="sample-query"></a>Пример запроса

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=gas%20station&limit=3&lat=47.6413362&lon=-122.1327968
```

#### <a name="response"></a>Ответ

```JSON
{
    "summary": {
        "query": "gas station",
        "queryType": "NON_NEAR",
        "queryTime": 276,
        "numResults": 3,
        "offset": 0,
        "totalResults": 762680,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6413362,
            "lon": -122.1327968
        }
    },
    "results": [
        {
            "type": "POI",
            "id": "US/POI/p0/8831765",
            "score": 5.6631999015808105,
            "dist": 1037.0280221303253,
            "info": "search:ta:840531000004190-US",
            "poi": {
                "name": "Chevron",
                "phone": "+(1)-(425)-6532200",
                "brands": [
                    {
                        "name": "Chevron"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.chevron.com",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "2444",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Northeast Bellevue, Bellevue",
            },
            "position": {
                "lat": 47.63201,
                "lon": -122.13281
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63291,
                    "lon": -122.13414,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63111,
                    "lon": -122.13148,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.63222,
                        "lon": -122.13312,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "POI",
            "id": "US/POI/p0/8831752",
            "score": 5.662710189819336,
            "dist": 1330.1278248163273,
            "info": "search:ta:840539001100326-US",
            "poi": {
                "name": "76",
                "phone": "+(1)-(425)-7472126",
                "brands": [
                    {
                        "name": "76"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.76.com",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "2421",
                "streetName": "148Th Ave Ne",
                "municipalitySubdivision": "Redmond, Bridle Trails, Bellevue",
                "municipality": "Redmond, Bellevue",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98007",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "2421 148Th Ave Ne, Bellevue, WA 98007",
                "localName": "Bellevue"
            },
            "position": {
                "lat": 47.63187,
                "lon": -122.14365
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63277,
                    "lon": -122.14498,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63097,
                    "lon": -122.14232,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "minor",
                    "position": {
                        "lat": 47.63187,
                        "lon": -122.14374,
                        "valid": true
                    }
                },
                {
                    "type": "main",
                    "position": {
                        "lat": 47.63186,
                        "lon": -122.14313,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "POI",
            "id": "US/POI/p0/8831764",
            "score": 5.662449836730957,
            "dist": 1458.645407416307,
            "info": "search:ta:840539000488527-US",
            "poi": {
                "name": "BROWN BEAR CAR WASH",
                "phone": "+(1)-(425)-6442868",
                "brands": [
                    {
                        "name": "Texaco"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.texaco.com/",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "15248",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Redmond",
            },
            "position": {
                "lat": 47.62843,
                "lon": -122.13628
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62933,
                    "lon": -122.13761,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.62753,
                    "lon": -122.13495,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.62827,
                        "lon": -122.13628,
                        "valid": true
                    }
                }
            ]
        }
    ]
}
```


### <a name="airport-search"></a>Поиск аэропортов

С помощью API поиска точек интереса вы можете искать аэропорты, используя их официальные коды. Например, для поиска международного аэропорта Сиэтл/Такома можно использовать код *SEA*: 

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=SEA 
```

### <a name="nearby-search"></a>Поиск поблизости

Для поиска точек интереса в окрестностях определенного места можно попытаться использовать [API-интерфейс Search Nearby](/rest/api/maps/search/getsearchnearby). Конечная точка возвращает только результаты, касающиеся точек интереса. Она не воспринимает параметры поискового запроса. 

Чтобы сузить результаты поиска, рекомендуется задать радиус поиска.

## <a name="understanding-the-responses"></a>Основные сведения об ответах

Давайте найдем нужный адрес в Сиэтле, отправив запрос на поиск адреса службе поиска Azure Maps. В следующем URL-адресе запроса мы задаем для параметра `countrySet` значение `US`, чтобы искать нужный адрес в США.

### <a name="sample-query"></a>Пример запроса

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1&query=400%20Broad%20Street%2C%20Seattle%2C%20WA&countrySet=US
```

### <a name="supported-types-of-results"></a>Поддерживаемые типы результатов

* **Точечный адрес**. Точка на карте с определенным адресом, содержащим название улицы и номер дома. Это самый высокий уровень точности для адресов. 

* **Диапазон адресов**. Диапазон адресных точек, которые интерполируются с начала и с конца улицы.  

* **Geography**. Области на карте, соответствующие административно-территориальным единицам страны (например, страна, регион, штат или город). 

* **Точка интереса**. Точки на карте, которые считаются интересными или заслуживающими внимания.

* **Улица**. Улицы на карте. Адреса приводятся к координатам широты и долготы улицы, на которой находится указанный адрес. Номера домов могут не учитываться. 

* **Перекресток**. Пересечения улиц. Перекресток — место пересечения двух улиц.

### <a name="response"></a>Ответ

Давайте рассмотрим структуру ответа. В указанном ниже ответе типы результирующих объектов отличаются друг от друга. Если вы посмотрите внимательно, то увидите три типа результирующих объектов.

* Точечный адрес
* Улица
* Перекресток

Обратите внимание, что поиск по адресу не дает результаты по точкам интереса.  

Параметр `Score` для каждого объекта ответа указывает, как показатель соответствия соотносится с аналогичными показателями других объектов в том же ответе. Дополнительные сведения о параметрах объекта ответа см. в разделе [Получение API Search Address](/rest/api/maps/search/getsearchaddress).

```JSON
{
    "summary": {
        "query": "400 broad street seattle wa",
        "queryType": "NON_NEAR",
        "queryTime": 146,
        "numResults": 6,
        "offset": 0,
        "totalResults": 7,
        "fuzzyLevel": 1
    },
    "results": [
        {
            "type": "Point Address",
            "id": "US/PAD/p0/28725082",
            "score": 9.893799781799316,
            "address": {
                "streetNumber": "400",
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.62039,
                "lon": -122.34928
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62129,
                    "lon": -122.35061,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61949,
                    "lon": -122.34795,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.61982,
                        "lon": -122.34886,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "Street",
            "id": "US/STR/p0/6700384",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.61724,
                "lon": -122.35207
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.61825,
                    "lon": -122.35336,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61626,
                    "lon": -122.35078,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/9701953",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.61965,
                "lon": -122.349
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62066,
                    "lon": -122.35041,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61857,
                    "lon": -122.34761,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/11721297",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
                "municipalitySubdivision": "Seattle, Downtown Seattle, Denny Regrade, Belltown",
                "municipality": "Seattle",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98121",
                "extendedPostalCode": "981211237",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Broad Street, Seattle, WA 98121"
            },
            "position": {
                "lat": 47.61825,
                "lon": -122.35078
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.61857,
                    "lon": -122.35078,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61825,
                    "lon": -122.35041,
                    "valid": true
                }
            }
        },
        {
            "type": "Cross Street",
            "id": "US/XSTR/p1/232144",
            "score": 6.754479885101318,
            "address": {
                "streetName": "Broad Street & Valley Street",
                "municipalitySubdivision": "South Lake Union, Seattle",
            },
            "position": {
                "lat": 47.62545,
                "lon": -122.33974
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62635,
                    "lon": -122.34107,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.62455,
                    "lon": -122.33841,
                    "valid": true
                }
            }
        }
    ]
}
```

### <a name="geometry"></a>Geometry

Ответ типа *Geometry* может содержать геометрический идентификатор, возвращаемый в объекте `dataSources` через параметры `geometry` и `id`. Например, вы можете использовать службу [Search Polygon](/rest/api/maps/search/getsearchpolygon) для запроса геометрических данных в формате GeoJSON. С помощью этого формата можно получить контур города или аэропорта для нужного набора сущностей. Затем эти данные о границах можно использовать для [настройки геозоны](./tutorial-geofence.md) или [поиска точек интереса в пределах заданного геометрического контура](/rest/api/maps/search/postsearchinsidegeometry).


Ответы на обращения к API-интерфейсу [Search Address](/rest/api/maps/search/getsearchaddress) или [Search Fuzzy](/rest/api/maps/search/getsearchfuzzy) могут содержать геометрический идентификатор, возвращаемый в объекте `dataSources` через параметры `geometry` и `id`:


```JSON 
"dataSources": { 
        "geometry": { 
            "id": "00005557-4100-3c00-0000-000059690938" // The geometry ID is returned in the dataSources object under "geometry" and "id".
        }
} 
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

> [!div class="nextstepaction"]
> [Создание запросов для службы поиска Azure Maps](./how-to-search-for-address.md)

> [!div class="nextstepaction"]
> [Документация по API службы поиска](/rest/api/maps/search)
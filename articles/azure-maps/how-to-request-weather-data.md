---
title: Запрос данных о погоде в реальном времени и прогнозировании с помощью Azure Mapsных служб погоды (Предварительная версия)
description: Узнайте, как запрашивать данные о погоде в реальном времени (текущий) и прогнозируемый (каждый час, ежечасный, ежедневный) с помощью служб Microsoft Azure карты погоды (Предварительная версия)
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: fe1b337fe3e1dcf499f9a7428f66543108d0c050
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97680409"
---
# <a name="request-real-time-and-forecasted-weather-data-using-azure-maps-weather-services-preview"></a>Запрос данных о погоде в реальном времени и прогнозировании с помощью Azure Mapsных служб погоды (Предварительная версия) 

> [!IMPORTANT]
> Службы погоды Azure Maps в настоящее время предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Mapsные [службы погоды](/rest/api/maps/weather) — это набор интерфейсов API RESTful, позволяющих разработчикам интегрировать в свои решения высокодинамические исторические данные, а также прогнозы погоды и визуализации. В этой статье мы покажем, как запрашивать данные о погоде в режиме реального времени и прогнозировании.

В этой статье вы узнаете, как:

* Запросите данные о погоде в реальном времени (текущие) с помощью [API получения текущих условий](/rest/api/maps/weather/getcurrentconditionspreview).
* Запросите серьезное оповещение о погоде, используя [API оповещений о получении серьезных погоды](/rest/api/maps/weather/getsevereweatheralertspreview).
* Запросите ежедневные прогнозы с помощью [API получения ежедневного прогноза](/rest/api/maps/weather/getdailyforecastpreview).
* Запросите почасовые прогнозы с помощью [API получения почасового прогноза](/rest/api/maps/weather/gethourlyforecastpreview).
* Запросить минуты по поминутному прогнозу с помощью [API-интерфейса получения минут](/rest/api/maps/weather/getminuteforecastpreview).

В этом видеоролике приведены примеры вызова функций RESTFUL для Azure Maps служб погоды.

</br>

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-Maps-Weather-services-for-developers/player?format=ny]

## <a name="prerequisites"></a>Предварительные требования

1. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](./how-to-manage-authentication.md).

    >[!IMPORTANT]
    >Для [API-интерфейса получения минут](/rest/api/maps/weather/getminuteforecastpreview) требуется ключ ценовой категории S1. Для всех других API требуется ключ ценовой категории S0.

В этом руководстве используется приложение [Postman](https://www.postman.com/), но вы можете выбрать другую среду разработки API.

## <a name="request-real-time-weather-data"></a>Запрос данных о погоде в реальном времени

[API получения текущих условий](/rest/api/maps/weather/getcurrentconditionspreview) возвращает подробные Погодные условия, такие как осадков, температура и обмотка, для заданного координатного расположения. Кроме того, можно получить наблюдения за последние 6 или 24 часа для определенного расположения. Ответ содержит такие сведения, как Дата и время наблюдения, краткое описание условий погоды, значок погоды, флаги индикатора осадков и температуру. Также возвращаются индексы Реалфил™ температуры и ултравиолет (UV).

В этом примере вы используете [API получения текущих условий](/rest/api/maps/weather/getcurrentconditionspreview) для получения текущих погодных условий в координатах, расположенных в Сиэтле, штат Вашингтон.

1. Откройте приложение Postman. В верхней части приложения Postman выберите элемент **Создать**. В окне **Create New** (Создание) выберите **Collection** (Коллекция).  Присвойте имя коллекции и нажмите кнопку **Создать**. Эта коллекция будет использоваться для остальных примеров в этом документе.

2. Чтобы создать запрос, нажмите кнопку **Создать** еще раз. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **Имя запроса** для запроса. Выберите коллекцию, созданную на предыдущем шаге, а затем нажмите кнопку **Сохранить**.

3. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/weather/currentConditions/json?api-version=1.0&query=47.60357,-122.32945&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

4. Нажмите синюю кнопку **Отправить** . Текст ответа содержит текущие сведения о погоде.

    ```json
    {
    "results": [
        {
            "dateTime": "2020-10-19T20:39:00+00:00",
            "phrase": "Cloudy",
            "iconCode": 7,
            "hasPrecipitation": false,
            "isDayTime": true,
            "temperature": {
                "value": 12.4,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperature": {
                "value": 13.7,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperatureShade": {
                "value": 13.7,
                "unit": "C",
                "unitType": 17
            },
            "relativeHumidity": 87,
            "dewPoint": {
                "value": 10.3,
                "unit": "C",
                "unitType": 17
            },
            "wind": {
                "direction": {
                    "degrees": 23.0,
                    "localizedDescription": "NNE"
                },
                "speed": {
                    "value": 4.5,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "windGust": {
                "speed": {
                    "value": 9.0,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "uvIndex": 1,
            "uvIndexPhrase": "Low",
            "visibility": {
                "value": 9.7,
                "unit": "km",
                "unitType": 6
            },
            "obstructionsToVisibility": "",
            "cloudCover": 100,
            "ceiling": {
                "value": 1494.0,
                "unit": "m",
                "unitType": 5
            },
            "pressure": {
                "value": 1021.2,
                "unit": "mb",
                "unitType": 14
            },
            "pressureTendency": {
                "localizedDescription": "Steady",
                "code": "S"
            },
            "past24HourTemperatureDeparture": {
                "value": -2.1,
                "unit": "C",
                "unitType": 17
            },
            "apparentTemperature": {
                "value": 15.0,
                "unit": "C",
                "unitType": 17
            },
            "windChillTemperature": {
                "value": 12.2,
                "unit": "C",
                "unitType": 17
            },
            "wetBulbTemperature": {
                "value": 11.3,
                "unit": "C",
                "unitType": 17
            },
            "precipitationSummary": {
                "pastHour": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past3Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past6Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past9Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past12Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past18Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past24Hours": {
                    "value": 0.4,
                    "unit": "mm",
                    "unitType": 3
                }
            },
            "temperatureSummary": {
                "past6Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 14.0,
                        "unit": "C",
                        "unitType": 17
                    }
                },
                "past12Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 14.0,
                        "unit": "C",
                        "unitType": 17
                    }
                },
                "past24Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 15.6,
                        "unit": "C",
                        "unitType": 17
                    }
                }
            }
        }
    ]
    }
    ```

## <a name="request-severe-weather-alerts"></a>Запросить серьезное оповещение о погоде

[API Azure Maps получения серьезных](/rest/api/maps/weather/getsevereweatheralertspreview) оповещений о погоде возвращает серьезные оповещения о погоде, которые доступны по всему миру из официальных правительственных учреждений объекта метеорологическом и ведущих поставщиков оповещений о погоде в мировом регионе. Служба может возвращать такие сведения, как тип оповещения, категория, уровень и подробные описания активных критических оповещений для запрошенного расположения, такие как ураганами, сундерстормс, молнии, тепловые волны или лес. Например, руководители логистики могут визуализировать серьезные Погодные условия на карте, а также бизнес-местоположения и запланированные маршруты, и координировать работу с драйверами и локальными сотрудниками.

В этом примере вы будете использовать [API получения серьезных оповещений о погоде](/rest/api/maps/weather/getsevereweatheralertspreview) для получения текущих условий погоды в координатах, расположенных в ЧЭЙЕННЕ, WY.

>[!NOTE]
>Этот пример извлекает серьезные оповещения о погоде на момент написания этой статьи. Скорее всего, в запрошенном расположении больше нет серьезных оповещений о погоде. Чтобы получить фактические данные серьезного предупреждения при выполнении этого примера, необходимо получить данные в другом координатном расположении.

1. Откройте приложение POST, щелкните **создать** и выберите **запрос**. Введите **Имя запроса** для запроса. Выберите коллекцию, созданную в предыдущем разделе, или создайте новую, а затем нажмите кнопку **сохранить**.

2. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/weather/severe/alerts/json?api-version=1.0&query=41.161079,-104.805450&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Нажмите синюю кнопку **Отправить** . Если нет серьезных оповещений о погоде, текст ответа будет содержать пустой `results[]` массив. При наличии серьезных оповещений о погоде текст ответа содержит нечто вроде следующего ответа JSON:

    ```json
    {
    "results": [
        {
            "countryCode": "US",
            "alertId": 2194734,
            "description": {
                "localized": "Red Flag Warning",
                "english": "Red Flag Warning"
            },
            "category": "FIRE",
            "priority": 54,
            "source": "U.S. National Weather Service",
            "sourceId": 2,
            "alertAreas": [
                {
                    "name": "Platte/Goshen/Central and Eastern Laramie",
                    "summary": "Red Flag Warning in effect until 7:00 PM MDT.  Source: U.S. National Weather Service",
                    "startTime": "2020-10-05T15:00:00+00:00",
                    "endTime": "2020-10-06T01:00:00+00:00",
                    "latestStatus": {
                        "localized": "Continue",
                        "english": "Continue"
                    },
                    "alertDetails": "...RED FLAG WARNING REMAINS IN EFFECT FROM 9 AM THIS MORNING TO\n7 PM MDT THIS EVENING FOR STRONG GUSTY WINDS AND LOW HUMIDITY...\n\n* WHERE...Fire weather zones 303, 304, 305, 306, 307, 308, 309,\n  and 310 in southeast Wyoming. Fire weather zone 313 in Nebraska.\n\n* WIND...West to northwest 15 to 30 MPH with gusts around 40 MPH.\n\n* HUMIDITY...10 to 15 percent.\n\n* IMPACTS...Any fires that develop will likely spread rapidly.\n  Outdoor burning is not recommended.\n\nPRECAUTIONARY/PREPAREDNESS ACTIONS...\n\nA Red Flag Warning means that critical fire weather conditions\nare either occurring now...or will shortly. A combination of\nstrong winds...low relative humidity...and warm temperatures can\ncontribute to extreme fire behavior.\n\n&&",
                    "alertDetailsLanguageCode": "en"
                }
            ]
            },...
        ]
    }
    ```

## <a name="request-daily-weather-forecast-data"></a>Запрос ежедневных данных прогноза погоды

[API получения ежедневного прогноза](/rest/api/maps/weather/getdailyforecastpreview) возвращает подробный ежедневный прогноз погоды, например температуру и обмотка. Запрос может указать, сколько дней следует вернуть: 1, 5, 10, 15, 25 или 45 дней для заданного координатного расположения. Ответ включает такие сведения, как "температура", "обмотка", "осадков", "качество воздуха" и "UV index".  В этом примере мы зазапрашиваем пять дней, установив `duration=5` .

>[!IMPORTANT]
>В ценовой категории S0 можно запросить ежедневный прогноз для следующих 1, 5, 10 и 15 дней. В ценовой категории S1 можно также запросить ежедневный прогноз на следующие 25 дней и 45 дней.

В этом примере вы используете [API получения ежедневного прогноза](/rest/api/maps/weather/getdailyforecastpreview) , чтобы получить 5-дневный прогноз погоды для координат, расположенных в Сиэтле, штат Вашингтон.

1. Откройте приложение POST, щелкните **создать** и выберите **запрос**. Введите **Имя запроса** для запроса. Выберите коллекцию, созданную в предыдущем разделе, или создайте новую, а затем нажмите кнопку **сохранить**.

2. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/weather/forecast/daily/json?api-version=1.0&query=47.60357,-122.32945&duration=5&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Нажмите синюю кнопку **Отправить** . Текст ответа содержит 5-дневные данные прогноза погоды. Для краткости в ответе JSON ниже показан прогноз на первый день.
    ```json
    {
    "summary": {
        "startDate": "2020-10-18T17:00:00+00:00",
        "endDate": "2020-10-19T23:00:00+00:00",
        "severity": 2,
        "phrase": "Snow, mixed with rain at times continuing through Monday evening and a storm total of 3-6 cm",
        "category": "snow/rain"
    },
    "forecasts": [
        {
            "date": "2020-10-19T04:00:00+00:00",
            "temperature": {
                "minimum": {
                    "value": -1.1,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 1.3,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "realFeelTemperature": {
                "minimum": {
                    "value": -6.0,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 0.5,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "realFeelTemperatureShade": {
                "minimum": {
                    "value": -6.0,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 0.7,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "hoursOfSun": 1.8,
            "degreeDaySummary": {
                "heating": {
                    "value": 18.0,
                    "unit": "C",
                    "unitType": 17
                },
                "cooling": {
                    "value": 0.0,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "airAndPollen": [
                {
                    "name": "AirQuality",
                    "value": 23,
                    "category": "Good",
                    "categoryValue": 1,
                    "type": "Ozone"
                },
                {
                    "name": "Grass",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Mold",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Ragweed",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Tree",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "UVIndex",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                }
            ],
            "day": {
                "iconCode": 22,
                "iconPhrase": "Snow",
                "hasPrecipitation": true,
                "precipitationType": "Mixed",
                "precipitationIntensity": "Light",
                "shortPhrase": "Chilly with snow, 2-4 cm",
                "longPhrase": "Chilly with snow, accumulating an additional 2-4 cm",
                "precipitationProbability": 90,
                "thunderstormProbability": 0,
                "rainProbability": 54,
                "snowProbability": 85,
                "iceProbability": 8,
                "wind": {
                    "direction": {
                        "degrees": 36.0,
                        "localizedDescription": "NE"
                    },
                    "speed": {
                        "value": 9.3,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "windGust": {
                    "direction": {
                        "degrees": 70.0,
                        "localizedDescription": "ENE"
                    },
                    "speed": {
                        "value": 25.9,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "totalLiquid": {
                    "value": 4.3,
                    "unit": "mm",
                    "unitType": 3
                },
                "rain": {
                    "value": 0.5,
                    "unit": "mm",
                    "unitType": 3
                },
                "snow": {
                    "value": 2.72,
                    "unit": "cm",
                    "unitType": 4
                },
                "ice": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "hoursOfPrecipitation": 9.0,
                "hoursOfRain": 1.0,
                "hoursOfSnow": 9.0,
                "hoursOfIce": 0.0,
                "cloudCover": 96
            },
            "night": {
                "iconCode": 29,
                "iconPhrase": "Rain and snow",
                "hasPrecipitation": true,
                "precipitationType": "Mixed",
                "precipitationIntensity": "Light",
                "shortPhrase": "Showers of rain and snow",
                "longPhrase": "A couple of showers of rain or snow this evening; otherwise, cloudy; storm total snowfall 1-3 cm",
                "precipitationProbability": 65,
                "thunderstormProbability": 0,
                "rainProbability": 60,
                "snowProbability": 54,
                "iceProbability": 4,
                "wind": {
                    "direction": {
                        "degrees": 16.0,
                        "localizedDescription": "NNE"
                    },
                    "speed": {
                        "value": 16.7,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "windGust": {
                    "direction": {
                        "degrees": 1.0,
                        "localizedDescription": "N"
                    },
                    "speed": {
                        "value": 35.2,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "totalLiquid": {
                    "value": 4.3,
                    "unit": "mm",
                    "unitType": 3
                },
                "rain": {
                    "value": 3.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "snow": {
                    "value": 0.79,
                    "unit": "cm",
                    "unitType": 4
                },
                "ice": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "hoursOfPrecipitation": 4.0,
                "hoursOfRain": 1.0,
                "hoursOfSnow": 3.0,
                "hoursOfIce": 0.0,
                "cloudCover": 94
            },
            "sources": [
                "AccuWeather"
            ]
        },...
    ]
    }
    ```

## <a name="request-hourly-weather-forecast-data"></a>Запрос данных прогноза погоды по часам

[API получения почасового прогноза](/rest/api/maps/weather/gethourlyforecastpreview) возвращает подробный прогноз погоды на час для следующих 1, 12, 24 (1 дня), 72 (3 дня), 120 (5 дней) и 240 часов (10 дней) для заданного координатного расположения. API возвращает такие сведения, как температура, влажность, обмотка, осадков и UV-индекс.

>[!IMPORTANT]
>В ценовой категории S0 можно запросить почасовой прогноз для следующих 1, 12, 24 часов (1 день) и 72 часов (3 дня). В ценовой категории S1 можно также запросить почасовой прогноз для следующих 120 (5 дней) и 240 часов (10 дней).

В этом примере вы используете [API получения почасового прогноза](/rest/api/maps/weather/gethourlyforecastpreview) , чтобы получить почасовую прогноз погоды на следующие 12 часов по координатам, расположенным в Сиэтле, штат Вашингтон.

1. Откройте приложение POST, щелкните **создать** и выберите **запрос**. Введите **Имя запроса** для запроса. Выберите коллекцию, созданную в предыдущем разделе, или создайте новую, а затем нажмите кнопку **сохранить**.

2. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/weather/forecast/hourly/json?api-version=1.0&query=47.60357,-122.32945&duration=12&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Нажмите синюю кнопку **Отправить** . Текст ответа содержит данные прогноза погоды на следующие 12 часов. Для краткости в ответе JSON ниже показан прогноз на первый час.

    ```json
    {
    "forecasts": [
        {
            "date": "2020-10-19T21:00:00+00:00",
            "iconCode": 12,
            "iconPhrase": "Showers",
            "hasPrecipitation": true,
            "precipitationType": "Rain",
            "precipitationIntensity": "Light",
            "isDaylight": true,
            "temperature": {
                "value": 14.7,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperature": {
                "value": 13.3,
                "unit": "C",
                "unitType": 17
            },
            "wetBulbTemperature": {
                "value": 12.0,
                "unit": "C",
                "unitType": 17
            },
            "dewPoint": {
                "value": 9.5,
                "unit": "C",
                "unitType": 17
            },
            "wind": {
                "direction": {
                    "degrees": 242.0,
                    "localizedDescription": "WSW"
                },
                "speed": {
                    "value": 9.3,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "windGust": {
                "speed": {
                    "value": 14.8,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "relativeHumidity": 71,
            "visibility": {
                "value": 9.7,
                "unit": "km",
                "unitType": 6
            },
            "cloudCover": 100,
            "ceiling": {
                "value": 1128.0,
                "unit": "m",
                "unitType": 5
            },
            "uvIndex": 1,
            "uvIndexPhrase": "Low",
            "precipitationProbability": 51,
            "rainProbability": 51,
            "snowProbability": 0,
            "iceProbability": 0,
            "totalLiquid": {
                "value": 0.3,
                "unit": "mm",
                "unitType": 3
            },
            "rain": {
                "value": 0.3,
                "unit": "mm",
                "unitType": 3
            },
            "snow": {
                "value": 0.0,
                "unit": "cm",
                "unitType": 4
            },
            "ice": {
                "value": 0.0,
                "unit": "mm",
                "unitType": 3
            }
        }...
    ]
    }
    ```
## <a name="request-minute-by-minute-weather-forecast-data"></a>Запрос данных прогноза погоды по минутам

 [API «получение прогноза](/rest/api/maps/weather/getminuteforecastpreview) за минуту» возвращает прогнозы по минутам для определенного местоположения в течение следующих 120 минут. Пользователи могут запрашивать прогноз погоды через интервалы от 1 до 5 до 15 минут. Ответ содержит такие сведения, как тип осадков (включая дождя, снег или сочетание обоих), время начала и значение интенсивности осадков (dBZ).

В этом примере используется [API-интерфейс получения минут](/rest/api/maps/weather/getminuteforecastpreview) для получения поминутного прогноза погоды по координатам, расположенным в Сиэтле, штат Вашингтон. Прогноз погоды предоставляется в течение следующих 120 минут. Наши запросы запрашивают, что прогноз выдается через 15-минутные интервалы, но можно настроить параметр в значение 1 или 5 минут.

1. Откройте приложение POST, щелкните **создать** и выберите **запрос**. Введите **Имя запроса** для запроса. Выберите коллекцию, созданную в предыдущем разделе, или создайте новую, а затем нажмите кнопку **сохранить**.

2. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/weather/forecast/minute/json?api-version=1.0&query=47.60357,-122.32945&interval=15&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Нажмите синюю кнопку **Отправить** . Текст ответа содержит данные прогноза погоды за следующие 120 минут с интервалом в 15 минут.

    ```json
    {
    "summary": {
        "briefPhrase60": "No precipitation for at least 60 min",
        "shortPhrase": "No precip for 120 min",
        "briefPhrase": "No precipitation for at least 120 min",
        "longPhrase": "No precipitation for at least 120 min",
        "iconCode": 7
    },
    "intervalSummaries": [
        {
            "startMinute": 0,
            "endMinute": 119,
            "totalMinutes": 120,
            "shortPhrase": "No precip for %MINUTE_VALUE min",
            "briefPhrase": "No precipitation for at least %MINUTE_VALUE min",
            "longPhrase": "No precipitation for at least %MINUTE_VALUE min",
            "iconCode": 7
        }
    ],
    "intervals": [
        {
            "startTime": "2020-10-19T20:51:00+00:00",
            "minute": 0,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:06:00+00:00",
            "minute": 15,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:21:00+00:00",
            "minute": 30,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:36:00+00:00",
            "minute": 45,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:51:00+00:00",
            "minute": 60,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:06:00+00:00",
            "minute": 75,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:21:00+00:00",
            "minute": 90,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:36:00+00:00",
            "minute": 105,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        }
        ]
    }
    ```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Основные понятия Azure Mapsных служб погоды (Предварительная версия)](./weather-services-concepts.md)

> [!div class="nextstepaction"]
> [Azure Mapsные службы погоды (Предварительная версия) REST API](/rest/api/maps/weather)
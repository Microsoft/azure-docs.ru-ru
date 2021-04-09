---
title: Руководство по Объединение данных от датчиков с прогнозом погоды с помощью Записных книжек Azure (Python) в Microsoft Azure Maps
description: В этом учебнике показано, как объединить данные от датчиков с данными Службы погоды Microsoft Azure Maps с помощью Записных книжек Azure (Python).
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 276dd5b7eba33081c5131eba722df91d8685adff
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98678168"
---
# <a name="tutorial-join-sensor-data-with-weather-forecast-data-by-using-azure-notebooks-python"></a>Руководство по Объединение данных от датчиков с прогнозом погоды с помощью Записных книжек Azure (Python)

> [!IMPORTANT]
> Службы погоды Azure Maps в настоящее время предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Сила ветра является одним из источников энергии, способных заменить ископаемое топливо и остановить изменение климата. Поскольку ветер по своей природе не постоянен, операторам ветроэнергетических установок необходимо построить модели машинного обучения (ML) для прогнозирования ветроэнергетической мощности. Этот прогноз необходим для удовлетворения спроса на электричество и обеспечения стабильной работы сетки. Из этого руководства мы узнаем, как объединить данные прогноза погоды из Azure Maps с демонстрационным набором данных. Данные о прогнозе погоды извлекаются с помощью запроса к службам погоды Azure Maps (предварительная версия).

Выполняя данное руководство, вы сделаете следующее:

> [!div class="checklist"]
> * Работать с файлами данных в [Записных книжках Azure](https://notebooks.azure.com) в облаке.
> * Загружать примеры данных из файла.
> * Вызывать REST API Azure Maps с помощью Python.
> * Отображать на карте данные о местоположении.
> * Обогащать пример данных, используя [дневные прогнозы](/rest/api/maps/weather/getdailyforecastpreview) из Azure Maps.
> * Наносить данные прогнозов погоды на графики.


## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

1. Создайте подписку Azure Maps с ценовой категорией S0, следуя инструкциям [по созданию учетной записи](quick-demo-map-app.md#create-an-azure-maps-account).
2. Чтобы получить первичный ключ подписки для учетной записи, выполните инструкции из [этой статьи](quick-demo-map-app.md#get-the-primary-key-for-your-account).


Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](./how-to-manage-authentication.md).

Ознакомьтесь с работой записных книжек Azure и узнайте, как их запускать, по инструкциям из [этой статьи](./tutorial-ev-routing.md#create-an-azure-notebooks-project).

> [!Note]
> Файл записной книжки Jupyter для этого проекта можно скачать из [репозитория записной книжки Jupyter Notebook для Azure Maps](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/tree/master/AzureMapsJupyterSamples/Tutorials/Analyze%20Weather%20Data).

## <a name="load-the-required-modules-and-frameworks"></a>Загрузка необходимых модулей и платформ

Чтобы загрузить все необходимые модули и платформы, выполните приведенный ниже скрипт:

```python
import pandas as pd
import datetime
from IPython.display import Image, display
!pip install aiohttp
import aiohttp
```

## <a name="import-weather-data"></a>Импорт данных о погоде

Для целей этого руководства мы будем использовать показания датчиков погоды, установленных на четырех разных ветрогенераторах. Образец данных состоит из показаний погоды за 30 дней. Эти показания собираются в метеорологических центрах, расположенных рядом с каждой точкой размещения ветрогенератора. Пример данных включает сведения о температуре, скорости и направлении ветра. Скачать эти демонстрационные данные можно [здесь](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook/tree/master/AzureMapsJupyterSamples/Tutorials/Analyze%20Weather%20Data/data). Следующий скрипт импортирует демонстрационные данные в записную книжку Azure.

```python
df = pd.read_csv("./data/weather_dataset_demo.csv")
```

## <a name="request-daily-forecast-data"></a>Запрос данных прогноза по дням

В нашем сценарии мы будем получать прогноз по дням для местоположений каждого датчика. Следующий скрипт вызывает [API прогноза погоды по дням](/rest/api/maps/weather/getdailyforecastpreview) службы погоды Azure Maps (предварительная версия). Этот интерфейс API возвращает ежедневный прогноз погоды для каждого ветрогенератора на 15 дней от текущей даты.


```python
subscription_key = "Your Azure Maps key"

# Get a lists of unique station IDs and their coordinates 
station_ids = pd.unique(df[['StationID']].values.ravel())
coords = pd.unique(df[['latitude','longitude']].values.ravel())

years,months,days = [],[],[]
dates_check=set()
wind_speeds, wind_direction = [], []

# Call azure maps Weather services (Preview) to get daily forecast data for 15 days from current date
session = aiohttp.ClientSession()
j=-1
for i in range(0, len(coords), 2):
    wind_speeds.append([])
    wind_direction.append([])
    
    query = str(coords[i])+', '+str(coords[i+1])
    forecast_response = await(await session.get("https://atlas.microsoft.com/weather/forecast/daily/json?query={}&api-version=1.0&subscription-key={}&duration=15".format(query, subscription_key))).json()
    j+=1
    for day in range(len(forecast_response['forecasts'])):
            date = forecast_response['forecasts'][day]['date'][:10]
            wind_speeds[j].append(forecast_response['forecasts'][day]['day']['wind']['speed']['value'])
            wind_direction[j].append(forecast_response['forecasts'][day]['day']['windGust']['direction']['degrees'])
            
            if date not in dates_check:
                year,month,day= date.split('-')
                years.append(year)
                months.append(month)
                days.append(day)
                dates_check.add(date)
            
await session.close()
```

Следующий скрипт наносит на карту расположения ветрогенераторов, обращаясь к [службе получения изображения карты](/rest/api/maps/render/getmapimage) в Azure Maps.

```python
# Render the turbine locations on the map by calling the Azure Maps Get Map Image service
session = aiohttp.ClientSession()

pins="default|la-25+60|ls12|lc003C62|co9B2F15||'Location A'{} {}|'Location B'{} {}|'Location C'{} {}|'Location D'{} {}".format(coords[1],coords[0],coords[3],coords[2],coords[5],coords[4], coords[7],coords[6])

image_response = "https://atlas.microsoft.com/map/static/png?subscription-key={}&api-version=1.0&layer=basic&style=main&zoom=6&center={},{}&pins={}".format(subscription_key,coords[7],coords[6],pins)

static_map_response = await session.get(image_response)

poi_range_map = await static_map_response.content.read()

await session.close()

display(Image(poi_range_map))
```

![Расположения ветрогенераторов](./media/weather-service-tutorial/location-map.png)


Данные прогноза погоды будут сгруппированы с демонстрационными данными на основе идентификатора станции. Идентификатор станции требуется для центра обработки данных погоды. Эта группировка дополняет демонстрационные данные данными прогноза.

```python
# Group forecasted data for all locations
df = df.reset_index(drop=True)
forecast_data = pd.DataFrame(columns=['StationID','latitude','longitude','Year','Month','Day','DryBulbCelsius','WetBulbFarenheit','WetBulbCelsius','DewPointFarenheit','DewPointCelsius','RelativeHumidity','WindSpeed','WindDirection'])

for i in range(len(station_ids)):
    loc_forecast = pd.DataFrame({'StationID':station_ids[i], 'latitude':coords[0], 'longitude':coords[1], 'Year':years, 'Month':months, 'Day':days, 'WindSpeed':wind_speeds[i], 'WindDirection':wind_direction[i]})
    forecast_data = pd.concat([forecast_data,loc_forecast], axis=0, sort=False)
    
combined_weather_data = pd.concat([df,forecast_data])
grouped_weather_data = combined_weather_data.groupby(['StationID'])
```

В следующей таблице показан объединенный набор исторических и прогнозируемых данных по расположению одного из ветрогенераторов.

```python
# Display data for first location
grouped_weather_data.get_group(station_ids[0]).reset_index()
```

<center>

![Сгруппированные данные](./media/weather-service-tutorial/grouped-data.png)</center>

## <a name="plot-forecast-data"></a>График прогнозируемых данных

Мы будем сопоставлять прогнозные значения с днями, на которые они рассчитываются. Этот график позволяет нам видеть изменения скорости и направления ветра в течение следующих 15 дней.

```python
# Plot wind speed
curr_date = datetime.datetime.now().date()
windsPlot_df = pd.DataFrame({ 'Location A': wind_speeds[0], 'Location B': wind_speeds[1], 'Location C': wind_speeds[2], 'Location D': wind_speeds[3]}, index=pd.date_range(curr_date,periods=15))
windsPlot = windsPlot_df.plot.line()
windsPlot.set_xlabel("Date")
windsPlot.set_ylabel("Wind speed")
```

```python
#Plot wind direction 
windsPlot_df = pd.DataFrame({ 'Location A': wind_direction[0], 'Location B': wind_direction[1], 'Location C': wind_direction[2], 'Location D': wind_direction[3]}, index=pd.date_range(curr_date,periods=15))
windsPlot = windsPlot_df.plot.line()
windsPlot.set_xlabel("Date")
windsPlot.set_ylabel("Wind direction")
```

Приведенные ниже графы визуализируют данные прогноза. Изменение скорости ветра см. на диаграмме слева. Сведения об изменении направления поворота см. на диаграмме справа. Эти данные являются прогнозом на следующие 15 дней с момента запроса данных.

<center>

![График скорости ветра](./media/weather-service-tutorial/speed-date-plot.png)![График направления ветра](./media/weather-service-tutorial/direction-date-plot.png)</center>

Из этого руководства вы узнали, как вызывать REST API для Azure Maps для получения данных прогнозов погоды. Кроме того, теперь вы знаете, как визуализировать данные на графиках.

Дополнительные сведения о вызове REST API для Azure Maps из Записных книжек Azure вы можете получить в [этой статье](./tutorial-ev-routing.md).

Для изучения API-интерфейсов Azure Maps, используемых в этом руководстве, прочтите следующие статьи:

* [Прогноз по дням](/rest/api/maps/weather/getdailyforecastpreview)
* [Render — Get Map Image](/rest/api/maps/render/getmapimage) (Отрисовка — получение изображения карты)

Полный список API-интерфейсов Azure Maps вы найдете на [этой странице](./consumption-model.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов, требующих очистки.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о службе "Записные книжки Azure" см. в следующей статье:

> [!div class="nextstepaction"]
> [Записные книжки Azure](https://notebooks.azure.com)
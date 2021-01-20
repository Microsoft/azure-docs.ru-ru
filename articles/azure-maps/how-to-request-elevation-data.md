---
title: Запрос данных о повышении прав с помощью службы повышения прав Azure Maps (Предварительная версия)
description: Узнайте, как запросить повышение прав с помощью службы повышения прав Azure Maps (Предварительная версия).
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: a303f5e6177d0dc4205eaec8c3b1911e8e004fe3
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/20/2021
ms.locfileid: "98602424"
---
# <a name="request-elevation-data-using-the-azure-maps-elevation-service-preview"></a>Запрос данных о повышении прав с помощью службы повышения прав Azure Maps (Предварительная версия)

> [!IMPORTANT]
> Служба повышения Azure Maps в настоящее время доступна в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[Служба повышения](https://docs.microsoft.com/rest/api/maps/elevation) Azure Maps предоставляет интерфейсы API для запроса данных о повышении прав в любом месте на поверхности земли. Можно запросить выборку данных повышения прав по путям, в определенном ограничивающем прямоугольнике или по определенным координатам. Кроме того, можно использовать [API плитки прорисовки v2](https://docs.microsoft.com/rest/api/maps/renderv2) , чтобы получить данные повышения прав в формате мозаики. Плитки доставляются в виде растрового формата Жеотифф. В этой статье показано, как использовать службу повышения уровня Azure Maps и API плитки для получения данных о повышении прав. Данные повышения прав могут быть запрошены в форматах геоjson и Жеотифф.

## <a name="prerequisites"></a>Предварительные требования

1. [Создание учетной записи Azure Maps в ценовой категории S1](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки.

Дополнительные сведения о проверке подлинности в Azure Maps см. [в подAzure Maps](how-to-manage-authentication.md).

В этой статье используется приложение [POST](https://www.postman.com/) , но вы можете выбрать другую среду разработки API.

## <a name="request-elevation-data-in-raster-tiled-format"></a>Запрос данных о повышении прав в растровом мозаичном формате

Чтобы запросить данные о повышении прав в формате растровой плитки, используйте [API прорисовки карты v2-Get](https://docs.microsoft.com/rest/api/maps/renderv2). Если плитка может быть найдена, API возвращает плитку как Жеотифф. В противном случае API возвращает значение 0. Все плитки растрового DEM используют режим GeoId (уровень моря). В этом примере мы запрашивать данные о повышении прав для MT. Эверест.

>[!TIP]
>Чтобы получить плитку в определенной области на карте мира, необходимо найти правильную плитку на соответствующем уровне масштабирования. Обратите внимание, что Ворлддем охватывает все глобальные ландмасс, но не охватывает океаны.  Дополнительные сведения см. в статье [Zoom levels and tile grid](zoom-levels-and-tile-grid.md) (Возможности масштабирования и параметры сетки).

1. Откройте приложение Postman. В верхней части приложения Postman выберите элемент **Создать**. В окне **Create New** (Создание) выберите **Collection** (Коллекция).  Присвойте имя коллекции и нажмите кнопку **Создать**.

2. Чтобы создать запрос, нажмите кнопку **Создать** еще раз. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **Имя запроса** для запроса. Выберите коллекцию, созданную на предыдущем шаге, а затем нажмите кнопку **Сохранить**.

3. Выберите метод **Get** HTTP на вкладке Построитель и введите следующий URL-адрес, чтобы запросить растровую плитку. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/map/tile?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=2.0&tilesetId=microsoft.dem&zoom=13&x=6074&y=3432
    ```

4. Нажмите кнопку **Отправить**. Вы должны получить растровую плитку, содержащую данные повышения прав в формате Жеотифф. Каждый пиксель в необработанных данных мозаичной плитки имеет тип `float` . Значение каждого пикселя представляет высоту возвышения в метрах.

## <a name="request-elevation-data-in-geojson-format"></a>Запрос данных о повышении прав в формате геоjson

Используйте API-интерфейсы службы повышения прав (Предварительная версия) для запроса данных о повышении прав в формате геоjson. В этом разделе будет показан каждый из трех интерфейсов API:

* [Получение данных для точек](/rest/api/maps/elevation/getdataforpoints)
* [Отправка данных для точек](/rest/api/maps/elevation/postdataforpoints)
* [Получение данных для ломаной линии](https://docs.microsoft.com/rest/api/maps/elevation/getdataforpolyline)
* [Опубликовать данные для ломаной линии](https://docs.microsoft.com/rest/api/maps/elevation/postdataforpolyline)
* [Получить данные для ограничивающего прямоугольника](https://docs.microsoft.com/rest/api/maps/elevation/getdataforboundingbox)

>[!IMPORTANT]
> Если данные не могут быть возвращены, все API возвращают `0` .

### <a name="request-elevation-data-for-points"></a>Запрос данных о повышении прав для точек

В этом примере мы будем использовать [API получения данных для точек](/rest/api/maps/elevation/getdataforpoints) для запроса данных о повышении прав в Mt. Эверест и Чамланг горы. Затем мы будем использовать [API POST Data for Points](/rest/api/maps/elevation/postdataforpoints) для запроса данных о повышении прав с помощью тех же двух точек. Широты и долготы в URL-адресе должны быть в десятичной степени WGS84 (геодезический System).

 >[!IMPORTANT]
 >Из-за ограничения длины символов URL-адреса, равного 2048, невозможно передать более 100 координат как строку с разделителями конвейера в запросе на получение URL-адреса. Если предполагается передавать более 100 координат в виде строки с разделителями-конвейером, используйте данные POST для точек.

1. Чтобы создать запрос, нажмите кнопку **Создать** еще раз. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **Имя запроса** для запроса. Выберите коллекцию, созданную на предыдущем шаге, а затем нажмите кнопку **Сохранить**.

2. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/elevation/point/json?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&points=-73.998672,40.714728|150.644,-34.397
    ```

3. Нажмите кнопку **Отправить**.  Вы получите следующий ответ JSON:

    ```json
    {
    "data": [
        {
            "coordinate": {
                "latitude": 40.714728,
                "longitude": -73.998672
            },
            "elevationInMeter": 12.142355447638208
        },
        {
            "coordinate": {
                "latitude": -34.397,
                "longitude": 150.644
            },
            "elevationInMeter": 384.47041445517846
        }
    ]
    }
    ```

4. Теперь мы вызываем [API POST Data for Points](/rest/api/maps/elevation/postdataforpoints) для получения данных о повышении прав для тех же двух точек. Выберите метод **POST** HTTP на вкладке Построитель и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/elevation/point/json?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

5. В **Заголовках** запроса **POST** установите для параметра `Content-Type` значение `application/json`. В **тексте** укажите сведения о точке координат ниже. Закончив, нажмите кнопку **Отправить**.

     ```json
    [
        {
            "lon": -73.998672,
            "lat": 40.714728
        },
        {
            "lon": 150.644,
            "lat": -34.397
        }
    ]
    ```

### <a name="request-elevation-data-samples-along-a-polyline"></a>Запрос образцов данных о повышении прав на ломаной линии

В этом примере мы будем использовать « [получить данные для ломаной линии](https://docs.microsoft.com/rest/api/maps/elevation/getdataforpolyline) », чтобы запрашивать пять одинаковых пробельных выборок данных, а также прямую линию между координатами в Mt. Эверест и Чамланг горы. Обе координаты должны быть определены в формате Long/lat. Если не указать значение для `samples` параметра, число выборок будет по умолчанию равно 10. Максимальное число выборок — 2 000.

Затем мы будем использовать «получить данные для ломаной линии», чтобы запрашивать три равные пробелы данных по пути. Мы определим точное расположение образцов, передав три пары координат "длинная/Широта".

Наконец, мы будем использовать [API POST Data for ломаной линии](https://docs.microsoft.com/rest/api/maps/elevation/postdataforpolyline) для запроса данных о повышении прав на тех же трех примерах с одинаковыми пробелами.

Широты и долготы в URL-адресе должны быть в десятичной степени WGS84 (геодезический System).

 >[!IMPORTANT]
 >Из-за ограничения длины символов URL-адреса, равного 2048, невозможно передать более 100 координат как строку с разделителями конвейера в запросе на получение URL-адреса. Если предполагается передавать более 100 координат в виде строки с разделителями-конвейером, используйте данные POST для точек.

1. Нажмите кнопку **Создать**. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **имя запроса** и выберите коллекцию. Выберите команду **Сохранить**.

2. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

   ```http
    https://atlas.microsoft.com/elevation/line/json?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&lines=-73.998672,40.714728|150.644,-34.397&samples=5
    ```

3. Нажмите кнопку **Отправить**.  Вы получите следующий ответ JSON:

    ```JSON
    {
        "data": [
            {
                "coordinate": {
                    "latitude": 40.714728,
                    "longitude": -73.998672
                },
                "elevationInMeter": 12.14236
            },
            {
                "coordinate": {
                    "latitude": 21.936796000000001,
                    "longitude": -17.838003999999998
                },
                "elevationInMeter": 0.0
            },
            {
                "coordinate": {
                    "latitude": 3.1588640000000012,
                    "longitude": 38.322664000000003
                },
                "elevationInMeter": 598.66943
            },
            {
                "coordinate": {
                    "latitude": -15.619067999999999,
                    "longitude": 94.483332000000019
                },
                "elevationInMeter": 0.0
            },
            {
                "coordinate": {
                    "latitude": -34.397,
                    "longitude": 150.644
                },
                "elevationInMeter": 384.47041
            }
        ]
    }
    ```

4. Теперь мы зазапрашиваем три примера данных повышения прав вдоль пути между координатами в Mount Эверест, Чамланг и Жанну горы. В разделе **params** скопируйте следующий массив координат для значения `lines` ключа запроса.

    ```html
        86.9797222, 27.775|86.9252778, 27.9880556 | 88.0444444, 27.6822222
    ```

5. Измените `samples` значение ключа запроса на `3` .  Новые значения показаны на рисунке ниже.

     :::image type="content" source="./media/how-to-request-elevation-data/get-elevation-samples.png" alt-text="Получите три образца данных повышения прав.":::

6. Нажмите синюю кнопку **Отправить** . Вы получите следующий ответ JSON:

    ```json
    {
        "data": [
            {
                "coordinate": {
                    "latitude": 27.775,
                    "longitude": 86.9797222
                },
                "elevationInMeter": 7116.0348851572589
            },
            {
                "coordinate": {
                    "latitude": 27.737403546316028,
                    "longitude": 87.411180791156454
                },
                "elevationInMeter": 1798.6945512521534
            },
            {
                "coordinate": {
                    "latitude": 27.682222199999998,
                    "longitude": 88.0444444
                },
                "elevationInMeter": 7016.9372013588072
            }
        ]
    }
    ```

7. Теперь мы вызываем [интерфейс POST Data for ломаной линии](https://docs.microsoft.com/rest/api/maps/elevation/postdataforpolyline) для получения данных о повышении прав для тех же трех точек. Выберите метод **POST** HTTP на вкладке Построитель и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/elevation/line/json?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}&samples=5
    ```

8. В **Заголовках** запроса **POST** установите для параметра `Content-Type` значение `application/json`. В **тексте** укажите сведения о точке координат ниже. Закончив, нажмите кнопку **Отправить**.

     ```json
    [
        {
            "lon": 86.9797222,
            "lat": 27.775
        },
        {
            "lon": 86.9252778,
            "lat": 27.9880556
        },
        {
            "lon": 88.0444444,
            "lat": 27.6822222
        }
    ]
    ```

### <a name="request-elevation-data-by-bounding-box"></a>Запрос данных о повышении прав по ограничивающему прямоугольнику

Теперь мы будем использовать поле [получить данные для ограничивающего прямоугольника](https://docs.microsoft.com/rest/api/maps/elevation/getdataforboundingbox) для запроса данных о повышении прав. Раиниер, штат Вашингтон. Данные повышения прав будут возвращены с одинаковыми местами в пределах ограничивающего прямоугольника. Ограничивающая область, определяемая (2) наборами данных Lat/Long (юго-Широта, Западная Долгота | Северная Широта, Восточная долгота), делится на строки и столбцы. Границы учетной записи ограничивающего прямоугольника для двух (2) строк и двух (2) столбцов. Для вершин сетки, созданных в пересечениях строк и столбцов, возвращаются повышенные права. В одном запросе может быть возвращено до 2000 прав.

В этом примере мы будем указывать строки = 3, а столбцы — 6. в ответе возвращаются 18 значений повышения прав. На следующей схеме значения повышения прав упорядочены, начиная с юго-западной части, а затем продолжаются до Восточной и Южной.  Точки повышения прав нумеруются в том порядке, в котором они возвращаются.

:::image type="content" source="./media/how-to-request-elevation-data/bounding-box.png" border="false" alt-text="Ограничивающие прямоугольники — это углы NE и SE.":::

1. Нажмите кнопку **Создать**. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **имя запроса** и выберите коллекцию. Выберите команду **Сохранить**.

2. На вкладке Построитель выберите метод **Get** HTTP и введите следующий URL-адрес. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/elevation/lattice/json?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&bounds=-121.66853362143818, 46.84646479863713,-121.65853362143818, 46.85646479863713&rows=2&columns=3
    ```

3. Нажмите синюю кнопку **Отправить** . 18 примеров данных повышения прав, по одной для каждой вершины сетки, возвращаются в ответе.

    ```json
    {
    "data": [
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2298.6581875651746
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2306.3980756609963
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2279.3385479564113
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2233.1549264690366
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2196.4485923541492
        },
        {
            "coordinate": {
                "latitude": 46.846464798637129,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 2133.1756767157253
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2345.3227848228803
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2292.2449195443587
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2270.5867788258074
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2296.8311427390604
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2266.0729430891065
        },
        {
            "coordinate": {
                "latitude": 46.849798131970459,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 2242.216346631234
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2378.460838833359
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2327.6761137260387
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2208.3782743402949
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2106.9526472760981
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2054.3270174034078
        },
        {
            "coordinate": {
                "latitude": 46.8531314653038,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 2030.6438331110671
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66853362143819
            },
            "elevationInMeter": 2318.753153399402
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66653362143819
            },
            "elevationInMeter": 2253.88875188271
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66453362143818
            },
            "elevationInMeter": 2136.6145845357587
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66253362143819
            },
            "elevationInMeter": 2073.6734467948486
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.66053362143818
            },
            "elevationInMeter": 2042.994055784251
        },
        {
            "coordinate": {
                "latitude": 46.856464798637127,
                "longitude": -121.65853362143818
            },
            "elevationInMeter": 1988.3631481900356
        }
    ]
    }
    ```

## <a name="samples-use-elevation-service-preview-apis-in-azure-maps-control"></a>Примеры: использование API-интерфейсов службы повышения прав (Предварительная версия) в элементе управления Azure Maps

### <a name="get-elevation-data-by-coordinate-position"></a>Получение данных повышения прав по координатам

На следующем образце веб-страницы показано, как использовать элемент управления картой для отображения данных повышения прав в координатной точке. Когда пользователь перетаскивает маркер, на карте отображаются данные о повышении прав во всплывающем окне.

<br/>

<iframe height="500" style="width:100%;" scrolling="no" title="Получить повышение прав в позиции" src="https://codepen.io/azuremaps/embed/c840b510e113ba7cb32809591d5f96a2?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/c840b510e113ba7cb32809591d5f96a2'>повышение прав на перо в позиции</a> , Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="get-elevation-data-by-bounding-box"></a>Получение данных повышения прав по ограничивающему прямоугольнику

На следующем образце веб-страницы показано, как использовать элемент управления картой для отображения данных повышения прав, содержащихся в ограничивающем прямоугольнике. Пользователь определяет ограничивающий прямоугольник, щелкая `square` значок в левом верхнем углу и рисуя квадрат в любом месте на карте. После этого элемент управления картой будет отображать данные повышения в соответствии с цветами, указанными в правом верхнем углу.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Повышение прав по ограничивающему прямоугольнику" src="https://codepen.io/azuremaps/embed/619c888c70089c3350a3e95d499f3e48?height=500&theme-id=default&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
См. Дополнительные сведения о <a href='https://codepen.io/azuremaps/pen/619c888c70089c3350a3e95d499f3e48'>повышении пера по ограничивающему прямоугольнику</a> Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="get-elevation-data-by-polyline-path"></a>Получение данных о повышении уровня по пути ломаной линии

На следующем примере веб-страницы показано, как использовать элемент управления картой для отображения данных повышения прав вдоль пути. Пользователь определяет путь, щелкнув `Polyline` значок в левом верхнем углу и нарисовав ломаную линию на карте. Затем элемент управления картой визуализирует данные повышения прав в цветах, заданных в ключе, расположенном в правом верхнем углу.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Градиентный путь повышения" src="https://codepen.io/azuremaps/embed/7bee08e5cb13d05cb0a11636b60f14ca?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
См. <a href='https://codepen.io/azuremaps/pen/7bee08e5cb13d05cb0a11636b60f14ca'>градиент пути повышения прав</a> пера Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения об API повышения уровня Azure Maps (Предварительная версия) см. в следующих статьях:

> [!div class="nextstepaction"]
> [Повышение прав (Предварительная версия) — получение данных для длинных координат lat](/rest/api/maps/elevation/getdataforpoints)

> [!div class="nextstepaction"]
> [Повышение прав (Предварительная версия) — получение данных для ограничивающего прямоугольника](https://docs.microsoft.com/rest/api/maps/elevation/getdataforboundingbox)

> [!div class="nextstepaction"]
> [Повышение прав (Предварительная версия) — получение данных для ломаной линии](https://docs.microsoft.com/rest/api/maps/elevation/getdataforpolyline)

> [!div class="nextstepaction"]
> [Прорисовка v2 — получение плитки карты](https://docs.microsoft.com/rest/api/maps/renderv2)

Полный список API-интерфейсов Azure Maps вы найдете здесь:

> [!div class="nextstepaction"]
> [Интерфейсы REST API для Azure Maps](https://docs.microsoft.com/rest/api/maps/)

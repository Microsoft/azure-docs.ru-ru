---
title: Руководство по Поиск нескольких маршрутов по видам транспорта | Microsoft Azure Maps
description: Руководство по поиску маршрутов к точкам интереса для определенных способов передвижения с помощью Azure Maps Узнайте, как реализовать отображение нескольких маршрутов на карте.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/10/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-js
ms.openlocfilehash: 5e7f38e4bbc95e04bf54f8f7a2dd80e21891e8df
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102215499"
---
# <a name="tutorial-find-and-display-routes-for-different-modes-of-travel-using-azure-maps"></a>Руководство по поиску и отображению маршрутов для различных способов передвижения с помощью Azure Maps

В этом учебнике показано, как использовать [службу "Построение маршрутов"](/rest/api/maps/route) и [элемент управления картой](./how-to-use-map-control.md) Azure Maps, чтобы отображать направления маршрута как для частных автомобилей, так и для коммерческих (грузовых) автомобилей с грузами типа `USHazmatClass2`. Кроме того, мы рассмотрим визуализацию данных о дорожном движении на карте в реальном времени. В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание и отображение элемента управления картой на веб-странице;
> * отображение данных о дорожном движении на карте в реальном времени;
> * запрос и отображение на карте частных и коммерческих маршрутов.

## <a name="prerequisites"></a>Предварительные требования

1. Войдите на [портал Azure](https://portal.azure.com).

2. [Создайте учетную запись службы Azure Maps.](quick-demo-map-app.md#create-an-azure-maps-account)

3. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](how-to-manage-authentication.md).

Полный исходный код для этого примера можно получить [здесь](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/truckRoute.html). Работающий пример можно изучить [здесь](https://azuremapscodesamples.azurewebsites.net/?sample=Multiple%20routes%20by%20mode%20of%20travel).

## <a name="create-a-new-web-page-using-the-map-control-api"></a>создание веб-страницы с помощью API элементов управления картой;

В следующих шагах описаны создание и отображение элемента управления картой на веб-странице.

1. На локальном компьютере создайте файл **MapTruckRoute.html**.
2. Скопируйте следующую разметку HTML и вставьте ее в файл.

    ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>Map Route</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

        <!-- Add a reference to the Azure Maps Services Module JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>

        <script>
            var map, datasource, client;

            function GetMap() {
                //Add Map Control JavaScript code here.
            }
        </script>
        <style>
            html,
            body {
                width: 100%;
                height: 100%;
                padding: 0;
                margin: 0;
            }

            #myMap {
                width: 100%;
                height: 100%;
            }
        </style>
    </head>
    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>
    </html>
    ```

     Заголовок этого кода HTML включает файлы ресурсов CSS и JavaScript, размещенные в библиотеке Azure Map Control. Событие `onload` из элемента body вызывает функцию `GetMap`. На следующем шаге мы добавим код инициализации элемента управления картой.

3. Добавьте следующий код JavaScript в функцию `GetMap`. Замените строку `<Your Azure Maps Key>` первичным ключом, скопированным из учетной записи службы Maps.

    ```JavaScript
    //Instantiate a map object
    var map = new atlas.Map("myMap", {
        //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
        authOptions: {
            authType: 'subscriptionKey',
            subscriptionKey: '<Your Azure Maps Key>'
        }
    });
    ```

4. Сохраните файл и откройте его в браузере. Вы увидите простой элемент управления картой.

    :::image type="content" source="./media/tutorial-prioritized-routes/basic-map.png" alt-text="Простейший пример отрисовки карты на элементе управления картой":::

## <a name="render-real-time-traffic-data-on-a-map"></a>отображение данных о дорожном движении на карте в реальном времени;

1. Добавьте следующий код JavaScript в конец функции `GetMap`. Этот код реализует обработчик событий `ready` для элемента управления картой. Остальную часть кода, приведенного в этом руководстве, мы поместим в обработчик события `ready`.

    ```javascript
    map.events.add("ready", function() {
        // Add Traffic Flow to the Map
        map.setTraffic({
            flow: "relative"
        });
    });
    ```

    В обработчике событий `ready` карты для параметра дорожного движения установлено значение `relative`, что соответствует относительно свободному движению. Дополнительные параметры дорожного движения описаны в разделе [Интерфейс TrafficOptions](/javascript/api/azure-maps-control/atlas.trafficoptions).

2. Сохраните файл **MapTruckRoute.html** и обновите страницу браузера. Если вы увеличите любой город, например Лос-Анджелес, то увидите, что улицы отображаются с учетом текущих данных о дорожном движении.

    :::image type="content" source="./media/tutorial-prioritized-routes/traffic-map.png" alt-text="Просмотр данных дорожного движения на карте":::

<a id="queryroutes"></a>

## <a name="define-route-display-rendering"></a>Определение отрисовки для отображения маршрута

В этом руководстве будут вычислены и отображены на карте два маршрута. Первый маршрут будет рассчитан для частного автомобиля (легкового). Второй маршрут будет рассчитан для коммерческого автомобиля (грузового), чтобы продемонстрировать разницу между результатами. При визуализации на карте будет отображен значок для обозначения начальной и конечной точек, а также геометрические объекты линий разного цвета для обозначения каждого пути маршрута. Дополнительные сведения о добавлении слоев линий см. в статье [Добавление слоя линий на карту](map-add-line-layer.md). Дополнительные сведения о слоях символов см. в статье [Добавление слоя символов на карту](map-add-pin.md).

1. В конец обработчика событий `ready` элемента управления картой добавьте следующий код.

    ```JavaScript

    //Create a data source and add it to the map.
    datasource = new atlas.source.DataSource();
    map.sources.add(datasource);

    //Add a layer for rendering the route lines and have it render under the map labels.
    map.layers.add(new atlas.layer.LineLayer(datasource, null, {
        strokeColor: ['get', 'strokeColor'],
        strokeWidth: ['get', 'strokeWidth'],
        lineJoin: 'round',
        lineCap: 'round'
    }), 'labels');

    //Add a layer for rendering point data.
    map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
        iconOptions: {
            image: ['get', 'icon'],
            allowOverlap: true
        },
        textOptions: {
            textField: ['get', 'title'],
            offset: [0, 1.2]
        },
        filter: ['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']] //Only render Point or MultiPoints in this layer.
    }));

    ```


    В обработчике событий `ready` элемента управления картой создается источник данных для хранения маршрута от начала до конца. [Выражения](data-driven-style-expressions-web-sdk.md) используются для извлечения ширины и цвета линии из свойств функции линии маршрута. Чтобы линия маршрута не перекрывала обозначения дорог, мы передали второй параметр со значением `'labels'`.

    Теперь создаем слой символов и привязываем его к источнику данных. Этот слой определяет, как отображаются начальная и конечная точки. Были добавлены выражения для извлечения изображения значка и текстовых подписей из свойств в каждом объекте точки. Чтобы узнать больше о выражениях, ознакомьтесь со [стилистическими выражениями на основе данных](data-driven-style-expressions-web-sdk.md).

2. Задайте в качестве начальной точки вымышленную компанию Fabrikam в Сиэтле, а в качестве конечной точки — офис корпорации Майкрософт.  В конец обработчика событий `ready` элемента управления картой добавьте следующий код.


    ```JavaScript
    //Create the GeoJSON objects which represent the start and end point of the route.
    var startPoint = new atlas.data.Feature(new atlas.data.Point([-122.356099, 47.580045]), {
        title: 'Fabrikam, Inc.',
        icon: 'pin-blue'
    });

    var endPoint = new atlas.data.Feature(new atlas.data.Point([-122.201164, 47.616940]), {
        title: 'Microsoft - Lincoln Square',
        icon: 'pin-round-blue'
    });

    //Add the data to the data source.
    datasource.add([startPoint, endPoint]);

    //Fit the map window to the bounding box defined by the start and end positions.
    map.setCamera({
        bounds: atlas.data.BoundingBox.fromData([startPoint, endPoint]),
        padding: 100
    });

    ```

    Этот код создает два [объекта Point в формате GeoJSON](https://en.wikipedia.org/wiki/GeoJSON), которые будут представлять начальную и конечную точки, и добавляет эти объекты в источник данных.

    Последний блок кода задает позицию камеры, используя данные о широте и долготе начальной и конечной точек. Начальная и конечная точки добавляются в источник данных. Ограничивающий прямоугольник для начальной и конечной точек вычисляется с использованием функции `atlas.data.BoundingBox.fromData`. Этот ограничивающий прямоугольник используется для формирования вида с камер карты по всему пути с помощью функции `map.setCamera`. Для компенсации размеров пикселей значков символов добавляется заполнение. Дополнительные сведения о свойстве setCamera элемента управления картой см. в документации по свойству [setCamera(CameraOptions | CameraBoundsOptions & AnimationOptions)](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-).

3. Сохраните файл **TruckRoute.html** и обновите страницу в браузере. Карта сместится так, что в центре окажется Сиэтл. Синий маркер в форме капли обозначает начальную точку. Круглый синий маркер обозначает конечную точку маршрута.

   :::image type="content" source="./media/tutorial-prioritized-routes/pins-map.png" alt-text="Просмотр карт с начальной и конечной точкой":::

<a id="multipleroutes"></a>

## <a name="request-and-display-private-and-commercial-vehicle-routes-on-a-map"></a>запрос и отображение на карте частных и коммерческих маршрутов.

Из этого раздела вы узнаете, как использовать службу "Построение маршрутов" Azure Maps для получения направлений по движению от одной точки к другой в зависимости от способа передвижения. Мы будем использовать два способа передвижения: грузовик и легковой автомобиль.

>[!TIP]
>Служба "Построение маршрутов" предоставляет интерфейсы API для планирования самого *быстрого*, *короткого*, *экономичного* или *захватывающего* маршрута в зависимости от расстояния, условий дорожного движения и способа передвижения. Кроме того, с помощью этой службы пользователи могут планировать маршруты с учетом исторических данных о плотности трафика. Пользователь может получить прогноз продолжительности поездки для любого заданного времени. Дополнительные сведения см. в статье [Route — Get Route Directions](/rest/api/maps/route/getroutedirections) (Маршрут. Получение направления маршрута).

1. В функции `GetMap` внутри обработчика событий `ready` элемента управления добавьте в код JavaScript следующее:

    ```JavaScript
    // Use SubscriptionKeyCredential with a subscription key
    var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

    // Use subscriptionKeyCredential to create a pipeline
    var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential);

    // Construct the RouteURL object
    var routeURL = new atlas.service.RouteURL(pipeline);
    ```

   `SubscriptionKeyCredential` создает политику `SubscriptionKeyCredentialPolicy` для аутентификации HTTP-запросов для Azure Maps с помощью ключа подписки. `atlas.service.MapsURL.newPipeline()` принимает политику `SubscriptionKeyCredential` и создает экземпляр [конвейера](/javascript/api/azure-maps-rest/atlas.service.pipeline). `routeURL` представляет собой URL-адрес для операций [маршрута](/rest/api/maps/route) Azure Maps.

2. После настройки учетных данных и URL-адреса добавьте приведенный ниже код JavaScript для создания маршрута грузовика из начальной точки в конечную. Этот маршрут создается и отображается для грузовика, перевозящего груз класса `USHazmatClass2`.

    ```JavaScript
    //Start and end point input to the routeURL
    var coordinates= [[startPoint.geometry.coordinates[0], startPoint.geometry.coordinates[1]], [endPoint.geometry.coordinates[0], endPoint.geometry.coordinates[1]]];

    //Make a search route request for a truck vehicle type
    routeURL.calculateRouteDirections(atlas.service.Aborter.timeout(10000), coordinates,{
        travelMode: 'truck',
        vehicleWidth: 2,
        vehicleHeight: 2,
        vehicleLength: 5,
        vehicleLoadType: 'USHazmatClass2'
    }).then((directions) => {
        //Get data features from response
        var data = directions.geojson.getFeatures();

        //Get the route line and add some style properties to it.  
        var routeLine = data.features[0];
        routeLine.properties.strokeColor = '#2272B9';
        routeLine.properties.strokeWidth = 9;

        //Add the route line to the data source. We want this to render below the car route which will likely be added to the data source faster, so insert it at index 0.
        datasource.add(routeLine, 0);
    });
    ```

    Приведенный выше код запрашивает службу "Построение маршрутов" Azure Maps посредством [API направлений маршрута Azure Maps](/javascript/api/azure-maps-rest/atlas.service.routeurl#calculateroutedirections-aborter--geojson-position----calculateroutedirectionsoptions-). Затем линия маршрута извлекается из ответа коллекции компонентов GeoJSON, который извлекается с помощью метода `geojson.getFeatures()`. Наконец, линия маршрута добавляется в источник данных. Мы добавляем ее с индексом 0, чтобы убедиться, что маршрут грузовика отобразится раньше других линий в источнике данных, так как вычисление маршрута грузовика, как правило, будет выполняться медленнее, чем вычисление маршрута легкового автомобиля. Если линия маршрута грузовика добавляется в источник данных после маршрута автомобиля, она будет отображаться над ним. К линии маршрута грузовика добавляются два свойства: цвет линии (синий) и ее ширина (девять пикселей).

    >[!TIP]
    > Чтобы просмотреть все возможные параметры и значения для API направлений маршрута Azure Maps, ознакомьтесь с [параметрами универсального кода ресурса (URI) для запроса направлений маршрута](/rest/api/maps/route/postroutedirections#uri-parameters).

3. Теперь, чтобы создать маршрут для легкового автомобиля, добавьте следующий код JavaScript.

    ```JavaScript
    routeURL.calculateRouteDirections(atlas.service.Aborter.timeout(10000), coordinates).then((directions) => {

        //Get data features from response
        var data = directions.geojson.getFeatures();

        //Get the route line and add some style properties to it.  
        var routeLine = data.features[0];
        routeLine.properties.strokeColor = '#B76DAB';
        routeLine.properties.strokeWidth = 5;

        //Add the route line to the data source. This will add the car route after the truck route.  
        datasource.add(routeLine);
    });
    ```

    Приведенный выше код запрашивает службу "Построение маршрутов" Azure Maps посредством метода [API направлений маршрута Azure Maps](/javascript/api/azure-maps-rest/atlas.service.routeurl#calculateroutedirections-aborter--geojson-position----calculateroutedirectionsoptions-). Затем линия маршрута извлекается из ответа коллекции компонентов GeoJSON, который извлекается с помощью метода `geojson.getFeatures()`. Наконец, линия маршрута добавляется в источник данных. К линии маршрута грузовика добавляются два свойства: цвет линии (сиреневый) и ее ширина (пять пикселей).

4. Сохраните файл **TruckRoute.html** и обновите страницу в браузере. Теперь на карте должны отображаться маршруты для грузовика и легкового автомобиля.

    :::image type="content" source="./media/tutorial-prioritized-routes/prioritized-routes.png" alt-text="Маршруты для частного и коммерческого автомобилей на карте в службе &quot;Построение маршрутов&quot; Azure":::

    Маршрут грузовика отображается толстой синей линией. Маршрут легкового автомобиля отображается тонкой сиреневой линией. Маршрут легкового автомобиля пересекает Лейк-Вашингтон по трассе I-90, проходя через туннели под жилыми районами. Так как туннели проходят вблизи жилых районов, перевозка опасных отходов по ним запрещена. Маршрут грузовика, для которого указан тип груза `USHazmatClass2`, проходит через другую магистраль.

Полный исходный код для этого примера можно получить [здесь](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/truckRoute.html). Работающий пример можно изучить [здесь](https://azuremapscodesamples.azurewebsites.net/?sample=Multiple%20routes%20by%20mode%20of%20travel).

Вы также можете [использовать стилистические выражения на основе данных](data-driven-style-expressions-web-sdk.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов, требующих очистки.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве показано, как создать простой указатель магазинов с помощью Azure Maps.

> [!div class="nextstepaction"]
> [Создание указателя магазинов с помощью Azure Maps](./tutorial-create-store-locator.md)
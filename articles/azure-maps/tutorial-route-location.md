---
title: Руководство по Поиск маршрута к местоположению | Microsoft Azure Maps
description: Руководство по поиску маршрута к точке интереса. Узнайте, как задать координаты адреса и отправить в службу маршрутов Azure Maps запрос на получение маршрута к точке.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/01/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-js
ms.openlocfilehash: 1f1647766eeae917f7a2266ebb8f00fdfab3452e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102210586"
---
# <a name="tutorial-how-to-display-route-directions-using-azure-maps-route-service-and-map-control"></a>Руководство по отображению направлений маршрута с помощью службы "Построение маршрутов" Azure Maps и элемента управления картой

В этом руководстве показано, как применить [API службы "Построение маршрутов"](/rest/api/maps/route) Azure Maps и [элемент управления картой](./how-to-use-map-control.md) для отображения направлений маршрута от начальной до конечной точки. Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * создание и отображение элемента управления картой на веб-странице; 
> * определение отрисовки маршрута через [слои символов](map-add-pin.md) и [слои линий](map-add-line-layer.md);
> * создание и добавление на карту объектов GeoJSON, которые будут представлять начальную и конечную точки;
> * получение направлений маршрута от начальной до конечной точки, используя [API "Получение направлений маршрута"](/rest/api/maps/route/getroutedirections).

Полный исходный код для этого примера можно получить [здесь](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/route.html). Работающий пример можно изучить [здесь](https://azuremapscodesamples.azurewebsites.net/?sample=Route%20to%20a%20destination).

## <a name="prerequisites"></a>Предварительные требования

1. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки.

<a id="getcoordinates"></a>

## <a name="create-and-display-the-map-control"></a>Создание и отображение элемента управления картой

В следующих шагах описаны создание и отображение элемента управления картой на веб-странице.

1. На локальном компьютере создайте файл **MapRoute.html**.
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

     :::image type="content" source="./media/tutorial-route-location/basic-map.png" alt-text="Простейший пример отрисовки карты на элементе управления картой":::

## <a name="define-route-display-rendering"></a>Определение отрисовки для отображения маршрута

В этом руководстве мы выполним отрисовку маршрута с помощью слоя линий. Начальная и конечная точки будут представлены с помощью слоя символов. Дополнительные сведения о добавлении слоев линий см. в статье [Добавление слоя линий на карту](map-add-line-layer.md). Дополнительные сведения о слоях символов см. в статье [Добавление слоя символов на карту](map-add-pin.md).

1. Добавьте следующий код JavaScript в конец функции `GetMap`. Этот код реализует обработчик событий `ready` для элемента управления картой. Остальную часть кода, приведенного в этом руководстве, мы поместим в обработчик события `ready`.

    ```JavaScript
    //Wait until the map resources are ready.
    map.events.add('ready', function() {

        //Create a data source and add it to the map.
        datasource = new atlas.source.DataSource();
        map.sources.add(datasource);

        //Add a layer for rendering the route lines and have it render under the map labels.
        map.layers.add(new atlas.layer.LineLayer(datasource, null, {
            strokeColor: '#2272B9',
            strokeWidth: 5,
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
    });
    ```

    В обработчике событий `ready` элемента управления картой создается источник данных для хранения маршрута от начальной до конечной точки. Чтобы определить, как будет отображаться линия маршрута, мы создаем слой линий и подключаем его к источнику данных.  Чтобы линия маршрута не перекрывала обозначения дорог, мы передали второй параметр со значением `'labels'`.

    Теперь создаем слой символов и привязываем его к источнику данных. Этот слой определяет, как отображаются начальная и конечная точки. Были добавлены выражения для извлечения изображения значка и текстовых подписей из свойств в каждом объекте точки. Чтобы узнать больше о выражениях, ознакомьтесь со [стилистическими выражениями на основе данных](data-driven-style-expressions-web-sdk.md).

2. Задайте в качестве начальной точки офис корпорации Майкрософт, а в качестве конечной — заправку в Сиэтле.  В конец обработчика событий `ready` элемента управления картой добавьте следующий код.

    ```JavaScript
    //Create the GeoJSON objects which represent the start and end points of the route.
    var startPoint = new atlas.data.Feature(new atlas.data.Point([-122.130137, 47.644702]), {
        title: "Redmond",
        icon: "pin-blue"
    });

    var endPoint = new atlas.data.Feature(new atlas.data.Point([-122.3352, 47.61397]), {
        title: "Seattle",
        icon: "pin-round-blue"
    });

    //Add the data to the data source.
    datasource.add([startPoint, endPoint]);

    map.setCamera({
        bounds: atlas.data.BoundingBox.fromData([startPoint, endPoint]),
        padding: 80
    });
    ```

    Этот код создает два [объекта Point в формате GeoJSON](https://en.wikipedia.org/wiki/GeoJSON), которые будут представлять начальную и конечную точки, и добавляет эти объекты в источник данных. 

    Последний блок кода задает позицию камеры, используя данные о широте и долготе начальной и конечной точек. Начальная и конечная точки добавляются в источник данных. Ограничивающий прямоугольник для начальной и конечной точек вычисляется с использованием функции `atlas.data.BoundingBox.fromData`. Этот ограничивающий прямоугольник используется для формирования вида с камер карты по всему пути с помощью функции `map.setCamera`. Для компенсации размеров пикселей значков символов добавляется заполнение. Дополнительные сведения о свойстве setCamera элемента управления картой см. в документации по свойству [setCamera(CameraOptions | CameraBoundsOptions & AnimationOptions)](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-).

3. Сохраните файл **MapRoute.html** и обновите страницу в браузере. Карта сместится так, что в центре окажется Сиэтл. Синий маркер в форме капли обозначает начальную точку. Круглый синий маркер обозначает конечную точку маршрута.

    :::image type="content" source="./media/tutorial-route-location/map-pins.png" alt-text="Просмотр начальной и конечной точки маршрута на карте":::

<a id="getroute"></a>

## <a name="get-route-directions"></a>Получение направлений маршрута

Из этого раздела вы узнаете, как использовать API направлений маршрута Azure Maps, чтобы получить направление маршрута и предполагаемое время прибытия от одной точки к другой.

>[!TIP]
>API-интерфейсы предлагают службы "Построение маршрутов" Azure Maps для планирования маршрутов на основе различных типов, таких как *самые быстрые*, *кратчайшие*, *экономичные* или *захватывающие* маршруты на основе расстояния, плотности трафика и способа перемещения. Кроме того, с помощью этой службы пользователи могут планировать маршруты с учетом исторических данных о плотности трафика. Пользователь может получить прогноз продолжительности поездки для любого заданного времени. Дополнительные сведения см. в статье [Route — Get Route Directions](/rest/api/maps/route/getroutedirections) (Маршрут. Получение направления маршрута).

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

2. Указав учетные данные и URL-адрес, добавьте следующий код в конец обработчика событий `ready` элемента управления. Этот код создает маршрут от начальной точки к конечной точке. `routeURL` обращается к службе "Построение маршрутов" Azure Maps, чтобы рассчитать направления маршрута. Коллекция компонентов GeoJSON из ответа извлекается с помощью метода `geojson.getFeatures()` и добавляется в источник данных.

    ```JavaScript
    //Start and end point input to the routeURL
    var coordinates= [[startPoint.geometry.coordinates[0], startPoint.geometry.coordinates[1]], [endPoint.geometry.coordinates[0], endPoint.geometry.coordinates[1]]];

    //Make a search route request
    routeURL.calculateRouteDirections(atlas.service.Aborter.timeout(10000), coordinates).then((directions) => {
        //Get data features from response
        var data = directions.geojson.getFeatures();
        datasource.add(data);
    });
    ```

3. Сохраните файл **MapRoute.html** и обновите страницу в браузере. Теперь на карте появится маршрут от начальной до конечной точки.

     :::image type="content" source="./media/tutorial-route-location/map-route.png" alt-text="Элемент управления картой в Azure и служба &quot;Построение маршрутов&quot;":::

Полный исходный код для этого примера можно получить [здесь](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/route.html). Работающий пример можно изучить [здесь](https://azuremapscodesamples.azurewebsites.net/?sample=Route%20to%20a%20destination).

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов, требующих очистки.

## <a name="next-steps"></a>Дальнейшие действия

В следующем руководстве показано, как создать запрос маршрута с ограничениями, например по типу транспорта или характеристикам груза. Затем вы попробуете отобразить несколько маршрутов на одной карте.

> [!div class="nextstepaction"]
> [Поиск маршрутов для различных способов перемещения](./tutorial-prioritized-routes.md)
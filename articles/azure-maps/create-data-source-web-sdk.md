---
title: Создание источника данных для карты в Microsoft Azure Maps
description: 'Узнайте, как создать источник данных для схемы. Сведения об источниках данных, используемых Azure Maps веб-пакетом SDK: источники геоjson и векторные плитки.'
author: rbrundritt
ms.author: richbrun
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 9964c99ddfb59811fc67df634b41cede5847ede0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97678845"
---
# <a name="create-a-data-source"></a>Создание источника данных

Azure Maps веб-пакет SDK хранит данные в источниках данных. Использование источников данных оптимизирует операции с данными для выполнения запросов и подготовки к просмотру. В настоящее время существует два типа источников данных:

* **Источник** географического формата. управляет данными необработанных расположений в формате геоjson локально. Подходит для небольших и средних наборов данных (в направлении сотен тысяч фигур).
* **Источник плитки вектора**: загружает данные, отформатированные как векторные плитки для текущего представления карты, на основе системы разбиения на карты. Идеально подходит для больших и огромных наборов данных (миллионы или миллиарды фигур).

## <a name="geojson-data-source"></a>Источник данных геоjson

Источник данных на основе геообъектного формата JSON загружает и сохраняет данные локально с помощью `DataSource` класса. Данные геоjson можно создавать или создавать вручную с помощью вспомогательных классов в пространстве имен [Atlas. Data](/javascript/api/azure-maps-control/atlas.data) . `DataSource`Класс предоставляет функции для импорта локальных и удаленных геоjson файлов. Удаленные геоjson-файлы должны размещаться на конечной точке с поддержкой CORs. `DataSource`Класс предоставляет функциональные возможности для данных точки кластеризации. И можно легко добавлять, удалять и обновлять данные с помощью `DataSource` класса. В следующем коде показано, как можно создавать геообъектные данные JSON в Azure Maps.

```javascript
//Create raw GeoJSON object.
var rawGeoJson = {
     "type": "Feature",
     "geometry": {
         "type": "Point",
         "coordinates": [-100, 45]
     },
     "properties": {
         "custom-property": "value"
     }
};

//Create GeoJSON using helper classes (less error prone and less typing).
var geoJsonClass = new atlas.data.Feature(new atlas.data.Point([-100, 45]), {
    "custom-property": "value"
}); 
```

После создания источники данных можно добавить на карту с помощью `map.sources` свойства, которое является [саурцеманажер](/javascript/api/azure-maps-control/atlas.sourcemanager). В следующем коде показано, как создать объект `DataSource` и добавить его в карту.

```javascript
//Create a data source and add it to the map.
var source = new atlas.source.DataSource();
map.sources.add(source);
```

В следующем коде показаны различные способы добавления данных геообъектного формата JSON в `DataSource` .

```javascript
//GeoJsonData in the following code can be a single or array of GeoJSON features or geometries, a GeoJSON feature colleciton, or a single or array of atlas.Shape objects.

//Add geoJSON object to data source. 
source.add(geoJsonData);

//Load geoJSON data from URL. URL should be on a CORs enabled endpoint.
source.importDataFromUrl(geoJsonUrl);

//Overwrite all data in data source.
source.setShapes(geoJsonData);
```

> [!TIP]
> Допустим, вы хотите перезаписать все данные в `DataSource` . При вызове `clear` `add` функций then карту могут повторно отрисовываться дважды, что может привести к некоторой задержке. Вместо этого используйте `setShapes` функцию, которая удалит и заменит все данные в источнике данных и запустит только одну повторную визуализацию схемы.

## <a name="vector-tile-source"></a>Источник плитки вектора

Источник векторной плитки описывает, как получить доступ к слою векторной плитки. Используйте класс [вектортилесаурце](/javascript/api/azure-maps-control/atlas.source.vectortilesource) для создания экземпляра источника плитки вектора. Векторные мозаичные слои похожи на мозаичные слои, но они не одинаковы. Мозаичный слой — это растровое изображение. Векторные мозаичные слои — это сжатый файл в формате **ПБФ** . Этот сжатый файл содержит данные о векторной карте и один или несколько слоев. Файл может быть визуализирован и применен на стороне клиента в зависимости от стиля каждого слоя. Данные в векторной плитке содержат географические функции в виде точек, линий и многоугольников. Существует несколько преимуществ использования векторных слоев, а не растровых слоев мозаики.

* Размер файла плитки вектора обычно намного меньше, чем эквивалентная растровая плитка. Поэтому используется меньшая пропускная способность. Это означает более низкую задержку, более быструю карту и удобство работы пользователей.
* Так как векторные плитки отображаются на клиенте, они адаптируются к разрешению устройства, на котором они отображаются. В результате отображаемые карты будут более четко определены с помощью меток Crystal Clear.
* Изменение стиля данных в картах вектора не требует повторной загрузки данных, так как на клиенте можно применить новый стиль. В отличие от этого, изменение стиля растрового слоя, как правило, требует загрузки плиток с сервера, а затем применения нового стиля.
* Поскольку данные доставляются в виде векторной формы, для подготовки данных требуется меньше обработки на стороне сервера. В результате новые данные можно сделать доступными быстрее.

Azure Maps соответствует [спецификации плитки вектора мапбокс](https://github.com/mapbox/vector-tile-spec)— открытым стандартом. Azure Maps предоставляет следующие службы векторных плиток в составе платформы:

- [](/rest/api/maps/renderv2/getmaptilepreview)  |  [Сведения о формате данных](https://developer.tomtom.com/maps-api/maps-api-documentation-vector/tile) документации для дорожных плиток
- [](/rest/api/maps/traffic/gettrafficincidenttile)  |  [Сведения о формате данных](https://developer.tomtom.com/traffic-api/traffic-api-documentation-traffic-incidents/vector-incident-tiles) в документации по инцидентам трафика
- [](/rest/api/maps/traffic/gettrafficflowtile)  |  [Сведения о формате данных](https://developer.tomtom.com/traffic-api/traffic-api-documentation-traffic-flow/vector-flow-tiles) документации потока трафика
- Создатель Azure Maps (Предварительная версия) также позволяет создавать пользовательские векторные плитки и получать к ним доступ с помощью [прорисовки плиток v2](/rest/api/maps/renderv2/getmaptilepreview) .

> [!TIP]
> При использовании векторных или растровых изображений из Azure Maps службы отрисовки с веб-пакетом SDK можно заменить `atlas.microsoft.com` заполнительом `{azMapsDomain}` . Этот заполнитель будет заменен тем же доменом, который используется картой, и будет автоматически добавлять те же сведения проверки подлинности. Это значительно упрощает проверку подлинности с помощью службы Render при использовании Azure Active Directory проверки подлинности.

Чтобы отобразить данные из источника векторной плитки на карте, подключите источник к одному из слоев подготовки данных. Все слои, использующие Векторный источник, должны указывать `sourceLayer` значение в параметрах. Следующий код загружает службу плитки векторного потока трафика Azure Maps в качестве источника векторной плитки, а затем отображает ее на карте с помощью слоя линий. Этот источник векторной плитки содержит один набор данных на исходном слое с именем "поток трафика". Данные строки в этом наборе данных имеют свойство с именем `traffic_level` , которое используется в этом коде для выбора цвета и масштабирования размера строк.

```javascript
//Create a vector tile source and add it to the map.
var source = new atlas.source.VectorTileSource(null, {
    tiles: ['https://{azMapsDomain}/traffic/flow/tile/pbf?api-version=1.0&style=relative&zoom={z}&x={x}&y={y}'],
    maxZoom: 22
});
map.sources.add(source);

//Create a layer for traffic flow lines.
var flowLayer = new atlas.layer.LineLayer(source, null, {
    //The name of the data layer within the data source to pass into this rendering layer.
    sourceLayer: 'Traffic flow',

    //Color the roads based on the traffic_level property. 
    strokeColor: [
        'interpolate',
        ['linear'],
        ['get', 'traffic_level'],
        0, 'red',
        0.33, 'orange',
        0.66, 'green'
    ],

    //Scale the width of roads based on the traffic_level property. 
    strokeWidth: [
        'interpolate',
        ['linear'],
        ['get', 'traffic_level'],
        0, 6,
        1, 1
    ]
});

//Add the traffic flow layer below the labels to make the map clearer.
map.layers.add(flowLayer, 'labels');
```

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Слой линий плитки вектора" src="https://codepen.io/azuremaps/embed/wvMXJYJ?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
См. <a href='https://codepen.io/azuremaps/pen/wvMXJYJ'>слой линии плитки вектора</a> пера, Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="connecting-a-data-source-to-a-layer"></a>Подключение источника данных к слою

Данные подготавливаются к просмотру на карте с помощью слоев подготовки отчетов. Один или несколько уровней подготовки отчетов могут ссылаться на один источник данных. Для следующих слоев подготовки отчетов требуется источник данных:

* [Пузырьковый слой](map-add-bubble-layer.md) — отображает данные точек в виде масштабируемых кругов на карте.
* [Слой символов](map-add-pin.md) — отрисовывает данные точек в виде значков или текста.
* [Уровень тепловой карт](map-add-heat-map-layer.md) — отрисовывает данные точек в виде плотной тепловой карте.
* [Слой линии](map-add-shape.md) — прорисовка линии и отображение контура многоугольников. 
* [Слой многоугольников](map-add-shape.md) . Заливка области многоугольника сплошным цветом или узором изображения.

В следующем коде показано, как создать источник данных, добавить его на карту и подключить его к пузырьковой слою. Затем импортируйте данные географической точки JSON из удаленного расположения в источник данных. 

```javascript
//Create a data source and add it to the map.
var source = new atlas.source.DataSource();
map.sources.add(source);

//Create a layer that defines how to render points in the data source and add it to the map.
map.layers.add(new atlas.layer.BubbleLayer(source));

//Load the earthquake data.
source.importDataFromUrl('https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.geojson');
```

Существуют дополнительные слои подготовки к просмотру, которые не подключаются к этим источникам данных, но они напрямую загружают данные для подготовки к просмотру. 

* [Слой изображения](map-add-image-layer.md) — наложение одного изображения поверх схемы и привязка его углов к набору заданных координат.
* [Мозаичный слой](map-add-tile-layer.md) . переносит растровый слой мозаики поверх карты.

## <a name="one-data-source-with-multiple-layers"></a>Один источник данных с несколькими слоями

К одному источнику данных можно подключить несколько слоев. Существует множество различных сценариев, в которых этот вариант полезен. Например, рассмотрим ситуацию, в которой пользователь рисует многоугольник. Мы должны подготовить и заполнить область многоугольника при добавлении пользователем точек на карту. Добавление линии стиля для контура многоугольник упрощает просмотр краев многоугольника по мере рисования пользователем. Чтобы изменить отдельную точку на многоугольнике, мы можем добавить маркер, например ПИН-код или маркер, над каждой позицией.

![Схема, отображающая несколько слоев данные для отрисовки из одного источника данных](media/create-data-source-web-sdk/multiple-layers-one-datasource.png)

Для большинства платформ сопоставления требуется объект многоугольника, объект Line и ПИН-код для каждой из позиций многоугольника. После изменения многоугольника необходимо вручную обновить линию и ПИН-коды, что может быстро стать сложным.

В Azure Maps необходимо всего один многоугольник в источнике данных, как показано в приведенном ниже коде.

```javascript
//Create a data source and add it to the map.
var source = new atlas.source.DataSource();
map.sources.add(source);

//Create a polygon and add it to the data source.
source.add(new atlas.data.Polygon([[[/* Coordinates for polygon */]]]));

//Create a polygon layer to render the filled in area of the polygon.
var polygonLayer = new atlas.layer.PolygonLayer(source, 'myPolygonLayer', {
     fillColor: 'rgba(255,165,0,0.2)'
});

//Create a line layer for greater control of rendering the outline of the polygon.
var lineLayer = new atlas.layer.LineLayer(source, 'myLineLayer', {
     color: 'orange',
     width: 2
});

//Create a bubble layer to render the vertices of the polygon as scaled circles.
var bubbleLayer = new atlas.layer.BubbleLayer(source, 'myBubbleLayer', {
     color: 'orange',
     radius: 5,
     strokeColor: 'white',
     strokeWidth: 2
});

//Add all layers to the map.
map.layers.add([polygonLayer, lineLayer, bubbleLayer]);
```

> [!TIP]
> При добавлении слоев к сопоставлению с помощью `map.layers.add` функции идентификатор или экземпляр существующего слоя можно передать в качестве второго параметра. Это указывает на то, что на карте будет вставлен новый слой, добавляемый под существующим слоем. Кроме передачи идентификатора слоя, этот метод также поддерживает следующие значения.
>
> * `"labels"` — Вставляет новый слой под слоями меток карт.
> * `"transit"` — Вставляет новый слой под слоями «дорога» и «транзитный».

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource)

> [!div class="nextstepaction"]
> [DataSourceOptions](/javascript/api/azure-maps-control/atlas.datasourceoptions)

> [!div class="nextstepaction"]
> [вектортилесаурце](/javascript/api/azure-maps-control/atlas.source.vectortilesource)

> [!div class="nextstepaction"]
> [вектортилесаурцеоптионс](/javascript/api/azure-maps-control/atlas.vectortilesourceoptions)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление всплывающего окна](map-add-popup.md)

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](map-add-pin.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](map-add-shape.md)

> [!div class="nextstepaction"]
> [Добавление тепловой карты](map-add-heat-map-layer.md)

> [!div class="nextstepaction"]
> [Примеры кода](/samples/browse/?products=azure-maps)
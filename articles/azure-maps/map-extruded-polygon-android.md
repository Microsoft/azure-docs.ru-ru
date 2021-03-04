---
title: Добавление слоя объемной фигуры в карту Android | Карты Microsoft Azure
description: Как добавить слой объемной фигуры многоугольника в Microsoft Azure карты пакет SDK для Android.
author: rbrundritt
ms.author: richbrun
ms.date: 02/19/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: b62c2540dc8cf2c7d4f67d465b2d464cbc0c3091
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102055055"
---
# <a name="add-a-polygon-extrusion-layer-to-the-map-android-sdk"></a>Добавление слоя объема фигуры в карту (пакет SDK для Android)

В этой статье показано, как использовать слой объемного многоугольника для отрисовки областей `Polygon` и `MultiPolygon` геометрических объектов в виде вытянутых фигур.

## <a name="use-a-polygon-extrusion-layer"></a>Использование уровня объемной фигуры многоугольника

Соедините слой объема многоугольников с источником данных. Затем загрузили его на карту. Слой объемов многоугольников отображает области `Polygon` компонентов и в `MultiPolygon` виде вытянутых фигур. `height`Свойства и `base` уровня объема фигуры многоугольника определяют базовое расстояние от земли и высоты вытянутой фигуры в **метрах**. В следующем коде показано, как создать многоугольник, добавить его в источник данных и подготовить к просмотру с помощью класса уровня объема многоугольников.

> [!Note]
> `base`Значение, определенное на уровне объема многоугольника, должно быть меньше или равно значению объекта `height` .

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Create a polygon.
source.add(Polygon.fromLngLats(
    Arrays.asList(
        Arrays.asList(
            Point.fromLngLat(-73.95838379859924, 40.80027995478159),
            Point.fromLngLat(-73.98154735565186, 40.76845986171129),
            Point.fromLngLat(-73.98124694824219, 40.767761062136955),
            Point.fromLngLat(-73.97361874580382, 40.76461637311633),
            Point.fromLngLat(-73.97306084632874, 40.76512830937617),
            Point.fromLngLat(-73.97259950637817, 40.76490890860481),
            Point.fromLngLat(-73.9494466781616, 40.79658450499243),
            Point.fromLngLat(-73.94966125488281, 40.79708807289436),
            Point.fromLngLat(-73.95781517028809, 40.80052360358227),
            Point.fromLngLat(-73.95838379859924, 40.80027995478159)
        )
    )
));

//Create and add a polygon extrusion layer to the map below the labels so that they are still readable.
PolygonExtrusionLayer layer = new PolygonExtrusionLayer(source,
    fillColor("#fc0303"),
    fillOpacity(0.7f),
    height(500f)
);

//Create and add a polygon extrusion layer to the map below the labels so that they are still readable.
map.layers.add(layer, "labels");
```

На следующем снимке экрана показан приведенный выше код, отрисовки многоугольник, растянутый по вертикали с помощью слоя объемной фигуры многоугольника.

![Схема с растяжением многоугольника по вертикали с помощью слоя объемной фигуры](media/map-extruded-polygon-android/polygon-extrusion-layer.jpg)

## <a name="add-data-driven-polygons"></a>Добавить управляемые данными многоугольники

Карту хороплет можно визуализировать с помощью слоя объема фигуры многоугольника. Установите `height` Свойства и `fillColor` уровня объемной фигуры в измерение статистической переменной в `Polygon` `MultiPolygon` геометрических компонентах и. В следующем образце кода показана вытянутая хороплетная схема США на основе измерения плотности Генеральной совокупности по штату.

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Import the geojson data and add it to the data source.
Utils.importData("https://azuremapscodesamples.azurewebsites.net/Common/data/geojson/US_States_Population_Density.json", this,  (String result) -> {
    //Parse the data as a GeoJSON Feature Collection.
    FeatureCollection fc = FeatureCollection.fromJson(result);

    //Add the feature collection to the data source.
    source.add(fc);
});

//Create and add a polygon extrusion layer to the map below the labels so that they are still readable.
PolygonExtrusionLayer layer = new PolygonExtrusionLayer(source,
    fillOpacity(0.7f),
    fillColor(
        step(
            get("density"),
            literal("rgba(0, 255, 128, 1)"),
            stop(10, "rgba(9, 224, 118, 1)"),
            stop(20, "rgba(11, 191, 103, 1)"),
            stop(50, "rgba(247, 227, 5, 1)"),
            stop(100, "rgba(247, 199, 7, 1)"),
            stop(200, "rgba(247, 130, 5, 1)"),
            stop(500, "rgba(247, 94, 5, 1)"),
            stop(1000, "rgba(247, 37, 5, 1)")
        )
    ),
    height(
        interpolate(
            linear(),
            get("density"),
            stop(0, 100),
            stop(1200, 960000)
        )
    )
);

//Create and add a polygon extrusion layer to the map below the labels so that they are still readable.
map.layers.add(layer, "labels");
```

На следующем снимке экрана показана хороплетная схема штатов США, которая растягивается по вертикали как вытянутые многоугольники на основе плотности населения.

![Хороплетная схема штатов США, которая растягивается по вертикали как вытянутые многоугольники на основе плотности населения](media/map-extruded-polygon-android/android-extruded-choropleth.jpg)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Создание источника данных](create-data-source-android-sdk.md)

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](data-driven-style-expressions-android-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](how-to-add-shapes-to-android-map.md)

---
title: Добавление пузырькового слоя в карты Android | Карты Microsoft Azure
description: Узнайте, как визуализировать точки на картах как круги с фиксированными размерами. Сведения об использовании пакет SDK для Android Azure Maps для добавления и настройки пузырьковых слоев для этой цели.
author: rbrundritt
ms.author: richbrun
ms.date: 2/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: f3c175f30c5c0e6206f4fee274e0f3f000e55a74
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102100174"
---
# <a name="add-a-bubble-layer-to-a-map-android-sdk"></a>Добавление пузырькового слоя на карту (пакет SDK для Android)

В этой статье показано, как визуализировать данные точек из источника данных в виде пузырькового слоя на карте. Пузырьковые слои отображают точки в виде кругов на карте с фиксированным радиусом пикселя.

> [!TIP]
> Слои пузырьков по умолчанию отображают координаты всех геометрических объектов в источнике данных. Чтобы ограничить слой таким образом, чтобы он отображал только возможности геометрических точек, установите `filter` параметр слоя в значение `eq(geometryType(), "Point")` . Если вы хотите включить компоненты MultiPoint, установите `filter` параметр слоя в значение `any(eq(geometryType(), "Point"), eq(geometryType(), "MultiPoint"))` .

## <a name="prerequisites"></a>Предварительные требования

Не забудьте выполнить действия, описанные в разделе [Краткое руководство. Создание документа приложения Android](quick-android-map.md) . Блоки кода в этой статье можно вставить в `onReady` обработчик событий Maps.

## <a name="add-a-bubble-layer"></a>Добавление слоя пузырьков

Следующий код загружает массив точек в источник данных. Затем он соединяет точки данных с пузырьковым слоем. Пузырьковый слой отображает радиус каждого пузырька с пятью пикселями и цветом заливки белого цвета. И, цвет обводки синего и ширина штриха, равная шести пикселям.

::: zone pivot="programming-language-java-android"

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Create point locations.
Point[] points = new Point[] {
    Point.fromLngLat(-73.985708, 40.75773),
    Point.fromLngLat(-73.985600, 40.76542),
    Point.fromLngLat(-73.985550, 40.77900),
    Point.fromLngLat(-73.975550, 40.74859),
    Point.fromLngLat(-73.968900, 40.78859)
};

//Add multiple points to the data source.
source.add(points);

//Create a bubble layer to render the filled in area of the circle, and add it to the map.
BubbleLayer layer = new BubbleLayer(source, 
    bubbleRadius(5f),
    bubbleColor("white"),
    bubbleStrokeColor("#4288f7"),
    bubbleStrokeWidth(6f)
);

map.layers.add(layer);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create a data source and add it to the map.
val source = DataSource()
map.sources.add(source)

//Create point locations.
val points: Array<Point> = arrayOf<Point>(
    Point.fromLngLat(-73.985708, 40.75773),
    Point.fromLngLat(-73.985600, 40.76542),
    Point.fromLngLat(-73.985550, 40.77900),
    Point.fromLngLat(-73.975550, 40.74859),
    Point.fromLngLat(-73.968900, 40.78859)
)

//Add multiple points to the data source.
source.add(points)

//Create a bubble layer to render the filled in area of the circle, and add it to the map.
val layer = BubbleLayer(
    source,
    bubbleRadius(5f),
    bubbleColor("white"),
    bubbleStrokeColor("#4288f7"),
    bubbleStrokeWidth(6f)
)

map.layers.add(layer)
```

::: zone-end

На следующем снимке экрана показан приведенный выше код для отрисовки точек в пузырьковой слое.

![Сопоставлять с точками, отображаемыми с помощью пузырькового слоя](media/map-add-bubble-layer-android/android-bubble-layer.png)

## <a name="show-labels-with-a-bubble-layer"></a>Отображение меток с помощью слоя пузырьков

Этот код показывает, как использовать пузырьковый слой для отрисовки точки на карте. И, как использовать слой символов для отрисовки метки. Чтобы скрыть значок слоя символов, задайте `iconImage` для параметра значение `"none"` .

::: zone pivot="programming-language-java-android"

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Add a data point to the map.
source.add(Point.fromLngLat(-122.336641,47.627631));

//Add a bubble layer.
map.layers.add(new BubbleLayer(source,
    bubbleRadius(5f),
    bubbleColor("white"),
    bubbleStrokeColor("#4288f7"),
    bubbleStrokeWidth(6f)
));

//Add a symbol layer to display text, hide the icon image.
map.layers.add(new SymbolLayer(source,
    //Hide the icon image.
    iconImage("none"),
    textField("Museum of History & Industry (MOHAI)"),
    textColor("#005995"),
    textOffset(new Float[]{0f, -2.2f})
));
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create a data source and add it to the map.
val source = DataSource()
map.sources.add(source)

//Add a data point to the map.
source.add(Point.fromLngLat(-122.336641, 47.627631))

//Add a bubble layer.
map.layers.add(
    BubbleLayer(
        source,
        bubbleRadius(5f),
        bubbleColor("white"),
        bubbleStrokeColor("#4288f7"),
        bubbleStrokeWidth(6f)
    )
)

//Add a symbol layer to display text, hide the icon image.
map.layers.add(
    SymbolLayer(
        source,  //Hide the icon image.
        iconImage("none"),
        textField("Museum of History & Industry (MOHAI)"),
        textColor("#005995"),
        textOffset(arrayOf(0f, -2.2f))
    )
)
```

::: zone-end

На следующем снимке экрана показан приведенный выше код, отрисовки точку на пузырьковой слое и текстовую метку для точки с уровнем символов.

![Сопоставьте с точкой, отображаемой с помощью пузырькового слоя, и текстовой метки с уровнем символов](media/map-add-bubble-layer-android/android-bubble-symbol-layer.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Создание источника данных](create-data-source-android-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](data-driven-style-expressions-android-sdk.md)

> [!div class="nextstepaction"]
> [Отображение сведений о компоненте](display-feature-information-android.md)

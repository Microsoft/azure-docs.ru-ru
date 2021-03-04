---
title: Задание стиля карты в картах Android | Карты Microsoft Azure
description: Узнайте о двух способах настройки стиля схемы. Сведения об изменении стиля см. в разделе Использование Azure Maps пакет SDK для Android в файле макета или в классе действия.
author: rbrundritt
ms.author: richbrun
ms.date: 02/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: aef8fbacf8302fb5dd4b5fe28afc615c6bf56090
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102100990"
---
# <a name="set-map-style-android-sdk"></a>Задать стиль схемы (пакет SDK для Android)

В этой статье показано два способа установки стилей карт с помощью пакет SDK для Android Azure Maps. Azure Maps имеет шесть различных стилей карт для выбора. Дополнительные сведения о поддерживаемых стилях карт см. [в разделе Поддерживаемые стили карт в Azure Maps](supported-map-styles.md).

## <a name="prerequisites"></a>Предварительные требования

Не забудьте выполнить действия, описанные в разделе [Краткое руководство. Создание документа приложения Android](quick-android-map.md) .

## <a name="set-map-style-in-the-layout"></a>Задать стиль схемы в макете

Вы можете задать стиль схемы в файле макета для класса действия при добавлении элемента управления картой. Следующий код задает Центральный расположение, масштаб и стиль отображения.

```XML
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/mapcontrol"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_centerLat="47.602806"
    app:mapcontrol_centerLng="-122.329330"
    app:mapcontrol_zoom="12"
    app:mapcontrol_style="grayscale_dark"
    />
```

На следующем снимке экрана показан приведенный выше код, отображающий дорожную карту с темным оттенком серого.

![Схема с темно-оттенком серого стиля карт](media/set-android-map-styles/android-grayscale-dark.png)

## <a name="set-map-style-in-code"></a>Задание стиля схемы в коде

Стиль схемы можно задать в коде программно с помощью `setStyle` метода на карте. Следующий код задает центральное расположение и уровень масштабирования с помощью метода Maps `setCamera` и стиля карты `SATELLITE_ROAD_LABELS` .

::: zone pivot="programming-language-java-android"

```java
mapControl.onReady(map -> {

    //Set the camera of the map.
    map.setCamera(center(Point.fromLngLat(-122.33, 47.64)), zoom(14));

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE_ROAD_LABELS));
});
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
mapControl!!.onReady { map: AzureMap ->
    //Set the camera of the map.
    map.setCamera(center(Point.fromLngLat(-122.33, 47.64)), zoom(14))

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE_ROAD_LABELS))
}
```

::: zone-end

На следующем снимке экрана показан приведенный выше код, отображающий карту со стилем меток вспомогательной дороги.

![Стиль схемы с метками спутниковой дороги](media/set-android-map-styles/android-satellite-road-labels.png)

## <a name="setting-the-map-camera"></a>Настройка камеры на карте

Камера на карте управляет тем, какая часть схемы отображается на карте. Камера может располагаться в макете программным способом в коде. При его задании в коде существует два основных метода установки расположения Map. Использование центра и масштаба или передача ограничивающего прямоугольника. В следующем коде показано, как задать все необязательные параметры камеры при использовании `center` и `zoom` .

::: zone pivot="programming-language-java-android"

```java
//Set the camera of the map using center and zoom.
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.64)), 

    //The zoom level. Typically a value between 0 and 22.
    zoom(14),

    //The amount of tilt in degrees the map where 0 is looking straight down.
    pitch(45),

    //Direction the top of the map is pointing in degrees. 0 = North, 90 = East, 180 = South, 270 = West
    bearing(90),

    //The minimum zoom level the map will zoom-out to when animating from one location to another on the map.
    minZoom(10),
    
    //The maximium zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Set the camera of the map using center and zoom.
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.64)), 

    //The zoom level. Typically a value between 0 and 22.
    zoom(14),

    //The amount of tilt in degrees the map where 0 is looking straight down.
    pitch(45),

    //Direction the top of the map is pointing in degrees. 0 = North, 90 = East, 180 = South, 270 = West
    bearing(90),

    //The minimum zoom level the map will zoom-out to when animating from one location to another on the map.
    minZoom(10),
    
    //The maximium zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
)
```

::: zone-end

Часто желательно сосредоточиться на карте на наборе данных. Ограничивающий прямоугольник можно вычислить на основе функций с помощью `MapMath.fromData` метода и передать в `bounds` параметр камеры на карте. При задании представления карт на основе ограничивающего прямоугольника часто бывает полезно указать `padding` значение, которое будет учитывать размер точек, отображаемых в виде пузырьков или символов. В следующем коде показано, как задать все необязательные параметры камеры при использовании ограничивающего прямоугольника для задания расположения камеры.

::: zone pivot="programming-language-java-android"

```java
//Set the camera of the map using a bounding box.
map.setCamera(
    //The area to focus the map on.
    bounds(BoundingBox.fromLngLats(
        //West
        -122.4594,

        //South
        47.4333,
        
        //East
        -122.21866,
        
        //North
        47.75758
    )),

    //Amount of pixel buffer around the bounding box to provide extra space around the bounding box.
    padding(20),

    //The maximium zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Set the camera of the map using a bounding box.
map.setCamera(
    //The area to focus the map on.
    bounds(BoundingBox.fromLngLats(
        //West
        -122.4594,

        //South
        47.4333,
        
        //East
        -122.21866,
        
        //North
        47.75758
    )),

    //Amount of pixel buffer around the bounding box to provide extra space around the bounding box.
    padding(20),

    //The maximium zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
)
```

::: zone-end

Обратите внимание, что пропорции ограничивающего прямоугольника могут не совпадать с пропорциями на карте, так как такая схема часто показывает полный ограничивающий прямоугольник, но часто будет по вертикали или по горизонтали.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer-android.md)

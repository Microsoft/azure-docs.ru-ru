---
title: Добавление слоя изображения на карту Android | Карты Microsoft Azure
description: Узнайте, как добавлять изображения на карту. Сведения об использовании пакет SDK для Android Azure Maps для настройки слоев изображений и наложения изображений на фиксированных наборах координат.
author: rbrundritt
ms.author: richbrun
ms.date: 02/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: e1d99297c0357039606149bdf7e5a526258fc7c5
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102054942"
---
# <a name="add-an-image-layer-to-a-map-android-sdk"></a>Добавление слоя изображения на карту (пакет SDK для Android)

В этой статье показано, как наложить изображение на фиксированный набор координат. Ниже приведено несколько примеров различных типов изображений, которые могут быть наложены на карты:

* Образы, полученные из дроны
* Создание флурпланс
* Исторические или другие специализированные изображения карт
* Схемы сайтов заданий

> [!TIP]
> Слой изображений — это простой способ наложения изображения на карте. Обратите внимание, что крупные изображения могут потреблять много памяти и могут вызвать проблемы с производительностью. В этом случае рассмотрите возможность разбить изображение на плитки и загрузить их на карту как мозаичный слой.

## <a name="add-an-image-layer"></a>Добавление слоя изображений

Следующий код накладывает изображение на карту [Ньюарке, Нью-Джерси, с 1922](https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg) на карте. Это изображение добавляется в `drawable` папку проекта. Слой изображений создается путем задания изображения и координат для четырех углов в формате `[Top Left Corner, Top Right Corner, Bottom Right Corner, Bottom Left Corner]` . Часто желательно добавлять слои изображений под `label` слоем.

::: zone pivot="programming-language-java-android"

```java
//Create an image layer.
ImageLayer layer = new ImageLayer(
    imageCoordinates(
        new Position[] {
            new Position(-74.22655, 40.773941), //Top Left Corner
            new Position(-74.12544, 40.773941), //Top Right Corner
            new Position(-74.12544, 40.712216), //Bottom Right Corner
            new Position(-74.22655, 40.712216)  //Bottom Left Corner
        }
    ),
    setImage(R.drawable.newark_nj_1922)
);

//Add the image layer to the map, below the label layer.
map.layers.add(layer, "labels");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create an image layer.
val layer = ImageLayer(
    imageCoordinates(
        arrayOf<Position>(
            Position(-74.22655, 40.773941),  //Top Left Corner
            Position(-74.12544, 40.773941),  //Top Right Corner
            Position(-74.12544, 40.712216),  //Bottom Right Corner
            Position(-74.22655, 40.712216)   //Bottom Left Corner
        )
    ),
    setImage(R.drawable.newark_nj_1922)
)

//Add the image layer to the map, below the label layer.
map.layers.add(layer, "labels")
```

::: zone-end

Кроме того, можно указать URL-адрес изображения, размещенного в сети. Однако если сценарий допускает, добавьте образ в папку Projects `drawable` , который будет загружаться быстрее, так как образ будет локально доступен и не будет загружаться.

::: zone pivot="programming-language-java-android"

```java
//Create an image layer.
ImageLayer layer = new ImageLayer(
    imageCoordinates(
        new Position[] {
            new Position(-74.22655, 40.773941), //Top Left Corner
            new Position(-74.12544, 40.773941), //Top Right Corner
            new Position(-74.12544, 40.712216), //Bottom Right Corner
            new Position(-74.22655, 40.712216)  //Bottom Left Corner
        }
    ),
    setUrl("https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg")
);

//Add the image layer to the map, below the label layer.
map.layers.add(layer, "labels");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create an image layer.
val layer = ImageLayer(
    imageCoordinates(
        arrayOf<Position>(
            Position(-74.22655, 40.773941),  //Top Left Corner
            Position(-74.12544, 40.773941),  //Top Right Corner
            Position(-74.12544, 40.712216),  //Bottom Right Corner
            Position(-74.22655, 40.712216) //Bottom Left Corner
        )
    ),
    setUrl("https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg")
)

//Add the image layer to the map, below the label layer.
map.layers.add(layer, "labels")
```

::: zone-end

На следующем снимке экрана показана схема Ньюарке, Нью-Джерси, с 1922, наложенная с помощью слоя изображений.

![Схема Ньюарке, Нью Джерси, с 1922, с помощью слоя изображения](media/map-add-image-layer-android/android-image-layer.gif)

## <a name="import-a-kml-file-as-ground-overlay"></a>Импортировать файл КМЛ как наложение заземления

В этом образце показано, как добавить сведения о наложении КМЛ на землю в виде слоя изображения на карте. Наложение заземления КМЛ предоставляет координаты Севера, Юг, Восток и Запад и поворот по часовой стрелке. Но уровень изображения ожидает координаты для каждого угла изображения. Наложение заземления КМЛ в этом примере предназначено для Чартрес Каседрал, а источник — из [некоммерческого](https://commons.wikimedia.org/wiki/File:Chartres.svg/overlay.kml).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="http://www.opengis.net/kml/2.2" xmlns:gx="http://www.google.com/kml/ext/2.2" xmlns:kml="http://www.opengis.net/kml/2.2" xmlns:atom="http://www.w3.org/2005/Atom">
<GroundOverlay>
    <name>Map of Chartres cathedral</name>
    <Icon>
        <href>https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/Chartres.svg/1600px-Chartres.svg.png</href>
        <viewBoundScale>0.75</viewBoundScale>
    </Icon>
    <LatLonBox>
        <north>48.44820923628113</north>
        <south>48.44737203258976</south>
        <east>1.488833825534365</east>
        <west>1.486788581643038</west>
        <rotation>46.44067597839695</rotation>
    </LatLonBox>
</GroundOverlay>
</kml>
```

Код использует статический `getCoordinatesFromEdges` метод из `ImageLayer` класса. Этот метод вычисляет четыре угла изображения, используя сведения о севере, Южной, Восточной, Западной и повороте наложения КМЛ.

::: zone pivot="programming-language-java-android"

```java
//Calculate the corner coordinates of the ground overlay.
Position[] corners = ImageLayer.getCoordinatesFromEdges(
    //North, south, east, west
    48.44820923628113, 48.44737203258976, 1.488833825534365, 1.486788581643038,

    //KML rotations are counter-clockwise, subtract from 360 to make them clockwise.
    360 - 46.44067597839695
);

//Create an image layer.
ImageLayer layer = new ImageLayer(
    imageCoordinates(corners),
    setUrl("https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/Chartres.svg/1600px-Chartres.svg.png")
);

//Add the image layer to the map, below the label layer.
map.layers.add(layer, "labels");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Calculate the corner coordinates of the ground overlay.
val corners: Array<Position> =
    ImageLayer.getCoordinatesFromEdges( //North, south, east, west
        48.44820923628113,
        48.44737203258976,
        1.488833825534365,
        1.486788581643038,  //KML rotations are counter-clockwise, subtract from 360 to make them clockwise.
        360 - 46.44067597839695
    )

//Create an image layer.
val layer = ImageLayer(
    imageCoordinates(corners),
    setUrl("https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/Chartres.svg/1600px-Chartres.svg.png")
)

//Add the image layer to the map, below the label layer.
map.layers.add(layer, "labels")
```

::: zone-end

На следующем снимке экрана показана схема с наложенным наложением КМЛ с использованием слоя изображений.

![Сопоставьте с наложенным наложением КМЛ с помощью слоя изображений](media/map-add-image-layer-android/android-ground-overlay.jpg)

> [!TIP]
> Используйте `getPixels` методы и `getPositions` класса Image Layer для преобразования между географическими координатами позиционированного слоя изображения и координатами пикселей локального изображения.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о способах наложения изображений на карте см. в следующей статье.

> [!div class="nextstepaction"]
> [Добавление слоя фрагментов](how-to-add-tile-layer-android-map.md)
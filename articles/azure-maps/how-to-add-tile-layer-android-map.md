---
title: Добавление мозаичного слоя в карты Android | Карты Microsoft Azure
description: Узнайте, как добавить мозаичный слой на карту. См. пример, использующий Azure Maps пакет SDK для Android для добавления лепестковой диаграммы погоды на карту.
author: rbrundritt
ms.author: richbrun
ms.date: 3/25/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: ac37a4e6d68decdf6780560963a0c534689e8dbb
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105608991"
---
# <a name="add-a-tile-layer-to-a-map-android-sdk"></a>Добавление мозаичного слоя на карту (пакет SDK для Android)

В этой статье показано, как визуализировать мозаичный слой на карте с помощью пакет SDK для Android Azure Maps. Слои фрагментов позволяют накладывать изображения поверх фрагментов карты в Azure Maps. Дополнительные сведения о системе фрагментов Azure Maps см. в статье [Уровни увеличения и параметры сетки](zoom-levels-and-tile-grid.md).

Мозаичный слой загружает плитки с сервера. Эти образы можно предварительно визуализировать и сохранить как любой другой образ на сервере, используя соглашение об именовании, которое понимает мозаичный слой. Или эти образы можно визуализировать с помощью динамической службы, которая создает изображения в режиме реального времени. Существует три разных соглашения об именовании для службы плиток, поддерживаемых Azure Maps классом Тилелайер:

* X, Y, нотация увеличения. В зависимости от уровня увеличения, x — это столбец, а y — позиция строки фрагмента в сетке фрагментов.
* Нотация Quadkey. Сочетание данных x, y и увеличения в одном строковом значении, которое является уникальным идентификатором для фрагмента.
* Ограничивающие прямоугольники ограничивающие прямоугольники можно использовать для указания изображения в формате `{west},{south},{east},{north}` , который обычно используется [службами веб-СОПОСТАВЛЕНИЯ (WMS)](https://www.opengeospatial.org/standards/wms).

> [!TIP]
> Класс TileLayer — отличный способ визуализировать большие объемы данных на карте. С его помощью можно не только создать слой фрагментов из образа, но и отобразить векторные данные в виде слоя фрагментов. При отрисовке векторных данных в виде мозаичного слоя элементу управления картой необходимо загрузить только плитки, размер файла которого может быть намного меньше, чем у данных, которые они представляют. Эта техника часто используется, когда нужно преобразовать для просмотра огромное количество строк данных на карте.

URL-адрес фрагмента, передаваемый в слой фрагментов, должен быть URL-адресом с http/https к ресурсу TileJSON или шаблоном URL-адреса фрагмента, который использует следующие параметры:

* `{x}` — позиция X фрагмента. Также нужны `{y}` и `{z}`.
* `{y}` — позиция Y фрагмента. Также нужны `{x}` и `{z}`.
* `{z}` — уровень увеличения фрагмента. Также нужны `{x}` и `{y}`.
* `{quadkey}` — идентификатор quadkey на основе соглашения об именовании системы фрагментов Bing Maps.
* `{bbox-epsg-3857}` — строка ограничивающего прямоугольника в формате `{west},{south},{east},{north}` в системе пространственных ссылок 3857 EPSG.
* `{subdomain}` — Заполнитель для значений поддомена, если указано значение поддомена.
* `azmapsdomain.invalid` — Заполнитель для согласования домена и проверки подлинности запросов плитки с теми же значениями, которые используются картой. Используйте его при вызове службы плитки, размещенной Azure Maps.

## <a name="prerequisites"></a>Предварительные требования

Чтобы завершить процесс, описанный в этой статье, необходимо установить [Azure Maps пакет SDK для Android](how-to-use-android-map-control-library.md) , чтобы загрузить карту.

## <a name="add-a-tile-layer-to-the-map"></a>Добавление мозаичного слоя на карту

В этом примере показано, как создать мозаичный слой, указывающий на набор плиток. В этом образце используется система заполнения "x, y, Zoom". Источником этого мозаичного слоя является [проект опенсеамап](https://openseamap.org/index.php), который содержит исходные диаграммы морских публика. Часто при просмотре мозаичных слоев желательно иметь возможность четко видеть метки городов на карте. Это поведение можно сделать, вставив мозаичный слой под слоями "метка карты".

::: zone pivot="programming-language-java-android"

```java
TileLayer layer = new TileLayer(
    tileUrl("https://tiles.openseamap.org/seamark/{z}/{x}/{y}.png"),
    opacity(0.8f),
    tileSize(256),
    minSourceZoom(7),
    maxSourceZoom(17)
);

map.layers.add(layer, "labels");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = TileLayer(
    tileUrl("https://tiles.openseamap.org/seamark/{z}/{x}/{y}.png"),
    opacity(0.8f),
    tileSize(256),
    minSourceZoom(7),
    maxSourceZoom(17)
)

map.layers.add(layer, "labels")
```

::: zone-end

На следующем снимке экрана показан приведенный выше код, отображающий мозаичный слой сведений о морских на карте с темно-черным стилем.

![Схема Android, отображающая мозаичный слой](media/how-to-add-tile-layer-android-map/xyz-tile-layer-android.png)

## <a name="add-an-ogc-web-mapping-service-wms"></a>Добавление службы веб-сопоставления OGC (WMS)

Служба веб-сопоставлений (ВМТС) — это стандартный Открытый геопространственный консорциум (OGC) для обслуживания изображений данных карты. В этом формате доступно множество открытых наборов данных, которые можно использовать с Azure Maps. Этот тип службы можно использовать с мозаичным слоем, если служба поддерживает `EPSG:3857` систему координат (CR). При использовании службы WMS задайте для параметров Width и Height то же значение, которое поддерживается службой, и убедитесь, что это же значение задано в `tileSize` параметре. В поле форматированный URL-адрес задайте для `BBOX` параметра службы `{bbox-epsg-3857}` заполнитель.

::: zone pivot="programming-language-java-android"

``` java
TileLayer layer = new TileLayer(
    tileUrl("https://mrdata.usgs.gov/services/gscworld?FORMAT=image/png&HEIGHT=1024&LAYERS=geology&REQUEST=GetMap&STYLES=default&TILED=true&TRANSPARENT=true&WIDTH=1024&VERSION=1.3.0&SERVICE=WMS&CRS=EPSG:3857&BBOX={bbox-epsg-3857}"),
    tileSize(1024)
);

map.layers.add(layer, "labels");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = TileLayer(
    tileUrl("https://mrdata.usgs.gov/services/gscworld?FORMAT=image/png&HEIGHT=1024&LAYERS=geology&REQUEST=GetMap&STYLES=default&TILED=true&TRANSPARENT=true&WIDTH=1024&VERSION=1.3.0&SERVICE=WMS&CRS=EPSG:3857&BBOX={bbox-epsg-3857}"),
    tileSize(1024)
)

map.layers.add(layer, "labels")
```

::: zone-end

На следующем снимке экрана показан приведенный выше код, перекрывающий веб-службу сопоставления данных геологической из [геологическойного опроса США (USGS)](https://mrdata.usgs.gov/) на карте под метками.

![Схема Android с отображением мозаичного слоя WMS](media/how-to-add-tile-layer-android-map/android-tile-layer-wms.jpg)

## <a name="add-an-ogc-web-mapping-tile-service-wmts"></a>Добавление службы плиток веб-сопоставления OGC (ВМТС)

Служба плиток веб-сопоставлений (ВМТС) Открытый геопространственный консорциум является стандартом (OGC) для обслуживания наложения мозаики для карт. В этом формате доступно множество открытых наборов данных, которые можно использовать с Azure Maps. Этот тип службы можно использовать с мозаичным слоем, если служба поддерживает `EPSG:3857` или `GoogleMapsCompatible` систему координат (CR). При использовании службы ВМТС установите для параметров Width и Height то же значение, которое поддерживается службой, и убедитесь, что это же значение задано в `tileSize` параметре. В форматированном URL-адресе Замените следующие заполнители соответствующим образом:

* `{TileMatrix}` => `{z}`
* `{TileRow}` => `{y}`
* `{TileCol}` => `{x}`

::: zone pivot="programming-language-java-android"

``` java
TileLayer layer = new TileLayer(
    tileUrl("https://basemap.nationalmap.gov/arcgis/rest/services/USGSImageryOnly/MapServer/WMTS/tile/1.0.0/USGSImageryOnly/default/GoogleMapsCompatible/{z}/{y}/{x}"),
    tileSize(256),
    bounds(-173.25000107492872, 0.0005794121990209753, 146.12527718104752, 71.506811402077),
    maxSourceZoom(18)
);

map.layers.add(layer, "transit");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = TileLayer(
    tileUrl("https://basemap.nationalmap.gov/arcgis/rest/services/USGSImageryOnly/MapServer/WMTS/tile/1.0.0/USGSImageryOnly/default/GoogleMapsCompatible/{z}/{y}/{x}"),
    tileSize(256),
    bounds(-173.25000107492872, 0.0005794121990209753, 146.12527718104752, 71.506811402077),
    maxSourceZoom(18)
)

map.layers.add(layer, "transit")
```

::: zone-end

На следующем снимке экрана показан приведенный выше код с перераспределением службы плиток веб-сопоставления с помощью [геологической исследования США (USGS)](https://viewer.nationalmap.gov/services/) , расположенной в верхней части карты, под разделом и метками.

![Схема Android, отображающая мозаичный слой ВМТС](media/how-to-add-tile-layer-android-map/android-tile-layer-wmts.jpg)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о способах наложения изображений на карте см. в следующей статье.

> [!div class="nextstepaction"]
> [Слой изображения](map-add-image-layer-android.md)

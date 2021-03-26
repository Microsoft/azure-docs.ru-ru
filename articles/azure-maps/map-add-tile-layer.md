---
title: Добавление мозаичного слоя в карту | Карты Microsoft Azure
description: Узнайте, как переложить изображения на картах. См. пример, использующий веб-пакет SDK Azure Maps для добавления мозаичного слоя, содержащего лепестковую диаграмму погоды на карту.
author: rbrundritt
ms.author: richbrun
ms.date: 3/25/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: e0fda77be23f6ea16d5e64b5d4796813c53f0e94
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105608108"
---
# <a name="add-a-tile-layer-to-a-map"></a>Добавление слоя фрагментов на карту

В этой статье показано, как наложение мозаичного слоя на карте. Слои фрагментов позволяют накладывать изображения поверх фрагментов карты в Azure Maps. Дополнительные сведения о Azure Maps системе мозаичного заполнения см. в разделе [уровни масштабирования и сетка плиток](zoom-levels-and-tile-grid.md).

Мозаичный слой загружает плитки с сервера. Эти образы можно предварительно визуализировать или отобразить в динамическом режиме. Предварительно подготовленные изображения сохраняются как любые другие изображения на сервере с использованием соглашения об именовании, которое понимается мозаичным слоем. Динамически визуализированные изображения используют службу для загрузки изображений близко к реальному времени. Существует три разных соглашения об именовании для службы плиток, поддерживаемых Azure Maps классом [тилелайер](/javascript/api/azure-maps-control/atlas.layer.tilelayer) :

* X, Y, Zoom Notation-X — это столбец, Y — это расположение плитки в сетке плиток, а в записи масштаба — значение, основанное на уровне масштаба.
* Нотация куадкэй. объединяет сведения о x, y и масштабе в одно строковое значение. Это строковое значение превращается в уникальный идентификатор для одной плитки.
* Ограничивающий прямоугольник — укажите изображение в формате координат ограничивающего прямоугольника: `{west},{south},{east},{north}` . Этот формат обычно используется [службами веб-сопоставления (WMS)](https://www.opengeospatial.org/standards/wms).

> [!TIP]
> Класс [TileLayer](/javascript/api/azure-maps-control/atlas.layer.tilelayer) — отличный способ визуализировать большие объемы данных на карте. Можно не только создать мозаичный слой из изображения, но и векторные данные также можно визуализировать как мозаичный слой. При отрисовке векторных данных в виде мозаичного слоя элементу управления картой необходимо загрузить только плитки, размер которых меньше размера файла, чем представленные в них векторные данные. Этот метод обычно используется для визуализации миллионов строк данных на карте.

URL-адрес плитки, переданный в мозаичный слой, должен быть URL-адресом HTTP или HTTPS для ресурса Тилежсон или шаблоном URL-адреса плитки, который использует следующие параметры:

* `{x}` — позиция X фрагмента. Также нужны `{y}` и `{z}`.
* `{y}` — позиция Y фрагмента. Также нужны `{x}` и `{z}`.
* `{z}` — уровень увеличения фрагмента. Также нужны `{x}` и `{y}`.
* `{quadkey}` — идентификатор quadkey на основе соглашения об именовании системы фрагментов Bing Maps.
* `{bbox-epsg-3857}` — строка ограничивающего прямоугольника в формате `{west},{south},{east},{north}` в системе пространственных ссылок 3857 EPSG.
* `{subdomain}` — Заполнитель для значений поддомена, если он указан, `subdomain` будет добавлен.
* `{azMapsDomain}` — Заполнитель для согласования домена и проверки подлинности запросов плитки с теми же значениями, которые используются картой.

## <a name="add-a-tile-layer"></a>Добавление слоя фрагментов

 В этом примере показано, как создать мозаичный слой, указывающий на набор плиток. В этом образце используется система разбиения на окна масштабирования x, y. источником этого мозаичного слоя является [проект опенсеамап](https://openseamap.org/index.php), который содержит исходные диаграммы морских публика. При просмотре лепестковых данных пользователи, в идеале, увидят метки городов при переходе по карте. Это поведение можно реализовать, вставив мозаичный слой под `labels` слоем.

```javascript
//Create a tile layer and add it to the map below the label layer.
map.layers.add(new atlas.layer.TileLayer({
    tileUrl: 'https://tiles.openseamap.org/seamark/{z}/{x}/{y}.png',
    opacity: 0.8,
    tileSize: 256,
    minSourceZoom: 7,
    maxSourceZoom: 17
}), 'labels');
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Слой фрагментов с X, Y и Z' src='//codepen.io/azuremaps/embed/BGEQjG/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/BGEQjG/'>Слой фрагментов с X, Y и Z</a> в Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-an-ogc-web-mapping-service-wms"></a>Добавление службы веб-сопоставления OGC (WMS)

Служба веб-сопоставлений (ВМТС) — это стандартный Открытый геопространственный консорциум (OGC) для обслуживания изображений данных карты. В этом формате доступно множество открытых наборов данных, которые можно использовать с Azure Maps. Этот тип службы можно использовать с мозаичным слоем, если служба поддерживает `EPSG:3857` систему координат (CR). При использовании службы WMS задайте для параметров Width и Height то же значение, которое поддерживается службой, и убедитесь, что это же значение задано в `tileSize` параметре. В поле форматированный URL-адрес задайте для `BBOX` параметра службы `{bbox-epsg-3857}` заполнитель.

На следующем снимке экрана показан приведенный выше код, перекрывающий веб-службу сопоставления данных геологической из [геологическойного опроса США (USGS)](https://mrdata.usgs.gov/) на карте под метками.

<br/>

<iframe height="265" style="width: 100%;" scrolling="no" title="Мозаичный слой WMS" src="https://codepen.io/azuremaps/embed/BapjZqr?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
См. <a href='https://codepen.io/azuremaps/pen/BapjZqr'>слой плиток</a> перьевого WMS, Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-an-ogc-web-mapping-tile-service-wmts"></a>Добавление службы плиток веб-сопоставления OGC (ВМТС)

Служба плиток веб-сопоставлений (ВМТС) Открытый геопространственный консорциум является стандартом (OGC) для обслуживания наложения мозаики для карт. В этом формате доступно множество открытых наборов данных, которые можно использовать с Azure Maps. Этот тип службы можно использовать с мозаичным слоем, если служба поддерживает `EPSG:3857` или `GoogleMapsCompatible` систему координат (CR). При использовании службы ВМТС установите для параметров Width и Height то же значение, которое поддерживается службой, и убедитесь, что это же значение задано в `tileSize` параметре. В форматированном URL-адресе Замените следующие заполнители соответствующим образом:

* `{TileMatrix}` => `{z}`
* `{TileRow}` => `{y}`
* `{TileCol}` => `{x}`

На следующем снимке экрана показан приведенный выше код с перераспределением службы плиток веб-сопоставления с помощью [геологической исследования США (USGS)](https://viewer.nationalmap.gov/services/) , расположенной в верхней части карты, под разделом и метками.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Мозаичный слой ВМТС" src="https://codepen.io/azuremaps/embed/BapjZVY?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
ВМТС () на CodePen см. в разделе <a href='https://codepen.io/azuremaps/pen/BapjZVY'>слой мозаичных</a> элементов пера с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> <a href='https://codepen.io'></a>).
</iframe>

## <a name="customize-a-tile-layer"></a>Настройка слоя фрагментов

Класс мозаичного слоя имеет множество параметров стилей. которые можно опробовать с помощью представленного ниже средства.

<br/>

<iframe height='700' scrolling='no' title='Параметры слоя фрагментов' src='//codepen.io/azuremaps/embed/xQeRWX/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/xQeRWX/'>Параметры слоя фрагментов</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [TileLayer](/javascript/api/azure-maps-control/atlas.layer.tilelayer)

> [!div class="nextstepaction"]
> [TileLayerOptions](/javascript/api/azure-maps-control/atlas.tilelayeroptions)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление слоя изображений](./map-add-image-layer.md)

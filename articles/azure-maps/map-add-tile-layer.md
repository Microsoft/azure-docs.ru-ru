---
title: Добавление мозаичного слоя в карту | Карты Microsoft Azure
description: Узнайте, как переложить изображения на картах. См. пример, использующий веб-пакет SDK Azure Maps для добавления мозаичного слоя, содержащего лепестковую диаграмму погоды на карту.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: b3619995739c51d68b00f37ebea3a38680a6b6e7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92890983"
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

 В этом примере показано, как создать мозаичный слой, указывающий на набор плиток. В этом образце используется система разбиения на окна масштабирования x, y. Источником этого слоя фрагментов является наложение радара погоды из [лаборатории окружающей среды Университета штата Айова](https://mesonet.agron.iastate.edu/ogc/). При просмотре лепестковых данных пользователи, в идеале, увидят метки городов при переходе по карте. Это поведение можно реализовать, вставив мозаичный слой под `labels` слоем.

```javascript
//Create a tile layer and add it to the map below the label layer.
//Weather radar tiles from Iowa Environmental Mesonet of Iowa State University.
map.layers.add(new atlas.layer.TileLayer({
    tileUrl: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png',
    opacity: 0.8,
    tileSize: 256
}), 'labels');
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Слой фрагментов с X, Y и Z' src='//codepen.io/azuremaps/embed/BGEQjG/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/BGEQjG/'>Слой фрагментов с X, Y и Z</a> в Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
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
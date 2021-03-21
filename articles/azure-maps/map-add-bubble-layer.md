---
title: Добавление пузырькового слоя в карту | Карты Microsoft Azure
description: Узнайте, как визуализировать точки на картах как круги с фиксированными размерами. См. статью использование веб-пакета SDK Azure Maps для добавления и настройки пузырьковых слоев для этой цели.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: cae29dcc0d334a2296199da0d8e3bc4562e275d3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895334"
---
# <a name="add-a-bubble-layer-to-a-map"></a>Добавление слоя пузырьков на карту

В этой статье показано, как визуализировать данные точек из источника данных в виде пузырькового слоя на карте. Пузырьковые слои отображают точки в виде кругов на карте с фиксированным радиусом пикселя. 

> [!TIP]
> Слои пузырьков по умолчанию отображают координаты всех геометрических объектов в источнике данных. Чтобы ограничить слой таким образом, чтобы он отображал только возможности геометрических точек, задайте `filter` для свойства слоя значение `['==', ['geometry-type'], 'Point']` или `['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']]` , если требуется включить компоненты MultiPoint.

## <a name="add-a-bubble-layer"></a>Добавление слоя пузырьков

Следующий код загружает массив точек в источник данных. Затем он соединяет точки данных с [пузырьковым слоем](/javascript/api/azure-maps-control/atlas.layer.bubblelayer). Пузырьковый слой отображает радиус каждого пузырька с пятью пикселями и цветом заливки белого цвета. И, цвет обводки синего и ширина штриха, равная шести пикселям. 

```javascript
//Add point locations.
var points = [
    new atlas.data.Point([-73.985708, 40.75773]),
    new atlas.data.Point([-73.985600, 40.76542]),
    new atlas.data.Point([-73.985550, 40.77900]),
    new atlas.data.Point([-73.975550, 40.74859]),
    new atlas.data.Point([-73.968900, 40.78859])
];

//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Add multiple points to the data source.
dataSource.add(points);

//Create a bubble layer to render the filled in area of the circle, and add it to the map.
map.layers.add(new atlas.layer.BubbleLayer(dataSource, null, {
    radius: 5,
    strokeColor: "#4288f7",
    strokeWidth: 6, 
    color: "white" 
}));
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Источник данных BubbleLayer' src='//codepen.io/azuremaps/embed/mzqaKB/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Посмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/mzqaKB/'>Источник данных BubbleLayer</a> от Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="show-labels-with-a-bubble-layer"></a>Отображение меток с помощью слоя пузырьков

Этот код показывает, как использовать пузырьковый слой для отрисовки точки на карте. И, как использовать слой символов для отрисовки метки. Чтобы скрыть значок слоя символов, задайте `image` для свойства параметров значка значение `'none'` .

<br/>

<iframe height='500' scrolling='no' title='Источник данных MultiLayer' src='//codepen.io/azuremaps/embed/rqbQXy/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Посмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/rqbQXy/'>Источник данных MultiLayer</a> от Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-a-bubble-layer"></a>Настройка слоя пузырьков

Слой пузырьков имеет несколько параметров стиля, которые можно опробовать с помощью представленного ниже средства.

<br/>

<iframe height='700' scrolling='no' title='Параметры слоя пузырьков' src='//codepen.io/azuremaps/embed/eQxbGm/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/eQxbGm/'>Параметры слоя пузырьков</a> от Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [BubbleLayer](/javascript/api/azure-maps-control/atlas.layer.bubblelayer)

> [!div class="nextstepaction"]
> [BubbleLayerOptions](/javascript/api/azure-maps-control/atlas.bubblelayeroptions)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Создание источника данных](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](map-add-pin.md)

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Примеры кода](/samples/browse/?products=azure-maps)
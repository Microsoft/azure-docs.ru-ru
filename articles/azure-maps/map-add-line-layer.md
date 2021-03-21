---
title: Добавление слоя линий на карту | Microsoft Azure Maps
description: Узнайте, как добавлять линии в карты. См. примеры использования веб-пакета SDK Azure Maps для добавления слоев линий к картам и настройки линий с символами и градиентами цвета.
author: rbrundritt
ms.author: richbrun
ms.date: 08/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: fedf1bc37661100404106859d3aef52b3eddf06d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102048143"
---
# <a name="add-a-line-layer-to-the-map"></a>Добавление на карту слоя линий

Слой линий можно использовать для отображения компонентов `LineString` и `MultiLineString` в виде путей или маршрутов на карте. Слой линий также можно использовать для отображения контура компонентов `Polygon` и `MultiPolygon`. Источник данных подключен к слою линий, чтобы предоставить ему данные для подготовки к просмотру. 

> [!TIP]
> Слои линий по умолчанию отображают координаты многоугольников, а также линий в источнике данных. Чтобы ограничить слой таким образом, чтобы он отображал только компоненты LineString, задайте для свойства `filter` слоя значение `['==', ['geometry-type'], 'LineString']` или `['any', ['==', ['geometry-type'], 'LineString'], ['==', ['geometry-type'], 'MultiLineString']]`, если требуется также включить функции MultiLineString.

Ниже приведен код, создающий линию. Добавьте строку в источник данных, а затем отрисуйте ее с помощью слоя линий, используя класс [LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer).

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a line and add it to the data source.
dataSource.add(new atlas.data.LineString([[-73.972340, 40.743270], [-74.004420, 40.756800]]));
  
//Create a line layer to render the line to the map.
map.layers.add(new atlas.layer.LineLayer(dataSource, null, {
    strokeColor: 'blue',
    strokeWidth: 5
}));
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Добавление линии на карту' src='//codepen.io/azuremaps/embed/qomaKv/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/qomaKv/'>Добавление линии на карту</a> службы "Карты Azure" (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

Задать стиль слоев линий можно с помощью [LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions) и раздела [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md).

## <a name="add-symbols-along-a-line"></a>Добавление символов вдоль линии

В этом примере показано, как добавить значки стрелок вдоль линии на карте. При использовании слоя символов задайте для параметра "размещение" значение "линия". Этот параметр позволяет отображать символы вдоль линии и вращать значки (0 градусов = вправо).

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Отображение стрелки вдоль линии" src="//codepen.io/azuremaps/embed/drBJwX/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Просмотрите фрагмент кода для <a href='https://codepen.io/azuremaps/pen/drBJwX/'>отображения стрелки вдоль линии</a> в Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Веб-пакет SDK Azure Maps предоставляет несколько настраиваемых шаблонов изображений, которые можно использовать со слоем символов. Дополнительные сведения см. в статье [Использование шаблонов изображений](how-to-use-image-templates-web-sdk.md).

<a name="line-stroke-gradient"></a>

## <a name="add-a-stroke-gradient-to-a-line"></a>Добавление градиента штрихов на линию

К линии можно применить один цвет штриха. Можно также заполнить линию градиентом цветов, чтобы отобразить переход от одного сегмента к другому. Например, градиенты линий можно использовать для представления изменений по времени и расстоянию, а также различных температур в связанных линиях объектов. Чтобы применить эту функцию к строке, источник данных должен иметь `lineMetrics` параметр со значением `true` , а затем выражение цвета градиента может быть передано в `strokeColor` параметр строки. Выражение градиента контура должно ссылаться на выражение данных `['line-progress']`, которое предоставляет вычисляемые показатели линий выражению.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Линия с градиентом контура" src="//codepen.io/azuremaps/embed/wZwWJZ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
См. фрагмент кода <a href='https://codepen.io/azuremaps/pen/wZwWJZ/'>Линия с градиентом контура</a> в Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-a-line-layer"></a>Настройка слоя линий

Слой линий имеет несколько параметров стиля, которые можно опробовать с помощью представленного ниже средства.

<br/>

<iframe height='700' scrolling='no' title='Параметры слоя линий' src='//codepen.io/azuremaps/embed/GwLrgb/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода с <a href='https://codepen.io/azuremaps/pen/GwLrgb/'>параметрами слоя линий</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer) 

> [!div class="nextstepaction"]
> [LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Создание источника данных](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Добавление всплывающего окна](map-add-popup.md)

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Использование шаблонов изображений](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](map-add-shape.md)
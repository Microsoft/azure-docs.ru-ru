---
title: Добавление слоя многоугольников на карту | Карты Microsoft Azure
description: Узнайте, как добавлять к картам многоугольники или круги. Сведения об использовании веб-пакета SDK Azure Maps для настройки геометрических фигур и упрощении их обновления и сопровождения.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 96553e75d4b982cfe67d03961d4356a3844f253c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92891003"
---
# <a name="add-a-polygon-layer-to-the-map"></a>Добавление слоя многоугольников на карту

В этой статье показано, как визуализировать области геометрии компонентов `Polygon` и `MultiPolygon` на карте с помощью слоя многоугольников. Веб-пакет SDK Azure Maps также поддерживает создание геометрических объектов, как определено в [расширенной схеме GeoJSON](extend-geojson.md#circle). Эти круги преобразуются в многоугольники при отображении на карте. Все геометрии компонентов можно легко обновить при помощи класса [atlas.Shape](/javascript/api/azure-maps-control/atlas.shape).

## <a name="use-a-polygon-layer"></a>Использование слоя многоугольников 

Если слой многоугольников подключен к источнику данных и загружен на карту, он отображает область с компонентами `Polygon` и `MultiPolygon`. Чтобы создать многоугольник, добавьте его в источник данных и выводите на слой многоугольников с помощью класса [PolygonLayer](/javascript/api/azure-maps-control/atlas.layer.polygonlayer).

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a rectangular polygon.
dataSource.add(new atlas.data.Feature(
    new atlas.data.Polygon([[
        [-73.98235, 40.76799],
        [-73.95785, 40.80044],
        [-73.94928, 40.7968],
        [-73.97317, 40.76437],
        [-73.98235, 40.76799]
    ]])
));

//Create and add a polygon layer to render the polygon to the map, below the label layer.
map.layers.add(new atlas.layer.PolygonLayer(dataSource, null,{
    fillColor: 'red',
    fillOpacity: 0.7
}), 'labels');
```

Ниже приведен полный и выполняемый пример отображающегося выше кода.

<br/>

<iframe height='500' scrolling='no' title='Добавление многоугольника на карту ' src='//codepen.io/azuremaps/embed/yKbOvZ/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/yKbOvZ/'>Добавление многоугольника на карту</a> службы "Карты Azure" (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="use-a-polygon-and-line-layer-together"></a>Использование многоугольников и слоев линий вместе

Контуры многоугольников можно визуализировать с помощью слоев линий. Следующий пример кода визуализирует многоугольник, как в предыдущем примере, но теперь добавляет слой линии. Этот слой линий является вторым слоем, подключенным к источнику данных.  

<br/>

<iframe height='500' scrolling='no' title='Слои многоугольников и линий для добавления многоугольника' src='//codepen.io/azuremaps/embed/aRyEPy/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. фрагмент кода для <a href='https://codepen.io/azuremaps/pen/aRyEPy/'>добавления многоугольников с помощью слоев многоугольников и линий</a> от разработчиков Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="fill-a-polygon-with-a-pattern"></a>Заливка многоугольника с помощью узора

Кроме заливки многоугольника цветом, можно использовать изображение узора для заливки многоугольника. Загрузите шаблон изображения в ресурсы спрайтов изображения карт, а затем сошлитесь на это изображение со свойством `fillPattern` слоя полигонов.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Шаблон заливки многоугольника" src="//codepen.io/azuremaps/embed/JzQpYX/?height=500&theme-id=0&default-tab=js,result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. фрагмент кода <a href='https://codepen.io/azuremaps/pen/JzQpYX/'>Шаблон заливки многоугольника</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>


> [!TIP]
> Веб-пакет SDK Azure Maps предоставляет несколько настраиваемых шаблонов изображений, которые можно использовать в качестве шаблонов заливки. Дополнительные сведения см. в статье [Использование шаблонов изображений](how-to-use-image-templates-web-sdk.md).

## <a name="customize-a-polygon-layer"></a>Настройка слоя многоугольников

Слой прямоугольников имеет несколько параметров стиля, которые можно опробовать с помощью представленного ниже средства.

<br/>

<iframe height='700' scrolling='no' title='LXvxpg' src='//codepen.io/azuremaps/embed/LXvxpg/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/LXvxpg/'>LXvxpgL</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

<a id="addACircle"></a>

## <a name="add-a-circle-to-the-map"></a>Добавление круга на карту

Azure Maps использует расширенную версию схемы GeoJSON, которая предоставляет определение для кругов, как отмечалось [здесь](extend-geojson.md#circle). Круг отображается на карте путем создания компонента `Point`. Этот `Point` имеет свойство `subType` со значением `"Circle"` и свойством `radius` с числом, представляющим радиус в метрах. 

```javascript
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.126986, 47.639754]
    },
    "properties": {
        "subType": "Circle",
        "radius": 100
    }
}  
```

Веб-пакет SDK Azure Maps преобразует эти компоненты `Point` в функции `Polygon`. Затем эти компоненты отображаются на карте с использованием многоугольников и слоев линий, как показано в следующем примере кода.

<br/>

<iframe height='500' scrolling='no' title='Добавление круга на карту' src='//codepen.io/azuremaps/embed/PRmzJX/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/PRmzJX/'>Добавление круга на карту</a> службы "Карты Azure" (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="make-a-geometry-easy-to-update"></a>Упрощение обновления геометрии

Класс `Shape` позволяет создать оболочку для [Геометрии](/javascript/api/azure-maps-control/atlas.data.geometry) или [Компонента](/javascript/api/azure-maps-control/atlas.data.feature) и упрощает их обновление и обслуживание. Чтобы создать экземпляр переменной фигуры, передайте геометрию или набор свойств в конструктор фигур.

```javascript
//Creating a shape by passing in a geometry and a object containing properties.
var shape1 = new atlas.Shape(new atlas.data.Point[0,0], { myProperty: 1 });

//Creating a shape using a feature.
var shape2 = new atlas.Shape(new atlas.data.Feature(new atlas.data.Point[0,0], { myProperty: 1 });
```

В следующем образце кода показано, как заключить объект круга GeoJSON в класс фигур. При изменении значения радиуса в фигуре круг автоматически отображается на карте.

<br/>

<iframe height='500' scrolling='no' title='Обновление свойств фигуры' src='//codepen.io/azuremaps/embed/ZqMeQY/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. фрагмент кода для <a href='https://codepen.io/azuremaps/pen/ZqMeQY/'>обновления свойств фигуры</a> от разработчиков Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Многоугольник](/javascript/api/azure-maps-control/atlas.data.polygon)

> [!div class="nextstepaction"]
> [PolygonLayer](/javascript/api/azure-maps-control/atlas.layer.polygonlayer)

> [!div class="nextstepaction"]
> [PolygonLayerOptions](/javascript/api/azure-maps-control/atlas.polygonlayeroptions)

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
> [Добавление слоя линий](map-add-line-layer.md)

Дополнительные ресурсы:

> [!div class="nextstepaction"]
> [Расширение спецификации GeoJSON Azure Maps](extend-geojson.md#circle)
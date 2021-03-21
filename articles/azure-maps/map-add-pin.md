---
title: Добавление слоя символов к карте | Карты Microsoft Azure
description: Узнайте, как добавлять в карты настраиваемые символы, например текст или значки. Дополнительные сведения об использовании источников данных и слоев символов в Azure Maps веб-пакете SDK для этой цели.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: c5434406af1f912c1c89123baa344dd3f9c7ff91
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92891076"
---
# <a name="add-a-symbol-layer-to-a-map"></a>Добавление слоя символов на карту

Соедините символ с источником данных и используйте его для отображения значка или текста в заданной точке. 

Слои символов преобразовываются для просмотра с помощью WebGL. Используйте слой символов для визуализации больших коллекций точек на карте. По сравнению с HTML-маркером, слой символов отображает большое количество данных точек на карте с более высокой производительностью. Однако слой символов не поддерживает традиционные элементы CSS и HTML для стилей.  

> [!TIP]
> Слои символов по умолчанию отображают координаты всех геометрических объектов в источнике данных. Чтобы ограничить слой таким образом, чтобы он отображал только возможности геометрических точек `filter` , задайте для свойства слоя значение `['==', ['geometry-type'], 'Point']` или `['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']]` , если требуется, можно включить также функции MultiPoint.

Диспетчер спрайтов Maps загружает пользовательские изображения, используемые на уровне символов. Он поддерживает следующие форматы изображений:

- JPEG
- PNG
- SVG
- BMP
- GIF (без анимации)

## <a name="add-a-symbol-layer"></a>Добавление слоя символов

Прежде чем добавить на карту слой символов, необходимо выполнить несколько действий. Сначала создайте источник данных и добавьте его на карту. Создание слоя символов. Затем передайте источник данных на слой символов, чтобы получить данные из источника данных. Наконец, добавьте данные в источник данных, чтобы отобразить что-либо. 

В приведенном ниже коде показано, что следует добавить к карте после ее загрузки. Этот пример отображает единую точку на карте с помощью слоя символов. 

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a symbol layer to render icons and/or text at points on the map.
var layer = new atlas.layer.SymbolLayer(dataSource);

//Add the layer to the map.
map.layers.add(layer);

//Create a point and add it to the data source.
dataSource.add(new atlas.data.Point([0, 0]));
```

Существует четыре различных типа данных точек, которые можно добавить на карту:

- Геометрическая точка в формате JSON. Этот объект содержит только координаты точки и ничего другого. `atlas.data.Point`Вспомогательный класс можно использовать для простого создания этих объектов.
- Геометрическая геометрия MultiPoint — этот объект содержит координаты нескольких точек и ничего другого. `atlas.data.MultiPoint`Вспомогательный класс можно использовать для простого создания этих объектов.
- Функция геоjson — этот объект состоит из любой геометрической фигуры и набора свойств, содержащих метаданные, связанные с геометрическим объектом. `atlas.data.Feature`Вспомогательный класс можно использовать для простого создания этих объектов.
- `atlas.Shape` класс похож на функцию геоjson. Оба состоят из геометрической геометрии и набора свойств, содержащих метаданные, связанные с геометрическим объектом. Если объект геоjson добавляется в источник данных, его можно легко отобразить в слое. Однако при обновлении свойства координаты этого объекта геоjson источник данных и схема не изменяются. Это обусловлено тем, что в объекте JSON нет механизма для активации обновления. Класс Shape предоставляет функции для обновления содержащихся в ней данных. При внесении изменений источник данных и схема автоматически уведомляются и обновляются. 

В следующем примере кода создается геометрическая точка в формате JSON, которая передается в `atlas.Shape` класс, чтобы упростить обновление. Центр схемы изначально используется для отрисовки символа. На карту добавляется событие щелчка, чтобы при его срабатывании координаты мыши использовались с функцией Shapes `setCoordinates` . Координаты мыши записываются во время события щелчка. Затем `setCoordinates` обновляет расположение символа на карте.

<br/>

<iframe height='500' scrolling='no' title='Переключение расположения маркера' src='//codepen.io/azuremaps/embed/ZqJjRP/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/ZqJjRP/'>Переключение расположения маркера</a> от Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> По умолчанию слои символов оптимизируют отрисовку символов путем скрытия перекрывающихся символов. При изменении масштаба скрытые символы становятся видимыми. Чтобы отключить эту функцию и вывести все символы в любое время, задайте `allowOverlap` для свойства параметра значение `iconOptions` `true` .

## <a name="add-a-custom-icon-to-a-symbol-layer"></a>Добавление пользовательского значка в слой символов

Слои символов преобразовываются для просмотра с помощью WebGL. Таким образом, все ресурсы (например, образы значков) необходимо загрузить в контекст WebGL. В этом примере показано, как добавить пользовательский значок в ресурсы Map. Этот значок используется для отрисовки данных точек с помощью пользовательского символа на карте. Свойство `textField` слоя символа требует указания выражения. В этом случае нам нужно отобразить свойство температуры. Так как температура является числом, ее необходимо преобразовать в строку. Кроме того, мы хотим добавить к нему «° F». Для этого объединения можно использовать выражение. `['concat', ['to-string', ['get', 'temperature']], '°F']`. 

<br/>

<iframe height='500' scrolling='no' title='Значок изображения пользовательских символов' src='//codepen.io/azuremaps/embed/WYWRWZ/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. ручку <a href='https://codepen.io/azuremaps/pen/WYWRWZ/'>Custom Symbol Image Icon</a> (Значок изображения пользовательских символов) от Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Веб-пакет SDK Azure Maps предоставляет несколько настраиваемых шаблонов изображений, которые можно использовать со слоем символов. Дополнительные сведения см. в статье [Использование шаблонов изображений](how-to-use-image-templates-web-sdk.md).

## <a name="customize-a-symbol-layer"></a>Настройка слоя символа 

Слой символа имеет множество доступных вариантов стилизации. Вот средство для проверки этих различных параметров.

<br/>

<iframe height='700' scrolling='no' title='Параметры слоя символов' src='//codepen.io/azuremaps/embed/PxVXje/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите ручку <a href='https://codepen.io/azuremaps/pen/PxVXje/'>Symbol Layer Options</a> (Параметры слоя символов) от Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Если требуется отобразить только текст с помощью слоя символов, можно скрыть значок, задав `image` для свойства параметров значка значение `'none'` .

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [SymbolLayer class](/javascript/api/azure-maps-control/atlas.layer.symbollayer) (Класс SymbolLayer)

> [!div class="nextstepaction"]
> [SymbolLayerOptions interface](/javascript/api/azure-maps-control/atlas.symbollayeroptions) (Интерфейс SymbolLayerOptions)

> [!div class="nextstepaction"]
> [IconOptions interface](/javascript/api/azure-maps-control/atlas.iconoptions) (Интерфейс IconOptions)

> [!div class="nextstepaction"]
> [текстоптионс](/javascript/api/azure-maps-control/atlas.textoptions)

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

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](map-add-shape.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков на карту](map-add-bubble-layer.md)
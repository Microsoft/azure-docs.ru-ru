---
title: Добавление элементов управления в карту | Карты Microsoft Azure
description: Добавление элемента управления "Масштаб", элемента управления "высота", элемента управления "поворот" и выбора стиля на карту в Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-js
ms.openlocfilehash: bbcfe4935843a00bc5274273e79f4ffc72dc25ed
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102051061"
---
# <a name="add-controls-to-a-map"></a>Добавление элементов управления на карту

В этой статье показано, как добавить элементы управления на карту. Вы также узнаете, как создать карту со всеми элементами управления и [выбором стиля](./choose-map-style.md).

## <a name="add-zoom-control"></a>Добавление элемента управления масштабом

Элемент управления Zoom добавляет кнопки для масштабирования и отображения карт. В следующем примере кода создается экземпляр класса [зумконтрол](/javascript/api/azure-maps-control/atlas.control.zoomcontrol) , который добавляется в правый нижний угол на карте.

```javascript
//Construct a zoom control and add it to the map.
map.controls.add(new atlas.control.ZoomControl(), {
    position: 'bottom-right'
});
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Добавление элемента управления масштабом' src='//codepen.io/azuremaps/embed/WKOQyN/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/WKOQyN/'>Добавление элемента управления масштабом</a> для службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-pitch-control"></a>Добавление элемента управления наклоном

Элемент управления "тон" добавляет кнопки для наклона высоты для отображения относительно горизонта. В следующем примере кода создается экземпляр класса [питчконтрол](/javascript/api/azure-maps-control/atlas.control.pitchcontrol) . Он добавляет Питчконтрол в правый верхний угол на карте.

```javascript
//Construct a pitch control and add it to the map.
map.controls.add(new atlas.control.PitchControl(), {
    position: 'top-right'
});
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Добавление элемента управления наклоном' src='//codepen.io/azuremaps/embed/xJrwaP/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/xJrwaP/'>Добавление элемента управления наклоном</a> в службе Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-compass-control"></a>Добавление элемента управления расстоянием

Элемент управления компаса добавляет кнопку для поворота схемы. В следующем примере кода создается экземпляр класса [компассконтрол](/javascript/api/azure-maps-control/atlas.control.compasscontrol) , который добавляется в левый нижний угол на карте.

```javascript
//Construct a compass control and add it to the map.
map.controls.add(new atlas.control.CompassControl(), {
    position: 'bottom-left'
});
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Добавление элемента управления поворотом' src='//codepen.io/azuremaps/embed/GBEoRb/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/GBEoRb/'>Добавление элемента управления поворотом</a> в службе Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="a-map-with-all-controls"></a>Карта со всеми элементами управления

Несколько элементов управления могут быть помещены в массив и добавлены в карту сразу и расположены в одной области на карте, чтобы упростить разработку. Следующий пример добавляет стандартные элементы управления навигацией в карту с помощью этого подхода.

```javascript
map.controls.add([
    new atlas.control.ZoomControl(),
    new atlas.control.CompassControl(),
    new atlas.control.PitchControl(),
    new atlas.control.StyleControl()
], {
    position: "top-right"
});
```

В следующем примере кода добавляются элементы управления "Масштаб", "Компас", "высота" и "Выбор стиля" в правый верхний угол сопоставлений. Обратите внимание, как они автоматически располагаются в стеке. Порядок выполнения элементов управления из сценария определяет их порядок появления на карте. Чтобы изменить порядок элементов управления на карте, можно изменить их порядок в массиве.

<br/>

<iframe height='500' scrolling='no' title='Карта со всеми элементами управления' src='//codepen.io/azuremaps/embed/qyjbOM/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/qyjbOM/'>Карта со всеми элементами управления </a> в службе Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

Элемент управления выбора стиля определяется классом [стилеконтрол](/javascript/api/azure-maps-control/atlas.control.stylecontrol) . Дополнительные сведения об использовании элемента управления "Выбор стиля" см. в разделе [Выбор стиля схемы](choose-map-style.md).

## <a name="customize-controls"></a>Настройка элементов управления

Ниже приведено средство для проверки различных параметров настройки элементов управления.

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Параметры элемента управления навигацией" src="//codepen.io/azuremaps/embed/LwBZMx/?height=700&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/LwBZMx/'>Параметры навигации по управлению</a> пером с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

Если необходимо создать настраиваемые элементы управления навигацией, создайте класс, который расширяется из `atlas.Control` класса, или Создайте HTML-элемент и поместите его над элементом div Map. Этот элемент управления пользовательского интерфейса вызывает `setCamera` функцию Maps для перемещения карты. 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Элемент управления компас](/javascript/api/azure-maps-control/atlas.control.compasscontrol)

> [!div class="nextstepaction"]
> [PitchControl](/javascript/api/azure-maps-control/atlas.control.pitchcontrol); 

> [!div class="nextstepaction"]
> [StyleControl](/javascript/api/azure-maps-control/atlas.control.stylecontrol). 

> [!div class="nextstepaction"]
> [ZoomControl](/javascript/api/azure-maps-control/atlas.control.zoomcontrol); 

Полные примеры кода см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление ПИН-кода](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Добавление всплывающего окна](./map-add-popup.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](map-add-shape.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer.md)
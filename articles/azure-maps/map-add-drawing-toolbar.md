---
title: Добавить панель инструментов средств рисования для отображения | Карты Microsoft Azure
description: Добавление панели инструментов рисования в карту с помощью веб-пакета SDK Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/04/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: b00628ec5a9f41b027bf90b93421f3aa1404e97a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92896401"
---
# <a name="add-a-drawing-tools-toolbar-to-a-map"></a>Добавление панели инструментов средств рисования на карту

В этой статье показано, как использовать модуль средств рисования и отображать панель инструментов рисования на карте. Элемент управления [дравингтулбар](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar) добавляет панель инструментов рисования на карте. Вы узнаете, как создавать карты с одним и всеми инструментами рисования, а также как настраивать отрисовку графических фигур в диспетчере рисунков.

## <a name="add-drawing-toolbar"></a>Добавление панели инструментов рисования

Следующий код создает экземпляр диспетчера рисунков и отображает панель инструментов на карте.

```javascript
//Create an instance of the drawing manager and display the drawing toolbar.
drawingManager = new atlas.drawing.DrawingManager(map, {
        toolbar: new atlas.control.DrawingToolbar({
            position: 'top-right',
            style: 'dark'
        })
    });
```

Ниже приведен полный пример выполнения кода функции, приведенной выше.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Добавление панели инструментов рисования" src="//codepen.io/azuremaps/embed/ZEzLeRg/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/ZEzLeRg/'>Добавление панели инструментов рисования</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="limit-displayed-toolbar-options"></a>Параметр "ограничить отображаемые параметры панели инструментов"

Следующий код создает экземпляр диспетчера рисунков и отображает панель инструментов, используя только инструмент рисования многоугольников на карте. 

```javascript
//Create an instance of the drawing manager and display the drawing toolbar with polygon drawing tool.
drawingManager = new atlas.drawing.DrawingManager(map, {
        toolbar: new atlas.control.DrawingToolbar({
            position: 'top-right',
            style: 'light',
            buttons: ["draw-polygon"]
        })
    });
```

Ниже приведен полный пример выполнения кода функции, приведенной выше.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Добавление инструмента рисования многоугольников" src="//codepen.io/azuremaps/embed/OJLWWMy/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Ознакомьтесь с пером <a href='https://codepen.io/azuremaps/pen/OJLWWMy/'>Добавление инструмента рисования многоугольника</a> Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="change-drawing-rendering-style"></a>Изменить стиль отрисовки изображения

Стиль рисуемых фигур можно настроить, извлекая нижележащие слои диспетчера рисунков с помощью `drawingManager.getLayers()` функции, а затем устанавливая параметры на отдельных слоях. Маркеры перетаскивания, которые отображаются для координат при редактировании фигуры, являются HTML-маркерами. Стиль маркеров перетаскивания можно настроить, передав параметры маркера HTML в `dragHandleStyle` `secondaryDragHandleStyle` Параметры и диспетчера рисунков.  

Следующий код получает слои подготовки отчетов из диспетчера рисунков и изменяет их параметры, чтобы изменить стиль отрисовки для рисования. В этом случае точки отображаются с синим значком маркера. Линии будут иметь красный и четыре пиксела в ширину. Многоугольники будут иметь зеленый цвет заливки и оранжевый контур. После этого стили маркеров перетаскивания меняются на квадратные значки. 

```javascript
//Get rendering layers of drawing manager.
var layers = drawingManager.getLayers();

//Change the icon rendered for points.
layers.pointLayer.setOptions({
    iconOptions: {
        image: 'marker-blue'
    }
});

//Change the color and width of lines.
layers.lineLayer.setOptions({
    strokeColor: 'red',
    strokeWidth: 4
});

//Change fill color of polygons.
layers.polygonLayer.setOptions({
    fillColor: 'green'
});

//Change the color of polygon outlines.
layers.polygonOutlineLayer.setOptions({
    strokeColor: 'orange'
});

//Update the style of the drag handles that appear when editting.
drawingManager.setOptions({
    //Primary drag handle that represents coordinates in the shape.
    dragHandleStyle: {
        anchor: 'center',
        htmlContent: '<svg width="15" height="15" viewBox="0 0 15 15" xmlns="http://www.w3.org/2000/svg" style="cursor:pointer"><rect x="0" y="0" width="15" height="15" style="stroke:black;fill:white;stroke-width:4px;"/></svg>',
        draggable: true
    },

    //Secondary drag hanle that represents mid-point coordinates that users can grab to add new cooridnates in the middle of segments.
    secondaryDragHandleStyle: {
        anchor: 'center',
        htmlContent: '<svg width="10" height="10" viewBox="0 0 10 10" xmlns="http://www.w3.org/2000/svg" style="cursor:pointer"><rect x="0" y="0" width="10" height="10" style="stroke:white;fill:black;stroke-width:4px;"/></svg>',
        draggable: true
    }
});  
```

Ниже приведен полный пример выполнения кода функции, приведенной выше.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Изменить стиль отрисовки изображения" src="//codepen.io/azuremaps/embed/OJLWpyj/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/OJLWpyj/'>изменение перьевого отображения рисунка</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как использовать дополнительные функции модуля инструментов рисования:

> [!div class="nextstepaction"]
> [Получение данных о фигуре](map-get-shape-data.md)

> [!div class="nextstepaction"]
> [Реакция на события рисования](drawing-tools-events.md)

> [!div class="nextstepaction"]
> [Типы взаимодействия и сочетания клавиш](drawing-tools-interactions-keyboard-shortcuts.md)

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Схема](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [Панель инструментов рисования](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)

> [!div class="nextstepaction"]
> [Диспетчер рисунков](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)
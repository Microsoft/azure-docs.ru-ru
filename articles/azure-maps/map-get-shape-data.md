---
title: Получение данных из фигур на карте | Карты Microsoft Azure
description: Из этой статьи вы узнаете, как получить данные фигуры на карте с помощью веб-пакета SDK Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/04/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: ddb8009e544ede82d1c56d112950ff247a87380c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92890706"
---
# <a name="get-shape-data"></a>Получение данных о фигуре

В этой статье показано, как получить данные фигур, рисуемых на карте. Мы используем функцию **дравингманажер.-Source ()** в [диспетчере рисунков](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager#getsource--). Существуют различные сценарии, когда необходимо извлечь данные геообъектно-JSON рисуемой фигуры и использовать их в других местах.  


## <a name="get-data-from-drawn-shape"></a>Получение данных из рисуемой фигуры

Следующая функция получает исходные данные рисуемой фигуры и выводит их на экран. 

```javascript
function getDrawnShapes() {
    var source = drawingManager.getSource();

    document.getElementById('CodeOutput').value = JSON.stringify(source.toJson(), null, '    ');
}
```

Ниже приведен полный пример выполнения кода, в котором можно нарисовать фигуру для проверки функциональности:

<br/>

<iframe height="686" title="Получение данных о фигуре" src="//codepen.io/azuremaps/embed/xxKgBVz/?height=265&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true" style='width: 100%;'>См. раздел <a href='https://codepen.io/azuremaps/pen/xxKgBVz/'>Получение данных формы</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как использовать дополнительные функции модуля инструментов рисования:

> [!div class="nextstepaction"]
> [Реакция на события рисования](drawing-tools-events.md)

> [!div class="nextstepaction"]
> [Типы взаимодействия и сочетания клавиш](drawing-tools-interactions-keyboard-shortcuts.md)

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Схема](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [Диспетчер рисунков](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)

> [!div class="nextstepaction"]
> [Панель инструментов рисования](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)
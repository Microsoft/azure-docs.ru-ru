---
title: События инструмента рисования | Карты Microsoft Azure
description: В этой статье вы узнаете, как добавить панель инструментов рисования на карту с помощью веб-пакета SDK для Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 12/05/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: 6abe0ed88adbdf8263aa27d340fb2fff156d98e6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90089349"
---
# <a name="drawing-tool-events"></a>События инструмента рисования

При использовании инструментов рисования на карте полезно реагировать на определенные события, когда пользователь рисует на карте. В этой таблице перечислены все события, поддерживаемые классом `DrawingManager`.

| Событие | Описание |
|-------|-------------|
| `drawingchanged` | Срабатывает при добавлении или изменении любой координаты в фигуре. | 
| `drawingchanging` | Срабатывает при отображении любой координаты предварительного просмотра для фигуры. Например, это событие будет срабатывать несколько раз при перетаскивании координат. | 
| `drawingcomplete` | Срабатывает после завершения рисования фигуры или выхода из режима редактирования. |
| `drawingmodechanged` | Срабатывает при изменении режима рисования. Новый режим рисования передается в обработчик событий. |
| `drawingstarted` | Возникает, когда пользователь начинает рисовать фигуру или помещает фигуру в режим правки.  |

В следующем коде показано, как работают события в модуле инструментов рисования. Нарисуйте фигуры на карте и понаблюдайте за срабатыванием соответствующих событий.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="События инструментов рисования" src="https://codepen.io/azuremaps/embed/dyPMRWo?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/dyPMRWo'>События инструментов рисования</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="examples"></a>Примеры

Рассмотрим некоторые распространенные сценарии, в которых используются события инструментов рисования.

### <a name="select-points-in-polygon-area"></a>Выбор точек в области многоугольника

Этот код демонстрирует, как отслеживать событие рисования фигур пользователем. В этом примере код отслеживает многоугольники, прямоугольники и круги. Затем он определяет, какие точки данных на карте находятся в рисуемой области. Событие `drawingcomplete` используется для активации логики выбора. В логике выбора код выполняет перебор всех точек данных на карте. Он проверяет, существует ли пересечение точки и области рисования фигуры. В этом примере для выполнения вычисления пространственного пересечения используется библиотека [Turf.js](https://turfjs.org/) с открытым исходным кодом.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Выбор данных в области рисования многоугольника" src="https://codepen.io/azuremaps/embed/XWJdeja?height=500&theme-id=default&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/XWJdeja'>Выбор данных в области рисования многоугольника</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

### <a name="draw-and-search-in-polygon-area"></a>Рисование и поиск в области многоугольника

Этот код выполняет поиск точек интереса в области фигуры после того, как пользователь завершил рисование фигуры. Чтобы изменить и выполнить код, нажмите кнопку Edit on CodePen (Изменить в CodePen) в правом верхнем углу. Событие `drawingcomplete` используется для активации логики поиска. Если пользователь рисует прямоугольник или многоугольник, выполняется поиск в геометрии. При рисовании окружности точки радиуса и центра используются для выполнения поиска точки интереса. Событие `drawingmodechanged` используется для определения того, когда пользователь переключается в режим рисования, и это событие очищает холст.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Рисование и поиск в области многоугольника" src="https://codepen.io/azuremaps/embed/eYmZGNv?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/eYmZGNv'>Рисование и поиск в области многоугольника</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

### <a name="create-a-measuring-tool"></a>Создание измерительного инструмента

В приведенном ниже коде показано, как можно использовать события рисования для создания измерительного инструмента. `drawingchanging` используется для отслеживания фигуры по мере ее рисования. По мере того как пользователь перемещает мышь, выполняется вычисление размеров фигуры. Событие `drawingcomplete` используется для выполнения окончательного вычисления фигуры после ее рисования. Событие `drawingmodechanged` используется, чтобы определить, когда пользователь переключается в режим рисования. Кроме того, событие `drawingmodechanged` очищает холст рисования и удаляет старую информацию об измерении.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Измерительный инструмент" src="https://codepen.io/azuremaps/embed/RwNaZXe?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/RwNaZXe'>Измерительный инструмент</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как использовать дополнительные функции модуля инструментов рисования:

> [!div class="nextstepaction"]
> [Получение данных о фигуре](map-get-shape-data.md)

> [!div class="nextstepaction"]
> [Типы взаимодействия и сочетания клавиш](drawing-tools-interactions-keyboard-shortcuts.md)

Подробнее о модуле "Службы":

> [!div class="nextstepaction"]
> [Модуль служб](how-to-use-services-module.md)

Изучите дополнительные примеры кода:

> [!div class="nextstepaction"]
> [Страница с примерами кода](https://aka.ms/AzureMapsSamples)

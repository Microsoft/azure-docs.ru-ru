---
title: Модуль средств рисования | Карты Microsoft Azure
description: В этой статье вы узнаете, как задать данные параметров рисования с помощью веб-пакета SDK для карт Microsoft Azure.
author: anastasia-ms
ms.author: v-stharr
ms.date: 01/29/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 95a04d763fa5982181cc1c797bce969d9857ae4b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92890638"
---
# <a name="use-the-drawing-tools-module"></a>Использование модуля средств рисования

Веб-пакет SDK Azure Maps предоставляет *модуль средств рисования*. Этот модуль упрощает Рисование и изменение фигур на карте с помощью устройства ввода, такого как мышь или сенсорный экран. Основным классом этого модуля является [Диспетчер рисунков](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager#setoptions-drawingmanageroptions-). Диспетчер рисунков предоставляет все возможности, необходимые для рисования и редактирования фигур на карте. Его можно использовать напрямую и интегрировать с настраиваемым пользовательским ИНТЕРФЕЙСом панели инструментов. Можно также использовать встроенный класс [панели инструментов рисования](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar) . 

## <a name="loading-the-drawing-tools-module-in-a-webpage"></a>Загрузка модуля средств рисования на веб-страницу

1. Создайте новый HTML-файл и [реализуйте карту как обычно](./how-to-use-map-control.md).
2. Загрузите модуль инструментов для рисования Azure Maps. Загрузить его можно одним из двух способов:
    - Используйте глобально размещенную версию сети доставки содержимого Azure модуля Azure Maps Services. Добавьте ссылку на таблицу стилей JavaScript и CSS в `<head>` элемент файла:

        ```html
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.js"></script>
        ```

    - Также можно загрузить модуль средств рисования для исходного кода веб-пакета SDK Azure Maps с помощью пакета [Azure-Maps-Drawing-Tools](https://www.npmjs.com/package/azure-maps-drawing-tools) NPM, а затем разместить его в приложении. Этот пакет также включает определения TypeScript. Используйте следующую команду:
    
        > **NPM. Установка Azure-Maps-Drawing-Tools**
    
        Затем добавьте ссылку на таблицу стилей JavaScript и CSS в `<head>` элемент файла:

         ```html
        <link rel="stylesheet" href="node_modules/azure-maps-drawing-tools/dist/atlas-drawing.min.css" type="text/css" />
        <script src="node_modules/azure-maps-drawing-tools/dist/atlas-drawing.min.js"></script>
         ```

## <a name="use-the-drawing-manager-directly"></a>Использование диспетчера рисунков напрямую

После загрузки модуля инструменты рисования в приложение можно включить возможности рисования и редактирования с помощью [диспетчера рисунков](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager#setoptions-drawingmanageroptions-). Можно указать параметры для диспетчера рисунков при его создании или использовать `drawingManager.setOptions()` функцию.

### <a name="set-the-drawing-mode"></a>Установка режима рисования

Следующий код создает экземпляр диспетчера рисунков и устанавливает параметр **режима** рисования. 

```javascript
//Create an instance of the drawing manager and set drawing mode.
drawingManager = new atlas.drawing.DrawingManager(map,{
    mode: "draw-polygon"
});
```

Приведенный ниже код является полным запуском примера того, как настроить режим рисования диспетчера рисунков. Щелкните карту, чтобы начать рисование многоугольника.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Рисование многоугольника" src="//codepen.io/azuremaps/embed/YzKVKRa/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
См. перо <a href='https://codepen.io/azuremaps/pen/YzKVKRa/'>Рисование многоугольника</a> на Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>


### <a name="set-the-interaction-type"></a>Установка типа взаимодействия

Диспетчер рисунков поддерживает три разных способа взаимодействия с картой для рисования фигур.

* `click` — Координаты добавляются при щелчке мышью или касанием.
* `freehand ` — Координаты добавляются при перетаскивании указателя мыши или касания на карту. 
* `hybrid` — Координаты добавляются при нажатии или перетаскивании мыши или касания.

Следующий код включает режим рисования многоугольника и задает тип взаимодействия рисования, которым должен соответствовать Диспетчер рисунков `freehand` . 

```javascript
//Create an instance of the drawing manager and set drawing mode.
drawingManager = new atlas.drawing.DrawingManager(map,{
    mode: "draw-polygon",
    interactionType: "freehand"
});
```

 Этот пример кода реализует функции рисования многоугольника на карте. Просто удерживайте левую кнопку мыши и перетащите ее вокруг, свободно.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Произвольная прорисовка" src="//codepen.io/azuremaps/embed/ZEzKoaj/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
См. <a href='https://codepen.io/azuremaps/pen/ZEzKoaj/'>рисунок с изображением без</a> пера, Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>


### <a name="customizing-drawing-options"></a>Настройка параметров рисования

В предыдущих примерах было показано, как настроить параметры рисования при создании экземпляра диспетчера рисунков. Параметры диспетчера рисунков также можно задать с помощью `drawingManager.setOptions()` функции. Ниже приведен инструмент для проверки настройки всех параметров диспетчера рисунков с помощью функции Сетоптионс.

<br/>

<iframe height="685" title="Настройка диспетчера рисунков" src="//codepen.io/azuremaps/embed/LYPyrxR/?height=600&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true" style='width: 100%;'>См. раздел <a href='https://codepen.io/azuremaps/pen/LYPyrxR/'>Получение данных формы</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как использовать дополнительные функции модуля инструментов рисования:

> [!div class="nextstepaction"]
> [Добавление панели средств рисования](map-add-drawing-toolbar.md)

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
> [Диспетчер рисунков](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)

> [!div class="nextstepaction"]
> [Панель инструментов рисования](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)
---
title: Добавление маркера HTML на карту | Microsoft Azure Maps
description: Узнайте, как добавлять к картам маркеры HTML. Узнайте, как использовать веб-пакет SDK Azure Maps для настройки маркеров и добавления всплывающих окон и событий мыши в маркер.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 1c4367e2a649f4e239e2dab374afc4fb867e517b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92891199"
---
# <a name="add-html-markers-to-the-map"></a>Добавление маркеров HTML на карту

В этой статье показано, как добавить на карту настраиваемый элемент HTML, например файл изображения, в качестве маркера HTML.

> [!NOTE]
> Маркеры HTML не подключаются к источникам данных. Вместо этого сведения о положении добавляются непосредственно к маркеру, который добавляется в свойство карт `markers` — [HtmlMarkerManager](/javascript/api/azure-maps-control/atlas.htmlmarkermanager).

> [!IMPORTANT]
> В отличие от большинства слоев в веб-элементе управления Azure Maps, использующих WebGL для отрисовки, маркеры HTML используют традиционные элементы модели DOM для отрисовки. Таким образом, чем больше дополнительных маркеров HTML добавлено на страницу, тем больше элементов модели DOM. Производительность может снизиться после добавления нескольких сотен маркеров HTML. Для более крупных наборов данных рекомендуется кластеризация данных или использование слоя символа или пузырьков.

## <a name="add-an-html-marker"></a>Добавление маркера HTML

Класс [HtmlMarker](/javascript/api/azure-maps-control/atlas.htmlmarker) имеет стиль по умолчанию. Маркер можно настроить, задав параметры цвета и текста маркера. Стиль по умолчанию класса маркера HTML — это шаблон SVG с заполнителем `{color}` и `{text}`. Задайте свойства цвета и текста в параметрах маркера HTML для быстрой настройки. 

Следующий код создает маркер HTML и присваивает свойству color значение "DodgerBlue", а свойству text — значение 10. К маркеру прикрепляется всплывающее окно, а для переключения видимости всплывающего окна используется событие `click`.

```javascript
//Create an HTML marker and add it to the map.
var marker = new atlas.HtmlMarker({
    color: 'DodgerBlue',
    text: '10',
    position: [0, 0],
    popup: new atlas.Popup({
        content: '<div style="padding:10px">Hello World</div>',
        pixelOffset: [0, -30]
    })
});

map.markers.add(marker);

//Add a click event to toggle the popup.
map.events.add('click',marker, () => {
    marker.togglePopup();
});
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Добавление на карту маркера HTML' src='//codepen.io/azuremaps/embed/MVoeVw/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/MVoeVw/'>Добавление маркера HTML на карту</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="create-svg-templated-html-marker"></a>Создание маркера HTML шаблона SVG

Значение по умолчанию `htmlContent` маркера HTML — это шаблон SVG с папками места `{color}` и `{text}`. Вы можете создавать настраиваемые строки SVG и добавлять эти примеры заполнителей в свои SVG, чтобы при настройке параметров `color` и `text` маркера заполнители обновлялись в SVG.

<br/>

<iframe height='500' scrolling='no' title='Маркер HTML с пользовательским шаблоном SVG' src='//codepen.io/azuremaps/embed/LXqMWx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/LXqMWx/'>Маркер HTML с пользовательским шаблоном SVG</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Веб-пакет SDK Azure Maps предоставляет несколько шаблонов изображений SVG, которые можно использовать с маркерами HTML. Дополнительные сведения см. в статье [Использование шаблонов изображений](how-to-use-image-templates-web-sdk.md).

## <a name="add-a-css-styled-html-marker"></a>Добавление маркера HTML стиля CSS

Одно из преимуществ HTML заключается в том, что имеется множество отличных настроек, которые можно сделать с помощью CSS. В этом примере содержимое HtmlMarker состоит из HTML и CSS, которые создают анимированное закрепление, фиксируемое на месте и действующее динамически.

<br/>

<iframe height='500' scrolling='no' title='Источник данных HTML' src='//codepen.io/azuremaps/embed/qJVgMx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Посмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/qJVgMx/'>Источник данных HTML</a><a href='https://codepen.io/azuremaps'>от Azure Maps (@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="draggable-html-markers"></a>Перетаскиваемые маркеры HTML

В этом примере показано, как сделать перетаскиваемый маркер HTML. Маркеры HTML поддерживают события `drag`, `dragstart` и `dragend`.

<br/>

<iframe height='500' scrolling='no' title='Перетаскиваемый маркер HTML' src='//codepen.io/azuremaps/embed/wQZoEV/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/wQZoEV/'>Перетаскиваемый маркер HTML</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-mouse-events-to-html-markers"></a>Добавление событий мыши к маркерам HTML

Эти примеры демонстрируют добавление событий мыши и перетаскивания к маркеру HTML.

<br/>

<iframe height='500' scrolling='no' title='Добавление событий мыши к маркерам HTML' src='//codepen.io/azuremaps/embed/RqOKRz/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/RqOKRz/'>Добавление событий мыши к маркерам HTML</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [HtmlMarker](/javascript/api/azure-maps-control/atlas.htmlmarker)

> [!div class="nextstepaction"]
> [HtmlMarkerOptions](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

> [!div class="nextstepaction"]
> [HtmlMarkerManager](/javascript/api/azure-maps-control/atlas.htmlmarkermanager)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Использование шаблонов изображений](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](./map-add-bubble-layer.md)
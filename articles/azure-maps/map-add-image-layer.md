---
title: Добавление слоя изображения к карте | Карты Microsoft Azure
description: Узнайте, как добавлять изображения на карту. Узнайте, как использовать веб-пакет SDK Azure Maps для настройки слоев изображения и наложения изображений по фиксированным наборам координат.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: f0c24940e606020dc45e5a000cfcb030fd806d4b
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102047491"
---
# <a name="add-an-image-layer-to-a-map"></a>Добавление слоя изображений на карту

В этой статье показано, как наложить изображение на фиксированный набор координат. Ниже приведено несколько примеров различных типов изображений, которые могут быть наложены на карты:

* Образы, полученные из дроны
* Создание флурпланс
* Исторические или другие специализированные изображения карт
* Схемы сайтов заданий
* Лепестковые изображения погоды

> [!TIP]
> [Имажелайер](/javascript/api/azure-maps-control/atlas.layer.imagelayer) — это простой способ наложения изображения на карту. Обратите внимание, что обозреватели могут испытывать трудности при загрузке большого изображения. В этом случае рассмотрите возможность разбить изображение на плитки и загрузить их на карту как [тилелайер](/javascript/api/azure-maps-control/atlas.layer.tilelayer).

Слой изображений поддерживает следующие форматы изображений:

- JPEG
- PNG
- BMP
- GIF (без анимации)

## <a name="add-an-image-layer"></a>Добавление слоя изображений

Следующий код накладывает изображение на карту [Ньюарке, Нью-Джерси, с 1922](https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg) на карте. [Имажелайер](/javascript/api/azure-maps-control/atlas.layer.imagelayer) создается путем передачи URL-адреса изображению и координат для четырех углов в формате `[Top Left Corner, Top Right Corner, Bottom Right Corner, Bottom Left Corner]` .

```javascript
//Create an image layer and add it to the map.
map.layers.add(new atlas.layer.ImageLayer({
    url: 'newark_nj_1922.jpg',
    coordinates: [
        [-74.22655, 40.773941], //Top Left Corner
        [-74.12544, 40.773941], //Top Right Corner
        [-74.12544, 40.712216], //Bottom Right Corner
        [-74.22655, 40.712216]  //Bottom Left Corner
    ]
}));
```

Ниже приведен полный пример выполнения кода предыдущего кода.

<br/>

<iframe height='500' scrolling='no' title='Простой слой изображений' src='//codepen.io/azuremaps/embed/eQodRo/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/eQodRo/'>простого слоя карты</a>, опубликованный для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в репозитории <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="import-a-kml-file-as-ground-overlay"></a>Импортировать файл КМЛ как наложение заземления

В этом образце показано, как добавить сведения о наложении КМЛ на землю в виде слоя изображения на карте. Наложение заземления КМЛ предоставляет координаты Севера, Юг, Восток и Запад и поворот по часовой стрелке. Но уровень изображения ожидает координаты для каждого угла изображения. Наложение заземления КМЛ в этом примере предназначено для Чартрес Каседрал, а источник — из [некоммерческого](https://commons.wikimedia.org/wiki/File:Chartres.svg/overlay.kml).

В коде используется статическая `getCoordinatesFromEdges` функция из класса [имажелайер](/javascript/api/azure-maps-control/atlas.layer.imagelayer) . Он вычисляет четыре угла изображения, используя сведения о севере, Южной, Восточной, Западной и повороте наложения заземления КМЛ.

<br/>

<iframe height='500' scrolling='no' title='Наземное наложение KML как слоя изображений' src='//codepen.io/azuremaps/embed/EOJgpj/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/EOJgpj/'>наземное наложение KML как слоя изображений</a>, опубликованный для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в репозитории <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Используйте `getPixels` функции и `getPositions` класса Image Layer для преобразования между географическими координатами позиционированного слоя изображения и координатами пикселей локального изображения.

## <a name="customize-an-image-layer"></a>Настройка слоя изображений

На уровне изображения имеется много параметров стилей. Вот средство для пробных попыток.

<br/>

<iframe height='700' scrolling='no' title='Параметры слоя изображений' src='//codepen.io/azuremaps/embed/RqOGzx/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода с <a href='https://codepen.io/azuremaps/pen/RqOGzx/'>параметрами слоя изображений</a>, опубликованный для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Класс ImageLayer](/javascript/api/azure-maps-control/atlas.layer.imagelayer)

> [!div class="nextstepaction"]
> [Интерфейс ImageLayerOptions](/javascript/api/azure-maps-control/atlas.imagelayeroptions)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление слоя фрагментов](./map-add-tile-layer.md)
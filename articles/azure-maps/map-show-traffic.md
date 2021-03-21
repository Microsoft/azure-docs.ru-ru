---
title: Отображение данных дорожного движения на карте | Microsoft Azure Maps
description: Узнайте, как добавить данные трафика в карты. Сведения о потоковых данных и о том, как использовать веб-пакет SDK Azure Maps для добавления данных о инцидентах и потоковых данных в карты.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: b6fcf39e05c4649503f0b7a80aadaaa16df24674
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98599561"
---
# <a name="show-traffic-on-the-map"></a>Отображение данных дорожного движения на карте

В Azure Maps доступны два типа данных трафика:

- Данные по происшествиям — это данные в виде точек и линий, которые обозначают, помимо прочего, строительные работы, закрытие дорог и аварии.
- Данные по потоку — метрики потока уличного движения. Данные по потоку уличного движения часто используются для раскрашивания дорог. Цвет зависит от того, насколько скорость движения потока меньше максимальной разрешенной скорости, или от другой метрики. В параметре трафика на карте можно передать четыре значения `flow` .

    |Значение потока | Описание|
    | :-- | :-- |
    | `none` | Не отображает данные трафика на карте |
    | `relative` | Показывает данные трафика, относящиеся к скорости произвольного потока в дороге |
    | `relative-delay` | Отображает области, которые выполняются медленнее среднего ожидаемого значения задержки |
    | `absolute` | Показывает абсолютную скорость всех транспортных средств в дороге |

Ниже приведен код для отображения данных дорожного движения на карте.

```javascript
//Show traffic on the map using the traffic options.
map.setTraffic({
    incidents: true,
    flow: 'relative'
});
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Отображение данных дорожного движения на карте' src='//codepen.io/azuremaps/embed/WMLRPw/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода для <a href='https://codepen.io/azuremaps/pen/WMLRPw/'>отображения данных дорожного движения на карте</a> службы "Карты Azure" (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="traffic-overlay-options"></a>Параметры наложения сведений о дорожном движении

Представленное ниже средство позволяет переключаться между различными параметрами наложения сведений о дорожном движении, чтобы увидеть, как изменяется визуализация. 

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Параметры наложения сведений о дорожном движении" src="//codepen.io/azuremaps/embed/RwbPqRY/?height=700&theme-id=0&default-tab=result" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/RwbPqRY/'>Параметры наложения сведений о дорожном движении</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="add-traffic-controls"></a>Добавление элементов управления трафиком

Существует два разных элемента управления трафиком, которые можно добавить на карту. Первый элемент управления, `TrafficControl` , добавляет выключатель, который можно использовать для включения и отключения входящего и исходящего трафика. Параметры этого элемента управления позволяют указать, когда следует использовать параметры трафика при отображении трафика. По умолчанию этот элемент управления будет отображать относительный поток трафика и данные об инциденте, однако можно изменить это значение, чтобы показать абсолютный поток трафика и при необходимости инциденты не будут. Второй элемент управления, `TrafficLegendControl` , добавляет к карте условные обозначения потока трафика, помогающие пользователю понять, что означает цветовой световой код цвета. Этот элемент управления отображается на карте, только когда данные потока трафика отображаются на карте и будут скрыты в любое другое время.

В следующем коде показано, как добавить элементы управления трафиком на карту.

```JavaScript
//Att the traffic control toogle button to the top right corner of the map.
map.controls.add(new atlas.control.TrafficControl(), { position: 'top-right' });

//Att the traffic legend control to the bottom left corner of the map.
map.controls.add(new atlas.control.TrafficLegendControl(), { position: 'bottom-left' });
```

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Элементы управления трафиком" src="https://codepen.io/azuremaps/embed/ZEWaeLJ?height500&theme-id=0&default-tab=js,result&embed-version=2&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/ZEWaeLJ'>Управление трафиком</a> пера с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Схема](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [TrafficOptions](/javascript/api/azure-maps-control/atlas.trafficoptions)

Расширьте возможности пользователей:

> [!div class="nextstepaction"]
> [Взаимодействие карты с событиями мыши](map-events.md)

> [!div class="nextstepaction"]
> [Создание карты](map-accessibility.md)

> [!div class="nextstepaction"]
> [Страница с примерами кода](https://aka.ms/AzureMapsSamples)
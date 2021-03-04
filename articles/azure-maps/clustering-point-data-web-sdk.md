---
title: Кластеризация данных точек на карте | Microsoft Azure Maps
description: Узнайте, как выполнять кластерные данные на картах. Узнайте, как использовать веб-пакет SDK Azure Maps для данных кластера, реагировать на события мыши в кластере и отображать статистические выражения кластера.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 0af327f624d0fed648012ce1d12bacabca688cec
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102044193"
---
# <a name="clustering-point-data"></a>Кластеризация данных точек

При визуализации большого количества точек данных на карте они могут перекрывать друг друга. Перекрытие может привести к тому, что карта станет перегруженной и очень сложной для восприятия. Кластеризация данных точек — это процесс объединения данных точек, находящихся рядом друг с другом, и представления их на карте в виде единой кластеризованной точки данных. По мере того как пользователь масштабирует карту, кластеры разбиваются на отдельные точки данных. При работе с большим количеством точек данных используйте процессы кластеризации, чтобы улучшить взаимодействие с пользователем.

</br>

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Clustering-point-data-in-Azure-Maps/player?format=ny]

## <a name="enabling-clustering-on-a-data-source"></a>Включение кластеризации в источнике данных

Включите кластеризацию в классе `DataSource`, задав для параметра `cluster` значение true. Задайте `clusterRadius` , чтобы выбрать соседние точки и объединить их в кластер. Значение `clusterRadius` указывается в пикселях. Используйте `clusterMaxZoom` , чтобы указать уровень масштабирования для отключения логики кластеризации. Ниже приведен пример включения кластеризации в источнике данных.

```javascript
//Create a data source and enable clustering.
var datasource = new atlas.source.DataSource(null, {
    //Tell the data source to cluster point data.
    cluster: true,
    
    //The radius in pixels to cluster points together.
    clusterRadius: 45,
    
    //The maximum zoom level in which clustering occurs.
    //If you zoom in more than this, all points are rendered as symbols.
    clusterMaxZoom: 15
});
```

> [!TIP]
> Если две точки данных находятся близко друг к другу, возможно, что кластер никогда не будет разбит, независимо от того, насколько близко пользователь увеличит масштаб. Для решения этой задачи можно задать параметр `clusterMaxZoom`, чтобы отключить логику кластеризации и просто отобразить все данные.

`DataSource`Класс предоставляет также следующие методы, связанные с кластеризацией.

| Метод | Возвращаемый тип | Описание |
|--------|-------------|-------------|
| getClusterChildren(clusterId: число) | Promise&lt;Массив&lt;Компонент&lt;Геометрия, любая&gt; \| Фигура&gt;&gt; | Извлекает дочерние элементы заданного кластера на следующем уровне масштабирования. Эти дочерние элементы могут быть сочетанием фигур и подкластеров. Подкластеры будут представлять собой компоненты со свойствами, соответствующими ClusteredProperties. |
| getClusterExpansionZoom(clusterId: число) | Promise&lt;число&gt; | Вычисляет уровень масштабирования, при котором кластер начинает расширяться или разбиваться на части. |
| getClusterLeaves(clusterId: число, предел: число, смещение: число) | Promise&lt;Массив&lt;Компонент&lt;Геометрия, любая&gt; \| Фигура&gt;&gt; | Извлекает все точки в кластере. Задайте `limit`, чтобы вернуть подмножество точек, и используйте `offset`, чтобы пролистать страницы с точками. |

## <a name="display-clusters-using-a-bubble-layer"></a>Отображение кластеров с помощью слоя пузырьков

Слой пузырьков — это отличный способ визуализации кластеризованных точек. Используйте выражения для масштабирования радиуса и изменения цвета в зависимости от количества точек в кластере. При отображении кластеров с помощью слоя пузырьков следует использовать отдельный слой для отображения некластеризованных точек данных.

Чтобы отобразить размер кластера поверх пузырька, используйте слой символов с текстом и не используйте значок.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Базовая кластеризация с помощью слоя пузырьков" src="//codepen.io/azuremaps/embed/qvzRZY/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/qvzRZY/'>базовой кластеризации с помощью слоя пузырьков</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="display-clusters-using-a-symbol-layer"></a>Отображение кластеров с помощью слоя символов

При визуализации точек данных слой символов автоматически скрывает символы, которые накладываются друг на друга, чтобы обеспечить удобство работы с картой для пользователя. Это поведение по умолчанию может оказаться нежелательным, если на карте необходимо отобразить плотность точек данных. Однако эти параметры можно изменить. Чтобы отобразить все символы, задайте для параметра `allowOverlap` в свойстве `iconOptions` слоя символов значение `true`. 

Используйте кластеризацию для отображения плотности точек данных для отображения пользовательского интерфейса, не перегруженного информацией. В примере ниже показано, как добавить настраиваемые символы и представить кластеры и отдельные точки данных с помощью слоя символов.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Кластеризованный слой символов" src="//codepen.io/azuremaps/embed/Wmqpzz/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/Wmqpzz/'>кластеризованного слоя символов</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="clustering-and-the-heat-maps-layer"></a>Кластеризация и слой тепловых карт

Тепловые карты — это отличный способ отображения плотности данных на карте. Этот метод визуализации может самостоятельно обрабатывать большое количество точек данных. Если точки данных кластеризованы, а размер кластера используется в качестве веса тепловой карты, то с помощью тепловой карты можно обработать еще больший объем данных. Чтобы получить этот параметр, задайте для параметра `weight` слоя тепловой карты значение `['get', 'point_count']`. Если радиус кластера мал, то тепловая карта будет выглядеть почти так же, как на тепловой карте с использованием некластеризованных точек данных, однако она будет работать гораздо лучше. Тем не менее чем меньше радиус кластера, тем точнее, но менее производительной, будет тепловая карта.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Кластеризация взвешенной тепловой карты" src="//codepen.io/azuremaps/embed/VRJrgO/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/VRJrgO/'>кластеризации взвешенной тепловой карты</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="mouse-events-on-clustered-data-points"></a>События мыши в кластеризованных точках данных

Когда на слое, на котором содержатся кластеризованные точки данных, возникают события мыши, кластеризованная точка данных возвращается в событие в виде объекта признака точки GeoJSON. Этот признак точки будет иметь следующие свойства:

| Имя свойства             | Тип    | Описание   |
|---------------------------|---------|---------------|
| `cluster`                 | Логическое | Указывает, представляет ли компонент кластер. |
| `cluster_id`              | строка  | Уникальный идентификатор кластера, который можно использовать с методами `getClusterExpansionZoom`, `getClusterChildren` и `getClusterLeaves` для DataSource. |
| `point_count`             | number  | Число точек, содержащихся в кластере.  |
| `point_count_abbreviated` | строка  | Строка, которая сокращает значение `point_count`, если оно длинное. (например, 4000 преобразуется в 4K).  |

В этом примере используется слой пузырьков, который визуализирует точки кластера и добавляет событие щелчка. При срабатывании события щелчка код вычисляет и масштабирует карту до следующего уровня масштаба, при котором кластер разбивается. Эта функция реализуется с помощью метода `getClusterExpansionZoom` класса `DataSource` и свойства `cluster_id` кластеризованной точки данных, которую щелкнули.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Метод getClusterExpansionZoom для кластера" src="//codepen.io/azuremaps/embed/moZWeV/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/moZWeV/'>с методом getClusterExpansionZoom для кластера</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="display-cluster-area"></a>Отображение области кластера 

Данные точки, которые представляет кластер, распределяются по области. В этом примере при наведении указателя мыши на кластер происходит два основных действия. Во-первых, вычисляется выпуклая оболочка по отдельным точкам данных, содержащимся в кластере. Затем на карте отображается выпуклая оболочка для отображения области.  Выпуклая оболочка — это многоугольник, который ограничивает набор точек, например эластичной полосой, и может быть вычислен с помощью метода `atlas.math.getConvexHull`. Все точки, содержащиеся в кластере, можно получить из источника данных с помощью метода `getClusterLeaves`.

<br/>

 <iframe height="500" style="width: 100%;" scrolling="no" title="Выпуклая оболочка области кластера" src="//codepen.io/azuremaps/embed/QoXqWJ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/QoXqWJ/'>выпуклой оболочки области кластера</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="aggregating-data-in-clusters"></a>Статистическая обработка данных в кластерах

Зачастую кластеры обозначаются символом с количеством точек в кластере. Но иногда возникает необходимость настроить стиль кластеров с дополнительными метриками. При использовании статистических выражений кластера настраиваемые свойства можно создавать и заполнять путем вычисления [статистического выражения](data-driven-style-expressions-web-sdk.md#aggregate-expression).  Статистические выражения кластера можно определить в параметре`clusterProperties` элемента `DataSource`.

В следующем примере используется статистическое выражение. Код вычисляет значение счетчика на основе свойства типа сущности каждой точки данных в кластере. Когда пользователь щелкает кластер, появляется всплывающее окно с дополнительными сведениями о кластере.

<iframe height="500" style="width: 100%;" scrolling="no" title="Статистические выражения кластера" src="//codepen.io/azuremaps/embed/jgYyRL/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/jgYyRL/'>статических выражений кластера</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Класс DataSource](/javascript/api/azure-maps-control/atlas.source.datasource)

> [!div class="nextstepaction"]
> [Объект DataSourceOptions](/javascript/api/azure-maps-control/atlas.datasourceoptions)

> [!div class="nextstepaction"]
> [Пространство имен atlas.math](/javascript/api/azure-maps-control/atlas.math)

См. примеры кодов для добавления функций в приложение:

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](map-add-pin.md)

> [!div class="nextstepaction"]
> [Добавление слоя тепловой карты](map-add-heat-map-layer.md)
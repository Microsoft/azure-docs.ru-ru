---
title: Создание схемы с помощью Azure Maps | Карты Microsoft Azure
description: Узнайте, как добавлять карты на веб-страницы с помощью веб-пакета SDK для Azure Maps. Сведения о параметрах анимации, стиля, камеры, служб и взаимодействий пользователей.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/26/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 833b6413cc5dfde1129075a286e5fe93a06e159f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92890927"
---
# <a name="create-a-map"></a>Создание карты

В этой статье показаны способы, позволяющие создать и анимировать карту.  

## <a name="loading-a-map"></a>Загрузка схемы

Чтобы загрузить карту, создайте новый экземпляр [класса Map](/javascript/api/azure-maps-control/atlas.map). При инициализации карты передайте идентификатор элемента DIV для отрисовки карты и передайте набор параметров для использования при загрузке карты. Если в пространстве имен не указаны сведения для проверки подлинности по умолчанию `atlas` , эти сведения необходимо указать в параметрах Map при загрузке схемы. Эта схема асинхронно загружает несколько ресурсов для повышения производительности. Таким образом, после создания экземпляра Map присоедините `ready` `load` событие или к карте, а затем добавьте дополнительный код, взаимодействующий с картой с обработчиком событий. `ready`Событие срабатывает, как только на карте загружено достаточно ресурсов для взаимодействия с программным способом. `load`Событие срабатывает после полного завершения загрузки представления первоначальной таблицы. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Базовая Загрузка карт" src="//codepen.io/azuremaps/embed/rXdBXx/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел " <a href='https://codepen.io/azuremaps/pen/rXdBXx/'>Базовая схема</a> пера: Загрузка" с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> На одной странице можно загрузить несколько карт. Несколько карт на одной странице могут использовать одинаковые или разные параметры проверки подлинности и языка.

## <a name="show-a-single-copy-of-the-world"></a>Отображение одной копии мира

Когда на широком экране отображается изображение, несколько копий мира отображаются по горизонтали. Этот вариант прекрасно подходит для некоторых сценариев, но для других приложений желательно просмотреть единую копию мира. Это поведение реализуется путем установки параметра Maps `renderWorldCopies` в значение `false` .

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Рендерворлдкопиес = false" src="//codepen.io/azuremaps/embed/eqMYpZ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. Рендерворлдкопиес (перо) <a href='https://codepen.io/azuremaps/pen/eqMYpZ/'>= false</a> by Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="map-options"></a>Параметры карт

При создании схемы существует несколько различных типов параметров, которые могут быть переданы для настройки функций Map, как показано ниже.

- [Камераоптионс](/javascript/api/azure-maps-control/atlas.cameraoptions) и [камерабаундоптионс](/javascript/api/azure-maps-control/atlas.cameraboundsoptions) используются для указания области, отображаемой картой.
- [Сервицеоптионс](/javascript/api/azure-maps-control/atlas.serviceoptions) используются для указания того, как эта схема должна взаимодействовать со службами, которые работают с картой.
- [Стилеоптионс](/javascript/api/azure-maps-control/atlas.styleoptions) используются для указания того, должен ли сопоставленный стиль и отрисовываться.
- [Усеринтерактионоптионс](/javascript/api/azure-maps-control/atlas.userinteractionoptions) используются для указания способа, которым должна достигаться схема при взаимодействии пользователя с картой. 

Эти параметры можно также обновить после загрузки схемы с помощью `setCamera` функций,, `setServiceOptions` `setStyle` и `setUserInteraction` . 

## <a name="controlling-the-map-camera"></a>Управление камерой карт

Существует два способа задать отображаемую область на карте с помощью камеры на карте. При загрузке схемы можно задать параметры камеры. Или можно вызвать `setCamera` параметр в любое время после загрузки картой, чтобы программно обновить представление Map.  

<a id="setCameraOptions"></a>

### <a name="set-the-camera"></a>Настройка камеры

Камера на карте управляет тем, что отображается в области просмотра холста Map. Параметры камеры можно передать в параметры карты при инициализации или передаче в `setCamera` функцию Maps.

```javascript
//Set the camera options when creating the map.
var map = new atlas.Map('map', {
    center: [-122.33, 47.6],
    zoom: 12

    //Additional map options.
};

//Update the map camera at anytime using setCamera function.
map.setCamera({
    center: [-110, 45],
    zoom: 5 
});
```

В следующем коде создается [объект Map](/javascript/api/azure-maps-control/atlas.map) , а также устанавливаются параметры центрирования и масштабирования. Свойства карт, такие как выравнивание по центру и масштабу, являются частью [камераоптионс](/javascript/api/azure-maps-control/atlas.cameraoptions).

<br/>

<iframe height='500' scrolling='no' title='Создание карты с помощью CameraOptions' src='//codepen.io/azuremaps/embed/qxKBMN/?height=543&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Посмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/qxKBMN/'>Создать карту через `CameraOptions` </a>от Azure Location Based Services (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

<a id="setCameraBoundsOptions"></a>

### <a name="set-the-camera-bounds"></a>Настройка границ камеры

Ограничивающий прямоугольник можно использовать для обновления камеры на карте. Если ограничивающий прямоугольник был вычислен из данных Point, часто бывает полезно указать значение заполнения пикселов в параметрах камеры для учета размера значка. Это позволит гарантировать, что точки не выходят за границы окна просмотра карт.

```javascript
map.setCamera({
    bounds: [-122.4, 47.6, -122.3, 47.7],
    padding: 10
});
```

В следующем коде [объект Map](/javascript/api/azure-maps-control/atlas.map) создается с помощью `new atlas.Map()` . Свойства карты, например `CameraBoundsOptions`, можно определить через функцию [setCamera](/javascript/api/azure-maps-control/atlas.map) класса Map. Свойства границ и заполнения задаются с помощью `setCamera`.

<br/>

<iframe height='500' scrolling='no' title='Создание карты с помощью CameraBoundsOptions' src='//codepen.io/azuremaps/embed/ZrRbPg/?height=543&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Посмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/ZrRbPg/'>Создание карты через `CameraBoundsOptions` </a>от Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="animate-map-view"></a>Просмотр анимации карты

При настройке параметров камеры на карте можно также задать [Параметры анимации](/javascript/api/azure-maps-control/atlas.animationoptions) . Эти параметры определяют тип анимации и длительность, которую необходимо предпринять для перемещения камеры.

```javascript
map.setCamera({
    center: [-122.33, 47.6],
    zoom: 12,
    duration: 1000,
    type: 'fly'  
});
```

В следующем коде первый блок кода создает карту и задает стили карт ввода и масштабирования. Во втором блоке кода для кнопки анимировать создается обработчик событий нажатия. При нажатии этой кнопки `setCamera` функция вызывается с некоторыми случайными значениями для [Камераоптионс](/javascript/api/azure-maps-control/atlas.cameraoptions) и [аниматионоптионс](/javascript/api/azure-maps-control/atlas.animationoptions).

<br/>

<iframe height='500' scrolling='no' title='Просмотр анимации карты' src='//codepen.io/azuremaps/embed/WayvbO/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Изучите фрагмент кода <a href='https://codepen.io/azuremaps/pen/WayvbO/'>Просмотр анимации карты</a> для Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="request-transforms"></a>Преобразования запросов

Иногда полезно иметь возможность изменять HTTP-запросы, выполняемые элементом управления картой. Пример:

- Добавление дополнительных заголовков в запросы плиток. Это часто делается для защищенных паролем служб.
- Измените URL-адреса для выполнения запросов через прокси-службу.

[Параметры службы](/javascript/api/azure-maps-control/atlas.serviceoptions) на карте имеют объект `transformRequest` , который можно использовать для изменения всех запросов, сделанных картой, до их внесения. `transformRequest`Параметр — это функция, которая принимает два параметра: Строковый URL-адрес и строку типа ресурса, которая указывает, для чего используется запрос. Эта функция должна возвращать результат [столбец requestparameters](/javascript/api/azure-maps-control/atlas.requestparameters) .

```JavaScript
transformRequest: (url: string, resourceType: string) => RequestParameters
```

В следующем примере показано, как использовать этот параметр для изменения всех запросов до размера `https://example.com` путем добавления имени пользователя и пароля в качестве заголовков для запроса.

```JavaScript
var map = new atlas.Map('myMap', {
    transformRequest: function (url, resourceType) {
        //Check to see if the request is to the specified endpoint.
        if (url.indexOf('https://examples.com') > -1) {
            //Add custom headers to the request.
            return {
                url: url,
                header: {
                    username: 'myUsername',
                    password: 'myPassword'
                }
            };
        }

        //Return the URL unchanged by default.
        return { url: url };
    },

    authOptions: {
        authType: 'subscriptionKey',
        subscriptionKey: '<Your Azure Maps Key>'
    }
});
```

## <a name="try-out-the-code"></a>Тестирование кода

Взгляните на примеры кода. Код JavaScript можно изменить на **вкладке JS** и просмотреть изменения в представлении Map на **вкладке результат**. Можно также щелкнуть **изменить в CodePen** в правом верхнем углу и изменить код в CodePen.

<a id="relatedReference"></a>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Схема](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [камераоптионс](/javascript/api/azure-maps-control/atlas.cameraoptions)

> [!div class="nextstepaction"]
> [аниматионоптионс](/javascript/api/azure-maps-control/atlas.animationoptions)

См. примеры кодов для добавления функций в приложение:

> [!div class="nextstepaction"]
> [Изменение стиля карты](choose-map-style.md)

> [!div class="nextstepaction"]
> [Добавление элементов управления на карту](map-add-controls.md)

> [!div class="nextstepaction"]
> [Примеры кода](/samples/browse/?products=azure-maps)
---
title: Изменение стиля веб-Map Control Azure Maps
description: Узнайте, как изменить стиль и параметры схемы. См. раздел Добавление элемента управления "Выбор стиля" на карту в Azure Maps, чтобы пользователи могли переключаться между различными стилями.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 556e265cc0d1aae33823185ec98d23f191ed1694
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97680022"
---
# <a name="change-the-style-of-the-map"></a>Изменение стиля карты

Элемент управления картой поддерживает несколько различных [параметров стиля](/javascript/api/azure-maps-control/atlas.styleoptions) отображения и [базовых стилей карт](supported-map-styles.md). Все стили можно задать при инициализации элемента управления Map. Или можно задать стили с помощью функции элемента управления Map `setStyle` . В этой статье показано, как использовать эти параметры стиля для настройки внешнего вида схемы. Кроме того, вы узнаете, как реализовать элемент управления "Выбор стиля" на карте. Элемент управления Выбор стиля позволяет пользователю переключаться между различными базовыми стилями.

## <a name="set-map-style-options"></a>Установка параметров стиля схемы

Параметры стиля можно задать во время инициализации веб-элемента управления. Можно также обновить параметры стиля, вызвав функцию элемента управления Map `setStyle` . Чтобы просмотреть все доступные параметры стиля, см. раздел [Параметры стиля](/javascript/api/azure-maps-control/atlas.styleoptions).

```javascript
//Set the style options when creating the map.
var map = new atlas.Map('map', {
    renderWorldCopies: false,
    showBuildingModels: false,
    showLogo: true,
    showFeedbackLink: true,
    style: 'road'

    //Additional map options.
};

//Update the style options at anytime using `setStyle` function.
map.setStyle({
    renderWorldCopies: true,
    showBuildingModels: true,
    showLogo: false,
    showFeedbackLink: false
});
```

В следующем средстве показано, как различные параметры стиля изменяют способ отрисовки карт. Чтобы увидеть трехмерные строения, увеличивайте масштаб до основного города.

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Параметры стиля карт" src="https://codepen.io/azuremaps/embed/eYNMjPb?height=700&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/eYNMjPb'>Параметры стиля отображения</a> перьев по Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="set-a-base-map-style"></a>Задать базовый стиль карт

Можно также инициализировать элемент управления картой с помощью одного из [стилей базовой схемы](supported-map-styles.md) , доступных в веб-пакете SDK. Затем можно использовать `setStyle` функцию для обновления базового стиля с другим стилем отображения.

### <a name="set-a-base-map-style-on-initialization"></a>Задать базовый стиль отображения при инициализации

Базовые стили элемента управления картой можно задать во время инициализации. В следующем коде `style` параметру элемента управления картой присвоено значение [ `grayscale_dark` базового стиля Map](supported-map-styles.md#grayscale_dark).  

```javascript
var map = new atlas.Map('map', {
    style: 'grayscale_dark',

    //Additional map options
);
```

<br/>

<iframe height='500' scrolling='no' title='Установка стиля при загрузке карты' src='//codepen.io/azuremaps/embed/WKOQRq/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. фрагмент кода <a href='https://codepen.io/azuremaps/pen/WKOQRq/'>Установка стиля при загрузке карты</a> в службе Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="update-the-base-map-style"></a>Обновление базового стиля Map

Базовый стиль карт можно обновить с помощью `setStyle` функции и задав `style` параметр, чтобы изменить базовый стиль базовой карте или добавить дополнительные параметры стиля.

```javascript
map.setStyle({ style: 'satellite' });
```

В следующем коде после загрузки экземпляра Map стиль схемы обновляется с `grayscale_dark` на с `satellite` помощью функции [SetStyle](/javascript/api/azure-maps-control/atlas.map#setstyle-styleoptions-) .

<br/>

<iframe height='500' scrolling='no' title='Обновление стиля' src='//codepen.io/azuremaps/embed/yqXYzY/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. фрагмент кода <a href='https://codepen.io/azuremaps/pen/yqXYzY/'>Обновление стиля</a> в службе Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-the-style-picker-control"></a>Добавление элемента управления "Выбор стиля"

Элемент управления Выбор стиля позволяет легко использовать кнопку с раскрывающейся панелью, которая может использоваться конечным пользователем для переключения между базовыми стилями.

Средство выбора стиля имеет два разных варианта макета: `icon` и `list` . Кроме того, средство выбора стилей позволяет выбрать два разных варианта управления Выбор стиля `style` : `light` и `dark` . В этом примере средство выбора стиля использует `icon` Макет и отображает список выбора стилей базовых карт в виде значков. Средство выбора элементов стиля включает следующий базовый набор стилей: `["road", "grayscale_light", "grayscale_dark", "night", "road_shaded_relief"]` . Дополнительные сведения о параметрах элемента управления "Выбор стиля" см. в разделе [Параметры элемента управления стиля](/javascript/api/azure-maps-control/atlas.stylecontroloptions).

На рисунке ниже показан элемент управления "Выбор стиля", отображаемый в `icon` макете.

:::image type="content" source="./media/choose-map-style/style-picker-icon-layout.png" alt-text="Макет значка выбора стиля":::

На рисунке ниже показан элемент управления "Выбор стиля", отображаемый в `list` макете.

:::image type="content" source="./media/choose-map-style/style-picker-list-layout.png" alt-text="Макет списка средств выбора стилей":::

> [!IMPORTANT]
> По умолчанию элемент управления Выбор стиля перечисляет все стили, доступные в ценовой категории S0 Azure Maps. Если требуется сократить число стилей в этом списке, передайте массив стилей, которые должны отображаться в списке, в `mapStyle` параметр выбора стиля. Если вы используете S1 и хотите отобразить все доступные стили, задайте `mapStyles` для параметра Выбор стиля значение `"all"` .

В следующем коде показано, как переопределить `mapStyles` список базовых стилей по умолчанию. В этом примере мы задаете `mapStyles` параметр для перечисления базовых стилей, которые должны отображаться элементом управления "Выбор стиля".

<br/>

<iframe height='500' scrolling='no' title='Добавление элемента выбора стиля' src='//codepen.io/azuremaps/embed/OwgyvG/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. фрагмент кода <a href='https://codepen.io/azuremaps/pen/OwgyvG/'>Добавление элемента выбора стиля</a> в службе Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, используемых в этой статье.

> [!div class="nextstepaction"]
> [Схема](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [стилеоптионс](/javascript/api/azure-maps-control/atlas.styleoptions)

> [!div class="nextstepaction"]
> [StyleControl](/javascript/api/azure-maps-control/atlas.control.stylecontrol).

> [!div class="nextstepaction"]
> [стилеконтролоптионс](/javascript/api/azure-maps-control/atlas.stylecontroloptions)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление элементов управления картой в службе Azure Maps](map-add-controls.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](map-add-pin.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer.md)

---
title: Шаблоны изображений в Azure Maps веб-пакете SDK | Карты Microsoft Azure
description: Узнайте, как добавлять значки изображений и закрашенные многоугольники для карт с помощью веб-пакета SDK Azure Maps. Просмотр доступных шаблонов образцов изображений и заливок.
author: rbrundritt
ms.author: richbrun
ms.date: 8/6/2019
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: cpendleton
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 5f455a1132a0f63e1ba3eb5d2a57a1f9bfa9a867
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895687"
---
# <a name="how-to-use-image-templates"></a>Использование шаблонов образов

Изображения можно использовать с маркерами HTML и различными слоями в Azure Maps веб-пакете SDK:

 - Слои символов позволяют отображать точки на карте с помощью значка изображения. Символы также можно визуализировать по линиям.
 - Слои многоугольников можно визуализировать с помощью изображения шаблона заливки. 
 - Маркеры HTML могут отображать точки с помощью изображений и других элементов HTML.

Чтобы обеспечить хорошую производительность при работе с слоями, перед отрисовкой Загрузите изображения в ресурс с изображением Sprite. [Иконоптионс](/javascript/api/azure-maps-control/atlas.iconoptions)(симболлайер) предварительно загружает несколько изображений маркера в несколько цветов в спрайт изображения Map по умолчанию. Эти изображения маркеров и многое другое доступны в виде шаблонов SVG. Они могут использоваться для создания изображений с настраиваемыми масштабами или использования в качестве основного и дополнительного цветов клиента. В итоге предусмотрено 42 шаблонов изображений: 27 значков символов и 15 узоров заливки многоугольников.

Шаблоны изображений можно добавить в ресурсы с изображением Sprite с помощью `map.imageSprite.createFromTemplate` функции. Эта функция позволяет передавать до пяти параметров;

```javascript
createFromTemplate(id: string, templateName: string, color?: string, secondaryColor?: string, scale?: number): Promise<void>
```

`id`— Это уникальный идентификатор, который вы создаете. `id`Назначается образу при его добавлении в спрайт изображений Maps. Используйте этот идентификатор в слоях, чтобы указать, какой ресурс изображения следует визуализировать. `templateName`Указывает используемый шаблон изображения. `color`Параметр задает основной цвет изображения, а `secondaryColor` Параметры задают вторичный цвет изображения. `scale`Параметр масштабирует шаблон изображения перед его применением к спрайту изображений. Когда изображение применяется к спрайту изображений, оно преобразуется в PNG. Чтобы обеспечить четкое отображение, лучше масштабировать шаблон изображения перед добавлением его к спрайту, чем масштабировать его в слое.

Эта функция асинхронно загружает изображение в спрайт изображений. Таким же методом возвращается обещание, которое можно ожидать завершения этой функции.

В следующем коде показано, как создать изображение из одного из встроенных шаблонов и использовать его с уровнем символов.

```javascript
map.imageSprite.createFromTemplate('myTemplatedIcon', 'marker-flat', 'teal', '#fff').then(function () {

    //Add a symbol layer that uses the custom created icon.
    map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
        iconOptions: {
            image: 'myTemplatedIcon'
        }
    }));
});
```

## <a name="use-an-image-template-with-a-symbol-layer"></a>Использование шаблона изображения с уровнем символов

После загрузки шаблона изображения в спрайт изображений карт его можно визуализировать как символ на уровне символов, ссылаясь на идентификатор ресурса изображения в `image` параметре `iconOptions` .

В следующем примере слой символов визуализируется с помощью `marker-flat` шаблона изображения с бирюзовым основным цветом и белым вторичным цветом. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Слой символов с встроенным шаблоном значка" src="//codepen.io/azuremaps/embed/VoQMPp/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/VoQMPp/'>слой символов пера со встроенным шаблоном значка</a> Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="use-an-image-template-along-a-lines-path"></a>Использование шаблона изображения вдоль строкового пути

После загрузки шаблона изображения в спрайт изображений карт его можно визуализировать по пути линии, добавив LineString к источнику данных и используя слой символов с `lineSpacing` параметром и обратившись к идентификатору ресурса изображения в `image` параметре TH `iconOptions` . 

Следующий пример выводит розовую линию на карте и использует слой символов с помощью `car` шаблона изображения с синим основным цветом тускло-Васильковый и белым вторичным цветом. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Слой линий со встроенным шаблоном значка" src="//codepen.io/azuremaps/embed/KOQvJe/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/KOQvJe/'>слой линий пера со встроенным шаблоном значка</a> Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Если вы указываете шаблон изображения, установите `rotation` для параметра значок на уровне символов значение 90, если хотите, чтобы он указывал на то же направление, что и линия.

## <a name="use-an-image-template-with-a-polygon-layer"></a>Использование шаблона изображения с слоем многоугольников

После загрузки шаблона изображения в спрайт изображений карт его можно визуализировать как шаблон заливки на слое многоугольников, ссылаясь на идентификатор ресурса изображения в `fillPattern` параметре слоя.

В следующем примере слой многоугольников визуализируется с помощью `dot` шаблона изображения с красным основным цветом и прозрачным вторичным цветом.  

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Заливка многоугольника с помощью встроенного шаблона значка" src="//codepen.io/azuremaps/embed/WVMEmz/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/WVMEmz/'>Заливка многоугольника с помощью встроенного шаблона значка</a> Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Установка вторичного цвета шаблонов заливки упрощает просмотр базовой схемы, тем самым предоставляя первичный шаблон. 

## <a name="use-an-image-template-with-an-html-marker"></a>Использование шаблона изображения с маркером HTML

Шаблон изображения можно получить с помощью `altas.getImageTemplate` функции и использовать в качестве содержимого HTML-маркера. Шаблон можно передать в `htmlContent` параметр маркера, а затем настроить с помощью `color` `secondaryColor` параметров, и `text` .

В следующем примере используется `marker-arrow` шаблон с красным основным цветом, розовым вторичным цветом и текстовым значением "00".

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="HTML-маркер со встроенным шаблоном значка" src="//codepen.io/azuremaps/embed/EqQvzq/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Просмотрите HTML- <a href='https://codepen.io/azuremaps/pen/EqQvzq/'>маркер пера со встроенным шаблоном значка</a> Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>


> [!TIP]
> Шаблоны изображений можно использовать за пределами схемы. Жетимажетемплате масштабируются возвращает строку SVG, которая содержит заполнители. `{color}`, `{secondaryColor}`, `{scale}`, `{text}`. Замените эти значения заполнителей, чтобы создать допустимую строку SVG. Затем можно либо добавить строку SVG непосредственно в HTML DOM, либо преобразовать ее в универсальный код ресурса (URI) данных и вставить в тег Image. Пример:
> ```JavaScript
> //Retrieve an SVG template and replace the placeholder values.
> var svg = atlas.getImageTemplate('marker').replace(/{color}/, 'red').replace(/{secondaryColor}/, 'white').replace(/{text}/, '').replace(/{scale}/, 1);
>
> //Convert to data URI for use in image tags.
> var dataUri = 'data:image/svg+xml;base64,' + btoa(svg);
> ```

## <a name="create-custom-reusable-templates"></a>Создание настраиваемых многократно используемых шаблонов

Если приложение использует один и тот же значок с разными значками или если вы создаете модуль, который добавляет дополнительные шаблоны изображений, вы можете легко добавлять и получать эти значки из веб-пакета SDK Azure Maps. Используйте следующие статические функции в `atlas` пространстве имен.

| Имя | Тип возвращаемых данных | Описание | 
|-|-|-|
| `addImageTemplate(templateName: string, template: string, override: boolean)` | | Добавляет пользовательский шаблон изображения SVG в пространство имен Atlas. |
|  `getImageTemplate(templateName: string, scale?: number)`| строка | Извлекает шаблон SVG по имени. |
| `getAllImageTemplateNames()` | string[] |  Извлекает шаблон SVG по имени. |

Шаблоны изображений SVG поддерживают следующие значения заполнителей:

| Заполнитель | Описание |
|-|-|
| `{color}` | Основной цвет. | 
| `{secondaryColor}` | Вторичный цвет. | 
| `{scale}` | Изображение SVG преобразуется в изображение PNG при добавлении в спрайт изображений карт. Этот заполнитель можно использовать для масштабирования шаблона перед его преобразованием, чтобы обеспечить его четкое отображение. | 
| `{text}` | Расположение для отрисовки текста при использовании с маркером HTML. |

В следующем примере показано, как взять шаблон SVG и добавить его в Azure Maps веб-пакет SDK в качестве шаблона многократно используемого значка. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Добавление пользовательского шаблона значка в пространство имен Atlas" src="//codepen.io/azuremaps/embed/NQyvEX/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. раздел <a href='https://codepen.io/azuremaps/pen/NQyvEX/'>Добавление пользовательского шаблона значка в пространство имен Atlas</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="list-of-image-templates"></a>Список шаблонов изображений

В этой таблице перечислены все шаблоны изображений, доступные в данный момент в Azure Maps веб-пакете SDK. Имя шаблона выше каждого изображения. По умолчанию основным цветом является синий цвет, а дополнительный — белым. Чтобы сделать дополнительный цвет более удобным для просмотра на белом фоне, на следующих изображениях дополнительный цвет устанавливается в черный.

**Шаблоны значков символов**

:::row:::
   :::column span="":::
      marker
   :::column-end:::
   :::column span="":::
      жирный маркер
   :::column-end:::
   :::column span="":::
      маркер-окружность
   :::column-end:::
   :::column span="":::
      Плоский маркер
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок маркера](./media/image-templates/marker.png)
   :::column-end:::
   :::column span="":::
      ![значок жирного маркера](./media/image-templates/marker-thick.png)
   :::column-end:::
   :::column span="":::
      ![значок "маркер-окружность"](./media/image-templates/marker-circle.png)
   :::column-end:::
   :::column span="":::
      ![значок с плоским маркером](./media/image-templates/marker-flat.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      квадратный маркер
   :::column-end:::
   :::column span="":::
      маркер-квадратный — кластер
   :::column-end:::
   :::column span="":::
      стрелка-маркер
   :::column-end:::
   :::column span="":::
      маркер-шарик-закрепление
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок квадратного маркера](./media/image-templates/marker-square.png)
   :::column-end:::
   :::column span="":::
      ![значок "маркер — квадратный — кластер"](./media/image-templates/marker-square-cluster.png)
   :::column-end:::
   :::column span="":::
      ![значок стрелки-маркера](./media/image-templates/marker-arrow.png)
   :::column-end:::
   :::column span="":::
      ![значок "маркер-шарик-закрепление"](./media/image-templates/marker-ball-pin.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      маркер-квадратное округление
   :::column-end:::
   :::column span="":::
      маркер-квадратный-Скругленный — кластер
   :::column-end:::
   :::column span="":::
      flag
   :::column-end:::
   :::column span="":::
      Flag-треугольник
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок "маркер-квадрат-Скругленный"](./media/image-templates/marker-square-rounded.png)
   :::column-end:::
   :::column span="":::
      ![значок "маркер-квадрат-Скругленный — кластер"](./media/image-templates/marker-square-rounded-cluster.png)
   :::column-end:::
   :::column span="":::
      ![значок флага](./media/image-templates/flag.png)
   :::column-end:::
   :::column span="":::
      ![значок флага-треугольник](./media/image-templates/flag-triangle.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      треугольник
   :::column-end:::
   :::column span="":::
      треугольник — толстый
   :::column-end:::
   :::column span="":::
      стрелка на треугольник вверх
   :::column-end:::
   :::column span="":::
      треугольная стрелка — слева
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок треугольника](./media/image-templates/triangle.png)
   :::column-end:::
   :::column span="":::
      ![значок с толстым треугольником](./media/image-templates/triangle-thick.png)
   :::column-end:::
   :::column span="":::
      ![значок стрелки "треугольник"](./media/image-templates/triangle-arrow-up.png)
   :::column-end:::
   :::column span="":::
      ![значок стрелки влево](./media/image-templates/triangle-arrow-left.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      шестиугольник
   :::column-end:::
   :::column span="":::
      шестиугольник — толстая
   :::column-end:::
   :::column span="":::
      шестиугольник — округленный
   :::column-end:::
   :::column span="":::
      шестиугольник-Скругленный-толстый
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок шестиугольника](./media/image-templates/hexagon.png)
   :::column-end:::
   :::column span="":::
      ![шестиугольник — толстый значок](./media/image-templates/hexagon-thick.png)
   :::column-end:::
   :::column span="":::
      ![значок в кружке](./media/image-templates/hexagon-rounded.png)
   :::column-end:::
   :::column span="":::
      ![значок в шестиугольнике-закругленный — толстый](./media/image-templates/hexagon-rounded-thick.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      pin
   :::column-end:::
   :::column span="":::
      закрепить-округлить
   :::column-end:::
   :::column span="":::
      Скругленный квадрат
   :::column-end:::
   :::column span="":::
      скругленное-квадратное-толстое
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок закрепления](./media/image-templates/pin.png)
   :::column-end:::
   :::column span="":::
      ![значок "закрепить на круге"](./media/image-templates/pin-round.png)
   :::column-end:::
   :::column span="":::
      ![значок скругленного квадрата](./media/image-templates/rounded-square.png)
   :::column-end:::
   :::column span="":::
      ![значок со скругленной квадратной линией](./media/image-templates/rounded-square-thick.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      Стрелка вверх
   :::column-end:::
   :::column span="":::
      стрелка — тонкая
   :::column-end:::
   :::column span="":::
      car
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок стрелки "вверх"](./media/image-templates/arrow-up.png)
   :::column-end:::
   :::column span="":::
      ![Стрелка-вверх-тонкий значок](./media/image-templates/arrow-up-thin.png)
   :::column-end:::
   :::column span="":::
      ![значок автомобиля](./media/image-templates/car.png)
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::


**Шаблоны узорной заливки многоугольников**

:::row:::
   :::column span="":::
      программы
   :::column-end:::
   :::column span="":::
      повернутый
   :::column-end:::
   :::column span="":::
      круги
   :::column-end:::
   :::column span="":::
      круги — с интервалами
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок проверки](./media/image-templates/checker.png)
   :::column-end:::
   :::column span="":::
      ![значок, повернутый в шашку](./media/image-templates/checker-rotated.png)
   :::column-end:::
   :::column span="":::
      ![значок кругов](./media/image-templates/circles.png)
   :::column-end:::
   :::column span="":::
      ![значок "кружки — пробел"](./media/image-templates/circles-spaced.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      по диагонали вверх
   :::column-end:::
   :::column span="":::
      диагональные линии вниз
   :::column-end:::
   :::column span="":::
      диагональные полосы
   :::column-end:::
   :::column span="":::
      диагональные полосы-вниз
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок "диагональные линии"](./media/image-templates/diagonal-lines-up.png)
   :::column-end:::
   :::column span="":::
      ![значок "диагональные линии"](./media/image-templates/diagonal-lines-down.png)
   :::column-end:::
   :::column span="":::
      ![значок "диагональные полосы"](./media/image-templates/diagonal-stripes-up.png)
   :::column-end:::
   :::column span="":::
      ![значок "диагональные полосы"](./media/image-templates/diagonal-stripes-down.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      Сетка-линии
   :::column-end:::
   :::column span="":::
      повернутый-Grid-Line
   :::column-end:::
   :::column span="":::
      повернутый-Grid-полосковые полосы
   :::column-end:::
   :::column span="":::
      Заливка x
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок сетки — линии](./media/image-templates/grid-lines.png)
   :::column-end:::
   :::column span="":::
      ![значок "повернутый-Grid-Line"](./media/image-templates/rotated-grid-lines.png)
   :::column-end:::
   :::column span="":::
      ![значок "повернутый-Grid-полоск"](./media/image-templates/rotated-grid-stripes.png)
   :::column-end:::
   :::column span="":::
      ![значок "x-Fill"](./media/image-templates/x-fill.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      зигзаг — зигзаг
   :::column-end:::
   :::column span="":::
      зигзаг-зигзаг-по вертикали
   :::column-end:::
   :::column span="":::
      количество
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![значок зигзаг-зигзаг](./media/image-templates/zig-zag.png)
   :::column-end:::
   :::column span="":::
      ![значок зигзаг-зигзаг-Vertical](./media/image-templates/zig-zag-vertical.png)
   :::column-end:::
   :::column span="":::
      ![значок точек](./media/image-templates/dots.png)
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::
<br>

**Значки предварительно загруженных изображений**

Карта предварительно загружает набор значков в спрайт изображения Maps с помощью `marker` `pin` шаблонов, и `pin-round` . Эти имена значков и их цветовые значения перечислены в таблице ниже.

| имя значка | color | секондариколор |
|-----------|-------|----------------|
| `marker-black` | `#231f20` | `#ffffff` |
| `marker-blue` | `#1a73aa` | `#ffffff` |
| `marker-darkblue` | `#003963` | `#ffffff` |
| `marker-red` | `#ef4c4c` | `#ffffff` |
| `marker-yellow` | `#f2c851` | `#ffffff` |
| `pin-blue` | `#2072b8` | `#ffffff` |
| `pin-darkblue` | `#003963` | `#ffffff` |
| `pin-red` | `#ef4c4c` | `#ffffff` |
| `pin-round-blue` | `#2072b8` | `#ffffff` |
| `pin-round-darkblue` | `#003963` | `#ffffff` |
| `pin-round-red` | `#ef4c4c` | `#ffffff` |


## <a name="try-it-now-tool"></a>Попробуйте это средство

С помощью следующего средства можно отобразить различные встроенные шаблоны изображений различными способами и настроить основной и дополнительный цвета и масштаб.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Параметры шаблона значка" src="//codepen.io/azuremaps/embed/NQyaaO/?height=500&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Сведения о <a href='https://codepen.io/azuremaps/pen/NQyaaO/'>параметрах шаблона значок</a> пера см. Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [имажеспритеманажер](/javascript/api/azure-maps-control/atlas.imagespritemanager)

> [!div class="nextstepaction"]
> [пространство имен Atlas](/javascript/api/azure-maps-control/atlas#functions
)

Дополнительные примеры кода, в которых можно использовать шаблоны изображений, см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление слоя символов](map-add-pin.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](map-add-shape.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков на карту](map-add-bubble-layer.md)
---
title: Добавление всплывающего окна в точку на карте | Карты Microsoft Azure
description: Сведения о всплывающих окнах, шаблонах всплывающих окон и событиях всплывающих окон в Azure Maps. Узнайте, как добавить всплывающее окно в точку на карте и как повторно использовать и настраивать всплывающие окна.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 903574a8c8696f10d9903a685bab8d12b3e6395f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92891042"
---
# <a name="add-a-popup-to-the-map"></a>Добавление всплывающего окна на карту

В этой статье объясняется, как добавить всплывающее окно в точку на карте.

## <a name="understand-the-code"></a>Изучение кода

Следующий код добавляет функцию Point, которая имеет `name` `description` Свойства и, в карту с помощью слоя символов. Экземпляр [класса Popup](/javascript/api/azure-maps-control/atlas.popup) создан, но не отображается. События мыши добавляются на слой символов для активации открытия и закрытия всплывающего окна. При наведении указателя мыши свойство всплывающего окна `position` обновляется с указанием позиции маркера, а `content` параметр обновляется с помощью HTML-кода, который заключает в оболочку  `name` Свойства и `description` функции Point, на которую наведен указатель мыши. Затем всплывающее окно отображается на карте с помощью его `open` функции.

```javascript
//Define an HTML template for a custom popup content laypout.
var popupTemplate = '<div class="customInfobox"><div class="name">{name}</div>{description}</div>';

//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

dataSource.add(new atlas.data.Feature(new atlas.data.Point([-122.1333, 47.63]), {
  name: 'Microsoft Building 41', 
  description: '15571 NE 31st St, Redmond, WA 98052'
}));

//Create a layer to render point data.
var symbolLayer = new atlas.layer.SymbolLayer(dataSource);

//Add the polygon and line the symbol layer to the map.
map.layers.add(symbolLayer);

//Create a popup but leave it closed so we can update it and display it later.
popup = new atlas.Popup({
  pixelOffset: [0, -18],
  closeButton: false
});

//Add a hover event to the symbol layer.
map.events.add('mouseover', symbolLayer, function (e) {
  //Make sure that the point exists.
  if (e.shapes && e.shapes.length > 0) {
    var content, coordinate;
    var properties = e.shapes[0].getProperties();
    content = popupTemplate.replace(/{name}/g, properties.name).replace(/{description}/g, properties.description);
    coordinate = e.shapes[0].getCoordinates();

    popup.setOptions({
      //Update the content of the popup.
      content: content,

      //Update the popup's position with the symbol's coordinate.
      position: coordinate

    });
    //Open the popup.
    popup.open(map);
  }
});

map.events.add('mouseleave', symbolLayer, function (){
  popup.close();
});
```

Ниже приведен полный и выполняемый пример этой функциональности.

<br/>

<iframe height='500' scrolling='no' title='Добавление всплывающего окна с помощью Azure Maps' src='//codepen.io/azuremaps/embed/MPRPvz/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/MPRPvz/'>Добавление всплывающего окна с помощью Azure Maps</a> (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="reusing-a-popup-with-multiple-points"></a>Повторное использование всплывающего окна с несколькими точками

В некоторых случаях лучшим подходом является создание одного всплывающего окна и его повторное использование. Например, у вас может быть большое количество точек, и в каждый момент времени нужно отображать только одно всплывающее окно. При повторном использовании всплывающего окна количество элементов DOM, созданных приложением, значительно сокращается, что может повысить производительность. В следующем примере создаются 3-точечные функции. Если щелкнуть любую из них, отобразится всплывающее окно с содержимым этой функции точки.

<br/>

<iframe height='500' scrolling='no' title='Повторное использование всплывающего окна с несколькими закреплениями' src='//codepen.io/azuremaps/embed/rQbjvK/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода для <a href='https://codepen.io/azuremaps/pen/rQbjvK/'>повторного использования всплывающего окна с несколькими закреплениями</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customizing-a-popup"></a>Настройка всплывающего окна

По умолчанию всплывающее окно имеет белый фон, стрелку указателя внизу и кнопку Закрыть в правом верхнем углу. В следующем примере цвет фона изменяется на черный с помощью `fillColor` параметра всплывающего окна. Кнопка Закрыть удаляется путем присвоения `CloseButton` параметру значения false. HTML-содержимое всплывающего окна использует отступ от 10 пикселей от края всплывающего окна. Текст становится белым, поэтому он хорошо отображается на черном фоне.  

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Настраиваемое всплывающее окно" src="//codepen.io/azuremaps/embed/ymKgdg/?height=500&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. <a href='https://codepen.io/azuremaps/pen/ymKgdg/'>всплывающее окно, настроенное</a> пером, Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-popup-templates-to-the-map"></a>Добавление всплывающих шаблонов на карту

Всплывающие Шаблоны упрощают создание макетов, управляемых данными, для всплывающих окон. В следующих разделах показано использование различных шаблонов всплывающих окон для создания форматированного содержимого с помощью свойств функций.

> [!NOTE]
> По умолчанию все содержимое, которое отображается, использует шаблон всплывающего окна, изолированный внутри iframe в качестве функции безопасности. Однако существуют ограничения.
>
> - Все скрипты, формы, функции блокировки указателей и навигации с верхней навигацией отключены. Ссылки могут открываться на новой вкладке при щелчке. 
> - Более старые браузеры, которые не поддерживают `srcdoc` параметр в iframe, будут ограничены визуализацией небольшого объема содержимого.
> 
> Если вы доверяете данным, загружаемым в всплывающие окна, и потенциально хотите, чтобы эти скрипты, загруженные в всплывающие окна, могли получить доступ к приложению, вы можете отключить это, установив для параметра Popup Templates значение `sandboxContent` false. 

### <a name="string-template"></a>Строковый шаблон

Шаблон строки заменяет заполнители значениями свойств компонента. Свойствам компонента не обязательно присваивать значение типа String. Например, `value1` содержит целое число. Затем эти значения передаются в свойство Content объекта `popupTemplate` . 

`numberFormat`Параметр задает формат отображаемого числа. Если параметр `numberFormat` не указан, то код будет использовать формат даты всплывающего шаблона. `numberFormat`Параметр форматирует числа с помощью функции [Number. toLocaleString](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString) . Для форматирования больших чисел рекомендуется использовать `numberFormat` параметр с функциями из [NumberFormat. Format](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat/format). Например, в приведенном ниже фрагменте кода используется `maximumFractionDigits` для ограничения числа десятичных разрядов до двух.

> [!Note]
> Существует только один способ, в котором шаблон строки может отображать изображения. Сначала строковый шаблон должен содержать тег Image. Значение, передаваемое в тег Image, должно быть URL-адресом изображения. Затем шаблон строки должен иметь значение `isImage` true в `HyperLinkFormatOptions` . `isImage`Параметр указывает, что гиперссылка предназначена для изображения, и Гиперссылка будет загружена в тег Image. При щелчке гиперссылки откроется изображение.

```javascript
var templateOptions = {
  content: 'This template uses a string template with placeholders.<br/><br/> - Value 1 = {value1}<br/> - Value 2 = {value2/subValue}<br/> - Array value [2] = {arrayValue/2}',
  numberFormat: {
    maximumFractionDigits: 2
  }
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 1 - String template',
    value1: 1.2345678,
    value2: {
        subValue: 'Pizza'
    },
    arrayValue: [3, 4, 5, 6]
});

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="propertyinfo-template"></a>PropertyInfo, шаблон

Шаблон PropertyInfo отображает доступные свойства компонента. `label`Параметр задает текст, отображаемый для пользователя. Если параметр `label` не указан, будет отображаться гиперссылка. Если гиперссылка является изображением, будет отображено значение, присвоенное тегу "Alt". `dateFormat`Задает формат даты, и если формат даты не указан, то дата будет отображена в виде строки. `hyperlinkFormat`Параметр отображает ссылки, которые можно щелкать, так же как и `email` при отображении просматриваемых адресов электронной почты.

Перед тем как шаблон PropertyInfo отобразит свойства для конечного пользователя, он рекурсивно проверяет, что для этой функции действительно определены свойства. Он также игнорирует отображение свойств Style и Title. Например, он не будет отображать `color` , `size` , `anchor` , `strokeOpacity` и `visibility` . Таким образом, после завершения проверки пути свойства в серверной части шаблон PropertyInfo отображает содержимое в табличном формате.

```javascript
var templateOptions = {
  content: [
    {
        propertyPath: 'createDate',
        label: 'Created Date'
    },
    {
        propertyPath: 'dateNumber',
        label: 'Formatted date from number',
        dateFormat: {
          weekday: 'long',
          year: 'numeric',
          month: 'long',
          day: 'numeric',
          timeZone: 'UTC',
          timeZoneName: 'short'
        }
    },
    {
        propertyPath: 'url',
        label: 'Code samples',
        hideLabel: true,
        hyperlinkFormat: {
          lable: 'Go to code samples!',
          target: '_blank'
        }
    },
    {
        propertyPath: 'email',
        label: 'Email us',
        hideLabel: true,
        hyperlinkFormat: {
          target: '_blank',
          scheme: 'mailto:'
        }
    }
  ]
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 2 - PropertyInfo',
    createDate: new Date(),
    dateNumber: 1569880860542,
    url: 'https://aka.ms/AzureMapsSamples',
    email: 'info@microsoft.com'
}),

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="multiple-content-templates"></a>Несколько шаблонов содержимого

Функция также может отображать содержимое, используя сочетание шаблона строк и шаблона PropertyInfo. В этом случае шаблон строки подготавливает значения заполнителей на белом фоне.  И шаблон PropertyInfo визуализирует изображение с полной шириной внутри таблицы. Свойства в этом образце похожи на свойства, описанные в предыдущих примерах.

```javascript
var templateOptions = {
  content: [
    'This template has two pieces of content; a string template with placeholders and a array of property info which renders a full width image.<br/><br/> - Value 1 = {value1}<br/> - Value 2 = {value2/subValue}<br/> - Array value [2] = {arrayValue/2}',
    [{
      propertyPath: 'imageLink',
      label: 'Image',
      hideImageLabel: true,
      hyperlinkFormat: {
        isImage: true
      }
    }]
  ],
  numberFormat: {
    maximumFractionDigits: 2
  }
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 3 - Multiple content template',
    value1: 1.2345678,
    value2: {
    subValue: 'Pizza'
    },
    arrayValue: [3, 4, 5, 6],
    imageLink: 'https://azuremapscodesamples.azurewebsites.net/common/images/Pike_Market.jpg'
});

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="points-without-a-defined-template"></a>Точки без определенного шаблона

Если шаблон всплывающего окна не определен как строковый шаблон, шаблон PropertyInfo или сочетание обоих параметров, то используются параметры по умолчанию. Когда `title` Свойства и `description` являются единственными назначенными свойствами, в шаблоне всплывающего окна отображается белый фон, кнопка закрытия в правом верхнем углу. А на небольших и средних экранах в нижней части отображается стрелка. Параметры по умолчанию отображают внутри таблицы все свойства, кроме `title` и `description` . Даже при возврате к значениям по умолчанию шаблон Popup по-прежнему может управляться программно. Например, пользователи могут отключить обнаружение гиперссылок, и параметры по умолчанию будут применяться к другим свойствам.

Щелкните точки на карте в CodePen. Существует точка на карте для каждого из следующих контекстных шаблонов: строковый шаблон, шаблон PropertyInfo и несколько шаблонов содержимого. Также есть три пункта, демонстрирующие отрисовку шаблонов с использованием параметров по умолчанию.

<br/>

<iframe height='500' scrolling='no' title='попуптемплатес' src='//codepen.io/azuremaps/embed/dyovrzL/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. <a href='https://codepen.io/azuremaps/pen/dyovrzL/'>Попуптемплатес</a> пера by Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="reuse-popup-template"></a>Повторное использование шаблона всплывающего окна

Как и при повторном использовании всплывающего окна, можно многократно использовать всплывающие шаблоны. Этот подход удобен, если для нескольких точек требуется одновременно отобразить только один шаблон всплывающего окна. При повторном использовании шаблона всплывающего окна количество элементов DOM, созданных приложением, уменьшается, что повышает производительность приложения. В следующем примере используется один и тот же шаблон всплывающего окна для трех точек. Если щелкнуть любую из них, отобразится всплывающее окно с содержимым этой функции точки.

<br/>

<iframe height='500' scrolling='no' title='реусепопуптемплате' src='//codepen.io/azuremaps/embed/WNvjxGw/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. <a href='https://codepen.io/azuremaps/pen/WNvjxGw/'>Реусепопуптемплате</a> пера by Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="popup-events"></a>События всплывающего окна

Всплывающие окна можно открывать, закрывать и перетаскивать. Класс Popup предоставляет события, помогающие разработчикам реагировать на эти события. В следующем примере показано, какие события срабатывают, когда пользователь открывает, закрывает или перетаскивает всплывающее окно. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="События всплывающего окна" src="//codepen.io/azuremaps/embed/BXrpvB/?height=500&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
См. <a href='https://codepen.io/azuremaps/pen/BXrpvB/'>события всплывающего окна</a> пера по Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Popup](/javascript/api/azure-maps-control/atlas.popup)

> [!div class="nextstepaction"]
> [PopupOptions](/javascript/api/azure-maps-control/atlas.popupoptions)

> [!div class="nextstepaction"]
> [попуптемплате](/javascript/api/azure-maps-control/atlas.popuptemplate)

Полные примеры кода см. в следующих превосходных статьях:

> [!div class="nextstepaction"]
> [Добавление слоя символов](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Добавление маркера HTML](./map-add-custom-html.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](map-add-shape.md)
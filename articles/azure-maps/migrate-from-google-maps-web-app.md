---
title: Учебник. Перенос веб-приложения из Google Карт в Microsoft Azure Maps
description: В руководстве показано, как перенести веб-приложение из Google Карт в Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: devx-track-js
ms.openlocfilehash: fcb8090427530271600a6699fafa5c488c426784
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97680878"
---
# <a name="tutorial-migrate-a-web-app-from-google-maps"></a>Руководство по Перенос веб-приложения из Google Карт

Большинство веб-приложений, работающих с Google Картами, использует пакет SDK JavaScript для Google Карт версии 3. Подходящий пакет SDK на основе Azure для переноса — веб-пакет SDK для Azure Maps. Веб-пакет SDK Azure Maps позволяет настраивать интерактивные карты, используя собственные изображения и содержимое. Приложение можно запускать как в веб-, так и в мобильных приложениях. Этот элемент управления использует WebGL, позволяя вам отображать большие наборы данных с высокой производительностью. Вы можете выполнять разработку с помощью пакета SDK, используя JavaScript или TypeScript. В этом учебнике рассматривается следующее.

> [!div class="checklist"]
> * Загрузка карты
> * локализация карты;
> * добавление маркеров, ломаных линий и многоугольников;
> * отображение сведений во всплывающем окне или окне сведений;
> * загрузка и отображение данных KML и GeoJSON;
> * маркеры кластера;
> * Наложение слоя фрагментов
> * Отображение данных о трафике
> * Добавление наземного наложения

Кроме того, вы:

> [!div class="checklist"]
> * научитесь выполнять распространенные задачи сопоставления с помощью веб-пакета SDK для Azure Maps;
> * получите рекомендации по повышению производительности и удобства работы пользователей;
> * получите советы по созданию в приложении дополнительных функций, доступных в Azure Maps.

При переносе имеющегося веб-приложения проверьте, использует ли оно библиотеку элементов управления картой с открытым кодом. Примерами такой библиотеки могут быть Cesium, Leaflet или OpenLayers. Вы все равно можете выполнить перенос, если ваше приложение использует библиотеку элементов управления картой с открытым кодом и вы не хотите использовать веб-пакет SDK Azure Maps. В таком случае подключите приложение к службам фрагментов Azure Maps ([фрагменты карты дорог](/rest/api/maps/render/getmaptile) \| [фрагменты спутниковой карты](/rest/api/maps/render/getmapimagerytile)). Ниже приведены сведения о применении Azure Maps в некоторых часто используемых библиотеках элементов управления картой с открытым кодом.

* Cesium — элемент управления трехмерными картами для Интернета. [Пример кода](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Raster%20Tiles%20in%20Cesium%20JS) \| [документация](https://cesiumjs.org/)
* Leaflet — упрощенный элемент управления двухмерными картами для Интернета. [Пример кода](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Azure%20Maps%20Raster%20Tiles%20in%20Leaflet%20JS) \| [документация](https://leafletjs.com/)
* OpenLayers — элемент управления двухмерными картами для Интернета, который поддерживает проекции. [Пример кода](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Raster%20Tiles%20in%20OpenLayers) \| [документация](https://openlayers.org/)

Для разработки с использованием платформы JavaScript может оказаться полезным один из следующих проектов с открытым исходным кодом:

* [ng-azure-maps](https://github.com/arnaudleclerc/ng-azure-maps) — это программа-оболочка Angular 10 для Azure Maps.
* [AzureMapsControl.Components](https://github.com/arnaudleclerc/AzureMapsControl.Components) — компонент Azure Maps Blazor.
* [Компонент React Azure Maps](https://github.com/WiredSolutions/react-azure-maps) — программа-оболочка React для управления Azure Maps.
* [Vue Azure Maps](https://github.com/rickyruiz/vue-azure-maps) — компонент Azure Maps для приложения Vue.

## <a name="prerequisites"></a>Предварительные требования

1. Войдите на [портал Azure](https://portal.azure.com). Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
2. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](how-to-manage-authentication.md).

## <a name="key-features-support"></a>Поддержка основных функций

В таблице перечислены основные функции API, доступные в пакете SDK JavaScript для Google Карт версии 3, и сведения о поддержке функций API в веб-пакете SDK Azure Maps.

| Функция Google Карт     | Поддержка в веб-пакете SDK для Azure Maps |
|-------------------------|:--------------------------:|
| Метки                 | ✓                          |
| Кластеризация меток       | ✓                          |
| Ломаные линии и многоугольники    | ✓                          |
| Уровни данных             | ✓                          |
| Наземное наложение         | ✓                          |
| Тепловые карты               | ✓                          |
| Слои фрагментов             | ✓                          |
| Слой KML               | ✓                          |
| Инструменты для рисования           | ✓                          |
| Служба геокодирования        | ✓                          |
| Служба направлений      | ✓                          |
| Служба матрицы расстояний | ✓                          |
| Служба высоты       | ✓                          |

## <a name="notable-differences-in-the-web-sdks"></a>Важные различия в веб-пакетах SDK

Ниже приведены некоторые ключевые различия между веб-пакетами SDK для Google Карт и Azure Maps, о которых следует знать.

- Помимо предоставления размещенной конечной точки для доступа к веб-пакету SDК Azure Maps, можно также использовать пакет NPM. Вы можете внедрить веб-пакет SDK в приложения. Дополнительные сведения см. в этой [документации](how-to-use-map-control.md). Этот пакет также включает определения TypeScript.
- Сначала необходимо создать экземпляр класса Map в Azure Maps. Дождитесь срабатывания события карты `ready` или `load`, прежде чем программно взаимодействовать с картой. Этот порядок событий обеспечивает, что все ресурсы карты будут загружены и готовы к доступу.
- Обе платформы используют аналогичную систему разбиения для базовых карт. Размер фрагментов в Google Картах Maps составляет 256 пикселей, а в Azure Maps — 512 пикселей. Чтобы получить тот же вид карты в Azure Maps, что и в Google Картах, из коэффициента масштабирования, используемого в Google Картах, в Azure Maps нужно вычесть единицу.
- В Google Картах координаты используются в формате `latitude,longitude`, а в Azure Maps — в формате `longitude,latitude`. Формат Azure Maps соответствует стандартным координатам `[x, y]`, которые используются на большинстве платформ ГИС.
- Фигуры в веб-пакете SDK для Azure Maps основаны на схеме GeoJSON. Вспомогательные классы предоставляются с помощью [пространства имен](/javascript/api/azure-maps-control/atlas.data) *atlas.data*. Существует также класс [*atlas.Shape*](/javascript/api/azure-maps-control/atlas.shape). Его можно использовать для упаковки объектов GeoJSON, чтобы упростить их обновление и использовать для привязки данных.
- Координаты в Azure Maps определяются как объекты положения. Координаты указываются как массив чисел в формате `[longitude,latitude]`. Или они указываются с помощью new atlas.data.Position(долгота, широта).
    > [!TIP]
    > Класс положения имеет статический вспомогательный метод для импорта координат, которые находятся в формате "широта, долгота". Метод [atlas.data.Position.fromLatLng](/javascript/api/azure-maps-control/atlas.data.position) часто можно заменить методом `new google.maps.LatLng` в коде Google Карт.
- Вместо того чтобы указывать сведения о стилях для каждой фигуры, добавляемой на карту, Azure Maps отделяет стили от данных. Данные хранятся в источнике данных и подключены к слоям отрисовки. Код Azure Maps использует источники данных для визуализации данных. Такой подход обеспечивает повышение производительности. Кроме того, многие слои поддерживают управляемый данными стиль, где к параметрам стиля слоя можно добавить бизнес-логику. Такая поддержка изменяет способ преобразования отдельных фигур для просмотра в слое на основе свойств, определенных в фигуре.

## <a name="web-sdk-side-by-side-examples"></a>Параллельные примеры веб-пакетов SDK

Ниже приведена коллекция примеров кода для каждой платформы, охватывающих распространенные варианты использования. Они помогут перенести веб-приложение из пакета SDK JavaScript для Google Карт версии 3 в веб-пакет SDK для Azure Maps. Примеры кода, связанные с веб-приложениями, предоставляются на языке JavaScript. Но Azure Maps также предоставляет определения TypeScript в качестве дополнительной возможности с использованием модуля [NPM](how-to-use-map-control.md).

**Разделы**

* [Загрузка карты](#load-a-map)
* [Локализация карты](#localizing-the-map)
* [Настройка представления карты](#setting-the-map-view)
* [Добавление метки](#adding-a-marker)
* [Добавление настраиваемой метки](#adding-a-custom-marker)
* [Добавление ломаной линии](#adding-a-polyline)
* [Добавление многоугольника](#adding-a-polygon)
* [Отображение окна сведений](#display-an-info-window)
* [Импорт файла GeoJSON](#import-a-geojson-file)* 
* [Кластеризация меток](#marker-clustering)
* [Добавление тепловой карты](#add-a-heat-map)
* [Наложение слоя фрагментов](#overlay-a-tile-layer)
* [Отображение данных о трафике](#show-traffic-data)
* [Добавление наземного наложения](#add-a-ground-overlay)
* [Добавление данных KML на карту](#add-kml-data-to-the-map)

### <a name="load-a-map"></a>Загрузка карты

Оба пакета SDK используют одинаковые шаги для загрузки схемы.

* Добавьте ссылку на пакет SDK для карт.
* Добавьте в текст страницы тег `div`. Он будет служить заполнителем для карты.
* Создайте функцию JavaScript, которая вызывается после загрузки страницы.
* Создайте экземпляр соответствующего класса Map.

**Некоторые ключевые отличия**

* В Google Картах необходимо указать ключ учетной записи в ссылке на скрипт API. Учетные данные проверки подлинности для Azure Maps указываются в качестве параметров класса Map. Это может быть ключ подписки или сведения Azure Active Directory.
* Google Карты принимают функцию обратного вызова в ссылке на сценарий API, которая используется для вызова функции инициализации для загрузки карты. При работе с Azure Maps следует использовать событие OnLoad страницы.
* При указании ссылки на элемент `div`, в котором будет выполняться визуализация карты, классу `Map` в Azure Maps требуется только значение `id`, тогда как для Google Карт требуется объект `HTMLElement`.
* Координаты в Azure Maps определяются как объекты положения, которые можно указать как простой массив чисел в формате `[longitude, latitude]`.
* Уровень масштаба в Azure Maps на один меньше, чем уровень масштаба в Google Карт. Это несоответствие обусловлено разницей в размерах системы разбиения между платформами.
* Azure Maps не добавляет на холст карты элементы управления навигацией. Поэтому по умолчанию на карте отсутствуют кнопки масштабирования и кнопки стилей карт. Однако на ней доступны элементы управления для добавления средств выбора стиля карт, кнопок масштабирования, компаса или поворота, а также средства управления наклоном.
* Обработчик событий добавляется в Azure Maps для отслеживания события `ready` экземпляра карты. Это событие происходит, когда завершается загрузка контекста WebGL и всех необходимых ресурсов на карте. Добавьте в этот обработчик событий любой код, который требуется выполнить после завершения загрузки карты.

В простых примерах, приведенных ниже, Google Карты используются для загрузки карты с координатами Нью-Йорка в центре. Долгота -73,985, широта 40,747, а коэффициент масштабирования карты — 12.

#### <a name="before-google-maps"></a>До: Google Карты

Отображение Google Карт с увеличенным расположением по центру.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(40.747, -73.985),
                zoom: 12
            });
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

При выполнении этого кода в браузере отобразится показанная ниже карта:

![Простые Google Карты](media/migrate-google-maps-web-app/simple-google-map.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Загрузка карты с тем же видом в Azure Maps, а также с элементами управления стилем карты и кнопками масштабирования.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;

        function initMap() {
            map = new atlas.Map('myMap', {
                center: [-73.985, 40.747],  //Format coordinates as longitude, latitude.
                zoom: 11,   //Subtract the zoom level by one.

                //Specify authentication information when loading the map.
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {
                //Add zoom controls to bottom right of map.
                map.controls.add(new atlas.control.ZoomControl(), {
                    position: 'bottom-right'
                });

                //Add map style control in top left corner of map.
                map.controls.add(new atlas.control.StyleControl(), {
                    position: 'top-left'
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

При выполнении этого кода в браузере отобразится показанная ниже карта:

![Простые службы Azure Maps](media/migrate-google-maps-web-app/simple-azure-maps.png)

Подробную документацию по настройке и использованию элементов управления картой Azure Maps в веб-приложении можно найти [здесь](how-to-use-map-control.md).

> [!NOTE]
> В отличие от Google Карт, для Azure Maps не требуется указывать изначальный центр и коэффициент масштабирования при загрузке карты. Если эти сведения не указаны при загрузке карты, Azure Maps попытается определить город пользователя. Он будет помещен в центре карты и увеличен.

**Дополнительные ресурсы:**

* Azure Maps также предоставляет элементы управления навигацией для поворота и наклона представления карты, как описано [здесь](map-add-controls.md).

### <a name="localizing-the-map"></a>Локализация карты

Если ваша аудитория распределена по нескольким странам или регионам либо говорит на разных языках, необходима локализация.

#### <a name="before-google-maps"></a>До: Google Карты

Чтобы локализовать Google Карты, необходимо добавить параметры языка и региона.

```html
<script type="text/javascript" src=" https://maps.googleapis.com/maps/api/js?callback=initMap&key=[api_key]& language=[language_code]&region=[region_code]" async defer></script>
```

Ниже приведен пример использования Google Карт с языком, имеющим значение fr-FR.

![Локализация Google Карт](media/migrate-google-maps-web-app/google-maps-localization.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Azure Maps предоставляет два разных способа настройки языка и регионального представления карты. Первый вариант — добавить эту информацию в глобальное пространство имен *atlas*. Это приведет к тому, что все экземпляры элементов управления картой в приложении будут сброшены к этим параметрам. В приведенном ниже примере кода устанавливается французский язык (fr-FR) и региональное представление со значением Auto:

```javascript
atlas.setLanguage('fr-FR');
atlas.setView('auto');
```

Второй вариант — передать эту информацию в параметры карты при ее загрузке следующим образом. Пример:

```javascript
map = new atlas.Map('myMap', {
    language: 'fr-FR',
    view: 'auto',

    authOptions: {
        authType: 'subscriptionKey',
        subscriptionKey: '<Your Azure Maps Key>'
    }
});
```

> [!NOTE]
> В Azure Maps можно загрузить несколько экземпляров карты на одной странице с разными настройками языка и региональными параметрами. Кроме того, можно обновить эти параметры для карты после ее загрузки.

Подробный список поддерживаемых языков в Azure Maps можно найти [здесь](supported-languages.md).

Ниже приведен пример Azure Maps с установленным языком fr и регионом пользователя fr-FR.

![Локализация Azure Maps](media/migrate-google-maps-web-app/azure-maps-localization.png)

### <a name="setting-the-map-view"></a>Настройка представления карты

Динамические карты в Azure Maps и Google Картах можно программно перемещать в новые географические расположения. Для этого вызовите соответствующие функции на JavaScript. В примерах ниже показано, как сделать так, чтобы на карте отображались спутниковые аэроснимки, как центрировать карту по расположению и изменить уровень масштабирования на 15 в Google Картах. Используются следующие координаты расположения: долгота — −111,0225 и широта — 35,0272.

> [!NOTE]
> В Google Картах используются фрагменты размером 256 пикселей, а в Azure Maps — более крупные фрагменты размером 512 пикселей. Благодаря этому сокращается количество сетевых запросов, необходимых Azure Maps для загрузки той же области карты, что и в Google Картах. Ввиду того, как пирамиды фрагментов работают с элементами управления картой, при использовании Azure Maps необходимо вычесть единицу из коэффициента масштабирования, используемого в Google Картах. Эта арифметическая операция гарантирует, что более крупные фрагменты в Azure Maps будут отображать ту же область карты, что и в Google Картах.

#### <a name="before-google-maps"></a>До: Google Карты

Перемещение элемента управления картой Google Карт с помощью метода `setOptions`. Этот метод позволяет указать центр карты и уровень масштабирования.

```javascript
map.setOptions({
    mapTypeId: google.maps.MapTypeId.SATELLITE,
    center: new google.maps.LatLng(35.0272, -111.0225),
    zoom: 15
});
```

![Заданное представление Google Карт](media/migrate-google-maps-web-app/google-maps-set-view.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

В Azure Maps изменить расположение на карте можно с помощью метода `setCamera`, а стиль карты — с помощью метода `setStyle`. Координаты в Azure Maps указываются в формате "долгота, широта", а значение уровня масштаба вычитается на единицу.

```javascript
map.setCamera({
    center: [-111.0225, 35.0272],
    zoom: 14
});

map.setStyle({
    style: 'satellite'
});
```

![Заданное представление Azure Maps](media/migrate-google-maps-web-app/azure-maps-set-view.jpeg)

**Дополнительные ресурсы:**

* [Выбор стиля карты](choose-map-style.md)
* [Поддерживаемые стили карт](supported-map-styles.md)

### <a name="adding-a-marker"></a>Добавление метки

В Azure Maps существует несколько способов, с помощью которых данные точек можно визуализировать на карте.

* **Метки HTML** — отрисовывают точки с помощью традиционных элементов DOM. Метки HTML поддерживают перетаскивание.
* **Слой символов** — отрисовывает точки со значком или текстом в контексте WebGL.
* **Пузырьковый слой** — отрисовывает точки в виде окружностей на карте. Радиусы окружностей можно масштабировать на основе свойств данных.

Слои символов и пузырьков отображаются в контексте WebGL. Оба слоя могут отображать большие наборы точек на карте. Данные для этих слоев должны храниться в источнике данных. Источники данных и слои отрисовки необходимо добавлять на карту после запуска события `ready`. Метки HTML отображаются как элементы DOM на странице и не используют источник данных. Чем больше элементов DOM содержит страница, тем медленнее она будет обрабатываться. При отрисовке более чем нескольких сотен точек на карте рекомендуется использовать один из слоев отрисовки.

Давайте добавим на карту маркер с номером 10, наложенным как метка. Используйте долготу –0,2 и широту 51,5.

#### <a name="before-google-maps"></a>До: Google Карты

При использовании Google Карт метки добавляются на карту с помощью класса `google.maps.Marker` и путем указания карты в качестве одного из параметров.

```javascript
//Create a marker and add it to the map.
var marker = new google.maps.Marker({
    position: new google.maps.LatLng(51.5, -0.2),
    label: '10',
    map: map
});
```

![Метка Google Карт](media/migrate-google-maps-web-app/google-maps-marker.png)

**После: использование меток HTML в Azure Maps**

В Azure Maps метки HTML можно использовать для отображения точки на карте. Их рекомендуется применять для простых приложений, в которых необходимо показывать небольшое количество точек на карте. Чтобы использовать метку HTML, создайте экземпляр класса `atlas.HtmlMarker`. Задайте параметры текста и расположения, затем добавьте метку на карту с помощью метода `map.markers.add`.

```javascript
//Create a HTML marker and add it to the map.
map.markers.add(new atlas.HtmlMarker({
    text: '10',
    position: [-0.2, 51.5]
}));
```

![Метка HTML Azure Maps](media/migrate-google-maps-web-app/azure-maps-html-marker.png)

**После: использование слоя символов в Azure Maps**

Добавьте данные для слоя символов в источник данных. Подключите этот источник данных к слою. Кроме того, источник данных и слой нужно добавить на карту после запуска события `ready`. Чтобы отобразить уникальное текстовое значение над символом, текстовая информация должна храниться как свойство точки данных. На это свойство должен ссылаться параметр `textField` слоя. Этот способ немного сложнее, чем использование меток HTML, но он повышает производительность.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map, datasource;

        function initMap() {
            map = new atlas.Map('myMap', {
                center: [-0.2, 51.5],
                zoom: 9,
                
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Create a point feature, add a property to store a label for it, and add it to the data source.
                datasource.add(new atlas.data.Feature(new atlas.data.Point([-0.2, 51.5]), {
                    label: '10'
                }));

                //Add a layer for rendering point data as symbols.
                map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
                    textOptions: {
                        //Use the label property to populate the text for the symbols.
                        textField: ['get', 'label'],
                        color: 'white',
                        offset: [0, -1]
                    }
                }));
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Слой символов Azure Maps](media/migrate-google-maps-web-app/azure-maps-symbol-layer.png)

**Дополнительные ресурсы:**

- [Создание источника данных](create-data-source-web-sdk.md)
- [Добавление слоя символов](map-add-pin.md)
- [Добавление слоя пузырьков](map-add-bubble-layer.md)
- [Данные точек кластера](clustering-point-data-web-sdk.md)
- [Добавление меток HTML](map-add-custom-html.md)
- [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)
- [Параметры значков слоя символов](/javascript/api/azure-maps-control/atlas.iconoptions)
- [Параметры текста слоя символов](/javascript/api/azure-maps-control/atlas.textoptions)
- [Класс метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
- [Параметры метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-custom-marker"></a>Добавление настраиваемой метки

Для представления точек на карте можно использовать пользовательские изображения. В приведенной ниже карте пользовательское изображение применяется для отображения точки на карте. Координаты точки: широта — 51,5, долгота −0,2. Привязка смещает положение маркера таким образом, чтобы точка значка вешки была правильно размещена на карте.

<center>

![Изображение желтой вешки](media/migrate-google-maps-web-app/yellow-pushpin.png)<br/>
yellow-pushpin.png</center>

#### <a name="before-google-maps"></a>До: Google Карты

Создайте пользовательскую метку, указав объект `Icon`, содержащий `url`-адрес изображения. Укажите точку `anchor`, чтобы сопоставить точку изображения вешки с координатами на карте. Значение привязки в Google Картах связано относительно левого верхнего угла изображения.

```javascript
var marker = new google.maps.Marker({
    position: new google.maps.LatLng(51.5, -0.2),
    icon: {
        url: 'https://azuremapscodesamples.azurewebsites.net/Common/images/icons/ylw-pushpin.png',
        anchor: new google.maps.Point(5, 30)
    },
    map: map
});
```

![Пользовательская метка Google Карт](media/migrate-google-maps-web-app/google-maps-custom-marker.png)

**После: использование меток HTML в Azure Maps**

Чтобы настроить метку HTML, можно передать в параметр `htmlContent` метки объект HTML `string` или `HTMLElement`. Используйте параметр `anchor`, чтобы указать положение метки относительно координат положения. Присвойте параметру `anchor` одну из девяти определенных опорных точек. Это следующие значения: "center", "top", "bottom", "left", "right", "top-left", "top-right", "bottom-left", "bottom-right". Содержимое привязывается к нижнему центру содержимого HTML по умолчанию. Чтобы упростить перенос кода из Google Карт, установите для `anchor` значение top-left (вверху слева), а затем используйте параметр `pixelOffset` с тем же смещением, которое использовалось в Google Картах. Смещение в Azure Maps выполняется в противоположном направлении по отношению к Google Картам. Поэтому умножьте его значение на минус один.

> [!TIP]
> Добавьте `pointer-events:none` в качестве стиля в содержимое HTML, чтобы отключить поведение перетаскивания по умолчанию в Microsoft Edge, при котором будет отображаться нежелательный значок.

```javascript
map.markers.add(new atlas.HtmlMarker({
    htmlContent: '<img src="https://azuremapscodesamples.azurewebsites.net/Common/images/icons/ylw-pushpin.png" style="pointer-events: none;" />',
    anchor: 'top-left',
    pixelOffset: [-5, -30],
    position: [-0.2, 51.5]
}));
```

![Пользовательская метка HTML в Azure Maps](media/migrate-google-maps-web-app/azure-maps-custom-html-marker.png)

**После: использование слоя символов в Azure Maps**

Слои символов в Azure Maps поддерживают также пользовательские изображения. Сначала загрузите изображение в ресурсы карты и назначьте ему уникальный идентификатор. Добавьте ссылку на это изображение в слой символов. Укажите параметр `offset`, чтобы сопоставить изображение с соответствующей точкой на карте. Используйте параметр `anchor`, чтобы указать положение символа относительно координат положения. Используйте одну из девяти определенных опорных точек. Это следующие значения: "center", "top", "bottom", "left", "right", "top-left", "top-right", "bottom-left", "bottom-right". Содержимое привязывается к нижнему центру содержимого HTML по умолчанию. Чтобы упростить перенос кода из Google Карт, установите для `anchor` значение top-left (вверху слева), а затем используйте параметр `offset` с тем же смещением, которое использовалось в Google Картах. Смещение в Azure Maps выполняется в противоположном направлении по отношению к Google Картам. Поэтому умножьте его значение на минус один.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map, datasource;

        function initMap() {
            map = new atlas.Map('myMap', {
                center: [-0.2, 51.5],
                zoom: 9,
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Load the custom image icon into the map resources.
                map.imageSprite.add('my-yellow-pin', 'https://azuremapscodesamples.azurewebsites.net/Common/images/icons/ylw-pushpin.png').then(function () {

                    //Create a data source and add it to the map.
                    datasource = new atlas.source.DataSource();
                    map.sources.add(datasource);

                    //Create a point and add it to the data source.
                    datasource.add(new atlas.data.Point([-0.2, 51.5]));

                    //Add a layer for rendering point data as symbols.
                    map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
                        iconOptions: {
                            //Set the image option to the id of the custom icon that was loaded into the map resources.
                            image: 'my-yellow-pin',
                            anchor: 'top-left',
                            offset: [-5, -30]
                        }
                    }));
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Пользовательский слой значков Azure Maps](media/migrate-google-maps-web-app/azure-maps-custom-icon-symbol-layer.png)</

> [!TIP]
> Чтобы отобразить расширенные пользовательские точки, используйте одновременно несколько слоев отрисовки. Например, если требуется использовать несколько вешек с одинаковым значком на разных цветных окружностях. Вместо того чтобы создавать множество изображений для каждого цвета, наложите слой символов поверх слоя пузырьков. Вешки должны ссылаться на один и тот же источник данных. Это будет намного эффективнее, чем создавать и сохранять на карте множество различных изображений.

**Дополнительные ресурсы:**

- [Создание источника данных](create-data-source-web-sdk.md)
- [Добавление слоя символов](map-add-pin.md)
- [Добавление меток HTML](map-add-custom-html.md)
- [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)
- [Параметры значков слоя символов](/javascript/api/azure-maps-control/atlas.iconoptions)
- [Параметры текста слоя символов](/javascript/api/azure-maps-control/atlas.textoptions)
- [Класс метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
- [Параметры метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-polyline"></a>Добавление ломаной линии

Ломаные линии используются для представления линий или траекторий на карте. Давайте создадим на карте пунктирную ломаную линию.

#### <a name="before-google-maps"></a>До: Google Карты

Класс Polyline принимает набор параметров. Массив координат передается в параметре `path` ломаной линии.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Define a symbol using SVG path notation, with an opacity of 1.
var lineSymbol = {
    path: 'M 0,-1 0,1',
    strokeOpacity: 1,
    scale: 4
};

//Create the polyline.
var line = new google.maps.Polyline({
    path: [
        center,
        new google.maps.LatLng(center.lat() - 0.5, center.lng() - 1),
        new google.maps.LatLng(center.lat() - 0.5, center.lng() + 1)
    ],
    strokeColor: 'red',
    strokeOpacity: 0,
    strokeWeight: 4,
    icons: [{
        icon: lineSymbol,
        offset: '0',
        repeat: '20px'
    }]
});

//Add the polyline to the map.
line.setMap(map);
```

![Ломаная линия в Google Картах](media/migrate-google-maps-web-app/google-maps-polyline.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Ломаные линии называются объектами `LineString` или `MultiLineString`. Эти объекты можно добавлять в источник данных и отображать с помощью слоя линий. Добавьте `LineString` в источник данных, а затем добавьте этот источник данных в `LineLayer` для отображения.

```javascript
//Get the center of the map.
var center = map.getCamera().center;

//Create a data source and add it to the map.
var datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Create a line and add it to the data source.
datasource.add(new atlas.data.LineString([
    center,
    [center[0] - 1, center[1] - 0.5],
    [center[0] + 1, center[1] - 0.5]
]));

//Add a layer for rendering line data.
map.layers.add(new atlas.layer.LineLayer(datasource, null, {
    strokeColor: 'red',
    strokeWidth: 4,
    strokeDashArray: [3, 3]
}));
```

![Ломаная линия в Azure Maps](media/migrate-google-maps-web-app/azure-maps-polyline.png)

**Дополнительные ресурсы:**

- [Добавление линий на карту](map-add-line-layer.md)
- [Параметры слоя линий](/javascript/api/azure-maps-control/atlas.linelayeroptions)
- [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

### <a name="adding-a-polygon"></a>Добавление многоугольника

Azure Maps и Google Карты обеспечивают похожую поддержку многоугольников. Многоугольники используются для представления области на карте. В следующих примерах показано, как создать многоугольник, который образует треугольник на основе центральной координаты на карте.

#### <a name="before-google-maps"></a>До: Google Карты

Класс Polygon принимает набор параметров. Массив координат передается в параметре `paths` многоугольника.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Create a polygon.
var polygon = new google.maps.Polygon({
    paths: [
        center,
        new google.maps.LatLng(center.lat() - 0.5, center.lng() - 1),
        new google.maps.LatLng(center.lat() - 0.5, center.lng() + 1),
        center
    ],
    strokeColor: 'red',
    strokeWeight: 2,
    fillColor: 'rgba(0, 255, 0, 0.5)'
});

//Add the polygon to the map
polygon.setMap(map);
```

![Многоугольник в Google Картах](media/migrate-google-maps-web-app/google-maps-polygon.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

В источник данных можно добавить объект `Polygon` или `MultiPolygon`. Объект отрисовывается на карте с помощью слоев. Область многоугольника можно отобразить на слое многоугольников. Контур многоугольника можно визуализировать с помощью слоя линий.

```javascript
//Get the center of the map.
var center = map.getCamera().center;

//Create a data source and add it to the map.
datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Create a polygon and add it to the data source.
datasource.add(new atlas.data.Polygon([
    center,
    [center[0] - 1, center[1] - 0.5],
    [center[0] + 1, center[1] - 0.5],
    center
]));

//Add a polygon layer for rendering the polygon area.
map.layers.add(new atlas.layer.PolygonLayer(datasource, null, {
    fillColor: 'rgba(0, 255, 0, 0.5)'
}));

//Add a line layer for rendering the polygon outline.
map.layers.add(new atlas.layer.LineLayer(datasource, null, {
    strokeColor: 'red',
    strokeWidth: 2
}));
```

![Многоугольник в Azure Maps](media/migrate-google-maps-web-app/azure-maps-polygon.png)

**Дополнительные ресурсы:**

- [Добавление многоугольника на карту](map-add-shape.md)
- [Добавление окружности на карту](map-add-shape.md#add-a-circle-to-the-map)
- [Параметры слоя многоугольников](/javascript/api/azure-maps-control/atlas.polygonlayeroptions)
- [Параметры слоя линий](/javascript/api/azure-maps-control/atlas.linelayeroptions)
- [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

### <a name="display-an-info-window"></a>Отображение окна сведений

Дополнительные сведения для сущности можно показать на карте, используя класс `google.maps.InfoWindow` в Google Картах. В Azure Maps это можно сделать с помощью класса `atlas.Popup`. В следующих примерах на карту добавляется метка. При щелчке этой метки отображается окно сведений или всплывающий элемент.

#### <a name="before-google-maps"></a>До: Google Карты

Экземпляр окна сведений можно создать с помощью конструктора `google.maps.InfoWindow`.

```javascript
//Add a marker in which to display an infowindow for.
var marker = new google.maps.Marker({
    position: new google.maps.LatLng(47.6, -122.33),
    map: map
});

//Create an infowindow.
var infowindow = new google.maps.InfoWindow({
    content: '<div style="padding:5px"><b>Hello World!</b></div>'
});

//Add a click event to the marker to open the infowindow.
marker.addListener('click', function () {
    infowindow.open(map, marker);
});
```

![Всплывающее окно в Google Картах](media/migrate-google-maps-web-app/google-maps-popup.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Давайте используем всплывающий элемент, чтобы отобразить дополнительные сведения о расположении. Объект HTML `string` или `HTMLElement` можно передать в параметр `content` всплывающего элемента. При необходимости всплывающие элементы можно показывать независимо от любой фигуры. Поэтому для всплывающих окон требуется указать значение `position`. Укажите значение `position`. Чтобы отобразить всплывающий элемент, вызовите метод `open` и передайте объект `map`, в котором должен отображаться всплывающий элемент.

```javascript
//Add a marker to the map in which to display a popup for.
var marker = new atlas.HtmlMarker({
    position: [-122.33, 47.6]
});

//Add the marker to the map.
map.markers.add(marker);

//Create a popup.
var popup = new atlas.Popup({
    content: '<div style="padding:5px"><b>Hello World!</b></div>',
    position: [-122.33, 47.6],
    pixelOffset: [0, -35]
});

//Add a click event to the marker to open the popup.
map.events.add('click', marker, function () {
    //Open the popup
    popup.open(map);
});
```

![Всплывающее окно в Azure Maps](media/migrate-google-maps-web-app/azure-maps-popup.png)

> [!NOTE]
> Чтобы сделать то же самое со слоем символов, пузырьков, линий или многоугольников, можно передать выбранный слой в код события карты вместо метки.

**Дополнительные ресурсы:**

- [Добавление всплывающего окна](map-add-popup.md)
- [Всплывающее окно с мультимедийным содержимым](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popup%20with%20Media%20Content)
- [Всплывающие окна на фигурах](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popups%20on%20Shapes)
- [Повторное использование всплывающего окна с несколькими точками](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Reusing%20Popup%20with%20Multiple%20Pins)
- [Класс Popup](/javascript/api/azure-maps-control/atlas.popup)
- [Параметры всплывающего окна](/javascript/api/azure-maps-control/atlas.popupoptions)

### <a name="import-a-geojson-file"></a>Импорт файла GeoJSON

Google Карты поддерживают загрузку и динамическую стилизацию данных GeoJSON с помощью класса `google.maps.Data`. Функциональность этого класса в значительной степени совпадает с основанной на данных стилизацией Azure Maps. Но существует важное отличие: в Google Картах указывается функция обратного вызова. и бизнес-логику для стилизации каждого компонента, отдельно обрабатываемого в потоке пользовательского интерфейса. В Azure Maps слои поддерживают указание выражений, управляемых данными, в виде параметров стилизации. Эти выражения обрабатываются во время преобразования для просмотра в отдельном потоке. Подход Azure Maps повышает производительность отрисовки, а также позволяет быстрее визуализировать большие наборы данных.

В следующих примерах загружается канал GeoJSON с данными обо всех землетрясениях за последние семь дней, сообщенными Геологической службой США. Эти данные отображаются на карте в виде кругов разного размера. Цвет и масштаб каждой окружности зависят от магнитуды каждого землетрясения, которая хранится в свойстве `"mag"` каждого компонента в наборе данных. Если магнитуда больше или равна пяти, окружность будет красной. Если больше или равна трем, но меньше пяти, окружность будет оранжевой. Если она меньше трех, окружность будет зеленой. Радиус каждой окружности будет экспонентой магнитуды, умноженной на 0,1.

#### <a name="before-google-maps"></a>До: Google Карты

Функцию обратного вызова следует указать в методе `map.data.setStyle`. Внутри функции обратного вызова можно применить бизнес-логику к каждому компоненту. Загрузить веб-канал GeoJSON можно с помощью метода `map.data.loadGeoJson`.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(20, -160),
                zoom: 2
            });

            //Define a callback to style each feature.
            map.data.setStyle(function (feature) {

                //Extract the 'mag' property from the feature.
                var mag = parseFloat(feature.getProperty('mag'));

                //Set the color value to 'green' by default.
                var color = 'green';

                //If the mag value is greater than 5, set the color to 'red'.
                if (mag >= 5) {
                    color = 'red';
                }
                //If the mag value is greater than 3, set the color to 'orange'.
                else if (mag >= 3) {
                    color = 'orange';
                }

                return /** @type {google.maps.Data.StyleOptions} */({
                    icon: {
                        path: google.maps.SymbolPath.CIRCLE,

                        //Scale the radius based on an exponential of the 'mag' value.
                        scale: Math.exp(mag) * 0.1,
                        fillColor: color,
                        fillOpacity: 0.75,
                        strokeWeight: 2,
                        strokeColor: 'white'
                    }
                });
            });

            //Load the data feed.
            map.data.loadGeoJson(earthquakeFeed);
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![GeoJSON в Google Картах](media/migrate-google-maps-web-app/google-maps-geojson.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

GeoJSON является базовым типом данных в Azure Maps. Его можно импортировать в источник данных с помощью метода `datasource.importFromUrl`. Использование слоя пузырьков. Слой пузырьков предоставляет функции для отрисовки масштабируемых окружностей на основе свойств компонентов в источнике данных. Вместо функции обратного вызова бизнес-логика преобразуется в выражение и передается в параметры стиля. Выражения определяют, как работает бизнес-логика. Выражения можно передавать в другой поток и вычислять по данным компонентов. В Azure Maps можно добавлять несколько источников данных и слоев, у каждого из которых разная бизнес-логика. Это позволяет показывать несколько наборов данных на карте разными способами.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-160, 20],
                zoom: 1,

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Load the earthquake data.
                datasource.importDataFromUrl(earthquakeFeed);

                //Create a layer to render the data points as scaled circles.
                map.layers.add(new atlas.layer.BubbleLayer(datasource, null, {
                    //Make the circles semi-transparent.
                    opacity: 0.75,

                    color: [
                        'case',

                        //If the mag value is greater than 5, return 'red'.
                        ['>=', ['get', 'mag'], 5],
                        'red',

                        //If the mag value is greater than 3, return 'orange'.
                        ['>=', ['get', 'mag'], 3],
                        'orange',

                        //Return 'green' as a fallback.
                        'green'
                    ],

                    //Scale the radius based on an exponential of the 'mag' value.
                    radius: ['*', ['^', ['e'], ['get', 'mag']], 0.1]
                }));
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![GeoJSON в Azure Maps](media/migrate-google-maps-web-app/azure-maps-geojson.png)

**Дополнительные ресурсы:**

* [Добавление слоя символов](map-add-pin.md)
* [Добавление слоя пузырьков](map-add-bubble-layer.md)
* [Данные точек кластера](clustering-point-data-web-sdk.md)
* [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

### <a name="marker-clustering"></a>Кластеризация меток

При визуализации большого количества точек данных на карте они могут перекрывать друг друга. Перекрытие загромождает карту, и она становится трудной для чтения и использования. Объединение данных точек — это процесс объединения точек данных, находящихся рядом друг с другом, и представления их на карте в виде одной объединенной точки данных (кластера). По мере того как пользователь масштабирует карту, кластеры разбиваются на отдельные точки данных. Объединяйте точки данных, чтобы повысить удобство работы пользователей и производительность карт.

В следующих примерах код загружает веб-канал GeoJSON с данными о землетрясениях за прошлую неделю, после чего он добавляется на карту. Кластеры отображаются в виде цветных кругов разного размера. Размер и цвет кругов зависят от числа содержащихся в них точек.

> [!NOTE]
> В Google Картах и Azure Maps используются немного разные алгоритмы объединения. Поэтому распределение точек в кластерах иногда может отличаться.

#### <a name="before-google-maps"></a>До: Google Карты

Использование библиотеки MarkerCluster для меток кластеров. Значками кластера могут быть только изображения, которые содержат числа от одного до пяти в качестве имени. Они размещаются в одном каталоге.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(20, -160),
                zoom: 2
            });

            //Download the GeoJSON data.
            fetch(earthquakeFeed)
                .then(function (response) {
                    return response.json();
                }).then(function (data) {
                    //Loop through the GeoJSON data and create a marker for each data point.
                    var markers = [];

                    for (var i = 0; i < data.features.length; i++) {

                        markers.push(new google.maps.Marker({
                            position: new google.maps.LatLng(data.features[i].geometry.coordinates[1], data.features[i].geometry.coordinates[0])
                        }));
                    }

                    //Create a marker clusterer instance and tell it where to find the cluster icons.
                    var markerCluster = new MarkerClusterer(map, markers,
                        { imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m' });
                });
        }
    </script>

    <!-- Load the marker cluster library. -->
    <script src="https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/markerclusterer.js"></script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Кластеризация в Google Картах](media/migrate-google-maps-web-app/google-maps-clustering.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Добавление данных в источник данных и управление ими. Можно подключить источники данных и слои, а затем отобразить данные. Класс `DataSource` в Azure Maps предоставляет несколько параметров кластеризации.

* `cluster` — указывает источнику данных выполнить кластеризацию данных точек.
* `clusterRadius` — радиус в пикселях, используемый для кластеризации точек.
* `clusterMaxZoom` — максимальный уровень масштаба, в котором происходит кластеризация. При дополнительном увеличении все точки отрисовываются как символы.
* `clusterProperties` — определяет пользовательские свойства, которые рассчитываются с помощью выражений для всех точек в каждом кластере и добавляются в свойства каждой точки кластера.

Если кластеризация включена, источник данных будет отсылать кластеризованные и некластеризованные точки данных слоям для отрисовки. Источник данных способен выполнять кластеризацию сотен тысяч точек данных. Кластеризованная точка данных имеет следующие свойства.

| Имя свойства             | Тип    | Описание   |
|---------------------------|---------|---------------|
| `cluster`                 | Логическое | Указывает, представляет ли компонент кластер. |
| `cluster_id`              | строка  | Уникальный идентификатор кластера, который можно использовать с методами `getClusterExpansionZoom`, `getClusterChildren` и `getClusterLeaves` для DataSource. |
| `point_count`             | number  | Число точек, содержащихся в кластере.  |
| `point_count_abbreviated` | строка  | Строка, которая сокращает значение `point_count`, если оно длинное (например, 4000 преобразуется в 4K).  |

Класс `DataSource` имеет следующую вспомогательную функцию для доступа к дополнительным сведениям о кластере с помощью `cluster_id`.

| Метод | Возвращаемый тип | Описание |
|--------|-------------|-------------|
| `getClusterChildren(clusterId: number)` | Promise&lt;Массив&lt;Компонент&lt;Геометрия, любая&gt; \| Фигура&gt;&gt; | Извлекает дочерние элементы заданного кластера на следующем уровне масштабирования. Эти дочерние элементы могут быть сочетанием фигур и подкластеров. Подкластеры будут представлять собой компоненты со свойствами, соответствующими ClusteredProperties. |
| `getClusterExpansionZoom(clusterId: number)` | Promise&lt;число&gt; | Вычисляет уровень масштабирования, при котором кластер начинает расширяться или разбиваться на части. |
| `getClusterLeaves(clusterId: number, limit: number, offset: number)` | Promise&lt;Массив&lt;Компонент&lt;Геометрия, любая&gt; \| Фигура&gt;&gt; | Извлекает все точки в кластере. Задайте `limit`, чтобы вернуть подмножество точек, и используйте `offset`, чтобы пролистать страницы с точками. |

При отрисовке кластеризованных данных на карте зачастую проще использовать два или более слоев. В следующем примере используются три слоя. Слой пузырьков для визуализации масштабируемых цветных окружностей на основе размеров кластеров. Слой символов для отрисовки размера кластера в виде текста и второй слой символов для визуализации некластеризованных точек. Существует много других способов отображения кластеризованных данных в Azure Maps. Дополнительные сведения см. в статье [Clustering point data](clustering-point-data-web-sdk.md) (Данные точки кластеризации).

Данные GeoJSON можно импортировать непосредственно в карту Azure Maps с помощью функции `importDataFromUrl` в классе `DataSource`.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map, datasource;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson';

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-160, 20],
                zoom: 1,

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource(null, {
                    //Tell the data source to cluster point data.
                    cluster: true
                });
                map.sources.add(datasource);

                //Create layers for rendering clusters, their counts and unclustered points and add the layers to the map.
                map.layers.add([
                    //Create a bubble layer for rendering clustered data points.
                    new atlas.layer.BubbleLayer(datasource, null, {
                        //Scale the size of the clustered bubble based on the number of points inthe cluster.
                        radius: [
                            'step',
                            ['get', 'point_count'],
                            20,         //Default of 20 pixel radius.
                            100, 30,    //If point_count >= 100, radius is 30 pixels.
                            750, 40     //If point_count >= 750, radius is 40 pixels.
                        ],

                        //Change the color of the cluster based on the value on the point_cluster property of the cluster.
                        color: [
                            'step',
                            ['get', 'point_count'],
                            'lime',            //Default to lime green. 
                            100, 'yellow',     //If the point_count >= 100, color is yellow.
                            750, 'red'         //If the point_count >= 750, color is red.
                        ],
                        strokeWidth: 0,
                        filter: ['has', 'point_count'] //Only rendered data points which have a point_count property, which clusters do.
                    }),

                    //Create a symbol layer to render the count of locations in a cluster.
                    new atlas.layer.SymbolLayer(datasource, null, {
                        iconOptions: {
                            image: 'none' //Hide the icon image.
                        },
                        textOptions: {
                            textField: ['get', 'point_count_abbreviated'],
                            offset: [0, 0.4]
                        }
                    }),

                    //Create a layer to render the individual locations.
                    new atlas.layer.SymbolLayer(datasource, null, {
                        filter: ['!', ['has', 'point_count']] //Filter out clustered points from this layer.
                    })
                ]);

                //Retrieve a GeoJSON data set and add it to the data source. 
                datasource.importDataFromUrl(earthquakeFeed);
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Кластеризация в Azure Maps](media/migrate-google-maps-web-app/azure-maps-clustering.png)

**Дополнительные ресурсы:**

* [Добавление слоя символов](map-add-pin.md)
* [Добавление слоя пузырьков](map-add-bubble-layer.md)
* [Данные точек кластера](clustering-point-data-web-sdk.md)
* [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

### <a name="add-a-heat-map"></a>Добавление тепловой карты

Тепловая карта, которую также называют картой точек плотности, представляет собой тип визуализации данных. Они используются для визуализации плотности данных с помощью диапазона цветов. Такие визуализации часто используются для отображения "горячих точек" данных на карте. Тепловые карты особенно полезны для визуализации больших наборов данных точек.

В следующих примерах загружается веб-канал GeoJSON с данными обо всех землетрясениях за прошлый месяц из Геологической службы США, которые отображаются в виде взвешенной тепловой карты. В качестве веса используется свойство `"mag"`.

#### <a name="before-google-maps"></a>До: Google Карты

Чтобы создать тепловую карту, необходимо загрузить библиотеку визуализаций, добавив `&libraries=visualization` в URL-адрес сценария API. Слой тепловой карты в Google Картах не поддерживает прямое преобразование данных в формате GeoJSON. Сначала скачайте данные и преобразуйте их в массив взвешенных точек данных.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.geojson';

        function initMap() {

            var map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(20, -160),
                zoom: 2,
                mapTypeId: 'satellite'
            });

            //Download the GeoJSON data.
            fetch(url).then(function (response) {
                return response.json();
            }).then(function (res) {
                var points = getDataPoints(res);

                var heatmap = new google.maps.visualization.HeatmapLayer({
                    data: points
                });
                heatmap.setMap(map);
            });
        }

        function getDataPoints(geojson, weightProp) {
            var points = [];

            for (var i = 0; i < geojson.features.length; i++) {
                var f = geojson.features[i];

                if (f.geometry.type === 'Point') {
                    points.push({
                        location: new google.maps.LatLng(f.geometry.coordinates[1], f.geometry.coordinates[0]),
                        weight: f.properties[weightProp]
                    });
                }
            }

            return points;
        } 
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]&libraries=visualization" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Тепловая карта в Google Картах](media/migrate-google-maps-web-app/google-maps-heatmap.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Следует загрузить данные GeoJSON в источник данных и подключить его к слою тепловой карты. Свойство, которое будет использоваться для веса, можно передать в параметр `weight` с помощью выражения. Данные GeoJSON можно импортировать непосредственно в Azure Maps с помощью функции `importDataFromUrl` в классе `DataSource`.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;
        var earthquakeFeed = 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.geojson';

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-160, 20],
                zoom: 1,
                style: 'satellite',

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Load the earthquake data.
                datasource.importDataFromUrl(earthquakeFeed);

                //Create a layer to render the data points as a heat map.
                map.layers.add(new atlas.layer.HeatMapLayer(datasource, null, {
                    weight: ['get', 'mag'],
                    intensity: 0.005,
                    opacity: 0.65,
                    radius: 10
                }));
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Тепловая карта в Azure Maps](media/migrate-google-maps-web-app/azure-maps-heatmap.png)

**Дополнительные ресурсы:**

- [Добавление слоя тепловой карты](map-add-heat-map-layer.md)
- [Класс слоя тепловой карты](/javascript/api/azure-maps-control/atlas.layer.heatmaplayer)
- [Параметры слоя тепловой карты](/javascript/api/azure-maps-control/atlas.heatmaplayeroptions)
- [Использование стилистических выражений на основе данных](data-driven-style-expressions-web-sdk.md)

### <a name="overlay-a-tile-layer"></a>Наложение слоя фрагментов

Слои фрагментов Azure Maps в Google Картах называются наложениями изображений. Они позволяют накладывать большие изображения, разбитые на небольшие фрагменты, которые соответствуют системе разбиения карты. Такой подход часто используется для наложения больших изображений или больших наборов данных.

В приведенных ниже примерах накладывается слой фрагментов радара погоды из лаборатории окружающей среды Университета штата Айова.

#### <a name="before-google-maps"></a>До: Google Карты

В Google Картах слои фрагментов можно создавать с помощью класса `google.maps.ImageMapType`.

```javascript
map.overlayMapTypes.insertAt(0, new google.maps.ImageMapType({
    getTileUrl: function (coord, zoom) {
        return "https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/" + zoom + "/" + coord.x + "/" + coord.y;
    },
    tileSize: new google.maps.Size(256, 256),
    opacity: 0.8
}));
```

![Слой фрагментов в Google Картах](media/migrate-google-maps-web-app/google-maps-tile-layer.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Слой фрагментов можно добавить на карту точно так же, как и любой другой слой. Следует использовать форматированный URL-адрес с заполнителями значений x, y и масштаба (`{x}`, `{y}` и `{z}` соответственно), чтобы указать, где обращаться к фрагментам. Слои фрагментов в Azure Maps также поддерживают заполнители `{quadkey}`, `{bbox-epsg-3857}` и `{subdomain}`.

> [!TIP]
> В Azure Maps слои можно легко отобразить под другими слоями, в том числе под базовыми слоями карт. Часто желательно отображать слои фрагментов под метками карты, чтобы их можно было легко читать. Метод `map.layers.add` принимает второй параметр — идентификатор слоя, в который следует вставить новый слой. Чтобы вставить слой фрагментов под метками карты, используйте следующий код: `map.layers.add(myTileLayer, "labels");`

```javascript
//Create a tile layer and add it to the map below the label layer.
map.layers.add(new atlas.layer.TileLayer({
    tileUrl: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png',
    opacity: 0.8,
    tileSize: 256
}), 'labels');
```

![Слой фрагментов в Azure Maps](media/migrate-google-maps-web-app/azure-maps-tile-layer.png)

> [!TIP]
> Запросы на фрагмент можно записать с помощью параметра `transformRequest` карты. Это позволит при необходимости изменить или добавить заголовки в запрос.

**Дополнительные ресурсы:**

- [Добавление слоев фрагмента карты](map-add-tile-layer.md)
- [Класс слоя фрагментов](/javascript/api/azure-maps-control/atlas.layer.tilelayer)
- [Параметры слоя фрагментов](/javascript/api/azure-maps-control/atlas.tilelayeroptions)

### <a name="show-traffic-data"></a>Отображение данных о трафике

Данные дорожного движения можно перенести на карту как в Azure Maps, так и в Google Картах.

#### <a name="before-google-maps"></a>До: Google Карты

Наложение данных дорожного движения на карте с помощью слоя дорожного движения.

```javascript
var trafficLayer = new google.maps.TrafficLayer();
trafficLayer.setMap(map);
```

![Данные дорожного движения в Google Картах](media/migrate-google-maps-web-app/google-maps-traffic.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Azure Maps предоставляет несколько различных вариантов отображения дорожного движения. Дорожно-транспортные происшествия, такие как закрытие дорог и аварии, можно отображать на карте в виде значков. На карту можно наложить сведения о транспортном потоке с цветовым кодированием сегментов дорог. Цвета можно изменять в соответствии с ограничениями скорости, относительно нормальной ожидаемой или абсолютной задержки. Данные о дорожно-транспортных происшествиях в Azure Maps обновляются каждую минуту, а данные о дорожном движении — каждые две минуты.

Назначьте нужные значения для параметров `setTraffic`.

```javascript
map.setTraffic({
    incidents: true,
    flow: 'relative'
});
```

![Данные дорожного движения в Azure Maps](media/migrate-google-maps-web-app/azure-maps-traffic.png)

Если щелкнуть один из значков дорожного движения в Azure Maps, во всплывающем окне отобразятся дополнительные сведения.

![Дорожно-транспортные происшествия в Azure Maps](media/migrate-google-maps-web-app/azure-maps-traffic-incident.png)

**Дополнительные ресурсы:**

* [Отображение данных дорожного движения на карте](map-show-traffic.md)
* [Параметры наложения сведений о дорожном движении](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Traffic%20Overlay%20Options)

### <a name="add-a-ground-overlay"></a>Добавление наземного наложения

Azure Maps и Google Карты поддерживают наложение изображений с геоссылками на карту. Изображения с геоссылками перемещаются и изменяют свой размер при сдвиге и масштабировании карты. В Google Картах они называются наземными наложениями, а в Azure Maps — слоями изображений. Они отлично подходят для создания планов этажей, наложения старых карт или изображений, снятых с помощью дронов.

#### <a name="before-google-maps"></a>До: Google Карты

Укажите URL-адрес изображения, которое необходимо наложить, и ограничивающий прямоугольник для привязки изображения к карте. В этом примере показано, как наложить на карту изображение [карты Ньюарка, Нью-Джерси, (1922 г.)](https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg).

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map, historicalOverlay;

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(40.740, -74.18),
                zoom: 12
            });

            var imageBounds = {
                north: 40.773941,
                south: 40.712216,
                east: -74.12544,
                west: -74.22655
            };

            historicalOverlay = new google.maps.GroundOverlay(
                'https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg',
                imageBounds);
            historicalOverlay.setMap(map);
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id="myMap" style="position:relative;width:600px;height:400px;"></div>
</body>
</html>
```

При выполнении этого кода в браузере отобразится показанная ниже карта:

![Наложение изображений в Google Картах](media/migrate-google-maps-web-app/google-maps-image-overlay.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

Для наложения изображений с геоссылками используется класс `atlas.layer.ImageLayer`. Этому классу требуется URL-адрес изображения и набор координат для четырех углов изображения. Изображение должно размещаться либо в том же домене, либо поддерживать CORS.

> [!TIP]
> Если у вас есть только информация о координатах севера, юга, востока, запада и повороте, а не координаты каждого угла изображения, можно использовать статический метод [`atlas.layer.ImageLayer.getCoordinatesFromEdges`](/javascript/api/azure-maps-control/atlas.layer.imagelayer#getcoordinatesfromedges-number--number--number--number--number-).

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <script type='text/javascript'>
        var map;

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                center: [-74.18, 40.740],
                zoom: 12,

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

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
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Наложение изображений в Azure Maps](media/migrate-google-maps-web-app/azure-maps-image-overlay.png)

**Дополнительные ресурсы:**

- [Наложение изображения](map-add-image-layer.md)
- [Класс слоя изображений](/javascript/api/azure-maps-control/atlas.layer.imagelayer)

### <a name="add-kml-data-to-the-map"></a>Добавление данных KML на карту

Azure Maps и Google Карты позволяют импортировать данные KML, KMZ и GeoRSS и визуализировать их на карте. Azure Maps также поддерживает файлы GPX, GML, пространственные файлы CSV, GeoJSON, WKT, веб-службы сопоставления (WMS), веб-службы фрагментирования (WMTS) и веб-службы компонентов (WFS). Azure Maps считывает файлы локально в память и в большинстве случаев может работать с гораздо большими файлами KML. 

#### <a name="before-google-maps"></a>До: Google Карты

```javascript
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
        var map, historicalOverlay;

        function initMap() {
            map = new google.maps.Map(document.getElementById('myMap'), {
                center: new google.maps.LatLng(0, 0),
                zoom: 1
            });

             var layer = new google.maps.KmlLayer({
              url: 'https://googlearchive.github.io/js-v2-samples/ggeoxml/cta.kml',
              map: map
            });
        }
    </script>

    <!-- Google Maps Script Reference -->
    <script src="https://maps.googleapis.com/maps/api/js?callback=initMap&key=[Your Google Maps Key]" async defer></script>
</head>
<body>
    <div id="myMap" style="position:relative;width:600px;height:400px;"></div>
</body>
</html>
```

При выполнении этого кода в браузере отобразится показанная ниже карта:

![Google Карты с KML](media/migrate-google-maps-web-app/google-maps-kml.png)

#### <a name="after-azure-maps"></a>После: Azure Maps

В Azure Maps GeoJSON — это основной формат данных, используемый в веб-пакете SDK. Дополнительные форматы пространственных данных можно легко интегрировать в [модуль пространственного ввода-вывода](/javascript/api/azure-maps-spatial-io/). Этот модуль содержит функции для чтения и записи пространственных данных, а также уровень простых данных, с помощью которого можно легко показать данные из любого из перечисленных форматов пространственных данных. Для чтения данных в файле пространственных данных передайте URL-адрес или необработанные данные в виде строки или большого двоичного объекта в функцию `atlas.io.read`. При этом будут возвращены все проанализированные данные из файла, которые затем можно добавить на карту. Формат KML более сложный, чем большинство форматов пространственных данных, так как содержит гораздо больше сведений о стилях. Класс `SpatialDataLayer` поддерживает отрисовку большинства из этих стилей, но перед загрузкой данных компонентов изображения значков должны быть загружены на карту, а наземные наложения необходимо добавлять на карту отдельно. При загрузке данных по URL-адресу данные должны быть размещены на конечной точке с поддержкой COR, либо в качестве параметра функции чтения должна передаваться прокси-служба. 

```javascript
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <!-- Add reference to the Azure Maps Spatial IO module. -->
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>

    <script type='text/javascript'>
        var map, datasource, layer;

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                view: 'Auto',

                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {
            
                //Create a data source and add it to the map.
                datasource = new atlas.source.DataSource();
                map.sources.add(datasource);

                //Add a simple data layer for rendering the data.
                layer = new atlas.layer.SimpleDataLayer(datasource);
                map.layers.add(layer);

                //Read a KML file from a URL or pass in a raw KML string.
                atlas.io.read('https://googlearchive.github.io/js-v2-samples/ggeoxml/cta.kml').then(async r => {
                    if (r) {

                        //Check to see if there are any icons in the data set that need to be loaded into the map resources.
                        if (r.icons) {
                            //For each icon image, create a promise to add it to the map, then run the promises in parrallel.
                            var imagePromises = [];

                            //The keys are the names of each icon image.
                            var keys = Object.keys(r.icons);

                            if (keys.length !== 0) {
                                keys.forEach(function (key) {
                                    imagePromises.push(map.imageSprite.add(key, r.icons[key]));
                                });

                                await Promise.all(imagePromises);
                            }
                        }

                        //Load all features.
                        if (r.features && r.features.length > 0) {
                            datasource.add(r.features);
                        }

                        //Load all ground overlays.
                        if (r.groundOverlays && r.groundOverlays.length > 0) {
                            map.layers.add(r.groundOverlays);
                        }

                        //If bounding box information is known for data, set the map view to it.
                        if (r.bbox) {
                            map.setCamera({ bounds: r.bbox, padding: 50 });
                        }
                    }
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![KML в Azure Maps](media/migrate-google-maps-web-app/azure-maps-kml.png)</center>

**Дополнительные ресурсы:**

- [Функция atlas.io.read](/javascript/api/azure-maps-spatial-io/atlas.io#read-string---arraybuffer---blob--spatialdatareadoptions-)
- [SimpleDataLayer](/javascript/api/azure-maps-spatial-io/atlas.layer.simpledatalayer)
- [SimpleDataLayerOptions](/javascript/api/azure-maps-spatial-io/atlas.simpledatalayeroptions)

## <a name="additional-code-samples"></a>Дополнительные примеры кода

Ниже приведены некоторые дополнительные примеры кода, связанные с перемещением из Google Карт.

* [Инструменты для рисования](map-add-drawing-toolbar.md)
* [Ограничение карты до масштабирования с помощью двух пальцев](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Limit%20Map%20to%20Two%20Finger%20Panning)
* [Ограничение карты до масштабирования с помощью колесика прокрутки](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Limit%20Scroll%20Wheel%20Zoom)
* [Создание полноэкранного элемента управления](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Create%20a%20Fullscreen%20Control)

**Службы.**

* [Использование модуля служб Azure Maps](how-to-use-services-module.md)
* [Поиск достопримечательностей](map-search-location.md)
* [Получение информации из координаты (обратный геокод)](map-get-information-from-coordinate.md)
* [Отображение направлений от точки А до точки Б](map-route.md)
* [Автозаполнение в поисковых запросах с использованием пользовательского интерфейса JQuery](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Search%20Autosuggest%20and%20JQuery%20UI)

## <a name="google-maps-v3-to-azure-maps-web-sdk-class-mapping"></a>Сопоставление классов пакета SDK для Google Карт версии 3 с классами веб-пакета SDK для Azure Maps

В следующем приложении приводится перекрестное сопоставление наиболее часто используемых классов в пакете SDK Google Карт версии 3 с их эквивалентами в веб-пакете SDK Azure Maps.

### <a name="core-classes"></a>Основные классы

| Google Карты   | Azure Maps  |
|---------------|-------------|
| `google.maps.Map` | [atlas.Map](/javascript/api/azure-maps-control/atlas.map)  |
| `google.maps.InfoWindow` | [atlas.Popup](/javascript/api/azure-maps-control/atlas.popup)  |
| `google.maps.InfoWindowOptions` | [atlas.PopupOptions](https://docs.microsoft.com/) |
| `google.maps.LatLng`  | [atlas.data.Position](/javascript/api/azure-maps-control/atlas.data.position)  |
| `google.maps.LatLngBounds` | [atlas.data.BoundingBox](/javascript/api/azure-maps-control/atlas.data.boundingbox) |
| `google.maps.MapOptions`  | [atlas.CameraOptions](/javascript/api/azure-maps-control/atlas.cameraoptions)<br/>[atlas.CameraBoundsOptions](/javascript/api/azure-maps-control/atlas.cameraboundsoptions)<br/>[atlas.ServiceOptions](/javascript/api/azure-maps-control/atlas.serviceoptions)<br/>[atlas.StyleOptions](/javascript/api/azure-maps-control/atlas.styleoptions)<br/>[atlas.UserInteractionOptions](/javascript/api/azure-maps-control/atlas.userinteractionoptions) |
| `google.maps.Point`  | [atlas.Pixel](/javascript/api/azure-maps-control/atlas.pixel)   |

## <a name="overlay-classes"></a>Классы наложения

| Google Карты  | Azure Maps  |
|--------------|-------------|
| `google.maps.Marker` | [atlas.HtmlMarker](/javascript/api/azure-maps-control/atlas.htmlmarker)<br/>[atlas.data.Point](/javascript/api/azure-maps-control/atlas.data.point)  |
| `google.maps.MarkerOptions`  | [atlas.HtmlMarkerOptions](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)<br/>[atlas.layer.SymbolLayer](/javascript/api/azure-maps-control/atlas.layer.symbollayer)<br/>[atlas.SymbolLayerOptions](/javascript/api/azure-maps-control/atlas.symbollayeroptions)<br/>[atlas.IconOptions](/javascript/api/azure-maps-control/atlas.iconoptions)<br/>[atlas.TextOptions](/javascript/api/azure-maps-control/atlas.textoptions)<br/>[atlas.layer.BubbleLayer](/javascript/api/azure-maps-control/atlas.layer.bubblelayer)<br/>[atlas.BubbleLayerOptions](/javascript/api/azure-maps-control/atlas.bubblelayeroptions) |
| `google.maps.Polygon`  | [atlas.data.Polygon](/javascript/api/azure-maps-control/atlas.data.polygon)               |
| `google.maps.PolygonOptions` |[atlas.layer.PolygonLayer](/javascript/api/azure-maps-control/atlas.layer.polygonlayer)<br/> [atlas.PolygonLayerOptions](/javascript/api/azure-maps-control/atlas.polygonlayeroptions)<br/> [atlas.layer.LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer)<br/> [atlas.LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions)|
| `google.maps.Polyline` | [atlas.data.LineString](/javascript/api/azure-maps-control/atlas.data.linestring)         |
| `google.maps.PolylineOptions` | [atlas.layer.LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer)<br/>[atlas.LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions) |
| `google.maps.Circle`  | См. сведения в раздела о [добавлении окружности на карту](map-add-shape.md#add-a-circle-to-the-map).                                     |
| `google.maps.ImageMapType`  | [atlas.TileLayer](/javascript/api/azure-maps-control/atlas.layer.tilelayer)         |
| `google.maps.ImageMapTypeOptions` | [atlas.TileLayerOptions](/javascript/api/azure-maps-control/atlas.tilelayeroptions) |
| `google.maps.GroundOverlay`  | [atlas.layer.ImageLayer](/javascript/api/azure-maps-control/atlas.layer.imagelayer)<br/>[atlas.ImageLayerOptions](/javascript/api/azure-maps-control/atlas.imagelayeroptions) |

## <a name="service-classes"></a>Классы служб

Веб-пакет SDK Azure Maps содержит модуль служб, который можно загрузить отдельно. Этот модуль создает оболочку для служб REST Azure Maps с веб-API и может использоваться в приложениях JavaScript, TypeScript и Node.js.

| Google Карты | Azure Maps  |
|-------------|-------------|
| `google.maps.Geocoder` | [atlas.service.SearchUrl](/javascript/api/azure-maps-rest/atlas.service.searchurl)  |
| `google.maps.GeocoderRequest`  | [atlas.SearchAddressOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressoptions)<br/>[atlas.SearchAddressRevrseOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressreverseoptions)<br/>[atlas.SearchAddressReverseCrossStreetOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressreversecrossstreetoptions)<br/>[atlas.SearchAddressStructuredOptions](/javascript/api/azure-maps-rest/atlas.service.searchaddressstructuredoptions)<br/>[atlas.SearchAlongRouteOptions](/javascript/api/azure-maps-rest/atlas.service.searchalongrouteoptions)<br/>[atlas.SearchFuzzyOptions](/javascript/api/azure-maps-rest/atlas.service.searchfuzzyoptions)<br/>[atlas.SearchInsideGeometryOptions](/javascript/api/azure-maps-rest/atlas.service.searchinsidegeometryoptions)<br/>[atlas.SearchNearbyOptions](/javascript/api/azure-maps-rest/atlas.service.searchnearbyoptions)<br/>[atlas.SearchPOIOptions](/javascript/api/azure-maps-rest/atlas.service.searchpoioptions)<br/>[atlas.SearchPOICategoryOptions](/javascript/api/azure-maps-rest/atlas.service.searchpoicategoryoptions) |
| `google.maps.DirectionsService`  | [atlas.service.RouteUrl](/javascript/api/azure-maps-rest/atlas.service.routeurl)  |
| `google.maps.DirectionsRequest`  | [atlas.CalculateRouteDirectionsOptions](/javascript/api/azure-maps-rest/atlas.service.calculateroutedirectionsoptions) |
| `google.maps.places.PlacesService` | [atlas.service.SearchUrl](/javascript/api/azure-maps-rest/atlas.service.searchurl)  |

## <a name="libraries"></a>Библиотеки

Библиотеки добавляют дополнительные функции к картам. Многие из этих функций содержатся в основном пакете SDK Azure Maps. Ниже приведены некоторые эквивалентные классы, которые можно использовать вместо этих библиотек Google Карт.

| Google Карты           | Azure Maps   |
|-----------------------|--------------|
| Библиотека средств рисования       | [Модуль средств рисования](set-drawing-options.md) |
| Библиотека геометрических фигур      | [atlas.math](/javascript/api/azure-maps-control/atlas.math)   |
| Библиотека визуализаций | [Слой тепловой карты](map-add-heat-map-layer.md) |

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов для очистки.

## <a name="next-steps"></a>Следующие шаги

Узнайте больше о переходе на Azure Maps:

> [!div class="nextstepaction"]
> [Перенос веб-службы](migrate-from-google-maps-web-services.md)

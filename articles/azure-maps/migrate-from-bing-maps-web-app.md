---
title: Руководство по Перенос веб-приложения из Карт Bing | Microsoft Azure Maps
description: В руководстве показано, как перенести веб-приложение из Карт Bing в Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 9/10/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: devx-track-js
ms.openlocfilehash: 2b072107275fba1ff83ab3ddac63ed8bf7766356
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100389116"
---
# <a name="tutorial-migrate-a-web-app-from-bing-maps"></a>Руководство по Перенос веб-приложения из Карт Bing

Веб-приложения, использующие Карты Bing, часто используют пакет SDK Карт Bing версии 8 для JavaScript. Подходящий пакет SDK на основе Azure для переноса — веб-пакет SDK для Azure Maps. Веб-пакет SDK Azure Maps позволяет настраивать интерактивные карты, используя собственные изображения и содержимое для отображения в веб-приложениях или мобильных приложениях. Этот элемент управления использует WebGL, позволяя вам отображать большие наборы данных с высокой производительностью. Вы можете выполнять разработку с помощью пакета SDK, используя JavaScript или TypeScript. В этом учебнике рассматривается следующее.

> [!div class="checklist"]
> * Загрузка карты
> * локализация карты;
> * добавление канцелярских кнопок, ломаных линий и многоугольников.
> * отображение сведений во всплывающем окне или окне сведений;
> * загрузка и отображение данных KML и GeoJSON;
> * кластерные канцелярские кнопки;
> * Наложение слоя фрагментов
> * Отображение данных о трафике
> * Добавление наземного наложения

При переносе имеющегося веб-приложения проверьте, использует ли оно библиотеку элементов управления картой с открытым кодом, например Cesium, Leaflet или OpenLayers. Если это так, и вы предпочитаете использовать эту библиотеку в дальнейшем, можно подключить ее к службе фрагментов карт Azure Maps ([фрагменты дорожных карт](/rest/api/maps/render/getmaptile) \| [фрагменты спутниковых карт](/rest/api/maps/render/getmapimagerytile)). По ссылкам ниже приведены сведения об использовании Azure Maps в некоторых часто используемых библиотеках элементов управления картой с открытым кодом.

* [Cesium](https://cesiumjs.org/) — элемент управления трехмерными картами для Интернета. [Примеры кода](https://azuremapscodesamples.azurewebsites.net/?search=Cesium) \| [Репозиторий подключаемого модуля]()
* [Leaflet](https://leafletjs.com/) — упрощенный элемент управления двухмерными картами для Интернета. [Примеры кода](https://azuremapscodesamples.azurewebsites.net/?search=leaflet) \| [Репозиторий подключаемого модуля]()
* [OpenLayers](https://openlayers.org/) — элемент управления двухмерными картами для Интернета, который поддерживает проекции. [Примеры кода](https://azuremapscodesamples.azurewebsites.net/?search=openlayers) \| [Репозиторий подключаемого модуля]()

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

В следующей таблице перечислены основные функции API в пакете SDK для JavaScript в Картах Bing версии 8 и сведения о поддержке аналогичного API в веб-пакете SDK для Azure Maps.

| Функция Карт Bing        | Поддержка в веб-пакете SDK для Azure Maps                                                             |
|--------------------------|:--------------------------------------------------------------------------------------:|
| Вешки                 | ✓                                                                                      |
| Кластеризация вешек       | ✓                                                                                      |
| Ломаные линии и многоугольники     | ✓                                                                                      |
| Наземное наложение          | ✓                                                                                      |
| Тепловые карты                | ✓                                                                                      |
| Слои фрагментов              | ✓                                                                                      |
| Слой KML                | ✓                                                                                      |
| Слой контура            | [Примеры](https://azuremapscodesamples.azurewebsites.net/?search=contour)              |
| Слой группирования данных       | Включается в [модуль источника данных с координатной привязкой](https://github.com/Azure-Samples/azure-maps-gridded-data-source) Azure Maps с открытым исходным кодом       |
| Анимированный слой фрагмента      | Включен в [модуль анимации](https://github.com/Azure-Samples/azure-maps-animations) Azure Maps с открытым исходным кодом |
| Инструменты для рисования            | ✓                                                                                      |
| Служба геокодирования         | ✓                                                                                      |
| Служба направлений       | ✓                                                                                      |
| Служба матрицы расстояний  | ✓                                                                                      |
| Служба пространственных данных     | Недоступно                                                                                    |
| Спутниковые изображения или аэроснимки | ✓                                                                                      |
| Изображение с высоты птичьего полета         | Н/Д                                                                                |
| Изображение Streetside       | Н/Д                                                                                |
| Поддержка GeoJSON          | ✓                                                                                      |
| Поддержка GeoXML           | ✓ [Модуль ввода-вывода пространственных данных](how-to-use-spatial-io-module.md)                                                                                     |
| Поддержка формата Well-Known Text  | ✓                                                                                      |
| Настраиваемые стили карт        | Partial                                                                                |

Azure Maps также содержит множество дополнительных [модулей с открытым кодом для веб-пакета SDK](open-source-projects.md#open-web-sdk-modules), расширяющих возможности этого набора служб.

## <a name="notable-differences-in-the-web-sdks"></a>Важные различия в веб-пакетах SDK

Ниже приведены некоторые ключевые различия между веб-пакетами SDK для Карт Bing и Azure Maps, о которых следует знать.

* Помимо предоставления размещенной конечной точки для доступа к веб-пакету SDК Azure Maps, можно также при необходимости использовать пакет NPM для внедрения веб-пакета SDK в приложения. Дополнительные сведения см. в [документации](./how-to-use-map-control.md). Этот пакет также включает определения TypeScript.
* Пакет SDK для Карт Bing предоставляется в двух размещенных ветвях: ветвь выпусков и экспериментальная ветвь. Когда ведется разработка новых компонентов в экспериментальной ветви может выходить до нескольких обновлений в день. В Azure Maps размещается только ветвь выпусков, а экспериментальные функции создаются в виде пользовательских модулей в проекте примеров кода Azure Maps с открытым кодом. Карты Bing, используемые для зафиксированной ветви, также обновляются реже, что уменьшает риск критических изменений в связи с выпуском. В Azure Maps можно использовать модуль NPM и указать выпуск с любым предыдущим дополнительным номером версии.

> [!TIP]
> Azure Maps публикует минифицированные и не минифицированные версии пакета SDK. Простое удалите `.min` из имен файлов. Не минифицированная версия полезна при отладке проблем, но в рабочей среде обязательно используйте минифицированную версию, преимуществом которой является меньший размер файла.

* После создания экземпляра класса Map в Azure Maps код должен ждать запуска для карты события `ready` или `load`, прежде чем взаимодействовать с ней. Эти события подтверждают, что все ресурсы карты загружены и готовы к доступу.
* Обе платформы используют аналогичную систему фрагментов для базовых карт, однако фрагменты в Картах Bing имеют размер 256 пикселей, а фрагменты в Azure Maps — 512. Таким образом, чтобы получить тот же вид карты в Azure Maps, что и в Картах Bing, от уровня масштабирования, используемого в Картах Bing, в Azure Maps следует вычесть единицу.
* В Картах Bing координаты используются в формате `latitude, longitude`, а в Azure Maps — в формате `longitude, latitude`. Этот формат соответствует стандартным координатам `[x, y]`, которые используются большинством ГИС-платформ.

* Фигуры в веб-пакете SDK для Azure Maps основаны на схеме GeoJSON. Вспомогательные классы предоставляются с помощью пространства имен [atlas.data](/javascript/api/azure-maps-control/atlas.data). Существует также класс [atlas.Shape](/javascript/api/azure-maps-control/atlas.shape), который можно использовать для упаковки объектов GeoJSON и упрощения их обновления и поддержки с использованием привязок данных.
* Координаты в Azure Maps определяются как объекты положения, которые можно указать как простой массив чисел в формате `[longitude, latitude]` или `new atlas.data.Position(longitude, latitude)`.

> [!TIP]
> Класс Position содержит статическую вспомогательную функцию для импорта координат в формате `latitude, longitude`. В коде Карт Bing функцию [atlas.data.Position.fromLatLng](/javascript/api/azure-maps-control/atlas.data.position) зачастую можно заменить функцией `new Microsoft.Maps.Location`.

* Вместо того чтобы указывать сведения о стилях для каждой фигуры, добавляемой на карту, Azure Maps отделяет стили от данных. Данные хранятся в источниках данных и подключены к слоям подготовки к просмотру, которые код Azure Maps использует для визуализации данных. Такой подход обеспечивает повышение производительности. Кроме того, многие слои поддерживают управляемый данными стиль, где бизнес-логика может быть добавлена к параметрам стиля слоя, которые изменят способ отображения отдельных фигур в слое на основе свойств, определенных в фигуре.
* Azure Maps предоставляет целый ряд полезных пространственных математических функций в пространстве имен `atlas.math`, но они отличаются от тех, которые входят в модуль пространственных математических вычислений в Картах Bing. Основное отличие заключается в том, что Azure Maps не предоставляет встроенные функции для бинарных операций, например объединения и пересечения множеств, однако поскольку в основе Azure Maps лежит открытый стандарт GeoJSON, для этих служб доступно множество библиотек с открытым кодом. Одно из популярных средств, которое хорошо работает с Azure Maps и предоставляет массу возможностей для пространственных математических вычислений, — [turf js](http://turfjs.org/).

Подробный список терминологии, связанной с Azure Maps, см. также в [глоссарии Azure Maps](./glossary.md).

## <a name="web-sdk-side-by-side-examples"></a>Параллельные примеры веб-пакетов SDK

Ниже приведена коллекция примеров кода для каждой платформы, охватывающих распространенные варианты использования, которые помогут перенести веб-приложение из пакета SDK для JavaScript в Картах Bing версии 8 в веб-пакет SDK для Azure Maps. Примеры кода, связанные с веб-приложениями, предоставляются в формате JavaScript. Однако Azure Maps также предоставляет определения TypeScript в качестве дополнительной возможности с использованием модуля [NPM](./how-to-use-map-control.md).

**Разделы**

* [Загрузка карты](#load-a-map)
* [Локализация карты](#localizing-the-map)
* [Настройка представления карты](#setting-the-map-view)
* [Добавление вешки](#adding-a-pushpin)
* [Добавление пользовательской вешки](#adding-a-custom-pushpin)
* [Добавление ломаной линии](#adding-a-polyline)
* [Добавление многоугольника](#adding-a-polygon)
* [Отображение информационного поля](#display-an-infobox)
* [Кластеризация вешек](#pushpin-clustering)
* [Добавление тепловой карты](#add-a-heat-map)
* [Наложение слоя фрагментов](#overlay-a-tile-layer)
* [Отображение данных о трафике](#show-traffic-data)
* [Добавление наземного наложения](#add-a-ground-overlay)
* [Добавление данных KML на карту](#add-kml-data-to-the-map)
* [Добавление инструментов рисования](#add-drawing-tools)

### <a name="load-a-map"></a>Загрузка карты

Для загрузки карты в обоих пакетах SDK необходимо выполнить один и тот же набор действий.

* Добавьте ссылку на пакет SDK для карт.
* Добавьте в текст страницы тег `div`. Он будет служить заполнителем для карты.
* Создайте функцию JavaScript, которая вызывается после загрузки страницы.
* Создайте экземпляр соответствующего класса Map.

**Некоторые ключевые отличия**

* В Картах Bing необходимо указать ключ учетной записи в ссылке на скрипт API или в качестве параметра карты. Учетные данные проверки подлинности для Azure Maps указываются в качестве параметров класса Map. Это может быть ключ подписки или сведения Azure Active Directory.
* Карты Bing принимают функцию обратного вызова в ссылке на скрипт API, которая используется для вызова функции инициализации для загрузки карты. При работе с Azure Maps следует использовать событие OnLoad страницы.
* При использовании идентификатора для ссылки на элемент `div`, в котором карта будет преобразована для просмотра, в Картах Bing используется селектор HTML (т. е. `#myMap`), а в Azure Maps — только значение идентификатора (т. е. `myMap`).
* Координаты в Azure Maps определяются как объекты положения, которые можно указать как простой массив чисел в формате `[longitude, latitude]`.
* Уровень масштаба в Azure Maps на один уровень меньше, чем в примере Карт Bing, из-за разницы в размерах системы фрагментов между платформами.
* По умолчанию Azure Maps не добавляет на холст карты элементы управления навигацией, такие как кнопки масштабирования и кнопки стилей карт. Однако есть элементы управления для добавления средств выбора стиля карт, кнопок масштабирования, компаса или поворота, а также средства управления наклоном.
* Обработчик событий добавляется в Azure Maps для отслеживания события `ready` экземпляра карты. Он сработает, когда на карте закончится загрузка контекста WebGL и всех необходимых ресурсов. В этот обработчик событий можно добавить любой код заключительной загрузки.

В приведенных ниже примерах показано, как загрузить базовую карту, которая выравнивается по центру Нью-Йорка в точке координат (долгота: −-73,985; широта: 40,747) и находится на уровне масштабирования 12 в Картах Bing.

**До: Карты Bing**

В следующем примере кода показано, как отобразить Карту Bing по центру и увеличить ее расположение.

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
            map = new Microsoft.Maps.Map('#myMap', {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(40.747, -73.985),
                zoom: 12
            });
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

При выполнении этого кода в браузере отобразится показанная ниже карта:

![Карта в службе "Карты Bing"](media/migrate-bing-maps-web-app/bing-maps-load-map.jpg)

**После: Azure Maps**

В следующем примере кода показано, как загрузить карту с тем же представлением в Azure Maps, а также с элементами управления стилем карт и кнопками масштабирования.

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

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {
                //Add zoom and map style controls to top right of map.
                map.controls.add([
                        new atlas.control.StyleControl(),
                        new atlas.control.ZoomControl()
                    ], {
                        position: 'top-right'
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

![Карта в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-load-map.jpg)

Подробную документацию по настройке и использованию элементов управления картой Azure Maps в веб-приложении см. [здесь](how-to-use-map-control.md).

> [!TIP]
> Azure Maps публикует минифицированные и не минифицированные версии пакета SDK. Удалите `.min` из имен файлов. Не минифицированная версия полезна при отладке проблем, но в рабочей среде обязательно используйте минифицированную версию, преимуществом которой является меньший размер файла.

**Дополнительные ресурсы**

* Azure Maps также предоставляет элементы управления навигацией для поворота и наклона представления карты, как описано [здесь](map-add-controls.md).

### <a name="localizing-the-map"></a>Локализация карты

Если ваша аудитория распределена по нескольким странам или говорит на разных языках, необходима локализация.

**До: Карты Bing**

Чтобы локализовать карты Bing, следует задать язык и регион с помощью параметров `setLang` и `UR`, добавляемых в ссылку на API в теге `<script>`. Некоторые функции в Картах Bing доступны только на определенных рынках. Рынок, к которому принадлежит пользователь, указывается с помощью параметра `setMkt`.

```html
<script type="text/javascript" src="https://www.bing.com/api/maps/mapcontrol?callback=initMap&setLang=[language_code]&setMkt=[market]&UR=[region_code]" async defer></script>
```

Ниже приведен пример использования Карт Bing для языка "fr-FR".

![Карта в локализованной службе "Карты Bing"](media/migrate-bing-maps-web-app/bing-maps-localized-map.jpg)

**После: Azure Maps**

Azure Maps предоставляет только возможность настройки языка и регионального представления карты. Возможности не ограничиваются с помощью параметра, определяющего рынок. Настроить язык и региональное представление карты можно двумя разными способами. Первый вариант — добавить эту информацию в глобальное пространство имен `atlas`. Это приведет к тому, что все экземпляры элементов управления картой в вашем приложении будут по умолчанию использовать эти параметры. В приведенном ниже примере кода устанавливается французский язык (fr-FR) и региональное представление со значением `"Auto"`.

```javascript
atlas.setLanguage('fr-FR');
atlas.setView('auto');
```

Второй вариант — передать эту информацию в параметры карты при загрузке карты следующим образом:

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
> В Azure Maps можно загрузить несколько экземпляров карт на одной странице с разными настройками языка и региональными параметрами. Кроме того, можно обновить эти параметры на карте после их загрузки. Подробный список поддерживаемых языков в Azure Maps можно найти [здесь](./supported-languages.md).

Ниже приведен пример Azure Maps с установленным языком fr и регионом пользователя fr-FR.

![Локализованная карта Azure Maps](media/migrate-bing-maps-web-app/bing-maps-localized-map.jpg)

### <a name="setting-the-map-view"></a>Настройка представления карты

Динамические карты в Картах Bing и Azure Maps можно программно перемещать в новые географические расположения путем вызова соответствующих функций в JavaScript. В приведенных ниже примерах показано, как сделать так, чтобы на карте отображались спутниковые аэроснимки, центрировать карту по местоположению с использованием координат (долгота: −111,0225; широта: 35,0272) и изменить уровень масштабирования на 15 в Картах Bing.

> [!NOTE]
> В Картах Bing используются фрагменты размером 256 пикселей, а в Azure Maps — более крупные фрагменты размером 512 пикселей. Благодаря этому сокращается количество сетевых запросов, необходимых Azure Maps для загрузки той же области карты, что и в Картах Bing. Однако из-за того, как пирамиды фрагментов работают в элементах управления карты, большие фрагменты в Azure Maps означают, что для достижения той же видимой области, что и на карте в Картах Bing, при использовании Azure Maps необходимо вычесть 1 из уровня масштабирования, используемого в Картах Bing.

**До: Карты Bing**

Элемент управления в Картах Bing можно переместить программным способом с помощью функции `setView`, которая позволяет указать центр карты и уровень масштабирования.

```javascript
map.setView({
    mapTypeId: Microsoft.Maps.MapTypeId.aerial,
    center: new Microsoft.Maps.Location(35.0272, -111.0225),
    zoom: 15
});
```

![Карты Bing с заданным представлением карты](media/migrate-bing-maps-web-app/bing-maps-set-map-view.jpg)

**После: Azure Maps**

В Azure Maps положение карты можно изменить программно с помощью функции `setCamera` карты, а стиль карты — с помощью функции `setStyle`. Обратите внимание, что координаты в Azure Maps находятся в формате "долгота, широта", а из значения уровня масштаба вычитается 1.

```javascript
map.setCamera({
    center: [-111.0225, 35.0272],
    zoom: 14
});

map.setStyle({
    style: 'satellite_with_roads'
});
```

![Azure Maps с заданным представлением карты](media/migrate-bing-maps-web-app/azure-maps-set-map-view.jpg)

**Дополнительные ресурсы**

-   [Выбор стиля карты](./choose-map-style.md)
-   [Поддерживаемые стили карт](./supported-map-styles.md)

### <a name="adding-a-pushpin"></a>Добавление вешки

В Azure Maps данные точек можно визуализировать на карте несколькими способами.

* Метки HTML — отрисовывают точки с помощью традиционных элементов DOM. Метки HTML поддерживают перетаскивание.
* Слой символов — отрисовывает точки со значками или текстом в контексте WebGL.
* Пузырьковый слой — отрисовывает точки в виде окружностей на карте. Радиусы окружностей можно масштабировать на основе свойств данных.

Слои символов и пузырьков отображаются в контексте WebGL и могут визуализировать очень большие наборы точек на карте. Данные для этих слоев должны храниться в источнике данных. Источники данных и слои отрисовки необходимо добавлять на карту после запуска события `ready`. Метки HTML отображаются как элементы модели DOM на странице и не используют источник данных. Чем больше элементов DOM содержит страница, тем медленнее она будет обрабатываться. При отрисовке более чем нескольких сотен точек на карте рекомендуется использовать один из слоев отрисовки.

В приведенных ниже примерах на карту в точке координат (долгота: –0,2; широта: 51,5) добавляется метка с наложенным поверх числом 10.

**До: Карты Bing**

При использовании Карт Bing маркеры добавляются на карту с помощью класса `Microsoft.Maps.Pushpin`*. Затем с помощью одной из двух функций на карту добавляются вешки.

Первая функция предназначена для создания слоя, вставки в него вешки и добавления его в свойство `layers` карты.

```javascript
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(51.5, -0.2), {
    text: '10'
});

var layer = new Microsoft.Maps.Layer();
layer.add(pushpin);
map.layers.insert(layer);
```

Вторая — для добавления его с помощью свойства `entities` карты. Эта функция отмечена как нерекомендуемая в документации по Картам Bing версии 8, однако она осталась частично применимой для основных сценариев.

```javascript
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(51.5, -0.2), {
    text: '10'
});

map.entities.add(pushpin);
```

![Добавление вешки в Картах Bing](media/migrate-bing-maps-web-app/bing-maps-add-pushpin.jpg)

**После: использование меток HTML в Azure Maps**

В Azure Maps метки HTML удобно использовать для отображения точки на карте. Их рекомендовано использовать для простых приложений, которые должны отображать небольшое количество точек на карте. Чтобы использовать метку HTML, создайте экземпляр класса `atlas.HtmlMarker`, задайте параметры текста и положения, а затем добавьте на карту метку с помощью функции `map.markers.add`.

```javascript
//Create a HTML marker and add it to the map.
map.markers.add(new atlas.HtmlMarker({
    text: '10',
    position: [-0.2, 51.5]
}));
```

![Azure Maps с добавленной меткой](media/migrate-bing-maps-web-app/azure-maps-add-pushpin.jpg)

**После: использование слоя символов в Azure Maps**

При использовании слоя символов данные необходимо добавить в источник данных, а источник данных — присоединить к слою. Кроме того, источник данных и слой нужно добавить на карту после запуска события `ready`. Чтобы отобразить уникальное текстовое значение над символом, текстовая информация должна храниться как свойство точки данных, а на это свойство должен ссылаться параметр `textField` слоя. Этот способ немного сложнее, чем использование меток HTML, но он значительно повышает производительность.

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

![Azure Maps с добавленным слоем символов](media/migrate-bing-maps-web-app/azure-maps-add-pushpin.jpg)

**Дополнительные ресурсы**

-   [Создание источника данных](./create-data-source-web-sdk.md)
-   [Добавление слоя символов](./map-add-pin.md)
-   [Добавление слоя пузырьков](./map-add-bubble-layer.md)
-   [Данные точек кластера](./clustering-point-data-web-sdk.md)
-   [Добавление меток HTML](./map-add-custom-html.md)
-   [Использование стилистических выражений на основе данных](./data-driven-style-expressions-web-sdk.md)
-   [Параметры значков слоя символов](/javascript/api/azure-maps-control/atlas.iconoptions)
-   [Параметры текста слоя символов](/javascript/api/azure-maps-control/atlas.textoptions)
-   [Класс метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
-   [Параметры метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-custom-pushpin"></a>Добавление пользовательской вешки

Для представления точек на карте можно использовать пользовательские изображения. В приведенных ниже примерах используется следующее пользовательское изображение для отображения точки на карте по координатам (широта: 51,5, долгота: −0,2). Оно смещает позицию маркера таким образом, чтобы точка значка вешки была правильно размещена на карте.

| ![Добавление вешки в Azure Maps](media/migrate-bing-maps-web-app/yellow-pushpin.png)|
|:-----------------------------------------------------------------------:|
| yellow-pushpin.png                                                        |

**До: Карты Bing**

В Картах Bing пользовательский маркер создается путем передачи URL-адреса изображения в параметры `icon` вешки. Параметр `anchor` используется, чтобы сопоставить точку изображения вешки с координатами на карте. Значение привязки в Картах Bing относительно левого верхнего угла изображения.

```javascript
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(51.5, -0.2), {
    icon: 'ylw-pushpin.png',
    anchor: new Microsoft.Maps.Point(5, 30)
});

var layer = new Microsoft.Maps.Layer();
layer.add(pushpin);
map.layers.insert(layer);
```

![Карты Bing с добавленной пользовательский вешкой](media/migrate-bing-maps-web-app/bing-maps-add-custom-pushpin.jpg)

**После: использование меток HTML в Azure Maps**

Чтобы настроить метку HTML в Azure Maps, можно передать в параметр `htmlContent` метки объект `string` или `HTMLElement` HTML. В Azure Maps параметр `anchor` используется для указания относительной позиции метки относительно координаты позиции с помощью одной из девяти определенных опорных точек: "центр", "внизу", "слева", "справа", "вверху слева", "вверху справа", "внизу слева", "внизу справа" (в коде используются английские эквиваленты). Содержимое привязывается, и для него по умолчанию устанавливается значение "bottom", соответствующее нижней центральной точке содержимого HTML. Чтобы упростить перенос кода из Карт Bing, установите для привязки значение top-left (вверху слева), а затем используйте параметр `offset` с тем же смещением, которое использовалось в Картах Bing. Смещение в Azure Maps выполняется в противоположном направлении по отношению к Картам Bing, поэтому умножьте его на минус единицу.

> [!TIP]
> Добавьте `pointer-events:none` в качестве стиля в содержимое HTML, чтобы отключить поведение перетаскивания по умолчанию в Microsoft Edge, при котором будет отображаться нежелательный значок.

```html
map.markers.add(new atlas.HtmlMarker({
    htmlContent: '<img src="ylw-pushpin.png" style="pointer-events: none;" />',
    anchor: 'top-left',
    pixelOffset: [-5, -30],
    position: [-0.2, 51.5]
}));
```

![Azure Maps с добавленной пользовательской меткой](media/migrate-bing-maps-web-app/azure-maps-add-custom-marker.jpg)

**После: использование слоя символов в Azure Maps**

Слои символов в Azure Maps также поддерживают пользовательские изображения, но сначала необходимо загрузить изображение в ресурсы карты и назначить ему уникальный идентификатор. Затем слой символов сможет ссылаться на этот идентификатор. Символ может быть смещен для выравнивания в соответствии с нужной точкой на изображении с помощью значка `offset`. В Azure Maps параметр `anchor` используется для указания относительной позиции символа относительно координаты позиции с помощью одной из девяти определенных опорных точек: "центр", "внизу", "слева", "справа", "вверху слева", "вверху справа", "внизу слева", "внизу справа" (в коде используются английские эквиваленты). Содержимое привязывается, и для него по умолчанию устанавливается значение "bottom", соответствующее нижней центральной точке содержимого HTML. Чтобы упростить перенос кода из Карт Bing, установите для привязки значение top-left (вверху слева), а затем используйте параметр `offset` с тем же смещением, которое использовалось в Картах Bing. Смещение в Azure Maps выполняется в противоположном направлении по отношению к Картам Bing, поэтому умножьте его на минус единицу.

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
                map.imageSprite.add('my-yellow-pin', 'ylw-pushpin.png').then(function () {

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

![Карты Bing с добавленным пользовательским слоем символов](media/migrate-bing-maps-web-app/azure-maps-add-custom-symbol-layer.jpg)

> [!TIP]
> Чтобы создать расширенную пользовательскую отрисовку точек, используйте одновременно несколько слоев отрисовки. Например, если требуется несколько вешек с одинаковым значком на разных цветных окружностях, вместо того чтобы создавать множество изображений для каждого цвета, наложите слой символов поверх слоя пузырьков таким образом, чтобы эти слои ссылались на один и тот же источник данных. Это будет намного эффективнее, чем создавать и сохранять на карте множество различных изображений.

**Дополнительные ресурсы**

-   [Создание источника данных](./create-data-source-web-sdk.md)
-   [Добавление слоя символов](./map-add-pin.md)
-   [Добавление меток HTML](./map-add-custom-html.md)
-   [Использование стилистических выражений на основе данных](./data-driven-style-expressions-web-sdk.md)
-   [Параметры значков слоя символов](/javascript/api/azure-maps-control/atlas.iconoptions)
-   [Параметры текста слоя символов](/javascript/api/azure-maps-control/atlas.textoptions)
-   [Класс метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarker)
-   [Параметры метки HTML](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

### <a name="adding-a-polyline"></a>Добавление ломаной линии

Ломаные линии используются для представления линий или траекторий на карте. В приведенных ниже примерах показано, как создать пунктирную ломаную линию на карте.

**До: Карты Bing**

В картах Bing класс Polyline принимает массив расположений и набор параметров.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Create the polyline.
var polyline = new Microsoft.Maps.Polyline([
        center,
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude - 1),
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude + 1)
    ], {
        strokeColor: 'red',
        strokeThickness: 4,
        strokeDashArray: [3, 3]
    });

//Add the polyline to the map using a layer.
var layer = new Microsoft.Maps.Layer();
layer.add(polyline);
map.layers.insert(layer);
```

![Ломаная линия в Картах Bing](media/migrate-bing-maps-web-app/bing-maps-line.jpg)

**После: Azure Maps**

В Azure Maps ломаные линии называются более распространенными геопространственными терминами: объектами `LineString` или `MultiLineString`. Эти объекты можно добавлять в источник данных и отображать с помощью слоя линий. Параметры для цвета штриха, ширины и массива штрихов практически идентичны на обеих платформах.

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

![Линия Azure Maps](media/migrate-bing-maps-web-app/azure-maps-line.jpg)

**Дополнительные ресурсы**

-   [Добавление линий на карту](./map-add-line-layer.md)
-   [Параметры слоя линий](/javascript/api/azure-maps-control/atlas.linelayeroptions)
-   [Использование стилистических выражений на основе данных](./data-driven-style-expressions-web-sdk.md)

### <a name="adding-a-polygon"></a>Добавление многоугольника

Многоугольники используются для представления области на карте. Azure Maps и Карты Bing обеспечивают очень похожую поддержку многоугольников. В приведенных ниже примерах показано, как создать многоугольник, который образует треугольник на основе центральной координаты на карте.

**До: Карты Bing**

В картах Bing класс Polygon принимает массив координат или координатные кольца и набор параметров.

```javascript
//Get the center of the map.
var center = map.getCenter();

//Create the polygon.
var polygon = new Microsoft.Maps.Polygon([
        center,
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude - 1),
        new Microsoft.Maps.Location(center.latitude - 0.5, center.longitude + 1),
        center
    ], {
        fillColor: 'rgba(0, 255, 0, 0.5)',
        strokeColor: 'red',
        strokeThickness: 2
    });

//Add the polygon to the map using a layer.
var layer = new Microsoft.Maps.Layer();
layer.add(polygon);
map.layers.insert(layer);
```

![Многоугольник в Картах Bing](media/migrate-bing-maps-web-app/azure-maps-polygon.jpg)

**После: Azure Maps**

В Azure Maps объекты Polygon и MultiPolygon можно добавить в источник данных и отобразить на карте с помощью слоев. Площадь многоугольника можно отобразить на слое многоугольников. Контур многоугольника можно визуализировать с помощью слоя линий.

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

![Многоугольник в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-polygon.jpg)

**Дополнительные ресурсы**

-   [Добавление многоугольника на карту](./map-add-shape.md#use-a-polygon-layer)
-   [Добавление окружности на карту](./map-add-shape.md#add-a-circle-to-the-map)
-   [Параметры слоя многоугольников](/javascript/api/azure-maps-control/atlas.polygonlayeroptions)
-   [Параметры слоя линий](/javascript/api/azure-maps-control/atlas.linelayeroptions)
-   [Использование стилистических выражений на основе данных](./data-driven-style-expressions-web-sdk.md)

### <a name="display-an-infobox"></a>Отображение информационного поля

Дополнительные сведения для сущности можно отобразить на карте с помощью класса `Microsoft.Maps.Infobox` в Картах многоугольников и класса `atlas.Popup` в Azure Maps. В приведенных ниже примерах на карту добавляется вешка или маркер, а при щелчке по ним отображается информационное поле или всплывающее окно.

**До: Карты Bing**

При использовании Карт Bing информационное поле создается с помощью конструктора `Microsoft.Maps.Infobox`.

```javascript
//Add a pushpin where we want to display an infobox.
var pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(47.6, -122.33));

//Add the pushpin to the map using a layer.
var layer = new Microsoft.Maps.Layer();
layer.add(pushpin);
map.layers.insert(layer);

//Create an infobox and bind it to the map.
var infobox = new Microsoft.Maps.Infobox(new Microsoft.Maps.Location(47.6, -122.33), {
    description: '<div style="padding:5px"><b>Hello World!</b></div>',
    visible: false
});
infobox.setMap(map);

//Add a click event to the pushpin to open the infobox.
Microsoft.Maps.Events.addHandler(pushpin, 'click', function () {
    infobox.setOptions({ visible: true });
});
```

![Информационное поле в Картах Bing](media/migrate-bing-maps-web-app/bing-maps-infobox.jpg)

**После: Azure Maps**

В Azure Maps во всплывающем окне можно вывести дополнительные сведения о расположении. Объект HTML `string` или `HTMLElement` можно передать в параметр `content` всплывающего окна. Всплывающие окна могут отображаться независимо от любой фигуры, если это необходимо, поэтому требуется указать значение `position`. Чтобы отобразить всплывающее окно, вызовите функцию `open` и передайте карту, в которой должно отображаться всплывающее окно.

```javascript
//Add a marker to the map that to display a popup for.
var marker = new atlas.HtmlMarker({
    position: [-122.33, 47.6]
});

//Add the marker to the map.
map.markers.add(marker);

//Create a popup.
var popup = new atlas.Popup({
    content: '<div style="padding:10px"><b>Hello World!</b></div>',
    position: [-122.33, 47.6],
    pixelOffset: [0, -35]
});

//Add a click event to the marker to open the popup.
map.events.add('click', marker, function () {
    //Open the popup
    popup.open(map);
});
```

![Всплывающее окно в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-popup.jpg)

> [!NOTE]
> Чтобы сделать то же самое со слоем символов, пузырьков, линией или слоем многоугольников, передайте слой в код события карт вместо метки.

**Дополнительные ресурсы**

-   [Добавление всплывающего окна](./map-add-popup.md)
-   [Всплывающее окно с мультимедийным содержимым](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popup%20with%20Media%20Content)
-   [Всплывающие окна на фигурах](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Popups%20on%20Shapes)
-   [Повторное использование всплывающего окна с несколькими точками](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Reusing%20Popup%20with%20Multiple%20Pins)
-   [Класс Popup](/javascript/api/azure-maps-control/atlas.popup)
-   [Параметры всплывающего окна](/javascript/api/azure-maps-control/atlas.popupoptions)

### <a name="pushpin-clustering"></a>Кластеризация вешек

При визуализации большого количества точек данных на карте точки перекрываются, а сама карта выглядит перегруженной, из-за чего ее будет сложно просматривать и использовать. С помощью кластеризации точек данных можно улучшить удобство работы пользователей, а также повысить производительность. Кластеризация данных точек — это процесс объединения данных точек, находящихся рядом друг с другом, и представления их на карте в виде единой кластеризованной точки данных. По мере того как пользователь масштабирует карту, кластеры разбиваются на отдельные точки данных.

В приведенных ниже примерах загружается канал GeoJSON с данными о землетрясениях за прошлую неделю, после чего он добавляется на карту. Кластеры отрисовываются как масштабируемые и цветные окружности в зависимости от количества содержащихся в них точек.

> [!NOTE]
> Существует несколько различных алгоритмов кластеризации вешек. В Картах Bing используется простая функция на основе сетки, а в Azure Maps — более сложные и визуально привлекательные методы кластеризации на основе точек.

**До: Карты Bing**

В Картах Bing данные GeoJSON можно загрузить с помощью модуля GeoJSON. Вешки можно объединить в кластер, загрузив их в модуль кластеризации и использовав содержащийся в нем слой кластеризации.

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
            map = new Microsoft.Maps.Map(document.getElementById('myMap'), {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(20, -160),
                zoom: 2
            });

            //Load the GeoJSON and Clustering modules.
            Microsoft.Maps.loadModule(['Microsoft.Maps.GeoJson', 'Microsoft.Maps.Clustering'], function () {

                //Load the GeoJSON data from a URL.
                Microsoft.Maps.GeoJson.readFromUrl(earthquakeFeed, function (pins) {

                    //Create a ClusterLayer with options and add it to the map.
                    clusterLayer = new Microsoft.Maps.ClusterLayer(pins, {
                        clusteredPinCallback: createCustomClusteredPin,
                        gridSize: 100
                    });

                    map.layers.insert(clusterLayer);
                });
            });
        }

        //A function that defines how clustered pins are rendered.
        function createCustomClusteredPin(cluster) {
            //Get the number of pushpins in the cluster
            var clusterSize = cluster.containedPushpins.length;

            var radius = 20;    //Default radius to 20 pixels.
            var fillColor = 'lime';     //Default to lime green.

            if (clusterSize >= 750) {
                radius = 40;   //If point_count >= 750, radius is 40 pixels.
                fillColor = 'red';    //If the point_count >= 750, color is red.
            } else if (clusterSize >= 100) {
                radius = 30;    //If point_count >= 100, radius is 30 pixels.
                fillColor = 'yellow';    //If the point_count >= 100, color is yellow.
            }

            //Create an SVG string of a circle with the specified radius and color.
            var svg = ['<svg xmlns="http://www.w3.org/2000/svg" width="', (radius * 2), '" height="', (radius * 2), '">',
                '<circle cx="', radius, '" cy="', radius, '" r="', radius, '" fill="', fillColor, '"/>',
                '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle" style="font-size:12px;font-family:arial;fill:black;" >{text}</text>',
                '</svg>'];

            //Customize the clustered pushpin using the generated SVG and anchor on its center.
            cluster.setOptions({
                icon: svg.join(''),
                anchor: new Microsoft.Maps.Point(radius, radius),
                textOffset: new Microsoft.Maps.Point(0, radius - 8) //Subtract 8 to compensate for height of text.
            });
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Кластеризация Карт Bing](media/migrate-bing-maps-web-app/bing-maps-clustering.jpg)

**После: Azure Maps**

В Azure Maps добавление данных и управление ими осуществляется с помощью источника данных. Слои подключаются к источникам данных и визуализируют данные в них. Класс `DataSource` в Azure Maps предоставляет несколько параметров кластеризации.

* `cluster` — указывает источнику данных выполнить кластеризацию данных точек. 
* `clusterRadius` — радиус в пикселях, используемый для кластеризации точек.
* `clusterMaxZoom` — максимальный уровень масштаба, в котором происходит кластеризация. При дополнительном увеличении все точки отрисовываются как символы.
* `clusterProperties` — определяет пользовательские свойства, которые рассчитываются с помощью выражений для всех точек в каждом кластере и добавляются в свойства каждой точки кластера.

Если кластеризация включена, источник данных будет отсылать кластеризованные и некластеризованные точки данных слоям для отрисовки. Источник данных способен выполнять кластеризацию сотен тысяч точек данных. Кластеризованная точка данных имеет следующие свойства.

| Имя свойства               | Тип    | Описание                                    |
|-----------------------------|---------|------------------------------------------------|
| `cluster`                   | Логическое | Указывает, представляет ли компонент кластер.     |
| `cluster_id`                | строка  | Уникальный идентификатор кластера, который можно использовать с функциями `getClusterExpansionZoom`, `getClusterChildren` и `getClusterLeaves` класса `DataSource`. |
| `point_count`               | number  | Число точек, содержащихся в кластере.     |
| `point_count_abbreviated`   | строка  | Строка, которая сокращает значение `point_count`, если оно длинное (например, 4000 преобразуется в 4K). |

Класс `DataSource` имеет следующую вспомогательную функцию для доступа к дополнительным сведениям о кластере с помощью `cluster_id`.

| Компонент       | Возвращаемый тип        | Описание     |
|----------------|--------------------|-----------------|
| `getClusterChildren(clusterId: number)`                              | `Promise<Feature<Geometry, any> | Shape>` | Извлекает дочерние элементы заданного кластера на следующем уровне масштабирования. Эти дочерние элементы могут быть сочетанием фигур и подкластеров. Подкластеры будут представлять собой компоненты со свойствами, соответствующими свойствам кластера. |
| `getClusterExpansionZoom(clusterId: number)`                         | `Promise<number>`                            | Вычисляет уровень масштабирования, при котором кластер начинает расширяться или разбиваться на части.    |
| `getClusterLeaves(clusterId: number, limit: number, offset: number)` | `Promise<Feature<Geometry, any> | Shape>` | Извлекает все точки в кластере. Задайте `limit`, чтобы вернуть подмножество точек, и используйте `offset`, чтобы пролистать страницы с точками.    |

При отрисовке кластеризованных данных на карте зачастую проще использовать два или более слоев. В приведенном ниже примере используются три слоя — слой пузырьков для визуализации масштабируемых цветных окружностей на основе размера кластеров, уровень символов для отрисовки размера кластера в виде текста и второй уровень символов для визуализации некластеризованных точек. Существует много других способов отображения кластеризованных данных в Azure Maps, которые описаны в [этой статье](./clustering-point-data-web-sdk.md).

Данные GeoJSON можно импортировать непосредственно в Azure Maps с помощью функции `importDataFromUrl` в классе `DataSource`.

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

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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
                        filter: ['has', 'point_count'] //Only rendered data points that have a point_count property, which clusters do.
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

![Кластеризация в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-clustering.jpg)

**Дополнительные ресурсы**

-   [Добавление слоя символов](./map-add-pin.md)
-   [Добавление слоя пузырьков](./map-add-bubble-layer.md)
-   [Данные точек кластера](./clustering-point-data-web-sdk.md)
-   [Использование стилистических выражений на основе данных](./data-driven-style-expressions-web-sdk.md)

### <a name="add-a-heat-map"></a>Добавление тепловой карты

Тепловая карта, которую также называют картой точек плотности, это тип визуализации данных для представления плотности данных с помощью диапазона цветов. Такие визуализации часто используются для отображения "горячих точек" данных на карте, и они особенно полезны, чтобы преобразовывать для просмотра набор данных больших точек.

В приведенных ниже примерах загружается веб-канал GeoJSON с данными обо всех землетрясениях за прошлый месяц из Геологической службы США, которые отображаются в виде тепловой карты.

**До: Карты Bing**

В Картах Bing для создания тепловой карты необходимо загрузить модуль тепловых карт. Аналогично, для обеспечения поддержки данных GeoJSON следует загрузить модуль GeoJSON.

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
            map = new Microsoft.Maps.Map(document.getElementById('myMap'), {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(20, -160),
                zoom: 2,
                mapTypeId: Microsoft.Maps.MapTypeId.aerial
            });

            //Load the GeoJSON and HeatMap modules.
            Microsoft.Maps.loadModule(['Microsoft.Maps.GeoJson', 'Microsoft.Maps.HeatMap'], function () {

                //Load the GeoJSON data from a URL.
                Microsoft.Maps.GeoJson.readFromUrl(earthquakeFeed, function (shapes) {

                    //Create a heat map and add it to the map.
                    var heatMap = new Microsoft.Maps.HeatMapLayer(shapes, {
                        opacity: 0.65,
                        radius: 10
                    });
                    map.layers.insert(heatMap);
                });
            });
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![Тепловая карта Карт Bing](media/migrate-bing-maps-web-app/bing-maps-heatmap.jpg)

**После: Azure Maps**

В Azure Maps загрузите данные GeoJSON в источник данных и подключите источник к слою тепловой карты. Данные GeoJSON можно импортировать непосредственно в Azure Maps с помощью функции `importDataFromUrl` в классе `DataSource`.

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
                style: 'satellite_with_roads',

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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

![Тепловая карта Azure Maps](media/migrate-bing-maps-web-app/azure-maps-heatmap.jpg)

**Дополнительные ресурсы**

-   [Добавление слоя тепловой карты](./map-add-heat-map-layer.md)
-   [Класс слоя тепловой карты](/javascript/api/azure-maps-control/atlas.layer.heatmaplayer)
-   [Параметры слоя тепловой карты](/javascript/api/azure-maps-control/atlas.heatmaplayeroptions)
-   [Использование стилистических выражений на основе данных](./data-driven-style-expressions-web-sdk.md)

### <a name="overlay-a-tile-layer"></a>Наложение слоя фрагментов

Они позволяют накладывать большие изображения, разбитые на небольшие фрагменты, которые соответствуют разбиению карты. Это распространенный способ наложения больших изображений или очень больших наборов данных.

В приведенных ниже примерах накладывается слой фрагментов радара погоды из лаборатории окружающей среды Университета штата Айова, для которого используется схема именования системы фрагментов с x, y и увеличением.

**До: Карты Bing**

В Картах Bing слои фрагментов можно создавать с помощью класса `Microsoft.Maps.TileLayer`.

```javascript
var weatherTileLayer = new Microsoft.Maps.TileLayer({
    mercator: new Microsoft.Maps.TileSource({
        uriConstructor: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{zoom}/{x}/{y}.png'
    })
});
map.layers.insert(weatherTileLayer);
```

![Взвешенная тепловая карта в службе "Карты Bing"](media/migrate-bing-maps-web-app/bing-maps-weighted-heatmap.jpg)

**После: Azure Maps**

В Azure Maps слой фрагментов можно добавить на карту во многом так же, как и любой другой слой. Форматированный URL-адрес с заполнителями значений x, y, масштаба (`{x}`, `{y}`, `{z}` соответственно) используется для указания уровня доступа к фрагментам. Слои фрагментов в Azure Maps также поддерживают заполнители `{quadkey}`, `{bbox-epsg-3857}` и `{subdomain}`.

> [!TIP]
> В Azure Maps слои можно легко отобразить под другими слоями, в том числе под базовыми слоями карт. Часто желательно отображать слои фрагментов под метками карты, чтобы их можно было легко читать. Функция `map.layers.add` принимает второй параметр — идентификатор второго слоя, в который следует вставить новый слой. Чтобы вставить слой фрагментов под метками карты, можно использовать следующий код:
>
> `map.layers.add(myTileLayer, "labels");`

```javascript
//Create a tile layer and add it to the map below the label layer.
map.layers.add(new atlas.layer.TileLayer({
    tileUrl: 'https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png',
    opacity: 0.8,
    tileSize: 256
}), 'labels');
```

![Взвешенная тепловая карта Azure Maps](media/migrate-bing-maps-web-app/azure-maps-weighted-heatmap.jpg)

> [!TIP]
> Запросы на фрагмент можно записать с помощью параметра `transformRequest` карты. Это позволит при необходимости изменить или добавить заголовки в запрос.

**Дополнительные ресурсы**

-   [Добавление слоев фрагмента карты](./map-add-tile-layer.md)
-   [Класс слоя фрагментов](/javascript/api/azure-maps-control/atlas.layer.tilelayer)
-   [Параметры слоя фрагментов](/javascript/api/azure-maps-control/atlas.tilelayeroptions)

### <a name="show-traffic-data"></a>Отображение данных о трафике

Данные дорожного движения можно перенести на карту как в Картах Bing, так и в Azure Maps.

**До: Карты Bing**

Данные дорожного движения в Картах Bing можно наложить на карту с помощью модуля дорожного движения.

```javascript
Microsoft.Maps.loadModule('Microsoft.Maps.Traffic', function () {
    var manager = new Microsoft.Maps.Traffic.TrafficManager(map);
    manager.show();
});
```

![Дорожное движение в Картах Bing](media/migrate-bing-maps-web-app/bing-maps-traffic.jpg)

**После: Azure Maps**

Azure Maps предоставляет несколько различных вариантов отображения дорожного движения. Дорожно-транспортные происшествия, такие как закрытие дорог и аварии, можно отображать на карте в виде значков. На карте можно наложить транспортный поток с использованием дорог с цветовой кодировкой, а цвета можно изменять в соответствии с ограничениями скорости, относительно нормальной ожидаемой или абсолютной задержки. Данные о дорожно-транспортных происшествиях в Azure Maps обновляются каждую минуту, а данные о дорожном движении — каждые 2 минуты.

```javascript
map.setTraffic({
    incidents: true,
    flow: 'relative'
});
```

![Данные дорожного движения в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-traffic.jpg)

Если щелкнуть один из значков дорожного движения в Azure Maps, во всплывающем окне отобразятся дополнительные сведения.

![Всплывающее окно с данными о дорожном движении в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-traffic-popup.jpg)

**Дополнительные ресурсы**

-   [Отображение данных дорожного движения на карте](./map-show-traffic.md)
-   [Параметры наложения сведений о дорожном движении](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Traffic%20Overlay%20Options)
-   [Элемент управления для дорожного движения](https://azuremapscodesamples.azurewebsites.net/?sample=Traffic%20controls)

### <a name="add-a-ground-overlay"></a>Добавление наземного наложения

Карты Bing и Azure Maps поддерживают наложение изображений с геоссылками на карте таким образом, что они перемещаются и масштабируются по мере сдвига и масштабирования карты. В Картах Bing они называются наземными наложениями, а в Azure Maps — слоями изображений. Они отлично подходят для создания планов этажей, наложения старых карт или изображений, снятых с помощью дронов.

**До: Карты Bing**

При создании наземного наложения в Картах Bing необходимо указать URL-адрес изображения, которое необходимо наложить, и ограничивающий прямоугольник для выделения изображения на карте. В этом примере показано, как наложить на карту изображение [карты Ньюарка, Нью-Джерси, (1922 г.)](https://www.lib.utexas.edu/maps/historical/newark_nj_1922.jpg).

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
            map = new Microsoft.Maps.Map(document.getElementById('myMap'), {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(40.740, -74.18),
                zoom: 12
            });

            var overlay = new Microsoft.Maps.GroundOverlay({
                //Create a LocationRect from the edges of the bounding box; north, west, south, east.
                bounds: Microsoft.Maps.LocationRect.fromEdges(40.773941, -74.22655, 40.712216, -74.12544),
                imageUrl: 'newark_nj_1922.jpg'
            });
            map.layers.insert(overlay);
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

При выполнении этого кода в браузере отобразится показанная ниже карта:

![Наземное наложение в Картах Bing](media/migrate-bing-maps-web-app/bing-maps-ground-overlay.jpg)

**После: Azure Maps**

В Azure Maps изображения с геоссылками можно накладывать с помощью класса `atlas.layer.ImageLayer`. Этому классу требуется URL-адрес изображения и набор координат для четырех углов изображения. Изображение должно размещаться либо в том же домене, либо поддерживать CORS.

> [!TIP]
> Если у вас есть только информация о координатах севера, юга, востока, запада и повороте, а не координаты каждого угла изображения, можно использовать статическую функцию [atlas.layer.ImageLayer.getCoordinatesFromEdges](/javascript/api/azure-maps-control/atlas.layer.imagelayer#getcoordinatesfromedges-number--number--number--number--number-).

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

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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

![Наземное наложение в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-ground-overlay.jpg)

**Дополнительные ресурсы**

-   [Наложение изображения](./map-add-image-layer.md)
-   [Класс слоя изображений](/javascript/api/azure-maps-control/atlas.layer.imagelayer)

### <a name="add-kml-data-to-the-map"></a>Добавление данных KML на карту

Azure Maps и Карты Bing позволяют импортировать и преобразовать для просмотра данные KML, KMZ, GeoRSS, GeoJSON и Well-Known Text (WKT) на карте. Azure Maps также поддерживает файлы GPX, GML, пространственные файлы CSV, веб-службы сопоставления (WMS), веб-службы фрагментирования (WMTS) и веб-службы компонентов (WFS).

**До: Карты Bing**

При выполнении этого кода в браузере отобразится показанная ниже карта:

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
            map = new Microsoft.Maps.Map('#myMap', {
                credentials: '<Your Bing Maps Key>',
                center: new Microsoft.Maps.Location(40.747, -73.985),
                zoom: 12
            });
                
            Microsoft.Maps.loadModule('Microsoft.Maps.GeoXml', function () {
                var callback = function (dataset) {
                    if (dataset.shapes) {
                        var l = new Microsoft.Maps.Layer();
                        l.add(dataset.shapes);
                        map.layers.insert(l);
                    }
                    if (dataset.layers) {
                        for (var i = 0, len = dataset.layers.length; i < len; i++) {
                            map.layers.insert(dataset.layers[i]);
                        }
                    }
                };
                Microsoft.Maps.GeoXml.readFromUrl('myKMLFile.kml', { error: function (msg) { alert(msg); } }, callback);
            });                
        }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>
```

![KML в службе "Карты Bing"](media/migrate-bing-maps-web-app/bing-maps-kml.jpg)

**После: Azure Maps**

В Azure Maps GeoJSON — это основной формат данных, используемый в веб-пакете SDK. Дополнительные форматы пространственных данных можно легко интегрировать в [модуль пространственного ввода-вывода](/javascript/api/azure-maps-spatial-io/). Этот модуль содержит функции для чтения и записи пространственных данных, а также уровень простых данных, с помощью которого можно легко показать данные из любого из перечисленных форматов пространственных данных. Для чтения данных в файле пространственных данных передайте URL-адрес или необработанные данные в виде строки или большого двоичного объекта в функцию `atlas.io.read`. При этом будут возвращены все проанализированные данные из файла, которые затем можно добавить на карту. Формат KML более сложный, чем большинство форматов пространственных данных, так как содержит гораздо больше сведений о стилях. Класс `SpatialDataLayer` поддерживает отрисовку большинства из этих стилей, но перед загрузкой данных компонентов изображения значков должны быть загружены на карту, а наземные наложения необходимо добавлять на карту отдельно. При загрузке данных по URL-адресу данные должны быть размещены на конечной точке с поддержкой COR, либо в качестве параметра функции чтения должна передаваться прокси-служба.

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

    <!-- Add reference to the Azure Maps Spatial IO module. -->
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>

    <script type='text/javascript'>
        var map, datasource, layer;

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                view: 'Auto',

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.
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
                atlas.io.read('myKMLFile.kml').then(async r => {
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

![KML в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-kml.jpg)

**Дополнительные ресурсы**

-   [Функция atlas.io.read](/javascript/api/azure-maps-spatial-io/atlas.io#read-string---arraybuffer---blob--spatialdatareadoptions-)
-   [SimpleDataLayer](/javascript/api/azure-maps-spatial-io/atlas.layer.simpledatalayer)
-   [SimpleDataLayerOptions](/javascript/api/azure-maps-spatial-io/atlas.simpledatalayeroptions)

### <a name="add-drawing-tools"></a>Добавление инструментов рисования

Карты Bing и Azure Maps предоставляют модуль, благодаря которому пользователь может рисовать и редактировать фигуры на карте с помощью мыши или другого устройства ввода. Они поддерживают рисование вешек, линий и многоугольников. В Azure Maps можно также рисовать круги и прямоугольники.

**До: Карты Bing**

В Картах Bing модуль `DrawingTools` загружается с помощью функции `Microsoft.Maps.loadModule`. После загрузки можно создать экземпляр класса DrawingTools. Тогда при вызове функции `showDrawingManager` на карту будет добавлена панель инструментов.

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
    <meta http-equiv="x-ua-compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />

    <script type='text/javascript'>
    var map, drawingManager;

    function initMap() {
        map = new Microsoft.Maps.Map('#myMap', {
            credentials: '<Your Bing Maps Key>'
        });

        //Load the DrawingTools module
        Microsoft.Maps.loadModule('Microsoft.Maps.DrawingTools', function () {
            //Create an instance of the DrawingTools class and bind it to the map.
            var tools = new Microsoft.Maps.DrawingTools(map);

            //Show the drawing toolbar and enable editting on the map.
            tools.showDrawingManager(function (manager) {
                //Store a reference to the drawing manager as it will be useful later.
                drawingManager = manager;
            });
        });
    }
    </script>

    <!-- Bing Maps Script Reference -->
    <script src="https://www.bing.com/api/maps/mapcontrol?callback=initMap" async defer></script>
</head>
<body>
    <div id='myMap' style='position:relative;width:600px;height:400px;'></div>
</body>
</html>

```

![Средства рисования в Картах Bing](media/migrate-bing-maps-web-app/bing-maps-drawing-tools.jpg)

**После: Azure Maps**

В Azure Maps требуется загрузить модуль средств рисования, загрузив файлы JavaScript и каскадные таблицы стилей, на которые необходимо ссылаться в приложении. После загрузки карты можно создать экземпляр класса `DrawingManager` и присоединить к нему экземпляр `DrawingToolbar`.

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

    <!-- Add references to the Azure Maps Map Drawing Tools JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.js"></script>
    
    <script type='text/javascript'>
        var map, drawingManager;

        function initMap() {
            //Initialize a map instance.
            map = new atlas.Map('myMap', {
                view: 'Auto',

                //Add your Azure Maps key to the map SDK. Get an Azure Maps key at https://azure.com/maps. NOTE: The primary key should be used as the key.                
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });

            //Wait until the map resources are ready.
            map.events.add('ready', function () {

                //Create an instance of the drawing manager and display the drawing toolbar.
                drawingManager = new atlas.drawing.DrawingManager(map, {
                    toolbar: new atlas.control.DrawingToolbar({ position: 'top-left' })
                });
            });
        }
    </script>
</head>
<body onload="initMap()">
    <div id="myMap" style="position:relative;width:600px;height:400px;"></div>
</body>
</html>
```

![Средства рисования в Azure Maps](media/migrate-bing-maps-web-app/azure-maps-drawing-tools.jpg)

> [!TIP]
> В слоях Azure Maps средства рисования позволяют рисовать фигуры несколькими способами. Например, пользователь может нарисовать многоугольник, щелкая каждую точку, или перетащить указатель мыши, удерживая нажатой ее левую кнопку, чтобы начертить путь. Это можно изменить с помощью параметра `interactionType` класса `DrawingManager`.

**Дополнительные ресурсы**

-   [Документация](./set-drawing-options.md)
-   [Примеры кода](https://azuremapscodesamples.azurewebsites.net/#Drawing-Tools-Module)

## <a name="additional-resources"></a>Дополнительные ресурсы

Ознакомьтесь с [модулями веб-пакета SDK Azure Maps с открытым кодом](open-source-projects.md#open-web-sdk-modules). Эти модули полностью настраиваются и предоставляют массу дополнительных функциональных возможностей.

Ознакомьтесь с примерами кода, связанными с переносом других функций Карт Bing:

**Визуализации данных**

> [!div class="nextstepaction"]
> [Слой контура](https://azuremapscodesamples.azurewebsites.net/?search=contour)

> [!div class="nextstepaction"]
> [Привязка данных](https://azuremapscodesamples.azurewebsites.net/?search=data%20binning)

**Службы**

> [!div class="nextstepaction"]
> [Использование модуля служб Azure Maps](./how-to-use-services-module.md)

> [!div class="nextstepaction"]
> [Поиск достопримечательностей](./map-search-location.md)

> [!div class="nextstepaction"]
> [Получение информации из координаты (обратный геокод)](./map-get-information-from-coordinate.md)

> [!div class="nextstepaction"]
> [Отображение направлений от точки А до точки Б](./map-route.md)

> [!div class="nextstepaction"]
> [Автозаполнение в поисковых запросах с использованием пользовательского интерфейса JQuery](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Search%20Autosuggest%20and%20JQuery%20UI)

Ознакомьтесь с дополнительными сведениями о веб-пакете SDK для Azure Maps.

> [!div class="nextstepaction"]
> [Как использовать библиотеку Map Control в службе "Карты Azure"](how-to-use-map-control.md)

> [!div class="nextstepaction"]
> [Использование модуля служб Azure Maps](how-to-use-services-module.md)

> [!div class="nextstepaction"]
> [Использование модуля средств рисования](set-drawing-options.md)

> [!div class="nextstepaction"]
> [Примеры кода](/samples/browse/?products=azure-maps)

> [!div class="nextstepaction"]
> [Справочная документация по API службы веб-пакета SDK Azure Maps](/javascript/api/azure-maps-control/)

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов для очистки.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о переходе из Карт Bing на Azure Maps.

> [!div class="nextstepaction"]
> [Перенос веб-службы](migrate-from-bing-maps-web-services.md)
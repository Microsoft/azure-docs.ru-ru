---
title: Руководство по Поиск ближайших объектов на карте | Microsoft Azure Maps
description: Руководство по поиску точек интереса на карте. Узнайте, как использовать веб-пакет SDK Azure Maps для добавления возможностей поиска и интерактивных всплывающих окон на карту.
author: anastasia-ms
ms.author: v-stharr
ms.date: 1/15/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-js
ms.openlocfilehash: 31dd1c06b0f17b469454593131ccdc93b45b2446
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98624972"
---
# <a name="tutorial-search-nearby-points-of-interest-using-azure-maps"></a>Руководство по Поиск ближайшей точки интереса с помощью службы "Карты Azure"

В этом руководстве показано, как настроить учетную запись службы "Карты Azure", а затем использовать API службы "Карты Azure" для поиска точки интереса. В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание учетной записи службы Azure Maps
> * получение первичного ключа для учетной записи службы "Карты Azure";
> * создание веб-страницы с помощью API элементов управления картой;
> * использование службы поиска "Карты Azure" для поиска ближайшей точки интереса.

## <a name="prerequisites"></a>Предварительные требования

<a id="createaccount"></a>
<a id="getkey"></a>

1. Войдите на [портал Azure](https://portal.azure.com). Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
2. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](how-to-manage-authentication.md).

<a id="createmap"></a>

## <a name="create-a-new-map"></a>Создание карты

Map Control API является удобной клиентской библиотекой. Этот API позволяет легко интегрировать Maps в веб-приложение. Она скрывает сложность "чистых" вызовов службы REST и ускоряет работу с помощью компонентов, поддерживающих настройку. Чтобы создать статическую HTML-страницу со встроенным API элементов управления картой, сделайте следующее.

1. На локальном компьютере создайте файл **MapSearch.html**.
2. Добавьте в него следующие компоненты HTML.

   ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>Map Search</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

        <!-- Add a reference to the Azure Maps Services Module JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>

        <script>
        function GetMap(){
            //Add Map Control JavaScript code here.
        }
        </script>

        <style>
            html,
            body {
                width: 100%;
                height: 100%;
                padding: 0;
                margin: 0;
            }

            #myMap {
                width: 100%;
                height: 100%;
            }
        </style>
    </head>
    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>
    </html>
    ```

   Обратите внимание на то, что заголовок HTML содержит файлы ресурсов CSS и JavaScript, размещенные в библиотеке Azure Map Control. Обратите внимание на событие `onload` в тексте страницы, которое вызовет функцию `GetMap` после загрузки текста страницы. Функция `GetMap` будет содержать внутренний код JavaScript для доступа к интерфейсам службы Azure Maps.

3. Добавьте следующий код JavaScript к функции `GetMap` HTML-файла. Замените строку `<Your Azure Maps Key>` первичным ключом, скопированным из учетной записи службы Maps.

    ```JavaScript
    //Instantiate a map object
    var map = new atlas.Map("myMap", {
        //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
        authOptions: {
            authType: 'subscriptionKey',
            subscriptionKey: '<Your Azure Maps Key>'
        }
    });
    ```

   Этот сегмент инициирует API Map Control для ключа учетной записи службы Azure Maps. `atlas` — это пространство имен, которое содержит API и связанные визуальные компоненты. `atlas.Map` предоставляет элемент управления для визуальной интерактивной веб-карты.

4. Сохраните изменения в файл и откройте HTML-страницу в браузере. Отображаемая карта — это самая простая карта, которую можно создать, вызвав `atlas.Map`, используя ключ учетной записи.

   ![Просмотр карты](./media/tutorial-search-location/basic-map.png)

5. В функции `GetMap` после инициализации карты добавьте следующий код JavaScript.

    ```JavaScript
    //Wait until the map resources are ready.
    map.events.add('ready', function() {

        //Create a data source and add it to the map.
        datasource = new atlas.source.DataSource();
        map.sources.add(datasource);

        //Add a layer for rendering point data.
        var resultLayer = new atlas.layer.SymbolLayer(datasource, null, {
            iconOptions: {
                image: 'pin-round-darkblue',
                anchor: 'center',
                allowOverlap: true
            },
            textOptions: {
                anchor: "top"
            }
        });

        map.layers.add(resultLayer);
    });
    ```

   В этом сегменте кода на карту добавляется событие `ready`, которое сработает, когда ресурсы карты будут загружены, а карта готова к доступу. В обработчике событий карты `ready` создается источник данных для хранения выходных данных. Слой символов создается и привязывается к источнику данных. Этот уровень определяет способ подготовки к преобразованию результирующих данных в источнике данных. В этом случае результат визуализируется с помощью темно-синего скругленного круга, центрированного по координатам результатов и позволяет другим значкам перекрываться. Слой результатов добавляется к слоям карты.

<a id="usesearch"></a>

## <a name="add-search-capabilities"></a>Добавление возможностей поиска

В этом разделе показано, как с помощью [API поиска](/rest/api/maps/search) службы Maps искать точки интереса на карте. Это RESTful API, предназначенный для разработчиков. Он позволяет находить адреса, объекты и другие географические сведения. Служба поиска назначает широту и долготу для определенного адреса. С помощью описанного ниже **модуля службы** можно искать расположение, используя API поиска службы "Карты Azure".

### <a name="service-module"></a>Модуль службы

1. В обработчике событий карты `ready` создайте URL-адрес службы поиска, добавив приведенный ниже код JavaScript.

    ```JavaScript
   // Use SubscriptionKeyCredential with a subscription key
   var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

   // Use subscriptionKeyCredential to create a pipeline
   var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential);

   // Construct the SearchURL object
   var searchURL = new atlas.service.SearchURL(pipeline); 
   ```

   `SubscriptionKeyCredential` создает политику `SubscriptionKeyCredentialPolicy` для аутентификации HTTP-запросов для Azure Maps с помощью ключа подписки. `atlas.service.MapsURL.newPipeline()` принимает политику `SubscriptionKeyCredential` и создает экземпляр [конвейера](/javascript/api/azure-maps-rest/atlas.service.pipeline). `searchURL` представляет собой URL-адрес к операциям [поиска](/rest/api/maps/search) службы Azure Maps.

2. Затем, чтобы создать поисковый запрос, добавьте следующий блок сценария. В нем используется служба поиска нечетких соответствий, которая является базовым API поиска Службы поиска. Служба поиска нечетких соответствий обрабатывает большинство нечетких входных данных, например адреса, места и точки интереса (POI). Этот код выполняет поиск ближайших бензозаправочных станций для заданных координат широты и долготы в пределах указанного адреса. Затем с помощью метода `geojson.getFeatures()` из ответа извлекается коллекция функций GeoJSON и добавляется к источнику данных, после чего данные автоматически отображаются на карте с помощью слоя символов. Последняя часть сценария устанавливает представление с камер на карте, используя ограничивающий прямоугольник с результатами, с помощью свойства карты [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-).

    ```JavaScript
    var query =  'gasoline-station';
    var radius = 9000;
    var lat = 47.64452336193245;
    var lon = -122.13687658309935;

    searchURL.searchPOI(atlas.service.Aborter.timeout(10000), query, {
        limit: 10,
        lat: lat,
        lon: lon,
        radius: radius
    }).then((results) => {

        // Extract GeoJSON feature collection from the response and add it to the datasource
        var data = results.geojson.getFeatures();
        datasource.add(data);

        // set camera to bounds to show the results
        map.setCamera({
            bounds: data.bbox,
            zoom: 10
        });
    });
    ```

3. Сохраните файл **MapSearch.html** и обновите страницу в браузере. Вы увидите карту, расположенную в Сиэтле, с круглыми-синими пометками, для обозначения местоположения бензозаправочных станций в области.

   ![Просмотр карты с результатами поиска](./media/tutorial-search-location/pins-map.png)

4. Вы можете увидеть необработанные данные, которые отображаются на карте в преобразованном для просмотра виде, введя следующий HTTP-запрос в вашем браузере. Замените \<Your Azure Maps Key\> своим первичным ключом.

   ```http
   https://atlas.microsoft.com/search/poi/json?api-version=1.0&query=gasoline%20station&subscription-key=<subscription-key>&lat=47.6292&lon=-122.2337&radius=100000
   ```

На этом этапе страница MapSearch может отображать расположения точек интереса, возвращаемых из запроса нечеткого поиска. Давайте добавим некоторые интерактивные возможности и дополнительные сведения о расположениях.

## <a name="add-interactive-data"></a>Добавление интерактивных данных

Карта, которую мы создали, учитывает только данные широты и долготы в результатах поиска. Однако необработанный код JSON, возвращаемый службой "Поиск" для Maps, содержит дополнительные сведения о каждой бензозаправочной станции. Включая имя и почтовый адрес. Эти данные можно включить в карту с помощью интерактивных всплывающих окон.

1. Добавьте приведенные ниже строки кода в обработчик событий карты `ready` после кода для запроса службы поиска нечетких соответствий. Этот код создаст экземпляр всплывающего окна и добавит событие наведения указателя мыши на слой символов.

    ```JavaScript
   //Create a popup but leave it closed so we can update it and display it later.
    popup = new atlas.Popup();

    //Add a mouse over event to the result layer and display a popup when this event fires.
    map.events.add('mouseover', resultLayer, showPopup);
    ```

    `*atlas.Popup` API предоставляет информационное окно, которое можно привязать к нужной позиции на карте. 

2. В функции `GetMap` добавьте следующий код для отображения во всплывающем окне событий наведения указателя мыши поверх сведений о результатах.

    ```JavaScript
    function showPopup(e) {
        //Get the properties and coordinates of the first shape that the event occurred on.

        var p = e.shapes[0].getProperties();
        var position = e.shapes[0].getCoordinates();

        //Create HTML from properties of the selected result.
        var html = `
          <div style="padding:5px">
            <div><b>${p.poi.name}</b></div>
            <div>${p.address.freeformAddress}</div>
            <div>${position[1]}, ${position[0]}</div>
          </div>`;

        //Update the content and position of the popup.
        popup.setPopupOptions({
            content: html,
            position: position
        });

        //Open the popup.
        popup.open(map);
    }
    ```

3. Сохраните файл и обновите страницу в браузере. Теперь на карте в браузере при наведении указателя мыши на любой показанный маркер поиска отображаются информационные всплывающие элементы.

    ![Azure Map Control и служба поиска](./media/tutorial-search-location/popup-map.png)

Чтобы просмотреть весь код для этого руководства, щелкните [здесь](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/search.html). Чтобы просмотреть рабочий пример, щелкните [здесь](https://azuremapscodesamples.azurewebsites.net/?sample=Search%20for%20points%20of%20interest).

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов, требующих очистки.

## <a name="next-steps"></a>Дальнейшие действия

В следующем руководстве показано, как отобразить маршрут между двумя расположениями.

> [!div class="nextstepaction"]
> [Прокладывание маршрута до точки интереса в службе "Карты Azure"](./tutorial-route-location.md)
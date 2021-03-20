---
title: Начало работы с элементом управления веб-карты Microsoft Azure Maps
description: Узнайте, как добавлять карты в веб-и мобильные приложения с помощью Map Control библиотеки JavaScript на стороне клиента в Azure Maps. См. раздел локализация карт.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/20/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 27d9d5d30dc4c96516d23acba6a11b411827071e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100378712"
---
# <a name="use-the-azure-maps-map-control"></a>Использование элемента управления картой в Azure Maps

Библиотека JavaScript на стороне клиента Map Control позволяет отображать карты и встроенные функции Azure Maps в веб-приложение или мобильном приложении.

В этой документации используется веб-пакет SDK Azure Maps, однако службы Azure Maps можно использовать с любым элементом управления картой. [Здесь](open-source-projects.md#third-part-map-control-plugins) приведены некоторые популярные элементы управления картой с открытым кодом, для которых команда Azure Maps создала подключаемый модуль.

## <a name="prerequisites"></a>Предварительные требования

Для использования Map Control на веб-странице необходимо выполнить одно из следующих условий.

* [Создайте учетную запись Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account) и [получите ключ первичной подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), также известный как первичный ключ или ключ подписки.

* Получите учетные данные Azure Active Directory (AAD) с [параметрами проверки подлинности](/javascript/api/azure-maps-control/atlas.authenticationoptions).

## <a name="create-a-new-map-in-a-web-page"></a>Создание карты на веб-странице

Карту можно внедрить на веб-страницу с помощью Map Control библиотеки JavaScript на стороне клиента.

1. Создайте HTML-файл.

2. Загрузите в веб-пакет SDK Azure Maps. Вы можете выбрать один из вариантов:

    * Используйте глобально размещенную версию CDN Azure Maps веб-пакета SDK, добавив ссылки на JavaScript и таблицу стилей в `<head>` ЭЛЕМЕНТЕ HTML-файла:

      ```html
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
      <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>
      ```

    * Загрузите исходный код веб-пакета SDK Azure Maps локально с помощью пакета [Azure-Maps-Control](https://www.npmjs.com/package/azure-maps-control) NPM и разместите его в своем приложении. Этот пакет также включает определения TypeScript.

      > **npm install azure-maps-control**

    Затем добавьте ссылки на таблицу стилей Azure Maps в `<head>` элемент файла:

    ```html
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
     ```

    > [!NOTE]
    > Определения typescript можно импортировать в приложение, добавив следующий код:
    >
    > ```javascript
    > import * as atlas from 'azure-maps-control';
    > ```

3. Чтобы отобразить карту так, чтобы она занимала основную область страницы, добавьте следующий элемент `<style>` к элементу `<head>`.

   ```HTML
    <style>
        html, body {
            margin: 0;
        }

        #myMap {
            height: 100vh;
            width: 100vw;
        }
    </style>
   ```

4. В основной области страницы добавьте элемент `<div>` и присвойте ему `id`**myMap**.

   ```HTML
    <body onload="InitMap()">
        <div id="myMap"></div>
    </body>
   ```

5. Теперь мы инициализируем элемент управления картой. Чтобы проверить подлинность элемента управления, необходимо либо владеть Azure Maps ключом подписки, либо использовать учетные данные Azure Active Directory (AAD) с [параметрами проверки подлинности](/javascript/api/azure-maps-control/atlas.authenticationoptions).

    Если вы используете ключ подписки для проверки подлинности, скопируйте и вставьте следующий элемент script в `<head>` элемент и ниже первого `<script>` элемента. Замените на `<Your Azure Maps Key>` ключ первичной подписки Azure Maps.

     ```HTML
    <script type="text/javascript">
        function InitMap()
        {
            var map = new atlas.Map('myMap', {
                center: [-122.33, 47.6],
                zoom: 12,
                language: 'en-US',
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            });
       }
    </script>
    ```

    Если для проверки подлинности используется Azure Active Directory (AAD), скопируйте и вставьте следующий элемент script в `<head>` элемент и ниже первого `<script>` элемента.

      ```HTML
    <script type="text/javascript">
        function InitMap()
        {
            var map = new atlas.Map('myMap', {
                center: [-122.33, 47.6],
                zoom: 12,
                language: 'en-US',
                authOptions: {
                    authType: 'aad',
                    clientId: '<Your AAD Client Id>',
                    aadAppId: '<Your AAD App Id>',
                    aadTenant: '<Your AAD Tenant Id>'
                }
            });
        }
    </script>
   ```

    Дополнительные сведения о проверке подлинности с помощью Azure Maps см. в статье [Проверка подлинности с помощью Azure Maps](azure-maps-authentication.md) документа. Кроме того, список примеров, демонстрирующих интеграцию Azure Active Directory (AAD) с Azure Maps, можно найти [здесь](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples).

    >[!TIP]
    >В этом примере мы передали `id` карту `<div>` . Другой способ сделать это — передать `HTMLElement` объект, передав в `document.getElementById('myMap')` качестве первого параметра.

6. Кроме того, может оказаться полезным добавить следующие `meta` элементы в `head` элемент страницы:

   ```HTML
    <!-- Ensures that IE and Edge uses the latest version and doesn't emulate an older version -->
    <meta http-equiv="x-ua-compatible" content="IE=Edge">

    <!-- Ensures the web page looks good on all screen sizes. -->
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
   ```

7. Если поместить все вместе, ваш HTML-файл должен выглядеть примерно так, как в следующей разметке:

   ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title></title>

        <meta charset="utf-8">

        <!-- Ensures that IE and Edge uses the latest version and doesn't emulate an older version -->
        <meta http-equiv="x-ua-compatible" content="IE=Edge">

        <!-- Ensures the web page looks good on all screen sizes. -->
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>


        <script type="text/javascript">
            //Create an instance of the map control and set some options.
            function InitMap()
            {
                var map = new atlas.Map('myMap', {
                    center: [-122.33, 47.6],
                    zoom: 12,
                    language: 'en-US',
                    authOptions: {
                        authType: 'subscriptionKey',
                        subscriptionKey: '<Your Azure Maps Key>'
                    }
                });
            }
        </script>

        <style>
            html, body {
                margin: 0;
            }

            #myMap {
                height: 100vh;
                width: 100vw;
            }
        </style>
    </head>
    <body onload="InitMap()">
        <div id="myMap"></div>
    </body>
    </html>
    ```

8. Откройте файл в веб-браузере и вы увидите готовую для просмотра карту. Оно должно выглядеть, как показано ниже:

   ![Изображение схемы, показывающее результат визуализации](./media/how-to-use-map-control/map-of-seattle.png)

## <a name="localizing-the-map"></a>Локализация карты

Azure Maps предоставляет два разных способа настройки языкового и регионального представления для отображаемой схемы. Первый вариант — Добавить эту информацию в глобальное `atlas` пространство имен, что приведет к тому, что все экземпляры элементов управления картой в приложении будут по умолчанию иметь эти параметры. В приведенном ниже примере кода устанавливается французский язык (fr-FR) и региональное представление со значением Auto:

```javascript
atlas.setLanguage('fr-FR');
atlas.setView('Auto');
```

Второй вариант заключается в передаче этой информации в параметры Map при загрузке карты следующим образом:

```javascript
map = new atlas.Map('myMap', {
    language: 'fr-FR',
    view: 'Auto',

    authOptions: {
        authType: 'aad',
        clientId: '<Your AAD Client Id>',
        aadAppId: '<Your AAD App Id>',
        aadTenant: '<Your AAD Tenant Id>'
    }
});
```

> [!NOTE]
> Можно загрузить несколько экземпляров карт на одной странице с разными языками и региональными параметрами. Кроме того, эти параметры можно обновить после загрузки сопоставлений с помощью `setStyle` функции Map.

Ниже приведен пример Azure Maps с языком, для которого задано значение "fr-FR", а для регионального представления задано значение "Авто".

![Изображение схемы, показывающее метки на французском языке](./media/how-to-use-map-control/websdk-localization.png)

Полный список поддерживаемых языков и региональных представлений приведен [здесь](supported-languages.md).

## <a name="azure-government-cloud-support"></a>Облачная поддержка Azure для государственных организаций

Azure Maps веб-пакет SDK поддерживает облако Azure для государственных организаций. Все URL-адреса JavaScript и CSS, используемые для доступа к Azure Maps Web SDK, остаются неизменными. Для подключения к облачной версии Azure Maps платформы Azure для государственных организаций необходимо выполнить следующие задачи.

При использовании интерактивного элемента управления картой добавьте следующую строку кода перед созданием экземпляра `Map` класса.

```javascript
atlas.setDomain('atlas.azure.us');
```

При проверке подлинности карт и служб обязательно используйте сведения о проверке подлинности Azure Maps на облачной платформе Azure для государственных организаций.

При использовании модуля служб необходимо задать домен для служб при создании экземпляра конечной точки URL-адреса API. Например, следующий код создает экземпляр `SearchURL` класса и указывает домен в облаке Azure для государственных организаций.

```javascript
var searchURL = new atlas.service.SearchURL(pipeline, 'atlas.azure.us');
```

При непосредственном доступе к службам Azure Maps RESTFUL измените домен URL-адреса на `atlas.azure.us` . Например, при использовании службы API поиска измените домен URL-адреса с `https://atlas.microsoft.com/search/` на `https://atlas.azure.us/search/` .

## <a name="javascript-frameworks"></a>Платформы JavaScript

Для разработки с использованием платформы JavaScript может оказаться полезным один из следующих проектов с открытым исходным кодом:

- [ng-azure-maps](https://github.com/arnaudleclerc/ng-azure-maps) — это программа-оболочка Angular 10 для Azure Maps.
- [AzureMapsControl.Components](https://github.com/arnaudleclerc/AzureMapsControl.Components) — компонент Azure Maps Blazor.
- [Компонент React Azure Maps](https://github.com/WiredSolutions/react-azure-maps) — программа-оболочка React для управления Azure Maps.
- [Vue Azure Maps](https://github.com/rickyruiz/vue-azure-maps) — компонент Azure Maps для приложения Vue.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как создавать и взаимодействовать с картой:

> [!div class="nextstepaction"]
> [Создание карты](map-create.md)

Узнайте, как изменить стиль карты:

> [!div class="nextstepaction"]
> [Выбор стиля карты](choose-map-style.md)

Чтобы добавить дополнительные данные на карту, выполните следующие действия.

> [!div class="nextstepaction"]
> [Создание карты](map-create.md)

> [!div class="nextstepaction"]
> [Примеры кода](/samples/browse/?products=azure-maps)

Список примеров, демонстрирующих интеграцию Azure Active Directory (AAD) с Azure Maps, см. в следующих статьях:

> [!div class="nextstepaction"]
> [Примеры проверки подлинности Azure AD](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples)
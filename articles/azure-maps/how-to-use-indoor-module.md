---
title: Использование модуля Azure Maps Maps с Microsoft Creator Services (Предварительная версия)
description: Узнайте, как с помощью модуля схем помещений Microsoft Azure Maps выполнять отрисовку карт путем внедрения библиотек JavaScript из этого модуля.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/20/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: e527cf5fa6a7caaeaf56ea19d684dd0830d5ca8a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708685"
---
# <a name="use-the-azure-maps-indoor-maps-module"></a>Использование модуля схем помещений Azure Maps

> [!IMPORTANT]
> Службы Создателя Azure Maps в настоящее время предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Веб-пакет SDK Azure Maps включает модуль *схем помещений*. Модуль  *Azure Mapsной* части позволяет визуализировать карты, созданные в службах Azure Maps Creator (Предварительная версия) 

## <a name="prerequisites"></a>Предварительные требования

1. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Создание ресурса Создателя (предварительная версия)](how-to-manage-creator.md)
3. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки.
4. Получите `tilesetId` и `statesetId`, выполнив инструкции из [руководства по созданию схем помещений](tutorial-creator-indoor-maps.md).
 Эти идентификаторы вам потребуются для отрисовки схем помещений в модуле схем помещений Azure Maps.

## <a name="embed-the-indoor-maps-module"></a>Внедрение модуля схем помещений

Вы можете установить и внедрить модуль *схем помещений Azure Maps* одним из двух способов.

Чтобы использовать для модуля *схем помещений Azure Maps* глобально размещенную версию сети доставки содержимого Azure, добавьте ссылки на следующие ресурсы JavaScript и таблиц стилей в элементе `<head>` HTML-файла:

```html
<link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.css" type="text/css"/>
<script src="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.js"></script>
```

 Или же скачайте модуль *схем помещений Azure Maps*. Модуль *схем помещений Azure Maps* содержит клиентскую библиотеку для доступа к службам Azure Maps. Выполните следующие действия, чтобы установить и загрузить модуль *схем помещений* в веб-приложение:  
  
  1. Установите [пакет Azure-Maps-помещений](https://www.npmjs.com/package/azure-maps-indoor).
  
      ```powershell
      >npm install azure-maps-indoor
      ```

  2. Укажите ссылки на ресурсы JavaScript и таблиц стилей для модуля *схем помещений Azure Maps* в элементе `<head>` HTML-файла:

      ```html
      <link rel="stylesheet" href="node_modules/azure-maps-drawing-tools/dist/atlas-indoor.min.css" type="text/css" />
      <script src="node_modules/azure-maps-drawing-tools/dist/atlas-indoor.min.js"></script>
      ```

## <a name="instantiate-the-map-object"></a>Создайте экземпляр объекта Map.

Для начала создайте объект *Map*. Объект *Map* мы применим на следующем шаге, чтобы создать экземпляр объекта *диспетчера схем помещений*.  В приведенном ниже коде показано, как создать экземпляр *объекта Map*.

```javascript
const subscriptionKey = "<Your Azure Maps Primary Subscription Key>";

const map = new atlas.Map("map-id", {
  //use your facility's location
  center: [-122.13203, 47.63645],
  //or, you can use bounds: [# west, # south, # east, # north] and replace # with your map's bounds
  style: "blank",
  view: 'Auto',
  authOptions: {
      authType: 'subscriptionKey',
      subscriptionKey: subscriptionKey
  },
  zoom: 19,
});
```

## <a name="instantiate-the-indoor-manager"></a>Создание экземпляра диспетчера схем помещений

Чтобы загрузить фрагменты карты помещений и стили карты для этих фрагментов, вам потребуется экземпляр *диспетчера схем помещений*. Создайте экземпляр *диспетчера схем помещений*, предоставив ему *объект Map* и соответствующий `tilesetId`. Если вам нужна поддержка [динамических стилей карты](indoor-map-dynamic-styling.md), передайте также `statesetId`. В имени переменной `statesetId` учитывается регистр. Вы получите такой код JavaScript, как показано ниже.

```javascript
const tilesetId = "<tilesetId>";
const statesetId = "<statesetId>";

const indoorManager = new atlas.indoor.IndoorManager(map, {
    tilesetId: tilesetId,
    statesetId: statesetId // Optional
});
```

Чтобы разрешить получение предоставляемых данных о состоянии, передайте `statesetId` и вызовите `indoorManager.setDynamicStyling(true)`. Опрос для получения данных о состоянии позволяет динамически обновлять состояние динамических свойств (*состояний*). Например, для компонента комнаты может существовать динамическое свойство (*состояние*) с именем `occupancy`. Приложение может выполнять опрос для получения сведений об изменениях *состояния*, чтобы отражать изменения на визуальной карте. Следующий пример кода демонстрирует поддержку опроса состояний:

```javascript
const tilesetId = "<tilesetId>";
const statesetId = "<statesetId>";

const indoorManager = new atlas.indoor.IndoorManager(map, {
    tilesetId: tilesetId,
    statesetId: statesetId // Optional
});

if (statesetId.length > 0) {
    indoorManager.setDynamicStyling(true);
}
```

## <a name="indoor-level-picker-control"></a>Элемент управления Indoor Level Picker (Выбор этажа помещения)

 Элемент управления *Indoor Level Picker* (Выбор этажа помещения) позволяет переключать уровни на отображаемой карте. Для необязательного элемента управления *Indoor Level Picker* (Выбор этажа помещения) можно создать экземпляр через *диспетчер схем помещений*. В следующем примере кода инициализируется средство выбора этажа:

```javascript
const levelControl = new atlas.control.LevelControl({ position: "top-right" });
indoorManager.setOptions({ levelControl });
```

## <a name="indoor-events"></a>События схемы помещений

 Модуль *схем помещений Azure Maps* поддерживает события *объекта Map*. Прослушиватели событий *объекта Map* вызываются при изменении этажа или строения. Если вы хотите выполнять определенный код при изменении этажа и (или) строения, поместите этот код в прослушиватель событий. Приведенный ниже пример кода демонстрирует, как добавить прослушиватели событий в *объект Map*.

```javascript
map.events.add("levelchanged", indoorManager, (eventData) => {

  //code that you want to run after a level has been changed
  console.log("The level has changed: ", eventData);
});

map.events.add("facilitychanged", indoorManager, (eventData) => {

  //code that you want to run after a facility has been changed
  console.log("The facility has changed: ", eventData);
});
```

Переменная `eventData` содержит сведения об этаже или строении, которые вызвали событие `levelchanged` или `facilitychanged`, соответственно. При изменении этажа объект `eventData` будет содержать `facilityId`, новое значение `levelNumber` и другие метаданные. При изменении строения объект `eventData` будет содержать новые значения `facilityId` и `levelNumber`, а также другие метаданные.

## <a name="example-use-the-indoor-maps-module"></a>Пример Использование модуля схем помещений

В этом примере показано, как использовать в приложении модуль *схем помещений Azure Maps*. Несмотря на ограниченные возможности этого примера, он охватывает все основные сведения, позволяющие начать работу с модулем *схем помещений Azure Maps*. После этих этапов приводится полный готовый код HTML.

1. С помощью [параметра](#embed-the-indoor-maps-module) сети доставки содержимого Azure установите модуль *схем помещений Azure Maps*.

2. Создание HTML-файла

3. В заголовке HTML-файла укажите ссылки на ресурсы JavaScript и таблиц стилей для модуля *схем помещений Azure Maps*.

4. Инициализируйте *объект Map*. *Объект Map* поддерживает следующие параметры:
    - `Subscription key` содержит первичный ключ подписки на Azure Maps.
    - `center` определяет широту и долготу, соответствующие центру схемы помещения. Задайте значение `center`, если не будет указано значение `bounds`. Поддерживается следующий формат `center`: [-122.13315, 47.63637].
    - `bounds` определяет наименьший возможный прямоугольник, который охватывает все данные элементов карты. Задайте значение `bounds`, если не будет указано значение `center`. Чтобы получить сведения о границах карты, вызовите API [списка фрагментов карты](/rest/api/maps/tileset/listpreview). API списка фрагментов карты возвращает `bbox`, значение которого можно анализировать и присваивать `bounds`. Формат должен выглядеть следующим образом `bounds` : [# Западная, # Южный, # Восток, # Север].
    - `style` позволяет установить цвет фона. Чтобы отобразить белый фон, определите для `style` значение "blank".
    - `zoom` позволяет указать минимальный и максимальный уровни масштабирования для схемы.

5. Теперь создайте модуль *диспетчера схем помещений*. Назначьте для *схемы помещения Azure Maps* значение `tilesetId`. При желании добавьте `statesetId`.

6. Создайте экземпляр элемента управления *Indoor Level Picker* (Выбор этажа помещения).

7. Добавьте прослушиватели событий *объекта Map*.  

Теперь файл должен содержать примерно такой HTML-код:

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8" />
      <meta name="viewport" content="width=device-width, user-scalable=no" />
      <title>Indoor Maps App</title>
      
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.css" type="text/css"/>

      <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>
      <script src="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.js"></script>
        
      <style>
        html,
        body {
          width: 100%;
          height: 100%;
          padding: 0;
          margin: 0;
        }

        #map-id {
          width: 100%;
          height: 100%;
        }
      </style>
    </head>

    <body>
      <div id="map-id"></div>
      <script>
        const subscriptionKey = "<Your Azure Maps Primary Subscription Key>";
        const tilesetId = "<your tilesetId>";
        const statesetId = "<your statesetId>";

        const map = new atlas.Map("map-id", {
          //use your facility's location
          center: [-122.13315, 47.63637],
          //or, you can use bounds: [# west, # south, # east, # north] and replace # with your Map bounds
          style: "blank",
          view: 'Auto',
          authOptions: { 
              authType: 'subscriptionKey',
              subscriptionKey: subscriptionKey
          },
          zoom: 19,
        });

        const levelControl = new atlas.control.LevelControl({
          position: "top-right",
        });

        const indoorManager = new atlas.indoor.IndoorManager(map, {
          levelControl: levelControl, //level picker
          tilesetId: tilesetId,
          statesetId: statesetId // Optional
        });

        if (statesetId.length > 0) {
          indoorManager.setDynamicStyling(true);
        }

        map.events.add("levelchanged", indoorManager, (eventData) => {
          //put code that runs after a level has been changed
          console.log("The level has changed:", eventData);
        });

        map.events.add("facilitychanged", indoorManager, (eventData) => {
          //put code that runs after a facility has been changed
          console.log("The facility has changed:", eventData);
        });
      </script>
    </body>
  </html>
  ```

Чтобы просмотреть созданную схему, загрузите ее в веб-браузер. Она должна выглядеть так, как на изображении ниже. Если щелкнуть компонент лестницы, справа в верхнем углу появится *средство выбора этажа*.

  ![Изображение схемы помещения](media/how-to-use-indoor-module/indoor-map-graphic.png)

[Ознакомьтесь с демонстрацией в реальном времени](https://azuremapscodesamples.azurewebsites.net/?sample=Creator%20indoor%20maps)

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с интерфейсами API, которые имеют отношение к модулю *схем помещений Azure Maps*:

> [!div class="nextstepaction"]
> [Требования к пакету рисунков](drawing-requirements.md)

>[!div class="nextstepaction"]
> [Создатель (Предварительная версия) для карт для помещений](creator-indoor-maps.md)

Дополнительные сведения о добавлении данных к схеме:

> [!div class="nextstepaction"]
> [Динамические стили для схем помещений](indoor-map-dynamic-styling.md)

> [!div class="nextstepaction"]
> [Примеры кода](/samples/browse/?products=azure-maps)
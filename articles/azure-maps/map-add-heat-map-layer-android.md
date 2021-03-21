---
title: Добавление слоя тепловой карты в карты Android | Карты Microsoft Azure
description: Узнайте, как создать тепловую карту. См. статью использование пакета SDK Azure Мапсандроид для добавления слоя тепловой карт в карту. Узнайте, как настроить уровни тепловой карт.
author: rbrundritt
ms.author: richbrun
ms.date: 02/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: fce2c2d007f92c43e763826f9345f773324e885e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102100191"
---
# <a name="add-a-heat-map-layer-android-sdk"></a>Добавление слоя тепловой карт (пакет SDK для Android)

Тепловая карта, которую также называют картой точек плотности, представляет собой тип визуализации данных. Они используются для представления плотности данных с использованием диапазона цветов и отображения «горячих точек» данных на карте. Тепловые карты — это отличный способ отображения наборов данных с большим количеством точек. 

Отрисовка десятков тысяч точек в виде символов может охватывать большую часть области карт. В этом случае, скорее всего, происходит перекрытие нескольких символов. Затрудняете понимание данных. Однако визуализация этого же набора данных в качестве тепловой карт позволяет легко увидеть плотность и относительную плотность каждой точки данных.

Карты рисков можно использовать в различных сценариях, в том числе:

- **Данные температуры**: выводит приблизительные показатели температуры между двумя точками данных.
- **Данные для датчиков шума**. показывает не только интенсивность шума, в которой находится датчик, но также позволяет получить представление о расположении на расстоянии. Уровень шума на одном сайте может быть невысоким. Если область действия шума из нескольких датчиков перекрывается, возможно, эта перекрывающаяся область может иметь более высокие уровни шума. Таким образом, область перекрытия будет видна на тепловой карте.
- **Трассировка GPS**: включает скорость как карту взвешенной высоты, где интенсивность каждой точки данных зависит от скорости. Например, эта функция позволяет узнать, где был ускорен процесс.

> [!TIP]
> По умолчанию уровни тепловой карт отображают координаты всех геометрических объектов в источнике данных. Чтобы ограничить слой таким образом, чтобы он отображал только возможности геометрических точек, установите `filter` параметр слоя в значение `eq(geometryType(), "Point")` . Если вы хотите включить компоненты MultiPoint, установите `filter` параметр слоя в значение `any(eq(geometryType(), "Point"), eq(geometryType(), "MultiPoint"))` .

</br>

> [!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Heat-Maps-and-Image-Overlays-in-Azure-Maps/player?format=ny]

## <a name="prerequisites"></a>Предварительные требования

Не забудьте выполнить действия, описанные в разделе [Краткое руководство. Создание документа приложения Android](quick-android-map.md) . Блоки кода в этой статье можно вставить в `onReady` обработчик событий Maps.

## <a name="add-a-heat-map-layer"></a>Добавление слоя тепловой карты

Для отображения источника данных точек в качестве тепловой карты передайте источник данных в экземпляр `HeatMapLayer` класса и добавьте его на карту.

В следующем примере кода загружается веб-канал геоземлетрясения с прошедшей стороной за прошлую неделю и готовится к просмотру в виде тепловой картой. Каждая точка данных отображается с радиусом 10 пикселей на всех уровнях масштабирования. Чтобы обеспечить лучшую работу пользователей, тепловая схема находится ниже слоя меток, поэтому метки остаются видимыми. Данные в этом примере находятся в [программе USGS землетрясения опасные программы](https://earthquake.usgs.gov/). Этот пример загружает данные геоjson из Интернета с помощью блока кода служебной программы импорта данных, приведенного в документе [Создание источника данных](create-data-source-android-sdk.md) .

::: zone pivot="programming-language-java-android"

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Create a heat map layer.
HeatMapLayer layer = new HeatMapLayer(source,
  heatmapRadius(10f),
  heatmapOpacity(0.8f)
);

//Add the layer to the map, below the labels.
map.layers.add(layer, "labels");

//Import the geojson data and add it to the data source.
Utils.importData("https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson",
    this,
    (String result) -> {
        //Parse the data as a GeoJSON Feature Collection.
        FeatureCollection fc = FeatureCollection.fromJson(result);

        //Add the feature collection to the data source.
        source.add(fc);

        //Optionally, update the maps camera to focus in on the data.

        //Calculate the bounding box of all the data in the Feature Collection.
        BoundingBox bbox = MapMath.fromData(fc);

        //Update the maps camera so it is focused on the data.
        map.setCamera(
            bounds(bbox),
            padding(20));
    });
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create a data source and add it to the map.
val source = DataSource()
map.sources.add(source)

//Create a heat map layer.
val layer = HeatMapLayer(
    source,
    heatmapRadius(10f),
    heatmapOpacity(0.8f)
)

//Add the layer to the map, below the labels.
map.layers.add(layer, "labels")

//Import the geojson data and add it to the data source.
Utils.importData("https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson",
    this
) { result: String? ->
    //Parse the data as a GeoJSON Feature Collection.
    val fc = FeatureCollection.fromJson(result!!)

    //Add the feature collection to the data source.
    source.add(fc)

    //Optionally, update the maps camera to focus in on the data.
    //Calculate the bounding box of all the data in the Feature Collection.
    val bbox = MapMath.fromData(fc)

    //Update the maps camera so it is focused on the data.
    map.setCamera(
        bounds(bbox),
        padding(20)
    )
}
```

::: zone-end

На следующем снимке экрана показана схема загрузки тепловой карт с помощью приведенного выше кода.

![Схема с уровнем тепловой карт недавних землетрясения](media/map-add-heat-map-layer-android/android-heat-map-layer.png)

## <a name="customize-the-heat-map-layer"></a>Настройка слоя тепловой карт

В предыдущем примере настройка тепловой карты заключалась в установке параметров радиуса и прозрачности. Слой тепловой карт предоставляет несколько вариантов настройки, включая следующие:

- `heatmapRadius`: Определяет пиксельный радиус для отрисовки каждой точки данных. Вы можете задать для радиуса фиксированное число или как выражение. С помощью выражения можно масштабировать радиус в зависимости от уровня масштаба и представить единообразную пространственное область на карте (например, радиус 5 км).
- `heatmapColor`: Указывает, как раскрасить тепловую карту. Цветовой градиент — это общая функция карт рисков. Результат можно добиться с помощью `interpolate` выражения. Можно также использовать `step` выражение для раскрашивания тепловой карт, разбивая его на диапазоны, похожие на карту в виде контура или лепестковой диаграммы. Эти цветовые палитры определяют цвета в диапазоне от минимальной до максимальной плотности.

  Значения цвета для тепловых карт задаются в виде выражения для `heatmapDensity` значения. Цвет области, в которой отсутствуют данные в индексе 0 выражения интерполяции, или цвет по умолчанию для "пошагового" выражения. Это значение можно использовать для определения цвета фона. Часто это значение равно прозрачному или полупрозрачному черному. 

  Ниже приведены примеры выражений цвета.

  | Выражение цвета интерполяции | Выражение пошагового цвета |
  |--------------------------------|--------------------------|
  | интерполяции<br/>&nbsp;&nbsp;&nbsp;&nbsp;Линейная (), <br/>&nbsp;&nbsp;&nbsp;&nbsp;Хеатмапденсити (),<br/>&nbsp;&nbsp;&nbsp;&nbsp;прерывать (0, Color (цвет. прозрачный)),<br/>&nbsp;&nbsp;&nbsp;&nbsp;прерывать (0,01, Color (Color. пурпурный)),<br/>&nbsp;&nbsp;&nbsp;&nbsp;останавливаться (0,5, Color (Парсеколор ("#fb00fb"))),<br/>&nbsp;&nbsp;&nbsp;&nbsp;останавливаться (1, цвет (Парсеколор ("#00c3ff")))<br/>)` | первом<br/>&nbsp;&nbsp;&nbsp;&nbsp;Хеатмапденсити (),<br/>&nbsp;&nbsp;&nbsp;&nbsp;Color (цвет. прозрачный),<br/>&nbsp;&nbsp;&nbsp;&nbsp;останавливает (0,01, Color (Парсеколор ("#000080"))),<br/>&nbsp;&nbsp;&nbsp;&nbsp;прерывать (0,25, Color (Парсеколор ("#000080"))),<br/>&nbsp;&nbsp;&nbsp;&nbsp;прерывать (0,5, Color (цвет. GREEN)),<br/>&nbsp;&nbsp;&nbsp;&nbsp;прерывать (0,5, Color (цвет. YELLOW)),<br/>&nbsp;&nbsp;&nbsp;&nbsp;прерывать (1, цвет (цвет. красный))<br/>) |

- `heatmapOpacity`: Определяет степень непрозрачности или прозрачности слоя тепловой карт.
- `heatmapIntensity`: Применяет множитель к весу каждой точки данных, чтобы увеличить общую интенсивность тепловой карты. Это приводит к разнице в весе точек данных, что упрощает визуализацию.
- `heatmapWeight`: По умолчанию все точки данных имеют вес 1 и имеют одинаковое взвешенное значение. Параметр Weight выступает в качестве множителя, и его можно задать в виде числа или выражения. Если в качестве веса задано число, то это эквивалентное размещение каждой точки данных на карте дважды. Например, если весовой коэффициент равен `2` , то плотность удваивается. Фиксированное число в качестве значения параметра weight изменяет тепловую карту точно так же, как изменение параметра intensity.

  Однако при использовании выражения весовые коэффициенты каждой точки данных могут основываться на свойствах каждой точки данных. Например, предположим, что каждая точка данных представляет землетрясения. Значение величины было важной метрикой для каждой точки данных землетрясения. Землетрясения выполняется все время, но большинство из них имеет низкую величину и не Заметится. Значение величины в выражении используется для назначения веса каждой точке данных. Используя значение величины для назначения веса, вы получаете лучшее представление о значимости землетрясения в тепловой карте.
- `minZoom` и `maxZoom` : диапазон уровня масштаба, в котором должен отображаться слой.
- `filter`: Выражение фильтра, используемое для ограничения извлекаемого из источника и отрисовки в слое.
- `sourceLayer`: Если источник данных, подключенный к слою, является источником векторной плитки, необходимо указать слой источника в векторных мозаиках.
- `visible`: Скрывает или отображает слой.

Ниже приведен пример тепловой карте, в которой для создания плавного градиента цвета используется выражение интерполяции. `mag`Свойство, определенное в данных, используется с экспоненциальной интерполяцией для установки веса или релевантности каждой точки данных.

::: zone pivot="programming-language-java-android"

```java
HeatMapLayer layer = new HeatMapLayer(source,
    heatmapRadius(10f),

    //A linear interpolation is used to create a smooth color gradient based on the heat map density.
    heatmapColor(
        interpolate(
            linear(),
            heatmapDensity(),
            stop(0, color(Color.TRANSPARENT)),
            stop(0.01, color(Color.BLACK)),
            stop(0.25, color(Color.MAGENTA)),
            stop(0.5, color(Color.RED)),
            stop(0.75, color(Color.YELLOW)),
            stop(1, color(Color.WHITE))
        )
    ),

    //Using an exponential interpolation since earthquake magnitudes are on an exponential scale.
    heatmapWeight(
       interpolate(
            exponential(2),
            get("mag"),
            stop(0,0),

            //Any earthquake above a magnitude of 6 will have a weight of 1
            stop(6, 1)
       )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = HeatMapLayer(source,
    heatmapRadius(10f),

    //A linear interpolation is used to create a smooth color gradient based on the heat map density.
    heatmapColor(
        interpolate(
            linear(),
            heatmapDensity(),
            stop(0, color(Color.TRANSPARENT)),
            stop(0.01, color(Color.BLACK)),
            stop(0.25, color(Color.MAGENTA)),
            stop(0.5, color(Color.RED)),
            stop(0.75, color(Color.YELLOW)),
            stop(1, color(Color.WHITE))
        )
    ),

    //Using an exponential interpolation since earthquake magnitudes are on an exponential scale.
    heatmapWeight(
       interpolate(
            exponential(2),
            get("mag"),
            stop(0,0),

            //Any earthquake above a magnitude of 6 will have a weight of 1
            stop(6, 1)
       )
    )
)
```

::: zone-end

На следующем снимке экрана показан пользовательский слой тепловой карт, использующий те же данные, что и в предыдущем примере тепловой карт.

![Схема с пользовательским уровнем тепловой карт недавних землетрясения](media/map-add-heat-map-layer-android/android-custom-heat-map-layer.png)

## <a name="consistent-zoomable-heat-map"></a>Последовательная тепловая схема масштабируемый

По умолчанию радиусы точек данных, отображаемых на уровне тепловой карт, имеют фиксированный пиксельный радиус для всех уровней масштабирования. По мере масштабирования карт данные объединяются, и слой тепловой карт выглядит иначе. В следующем видео показано поведение по умолчанию для тепловой карты, где он поддерживает пиксельный радиус при изменении масштаба карты.

![Анимация, показывающая изменение размера карт с помощью слоя тепловой карт с последовательным размером в пикселях](media/map-add-heat-map-layer-android/android-heat-map-layer-zoom.gif)

Используйте `zoom` выражение для масштабирования радиуса каждого уровня масштаба, чтобы каждая точка данных охватывает одну и ту же физическую область на карте. Это выражение делает внешний вид слоя тепловой карт более статическим и последовательным. Каждый уровень масштаба на карте имеет вдвое больше пикселей по вертикали и горизонтали, чем предыдущий уровень масштаба.

При масштабировании радиуса, чтобы он вдвое соответствовал каждому уровню масштабирования, создает тепловую карту, которая выглядит единообразно на всех уровнях масштабирования. Чтобы применить это масштабирование, используйте `zoom` с базовым `exponential interpolation` выражением 2 с набором радиуса пикселей для минимального масштаба и масштабируемым радиусом для максимального уровня масштаба, вычисленным как `2 * Math.pow(2, minZoom - maxZoom)` показано в следующем примере. Масштабировать карту, чтобы увидеть, как тепловая схема масштабируется с учетом масштаба.

::: zone pivot="programming-language-java-android"

```java
HeatMapLayer layer = new HeatMapLayer(source,
  heatmapRadius(
    interpolate(
      exponential(2),
      zoom(),

      //For zoom level 1 set the radius to 2 pixels.
      stop(1, 2f),

      //Between zoom level 1 and 19, exponentially scale the radius from 2 pixels to 2 * (maxZoom - minZoom)^2 pixels.
      stop(19, Math.pow(2, 19 - 1) * 2f)
    )
  ),
  heatmapOpacity(0.75f)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = HeatMapLayer(source,
  heatmapRadius(
    interpolate(
      exponential(2),
      zoom(),

      //For zoom level 1 set the radius to 2 pixels.
      stop(1, 2f),

      //Between zoom level 1 and 19, exponentially scale the radius from 2 pixels to 2 * (maxZoom - minZoom)^2 pixels.
      stop(19, Math.pow(2.0, 19 - 1.0) * 2f)
    )
  ),
  heatmapOpacity(0.75f)
)
```

::: zone-end

В следующем видео показана карта, на которой выполняется приведенный выше код, который масштабирует радиус при масштабировании карты, чтобы создать согласованную визуализацию тепловой карты на уровне масштаба.

![Анимация, показывающая изменение размера карт с помощью слоя тепловой карт с одинаковым геопространственном размером](media/map-add-heat-map-layer-android/android-consistent-zoomable-heat-map-layer.gif)

## <a name="next-steps"></a>Следующие шаги

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Создание источника данных](create-data-source-android-sdk.md)

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](data-driven-style-expressions-android-sdk.md)

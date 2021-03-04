---
title: Добавление элементов управления в карту Android | Карты Microsoft Azure
description: Добавление элемента управления "Масштаб", элемента управления "высота", элемента управления "поворот" и выбора стиля на карту в Microsoft Azure Maps пакет SDK для Androids.
author: rbrundritt
ms.author: richbrun
ms.date: 02/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: 90d037fc02bdc1c4d6fe682386790561c890c1e6
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102100225"
---
# <a name="add-controls-to-a-map-android-sdk"></a>Добавление элементов управления на карту (пакет SDK для Android)

В этой статье показано, как добавить элементы управления ИП на карту.

## <a name="add-zoom-control"></a>Добавление элемента управления масштабом

Элемент управления Zoom добавляет кнопки для масштабирования и отображения карт. В следующем примере кода создается экземпляр `ZoomControl` класса и добавляется в карту.

::: zone pivot="programming-language-java-android"

```java
//Construct a zoom control and add it to the map.
map.controls.add(new ZoomControl());
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Construct a zoom control and add it to the map.
map.controls.add(ZoomControl())
```

::: zone-end

Ниже приведен снимок экрана с элементом управления масштабом, загруженным на карте.

![Элемент управления Zoom добавлен в карту](media/map-add-controls-android/android-zoom-control.jpg)

## <a name="add-pitch-control"></a>Добавление элемента управления наклоном

Элемент управления "тон" добавляет кнопки для наклона высоты для отображения относительно горизонта. В следующем примере кода создается экземпляр `PitchControl` класса и добавляется в карту.

::: zone pivot="programming-language-java-android"

```java
//Construct a pitch control and add it to the map.
map.controls.add(new PitchControl());
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Construct a pitch control and add it to the map.
map.controls.add(PitchControl())
```

::: zone-end

На снимке экрана ниже показан элемент управления "тон", загруженный на карте.

![Элемент управления "шаг" добавлен в карту](media/map-add-controls-android/android-pitch-control.jpg)

## <a name="add-compass-control"></a>Добавление элемента управления расстоянием

Элемент управления компаса добавляет кнопку для поворота схемы. В следующем примере кода создается экземпляр `CompassControl` класса и добавляется в карту.

::: zone pivot="programming-language-java-android"

```java
//Construct a compass control and add it to the map.
map.controls.add(new CompassControl());
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Construct a compass control and add it to the map.
map.controls.add(CompassControl())
```

::: zone-end

На следующем снимке экрана показан элемент управления компаса, загруженный на карте.

![Элемент управления компасом добавлен в карту](media/map-add-controls-android/android-compass-control.jpg)

## <a name="add-traffic-control"></a>Добавить управление трафиком

Элемент управления потоком добавляет кнопку для переключения видимости данных трафика на карте. В следующем примере кода создается экземпляр `TrafficControl` класса и добавляется в карту.

::: zone pivot="programming-language-java-android"

```java
//Construct a traffic control and add it to the map.
map.controls.add(new TrafficControl());
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Construct a traffic control and add it to the map.
map.controls.add(TrafficControl())
```

::: zone-end

На следующем снимке экрана показан элемент управления трафиком, загруженный на карте.

![Элемент управления трафиком, добавленный в карту](media/map-add-controls-android/android-traffic-control.jpg)

## <a name="a-map-with-all-controls"></a>Карта со всеми элементами управления

Несколько элементов управления могут быть помещены в массив и добавлены в карту сразу и расположены в одной области на карте, чтобы упростить разработку. Следующий пример добавляет стандартные элементы управления навигацией в карту с помощью этого подхода.

::: zone pivot="programming-language-java-android"

```java
map.controls.add(
    new Control[]{
        new ZoomControl(),
        new CompassControl(),
        new PitchControl(),
        new TrafficControl()
    }
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
map.controls.add(
    arrayOf<Control>(
        ZoomControl(),
        CompassControl(),
        PitchControl(),
        TrafficControl()
    )
)
```

::: zone-end

На следующем снимке экрана показаны все элементы управления, загруженные на карте. Обратите внимание, что порядок, в котором они добавляются к карте, — это порядок, в котором они будут отображаться.

![Все элементы управления, добавленные в карту](media/map-add-controls-android/android-all-controls.jpg)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer-android.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](android-map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](how-to-add-shapes-to-android-map.md)

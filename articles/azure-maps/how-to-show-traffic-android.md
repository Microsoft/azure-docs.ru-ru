---
title: Отображение данных трафика в картах Android | Карты Microsoft Azure
description: Из этой статьи вы узнаете, как отображать данные трафика на карте с помощью Microsoft Azure карт пакет SDK для Android.
author: rbrundritt
ms.author: richbrun
ms.date: 2/26/2021
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: 36b3666f12b48468467e76f4c281d58d8018478c
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102098542"
---
# <a name="show-traffic-data-on-the-map-android-sdk"></a>Отображение данных трафика на карте (пакет SDK для Android)

Данные потока и инциденты — это два типа данных трафика, которые могут отображаться на карте. В этом руководство показано, как отобразить оба типа данных трафика. Данные инцидентов состоят из данных на основе точек и строк для таких вещей, как конструкции, замыкания на пути и всякое. Данные о потоке показывают метрики потока трафика в дороге.

## <a name="prerequisites"></a>Предварительные требования

Не забудьте выполнить действия, описанные в разделе [Краткое руководство. Создание документа приложения Android](quick-android-map.md) . Блоки кода в этой статье можно вставить в `onReady` обработчик событий Maps.

## <a name="show-traffic-on-the-map"></a>Отображение данных дорожного движения на карте

В Azure Maps доступны два типа данных трафика:

- Данные по происшествиям — это данные в виде точек и линий, которые обозначают, помимо прочего, строительные работы, закрытие дорог и аварии.
- Данные по потоку — метрики потока уличного движения. Данные по потоку уличного движения часто используются для раскрашивания дорог. Цвет зависит от того, насколько скорость движения потока меньше максимальной разрешенной скорости, или от другой метрики. В параметре трафика на карте можно передать четыре значения `flow` .

    |Значение потока | Описание|
    | :-- | :-- |
    | Траффикфлов. NONE | Не отображает данные трафика на карте |
    | Траффикфлов. RELATIVE | Показывает данные трафика, относящиеся к скорости произвольного потока в дороге |
    | TrafficFlow.RELATIVE_DELAY | Отображает области, которые выполняются медленнее среднего ожидаемого значения задержки |
    | Траффикфлов. ABSOLUTE | Показывает абсолютную скорость всех транспортных средств в дороге |

Ниже приведен код для отображения данных дорожного движения на карте.

::: zone pivot="programming-language-java-android"

```java
//Show traffic on the map using the traffic options.
map.setTraffic(
    incidents(true),
    flow(TrafficFlow.RELATIVE)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
map.setTraffic(
    incidents(true),
    flow(TrafficFlow.RELATIVE)
)
```

::: zone-end

На следующем снимке экрана показан приведенный выше код, отрисовки сведения о трафике в режиме реального времени на карте.

![Схема отображения сведений о трафике в реальном времени](media/how-to-show-traffic-android/android-show-traffic.png)

## <a name="get-traffic-incident-details"></a>Получение сведений об инциденте трафика

Сведения об инциденте трафика доступны в свойствах функции, используемой для отображения инцидента на карте. Инциденты трафика добавляются к сопоставлению с помощью службы плиток "вектор инцидента трафика Azure Maps". Формат данных в этих векторных плитках, если они [описаны здесь](https://developer.tomtom.com/traffic-api/traffic-api-documentation-traffic-incidents/vector-incident-tiles). Следующий код добавляет событие щелчка на карту и извлекает выбранную функцию инцидента трафика и отображает всплывающее сообщение с некоторыми сведениями.

::: zone pivot="programming-language-java-android"

```java
//Show traffic information on the map.
map.setTraffic(
    incidents(true),
    flow(TrafficFlow.RELATIVE)
);

//Add a click event to the map.
map.events.add((OnFeatureClick) (features) -> {

    if (features != null && features.size() > 0) {
        Feature incident = features.get(0);

        //Ensure that the clicked feature is an traffic incident feature.
        if (incident.properties() != null && incident.hasProperty("incidentType")) {

            StringBuilder sb = new StringBuilder();
            String incidentType = incident.getStringProperty("incidentType");

            if (incidentType != null) {
                sb.append(incidentType);
            }

            if (sb.length() > 0) {
                sb.append("\n");
            }

            //If the road is closed, find out where it is closed from.
            if ("Road Closed".equals(incidentType)) {
                String from = incident.getStringProperty("from");

                if (from != null) {
                    sb.append(from);
                }
            } else {
                //Get the description of the traffic incident.
                String description = incident.getStringProperty("description");

                if (description != null) {
                    sb.append(description);
                }
            }

            String message = sb.toString();

            if (message.length() > 0) {
                Toast.makeText(this, message, Toast.LENGTH_LONG).show();
            }
        }
    }
});
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Show traffic information on the map.
map.setTraffic(
    incidents(true),
    flow(TrafficFlow.RELATIVE)
)

//Add a click event to the map.
map.events.add(OnFeatureClick { features: List<Feature>? ->
    if (features != null && features.size > 0) {
        val incident = features[0]

        //Ensure that the clicked feature is an traffic incident feature.
        if (incident.properties() != null && incident.hasProperty("incidentType")) {
            val sb = StringBuilder()
            val incidentType = incident.getStringProperty("incidentType")

            if (incidentType != null) {
                sb.append(incidentType)
            }

            if (sb.length > 0) {
                sb.append("\n")
            }

            //If the road is closed, find out where it is closed from.
            if ("Road Closed" == incidentType) {
                val from = incident.getStringProperty("from")
                if (from != null) {
                    sb.append(from)
                }
            } else { //Get the description of the traffic incident.
                val description = incident.getStringProperty("description")
                if (description != null) {
                    sb.append(description)
                }
            }

            val message = sb.toString()
            if (message.length > 0) {
                Toast.makeText(this, message, Toast.LENGTH_LONG).show()
            }
        }
    }
})
```

::: zone-end

На следующем снимке экрана показан приведенный выше код, отрисовки сведения о трафике в режиме реального времени на карте с всплывающим сообщением, отображающим сведения об инциденте.

![Отображение сведений о трафике в реальном времени с помощью всплывающего сообщения с информацией об инциденте](media/how-to-show-traffic-android/android-traffic-details.png)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как добавить дополнительные данные на карту, ознакомьтесь со следующими руководствами.

> [!div class="nextstepaction"]
> [Добавление слоя фрагментов](how-to-add-tile-layer-android-map.md)

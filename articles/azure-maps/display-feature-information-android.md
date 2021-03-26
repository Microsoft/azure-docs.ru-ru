---
title: Отображение сведений о функциях в картах Android | Карты Microsoft Azure
description: Сведения о том, как отображать информацию, когда пользователи взаимодействуют с функциями карт. Используйте пакет SDK для Android Azure Maps для вывода всплывающих сообщений и других типов сообщений.
author: rbrundritt
ms.author: richbrun
ms.date: 2/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: ea9ed2c74651a7719533340a24c4741d4e25b805
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105605728"
---
# <a name="display-feature-information"></a>Отображение сведений о компоненте

Пространственные данные часто представляются с помощью точек, линий и многоугольников. С этими данными часто связаны метаданные. Например, точка может представлять местоположение ресторана, а метаданные об этом ресторане могут быть именем, адресом и типом пищи, который он обслуживает. Эти метаданные можно добавить как свойства геоjson `Feature` . Следующий код создает простую функцию Point со `title` свойством, имеющим значение "Hello World!"

::: zone pivot="programming-language-java-android"

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Create a point feature.
Feature feature = Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64));

//Add a property to the feature.
feature.addStringProperty("title", "Hello World!");

//Create a point feature, pass in the metadata properties, and add it to the data source.
source.add(feature);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create a data source and add it to the map.
val source = DataSource()
map.sources.add(source)

//Create a point feature.
val feature = Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64))

//Add a property to the feature.
feature.addStringProperty("title", "Hello World!")

//Create a point feature, pass in the metadata properties, and add it to the data source.
source.add(feature)
```

::: zone-end

Способы создания и добавления данных на карту см. в документации по [созданию источника данных](create-data-source-android-sdk.md) .

Когда пользователь взаимодействует с компонентом на карте, события могут использоваться для реагирования на эти действия. Распространенным сценарием является отображение сообщения, сопоставленного со свойствами метаданных функции, с которой взаимодействует пользователь. `OnFeatureClick`Событие является основным событием, используемым для определения того, когда пользователь нажал на карту. Существует также `OnLongFeatureClick` событие. При добавлении `OnFeatureClick` события к сопоставлению его можно ограничить одним слоем, ПЕРЕДАВ идентификатор слоя для ограничения. Если идентификатор слоя не передается, то при нажатии любой функции на карте, независимо от того, в каком слое он находится, будет срабатывать это событие. Следующий код создает слой символов для отрисовки данных точек на карте, затем добавляет `OnFeatureClick` событие и ограничивает его на этот уровень символов.

::: zone pivot="programming-language-java-android"

```java
//Create a symbol and add it to the map.
SymbolLayer layer = new SymbolLayer(source);
map.layers.add(layer);

//Add a feature click event to the map.
map.events.add((OnFeatureClick) (features) -> {
    //Retrieve the title property of the feature as a string.
    String msg = features.get(0).getStringProperty("title");

    //Do something with the message.

    //Return a boolean indicating if event should be consumed or continue bubble up.
    return false;
}, layer.getId());    //Limit this event to the symbol layer.
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create a symbol and add it to the map.
val layer = SymbolLayer(source)
map.layers.add(layer)

//Add a feature click event to the map.
map.events.add(OnFeatureClick { features: List<Feature> ->
    //Retrieve the title property of the feature as a string.
    val msg = features[0].getStringProperty("title")

    //Do something with the message.

    //Return a boolean indicating if event should be consumed or continue bubble up.
    return false
}, layer.getId()) //Limit this event to the symbol layer.
```

::: zone-end

## <a name="display-a-toast-message"></a>Отображение всплывающего сообщения

Всплывающее сообщение — это один из самых простых способов отобразить сведения для пользователя и доступен во всех версиях Android. Он не поддерживает никаких типов вводимых пользователем данных и отображается только в течение короткого периода времени. Если вы хотите быстро предоставить пользователю сведения о том, что они применяют, то может быть хорошим вариантом. В следующем коде показано, как можно использовать всплывающее сообщение с `OnFeatureClick` событием.

::: zone pivot="programming-language-java-android"

```java
//Add a feature click event to the map.
map.events.add((OnFeatureClick) (features) -> {
    //Retrieve the title property of the feature as a string.
    String msg = features.get(0).getStringProperty("title");

    //Display a toast message with the title information.
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();

    //Return a boolean indicating if event should be consumed or continue bubble up.
    return false;
}, layer.getId());    //Limit this event to the symbol layer.
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Add a feature click event to the map.
map.events.add(OnFeatureClick { features: List<Feature> ->
    //Retrieve the title property of the feature as a string.
    val msg = features[0].getStringProperty("title")

    //Display a toast message with the title information.
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()

    //Return a boolean indicating if event should be consumed or continue bubble up.
    return false
}, layer.getId()) //Limit this event to the symbol layer.
```

::: zone-end

![Анимация касания функции и отображение всплывающего сообщения](media/display-feature-information-android/symbol-layer-click-toast-message.gif)

Помимо всплывающих сообщений существует много других способов представления свойств метаданных функции, например:

- [](https://developer.android.com/training/snackbar/showing.html)  -  Мини `Snackbars` -приложение снаккбар Предоставьте упрощенную обратную связь над операцией. Они показывают короткое сообщение в нижней части экрана на мобильных устройствах, а в нижнем левом углу — на большем. `Snackbars` отображается над всеми другими элементами на экране, и только один может отображаться за раз.
- [Диалоговые окна](https://developer.android.com/guide/topics/ui/dialogs) — это небольшое окно, предлагающее пользователю принять решение или ввести дополнительные сведения. Диалоговое окно не заполняет экран и обычно используется для модальных событий, требующих от пользователя выполнения действия, прежде чем они смогут продолжить работу.
- Добавление [фрагмента](https://developer.android.com/guide/components/fragments) к текущему действию.
- Переход к другому действию или представлению.

## <a name="display-a-popup"></a>Отображение всплывающего окна

Azure Maps пакет SDK для Android предоставляет `Popup` класс, который упрощает создание элементов аннотации пользовательского интерфейса, привязанных к положению на карте. Для всплывающих окон необходимо передать представление с относительным макетом в `content` параметр всплывающего окна. Ниже приведен простой пример макета, отображающий темный текст на фоне.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:background="#ffffff"
    android:layout_margin="8dp"
    android:padding="10dp"

    android:layout_height="match_parent">

    <TextView
        android:id="@+id/message"
        android:layout_width="wrap_content"
        android:text=""
        android:textSize="18dp"
        android:textColor="#222"
        android:layout_height="wrap_content"
        android:width="200dp"/>

</RelativeLayout>
```

Если приведенный выше макет хранится в файле, который называется `popup_text.xml` в `res -> layout` папке приложения, следующий код создает всплывающее окно, добавляет его в карту. При нажатии на эту функцию `title` свойство отображается с помощью `popup_text.xml` макета и снизу от центра макета, привязанного к заданной точке на карте.

::: zone pivot="programming-language-java-android"

```java
//Create a popup and add it to the map.
Popup popup = new Popup();
map.popups.add(popup);

map.events.add((OnFeatureClick)(feature) -> {
    //Get the first feature and it's properties.
    Feature f = feature.get(0);
    JsonObject props = f.properties();

    //Retrieve the custom layout for the popup.
    View customView = LayoutInflater.from(this).inflate(R.layout.popup_text, null);

    //Access the text view within the custom view and set the text to the title property of the feature.
    TextView tv = customView.findViewById(R.id.message);
    tv.setText(props.get("title").getAsString());

    //Get the coordinates from the clicked feature and create a position object.
    List<Double> c = ((Point)(f.geometry())).coordinates();
    Position pos = new Position(c.get(0), c.get(1));

    //Set the options on the popup.
    popup.setOptions(
        //Set the popups position.
        position(pos),

        //Set the anchor point of the popup content.
        anchor(AnchorType.BOTTOM),

        //Set the content of the popup.
        content(customView)

        //Optionally, hide the close button of the popup.
        //, closeButton(false)
    );

    //Open the popup.
    popup.open();

    //Return a boolean indicating if event should be consumed or continue bubble up.
    return false;
});
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create a popup and add it to the map.
val popup = Popup()
map.popups.add(popup)

map.events.add(OnFeatureClick { feature: List<Feature> ->
    //Get the first feature and it's properties.
    val f = feature[0]
    val props = f.properties()

    //Retrieve the custom layout for the popup.
    val customView: View = LayoutInflater.from(this).inflate(R.layout.popup_text, null)

    //Access the text view within the custom view and set the text to the title property of the feature.
    val tv: TextView = customView.findViewById(R.id.message)
    tv.text = props!!["title"].asString

    //Get the coordinates from the clicked feature and create a position object.
    val c: List<Double> = (f.geometry() as Point?).coordinates()
    val pos = Position(c[0], c[1])

    //Set the options on the popup.
    popup.setOptions( 
        //Set the popups position.
        position(pos),  

        //Set the anchor point of the popup content.
        anchor(AnchorType.BOTTOM),  

        //Set the content of the popup.
        content(customView) 

        //Optionally, hide the close button of the popup.
        //, closeButton(false)
    )

    //Open the popup.
    popup.open()

    //Return a boolean indicating if event should be consumed or continue bubble up.
    return false
})
```

::: zone-end

На следующем снимке экрана показаны всплывающие окна, которые появляются при щелчке компонентов и привязаны к заданному расположению на карте при перемещении.

![Анимация отображаемого всплывающего окна и перемещение на карту с помощью всплывающего окна, привязанного к положению на карте](./media/display-feature-information-android/android-popup.gif)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы добавить дополнительные данные на карту, выполните следующие действия.

> [!div class="nextstepaction"]
> [Реагирование на события Map](android-map-events.md)

> [!div class="nextstepaction"]
> [Создание источника данных](create-data-source-android-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer-android.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](android-map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](how-to-add-shapes-to-android-map.md)

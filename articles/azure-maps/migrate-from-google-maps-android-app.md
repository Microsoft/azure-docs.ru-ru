---
title: Руководство по переносу приложения Android | Microsoft Azure Maps
description: В руководстве показано, как перенести приложение Android из Google Карт в Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 08/19/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 7f3d32a782c653b3be8b3d6f6714bc9065a73518
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102042901"
---
# <a name="tutorial-migrate-an-android-app-from-google-maps"></a>Руководство по Перенос приложения Android из Google Карт

Пакет SDK Azure Maps для Android содержит интерфейс API, который похож на веб-пакет SDK. Если вы использовали один из таких пакетов SDK для разработки, большинство основных понятий, рекомендаций и архитектур будут аналогичными. В этом учебнике рассматривается следующее.

> [!div class="checklist"]
> * Загрузка карты
> * локализация карты;
> * добавление маркеров, ломаных линий и многоугольников;
> * Наложение слоя фрагментов
> * Отображение данных о трафике

Пакет SDK для Android в Azure Maps поддерживает минимальную версию API 21 для Android: Android 5.0.0 (Lollipop).

Все примеры предоставлены на языке Java, но с пакетом SDK Azure Maps для Android можно также использовать Kotlin.

Дополнительные сведения о разработке с помощью пакета SDK Azure Maps для Android см. в [этих практических руководствах](how-to-use-android-map-control-library.md).

## <a name="prerequisites"></a>Предварительные требования

1. Создайте учетную запись Azure Maps, выполнив вход на [портал Azure](https://portal.azure.com). Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
2. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](how-to-manage-authentication.md).

## <a name="load-a-map"></a>Загрузка карты

Загрузка карты в приложении Android с помощью Google Карт или Azure Maps состоит из аналогичных этапов. Для использования любого из пакетов SDK потребуется следующее:

* Получить ключ API или подписки для доступа к любой из платформ.
* Добавить XML-код в действие, чтобы указать, где должна отображаться карта и как она должна быть размещена.
* Переопределить все методы жизненного цикла из действия, содержащего представление карты, соответствующими объектами в классе map. В частности, необходимо переопределить следующие методы.
  * `onCreate(Bundle)`
  * `onStart()`
  * `onResume()`
  * `onPause()`
  * `onStop()`
  * `onDestroy()`
  * `onSaveInstanceState(Bundle)`
  * `onLowMemory()`
* Дождаться готовности схемы, прежде чем пытаться получить к ней доступ и программировать ее.

### <a name="before-google-maps"></a>До: Google Карты

Чтобы отобразить карту с помощью пакета SDK для Android в Google Картах, необходимо сделать следующее:

1. Убедитесь, что службы Google Play установлены.
2. Добавьте зависимость для службы Google Карт в файл **gradle.build** модуля:

    `implementation 'com.google.android.gms:play-services-maps:17.0.0'`

3. Добавьте ключ API Google Карт в раздел приложения файла **google\_maps\_api.xml**:

    ```xml
    <meta-data android:name="com.google.android.geo.API_KEY" android:value="YOUR_GOOGLE_MAPS_KEY"/>
    ```

4. Добавьте фрагмент карты в основное действие:

    ```xml
    <com.google.android.gms.maps.MapView
            android:id="@+id/myMap"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>
    ```

5. В файле **MainActivity.java** потребуется импортировать пакет SDK для Google Карт. Перешлите все методы жизненного цикла из действия, содержащего представление карты, в соответствующие объекты в классе map. Извлеките экземпляр `MapView` из фрагмента карты с помощью метода `getMapAsync(OnMapReadyCallback)`. `MapView` автоматически инициализирует систему карт и представление. Измените файл **MainActivity.java** следующим образом:

    ```java
    import com.google.android.gms.maps.GoogleMap;
    import com.google.android.gms.maps.MapView;
    import com.google.android.gms.maps.OnMapReadyCallback;
 
    import android.support.v7.app.AppCompatActivity;
    import android.os.Bundle;
    
    public class MainActivity extends AppCompatActivity implements OnMapReadyCallback {
    
        MapView mapView;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            mapView = findViewById(R.id.myMap);
    
            mapView.onCreate(savedInstanceState);
            mapView.getMapAsync(this);
        }
    
        @Override
    
        public void onMapReady(GoogleMap map) {
            //Add your post map load code here.
        }
    
        @Override
        public void onResume() {
            super.onResume();
            mapView.onResume();
        }
    
        @Override
        protected void onStart(){
            super.onStart();
            mapView.onStart();
        }
    
        @Override
        public void onPause() {
            super.onPause();
            mapView.onPause();
        }
    
        @Override
        public void onStop() {
            super.onStop();
            mapView.onStop();
        }
    
        @Override
        public void onLowMemory() {
            super.onLowMemory();
            mapView.onLowMemory();
        }
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
            mapView.onDestroy();
        }
    
        @Override
        protected void onSaveInstanceState(Bundle outState) {
            super.onSaveInstanceState(outState);
            mapView.onSaveInstanceState(outState);
        }
    }
    ```

При запуске приложения загружается элемент управления картой, как показано на следующем рисунке.

![Простые Google Карты](media/migrate-google-maps-android-app/simple-google-maps.png)

### <a name="after-azure-maps"></a>После: Azure Maps

Чтобы отобразить карту с помощью пакета SDK для Android в Azure Maps, необходимо сделать следующее:

1. Откройте файл верхнего уровня **build.gradle** и добавьте приведенный ниже код в раздел блока **all projects** (все проекты):

    ```java
    maven {
        url "https://atlas.microsoft.com/sdk/android"
    }
    ```

2. Обновите файл **app/build.gradle** и добавьте в него следующий код.

    1. Убедитесь, что **minSdkVersion** проекта находится на уровне версии API 21 или более поздней.

    2. Добавьте в раздел Android следующий код:

        ```java
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }
        ```

    3. Обновите блок зависимостей. Добавьте новую строку зависимости реализации для последнего пакета SDK Azure Maps для Android.

        ```Java
        implementation "com.microsoft.azure.maps:mapcontrol:0.7"
        ```

        > [!Note]
        > Для номера версии можно задать значение "0+", чтобы в вашем коде всегда использовалась последняя версия.

    4. Перейдите в раздел **Файл** на панели инструментов, а затем щелкните **Sync Project with Gradle Files** (Синхронизировать проект с файлами Gradle).

3. Добавьте фрагмент карты в основное действие (пароль к ресурсам \> макет \> activity\_main.xml):

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >

        <com.microsoft.azure.maps.mapcontrol.MapControl
            android:id="@+id/mapcontrol"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            />
    </FrameLayout>
    ```

4. В файле **MainActivity.java** нужно сделать следующее:

    * импортировать пакет SDK Azure Maps;
    * настроить сведения об аутентификации Azure Maps;
    * получить экземпляр элемента управления картой в методе **onCreate**;

     задать сведения для аутентификации в классе `AzureMaps` с помощью метода `setSubscriptionKey` или `setAadProperties`. Это глобальное обновление. Убедитесь, что данные аутентификации добавляются в каждое представление.

    Элемент управления картой содержит собственные методы жизненного цикла для управления жизненным циклом OpenGL Android. Они должны вызываться непосредственно из содержащегося действия. Чтобы правильно вызывать методы жизненного цикла элемента управления картой, переопределите приведенные ниже методы жизненного цикла в действии, содержащем элемент управления картой. Вызовите соответствующий метод элемента управления картой.

    * `onCreate(Bundle)`
    * `onStart()`
    * `onResume()`
    * `onPause()`
    * `onStop()`
    * `onDestroy()`
    * `onSaveInstanceState(Bundle)`
    * `onLowMemory()`

    Измените файл **MainActivity.java** следующим образом:

    ```Java
    package com.example.myapplication;
    
    //For older versions use: import android.support.v7.app.AppCompatActivity; 
    import androidx.appcompat.app.AppCompatActivity;
    import com.microsoft.azure.maps.mapcontrol.AzureMaps;
    import com.microsoft.azure.maps.mapcontrol.MapControl;
    import com.microsoft.azure.maps.mapcontrol.layer.SymbolLayer;
    import com.microsoft.azure.maps.mapcontrol.options.MapStyle;
    import com.microsoft.azure.maps.mapcontrol.source.DataSource;
    
    public class MainActivity extends AppCompatActivity {
    
    static {
        AzureMaps.setSubscriptionKey("<Your Azure Maps subscription key>");

        //Alternatively use Azure Active Directory authenticate.
        //AzureMaps.setAadProperties("<Your aad clientId>", "<Your aad AppId>", "<Your aad Tenant>");
    }

    MapControl mapControl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mapControl = findViewById(R.id.mapcontrol);

        mapControl.onCreate(savedInstanceState);

        //Wait until the map resources are ready.
        mapControl.onReady(map -> {
            //Add your post map load code here.

        });
    }

    @Override
    public void onResume() {
        super.onResume();
        mapControl.onResume();
    }

    @Override
    protected void onStart(){
        super.onStart();
        mapControl.onStart();
    }

    @Override
    public void onPause() {
        super.onPause();
        mapControl.onPause();
    }

    @Override
    public void onStop() {
        super.onStop();
        mapControl.onStop();
    }

    @Override
    public void onLowMemory() {
        super.onLowMemory();
        mapControl.onLowMemory();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mapControl.onDestroy();
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        mapControl.onSaveInstanceState(outState);
    }}
    ```

При запуске приложения будет загружаться элемент управления картой, как показано ниже.

![Простые службы Azure Maps](media/migrate-google-maps-android-app/simple-azure-maps.png)

Обратите внимание, что элемент управления Azure Maps поддерживает более масштабное отображение и предоставляет большее представление целого мира.

> [!TIP]
> При использовании эмулятора Android на компьютере с Windows карта может не отображаться из-за конфликтов с OpenGL и отрисовкой графики с программным ускорением. Ниже описано, как устранить эту проблему. Откройте диспетчер AVD и выберите виртуальное устройство, которое необходимо изменить. Прокрутите страницу вниз до панели **Проверить конфигурацию**. В разделе **Emulated Performance** (Эмулированная производительность) для параметра **Графика** задайте значение **Оборудование**.

## <a name="localizing-the-map"></a>Локализация карты

Если ваша аудитория распределена по нескольким странам или регионам либо говорит на разных языках, то необходима локализация.

### <a name="before-google-maps"></a>До: Google Карты

Добавьте приведенный ниже код в метод `onCreate`, чтобы задать язык карты. Этот код должен размещаться перед фрагментом, который задает представление контекста карты. Код языка "fr" ограничивает язык французским.

```java
String languageToLoad = "fr";
Locale locale = new Locale(languageToLoad);
Locale.setDefault(locale);
Configuration config = new Configuration();
config.locale = locale;
getBaseContext().getResources().updateConfiguration(config,
        getBaseContext().getResources().getDisplayMetrics());
```

Ниже приведен пример использования Google Карт с языком, имеющим значение fr.

![Локализация Google Карт](media/migrate-google-maps-android-app/google-maps-localization.png)

### <a name="after-azure-maps"></a>После: Azure Maps

Azure Maps предоставляет три разных способа настройки языка и регионального представления для карты. Первый вариант — передать сведения о языке и региональном представлении в класс `AzureMaps`. В этом случае глобально используются статические методы `setLanguage` и `setView`. Это позволяет задать язык и региональное представление по умолчанию для всех элементов управления Azure Maps, загруженных в приложение. В этом примере задается французский язык с помощью кода языка "fr-FR".

```java
static {
    //Set your Azure Maps Key.
    AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");

    //Set the language to be used by Azure Maps.
    AzureMaps.setLanguage("fr-FR");

    //Set the regional view to be used by Azure Maps.
    AzureMaps.setView("Auto");
}
```

Второй вариант — передать сведения о языке и представлении в коде XML элемента управления картой.

```xml
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_language="fr-FR"
    app:mapcontrol_view="Auto"
    />
```

Третий вариант — запрограммировать язык и региональное представление карты с помощью метода `setStyle` карт. Такой подход позволяет изменить язык и региональное представление в любой момент при выполнении кода.

```java
mapControl.onReady(map -> {
    map.setStyle(
        language("fr-FR"),
        view("Auto")
    );
});
```

Ниже приведен пример использования Azure Maps с языком, имеющим значение fr-FR.

![Локализация Azure Maps](media/migrate-google-maps-android-app/azure-maps-localization.png)

Просмотрите полный список [поддерживаемых языков](supported-languages.md).

## <a name="setting-the-map-view"></a>Настройка представления карты

Динамические карты в Azure Maps и Google Картах можно программно перемещать в новые географические расположения путем вызова соответствующих методов. Давайте сделаем так, чтобы на карте отображались спутниковые аэроснимки, отцентрируем карту по местоположению с использованием координат и изменим масштаб. В этом примере мы используем широту 35,0272, долготу –111,0225 и коэффициент масштабирования 15.

### <a name="before-google-maps"></a>До: Google Карты

Камеру элемента управления картой в Google Картах можно программно перемещать с помощью метода `moveCamera`. Метод `moveCamera` позволяет указать центр карты и коэффициент масштабирования. Метод `setMapType` изменяет тип отображаемой карты.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.moveCamera(CameraUpdateFactory.newLatLngZoom(new LatLng(35.0272, -111.0225), 15));
    mapView.setMapType(GoogleMap.MAP_TYPE_SATELLITE);
}
```

![Заданное представление Google Карт](media/migrate-google-maps-android-app/google-maps-set-view.png)

> [!NOTE]
> В Google Картах используются фрагменты размером 256 пикселей, а в Azure Maps — более крупные фрагменты размером 512 пикселей. Благодаря этому сокращается количество сетевых запросов, необходимых Azure Maps для загрузки той же области карты, что и в Google Картах. Чтобы при использовании Azure Maps добиться видимой области, как у карты в Google Картах, из коэффициента масштабирования, используемого в Google Картах, необходимо вычесть единицу.

### <a name="after-azure-maps"></a>После: Azure Maps

Как отмечалось ранее, чтобы получить ту же видимую область в Azure Maps, вычтите из коэффициента масштабирования, используемого в Google Картах, единицу. В данном случае используйте коэффициент масштабирования 14.

Начальное представление карты можно задать в атрибутах XML в элементе управления картой.

```xml
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_cameraLat="35.0272"
    app:mapcontrol_cameraLng="-111.0225"
    app:mapcontrol_zoom="14"
    app:mapcontrol_style="satellite"
    />
```

Представление карты можно программировать с помощью методов `setCamera` и `setStyle` карт.

```java
mapControl.onReady(map -> {
    //Set the camera of the map.
    map.setCamera(center(Point.fromLngLat(-111.0225, 35.0272)), zoom(14));

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE));
});
```

![Заданное представление Azure Maps](media/migrate-google-maps-android-app/azure-maps-set-view.png)

**Дополнительные ресурсы:**

* [Поддерживаемые стили карт](supported-map-styles.md)

## <a name="adding-a-marker"></a>Добавление метки

Данные точек часто отображаются на карте с помощью изображения. Эти изображения называются метками, вешками, кнопками или символами. В следующих примерах данные точек отображаются в виде меток на карте по координатам: широта — 51,5, долгота — −0,2.

### <a name="before-google-maps"></a>До: Google Карты

При использовании Google Карт метки добавляются с помощью метода `addMarker` карт.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.addMarker(new MarkerOptions().position(new LatLng(47.64, -122.33)));
}
```

![Метка Google Карт](media/migrate-google-maps-android-app/google-maps-marker.png)

### <a name="after-azure-maps"></a>После: Azure Maps

Чтобы отобразить данные точек на карте в Azure Maps, сначала добавьте эти данные в источник данных. Затем подключите этот источник данных к слою символов. Источник данных позволяет оптимизировать управление пространственными данными в элементе управления картой. Слой символов определяет способ отображения данных (в виде изображения или текста).

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create a point feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));

    //Create a symbol layer and add it to the map.
    map.layers.add(new SymbolLayer(dataSource));
});
```

![Метка Azure Maps](media/migrate-google-maps-android-app/azure-maps-marker.png)

## <a name="adding-a-custom-marker"></a>Добавление настраиваемой метки

Для представления точек на карте можно использовать пользовательские изображения. В приведенных ниже примерах карты пользовательское изображение применяется для отображения точки на карте. Координаты точки (широта: 51,5, долгота −0,2. Привязка смещает положение маркера таким образом, чтобы точка значка вешки была правильно размещена на карте.

![Изображение желтой вешки](media/migrate-google-maps-web-app/yellow-pushpin.png)<br/>
yellow-pushpin.png

В обоих примерах приведенное выше изображение добавляется в папку с прорисовываемыми ресурсами приложений.

### <a name="before-google-maps"></a>До: Google Карты

В Google Картах пользовательские изображения можно использовать в качестве меток. Для этого загрузите их с помощью параметра `icon` метки. Параметр `anchor` позволяет сопоставить точку изображения с координатами. Привязка задается относительно размеров изображения. В этом случае привязка имеет ширину 0,2 единицы и высоту 1 единицу.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.addMarker(new MarkerOptions().position(new LatLng(47.64, -122.33))
    .icon(BitmapDescriptorFactory.fromResource(R.drawable.yellow-pushpin))
    .anchor(0.2f, 1f));
}
```

![Пользовательская метка Google Карт](media/migrate-google-maps-android-app/google-maps-custom-marker.png)

### <a name="after-azure-maps"></a>После: Azure Maps

Слои символов в Azure Maps также поддерживают пользовательские изображения, но для этого необходимо заранее загрузить изображение в ресурсы карты и назначить ему уникальный идентификатор. Затем слой символов должен ссылаться на этот идентификатор. Задайте смещение для символа, чтобы он указывал на нужную точку на изображении, используя параметр `iconOffset`. Смещение значка указывается в пикселях. По умолчанию смещение задается относительно нижней центральной точки изображения, но это можно изменить с помощью параметра `iconAnchor`. В этом примере для параметра `iconAnchor` задается значение `"center"`. Используется смещение значка для перемещения изображения на 5 пикселей вправо и на 15 пикселей вверх в соответствии с точкой изображения вешки.

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create a point feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));

    //Load the custom image icon into the map resources.
    map.images.add("my-yellow-pin", R.drawable.yellow_pushpin);

    //Create a symbol that uses the custom image icon and add it to the map.
    map.layers.add(new SymbolLayer(dataSource,
        iconImage("my-yellow-pin"),
        iconAnchor(AnchorType.CENTER
        iconOffset(new Float[]{5f, -15f})));
});
```

![Пользовательская метка Azure Maps](media/migrate-google-maps-android-app/azure-maps-custom-marker.png)

## <a name="adding-a-polyline"></a>Добавление ломаной линии

Ломаные линии используются для представления линий или траекторий на карте. В следующих примерах показано, как создать пунктирную ломаную линию на карте.

### <a name="before-google-maps"></a>До: Google Карты

В Google Картах для отрисовки ломаной линии используется класс `PolylineOptions`. Добавить ломаную линию на карту можно с помощью метода `addPolyline`. Задать цвет штриха можно с помощью параметра `color`. Задать ширину штриха можно с помощью параметра `width`. Добавить массив штриха можно с помощью параметра `pattern`.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the polyline.
    PolylineOptions lineOptions = new PolylineOptions()
            .add(new LatLng(46, -123))
            .add(new LatLng(49, -122))
            .add(new LatLng(46, -121))
            .color(Color.RED)
            .width(10f)
            .pattern(Arrays.<PatternItem>asList(
                    new Dash(30f), new Gap(30f)));

    //Add the polyline to the map.
    Polyline polyline = mapView.addPolyline(lineOptions);
}
```

![Ломаная линия в Google Картах](media/migrate-google-maps-android-app/google-maps-polyline.png)

### <a name="after-azure-maps"></a>После: Azure Maps

В Azure Maps ломаные линии называются объектами `LineString` или `MultiLineString`. Эти объекты можно добавлять в источник данных и отобразить с помощью слоя линий. Задать ширину штриха можно с помощью параметра `strokeWidth`. Добавить массив штриха можно с помощью параметра `strokeDashArray`.

Единица ширины штриха и единица "пиксель" массива штриха в веб-пакете SDK Azure Maps и в службе Google Maps аналогичны. Оба параметра выдают одинаковые результаты при одинаковых входных значениях.

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create an array of points.
    List<Point> points = Arrays.asList(
        Point.fromLngLat(-123, 46),
        Point.fromLngLat(-122, 49),
        Point.fromLngLat(-121, 46));

    //Create a LineString feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(LineString.fromLngLats(points)));

    //Create a line layer and add it to the map.
    map.layers.add(new LineLayer(dataSource,
        strokeColor("red"),
        strokeWidth(4f),
        strokeDashArray(new Float[]{3f, 3f})));
});
```

![Ломаная линия в Azure Maps](media/migrate-google-maps-android-app/azure-maps-polyline.png)

## <a name="adding-a-polygon"></a>Добавление многоугольника

Многоугольники используются для представления области на карте. В следующих примерах показано, как создать многоугольник. Этот многоугольник образует треугольник вокруг координат центральной точки на карте.

### <a name="before-google-maps"></a>До: Google Карты

С помощью Google Карт можно отрисовать многоугольник, используя класс `PolygonOptions`. Добавить многоугольник на карту можно с помощью метода `addPolygon`. Чтобы задать цвета заливки и обводки, можно использовать параметры `fillColor` и `strokeColor` соответственно. Задать ширину штриха можно с помощью параметра `strokeWidth`.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the polygon.
    PolygonOptions polygonOptions = new PolygonOptions()
            .add(new LatLng(46, -123))
            .add(new LatLng(49, -122))
            .add(new LatLng(46, -121))
            .add(new LatLng(46, -123))  //Close the polygon.
            .fillColor(Color.argb(128, 0, 128, 0))
            .strokeColor(Color.RED)
            .strokeWidth(5f);

    //Add the polygon to the map.
    Polygon polygon = mapView.addPolygon(polygonOptions);
}
```

![Многоугольник в Google Картах](media/migrate-google-maps-android-app/google-maps-polygon.png)

### <a name="after-azure-maps"></a>После: Azure Maps

В Azure Maps объекты `Polygon` и `MultiPolygon` можно добавить в источник данных и отобразить их на карте с помощью слоев. Область многоугольника можно отобразить на слое многоугольников. Контур многоугольника можно визуализировать с помощью слоя линий. Чтобы задать цвет и ширину обводки, используются параметры `strokeColor` и `strokeWidth`.

Единица ширины штриха и единица "пиксель" массива штриха в веб-пакете SDK Azure Maps и Google Maps аналогичны. Оба параметра выдают одинаковые результаты при одинаковых входных значениях.

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create an array of point arrays to create polygon rings.
    List<List<Point>> rings = Arrays.asList(Arrays.asList(
        Point.fromLngLat(-123, 46),
        Point.fromLngLat(-122, 49),
        Point.fromLngLat(-121, 46),
        Point.fromLngLat(-123, 46)));

    //Create a Polygon feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Polygon.fromLngLats(rings)));

    //Add a polygon layer for rendering the polygon area.
    map.layers.add(new PolygonLayer(dataSource,
        fillColor("green"),
        fillOpacity(0.5f)));

    //Add a line layer for rendering the polygon outline.
    map.layers.add(new LineLayer(dataSource,
        strokeColor("red"),
        strokeWidth(2f)));
});
```

![Многоугольник в Azure Maps](media/migrate-google-maps-android-app/azure-maps-polygon.png)

## <a name="overlay-a-tile-layer"></a>Наложение слоя фрагментов

 Слои фрагментов позволяют накладывать изображения слоев, разбитые на небольшие фрагменты, которые соответствуют системе разбиения карты. Это распространенный подход для наложения изображений слоев или больших наборов данных. Слои фрагментов в Google Картах называются наложениями изображений.

В приведенных ниже примерах накладывается слой фрагментов радара погоды из лаборатории окружающей среды Университета штата Айова. Размер фрагментов составляет 256 пикселей.

### <a name="before-google-maps"></a>До: Google Карты

С помощью Google Карт слой фрагментов можно разместить поверх карты. Для этого используется класс `TileOverlayOptions`. Добавить слой фрагментов на карту можно с помощью метода `addTileLauer`. Чтобы сделать фрагменты карты полупрозрачными, для параметра `transparency` необходимо задать значение 0,2 или 20 % прозрачности.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the tile layer.
    TileOverlayOptions tileLayer = new TileOverlayOptions()
        .tileProvider(new UrlTileProvider(256, 256) {
            @Override
            public URL getTileUrl(int x, int y, int zoom) {

                try {
                    //Define the URL pattern for the tile images.
                    return new URL(String.format("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/%d/%d/%d.png", zoom, x, y));
                }catch (MalformedURLException e) {
                    throw new AssertionError(e);
                }
            }
        }).transparency(0.2f);

    //Add the tile layer to the map.
    mapView.addTileOverlay(tileLayer);
}
```

![Слой фрагментов в Google Картах](media/migrate-google-maps-android-app/google-maps-tile-layer.png)

### <a name="after-azure-maps"></a>После: Azure Maps

Слой фрагментов можно добавить на карту почти так же, как любой другой слой. Форматированный URL-адрес с заполнителями значений x, y и масштаба (`{x}`, `{y}`, `{z}` соответственно) используется для указания уровню места доступа к фрагментам. Слои фрагментов в Azure Maps также поддерживают заполнители `{quadkey}`, `{bbox-epsg-3857}` и `{subdomain}`. Чтобы слой фрагментов был частично прозрачным, используется значение непрозрачности 0,8. Обратите внимание, что непрозрачность и прозрачность обозначают один и тот же параметр, но их значения противоположны. Чтобы преобразовать одно из них в другое, вычтите исходное значение из единицы.

> [!TIP]
> В Azure Maps слои можно легко отобразить под другими слоями, в том числе под базовыми слоями карт. Также часто желательно отображать слои фрагментов под метками карты, чтобы их можно было легко читать. Метод `map.layers.add` принимает в качестве второго параметра идентификатор слоя, под которым нужно вставить новый слой. Чтобы вставить слой фрагментов под метками карты, можно использовать следующий код: `map.layers.add(myTileLayer, "labels");`

```java
mapControl.onReady(map -> {
    //Add a tile layer to the map, below the map labels.
    map.layers.add(new TileLayer(
        tileUrl("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png"),
        opacity(0.8f),
        tileSize(256)
    ), "labels");
});
```

![Слой фрагментов в Azure Maps](media/migrate-google-maps-android-app/azure-maps-tile-layer.png)

## <a name="show-traffic"></a>Отображение данных дорожного движения

И в Azure Maps, и в Google Картах есть параметры для наложения данных дорожного движения.

### <a name="before-google-maps"></a>До: Google Карты

В Google Картах данные дорожного движения можно переносить на карту, передавая значение true в метод `setTrafficEnabled` карты.

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.setTrafficEnabled(true);
}
```

![Данные дорожного движения в Google Картах](media/migrate-google-maps-android-app/google-maps-traffic.png)

### <a name="after-azure-maps"></a>После: Azure Maps

Azure Maps предоставляет несколько различных вариантов отображения дорожного движения. Дорожно-транспортные происшествия, такие как закрытие дорог и аварии, можно отображать на карте в виде значков. На карту можно наложить сведения о транспортном потоке с цветовым кодированием сегментов дорог. Цвета сегментов можно изменять в соответствии с ограничениями скорости, относительно нормальной ожидаемой или абсолютной задержки. Данные о дорожно-транспортных происшествиях в Azure Maps обновляются каждую минуту, а данные о дорожном движении — каждые две минуты.

```java
mapControl.onReady(map -> {
    map.setTraffic(
        incidents(true),
        flow(TrafficFlow.RELATIVE));
});
```

![Данные дорожного движения в Azure Maps](media/migrate-google-maps-android-app/azure-maps-traffic.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов для очистки.

## <a name="next-steps"></a>Следующие шаги

Узнайте больше о переходе на Azure Maps:

> [!div class="nextstepaction"]
> [Перенос приложения Android](migrate-from-google-maps-android-app.md)

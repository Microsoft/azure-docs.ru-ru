---
title: Создание источника данных для карт Android | Карты Microsoft Azure
description: 'Узнайте, как создать источник данных для схемы. Сведения об источниках данных, которые использует Azure Maps пакет SDK для Android: источники и векторные плитки геообъектной JSON.'
author: rbrundritt
ms.author: richbrun
ms.date: 12/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: fc68dc25aad3671a55e5c11cbee094b4027e7070
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102047678"
---
# <a name="create-a-data-source-android-sdk"></a>Создание источника данных (пакет SDK для Android)

Azure Maps пакет SDK для Android сохраняет данные в источниках данных. Использование источников данных оптимизирует операции с данными для выполнения запросов и подготовки к просмотру. В настоящее время существует два типа источников данных:

- **Источник** географического формата. управляет данными необработанных расположений в формате геоjson локально. Подходит для небольших и средних наборов данных (в направлении сотен тысяч фигур).
- **Источник плитки вектора**: загружает данные, отформатированные как векторные плитки для текущего представления карты, на основе системы разбиения на карты. Идеально подходит для больших и огромных наборов данных (миллионы или миллиарды фигур).

## <a name="geojson-data-source"></a>Источник данных геоjson

Azure Maps использует геоjson в качестве одной из основных моделей данных. Геообъектный формат JSON представляет собой открытый геопространственно стандартный способ представления геопространственных данных в формате JSON. Классы геоjson, доступные в Azure Maps пакет SDK для Android для простоты создания и сериализации данных геоjson. Загрузка и хранение данных геообъектного формата JSON в `DataSource` классе и их отрисовка с помощью слоев. В следующем коде показано, как можно создать геообъектные объекты JSON в Azure Maps.

```java
/*
    Raw GeoJSON feature
    
    {
         "type": "Feature",
         "geometry": {
             "type": "Point",
             "coordinates": [-100, 45]
         },
         "properties": {
             "custom-property": "value"
         }
    }

*/

//Create a point feature.
Feature feature = Feature.fromGeometry(Point.fromLngLat(-100, 45));

//Add a property to the feature.
feature.addStringProperty("custom-property", "value");

//Add the feature to the data source.
source.add(feature);
```

Кроме того, свойства можно загрузить в JsonObject в первую очередь, а затем передать в функцию при ее создании, как показано ниже.

```java
//Create a JsonObject to store properties for the feature.
JsonObject properties = new JsonObject();
properties.addProperty("custom-property", "value");

Feature feature = Feature.fromGeometry(Point.fromLngLat(-100, 45), properties);
```

После создания функции геоjson источник данных можно добавить на карту с помощью `sources` Свойства Map. В следующем коде показано, как создать `DataSource` , добавить его в карту и добавить в источник данных функцию.

```javascript
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Add GeoJSON feature to the data source.
source.add(feature);
```

В следующем коде показано несколько способов создания функции геоjson, Феатуреколлектион и геометрических объектов.

```java
//GeoJSON Point Geometry
Point point = Point.fromLngLat(LONGITUDE, LATITUDE);

//GeoJSON Point Geometry
LineString linestring = LineString.fromLngLats(PointList);

//GeoJSON Polygon Geometry
Polygon polygon = Polygon.fromLngLats(listOfPointList);
 
Polygon polygonFromOuterInner = Polygon.fromOuterInner(outerLineStringObject,innerLineStringObject);

//GeoJSON MultiPoint Geometry
MultiPoint multiPoint = MultiPoint.fromLngLats(PointList);

//GeoJSON MultiLineString Geometry
MultiLineString multiLineStringFromLngLat = MultiLineString.fromLngLats(listOfPointList);

MultiLineString multiLineString = MultiLineString.fromLineString(singleLineString);

//GeoJSON MultiPolygon Geometry
MultiPolygon multiPolygon = MultiPolygon.fromLngLats(listOflistOfPointList);

MultiPolygon multiPolygonFromPolygon = MultiPolygon.fromPolygon(polygon);

MultiPolygon multiPolygonFromPolygons = MultiPolygon.fromPolygons(PolygonList);

//GeoJSON Feature
Feature pointFeature = Feature.fromGeometry(Point.fromLngLat(LONGITUDE, LATITUDE));
 
//GeoJSON FeatureCollection 
FeatureCollection featureCollectionFromSingleFeature = FeatureCollection.fromFeature(pointFeature);
 
FeatureCollection featureCollection = FeatureCollection.fromFeatures(listOfFeatures);
```

### <a name="serialize-and-deserialize-geojson"></a>Сериализация и десериализация геоjson

Классы коллекций функций, функций и геометрических методов имеют `fromJson()` и `toJson()` статические методы, которые помогают при сериализации. Отформатированная допустимая строка JSON, переданная с помощью метода, создаст `fromJson()` объект Geometry. Этот `fromJson()` метод также означает, что вы можете использовать Gson или другие стратегии сериализации и десериализации. В следующем коде показано, как взять переведенные геообъект JSON и десериализовать его в класс Feature, а затем сериализовать его обратно в строку геоjson.

```java
//Take a stringified GeoJSON object.
String GeoJSON_STRING = "{"
    + "      \"type\": \"Feature\","            
    + "      \"geometry\": {"
    + "            \"type\": \"Point\""
    + "            \"coordinates\": [-100, 45]"
    + "      },"
    + "      \"properties\": {"
    + "            \"custom-property\": \"value\""
    + "      },"
    + "}";

//Deserialize the JSON string into a feature.
Feature feature = Feature.fromJson(GeoJSON_STRING);
 
//Serialize a feature collection to a string.
String featureString = feature.toJson();
```

### <a name="import-geojson-data-from-web-or-assets-folder"></a>Импорт данных GeoJSON из Интернета или из папки assets

В большинстве геофайловых файлов JSON содержится Феатуреколлектион. Чтение геофайловых файлов JSON в виде строк и использование `FeatureCollection.fromJson` метода для десериализации.

Следующий код является повторно используемым классом для импорта данных из папки Web или local Assets в виде строки и возврата их в поток пользовательского интерфейса с помощью функции обратного вызова.

```java
import android.content.Context;
import android.os.Handler;
import android.os.Looper;
import android.webkit.URLUtil;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.URL;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import javax.net.ssl.HttpsURLConnection;

public class Utils {

    interface SimpleCallback {
        void notify(String result);
    }

    /**
     * Imports data from a web url or asset file name and returns it to a callback.
     * @param urlOrFileName A web url or asset file name that points to data to load.
     * @param context The context of the app.
     * @param callback The callback function to return the data to.
     */
    public static void importData(String urlOrFileName, Context context, SimpleCallback callback){
        importData(urlOrFileName, context, callback, null);
    }
    
    /**
     * Imports data from a web url or asset file name and returns it to a callback.
     * @param urlOrFileName A web url or asset file name that points to data to load.
     * @param context The context of the app.
     * @param callback The callback function to return the data to.
     * @param error A callback function to return errors to.
     */
    public static void importData(String urlOrFileName, Context context, SimpleCallback callback, SimpleCallback error){
        if(urlOrFileName != null && callback != null) {
            ExecutorService executor = Executors.newSingleThreadExecutor();
            Handler handler = new Handler(Looper.getMainLooper());

            executor.execute(() -> {
                String data = null;

                try {

                    if(URLUtil.isNetworkUrl(urlOrFileName)){
                        data = importFromWeb(urlOrFileName);
                    } else {
                        //Assume file is in assets folder.
                        data = importFromAssets(context, urlOrFileName);
                    }

                    final String result = data;

                    handler.post(() -> {
                        //Ensure the resulting data string is not null or empty.
                        if (result != null && !result.isEmpty()) {
                            callback.notify(result);
                        } else {
                            error.notify("No data imported.");
                        }
                    });
                } catch(Exception e) {
                    if(error != null){
                        error.notify(e.getMessage());
                    }
                }
            });
        }
    }

    /**
     * Imports data from an assets file as a string.
     * @param context The context of the app.
     * @param fileName The asset file name.
     * @return
     * @throws IOException
     */
    private static String importFromAssets(Context context, String fileName) throws IOException {
        InputStream stream = null;

        try {
            stream = context.getAssets().open(fileName);

            if(stream != null) {
                return readStreamAsString(stream);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // Close Stream and disconnect HTTPS connection.
            if (stream != null) {
                stream.close();
            }
        }

        return null;
    }

    /**
     * Imports data from the web as a string.
     * @param url URL to the data.
     * @return
     * @throws IOException
     */
    private static String importFromWeb(String url) throws IOException {
        InputStream stream = null;
        HttpsURLConnection connection = null;
        String result = null;

        try {
            connection = (HttpsURLConnection) new URL(url).openConnection();

            //For this use case, set HTTP method to GET.
            connection.setRequestMethod("GET");

            //Open communications link (network traffic occurs here).
            connection.connect();

            int responseCode = connection.getResponseCode();
            if (responseCode != HttpsURLConnection.HTTP_OK) {
                throw new IOException("HTTP error code: " + responseCode);
            }

            //Retrieve the response body as an InputStream.
            stream = connection.getInputStream();

            if (stream != null) {
                return readStreamAsString(stream);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // Close Stream and disconnect HTTPS connection.
            if (stream != null) {
                stream.close();
            }
            if (connection != null) {
                connection.disconnect();
            }
        }

        return result;
    }

    /**
     * Reads an input stream as a string.
     * @param stream Stream to convert.
     * @return
     * @throws IOException
     */
    private static String readStreamAsString(InputStream stream) throws IOException {
        //Convert the contents of an InputStream to a String.
        BufferedReader in = new BufferedReader(new InputStreamReader(stream, "UTF-8"));

        String inputLine;
        StringBuffer response = new StringBuffer();

        while ((inputLine = in.readLine()) != null) {
            response.append(inputLine);
        }

        in.close();

        return response.toString();
    }
}
```

В приведенном ниже коде показано, как использовать эту служебную программу для импорта геообъектных данных JSON в виде строки и возврата их в поток пользовательского интерфейса с помощью обратного вызова. В обратном вызове строковые данные можно сериализовать в коллекцию функций геоjson и добавить в источник данных. При необходимости обновите камеру карты, чтобы сосредоточиться на данных.

```java
//Create a data source and add it to the map.
DataSource dataSource = new DataSource();
map.sources.add(dataSource);

//Import the geojson data and add it to the data source.
Utils.importData("URL_or_FilePath_to_GeoJSON_data",
    this,
    (String result) -> {
        //Parse the data as a GeoJSON Feature Collection.
        FeatureCollection fc = FeatureCollection.fromJson(result);

        //Add the feature collection to the data source.
        dataSource.add(fc);

        //Optionally, update the maps camera to focus in on the data.

        //Calculate the bounding box of all the data in the Feature Collection.
        BoundingBox bbox = MapMath.fromData(fc);

        //Update the maps camera so it is focused on the data.
        map.setCamera(
            bounds(bbox),
            padding(20));
    });
```

## <a name="vector-tile-source"></a>Источник плитки вектора

Источник векторной плитки описывает, как получить доступ к слою векторной плитки. Используйте `VectorTileSource` класс для создания экземпляра источника плитки вектора. Векторные мозаичные слои похожи на мозаичные слои, но они не одинаковы. Мозаичный слой — это растровое изображение. Векторные мозаичные слои — это сжатый файл в формате **ПБФ** . Этот сжатый файл содержит данные о векторной карте и один или несколько слоев. Файл может быть визуализирован и применен на стороне клиента в зависимости от стиля каждого слоя. Данные в векторной плитке содержат географические функции в виде точек, линий и многоугольников. Существует несколько преимуществ использования векторных слоев, а не растровых слоев мозаики.

- Размер файла плитки вектора обычно намного меньше, чем эквивалентная растровая плитка. Поэтому используется меньшая пропускная способность. Это означает более низкую задержку, более быструю карту и удобство работы пользователей.
- Так как векторные плитки отображаются на клиенте, они адаптируются к разрешению устройства, на котором они отображаются. В результате отображаемые карты будут более четко определены с помощью меток Crystal Clear.
- Изменение стиля данных в картах вектора не требует повторной загрузки данных, так как на клиенте можно применить новый стиль. В отличие от этого, изменение стиля растрового слоя, как правило, требует загрузки плиток с сервера, а затем применения нового стиля.
- Поскольку данные доставляются в виде векторной формы, для подготовки данных требуется меньше обработки на стороне сервера. В результате новые данные можно сделать доступными быстрее.

Azure Maps соответствует [спецификации плитки вектора мапбокс](https://github.com/mapbox/vector-tile-spec)— открытым стандартом. Azure Maps предоставляет следующие службы векторных плиток в составе платформы:

- [](/rest/api/maps/renderv2/getmaptilepreview)  |  [Сведения о формате данных](https://developer.tomtom.com/maps-api/maps-api-documentation-vector/tile) документации для дорожных плиток
- [](/rest/api/maps/traffic/gettrafficincidenttile)  |  [Сведения о формате данных](https://developer.tomtom.com/traffic-api/traffic-api-documentation-traffic-incidents/vector-incident-tiles) в документации по инцидентам трафика
- [](/rest/api/maps/traffic/gettrafficflowtile)  |  [Сведения о формате данных](https://developer.tomtom.com/traffic-api/traffic-api-documentation-traffic-flow/vector-flow-tiles) документации потока трафика
- Создатель Azure Maps также позволяет создавать пользовательские векторные плитки и получать к ним доступ через элемент [Get плитка для прорисовки v2](/rest/api/maps/renderv2/getmaptilepreview) .

> [!TIP]
> При использовании векторных или растровых изображений из Azure Maps службы отрисовки с веб-пакетом SDK можно заменить `atlas.microsoft.com` заполнительом `azmapsdomain.invalid` . Этот заполнитель будет заменен тем же доменом, который используется картой, и будет автоматически добавлять те же сведения проверки подлинности. Это значительно упрощает проверку подлинности с помощью службы Render при использовании Azure Active Directory проверки подлинности.

Чтобы отобразить данные из источника векторной плитки на карте, подключите источник к одному из слоев подготовки данных. Все слои, использующие Векторный источник, должны указывать `sourceLayer` значение в параметрах. Следующий код загружает службу плитки векторного потока трафика Azure Maps в качестве источника векторной плитки, а затем отображает ее на карте с помощью слоя линий. Этот источник векторной плитки содержит один набор данных на исходном слое с именем "поток трафика". Данные строки в этом наборе данных имеют свойство с именем `traffic_level` , которое используется в этом коде для выбора цвета и масштабирования размера строк.

```java
//Formatted URL to the traffic flow vector tiles, with the maps subscription key appended to it.
String trafficFlowUrl = "https://azmapsdomain.invalid/traffic/flow/tile/pbf?api-version=1.0&style=relative&zoom={z}&x={x}&y={y}";

//Create a vector tile source and add it to the map.
VectorTileSource source = new VectorTileSource(
    tiles(new String[] { trafficFlowUrl }),
    maxSourceZoom(22)
);
map.sources.add(source);

//Create a layer for traffic flow lines.
LineLayer layer = new LineLayer(source,
    //The name of the data layer within the data source to pass into this rendering layer.
    sourceLayer("Traffic flow"),

    //Color the roads based on the traffic_level property.
    strokeColor(
        interpolate(
            linear(),
            get("traffic_level"),
            stop(0, color(Color.RED)),
            stop(0.33, color(Color.YELLOW)),
            stop(0.66, color(Color.GREEN))
        )
    ),

    //Scale the width of roads based on the traffic_level property.
    strokeWidth(
        interpolate(
            linear(),
            get("traffic_level"),
            stop(0, 6),
            stop(1,1)
        )
    )
);

//Add the traffic flow layer below the labels to make the map clearer.
map.layers.add(layer, "labels");
```

![Map с цветными полосами дорожек, демонстрирующими уровни потока трафика](media/create-data-source-android-sdk/android-vector-tile-source-line-layer.png)

## <a name="connecting-a-data-source-to-a-layer"></a>Подключение источника данных к слою

Данные подготавливаются к просмотру на карте с помощью слоев подготовки отчетов. Один или несколько уровней подготовки отчетов могут ссылаться на один источник данных. Для следующих слоев подготовки отчетов требуется источник данных:

- [Пузырьковый слой](map-add-bubble-layer-android.md) — отображает данные точек в виде масштабируемых кругов на карте.
- [Слой символов](how-to-add-symbol-to-android-map.md) — отрисовывает данные точек в виде значков или текста.
- [Уровень тепловой карт](map-add-heat-map-layer-android.md) — отрисовывает данные точек в виде плотной тепловой карте.
- [Слой линии](android-map-add-line-layer.md) — прорисовка линии и отображение контура многоугольников.
- [Слой многоугольников](how-to-add-shapes-to-android-map.md) . Заливка области многоугольника сплошным цветом или узором изображения.

В следующем коде показано, как создать источник данных, добавить его на карту и подключить его к пузырьковой слою. Затем импортируйте данные географической точки JSON из удаленного расположения в источник данных.

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Create a layer that defines how to render points in the data source and add it to the map.
BubbleLayer layer = new BubbleLayer(source);
map.layers.add(layer);

//Import the geojson data and add it to the data source.
Utils.importData("URL_or_FilePath_to_GeoJSON_data",
    this,
    (String result) -> {
        //Parse the data as a GeoJSON Feature Collection.
        FeatureCollection fc = FeatureCollection.fromJson(result);

        //Add the feature collection to the data source.
        dataSource.add(fc);

        //Optionally, update the maps camera to focus in on the data.

        //Calculate the bounding box of all the data in the Feature Collection.
        BoundingBox bbox = MapMath.fromData(fc);

        //Update the maps camera so it is focused on the data.
        map.setCamera(
            bounds(bbox),
            padding(20));
    });
```

Существуют дополнительные слои подготовки к просмотру, которые не подключаются к этим источникам данных, но они напрямую загружают данные для подготовки к просмотру.

- [Мозаичный слой](how-to-add-tile-layer-android-map.md) . переносит растровый слой мозаики поверх карты.

## <a name="one-data-source-with-multiple-layers"></a>Один источник данных с несколькими слоями

К одному источнику данных можно подключить несколько слоев. Существует множество различных сценариев, в которых этот вариант полезен. Например, рассмотрим ситуацию, в которой пользователь рисует многоугольник. Мы должны подготовить и заполнить область многоугольника при добавлении пользователем точек на карту. Добавление линии стиля для контура многоугольник упрощает просмотр краев многоугольника по мере рисования пользователем. Чтобы изменить отдельную точку на многоугольнике, мы можем добавить маркер, например ПИН-код или маркер, над каждой позицией.

![Схема, отображающая несколько слоев данные для отрисовки из одного источника данных](media/create-data-source-android-sdk/multiple-layers-one-datasource.png)

Для большинства платформ сопоставления требуется объект многоугольника, объект Line и ПИН-код для каждой из позиций многоугольника. После изменения многоугольника необходимо вручную обновить линию и ПИН-коды, что может быстро стать сложным.

В Azure Maps необходимо всего один многоугольник в источнике данных, как показано в приведенном ниже коде.

```java
//Create a data source and add it to the map.
DataSource source = new DataSource();
map.sources.add(source);

//Create a polygon and add it to the data source.
source.add(Polygon.fromLngLats(/* List of points */));

//Create a polygon layer to render the filled in area of the polygon.
PolygonLayer polygonLayer = new PolygonLayer(source,
    fillColor("rgba(255,165,0,0.2)")
);

//Create a line layer for greater control of rendering the outline of the polygon.
LineLayer lineLayer = new LineLayer(source,
    strokeColor("orange"),
    strokeWidth(2f)
);

//Create a bubble layer to render the vertices of the polygon as scaled circles.
BubbleLayer bubbleLayer = new BubbleLayer(source,
    bubbleColor("orange"),
    bubbleRadius(5f),
    bubbleStrokeColor("white"),
    bubbleStrokeWidth(2f)
);

//Add all layers to the map.
map.layers.add(new Layer[] { polygonLayer, lineLayer, bubbleLayer });
```

> [!TIP]
> При добавлении слоев к сопоставлению с помощью `map.layers.add` метода идентификатор или экземпляр существующего слоя можно передать в качестве второго параметра. Это указывает на то, что на карте будет вставлен новый слой, добавляемый под существующим слоем. В помимо для передачи идентификатора слоя этот метод также поддерживает следующие значения.
>
> - `"labels"` — Вставляет новый слой под слоями меток карт.
> - `"transit"` — Вставляет новый слой под слоями «дорога» и «транзитный».

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](create-data-source-android-sdk.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer-android.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](android-map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](how-to-add-shapes-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление тепловой карты](map-add-heat-map-layer-android.md)

> [!div class="nextstepaction"]
> [Примеры кода веб-пакета SDK](/samples/browse/?products=azure-maps)
---
title: Руководство. Загрузка данных GeoJSON в пакет SDK Azure Maps для Android | Microsoft Azure Maps
description: Руководство по загрузке файла данных GeoJSON в пакет SDK Azure Maps для Android.
author: rbrundritt
ms.author: richbrun
ms.date: 12/10/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: 8300a7c120ce816c8068a88fa69f4f978fa664ca
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102034512"
---
# <a name="tutorial-load-geojson-data-into-azure-maps-android-sdk"></a>Руководство. Загрузка данных GeoJSON в пакет SDK Azure Maps для Android

В этом руководстве показано, как импортировать GeoJSON-файл с данными о расположении в пакет SDK Azure Maps для Android. Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * добавление Azure Maps в приложение Android;
> * создание источника данных и загрузка данных в GeoJSON-файл из локального файла или из Интернета;
> * отображение данных на карте.

## <a name="prerequisites"></a>Предварительные требования

1. Выполните инструкции из [краткого руководства по созданию приложения Android](quick-android-map.md). Здесь мы расширим код, используемый в том кратком руководстве.
2. Скачайте [пример GeoJSON-файла с точками интереса](https://raw.githubusercontent.com/Azure-Samples/AzureMapsCodeSamples/master/AzureMapsCodeSamples/Common/data/geojson/SamplePoiDataSet.json).

### <a name="import-geojson-data-from-web-or-assets-folder"></a>Импорт данных GeoJSON из Интернета или из папки assets

В большинстве файлов GeoJSON все данные включены в объект `FeatureCollection`. С учетом этого, если файлы GeoJSON загружаются в приложение в виде строки, их можно передать в статический метод `fromJson` коллекции функций, который десериализует строку в GeoJSON-объект `FeatureCollection`. Этот объект можно затем добавить в сопоставление.

Ниже показано, как импортировать GeoJSON-файл в приложение и десериализовать его в GeoJSON-объект `FeatureCollection`.

1. Выполните инструкции из [краткого руководства по созданию приложения Android](quick-android-map.md), так как последующие шаги будут выполняться с этим приложением.
2. На панели проекта в Android Studio щелкните правой кнопкой мыши папку **app** и выберите `New > Folder > Assets Folder`.
3. Перетащите [пример GeoJSON-файла с точками интереса](https://raw.githubusercontent.com/Azure-Samples/AzureMapsCodeSamples/master/AzureMapsCodeSamples/Common/data/geojson/SamplePoiDataSet.json) в папку assets.

::: zone pivot="programming-language-java-android"

4. Создайте файл с именем **Utils.java** и добавьте в него приведенный ниже код. В этом коде предоставляется статический метод `importData`, который асинхронно импортирует файл из папки `assets` приложения или из Интернета, используя URL-адрес в качестве строки, и возвращает файл обратно в поток пользовательского интерфейса с помощью простого метода обратного вызова.

    ```java
    //Modify the package name as needed to align with your application.
    package com.example.myapplication;
    
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

5. Откройте файл **MainActivity.java** и добавьте приведенный ниже код в обратный вызов для события `mapControl.onReady`, которое находится внутри метода `onCreate`. Этот код использует служебную программу импорта для чтения файла **SamplePoiDataSet.json** в виде строки, а затем десериализует файл в коллекцию функций с помощью статического метода `fromJson` класса `FeatureCollection`. Этот код также вычисляет область ограничивающего прямоугольника для всех данных в коллекции функций и использует его для настройки фокуса камеры карты на основе данных.

    ```java
    //Create a data source and add it to the map.
    DataSource source = new DataSource();
    map.sources.add(source);
    
    //Import the GeoJSON data and add it to the data source.
    Utils.importData("SamplePoiDataSet.json",
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

                //Padding added to account for pixel size of rendered points.
                padding(20)
            );
        });
    ```

6. Используя код для загрузки данных GeoJSON из источника данных, нам теперь необходимо указать, как эти данные должны отображаться на карте. Существует несколько слоев отрисовки для данных точек. Наиболее часто используются [слой пузырьков](map-add-bubble-layer-android.md), [слой символов](how-to-add-symbol-to-android-map.md) и [слой тепловой карты](map-add-heat-map-layer-android.md). Чтобы данные отображались на слое пузырьков, добавьте приведенный ниже код в обратный вызов для события `mapControl.onReady` после кода для импорта данных.

    ```java
    //Create a layer and add it to the map.
    BubbleLayer layer = new BubbleLayer(source);
    map.layers.add(layer);
    ```

::: zone-end

::: zone pivot="programming-language-kotlin"

4. Создайте файл с именем **Utils.kt** и добавьте в него приведенный ниже код. В этом коде предоставляется статический метод `importData`, который асинхронно импортирует файл из папки `assets` приложения или из Интернета, используя URL-адрес в качестве строки, и возвращает файл обратно в поток пользовательского интерфейса с помощью простого метода обратного вызова.

    ```kotlin
    //Modify the package name as needed to align with your application.
    package com.example.myapplication;

    import android.content.Context
    import android.os.Handler
    import android.os.Looper
    import android.webkit.URLUtil
    import java.net.URL
    import java.util.concurrent.ExecutorService
    import java.util.concurrent.Executors
    
    class Utils {
        companion object {
    
            /**
             * Imports data from a web url or asset file name and returns it to a callback.
             * @param urlOrFileName A web url or asset file name that points to data to load.
             * @param context The context of the app.
             * @param callback The callback function to return the data to.
             */
            fun importData(urlOrFileName: String?, context: Context, callback: (String?) -> Unit) {
                importData(urlOrFileName, context, callback, null)
            }
    
            /**
             * Imports data from a web url or asset file name and returns it to a callback.
             * @param urlOrFileName A web url or asset file name that points to data to load.
             * @param context The context of the app.
             * @param callback The callback function to return the data to.
             * @param error A callback function to return errors to.
             */
            public fun importData(urlOrFileName: String?, context: Context, callback: (String?) -> Unit, error: ((String?) -> Unit)?) {
                if (urlOrFileName != null && callback != null) {
                    val executor: ExecutorService = Executors.newSingleThreadExecutor()
                    val handler = Handler(Looper.getMainLooper())
                    executor.execute {
                        var data: String? = null
                        
                        try {
                            data = if (URLUtil.isNetworkUrl(urlOrFileName)) {
                                URL(urlOrFileName).readText()
                            } else { //Assume file is in assets folder.
                                context.assets.open(urlOrFileName).bufferedReader().use{
                                    it.readText()
                                }
                            }
    
                            handler.post {
                                //Ensure the resulting data string is not null or empty.
                                if (data != null && !data.isEmpty()) {
                                    callback(data)
                                } else {
                                    error!!("No data imported.")
                                }
                            }
                        } catch (e: Exception) {
                            error!!(e.message)
                        }
                    }
                }
            }
        }
    }
    ```

5. Откройте файл **MainActivity.kt** и добавьте приведенный ниже код в обратный вызов для события `mapControl.onReady`, которое находится внутри метода `onCreate`. Этот код использует служебную программу импорта для чтения файла **SamplePoiDataSet.json** в виде строки, а затем десериализует файл в коллекцию функций с помощью статического метода `fromJson` класса `FeatureCollection`. Этот код также вычисляет область ограничивающего прямоугольника для всех данных в коллекции функций и использует его для настройки фокуса камеры карты на основе данных.

    ```kotlin
    //Create a data source and add it to the map.
    DataSource source = new DataSource();
    map.sources.add(source);
    
    //Import the GeoJSON data and add it to the data source.
    Utils.importData("SamplePoiDataSet.json", this) { 
        result: String? ->
            //Parse the data as a GeoJSON Feature Collection.
             val fc = FeatureCollection.fromJson(result!!)
    
            //Add the feature collection to the data source.
            source.add(fc)
    
            //Optionally, update the maps camera to focus in on the data.
    
            //Calculate the bounding box of all the data in the Feature Collection.
            val bbox = MapMath.fromData(fc);

            //Update the maps camera so it is focused on the data.
            map.setCamera(
                bounds(bbox),

                //Padding added to account for pixel size of rendered points.
                padding(20)
            )
        }
    ```

6. Используя код для загрузки данных GeoJSON из источника данных, нам теперь необходимо указать, как эти данные должны отображаться на карте. Существует несколько слоев отрисовки для данных точек. Наиболее часто используются [слой пузырьков](map-add-bubble-layer-android.md), [слой символов](how-to-add-symbol-to-android-map.md) и [слой тепловой карты](map-add-heat-map-layer-android.md). Чтобы данные отображались на слое пузырьков, добавьте приведенный ниже код в обратный вызов для события `mapControl.onReady` после кода для импорта данных.

    ```kotlin
    //Create a layer and add it to the map.
    val layer = new BubbleLayer(source)
    map.layers.add(layer)
    ```

::: zone-end

7. Запустите приложение. Отобразится карта с фокусом на США, на которой показаны кружки для каждого расположения, описанного в GeoJSON-файле.

    ![Отображение карты США с данными из GeoJSON-файла](media/tutorial-load-geojson-file-android/android-import-geojson.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, используемые при работе с этим руководством, сделайте следующее:

1. Закройте Android Studio и удалите созданное приложение.
2. Если вы тестировали приложение на внешнем устройстве, удалите приложение с этого устройства.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода и сведения о возможностях интерактивного программирования см. здесь:

> [!div class="nextstepaction"]
> [Использование стилистических выражений на основе данных](data-driven-style-expressions-android-sdk.md)

> [!div class="nextstepaction"]
> [Отображение сведений о компоненте](display-feature-information-android.md)

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](android-map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](how-to-add-shapes-to-android-map.md)

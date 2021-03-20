---
title: Чтение и запись пространственных данных | Карты Microsoft Azure
description: Узнайте, как считывать и записывать данные с помощью модуля пространственных операций ввода-вывода, предоставляемого Azure Maps Web SDK.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/01/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 133674e6764e12742f5b238946e943d9b5011cd2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92891335"
---
# <a name="read-and-write-spatial-data"></a>Чтение и запись пространственных данных

В таблице ниже перечислены форматы пространственных файлов, которые поддерживаются для операций чтения и записи с помощью модуля пространственного ввода-вывода.

| Формат данных       | Чтение | запись |
|-------------------|------|-------|
| GeoJSON           | ✓  |  ✓  |
| жеорсс            | ✓  |  ✓  |
| GML               | ✓  |  ✓  |
| GPX)               | ✓  |  ✓  |
| кмл               | ✓  |  ✓  |
| кмз               | ✓  |  ✓  |
| Пространственный CSV       | ✓  |  ✓  |
| Well-Known текст   | ✓  |  ✓  |

В следующих разделах описаны различные средства чтения и записи пространственных данных с помощью модуля пространственного ввода-вывода.

## <a name="read-spatial-data"></a>Чтение пространственных данных

`atlas.io.read`Функция является основной функцией, используемой для чтения стандартных форматов пространственных данных, таких как КМЛ, GPX), жеорсс, геоjson и CSV-файлы с пространственными данными. Эта функция также может считывать сжатые версии этих форматов в виде ZIP-файла или файла КМЗ. Формат файла КМЗ — это сжатая версия КМЛ, которая также может включать ресурсы, например изображения. Кроме того, функция Read может принимать URL-адрес, указывающий на файл в любом из этих форматов. URL-адреса должны размещаться на конечной точке с поддержкой CORS, или в параметрах чтения должна быть указана прокси-служба. Прокси-служба используется для загрузки ресурсов в доменах, которые не включены в CORS. Функция Read возвращает обещание для добавления значков изображения на карту и асинхронно обрабатывает данные, чтобы снизить воздействие на поток пользовательского интерфейса.

При чтении сжатого файла в формате ZIP или КМЗ он будет распакован и проверен на наличие первого допустимого файла. Например, doc. КМЛ или файл с другим допустимым расширением, например:. КМЛ,. XML, геоjson,. JSON,. csv,. tsv или. txt. Затем образы, указанные в файлах КМЛ и Жеорсс, предварительно загружаются, чтобы обеспечить их доступность. Недоступные данные изображения могут загружать альтернативное резервное изображение или будут удалены из стилей. Образы, извлеченные из файлов КМЗ, будут преобразованы в универсальные коды ресурса (URI) данных.

Результатом функции Read является `SpatialDataSet` объект. Этот объект расширяет Феатуреколлектион класс геоjson. Его можно легко передать в виде " `DataSource` как есть" для отрисовки его функций на карте. `SpatialDataSet`Содержит не только сведения о компоненте, но также включает КМЛ наложения, метрики обработки и другие сведения, как описано в следующей таблице.

| Имя свойства | Тип | Описание | 
|---------------|------|-------------|
| `bbox` | `BoundingBox` | Ограничивающий прямоугольник всех данных в наборе данных. |
| `features` | `Feature[]` | Функции геоjson в наборе данных. |
| `groundOverlays` | `(atlas.layer.ImageLayer | atlas.layers.OgcMapLayer)[]` | Массив КМЛ Граундоверлайс. |
| `icons` | &lt;Строка записи, строка&gt; | Набор URL-адресов значков. Key = имя значка, значение = URL-адрес. |
| properties | any | Сведения о свойствах, предоставляемые на уровне документа для пространственного набора данных. |
| `stats` | `SpatialDataSetStats` | Статистика по содержимому и времени обработки пространственных наборов данных. |
| `type` | `'FeatureCollection'` | Значение типа геотабличного формата JSON, доступное только для чтения. |

## <a name="examples-of-reading-spatial-data"></a>Примеры чтения пространственных данных

В следующем коде показано, как прочитать пространственный набор данных и отобразить его на карте с помощью `SimpleDataLayer` класса. В коде используется файл GPX), на который указывает URL-адрес.

<br/>

<iframe height='500' scrolling='no' title='Простая загрузка пространственных данных' src='//codepen.io/azuremaps/embed/yLNXrZx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Дополнительные сведения о <a href='https://codepen.io/azuremaps/pen/yLNXrZx/'>загрузке пространственных данных</a> с помощью пера см <a href='https://codepen.io/azuremaps'>@azuremaps</a> . Azure Maps () в <a href='https://codepen.io'>CodePen</a>.
</iframe>

В следующей демонстрации кода показано, как считывать и загружать КМЛ или КМЗ на карту. КМЛ может содержать наложения заземления, которые будут иметь форму `ImageLyaer` или `OgcMapLayer` . Эти наложения должны быть добавлены на карту отдельно от функций. Кроме того, если в наборе данных есть настраиваемые значки, эти значки должны быть загружены в ресурсы карты до загрузки компонентов.

<br/>

<iframe height='500' scrolling='no' title='Загрузить КМЛ на карту' src='//codepen.io/azuremaps/embed/XWbgwxX/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. <a href='https://codepen.io/azuremaps/pen/XWbgwxX/'>КМЛ нагрузки на карту</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

При необходимости вы можете предоставить прокси-службу для доступа к междоменным ресурсам, которые могут не включать CORS. Функция Read сначала попытается получить доступ к файлам в другом домене, используя CORS. После первого сбоя доступа к любому ресурсу в другом домене с помощью CORS он запрашивает дополнительные файлы, только если указана прокси-служба. Функция Read добавляет URL-адрес файла в конец указанного URL-адреса прокси. В этом фрагменте кода показано, как передать прокси-службу в функцию Read:

```javascript
//Read a file from a URL or pass in a raw data as a string.
atlas.io.read('https://nonCorsDomain.example.com/mySuperCoolData.xml', {
    //Provide a proxy service
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
}).then(async r => {
    if (r) {
        // Some code goes here . . .
    }
});

```

В приведенной ниже демонстрации показано, как прочитать файл с разделителями и отобразить его на карте. В этом случае в коде используется CSV-файл со столбцами пространственных данных.

<br/>

<iframe height='500' scrolling='no' title='Добавление файла с разделителями' src='//codepen.io/azuremaps/embed/ExjXBEb/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Ознакомьтесь с пером <a href='https://codepen.io/azuremaps/pen/ExjXBEb/'>Добавление файла с разделителями</a> Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="write-spatial-data"></a>Запись пространственных данных

В модуле пространственного ввода-вывода есть две основные функции записи. `atlas.io.write`Функция создает строку, а `atlas.io.writeCompressed` функция создает сжатый ZIP-файл. Сжатый ZIP-файл будет содержать текстовый файл с пространственными данными. Обе эти функции возвращают обещание для добавления данных в файл. И они могут записывать любые из следующих данных: `SpatialDataSet` , `DataSource` , `ImageLayer` , `OgcMapLayer` , коллекцию функций, функцию, геометрию или массив любого сочетания этих типов данных. При записи с помощью любой из этих функций можно указать требуемый формат файла. Если формат файла не указан, данные будут записаны как КМЛ.

В приведенном ниже средстве показаны большинство параметров записи, которые можно использовать с `atlas.io.write` функцией.

<br/>

<iframe height='700' scrolling='no' title='Параметры записи пространственных данных' src='//codepen.io/azuremaps/embed/YzXxXPG/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. раздел <a href='https://codepen.io/azuremaps/pen/YzXxXPG/'>Параметры записи пространственных данных</a> пера с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="example-of-writing-spatial-data"></a>Пример записи пространственных данных

Следующий пример позволяет перетаскивать и загружать пространственные файлы на карте. Данные геоjson можно экспортировать из схемы и записывать в один из поддерживаемых форматов пространственных данных в виде строки или в виде сжатого файла.

<br/>

<iframe height='700' scrolling='no' title='Перетаскивание пространственных файлов на карту' src='//codepen.io/azuremaps/embed/zYGdGoO/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. раздел <a href='https://codepen.io/azuremaps/pen/zYGdGoO/'>перетаскивание пространственных файлов на карте</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

При необходимости вы можете предоставить прокси-службу для доступа к междоменным ресурсам, которые могут не включать CORS. В этом фрагменте кода показано, как можно включить прокси-службу:

```javascript
atlas.io.read(data, {
    //Provide a proxy service
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
}).then(
    //Success
    function(r) {
        //some code goes here ...
    }
);
```

## <a name="read-and-write-well-known-text-wkt"></a>Чтение и запись Well-Known текста (WKT)

[Хорошо известный текст](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) (WKT) — это стандартный открытый ГЕОПРОСТРАНСТВЕННЫЙ консорциум (OGC), представляющий пространственные геометрические объекты в виде текста. Многие геопространственные системы поддерживают WKT, такие как Azure SQL и Azure PostgreSQL, с помощью подключаемого модуля PostGIS. Как и большинство стандартов OGC, координаты форматируются как "Долгота широты" для согласования с соглашением "x y". Например, точка со долготой-110 и Широта 45 может быть написана `POINT(-110 45)` с использованием формата WKT.

Хорошо известный текст можно считать с помощью `atlas.io.ogc.WKT.read` функции и написать с помощью `atlas.io.ogc.WKT.write` функции.

## <a name="examples-of-reading-and-writing-well-known-text-wkt"></a>Примеры чтения и записи Well-Known текста (WKT)

В следующем коде показано, как прочитать хорошо известную текстовую строку `POINT(-122.34009 47.60995)` и отобразить ее на карте с помощью пузырькового слоя.

<br/>

<iframe height='500' scrolling='no' title='Чтение Well-Known текста' src='//codepen.io/azuremaps/embed/XWbabLd/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. раздел <a href='https://codepen.io/azuremaps/pen/XWbabLd/'>чтение Well-Known текста</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

Следующий код демонстрирует чтение и запись хорошо известного текста.

<br/>

<iframe height='700' scrolling='no' title='Чтение и запись Well-Known текста' src='//codepen.io/azuremaps/embed/JjdyYav/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'><a href='https://codepen.io/azuremaps/pen/JjdyYav/'>Прочтите текст Well-Known чтения и записи</a> с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="read-and-write-gml"></a>Чтение и запись GML

GML — это спецификация пространственных XML-файлов, которая часто используется в качестве расширения для других спецификаций XML. Данные геоjson можно записать в виде XML-кода с помощью тегов GML, используя `atlas.io.core.GmlWriter.write` функцию. XML-код, содержащий GML, можно считать с помощью `atlas.io.core.GmlReader.read` функции. Функция Read имеет два варианта:

- `isAxisOrderLonLat`Параметр. порядок осей координат "Широта, Долгота" или "Долгота" может различаться между наборами данных, и он не всегда определен правильно. По умолчанию средство чтения GML считывает данные координаты как "Широта, Долгота", но при установке этого параметра в значение "true" будет считаться "Долгота, Широта".
- `propertyTypes`Параметр — это таблица уточняющих значений ключа, где ключ — это имя свойства в наборе данных. Значение — это тип объекта для приведения значения при синтаксическом анализе. Поддерживаются следующие значения типа: `string` , `number` , `boolean` и  `date` . Если свойство отсутствует в таблице подстановки или тип не определен, свойство будет проанализировано как строка.

`atlas.io.read`Функция будет по умолчанию выполнять `atlas.io.core.GmlReader.read` функцию, когда обнаруживает, что входные данные являются XML, но данные не являются одной из других поддерживаемых ПРОСТРАНСТВЕННЫХ форматов XML.

Объект `GmlReader` будет анализировать координаты, имеющие один из следующих SRID:

- EPSG: 4326 (предпочтительно)
- EPSG: 4269, EPSG: 4283, EPSG: 4258, EPSG: 4308, EPSG: 4230, EPSG: 4272, EPSG: 4271, EPSG: 4267, EPSG: 4608, EPSG: 4674, возможно, с небольшим полем ошибки.
- EPSG: 3857:, EPSG: 102100, EPSG: 3785, EPSG: 900913, EPSG: 102113, EPSG: 41001, EPSG: 54004

## <a name="more-resources"></a>Дополнительные ресурсы

Дополнительные сведения о классах и методах, которые используются в этой статье:

[статические функции atlas.io](/javascript/api/azure-maps-spatial-io/atlas.io)

[спатиалдатасет](/javascript/api/azure-maps-spatial-io/atlas.spatialdataset)

[спатиалдатасетстатс](/javascript/api/azure-maps-spatial-io/atlas.spatialdatasetstats)

[гмлреадер](/javascript/api/azure-maps-spatial-io/atlas.io.core.gmlreader)

[гмлвритер](/javascript/api/azure-maps-spatial-io/atlas.io.core.gmlwriter)

[функции Atlas. IO. OGC. WKT](/javascript/api/azure-maps-spatial-io/atlas.io.ogc.wkt)

[Подключение службе WFS](spatial-io-connect-wfs-service.md)

[Использование основных операций](spatial-io-core-operations.md)

[Сведения о поддерживаемых форматах данных](spatial-io-supported-data-format-details.md)


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

[Добавление слоя карты OGC](spatial-io-add-ogc-map-layer.md)
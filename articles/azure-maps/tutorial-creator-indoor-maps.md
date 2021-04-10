---
title: Руководство по Использование Создателя Microsoft Azure Maps (предварительная версия) для создания схем помещений
description: Учебник по использованию Создателя Microsoft Azure Maps (предварительная версия) для создания схем помещений.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 9ac53dab29feddd36a95b8b2b041caaf5c3598d5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101720143"
---
# <a name="tutorial-use-creator-preview-to-create-indoor-maps"></a>Руководство по Использование Создателя (предварительная версия) для создания схем помещений

> [!IMPORTANT]
> Службы Создателя Azure Maps в настоящее время предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этом учебнике показано, как создавать схемы помещений. Из этого учебника вы узнаете, как использовать API, чтобы выполнять следующие задачи:

> [!div class="checklist"]
> * Отправка пакета рисунков со схемой помещений
> * Преобразование пакета рисунков в данные схемы
> * Создание набора данных на основе данных схемы
> * Создание набора плиток из данных в наборе данных
> * Запрос API Azure Maps Web Feature Service (WFS), чтобы узнать о функциях схемы
> * Создание набора состояний функции с помощью функций схемы и данных в наборе данных
> * Обновление набора состояний вашей функции

## <a name="prerequisites"></a>Предварительные требования

Создание схем помещений:

1. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки.
3. [Создание ресурса Создателя (предварительная версия)](how-to-manage-creator.md)
4. Скачайте [пример пакета рисунков](https://github.com/Azure-Samples/am-creator-indoor-data-examples/blob/master/Sample%20-%20Contoso%20Drawing%20Package.zip).

В этом руководстве используется приложение [Postman](https://www.postman.com/), но вы можете выбрать другую среду разработки API.

>[!IMPORTANT]
> URL-адреса API в этом документе могут быть скорректированы в соответствии с расположением ресурса Создателя. Дополнительные сведения см. в разделе [Получение доступа к службам Создателя](how-to-manage-creator.md#access-to-creator-services).

## <a name="upload-a-drawing-package"></a>Отправка пакета рисунков

Используйте [API отправки данных](/rest/api/maps/data/uploadpreview), чтобы передать пакет рисунков в ресурсы Azure Maps.

API отправки данных — это долго выполняющаяся транзакция, которая реализует шаблон, определенный здесь. После завершения операции мы будем использовать `udid` для доступа к переданному пакету для его преобразования. Чтобы получить `udid`, выполните следующие действия.

1. Откройте приложение Postman. В верхней части приложения Postman выберите элемент **Создать**. В окне **Create New** (Создание) выберите **Collection** (Коллекция).  Присвойте имя коллекции и нажмите кнопку **Создать**.

2. Чтобы создать запрос, нажмите кнопку **Создать** еще раз. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **Имя запроса** для запроса. Выберите коллекцию, созданную на предыдущем шаге, а затем нажмите кнопку **Сохранить**.

3. Выберите HTTP-метод **POST** на вкладке "Построитель" и введите следующий URL-адрес, чтобы отправить пакет рисунков в службу Azure Maps. Для этого запроса и других запросов, упомянутых в этой статье, замените `{Azure-Maps-Primary-Subscription-key}` первичным ключом подписки.

    ```http
    https://atlas.microsoft.com/mapData/upload?api-version=1.0&dataFormat=zip&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

4. На вкладке **Заголовки** укажите значение для ключа `Content-Type`. Пакет рисунков — это папка в ZIP-архиве, поэтому используйте значение `application/octet-stream`. На вкладке **Текст** выберите **двоичный**. Щелкните элемент **Выбрать файл** и выберите пакет рисунков.

     ![управление данными](./media/tutorial-creator-indoor-maps/enter-content-type-dialog.png)

5. Нажмите синюю кнопку **Отправить** и дождитесь обработки запроса. После завершения запроса перейдите на вкладку **Заголовки** ответа. Скопируйте значение ключа **Расположение**, который является `status URL`.

6. Чтобы проверить состояние вызова API, создайте HTTP-запрос **GET** по `status URL`. Вам потребуется добавить первичный ключ подписки к URL-адресу для проверки подлинности. Запрос **GET** должен выглядеть, как в этом URL-адресе:

    ```http
    https://atlas.microsoft.com/mapData/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

7. Когда HTTP-запрос **GET** завершается успешно, он возвращает `resourceLocation`. `resourceLocation` содержит уникальное значение `udid` для переданного содержимого. Также вы можете использовать URL-адрес `resourceLocation`, чтобы получить метаданные из этого ресурса на следующем шаге.

    ```json
    {
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0"
    }
    ```

8. Чтобы получить метаданные содержимого, создайте HTTP-запрос **GET** по URL-адресу `resourceLocation`, который вы получили на шаге 7. Не забудьте добавить первичный ключ подписки к этому URL-адресу для проверки подлинности. Запрос **GET** должен быть похож на следующий URL-адрес.

    ```http
   https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

9. После успешного завершения HTTP-запрос **GET** возвращает ответ, текст которого содержит значение `udid`, определенное в `resourceLocation` на шаге 7, а также расположение для последующего доступа к содержимому (скачивания) и набор других метаданных для этого содержимого (даты создания и обновления, размер и т. д.). Пример общего ответа:

    ```json
    {
        "udid": "{udid}",
        "location": "https://atlas.microsoft.com/mapData/{udid}?api-version=1.0",
        "created": "2020-02-03T02:32:25.0509366+00:00",
        "updated": "2020-02-11T06:12:13.0309351+00:00",
        "sizeInBytes": 766,
        "uploadStatus": "Completed"
    }
    ```

## <a name="convert-a-drawing-package"></a>Преобразование пакета рисунков

 После отправки пакета рисунков мы применим к нему `udid`, чтобы преобразовать этот пакет в данные схемы. API-интерфейс преобразования использует длительную транзакцию, которая реализует шаблон, определенный [здесь](creator-long-running-operation.md). После завершения операции мы будем использовать `conversionId` для доступа к преобразованным данным. Чтобы получить `conversionId`, выполните следующие действия.

1. Нажмите кнопку **Создать**. В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **имя запроса** и выберите коллекцию. Выберите команду **Сохранить**.

2. Выберите HTTP-метод **POST** на вкладке "Построитель" и введите следующий URL-адрес для преобразования отправленного пакета рисунков в данные схемы. Используйте `udid` для отправленного пакета.

    ```http
    https://atlas.microsoft.com/conversion/convert?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&udid={udid}&inputType=DWG
    ```

    >[!IMPORTANT]
    > URL-адреса API в этом документе могут быть скорректированы в соответствии с расположением ресурса Создателя. Дополнительные сведения см. в статье [Доступ к службам Создателя (предварительная версия) ](how-to-manage-creator.md#access-to-creator-services).
    > Если вы получаете сообщение об ошибке с кодом `"RequiresCreatorResource"`, убедитесь, что вы [подготовили ресурс Creator в Azure Maps](how-to-manage-creator.md) в учетной записи Azure Maps.

3. Нажмите кнопку **Отправить** и дождитесь обработки запроса. После завершения запроса перейдите на вкладку **Заголовки** ответа и найдите ключ **Расположение**. Скопируйте значение ключа **Расположение**, который является `status URL` для запроса на преобразование. Оно понадобится вам на следующем шаге.

    :::image type="content" source="./media/tutorial-creator-indoor-maps/copy-location-uri-dialog.png" border="true" alt-text="Скопируйте значение ключа расположения":::

4. Запустите новый HTTP-метод **GET** на вкладке "Построитель". Добавьте первичный ключ подписки Azure Maps в `status URL`. Выполните запрос **GET** с использованием значения `status URL`, которое вы скопировали на шаге 3. `status URL` выглядит, как следующий URL-адрес:

    ```http
    https://atlas.microsoft.com/conversion/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

    Если процесс преобразования еще не завершен, вы можете увидеть нечто вроде следующего ответа JSON:

    ```json
    {
        "operationId": "<operationId>",
        "created": "2020-04-22T19:39:54.9518496+00:00",
        "status": "Running"
    }
    ```

5. После успешного выполнения запроса в тексте ответа отобразится успешное сообщение о состоянии.  Скопируйте `conversionId` из URL-адреса `resourceLocation` преобразованного пакета. `conversionId` используется другим API для доступа к данным преобразованной схемы.

    ```json
   {
        "operationId": "<operationId>",
        "created": "2020-04-22T19:39:54.9518496+00:00",
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/conversion/{conversionId}?api-version=1.0",
        "properties": {}
    }
    ```

>[!NOTE]
>Приложение Postman изначально не поддерживает длительные запросы HTTP. В результате вы можете заметить долгую задержку при выполнении запроса **GET** по URL-адресу состояния.  Выждите примерно тридцать секунд и снова нажмите кнопку **Отправить**, пока не появится сообщение об успешном или неудачном выполнении.

Образец пакета рисунков должен быть преобразован без ошибок или предупреждений. Однако если для собственного пакета рисунков вы получаете ошибки или предупреждения, ответ JSON вернет ссылку на [визуализатор ошибок рисунков](drawing-error-visualizer.md). Визуализатор ошибок рисунков позволяет просматривать сведения об ошибках и предупреждениях. Чтобы получить рекомендации по устранению ошибок и предупреждений при преобразовании, см. сведения в разделе [Ошибки и предупреждения, связанные с преобразованием рисунков](drawing-conversion-error-codes.md).

```json
{
    "operationId": "<operationId>",
    "created": "2020-04-22T19:39:54.9518496+00:00",
    "status": "Failed",
    "resourceLocation": "https://atlas.microsoft.com/conversion/{conversionId}?api-version=1.0",
    "properties": {
        "diagnosticPackageLocation": "https://atlas.microsoft.com/mapData/ce61c3c1-faa8-75b7-349f-d863f6523748?api-version=1.0"
    }
}
```

## <a name="create-a-dataset"></a>Создание набора данных

Набор данных — это коллекция функций схем, таких как здания, этажи и помещения. Чтобы создать набор данных, используйте [API создания набора данных](/rest/api/maps/dataset/createpreview). API создания набора данных принимает `conversionId` для преобразованного пакета рисунков и возвращает `datasetId` созданного набора данных. Ниже показано, как создать набор данных.

1. В приложении Postman выберите **New** (Создать). В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **имя запроса** и выберите коллекцию. Щелкните **Сохранить**.

2. Выполните запрос **POST** к [API создания набора данных](/rest/api/maps/dataset/createpreview), чтобы создать новый набор данных. Перед отправкой запроса добавьте ключ подписки и `conversionId` с `conversionId`, полученным в процессе преобразования на шаге 5.  Запрос должен выглядеть, как в этом URL-адресе:

    ```http
    https://atlas.microsoft.com/dataset/create?api-version=1.0&conversionID={conversionId}&type=facility&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Получите `statusURL` из ключа **Расположение** ответа **Заголовки**.

4. Чтобы получить `datasetId`, выполните запрос на **GET** к `statusURL`. Добавьте первичный ключ подписки Azure Maps для проверки подлинности. Запрос должен выглядеть, как в этом URL-адресе:

    ```http
    https://atlas.microsoft.com/dataset/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

5. После успешного завершения HTTP-запроса **GET** заголовок ответа будет содержать `datasetId` для созданного набора данных. Скопируйте `datasetId`. Чтобы создать набор фрагментов, необходимо использовать `datasetId`.

    ```json
    {
        "operationId": "<operationId>",
        "created": "2020-04-22T19:52:38.9352189+00:00",
        "status": "Succeeded",
        "resourceLocation": "https://azure.microsoft.com/dataset/{datasetiId}?api-version=1.0"
     }
    ```

## <a name="create-a-tileset"></a>Создание набора фрагментов

Набор фрагментов — это ряд векторных плиток, которые отображаются на схеме. Наборы фрагментов создаются на основе существующих наборов данных. Однако набор фрагментов не зависит от набора данных, который послужил его источником. Если набор данных удален, набор фрагментов по-прежнему будет существовать. Чтобы создать набор фрагментов, выполните следующие действия:

1. В приложении Postman выберите **New** (Создать). В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **имя запроса** и выберите коллекцию. Щелкните **Сохранить**.

2. Выполните запрос **POST** на вкладке "Построитель". URL-адрес запроса должен выглядеть как этот URL-адрес:

    ```http
    https://atlas.microsoft.com/tileset/create/vector?api-version=1.0&datasetID={datasetId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Выполните запрос **GET** в `statusURL` для этого набора фрагментов. Добавьте первичный ключ подписки Azure Maps для проверки подлинности. Запрос должен выглядеть, как в этом URL-адресе:

   ```http
    https://atlas.microsoft.com/tileset/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

4. После успешного завершения HTTP-запроса **GET** заголовок ответа будет содержать `tilesetId` для созданного набора фрагментов. Скопируйте `tilesetId`.

    ```json
    {
        "operationId": "<operationId>",
        "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/tileset/{tilesetId}?api-version=1.0"
    }
    ```

## <a name="query-datasets-with-wfs-api"></a>Запросы к наборам данных с помощью API WFS

 Наборы данных можно запрашивать с помощью [API WFS](/rest/api/maps/wfs). С помощью API WFS можно запросить коллекции функций, определенную коллекцию или функцию с **идентификатором** функции. **Идентификатор** функции однозначно определяет функцию в наборе данных. Он используется, например, для указания, какое состояние функции следует обновить в заданном наборе состояний.

1. В приложении Postman выберите **New** (Создать). В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **имя запроса** и выберите коллекцию. Щелкните **Сохранить**.

2. Выполните запрос **GET**, чтобы просмотреть список коллекций в наборе данных. Замените `<dataset-id>` на `datasetId`. Используйте первичный ключ Azure Maps вместо заполнителя. Запрос должен выглядеть, как в этом URL-адресе:

    ```http
    https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

3. Текст ответа будет доставляться в формате GeoJSON и будет содержать все коллекции в этом наборе данных. Для простоты в этом примере показана только коллекция `unit`. Пример, содержащий все коллекции, см. в разделе [API коллекций описания WFS](/rest/api/maps/wfs/collectiondescriptionpreview). Чтобы узнать больше о любой коллекции, можно щелкнуть любой URL-адрес в элементе `link`.

    ```json
    {
    "collections": [
        {
            "name": "unit",
            "description": "A physical and non-overlapping area which might be occupied and traversed by a navigating agent. Can be a hallway, a room, a courtyard, etc. It is surrounded by physical obstruction (wall), unless the is_open_area attribute is equal to true, and one must add openings where the obstruction shouldn't be there. If is_open_area attribute is equal to true, all the sides are assumed open to the surroundings and walls are to be added where needed. Walls for open areas are represented as a line_element or area_element with is_obstruction equal to true.",
            "links": [
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/definition?api-version=1.0",
                    "rel": "describedBy",
                    "title": "Metadata catalogue for unit"
                },
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/items?api-version=1.0",
                    "rel": "data",
                    "title": "unit"
                }
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit?api-version=1.0",
                    "rel": "self",
                    "title": "Metadata catalogue for unit"
                }
            ]
        },
    ```

4. Выполните запрос **GET** для коллекций функций `unit`.  Замените `{datasetId}` на `datasetId`. Используйте первичный ключ Azure Maps вместо заполнителя. Текст ответа будет содержать все функции коллекции `unit`. Запрос должен выглядеть, как в этом URL-адресе:

    ```http
    https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/items?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

5. Скопируйте `id` функции для функции секции, имеющей свойства стиля, которые можно изменять динамически.  Так как состояние заполнения секции и температуру можно обновлять динамически, мы будем использовать `id` этой функции в следующем разделе. В следующем примере `id` функции — "UNIT26". Мы будем называть свойства стиля этой функции состояниями, и будем использовать эту функцию для создания набора состояний.

     ```json
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Polygon",
                    "coordinates": ["..."]
                },
                "properties": {
                    "original_id": "b7410920-8cb0-490b-ab23-b489fd35aed0",
                    "category_id": "CTG8",
                    "is_open_area": true,
                    "navigable_by": [
                        "pedestrian"
                    ],
                    "route_through_behavior": "allowed",
                    "level_id": "LVL14",
                    "occupants": [],
                    "address_id": "DIR1",
                    "name": "157"
                },
                "id": "UNIT26",
                "featureType": ""
            }, {"..."}
        ]
    }
    ```

## <a name="create-a-feature-stateset"></a>Создание набора состояний функции

1. В приложении Postman выберите **New** (Создать). В окне **Create New** (Создание) выберите **Request** (Запрос). Введите **имя запроса** и выберите коллекцию. Щелкните **Сохранить**.

2. Выполните запрос **POST** к [API создания набора состояний](/rest/api/maps/featurestate/createstatesetpreview). Используйте `datasetId` набора данных, содержащего состояние, которое необходимо изменить. Запрос должен выглядеть, как в этом URL-адресе:

    ```http
    https://atlas.microsoft.com/featureState/stateset?api-version=1.0&datasetId={datasetId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. В **Заголовках** запроса **POST** установите для параметра `Content-Type` значение `application/json`. В поле **Текст** укажите следующие стили необработанного текста JSON, чтобы отразить изменения в *состояниях* `occupied` и `temperature`. Закончив, нажмите кнопку **Отправить**.

    ```json
    {
       "styles":[
          {
             "keyname":"occupied",
             "type":"boolean",
             "rules":[
                {
                   "true":"#FF0000",
                   "false":"#00FF00"
                }
             ]
          },
          {
             "keyname":"temperature",
             "type":"number",
             "rules":[
                {
                   "range":{
                      "exclusiveMaximum":66
                   },
                   "color":"#00204e"
                },
                {
                   "range":{
                      "minimum":66,
                      "exclusiveMaximum":70
                   },
                   "color":"#0278da"
                },
                {
                   "range":{
                      "minimum":70,
                      "exclusiveMaximum":74
                   },
                   "color":"#187d1d"
                },
                {
                   "range":{
                      "minimum":74,
                      "exclusiveMaximum":78
                   },
                   "color":"#fef200"
                },
                {
                   "range":{
                      "minimum":78,
                      "exclusiveMaximum":82
                   },
                   "color":"#fe8c01"
                },
                {
                   "range":{
                      "minimum":82
                   },
                   "color":"#e71123"
                }
             ]
          }
       ]
    }
    ```

4. Скопируйте `statesetId` из текста ответа.

5. Создайте запрос **POST** для обновления состояния: Передайте идентификатор statesetId и `ID` функции с помощью ключа подписки Azure Maps. Запрос должен выглядеть, как в этом URL-адресе:

    ```http
    https://atlas.microsoft.com/featureState/state?api-version=1.0&statesetID={statesetId}&featureID={featureId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

6. В **Заголовках** запроса **POST** установите для параметра `Content-Type` значение `application/json`. В поле **Текст** запроса **POST**, скопируйте и вставьте JSON в примере ниже.

    ```json
    {
        "states": [
            {
                "keyName": "occupied",
                "value": true,
                "eventTimestamp": "2019-11-14T17:10:20"
            }
        ]
    }
    ```

    >[!NOTE]
    > Обновление будет сохранено только в том случае, если отметка времени будет располагаться после метки времени предыдущего запроса. Мы можем передать любое имя ключа, настроенное ранее во время создания.

7. После успешного обновления вы получите код состояния HTTP `200 OK`. При наличии [динамического стиля, реализованного](indoor-map-dynamic-styling.md) для схемы помещений, обновление будет отображаться на преобразованной для просмотра схеме в указанной отметке времени.

[API состояний получения функций](/rest/api/maps/featurestate/getstatespreview) позволяет извлекать состояние компонента с помощью функции `ID`. Кроме того, можно удалить набор состояний и его ресурсы, используя [API удаления состояния функции](/rest/api/maps/featurestate/deletestatesetpreview).

Дополнительные сведения о различных службах Создателя Azure Maps (предварительная версия), обсуждаемых в этой статье, см. в статье [Создатель для схем помещений](creator-indoor-maps.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов, требующих очистки.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как использовать модуль Indoor Maps, см. статью

> [!div class="nextstepaction"]
> [Использование модуля схем помещений](how-to-use-indoor-module.md)
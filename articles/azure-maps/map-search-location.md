---
title: Отображение результатов поиска на карте | Microsoft Azure Maps
description: В этой статье рассказывается, как выполнить поисковый запрос с помощью веб-пакета SDK для Microsoft Azure Maps и отобразить результаты на карте.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 647a8fc25f27ef7f441ed7459ecd543d4f35581e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895279"
---
# <a name="show-search-results-on-the-map"></a>Отображение результатов поиска на карте

В этой статье объясняется, как выполнить поиск расположения и вывести результаты поиска на карту.

Существует два способа найти нужное место. Во-первых, можно использовать модуль службы для поискового запроса. Во-вторых, можно сделать поисковой запрос через [API поиска нечетких соответствий Azure Maps](/rest/api/maps/search/getsearchfuzzy) с помощью [Fetch API](https://fetch.spec.whatwg.org/). Оба способа описаны ниже.

## <a name="make-a-search-request-via-service-module"></a>Поисковой запрос через модуль службы

<iframe height='500' scrolling='no' title='Отображение результатов поиска на карте (модуль службы)' src='//codepen.io/azuremaps/embed/zLdYEB/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода для <a href='https://codepen.io/azuremaps/pen/zLdYEB/'>отображения результатов поиска на карте (служебный модуль)</a> службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

В приведенном выше коде первый блок создает объект карты и настраивает механизм проверки подлинности на использование маркера доступа. См. инструкции по [созданию карты](./map-create.md).

Второй блок кода создает `TokenCredential` для проверки подлинности HTTP-запросов к Azure Maps с маркером доступа. Затем он передает `TokenCredential` в `atlas.service.MapsURL.newPipeline()` и создает экземпляр [конвейера](/javascript/api/azure-maps-rest/atlas.service.pipeline). `searchURL` представляет собой URL-адрес к операциям [поиска](/rest/api/maps/search) службы Azure Maps.

В третьем блоке кода создается объект источника данных с помощью класса [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource), к которому добавляются результаты поиска. В [слое символов](/javascript/api/azure-maps-control/atlas.layer.symbollayer) данные на основе точек в оболочке [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource) преобразовываются для просмотра в качестве символов на карте с помощью текста или значков.  Затем создается слой символов. Источник данных добавляется в слой символов, который затем добавляется на карту.

Четвертый блок кода использует метод [SearchFuzzy](/javascript/api/azure-maps-rest/atlas.service.models.searchgetsearchfuzzyoptionalparams) в [модуле службы](how-to-use-services-module.md). Он позволяет выполнять поиск текста произвольной формы с помощью [REST API поиска нечетких соответствий](/rest/api/maps/search/getsearchfuzzy) для поиска точки интереса. Запросы GET к API поиска нечетких соответствий могут работать с любым сочетанием нечетких входных данных. Затем с помощью метода `geojson.getFeatures()` из ответа извлекается коллекция функций GeoJSON и добавляется к источнику данных, после чего данные автоматически отображаются на карте с помощью слоя символов.

Последний блок кода корректирует границы камеры для карты с помощью свойства карты [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-).

Поисковый запрос, источник данных, слой символов и границы камеры находятся внутри [прослушивателя событий](/javascript/api/azure-maps-control/atlas.map#events) карты. Нам необходимо убедиться, что результаты отображаются после полной загрузки карты.


## <a name="make-a-search-request-via-fetch-api"></a>Поиск через Fetch API

<iframe height='500' scrolling='no' title='Отображение результатов поиска на карте' src='//codepen.io/azuremaps/embed/KQbaeM/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода для <a href='https://codepen.io/azuremaps/pen/KQbaeM/'>отображения результатов поиска на карте</a> службы "Карты Azure" (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

В первом блоке приведенного выше кода создается объект карты. Он задает механизм проверки подлинности для использования маркера доступа. См. инструкции по [созданию карты](./map-create.md).

Второй блок кода создает URL-адрес для выполнения поискового запроса. Он также создает два массива для хранения границ и закреплений для результатов поиска.

В третьем блоке кода используется [Fetch API](https://fetch.spec.whatwg.org/). [Fetch API](https://fetch.spec.whatwg.org/) используется для выполнения запроса к [API поиска нечетких соответствий Azure Maps](/rest/api/maps/search/getsearchfuzzy) для поиска точек интереса. API поиска нечетких соответствий может обрабатывать любую комбинацию нечетких входных данных. Затем он обрабатывает и анализирует ответ поиска и добавляет результирующие закрепления в массив searchPins.

В четвертом блоке кода создается объект источника данных с помощью класса [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource). В коде мы добавляем результаты поиска в исходный объект. В [слое символов](/javascript/api/azure-maps-control/atlas.layer.symbollayer) данные на основе точек в оболочке [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource) преобразовываются для просмотра в качестве символов на карте с помощью текста или значков. Затем создается слой символов. Источник данных добавляется в слой символов, который затем добавляется на карту.

Последний блок кода создает объект [BoundingBox](/javascript/api/azure-maps-control/atlas.data.boundingbox). Он использует массив результатов, а затем настраивает границы камеры для карты с помощью [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) карты. Затем выполняется отрисовка результатов.

Поисковый запрос, источник данных, слой символов и границы камеры задаются в [прослушивателе событий](/javascript/api/azure-maps-control/atlas.map#events) карты. Таким образом, результаты отображаются после полной загрузки карты.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Рекомендации по использованию службы поиска Azure Maps](how-to-use-best-practices-for-search.md)

Дополнительные сведения о **поиске нечетких соответствий**:

> [!div class="nextstepaction"]
> [API поиска нечетких соответствий службы "Карты Azure"](/rest/api/maps/search/getsearchfuzzy)

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Схема](/javascript/api/azure-maps-control/atlas.map)

Полные примеры кода см. в следующих статьях:

> [!div class="nextstepaction"]
> [Получение сведений на основе координат](map-get-information-from-coordinate.md)
<!-- Comment added to suppress false positive warning -->
> [!div class="nextstepaction"]
> [Отображение направлений от точки А до точки Б](map-route.md)
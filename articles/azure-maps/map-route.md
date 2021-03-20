---
title: Отображение направлений маршрута на карте | Карты Microsoft Azure
description: В этой статье вы узнаете, как отображать направления между двумя расположениями на карте с помощью веб-пакета SDK Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: codepen
ms.openlocfilehash: 9d0197a16c8074ce961c2b403724149929f566f7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92890723"
---
# <a name="show-directions-from-a-to-b"></a>Отображение направлений от точки А до точки Б

В этой статье объясняется, как выполнить запрос маршрута и отобразить маршрут на карте.

Это можно сделать двумя способами. Во-первых — запрос к [API маршрута Azure Maps](/rest/api/maps/route/getroutedirections) через модуль службы. Второй способ — использовать [API выборки](https://fetch.spec.whatwg.org/) , чтобы выполнить поисковый запрос к [Azure Maps API маршрута](/rest/api/maps/route/getroutedirections). Оба способа описаны ниже.

## <a name="query-the-route-via-service-module"></a>Запрос маршрута с помощью модуля службы

<iframe height='500' scrolling='no' title='Отображение направлений от точки А до точки Б на карте (модуль службы)' src='//codepen.io/azuremaps/embed/RBZbep/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/RBZbep/'>Отображение направлений от точки А до точки Б на карте (модуль службы)</a> для службы Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) на сайте <a href='https://codepen.io'>CodePen</a>.
</iframe>

В приведенном выше коде первый блок конструирует объект Map и устанавливает механизм проверки подлинности для использования маркера доступа. См. инструкции по [созданию карты](./map-create.md).

Второй блок кода создает `TokenCredential` для проверки подлинности HTTP-запросов к Azure Maps с маркером доступа. Затем он передает `TokenCredential` в `atlas.service.MapsURL.newPipeline()` и создает экземпляр [конвейера](/javascript/api/azure-maps-rest/atlas.service.pipeline). `routeURL` представляет собой URL-адрес для операций [маршрута](/rest/api/maps/route) Azure Maps.

Третий блок кода создает и добавляет объект [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource) к сопоставлению.

Четвертый блок кода создает объекты начальной и конечной [точек](/javascript/api/azure-maps-control/atlas.data.point) и добавляет их в объект DataSource.

Линия — это [функция](/javascript/api/azure-maps-control/atlas.data.feature) для LineString. [LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer) отвечает за отображение линейных объектов в оболочке [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource) в качестве линий на карте. Четвертый блок кода позволяет создать и добавить на карту слой линий. Ознакомьтесь со свойствами слоя линий в разделе о классе [LinestringLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions).

[Слой символов](/javascript/api/azure-maps-control/atlas.layer.symbollayer) использует текст или значки для визуализации данных на основе точек, инкапсулированных в [источнике](/javascript/api/azure-maps-control/atlas.source.datasource)данных. Текст или значки отображаются в виде символов на карте. Пятый блок кода создает и добавляет на карту слой символов.

Шестой блок кода запрашивает службу маршрутизации Azure Maps, которая является частью [модуля службы](how-to-use-services-module.md). Метод [калкулатераутедиректионс](/javascript/api/azure-maps-rest/atlas.service.routeurl#methods) в RouteUrl используется для получения маршрута между начальной и конечной точками. После этого коллекция функций геоjson из ответа извлекается с помощью `geojson.getFeatures()` метода и добавляется в источник данных. После этого ответ отображается в виде маршрута на карте. Дополнительные сведения о добавлении линии на карту см. [здесь](map-add-line-layer.md).

Последний блок кода задает границы сопоставлений с помощью свойства [сеткамера](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) на карте.

Запрос маршрута, источник данных, символ, уровни линии и границы камеры создаются внутри [прослушивателя событий](/javascript/api/azure-maps-control/atlas.map#events). Эта структура кода гарантирует, что результаты будут отображаться только после полной загрузки схемы.

## <a name="query-the-route-via-fetch-api"></a>Отправка запроса через API FETCH

<iframe height='500' scrolling='no' title='Отображение направлений от точки А до точки Б на карте' src='//codepen.io/azuremaps/embed/zRyNmP/?height=469&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/zRyNmP/'>Отображение направлений от точки А до точки Б на карте</a> службы "Карты Azure" (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

В приведенном выше коде первый блок кода конструирует объект карты и устанавливает механизм проверки подлинности для использования маркера доступа. См. инструкции по [созданию карты](./map-create.md).

Второй блок кода создает и добавляет на карту объект [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource).

Третий блок кода создает начальную и конечную точки для маршрута. Затем он добавляет их в источник данных. Инструкции по использованию [addPins](/javascript/api/azure-maps-control/atlas.map) см. в разделе о [добавлении булавки на карту](map-add-pin.md).

[LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer) отвечает за отображение линейных объектов в оболочке [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource) в качестве линий на карте. Четвертый блок кода позволяет создать и добавить на карту слой линий. Ознакомьтесь со свойствами слоя линий в разделе о классе [LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions).

В [слое символов](/javascript/api/azure-maps-control/atlas.layer.symbollayer) данные на основе точек в оболочке [DataSource](/javascript/api/azure-maps-control/atlas.source.datasource) преобразовываются для просмотра в качестве символов на карте с помощью текста или значков. Пятый блок кода создает и добавляет на карту слой символов. Сведения о свойствах слоя символов см. в разделе [SymbolLayerOptions](/javascript/api/azure-maps-control/atlas.symbollayeroptions).

Следующий блок кода создает точки `SouthWest` и `NorthEast` от точек начала и назначения и задает границы карты с помощью свойства карты [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-).

Последний блок кода использует [API выборки](https://fetch.spec.whatwg.org/) для выполнения запроса поиска в [Azure Maps API маршрута](/rest/api/maps/route/getroutedirections). Затем выполняется синтаксический анализ ответа. Если ответ был успешным, сведения широты и долготы используются для создания массива с линией, соединяющей эти точки. Затем данные строки добавляются в источник данных для отрисовки маршрута на карте. См. инструкции по [добавлению линий на карту](map-add-line-layer.md).

Запрос маршрута, источник данных, символ, уровни линии и границы камеры создаются внутри [прослушивателя событий](/javascript/api/azure-maps-control/atlas.map#events). Опять же, мы хотим убедиться, что результаты отображаются после полной загрузки карт.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Рекомендации по использованию службы построения маршрутов](how-to-use-best-practices-for-search.md)

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [Схема](/javascript/api/azure-maps-control/atlas.map)

Полные примеры кода см. в следующих статьях:

> [!div class="nextstepaction"]
> [Отображение данных дорожного движения на карте](./map-show-traffic.md)

> [!div class="nextstepaction"]
> [Взаимодействие с событиями Map-Mouse](./map-events.md)
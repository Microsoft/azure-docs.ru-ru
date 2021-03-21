---
title: Подключение к службе веб-компонентов (ВФС) | Карты Microsoft Azure
description: Узнайте, как подключиться к службе ВФС, а затем запросите службу ВФС с помощью веб-пакета SDK Azure Maps и модуля пространственного ввода/вывода.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 75a7da41e4f0d5356ca135eb3cb4cbe5f29e8116
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92891467"
---
# <a name="connect-to-a-wfs-service"></a>Подключение службе WFS

Служба веб-компонентов (ВФС) — это веб-служба для запроса пространственных данных с стандартизированным API, определяемым Открытый геопространственный консорциум (OGC). `WfsClient`Класс в модуле пространственного ввода-вывода позволяет разработчикам подключаться к службе ВФС и запрашивать данные из службы.

Класс поддерживает следующие функции `WfsClient` :

- Поддерживаемые версии: `1.0.0` , `1.1.0` и `2.0.0`
- Поддерживаемые операторы фильтров: двоичные сравнения, логические, математические, значения и `bbox` .
- Запросы выполняются только с помощью `HTTP GET` .
- Поддерживаемые операции:

    | Операция | Описание |
    | :-- | :-- |
    | Возможности | Создает документ метаданных с допустимыми операциями и параметрами ВФС |
    | Функцияического | Возвращает набор компонентов из источника данных |
    | дескрибефеатуретипе | Возвращает поддерживаемые типы компонентов |

## <a name="using-the-wfs-client"></a>Использование клиента ВФС

`atlas.io.ogc.WfsClient`Класс в модуле пространственных ввода-вывода упрощает выполнение запросов к службе ВФС и преобразование ответов в объекты геообъектного формата JSON. Затем этот объект геоjson можно использовать для других целей сопоставления.

Следующий код выполняет запрос к службе ВФС и отображает возвращенные функции на карте.

<br/>

<iframe height='700' scrolling='no' title='Пример простого ВФС' src='//codepen.io/azuremaps/embed/MWwvVYY/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. <a href='https://codepen.io/azuremaps/pen/MWwvVYY/'>Пример простой ВФС</a> с пером Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="supported-filters"></a>Поддерживаемые фильтры

Спецификация для стандарта ВФС использует фильтры OGC. Перечисленные ниже фильтры поддерживаются клиентом ВФС, предполагая, что вызываемая служба также поддерживает эти фильтры. Строки настраиваемых фильтров можно передать в `CustomFilter` класс.

**Логические операторы**

- `And`
- `Or`
- `Not`

**Операторы значений**

- `GmlObjectId`
- `ResourceId`

**Математические операторы**

- `Add`
- `Sub`
- `Mul`
- `Div`

**Операторы сравнения**

- `PropertyIsEqualTo`
- `PropertyIsNotEqualTo`
- `PropertyIsLessThan`
- `PropertyIsGreaterThan`
- `PropertyIsLessThanOrEqualTo`
- `PropertyIsGreaterThanOrEqualTo`
- `PropertyIsLike`
- `PropertyIsNull`
- `PropertyIsNil`
- `PropertyIsBetween`

В следующем коде показано использование различных фильтров с клиентом ВФС.

<br/>

<iframe height='500' scrolling='no' title= 'Примеры фильтров ВФС' src='//codepen.io/azuremaps/embed/NWqvYrV/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. <a href='https://codepen.io/azuremaps/pen/NWqvYrV/'>Примеры фильтров ВФС</a> пера с помощью Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="wfs-service-explorer"></a>ВФС Service Explorer

В следующем коде для изучения служб ВФС используется клиент ВФС. Выберите слой типа свойства в службе и просмотрите связанные условные обозначения.

<br/>

<iframe height='700' style='width: 100%;' scrolling='no' title= 'ВФС Service Explorer' src='//codepen.io/azuremaps/embed/bGdrvmG/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. раздел <a href='https://codepen.io/azuremaps/pen/bGdrvmG/'>ВФС Service Explorer</a> by Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) на <a href='https://codepen.io'>CodePen</a>.
</iframe>

Для доступа к службам ВФС, размещенным на конечных точках, не поддерживающих CORS, прокси-служба с поддержкой CORS может быть передана в `proxyService` параметр клиента ВФС, как показано ниже. 

```JavaScript
//Create the WFS client to access the service and use the proxy service settings
client = new atlas.io.ogc.WfsClient({
    url: url,
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
});
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [вфсклиент](/JavaScript/api/azure-maps-spatial-io/atlas.io.ogc.wfsclient)

> [!div class="nextstepaction"]
> [вфссервицеоптионс](/JavaScript/api/azure-maps-spatial-io/atlas.wfsserviceoptions)

Дополнительные примеры кода для добавления в карты см. в следующих статьях:

> [!div class="nextstepaction"]
> [Использование основных операций](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Сведения о поддерживаемых форматах данных](spatial-io-supported-data-format-details.md)
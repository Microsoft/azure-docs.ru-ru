---
title: Добавление слоя-схемы Открытый геопространственный консорциум (OGC) | Карты Microsoft Azure
description: Узнайте, как наложение слоя карт OGC на карте и использование различных параметров в классе Огкмаплайер.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/02/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 7d94a681fa987a4b23dbcda744c2f8516da4437a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92891671"
---
# <a name="add-a-map-layer-from-the-open-geospatial-consortium-ogc"></a>Добавление слоя карт из Открытый геопространственный консорциум (OGC)

`atlas.layer.OgcMapLayer`Класс может накладывать образы служб веб-карты (WMS) и изображения служб веб-карты (ВМТС) на карте. WMS — это стандартный протокол, разработанный OGC для обслуживания географических изображений карт через Интернет. Географические ссылки на изображения — это процессы связывания изображения с географическим расположением. ВМТС также является стандартным протоколом, разработанным OGC. Он предназначен для обслуживания предварительно подготовленных и геоссылок плиток карт.

В следующих разделах описаны функции службы веб-карт, поддерживаемые `OgcMapLayer` классом.

**Служба веб-карт (WMS)**

- Поддерживаемые версии: `1.0.0` , `1.1.0` , `1.1.1` и `1.3.0`
- Служба должна поддерживать `EPSG:3857` систему проекции или обработку репроекций.
- Жетфеатуреинфо требует, чтобы служба поддерживала `EPSG:4326` или обрабатывал РЕПРОЕКЦИИ. 
- Поддерживаемые операции:

    | Операция | Описание |
    | :-- | :-- |
    | Возможности | Извлекает метаданные о службе с поддерживаемыми возможностями |
    | жетмап | Извлекает изображение схемы для указанной области. |
    | жетфеатуреинфо | Возвращает `feature_info` , который содержит базовые данные о компоненте |

**Служба плиток веб-карты (ВМТС)**

- Поддерживаемые версии: `1.0.0`
- Плитки должны быть квадратными, то есть `TileWidth == TileHeight` .
- Поддерживаемые CR: `EPSG:3857` или `GoogleMapsCompatible` 
- Идентификатор Тилематрикс должен быть целочисленным значением, соответствующим уровню масштабирования на карте. На карте Azure уровень масштабирования — это значение в диапазоне от `"0"` до `"22"` . Таким образом, `"0"` поддерживается, но `"00"` не поддерживается.
- Поддерживаемые операции:

    | Операция | Описание |
    | :-- | :-- |
    | Возможности | Извлекает поддерживаемые операции и функции |
    | GetTile | Извлекает изображения для определенной плитки |

## <a name="overlay-an-ogc-map-layer"></a>Наложение слоя OGC Map

`url`Может быть базовым URL-адресом для службы или полным URL-адресом с запросом для получения возможностей службы. В зависимости от предоставленных сведений клиент ВФС может испытать несколько стандартных форматов URL-адресов, чтобы определить, как изначально получить доступ к службе.

В следующем коде показано, как наложить слой OGC Map на карте.

<br/>

<iframe height='700' scrolling='no' title='Пример слоя OGC Map' src='//codepen.io/azuremaps/embed/xxGLZWB/?height=700&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>См. <a href='https://codepen.io/azuremaps/pen/xxGLZWB/'>Пример OGC</a> ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) в <a href='https://codepen.io'>CodePen</a>с помощью Azure Maps ().
</iframe>

## <a name="ogc-map-layer-options"></a>Параметры слоя карт OGC

В приведенном ниже примере показаны различные параметры слоя карт OGC. Вы можете нажать кнопку с пером Code в правом верхнем углу, чтобы изменить перо кода.

<br/>

<iframe height='700' scrolling='no' title='Параметры слоя карт OGC' src='//codepen.io/azuremaps/embed/abOyEVQ/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Дополнительные сведения о Azure Maps <a href='https://codepen.io/azuremaps/pen/abOyEVQ/'>параметрах слоя OGC на карте</a> пера см <a href='https://codepen.io/azuremaps'>@azuremaps</a> . в разделе <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="ogc-web-map-service-explorer"></a>Обозреватель службы веб-карт OGC

Следующие средства накладывают изображения из служб веб-карт (WMS) и службы веб-карты (ВМТС) как уровни. Вы можете выбрать, какие слои в службе будут подготавливаться к просмотру на карте. Вы также можете просмотреть связанные условные обозначения для этих слоев.

<br/>

<iframe height='750' style='width: 100%;' scrolling='no' title='Обозреватель службы веб-карт OGC' src='//codepen.io/azuremaps/embed/YzXxYdX/?height=750&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Ознакомьтесь с разAzure Maps () в CodePen с помощью <a href='https://codepen.io/azuremaps/pen/YzXxYdX/'>OGC</a> ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) <a href='https://codepen.io'></a>.
</iframe>

Можно также указать параметры отображения для использования прокси-службы. Прокси-служба позволяет загружать ресурсы, размещенные в доменах, для которых не включена технология CORS.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> [огкмаплайер](/javascript/api/azure-maps-spatial-io/atlas.layer.ogcmaplayer)

> [!div class="nextstepaction"]
> [огкмаплайероптионс](/javascript/api/azure-maps-spatial-io/atlas.ogcmaplayeroptions)

См. следующие статьи, содержащие примеры кода, которые можно добавить в карты:

> [!div class="nextstepaction"]
> [Подключение службе WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Использование основных операций](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Сведения о поддерживаемых форматах данных](spatial-io-supported-data-format-details.md)
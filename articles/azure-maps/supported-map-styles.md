---
title: Поддерживаемые встроенные стили карт Azure Maps
description: Сведения о встроенных стилях карт, которые Azure Maps поддерживаются, например Road, blank_accessible, Спутниковое, satellite_road_labels, road_shaded_relief и ночное время.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/24/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: b3404c02dc159309ed1e9fcd6f9f6bb593fc7552
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92896945"
---
# <a name="azure-maps-supported-built-in-map-styles"></a>Azure Maps поддерживаемые встроенные стили карт

Служба Azure Maps поддерживает несколько различных встроенных стилей карт, как описано ниже.

## <a name="road"></a>road

**Дорожная** схема — это стандартная схема, которая отображает дороги. Он также отображает естественные и искусственные функции и метки для этих функций.

![стиль схемы дороги](./media/supported-map-styles/road.png)

**Применимые API:**

* [Изображение карты](/rest/api/maps/render/getmapimage)
* [Плитка карты](/rest/api/maps/render/getmaptile)
* Элемент управления картой веб-пакета SDK
* Элемент управления картой Android
* визуальный элемент Power BI;

## <a name="blank-and-blank_accessible"></a>пусто и blank_accessible

**Пустые** и **blank_accessible** стили карт предоставляют пустой холст для визуализации данных. Стиль **blank_accessible** продолжит предоставлять обновления средства чтения с экрана с подробными сведениями о расположении, даже если базовая схема не отображается.

> [!Note]
> В веб-пакете SDK можно изменить цвет фона на карте, установив `background-color` стиль CSS для элемента Map div.

**Применимые API:**

* Элемент управления картой веб-пакета SDK

## <a name="satellite"></a>satellite

Стиль **спутник** представляет собой комбинацию спутниковых и аэроснимков.

![стиль карты вспомогательных плиток](./media/supported-map-styles/satellite.png)

**Применимые API:**

* [Фрагмент спутниковой карты](/rest/api/maps/render/getmapimagerytilepreview)
* Элемент управления картой веб-пакета SDK
* Элемент управления картой Android
* визуальный элемент Power BI;

## <a name="satellite_road_labels"></a>satellite_road_labels

Этот стиль карты представляет собой гибрид дорог и ярлыков, наложенных поверх спутниковых и аэроснимков.

![satellite_road_labels стиль схемы](./media/supported-map-styles/satellite-road-labels.png)

**Применимые API:**

* Элемент управления картой веб-пакета SDK
* Элемент управления картой Android
* визуальный элемент Power BI;

## <a name="grayscale_dark"></a>grayscale_dark

**Темно-серый** — это темная версия стиля дорожной карты.

![gray_scale стиль схемы](./media/supported-map-styles/grayscale-dark.png)

**Применимые API:**

* [Изображение карты](/rest/api/maps/render/getmapimage)
* [Плитка карты](/rest/api/maps/render/getmaptile)
* Элемент управления картой веб-пакета SDK
* Элемент управления картой Android
* визуальный элемент Power BI;

## <a name="grayscale_light"></a>grayscale_light

**источник оттенков серого** — это небольшая версия стиля схемы дороги.

![стиль схемы оттенков серого](./media/supported-map-styles/grayscale-light.png)

**Применимые API:**
* Элемент управления картой веб-пакета SDK
* Элемент управления картой Android
* визуальный элемент Power BI;

## <a name="night"></a>ночь

**Ночь** — это темная версия стиля дорожной карты с цветными дорогами и символами.

![стиль карт ночью](./media/supported-map-styles/night.png)

**Применимые API:**

* Элемент управления картой веб-пакета SDK
* Элемент управления картой Android
* визуальный элемент Power BI;

## <a name="road_shaded_relief"></a>road_shaded_relief

**road shaded relief** — это основной стиль Azure Maps, дополненный рельефами Земли.

![Затененный стиль карт рельефов](./media/supported-map-styles/shaded-relief.png)

**Применимые API:**

* [Плитка карты](/rest/api/maps/render/getmaptile)
* Элемент управления картой веб-пакета SDK
* Элемент управления картой Android
* визуальный элемент Power BI;

## <a name="high_contrast_dark"></a>high_contrast_dark

**high_contrast_dark** является темным стилем отображения с более высокой контрастностью, чем другие стили.

![Темный стиль карт с высокой контрастностью](./media/supported-map-styles/high-contrast-dark.png)

**Применимые API:**

* Элемент управления картой веб-пакета SDK
* визуальный элемент Power BI;

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как задать стиль схемы в Azure Maps.

[Выбор стиля карты](./choose-map-style.md)
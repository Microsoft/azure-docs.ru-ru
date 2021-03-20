---
title: Отображение трафика в режиме реального времени на Azure Maps Power BI Visual | Карты Microsoft Azure
description: В этой статье вы узнаете, как отображать трафик в режиме реального времени на визуальном элементе Microsoft Azure Maps для Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: fe9e86f3c1d155a1f0b29f500de3954e360b4aa7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86262106"
---
# <a name="show-real-time-traffic"></a>Отображение трафика в режиме реального времени

Функция уровня трафика накладывает данные трафика в реальном времени на карте. Чтобы включить эту функцию, переместите ползунок « **уровень трафика** » на панели « **Формат** » в положение « **вкл** .». Это приведет к наложению данных потока трафика в качестве цветовой маркировки дорог.

> [!div class="mx-imgBorder"]
> ![Схема отображения данных трафика в режиме реального времени](media/power-bi-visual/traffic-layer.png)

В разделе **уровень трафика** доступны следующие параметры.

| Параметр         | Описание    |
|-----------------|----------------|
| Показывать инциденты  | Указывает, должны ли на карте отображаться такие инциденты трафика, как замыкание на командировки и создание. |
| Элемент управления для дорожного движения | Добавляет на карту кнопку, позволяющую читателям отчетов включать или отключать уровень трафика.  |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure Maps Power BI визуального элемента:

> [!div class="nextstepaction"]
> [Основные сведения о слоях в Azure Maps Power BI Visual](power-bi-visual-understanding-layers.md)

> [!div class="nextstepaction"]
> [Управление Azure Maps визуального элемента в Организации](power-bi-visual-manage-access.md)

---
title: Схема индекса текста Kinect для Azure
description: Узнайте, как выполнить запрос к карте индекса отслеживания текста в Azure Kinect DK.
author: qm13
ms.author: quentinm
ms.reviewer: yijwan
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, перенос, основной текст, отслеживание, индекс, сегментация, Map
ms.openlocfilehash: 8c128e59cef515aafb4f59794f0f644b90fd625d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85277029"
---
# <a name="azure-kinect-body-tracking-index-map"></a>Схема индекса отслеживания текста Kinect для Azure

Схема индексов текста включает карту сегментации экземпляров для каждого текста в записи с глубиной глубины. Каждый пиксель сопоставляется с соответствующим пикселем в глубине или в виде ИНФРАКРАСного изображения. Значение для каждого пикселя соответствует тексту, к которому принадлежит пиксель. Это может быть либо Background (значение `K4ABT_BODY_INDEX_MAP_BACKGROUND` ), либо индекс обнаруженного объекта `k4abt_body_t` .

![Пример таблицы индексов тела](./media/concepts/body-index-map.png)

>[!NOTE]
> Индекс текста отличается от идентификатора тела. Можно запросить идентификатор тела из заданного индекса тела, вызвав API: [k4abt_frame_get_body_id ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga1d612404d133a279af847974e9359a92.html#ga1d612404d133a279af847974e9359a92).


## <a name="using-body-index-map"></a>Использование схемы индекса тела

Схема индекса тела хранится в виде `k4a_image_t` и имеет то же разрешение, что и изображение глубины или IR. Каждый пиксель имеет 8-битное значение. Его можно запросить из, `k4abt_frame_t` вызвав `k4abt_frame_get_body_index_map` . Разработчик несет ответственность за освобождение памяти для гиперкарты индекса текста путем вызова `k4a_image_release()` .

## <a name="next-steps"></a>Дальнейшие действия

[Создание первого приложения для отслеживания тела](build-first-body-app.md)

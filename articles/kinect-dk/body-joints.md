---
title: Kinect для отслеживания тела Azure
description: Общие сведения о фрейме тела, соединениях, связных координатах и параллельной иерархии в Azure Kinect DK.
author: qm13
ms.author: quentinm
ms.reviewer: cedmonds, abalan
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, перенос, основной текст, отслеживание, совместная, иерархия, кость, подключение
ms.openlocfilehash: 4cf6ac13a93d0674f9fa144abcc3153a2d7c3350
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276997"
---
# <a name="azure-kinect-body-tracking-joints"></a>Kinect для отслеживания тела Azure

Отслеживание тела Azure Kinect может одновременно отслеживать несколько кадров. Каждый текст включает идентификатор для временной корреляции между кадрами и скелетом с заготовкой. Количество основных текстов, обнаруженных в каждом кадре, можно получить с помощью `k4abt_frame_get_num_bodies()` .

## <a name="joints"></a>Костей

Совместное положение и ориентация определяются относительно области ссылки на глобальный датчик глубины. Это расположение указывается в миллиметрах. Ориентация выражается как нормализованный кватернион.

## <a name="joint-coordinates"></a>Совместное координата

Положение и ориентация каждого совместного соединения образуют собственную систему координат. Все совместно используемые системы координат являются абсолютными системами координат по сравнению с трехмерной системой координат камеры.

> [!NOTE]
> Совместная координата имеет координаты оси. Ориентация на оси широко используется для коммерческих аватаров, игровых модулей и программ подготовки к просмотру. Использование ориентации осей упрощает зеркальные перемещения, например, вызывайте обе руки на 20 градусов.

![Совместное координата](./media/concepts/joint-coordinates.png)

## <a name="joint-hierarchy"></a>Совместная иерархия

Скелет включает 32 соединений с Объединенной иерархией, передаваемых из центра тела в екстремитиес. Каждое соединение (кость) связывает родительский друг с другом дочернего соединения. На рисунке показаны совместное размещение и подключение относительно текста человека.

![Совместная иерархия](./media/concepts/joint-hierarchy.png)

В следующей таблице перечисляются стандартные соединения соединений.

|Индекс |Совместное имя     | Родительский совместный   |
|------|---------------|----------------|
| 0    |пелвис         | -              |
| 1    |SPINE_NAVAL    | пелвис         |
| 2    |SPINE_CHEST    | SPINE_NAVAL    |
| 3    |ГОРЛЫШКА           | SPINE_CHEST    |
| 4    |CLAVICLE_LEFT  | SPINE_CHEST    |
| 5    |SHOULDER_LEFT  | CLAVICLE_LEFT  |
| 6    |ELBOW_LEFT     | SHOULDER_LEFT  |
| 7    |WRIST_LEFT     | ELBOW_LEFT     |
| 8    |HAND_LEFT      | WRIST_LEFT     |
| 9    |HANDTIP_LEFT   | HAND_LEFT      |
| 10   |THUMB_LEFT     | WRIST_LEFT     |
| 11   |CLAVICLE_RIGHT | SPINE_CHEST    |
| 12   |SHOULDER_RIGHT | CLAVICLE_RIGHT |
| 13   |ELBOW_RIGHT    | SHOULDER_RIGHT |
| 14   |WRIST_RIGHT    | ELBOW_RIGHT    |
| 15   |HAND_RIGHT     | WRIST_RIGHT    |
| 16   |HANDTIP_RIGHT  | HAND_RIGHT     |
| 17   |THUMB_RIGHT    | WRIST_RIGHT    |
| 18   |HIP_LEFT       | пелвис         |
| 19   |KNEE_LEFT      | HIP_LEFT       |
| 20   |ANKLE_LEFT     | KNEE_LEFT      |
| 21   |FOOT_LEFT      | ANKLE_LEFT     |
| 22   |HIP_RIGHT      | пелвис         |
| 23   |KNEE_RIGHT     | HIP_RIGHT      |
| 24   |ANKLE_RIGHT    | KNEE_RIGHT     |
| 25   |FOOT_RIGHT     | ANKLE_RIGHT    |
| 26   |HEAD           | ГОРЛЫШКА           |
| 27   |НОС           | HEAD           |
| 28   |EYE_LEFT       | HEAD           |
| 29   |EAR_LEFT       | HEAD           |
| 30   |EYE_RIGHT      | HEAD           |
| 31   |EAR_RIGHT      | HEAD           |

## <a name="next-steps"></a>Дальнейшие действия

[Отслеживание тела — карта индекса](body-index-map.md)

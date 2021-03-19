---
title: Доступ к данным Azure Kinect DK в основном фрейме
description: Сведения о данных в кадре тела и функциях для доступа к этим данным в Azure Kinect DK.
author: qm13
ms.author: quentinm
ms.reviewer: yijwan
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: how-to
keywords: текст, фрейм, Azure, Kinect, текст, отслеживание, советы
ms.openlocfilehash: be44f59cb84e99129bf526575293eee69ca64598
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85277045"
---
# <a name="access-data-in-body-frame"></a>Получение доступа к данным о корпусе

В этой статье описываются данные, содержащиеся в кадре тела, а также функции для доступа к этим данным.

Рассматриваются следующие функции:

- [k4abt_frame_get_body_id ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga1d612404d133a279af847974e9359a92.html#ga1d612404d133a279af847974e9359a92)
- [k4abt_frame_get_body_index_map ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga0e4f2d0d7e330d444de7070fb1fee4f6.html#ga0e4f2d0d7e330d444de7070fb1fee4f6)
- [k4abt_frame_get_body_skeleton ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gac7032ab06268253538556750775064fb.html#gac7032ab06268253538556750775064fb)
- [k4abt_frame_get_capture ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gad9eef11f6496bbfe997536c374217d9a.html#gad9eef11f6496bbfe997536c374217d9a)
- [k4abt_frame_get_num_bodies ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga29ab088b1a0d1a246bdb5542e21aa3c3.html#ga29ab088b1a0d1a246bdb5542e21aa3c3)
- [k4abt_frame_get_device_timestamp_usec ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga04be7b814b40296cd6b97044ed7283e4.html#ga04be7b814b40296cd6b97044ed7283e4)

## <a name="key-components-of-a-body-frame"></a>Ключевые компоненты основного фрейма

Каждый фрейм тела содержит коллекцию структур текста, карту индекса 2D-тела и захваченную запись, создавшую этот результат.

![Компоненты основного фрейма](./media/how-to-guides/body-frame.png)

## <a name="access-the-collection-of-body-structs"></a>Доступ к коллекции структур тела

В одной записи может быть обнаружено несколько текстов. Можно запросить количество тела, вызвав функцию [k4abt_frame_get_num_bodies ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga29ab088b1a0d1a246bdb5542e21aa3c3.html#ga29ab088b1a0d1a246bdb5542e21aa3c3) .

```C
size_t num_bodies = k4abt_frame_get_num_bodies(body_frame);
```

Функции [k4abt_frame_get_body_id ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga1d612404d133a279af847974e9359a92.html#ga1d612404d133a279af847974e9359a92) и [k4abt_frame_get_body_skeleton ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gac7032ab06268253538556750775064fb.html#gac7032ab06268253538556750775064fb) используются для прохода по каждому индексу тела, чтобы найти идентификатор тела и сведения о положении или ориентации.

```C
for (size_t i = 0; i < num_bodies; i++)
{
    k4abt_skeleton_t skeleton;
    k4abt_frame_get_body_skeleton(body_frame, i, &skeleton);
    uint32_t id = k4abt_frame_get_body_id(body_frame, i);
}
```

## <a name="access-the-body-index-map"></a>Доступ к карте индекса текста

Для доступа к карте индекса тела используется функция [k4abt_frame_get_body_index_map ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga0e4f2d0d7e330d444de7070fb1fee4f6.html#ga0e4f2d0d7e330d444de7070fb1fee4f6) . Подробные сведения о карте индексов тела см. в [таблице индексов текста](body-index-map.md) . Не забудьте освободить таблицу индексов текста, если она больше не нужна.

```C
k4a_image_t body_index_map = k4abt_frame_get_body_index_map(body_frame);
...  // Do your work with the body index map
k4a_image_release(body_index_map);
```

## <a name="access-the-input-capture"></a>Доступ к записи входных данных

Средство записи текста является асинхронным API. Исходная запись может быть уже освобождена на момент извлечения результата. Используйте функцию [k4abt_frame_get_capture ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gad9eef11f6496bbfe997536c374217d9a.html#gad9eef11f6496bbfe997536c374217d9a) , чтобы запросить входную запись, используемую для создания этого результата отслеживания текста. Счетчик ссылок для k4a_capture_t увеличивается при каждом вызове этой функции. Используйте функцию [k4a_capture_release ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6) , если запись больше не нужна.

```C
k4a_capture_t input_capture = k4abt_frame_get_capture(body_frame);
... // Do your work with the input capture
k4a_capture_release(input_capture);
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Пакет средств разработки для отслеживания тела в Azure Kinect](https://microsoft.github.io/Azure-Kinect-Body-Tracking/)

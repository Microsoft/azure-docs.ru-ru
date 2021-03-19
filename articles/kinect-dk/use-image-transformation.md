---
title: Использование преобразований образов SDK для датчика Kinect для Azure
description: Узнайте, как использовать функции преобразования изображений из пакета SDK для датчика Kinect для Azure.
author: cdedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.topic: conceptual
ms.date: 06/26/2019
keywords: Kinect, Azure, датчик, пакет SDK, система координат, калибровка, проект, отменяющий проект, преобразование, RGB-d, облако точек
ms.openlocfilehash: df7f2aa13c0e9c0241494e96e720b30f3ff1d8a5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276905"
---
# <a name="use-azure-kinect-sensor-sdk-image-transformations"></a>Использование преобразований образов SDK для датчика Kinect для Azure

Следуйте конкретным функциям, чтобы использовать и преобразовывать образы между системами с координированной камерой в Azure Kinect DK.

## <a name="k4a_transformation-functions"></a>функции k4a_transformation

 Все функции с префиксом *k4a_transformation* работают с целыми изображениями. Для них требуется, чтобы маркер преобразования [k4a_transformation_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__transformation__t.html) получен с помощью [k4a_transformation_create ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga853a1a5b6d521bbdf523a69e890c4f10.html#ga853a1a5b6d521bbdf523a69e890c4f10) и не выделены с помощью [k4a_transformation_destroy ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7d3ecaae66f26c1a89da9042b1bc6d44.html#ga7d3ecaae66f26c1a89da9042b1bc6d44). Можно также обратиться к [примеру преобразования](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/transformation) SDK, в котором показано, как использовать три функции в этом разделе.

Рассматриваются следующие функции:

* [`k4a_transformation_depth_image_to_color_camera()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514)
* [`k4a_transformation_depth_image_to_color_camera_custom()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gac00dd00e7612a86382e3d0a130f276bb.html#gac00dd00e7612a86382e3d0a130f276bb)
* [`k4a_transformation_color_image_to_depth_camera()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf3a941f07bb0185cd7a72699a648fc29.html#gaf3a941f07bb0185cd7a72699a648fc29)
* [`k4a_transformation_depth_image_to_point_cloud()`](
https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7385eb4beb9d8892e8a88cf4feb3be70.html#ga7385eb4beb9d8892e8a88cf4feb3be70)

### <a name="k4a_transformation_depth_image_to_color_camera"></a>k4a_transformation_depth_image_to_color_camera

#### <a name="overview"></a>Обзор

 Функция [k4a_transformation_depth_image_to_color_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514) преобразует карту глубины с точки зрения камеры глубины в точку зрения цветовой камеры. Эта функция предназначена для создания так называемых изображений RGB-D, где D представляет дополнительный канал изображения, записывающий значение глубины. Как видно на рисунке ниже, цветное изображение и вывод [k4a_transformation_depth_image_to_color_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514) выглядят так, как будто они взяты из одной точки зрения, то есть точки зрения цветовой камеры.

   ![Преобразование «изображение»](./media/how-to-guides/image-transformation.png)

#### <a name="implementation"></a>Реализация

 Эта функция преобразования сложнее, чем просто вызов [k4a_calibration_2d_to_2d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3b6bf6dedbfe67468e2f895dcce68ed4.html#ga3b6bf6dedbfe67468e2f895dcce68ed4) для каждого пикселя. Он деформации сетку треугольника от геометрии камеры глубины до геометрии цветовой камеры. Сетка треугольника используется, чтобы избежать создания отверстий в преобразованном изображении глубины. Z-буфер обеспечивает правильную обработку окклусионс. По умолчанию для этой функции включено ускорение GPU.

#### <a name="parameters"></a>Параметры

 Входные параметры — это маркер преобразования и изображение глубины. Разрешение изображения глубины должно соответствовать ```depth_mode``` указанному при создании маркера преобразования. Например, если маркер преобразования был создан с помощью ```K4A_DEPTH_MODE_WFOV_UNBINNED``` режима 1024x1024, то разрешение глубины изображения должно быть 1024x1024 пикселей. Выходные данные представляют собой преобразованное изображение глубины, которое должно быть выделено пользователем путем вызова [k4a_image_create ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). Разрешение преобразованного изображения глубины должно соответствовать значению, ```color_resolution``` указанному при создании маркера преобразования. Например, если для разрешения цвета задано значение `K4A_COLOR_RESOLUTION_1080P` , то разрешение выходного изображения должно быть 1920 x 1080 пикселей. Выходной образ STRIDE имеет значение `width * sizeof(uint16_t)` , так как в образе сохраняются 16-разрядные значения глубины.

### <a name="k4a_transformation_depth_image_to_color_camera_custom"></a>k4a_transformation_depth_image_to_color_camera_custom

#### <a name="overview"></a>Обзор

 Функция [k4a_transformation_depth_image_to_color_camera_custom ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gac00dd00e7612a86382e3d0a130f276bb.html#gac00dd00e7612a86382e3d0a130f276bb) преобразует карту глубины и пользовательский образ из точки зрения камеры с глубиной в точку зрения цветовой камеры. В качестве расширения [k4a_transformation_depth_image_to_color_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514)эта функция предназначена для создания соответствующего пользовательского изображения, для которого каждый пиксель соответствует соответствующим координатам пикселов цветовой камеры, дополнительным для преобразованного изображения глубины.

#### <a name="implementation"></a>Реализация

 Эта функция преобразования создает изображение с преобразованной глубиной так же, как [k4a_transformation_depth_image_to_color_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514). Для преобразования пользовательского образа эта функция предоставляет варианты использования линейной интерполяции или ближайшей интерполяции смежных соседних элементов. С помощью линейной интерполяции можно создать новые значения в преобразованном пользовательском изображении. Использование ближайшей интерполяции соседей предотвратит появление в выходном изображении значений, отсутствующих в исходном изображении, но приведет к уменьшению плавного изображения. Пользовательский образ должен быть одноканальным (8-или 16-разрядным). По умолчанию для этой функции включено ускорение GPU.

#### <a name="parameters"></a>Параметры

 Входные параметры — это маркер преобразования, изображение глубины, пользовательский образ и тип интерполяции. Изображение глубины и разрешение пользовательского изображения должны соответствовать `depth_mode` указанному при создании маркера преобразования. Например, если маркер преобразования был создан с помощью `K4A_DEPTH_MODE_WFOV_UNBINNED` режима 1024x1024, то разрешение изображения глубины и пользовательского изображения должно быть 1024x1024 пикселей. Значение `interpolation_type` должно быть либо `K4A_TRANSFORMATION_INTERPOLATION_TYPE_LINEAR` `K4A_TRANSFORMATION_INTERPOLATION_TYPE_NEAREST` . Выходные данные представляют собой преобразованное изображение глубины и преобразованный пользовательский образ, который пользователь должен выделить с помощью вызова [k4a_image_create ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). Разрешение преобразованного изображения глубины и преобразованного пользовательского изображения должно соответствовать `color_resolution` указанному при создании маркера преобразования. Например, если для разрешения цвета задано значение `K4A_COLOR_RESOLUTION_1080P` , то разрешение выходного изображения должно быть 1920 x 1080 пикселей. Глубина вывода изображения STRIDE имеет значение `width * sizeof(uint16_t)` , так как в образе сохраняются 16-разрядные значения глубины. Входной пользовательский образ и преобразованный пользовательский образ должны иметь формат `K4A_IMAGE_FORMAT_CUSTOM8` или соответствующим `K4A_IMAGE_FORMAT_CUSTOM16` образом задать соответствующий шаг изображения. 

### <a name="k4a_transformation_color_image_to_depth_camera"></a>k4a_transformation_color_image_to_depth_camera

#### <a name="overview"></a>Обзор

 Функция [k4a_transformation_color_image_to_depth_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf3a941f07bb0185cd7a72699a648fc29.html#gaf3a941f07bb0185cd7a72699a648fc29) преобразует цветное изображение из точки зрения цветовой камеры в точку обзора камеры глубины (см. рисунок выше). Его можно использовать для создания изображений RGB-D.

#### <a name="implementation"></a>Реализация

 Для каждого пикселя на карте глубины функция использует значение глубины пикселя, чтобы вычислить соответствующую координату точки в цветном изображении. Затем мы будем искать значение цвета по этой координате в цветном изображении. Билинейной интерполяция выполняется в цветном изображении для получения значения цвета с точностью до точки. Пикселу, у которого нет связанной глубины, присваивается значение BGRA `[0,0,0,0]` в выходном изображении. По умолчанию для этой функции включено ускорение GPU. Поскольку этот метод создает отверстия в изображении преобразованного цвета и не обрабатывает окклусионс, рекомендуется использовать функцию [k4a_transformation_depth_image_to_color_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514) .

#### <a name="parameters"></a>Параметры

Входными параметрами являются маркер преобразования, изображение глубины и цвет изображения. Разрешения изображений глубины и цвета должны соответствовать depth_mode и color_resolution, указанным при создании маркера преобразования. Выходные данные — это преобразованное изображение цвета, которое должно быть выделено пользователем путем вызова [k4a_image_create ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). Разрешение преобразованного цветного изображения должно соответствовать depth_resolution, заданному при создании маркера преобразования. В выходном образе хранятся 4 8-битные значения, представляющие BGRA для каждого пикселя. Таким образом, шаг изображения имеет значение ```width * 4 * sizeof(uint8_t)``` . Порядок данных помещается в пикселах, то есть синим значением — пиксель 0, зеленым значением — пиксель 0, красным значением — пиксель 0, альфа-значение — пиксель 0, синее значение — пиксель 1 и т. д.

### <a name="k4a_transformation_depth_image_to_point_cloud"></a>k4a_transformation_depth_image_to_point_cloud

#### <a name="overview"></a>Обзор

Функция [k4a_transformation_depth_image_to_point_cloud ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7385eb4beb9d8892e8a88cf4feb3be70.html#ga7385eb4beb9d8892e8a88cf4feb3be70) преобразует двухмерную карту глубины, полученную камерой, в облако с трехмерной точкой в системе координат той же камеры. Таким образом, Камера может быть глубиной или цветной камерой.

#### <a name="implementation"></a>Реализация

 Функция дает эквивалентные результаты для выполнения [k4a_calibration_2d_to_2d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3b6bf6dedbfe67468e2f895dcce68ed4.html#ga3b6bf6dedbfe67468e2f895dcce68ed4) для каждого пикселя, но имеет более эффективный вычислитель. При вызове [k4a_transformation_create ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga853a1a5b6d521bbdf523a69e890c4f10.html#ga853a1a5b6d521bbdf523a69e890c4f10)мы предрассчитываем так называемую таблицу уточняющего запроса, которая хранит коэффициенты x и y для каждого пикселя изображения. При вызове [k4a_transformation_depth_image_to_point_cloud ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7385eb4beb9d8892e8a88cf4feb3be70.html#ga7385eb4beb9d8892e8a88cf4feb3be70)мы получаем трехмерную координату x в пикселях путем умножения коэффициента x пикселя на координату Z пикселя. Аналогично, трехмерная координата Y вычислена путем умножения с коэффициентом Y. В [примере с быстрым примером облака](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/fastpointcloud) в пакете SDK показано, как вычисляются XY-таблицы. Пользователи могут следовать примеру кода, чтобы реализовать собственную версию этой функции, например, чтобы ускорить работу своего конвейера GPU.

#### <a name="parameters"></a>Параметры

 Входные параметры — это маркер преобразования, спецификатор камеры и изображение глубины. Если для описателя камеры задано значение Depth, разрешение изображения глубины должно соответствовать depth_mode, указанному при создании маркера преобразования. В противном случае, если для описателя задана цветовая камера, разрешение должно соответствовать разрешению выбранного color_resolution. Выходной параметр — это изображение XYZ, которое должно быть выделено пользователем путем вызова [k4a_image_create ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). Разрешение изображения XYZ должно соответствовать разрешению схемы глубины входных данных. Для каждого пикселя в миллиметре сохраняются три 16-битных значения координат со знаком. Таким образом, шаг с изображением XYZ имеет значение `width * 3 * sizeof(int16_t)` . Порядок данных состоит из чередования, то есть координаты X — пиксель 0, Y-координата — пиксель 0, Z-координата — пиксель 0, X-координата — пиксель 1 и т. д. Если пиксель не может быть преобразован в трехмерный режим, функция присваивает значения `[0,0,0]` пикселю.

## <a name="samples"></a>Примеры

[Пример преобразования](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/transformation)

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как использовать функции преобразования изображений из пакета SDK для датчика Kinect для Azure, вы также можете узнать о

>[!div class="nextstepaction"]
>[Функции калибровки SDK для датчика Kinect для Azure](use-calibration-functions.md)

Также вы можете ознакомиться с

[Системы координат](coordinate-systems.md)

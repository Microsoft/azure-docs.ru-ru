---
title: Использование функций калибровки Azure Kinect
description: Узнайте, как использовать функции калибровки для Azure Kinect DK.
author: cdedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: Kinect, Azure, датчик, пакет SDK, система координат, калибровка, функции, Камера, внутренний, внешний, проект, отменяющий проект, преобразование, RGB-d, облако точек
ms.openlocfilehash: a501765222154bde4ed35e878a9d18042893fca2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85276941"
---
# <a name="use-azure-kinect-calibration-functions"></a>Использование функций калибровки Azure Kinect

Функции калибровки позволяют преобразовывать точки между системами координат каждого датчика на устройстве Azure Kinect. Приложения, которым требуется преобразование целых изображений, могут использовать преимущества ускоренных операций, доступных в [функциях преобразования](use-image-transformation.md).

## <a name="retrieve-calibration-data"></a>Получение данных калибровки

Необходимо получить калибровку устройства для выполнения системных преобразований координат. Данные калибровки хранятся в [k4a_calibration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__calibration__t.html#details) типе данных. Он получается с устройства с помощью функции [k4a_device_get_calibration ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4e43940d8d8db48da266c7a7842c8d78.html#ga4e43940d8d8db48da266c7a7842c8d78). Данные калибровки относятся не только к каждому устройству, но и к режиму работы камер. Поэтому [k4a_device_get_calibration ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4e43940d8d8db48da266c7a7842c8d78.html#ga4e43940d8d8db48da266c7a7842c8d78) требует `depth_mode` Параметры и в `color_resolution` качестве входных данных.

### <a name="opencv-compatibility"></a>Совместимость OpenCV

Параметры калибровки совместимы с [opencv](https://opencv.org/). Дополнительные сведения о параметрах калибровки отдельных камер см. в разделе также [opencv документация](https://docs.opencv.org/3.2.0/d9/d0c/group__calib3d.html#gga7041b2a9c8f9f8ee93a2796981bc5546a204766e24f2e413e7a7c9f8b9e93f16c). См. также [Пример совместимости opencv](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/opencv_compatibility) пакета SDK, демонстрирующий преобразование между типом [k4a_calibration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__calibration__t.html#details) и соответствующими структурами данных opencv.

## <a name="coordinate-transformation-functions"></a>Функции преобразования координат

На рисунке ниже показаны различные системы координат Azure Kinect, а также функции для преобразования между ними. Мы не будем использовать трехмерные системы координат гироскопом и акселерометр, чтобы не усложнять рисунок.

   ![Преобразование координат](./media/how-to-guides/coordinate-transformation.png)

Зазамечание при искажении линз: двумерные координаты всегда ссылаются на искаженное изображение в пакете SDK. [Пример расискажений](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/undistort) в пакете SDK демонстрирует расискажение изображения. Обычно трехмерные точки не будут затронуты искажениями линз.

### <a name="convert-between-3d-coordinate-systems"></a>Преобразование между системами координат трехмерной графики

Функция [k4a_calibration_3d_to_3d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaacd1eab997ef964b5e436afa5295726e.html#gaacd1eab997ef964b5e436afa5295726e) преобразует трехмерную точку исходной системы координат в трехмерную точку целевой системы координат, используя для этого внешний режим калибровки камеры. Для исходных и целевых объектов можно задать любую из четырех трехмерных систем координат, то есть цветовой камеры, глубины камеры, гироскопом или акселерометр. Если исходный и целевой объекты идентичны, то неизмененная входная трехмерная точка возвращается в виде выходных данных.

### <a name="convert-between-2d-and-3d-coordinate-systems"></a>Преобразование между 2D-и трехмерными системами координат

Функция [k4a_calibration_3d_to_2d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ed8b51d727425caa942aab190fc2ba9.html#ga2ed8b51d727425caa942aab190fc2ba9) преобразует трехмерную точку исходной системы координат в двухмерную пиксельную координату целевой камеры. Эта функция часто называется функцией проекта. Хотя источник может быть установлен для любой из четырех трехмерных систем координат, целевой объект должен быть глубиной или цветной камерой. Если исходный и целевой объекты различаются, входные трехмерные точки преобразуются в трехмерную систему координат целевой камеры с помощью [k4a_calibration_3d_to_3d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaacd1eab997ef964b5e436afa5295726e.html#gaacd1eab997ef964b5e436afa5295726e). После представления трехмерной точки в целевой системе координат камеры соответствующие координаты 2D-пикселя вычисляются с помощью встроенной калибровки целевой камеры. Если трехмерная точка выходит за пределы видимой области целевой камеры, то допустимое значение устанавливается равным 0.

Функция [k4a_calibration_2d_to_3d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga664602bdb48dab38117a6c1d14b880de.html#ga664602bdb48dab38117a6c1d14b880de) Преобразует координату двухмерной точки исходной камеры в трехмерную точку в целевой системе координат камеры. Источник должен иметь цветовую или глубинную камеру. В качестве целевого объекта можно задать любую из четырех трехмерных систем координат. В дополнение к координатам двухмерной точки, значение глубины пикселя (в миллиметрах) на исходном изображении камеры требуется в качестве входных данных для функции. один из способов получения значения глубины в геометрической геометрии заключается в использовании функции [k4a_transformation_depth_image_to_color_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514). Функция вычислит трехмерный луч, ведущий из фокуса исходной камеры через заданную координату в пикселях, используя встроенную калибровку исходной камеры. Затем значение глубины используется для поиска точного расположения трехмерной точки на этом луч. Эта операция часто называется функцией unproject. Если исходная и Целевая камеры отличаются, функция преобразует трехмерную точку в систему координат целевого объекта с помощью [k4a_calibration_3d_to_3d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaacd1eab997ef964b5e436afa5295726e.html#gaacd1eab997ef964b5e436afa5295726e). Если координата двухмерной точки выходит за пределы видимой области исходной камеры, то допустимое значение равно 0.

### <a name="converting-between-2d-coordinate-systems"></a>Преобразование между системами координат 2D

Функция [k4a_calibration_2d_to_2d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3b6bf6dedbfe67468e2f895dcce68ed4.html#ga3b6bf6dedbfe67468e2f895dcce68ed4) преобразует пиксельную координату исходной камеры в двухмерную пиксельную координату целевой камеры. Источник и цель должны быть установлены на цветовую или глубинную камеру. Для функции требуется значение глубины пикселя (в миллиметрах) в исходном изображении камеры в качестве входных данных. один из способов получения значения глубины в геометрической геометрии заключается в использовании функции [k4a_transformation_depth_image_to_color_camera ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514). Он вызывает [k4a_calibration_2d_to_3d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga664602bdb48dab38117a6c1d14b880de.html#ga664602bdb48dab38117a6c1d14b880de) для преобразования в трехмерную точку исходной системы камеры. Затем он вызывает [k4a_calibration_3d_to_2d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ed8b51d727425caa942aab190fc2ba9.html#ga2ed8b51d727425caa942aab190fc2ba9) для преобразования в координату двухмерной точки изображения целевой камеры. Допустимое значение равно 0, если [k4a_calibration_2d_to_3d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga664602bdb48dab38117a6c1d14b880de.html#ga664602bdb48dab38117a6c1d14b880ded) или [k4a_calibration_3d_to_2d ()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ed8b51d727425caa942aab190fc2ba9.html#ga2ed8b51d727425caa942aab190fc2ba9) возвращает недопустимый результат.

## <a name="related-samples"></a>Связанные примеры

- [Пример совместимости OpenCV](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/opencv_compatibility)
- [Пример расискажений](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/undistort)
- [Пример быстрой облачной точки](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/fastpointcloud)

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знакомы с калибровками камеры, вы также можете узнать, как
>[!div class="nextstepaction"]
>[Синхронизация записи на устройствах устройств](capture-device-synchronization.md)

Также вы можете ознакомиться с

[Системы координат](coordinate-systems.md)

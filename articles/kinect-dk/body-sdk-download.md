---
title: Скачивание пакета SDK для отслеживания текста Kinect для Azure
description: Узнайте, как скачать каждую версию пакета SDK для датчика Kinect для Azure в Windows или Linux.
author: qm13
ms.author: quentinm
ms.prod: kinect-dk
ms.date: 03/18/2021
ms.topic: conceptual
keywords: Azure, Kinect, пакет SDK, скачать обновление, последние, доступные, установка, основной текст, отслеживание
ms.openlocfilehash: 60018df56433785f3cb723dd54cc96a8e5289b67
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104654498"
---
# <a name="download-azure-kinect-body-tracking-sdk"></a>Скачать пакет SDK для отслеживания текста Kinect для Azure

Этот документ содержит ссылки для установки каждой версии пакета SDK для отслеживания текста Kinect для Azure.

## <a name="azure-kinect-body-tracking-sdk-contents"></a>Содержимое пакета SDK для отслеживания текста Kinect для Azure

- Заголовки и библиотеки для создания приложения отслеживания текста с помощью Azure Kinect DK.
- Распространяемые библиотеки DLL, необходимые для приложений отслеживания текста с помощью Azure Kinect DK.
- Примеры приложений для отслеживания текста.

## <a name="windows-download-links"></a>Ссылки для загрузки Windows

Версия       | Скачать
--------------|----------
1.1.0 | [MSI](https://www.microsoft.com/en-us/download/details.aspx?id=102901)
1.0.1 | [](https://www.microsoft.com/en-us/download/details.aspx?id=100942) [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/1.0.1) MSI
1.0.0 | [](https://www.microsoft.com/en-us/download/details.aspx?id=100848) [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/1.0.0) MSI

## <a name="linux-installation-instructions"></a>Инструкции по установке (Linux)

В настоящее время единственным поддерживаемым дистрибутивом является Ubuntu 18,04 и 20,04. Чтобы запросить поддержку других дистрибутивов, перейдите на [эту страницу](https://aka.ms/azurekinectfeedback).

Сначала необходимо настроить [репозиторий пакетов Майкрософт](https://packages.microsoft.com/), следуя инструкциям, приведенным [здесь](/windows-server/administration/linux-package-repository-for-microsoft-software).

Пакет `libk4abt<major>.<minor>-dev` содержит заголовки и файлы CMake для сборки в `libk4abt`.
`libk4abt<major>.<minor>`Пакет содержит общие объекты, необходимые для запуска исполняемых файлов, зависящих от, а `libk4abt` также для примера средства просмотра.

Для работы с основными руководствами требуется пакет `libk4abt<major>.<minor>-dev`. Для установки выполните следующую команду:

`sudo apt install libk4abt<major>.<minor>-dev`

Если команда выполнена, это значит, что пакет SDK готов к использованию.

> [!NOTE]
> При установке пакета SDK запомните путь, по которому выполнена установка. Например, "C:\Program Files\azure Multi Kinect Body Tracking SDK 1.0.0". Примеры, упоминаемые в статьях, приведены в этом пути.
> Примеры отслеживания текста находятся в папке " [Body-Tracking-Samples](https://github.com/microsoft/Azure-Kinect-Samples/tree/master/body-tracking-samples) " в репозитории Azure-Kinect-Samples. Примеры, упоминаемые в статьях, можно найти здесь.

## <a name="change-log"></a>Журнал изменений

### <a name="v110"></a>v 1.1.0
* Функциями Добавлена поддержка Директмл (только для Windows) и Тенсоррт выполнения модели оценки типа "экземпляр". См. раздел часто задаваемые вопросы о новых средах выполнения.
* Функциями Добавить `model_path` в `k4abt_tracker_configuration_t` структуру. Позволяет пользователям указать путь для модели оценки объекта. По умолчанию используется `dnn_model_2_0_op11.onnx` Стандартная модель оценки, расположенная в текущем каталоге.
* Функциями Включить `dnn_model_2_0_lite_op11.onnx` модель оценки Lite. Эта модель имеет размер ~ 2 увеличение производительности для уменьшения точности на 5%.
* Функциями Проверенные примеры компилируются с помощью Visual Studio 2019 и обновления примеров для использования этого выпуска.
* [Критическое изменение] Обновите среду выполнения ONNX 1,6 с поддержкой ЦП, CUDA 11,1, Директмл (только для Windows) и Тенсоррт 7.2.1 среды выполнения. Требуется обновление драйвера NVIDIA для R455 или более высокого.
* [Критическое изменение] Установка NuGet отсутствует.
* [Исправление ошибки] Добавление поддержки для [видеоканала](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/1481) NVIDIA RTX 30Xx Series GPU
* [Исправление ошибки] Добавлена поддержка интегрированных процессоров AMD и Intel (только для [Windows)](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/1481)
* [Исправление ошибки] [Ссылка](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/1125) для обновления на CUDA 11,1
* [Исправление ошибки] Обновить [ссылку](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/1248) на 1.4.1 SDK для датчика

### <a name="v101"></a>v 1.0.1
* [Исправление ошибки] Устранена проблема, из – за которой пакет SDK аварийно завершает работу при загрузке onnxruntime.dll по пути в Windows Build 19025 или более поздней версии: [Link](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/932)

### <a name="v100"></a>v1.0.0
* Функциями Добавьте оболочку C# в установщик MSI.
* [Исправление ошибки] Устранена проблема, из – за которой невозможно правильно обнаружить вращение головного [канала: ссылка](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/997)
* [Исправление ошибки] Устранение проблемы, с которой загрузка ЦП проходит до 100% на компьютере Linux: [ссылка](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/1007)
* Регистрируют Добавьте два примера в репозиторий. В примере 1 показано, как преобразовать результаты отслеживания текста из пространства глубины в [ссылку](https://github.com/microsoft/Azure-Kinect-Samples/tree/master/body-tracking-samples/camera_space_transform_sample)цветового пространства; в примере 2 показано, как определить [ссылку](https://github.com/microsoft/Azure-Kinect-Samples/tree/master/body-tracking-samples/floor_detector_sample) на плоскость этажа

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор Azure Kinect DK](about-azure-kinect-dk.md)

- [Настройка Azure Kinect DK](set-up-azure-kinect-dk.md)

- [Настройка отслеживания текста Azure Kinect](body-sdk-setup.md)
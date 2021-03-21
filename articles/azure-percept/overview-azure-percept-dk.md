---
title: Обзор Azure Percept DK
description: Дополнительные сведения об Azure Перцепт DK
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: conceptual
ms.date: 02/18/2021
ms.custom: template-concept
ms.openlocfilehash: 74e7d1a54b1d760979dbf9833e85ec728b4e5e3a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104595895"
---
# <a name="azure-percept-dk-overview"></a>Обзор Azure Percept DK

Azure Перцепт DK — это пограничная версия пакета SDK для искусственного интеллекта и IoT, предназначенная для разработки концепций и экспериментов в виде аудиовизуальных данных. В сочетании с [Azure Перцепт Studio](./overview-azure-percept-studio.md) и [Azure перцепт Audio](./overview-azure-percept-audio.md)она стала мощным и простым в использовании платформой для создания пограничных решений искусственного интеллекта для широкого спектра концепций или приложений для работы с аудио AI. Его можно приобрести в [Microsoft Online Store](https://go.microsoft.com/fwlink/p/?LinkId=2155270).

> [!div class="nextstepaction"]
> [купить сейчас](https://go.microsoft.com/fwlink/p/?LinkId=2155270)

<!---
:::image type="content" source="./media/overview-azure-percept-dk/dk-image.png" alt-text="Azure Percept DK device.":::
--->
</br>

> [!VIDEO https://www.youtube.com/embed/Qj8NGn-7s5A]

## <a name="key-features"></a>Основные возможности

- **Возможность запуска AI на границе**. Встроенное аппаратное ускорение позволяет выполнять модели искусственного интеллекта без подключения к облаку.
- **Встроенная в аппаратное обеспечение безопасности доверенного корня**. Дополнительные сведения см. в этом обзоре [системы безопасности Azure перцепт](./overview-percept-security.md) .
- **Простая интеграция с [Azure перцепт Studio](https://go.microsoft.com/fwlink/?linkid=2135819)** и другими службами Azure. Например, центр Интернета вещей Azure, Azure Cognitive Services и [аналитика видео в реальном времени](https://docs.microsoft.com/azure/media-services/live-video-analytics-edge/overview)
- **Простая интеграция с дополнительным [аудио Azure перцепт](./overview-azure-percept-audio.md)**
- **Поддержка основных платформ искусственного интеллекта**. Например ONNX и TensorFlow.
- **Интеграция с системой салазок 80/20**. Упрощение создания прототипов в рабочих средах. Дополнительные сведения об [интеграции 80/20](./overview-8020-integration.md).

## <a name="hardware-components"></a>Компоненты оборудования

- Плата перевозчика Azure Перцепт DK
    - Процессор НКСП iMX8m
    - Доверенный платформенный модуль (TPM) (TPM) версии 2,0
    - Подключение WiFi и Bluetooth
    - Просмотреть полный [лист данных](./azure-percept-dk-datasheet.md)
- Система концепции Azure Перцепт в модуле (SoM)
    - Вычислительный блок Intel Мовидиус множество X (MA2085) (ВПУ)
    - Датчик камеры RGB с возможностью добавления второго
    - Просмотреть полный [лист данных](./azure-percept-vision-datasheet.md)

## <a name="get-started-with-the-azure-percept-dk"></a>Начало работы с Azure Перцепт DK

- Заполните эти краткие руководства
    - [Распаковка и сборка Azure Перцепт DK](./quickstart-percept-dk-unboxing.md)
    - [Настройка Azure Перцепт DK и выполнение первой концепции модели искусственного интеллекта](./quickstart-percept-dk-set-up.md)
- Начните создавать эксперименты с помощью этих учебников
    - [Создание решения компьютерного зрения в Azure Percept Studio без необходимости писать код](./tutorial-nocode-vision.md)
    - [Создание речевого помощника в Azure Перцепт Studio](./tutorial-no-code-speech.md)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Купить Azure Перцепт DK из Microsoft Online Store](https://go.microsoft.com/fwlink/p/?LinkId=2155270)

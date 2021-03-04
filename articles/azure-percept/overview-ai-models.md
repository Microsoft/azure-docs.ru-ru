---
title: Модели Azure Перцепт AI
description: Дополнительные сведения о моделях AI, доступных для создания прототипов и развертывания
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: conceptual
ms.date: 02/16/2021
ms.custom: template-concept
ms.openlocfilehash: 28a8de231f179cf69342da81e6a2ae1989d2a5d6
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102041592"
---
# <a name="azure-percept-ai-models"></a>Модели Azure Перцепт AI

Azure Перцепт позволяет разрабатывать и развертывать модели искусственного интеллекта непосредственно в Azure Перцепт DK из [Azure Перцепт Studio](https://go.microsoft.com/fwlink/?linkid=2135819). В развертывании моделей используется [центр Интернета вещей Azure](https://azure.microsoft.com/services/iot-hub/) и [Azure IOT Edge](https://azure.microsoft.com/services/iot-edge/#iotedge-overview).

## <a name="sample-ai-models"></a>Примеры моделей AI

Azure Перцепт Studio содержит образцы моделей для следующих приложений:

- Обнаружение людей
- Обнаружение автомобиля
- Общее обнаружение объектов
- продукты — обнаружение на полке

С предварительно обученными моделями не требуется сбор кода и обучающих данных. Просто разверните нужную модель в Azure Перцепт DK на портале и откройте видеопоток DevKit, чтобы увидеть, как будет изменяться в действии модель. Доступ к данным телеметрии модели также можно получить с помощью средства [Azure IOT Explorer](https://github.com/Azure/azure-iot-explorer/releases) .

## <a name="pre-built-solutions"></a>Предварительно созданные решения

Также доступна [предварительно созданное решение пространственной аналитики для обнаружения людей](https://github.com/george-moore/Santa-Cruz-AI-App) . Предварительно построенное решение — это приложение AI с открытым исходным кодом, которое обеспечивает подсчет людей с помощью определяемых пользователем записей зоны и событий выхода. Выходные данные видео и AI от локального устройства егресседся к [Azure Data Lake](https://azure.microsoft.com/solutions/data-lake/)с помощью пользовательского интерфейса, работающего как веб-сайт Azure. Выборка искусственного интеллекта обеспечивается моделью искусственного интеллекта с открытым исходным кодом для обнаружения людей.

:::image type="content" source="./media/overview-ai-models/people-detector.gif" alt-text="GIF предварительно построенное решение для пространственного анализа.":::

## <a name="custom-no-code-solutions"></a>Пользовательские решения без кода

С помощью Azure Перцепт Studio можно разрабатывать пользовательские [решения](./tutorial-nocode-vision.md) для распознавания и речи, не требующие написания кода.

Для решения пользовательских концепций доступны модели ИСКУССТВЕНного обнаружения и классификации объектов. Просто отправьте и пометьте свои обучающие изображения, которые можно получить непосредственно с помощью SoM Перцепт в Azure Перцепт DK, если это необходимо. Обучение и оценку модели легко выполнять в [пользовательское визуальное распознавание](https://www.customvision.ai/), который входит в состав [Cognitive Services Azure](https://azure.microsoft.com/services/cognitive-services/#overview).

Для пользовательских речевых решений шаблоны голосового помощника в настоящее время доступны для следующих приложений:

- Бизнес-зал — комната, оснащенная интеллектуальными устройствами с контролем голоса.
- Здравоохранение: поддержка интеллектуальных устройств с контролем голоса.
- Инвентаризация: центр инвентаризации оснащен интеллектуальными устройствами с контролем голоса.
- Автомобиль: автомобильный концентратор, оснащенный управляемыми голосовыми устройствами.

Предварительно созданные ключевые слова и команды для голоса помощника доступны непосредственно на портале. Пользовательские ключевые слова и команды могут быть созданы и обучены в [Speech Studio](https://speech.microsoft.com/), которая также является частью Cognitive Services Azure.

## <a name="advanced-development"></a>Расширенная разработка

Для опытных разработчиков доступная [Записная книжка Jupyter](https://github.com/microsoft/Project-Santa-Cruz-Preview/blob/main/Sample-Scripts-and-Notebooks/Official/Machine%20Learning%20Notebooks/Transferlearningusing_SSDLiteV2%20Model.ipynb) выполняет обучение с помощью предварительно обученной модели TensorFlow (MobileNetSSDV2Lite) в Python с пользовательским набором данных для обнаружения объектов. Записная книжка использует удаленные экземпляры вычислений с помощью [машинное обучение Azure](https://azure.microsoft.com/services/machine-learning/#product-overview) и может выполняться в облаке с помощью портала AzureML или локально в [Visual Studio Code](https://code.visualstudio.com/).

Также включены некоторые полезные [скрипты](https://github.com/microsoft/Project-Santa-Cruz-Preview/tree/main/Sample-Scripts-and-Notebooks/Official/Scripts) Python для управления наборами данных и [установщик пакета средств разработки](https://github.com/microsoft/Project-Santa-Cruz-Preview/blob/main/Sample-Scripts-and-Notebooks/Official/Machine%20Learning%20Notebooks/dev-tools-installer.md), который устанавливает и настраивает все средства, необходимые для разработки расширенного решения искусственного интеллекта.

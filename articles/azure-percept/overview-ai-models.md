---
title: Модели Azure Percept AI
description: Дополнительные сведения о моделях AI, доступных для создания прототипов и развертывания
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: conceptual
ms.date: 03/23/2021
ms.custom: template-concept
ms.openlocfilehash: f10a330de52d40f728cd628a1d7cb83b54ad1ff6
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105557366"
---
# <a name="azure-percept-ai-models"></a>Модели Azure Percept AI

Azure Перцепт позволяет разрабатывать и развертывать модели искусственного интеллекта непосредственно в [Azure ПЕРЦЕПТ DK](./overview-azure-percept-dk.md) из [Azure перцепт Studio](https://go.microsoft.com/fwlink/?linkid=2135819). В развертывании моделей используется [центр Интернета вещей Azure](https://azure.microsoft.com/services/iot-hub/) и [Azure IOT Edge](https://azure.microsoft.com/services/iot-edge/#iotedge-overview).

## <a name="sample-ai-models"></a>Примеры моделей AI

Azure Перцепт Studio содержит образцы моделей для следующих приложений:

- Обнаружение людей
- Обнаружение автомобиля
- Общее обнаружение объектов
- продукты — обнаружение на полке

С предварительно обученными моделями не требуется сбор кода и обучающих данных. Просто [разверните нужную модель](./how-to-deploy-model.md) в Azure перцепт DK на портале и откройте [видеопоток](./how-to-view-video-stream.md) DevKit, чтобы увидеть, как будет изменяться в действии модель. Доступ к данным [телеметрии модели](./how-to-view-telemetry.md) также можно получить с помощью средства [Azure IOT Explorer](https://github.com/Azure/azure-iot-explorer/releases) .

## <a name="reference-solutions"></a>Эталонные решения

Также доступна [справочное решение по инвентаризации людей](https://github.com/microsoft/Azure-Percept-Reference-Solutions/tree/main/people-detection-app) . Это эталонное решение — это приложение AI с открытым исходным кодом, обеспечивающее подсчеты людей с помощью определяемых пользователем записей зоны и событий выхода. Выходные данные видео и AI от локального устройства егресседся к [Azure Data Lake](https://azure.microsoft.com/solutions/data-lake/)с помощью пользовательского интерфейса, работающего как веб-сайт Azure. Выборка искусственного интеллекта обеспечивается моделью искусственного интеллекта с открытым исходным кодом для обнаружения людей.

:::image type="content" source="./media/overview-ai-models/people-detector.gif" alt-text="GIF предварительно построенное решение для пространственного анализа.":::

## <a name="custom-no-code-solutions"></a>Пользовательские решения без кода

С помощью Azure Перцепт Studio можно разрабатывать пользовательские [решения](./tutorial-nocode-vision.md) для [распознавания и речи](./tutorial-no-code-speech.md) , не требующие написания кода.

Для решения пользовательских концепций доступны модели ИСКУССТВЕНного обнаружения и классификации объектов. Просто отправьте и пометьте свои обучающие изображения, которые можно сделать напрямую с помощью SoM Перцепт в Azure Перцепт DK при необходимости. Обучение и оценку модели легко выполнять в [пользовательское визуальное распознавание](https://www.customvision.ai/), который входит в состав [Cognitive Services Azure](https://azure.microsoft.com/services/cognitive-services/#overview).

</br>

> [!VIDEO https://www.youtube.com/embed/9LvafyazlJM]

Для пользовательских речевых решений шаблоны голосового помощника в настоящее время доступны для следующих приложений:

- Бизнес-зал — комната, оснащенная интеллектуальными устройствами с контролем голоса.
- Здравоохранение: поддержка интеллектуальных устройств с контролем голоса.
- Инвентаризация: центр инвентаризации оснащен интеллектуальными устройствами с контролем голоса.
- Автомобиль: автомобильный концентратор, оснащенный управляемыми голосовыми устройствами.

Предварительно созданные ключевые слова и команды для голоса помощника доступны непосредственно на портале. Пользовательские ключевые слова и команды могут быть созданы и обучены в [Speech Studio](https://speech.microsoft.com/), которая также является частью Cognitive Services Azure.

## <a name="advanced-development"></a>Расширенная разработка

Ознакомьтесь с актуальными руководствами, учебниками и примерами [для следующих](https://github.com/microsoft/azure-percept-advanced-development) целей:

- Развертывание настраиваемой модели AI в Azure Перцепт DK
- Обновление поддерживаемой модели с помощью перемещения обучения
- И многое другое

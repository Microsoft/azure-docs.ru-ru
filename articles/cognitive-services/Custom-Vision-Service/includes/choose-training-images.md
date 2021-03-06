---
author: PatrickFarley
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: include
ms.date: 07/17/2019
ms.author: pafarley
ms.openlocfilehash: 9f8eadea198bbae3de2ffc1b3aaac48925719586
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "100106125"
---
Мы рекомендуем включить в начальный обучающий набор не менее 30 изображений для каждого тега. Кроме того, вам потребуются несколько дополнительных изображений для тестирования обученной модели.

Чтобы обучение модели было эффективным, используйте разнообразные изображения. Изображения должны отличаться по следующим аспектам:
* угол обзора камеры;
* освещение;
* background
* стиль изображения;
* отдельные объекты и группы;
* size
* type

Также убедитесь, что все обучающие изображения соответствуют следующим критериям:
* формат JPG, PNG, BMP или GIF;
* размер не более 6 МБ (4 МБ для прогнозирования изображений);
* не менее 256 пикселей по короткой стороне (Пользовательская служба визуального распознавания автоматически увеличивает изображения меньшего размера).

> [!NOTE]
> Вам нужен более широкий набор изображений для выполнения обучения? Trove, проект Microsoft Garage, позволяет создавать и покупать наборы изображений для обучения. После сбора изображений их можно скачать, а затем импортировать в проект Пользовательского визуального распознавания обычным способом. Чтобы узнать больше, посетите [страницу Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3).
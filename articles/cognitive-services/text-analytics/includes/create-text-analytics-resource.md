---
title: Создание ресурса анализа текста Cognitive Services
titleSuffix: Azure Cognitive Services
description: Узнайте, как создать ресурс Анализа текста Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 02/09/2021
ms.author: aahi
ms.openlocfilehash: 5b6479d48a51ba962f2f6bfba16dac3b0886a9ff
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101750458"
---
## <a name="create-a-cognitive-services-text-analytics-resource"></a>Создание ресурса анализа текста Cognitive Services

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите **Создать ресурс**, а затем последовательно выберите **Искусственный интеллект и машинное обучение** > **Анализ текста**.
   Или перейдите к разделу [создание анализ текста](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics).
1. Введите все необходимые параметры:

    |Параметр|Значение|
    |--|--|
    |Имя|Введите имя (2–64 символа).|
    |Подписка|Выберите соответствующую подписку.|
    |Расположение|Выберите ближайшее расположение.|
    |Ценовая категория| Введите **S** (ценовая категория "Стандартный").|
    |Группа ресурсов|Выберите доступную группу ресурсов.|

1. Нажмите **Создать** и подождите, пока ресурс будет создан. Браузер автоматически перенаправит на созданную страницу ресурсов.
1. Собирайте настроенный `endpoint` и ключ API:

    |Вкладка ресурсов на портале|Параметр|Значение|
    |--|--|--|
    |**Обзор**|Конечная точка|Скопируйте конечную точку. Он выглядит примерно так `https://my-resource.cognitiveservices.azure.com/text/analytics/v3.0` :.|
    |**Ключи**|Ключ API|Скопируйте один из двух ключей. Это буквенно-цифровая строка, состоящая из 32 символов без пробелов или дефисов: <`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`>.|

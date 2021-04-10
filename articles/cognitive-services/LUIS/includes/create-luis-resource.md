---
title: Создание ресурсов LUIS
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/20/2020
ms.author: aahi
ms.openlocfilehash: ee7fd384a198c5eff672b14b6cb479aac26cfe54
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "95972528"
---
<a name="create-luis-resources"></a>

## <a name="create-luis-resources-in-the-azure-portal"></a>Создание ресурсов LUIS на портале Azure

1. Используйте [эту ссылку](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne), чтобы начать создание ресурсов LUIS на портале Azure.

1. Введите все необходимые параметры:

    |Имя|Назначение|
    |--|--|
    |Подписка | Подписка, для которой будет выставлен счет за ресурс.|
    |Группа ресурсов| Выбранное или созданное пользователем имя группы ресурсов. Группы ресурсов позволяют группировать ресурсы Azure для осуществления доступа и управления.|
    |Имя| Выбранное пользователем имя. Используется в качестве пользовательского поддомена для запросов конечной точки разработки и прогнозирования.|
    |Место разработки|Регион, связанный с вашей моделью.|
    |Ценовая категория для разработки|Определяет максимальное количество транзакций в секунду за месяц.|
    |Расположение прогноза|Регион, связанный с опубликованной средой выполнения конечной точки прогнозирования.|
    |Ценовая категория прогнозирования|Определяет максимальное количество транзакций в секунду за месяц.|

    > [!div class="mx-imgBorder"]
    > [![Снимок экран: вкладка "Основные сведения" в разделе "Создание".](../media/luis-how-to-azure-subscription/create-resource-in-azure-small.png)](../media/luis-how-to-azure-subscription/create-resource-in-azure-small.png#lightbox)

1. Выберите элемент **Просмотр и создание** и дождитесь создания ресурса.
1. После создания обоих ресурсов выберите на портале Azure новый ресурс для разработки. Затем выберите элемент **Keys and Endpoint** (Ключи и конечная точка), чтобы получить **URL-адрес конечной точки** и **ключ** для программной разработки.

> [!TIP]
> Чтобы использовать ресурсы, [назначьте их](../luis-how-to-azure-subscription.md#assign-an-authoring-resource-in-the-luis-portal-for-all-apps) на портале LUIS.

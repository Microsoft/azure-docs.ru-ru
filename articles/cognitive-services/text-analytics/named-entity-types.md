---
title: Поддерживаемые категории для распознавания именованных сущностей
titleSuffix: Azure Cognitive Services
description: Сведения о поддерживаемых категориях сущностей в API анализа текста.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 03/11/2021
ms.author: aahi
ms.openlocfilehash: 8b596a5e54c0b59c4c0b49aa5cdc4fd6477d46dc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599333"
---
# <a name="supported-entity-categories-in-the-text-analytics-api-v3"></a>Поддерживаемые категории сущностей в API анализа текста v3

Используйте эту статью, чтобы найти категории сущностей, которые могут быть возвращены с помощью средства [распознавания сущностей](how-tos/text-analytics-how-to-entity-linking.md) (NER). NER запускает прогнозную модель для определения и категоризации именованных сущностей из входного документа.

Также доступна предварительная версия NER версии 3.1, которая включает возможность обнаружения персональных `PII` данных () и сведений о работоспособности ( `PHI` ). Кроме того, перейдите на вкладку **работоспособность** , чтобы просмотреть список поддерживаемых категорий в анализ текста для работоспособности. 

Список типов, возвращаемых версией 2,1, можно найти в разделе [руководств по миграции](migration-guide.md?tabs=named-entity-recognition) .

## <a name="entity-categories"></a>Категории сущностей

#### <a name="general"></a>[Общие сведения](#tab/general)

[!INCLUDE [supported entity types - general](./includes/entity-types/general-entities.md)]

#### <a name="pii"></a>[ПЕРСОНАЛЬНЫЕ ДАННЫЕ](#tab/personal)

[!INCLUDE [supported entity types - personally identifying information](./includes/entity-types/personal-information-entities.md)]

#### <a name="health"></a>[Здравоохранение](#tab/health)

[!INCLUDE [biomedical entity types](./includes/entity-types/health-entities.md)]

***

## <a name="next-steps"></a>Дальнейшие действия

* [Как использовать распознавание именованных сущностей в Анализ текста](how-tos/text-analytics-how-to-entity-linking.md)

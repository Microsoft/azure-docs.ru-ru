---
title: Проверка экземпляра контейнера извлечение ключевых фраз
titleSuffix: Azure Cognitive Services
description: Узнайте, как проверить экземпляр контейнера извлечение ключевых фраз.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 5dca4828a5c1127133461ddf9fc06099fc176b68
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96009946"
---
### <a name="verify-the-key-phrase-extraction-container-instance"></a>Проверка экземпляра контейнера извлечение ключевых фраз

1. Перейдите на вкладку **Обзор** и скопируйте IP-адрес.
1. Откройте новую вкладку браузера и введите IP-адрес. Например, введите `http://<IP-address>:5000 (http://55.55.55.55:5000` ). Откроется домашняя страница контейнера, в которой вы узнаете, что контейнер выполняется.

    ![Просмотрите домашнюю страницу контейнера, чтобы убедиться, что она запущена](../media/how-tos/container-instance/swagger-docs-on-container.png)

1. Выберите ссылку **API службы Description** , чтобы открыть страницу Swagger контейнера.

1. Выберите любой из API-интерфейсов **POST** и нажмите кнопку **попробовать**. Отображаются параметры, включая следующий пример входных данных:

    ```json
    {
      "documents": [
        {
          "id": "1",
          "text": "Hello world"
        },
        {
          "id": "2",
          "text": "Bonjour tout le monde"
        },
        {
          "id": "3",
          "text": "La carretera estaba atascada. Había mucho tráfico el día de ayer."
        },
        {
          "id": "4",
          "text": ":) :( :D"
        }
      ]
    }
    ```

1. Замените входные данные следующим содержимым JSON:

    ```json
    {
      "documents": [
        {
          "language": "en",
          "id": "7",
          "text": "I was fortunate to attend the KubeCon Conference in Barcelona, it is one of the best conferences I have ever attended. Great people, great sessions and I thoroughly enjoyed it!"
        }
      ]
    }
    ```

1. Задайте для **шовстатс** значение `true` .

1. Выберите **выполнить** , чтобы определить тональности текста.

    Модель, упакованная в контейнер, формирует оценку в диапазоне от 0 до 1, где 0 является отрицательным, а 1 — положительным.

    Возвращаемый ответ JSON включает тональности для обновленного текстового ввода:

    ```json
    {
      "documents": [
        {
          "id": "7",
          "keyPhrases": [
            "Great people",
            "great sessions",
            "KubeCon Conference",
            "Barcelona",
            "best conferences"
          ],
          "statistics": {
            "charactersCount": 176,
            "transactionsCount": 1
          }
        }
      ],
      "errors": [],
      "statistics": {
        "documentsCount": 1,
        "validDocumentsCount": 1,
        "erroneousDocumentsCount": 0,
        "transactionsCount": 1
      }
    }
    ```

Теперь можно сопоставить документ `id` JSON полезных данных ответа с исходным документом полезных данных запроса `id` . Результирующий документ содержит `keyPhrases` массив, который содержит список ключевых фраз, извлеченных из соответствующего входного документа. Кроме того, `characterCount` `transactionCount` для каждого результирующего документа существуют различные статистические данные, такие как и.
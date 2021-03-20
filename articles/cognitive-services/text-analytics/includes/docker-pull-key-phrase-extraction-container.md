---
title: Извлечение DOCKER для контейнера извлечение ключевых фраз
titleSuffix: Azure Cognitive Services
description: Команда DOCKER Pull для контейнера извлечение ключевых фраз
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 02dde8a27b9687e58bf1a09c1a951f306937f0d6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90906075"
---
#### <a name="docker-pull-for-the-key-phrase-extraction-container"></a>Извлечение DOCKER для контейнера извлечение ключевых фраз

Используйте [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) команду, чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

Полное описание доступных тегов для Анализ текста контейнеров см. в разделе контейнер [Извлечение ключевых фраз](https://go.microsoft.com/fwlink/?linkid=2018757) в центре DOCKER.

```
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/keyphrase:latest
```

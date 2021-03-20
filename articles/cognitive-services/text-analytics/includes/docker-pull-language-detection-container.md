---
title: Извлечение DOCKER для контейнера распознавание языка
titleSuffix: Azure Cognitive Services
description: Команда DOCKER Pull для контейнера распознавание языка
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 95bcb4b424010f63ac1ee4eb02f9e4793647051a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90906007"
---
#### <a name="docker-pull-for-the-language-detection-container"></a>Извлечение DOCKER для контейнера распознавание языка

Используйте [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) команду, чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

Полное описание доступных тегов для Анализ текста контейнеров см. в разделе контейнер [распознавание языка](https://go.microsoft.com/fwlink/?linkid=2018759) в центре DOCKER.

```
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest
```

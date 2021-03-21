---
title: Установка речевых контейнеров
titleSuffix: Azure Cognitive Services
description: Сведения о параметрах конфигурации диаграммы Helm текста в речь.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 22168974ab8b285413b4fa6e947c05f65a73ae12
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96002302"
---
### <a name="text-to-speech-sub-chart-chartstexttospeech"></a>Преобразование текста в речь (во вспомогательной диаграмме: Charts/Тексттоспич)

Чтобы переопределить диаграмму "тег", добавьте префикс `textToSpeech.` для любого параметра, чтобы сделать его более конкретным. Например, он переопределит соответствующий параметр, например `textToSpeech.numberOfConcurrentRequest` Overrides `numberOfConcurrentRequest` .

|Параметр|Описание|По умолчанию|
| -- | -- | -- |
| `enabled` | Включена ли служба преобразования **текста в речь** . | `false` |
| `numberOfConcurrentRequest` | Количество одновременных запросов для службы преобразования **текста в речь** . Эта диаграмма автоматически вычисляет ресурсы ЦП и памяти на основе этого значения. | `2` |
| `optimizeForTurboMode`| Должна ли служба оптимизировать ввод текста через текстовые файлы. Если `true` значение равно, эта диаграмма будет выделять больше ресурсов ЦП для службы. | `false` |
| `image.registry`| Реестр образов DOCKER **для преобразования текста в речь** . | `containerpreview.azurecr.io` |
| `image.repository` | Репозиторий изображений DOCKER **для преобразования текста в речь** . | `microsoft/cognitive-services-text-to-speech` |
| `image.tag` | Тег изображения DOCKER **для преобразования текста в речь** . | `latest` |
| `image.pullSecrets` | Секреты образа для получения образа DOCKER для преобразования **текста в речь** . | |
| `image.pullByHash`| Указывает, извлекается ли образ DOCKER по хэшу. Если `true` `image.hash` значение — является обязательным. | `false` |
| `image.hash`| Хэш образа DOCKER **для преобразования текста в речь** . Используется, только если `image.pullByHash: true` .  | |
| `image.args.eula` (обязательный) | Указывает, что вы приняли условия лицензии. Единственное допустимое значение: `accept` | |
| `image.args.billing` (обязательный) | Значение URI конечной точки выставления счетов доступно на странице обзора речи портал Azure. | |
| `image.args.apikey` (обязательный) | Используется для отслеживания данных для выставлении счетов. ||
| `service.type` | Тип службы Kubernetes для службы преобразования **текста в речь** . Дополнительные сведения и проверка поддержки поставщика облачных служб см. в [инструкциях по типам службы Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) . | `LoadBalancer` |
| `service.port`|  Порт службы преобразования **текста в речь** . | `80` |
| `service.annotations` | Заметки преобразования **текста в речь** для метаданных службы. Аннотации — это пары "ключ — значение". <br>`annotations:`<br>&nbsp;&nbsp;`some/annotation1: value1`<br>&nbsp;&nbsp;`some/annotation2: value2` | |
| `service.autoScaler.enabled` | Включен ли [Горизонтальный Автомасштабирование Pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) . Если `true` значение равно, то `text-to-speech-autoscaler` будет развернут в кластере Kubernetes. | `true` |
| `service.podDisruption.enabled` | Включен ли [бюджет на прерывание Pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) . Если `true` значение равно, то `text-to-speech-poddisruptionbudget` будет развернут в кластере Kubernetes. | `true` |
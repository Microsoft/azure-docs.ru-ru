---
title: Установка речевых контейнеров
titleSuffix: Azure Cognitive Services
description: Подробные сведения о параметрах конфигурации диаграммы "речь-текст Helm".
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 05/05/2020
ms.author: trbye
ms.openlocfilehash: 85c4e0641e1989ddea6c8aa8b8a8895a966a5ddb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96002301"
---
### <a name="speech-to-text-sub-chart-chartsspeechtotext"></a>Преобразование речи в текст (диаграмма/Спичтотекст)

Чтобы переопределить диаграмму "тег", добавьте префикс `speechToText.` для любого параметра, чтобы сделать его более конкретным. Например, он переопределит соответствующий параметр, например `speechToText.numberOfConcurrentRequest` Overrides `numberOfConcurrentRequest` .

|Параметр|Описание|По умолчанию|
| -- | -- | -- |
| `enabled` | Включена ли служба **распознавания речи в текст** . | `false` |
| `numberOfConcurrentRequest` | Количество одновременных запросов для службы преобразования **речи в текст** . Эта диаграмма автоматически вычисляет ресурсы ЦП и памяти на основе этого значения. | `2` |
| `optimizeForAudioFile`| Должна ли служба оптимизировать входные данные с помощью звуковых файлов. Если `true` значение равно, эта диаграмма будет выделять больше ресурсов ЦП для службы. | `false` |
| `image.registry`| Реестр образа DOCKER **для преобразования речи в текст** . | `containerpreview.azurecr.io` |
| `image.repository` | Репозиторий образов DOCKER **для преобразования речи в текст** . | `microsoft/cognitive-services-speech-to-text` |
| `image.tag` | Тег образа DOCKER **для преобразования речи в текст** . | `latest` |
| `image.pullSecrets` | Секреты образа для получения образа DOCKER для преобразования **речи в текст** . | |
| `image.pullByHash`| Указывает, извлекается ли образ DOCKER по хэшу. Если `true` `image.hash` значение — является обязательным. | `false` |
| `image.hash`| Хэш образа DOCKER **для преобразования речи в текст** . Используется, только если `image.pullByHash: true` .  | |
| `image.args.eula` (обязательный) | Указывает, что вы приняли условия лицензии. Единственное допустимое значение: `accept` | |
| `image.args.billing` (обязательный) | Значение URI конечной точки выставления счетов доступно на странице обзора речи портал Azure. | |
| `image.args.apikey` (обязательный) | Используется для отслеживания данных для выставлении счетов. ||
| `service.type` | Тип службы Kubernetes для службы преобразования **речи в текст** . Дополнительные сведения и проверка поддержки поставщика облачных служб см. в [инструкциях по типам службы Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) . | `LoadBalancer` |
| `service.port`|  Порт службы преобразования **речи в текст** . | `80` |
| `service.annotations` | Заметки преобразования **речи в текст** для метаданных службы. Аннотации — это пары "ключ — значение". <br>`annotations:`<br>&nbsp;&nbsp;`some/annotation1: value1`<br>&nbsp;&nbsp;`some/annotation2: value2` | |
| `service.autoScaler.enabled` | Включен ли [Горизонтальный Автомасштабирование Pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) . Если `true` значение равно, то `speech-to-text-autoscaler` будет развернут в кластере Kubernetes. | `true` |
| `service.podDisruption.enabled` | Включен ли [бюджет на прерывание Pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) . Если `true` значение равно, то `speech-to-text-poddisruptionbudget` будет развернут в кластере Kubernetes. | `true` |

#### <a name="sentiment-analysis-sub-chart-chartsspeechtotext"></a>Анализ тональности (во вспомогательной диаграмме: диаграммы/Спичтотекст)

Начиная с v 2.2.0 контейнера преобразования речи в текст и v 0.2.0 диаграммы Helm, для анализа тональности используются следующие параметры с API анализа текста.

|Параметр|Описание|Значения|По умолчанию|
| --- | --- | --- | --- |
|`textanalytics.enabled`| Включена ли служба **Text-Analytics**| true/false| `false`|
|`textanalytics.image.registry`| Реестр образов DOCKER для **Text-Analytics**| допустимый реестр образов DOCKER| |
|`textanalytics.image.repository`| Репозиторий образов DOCKER для **Text-Analytics**| допустимый репозиторий образов DOCKER| |
|`textanalytics.image.tag`| Тег Image для **Text-Analytics** DOCKER| допустимый тег образа DOCKER| |
|`textanalytics.image.pullSecrets`| Секреты образа для получения образа DOCKER **Text-Analytics**| допустимое имя секреты| |
|`textanalytics.image.pullByHash`| Указывает, извлекает ли образ DOCKER по хэшу.  Если `yes` значение `image.hash` равно, необходимо также указать. Если `no` задано значение, задайте для него значение false. По умолчанию — `false`.| true/false| `false`|
|`textanalytics.image.hash`| Хэш образа DOCKER для **Text-Analytics** . Используйте его только с `image.pullByHash:true` .| допустимый хэш образа DOCKER | |
|`textanalytics.image.args.eula`| Один из обязательных аргументов контейнера **Text-Analytics** , который указывает, что вы приняли условия лицензии. Значение этого параметра должно быть следующим: `accept` .| `accept`, если вы хотите использовать контейнер | |
|`textanalytics.image.args.billing`| Один из обязательных аргументов контейнера **Text-Analytics** , который указывает URI конечной точки выставления счетов. Значение URI конечной точки выставления счетов доступно на странице обзора речи портал Azure.|допустимый URI конечной точки выставления счетов||
|`textanalytics.image.args.apikey`| Один из обязательных аргументов контейнера **Text-Analytics** , который используется для трассировки сведений о выставлении счетов.| допустимый apiKey||
|`textanalytics.cpuRequest`| Запрошенный ЦП для контейнера **Text-Analytics**| INT| `3000m`|
|`textanalytics.cpuLimit`| Ограниченный ЦП для контейнера **Text-Analytics**| | `8000m`|
|`textanalytics.memoryRequest`| Запрошенная память для контейнера **Text-Analytics**| | `3Gi`|
|`textanalytics.memoryLimit`| Ограниченная память для контейнера **Text-Analytics**| | `8Gi`|
|`textanalytics.service.sentimentURISuffix`| Суффикс URI анализа тональности, полный URI имеет формат "http:// `<service>` : `<port>` / `<sentimentURISuffix>` ". | | `text/analytics/v3.0-preview/sentiment`|
|`textanalytics.service.type`| Тип службы **Text-Analytics** в Kubernetes. См. раздел [типы служб Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) | допустимый тип службы Kubernetes | `LoadBalancer` |
|`textanalytics.service.port`| Порт службы **Text-Analytics**| INT| `50085`|
|`textanalytics.service.annotations`| Заметки, которые пользователи могут добавлять в метаданные службы **Text-Analytics** . Например:<br/> **Примечания**<br/>`   `**Some/annotation1: значение1**<br/>`  `**Some/annotation2: значение2** | заметки, по одному на каждую строку| |
|`textanalytics.serivce.autoScaler.enabled`| Включен ли [Горизонтальный Автомасштабирование Pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) . Если параметр включен, `text-analytics-autoscaler` будет развернут в кластере Kubernetes | true/false| `true`|
|`textanalytics.service.podDisruption.enabled`| Включен ли [бюджет на прерывание Pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) . Если параметр включен, `text-analytics-poddisruptionbudget` будет развернут в кластере Kubernetes| true/false| `true`|

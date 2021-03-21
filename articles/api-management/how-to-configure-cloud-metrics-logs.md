---
title: Настройка метрик и журналов облака для самостоятельно размещенного шлюза управления API Azure | Документация Майкрософт
description: Узнайте, как настроить метрики и журналы облака для самостоятельно размещенного шлюза управления API Azure.
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/30/2020
ms.author: apimpm
ms.openlocfilehash: c420c62e6f8f09a2b29398590cdb4ad410e5d296
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100574049"
---
# <a name="configure-cloud-metrics-and-logs-for-azure-api-management-self-hosted-gateway"></a>Настройка метрик и журналов облака для самостоятельно размещенного шлюза управления API Azure

В этой статье содержатся сведения о настройке метрик и журналов облака для [самостоятельно размещенного шлюза](./self-hosted-gateway-overview.md).

Автономный шлюз должен быть связан со службой управления API и требует исходящего подключения TCP/IP к Azure через порт 443. Шлюз использует исходящее подключение для отправки данных телеметрии в Azure, если это настроено. 

## <a name="metrics"></a>Метрики
По умолчанию самостоятельно размещенный шлюз создает ряд метрик с помощью [Azure Monitor](https://azure.microsoft.com/services/monitor/), аналогично управляемому шлюзу [в облаке](api-management-howto-use-azure-monitor.md). 

Эту функцию можно включить или отключить с помощью `telemetry.metrics.cloud` ключа в ConfigMap развертывания шлюза. Ниже приведена разбивка доступных конфигураций.

| Поле  | По умолчанию | Описание |
| ------------- | ------------- | ------------- |
| telemetry.metrics.cloud  | `true` | Включает ведение журнала с помощью Azure Monitor. Значение может быть равно `true` , `false` . |


Ниже приведен пример конфигурации.

```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: contoso-gateway-environment
    data:
        config.service.endpoint: "<contoso-gateway-management-endpoint>"
        telemetry.metrics.cloud: "true"
```

В настоящее время автономный шлюз создает следующие метрики с помощью Azure Monitor:

| Метрика  | Описание |
| ------------- | ------------- |
| Requests  | Количество запросов API за период |
| Длительность запросов шлюза | Количество миллисекунд с момента поступления запроса в шлюз до момента полной отправки ответа |
| Длительность внутренних запросов | Время в миллисекундах, затраченное на все операции ввода-вывода серверной части (подключение, отправка и получение байтов)  |

## <a name="logs"></a>Журналы

В настоящее время самостоятельно размещенный шлюз не отправляет [журналы диагностики](./api-management-howto-use-azure-monitor.md#activity-logs) в облако. Однако можно [настроить и сохранить журналы локально](how-to-configure-local-metrics-logs.md) , где развернут локальный шлюз. 

Если шлюз развернут в [службе Azure Kubernetes](https://azure.microsoft.com/services/kubernetes-service/), можно включить [Azure Monitor для контейнеров](../azure-monitor/containers/container-insights-overview.md) , чтобы получать журналы из контейнеров и просматривать их в log Analytics. 


## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о самостоятельно размещенном шлюзе см. в статье [Обзор самостоятельного размещения шлюза в службе управления API Azure](self-hosted-gateway-overview.md) .
* Сведения о [настройке и сохранении журналов в локальной](how-to-configure-local-metrics-logs.md) среде

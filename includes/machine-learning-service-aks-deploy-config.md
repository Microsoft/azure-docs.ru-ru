---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 03/16/2020
ms.author: larryfr
ms.openlocfilehash: e17f14d3acf8d74d1715d14fbd914ee536d29931
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102510802"
---
Записи в документе `deploymentconfig.json` соответствуют параметрам для [AksWebservice.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration). В следующей таблице описано сопоставление сущностей в документе JSON и параметров метода:

| Сущность JSON | Параметр метода | Описание: |
| ----- | ----- | ----- |
| `computeType` | Н/Д | Целевой объект вычисления. Для AKS нужно задать значение `aks`. |
| `autoScaler` | Н/Д | Содержит элементы конфигурации для автомасштабирования. См. сведения об autoScaler в таблице. |
| &emsp;&emsp;`autoscaleEnabled` | `autoscale_enabled` | Позволяет указать, нужно ли включить автомасштабирование для веб-службы. Если `numReplicas` = `0` — `True`, в противном случае — `False`. |
| &emsp;&emsp;`minReplicas` | `autoscale_min_replicas` | Минимальное число используемых контейнеров при автомасштабировании этой веб-службы. Значение по умолчанию — `1`. |
| &emsp;&emsp;`maxReplicas` | `autoscale_max_replicas` | Максимальное число используемых контейнеров при автомасштабировании этой веб-службы. Значение по умолчанию — `10`. |
| &emsp;&emsp;`refreshPeriodInSeconds` | `autoscale_refresh_seconds` | Частота попыток автомасштабирования этой веб-службы. Значение по умолчанию — `1`. |
| &emsp;&emsp;`targetUtilization` | `autoscale_target_utilization` | Целевой объем использования, который нужно пытаться поддерживать при автомасштабировании этой веб-службы (в процентах до 100). Значение по умолчанию — `70`. |
| `dataCollection` | Н/Д | Содержит элементы конфигурации для коллекции данных. |
| &emsp;&emsp;`storageEnabled` | `collect_model_data` | Позволяет указать, нужно ли включить коллекцию данных модели для веб-службы. Значение по умолчанию — `False`. |
| `authEnabled` | `auth_enabled` | Позволяет указать, нужно ли для веб-службы включить проверку подлинности на основе ключа. Нельзя указать значение `True` одновременно для `tokenAuthEnabled` и `authEnabled`. Значение по умолчанию — `True`. |
| `tokenAuthEnabled` | `token_auth_enabled` | Позволяет указать, нужно ли для веб-службы включить проверку подлинности на основе маркера. Нельзя указать значение `True` одновременно для `tokenAuthEnabled` и `authEnabled`. Значение по умолчанию — `False`. |
| `containerResourceRequirements` | Н/Д | Контейнер для сущностей ЦП и памяти. |
| &emsp;&emsp;`cpu` | `cpu_cores` | Количество ядер ЦП, выделяемых для этой веб-службы. Значение по умолчанию — `0.1`. |
| &emsp;&emsp;`memoryInGB` | `memory_gb` | Объем памяти (в ГБ), выделяемой для этой веб-службы. Значение по умолчанию — `0.5`. |
| `appInsightsEnabled` | `enable_app_insights` | Позволяет указать, нужно ли включить ведение журнала в Application Insights для веб-службы. Значение по умолчанию — `False`. |
| `scoringTimeoutMs` | `scoring_timeout_ms` | Принудительно применяемое время ожидания вызовов оценки веб-службы. Значение по умолчанию — `60000`. |
| `maxConcurrentRequestsPerContainer` | `replica_max_concurrent_requests` | Максимальное число одновременных запросов на узел для этой веб-службы. Значение по умолчанию — `1`. |
| `maxQueueWaitMs` | `max_request_wait_time` | Максимальное время, в течение которого запрос будет оставаться в очереди (в миллисекундах), прежде чем отобразится сообщение об ошибке 503. Значение по умолчанию — `500`. |
| `numReplicas` | `num_replicas` | Количество контейнеров, выделяемых для этой веб-службы. Нет значения по умолчанию. Если этот параметр не задан, средство автомасштабирования будет включено по умолчанию. |
| `keys` | Н/Д | Содержит элементы конфигурации для ключей. |
| &emsp;&emsp;`primaryKey` | `primary_key` | Первичный ключ проверки подлинности, используемый для этой веб-службы. |
| &emsp;&emsp;`secondaryKey` | `secondary_key` | Вторичный ключ проверки подлинности, используемый для этой веб-службы. |
| `gpuCores` | `gpu_cores` | Число ядер GPU (на реплику контейнера), выделяемых для этой веб-службы. Значение по умолчанию: 1. Поддерживаются только целочисленные значения. |
| `livenessProbeRequirements` | Н/Д | Содержит элементы конфигурации для требований пробы активности. |
| &emsp;&emsp;`periodSeconds` | `period_seconds` | Частота (в секундах) выполнения пробы активности. Значение по умолчанию — 10 секунд. Минимальное значение — 1. |
| &emsp;&emsp;`initialDelaySeconds` | `initial_delay_seconds` | Количество секунд после запуска контейнера до начала пробы активности. Значение по умолчанию — 310. |
| &emsp;&emsp;`timeoutSeconds` | `timeout_seconds` | Количество секунд, после которого истекает время ожидания пробы активности. Значение по умолчанию — 2 секунды. Минимальное значение — 1. |
| &emsp;&emsp;`successThreshold` | `success_threshold` | Минимальное число последовательных успешных попыток пробы активности после сбоя, после которых проба будет считаться успешной. По умолчанию равен 1. Минимальное значение — 1. |
| &emsp;&emsp;`failureThreshold` | `failure_threshold` | Количество попыток (failureThreshold), выполняемых Kubernetes, до их полного прекращения в случае запуска pod, но сбоя пробы активности. Значение по умолчанию — 3. Минимальное значение — 1. |
| `namespace` | `namespace` | Пространство имен Kubernetes, в котором развернута веб-служба. Не больше 63 буквенно-цифровых символов (a–z, 0–9), включая дефисы (-), в нижнем регистре. Первый и последний символы не могут быть дефисами. |

Следующий код JSON — это пример конфигурации развертывания для использования с CLI:

```json
{
    "computeType": "aks",
    "autoScaler":
    {
        "autoscaleEnabled": true,
        "minReplicas": 1,
        "maxReplicas": 3,
        "refreshPeriodInSeconds": 1,
        "targetUtilization": 70
    },
    "dataCollection":
    {
        "storageEnabled": true
    },
    "authEnabled": true,
    "containerResourceRequirements":
    {
        "cpu": 0.5,
        "memoryInGB": 1.0
    }
}
```
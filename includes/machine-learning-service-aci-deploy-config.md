---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 03/16/2020
ms.author: larryfr
ms.openlocfilehash: 4983c1e1e7f235fa7a5b748a0ce5b1c79176c849
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102510987"
---
Записи в документе `deploymentconfig.json` соответствуют параметрам для [AciWebservice.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.aci.aciservicedeploymentconfiguration). В следующей таблице описано сопоставление сущностей в документе JSON и параметров метода:

| Сущность JSON | Параметр метода | Описание: |
| ----- | ----- | ----- |
| `computeType` | Н/Д | Целевой объект вычисления. Для ACI нужно задать значение `ACI`. |
| `containerResourceRequirements` | Н/Д | Контейнер для сущностей ЦП и памяти. |
| &emsp;&emsp;`cpu` | `cpu_cores` | Количество ядер ЦП для выделения. Значение по умолчанию — `0.1`. |
| &emsp;&emsp;`memoryInGB` | `memory_gb` | Объем памяти (в ГБ), выделяемой для этой веб-службы. Значение по умолчанию — `0.5`. |
| `location` | `location` | Регион Azure для развертывания этой веб-службы. Если не указать, будет использоваться расположение рабочей области. Дополнительные сведения о доступных регионах можно просмотреть здесь: [регионы ACI](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=container-instances) |
| `authEnabled` | `auth_enabled` | Позволяет включить аутентификацию для этой веб-службы. Значение по умолчанию — False. |
| `sslEnabled` | `ssl_enabled` | Позволяет включить SSL для этой веб-службы. Значение по умолчанию — False. |
| `appInsightsEnabled` | `enable_app_insights` | Позволяет включить AppInsights для этой веб-службы. Значение по умолчанию — False. |
| `sslCertificate` | `ssl_cert_pem_file` | Файл сертификата, требуемый при включении SSL. |
| `sslKey` | `ssl_key_pem_file` | Файл ключа, требуемый при включении SSL. |
| `cname` | `ssl_cname` | Запись CNAME, требуемая при включении SSL. |
| `dnsNameLabel` | `dns_name_label` | Метка DNS-имени для конечной точки оценки. Если не указать, для конечной точки оценки будет создана уникальная метка DNS-имени. |

Следующий код JSON — это пример конфигурации развертывания для использования с CLI:

```json
{
    "computeType": "aci",
    "containerResourceRequirements":
    {
        "cpu": 0.5,
        "memoryInGB": 1.0
    },
    "authEnabled": true,
    "sslEnabled": false,
    "appInsightsEnabled": false
}
```
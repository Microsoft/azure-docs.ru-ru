---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/24/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: f561747ff530a92103cffc7b7b11a228c0b47311
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105036358"
---
## <a name="cmmc-level-3"></a>CMMC уровня 3

Дополнительные сведения о том, в какой мере доступные встроенные компоненты Политики Azure для всех служб Azure отвечают этому стандарту соответствия, см. в статье [Сведения о встроенной инициативе по соответствию требованиям стандарта CMMC уровня 3](../../../../articles/governance/policy/samples/cmmc-l3.md).
Дополнительные сведения об этом стандарте соответствия см. в документе о [ сертификации модели зрелости кибербезопасности (CMMC)](https://www.acq.osd.mil/cmmc/docs/CMMC_Model_Main_20200203.pdf).

|Домен |Идентификатор элемента управления |Заголовок элемента управления |Политика<br /><sub>(портал Azure)</sub> |Версия политики<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Управление доступом |AC.1.001 |Ограничение доступа к информационной системе для полномочных пользователей, процессов, действующих от имени полномочных пользователей, и устройств (включая другие информационные системы). |[Средства CORS не должны разрешать всем доменам доступ к API для FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Управление доступом |AC.1.002 |Ограничение доступа к информационной системе за исключением выполнения тех типов транзакций и функций, которые разрешено выполнять полномочным пользователям. |[Средства CORS не должны разрешать всем доменам доступ к API для FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Управление доступом |AC.2.016 |Управление потоком контролируемой несекретной информации в соответствии с утвержденными авторизациями |[Средства CORS не должны разрешать всем доменам доступ к API для FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Управление конфигурацией |CM.3.068 |Ограничение, отключение или предотвращение использования ненужных программ, функций, портов, протоколов и служб. |[Средства CORS не должны разрешать всем доменам доступ к API для FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |
|Защита системы и средств передачи данных |SC.3.177 |Применение шифрования, проверенного FIPS, для защиты конфиденциальности контролируемой несекретной информации. |[Служба Azure API для FHIR должна использовать для шифрования неактивных данных ключ, управляемый клиентом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F051cba44-2429-45b9-9649-46cec11c7119) |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_EnableByok_Audit.json) |
|Защита системы и средств передачи данных |SC.3.183 |Запрет на весь сетевой трафик по умолчанию и разрешение сетевого трафика в соответствии с исключениями (т. е. запрещен весь трафик, кроме исключений). |[Средства CORS не должны разрешать всем доменам доступ к API для FHIR](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fea8f8a-4169-495d-8307-30ec335f387d) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20for%20FHIR/HealthcareAPIs_RestrictCORSAccess_Audit.json) |


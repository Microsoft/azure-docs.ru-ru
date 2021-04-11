---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 3a580c056d6c286a6024e4c7e12271ae485c1194
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106095254"
---
## <a name="azure-security-benchmark"></a>Тестирование безопасности Azure

[Azure Security Benchmark](../../../../articles/security/benchmarks/overview.md) содержит рекомендации по обеспечению безопасности облачных решений в Azure. Сведения о том, как эта служба полностью сопоставляется с Azure Security Benchmark, см. [на этой странице](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Дополнительную информацию о том, как доступные встроенные компоненты Политики Azure для всех служб Azure сопоставляются с этим стандартом соответствия, см. в статье [Подробные сведения о встроенной инициативе о соответствии нормативным требованиям теста производительности системы безопасности Azure](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Домен |Идентификатор элемента управления |Заголовок элемента управления |Политика<br /><sub>(портал Azure)</sub> |Версия политики<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Ведение журналов и обнаружение угроз |LT-4 |Включение ведения журналов для ресурсов Azure |[В Azure Data Lake Storage должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb) |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |

## <a name="azure-security-benchmark-v1"></a>Тесты производительности системы безопасности Azure версии 1

[Azure Security Benchmark](../../../../articles/security/benchmarks/overview.md) содержит рекомендации по обеспечению безопасности облачных решений в Azure. Сведения о том, как эта служба полностью сопоставляется с Azure Security Benchmark, см. [на этой странице](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Дополнительную информацию о том, как доступные встроенные компоненты Политики Azure для всех служб Azure сопоставляются с этим стандартом соответствия, см. в статье [Подробные сведения о встроенной инициативе о соответствии нормативным требованиям теста производительности системы безопасности Azure](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Домен |Идентификатор элемента управления |Заголовок элемента управления |Политика<br /><sub>(портал Azure)</sub> |Версия политики<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Ведение журналов и мониторинг |2.3 |Включение журналов аудита для ресурсов Azure |[В Azure Data Lake Storage должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb) |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |

## <a name="cis-microsoft-azure-foundations-benchmark-130"></a>CIS Microsoft Azure Foundations Benchmark 1.3.0

Дополнительные сведения о том, как доступные встроенные компоненты Политики Azure для всех служб Azure сопоставляются с этим стандартом соответствия, см. в статье [Обеспечение соответствия нормативным требованиям для Политики Azure — CIS Microsoft Azure Foundations Benchmark 1.3.0](../../../../articles/governance/policy/samples/cis-azure-1-3-0.md).
Дополнительные сведения об этом стандарте соответствия требованиям см. в статье [Тест производительности CIS для платформ Microsoft Azure](https://www.cisecurity.org/benchmark/azure/).

|Домен |Идентификатор элемента управления |Заголовок элемента управления |Политика<br /><sub>(портал Azure)</sub> |Версия политики<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Ведение журналов и мониторинг |5,3 |Обеспечение включения журналов диагностики для всех служб, которые их поддерживают. |[В Azure Data Lake Storage должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb) |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |

## <a name="cmmc-level-3"></a>CMMC уровня 3

Дополнительные сведения о том, в какой мере доступные встроенные компоненты Политики Azure для всех служб Azure отвечают этому стандарту соответствия, см. в статье [Сведения о встроенной инициативе по соответствию требованиям стандарта CMMC уровня 3](../../../../articles/governance/policy/samples/cmmc-l3.md).
Дополнительные сведения об этом стандарте соответствия см. в документе о [ сертификации модели зрелости кибербезопасности (CMMC)](https://www.acq.osd.mil/cmmc/docs/CMMC_Model_Main_20200203.pdf).

|Домен |Идентификатор элемента управления |Заголовок элемента управления |Политика<br /><sub>(портал Azure)</sub> |Версия политики<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Защита системы и средств передачи данных |SC.3.177 |Применение шифрования, проверенного FIPS, для защиты конфиденциальности контролируемой несекретной информации. |[Требовать шифрование учетных записей Data Lake Store](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7ff3161-0087-490a-9ad9-ad6217f4f43a) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStoreEncryption_Deny.json) |
|Защита системы и средств передачи данных |SC.3.191 |Защита конфиденциальности неактивной контролируемой несекретной информации |[Требовать шифрование учетных записей Data Lake Store](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7ff3161-0087-490a-9ad9-ad6217f4f43a) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStoreEncryption_Deny.json) |

## <a name="hipaa-hitrust-92"></a>HIPAA HITRUST 9.2

Дополнительную информацию о том, как доступные встроенные компоненты Политики Azure для всех служб Azure сопоставляются с этим стандартом соответствия, см. в статье со [сведениями о встроенной инициативе по обеспечению соответствия стандарту HIPAA HITRUST 9.2](../../../../articles/governance/policy/samples/hipaa-hitrust-9-2.md).
Дополнительные сведения об этом стандарте соответствия см. на странице описания [HIPAA HITRUST 9.2](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html).

|Домен |Идентификатор элемента управления |Заголовок элемента управления |Политика<br /><sub>(портал Azure)</sub> |Версия политики<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Ведение журнала аудита |1202.09aa1System.1 — 09.aa |Защищенная запись аудита создается для всех действий в системе (создание, чтение, обновление, удаление), связанных с соответствующей информацией. |[В Azure Data Lake Storage должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb) |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |
|Управление съемными носителями |0304.09o3Organizational.1 — 09.o |Организация ограничивает использование съемных носителей с возможностью записи и персональных съемных носителей в системах организации. |[Требовать шифрование учетных записей Data Lake Store](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7ff3161-0087-490a-9ad9-ad6217f4f43a) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStoreEncryption_Deny.json) |


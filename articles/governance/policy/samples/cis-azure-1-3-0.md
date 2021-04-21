---
title: Сведения о соответствии нормативным требованиям теста производительности CIS для платформ Microsoft Azure 1.3.0
description: Подробные сведения о встроенной инициативе о соответствии нормативным требованиям теста производительности CIS для платформ Microsoft Azure 1.3.0. Каждый элемент управления сопоставляется с одним или несколькими определениями Политики Azure, которые помогают выполнять оценку.
ms.date: 04/14/2021
ms.topic: sample
ms.custom: generated
ms.openlocfilehash: 3dc7578af93f36ba8f0400099bb21cdabdcffb70
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107497733"
---
# <a name="details-of-the-cis-microsoft-azure-foundations-benchmark-130-regulatory-compliance-built-in-initiative"></a>Подробные сведения о встроенной инициативе о соответствии нормативным требованиям теста производительности CIS для платформ Microsoft Azure 1.3.0.

В следующей статье подробно описано, как определение встроенной инициативы о соответствии нормативным требованиям Политики Azure сопоставляется с **областями соответствия нормативным требованиям** и **элементами управления** в тестах CIS оценки безопасности для платформ Microsoft Azure 1.3.0.
Дополнительные сведения об этом стандарте соответствия требованиям см. в статье [Тест производительности CIS для платформ Microsoft Azure 1.3.0](https://www.cisecurity.org/benchmark/azure/). Сведения о термине _Ответственность_ см. в статьях [Определение службы "Политика Azure"](../concepts/definition-structure.md#type) и [Разделение ответственности в облаке](../../../security/fundamentals/shared-responsibility.md).

Ниже приведены сопоставления, соответствующие элементам управления **CIS Microsoft Azure Foundations Benchmark 1.3.0**. Используйте панель навигации справа для перехода непосредственно к конкретной **области соответствия нормативным требованиям**. Многие элементы управления реализуются с помощью определения инициативы [Политики Azure](../overview.md). Чтобы просмотреть полное определение инициативы, откройте раздел **Политика** на портале Azure и перейдите на страницу **Определения**.
Затем найдите и выберите определение встроенной инициативы о соответствии требованиям **Azure Foundations Benchmark CIS для платформ Microsoft Azure версии 1.3.0**.

> [!IMPORTANT]
> Каждый элемент управления ниже связан с одним или несколькими определениями [Политики Azure](../overview.md).
> Такие политики помогут вам в [оценке соответствия](../how-to/get-compliance-data.md) с помощью элементов управления, но часто полное или точное соответствие между элементом управления и одной или несколькими политиками отсутствует. Поэтому состояние **Совместимый** в Политике Azure применимо только к самим определениям политики и не означает полное соответствие всем требованиям элемента управления. Кроме того, стандарт соответствия включает элементы управления, на которые сейчас не распространяются определения Политики Azure. Следовательно, сведения о соответствии в Политике Azure — это только частичное представление общего состояния соответствия. Связи между областями соответствия нормативным требованиям, элементами управления и определениями Политики Azure для этого стандарта соответствия со временем могут меняться. Историю изменений можно просмотреть на странице [журнала фиксаций в GitHub](https://github.com/Azure/azure-policy/commits/master/built-in-policies/policySetDefinitions/Regulatory%20Compliance/CISv1_3_0.json).

## <a name="identity-and-access-management"></a>Управление удостоверениями и доступом

### <a name="ensure-that-multi-factor-authentication-is-enabled-for-all-privileged-users"></a>Обеспечение того, что многофакторная проверка подлинности включена для всех привилегированных пользователей

**Идентификатор**. CIS Azure 1.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на запись в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |Следует включить MFA для всех учетных записей подписки с разрешениями на запись, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |
|[Для учетных записей с разрешениями владельца в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |Следует включить MFA для всех учетных записей подписки с разрешениями владельца, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |

### <a name="ensure-that-multi-factor-authentication-is-enabled-for-all-non-privileged-users"></a>Обеспечение того, что многофакторная проверка подлинности включена для всех непривилегированных пользователей

**Идентификатор**. CIS Azure 1.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на чтение в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |Следует включить многофакторную проверку подлинности (MFA) для всех учетных записей подписки с разрешениями на чтение, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |

### <a name="ensure-guest-users-are-reviewed-on-a-monthly-basis"></a>Ежемесячная проверка гостевых пользователей

**Идентификатор**. CIS Azure 1.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Внешние учетные записи с разрешениями владельца должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8456c1c-aa66-4dfb-861a-25d127b775c9) |Внешние учетные записи с разрешениями владельца должны быть удалены из подписки, чтобы предотвратить неотслеживаемый доступ. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWithOwnerPermissions_Audit.json) |
|[Внешние учетные записи с разрешениями на чтение должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5f76cf89-fbf2-47fd-a3f4-b891fa780b60) |Внешние учетные записи с правами на чтение должны быть удалены из подписки, чтобы предотвратить неотслеживаемый доступ. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsReadPermissions_Audit.json) |
|[Внешние учетные записи с разрешениями на запись должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c607a2e-c700-4744-8254-d77e7c9eb5e4) |Внешние учетные записи с правами на запись должны быть удалены из подписки, чтобы предотвратить неотслеживаемый доступ. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWritePermissions_Audit.json) |

### <a name="ensure-that-no-custom-subscription-owner-roles-are-created"></a>Обеспечение того, что роли владельца пользовательской подписки не созданы

**Идентификатор**. CIS Azure 1.21 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Роли владельца пользовательской подписки не должны существовать](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |Эта политика позволяет исключить наличие ролей владельца пользовательской подписки. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |

## <a name="security-center"></a>Центр безопасности

### <a name="ensure-that-azure-defender-is-set-to-on-for-servers"></a>Включение Azure Defender для серверов

**Идентификатор**. CIS Azure 2.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для серверов должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4da35fc9-c9e7-4960-aec9-797fe7d9051d) |Azure Defender для серверов обеспечивает защиту рабочих нагрузок сервера в реальном времени и создает рекомендации по улучшению безопасности, а также оповещения о подозрительных действиях. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnVM_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-app-service"></a>Включение Azure Defender для Службы приложений

**Идентификатор**. CIS Azure 2.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для Службы приложений должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2913021d-f2fd-4f3d-b958-22354e2bdbcb) |Azure Defender для Службы приложений использует масштаб облака, а также видимость, характерную для Azure как поставщика облачных служб, для отслеживания распространенных атак на веб-приложения. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnAppServices_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-azure-sql-database-servers"></a>Включение Azure Defender для серверов базы данных SQL Azure

**Идентификатор**. CIS Azure 2.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для серверов Базы данных SQL Azure должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7fe3b40f-802b-4cdd-8bd4-fd799c948cc2) |Azure Defender для SQL предоставляет возможности для выявления и устранения потенциальных уязвимостей баз данных, обнаружения необычных действий, которые могут представлять угрозу в базах данных SQL, а также для определения и классификации конфиденциальных данных. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedDataSecurityOnSqlServers_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-sql-servers-on-machines"></a>Включение Azure Defender для серверов SQL Server на компьютерах.

**Идентификатор**. CIS Azure 2.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для серверов SQL на компьютерах должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6581d072-105e-4418-827f-bd446d56421b) |Azure Defender для SQL предоставляет возможности для выявления и устранения потенциальных уязвимостей баз данных, обнаружения необычных действий, которые могут представлять угрозу в базах данных SQL, а также для определения и классификации конфиденциальных данных. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedDataSecurityOnSqlServerVirtualMachines_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-storage"></a>Включение Azure Defender для службы хранилища

**Идентификатор**. CIS Azure 2.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для службы хранилища должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F308fbb08-4ab8-4e67-9b29-592e93fb94fa) |Azure Defender для службы хранилища обнаруживает необычные и потенциально опасные попытки доступа к учетным записям хранения или их использования. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnStorageAccounts_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-kubernetes"></a>Включение Azure Defender для Kubernetes

**Идентификатор**. CIS Azure 2.6 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для Kubernetes должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F523b5cd1-3e23-492f-a539-13118b6d1e3a) |Azure Defender для Kubernetes обеспечивает защиту контейнерных сред от угроз в реальном времени и создает оповещения о подозрительных действиях. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnKubernetesService_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-container-registries"></a>Включение Azure Defender для реестров контейнеров

**Идентификатор**. CIS Azure 2.7 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для реестров контейнеров должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc25d9a16-bc35-4e15-a7e5-9db606bf9ed4) |Azure Defender для реестров контейнеров выполняет поиск уязвимостей всех образов, отправленных за последние 30 дней, помещенных в реестр или импортированных, и выводит подробные результаты для каждого образа. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnContainerRegistry_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-key-vault"></a>Включение Azure Defender для Key Vault

**Идентификатор**. CIS Azure 2.8 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Defender для Key Vault должен быть включен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e6763cc-5078-4e64-889d-ff4d9a839047) |Azure Defender для Key Vault обеспечивает дополнительный уровень защиты в виде механизма обнаружения угроз, который позволяет выявить необычные и потенциально опасные попытки получения и использования учетных записей хранилища ключей. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnKeyVaults_Audit.json) |

### <a name="ensure-that-automatic-provisioning-of-monitoring-agent-is-set-to-on"></a>Обеспечение того, что включен параметр "Автоматическая подготовка агента мониторинга"

**Идентификатор**. CIS Azure 2.11 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В подписке должна быть включена автоматическая подготовка агента Log Analytics](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |Чтобы отслеживать уязвимости и угрозы безопасности, Центр безопасности Azure собирает данные из виртуальных машин Azure. Для сбора данных используется агент Log Analytics (ранее — Microsoft Monitoring Agent, или MMA), который считывает разные конфигурации, связанные с безопасностью, и журналы событий с компьютера, а также копирует данные в рабочую область Log Analytics для анализа. Мы рекомендуем включить автоматическую подготовку для автоматического развертывания агента на всех поддерживаемых виртуальных машинах Azure, включая те, которые созданы недавно. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |

### <a name="ensure-additional-email-addresses-is-configured-with-a-security-contact-email"></a>Настройка дополнительных адресов электронной почты с использованием адреса электронной почты контактного лица по вопросам безопасности

**Идентификатор**. CIS Azure 2.13 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписки должны содержать адрес электронной почты контактного лица по вопросам безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f4f78b8-e367-4b10-a341-d9a4ad5cf1c7) |Чтобы настроить отправку уведомлений соответствующим специалистам о потенциальных нарушениях безопасности в одной из подписок в организации, включите для контактного лица по вопросам безопасности отправку уведомлений по электронной почте в Центре безопасности. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Security_contact_email.json) |

### <a name="ensure-that-notify-about-alerts-with-the-following-severity-is-set-to-high"></a>Настройка высокого уровня серьезности для уведомлений о предупреждениях

**Идентификатор**. CIS Azure 2.14 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для оповещений высокого уровня серьезности нужно включить уведомление по электронной почте](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6e2593d9-add6-4083-9c9b-4b7d2188c899) |Чтобы настроить отправку уведомлений соответствующим специалистам о потенциальных нарушениях безопасности в одной из подписок в организации, включите уведомления по электронной почте для отправки оповещений с высоким уровнем серьезности в Центре безопасности. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Email_notification.json) |

## <a name="storage-accounts"></a>Учетные записи хранения

### <a name="ensure-that-secure-transfer-required-is-set-to-enabled"></a>Обеспечение того, что для параметра "Требуется безопасное перемещение" задано значение "Включено"

**Идентификатор**. CIS Azure 3.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |

### <a name="ensure-that-public-access-level-is-set-to-private-for-blob-containers"></a>Обеспечение того, что для параметра "Уровень общего доступа" задано значение "Частный" для контейнеров BLOB-объектов

**Идентификатор**. CIS Azure 3.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Открытый доступ к учетной записи хранения должен быть запрещен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4fa4b6c0-31ca-4c0d-b10d-24b96f62a751) |Анонимный общий доступ на чтение к контейнерам и большим двоичным объектам в службе хранилища Azure — это удобный способ обмена данными, но он может представлять угрозу безопасности. Чтобы предотвратить утечки данных, возникающие в результате нежелательного анонимного доступа, корпорация Майкрософт рекомендует запретить общий доступ к учетной записи хранения, если он не требуется для вашего сценария. |Audit, Deny, Disabled |[2.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/ASC_Storage_DisallowPublicBlobAccess_Audit.json) |

### <a name="ensure-default-network-access-rule-for-storage-accounts-is-set-to-deny"></a>Обеспечение того, что в правиле сетевого доступа по умолчанию для учетных записей хранения задано значение "Отклонить"

**Идентификатор**. CIS Azure 3.6 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Учетные записи хранения должны ограничивать доступ к сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34c877ad-507e-4c82-993e-3452a6e0ad3c) |Сетевой доступ к учетным записям хранения должен быть ограниченным. Настройте правила сети так, чтобы учетная запись хранения была доступна только приложениям из разрешенных сетей. Чтобы разрешить подключения от конкретных локальных клиентов и интернет-клиентов, вы можете открыть доступ для трафика из конкретных виртуальных сетей Azure или определенных диапазонов общедоступных IP-адресов. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_NetworkAcls_Audit.json) |
|[Учетные записи хранения должны ограничивать доступ к сети с помощью правил виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2a1a9cdf-e04d-429a-8416-3bfb72a1b26f) |Защитите свои учетные записи хранения от потенциальных угроз с помощью правил виртуальной сети, используемых в качестве предпочтительного метода вместо фильтрации по IP-адресу. Отключение фильтрации по IP-адресу запрещает общедоступным IP-адресам доступ к учетным записям хранения. |Audit, Deny, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageAccountOnlyVnetRulesEnabled_Audit.json) |

### <a name="ensure-trusted-microsoft-services-is-enabled-for-storage-account-access"></a>Убедитесь что для доступа к учетной записи хранения включен параметр "Доверенные службы Майкрософт"

**Идентификатор**. CIS Azure 3.7 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Учетные записи хранения должны разрешать доступ из доверенных служб Майкрософт](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc9d007d0-c057-4772-b18c-01e546713bcd) |Некоторые службы Майкрософт, взаимодействующие с учетными записями хранения, работают из сетей, которым нельзя предоставить доступ через сетевые правила. Чтобы помочь таким службам работать как нужно, разрешите доверенным службам Майкрософт обходить сетевые правила. Эти службы будут использовать строгую проверку подлинности для доступа к учетной записи хранения. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageAccess_TrustedMicrosoftServices_Audit.json) |

### <a name="ensure-storage-for-critical-data-are-encrypted-with-customer-managed-key"></a>Обеспечение того, что хранилище важных данных зашифровано с помощью ключа, управляемого клиентом

**Идентификатор**. CIS Azure 3.9 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В учетных записях хранения должен использоваться управляемый клиентом ключ шифрования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6fac406b-40ca-413b-bf8e-0bf964659c25) |Обеспечьте большую гибкость своей учетной записи хранения с помощью управляемых клиентом ключей. Указанный ключ CMK используется для защиты доступа к ключу, который шифрует данные, и управления этим доступом. Использование управляемых клиентом ключей предоставляет дополнительные возможности для управления сменой ключей шифрования или криптографического стирания данных. |Audit, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageAccountCustomerManagedKeyEnabled_Audit.json) |

## <a name="database-services"></a>Службы баз данных

### <a name="ensure-that-auditing-is-set-to-on"></a>Обеспечение того, что для параметра "Аудит" задано значение "Вкл."

**Идентификатор**. CIS Azure 4.1.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить аудит на сервере SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa6fb4358-5bf4-4ad7-ba82-2cd2f41ce5e9) |Для отслеживания действий во всех базах данных на сервере и сохранения их в журнал аудита должен быть включен аудит вашего SQL Server. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_Audit.json) |

### <a name="ensure-that-data-encryption-is-set-to-on-on-a-sql-database"></a>Обеспечение того, что для параметра "Шифрование данных" в Базе данных SQL установлено значение "Вкл."

**Идентификатор**. CIS Azure 4.1.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В базах данных SQL должно применяться прозрачное шифрование данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17k78e20-9358-41c9-923c-fb736d382a12) |Для защиты неактивных данных и обеспечения соответствия требованиям должно быть включено прозрачное шифрование данных. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlDBEncryption_Audit.json) |

### <a name="ensure-that-auditing-retention-is-greater-than-90-days"></a>Обеспечение того, что задан период хранения данных аудита более 90 дней

**Идентификатор**. CIS Azure 4.1.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов SQL Server с аудитом в назначении учетной записи хранения следует настроить срок хранения длительностью 90 дней или более](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F89099bee-89e0-4b26-a5f4-165451757743) |Для исследования инцидентов мы рекомендуем задать срок хранения данных аудита SQL Server в назначении учетной записи хранения длительностью как минимум 90 дней. Убедитесь, что соблюдаются необходимые правила хранения для регионов, в которых вы работаете. Иногда это необходимо для обеспечения соответствия нормативным стандартам. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditingRetentionDays_Audit.json) |

### <a name="ensure-that-advanced-threat-protection-atp-on-a-sql-server-is-set-to-enabled"></a>Обеспечение того, что для службы "Расширенная защита от угроз" на SQL сервере установлено значение "Включено"

**Идентификатор**. CIS Azure 4.2.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемом экземпляре SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Проверка всех Управляемых экземпляров SQL без Расширенной защиты данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[На серверах SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Аудит серверов SQL без Расширенной защиты данных |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |

### <a name="ensure-that-vulnerability-assessment-va-is-enabled-on-a-sql-server-by-setting-a-storage-account"></a>Обеспечение того, что оценка уязвимостей включена на сервере SQL путем задания учетной записи хранения

**Идентификатор**. CIS Azure 4.2.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемых экземплярах SQL должна быть включена оценка уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1b7aa243-30e4-4c9e-bca8-d0d3022b634a) |Проверка каждого Управляемого экземпляра SQL с отключенной регулярной оценкой уязвимостей. Решение "Оценка уязвимостей" может обнаруживать, отслеживать и помогать исправлять потенциальные уязвимости баз данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnManagedInstance_Audit.json) |
|[На серверах SQL Server должна быть включена оценка уязвимости](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef2a8f2a-b3d9-49cd-a8a8-9a3aaaf647d9) |Аудит серверов Azure SQL, у которых отключена регулярная оценка уязвимостей. Решение "Оценка уязвимостей" может обнаруживать, отслеживать и помогать исправлять потенциальные уязвимости баз данных. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnServer_Audit.json) |

### <a name="ensure-that-va-setting-send-scan-reports-to-is-configured-for-a-sql-server"></a>Обеспечение того, что параметр оценки уязвимости "Куда отправлять отчеты о проверке" настроен для сервера SQL

**Идентификатор**. CIS Azure 4.2.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Параметры оценки уязвимостей для сервера SQL должны включать адрес электронной почты для получения отчетов о проверке](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057d6cfe-9c4f-4a6d-bc60-14420ea1f1a9) |Проверка указания адреса электронной почты в поле "Куда отправлять отчеты о проверке" в параметрах оценки уязвимостей. На этот адрес приходит сводка результатов при выполнении проверки на серверах SQL. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_VulnerabilityAssessmentEmails_Audit.json) |

### <a name="ensure-enforce-ssl-connection-is-set-to-enabled-for-postgresql-database-server"></a>Обеспечение того, что параметр "Принудительно использовать SSL-соединение" включен для сервера баз данных PostgreSQL

**Идентификатор**. CIS Azure 4.3.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |

### <a name="ensure-enforce-ssl-connection-is-set-to-enabled-for-mysql-database-server"></a>Обеспечение того, что параметр "Принудительно использовать SSL-соединение" включен для сервера баз данных MySQL

**Идентификатор**. CIS Azure 4.3.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |

### <a name="ensure-server-parameter-log_checkpoints-is-set-to-on-for-postgresql-database-server"></a>Обеспечение того, что параметр log_checkpoints включен для сервера базы данных PostgreSQL

**Идентификатор**. CIS Azure 4.3.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На серверах баз данных PostgreSQL должны быть включены контрольные точки журнала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e43d) |Эта политика помогает выполнить аудит наличия в вашей среде баз данных PostgreSQL, в которых не включен параметр log_checkpoints. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogCheckpoint_Audit.json) |

### <a name="ensure-server-parameter-log_connections-is-set-to-on-for-postgresql-database-server"></a>Обеспечение того, что параметр log_connections включен для сервера базы данных PostgreSQL

**Идентификатор**. CIS Azure 4.3.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На серверах баз данных PostgreSQL должны быть включены журналы подключений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e442) |Эта политика помогает выполнить аудит наличия в вашей среде баз данных PostgreSQL, для которых не включен параметр log_connections. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogConnections_Audit.json) |

### <a name="ensure-server-parameter-log_disconnections-is-set-to-on-for-postgresql-database-server"></a>Обеспечение того, что параметр log_disconnections включен для сервера базы данных PostgreSQL

**Идентификатор**. CIS Azure 4.3.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На серверах баз данных PostgreSQL должны быть включены журналы отключений.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e446) |Эта политика помогает выполнить аудит наличия в вашей среде баз данных PostgreSQL, для которых не включен параметр log_disconnections. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogDisconnections_Audit.json) |

### <a name="ensure-server-parameter-connection_throttling-is-set-to-on-for-postgresql-database-server"></a>Обеспечение того, что параметр connection_throttling включен для сервера базы данных PostgreSQL

**Идентификатор**. CIS Azure 4.3.6 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных PostgreSQL должно быть включено регулирование подключения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5345bb39-67dc-4960-a1bf-427e16b9a0bd) |Эта политика помогает выполнить аудит наличия в вашей среде баз данных PostgreSQL, для которых не включено регулирование подключения. Этот параметр обеспечивает временное регулирование подключений по IP-адресу при слишком большом числе неудачных попыток входа с паролем. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_ConnectionThrottling_Enabled_Audit.json) |

### <a name="ensure-that-azure-active-directory-admin-is-configured"></a>Обеспечение того, что администратор Azure Active Directory настроен

**Идентификатор**. CIS Azure 4.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов SQL должен быть подготовлен администратор Azure Active Directory](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1f314764-cb73-4fc9-b863-8eca98ac36e9) |Аудит подготовки администратора AAD для включения в SQL Server поддержки проверки подлинности AAD. Эта проверка подлинности упрощает контроль разрешений и обеспечивает централизованное управление удостоверениями пользователей баз данных и других служб Майкрософт. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SQL_DB_AuditServerADAdmins_Audit.json) |

### <a name="ensure-sql-servers-tde-protector-is-encrypted-with-customer-managed-key"></a>Обеспечение того, что предохранитель TDE сервера SQL зашифрован с помощью ключа, управляемого клиентом

**Идентификатор**. CIS Azure 4.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Управляемые экземпляры SQL должны использовать ключи, управляемые клиентом, для шифрования неактивных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F048248b0-55cd-46da-b1ff-39efd52db260) |Реализация прозрачного шифрования данных (TDE) с вашим собственным ключом обеспечивает повышенную прозрачность и контроль над предохранителем TDE, а также дополнительную защиту за счет использования внешней службы с поддержкой HSM и содействие разделению обязанностей. Эта рекомендация относится к организациям, имеющим связанное требование по соответствию. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |
|[Серверы SQL должны использовать управляемые клиентом ключи для шифрования неактивных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d134df8-db83-46fb-ad72-fe0c9428c8dd) |Реализация прозрачного шифрования данных (TDE) с вашим ключом обеспечивает повышенную прозрачность и контроль над предохранителем TDE, а также дополнительную защиту за счет использования внешней службы с поддержкой HSM и содействие разделению обязанностей. Эта рекомендация относится к организациям, имеющим связанное требование по соответствию. |AuditIfNotExists, Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

### <a name="ensure-the-storage-container-storing-the-activity-logs-is-not-publicly-accessible"></a>Обеспечение того, что хранение журналов действий в контейнере хранилища не является общедоступным

**Идентификатор**. CIS Azure 5.1.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Открытый доступ к учетной записи хранения должен быть запрещен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4fa4b6c0-31ca-4c0d-b10d-24b96f62a751) |Анонимный общий доступ на чтение к контейнерам и большим двоичным объектам в службе хранилища Azure — это удобный способ обмена данными, но он может представлять угрозу безопасности. Чтобы предотвратить утечки данных, возникающие в результате нежелательного анонимного доступа, корпорация Майкрософт рекомендует запретить общий доступ к учетной записи хранения, если он не требуется для вашего сценария. |Audit, Deny, Disabled |[2.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/ASC_Storage_DisallowPublicBlobAccess_Audit.json) |

### <a name="ensure-the-storage-account-containing-the-container-with-activity-logs-is-encrypted-with-byok-use-your-own-key"></a>Обеспечение того, что учетная запись хранения, содержащая контейнер с журналами действий, зашифрована с помощью BYOK

**Идентификатор**. CIS Azure 5.1.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Учетная запись хранения, содержащая контейнер с журналами действий, должна быть зашифрована с помощью BYOK](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffbb99e8e-e444-4da0-9ff1-75c92f5a85b2) |Эта политика проверяет, зашифрована ли учетная запись хранения, содержащая контейнер с журналами действий, с помощью BYOK. Эта политика применяется только в том случае, если проект реализован так, что учетная запись хранения находится в той же подписке, что и журналы действий. Дополнительные сведения о шифровании неактивных данных в службе хранилища Azure см. на странице [https://aka.ms/azurestoragebyok](https://aka.ms/azurestoragebyok).  |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_StorageAccountBYOK_Audit.json) |

### <a name="ensure-that-logging-for-azure-keyvault-is-enabled"></a>Обеспечение того, что для параметр ведения журнала для Azure KeyVault включен

**Идентификатор**. CIS Azure 5.1.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Key Vault должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf820ca0-f99e-4f3e-84fb-66e913812d21) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для анализа инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_AuditDiagnosticLog_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-policy-assignment"></a>Обеспечение того, что для операции создания назначения политики существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для определенных операций политики должно существовать оповещение журнала действий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc5447c04-a4d7-4ba8-a263-c9ee321a6858) |Эта политика выполняет аудит определенных операций политики без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_PolicyOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-policy-assignment"></a>Включение оповещения журнала действий для операции удаления назначения политики

**Идентификатор**. CIS Azure 5.2.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для определенных операций политики должно существовать оповещение журнала действий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc5447c04-a4d7-4ba8-a263-c9ee321a6858) |Эта политика выполняет аудит определенных операций политики без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_PolicyOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-network-security-group"></a>Обеспечение того, что для операции создания или обновления группы безопасности сети существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для выполнения определенных административных операций должно существовать оповещение журнала действий.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Эта политика выполняет аудит определенных административных операций без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-network-security-group"></a>Обеспечение того, что для операции удаления группы безопасности сети существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для выполнения определенных административных операций должно существовать оповещение журнала действий.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Эта политика выполняет аудит определенных административных операций без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-network-security-group-rule"></a>Обеспечение того, что для операции создания или обновления правила группы безопасности сети существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для выполнения определенных административных операций должно существовать оповещение журнала действий.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Эта политика выполняет аудит определенных административных операций без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-the-delete-network-security-group-rule"></a>Обеспечение того, что для операции удаления правила группы безопасности сети существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.6 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для выполнения определенных административных операций должно существовать оповещение журнала действий.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Эта политика выполняет аудит определенных административных операций без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-security-solution"></a>Обеспечение того, что для операции создания или обновления решения безопасности существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.7 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для определенных операций безопасности должно существовать оповещение журнала действий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b980d31-7904-4bb7-8575-5665739a8052) |Эта политика выполняет аудит определенных операций безопасности без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_SecurityOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-security-solution"></a>Обеспечение того, что для операции удаления решения безопасности существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.8 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для определенных операций безопасности должно существовать оповещение журнала действий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b980d31-7904-4bb7-8575-5665739a8052) |Эта политика выполняет аудит определенных операций безопасности без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_SecurityOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-or-delete-sql-server-firewall-rule"></a>Обеспечение того, что для операции создания, обновления или удаления правила брандмауэра SQL Server существует оповещение журнала действий

**Идентификатор**. CIS Azure 5.2.9 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для выполнения определенных административных операций должно существовать оповещение журнала действий.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Эта политика выполняет аудит определенных административных операций без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-diagnostic-logs-are-enabled-for-all-services-which-support-it"></a>Обеспечение включения журналов диагностики для всех служб, которые их поддерживают.

**Идентификатор**. CIS Azure 5.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В службах приложений должны быть включены журналы диагностики](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb607c5de-e7d9-4eee-9e5c-83f1bcee4fa0) |Аудит включения журналов диагностики для приложений. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditLoggingMonitoring_Audit.json) |
|[В Azure Data Lake Storage должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |
|[В Azure Stream Analytics должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9be5368-9bf5-4b84-9e0a-7850da98bb46) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Stream%20Analytics/StreamAnalytics_AuditDiagnosticLog_Audit.json) |
|[В учетных записях пакетной службы должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F428256e6-1fac-4f48-a757-df34c2b3336d) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Batch/Batch_AuditDiagnosticLog_Audit.json) |
|[В Data Lake Analytics должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc95c74d9-38fe-4f0d-af86-0c7d626a315c) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeAnalytics_AuditDiagnosticLog_Audit.json) |
|[В Центре событий должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F83a214f7-d01a-484b-91a9-ed54470c9a6a) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Event%20Hub/EventHub_AuditDiagnosticLog_Audit.json) |
|[В Центре Интернета вещей должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F383856f8-de7f-44a2-81fc-e5135b5c2aa4) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[3.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Internet%20of%20Things/IoTHub_AuditDiagnosticLog_Audit.json) |
|[В Key Vault должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf820ca0-f99e-4f3e-84fb-66e913812d21) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для анализа инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_AuditDiagnosticLog_Audit.json) |
|[В Logic Apps должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34f95f76-5386-4de7-b824-0d8478470c9d) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_AuditDiagnosticLog_Audit.json) |
|[В службах "Поиск" должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb4330a05-a843-4bc8-bf9a-cacce50c67f4) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Search/Search_AuditDiagnosticLog_Audit.json) |
|[В Служебной шине должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8d36e2f-389b-4ee4-898d-21aeb69a0f45) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Bus/ServiceBus_AuditDiagnosticLog_Audit.json) |
|[В Масштабируемых наборах виртуальных машин должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7c1b1214-f927-48bf-8882-84f0af6588b1) |Мы рекомендуем включить журналы для воссоздания следов действий и проведения исследования в случае инцидента или компрометации. |AuditIfNotExists, Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/ServiceFabric_and_VMSS_AuditVMSSDiagnostics.json) |

## <a name="networking"></a>Сеть

### <a name="ensure-that-rdp-access-is-restricted-from-the-internet"></a>Обеспечение того, что доступ по протоколу RDP через Интернет заблокирован

**Идентификатор**. CIS Azure 6.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Доступ по протоколу RDP через Интернет должен быть заблокирован](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe372f825-a257-4fb8-9175-797a8a8627d6) |Эта политика выполняет аудит всех правил безопасности сети, разрешающих доступ по протоколу RDP через Интернет. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroup_RDPAccess_Audit.json) |

### <a name="ensure-that-ssh-access-is-restricted-from-the-internet"></a>Обеспечение того, что доступ по протоколу SSH через Интернет заблокирован

**Идентификатор**. CIS Azure 6.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Доступ по протоколу SSH через Интернет должен быть заблокирован](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c89a2e5-7285-40fe-afe0-ae8654b92fab) |Эта политика выполняет аудит всех правил безопасности сети, разрешающих доступ по протоколу SSH через Интернет. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroup_SSHAccess_Audit.json) |

### <a name="ensure-that-network-watcher-is-enabled"></a>Обеспечение того, что служба "Наблюдатель за сетями" включена

**Идентификатор**. CIS Azure 6.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить Наблюдатель за сетями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6e2945c-0b7b-40f5-9233-7a5323b5cdc6) |Наблюдатель за сетями — это региональная служба, обеспечивающая мониторинг и диагностику условий на уровне сетевого сценария на платформе Azure. Мониторинг на уровне сценария позволяет диагностировать проблемы в сети с помощью комплексного представления сетевого уровня. Инструменты диагностики сети и визуализации, доступные в Наблюдателе за сетями, помогают понять, как работает сеть в Azure, диагностировать ее и получить ценную информацию. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkWatcher_Enabled_Audit.json) |

## <a name="virtual-machines"></a>Виртуальные машины

### <a name="ensure-virtual-machines-are-utilizing-managed-disks"></a>Обеспечение использование Управляемых дисков виртуальными машинами.

**Идентификатор**. CIS Azure 7.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит виртуальных машин, которые не используют управляемые диски](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F06a78e20-9358-41c9-923c-fb736d382a4d) |Эта политика выполняет аудит виртуальных машин, которые не используют управляемые диски. |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VMRequireManagedDisk_Audit.json) |

### <a name="ensure-that-os-and-data-disks-are-encrypted-with-cmk"></a>Обеспечение шифрования дисков ОС и дисков данных с помощью ключа, управляемого клиентом.

**Идентификатор**. CIS Azure 7.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах должно применяться шифрование дисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) |Виртуальные машины без шифрования дисков будут отслеживаться Центром безопасности Azure (это требование носит рекомендательный характер). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnencryptedVMDisks_Audit.json) |

### <a name="ensure-that-unattached-disks-are-encrypted-with-cmk"></a>Обеспечение шифрования неподключенных дисков с помощью ключа, управляемого клиентом.

**Идентификатор**. CIS Azure 7.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Неподключенные диски должны быть зашифрованы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c89a2e5-7285-40fe-afe0-ae8654b92fb2) |Эта политика выполняет аудит всех неподключенных дисков, на которых не активировано шифрование. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/UnattachedDisk_Encryption_Audit.json) |

### <a name="ensure-that-only-approved-extensions-are-installed"></a>Обеспечение того, что установлены только утвержденные расширения

**Идентификатор**. CIS Azure 7.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Должны быть установлены только утвержденные расширения виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc0e996f8-39cf-4af9-9f45-83fbde810432) |Эта политика управляет неутвержденными расширениями виртуальных машин. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VirtualMachines_ApprovedExtensions_Audit.json) |

### <a name="ensure-that-the-latest-os-patches-for-all-virtual-machines-are-applied"></a>Обеспечение того, что для всех виртуальных машин установлены последние исправления ОС

**Идентификатор**. CIS Azure 7.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На компьютерах должны быть установлены обновления системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86b3d65f-7626-441e-b690-81a8b71cff60) |Отсутствие обновлений системы безопасности на серверах будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingSystemUpdates_Audit.json) |

### <a name="ensure-that-the-endpoint-protection-for-all-virtual-machines-is-installed"></a>Обеспечение того, что защита конечной точки установлена для всех виртуальных машин

**Идентификатор**. CIS Azure 7.6 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Мониторинг отсутствия Endpoint Protection в Центре безопасности Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Серверы без установленного агента Endpoint Protection будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |

## <a name="other-security-considerations"></a>Прочие вопросы по безопасности

### <a name="ensure-that-the-expiration-date-is-set-on-all-keys"></a>Обеспечение того, что дата окончания срока действия задана для всех ключей

**Идентификатор**. CIS Azure 8.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для ключей Key Vault должна быть задана дата окончания срока действия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F152b15f7-8e1f-4c1f-ab71-8c010ba5dbc0) |Для криптографических ключей должна быть задана дата окончания срока действия. Они не должны быть постоянными. Если ключи, действительны всегда, у потенциального злоумышленника появляется дополнительное время для их взлома. В целях безопасности для криптографических ключей рекомендуется устанавливать даты истечения срока действия. |Audit, Deny, Disabled |[1.0.1 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/Keys_ExpirationSet.json) |

### <a name="ensure-that-the-expiration-date-is-set-on-all-secrets"></a>Обеспечение того, что дата окончания срока действия задана для всех секретов

**Идентификатор**. CIS Azure 8.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для секретов Key Vault должна быть задана дата окончания срока действия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F98728c90-32c7-4049-8429-847dc0f4fe37) |Для секретов должна быть задана дата окончания срока действия. Они не должны быть постоянными. Если секреты действительны всегда, у потенциального злоумышленника появляется дополнительное время для их взлома. В целях безопасности для секретов рекомендуется устанавливать даты истечения срока действия. |Audit, Deny, Disabled |[1.0.1 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/Secrets_ExpirationSet.json) |

### <a name="ensure-the-key-vault-is-recoverable"></a>Обеспечение того, что хранилище ключей поддерживает восстановление

**Идентификатор**. CIS Azure 8.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В хранилищах ключей должна быть включена защита от очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |Удаление хранилища ключей злоумышленником может привести к необратимой потере данных. Потенциальный злоумышленник в вашей организации может удалить и очистить хранилища ключей. Защита от очистки позволяет оградить вас от атак злоумышленников. Для этого применяется обязательный период хранения данных для хранилищ ключей, которые были обратимо удалены. Ни корпорация Майкрософт, ни пользователи вашей организации не смогут очистить хранилища ключей во время периода хранения при обратимом удалении. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |

### <a name="enable-role-based-access-control-rbac-within-azure-kubernetes-services"></a>Включение управления доступом на основе ролей (RBAC) в службе Azure Kubernetes

**Идентификатор**. CIS Azure 8.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В службах Kubernetes следует использовать управление доступом на основе ролей (RBAC)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |Чтобы обеспечить детальную фильтрацию действий, которые могут выполнять пользователи, используйте управление доступом на основе ролей (RBAC) для управления разрешениями в кластерах службы Kubernetes и настройте соответствующие политики авторизации. |Audit, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

## <a name="app-service"></a>Служба приложений

### <a name="ensure-app-service-authentication-is-set-on-azure-app-service"></a>Обеспечение того, что для Службы приложений Azure задана проверка подлинности в Службе приложений Azure

**Идентификатор**. CIS Azure 9.1 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Должна быть включена проверка подлинности для приложения API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4ebc54a-46e1-481a-bee2-d4411e95d828) |Проверка подлинности в Службе приложений Azure — это функция, позволяющая блокировать доступ к приложению API для анонимных HTTP-запросов и выполнять проверку подлинности для запросов с токеном. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_ApiApp_Audit.json) |
|[Должна быть включена проверка подлинности для приложения-функции](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc75248c1-ea1d-4a9c-8fc9-29a6aabd5da8) |Проверка подлинности в Службе приложений Azure — это функция, позволяющая блокировать доступ к приложению-функции для анонимных HTTP-запросов и выполнять проверку подлинности для запросов с токеном. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_functionapp_Audit.json) |
|[Должна быть включена проверка подлинности для веб-приложения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F95bccee9-a7f8-4bec-9ee9-62c3473701fc) |Проверка подлинности в Службе приложений Azure — это функция, позволяющая блокировать доступ к веб-приложению для анонимных HTTP-запросов и выполнять проверку подлинности для запросов с токеном. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_WebApp_Audit.json) |

### <a name="ensure-web-app-redirects-all-http-traffic-to-https-in-azure-app-service"></a>Обеспечение того, что веб-приложение перенаправляет весь трафик HTTP на HTTPS в Службе приложений Azure

**Идентификатор**. CIS Azure 9.2 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="ensure-web-app-is-using-the-latest-version-of-tls-encryption"></a>Проверка использования последней версии шифрования TLS в веб-приложении

**Идентификатор**. CIS Azure 9.3 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В вашем приложении API необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |

### <a name="ensure-the-web-app-has-client-certificates-incoming-client-certificates-set-to-on"></a>Параметр "Сертификаты клиента (входящие сертификаты клиента)" для веб-приложения должен иметь значение "Вкл."

**Идентификатор**. CIS Azure 9.4 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Параметр "Сертификаты клиента (входящие сертификаты клиента)" для приложения API должен иметь значение "Вкл"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0c192fe8-9cbb-4516-85b3-0ade8bd03886) |Сертификаты клиента позволяют приложению запрашивать сертификат для входящих запросов. Приложение будет доступно только клиентам с допустимым сертификатом. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_ClientCert.json) |
|[Параметр "Сертификаты клиента (входящие сертификаты клиента)" для веб-приложения должен иметь значение "Вкл"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5bb220d9-2698-4ee4-8404-b9c30c9df609) |Сертификаты клиента позволяют приложению запрашивать сертификат для входящих запросов. Приложение будет доступно только клиентам с допустимым сертификатом. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_ClientCert.json) |
|[Для приложений-функций должны быть включены сертификаты клиента (входящие сертификаты клиента)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feaebaea7-8013-4ceb-9d14-7eb32271373c) |Сертификаты клиента позволяют приложению запрашивать сертификат для входящих запросов. Приложение будет доступно только клиентам с допустимыми сертификатами. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_ClientCert.json) |

### <a name="ensure-that-register-with-azure-active-directory-is-enabled-on-app-service"></a>В службе приложений должна быть включена регистрация в Azure Active Directory

**Идентификатор**. CIS Azure 9.5 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В вашем приложении API необходимо использовать управляемое удостоверение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4d441f8-f9d9-4a9e-9cef-e82117cb3eef) |Используйте управляемое удостоверение для повышения безопасности проверки подлинности. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать управляемое удостоверение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0da106f2-4ca3-48e8-bc85-c638fe6aea8f) |Используйте управляемое удостоверение для повышения безопасности проверки подлинности. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать управляемое удостоверение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2b9ad585-36bc-4615-b300-fd4435808332) |Используйте управляемое удостоверение для повышения безопасности проверки подлинности. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_WebApp_Audit.json) |

### <a name="ensure-that-php-version-is-the-latest-if-used-to-run-the-web-app"></a>В веб-приложении должна использоваться последняя версия PHP

**Идентификатор**. CIS Azure 9.6 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В приложении API должна использоваться последняя версия PHP](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1bc1795e-d44a-4d48-9b3b-6fff0fd5f9ba) |Для программного обеспечения PHP периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для приложений API рекомендуется использовать последнюю версию PHP. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_PHP_Latest.json) |
|[В веб-приложении должна использоваться последняя версия PHP](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7261b898-8a84-4db8-9e04-18527132abb3) |Для программного обеспечения PHP периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для веб-приложений рекомендуется использовать последнюю версию PHP. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_PHP_Latest.json) |

### <a name="ensure-that-python-version-is-the-latest-if-used-to-run-the-web-app"></a>В веб-приложении должна использоваться последняя версия Python

**Идентификатор**. CIS Azure 9.7 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В приложении API должна использоваться последняя версия Python](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F74c3584d-afae-46f7-a20a-6f8adba71a16) |Для программного обеспечения Python периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для приложений API рекомендуется использовать последнюю версию Python. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_python_Latest.json) |
|[В приложении-функции должна использоваться последняя версия Python](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7238174a-fd10-4ef0-817e-fc820a951d73) |Для программного обеспечения Python периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для приложений-функций рекомендуется использовать последнюю версию Python. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_python_Latest.json) |
|[В веб-приложении должна использоваться последняя версия Python](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7008174a-fd10-4ef0-817e-fc820a951d73) |Для программного обеспечения Python периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для веб-приложений рекомендуется использовать последнюю версию Python. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_python_Latest.json) |

### <a name="ensure-that-java-version-is-the-latest-if-used-to-run-the-web-app"></a>В веб-приложении должна использоваться последняя версия Java

**Идентификатор**. CIS Azure 9.8 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В приложении API должна использоваться последняя версия Java](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F88999f4c-376a-45c8-bcb3-4058f713cf39) |Для Java периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для приложений API рекомендуется использовать последнюю версию Python. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_java_Latest.json) |
|[В приложении-функции должна использоваться последняя версия Java](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d0b6ea4-93e2-4578-bf2f-6bb17d22b4bc) |Для программного обеспечения Java периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для приложений-функций рекомендуется использовать последнюю версию Java. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_java_Latest.json) |
|[В веб-приложении должна использоваться последняя версия Java](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F496223c3-ad65-4ecd-878a-bae78737e9ed) |Для программного обеспечения Java периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для веб-приложений рекомендуется использовать последнюю версию Java. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_java_Latest.json) |

### <a name="ensure-that-http-version-is-the-latest-if-used-to-run-the-web-app"></a>В веб-приложении должна использоваться последняя версия HTTP

**Идентификатор**. CIS Azure 9.9 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В приложении API должна использоваться последняя версия HTTP](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F991310cd-e9f3-47bc-b7b6-f57b557d07db) |Для HTTP периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для веб-приложений рекомендуется использовать последнюю версию HTTP. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_HTTP_Latest.json) |
|[В приложении-функции должна использоваться последняя версия HTTP](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe2c1c086-2d84-4019-bff3-c44ccd95113c) |Для HTTP периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для веб-приложений рекомендуется использовать последнюю версию HTTP. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_HTTP_Latest.json) |
|[В веб-приложении должна использоваться последняя версия HTTP](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8c122334-9d20-4eb8-89ea-ac9a705b74ae) |Для HTTP периодически выпускаются новые версии, которые устраняют уязвимости безопасности или включают дополнительные функции. Чтобы получить эти возможности и преимущества, для веб-приложений рекомендуется использовать последнюю версию HTTP. Сейчас эта политика применяется только к веб-приложениям Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_HTTP_Latest.json) |

### <a name="ensure-ftp-deployments-are-disabled"></a>Обеспечение того, что развертывания по протоколу FTP отключены

**Идентификатор**. CIS Azure 9.10 **Ответственный**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В вашем приложении API необходимо принудительно использовать только FTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9a1b8c48-453a-4044-86c3-d8bfd823e4f5) |Включите принудительное использование FTPS для повышения безопасности. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditFTPS_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо принудительно использовать только FTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F399b2637-a50f-4f95-96f8-3a145476eb15) |Включите принудительное использование FTPS для повышения безопасности. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditFTPS_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо принудительно использовать FTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4d24b6d4-5e53-4a4f-a7f4-618fa573ee4b) |Включите принудительное использование FTPS для повышения безопасности. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditFTPS_WebApp_Audit.json) |

> [!NOTE]
> Возможность использования отдельных определений Политики Azure может отличаться в Azure для государственных организаций и других национальных облаках.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные статьи о Политике Azure:

- Обзор [соответствия нормативным требованиям](../concepts/regulatory-compliance.md).
- См. [структуру определения инициативы](../concepts/initiative-definition-structure.md).
- См. другие [примеры шаблонов для Политики Azure](./index.md).
- Изучите [сведения о действии политик](../concepts/effects.md).
- Узнайте, как [исправлять несоответствующие ресурсы](../how-to/remediate-resources.md).

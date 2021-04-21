---
title: Сведения о соответствии требованиям стандарта NIST SP 800-53 R4
description: Сведения о встроенной инициативе по соответствию требованиям стандарта NIST SP 800-53 R4. Каждый элемент управления сопоставляется с одним или несколькими определениями Политики Azure, которые помогают выполнять оценку.
ms.date: 04/14/2021
ms.topic: sample
ms.custom: generated
ms.openlocfilehash: 0fe88525bc574219729a7f90fba1bf82803e5854
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107505349"
---
# <a name="details-of-the-nist-sp-800-53-r4-regulatory-compliance-built-in-initiative"></a>Сведения о встроенной инициативе по соответствию требованиям стандарта NIST SP 800-53 R4

В следующей статье подробно описано, как определение встроенной инициативы Политики Azure по соответствию нормативным требованиям сопоставляется с **областями соответствия нормативным требованиям** и **элементами управления** в стандарте NIST SP 800-53 R4.
Дополнительные сведения об этом стандарте соответствия см. в разделе о [NIST SP 800-53 R4 — специальной публикации 800-53 Национального института стандартов и технологий (версия 4)](https://nvd.nist.gov/800-53). Сведения о термине _Ответственность_ см. в статьях [Определение службы "Политика Azure"](../concepts/definition-structure.md#type) и [Разделение ответственности в облаке](../../../security/fundamentals/shared-responsibility.md).

Далее представлены сопоставления для элементов управления, определяемых стандартом **NIST SP 800-53 R4**. Используйте панель навигации справа для перехода непосредственно к конкретной **области соответствия нормативным требованиям**. Многие элементы управления реализуются с помощью определения инициативы [Политики Azure](../overview.md). Чтобы просмотреть полное определение инициативы, откройте раздел **Политика** на портале Azure и перейдите на страницу **Определения**.
Затем найдите и выберите встроенное определение инициативы **NIST SP 800-53 R4** по обеспечению соответствия нормативным требованиям.

Эта встроенная инициатива развертывается в составе [примера схемы NIST SP 800-53 R4](../../blueprints/samples/nist-sp-800-53-r4.md).

> [!IMPORTANT]
> Каждый элемент управления ниже связан с одним или несколькими определениями [Политики Azure](../overview.md).
> Такие политики помогут вам в [оценке соответствия](../how-to/get-compliance-data.md) с помощью элементов управления, но часто полное или точное соответствие между элементом управления и одной или несколькими политиками отсутствует. Поэтому состояние **Совместимый** в Политике Azure применимо только к самим определениям политики и не означает полное соответствие всем требованиям элемента управления. Кроме того, стандарт соответствия включает элементы управления, на которые сейчас не распространяются определения Политики Azure. Следовательно, сведения о соответствии в Политике Azure — это только частичное представление общего состояния соответствия. Связи между областями соответствия нормативным требованиям, элементами управления и определениями Политики Azure для этого стандарта соответствия со временем могут меняться. Историю изменений можно просмотреть на странице [журнала фиксаций в GitHub](https://github.com/Azure/azure-policy/commits/master/built-in-policies/policySetDefinitions/Regulatory%20Compliance/NIST80053_audit.json).

## <a name="access-control"></a>Управление доступом

### <a name="access-control-policy-and-procedures"></a>Процедуры и политика управления доступом

**Идентификатор**. NIST SP 800-53 R4 AC-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1000, контролируемый корпорацией Майкрософт, — политика и процедуры управления доступом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2ef3cc79-733e-48ed-ab6f-7bf439e9b406) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1000.json) |
|[Элемент управления 1001, контролируемый корпорацией Майкрософт, — политика и процедуры управления доступом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4e26f8c3-4bf3-4191-b8fc-d888805101b7) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1001.json) |

### <a name="account-management"></a>Управление учетными записями

**Идентификатор**. NIST SP 800-53 R4 AC-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Устаревшие учетные записи должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6b1cbf55-e8b6-442f-ba4c-7246b6381474) |Устаревшие учетные записи должны быть удалены из подписок.  Устаревшими считаются те учетные записи, для которых заблокирована возможность входа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveDeprecatedAccounts_Audit.json) |
|[Устаревшие учетные записи с разрешениями владельца должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Febb62a0c-3560-49e1-89ed-27e074e9f8ad) |Устаревшие учетные записи с разрешениями владельца следует удалять из подписки.  Устаревшими считаются те учетные записи, для которых заблокирована возможность входа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveDeprecatedAccountsWithOwnerPermissions_Audit.json) |
|[Внешние учетные записи с разрешениями владельца должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8456c1c-aa66-4dfb-861a-25d127b775c9) |Внешние учетные записи с разрешениями владельца должны быть удалены из подписки, чтобы предотвратить неотслеживаемый доступ. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWithOwnerPermissions_Audit.json) |
|[Внешние учетные записи с разрешениями на чтение должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5f76cf89-fbf2-47fd-a3f4-b891fa780b60) |Внешние учетные записи с правами на чтение должны быть удалены из подписки, чтобы предотвратить неотслеживаемый доступ. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsReadPermissions_Audit.json) |
|[Внешние учетные записи с разрешениями на запись должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c607a2e-c700-4744-8254-d77e7c9eb5e4) |Внешние учетные записи с правами на запись должны быть удалены из подписки, чтобы предотвратить неотслеживаемый доступ. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWritePermissions_Audit.json) |
|[Элемент управления 1002, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F632024c2-8079-439d-a7f6-90af1d78cc65) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1002.json) |
|[Элемент управления 1003, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b68b179-3704-4ff7-b51d-7d65374d165d) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1003.json) |
|[Элемент управления 1004, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc17822dc-736f-4eb4-a97d-e6be662ff835) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1004.json) |
|[Элемент управления 1005, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5b626abc-26d4-4e22-9de8-3831818526b1) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1005.json) |
|[Элемент управления 1006, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faae8d54c-4bce-4c04-b3aa-5b65b67caac8) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1006.json) |
|[Элемент управления 1007, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17200329-bf6c-46d8-ac6d-abf4641c2add) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1007.json) |
|[Элемент управления 1008, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8356cfc6-507a-4d20-b818-08038011cd07) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1008.json) |
|[Элемент управления 1009, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb26f8610-e615-47c2-abd6-c00b2b0b503a) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1009.json) |
|[Элемент управления 1010, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F784663a8-1eb0-418a-a98c-24d19bc1bb62) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1010.json) |
|[Элемент управления 1011, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7e6a54f3-883f-43d5-87c4-172dfd64a1f5) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1011.json) |
|[Элемент управления 1012, контролируемый корпорацией Майкрософт, — управление учетными записями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fefd7b9ae-1db6-4eb6-b0fe-87e6565f9738) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1012.json) |

### <a name="account-management--automated-system-account-management"></a>Управление учетной записью | Автоматическое управление системной учетной записью

**Идентификатор**. NIST SP 800-53 R4 AC-2 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1013, контролируемый корпорацией Майкрософт, — управление учетной записью \| Автоматизированное управление системной учетной записью](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8fd7b917-d83b-4379-af60-51e14e316c61) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1013.json) |

### <a name="account-management--removal-of-temporary--emergency-accounts"></a>Управление учетными записями | Удаление временных или аварийных учетных записей

**Идентификатор**. NIST SP 800-53 R4 AC-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1014, контролируемый корпорацией Майкрософт — управление учетными записями \| Удаление временных или аварийных учетных записей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5dee936c-8037-4df1-ab35-6635733da48c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1014.json) |

### <a name="account-management--disable-inactive-accounts"></a>Управление учетными записями | Отключение неактивных учетных записей

**Идентификатор**. NIST SP 800-53 R4 AC-2 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1015, контролируемый корпорацией Майкрософт, — управление учетными записями \| Отключение неактивных учетных записей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F544a208a-9c3f-40bc-b1d1-d7e144495c14) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1015.json) |

### <a name="account-management--automated-audit-actions"></a>Управление учетными записями | Автоматизированные действия для аудита

**Идентификатор**. NIST SP 800-53 R4 AC-2 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1016, контролируемый корпорацией Майкрософт, — управление учетными записями \| Автоматизированные действия для аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd8b43277-512e-40c3-ab00-14b3b6e72238) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1016.json) |

### <a name="account-management--inactivity-logout"></a>Управление учетными записями | Выход при бездействии

**Идентификатор**. NIST SP 800-53 R4 AC-2 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1017, контролируемый корпорацией Майкрософт, — управление учетными записями \| Выход при бездействии](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fc3db37-e59a-48c1-84e9-1780cedb409e) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1017.json) |

### <a name="account-management--role-based-schemes"></a>Управление учетными записями | Схемы на основе ролей

**Идентификатор**. NIST SP 800-53 R4 AC-2 (7) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов SQL должен быть подготовлен администратор Azure Active Directory](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1f314764-cb73-4fc9-b863-8eca98ac36e9) |Аудит подготовки администратора AAD для включения в SQL Server поддержки проверки подлинности AAD. Эта проверка подлинности упрощает контроль разрешений и обеспечивает централизованное управление удостоверениями пользователей баз данных и других служб Майкрософт. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SQL_DB_AuditServerADAdmins_Audit.json) |
|[Аудит использования настраиваемых правил RBAC](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |Аудит встроенных ролей, таких как "Владелец", "Участник", "Читатель", вместо настраиваемых ролей RBAC, использование которых сопряжено с ошибками. Работа с пользовательскими ролями рассматривается как исключение и требует строгой проверки с моделированием угроз. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |
|[Элемент управления 1018, контролируемый корпорацией Майкрософт, — управление учетными записями \| Схемы на основе ролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc9121abf-e698-4ee9-b1cf-71ee528ff07f) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1018.json) |
|[Элемент управления 1019, контролируемый корпорацией Майкрософт, — управление учетными записями \| Схемы на основе ролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6a3ee9b2-3977-459c-b8ce-2db583abd9f7) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1019.json) |
|[Элемент управления 1020, контролируемый корпорацией Майкрософт, — управление учетными записями \| Схемы на основе ролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b291ee8-3140-4cad-beb7-568c077c78ce) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1020.json) |
|[Кластеры Service Fabric должны использовать только Azure Active Directory для проверки подлинности клиента](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb54ed75b-3e1a-44ac-a333-05ba39b99ff0) |Аудит проверки подлинности клиента только через Azure Active Directory в Service Fabric |Audit, Deny, Disabled |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Fabric/ServiceFabric_AuditADAuth_Audit.json) |

### <a name="account-management--restrictions-on-use-of-shared--group-accounts"></a>Управление учетными записями | Ограничения использования общих или групповых учетных записей

**Идентификатор**. NIST SP 800-53 R4 AC-2 (9) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1021, контролируемый корпорацией Майкрософт — управление учетными записями \| Ограничения на использование общих или групповых учетных записей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9a3eb0a3-428d-4669-baff-20a14eb4b551) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1021.json) |

### <a name="account-management--shared--group-account-credential-termination"></a>Управление учетными записями | Истечение срока действия учетных данных общей или групповой учетной записи

**Идентификатор**. NIST SP 800-53 R4 AC-2 (10) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1022, контролируемый корпорацией Майкрософт — управление учетными записями \| Истечение срока действия учетных данных общей или групповой учетной записи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F411f7e2d-9a0b-4627-a0b9-1700432db47d) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1022.json) |

### <a name="account-management--usage-conditions"></a>Управление учетными записями | Условия использования

**Идентификатор**. NIST SP 800-53 R4 AC-2 (11) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1023, контролируемый корпорацией Майкрософт, — управление учетными записями \| Условия использования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe55698b6-3dea-4aa9-99b9-d8218c6ab6e5) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1023.json) |

### <a name="account-management--account-monitoring--atypical-usage"></a>Управление учетными записями | Мониторинг учетной записи и нетипичное использование

**Идентификатор**. NIST SP 800-53 R4 AC-2 (12) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |
|[Элемент управления 1024, контролируемый корпорацией Майкрософт, — управление учетными записями \| Мониторинг учетной записи и нетипичное использование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F84914fb4-12da-4c53-a341-a9fd463bed10) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1024.json) |
|[Элемент управления 1025, контролируемый корпорацией Майкрософт, — управление учетными записями \| Мониторинг учетной записи и нетипичное использование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fadfe020d-0a97-45f4-a39c-696ef99f3a95) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1025.json) |

### <a name="account-management--disable-accounts-for-high-risk-individuals"></a>Управление учетными записями | Отключение учетных записей для пользователей с высоким уровнем риска

**Идентификатор**. NIST SP 800-53 R4 AC-2 (13) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1026, контролируемый корпорацией Майкрософт — управление учетными записями \| Отключение учетных записей для пользователей с высоким уровнем риска](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F55419419-c597-4cd4-b51e-009fd2266783) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1026.json) |

### <a name="access-enforcement"></a>Применение доступа

**Идентификатор**. NIST SP 800-53 R4 AC-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1027, контролируемый корпорацией Майкрософт, — применение доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa76ca9b0-3f4a-4192-9a38-b25e4f8ae48c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1027.json) |

### <a name="information-flow-enforcement"></a>Управление потоком информации

**Идентификатор**. NIST SP 800-53 R4 AC-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Функция CORS не должна разрешать всем ресурсам доступ к вашим веб-приложениям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5744710e-cc2f-4ee8-8809-3b11e89f4bc9) |Средства общего доступа к ресурсам независимо от источника (CORS) не должны разрешать доступ к вашему веб-приложению всем доменам. Разрешите взаимодействие с веб-приложением только необходимым доменам. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RestrictCORSAccess_WebApp_Audit.json) |
|[Элемент управления 1028, контролируемый корпорацией Майкрософт, — управление потоком информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff171df5c-921b-41e9-b12b-50801c315475) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1028.json) |

### <a name="information-flow-enforcement--security-policy-filters"></a>Управление потоком информации | Фильтры политики безопасности

**Идентификатор**. NIST SP 800-53 R4 AC-4 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1029, контролируемый корпорацией Майкрософт — управление потоком информации \| Фильтры политики безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F53ac8f8e-c2b5-4d44-8a2d-058e9ced9b69) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1029.json) |

### <a name="information-flow-enforcement--physical--logical-separation-of-information-flows"></a>Управление потоком информации | Физическое и логическое разделение потоков информации

**Идентификатор**. NIST SP 800-53 R4 AC-4 (21) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1030, контролируемый корпорацией Майкрософт, — управление потоком информации \| Физическое и логическое разделение потоков информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd3531453-b869-4606-9122-29c1cd6e7ed1) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1030.json) |

### <a name="separation-of-duties"></a>Разделение обязанностей

**Идентификатор**. NIST SP 800-53 R4 AC-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено не более 3 владельцев](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f11b553-d42e-4e3a-89be-32ca364cad4c) |Мы рекомендуем назначать подписке не более трех владельцев, чтобы снизить вероятность нарушения безопасности при компрометации владельца. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateLessThanXOwners_Audit.json) |
|[Аудит компьютеров с Windows без любого из указанных участников в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F30f71ea1-ac77-4f26-9fc5-2d926bbd4ba7) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если в локальной группе администраторов отсутствует один или несколько участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToInclude_AINE.json) |
|[Аудит компьютеров Windows с указанными участниками в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69bf4abd-ca1e-4cf6-8b5a-762d42e61d4f) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если локальная группа администраторов содержит одного или нескольких участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToExclude_AINE.json) |
|[Элемент управления 1031, контролируемый корпорацией Майкрософт, — разделение обязанностей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6b93a801-fe25-4574-a60d-cb22acffae00) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1031.json) |
|[Элемент управления 1032, контролируемый корпорацией Майкрософт, — разделение обязанностей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5aa85661-d618-46b8-a20f-ca40a86f0751) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1032.json) |
|[Элемент управления 1033, контролируемый корпорацией Майкрософт, — разделение обязанностей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48540f01-fc11-411a-b160-42807c68896e) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1033.json) |
|[Подписке должно быть назначено более одного владельца](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09024ccc-0c5f-475e-9457-b7c0d9ed487b) |Мы рекомендуем назначить более одного владельца подписки, чтобы обеспечить избыточность для административного доступа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateMoreThanOneOwner_Audit.json) |

### <a name="least-privilege"></a>Минимальные привилегии

**Идентификатор**. NIST SP 800-53 R4 AC-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1034, контролируемый корпорацией Майкрософт, — минимальные привилегии](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F02a5ed00-6d2e-4e97-9a98-46c32c057329) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1034.json) |

### <a name="least-privilege--authorize-access-to-security-functions"></a>Минимальные привилегии | Авторизация доступа к функциям безопасности

**Идентификатор**. NIST SP 800-53 R4 AC-6 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1035, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Авторизация доступа к функциям безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fca94b046-45e2-444f-a862-dc8ce262a516) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1035.json) |

### <a name="least-privilege--non-privileged-access-for-nonsecurity-functions"></a>Минимальные привилегии | Непривилегированный доступ для функций, не связанных с безопасностью

**Идентификатор**. NIST SP 800-53 R4 AC-6 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1036, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Непривилегированный доступ для функций, не связанных с безопасностью](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9a16d673-8cf0-4dcf-b1d5-9b3e114fef71) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1036.json) |

### <a name="least-privilege--network-access-to-privileged-commands"></a>Минимальные привилегии | Сетевой доступ к привилегированным командам

**Идентификатор**. NIST SP 800-53 R4 AC-6 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1037, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Сетевой доступ к привилегированным командам](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffa4c2a3d-1294-41a3-9ada-0e540471e9fb) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1037.json) |

### <a name="least-privilege--privileged-accounts"></a>Минимальные привилегии | Привилегированные учетные записи

**Идентификатор**. NIST SP 800-53 R4 AC-6 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1038, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Привилегированные учетные записи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F26692e88-71b7-4a5f-a8ac-9f31dd05bd8e) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1038.json) |

### <a name="least-privilege--review-of-user-privileges"></a>Минимальные привилегии | Проверка привилегий пользователя

**Идентификатор**. NIST SP 800-53 R4 AC-6 (7) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено не более 3 владельцев](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f11b553-d42e-4e3a-89be-32ca364cad4c) |Мы рекомендуем назначать подписке не более трех владельцев, чтобы снизить вероятность нарушения безопасности при компрометации владельца. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateLessThanXOwners_Audit.json) |
|[Аудит компьютеров с Windows без любого из указанных участников в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F30f71ea1-ac77-4f26-9fc5-2d926bbd4ba7) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если в локальной группе администраторов отсутствует один или несколько участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToInclude_AINE.json) |
|[Аудит компьютеров Windows с указанными участниками в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69bf4abd-ca1e-4cf6-8b5a-762d42e61d4f) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если локальная группа администраторов содержит одного или нескольких участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToExclude_AINE.json) |
|[Элемент управления 1039, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Проверка привилегий пользователя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3a7b9de4-a8a2-4672-914d-c5f6752aa7f9) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1039.json) |
|[Элемент управления 1040, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Проверка привилегий пользователя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F54205576-cec9-463f-ba44-b4b3f5d0a84c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1040.json) |
|[Подписке должно быть назначено более одного владельца](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09024ccc-0c5f-475e-9457-b7c0d9ed487b) |Мы рекомендуем назначить более одного владельца подписки, чтобы обеспечить избыточность для административного доступа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateMoreThanOneOwner_Audit.json) |

### <a name="least-privilege--privilege-levels-for-code-execution"></a>Минимальные привилегии | Уровни привилегий для выполнения кода

**Идентификатор**. NIST SP 800-53 R4 AC-6 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1041, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Уровни привилегий для выполнения кода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb3d8d15b-627a-4219-8c96-4d16f788888b) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1041.json) |

### <a name="least-privilege--auditing-use-of-privileged-functions"></a>Минимальные привилегии | Аудит использования привилегированных функций

**Идентификатор**. NIST SP 800-53 R4 AC-6 (9) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1042, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Аудит использования привилегированных функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F319dc4f0-0fed-4ac9-8fc3-7aeddee82c07) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1042.json) |

### <a name="least-privilege--prohibit-non-privileged-users-from-executing-privileged-functions"></a>Минимальные привилегии | Запрещение выполнения привилегированных функций непривилегированными пользователями

**Идентификатор**. NIST SP 800-53 R4 AC-6 (10) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1043, контролируемый корпорацией Майкрософт, — минимальные привилегии \| Запрещение выполнения привилегированных функций непривилегированными пользователями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F361a77f6-0f9c-4748-8eec-bc13aaaa2455) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1043.json) |

### <a name="unsuccessful-logon-attempts"></a>Неудачные попытки входа

**Идентификатор**. NIST SP 800-53 R4 AC-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1044, контролируемый корпорацией Майкрософт, — неудачные попытки входа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0abbac52-57cf-450d-8408-1208d0dd9e90) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1044.json) |
|[Элемент управления 1045, контролируемый корпорацией Майкрософт, — неудачные попытки входа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F554d2dd6-f3a8-4ad5-b66f-5ce23bd18892) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1045.json) |

### <a name="unsuccessful-logon-attempts--purge--wipe-mobile-device"></a>Неудачные попытки входа | Очистка данных или их удаление из мобильного устройства

**Идентификатор**. NIST SP 800-53 R4 AC-7 (2) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1046, контролируемый корпорацией Майкрософт, — автоматическая блокировка учетной записи \| Очистка мобильного устройства](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b1aa965-7502-41f9-92be-3e2fe7cc392a) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1046.json) |

### <a name="system-use-notification"></a>Уведомление об использовании системы

**Идентификатор**. NIST SP 800-53 R4 AC-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1047, контролируемый корпорацией Майкрософт, — уведомление об использовании системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1ff6d62-a55c-41ab-90ba-90bb5b7b6f62) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1047.json) |
|[Элемент управления 1048, контролируемый корпорацией Майкрософт, — уведомление об использовании системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F483e7ca9-82b3-45a2-be97-b93163a0deb7) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1048.json) |
|[Элемент управления 1049, контролируемый корпорацией Майкрософт, — уведомление об использовании системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9adf7ba7-900a-4f35-8d57-9f34aafc405c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1049.json) |

### <a name="concurrent-session-control"></a>Контроль одновременных сеансов

**Идентификатор**. NIST SP 800-53 R4 AC-10 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1050, контролируемый корпорацией Майкрософт, — контроль одновременных сеансов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbd20184c-b4ec-4ce5-8db6-6e86352d183f) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1050.json) |

### <a name="session-lock"></a>Блокировка сеанса

**Идентификатор**. NIST SP 800-53 R4 AC-11 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1051, контролируемый корпорацией Майкрософт, — блокировка сеанса](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7cac6ee9-b58b-40c8-a5ce-f0efc3d9b339) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1051.json) |
|[Элемент управления 1052, контролируемый корпорацией Майкрософт, — блокировка сеанса](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F027cae1c-ec3e-4492-9036-4168d540c42a) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1052.json) |

### <a name="session-lock--pattern-hiding-displays"></a>Блокировка сеанса | Отображения со скрытыми шаблонами

**Идентификатор**. NIST SP 800-53 R4 AC-11 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1053, контролируемый корпорацией Майкрософт, — блокировка сеанса \| Отображения с использованием шаблонов скрытия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7582b19c-9dba-438e-aed8-ede59ac35ba3) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1053.json) |

### <a name="session-termination"></a>Завершение сеанса

**Идентификатор**. NIST SP 800-53 R4 AC-12 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1054, контролируемый корпорацией Майкрософт, — завершение сеанса](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5807e1b4-ba5e-4718-8689-a0ca05a191b2) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1054.json) |

### <a name="session-termination--user-initiated-logouts--message-displays"></a>Завершение сеанса | Операции выхода, активированные пользователем, и отображение сообщения

**Идентификатор**. NIST SP 800-53 R4 AC-12 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1055, контролируемый корпорацией Майкрософт, — завершение сеанса \| Операции выхода, активированные пользователем, и отображение сообщений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F769efd9b-3587-4e22-90ce-65ddcd5bd969) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1055.json) |
|[Элемент управления 1056, контролируемый корпорацией Майкрософт, — завершение сеанса \| Операции выхода, активированные пользователем, и отображение сообщений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac43352f-df83-4694-8738-cfce549fd08d) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1056.json) |

### <a name="permitted-actions-without-identification-or-authentication"></a>Разрешенные действия без идентификации и проверки подлинности

**Идентификатор**. NIST SP 800-53 R4 AC-14 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1057, контролируемый корпорацией Майкрософт, — разрешенные действия без идентификации и аутентификации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F78255758-6d45-4bf0-a005-7016bc03b13c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1057.json) |
|[Элемент управления 1058, контролируемый корпорацией Майкрософт, — разрешенные действия без идентификации и аутентификации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F76e85d08-8fbb-4112-a1c1-93521e6a9254) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1058.json) |

### <a name="security-attributes"></a>Атрибуты безопасности

**Идентификатор**. NIST SP 800-53 R4 AC-16 **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемом экземпляре SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Проверка всех Управляемых экземпляров SQL без Расширенной защиты данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[На серверах SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Аудит серверов SQL без Расширенной защиты данных |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |

### <a name="remote-access"></a>Удаленный доступ

**Идентификатор**. NIST SP 800-53 R4 AC-17 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1059, контролируемый корпорацией Майкрософт, — удаленный доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa29b5d9f-4953-4afe-b560-203a6410b6b4) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1059.json) |
|[Элемент управления 1060, контролируемый корпорацией Майкрософт, — удаленный доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34a987fd-2003-45de-a120-014956581f2b) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1060.json) |

### <a name="remote-access--automated-monitoring--control"></a>Удаленный доступ | Автоматический мониторинг и управление

**Идентификатор**. NIST SP 800-53 R4 AC-17 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Добавление управляемого удостоверения, назначаемого системой, для включения назначений гостевой конфигурации на виртуальных машинах без удостоверений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3cf2ab00-13f1-4d0c-8971-2ac904541a7e) |Эта политика добавляет назначаемое системой управляемое удостоверение на виртуальные машины, размещенные в Azure, которые поддерживаются гостевой конфигурацией, но не имеют ни одного управляемого удостоверения. Назначаемое системой управляемое удостоверение является необходимым условием для всех назначений гостевой конфигурации, и его следует добавить на компьютеры перед тем, как использовать любые определения политик гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |modify |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AddSystemIdentityWhenNone_Prerequisite.json) |
|[Добавление управляемого удостоверения, назначаемого системой, для включения назначений гостевой конфигурации на виртуальных машинах с удостоверением, назначаемым пользователем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F497dff13-db2a-4c0f-8603-28fa3b331ab6) |Эта политика добавляет назначаемое системой управляемое удостоверение на виртуальные машины, размещенные в Azure, которые поддерживаются гостевой конфигурацией и имеют по меньшей мере одно назначаемое пользователем удостоверение, но не имеют назначаемого системой управляемого удостоверения. Назначаемое системой управляемое удостоверение является необходимым условием для всех назначений гостевой конфигурации, и его следует добавить на компьютеры перед тем, как использовать любые определения политик гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |modify |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AddSystemIdentityWhenUser_Prerequisite.json) |
|[Аудит компьютеров Linux, разрешающих удаленные подключения для учетных записей без паролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea53dbee-c6c9-4f0e-9f9e-de0039b78023) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если при установленной ОС Linux они разрешают удаленные подключения для учетных записей без паролей. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_LinuxPassword110_AINE.json) |
|[Развертывание расширения гостевой конфигурации Linux для включения назначений гостевой конфигурации на виртуальных машинах Linux](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F331e8ea8-378a-410f-a2e5-ae22f38bb0da) |Эта политика развертывает расширение гостевой конфигурации Linux на виртуальные машины Linux, размещенные в Azure и поддерживаемые гостевой конфигурацией. Расширение гостевой конфигурации Linux является необходимым условием для всех назначений гостевой конфигурации Linux и должно быть развернуто на компьютерах перед применением любых определений политики гостевой конфигурации Linux. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_DeployExtensionLinux_Prerequisite.json) |
|[Развертывание расширения гостевой конфигурации Windows для включения назначений гостевой конфигурации на виртуальных машинах Windows](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F385f5831-96d4-41db-9a3c-cd3af78aaae6) |Эта политика развертывает расширение гостевой конфигурации Windows на виртуальные машины Windows, размещенные в Azure и поддерживаемые гостевой конфигурацией. Расширение гостевой конфигурации Windows является необходимым условием для всех назначений гостевой конфигурации Windows и должно быть развернуто на компьютерах перед применением любых определений политики гостевой конфигурации Windows. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_DeployExtensionWindows_Prerequisite.json) |
|[Элемент управления 1061, контролируемый корпорацией Майкрософт, — удаленный доступ \| Автоматический мониторинг и управление](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7ac22808-a2e8-41c4-9d46-429b50738914) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1061.json) |
|[Удаленная отладка должна быть отключена для приложений API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe9c8d085-d9cc-4b17-9cdc-059f1f01f19e) |Для удаленной отладки нужно, чтобы в приложениях API были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_ApiApp_Audit.json) |
|[Удаленная отладка должна быть отключена для приложений-функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e60b895-3786-45da-8377-9c6b4b6ac5f9) |Для удаленной отладки нужно, чтобы в приложениях-функциях были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_FunctionApp_Audit.json) |
|[Удаленная отладка должна быть отключена для веб-приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcb510bfd-1cba-4d9f-a230-cb0976f4bb71) |Для удаленной отладки нужно, чтобы в веб-приложении были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_WebApp_Audit.json) |
|[Учетные записи хранения должны ограничивать доступ к сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34c877ad-507e-4c82-993e-3452a6e0ad3c) |Сетевой доступ к учетным записям хранения должен быть ограниченным. Настройте правила сети так, чтобы учетная запись хранения была доступна только приложениям из разрешенных сетей. Чтобы разрешить подключения от конкретных локальных клиентов и интернет-клиентов, вы можете открыть доступ для трафика из конкретных виртуальных сетей Azure или определенных диапазонов общедоступных IP-адресов. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_NetworkAcls_Audit.json) |

### <a name="remote-access--protection-of-confidentiality--integrity-using-encryption"></a>Удаленный доступ | Защита конфиденциальности и целостности шифрованием

**Идентификатор**. NIST SP 800-53 R4 AC-17 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1062, контролируемый корпорацией Майкрософт, — удаленный доступ \| Защита конфиденциальности и целостности шифрованием](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4708723f-e099-4af1-bbf9-b6df7642e444) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1062.json) |

### <a name="remote-access--managed-access-control-points"></a>Удаленный доступ | Управляемые точки контроля доступа

**Идентификатор**. NIST SP 800-53 R4 AC-17 (3) **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1063, контролируемый корпорацией Майкрософт, — удаленный доступ \| Управляемые точки контроля доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F593ce201-54b2-4dd0-b34f-c308005d7780) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1063.json) |

### <a name="remote-access--privileged-commands--access"></a>Удаленный доступ | Привилегированные команды и доступ

**Идентификатор**. NIST SP 800-53 R4 AC-17 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1064, контролируемый корпорацией Майкрософт, — удаленный доступ \| Привилегированные команды и доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb4d9508-cbf0-4a3c-bb5c-6c95b159f3fb) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1064.json) |
|[Элемент управления 1065, контролируемый корпорацией Майкрософт, — удаленный доступ \| Привилегированные команды и доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff87b8085-dca9-4cf1-8f7b-9822b997797c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1065.json) |

### <a name="remote-access--disconnect--disable-access"></a>Удаленный доступ | Отключение или запрещение доступа

**Идентификатор**. NIST SP 800-53 R4 AC-17 (9) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1066, контролируемый корпорацией Майкрософт, — удаленный доступ \| Отключение или запрещение доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4455c2e8-c65d-4acf-895e-304916f90b36) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1066.json) |

### <a name="wireless-access"></a>Беспроводной доступ

**Идентификатор**. NIST SP 800-53 R4 AC-18 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1067, контролируемый корпорацией Майкрософт, — беспроводной доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c5e54f6-0127-44d0-8b61-f31dc8dd6190) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1067.json) |
|[Элемент управления 1068, контролируемый корпорацией Майкрософт, — беспроводной доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2d045bca-a0fd-452e-9f41-4ec33769717c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1068.json) |

### <a name="wireless-access--authentication-and-encryption"></a>Беспроводной доступ | Проверка подлинности и шифрование

**Идентификатор**. NIST SP 800-53 R4 AC-18 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1069, контролируемый корпорацией Майкрософт, — беспроводной доступ \| Аутентификация и шифрование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F91c97b44-791e-46e9-bad7-ab7c4949edbb) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1069.json) |

### <a name="wireless-access--disable-wireless-networking"></a>Беспроводной доступ | Отключение беспроводных сетей

**Идентификатор**. NIST SP 800-53 R4 AC-18 (3) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1070, контролируемый корпорацией Майкрософт, — беспроводной доступ \| Отключение беспроводных сетей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F68f837d0-8942-4b1e-9b31-be78b247bda8) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1070.json) |

### <a name="wireless-access--restrict-configurations-by-users"></a>Беспроводной доступ | Ограничение конфигураций пользователями

**Идентификатор**. NIST SP 800-53 R4 AC-18 (4) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1071, контролируемый корпорацией Майкрософт, — беспроводной доступ \| Ограничение конфигураций, настраиваемых пользователями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1a437f5b-9ad6-4f28-8861-de404d511ae4) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1071.json) |

### <a name="wireless-access--antennas--transmission-power-levels"></a>Беспроводной доступ | Антенны и уровни мощности передачи

**Идентификатор**. NIST SP 800-53 R4 AC-18 (5) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1072, контролируемый корпорацией Майкрософт, — беспроводной доступ \| Антенны и уровни мощности передачи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1ca29e41-34ec-4e70-aba9-6248aca18c31) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1072.json) |

### <a name="access-control-for-mobile-devices"></a>Управление доступом мобильных устройств

**Идентификатор**. NIST SP 800-53 R4 AC-19 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1073, контролируемый корпорацией Майкрософт, — управление доступом мобильных устройств](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fab55cdb0-c7dd-4bd8-ae22-a7cea7594e9c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1073.json) |
|[Элемент управления 1074, контролируемый корпорацией Майкрософт, — управление доступом мобильных устройств](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F27a69937-af92-4198-9b86-08d355c7e59a) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1074.json) |

### <a name="access-control-for-mobile-devices--full-device--container-based--encryption"></a>Управление доступом для мобильных устройств | Полное шифрование устройства и шифрование на основе контейнера

**Идентификатор**. NIST SP 800-53 R4 AC-19 (5) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1075, контролируемый корпорацией Майкрософт, — управление доступом мобильных устройств \| Полное шифрование устройства и шифрование на основе контейнера](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffc933d22-04df-48ed-8f87-22a3773d4309) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1075.json) |

### <a name="use-of-external-information-systems"></a>Использование внешних информационных систем

**Идентификатор**. NIST SP 800-53 R4 AC-20 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1076, контролируемый корпорацией Майкрософт, — использование внешних информационных систем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F98a4bd5f-6436-46d4-ad00-930b5b1dfed4) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1076.json) |
|[Элемент управления 1077, контролируемый корпорацией Майкрософт, — использование внешних информационных систем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2dad3668-797a-412e-a798-07d3849a7a79) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1077.json) |

### <a name="use-of-external-information-systems--limits-on-authorized-use"></a>Использование внешних информационных систем | Ограничения авторизованного использования

**Идентификатор**. NIST SP 800-53 R4 AC-20 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1078, контролируемый корпорацией Майкрософт, — использование внешних информационных систем \| Ограничения авторизованного использования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb25faf85-8a16-4f28-8e15-d05c0072d64d) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1078.json) |
|[Элемент управления 1079, контролируемый корпорацией Майкрософт, — использование внешних информационных систем \| Ограничения авторизованного использования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F85c32733-7d23-4948-88da-058e2c56b60f) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1079.json) |

### <a name="use-of-external-information-systems--portable-storage-devices"></a>Использование внешних информационных систем | Переносимые запоминающие устройства

**Идентификатор**. NIST SP 800-53 R4 AC-20 (2) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1080, контролируемый корпорацией Майкрософт, — использование внешних информационных систем \| Переносимые запоминающие устройства](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F852981b4-a380-4704-aa1e-2e52d63445e5) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1080.json) |

### <a name="information-sharing"></a>Обмен информацией

**Идентификатор**. NIST SP 800-53 R4 AC-21 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1081, контролируемый корпорацией Майкрософт, — обмен информацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3867f2a9-23bb-4729-851f-c3ad98580caf) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1081.json) |
|[Элемент управления 1082, контролируемый корпорацией Майкрософт, — обмен информацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F24d480ef-11a0-4b1b-8e70-4e023bf2be23) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1082.json) |

### <a name="publicly-accessible-content"></a>Общедоступное содержимое

**Идентификатор**. NIST SP 800-53 R4 AC-22 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1083, контролируемый корпорацией Майкрософт, — общедоступное содержимое](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4e319cb6-2ca3-4a58-ad75-e67f484e50ec) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1083.json) |
|[Элемент управления 1084, контролируемый корпорацией Майкрософт, — общедоступное содержимое](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd0eb15db-dd1c-4d1d-b200-b12dd6cd060c) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1084.json) |
|[Элемент управления 1085, контролируемый корпорацией Майкрософт, — общедоступное содержимое](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F13d117e0-38b0-4bbb-aaab-563be5dd10ba) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1085.json) |
|[Элемент управления 1086, контролируемый корпорацией Майкрософт, — общедоступное содержимое](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffb321e6f-16a0-4be3-878f-500956e309c5) |Корпорация Майкрософт реализует этот элемент управления доступом |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1086.json) |

## <a name="awareness-and-training"></a>Повышение осведомленности и обучение

### <a name="security-awareness-and-training-policy-and-procedures"></a>Политики и процедуры повышения осведомленности по вопросам безопасности и обучения

**Идентификатор**. NIST SP 800-53 R4 AT-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1087, контролируемый корпорацией Майкрософт, — политика и процедуры повышения осведомленности по вопросам безопасности и обучения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F100c82ba-42e9-4d44-a2ba-94b209248583) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1087.json) |
|[Элемент управления 1088, контролируемый корпорацией Майкрософт, — политика и процедуры повышения осведомленности по вопросам безопасности и обучения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1d50f99d-1356-49c0-934a-45f742ba7783) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1088.json) |

### <a name="security-awareness-training"></a>Обучение для повышения осведомленности по вопросам безопасности

**Идентификатор**. NIST SP 800-53 R4 AT-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1089, контролируемый корпорацией Майкрософт, — обучение для повышения осведомленности по вопросам безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef080e67-0d1a-4f76-a0c5-fb9b0358485e) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1089.json) |
|[Элемент управления 1090, контролируемый корпорацией Майкрософт, — обучение для повышения осведомленности по вопросам безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2fb740e5-cbc7-4d10-8686-d1bf826652b1) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1090.json) |
|[Элемент управления 1091, контролируемый корпорацией Майкрософт, — обучение для повышения осведомленности по вопросам безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb23bd715-5d1c-4e5c-9759-9cbdf79ded9d) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1091.json) |

### <a name="security-awareness-training--insider-threat"></a>Обучение для повышения осведомленности по вопросам безопасности | Внутренняя угроза

**Идентификатор**. NIST SP 800-53 R4 AT-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1092, контролируемый корпорацией Майкрософт, — обучение для повышения осведомленности по вопросам безопасности \| Внутренняя угроза](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8a29d47b-8604-4667-84ef-90d203fcb305) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1092.json) |

### <a name="role-based-security-training"></a>Обучение мерам безопасности на основе ролей

**Идентификатор**. NIST SP 800-53 R4 AT-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1093, контролируемый корпорацией Майкрософт, — обучение мерам безопасности на основе ролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7a0bdeeb-15f4-47e8-a1da-9f769f845fdf) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1093.json) |
|[Элемент управления 1094, контролируемый корпорацией Майкрософт, — обучение мерам безопасности на основе ролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4b1853e0-8973-446b-b567-09d901d31a09) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1094.json) |
|[Элемент управления 1095, контролируемый корпорацией Майкрософт, — обучение мерам безопасности на основе ролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbc3f6f7a-057b-433e-9834-e8c97b0194f6) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1095.json) |

### <a name="role-based-security-training--practical-exercises"></a>Обучение мерам безопасности на основе ролей | Практические упражнения

**Идентификатор**. NIST SP 800-53 R4 AT-3 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1096, контролируемый корпорацией Майкрософт, — обучение мерам безопасности на основе ролей\| Практические упражнения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F420c1477-aa43-49d0-bd7e-c4abdd9addff) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1096.json) |

### <a name="role-based-security-training--suspicious-communications-and-anomalous-system-behavior"></a>Обучение мерам безопасности на основе ролей | Подозрительные подключения и аномальное поведение системы

**Идентификатор**. NIST SP 800-53 R4 AT-3 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1097, контролируемый корпорацией Майкрософт, — обучение мерам безопасности на основе ролей \| Подозрительные подключения и аномальное поведение системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf3e4836-f19e-47eb-a8cd-c3ca150452c0) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1097.json) |

### <a name="security-training-records"></a>Записи учебного курса по мерам безопасности

**Идентификатор**. NIST SP 800-53 R4 AT-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1098, контролируемый корпорацией Майкрософт, — записи учебного курса по мерам безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F84363adb-dde3-411a-9fc1-36b56737f822) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1098.json) |
|[Элемент управления 1099, контролируемый корпорацией Майкрософт, — записи учебного курса по мерам безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F01910bab-8639-4bd0-84ef-cc53b24d79ba) |Корпорация Майкрософт реализует этот элемент управления осведомленностью и обучением по вопросам безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1099.json) |

## <a name="audit-and-accountability"></a>Аудит и система отчетности

### <a name="audit-and-accountability-policy-and-procedures"></a>Процедуры и политика аудита и системы отчетности

**Идентификатор**. NIST SP 800-53 R4 AU-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1100, контролируемый корпорацией Майкрософт, — политика и процедуры аудита и системы отчетности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4057863c-ca7d-47eb-b1e0-503580cba8a4) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1100.json) |
|[Элемент управления 1101, контролируемый корпорацией Майкрософт, — политика и процедуры аудита и системы отчетности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7327b708-f0e0-457d-9d2a-527fcc9c9a65) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1101.json) |

### <a name="audit-events"></a>События аудита

**Идентификатор**. NIST SP 800-53 R4 AU-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1102, контролируемый корпорацией Майкрософт, — события аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9943c16a-c54c-4b4a-ad28-bfd938cdbf57) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1102.json) |
|[Элемент управления 1103, контролируемый корпорацией Майкрософт, — события аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F16feeb31-6377-437e-bbab-d7f73911896d) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1103.json) |
|[Элемент управления 1104, контролируемый корпорацией Майкрософт, — события аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcdd8d244-18b2-4306-a1d1-df175ae0935f) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1104.json) |
|[Элемент управления 1105, контролируемый корпорацией Майкрософт, — события аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5b73f57b-587d-4470-a344-0b0ae805f459) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1105.json) |

### <a name="audit-events--reviews-and-updates"></a>События аудита | Проверки и обновления

**Идентификатор**. NIST SP 800-53 R4 AU-2 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1106, контролируемый корпорацией Майкрософт, — события аудита \| Проверки и обновления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd2b4feae-61ab-423f-a4c5-0e38ac4464d8) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1106.json) |

### <a name="content-of-audit-records"></a>Содержимое записей аудита

**Идентификатор**. NIST SP 800-53 R4 AU-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1107, контролируемый корпорацией Майкрософт, — содержимое записей аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb29ed931-8e21-4779-8458-27916122a904) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1107.json) |

### <a name="content-of-audit-records--additional-audit-information"></a>Содержимое записей аудита | Дополнительные сведения об аудите

**Идентификатор**. NIST SP 800-53 R4 AU-3 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1108, контролируемый корпорацией Майкрософт, — содержимое записей аудита \| Дополнительные сведения об аудите](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9ad559e-c12d-415e-9a78-e50fdd7da7ba) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1108.json) |

### <a name="content-of-audit-records--centralized-management-of-planned-audit-record-content"></a>Содержимое записей аудита | Централизованное управление содержимым записей планового аудита

**Идентификатор**. NIST SP 800-53 R4 AU-3 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Проверка рабочей области Log Analytics для виртуальной машины — сообщение о несоответствии](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff47b5582-33ec-4c5c-87c0-b010a6b2e917) |Если для виртуальных машин не ведется журнал в рабочей области Log Analytics, указанной в назначении политики или инициативы, поступает сообщение о несоответствии виртуальных машин. |аудит |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_WorkspaceMismatch_VM_Audit.json) |
|[Агент Log Analytics должен быть включен для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F32133ab0-ee4b-4b44-98d6-042180979d50) |Помечает виртуальные машины как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_Audit.json) |
|[Агент Log Analytics должен быть включен в масштабируемых наборах виртуальных машин для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c3bc7b8-a64c-4e08-a9cd-7ff0f31e1138) |Помечает масштабируемые наборы виртуальных машин как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_VMSS_Audit.json) |
|[Элемент управления 1109, контролируемый корпорацией Майкрософт, — содержимое записей аудита \| Централизованное управление содержимым записей планового аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7d9ffa23-ad92-4d0d-b1f4-7db274cc2aec) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1109.json) |

### <a name="audit-storage-capacity"></a>Емкость хранилища записей аудита

**Идентификатор**. NIST SP 800-53 R4 AU-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1110, контролируемый корпорацией Майкрософт, — емкость хранилища записей аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6182bfa7-0f2a-43f5-834a-a2ddf31c13c7) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1110.json) |

### <a name="response-to-audit-processing-failures"></a>Реагирование на сбои при обработке операций аудита

**Идентификатор**. NIST SP 800-53 R4 AU-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемом экземпляре SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Проверка всех Управляемых экземпляров SQL без Расширенной защиты данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[На серверах SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Аудит серверов SQL без Расширенной защиты данных |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |
|[Аудит параметров диагностики](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7f89b1eb-583c-429a-8828-af049802c1d9) |Аудит параметров диагностики для выбранных типов ресурсов |AuditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/DiagnosticSettingsForTypes_Audit.json) |
|[Необходимо включить аудит на сервере SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa6fb4358-5bf4-4ad7-ba82-2cd2f41ce5e9) |Для отслеживания действий во всех базах данных на сервере и сохранения их в журнал аудита должен быть включен аудит вашего SQL Server. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_Audit.json) |
|[Элемент управления 1111, контролируемый корпорацией Майкрософт, — реагирование на сбои при обработке операций аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F21de687c-f15e-4e51-bf8d-f35c8619965b) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1111.json) |
|[Элемент управления 1112, контролируемый корпорацией Майкрософт, — реагирование на сбои при обработке операций аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd530aad8-4ee2-45f4-b234-c061dae683c0) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1112.json) |

### <a name="response-to-audit-processing-failures--audit-storage-capacity"></a>Реагирование на сбои при обработке операций аудита | Емкость хранилища записей аудита

**Идентификатор**. NIST SP 800-53 R4 AU-5 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1113, контролируемый корпорацией Майкрософт, — реагирование на сбои при обработке операций аудита \| Емкость хранилища записей аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F562afd61-56be-4313-8fe4-b9564aa4ba7d) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1113.json) |

### <a name="response-to-audit-processing-failures--real-time-alerts"></a>Реагирование на сбои при обработке операций аудита | Оповещения в режиме реального времени

**Идентификатор**. NIST SP 800-53 R4 AU-5 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1114, контролируемый корпорацией Майкрософт, — реагирование на сбои при обработке операций аудита \| Оповещения в режиме реального времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4c090801-59bc-4454-bb33-e0455133486a) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1114.json) |

### <a name="audit-review-analysis-and-reporting"></a>Обзор аудита, его анализ и создание отчетов

**Идентификатор**. NIST SP 800-53 R4 AU-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1115, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b653845-2ad9-4e09-a4f3-5a7c1d78353d) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1115.json) |
|[Элемент управления 1116, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5e47bc51-35d1-44b8-92af-e2f2d8b67635) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1116.json) |

### <a name="audit-review-analysis-and-reporting--process-integration"></a>Обзор аудита, его анализ и создание отчетов | Интеграция процессов

**Идентификатор**. NIST SP 800-53 R4 AU-6 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1117, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов\| Интеграция процессов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7fbfe680-6dbb-4037-963c-a621c5635902) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1117.json) |

### <a name="audit-review-analysis-and-reporting--correlate-audit-repositories"></a>Обзор аудита, анализ и создание отчетов | Сопоставление репозиториев аудита

**Идентификатор**. NIST SP 800-53 R4 AU-6 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1118, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов \| Сопоставление репозиториев аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa96f743d-a195-420d-983a-08aa06bc441e) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1118.json) |

### <a name="audit-review-analysis-and-reporting--central-review-and-analysis"></a>Обзор аудита, его анализ и создание отчетов | Централизованный обзор и анализ

**Идентификатор**. NIST SP 800-53 R4 AU-6 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Проверка рабочей области Log Analytics для виртуальной машины — сообщение о несоответствии](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff47b5582-33ec-4c5c-87c0-b010a6b2e917) |Если для виртуальных машин не ведется журнал в рабочей области Log Analytics, указанной в назначении политики или инициативы, поступает сообщение о несоответствии виртуальных машин. |аудит |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_WorkspaceMismatch_VM_Audit.json) |
|[Агент Log Analytics должен быть включен для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F32133ab0-ee4b-4b44-98d6-042180979d50) |Помечает виртуальные машины как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_Audit.json) |
|[Агент Log Analytics должен быть включен в масштабируемых наборах виртуальных машин для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c3bc7b8-a64c-4e08-a9cd-7ff0f31e1138) |Помечает масштабируемые наборы виртуальных машин как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_VMSS_Audit.json) |
|[Элемент управления 1119, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов \| Централизованный обзор и анализ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F845f6359-b764-4b40-b579-657aefe23c44) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1119.json) |

### <a name="audit-review-analysis-and-reporting--integration--scanning-and-monitoring-capabilities"></a>Обзор аудита, его анализ и создание отчетов | Интеграция, возможности сканирования и мониторинга

**Идентификатор**. NIST SP 800-53 R4 AU-6 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1120, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов \| Интеграция, возможности сканирования и мониторинга](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc69b870e-857b-458b-af02-bb234f7a00d3) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1120.json) |

### <a name="audit-review-analysis-and-reporting--correlation-with-physical-monitoring"></a>Обзор аудита, его анализ и создание отчетов | Сопоставление с физическим мониторингом

**Идентификатор**. NIST SP 800-53 R4 AU-6 (6) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1121, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов \| Сопоставление с использованием физического отслеживания](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc72b0eb9-1fc2-44e5-a866-e7cb0532f7c1) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1121.json) |

### <a name="audit-review-analysis-and-reporting--permitted-actions"></a>Обзор аудита, его анализ и создание отчетов | Разрешенные действия

**Идентификатор**. NIST SP 800-53 R4 AU-6 (7) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1122, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов\| Разрешенные действия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F243ec95e-800c-49d4-ba52-1fdd9f6b8b57) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1122.json) |

### <a name="audit-review-analysis-and-reporting--audit-level-adjustment"></a>Обзор аудита, его анализ и создание отчетов | Изменения на уровне аудита

**Идентификатор**. NIST SP 800-53 R4 AU-6 (10) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1123, контролируемый корпорацией Майкрософт, — обзор аудита, анализ результатов и создание отчетов \| Настройка уровней аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F03996055-37a4-45a5-8b70-3f1caa45f87d) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1123.json) |

### <a name="audit-reduction-and-report-generation"></a>Сокращение аудита и создание отчетов

**Идентификатор**. NIST SP 800-53 R4 AU-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1124, контролируемый корпорацией Майкрософт, — сокращение аудита и создание отчетов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc10152dd-78f8-4335-ae2d-ad92cc028da4) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1124.json) |
|[Элемент управления 1125, контролируемый корпорацией Майкрософт, — сокращение аудита и создание отчетов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc6ce745a-670e-47d3-a6c4-3cfe5ef00c10) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1125.json) |

### <a name="audit-reduction-and-report-generation--automatic-processing"></a>Сокращение аудита и создание отчетов | Автоматическая обработка

**Идентификатор**. NIST SP 800-53 R4 AU-7 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1126, контролируемый корпорацией Майкрософт, — сокращение аудита и создание отчетов\| Автоматическая обработка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7f37f71b-420f-49bf-9477-9c0196974ecf) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1126.json) |

### <a name="time-stamps"></a>Метки времени

**Идентификатор**. NIST SP 800-53 R4 AU-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1127, контролируемый корпорацией Майкрософт, — метки времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3ce328db-aef3-48ed-9f81-2ab7cf839c66) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1127.json) |
|[Элемент управления 1128, контролируемый корпорацией Майкрософт, — метки времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef212163-3bc4-4e86-bcf8-705127086393) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1128.json) |

### <a name="time-stamps--synchronization-with-authoritative-time-source"></a>Метки времени | Синхронизация с достоверным источником времени

**Идентификатор**. NIST SP 800-53 R4 AU-8 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1129, контролируемый корпорацией Майкрософт, — метки времени \| Синхронизация с достоверным источником времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F71bb965d-4047-4623-afd4-b8189a58df5d) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1129.json) |
|[Элемент управления 1130, контролируемый корпорацией Майкрософт, — метки времени \| Синхронизация с достоверным источником времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffd7c4c1d-51ee-4349-9dab-89a7f8c8d102) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1130.json) |

### <a name="protection-of-audit-information"></a>Защита сведений аудита

**Идентификатор**. NIST SP 800-53 R4 AU-9 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1131, контролируемый корпорацией Майкрософт, — защита сведений аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb472a17e-c2bc-493f-b50b-42d55a346962) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1131.json) |

### <a name="protection-of-audit-information--audit-backup-on-separate-physical-systems--components"></a>Защита сведений аудита | Резервное копирование сведений аудита в отдельных физических системах или компонентах

**Идентификатор**. NIST SP 800-53 R4 AU-9 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1132, контролируемый корпорацией Майкрософт, — защита сведений аудита \| Резервное копирование сведений аудита в отдельных физических системах или компонентах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F05938e10-cdbd-4a54-9b2b-1cbcfc141ad0) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1132.json) |

### <a name="protection-of-audit-information--cryptographic-protection"></a>Защита сведений аудита | Криптографическая защита

**Идентификатор**. NIST SP 800-53 R4 AU-9 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1133, контролируемый корпорацией Майкрософт, — защита сведений аудита \| Криптографическая защита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F90b60a09-133d-45bc-86ef-b206a6134bbe) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1133.json) |

### <a name="protection-of-audit-information--access-by-subset-of-privileged-users"></a>Защита сведений аудита | Доступ подмножества привилегированных пользователей

**Идентификатор**. NIST SP 800-53 R4 AU-9 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1134, контролируемый корпорацией Майкрософт, — защита сведений аудита \| Доступ подмножества привилегированных пользователей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4e95f70e-181c-4422-9da2-43079710c789) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1134.json) |

### <a name="non-repudiation"></a>Неотрекаемость

**Идентификатор**. NIST SP 800-53 R4 AU-10 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1135, контролируемый корпорацией Майкрософт, — неотрекаемость](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9c308b6b-2429-4b97-86cf-081b8e737b04) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1135.json) |

### <a name="audit-record-retention"></a>Хранение записей аудита

**Идентификатор**. NIST SP 800-53 R4 AU-11 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1136, контролируемый корпорацией Майкрософт, — хранение записей аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F97ed5bac-a92f-4f6d-a8ed-dc094723597c) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1136.json) |

### <a name="audit-generation"></a>Создание записей аудита

**Идентификатор**. NIST SP 800-53 R4 AU-12 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемом экземпляре SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Проверка всех Управляемых экземпляров SQL без Расширенной защиты данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[На серверах SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Аудит серверов SQL без Расширенной защиты данных |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |
|[Аудит параметров диагностики](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7f89b1eb-583c-429a-8828-af049802c1d9) |Аудит параметров диагностики для выбранных типов ресурсов |AuditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/DiagnosticSettingsForTypes_Audit.json) |
|[Проверка рабочей области Log Analytics для виртуальной машины — сообщение о несоответствии](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff47b5582-33ec-4c5c-87c0-b010a6b2e917) |Если для виртуальных машин не ведется журнал в рабочей области Log Analytics, указанной в назначении политики или инициативы, поступает сообщение о несоответствии виртуальных машин. |аудит |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_WorkspaceMismatch_VM_Audit.json) |
|[Необходимо включить аудит на сервере SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa6fb4358-5bf4-4ad7-ba82-2cd2f41ce5e9) |Для отслеживания действий во всех базах данных на сервере и сохранения их в журнал аудита должен быть включен аудит вашего SQL Server. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_Audit.json) |
|[Агент Log Analytics должен быть включен для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F32133ab0-ee4b-4b44-98d6-042180979d50) |Помечает виртуальные машины как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_Audit.json) |
|[Агент Log Analytics должен быть включен в масштабируемых наборах виртуальных машин для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c3bc7b8-a64c-4e08-a9cd-7ff0f31e1138) |Помечает масштабируемые наборы виртуальных машин как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_VMSS_Audit.json) |
|[Элемент управления 1137, контролируемый корпорацией Майкрософт, — создание записей аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4344df62-88ab-4637-b97b-bcaf2ec97e7c) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1137.json) |
|[Элемент управления 1138, контролируемый корпорацией Майкрософт, — создание записей аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9c284fc0-268a-4f29-af44-3c126674edb4) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1138.json) |
|[Элемент управления 1139, контролируемый корпорацией Майкрософт, — создание записей аудита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4ed62522-de00-4dda-9810-5205733d2f34) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1139.json) |

### <a name="audit-generation--system-wide--time-correlated-audit-trail"></a>Создание аудита | Журнал аудита во всей системе с сопоставлением по времени

**Идентификатор**. NIST SP 800-53 R4 AU-12 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1140, контролируемый корпорацией Майкрософт, — создание записей аудита \| Журнал аудита во всей системе с сопоставлением по времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F90d8b8ad-8ee3-4db7-913f-2a53fcff5316) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1140.json) |

### <a name="audit-generation--changes-by-authorized-individuals"></a>Создание записей аудита | Изменения авторизованными пользователями

**Идентификатор**. NIST SP 800-53 R4 AU-12 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1141, контролируемый корпорацией Майкрософт, — создание записей аудита \| Изменения авторизованными пользователями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6fdefbf4-93e7-4513-bc95-c1858b7093e0) |Корпорация Майкрософт реализует этот элемент управления аудитом и системой отчетности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1141.json) |

## <a name="security-assessment-and-authorization"></a>Оценка безопасности и полномочия

### <a name="security-assessment-and-authorization-policy-and-procedures"></a>Политика и процедуры оценки безопасности и авторизации

**Идентификатор**. NIST SP 800-53 R4 CA-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1142, контролируемый корпорацией Майкрософт, — политика и процедуры оценки безопасности и авторизации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F01524fa8-4555-48ce-ba5f-c3b8dcef5147) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1142.json) |
|[Элемент управления 1143, контролируемый корпорацией Майкрософт, — политика и процедуры оценки безопасности и авторизации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7c6de11b-5f51-4f7c-8d83-d2467c8a816e) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1143.json) |

### <a name="security-assessments"></a>Оценки безопасности

**Идентификатор**. NIST SP 800-53 R4 CA-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1144, контролируемый корпорацией Майкрософт, — оценки безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2fa15ff1-a693-4ee4-b094-324818dc9a51) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1144.json) |
|[Элемент управления 1145, контролируемый корпорацией Майкрософт, — оценки безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa0724970-9c75-4a64-a225-a28002953f28) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1145.json) |
|[Элемент управления 1146, контролируемый корпорацией Майкрософт, — оценки безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdd83410c-ecb6-4547-8f14-748c3cbdc7ac) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1146.json) |
|[Элемент управления 1147, контролируемый корпорацией Майкрософт, — оценки безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8fef824a-29a8-4a4c-88fc-420a39c0d541) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1147.json) |

### <a name="security-assessments--independent-assessors"></a>Оценки безопасности | Независимые аудиторы

**Идентификатор**. NIST SP 800-53 R4 CA-2 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1148, контролируемый корпорацией Майкрософт, — оценки безопасности \| Независимые аудиторы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F28e62650-c7c2-4786-bdfa-17edc1673902) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1148.json) |

### <a name="security-assessments--specialized-assessments"></a>Оценки безопасности | Специализированные оценки

**Идентификатор**. NIST SP 800-53 R4 CA-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1149, контролируемый корпорацией Майкрософт, — оценки безопасности \| Специализированные оценки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2e1b855b-a013-481a-aeeb-2bcb129fd35d) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1149.json) |

### <a name="security-assessments--external-organizations"></a>Оценки безопасности | Внешние организации

**Идентификатор**. NIST SP 800-53 R4 CA-2 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1150, контролируемый корпорацией Майкрософт, — оценки безопасности \| Внешние организации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd630429d-e763-40b1-8fba-d20ba7314afb) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1150.json) |

### <a name="system-interconnections"></a>Взаимодействие системы

**Идентификатор**. NIST SP 800-53 R4 CA-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1151, контролируемый корпорацией Майкрософт, — взаимосвязи в системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F347e3b69-7fb7-47df-a8ef-71a1a7b44bca) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1151.json) |
|[Элемент управления 1152, контролируемый корпорацией Майкрософт, — взаимосвязи в системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbeff0acf-7e67-40b2-b1ca-1a0e8205cf1b) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1152.json) |
|[Элемент управления 1153, контролируемый корпорацией Майкрософт, — взаимосвязи в системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F61cf3125-142c-4754-8a16-41ab4d529635) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1153.json) |

### <a name="system-interconnections--unclassified-non-national-security-system-connections"></a>Взаимодействие системы | Взаимодействие неклассифицируемой системы, не относящейся к обеспечению национальной безопасности

**Идентификатор**. NIST SP 800-53 R4 CA-3 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1154, контролируемый корпорацией Майкрософт, — взаимосвязи в системе \| Взаимосвязи неклассифицируемой системы, не относящейся к обеспечению национальной безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe757ceb9-93b3-45fe-a4f4-f43f64f1ac5a) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1154.json) |

### <a name="system-interconnections--restrictions-on-external-system-connections"></a>Взаимодействие системы | Ограничения подключений к внешним системам

**Идентификатор**. NIST SP 800-53 R4 CA-3 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1155, контролируемый корпорацией Майкрософт, — взаимосвязи в системе \| Ограничения подключений к внешним системам](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4d33f9f1-12d0-46ad-9fbd-8f8046694977) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1155.json) |

### <a name="plan-of-action-and-milestones"></a>План действий и вехи

**Идентификатор**. NIST SP 800-53 R4 CA-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1156, контролируемый корпорацией Майкрософт, — план действий и вехи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4d52e864-9a3b-41ee-8f03-520815fe5378) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1156.json) |
|[Элемент управления 1157, контролируемый корпорацией Майкрософт, — план действий и вехи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F15495367-cf68-464c-bbc3-f53ca5227b7a) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1157.json) |

### <a name="security-authorization"></a>Авторизация в системе безопасности

**Идентификатор**. NIST SP 800-53 R4 CA-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1158, контролируемый корпорацией Майкрософт, — авторизация в системе безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffff50cf2-28eb-45b4-b378-c99412688907) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1158.json) |
|[Элемент управления 1159, контролируемый корпорацией Майкрософт, — авторизация в системе безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0925f098-7877-450b-8ba4-d1e55f2d8795) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1159.json) |
|[Элемент управления 1160, контролируемый корпорацией Майкрософт, — авторизация в системе безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3e797ca6-2aa8-4333-b335-7036f1110c05) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1160.json) |

### <a name="continuous-monitoring"></a>Непрерывный мониторинг

**Идентификатор**. NIST SP 800-53 R4 CA-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1161, контролируемый корпорацией Майкрософт, — непрерывный мониторинг](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe2f8f6c6-dde4-436b-a79d-bc50e129eb3a) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1161.json) |
|[Элемент управления 1162, контролируемый корпорацией Майкрософт, — непрерывный мониторинг](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5770f3d6-8c2b-4f6f-bf0e-c8c8fc36d592) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1162.json) |
|[Элемент управления 1163, контролируемый корпорацией Майкрософт, — непрерывный мониторинг](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F961663a1-8a91-4e59-b6f5-1eee57c0f49c) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1163.json) |
|[Элемент управления 1164, контролируемый корпорацией Майкрософт, — непрерывный мониторинг](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0fb8d3ce-9e96-481c-9c68-88d4e3019310) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1164.json) |
|[Элемент управления 1165, контролируемый корпорацией Майкрософт, — непрерывный мониторинг](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47e10916-6c9e-446b-b0bd-ff5fd439d79d) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1165.json) |
|[Элемент управления 1166, контролируемый корпорацией Майкрософт, — непрерывный мониторинг](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbb02733d-3cc5-4bb0-a6cd-695ba2c2272e) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1166.json) |
|[Элемент управления 1167, контролируемый корпорацией Майкрософт, — непрерывный мониторинг](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcbb2be76-4891-430b-95a7-ca0b0a3d1300) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1167.json) |

### <a name="continuous-monitoring--independent-assessment"></a>Непрерывный мониторинг | Независимая оценка

**Идентификатор**. NIST SP 800-53 R4 CA-7 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1168, контролируемый корпорацией Майкрософт, — непрерывный мониторинг \| Независимая оценка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82409f9e-1f32-4775-bf07-b99d53a91b06) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1168.json) |

### <a name="continuous-monitoring--trend-analyses"></a>Непрерывный мониторинг | Анализ трендов

**Идентификатор**. NIST SP 800-53 R4 CA-7 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1169, контролируемый корпорацией Майкрософт, — непрерывный мониторинг \| Анализ тенденций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe7ba2cb3-5675-4468-8b50-8486bdd998a5) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1169.json) |

### <a name="penetration-testing"></a>Выполнение тестов на проникновение

**Идентификатор**. NIST SP 800-53 R4 CA-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1170, контролируемый корпорацией Майкрософт, — выполнение тестов на проникновение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8b78b9b3-ee3c-48e0-a243-ed6dba5b7a12) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1170.json) |

### <a name="penetration-testing--independent-penetration-agent-or-team"></a>Выполнение тестов на проникновение | Независимый агент или группа по выполнению тестов на проникновение

**Идентификатор**. NIST SP 800-53 R4 CA-8 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1171, контролируемый корпорацией Майкрософт, — выполнение тестов на проникновение \| Независимый агент или группа по выполнению тестов на проникновение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d4820bc-8b61-4982-9501-2123cb776c00) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1171.json) |

### <a name="internal-system-connections"></a>Внутренние подключения системы

**Идентификатор**. NIST SP 800-53 R4 CA-9 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1172, контролируемый корпорацией Майкрософт, — внутренние подключения системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb43e946e-a4c8-4b92-8201-4a39331db43c) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1172.json) |
|[Элемент управления 1173, контролируемый корпорацией Майкрософт, — внутренние подключения системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4aff9e7-2e60-46fa-86be-506b79033fc5) |Корпорация Майкрософт реализует этот элемент управления оценкой безопасности и полномочиями |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1173.json) |

## <a name="configuration-management"></a>Управление конфигурацией

### <a name="configuration-management-policy-and-procedures"></a>Политика и процедуры управления конфигурацией

**Идентификатор**. NIST SP 800-53 R4 CM-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1174, контролируемый корпорацией Майкрософт, — политика и процедуры управления конфигурацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F42a9a714-8fbb-43ac-b115-ea12d2bd652f) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1174.json) |
|[Элемент управления 1175, контролируемый корпорацией Майкрософт, — политика и процедуры управления конфигурацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6dab4254-c30d-4bb7-ae99-1d21586c063c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1175.json) |

### <a name="baseline-configuration"></a>Базовая конфигурация

**Идентификатор**. NIST SP 800-53 R4 CM-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1176, контролируемый корпорацией Майкрософт, — базовая конфигурация](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc30690a5-7bf3-467f-b0cd-ef5c7c7449cd) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1176.json) |

### <a name="baseline-configuration--reviews-and-updates"></a>Базовая конфигурация | Проверки и обновления

**Идентификатор**. NIST SP 800-53 R4 CM-2 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1177, контролируемый корпорацией Майкрософт, — базовая конфигурация \| Проверки и обновления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F63dbc7a8-e20b-4d38-b857-a7f6c0cd94bc) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1177.json) |
|[Элемент управления 1178, контролируемый корпорацией Майкрософт, — базовая конфигурация \| Проверки и обновления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7818b8f4-47c6-441a-90ae-12ce04e99893) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1178.json) |
|[Элемент управления 1179, контролируемый корпорацией Майкрософт, — базовая конфигурация \| Проверки и обновления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3f9ce557-c8ab-4e6c-bb2c-9b8ed002c46c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1179.json) |

### <a name="baseline-configuration--automation-support-for-accuracy--currency"></a>Базовая конфигурация | Автоматизированная поддержка точности и валюты

**Идентификатор**. NIST SP 800-53 R4 CM-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1180, контролируемый корпорацией Майкрософт, — базовая конфигурация \| Поддержка автоматизации для точности и актуальности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F874e7880-a067-42a7-bcbe-1a340f54c8cc) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1180.json) |

### <a name="baseline-configuration--retention-of-previous-configurations"></a>Базовая конфигурация | Хранение предыдущих конфигураций

**Идентификатор**. NIST SP 800-53 R4 CM-2 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1181, контролируемый корпорацией Майкрософт, — базовая конфигурация \| Хранение предыдущих конфигураций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F21839937-d241-4fa5-95c6-b669253d9ab9) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1181.json) |

### <a name="baseline-configuration--configure-systems-components-or-devices-for-high-risk-areas"></a>Базовая конфигурация | Настройка систем, компонентов или устройств для областей с высоким уровнем риска

**Идентификатор**. NIST SP 800-53 R4 CM-2 (7) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1182, контролируемый корпорацией Майкрософт, — базовая конфигурация \| Настройка систем, компонентов или устройств для областей с высоким уровнем риска](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f34f554-da4b-4786-8d66-7915c90893da) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1182.json) |
|[Элемент управления 1183, контролируемый корпорацией Майкрософт, — базовая конфигурация \| Настройка систем, компонентов или устройств для областей с высоким уровнем риска](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5352e3e0-e63a-452e-9e5f-9c1d181cff9c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1183.json) |

### <a name="configuration-change-control"></a>Управление изменениями конфигурации

**Идентификатор**. NIST SP 800-53 R4 CM-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1184, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F13579d0e-0ab0-4b26-b0fb-d586f6d7ed20) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1184.json) |
|[Элемент управления 1185, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6420cd73-b939-43b7-9d99-e8688fea053c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1185.json) |
|[Элемент управления 1186, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb95ba3bd-4ded-49ea-9d10-c6f4b680813d) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1186.json) |
|[Элемент управления 1187, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9f2b2f9e-4ba6-46c3-907f-66db138b6f85) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1187.json) |
|[Элемент управления 1188, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbb20548a-c926-4e4d-855c-bcddc6faf95e) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1188.json) |
|[Элемент управления 1189, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fee45e02a-4140-416c-82c4-fecfea660b9d) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1189.json) |
|[Элемент управления 1190, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc66a3d1e-465b-4f28-9da5-aef701b59892) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1190.json) |

### <a name="configuration-change-control--automated-document--notification--prohibition-of-changes"></a>Управление изменениями конфигурации | Автоматическое документирование, отправление уведомлений или запрет изменений

**Идентификатор**. NIST SP 800-53 R4 CM-3 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1191, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Автоматическое документирование, отправление уведомлений или запрет изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7f26a61b-a74d-467c-99cf-63644db144f7) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1191.json) |
|[Элемент управления 1192, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Автоматическое документирование, отправление уведомлений или запрет изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4ebd97f7-b105-4f50-8daf-c51465991240) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1192.json) |
|[Элемент управления 1193, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Автоматическое документирование, отправление уведомлений или запрет изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff5fd629f-3075-4cae-ab53-bad65495a4ac) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1193.json) |
|[Элемент управления 1194, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Автоматическое документирование, отправление уведомлений или запрет изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbc34667f-397e-4a65-9b72-d0358f0b6b09) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1194.json) |
|[Элемент управления 1195, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Автоматическое документирование, отправление уведомлений или запрет изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd1e1d65c-1013-4484-bd54-991332e6a0d2) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1195.json) |
|[Элемент управления 1196, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Автоматическое документирование, отправление уведомлений или запрет изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4e7f4ea4-dd62-44f6-8886-ac6137cf52b0) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1196.json) |

### <a name="configuration-change-control--test--validate--document-changes"></a>Управление изменениями конфигурации | Тестирование, проверка и документирование изменений

**Идентификатор**. NIST SP 800-53 R4 CM-3 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1197, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Тестирование, проверка и документирование изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa20d2eaa-88e2-4907-96a2-8f3a05797e5c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1197.json) |

### <a name="configuration-change-control--security-representative"></a>Управление изменениями конфигурации | Представитель службы безопасности

**Идентификатор**. NIST SP 800-53 R4 CM-3 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1198, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Представитель службы безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff56be5c3-660b-4c61-9078-f67cf072c356) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1198.json) |

### <a name="configuration-change-control--cryptography-management"></a>Управление изменениями конфигурации | Управление шифрованием

**Идентификатор**. NIST SP 800-53 R4 CM-3 (6) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1199, контролируемый корпорацией Майкрософт, — управление изменениями конфигурации \| Управление шифрованием](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa9a08d1c-09b1-48f1-90ea-029bbdf7111e) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1199.json) |

### <a name="security-impact-analysis"></a>Анализ влияния на безопасность

**Идентификатор**. NIST SP 800-53 R4 CM-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1200, контролируемый корпорацией Майкрософт, — анализ влияния на безопасность](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe98fe9d7-2ed3-44f8-93b7-24dca69783ff) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1200.json) |

### <a name="security-impact-analysis--separate-test-environments"></a>Анализ влияния на безопасность | Отдельные тестовые среды

**Идентификатор**. NIST SP 800-53 R4 CM-4 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1201, контролируемый корпорацией Майкрософт, — анализ влияния на безопасность \| Отдельные тестовые среды](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7daef997-fdd3-461b-8807-a608a6dd70f1) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1201.json) |

### <a name="access-restrictions-for-change"></a>Ограничение прав на доступ для внесения изменений

**Идентификатор**. NIST SP 800-53 R4 CM-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1202, контролируемый корпорацией Майкрософт, — ограничение прав на доступ для внесения изменений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F40a2a83b-74f2-4c02-ae65-f460a5d2792a) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1202.json) |

### <a name="access-restrictions-for-change--automated-access-enforcement--auditing"></a>Ограничения прав на доступ для внесения изменений | Автоматическое применение ограничений доступа и аудит

**Идентификатор**. NIST SP 800-53 R4 CM-5 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1203, контролируемый корпорацией Майкрософт, — ограничение прав на доступ для внесения изменений \| Автоматическое применение ограничений доступа и аудит](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9012d14-e3e6-4d7b-b926-9f37b5537066) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1203.json) |

### <a name="access-restrictions-for-change--review-system-changes"></a>Ограничения прав на доступ для внесения изменений | Просмотр изменений системы

**Идентификатор**. NIST SP 800-53 R4 CM-5 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1204, контролируемый корпорацией Майкрософт, — ограничение прав на доступ для внесения изменений \| Просмотр изменений системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0f4f6750-d1ab-4a4c-8dfd-af3237682665) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1204.json) |

### <a name="access-restrictions-for-change--signed-components"></a>Ограничение прав на внесение изменений | Подписанные компоненты

**Идентификатор**. NIST SP 800-53 R4 CM-5 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1205, контролируемый корпорацией Майкрософт, — ограничение прав на доступ для внесения изменений \| Подписанные компоненты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5b070cab-0fb8-4e48-ad29-fc90b4c2797c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1205.json) |

### <a name="access-restrictions-for-change--limit-production--operational-privileges"></a>Ограничения прав на доступ для внесения изменений | Ограничение привилегий в рабочей или операционной среде

**Идентификатор**. NIST SP 800-53 R4 CM-5 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1206, контролируемый корпорацией Майкрософт, — ограничение прав на доступ для внесения изменений \| Ограничение привилегий в рабочей или операционной среде](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe0de232d-02a0-4652-872d-88afb4ae5e91) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1206.json) |
|[Элемент управления 1207, контролируемый корпорацией Майкрософт, — ограничение прав на доступ для внесения изменений \| Ограничение привилегий в рабочей или операционной среде](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8713a0ed-0d1e-4d10-be82-83dffb39830e) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1207.json) |

### <a name="configuration-settings"></a>Параметры конфигурации

**Идентификатор**. NIST SP 800-53 R4 CM-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1208, контролируемый корпорацией Майкрософт, — параметры конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5ea87673-d06b-456f-a324-8abcee5c159f) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1208.json) |
|[Элемент управления 1209, контролируемый корпорацией Майкрософт, — параметры конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fce669c31-9103-4552-ae9c-cdef4e03580d) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1209.json) |
|[Элемент управления 1210, контролируемый корпорацией Майкрософт, — параметры конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3502c968-c490-4570-8167-1476f955e9b8) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1210.json) |
|[Элемент управления 1211, контролируемый корпорацией Майкрософт, — параметры конфигурации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6a8b9dc8-6b00-4701-aa96-bba3277ebf50) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1211.json) |

### <a name="configuration-settings--automated-central-management--application--verification"></a>Параметры конфигурации | Автоматизированное централизованное управление, применение и проверка

**Идентификатор**. NIST SP 800-53 R4 CM-6 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1212, контролируемый корпорацией Майкрософт, — параметры конфигурации \| Автоматизированное централизованное управление, применение и проверка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F56d970ee-4efc-49c8-8a4e-5916940d784c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1212.json) |

### <a name="configuration-settings--respond-to-unauthorized-changes"></a>Параметры конфигурации | Реагирование на несанкционированные изменения

**Идентификатор**. NIST SP 800-53 R4 CM-6 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1213, контролируемый корпорацией Майкрософт, — параметры конфигурации \| Реагирование на несанкционированные изменения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F81f11e32-a293-4a58-82cd-134af52e2318) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1213.json) |

### <a name="least-functionality"></a>Минимальная функциональность

**Идентификатор**. NIST SP 800-53 R4 CM-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1214, контролируемый корпорацией Майкрософт, — минимальная функциональность](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff714a4e2-b580-47b6-ae8c-f2812d3750f3) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1214.json) |
|[Элемент управления 1215, контролируемый корпорацией Майкрософт, — минимальная функциональность](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F88fc93e8-4745-4785-b5a5-b44bb92c44ff) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1215.json) |

### <a name="least-functionality--periodic-review"></a>Минимальная функциональность | Периодическая проверка

**Идентификатор**. NIST SP 800-53 R4 CM-7 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1216, контролируемый корпорацией Майкрософт, — минимальные функции \| Периодическая проверка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7894fe6a-f5cb-44c8-ba90-c3f254ff9484) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1216.json) |
|[Элемент управления 1217, контролируемый корпорацией Майкрософт, — минимальные функции \| Периодическая проверка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fedea4f20-b02c-4115-be75-86c080e5c0ed) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1217.json) |

### <a name="least-functionality--prevent-program-execution"></a>Минимальная функциональность | Предотвращение выполнения программы

**Идентификатор**. NIST SP 800-53 R4 CM-7 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На компьютерах должны быть включены адаптивные элементы управления приложениями для определения безопасных приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Включите элементы управления приложениями, чтобы определить список проверенных безопасных приложений, выполняющихся на ваших компьютерах, и настроить получение оповещений о запуске других приложений. Это поможет защитить компьютеры от вредоносных программ. Чтобы упростить процесс настройки и обслуживания правил, Центр безопасности использует машинное обучение для анализа приложений, выполняющихся на каждом компьютере, и предлагает список проверенных безопасных приложений. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |
|[Элемент управления 1218, контролируемый корпорацией Майкрософт, — минимальная функциональность \| Предотвращение выполнения программы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4a1d0394-b9f5-493e-9e83-563fd0ac4df8) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1218.json) |

### <a name="least-functionality--authorized-software--whitelisting"></a>Минимальная функциональность | Авторизованное программное обеспечение и список разрешений

**Идентификатор**. NIST SP 800-53 R4 CM-7 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На компьютерах должны быть включены адаптивные элементы управления приложениями для определения безопасных приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Включите элементы управления приложениями, чтобы определить список проверенных безопасных приложений, выполняющихся на ваших компьютерах, и настроить получение оповещений о запуске других приложений. Это поможет защитить компьютеры от вредоносных программ. Чтобы упростить процесс настройки и обслуживания правил, Центр безопасности использует машинное обучение для анализа приложений, выполняющихся на каждом компьютере, и предлагает список проверенных безопасных приложений. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |
|[Элемент управления 1219, контролируемый корпорацией Майкрософт, — минимальная функциональность \| Авторизованное программное обеспечение и список разрешений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2a39ac75-622b-4c88-9a3f-45b7373f7ef7) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1219.json) |
|[Элемент управления 1220, контролируемый корпорацией Майкрософт, — минимальная функциональность \| Авторизованное программное обеспечение и список разрешений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc40f31a7-81e1-4130-99e5-a02ceea2a1d6) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1220.json) |
|[Элемент управления 1221, контролируемый корпорацией Майкрософт, — минимальная функциональность \| Авторизованное программное обеспечение и список разрешений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22589a07-0007-486a-86ca-95355081ae2a) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1221.json) |

### <a name="information-system-component-inventory"></a>Учет компонентов информационной системы

**Идентификатор**. NIST SP 800-53 R4 CM-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1222, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffb39e62f-6bda-4558-8088-ec03d5670914) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1222.json) |
|[Элемент управления 1223, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F05a1bb01-ad5a-49c1-aad3-b0c893b2ec3a) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1223.json) |

### <a name="information-system-component-inventory--updates-during-installations--removals"></a>Учет компонентов информационной системы | Обновления в процессе установки или удаления

**Идентификатор**. NIST SP 800-53 R4 CM-8 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1224, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы \| Обновления в процессе установки или удаления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F28cfa30b-7f72-47ce-ba3b-eed26c8d2c82) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1224.json) |

### <a name="information-system-component-inventory--automated-maintenance"></a>Учет компонентов информационной системы | Автоматизированное обслуживание

**Идентификатор**. NIST SP 800-53 R4 CM-8 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1225, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы \| Автоматизированное обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8d096fe0-f510-4486-8b4d-d17dc230980b) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1225.json) |

### <a name="information-system-component-inventory--automated-unauthorized-component-detection"></a>Учет компонентов информационной системы | Автоматическое обнаружение несанкционированных компонентов

**Идентификатор**. NIST SP 800-53 R4 CM-8 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1226, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы \| Автоматическое обнаружение несанкционированных компонентов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc158eb1c-ae7e-4081-8057-d527140c4e0c) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1226.json) |
|[Элемент управления 1227, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы \| Автоматическое обнаружение несанкционированных компонентов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F03b78f5e-4877-4303-b0f4-eb6583f25768) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1227.json) |

### <a name="information-system-component-inventory--accountability-information"></a>Учет компонентов информационной системы | Сведения об отчетности

**Идентификатор**. NIST SP 800-53 R4 CM-8 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1228, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы \| Сведения об отчетности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F39c54140-5902-4079-8bb5-ad31936fe764) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1228.json) |

### <a name="information-system-component-inventory--no-duplicate-accounting-of-components"></a>Учет компонентов информационной системы | Отсутствие повторяющихся компонентов в отчетности

**Идентификатор**. NIST SP 800-53 R4 CM-8 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1229, контролируемый корпорацией Майкрософт, — учет компонентов информационной системы \| Отсутствие повторяющихся компонентов в отчетности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F03752212-103c-4ab8-a306-7e813022ca9d) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1229.json) |

### <a name="configuration-management-plan"></a>План управления конфигурацией

**Идентификатор**. NIST SP 800-53 R4 CM-9 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1230, контролируемый корпорацией Майкрософт, — план управления конфигурацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F11158848-f679-4e9b-aa7b-9fb07d945071) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1230.json) |
|[Элемент управления 1231, контролируемый корпорацией Майкрософт, — план управления конфигурацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F244e0c05-cc45-4fe7-bf36-42dcf01f457d) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1231.json) |
|[Элемент управления 1232, контролируемый корпорацией Майкрософт, — план управления конфигурацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F396ba986-eac1-4d6d-85c4-d3fda6b78272) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1232.json) |
|[Элемент управления 1233, контролируемый корпорацией Майкрософт, — план управления конфигурацией](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d79001f-95fe-45d0-8736-f217e78c1f57) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1233.json) |

### <a name="software-usage-restrictions"></a>Ограничения на использование программного обеспечения

**Идентификатор**. NIST SP 800-53 R4 CM-10 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1234, контролируемый корпорацией Майкрософт, — ограничения на использование программного обеспечения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb293f881-361c-47ed-b997-bc4e2296bc0b) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1234.json) |
|[Элемент управления 1235, контролируемый корпорацией Майкрософт, — ограничения на использование программного обеспечения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc49c610b-ece4-44b3-988c-2172b70d6e46) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1235.json) |
|[Элемент управления 1236, контролируемый корпорацией Майкрософт, — ограничения на использование программного обеспечения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9ba3ed84-c768-4e18-b87c-34ef1aff1b57) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1236.json) |

### <a name="software-usage-restrictions--open-source-software"></a>Ограничения на использование программного обеспечения | Программное обеспечение с открытым исходным кодом

**Идентификатор**. NIST SP 800-53 R4 CM-10 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1237, контролируемый корпорацией Майкрософт, — ограничения на использование программного обеспечения \| Программное обеспечение с открытым исходным кодом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe80b6812-0bfa-4383-8223-cdd86a46a890) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1237.json) |

### <a name="user-installed-software"></a>Программное обеспечение, установленное пользователем

**Идентификатор**. NIST SP 800-53 R4 CM-11 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На компьютерах должны быть включены адаптивные элементы управления приложениями для определения безопасных приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Включите элементы управления приложениями, чтобы определить список проверенных безопасных приложений, выполняющихся на ваших компьютерах, и настроить получение оповещений о запуске других приложений. Это поможет защитить компьютеры от вредоносных программ. Чтобы упростить процесс настройки и обслуживания правил, Центр безопасности использует машинное обучение для анализа приложений, выполняющихся на каждом компьютере, и предлагает список проверенных безопасных приложений. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |
|[Элемент управления 1238, контролируемый корпорацией Майкрософт, — программное обеспечение, установленное пользователем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa36cedd4-3ffd-4b1f-8b18-aa71d8d87ce1) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1238.json) |
|[Элемент управления 1239, контролируемый корпорацией Майкрософт, — программное обеспечение, установленное пользователем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0be51298-f643-4556-88af-d7db90794879) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1239.json) |
|[Элемент управления 1240, контролируемый корпорацией Майкрософт, — программное обеспечение, установленное пользователем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F129eb39f-d79a-4503-84cd-92f036b5e429) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1240.json) |

### <a name="user-installed-software--alerts-for-unauthorized-installations"></a>Программное обеспечение, установленное пользователем | Оповещения о несанкционированных установках

**Идентификатор**. NIST SP 800-53 R4 CM-11 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1241, контролируемый корпорацией Майкрософт, — программное обеспечение, установленное пользователем \| Оповещения о несанкционированных установках](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feca4d7b2-65e2-4e04-95d4-c68606b063c3) |Корпорация Майкрософт реализует этот элемент управления конфигурацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1241.json) |

## <a name="contingency-planning"></a>Планирование действий на непредвиденные случаи

### <a name="contingency-planning-policy-and-procedures"></a>Политика и процедуры составления плана на непредвиденные случаи

**Идентификатор**. NIST SP 800-53 R4 CP-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1242, контролируемый корпорацией Майкрософт, — политика и процедуры планирования действий на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf3b3293-667a-445e-a722-fa0b0afc0958) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1242.json) |
|[Элемент управления 1243, контролируемый корпорацией Майкрософт, — политика и процедуры планирования действий на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fca9a4469-d6df-4ab2-a42f-1213c396f0ec) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1243.json) |

### <a name="contingency-plan"></a>План на непредвиденные случаи

**Идентификатор**. NIST SP 800-53 R4 CP-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1244, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6a13a8f8-c163-4b1b-8554-d63569dab937) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1244.json) |
|[Элемент управления 1245, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa0e45314-57b8-4623-80cd-bbb561f59516) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1245.json) |
|[Элемент управления 1246, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F398eb61e-8111-40d5-a0c9-003df28f1753) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1246.json) |
|[Элемент управления 1247, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4e666db5-b2ef-4b06-aac6-09bfce49151b) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1247.json) |
|[Элемент управления 1248, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F50fc602d-d8e0-444b-a039-ad138ee5deb0) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1248.json) |
|[Элемент управления 1249, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd3bf4251-0818-42db-950b-afd5b25a51c2) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1249.json) |
|[Элемент управления 1250, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8de614d8-a8b7-4f70-a62a-6d37089a002c) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1250.json) |

### <a name="contingency-plan--coordinate-with-related-plans"></a>План на непредвиденные случаи | Координация со связанными планами

**Идентификатор**. NIST SP 800-53 R4 CP-2 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1251, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи \| Координация со связанными планами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5e2b3730-8c14-4081-8893-19dbb5de7348) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1251.json) |

### <a name="contingency-plan--capacity-planning"></a>План на непредвиденные случаи | Планирование загрузки

**Идентификатор**. NIST SP 800-53 R4 CP-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1252, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи \| Планирование ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa328fd72-8ff5-4f96-8c9c-b30ed95db4ab) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1252.json) |

### <a name="contingency-plan--resume-essential-missions--business-functions"></a>План на непредвиденные случаи | Возобновление критических задач и бизнес-функций

**Идентификатор**. NIST SP 800-53 R4 CP-2 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1253, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи \| Возобновление критических задач и бизнес-функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0afce0b3-dd9f-42bb-af28-1e4284ba8311) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1253.json) |

### <a name="contingency-plan--resume-all-missions--business-functions"></a>План на непредвиденные случаи | Возобновление всех задач и бизнес-функций

**Идентификатор**. NIST SP 800-53 R4 CP-2 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1254, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи \| Возобновление всех задач и бизнес-функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F704e136a-4fe0-427c-b829-cd69957f5d2b) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1254.json) |

### <a name="contingency-plan--continue--essential-missions--business-functions"></a>План на непредвиденные случаи | Непрерывность выполнения критических задач и бизнес-функций

**Идентификатор**. NIST SP 800-53 R4 CP-2 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1255, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи \| Непрерывность выполнения критических задач и бизнес-функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff3793f5e-937f-44f7-bfba-40647ef3efa0) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1255.json) |

### <a name="contingency-plan--identify-critical-assets"></a>План на непредвиденные случаи | Определение критических ресурсов

**Идентификатор**. NIST SP 800-53 R4 CP-2 (8) **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1256, контролируемый корпорацией Майкрософт, — план на непредвиденные случаи \| Определение критических ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F232ab24b-810b-4640-9019-74a7d0d6a980) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1256.json) |

### <a name="contingency-training"></a>Обучение реагированию на непредвиденные случаи

**Идентификатор**. NIST SP 800-53 R4 CP-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1257, контролируемый корпорацией Майкрософт, — обучение реагированию на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb958b241-4245-4bd6-bd2d-b8f0779fb543) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1257.json) |
|[Элемент управления 1258, контролируемый корпорацией Майкрософт, — обучение реагированию на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7814506c-382c-4d33-a142-249dd4a0dbff) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1258.json) |
|[Элемент управления 1259, контролируемый корпорацией Майкрософт, — обучение реагированию на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d9e18f7-bad9-4d30-8806-a0c9d5e26208) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1259.json) |

### <a name="contingency-training--simulated-events"></a>Обучение реагированию на непредвиденные случаи | Имитация событий

**Идентификатор**. NIST SP 800-53 R4 CP-3 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1260, контролируемый корпорацией Майкрософт, — обучение реагированию на непредвиденные случаи \| Имитации событий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F42254fc4-2738-4128-9613-72aaa4f0d9c3) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1260.json) |

### <a name="contingency-plan-testing"></a>Тестирование плана на непредвиденные случаи

**Идентификатор**. NIST SP 800-53 R4 CP-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1261, контролируемый корпорацией Майкрософт, — тестирование плана на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F65aeceb5-a59c-4cb1-8d82-9c474be5d431) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1261.json) |
|[Элемент управления 1262, контролируемый корпорацией Майкрософт, — тестирование плана на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F831e510e-db41-4c72-888e-a0621ab62265) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1262.json) |
|[Элемент управления 1263, контролируемый корпорацией Майкрософт, — тестирование плана на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F41472613-3b05-49f6-8fe8-525af113ce17) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1263.json) |

### <a name="contingency-plan-testing--coordinate-with-related-plans"></a>Тестирование плана на непредвиденные случаи | Координация со связанными планами

**Идентификатор**. NIST SP 800-53 R4 CP-4 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1264, контролируемый корпорацией Майкрософт, — тестирование плана на непредвиденные случаи \| Координация со связанными планами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdd280d4b-50a1-42fb-a479-ece5878acf19) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1264.json) |

### <a name="contingency-plan-testing--alternate-processing-site"></a>Тестирование плана на непредвиденные случаи | Альтернативный сайт обработки

**Идентификатор**. NIST SP 800-53 R4 CP-4 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1265, контролируемый корпорацией Майкрософт, — тестирование плана на непредвиденные случаи \| Альтернативный сайт обработки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa18adb5b-1db6-4a5b-901a-7d3797d12972) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1265.json) |
|[Элемент управления 1266, контролируемый корпорацией Майкрософт, — тестирование плана на непредвиденные случаи \| Альтернативный сайт обработки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b4a3eb2-c25d-40bf-ad41-5094b6f59cee) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1266.json) |

### <a name="alternate-storage-site"></a>Альтернативный сайт хранилища

**Идентификатор**. NIST SP 800-53 R4 CP-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1267, контролируемый корпорацией Майкрософт, — альтернативный сайт хранилища](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4e97ba1d-be5d-4953-8da4-0cccf28f4805) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1267.json) |
|[Элемент управления 1268, контролируемый корпорацией Майкрософт, — альтернативный сайт хранилища](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F23f6e984-3053-4dfc-ab48-543b764781f5) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1268.json) |

### <a name="alternate-storage-site--separation-from-primary-site"></a>Альтернативный сайт хранилища | Отделение от основного сайта

**Идентификатор**. NIST SP 800-53 R4 CP-6 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1269, контролируемый корпорацией Майкрософт, — альтернативный сайт хранилища \| Отделение от основного сайта](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F19b9439d-865d-4474-b17d-97d2702fdb66) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1269.json) |

### <a name="alternate-storage-site--recovery-time--point-objectives"></a>Альтернативный сайт хранилища | Целевые время и точки восстановления

**Идентификатор**. NIST SP 800-53 R4 CP-6 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1270, контролируемый корпорацией Майкрософт, — альтернативный сайт хранилища \| Целевые время и точки восстановления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F53c76a39-2097-408a-b237-b279f7b4614d) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1270.json) |

### <a name="alternate-storage-site--accessibility"></a>Альтернативный сайт хранилища | Доступность

**Идентификатор**. NIST SP 800-53 R4 CP-6 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1271, контролируемый корпорацией Майкрософт, — альтернативный сайт хранилища \| Доступность](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fda3bfb53-9c46-4010-b3db-a7ba1296dada) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1271.json) |

### <a name="alternate-processing-site"></a>Альтернативный сайт обработки

**Идентификатор**. NIST SP 800-53 R4 CP-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит виртуальных машин без аварийного восстановления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0015ea4d-51ff-4ce3-8d8c-f3f8f0179a56) |Аудит виртуальных машин, на которых не настроено аварийное восстановление. Дополнительные сведения об аварийном восстановлении см. здесь: [https://aka.ms/asr-doc](https://aka.ms/asr-doc). |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/RecoveryServices_DisasterRecovery_Audit.json) |
|[Элемент управления 1272, контролируемый корпорацией Майкрософт, — альтернативный сайт обработки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fae46cf7a-e3fd-427b-9b91-44bc78e2d9d8) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1272.json) |
|[Элемент управления 1273, контролируемый корпорацией Майкрософт, — альтернативный сайт обработки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe77fcbf2-a1e8-44f1-860e-ed6583761e65) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1273.json) |
|[Элемент управления 1274, контролируемый корпорацией Майкрософт, — альтернативный сайт обработки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2aee175f-cd16-4825-939a-a85349d96210) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1274.json) |

### <a name="alternate-processing-site--separation-from-primary-site"></a>Альтернативный сайт обработки | Отделение от основного сайта

**Идентификатор**. NIST SP 800-53 R4 CP-7 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1275, контролируемый корпорацией Майкрософт, — альтернативный сайт обработки \| Отделение от основного сайта](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa23d9d53-ad2e-45ef-afd5-e6d10900a737) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1275.json) |

### <a name="alternate-processing-site--accessibility"></a>Альтернативный сайт обработки | Доступность

**Идентификатор**. NIST SP 800-53 R4 CP-7 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1276, контролируемый корпорацией Майкрософт, — альтернативный сайт обработки \| Доступность](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe214e563-1206-4a43-a56b-ac5880c9c571) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1276.json) |

### <a name="alternate-processing-site--priority-of-service"></a>Альтернативный сайт обработки | Приоритет службы

**Идентификатор**. NIST SP 800-53 R4 CP-7 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1277, контролируемый корпорацией Майкрософт, — альтернативный сайт обработки \| Приоритет службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdc43e829-3d50-4a0a-aa0f-428d551862aa) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1277.json) |

### <a name="alternate-processing-site--preparation-for-use"></a>Альтернативный сайт обработки | Подготовка к использованию

**Идентификатор**. NIST SP 800-53 R4 CP-7 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1278, контролируемый корпорацией Майкрософт, — альтернативный сайт обработки \| Подготовка к использованию](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8e5ef485-9e16-4c53-a475-fbb8107eac59) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1278.json) |

### <a name="telecommunications-services"></a>Телекоммуникационные службы

**Идентификатор**. NIST SP 800-53 R4 CP-8 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1279, контролируемый корпорацией Майкрософт, — телекоммуникационные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7d00bcd6-963d-4c02-ad8e-b45fa50bf3b0) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1279.json) |

### <a name="telecommunications-services--priority-of-service-provisions"></a>Телекоммуникационные службы | Приоритет подготовки служб

**Идентификатор**. NIST SP 800-53 R4 CP-8 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1280, контролируемый корпорацией Майкрософт, — телекоммуникационные службы \| Приоритет подготовки служб](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffa108498-b3a8-4ffb-9e79-1107e76afad3) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1280.json) |
|[Элемент управления 1281, контролируемый корпорацией Майкрософт, — телекоммуникационные службы \| Приоритет подготовки служб](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8dc459b3-0e77-45af-8d71-cfd8c9654fe2) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1281.json) |

### <a name="telecommunications-services--single-points-of-failure"></a>Телекоммуникационные службы | Единые точки отказа

**Идентификатор**. NIST SP 800-53 R4 CP-8 (2) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1282, контролируемый корпорацией Майкрософт, — телекоммуникационные службы \| Единые точки отказа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34042a97-ec6d-4263-93d2-8c1c46823b2a) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1282.json) |

### <a name="telecommunications-services--separation-of-primary--alternate-providers"></a>Телекоммуникационные службы | Разделение основного и альтернативного поставщиков

**Идентификатор**. NIST SP 800-53 R4 CP-8 (3) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1283, контролируемый корпорацией Майкрософт, — телекоммуникационные службы \| Разделение основного и альтернативного поставщиков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa9172e76-7f56-46e9-93bf-75d69bdb5491) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1283.json) |

### <a name="telecommunications-services--provider-contingency-plan"></a>Телекоммуникационные службы | План поставщика на непредвиденные случаи

**Идентификатор**. NIST SP 800-53 R4 CP-8 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1284, контролируемый корпорацией Майкрософт, — телекоммуникационные службы \| План поставщика на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F942b3e97-6ae3-410e-a794-c9c999b97c0b) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1284.json) |
|[Элемент управления 1285, контролируемый корпорацией Майкрософт, — телекоммуникационные службы \| План поставщика на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F01f7726b-db54-45c2-bcb5-9bd7a43796ee) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1285.json) |
|[Элемент управления 1286, контролируемый корпорацией Майкрософт, — телекоммуникационные службы \| План поставщика на непредвиденные случаи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb4f9b47a-2116-4e6f-88db-4edbf22753f1) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1286.json) |

### <a name="information-system-backup"></a>Резервное копирование информационной системы

**Идентификатор**. NIST SP 800-53 R4 CP-9 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1287, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F819dc6da-289d-476e-8500-7e341ef8677d) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1287.json) |
|[Элемент управления 1288, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8d854c3b-a3e6-4ec9-9f0c-c7274dbaeb2f) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1288.json) |
|[Элемент управления 1289, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7a724864-956a-496c-b778-637cb1d762cf) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1289.json) |
|[Элемент управления 1290, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F92f85ce9-17b7-49ea-85ee-ea7271ea6b82) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1290.json) |

### <a name="information-system-backup--testing-for-reliability--integrity"></a>Резервное копирование информационной системы | Тестирование надежности и целостности

**Идентификатор**. NIST SP 800-53 R4 CP-9 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1291, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы \| Тестирование надежности и целостности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d8fd073-9c85-4ee2-a9d0-2e4ec9eb8912) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1291.json) |

### <a name="information-system-backup--test-restoration-using-sampling"></a>Резервное копирование информационной системы | Тестирование восстановления с помощью выборки

**Идентификатор**. NIST SP 800-53 R4 CP-9 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1292, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы \| Тестирование восстановления с помощью выборки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd03516cf-0293-489f-9b32-a18f2a79f836) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1292.json) |

### <a name="information-system-backup--separate-storage-for-critical-information"></a>Резервное копирование информационной системы | Отдельное хранилище для критической информации

**Идентификатор**. NIST SP 800-53 R4 CP-9 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1293, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы \| Отдельное хранилище для критической информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F87f7cd82-2e45-4d0f-9e2f-586b0962d142) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1293.json) |

### <a name="information-system-backup--transfer-to-alternate-storage-site"></a>Резервное копирование информационной системы | Передача на альтернативный сайт хранилища

**Идентификатор**. NIST SP 800-53 R4 CP-9 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1294, контролируемый корпорацией Майкрософт, — резервное копирование информационной системы \| Передача на альтернативный сайт хранилища](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F49dbe627-2c1e-438c-979e-dd7a39bbf81d) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1294.json) |

### <a name="information-system-recovery-and-reconstitution"></a>Восстановление и воссоздание информационной системы

**Идентификатор**. NIST SP 800-53 R4 CP-10 **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1295, контролируемый корпорацией Майкрософт, — восстановление и воссоздание информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa895fbdb-204d-4302-9689-0a59dc42b3d9) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1295.json) |

### <a name="information-system-recovery-and-reconstitution--transaction-recovery"></a>Восстановление и воссоздание информационной системы | Восстановление транзакций

**Идентификатор**. NIST SP 800-53 R4 CP-10 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1296, контролируемый корпорацией Майкрософт, — восстановление и воссоздание информационной системы \| Восстановление транзакций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe57b98a0-a011-4956-a79d-5d17ed8b8e48) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1296.json) |

### <a name="information-system-recovery-and-reconstitution--restore-within-time-period"></a>Восстановление и воссоздание информационной системы | Восстановление за определенный период времени

**Идентификатор**. NIST SP 800-53 R4 CP-10 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1297, контролируемый корпорацией Майкрософт, — восстановление и воссоздание информационной системы \| Восстановление за определенный период времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F93fd8af1-c161-4bae-9ba9-f62731f76439) |Корпорация Майкрософт реализует этот элемент управления планированием на непредвиденные случаи |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1297.json) |

## <a name="identification-and-authentication"></a>Идентификация и аутентификация

### <a name="identification-and-authentication-policy-and-procedures"></a>Политика и процедуры идентификации и аутентификации

**Идентификатор**. NIST SP 800-53 R4 IA-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1298, контролируемый корпорацией Майкрософт, — политика и процедуры идентификации и аутентификации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1dc784b5-4895-4d27-9d40-a06b032bd1ee) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1298.json) |
|[Элемент управления 1299, контролируемый корпорацией Майкрософт, — политика и процедуры идентификации и аутентификации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffd4e54f7-9ab0-4bae-b6cc-457809948a89) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1299.json) |

### <a name="identification-and-authentication-organizational-users"></a>Идентификация и аутентификация (пользователи организации)

**Идентификатор**. NIST SP 800-53 R4 IA-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1300, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F99deec7d-5526-472e-b07c-3645a792026a) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1300.json) |

### <a name="identification-and-authentication-organizational-users--network-access-to-privileged-accounts"></a>Идентификация и аутентификация (пользователи организации) | Сетевой доступ к привилегированным учетным записям

**Идентификатор**. NIST SP 800-53 R4 IA-2 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на запись в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |Следует включить MFA для всех учетных записей подписки с разрешениями на запись, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |
|[Для учетных записей с разрешениями владельца в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |Следует включить MFA для всех учетных записей подписки с разрешениями владельца, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |
|[Элемент управления 1301, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Доступ по сети к привилегированным учетным записям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6a8e0cc-ac23-468b-abe4-a8a1cc6d7a08) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1301.json) |

### <a name="identification-and-authentication-organizational-users--network-access-to-non-privileged-accounts"></a>Идентификация и аутентификация (пользователи организации) | Сетевой доступ к непривилегированным учетным записям

**Идентификатор**. NIST SP 800-53 R4 IA-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на чтение в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |Следует включить многофакторную проверку подлинности (MFA) для всех учетных записей подписки с разрешениями на чтение, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |
|[Элемент управления 1302, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Доступ по сети к непривилегированным учетным записям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09828c65-e323-422b-9774-9d5c646124da) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1302.json) |

### <a name="identification-and-authentication-organizational-users--local-access-to-privileged-accounts"></a>Идентификация и аутентификация (пользователи организации) | Локальный доступ к привилегированным учетным записям

**Идентификатор**. NIST SP 800-53 R4 IA-2 (3) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1303, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Локальный доступ к привилегированным учетным записям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F80ca0a27-918a-4604-af9e-723a27ee51e8) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1303.json) |

### <a name="identification-and-authentication-organizational-users--local-access-to-non-privileged-accounts"></a>Идентификация и аутентификация (пользователи организации) | Локальный доступ к непривилегированным учетным записям

**Идентификатор**. NIST SP 800-53 R4 IA-2 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1304, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Локальный доступ к непривилегированным учетным записям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6ca71be3-16cb-4d39-8b50-7f8fd5e2f11b) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1304.json) |

### <a name="identification-and-authentication-organizational-users--group-authentication"></a>Идентификация и аутентификация (пользователи организации) | Проверка подлинности группы

**Идентификатор**. NIST SP 800-53 R4 IA-2 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1305, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Групповая аутентификация](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d9166a8-1722-4b8f-847c-2cf3f2618b3d) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1305.json) |

### <a name="identification-and-authentication-organizational-users--network-access-to-privileged-accounts---replay-resistant"></a>Идентификация и аутентификация (пользователи организации) | Сетевой доступ к привилегированным учетным записям — защита от повторных атак

**Идентификатор**. NIST SP 800-53 R4 IA-2 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1306, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Сетевой доступ к привилегированным учетным записям — защита от повторных атак](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcafc6c3c-5fc5-4c5e-a99b-a0ccb1d34eff) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1306.json) |

### <a name="identification-and-authentication-organizational-users--network-access-to-non-privileged-accounts---replay-resistant"></a>Идентификация и проверка подлинности (пользователи организации) | Сетевой доступ к непривилегированным учетным записям — защита от повторных атак

**Идентификатор**. NIST SP 800-53 R4 IA-2 (9) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1307, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Сетевой доступ к непривилегированным учетным записям — защита от повторных атак](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F84e622c8-4bed-417c-84c6-b2fb0dd73682) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1307.json) |

### <a name="identification-and-authentication-organizational-users--remote-access----separate-device"></a>Идентификация и аутентификация (пользователи организации) | Удаленный доступ к отдельным устройствам

**Идентификатор**. NIST SP 800-53 R4 IA-2 (11) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1308, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Удаленный доступ — отдельное устройство](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F81817e1c-5347-48dd-965a-40159d008229) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1308.json) |

### <a name="identification-and-authentication-organizational-users--acceptance-of-piv-credentials"></a>Идентификация и аутентификация (пользователи организации) | Принятие учетных данных для проверки личности

**Идентификатор**. NIST SP 800-53 R4 IA-2 (12) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1309, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи организации) \| Принятие учетных данных для проверки личности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff355d62b-39a8-4ba3-abf7-90f71cb3b000) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1309.json) |

### <a name="device-identification-and-authentication"></a>Идентификация и аутентификация устройства

**Идентификатор**. NIST SP 800-53 R4 IA-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1310, контролируемый корпорацией Майкрософт, — идентификация и аутентификация устройства](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F450d7ede-823d-4931-a99d-57f6a38807dc) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1310.json) |

### <a name="identifier-management"></a>Управление идентификаторами

**Идентификатор**. NIST SP 800-53 R4 IA-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1311, контролируемый корпорацией Майкрософт, — управление идентификаторами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe7568697-0c9e-4ea3-9cec-9e567d14f3c6) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1311.json) |
|[Элемент управления 1312, контролируемый корпорацией Майкрософт, — управление идентификаторами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4d6a5968-9eef-4c18-8534-376790ab7274) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1312.json) |
|[Элемент управления 1313, контролируемый корпорацией Майкрософт, — управление идентификаторами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F36220f5b-79a1-4cdb-8c74-2d2449f9a510) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1313.json) |
|[Элемент управления 1314, контролируемый корпорацией Майкрософт, — управление идентификаторами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef0c8530-efd9-45b8-b753-f03083d06295) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1314.json) |
|[Элемент управления 1315, контролируемый корпорацией Майкрософт, — управление идентификаторами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3aa87116-f1a1-4edb-bfbf-14e036f8d454) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1315.json) |

### <a name="identifier-management--identify-user-status"></a>Управление идентификаторами | Определение состояния пользователей

**Идентификатор**. NIST SP 800-53 R4 IA-4 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1316, контролируемый корпорацией Майкрософт, — управление идентификаторами \| Определение статуса пользователей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8ce14753-66e5-465d-9841-26ef55c09c0d) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1316.json) |

### <a name="authenticator-management"></a>Управление структурой проверки подлинности

**Идентификатор**. NIST SP 800-53 R4 IA-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Добавление управляемого удостоверения, назначаемого системой, для включения назначений гостевой конфигурации на виртуальных машинах без удостоверений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3cf2ab00-13f1-4d0c-8971-2ac904541a7e) |Эта политика добавляет назначаемое системой управляемое удостоверение на виртуальные машины, размещенные в Azure, которые поддерживаются гостевой конфигурацией, но не имеют ни одного управляемого удостоверения. Назначаемое системой управляемое удостоверение является необходимым условием для всех назначений гостевой конфигурации, и его следует добавить на компьютеры перед тем, как использовать любые определения политик гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |modify |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AddSystemIdentityWhenNone_Prerequisite.json) |
|[Добавление управляемого удостоверения, назначаемого системой, для включения назначений гостевой конфигурации на виртуальных машинах с удостоверением, назначаемым пользователем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F497dff13-db2a-4c0f-8603-28fa3b331ab6) |Эта политика добавляет назначаемое системой управляемое удостоверение на виртуальные машины, размещенные в Azure, которые поддерживаются гостевой конфигурацией и имеют по меньшей мере одно назначаемое пользователем удостоверение, но не имеют назначаемого системой управляемого удостоверения. Назначаемое системой управляемое удостоверение является необходимым условием для всех назначений гостевой конфигурации, и его следует добавить на компьютеры перед тем, как использовать любые определения политик гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |modify |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AddSystemIdentityWhenUser_Prerequisite.json) |
|[Аудит компьютеров Linux без заданных разрешений 0644 для файла passwd](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe6955644-301c-44b5-a4c4-528577de6861) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если при установленной ОС Linux на них не заданы разрешения 0644 для файла passwd. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_LinuxPassword121_AINE.json) |
|[Аудит компьютеров Linux с учетными записями без паролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6ec09a3-78bf-4f8f-99dc-6c77182d0f99) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если при установленной ОС Linux на них действуют учетные записи без паролей. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_LinuxPassword232_AINE.json) |
|[Аудит компьютеров Windows, которые не хранят пароли с использованием обратимого шифрования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fda0f98fe-a24b-4ad5-af69-bd0400233661) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если при установленной ОС Windows они не хранят пароли с использованием обратимого шифрования. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordEncryption_AINE.json) |
|[Развертывание расширения гостевой конфигурации Linux для включения назначений гостевой конфигурации на виртуальных машинах Linux](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F331e8ea8-378a-410f-a2e5-ae22f38bb0da) |Эта политика развертывает расширение гостевой конфигурации Linux на виртуальные машины Linux, размещенные в Azure и поддерживаемые гостевой конфигурацией. Расширение гостевой конфигурации Linux является необходимым условием для всех назначений гостевой конфигурации Linux и должно быть развернуто на компьютерах перед применением любых определений политики гостевой конфигурации Linux. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_DeployExtensionLinux_Prerequisite.json) |
|[Развертывание расширения гостевой конфигурации Windows для включения назначений гостевой конфигурации на виртуальных машинах Windows](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F385f5831-96d4-41db-9a3c-cd3af78aaae6) |Эта политика развертывает расширение гостевой конфигурации Windows на виртуальные машины Windows, размещенные в Azure и поддерживаемые гостевой конфигурацией. Расширение гостевой конфигурации Windows является необходимым условием для всех назначений гостевой конфигурации Windows и должно быть развернуто на компьютерах перед применением любых определений политики гостевой конфигурации Windows. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_DeployExtensionWindows_Prerequisite.json) |
|[Элемент управления 1317, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8877f519-c166-47b7-81b7-8a8eb4ff3775) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1317.json) |
|[Элемент управления 1318, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffced5fda-3bdb-4d73-bfea-0e2c80428b66) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1318.json) |
|[Элемент управления 1319, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F66f7ae57-5560-4fc5-85c9-659f204e7a42) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1319.json) |
|[Элемент управления 1320, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6f54c732-71d4-4f93-a696-4e373eca3a77) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1320.json) |
|[Элемент управления 1321, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb627cc6-3a9d-46b5-96b7-5fca49178a37) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1321.json) |
|[Элемент управления 1322, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d1d971e-467e-4278-9633-c74c3d4fecc4) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1322.json) |
|[Элемент управления 1323, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabe8f70b-680f-470c-9b86-a7edfb664ecc) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1323.json) |
|[Элемент управления 1324, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cfea2b3-7f77-497e-ac20-0752f2ff6eee) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1324.json) |
|[Элемент управления 1325, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1845796a-7581-49b2-ae20-443121538e19) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1325.json) |
|[Элемент управления 1326, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8605fc00-1bf5-4fb3-984e-c95cec4f231d) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1326.json) |

### <a name="authenticator-management--password-based-authentication"></a>Управление структурами проверки подлинности | Аутентификация на основе пароля

**Идентификатор**. NIST SP 800-53 R4 IA-5 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Добавление управляемого удостоверения, назначаемого системой, для включения назначений гостевой конфигурации на виртуальных машинах без удостоверений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3cf2ab00-13f1-4d0c-8971-2ac904541a7e) |Эта политика добавляет назначаемое системой управляемое удостоверение на виртуальные машины, размещенные в Azure, которые поддерживаются гостевой конфигурацией, но не имеют ни одного управляемого удостоверения. Назначаемое системой управляемое удостоверение является необходимым условием для всех назначений гостевой конфигурации, и его следует добавить на компьютеры перед тем, как использовать любые определения политик гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |modify |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AddSystemIdentityWhenNone_Prerequisite.json) |
|[Добавление управляемого удостоверения, назначаемого системой, для включения назначений гостевой конфигурации на виртуальных машинах с удостоверением, назначаемым пользователем](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F497dff13-db2a-4c0f-8603-28fa3b331ab6) |Эта политика добавляет назначаемое системой управляемое удостоверение на виртуальные машины, размещенные в Azure, которые поддерживаются гостевой конфигурацией и имеют по меньшей мере одно назначаемое пользователем удостоверение, но не имеют назначаемого системой управляемого удостоверения. Назначаемое системой управляемое удостоверение является необходимым условием для всех назначений гостевой конфигурации, и его следует добавить на компьютеры перед тем, как использовать любые определения политик гостевой конфигурации. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |modify |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AddSystemIdentityWhenUser_Prerequisite.json) |
|[Аудит компьютеров с Windows с возможностью повторного использования прошлых 24 паролей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5b054a0d-39e2-4d53-bea3-9734cad2c69b) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если при установленной ОС Windows они разрешают повторно использовать прошлые 24 паролей. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordEnforce_AINE.json) |
|[Аудит компьютеров Windows, максимальный срок действия пароля на которых не равен 70 дням](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4ceb8dc2-559c-478b-a15b-733fbf1e3738) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры Windows считаются несоответствующими, если на них не настроен максимальный срок действия пароля в 70 дней. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsMaximumPassword_AINE.json) |
|[Аудит компьютеров Windows, минимальный срок действия пароля на которых не равен 1 дню](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F237b38db-ca4d-4259-9e47-7882441ca2c0) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры Windows считаются несоответствующими, если на них не настроен минимальный срок действия пароля в 1 день. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsMinimumPassword_AINE.json) |
|[Аудит компьютеров Windows без включенного параметра сложности пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbf16e0bb-31e1-4646-8202-60a235cc7e74) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры Windows считаются несоответствующими, если на них не установлен параметр сложности пароля. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordComplexity_AINE.json) |
|[Аудит компьютеров Windows, которые не ограничивают минимальную длину пароля 14 символами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa2d0e922-65d0-40c4-8f87-ea6da2d307a2) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры Windows считаются несоответствующими, если на них не настроена минимальная длина пароля в 14 символов. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordLength_AINE.json) |
|[Аудит компьютеров Windows, которые не хранят пароли с использованием обратимого шифрования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fda0f98fe-a24b-4ad5-af69-bd0400233661) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если при установленной ОС Windows они не хранят пароли с использованием обратимого шифрования. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordEncryption_AINE.json) |
|[Развертывание расширения гостевой конфигурации Linux для включения назначений гостевой конфигурации на виртуальных машинах Linux](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F331e8ea8-378a-410f-a2e5-ae22f38bb0da) |Эта политика развертывает расширение гостевой конфигурации Linux на виртуальные машины Linux, размещенные в Azure и поддерживаемые гостевой конфигурацией. Расширение гостевой конфигурации Linux является необходимым условием для всех назначений гостевой конфигурации Linux и должно быть развернуто на компьютерах перед применением любых определений политики гостевой конфигурации Linux. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_DeployExtensionLinux_Prerequisite.json) |
|[Развертывание расширения гостевой конфигурации Windows для включения назначений гостевой конфигурации на виртуальных машинах Windows](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F385f5831-96d4-41db-9a3c-cd3af78aaae6) |Эта политика развертывает расширение гостевой конфигурации Windows на виртуальные машины Windows, размещенные в Azure и поддерживаемые гостевой конфигурацией. Расширение гостевой конфигурации Windows является необходимым условием для всех назначений гостевой конфигурации Windows и должно быть развернуто на компьютерах перед применением любых определений политики гостевой конфигурации Windows. Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol). |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_DeployExtensionWindows_Prerequisite.json) |
|[Элемент управления 1327, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F03188d8f-1ae5-4fe1-974d-2d7d32ef937d) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1327.json) |
|[Элемент управления 1328, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff5c66fdc-3d02-4034-9db5-ba57802609de) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1328.json) |
|[Элемент управления 1329, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F498f6234-3e20-4b6a-a880-cbd646d973bd) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1329.json) |
|[Элемент управления 1330, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff75cedb2-5def-4b31-973e-b69e8c7bd031) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1330.json) |
|[Элемент управления 1331, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F05460fe2-301f-4ed1-8174-d62c8bb92ff4) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1331.json) |
|[Элемент управления 1332, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F068260be-a5e6-4b0a-a430-cd27071c226a) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1332.json) |

### <a name="authenticator-management--pki-based-authentication"></a>Управление структурами проверки подлинности | Аутентификация на основе PKI

**Идентификатор**. NIST SP 800-53 R4 IA-5 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1333, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе PKI](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3298d6bf-4bc6-4278-a95d-f7ef3ac6e594) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1333.json) |
|[Элемент управления 1334, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе PKI](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F44bfdadc-8c2e-4c30-9c99-f005986fabcd) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1334.json) |
|[Элемент управления 1335, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе PKI](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F382016f3-d4ba-4e15-9716-55077ec4dc2a) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1335.json) |
|[Элемент управления 1336, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе PKI](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F77f56280-e367-432a-a3b9-8ca2aa636a26) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1336.json) |

### <a name="authenticator-management--in-person-or-trusted-third-party-registration"></a>Управление структурами проверки подлинности | Личная или доверенная регистрация сторонних разработчиков

**Идентификатор**. NIST SP 800-53 R4 IA-5 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1337, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Личная или доверенная регистрация сторонних разработчиков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F463e5220-3f79-4e24-a63f-343e4096cd22) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1337.json) |

### <a name="authenticator-management--automated-support--for-password-strength-determination"></a>Управление структурами проверки подлинности | Автоматическая поддержка определения надежности пароля

**Идентификатор**. NIST SP 800-53 R4 IA-5 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1338, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Автоматическая поддержка определения надежности пароля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6c59a207-6aed-41dc-83a2-e1ff66e4a4db) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1338.json) |

### <a name="authenticator-management--protection-of-authenticators"></a>Управление структурой проверки подлинности | Защита структур проверки подлинности

**Идентификатор**. NIST SP 800-53 R4 IA-5 (6) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1339, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Защита структур проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F367ae386-db7f-4167-b672-984ff86277c0) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1339.json) |

### <a name="authenticator-management--no-embedded-unencrypted-static-authenticators"></a>Управление структурой проверки подлинности | Отсутствие внедренных незашифрованных статических структур проверки подлинности

**Идентификатор**. NIST SP 800-53 R4 IA-5 (7) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1340, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Отсутствие внедренных незашифрованных статических структур проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe51ff84b-e5ea-408f-b651-2ecc2933e4c6) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1340.json) |

### <a name="authenticator-management--multiple-information-system-accounts"></a>Управление структурой проверки подлинности | Наличие учетных записей в нескольких информационных системах

**Идентификатор**. NIST SP 800-53 R4 IA-5 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1341, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Наличие учетных записей в нескольких информационных системах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34cb7e92-fe4c-4826-b51e-8cd203fa5d35) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1341.json) |

### <a name="authenticator-management--hardware-token-based-authentication"></a>Управление структурами проверки подлинности | Аутентификация на основе аппаратного ключа

**Идентификатор**. NIST SP 800-53 R4 IA-5 (11) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1342, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Аутентификация на основе аппаратного ключа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F283a4e29-69d5-4c94-b99e-29acf003c899) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1342.json) |

### <a name="authenticator-management--expiration-of-cached-authenticators"></a>Управление структурой проверки подлинности | Истечение срока действия кэшированных структур проверки подлинности

**Идентификатор**. NIST SP 800-53 R4 IA-5 (13) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1343, контролируемый корпорацией Майкрософт, — управление структурой проверки подлинности \| Истечение срока действия кэшированных структур проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c251a55-31eb-4e53-99c6-e9c43c393ac2) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1343.json) |

### <a name="authenticator-feedback"></a>Отзыв о структуре проверки подлинности

**Идентификатор**. NIST SP 800-53 R4 IA-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1344, контролируемый корпорацией Майкрософт, — отзыв о структуре проверки подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c895fe7-2d8e-43a2-838c-3a533a5b355e) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1344.json) |

### <a name="cryptographic-module-authentication"></a>Аутентификация криптографического модуля

**Идентификатор**. NIST SP 800-53 R4 IA-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1345, контролируемый корпорацией Майкрософт, — аутентификация криптографического модуля](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff86aa129-7c07-4aa4-bbf5-792d93ffd9ea) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1345.json) |

### <a name="identification-and-authentication-non-organizational-users"></a>Идентификация и аутентификация (пользователи за пределами организации)

**Идентификатор**. NIST SP 800-53 R4 IA-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1346, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи за пределами организации)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F464dc8ce-2200-4720-87a5-dc5952924cc6) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1346.json) |

### <a name="identification-and-authentication-non-organizational-users--acceptance-of-piv-credentials-from-other-agencies"></a>Идентификация и аутентификация(пользователи за пределами организации) | Принятие учетных данных для проверки личности от других учреждений

**Идентификатор**. NIST SP 800-53 R4 IA-8 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1347, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи за пределами организации) \| Принятие учетных данных для проверки личности от других учреждений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F131a2706-61e9-4916-a164-00e052056462) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1347.json) |

### <a name="identification-and-authentication-non-organizational-users--acceptance-of-third-party-credentials"></a>Идентификация и аутентификация (пользователи за пределами организации) | Принятие учетных данных сторонних разработчиков

**Идентификатор**. NIST SP 800-53 R4 IA-8 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1348, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи за пределами организации) \| Принятие сторонних учетных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F855ced56-417b-4d74-9d5f-dd1bc81e22d6) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1348.json) |

### <a name="identification-and-authentication-non-organizational-users--use-of-ficam-approved-products"></a>Идентификация и аутентификация (пользователи за пределами организации) | Использование продуктов, утвержденных FICAM

**Идентификатор**. NIST SP 800-53 R4 IA-8 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1349, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи за пределами организации) \| Использование продуктов, утвержденных FICAM](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17641f70-94cd-4a5d-a613-3d1143e20e34) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1349.json) |

### <a name="identification-and-authentication-non-organizational-users--use-of-ficam-issued-profiles"></a>Идентификация и аутентификация (пользователи за пределами организации) | Использование профилей, выданных FICAM

**Идентификатор**. NIST SP 800-53 R4 IA-8 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1350, контролируемый корпорацией Майкрософт, — идентификация и аутентификация (пользователи за пределами организации) \| Использование профилей, выданных FICAM](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd77fd943-6ba6-4a21-ba07-22b03e347cc4) |Корпорация Майкрософт реализует этот элемент управления идентификацией и аутентификацией |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1350.json) |

## <a name="incident-response"></a>реагирование на инциденты.

### <a name="incident-response-policy-and-procedures"></a>Процедуры и политика реагирования на инциденты

**Идентификатор**. NIST SP 800-53 R4 IR-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1351, контролируемый корпорацией Майкрософт, — политика и процедуры реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbcfb6683-05e5-4ce6-9723-c3fbe9896bdd) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1351.json) |
|[Элемент управления 1352, контролируемый корпорацией Майкрософт, — политика и процедуры реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F518cb545-bfa8-43f8-a108-3b7d5037469a) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1352.json) |

### <a name="incident-response-training"></a>Обучение реагированию на инциденты

**Идентификатор**. NIST SP 800-53 R4 IR-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1353, контролируемый корпорацией Майкрософт, — обучение реагированию на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc785ad59-f78f-44ad-9a7f-d1202318c748) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1353.json) |
|[Элемент управления 1354, контролируемый корпорацией Майкрософт, — обучение реагированию на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9fd92c17-163a-4511-bb96-bbb476449796) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1354.json) |
|[Элемент управления 1355, контролируемый корпорацией Майкрософт, — обучение реагированию на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F90e01f69-3074-4de8-ade7-0fef3e7d83e0) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1355.json) |

### <a name="incident-response-training--simulated-events"></a>Обучение реагированию на инциденты | Имитация событий

**Идентификатор**. NIST SP 800-53 R4 IR-2 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1356, контролируемый корпорацией Майкрософт, — обучение реагированию на инциденты \| Имитации событий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8829f8f5-e8be-441e-85c9-85b72a5d0ef3) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1356.json) |

### <a name="incident-response-training--automated-training-environments"></a>Обучение реагированию на инциденты | Автоматизированная среда для обучения

**Идентификатор**. NIST SP 800-53 R4 IR-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1357, контролируемый корпорацией Майкрософт, — обучение реагированию на инциденты \| Автоматизированная среда для обучения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe4213689-05e8-4241-9d4e-8dd1cdafd105) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1357.json) |

### <a name="incident-response-testing"></a>Тестирование реагирования на инциденты

**Идентификатор**. NIST SP 800-53 R4 IR-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1358, контролируемый корпорацией Майкрософт, — тестирование реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feffbaeef-5bf4-400d-895e-ef8cbc0e64c7) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1358.json) |

### <a name="incident-response-testing--coordination-with-related-plans"></a>Тестирование реагирования на инциденты | Координация со связанными планами

**Идентификатор**. NIST SP 800-53 R4 IR-3 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1359, контролируемый корпорацией Майкрософт, — тестирование реагирования на инциденты \| Координация со связанными планами](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47bc7ea0-7d13-4f7c-a154-b903f7194253) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1359.json) |

### <a name="incident-handling"></a>Обработка инцидента

**Идентификатор**. NIST SP 800-53 R4 IR-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1360, контролируемый корпорацией Майкрософт, — обработка инцидентов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbe5b05e7-0b82-4ebc-9eda-25e447b1a41e) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1360.json) |
|[Элемент управления 1361, контролируемый корпорацией Майкрософт, — обработка инцидентов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F03ed3be1-7276-4452-9a5d-e4168565ac67) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1361.json) |
|[Элемент управления 1362, контролируемый корпорацией Майкрософт, — обработка инцидентов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5d169442-d6ef-439b-8dca-46c2c3248214) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1362.json) |

### <a name="incident-handling--automated-incident-handling-processes"></a>Обработка инцидента | Автоматизированные процессы обработки инцидентов

**Идентификатор**. NIST SP 800-53 R4 IR-4 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1363, контролируемый корпорацией Майкрософт, — обработка инцидента \| Автоматизированные процессы обработки инцидентов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea3e8156-89a1-45b1-8bd6-938abc79fdfd) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1363.json) |

### <a name="incident-handling--dynamic-reconfiguration"></a>Обработка инцидента | Динамическая перенастройка

**Идентификатор**. NIST SP 800-53 R4 IR-4 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1364, контролируемый корпорацией Майкрософт, — обработка инцидентов \| Динамическая перенастройка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4c615c2a-dc83-4dda-8220-abce7b50c9bc) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1364.json) |

### <a name="incident-handling--continuity-of-operations"></a>Обработка инцидента | Непрерывность операций

**Идентификатор**. NIST SP 800-53 R4 IR-4 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1365, контролируемый корпорацией Майкрософт, — обработка инцидента \| Непрерывность операций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4116891d-72f7-46ee-911c-8056cc8dcbd5) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1365.json) |

### <a name="incident-handling--information-correlation"></a>Обработка инцидента | Корреляция информации

**Идентификатор**. NIST SP 800-53 R4 IR-4 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1366, контролируемый корпорацией Майкрософт, — обработка инцидентов \| Корреляция информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F06c45c30-ae44-4f0f-82be-41331da911cc) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1366.json) |

### <a name="incident-handling--insider-threats---specific-capabilities"></a>Обработка инцидента | Внутренние угрозы — определенные возможности

**Идентификатор**. NIST SP 800-53 R4 IR-4 (6) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1367, контролируемый корпорацией Майкрософт, — обработка инцидентов \| Внутренние угрозы — специальные возможности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F435b2547-6374-4f87-b42d-6e8dbe6ae62a) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1367.json) |

### <a name="incident-handling--correlation-with-external-organizations"></a>Обработка инцидента | Корреляция с внешним организациям

**Идентификатор**. NIST SP 800-53 R4 IR-4 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1368, контролируемый корпорацией Майкрософт, — обработка инцидента \| Корреляция с внешним организациям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F465f32da-0ace-4603-8d1b-7be5a3a702de) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1368.json) |

### <a name="incident-monitoring"></a>Мониторинг инцидента

**Идентификатор**. NIST SP 800-53 R4 IR-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1369, контролируемый корпорацией Майкрософт, — отслеживание инцидентов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F18cc35ed-a429-486d-8d59-cb47e87304ed) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1369.json) |

### <a name="incident-monitoring--automated-tracking--data-collection--analysis"></a>Мониторинг инцидента | Автоматизированное отслеживание, сбор данных и анализ

**Идентификатор**. NIST SP 800-53 R4 IR-5 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1370, контролируемый корпорацией Майкрософт, — отслеживание инцидентов \| Автоматическое отслеживание, сбор данных и анализ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F924e1b2d-c502-478f-bfdb-a7e09a0d5c01) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1370.json) |

### <a name="incident-reporting"></a>Отчет об инциденте

**Идентификатор**. NIST SP 800-53 R4 IR-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1371, контролируемый корпорацией Майкрософт, — подготовка отчетов об инцидентах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9447f354-2c85-4700-93b3-ecdc6cb6a417) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1371.json) |
|[Элемент управления 1372, контролируемый корпорацией Майкрософт, — подготовка отчетов об инцидентах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F25b96717-c912-4c00-9143-4e487f411726) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1372.json) |

### <a name="incident-reporting--automated-reporting"></a>Отчет об инциденте | Автоматизированный отчет

**Идентификатор**. NIST SP 800-53 R4 IR-6 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1373, контролируемый корпорацией Майкрософт, — подготовка отчетов об инцидентах \| Автоматическая подготовка отчетов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4cca950f-c3b7-492a-8e8f-ea39663c14f9) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1373.json) |

### <a name="incident-response-assistance"></a>Поддержка реагирования на инциденты

**Идентификатор**. NIST SP 800-53 R4 IR-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1374, контролируемый корпорацией Майкрософт, — поддержка реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcc5c8616-52ef-4e5e-8000-491634ed9249) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1374.json) |

### <a name="incident-response-assistance--automation-support-for-availability-of-information--support"></a>Поддержка реагирования на инциденты | Поддержка автоматизирования для обеспечения доступности данных

**Идентификатор**. NIST SP 800-53 R4 IR-7 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1375, контролируемый корпорацией Майкрософт, — поддержка реагирования на инциденты \| Обеспечение автоматизации для предоставления доступа к информации и службе поддержки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F00379355-8932-4b52-b63a-3bc6daf3451a) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1375.json) |

### <a name="incident-response-assistance--coordination-with-external-providers"></a>Поддержка реагирования на инциденты | Координация с внешними поставщиками

**Идентификатор**. NIST SP 800-53 R4 IR-7 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1376, контролируемый корпорацией Майкрософт, — поддержка реагирования на инциденты \| Координация с внешними поставщиками](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F493a95f3-f2e3-47d0-af02-65e6d6decc2f) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1376.json) |
|[Элемент управления 1377, контролируемый корпорацией Майкрософт, — поддержка реагирования на инциденты \| Координация с внешними поставщиками](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F68434bd1-e14b-4031-9edb-a4adf5f84a67) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1377.json) |

### <a name="incident-response-plan"></a>План реагирования на инциденты

**Идентификатор**. NIST SP 800-53 R4 IR-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1378, контролируемый корпорацией Майкрософт, — план реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F97fceb70-6983-42d0-9331-18ad8253184d) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1378.json) |
|[Элемент управления 1379, контролируемый корпорацией Майкрософт, — план реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9442dd2c-a07f-46cd-b55a-553b66ba47ca) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1379.json) |
|[Элемент управления 1380, контролируемый корпорацией Майкрософт, — план реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb4319b7e-ea8d-42ff-8a67-ccd462972827) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1380.json) |
|[Элемент управления 1381, контролируемый корпорацией Майкрософт, — план реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe5368258-9684-4567-8126-269f34e65eab) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1381.json) |
|[Элемент управления 1382, контролируемый корпорацией Майкрософт, — план реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F841392b3-40da-4473-b328-4cde49db67b3) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1382.json) |
|[Элемент управления 1383, контролируемый корпорацией Майкрософт, — план реагирования на инциденты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd4558451-e16a-4d2d-a066-fe12a6282bb9) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1383.json) |

### <a name="information-spillage-response"></a>Утечка данных

**Идентификатор**. NIST SP 800-53 R4 IR-9 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1384, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F79fbc228-461c-4a45-9004-a865ca0728a7) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1384.json) |
|[Элемент управления 1385, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3e495e65-8663-49ca-9b38-9f45e800bc58) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1385.json) |
|[Элемент управления 1386, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5120193e-91fd-4f9d-bc6d-194f94734065) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1386.json) |
|[Элемент управления 1387, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3007185-3857-43a9-8237-06ca94f1084c) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1387.json) |
|[Элемент управления 1388, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c7c575a-d4c5-4f6f-bd49-dee97a8cba55) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1388.json) |
|[Элемент управления 1389, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc39e6fda-ae70-4891-a739-be7bba6d1062) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1389.json) |

### <a name="information-spillage-response--responsible-personnel"></a>Утечка данных | Ответственный персонал

**Идентификатор**. NIST SP 800-53 R4 IR-9 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1390, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации \| Ответственный персонал](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc3b65b63-09ec-4cb5-8028-7dd324d10eb0) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1390.json) |

### <a name="information-spillage-response--training"></a>Утечка данных | Обучение

**Идентификатор**. NIST SP 800-53 R4 IR-9 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1391, контролируемый корпорацией Майкрософт, — реагирование на разглашение данных \| Обучение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdd6ac1a1-660e-4810-baa8-74e868e2ed47) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1391.json) |

### <a name="information-spillage-response--post-spill-operations"></a>Утечка данных | Операции после утечки

**Идентификатор**. NIST SP 800-53 R4 IR-9 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1392, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации \| Операции после разглашения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86dc819f-15e1-43f9-a271-41ae58d4cecc) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1392.json) |

### <a name="information-spillage-response--exposure-to-unauthorized-personnel"></a>Утечка данных | Воздействие на неавторизованный персонал

**Идентификатор**. NIST SP 800-53 R4 IR-9 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1393, контролируемый корпорацией Майкрософт, — реагирование на разглашение информации \| Воздействие на неавторизованный персонал](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F731856d8-1598-4b75-92de-7d46235747c0) |Корпорация Майкрософт реализует этот элемент управления реагированием на инциденты |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1393.json) |

## <a name="maintenance"></a>Обслуживание

### <a name="system-maintenance-policy-and-procedures"></a>Процедуры и политика обслуживания системы

**Идентификатор**. NIST SP 800-53 R4 MA-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1394, контролируемый корпорацией Майкрософт, — политика и процедуры обслуживания системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4db56f68-3f50-45ab-88f3-ca46f5379a94) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1394.json) |
|[Элемент управления 1395, контролируемый корпорацией Майкрософт, — политика и процедуры обслуживания системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7207a023-a517-41c5-9df2-09d4c6845a05) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1395.json) |

### <a name="controlled-maintenance"></a>Управляемое обслуживание

**Идентификатор**. NIST SP 800-53 R4 MA-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1396, контролируемый корпорацией Майкрософт, — управляемое обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F276af98f-4ff9-4e69-99fb-c9b2452fb85f) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1396.json) |
|[Элемент управления 1397, контролируемый корпорацией Майкрософт, — управляемое обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F391af4ab-1117-46b9-b2c7-78bbd5cd995b) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1397.json) |
|[Элемент управления 1398, контролируемый корпорацией Майкрософт, — управляемое обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F443e8f3d-b51a-45d8-95a7-18b0e42f4dc4) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1398.json) |
|[Элемент управления 1399, контролируемый корпорацией Майкрософт, — управляемое обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2256e638-eb23-480f-9e15-6cf1af0a76b3) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1399.json) |
|[Элемент управления 1400, контролируемый корпорацией Майкрософт, — управляемое обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa96d5098-a604-4cdf-90b1-ef6449a27424) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1400.json) |
|[Элемент управления 1401, контролируемый корпорацией Майкрософт, — управляемое обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb78ee928-e3c1-4569-ad97-9f8c4b629847) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1401.json) |

### <a name="controlled-maintenance--automated-maintenance-activities"></a>Управляемое обслуживание | Автоматические действия по обслуживанию

**Идентификатор**. NIST SP 800-53 R4 MA-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1402, контролируемый корпорацией Майкрософт, — управляемое обслуживание \| Автоматические действия по обслуживанию](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0a560d32-8075-4fec-9615-9f7c853f4ea9) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1402.json) |
|[Элемент управления 1403, контролируемый корпорацией Майкрософт, — управляемое обслуживание \| Автоматические действия по обслуживанию](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F57149289-d52b-4f40-9fe6-5233c1ef80f7) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1403.json) |

### <a name="maintenance-tools"></a>Средства обслуживания

**Идентификатор**. NIST SP 800-53 R4 MA-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1404, контролируемый корпорацией Майкрософт, — средства обслуживания](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F13d8f903-0cd6-449f-a172-50f6579c182b) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1404.json) |

### <a name="maintenance-tools--inspect-tools"></a>Средства обслуживания | Проверка средств

**Идентификатор**. NIST SP 800-53 R4 MA-3 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1405, контролируемый корпорацией Майкрософт, — средства обслуживания \| Проверка средств](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffe1a0bf3-409a-4b00-b60d-0b1f917f7e7b) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1405.json) |

### <a name="maintenance-tools--inspect-media"></a>Средства обслуживания | Проверка носителей

**Идентификатор**. NIST SP 800-53 R4 MA-3 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1406, контролируемый корпорацией Майкрософт, — средства обслуживания \| Проверка носителей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa0f5339c-9292-43aa-a0bc-d27c6b8e30aa) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1406.json) |

### <a name="maintenance-tools--prevent-unauthorized-removal"></a>Средства обслуживания | Предотвращение несанкционированного изъятия

**Идентификатор**. NIST SP 800-53 R4 MA-3 (3) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1407, контролируемый корпорацией Майкрософт, — средства обслуживания \| Предотвращение несанкционированного изъятия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fff9fbd83-1d8d-4b41-aac2-94cb44b33976) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1407.json) |
|[Элемент управления 1408, контролируемый корпорацией Майкрософт, — средства обслуживания \| Предотвращение несанкционированного изъятия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc5f56ac6-4bb2-4086-bc41-ad76344ba2c2) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1408.json) |
|[Элемент управления 1409, контролируемый корпорацией Майкрософт, — средства обслуживания \| Предотвращение несанкционированного изъятия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd1880188-e51a-4772-b2ab-68f5e8bd27f6) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1409.json) |
|[Элемент управления 1410, контролируемый корпорацией Майкрософт, — средства обслуживания \| Предотвращение несанкционированного изъятия](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa2596a9f-e59f-420d-9625-6e0b536348be) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1410.json) |

### <a name="nonlocal-maintenance"></a>Нелокальное обслуживание

**Идентификатор**. NIST SP 800-53 R4 MA-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1411, контролируемый корпорацией Майкрософт, — нелокальное обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F898d4fe8-f743-4333-86b7-0c9245d93e7d) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1411.json) |
|[Элемент управления 1412, контролируемый корпорацией Майкрософт, — нелокальное обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3492d949-0dbb-4589-88b3-7b59601cc764) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1412.json) |
|[Элемент управления 1413, контролируемый корпорацией Майкрософт, — нелокальное обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faeedddb6-6bc0-42d5-809b-80048033419d) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1413.json) |
|[Элемент управления 1414, контролируемый корпорацией Майкрософт, — нелокальное обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2ce63a52-e47b-4ae2-adbb-6e40d967f9e6) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1414.json) |
|[Элемент управления 1415, контролируемый корпорацией Майкрософт, — нелокальное обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F61a1dd98-b259-4840-abd5-fbba7ee0da83) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1415.json) |

### <a name="nonlocal-maintenance--document-nonlocal-maintenance"></a>Нелокальное обслуживание | Составление документации по нелокальному обслуживанию

**Идентификатор**. NIST SP 800-53 R4 MA-4 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1416, контролируемый корпорацией Майкрософт, — нелокальное обслуживание \| Составление документации по нелокальному обслуживанию](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F38dfd8a3-5290-4099-88b7-4081f4c4d8ae) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1416.json) |

### <a name="nonlocal-maintenance--comparable-security--sanitization"></a>Нелокальное обслуживание | Безопасность и очистка

**Идентификатор**. NIST SP 800-53 R4 MA-4 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1417, контролируемый корпорацией Майкрософт, — нелокальное обслуживание \| Сопоставимый уровень безопасности и очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7522ed84-70d5-4181-afc0-21e50b1b6d0e) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1417.json) |
|[Элемент управления 1418, контролируемый корпорацией Майкрософт, — нелокальное обслуживание \| Сопоставимый уровень безопасности и очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F28e633fd-284e-4ea7-88b4-02ca157ed713) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1418.json) |

### <a name="nonlocal-maintenance--cryptographic-protection"></a>Нелокальное обслуживание | Криптографическая защита

**Идентификатор**. NIST SP 800-53 R4 MA-4 (6) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1419, контролируемый корпорацией Майкрософт, — нелокальное обслуживание \| Криптографическая защита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6747bf9-2b97-45b8-b162-3c8becb9937d) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1419.json) |

### <a name="maintenance-personnel"></a>Персонал по обслуживанию

**Идентификатор**. NIST SP 800-53 R4 MA-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1420, контролируемый корпорацией Майкрософт, — персонал по обслуживанию](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F05ae08cc-a282-413b-90c7-21a2c60b8404) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1420.json) |
|[Элемент управления 1421, контролируемый корпорацией Майкрософт, — персонал по обслуживанию](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe539caaa-da8c-41b8-9e1e-449851e2f7a6) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1421.json) |
|[Элемент управления 1422, контролируемый корпорацией Майкрософт, — персонал по обслуживанию](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea556850-838d-4a37-8ce5-9d7642f95e11) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1422.json) |

### <a name="maintenance-personnel--individuals-without-appropriate-access"></a>Персонал по обслуживанию | Пользователи без соответствующего уровня доступа

**Идентификатор**. NIST SP 800-53 R4 MA-5 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1423, контролируемый корпорацией Майкрософт, — персонал по обслуживанию \| Пользователи без соответствующего уровня доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7741669e-d4f6-485a-83cb-e70ce7cbbc20) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1423.json) |
|[Элемент управления 1424, контролируемый корпорацией Майкрософт, — персонал по обслуживанию \| Пользователи без соответствующего уровня доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf55fc87-48e1-4676-a2f8-d9a8cf993283) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1424.json) |

### <a name="timely-maintenance"></a>Своевременное обслуживание

**Идентификатор**. NIST SP 800-53 R4 MA-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1425, контролируемый корпорацией Майкрософт, — своевременное обслуживание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5983d99c-f39b-4c32-a3dc-170f19f6941b) |Корпорация Майкрософт реализует этот элемент управления обслуживанием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1425.json) |

## <a name="media-protection"></a>Защита данных на носителях

### <a name="media-protection-policy-and-procedures"></a>Политика и процедуры защиты данных на носителях

**Идентификатор**. NIST SP 800-53 R4 MP-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1426, контролируемый корпорацией Майкрософт, — политика и процедуры защиты данных на носителях](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F21f639bc-f42b-46b1-8f40-7a2a389c291a) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1426.json) |
|[Элемент управления 1427, контролируемый корпорацией Майкрософт, — политика и процедуры защиты данных на носителях](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbc90e44f-d83f-4bdf-900f-3d5eb4111b31) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1427.json) |

### <a name="media-access"></a>Доступ к носителям

**Идентификатор**. NIST SP 800-53 R4 MP-2 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1428, контролируемый корпорацией Майкрософт, — доступ к носителям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0a77fcc7-b8d8-451a-ab52-56197913c0c7) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1428.json) |

### <a name="media-marking"></a>Маркировка носителей

**Идентификатор**. NIST SP 800-53 R4 MP-3 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1429, контролируемый корпорацией Майкрософт, — маркировка носителей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb07c9b24-729e-4e85-95fc-f224d2d08a80) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1429.json) |
|[Элемент управления 1430, контролируемый корпорацией Майкрософт, — маркировка носителей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0f559588-5e53-4b14-a7c4-85d28ebc2234) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1430.json) |

### <a name="media-storage"></a>Устройство хранения данных

**Идентификатор**. NIST SP 800-53 R4 MP-4 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1431, контролируемый корпорацией Майкрософт, — устройство хранения данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7173c52-2b99-4696-a576-63dd5f970ef4) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1431.json) |
|[Элемент управления 1432, контролируемый корпорацией Майкрософт, — устройство хранения данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1140e542-b80d-4048-af45-3f7245be274b) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1432.json) |

### <a name="media-transport"></a>Передача носителя

**Идентификатор**. NIST SP 800-53 R4 MP-5 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1433, контролируемый корпорацией Майкрософт, — передача носителя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5b879b41-2728-41c5-ad24-9ee2c37cbe65) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1433.json) |
|[Элемент управления 1434, контролируемый корпорацией Майкрософт, — передача носителя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c18f06b-a68d-41c3-8863-b8cd3acb5f8f) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1434.json) |
|[Элемент управления 1435, контролируемый корпорацией Майкрософт, — передача носителя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffa8d221b-d130-4637-ba16-501e666628bb) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1435.json) |
|[Элемент управления 1436, контролируемый корпорацией Майкрософт, — передача носителя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F28aab8b4-74fd-4b7c-9080-5a7be525d574) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1436.json) |

### <a name="media-transport--cryptographic-protection"></a>Передача носителя | Криптографическая защита

**Идентификатор**. NIST SP 800-53 R4 MP-5 (4) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1437, контролируемый корпорацией Майкрософт, — передача носителя \| Криптографическая защита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d1eb6ed-bf13-4046-b993-b9e2aef0f76c) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1437.json) |

### <a name="media-sanitization"></a>Очистка носителя

**Идентификатор**. NIST SP 800-53 R4 MP-6 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1438, контролируемый корпорацией Майкрософт, — очистка носителя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F40fcc635-52a2-4dbc-9523-80a1f4aa1de6) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1438.json) |
|[Элемент управления 1439, контролируемый корпорацией Майкрософт, — очистка носителя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdce72873-c5f1-47c3-9b4f-6b8207fd5a45) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1439.json) |

### <a name="media-sanitization--review--approve--track--document--verify"></a>Очистка носителя | Просмотр, утверждение, отслеживание, документация и проверка

**Идентификатор**. NIST SP 800-53 R4 MP-6 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1440, контролируемый корпорацией Майкрософт, — очистка носителя \| Просмотр, утверждение, отслеживание, документирование и проверка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F881299bf-2a5b-4686-a1b2-321d33679953) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1440.json) |

### <a name="media-sanitization--equipment-testing"></a>Очистка носителя | Проверка оборудования

**Идентификатор**. NIST SP 800-53 R4 MP-6 (2) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1441, контролируемый корпорацией Майкрософт, — очистка носителя \| Тестирование оборудования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6519d7f3-e8a2-4ff3-a935-9a9497152ad7) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1441.json) |

### <a name="media-sanitization--nondestructive-techniques"></a>Очистка носителя | Методы обратимой очистки

**Идентификатор**. NIST SP 800-53 R4 MP-6 (3) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1442, контролируемый корпорацией Майкрософт, — очистка носителя \| Методы обратимой очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f26049b-2c5a-4841-9ff3-d48a26aae475) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1442.json) |

### <a name="media-use"></a>Использование носителя

**Идентификатор**. NIST SP 800-53 R4 MP-7 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1443, контролируемый корпорацией Майкрософт, — использование носителя](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcd0ec6fa-a2e7-4361-aee4-a8688659a9ed) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1443.json) |

### <a name="media-use--prohibit-use-without-owner"></a>Использование носителя | Запрет использования носителей, не имеющих владельца

**Идентификатор**. NIST SP 800-53 R4 MP-7 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1444, контролируемый корпорацией Майкрософт, — использование носителя \| Запрет использования носителей, не имеющих владельца](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F666143df-f5e0-45bd-b554-135f0f93e44e) |Корпорация Майкрософт реализует этот элемент управления защитой данных на носителях |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1444.json) |

## <a name="physical-and-environmental-protection"></a>Физическая безопасность и защита от неблагоприятного воздействия окружающей среды

### <a name="physical-and-environmental-protection-policy-and-procedures"></a>Политика и процедуры обеспечения физической безопасности и защиты от неблагоприятного воздействия окружающей среды

**Идентификатор**. NIST SP 800-53 R4 PE-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1445, контролируемый корпорацией Майкрософт, — политика и процедуры обеспечения физической безопасности и защиты от неблагоприятного воздействия окружающей среды](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F32d07d59-2716-4972-b37b-214a67ac4a37) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1445.json) |
|[Элемент управления 1446, контролируемый корпорацией Майкрософт, — политика и процедуры обеспечения физической безопасности и защиты от неблагоприятного воздействия окружающей среды](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbf6850fe-abba-468e-9ef4-d09ec7d983cd) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1446.json) |

### <a name="physical-access-authorizations"></a>Разрешение на физический доступ

**Идентификатор**. NIST SP 800-53 R4 PE-2 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1447, контролируемый корпорацией Майкрософт, — разрешения на физический доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb9783a99-98fe-4a95-873f-29613309fe9a) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1447.json) |
|[Элемент управления 1448, контролируемый корпорацией Майкрософт, — разрешения на физический доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F825d6494-e583-42f2-a3f2-6458e6f0004f) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1448.json) |
|[Элемент управления 1449, контролируемый корпорацией Майкрософт, — разрешения на физический доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff784d3b0-5f2b-49b7-b9f3-00ba8653ced5) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1449.json) |
|[Элемент управления 1450, контролируемый корпорацией Майкрософт, — разрешения на физический доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F134d7a13-ba3e-41e2-b236-91bfcfa24e01) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1450.json) |

### <a name="physical-access-control"></a>Контроль физического доступа

**Идентификатор**. NIST SP 800-53 R4 PE-3 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1451, контролируемый корпорацией Майкрософт, — контроль физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3f1e5a3-25c1-4476-8cb6-3955031f8e65) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1451.json) |
|[Элемент управления 1452, контролируемый корпорацией Майкрософт, — контроль физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82c76455-4d3f-4e09-a654-22e592107e74) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1452.json) |
|[Элемент управления 1453, контролируемый корпорацией Майкрософт, — контроль физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9693b564-3008-42bc-9d5d-9c7fe198c011) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1453.json) |
|[Элемент управления 1454, контролируемый корпорацией Майкрософт, — контроль физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fad58985d-ab32-4f99-8bd3-b7e134c90229) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1454.json) |
|[Элемент управления 1455, контролируемый корпорацией Майкрософт, — контроль физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F068a88d4-e520-434e-baf0-9005a8164e6a) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1455.json) |
|[Элемент управления 1456, контролируемый корпорацией Майкрософт, — контроль физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F733ba9e3-9e7c-440a-a7aa-6196a90a2870) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1456.json) |
|[Элемент управления 1457, контролируемый корпорацией Майкрософт, — контроль физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff2d9d3e6-8886-4305-865d-639163e5c305) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1457.json) |

### <a name="physical-access-control--information-system-access"></a>Контроль физического доступа | Доступ к информационной системе

**Идентификатор**. NIST SP 800-53 R4 PE-3 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1458, контролируемый корпорацией Майкрософт, — контроль физического доступа \| Доступ к информационной системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8c19ceb7-56e9-4488-8ddb-b1eb3aa6d203) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1458.json) |

### <a name="access-control-for-transmission-medium"></a>Управление доступом для среды передачи

**Идентификатор**. NIST SP 800-53 R4 PE-4 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1459, контролируемый корпорацией Майкрософт, — управление доступом для среды передачи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F75cc73c7-5cdb-479d-a06f-7b4d0dbb1da0) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1459.json) |

### <a name="access-control-for-output-devices"></a>Управление доступом устройств вывода

**Идентификатор**. NIST SP 800-53 R4 PE-5 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1460, контролируемый корпорацией Майкрософт, — управление доступом для устройств вывода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6f3ce1bb-4f77-4695-8355-70b08d54fdda) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1460.json) |

### <a name="monitoring-physical-access"></a>Мониторинг физического доступа

**Идентификатор**. NIST SP 800-53 R4 PE-6 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1461, контролируемый корпорацией Майкрософт, — мониторинг физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faafef03e-fea8-470b-88fa-54bd1fcd7064) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1461.json) |
|[Элемент управления 1462, контролируемый корпорацией Майкрософт, — мониторинг физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9b1f3a9a-13a1-4b40-8420-36bca6fd8c02) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1462.json) |
|[Элемент управления 1463, контролируемый корпорацией Майкрософт, — мониторинг физического доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F59721f87-ae25-4db0-a2a4-77cc5b25d495) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1463.json) |

### <a name="monitoring-physical-access--intrusion-alarms--surveillance-equipment"></a>Мониторинг физического доступа | Оповещение о вторжении и оборудование для наблюдения

**Идентификатор**. NIST SP 800-53 R4 PE-6 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1464, контролируемый корпорацией Майкрософт, — мониторинг физического доступа \| Оповещение о вторжении и оборудование для наблюдения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F41256567-1795-4684-b00b-a1308ce43cac) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1464.json) |

### <a name="monitoring-physical-access--monitoring-physical-access-to-information-systems"></a>Мониторинг физического доступа | Мониторинг физического доступа к информационным системам

**Идентификатор**. NIST SP 800-53 R4 PE-6 (4) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1465, контролируемый корпорацией Майкрософт, — мониторинг физического доступа \| Мониторинг физического доступа к информационным системам](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe6e41554-86b5-4537-9f7f-4fc41a1d1640) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1465.json) |

### <a name="visitor-access-records"></a>Записи доступа посетителей

**Идентификатор**. NIST SP 800-53 R4 PE-8 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1466, контролируемый корпорацией Майкрософт, — записи о доступе посетителей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d943a9c-a6f1-401f-a792-740cdb09c451) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1466.json) |
|[Элемент управления 1467, контролируемый корпорацией Майкрософт, — записи о доступе посетителей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5350cbf9-8bdd-4904-b22a-e88be84ca49d) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1467.json) |

### <a name="visitor-access-records--automated-records-maintenance--review"></a>Записи доступа посетителей | Автоматическое обслуживание или проверка записей

**Идентификатор**. NIST SP 800-53 R4 PE-8 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1468, контролируемый корпорацией Майкрософт, — записи о доступе посетителей \| Автоматическое обслуживание или проверка записей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F75603f96-80a1-4757-991d-5a1221765ddd) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1468.json) |

### <a name="power-equipment-and-cabling"></a>Источники питания и кабели

**Идентификатор**. NIST SP 800-53 R4 PE-9 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1469, контролируемый корпорацией Майкрософт, — источники питания и кабели](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff509c5b6-0de0-4a4e-9b2e-cd9cbf3a58fd) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1469.json) |

### <a name="emergency-shutoff"></a>Аварийное выключение

**Идентификатор**. NIST SP 800-53 R4 PE-10 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1470, контролируемый корпорацией Майкрософт, — аварийное выключение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc89ba09f-2e0f-44d0-8095-65b05bd151ef) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1470.json) |
|[Элемент управления 1471, контролируемый корпорацией Майкрософт, — аварийное выключение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7dd0e9ce-1772-41fb-a50a-99977071f916) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1471.json) |
|[Элемент управления 1472, контролируемый корпорацией Майкрософт, — аварийное выключение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef869332-921d-4c28-9402-3be73e6e50c8) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1472.json) |

### <a name="emergency-power"></a>Аварийное питание

**Идентификатор**. NIST SP 800-53 R4 PE-11 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1473, контролируемый корпорацией Майкрософт, — аварийное питание](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd7047705-d719-46a7-8bb0-76ad233eba71) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1473.json) |

### <a name="emergency-power--long-term-alternate-power-supply---minimal-operational-capability"></a>Аварийное питание | Долгосрочный альтернативный источник питания. Минимальная операционная способность

**Идентификатор**. NIST SP 800-53 R4 PE-11 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1474, контролируемый корпорацией Майкрософт, — аварийное питание \| Долгосрочный альтернативный источник питания. Минимальная операционная способность](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F03ad326e-d7a1-44b1-9a76-e17492efc9e4) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1474.json) |

### <a name="emergency-lighting"></a>Аварийное освещение

**Идентификатор**. NIST SP 800-53 R4 PE-12 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1475, контролируемый корпорацией Майкрософт, — аварийное освещение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34a63848-30cf-4081-937e-ce1a1c885501) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1475.json) |

### <a name="fire-protection"></a>Пожарная защита

**Идентификатор**. NIST SP 800-53 R4 PE-13 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1476, контролируемый корпорацией Майкрософт, — противопожарная защита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0f3c4ac2-3e35-4906-a80b-473b12a622d7) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1476.json) |

### <a name="fire-protection--detection-devices--systems"></a>Пожарная защита | Устройства и системы обнаружения

**Идентификатор**. NIST SP 800-53 R4 PE-13 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1477, контролируемый корпорацией Майкрософт, — противопожарная защита \| Устройства и системы обнаружения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4862a63c-6c74-4a9d-a221-89af3c374503) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1477.json) |

### <a name="fire-protection--suppression-devices--systems"></a>Пожарная защита | Системы и устройства пожаротушения

**Идентификатор**. NIST SP 800-53 R4 PE-13 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1478, контролируемый корпорацией Майкрософт, — противопожарная защита \| Устройства и системы пожаротушения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff997df46-cfbb-4cc8-aac8-3fecdaf6a183) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1478.json) |

### <a name="fire-protection--automatic-fire-suppression"></a>Пожарная защита | Автоматическое пожаротушение

**Идентификатор**. NIST SP 800-53 R4 PE-13 (3) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1479, контролируемый корпорацией Майкрософт, — противопожарная защита \| Автоматическое пожаротушение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe327b072-281d-4f75-9c28-4216e5d72f26) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1479.json) |

### <a name="temperature-and-humidity-controls"></a>Контроль температуры и влажности

**Идентификатор**. NIST SP 800-53 R4 PE-14 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1480, контролируемый корпорацией Майкрософт, — контроль температуры и влажности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F18a767cc-1947-4338-a240-bc058c81164f) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1480.json) |
|[Элемент управления 1481, контролируемый корпорацией Майкрософт, — контроль температуры и влажности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F717a1c78-a267-4f56-ac58-ee6c54dc4339) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1481.json) |

### <a name="temperature-and-humidity-controls--monitoring-with-alarms--notifications"></a>Контроль температуры и влажности | Мониторинг с помощью сигналов и оповещений

**Идентификатор**. NIST SP 800-53 R4 PE-14 (2) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1482, контролируемый корпорацией Майкрософт, — контроль температуры и влажности \| Мониторинг с помощью сигналов и уведомлений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9df4277e-8c88-4d5c-9b1a-541d53d15d7b) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1482.json) |

### <a name="water-damage-protection"></a>Защита от ущерба, причиненного водой

**Идентификатор**. NIST SP 800-53 R4 PE-15 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1483, контролируемый корпорацией Майкрософт, — защита от ущерба, причиняемого водой](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5cb81060-3c8a-4968-bcdc-395a1801f6c1) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1483.json) |

### <a name="water-damage-protection--automation-support"></a>Защита от ущерба, причиненного водой | Автоматические механизмы обнаружения

**Идентификатор**. NIST SP 800-53 R4 PE-15 (1) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1484, контролируемый корпорацией Майкрософт, — защита от ущерба, причиняемого водой \| Поддержка службы автоматизации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F486b006a-3653-45e8-b41c-a052d3e05456) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1484.json) |

### <a name="delivery-and-removal"></a>Доставка и вывозка

**Идентификатор**. NIST SP 800-53 R4 PE-16 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1485, контролируемый корпорацией Майкрософт, — доставка и удаление](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F50301354-95d0-4a11-8af5-8039ecf6d38b) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1485.json) |

### <a name="alternate-work-site"></a>Альтернативный веб-сайт

**Идентификатор**. NIST SP 800-53 R4 PE-17 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1486, контролируемый корпорацией Майкрософт, — альтернативный рабочий сайт](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcb790345-a51f-43de-934e-98dbfaf9dca5) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1486.json) |
|[Элемент управления 1487, контролируемый корпорацией Майкрософт, — альтернативный рабочий сайт](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe9c3371d-c30c-4f58-abd9-30b8a8199571) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1487.json) |
|[Элемент управления 1488, контролируемый корпорацией Майкрософт, — альтернативный рабочий сайт](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd8ef30eb-a44f-47af-8524-ac19a36d41d2) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1488.json) |

### <a name="location-of-information-system-components"></a>Расположение компонентов информационной системы

**Идентификатор**. NIST SP 800-53 R4 PE-18 **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1489, контролируемый корпорацией Майкрософт, — расположение компонентов информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d0a794f-1444-4c96-9534-e35fc8c39c91) |Корпорация Майкрософт реализует этот элемент управления физической безопасностью и защитой от неблагоприятного воздействия окружающей среды |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1489.json) |

## <a name="planning"></a>Планирование

### <a name="security-planning-policy-and-procedures"></a>Политика и процедуры планирования безопасности

**Идентификатор**. NIST SP 800-53 R4 PL-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1490, контролируемый корпорацией Майкрософт, — политика и процедуры планирования действий безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9e61da80-0957-4892-b70c-609d5eaafb6b) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1490.json) |
|[Элемент управления 1491, контролируемый корпорацией Майкрософт, — политика и процедуры планирования действий безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1571dd40-dafc-4ef4-8f55-16eba27efc7b) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1491.json) |

### <a name="system-security-plan"></a>План безопасности системы

**Идентификатор**. NIST SP 800-53 R4 PL-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1492, контролируемый корпорацией Майкрософт, — план безопасности системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7ad5f307-e045-46f7-8214-5bdb7e973737) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1492.json) |
|[Элемент управления 1493, контролируемый корпорацией Майкрософт, — план безопасности системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22b469b3-fccf-42da-aa3b-a28e6fb113ce) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1493.json) |
|[Элемент управления 1494, контролируемый корпорацией Майкрософт, — план безопасности системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9ed09d84-3311-4853-8b67-2b55dfa33d09) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1494.json) |
|[Элемент управления 1495, контролируемый корпорацией Майкрософт, — план безопасности системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff4978d0e-a596-48e7-9f8c-bbf52554ce8d) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1495.json) |
|[Элемент управления 1496, контролируемый корпорацией Майкрософт, — план безопасности системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ca96127-2f87-46ab-a4fc-0d2a786df1c8) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1496.json) |

### <a name="system-security-plan--plan--coordinate-with-other-organizational-entities"></a>План безопасности системы | Планирование и координация с другими организационными подразделениями

**Идентификатор**. NIST SP 800-53 R4 PL-2 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1497, контролируемый корпорацией Майкрософт, — план безопасности системы \| Планирование и координация с другими организационными подразделениями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2e3c5583-1729-4d36-8771-59c32f090a22) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1497.json) |

### <a name="rules-of-behavior"></a>Правила поведения

**Идентификатор**. NIST SP 800-53 R4 PL-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1498, контролируемый корпорацией Майкрософт, — правила поведения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F633988b9-cf2f-4323-8394-f0d2af9cd6e1) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1498.json) |
|[Элемент управления 1499, контролируемый корпорацией Майкрософт, — правила поведения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe59671ab-9720-4ee2-9c60-170e8c82251e) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1499.json) |
|[Элемент управления 1500, контролируемый корпорацией Майкрософт, — правила поведения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9dd5b241-03cb-47d3-a5cd-4b89f9c53c92) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1500.json) |
|[Элемент управления 1501, контролируемый корпорацией Майкрософт, — правила поведения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F88817b58-8472-4f6c-81fa-58ce42b67f51) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1501.json) |

### <a name="rules-of-behavior--social-media-and-networking-restrictions"></a>Правила поведения | Ограничения в отношении социальных сетей и сайтов

**Идентификатор**. NIST SP 800-53 R4 PL-4 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1502, контролируемый корпорацией Майкрософт, — правила поведения \| Ограничения в отношении социальных сетей и сайтов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe901375c-8f01-4ac8-9183-d5312f47fe63) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1502.json) |

### <a name="information-security-architecture"></a>Архитектура информационной безопасности

**Идентификатор**. NIST SP 800-53 R4 PL-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1503, контролируемый корпорацией Майкрософт, — архитектура информационной безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc1fa9c2f-d439-4ab9-8b83-81fb1934f81d) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1503.json) |
|[Элемент управления 1504, контролируемый корпорацией Майкрософт, — архитектура информационной безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9e7c35d0-12d4-4e0c-80a2-8a352537aefd) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1504.json) |
|[Элемент управления 1505, контролируемый корпорацией Майкрософт, — архитектура информационной безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F813a10a7-3943-4fe3-8678-00dc52db5490) |Корпорация Майкрософт реализует этот элемент управления планированием |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1505.json) |

## <a name="personnel-security"></a>Безопасность персонала

### <a name="personnel-security-policy-and-procedures"></a>Политика и процедуры обеспечения безопасности персонала

**Идентификатор**. NIST SP 800-53 R4 PS-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1506, контролируемый корпорацией Майкрософт, — политика и процедуры обеспечения безопасности персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff7d2ff17-d604-4dd9-b607-9ecf63f28ad2) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1506.json) |
|[Элемент управления 1507, контролируемый корпорацией Майкрософт, — политика и процедуры обеспечения безопасности персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86ccd1bf-e7ad-4851-93ce-6ec817469c1e) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1507.json) |

### <a name="position-risk-designation"></a>Обозначение должностного риска

**Идентификатор**. NIST SP 800-53 R4 PS-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1508, контролируемый корпорацией Майкрософт, — обозначение должностного риска](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F76f500cc-4bca-4583-bda1-6d084dc21086) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1508.json) |
|[Элемент управления 1509, контролируемый корпорацией Майкрософт, — обозначение должностного риска](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F70792197-9bfc-4813-905a-bd33993e327f) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1509.json) |
|[Элемент управления 1510, контролируемый корпорацией Майкрософт, — обозначение должностного риска](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F79da5b09-0e7e-499e-adda-141b069c7998) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1510.json) |

### <a name="personnel-screening"></a>Проверка данных персонала

**Идентификатор**. NIST SP 800-53 R4 PS-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1511, контролируемый корпорацией Майкрософт, — проверка данных персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa9eae324-d327-4539-9293-b48e122465f8) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1511.json) |
|[Элемент управления 1512, контролируемый корпорацией Майкрософт, — проверка данных персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5a8324ad-f599-429b-aaed-f9c6e8c987a8) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1512.json) |

### <a name="personnel-screening--information-with-special-protection-measures"></a>Проверка данных персонала | Информация со специальными мерами защиты

**Идентификатор**. NIST SP 800-53 R4 PS-3 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1513, контролируемый корпорацией Майкрософт, — проверка данных персонала \| Информация со специальными мерами защиты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc416970d-b12b-49eb-8af4-fb144cd7c290) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1513.json) |
|[Элемент управления 1514, контролируемый корпорацией Майкрософт, — проверка данных персонала \| Информация со специальными мерами защиты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9ed5ca00-0e43-434e-a018-7aab91461ba7) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1514.json) |

### <a name="personnel-termination"></a>Увольнение персонала

**Идентификатор**. NIST SP 800-53 R4 PS-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1515, контролируемый корпорацией Майкрософт, — увольнение персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F02dd141a-a2b2-49a7-bcbd-ca31142f6211) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1515.json) |
|[Элемент управления 1516, контролируемый корпорацией Майкрософт, — увольнение персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fda3cd269-156f-435b-b472-c3af34c032ed) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1516.json) |
|[Элемент управления 1517, контролируемый корпорацией Майкрософт, — увольнение персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8f5ad423-50d6-4617-b058-69908f5586c9) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1517.json) |
|[Элемент управления 1518, контролируемый корпорацией Майкрософт, — увольнение персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d58f734-c052-40e9-8b2f-a1c2bff0b815) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1518.json) |
|[Элемент управления 1519, контролируемый корпорацией Майкрософт, — увольнение персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2f13915a-324c-4ab8-b45c-2eefeeefb098) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1519.json) |
|[Элемент управления 1520, контролируемый корпорацией Майкрософт, — увольнение персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7f2c513b-eb16-463b-b469-c10e5fa94f0a) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1520.json) |

### <a name="personnel-termination--automated-notification"></a>Увольнение персонала | Автоматическое уведомление

**Идентификатор**. NIST SP 800-53 R4 PS-4 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1521, контролируемый корпорацией Майкрософт, — увольнение персонала \| Автоматическое уведомление](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3cbddf9c-a3aa-4330-a0f5-4c0c1f1862e5) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1521.json) |

### <a name="personnel-transfer"></a>Перевод персонала

**Идентификатор**. NIST SP 800-53 R4 PS-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1522, контролируемый корпорацией Майкрософт, — перевод персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F38b470cc-f939-4a15-80e0-9f0c74f2e2c9) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1522.json) |
|[Элемент управления 1523, контролируемый корпорацией Майкрософт, — перевод персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5577a310-2551-49c8-803b-36e0d5e55601) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1523.json) |
|[Элемент управления 1524, контролируемый корпорацией Майкрософт, — перевод персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F72f1cb4e-2439-4fe8-88ea-b8671ce3c268) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1524.json) |
|[Элемент управления 1525, контролируемый корпорацией Майкрософт, — перевод персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9be2f688-7a61-45e3-8230-e1ec93893f66) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1525.json) |

### <a name="access-agreements"></a>Соглашения о доступе

**Идентификатор**. NIST SP 800-53 R4 PS-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1526, контролируемый корпорацией Майкрософт, — соглашения о доступе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F953e6261-a05a-44fd-8246-000e1a3edbb9) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1526.json) |
|[Элемент управления 1527, контролируемый корпорацией Майкрософт, — соглашения о доступе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2823de66-332f-4bfd-94a3-3eb036cd3b67) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1527.json) |
|[Элемент управления 1528, контролируемый корпорацией Майкрософт, — соглашения о доступе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdeb9797c-22f8-40e8-b342-a84003c924e6) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1528.json) |

### <a name="third-party-personnel-security"></a>Безопасность стороннего персонала

**Идентификатор**. NIST SP 800-53 R4 PS-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1529, контролируемый корпорацией Майкрософт, — безопасность стороннего персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd74fdc92-1cb8-4a34-9978-8556425cd14c) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1529.json) |
|[Элемент управления 1530, контролируемый корпорацией Майкрософт, — безопасность стороннего персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6e8f9566-29f1-49cd-b61f-f8628a3cf993) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1530.json) |
|[Элемент управления 1531, контролируемый корпорацией Майкрософт, — безопасность стороннего персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0643e0c-eee5-4113-8684-c608d05c5236) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1531.json) |
|[Элемент управления 1532, контролируемый корпорацией Майкрософт, — безопасность стороннего персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa2c66299-9017-4d95-8040-8bdbf7901d52) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1532.json) |
|[Элемент управления 1533, контролируемый корпорацией Майкрософт, — безопасность стороннего персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbba2a036-fb3b-4261-b1be-a13dfb5fbcaa) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1533.json) |

### <a name="personnel-sanctions"></a>Меры в отношении персонала

**Идентификатор**. NIST SP 800-53 R4 PS-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1534, контролируемый корпорацией Майкрософт, — меры в отношении персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8b2b263e-cd05-4488-bcbf-4debec7a17d9) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1534.json) |
|[Элемент управления 1535, контролируемый корпорацией Майкрософт, — меры в отношении персонала](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9a165d2-967d-4733-8399-1074270dae2e) |Корпорация Майкрософт реализует этот элемент управления безопасностью персонала |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1535.json) |

## <a name="risk-assessment"></a>Оценка угроз для безопасности

### <a name="risk-assessment-policy-and-procedures"></a>Процедуры и политика оценки риска

**Идентификатор**. NIST SP 800-53 R4 RA-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1536, контролируемый корпорацией Майкрософт, — политика и процедуры оценки рисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6e40d9de-2ad4-4cb5-8945-23143326a502) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1536.json) |
|[Элемент управления 1537, контролируемый корпорацией Майкрософт, — политика и процедуры оценки рисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb19454ca-0d70-42c0-acf5-ea1c1e5726d1) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1537.json) |

### <a name="security-categorization"></a>Классификация угроз для безопасности

**Идентификатор**. NIST SP 800-53 R4 RA-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1538, контролируемый корпорацией Майкрософт, — классификация угроз для безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1d7658b2-e827-49c3-a2ae-6d2bd0b45874) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1538.json) |
|[Элемент управления 1539, контролируемый корпорацией Майкрософт, — классификация угроз для безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faabb155f-e7a5-4896-a767-e918bfae2ee0) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1539.json) |
|[Элемент управления 1540, контролируемый корпорацией Майкрософт, — классификация угроз для безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff771f8cb-6642-45cc-9a15-8a41cd5c6977) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1540.json) |

### <a name="risk-assessment"></a>Оценка угроз для безопасности

**Идентификатор**. NIST SP 800-53 R4 RA-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1541, контролируемый корпорацией Майкрософт, — оценка рисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F70f6af82-7be6-44aa-9b15-8b9231b2e434) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1541.json) |
|[Элемент управления 1542, контролируемый корпорацией Майкрософт, — оценка рисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feab340d0-3d55-4826-a0e5-feebfeb0131d) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1542.json) |
|[Элемент управления 1543, контролируемый корпорацией Майкрософт, — оценка рисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffd00b778-b5b5-49c0-a994-734ea7bd3624) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1543.json) |
|[Элемент управления 1544, контролируемый корпорацией Майкрософт, — оценка рисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F43ced7c9-cd53-456b-b0da-2522649a4271) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1544.json) |
|[Элемент управления 1545, контролируемый корпорацией Майкрософт, — оценка рисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3f4b171a-a56b-4328-8112-32cf7f947ee1) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1545.json) |

### <a name="vulnerability-scanning"></a>Сканирование уязвимостей

**Идентификатор**. NIST SP 800-53 R4 RA-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить решение для оценки уязвимостей на виртуальных машинах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F501541f7-f7e7-4cd6-868c-4190fdad3ac9) |Осуществляет аудит виртуальных машин, чтобы определить, работает ли на них поддерживаемое решение для оценки уязвимостей. Основным компонентом каждой программы безопасности и защиты от киберрисков является идентификация и анализ уязвимостей. Ценовая категория "Стандартный" Центра безопасности Azure предоставляет возможность выполнять проверку уязвимостей виртуальных машин без дополнительных затрат. Кроме того, Центр безопасности может автоматически развернуть этот инструмент. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ServerVulnerabilityAssessment_Audit.json) |
|[В Управляемом экземпляре SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Проверка всех Управляемых экземпляров SQL без Расширенной защиты данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[На серверах SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Аудит серверов SQL без Расширенной защиты данных |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |
|[Элемент управления 1546, контролируемый корпорацией Майкрософт, — поиск уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2ce1ea7e-4038-4e53-82f4-63e8859333c1) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1546.json) |
|[Элемент управления 1547, контролируемый корпорацией Майкрософт, — поиск уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58abf9b8-c6d4-4b4b-bfb9-fe98fe295f52) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1547.json) |
|[Элемент управления 1548, контролируемый корпорацией Майкрософт, — поиск уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3afe6c78-6124-4d95-b85c-eb8c0c9539cb) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1548.json) |
|[Элемент управления 1549, контролируемый корпорацией Майкрософт, — поиск уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd6976a08-d969-4df2-bb38-29556c2eb48a) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1549.json) |
|[Элемент управления 1550, контролируемый корпорацией Майкрософт, — поиск уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F902908fb-25a8-4225-a3a5-5603c80066c9) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1550.json) |
|[Уязвимости конфигурации безопасности на ваших компьютерах должны быть устранены](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1e5fd5d-3e4c-4ce1-8661-7d1873ae6b15) |Серверы, не соответствующие заданным базовым показателям, будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OSVulnerabilities_Audit.json) |
|[Уязвимости в конфигурации безопасности в масштабируемом наборе виртуальных машин должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3c735d8a-a4ba-4a3a-b7cf-db7754cf57f4) |Аудит наличия уязвимостей ОС в ваших масштабируемых наборах виртуальных машин для защиты их от атак. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssOSVulnerabilities_Audit.json) |
|[Уязвимости в базах данных SQL должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffeedbf84-6b99-488c-acc2-71c829aa5ffc) |Мониторинг результатов оценки уязвимостей и рекомендации по исправлению уязвимостей баз данных. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_SQLDbVulnerabilities_Audit.json) |

### <a name="vulnerability-scanning--update-tool-capability"></a>Сканирование уязвимостей | Поддержка средства обновления

**Идентификатор**. NIST SP 800-53 R4 RA-5 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1551, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Поддержка средства обновления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5bbda922-0172-4095-89e6-5b4a0bf03af7) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1551.json) |

### <a name="vulnerability-scanning--update-by-frequency--prior-to-new-scan--when-identified"></a>Сканирование уязвимостей | Обновление по частоте, до нового сканирования, при обнаружении

**Идентификатор**. NIST SP 800-53 R4 RA-5 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1552, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Обновление по частоте, до нового сканирования, при обнаружении](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F43684572-e4f1-4642-af35-6b933bc506da) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1552.json) |

### <a name="vulnerability-scanning--breadth--depth-of-coverage"></a>Сканирование уязвимостей | Ширина и глубина охвата

**Идентификатор**. NIST SP 800-53 R4 RA-5 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1553, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Широта и глубина охвата](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9e5225fe-cdfb-4fce-9aec-0fe20dd53b62) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1553.json) |

### <a name="vulnerability-scanning--discoverable-information"></a>Сканирование уязвимостей | Информация, доступная для обнаружения

**Идентификатор**. NIST SP 800-53 R4 RA-5 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1554, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Информация, доступная для обнаружения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10984b4e-c93e-48d7-bf20-9c03b04e9eca) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1554.json) |

### <a name="vulnerability-scanning--privileged-access"></a>Поиск уязвимостей | Привилегированный доступ

**Идентификатор**. NIST SP 800-53 R4 RA-5 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1555, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Привилегированный доступ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5afa8cab-1ed7-4e40-884c-64e0ac2059cc) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1555.json) |

### <a name="vulnerability-scanning--automated-trend-analyses"></a>Поиск уязвимостей | Автоматизированный анализ тенденций

**Идентификатор**. NIST SP 800-53 R4 RA-5 (6) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1556, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Автоматизированный анализ тенденций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F391ff8b3-afed-405e-9f7d-ef2f8168d5da) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1556.json) |

### <a name="vulnerability-scanning--review-historic-audit-logs"></a>Сканирование уязвимостей | Проверка исторических журналов аудита

**Идентификатор**. NIST SP 800-53 R4 RA-5 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1557, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Проверка журналов аудита за прошлые периоды](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F36fbe499-f2f2-41b6-880e-52d7ea1d94a5) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1557.json) |

### <a name="vulnerability-scanning--correlate-scanning-information"></a>Поиск уязвимостей | Сопоставление информации сканирования

**Идентификатор**. NIST SP 800-53 R4 RA-5 (10) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1558, контролируемый корпорацией Майкрософт, — поиск уязвимостей \| Сопоставление информации сканирования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F65592b16-4367-42c5-a26e-d371be450e17) |Корпорация Майкрософт реализует этот элемент управления оценкой угроз для безопасности |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1558.json) |

## <a name="system-and-services-acquisition"></a>Приобретение систем и служб

### <a name="system-and-services-acquisition-policy-and-procedures"></a>Политика и процедуры приобретения систем и служб

**Идентификатор**. NIST SP 800-53 R4 SA-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1559, контролируемый корпорацией Майкрософт, — политика и процедуры приобретения систем и служб](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F45692294-f074-42bd-ac54-16f1a3c07554) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1559.json) |
|[Элемент управления 1560, контролируемый корпорацией Майкрософт, — политика и процедуры приобретения систем и служб](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe29e0915-5c2f-4d09-8806-048b749ad763) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1560.json) |

### <a name="allocation-of-resources"></a>Выделение ресурсов

**Идентификатор**. NIST SP 800-53 R4 SA-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1561, контролируемый корпорацией Майкрософт, — выделение ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F40364c3f-c331-4e29-b1e3-2fbe998ba2f5) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1561.json) |
|[Элемент управления 1562, контролируемый корпорацией Майкрософт, — выделение ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd4142013-7964-4163-a313-a900301c2cef) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1562.json) |
|[Элемент управления 1563, контролируемый корпорацией Майкрософт, — выделение ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9afe2edf-232c-4fdf-8e6a-e867a5c525fd) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1563.json) |

### <a name="system-development-life-cycle"></a>Жизненный цикл разработки системы

**Идентификатор**. NIST SP 800-53 R4 SA-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1564, контролируемый корпорацией Майкрософт, — жизненный цикл разработки системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F157f0ef9-143f-496d-b8f9-f8c8eeaad801) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1564.json) |
|[Элемент управления 1565, контролируемый корпорацией Майкрософт, — жизненный цикл разработки системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F45ce2396-5c76-4654-9737-f8792ab3d26b) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1565.json) |
|[Элемент управления 1566, контролируемый корпорацией Майкрософт, — жизненный цикл разработки системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F50ad3724-e2ac-4716-afcc-d8eabd97adb9) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1566.json) |
|[Элемент управления 1567, контролируемый корпорацией Майкрософт, — жизненный цикл разработки системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe72edbf6-aa61-436d-a227-0f32b77194b3) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1567.json) |

### <a name="acquisition-process"></a>Процесс приобретения

**Идентификатор**. NIST SP 800-53 R4 SA-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1568, контролируемый корпорацией Майкрософт, — процесс приобретения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6a8eae8-9854-495a-ac82-d2cd3eac02a6) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1568.json) |
|[Элемент управления 1569, контролируемый корпорацией Майкрософт, — процесс приобретения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fad2f8e61-a564-4dfd-8eaa-816f5be8cb34) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1569.json) |
|[Элемент управления 1570, контролируемый корпорацией Майкрософт, — процесс приобретения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7fcf38d-bb09-4600-be7d-825046eb162a) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1570.json) |
|[Элемент управления 1571, контролируемый корпорацией Майкрософт, — процесс приобретения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb11c985b-f2cd-4bd7-85f4-b52426edf905) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1571.json) |
|[Элемент управления 1572, контролируемый корпорацией Майкрософт, — процесс приобретения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F04f5fb00-80bb-48a9-a75b-4cb4d4c97c36) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1572.json) |
|[Элемент управления 1573, контролируемый корпорацией Майкрософт, — процесс приобретения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58c93053-7b98-4cf0-b99f-1beb985416c2) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1573.json) |
|[Элемент управления 1574, контролируемый корпорацией Майкрософт, — процесс приобретения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0f935dab-83d6-47b8-85ef-68b8584161b9) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1574.json) |

### <a name="acquisition-process--functional-properties-of-security-controls"></a>Процесс приобретения | Функциональные свойства средств управления безопасностью

**Идентификатор**. NIST SP 800-53 R4 SA-4 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1575, контролируемый корпорацией Майкрософт, — процесс приобретения \| Функциональные свойства средств управления безопасностью](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F93e1bb73-1b08-4dbe-9c62-8e2e92e7ec41) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1575.json) |

### <a name="acquisition-process--design--implementation-information-for-security-controls"></a>Процесс приобретения | Информация по разработке и реализации для средств управления безопасностью

**Идентификатор**. NIST SP 800-53 R4 SA-4 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1576, контролируемый корпорацией Майкрософт, — процесс приобретения \| Информация по разработке и реализации для средств управления безопасностью](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5f18c885-ade3-48c5-80b1-8f9216019c18) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1576.json) |

### <a name="acquisition-process--continuous-monitoring-plan"></a>Процесс приобретения | План непрерывного мониторинга

**Идентификатор**. NIST SP 800-53 R4 SA-4 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1577, контролируемый корпорацией Майкрософт, — процесс приобретения \| План непрерывного мониторинга](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd922484a-8cfc-4a6b-95a4-77d6a685407f) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1577.json) |

### <a name="acquisition-process--functions--ports--protocols--services-in-use"></a>Процесс приобретения | Используемые функции, порты, протоколы и службы

**Идентификатор**. NIST SP 800-53 R4 SA-4 (9) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1578, контролируемый корпорацией Майкрософт, — процесс приобретения \| Используемые функции, порты, протоколы и службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F45b7b644-5f91-498e-9d89-7402532d3645) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1578.json) |

### <a name="acquisition-process--use-of-approved-piv-products"></a>Процесс приобретения | Использование утвержденных продуктов для проверки личности

**Идентификатор**. NIST SP 800-53 R4 SA-4 (10) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1579, контролируемый корпорацией Майкрософт, — процесс приобретения \| Использование утвержденных продуктов для проверки личности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4e54c7ef-7457-430b-9a3e-ef8881d4a8e0) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1579.json) |

### <a name="information-system-documentation"></a>Документация по информационной системе

**Идентификатор**. NIST SP 800-53 R4 SA-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1580, контролируемый корпорацией Майкрософт, — документация по информационной системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F854db8ac-6adf-42a0-bef3-b73f764f40b9) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1580.json) |
|[Элемент управления 1581, контролируемый корпорацией Майкрософт, — документация по информационной системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F742b549b-7a25-465f-b83c-ea1ffb4f4e0e) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1581.json) |
|[Элемент управления 1582, контролируемый корпорацией Майкрософт, — документация по информационной системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcd9e2f38-259b-462c-bfad-0ad7ab4e65c5) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1582.json) |
|[Элемент управления 1583, контролируемый корпорацией Майкрософт, — документация по информационной системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0882d488-8e80-4466-bc0f-0cd15b6cb66d) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1583.json) |
|[Элемент управления 1584, контролируемый корпорацией Майкрософт, — документация по информационной системе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5864522b-ff1d-4979-a9f8-58bee1fb174c) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1584.json) |

### <a name="security-engineering-principles"></a>Принципы техники обеспечения безопасности

**Идентификатор**. NIST SP 800-53 R4 SA-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1585, контролируемый корпорацией Майкрософт, — принципы техники обеспечения безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd57f8732-5cdc-4cda-8d27-ab148e1f3a55) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1585.json) |

### <a name="external-information-system-services"></a>Внешние информационные системные службы

**Идентификатор**. NIST SP 800-53 R4 SA-9 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1586, контролируемый корпорацией Майкрософт, — внешние информационные системные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6e3b2fbd-8f37-4766-a64d-3f37703dcb51) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1586.json) |
|[Элемент управления 1587, контролируемый корпорацией Майкрософт, — внешние информационные системные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F32820956-9c6d-4376-934c-05cd8525be7c) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1587.json) |
|[Элемент управления 1588, контролируемый корпорацией Майкрософт, — внешние информационные системные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F68ebae26-e0e0-4ecb-8379-aabf633b51e9) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1588.json) |

### <a name="external-information-system-services--risk-assessments--organizational-approvals"></a>Внешние информационные системные службы | Оценка риска и утверждения организации

**Идентификатор**. NIST SP 800-53 R4 SA-9 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1589, контролируемый корпорацией Майкрософт, — внешние информационные системные службы \| Оценка риска и утверждения организации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86ec7f9b-9478-40ff-8cfd-6a0d510081a8) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1589.json) |
|[Элемент управления 1590, контролируемый корпорацией Майкрософт, — внешние информационные системные службы \| Оценка риска и утверждения организации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbf296b8c-f391-4ea4-9198-be3c9d39dd1f) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1590.json) |

### <a name="external-information-system-services--identification-of-functions--ports--protocols--services"></a>Внешние информационные системные службы | Определение функций, портов, протоколов и служб

**Идентификатор**. NIST SP 800-53 R4 SA-9 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1591, контролируемый корпорацией Майкрософт, — внешние информационные системные службы \| Определение функций, портов, протоколов и служб](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff751cdb7-fbee-406b-969b-815d367cb9b3) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1591.json) |

### <a name="external-information-system-services--consistent-interests-of-consumers-and-providers"></a>Внешние информационные системные службы | Согласованные интересы клиентов и поставщиков

**Идентификатор**. NIST SP 800-53 R4 SA-9 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1592, контролируемый корпорацией Майкрософт, — внешние информационные системные службы \| Согласованные интересы клиентов и поставщиков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1d01ba6c-289f-42fd-a408-494b355b6222) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1592.json) |

### <a name="external-information-system-services--processing-storage-and-service-location"></a>Внешние информационные системные службы | Расположение экземпляра обработки, хранилища и службы

**Идентификатор**. NIST SP 800-53 R4 SA-9 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1593, контролируемый корпорацией Майкрософт, — внешние информационные системные службы \| Расположение экземпляра обработки, хранилища и службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2cd0a426-b5f5-4fe0-9539-a6043cdbc6fa) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1593.json) |

### <a name="developer-configuration-management"></a>Управление конфигурацией со стороны разработчика

**Идентификатор**. NIST SP 800-53 R4 SA-10 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1594, контролируемый корпорацией Майкрософт, — управление конфигурацией со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F042ba2a1-8bb8-45f4-b080-c78cf62b90e9) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1594.json) |
|[Элемент управления 1595, контролируемый корпорацией Майкрософт, — управление конфигурацией со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1e0414e7-6ef5-4182-8076-aa82fbb53341) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1595.json) |
|[Элемент управления 1596, контролируемый корпорацией Майкрософт, — управление конфигурацией со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F21e25e01-0ae0-41be-919e-04ce92b8e8b8) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1596.json) |
|[Элемент управления 1597, контролируемый корпорацией Майкрософт, — управление конфигурацией со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F68b250ec-2e4f-4eee-898a-117a9fda7016) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1597.json) |
|[Элемент управления 1598, контролируемый корпорацией Майкрософт, — управление конфигурацией со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fae7e1f5e-2d63-4b38-91ef-bce14151cce3) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1598.json) |

### <a name="developer-configuration-management--software--firmware-integrity-verification"></a>Управление конфигурацией со стороны разработчика | Проверка целостности программного обеспечения и встроенного ПО

**Идентификатор**. NIST SP 800-53 R4 SA-10 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1599, контролируемый корпорацией Майкрософт, — управление конфигурацией со стороны разработчика \| Проверка целостности программного обеспечения и встроенного ПО](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0004bbf0-5099-4179-869e-e9ffe5fb0945) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1599.json) |

### <a name="developer-security-testing-and-evaluation"></a>Тестирование и оценка безопасности со стороны разработчика

**Идентификатор**. NIST SP 800-53 R4 SA-11 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1600, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc53f3123-d233-44a7-930b-f40d3bfeb7d6) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1600.json) |
|[Элемент управления 1601, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ee79a0c-addf-4ce9-9b3c-d9576ed5e20e) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1601.json) |
|[Элемент управления 1602, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fddae2e97-a449-499f-a1c8-aea4a7e52ec9) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1602.json) |
|[Элемент управления 1603, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2b909c26-162f-47ce-8e15-0c1f55632eac) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1603.json) |
|[Элемент управления 1604, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F44dbba23-0b61-478e-89c7-b3084667782f) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1604.json) |

### <a name="developer-security-testing-and-evaluation--static-code-analysis"></a>Тестирование и оценка безопасности со стороны разработчика | Анализ статического кода

**Идентификатор**. NIST SP 800-53 R4 SA-11 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1605, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика \| Анализ статического кода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0062eb8b-dc75-4718-8ea5-9bb4a9606655) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1605.json) |

### <a name="developer-security-testing-and-evaluation--threat-and-vulnerability-analyses"></a>Тестирование и оценка безопасности со стороны разработчика | Анализ угроз и уязвимостей

**Идентификатор**. NIST SP 800-53 R4 SA-11 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1606, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика \| Анализ угроз и уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbaa8a9a4-5bbe-4c72-98f6-a3a47ae2b1ca) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1606.json) |

### <a name="developer-security-testing-and-evaluation--dynamic-code-analysis"></a>Тестирование и оценка безопасности со стороны разработчика | Анализ динамического кода

**Идентификатор**. NIST SP 800-53 R4 SA-11 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1607, контролируемый корпорацией Майкрософт, — тестирование и оценка безопасности со стороны разработчика \| Анализ динамического кода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F976a74cf-b192-4d35-8cab-2068f272addb) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1607.json) |

### <a name="supply-chain-protection"></a>Защита цепочки поставок

**Идентификатор**. NIST SP 800-53 R4 SA-12 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1608, контролируемый корпорацией Майкрософт, — защита цепочки поставок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb73b7b3b-677c-4a2a-b949-ad4dc4acd89f) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1608.json) |

### <a name="development-process-standards-and-tools"></a>Процессы, стандарты и средства разработки

**Идентификатор**. NIST SP 800-53 R4 SA-15 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1609, контролируемый корпорацией Майкрософт, — процессы, стандарты и средства разработки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9e93fa71-42ac-41a7-b177-efbfdc53c69f) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1609.json) |
|[Элемент управления 1610, контролируемый корпорацией Майкрософт, — процессы, стандарты и средства разработки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb9f3fb54-4222-46a1-a308-4874061f8491) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1610.json) |

### <a name="developer-provided-training"></a>Предоставляемое разработчиком обучение

**Идентификатор**. NIST SP 800-53 R4 SA-16 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1611, контролируемый корпорацией Майкрософт, — предоставляемое разработчиком обучение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffdda8a0c-ac32-43f6-b2f4-7dc1df03f43f) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1611.json) |

### <a name="developer-security-architecture-and-design"></a>Создание разработчиком архитектуры и структуры безопасности

**Идентификатор**. NIST SP 800-53 R4 SA-17 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1612, контролируемый корпорацией Майкрософт, — создание разработчиком архитектуры и структуры безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa2037b3d-8b04-4171-8610-e6d4f1d08db5) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1612.json) |
|[Элемент управления 1613, контролируемый корпорацией Майкрософт, — создание разработчиком архитектуры и структуры безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffe2ad78b-8748-4bff-a924-f74dfca93f30) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1613.json) |
|[Элемент управления 1614, контролируемый корпорацией Майкрософт, — создание разработчиком архитектуры и структуры безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8154e3b3-cc52-40be-9407-7756581d71f6) |Корпорация Майкрософт реализует этот элемент управления приобретением систем и служб |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1614.json) |

## <a name="system-and-communications-protection"></a>Защита системы и средств передачи данных

### <a name="system-and-communications-protection-policy-and-procedures"></a>Политика и процедуры защиты системы и средств передачи данных

**Идентификатор**. NIST SP 800-53 R4 SC-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1615, контролируемый корпорацией Майкрософт, — политика и процедуры защиты системы и средств передачи данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff35e02aa-0a55-49f8-8811-8abfa7e6f2c0) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1615.json) |
|[Элемент управления 1616, контролируемый корпорацией Майкрософт, — политика и процедуры защиты системы и средств передачи данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2006457a-48b3-4f7b-8d2e-1532287f9929) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1616.json) |

### <a name="application-partitioning"></a>Секционирование приложений

**Идентификатор**. NIST SP 800-53 R4 SC-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1617, контролируемый корпорацией Майкрософт, — секционирование приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa631d8f5-eb81-4f9d-9ee1-74431371e4a3) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1617.json) |

### <a name="security-function-isolation"></a>Изоляции функций безопасности

**Идентификатор**. NIST SP 800-53 R4 SC-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1618, контролируемый корпорацией Майкрософт, — изоляция функций безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff52f89aa-4489-4ec4-950e-8c96a036baa9) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1618.json) |

### <a name="information-in-shared-resources"></a>Данные в общих ресурсах

**Идентификатор**. NIST SP 800-53 R4 SC-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1619, контролируемый корпорацией Майкрософт, — информация в общих ресурсах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc722e569-cb52-45f3-a643-836547d016e1) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1619.json) |

### <a name="denial-of-service-protection"></a>Защита от отказа в обслуживании

**Идентификатор**. NIST SP 800-53 R4 SC-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Защита от атак DDoS Azure уровня "Стандартный" должна быть включена](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7aca53f-2ed4-4466-a25e-0b45ade68efd) |Стандарт защиты от DDoS должен быть включен для всех виртуальных сетей с подсетью, которая является частью шлюза приложения с общедоступным IP-адресом. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableDDoSProtection_Audit.json) |
|[Элемент управления 1620, контролируемый корпорацией Майкрософт, — защита от отказа в обслуживании](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd17c826b-1dec-43e1-a984-7b71c446649c) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1620.json) |

### <a name="resource-availability"></a>Доступность ресурсов

**Идентификатор**. NIST SP 800-53 R4 SC-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1621, контролируемый корпорацией Майкрософт, — доступность ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3cb9f731-744a-4691-a481-ca77b0411538) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1621.json) |

### <a name="boundary-protection"></a>Защита границ

**Идентификатор**. NIST SP 800-53 R4 SC-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Доступ ко всем сетевым портам должен быть ограничен в связанных с виртуальной машиной группах безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9daedab3-fb2d-461e-b861-71790eead4f6) |Центр безопасности Azure определил, что некоторые правила для входящих подключений в группах безопасности сети недостаточно строгие. Правила для входящих подключений не должны разрешать доступ из диапазонов "Любой" или "Интернет". В противном случае злоумышленники смогут атаковать ваши ресурсы. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnprotectedEndpoints_Audit.json) |
|[Элемент управления 1622, контролируемый корпорацией Майкрософт, — защита границ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fecf56554-164d-499a-8d00-206b07c27bed) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1622.json) |
|[Элемент управления 1623, контролируемый корпорацией Майкрософт, — защита границ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F02ce1b22-412a-4528-8630-c42146f917ed) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1623.json) |
|[Элемент управления 1624, контролируемый корпорацией Майкрософт, — защита границ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F37d079e3-d6aa-4263-a069-dd7ac6dd9684) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1624.json) |
|[Учетные записи хранения должны ограничивать доступ к сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34c877ad-507e-4c82-993e-3452a6e0ad3c) |Сетевой доступ к учетным записям хранения должен быть ограниченным. Настройте правила сети так, чтобы учетная запись хранения была доступна только приложениям из разрешенных сетей. Чтобы разрешить подключения от конкретных локальных клиентов и интернет-клиентов, вы можете открыть доступ для трафика из конкретных виртуальных сетей Azure или определенных диапазонов общедоступных IP-адресов. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_NetworkAcls_Audit.json) |

### <a name="boundary-protection--access-points"></a>Защита границ | Точки доступа

**Идентификатор**. NIST SP 800-53 R4 SC-7 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |
|[Элемент управления 1625, контролируемый корпорацией Майкрософт, — защита границ \| Точки доступа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb9b66a4d-70a1-4b47-8fa1-289cec68c605) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1625.json) |

### <a name="boundary-protection--external-telecommunications-services"></a>Защита границ | Внешние телекоммуникационные службы

**Идентификатор**. NIST SP 800-53 R4 SC-7 (4) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |
|[Элемент управления 1626, контролируемый корпорацией Майкрософт, — защита границ \| Внешние телекоммуникационные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe8f6bddd-6d67-439a-88d4-c5fe39a79341) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1626.json) |
|[Элемент управления 1627, контролируемый корпорацией Майкрософт, — защита границ \| Внешние телекоммуникационные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffd73310d-76fc-422d-bda4-3a077149f179) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1627.json) |
|[Элемент управления 1628, контролируемый корпорацией Майкрософт, — защита границ \| Внешние телекоммуникационные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F67de62b4-a737-4781-8861-3baed3c35069) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1628.json) |
|[Элемент управления 1629, контролируемый корпорацией Майкрософт, — защита границ \| Внешние телекоммуникационные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc171b095-7756-41de-8644-a062a96043f2) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1629.json) |
|[Элемент управления 1630, контролируемый корпорацией Майкрософт, — защита границ \| Внешние телекоммуникационные службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3643717a-3897-4bfd-8530-c7c96b26b2a0) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1630.json) |

### <a name="boundary-protection--deny-by-default--allow-by-exception"></a>Защита границ | Запрет по умолчанию или разрешение исключений

**Идентификатор**. NIST SP 800-53 R4 SC-7 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1631, контролируемый корпорацией Майкрософт, — защита границ \| Запрет по умолчанию или разрешение исключений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F74ae9b8e-e7bb-4c9c-992f-c535282f7a2c) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1631.json) |

### <a name="boundary-protection--prevent-split-tunneling-for-remote-devices"></a>Защита границ | Предотвращение раздельного туннелирования для удаленных устройств

**Идентификатор**. NIST SP 800-53 R4 SC-7 (7) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1632, контролируемый корпорацией Майкрософт, — защита границ \| Предотвращение раздельного туннелирования для удаленных устройств](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4ce9073a-77fa-48f0-96b1-87aa8e6091c2) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1632.json) |

### <a name="boundary-protection--route-traffic-to-authenticated-proxy-servers"></a>Защита границ | Маршрутизация трафика на аутентифицированные прокси-серверы

**Идентификатор**. NIST SP 800-53 R4 SC-7 (8) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1633, контролируемый корпорацией Майкрософт, — защита границ \| Маршрутизация трафика на аутентифицированные прокси-серверы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F07557aa0-e02f-4460-9a81-8ecd2fed601a) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1633.json) |

### <a name="boundary-protection--prevent-unauthorized-exfiltration"></a>Защита границ | Предотвращение несанкционированной утечки данных

**Идентификатор**. NIST SP 800-53 R4 SC-7 (10) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1634, контролируемый корпорацией Майкрософт, — защита границ \| Предотвращение несанкционированной утечки данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F292a7c44-37fa-4c68-af7c-9d836955ded2) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1634.json) |

### <a name="boundary-protection--host-based-protection"></a>Защита границ | Защиты на основе узлов

**Идентификатор**. NIST SP 800-53 R4 SC-7 (12) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1635, контролируемый корпорацией Майкрософт, — защита границ \| Защита на основе узлов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F87551b5d-1deb-4d0f-86cc-9dc14cb4bf7e) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1635.json) |

### <a name="boundary-protection--isolation-of-security-tools--mechanisms--support-components"></a>Защита границ | Изоляция инструментов, механизмов и компонентов обеспечения безопасности

**Идентификатор**. NIST SP 800-53 R4 SC-7 (13) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1636, контролируемый корпорацией Майкрософт, — защита границ \| Изоляция инструментов, механизмов и компонентов обеспечения безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7b694eed-7081-43c6-867c-41c76c961043) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1636.json) |

### <a name="boundary-protection--fail-secure"></a>Защита границ | Отказоустойчивость

**Идентификатор**. NIST SP 800-53 R4 SC-7 (18) **Ответственность**: Microsoft

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1637, контролируемый корпорацией Майкрософт, — защита границ \| Отказоустойчивость](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4075bedc-c62a-4635-bede-a01be89807f3) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1637.json) |

### <a name="boundary-protection--dynamic-isolation--segregation"></a>Защита границ | Динамическая изоляция и отделение

**Идентификатор**. NIST SP 800-53 R4 SC-7 (20) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1638, контролируемый корпорацией Майкрософт, — защита границ \| Динамическая изоляция и разделение](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F49b99653-32cd-405d-a135-e7d60a9aae1f) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1638.json) |

### <a name="boundary-protection--isolation-of-information-system-components"></a>Защита границ | Изоляция компонентов информационной системы

**Идентификатор**. NIST SP 800-53 R4 SC-7 (21) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1639, контролируемый корпорацией Майкрософт, — защита границ \| Изоляция компонентов информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F78e8e649-50f6-4fe3-99ac-fedc2e63b03f) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1639.json) |

### <a name="transmission-confidentiality-and-integrity"></a>Конфиденциальность и целостность передаваемых данных

**Идентификатор**. NIST SP 800-53 R4 SC-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1640, контролируемый корпорацией Майкрософт, — конфиденциальность и целостность передаваемых данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F05a289ce-6a20-4b75-a0f3-dc8601b6acd0) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1640.json) |

### <a name="transmission-confidentiality-and-integrity--cryptographic-or-alternate-physical-protection"></a>Конфиденциальность и целостность передаваемых данных | Шифрование или дополнительная физическая защита

**Идентификатор**. NIST SP 800-53 R4 SC-8 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |
|[Элемент управления 1641, контролируемый корпорацией Майкрософт, — конфиденциальность и целостность передаваемых данных \| Шифрование или дополнительная физическая защита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd39d4f68-7346-4133-8841-15318a714a24) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1641.json) |
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |
|[Веб-серверы Windows должны быть настроены на использование протоколов защищенного обмена данными](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5752e6d6-1206-46d8-8ab1-ecc2f71a8112) |Для защиты конфиденциальности данных, передаваемых через Интернет, веб-серверы должны использовать последнюю версию стандартного протокола шифрования TLS. Протокол TLS обеспечивает защиту при передаче данных по сети с помощью сертификатов безопасности для шифрования подключений между компьютерами. TLS 1.3 быстрее и безопаснее, чем предыдущие версии (TLS 1.0–1.2 и SSL 2–3), которые считаются устаревшими протоколами. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SecureWebProtocol_AINE.json) |

### <a name="network-disconnect"></a>Отключение сети

**Идентификатор**. NIST SP 800-53 R4 SC-10 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1642, контролируемый корпорацией Майкрософт, — отключение сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F53397227-5ee3-4b23-9e5e-c8a767ce6928) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1642.json) |

### <a name="cryptographic-key-establishment-and-management"></a>Установка криптографических ключей и управление ими

**Идентификатор**. NIST SP 800-53 R4 SC-12 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1643, контролируемый корпорацией Майкрософт, — установка криптографических ключей и управление ими](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d8d492c-dd7a-46f7-a723-fa66a425b87c) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1643.json) |

### <a name="cryptographic-key-establishment-and-management--availability"></a>Установка криптографических ключей и управление ими | Доступность

**Идентификатор**. NIST SP 800-53 R4 SC-12 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1644, контролируемый корпорацией Майкрософт, — установка криптографических ключей и управление ими \| Доступность](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7211477-c970-446b-b4af-062f37461147) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1644.json) |

### <a name="cryptographic-key-establishment-and-management--symmetric-keys"></a>Установка криптографических ключей и управление ими | Симметричные ключи

**Идентификатор**. NIST SP 800-53 R4 SC-12 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1645, контролируемый корпорацией Майкрософт, — установка криптографических ключей и управление ими \| Симметричные ключи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fafbd0baf-ff1a-4447-a86f-088a97347c0c) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1645.json) |

### <a name="cryptographic-key-establishment-and-management--asymmetric-keys"></a>Установка криптографических ключей и управление ими | Асимметричные ключи

**Идентификатор**. NIST SP 800-53 R4 SC-12 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1646, контролируемый корпорацией Майкрософт, — установка криптографических ключей и управление ими \| Асимметричные ключи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F506814fa-b930-4b10-894e-a45b98c40e1a) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1646.json) |

### <a name="cryptographic-protection"></a>Криптографическая защита

**Идентификатор**. NIST SP 800-53 R4 SC-13 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1647, контролируемый корпорацией Майкрософт, — криптографическая защита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F791cfc15-6974-42a0-9f4c-2d4b82f4a78c) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1647.json) |

### <a name="collaborative-computing-devices"></a>Совместное работающие вычислительные устройства

**Идентификатор**. NIST SP 800-53 R4 SC-15 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1648, контролируемый корпорацией Майкрософт, — совместно работающие вычислительные устройства](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3a9eb14b-495a-4ebb-933c-ce4ef5264e32) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1648.json) |
|[Элемент управления 1649, контролируемый корпорацией Майкрософт, — совместно работающие вычислительные устройства](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F26d292cc-b0b8-4c29-9337-68abc758bf7b) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1649.json) |

### <a name="public-key-infrastructure-certificates"></a>Сертификаты инфраструктуры открытых ключей

**Идентификатор**. NIST SP 800-53 R4 SC-17 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1650, контролируемый корпорацией Майкрософт, — сертификаты инфраструктуры открытого ключа](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F201d3740-bd16-4baf-b4b8-7cda352228b7) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1650.json) |

### <a name="mobile-code"></a>Мобильный код

**Идентификатор**. NIST SP 800-53 R4 SC-18 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1651, контролируемый корпорацией Майкрософт, — мобильный код](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6db63528-c9ba-491c-8a80-83e1e6977a50) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1651.json) |
|[Элемент управления 1652, контролируемый корпорацией Майкрософт, — мобильный код](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6998e84a-2d29-4e10-8962-76754d4f772d) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1652.json) |
|[Элемент управления 1653, контролируемый корпорацией Майкрософт, — мобильный код](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6b1c00a7-7fd0-42b0-8c5b-c45f6fa1f71b) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1653.json) |

### <a name="voice-over-internet-protocol"></a>Голосовая связь через протокол IP

**Идентификатор**. NIST SP 800-53 R4 SC-19 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1654, контролируемый корпорацией Майкрософт, — протокол VoIP](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0a2ee16e-ab1f-414a-800b-d1608835862b) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1654.json) |
|[Элемент управления 1655, контролируемый корпорацией Майкрософт, — протокол VoIP](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F121eab72-390e-4629-a7e2-6d6184f57c6b) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1655.json) |

### <a name="secure-name--address-resolution-service-authoritative-source"></a>Безопасная служба разрешения имен и адресов (заслуживающий доверия источник)

**Идентификатор**. NIST SP 800-53 R4 SC-20 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1656, контролируемый корпорацией Майкрософт, — безопасная служба разрешения имен и адресов (заслуживающий доверия источник)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1cb067d5-c8b5-4113-a7ee-0a493633924b) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1656.json) |
|[Элемент управления 1657, контролируемый корпорацией Майкрософт, — безопасная служба разрешения имен и адресов (заслуживающий доверия источник)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F90f01329-a100-43c2-af31-098996135d2b) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1657.json) |

### <a name="secure-name--address-resolution-service-recursive-or-caching-resolver"></a>Безопасная служба разрешения имен и адресов (рекурсивный или кэширующий сопоставитель)

**Идентификатор**. NIST SP 800-53 R4 SC-21 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1658, контролируемый корпорацией Майкрософт, — безопасная служба разрешения имен и адресов (рекурсивный или кэширующий сопоставитель)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F063b540e-4bdc-4e7a-a569-3a42ddf22098) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1658.json) |

### <a name="architecture-and-provisioning-for-name--address-resolution-service"></a>Архитектура службы разрешения имен и адресов и ее подготовка

**Идентификатор**. NIST SP 800-53 R4 SC-22 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1659, контролируемый корпорацией Майкрософт, — архитектура службы разрешения имен и адресов и ее подготовка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F35a4102f-a778-4a2e-98c2-971056288df8) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1659.json) |

### <a name="session-authenticity"></a>Подлинность сеансов

**Идентификатор**. NIST SP 800-53 R4 SC-23 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1660, контролируемый корпорацией Майкрософт, — подлинность сеансов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F63096613-ce83-43e5-96f4-e588e8813554) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1660.json) |

### <a name="session-authenticity--invalidate-session-identifiers-at-logout"></a>Подлинность сеанса | Аннулирование идентификаторов сеансов при выходе

**Идентификатор**. NIST SP 800-53 R4 SC-23 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1661, контролируемый корпорацией Майкрософт, — подлинность сеансов \| Аннулирование идентификаторов сеансов при выходе](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4c643c9a-1be7-4016-a5e7-e4bada052920) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1661.json) |

### <a name="fail-in-known-state"></a>Сбой с переходом в известное состояние

**Идентификатор**. NIST SP 800-53 R4 SC-24 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1662, контролируемый корпорацией Майкрософт, — сбой с переходом в известное состояние](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F165cb91f-7ea8-4ab7-beaf-8636b98c9d15) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1662.json) |

### <a name="protection-of-information-at-rest"></a>Защита неактивных данных

**Идентификатор**. NIST SP 800-53 R4 SC-28 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1663, контролируемый корпорацией Майкрософт, — защита неактивных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F60171210-6dde-40af-a144-bf2670518bfa) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1663.json) |

### <a name="protection-of-information-at-rest--cryptographic-protection"></a>Защита неактивных данных | Криптографическая защита

**Идентификатор**. NIST SP 800-53 R4 SC-28 (1) **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемом экземпляре SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Проверка всех Управляемых экземпляров SQL без Расширенной защиты данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[На серверах SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Аудит серверов SQL без Расширенной защиты данных |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |
|[На виртуальных машинах должно применяться шифрование дисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) |Виртуальные машины без шифрования дисков будут отслеживаться Центром безопасности Azure (это требование носит рекомендательный характер). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnencryptedVMDisks_Audit.json) |
|[Элемент управления 1664, контролируемый корпорацией Майкрософт, — защита неактивных данных \| Криптографическая защита](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa2cdf6b8-9505-4619-b579-309ba72037ac) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1664.json) |
|[В базах данных SQL должно применяться прозрачное шифрование данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17k78e20-9358-41c9-923c-fb736d382a12) |Для защиты неактивных данных и обеспечения соответствия требованиям должно быть включено прозрачное шифрование данных. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlDBEncryption_Audit.json) |

### <a name="process-isolation"></a>Изоляция процессов

**Идентификатор**. NIST SP 800-53 R4 SC-39 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1665, контролируемый корпорацией Майкрософт, — изоляция процессов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5df3a55c-8456-44d4-941e-175f79332512) |Корпорация Майкрософт реализует этот элемент управления защитой системы и средств передачи данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1665.json) |

## <a name="system-and-information-integrity"></a>Целостность системы и данных

### <a name="system-and-information-integrity-policy-and-procedures"></a>Политика и процедуры защиты целостности системы и данных

**Идентификатор**. NIST SP 800-53 R4 SI-1 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1666, контролируемый корпорацией Майкрософт, — политика и процедуры защиты целостности системы и данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F12e30ee3-61e6-4509-8302-a871e8ebb91e) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1666.json) |
|[Элемент управления 1667, контролируемый корпорацией Майкрософт, — политика и процедуры защиты целостности системы и данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd61880dc-6e38-4f2a-a30c-3406a98f8220) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1667.json) |

### <a name="flaw-remediation"></a>Исправление ошибок

**Идентификатор**. NIST SP 800-53 R4 SI-2 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить решение для оценки уязвимостей на виртуальных машинах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F501541f7-f7e7-4cd6-868c-4190fdad3ac9) |Осуществляет аудит виртуальных машин, чтобы определить, работает ли на них поддерживаемое решение для оценки уязвимостей. Основным компонентом каждой программы безопасности и защиты от киберрисков является идентификация и анализ уязвимостей. Ценовая категория "Стандартный" Центра безопасности Azure предоставляет возможность выполнять проверку уязвимостей виртуальных машин без дополнительных затрат. Кроме того, Центр безопасности может автоматически развернуть этот инструмент. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ServerVulnerabilityAssessment_Audit.json) |
|[Элемент управления 1668, контролируемый корпорацией Майкрософт, — исправление ошибок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8fb0966e-be1d-42c3-baca-60df5c0bcc61) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1668.json) |
|[Элемент управления 1669, контролируемый корпорацией Майкрософт, — исправление ошибок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48f2f62b-5743-4415-a143-288adc0e078d) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1669.json) |
|[Элемент управления 1670, контролируемый корпорацией Майкрософт, — исправление ошибок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc6108469-57ee-4666-af7e-79ba61c7ae0c) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1670.json) |
|[Элемент управления 1671, контролируемый корпорацией Майкрософт, — исправление ошибок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c5bbef7-a316-415b-9b38-29753ce8e698) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1671.json) |
|[В масштабируемых наборах виртуальных машин должны быть установлены обновления системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc3f317a7-a95c-4547-b7e7-11017ebdf2fe) |Аудит отсутствия обновлений для системы безопасности и критических обновлений, которые необходимо установить для защиты ваших масштабируемых наборов виртуальных машин с Windows и Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssMissingSystemUpdates_Audit.json) |
|[На компьютерах должны быть установлены обновления системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86b3d65f-7626-441e-b690-81a8b71cff60) |Отсутствие обновлений системы безопасности на серверах будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingSystemUpdates_Audit.json) |
|[Уязвимости конфигурации безопасности на ваших компьютерах должны быть устранены](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1e5fd5d-3e4c-4ce1-8661-7d1873ae6b15) |Серверы, не соответствующие заданным базовым показателям, будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OSVulnerabilities_Audit.json) |
|[Уязвимости в конфигурации безопасности в масштабируемом наборе виртуальных машин должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3c735d8a-a4ba-4a3a-b7cf-db7754cf57f4) |Аудит наличия уязвимостей ОС в ваших масштабируемых наборах виртуальных машин для защиты их от атак. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssOSVulnerabilities_Audit.json) |
|[Уязвимости в базах данных SQL должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffeedbf84-6b99-488c-acc2-71c829aa5ffc) |Мониторинг результатов оценки уязвимостей и рекомендации по исправлению уязвимостей баз данных. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_SQLDbVulnerabilities_Audit.json) |

### <a name="flaw-remediation--central-management"></a>Исправление ошибок | Централизованное управление

**Идентификатор**. NIST SP 800-53 R4 SI-2 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1672, контролируемый корпорацией Майкрософт, — исправление ошибок \| Централизованное управление](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb45fe972-904e-45a4-ac20-673ba027a301) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1672.json) |

### <a name="flaw-remediation--automated-flaw-remediation-status"></a>Исправление ошибок | Автоматическое определение состояния исправления ошибок

**Идентификатор**. NIST SP 800-53 R4 SI-2 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1673, контролируемый корпорацией Майкрософт, — исправление ошибок \| Автоматическое определение состояния исправления ошибок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdff0b90d-5a6f-491c-b2f8-b90aa402d844) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1673.json) |

### <a name="flaw-remediation--time-to-remediate-flaws--benchmarks-for-corrective-actions"></a>Исправление ошибок | Время устранения ошибок и измерение производительности корректирующих действий

**Идентификатор**. NIST SP 800-53 R4 SI-2 (3) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1674, контролируемый корпорацией Майкрософт, — исправление ошибок \| Время устранения ошибок и измерение производительности корректирующих действий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F93e9e233-dd0a-4bde-aea5-1371bce0e002) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1674.json) |
|[Элемент управления 1675, контролируемый корпорацией Майкрософт, — исправление ошибок \| Время устранения ошибок и измерение производительности корректирующих действий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffacb66e0-1c48-478a-bed5-747a312323e1) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1675.json) |

### <a name="malicious-code-protection"></a>Защита от вредоносного кода

**Идентификатор**. NIST SP 800-53 R4 SI-3 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В масштабируемых наборах виртуальных машин должно быть установлено решение Endpoint Protection](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F26a828e1-e88f-464e-bbb3-c134a282b9de) |Аудит наличия и работоспособности решения защиты конечных точек в ваших масштабируемых наборах виртуальных машин для защиты их от угроз и уязвимостей. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssMissingEndpointProtection_Audit.json) |
|[Элемент управления 1676, контролируемый корпорацией Майкрософт, — защита от вредоносного кода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc10fb58b-56a8-489e-9ce3-7ffe24e78e4b) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1676.json) |
|[Элемент управления 1677, контролируемый корпорацией Майкрософт, — защита от вредоносного кода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4a248e1e-040f-43e5-bff2-afc3a57a3923) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1677.json) |
|[Элемент управления 1678, контролируемый корпорацией Майкрософт, — защита от вредоносного кода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdd533cb0-b416-4be7-8e86-4d154824dfd7) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1678.json) |
|[Элемент управления 1679, контролируемый корпорацией Майкрософт, — защита от вредоносного кода](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2cf42a28-193e-41c5-98df-7688e7ef0a88) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1679.json) |
|[Мониторинг отсутствия Endpoint Protection в Центре безопасности Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Серверы без установленного агента Endpoint Protection будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |

### <a name="malicious-code-protection--central-management"></a>Защита от вредоносного кода | Централизованное управление

**Идентификатор**. NIST SP 800-53 R4 SI-3 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В масштабируемых наборах виртуальных машин должно быть установлено решение Endpoint Protection](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F26a828e1-e88f-464e-bbb3-c134a282b9de) |Аудит наличия и работоспособности решения защиты конечных точек в ваших масштабируемых наборах виртуальных машин для защиты их от угроз и уязвимостей. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssMissingEndpointProtection_Audit.json) |
|[Элемент управления 1680, контролируемый корпорацией Майкрософт, — защита от вредоносного кода \| Централизованное управление](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F399cd6ee-0e18-41db-9dea-cde3bd712f38) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1680.json) |
|[Мониторинг отсутствия Endpoint Protection в Центре безопасности Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Серверы без установленного агента Endpoint Protection будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |

### <a name="malicious-code-protection--automatic-updates"></a>Защита от вредоносного кода | Автоматическое обновление

**Идентификатор**. NIST SP 800-53 R4 SI-3 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1681, контролируемый корпорацией Майкрософт, — защита от вредоносного кода \| Автоматические обновления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F12623e7e-4736-4b2e-b776-c1600f35f93a) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1681.json) |

### <a name="malicious-code-protection--nonsignature-based-detection"></a>Защита от вредоносного кода | Обнаружение без использования сигнатур

**Идентификатор**. NIST SP 800-53 R4 SI-3 (7) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1682, контролируемый корпорацией Майкрософт, — защита от вредоносного кода \| Обнаружение без использования сигнатур](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F62b638c5-29d7-404b-8d93-f21e4b1ce198) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1682.json) |

### <a name="information-system-monitoring"></a>Мониторинг информационной системы

**Идентификатор**. NIST SP 800-53 R4 SI-4 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемом экземпляре SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Проверка всех Управляемых экземпляров SQL без Расширенной защиты данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[На серверах SQL должна быть включена Расширенная защита данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Аудит серверов SQL без Расширенной защиты данных |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |
|[Проверка рабочей области Log Analytics для виртуальной машины — сообщение о несоответствии](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff47b5582-33ec-4c5c-87c0-b010a6b2e917) |Если для виртуальных машин не ведется журнал в рабочей области Log Analytics, указанной в назначении политики или инициативы, поступает сообщение о несоответствии виртуальных машин. |аудит |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_WorkspaceMismatch_VM_Audit.json) |
|[Агент Log Analytics должен быть включен для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F32133ab0-ee4b-4b44-98d6-042180979d50) |Помечает виртуальные машины как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_Audit.json) |
|[Агент Log Analytics должен быть включен в масштабируемых наборах виртуальных машин для перечисленных образов виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c3bc7b8-a64c-4e08-a9cd-7ff0f31e1138) |Помечает масштабируемые наборы виртуальных машин как несовместимые, если образ виртуальной машины не находится в заданном списке, а агент не установлен. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/LogAnalytics_OSImage_VMSS_Audit.json) |
|[Элемент управления 1683, контролируемый корпорацией Майкрософт, — мониторинг информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8c79fee4-88dd-44ce-bbd4-4de88948c4f8) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1683.json) |
|[Элемент управления 1684, контролируемый корпорацией Майкрософт, — мониторинг информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F16bfdb59-db38-47a5-88a9-2e9371a638cf) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1684.json) |
|[Элемент управления 1685, контролируемый корпорацией Майкрософт, — мониторинг информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F36b0ef30-366f-4b1b-8652-a3511df11f53) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1685.json) |
|[Элемент управления 1686, контролируемый корпорацией Майкрософт, — мониторинг информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe17085c5-0be8-4423-b39b-a52d3d1402e5) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1686.json) |
|[Элемент управления 1687, контролируемый корпорацией Майкрософт, — мониторинг информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7a87fc7f-301e-49f3-ba2a-4d74f424fa97) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1687.json) |
|[Элемент управления 1688, контролируемый корпорацией Майкрософт, — мониторинг информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F063c3f09-e0f0-4587-8fd5-f4276fae675f) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1688.json) |
|[Элемент управления 1689, контролируемый корпорацией Майкрософт, — мониторинг информационной системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fde901f2f-a01a-4456-97f0-33cda7966172) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1689.json) |

### <a name="information-system-monitoring--system-wide-intrusion-detection-system"></a>Мониторинг информационной системы | Общая система обнаружения атак

**Идентификатор**. NIST SP 800-53 R4 SI-4 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1690, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Общая система обнаружения атак](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa2567a23-d1c3-4783-99f3-d471302a4d6b) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1690.json) |

### <a name="information-system-monitoring--automated-tools-for-real-time-analysis"></a>Мониторинг информационной системы | Автоматизированные инструменты для анализа в режиме реального времени

**Идентификатор**. NIST SP 800-53 R4 SI-4 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1691, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Автоматизированные инструменты для анализа в режиме реального времени](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F71475fb4-49bd-450b-a1a5-f63894c24725) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1691.json) |

### <a name="information-system-monitoring--inbound-and-outbound-communications-traffic"></a>Мониторинг информационной системы | Входящий и исходящий трафик

**Идентификатор**. NIST SP 800-53 R4 SI-4 (4) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1692, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Входящий и исходящий трафик](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7ecda928-9df4-4dd7-8f44-641a91e470e8) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1692.json) |

### <a name="information-system-monitoring--system-generated-alerts"></a>Мониторинг информационной системы | Системные оповещения

**Идентификатор**. NIST SP 800-53 R4 SI-4 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1693, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Системные оповещения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa450eba6-2efc-4a00-846a-5804a93c6b77) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1693.json) |

### <a name="information-system-monitoring--analyze-communications-traffic-anomalies"></a>Мониторинг информационной системы | Анализ аномалий в сетевом трафике

**Идентификатор**. NIST SP 800-53 R4 SI-4 (11) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1694, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Анализ аномалий в сетевом трафике](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F426c4ac9-ff17-49d0-acd7-a13c157081c0) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1694.json) |

### <a name="information-system-monitoring--wireless-intrusion-detection"></a>Мониторинг информационной системы | Обнаружения атак в беспроводных сетях

**Идентификатор**. NIST SP 800-53 R4 SI-4 (14) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1695, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Обнаружение вторжений в беспроводных сетях](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F13fcf812-ec82-4eda-9b89-498de9efd620) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1695.json) |

### <a name="information-system-monitoring--correlate-monitoring-information"></a>Мониторинг информационной системы | Сопоставление данных мониторинга

**Идентификатор**. NIST SP 800-53 R4 SI-4 (16) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1696, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Сопоставление данных мониторинга](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69d2a238-20ab-4206-a6dc-f302bf88b1b8) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1696.json) |

### <a name="information-system-monitoring--analyze-traffic--covert-exfiltration"></a>Мониторинг информационной системы | Анализ трафика и скрытая утечка

**Идентификатор**. NIST SP 800-53 R4 SI-4 (18) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1697, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Анализ трафика и скрытая утечка](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9873db2-18ad-46b3-a11a-1a1f8cbf0335) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1697.json) |

### <a name="information-system-monitoring--individuals-posing-greater-risk"></a>Мониторинг информационной системы | Лица, представляющие больший риск

**Идентификатор**. NIST SP 800-53 R4 SI-4 (19) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1698, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Лица, представляющие больший риск](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F31b752c1-05a9-432a-8fce-c39b56550119) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1698.json) |

### <a name="information-system-monitoring--privileged-users"></a>Мониторинг информационной системы | Привилегированные пользователи

**Идентификатор**. NIST SP 800-53 R4 SI-4 (20) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1699, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Привилегированные пользователи](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69c7bee8-bc19-4129-a51e-65a7b39d3e7c) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1699.json) |

### <a name="information-system-monitoring--unauthorized-network-services"></a>Мониторинг информационной системы | Несанкционированные сетевые службы

**Идентификатор**. NIST SP 800-53 R4 SI-4 (22) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1700, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Несанкционированные сетевые службы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7831b4ba-c3f4-4cb1-8c11-ef8d59438cd5) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1700.json) |

### <a name="information-system-monitoring--host-based-devices"></a>Мониторинг информационной системы | Устройства на основе узлов

**Идентификатор**. NIST SP 800-53 R4 SI-4 (23) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1701, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Устройства на основе узлов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff25bc08f-27cb-43b6-9a23-014d00700426) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1701.json) |

### <a name="information-system-monitoring--indicators-of-compromise"></a>Мониторинг информационной системы | Признаки компрометации

**Идентификатор**. NIST SP 800-53 R4 SI-4 (24) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1702, контролируемый корпорацией Майкрософт, — мониторинг информационной системы \| Признаки компрометации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4dfc0855-92c4-4641-b155-a55ddd962362) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1702.json) |

### <a name="security-alerts-advisories-and-directives"></a>Оповещения системы безопасности, рекомендации и директивы

**Идентификатор**. NIST SP 800-53 R4 SI-5 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1703, контролируемый корпорацией Майкрософт, — оповещения системы безопасности, предупреждения и директивы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F804faf7d-b687-40f7-9f74-79e28adf4205) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1703.json) |
|[Элемент управления 1704, контролируемый корпорацией Майкрософт, — оповещения системы безопасности, предупреждения и директивы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2d44b6fa-1134-4ea6-ad4e-9edb68f65429) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1704.json) |
|[Элемент управления 1705, контролируемый корпорацией Майкрософт, — оповещения системы безопасности, предупреждения и директивы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff82e3639-fa2b-4e06-a786-932d8379b972) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1705.json) |
|[Элемент управления 1706, контролируемый корпорацией Майкрософт, — оповещения системы безопасности, предупреждения и директивы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff475ee0e-f560-4c9b-876b-04a77460a404) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1706.json) |

### <a name="security-alerts-advisories-and-directives--automated-alerts-and-advisories"></a>Оповещения системы безопасности, рекомендации и директивы | Автоматически создаваемые оповещения и рекомендации

**Идентификатор**. NIST SP 800-53 R4 SI-5 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1707, контролируемый корпорацией Майкрософт, — оповещения системы безопасности, предупреждения и директивы \| Автоматически создаваемые оповещения и предупреждения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffd4a2ac8-868a-4702-a345-6c896c3361ce) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1707.json) |

### <a name="security-function-verification"></a>Проверка функций безопасности

**Идентификатор**. NIST SP 800-53 R4 SI-6 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1708, контролируемый корпорацией Майкрософт, — проверка функций безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7a1e2c88-13de-4959-8ee7-47e3d74f1f48) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1708.json) |
|[Элемент управления 1709, контролируемый корпорацией Майкрософт, — проверка функций безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F025992d6-7fee-4137-9bbf-2ffc39c0686c) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1709.json) |
|[Элемент управления 1710, контролируемый корпорацией Майкрософт, — проверка функций безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf2a93c8-e6dd-4c94-acdd-4a2eedfc478e) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1710.json) |
|[Элемент управления 1711, контролируемый корпорацией Майкрософт, — проверка функций безопасности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb083a535-a66a-41ec-ba7f-f9498bf67cde) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1711.json) |

### <a name="software-firmware-and-information-integrity"></a>Целостность программного обеспечения, встроенного ПО и данных

**Идентификатор**. NIST SP 800-53 R4 SI-7 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1712, контролируемый корпорацией Майкрософт, — целостность программного обеспечения, встроенного ПО и данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F44e543aa-41db-42aa-98eb-8a5eb1db53f0) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1712.json) |

### <a name="software-firmware-and-information-integrity--integrity-checks"></a>Целостность программного обеспечения, встроенного ПО и данных | Проверки целостности

**Идентификатор**. NIST SP 800-53 R4 SI-7 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1713, контролируемый корпорацией Майкрософт, — целостность программного обеспечения, встроенного ПО и данных \| Проверки целостности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d87c70b-5012-48e9-994b-e70dd4b8def0) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1713.json) |

### <a name="software-firmware-and-information-integrity--automated-notifications-of-integrity-violations"></a>Целостность программного обеспечения, встроенного ПО и данных | Автоматические уведомления о нарушении целостности

**Идентификатор**. NIST SP 800-53 R4 SI-7 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1714, контролируемый корпорацией Майкрософт, — целостность программного обеспечения, встроенного ПО и данных \| Автоматические уведомления о нарушении целостности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe12494fa-b81e-4080-af71-7dbacc2da0ec) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1714.json) |

### <a name="software-firmware-and-information-integrity--automated-response-to-integrity-violations"></a>Целостность программного обеспечения, встроенного ПО и данных | Автоматическое реагирование на нарушения целостности

**Идентификатор**. NIST SP 800-53 R4 SI-7 (5) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1715, контролируемый корпорацией Майкрософт, — целостность программного обеспечения, встроенного ПО и данных \| Автоматическое реагирование на нарушение целостности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdd469ae0-71a8-4adc-aafc-de6949ca3339) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1715.json) |

### <a name="software-firmware-and-information-integrity--integration-of-detection-and-response"></a>Целостность программного обеспечения, встроенного ПО и данных | Интеграция инструментов обнаружения и реагирования

**Идентификатор**. NIST SP 800-53 R4 SI-7 (7) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1716, контролируемый корпорацией Майкрософт, — целостность программного обеспечения, встроенного ПО и данных \| Интеграция инструментов обнаружения и реагирования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe54c325e-42a0-4dcf-b105-046e0f6f590f) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1716.json) |

### <a name="software-firmware-and-information-integrity--binary-or-machine-executable-code"></a>Целостность программного обеспечения, встроенного ПО и данных | Двоичный или машинный исполняемый код

**Идентификатор**. NIST SP 800-53 R4 SI-7 (14) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1717, контролируемый корпорацией Майкрософт, — целостность программного обеспечения, встроенного ПО и данных \| Двоичный или машинный исполняемый код](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F967773fc-d9ab-4a4e-8ff6-f5e9e3f5dbef) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1717.json) |
|[Элемент управления 1718, контролируемый корпорацией Майкрософт, — целостность программного обеспечения, встроенного ПО и данных \| Двоичный или машинный исполняемый код](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0dced7ab-9ce5-4137-93aa-14c13e06ab17) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1718.json) |

### <a name="spam-protection"></a>Защита от спама

**Идентификатор**. NIST SP 800-53 R4 SI-8 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1719, контролируемый корпорацией Майкрософт, — защита от спама](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc13da9b4-fe14-4fe2-853a-5997c9d4215a) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1719.json) |
|[Элемент управления 1720, контролируемый корпорацией Майкрософт, — защита от спама](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F44b9a7cd-f36a-491a-a48b-6d04ae7c4221) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1720.json) |

### <a name="spam-protection--central-management"></a>Защита от спама | Централизованное управление

**Идентификатор**. NIST SP 800-53 R4 SI-8 (1) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1721, контролируемый корпорацией Майкрософт, — защита от спама \| Централизованное управление](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd207aaef-7c4d-4f8c-9dce-4d62dfa3d29a) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1721.json) |

### <a name="spam-protection--automatic-updates"></a>Защита от спама | Автоматическое обновление

**Идентификатор**. NIST SP 800-53 R4 SI-8 (2) **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1722, контролируемый корпорацией Майкрософт, — защита от спама \| Автоматические обновления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1da06bd-25b6-4127-a301-c313d6873fff) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1722.json) |

### <a name="information-input-validation"></a>Проверка ввода данных

**Идентификатор**. NIST SP 800-53 R4 SI-10 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1723, контролируемый корпорацией Майкрософт, — проверка ввода данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe91927a0-ac1d-44a0-95f8-5185f9dfce9f) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1723.json) |

### <a name="error-handling"></a>Обработка ошибок

**Идентификатор**. NIST SP 800-53 R4 SI-11 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1724, контролируемый корпорацией Майкрософт, — обработка ошибок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd07594d1-0307-4c08-94db-5d71ff31f0f6) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1724.json) |
|[Элемент управления 1725, контролируемый корпорацией Майкрософт, — обработка ошибок](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fafc234b5-456b-4aa5-b3e2-ce89108124cc) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1725.json) |

### <a name="information-handling-and-retention"></a>Обработка и хранение данных

**Идентификатор**. NIST SP 800-53 R4 SI-12 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1726, контролируемый корпорацией Майкрософт, — обработка и хранение информации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbaff1279-05e0-4463-9a70-8ba5de4c7aa4) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1726.json) |

### <a name="memory-protection"></a>Защита памяти

**Идентификатор**. NIST SP 800-53 R4 SI-16 **Ответственность**: Совмещаемая блокировка

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Элемент управления 1727, контролируемый корпорацией Майкрософт, — защита памяти](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F697175a7-9715-4e89-b98b-c6f605888fa3) |Корпорация Майкрософт реализует этот элемент управления целостностью системы и данных |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Regulatory%20Compliance/MicrosoftManagedControl1727.json) |

> [!NOTE]
> Возможность использования отдельных определений Политики Azure может отличаться в Azure для государственных организаций и других национальных облаках.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные статьи о Политике Azure:

- Обзор [соответствия нормативным требованиям](../concepts/regulatory-compliance.md).
- См. [структуру определения инициативы](../concepts/initiative-definition-structure.md).
- См. другие [примеры шаблонов для Политики Azure](./index.md).
- Изучите [сведения о действии политик](../concepts/effects.md).
- Узнайте, как [исправлять несоответствующие ресурсы](../how-to/remediate-resources.md).

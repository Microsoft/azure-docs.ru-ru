---
title: Сведения о соответствии требованиям стандарта HIPAA HITRUST 9.2
description: Сведения о встроенной инициативе по соответствию требованиям стандарта HIPAA HITRUST 9.2. Каждый элемент управления сопоставляется с одним или несколькими определениями Политики Azure, которые помогают выполнять оценку.
ms.date: 04/14/2021
ms.topic: sample
ms.custom: generated
ms.openlocfilehash: ad5700090eda9fad4c607c1f24503f1483cd36fe
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107506250"
---
# <a name="details-of-the-hipaa-hitrust-92-regulatory-compliance-built-in-initiative"></a>Сведения о встроенной инициативе по соответствию требованиям стандарта HIPAA HITRUST 9.2

В следующей статье подробно описано, как определение встроенной инициативы Политики Azure по соответствию нормативным требованиям сопоставляется с **областями соответствия нормативным требованиям** и **элементами управления** в стандарте HIPAA HITRUST 9.2.
Дополнительные сведения об этом стандарте соответствия см. на странице описания [HIPAA HITRUST 9.2](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html). Сведения о термине _Ответственность_ см. в статьях [Определение службы "Политика Azure"](../concepts/definition-structure.md#type) и [Разделение ответственности в облаке](../../../security/fundamentals/shared-responsibility.md).

Далее представлены сопоставления для элементов управления **HIPAA HITRUST 9.2**. Используйте панель навигации справа для перехода непосредственно к конкретной **области соответствия нормативным требованиям**. Многие элементы управления реализуются с помощью определения инициативы [Политики Azure](../overview.md). Чтобы просмотреть полное определение инициативы, откройте раздел **Политика** на портале Azure и перейдите на страницу **Определения**.
Затем найдите и выберите встроенное определение инициативы **HITRUST/HIPAA** по обеспечению соответствия нормативным требованиям.

Эта встроенная инициатива развертывается в составе [примера схемы HITRUST/HIPAA 9.2](../../blueprints/samples/hipaa-hitrust-9-2.md).

> [!IMPORTANT]
> Каждый элемент управления ниже связан с одним или несколькими определениями [Политики Azure](../overview.md).
> Такие политики помогут вам в [оценке соответствия](../how-to/get-compliance-data.md) с помощью элементов управления, но часто полное или точное соответствие между элементом управления и одной или несколькими политиками отсутствует. Поэтому состояние **Совместимый** в Политике Azure применимо только к самим определениям политики и не означает полное соответствие всем требованиям элемента управления. Кроме того, стандарт соответствия включает элементы управления, на которые сейчас не распространяются определения Политики Azure. Следовательно, сведения о соответствии в Политике Azure — это только частичное представление общего состояния соответствия. Связи между областями соответствия нормативным требованиям, элементами управления и определениями Политики Azure для этого стандарта соответствия со временем могут меняться. Историю изменений можно просмотреть на странице [журнала фиксаций в GitHub](https://github.com/Azure/azure-policy/commits/master/built-in-policies/policySetDefinitions/Regulatory%20Compliance/HIPAA_HITRUST_audit.json).

## <a name="privilege-management"></a>Управление правами

### <a name="access-to-management-functions-or-administrative-consoles-for-systems-hosting-virtualized-systems-are-restricted-to-personnel-based-upon-the-principle-of-least-privilege-and-supported-through-technical-controls"></a>Доступ к функциям управления или консолям администрирования систем, на которых размещаются виртуализированные системы, предоставляется только сотрудникам согласно принципу наименьших привилегий с применением технических элементов управления.

**Идентификатор**. 11180.01c3System.6 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |

### <a name="privileges-are-formally-authorized-and-controlled-allocated-to-users-on-a-need-to-use-and-event-by-event-basis-for-their-functional-role-eg-user-or-administrator-and-documented-for-each-system-productelement"></a>Привилегии предоставляются и контролируются официально, выделяются пользователям по мере необходимости с учетом их функциональной роли (например, пользователь или администратор) для конкретного события и документируются для каждого системного продукта или элемента.

**Идентификатор**. 1143.01c1System.123 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления на виртуальных машинах должны быть закрыты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22730e10-96f6-4aac-ad84-9383d35b5917) |Открытые порты удаленного управления создают для вашей виртуальной машины высокий уровень риска атак из Интернета. Эти атаки используют метод подбора учетных данных для получения доступа к компьютеру от имени администратора. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OpenManagementPortsOnVirtualMachines_Audit.json) |

### <a name="the-organization-explicitly-authorizes-access-to-specific-security-relevant-functions-deployed-in-hardware-software-and-firmware-and-security-relevant-information"></a>Организация явно разрешает доступ к определенным функциям, связанным с безопасностью (развернутым в оборудовании, программном обеспечении и встроенном ПО), а также связанным с безопасностью сведениям.

**Идентификатор**. 1144.01c1System.4 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено не более 3 владельцев](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f11b553-d42e-4e3a-89be-32ca364cad4c) |Мы рекомендуем назначать подписке не более трех владельцев, чтобы снизить вероятность нарушения безопасности при компрометации владельца. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateLessThanXOwners_Audit.json) |

### <a name="role-based-access-control-is-implemented-and-capable-of-mapping-each-user-to-one-or-more-roles-and-each-role-to-one-or-more-system-functions"></a>Реализовано управление доступом на основе ролей, позволяющее сопоставлять каждого пользователя с одной или несколькими ролями, а каждую роль — с одной или несколькими системными функциями.

**Идентификатор**. 1145.01c2System.1 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено более одного владельца](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09024ccc-0c5f-475e-9457-b7c0d9ed487b) |Мы рекомендуем назначить более одного владельца подписки, чтобы обеспечить избыточность для административного доступа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateMoreThanOneOwner_Audit.json) |

### <a name="the-organization-promotes-the-development-and-use-of-programs-that-avoid-the-need-to-run-with-elevated-privileges-and-system-routines-to-avoid-the-need-to-grant-privileges-to-users"></a>Организация поощряет разработку и использование программ, в которых устранена необходимость запуска с повышенными привилегиями и использования системных подпрограмм, что избавляет от необходимости предоставлять пользователям привилегии.

**Идентификатор**. 1146.01c2System.23 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Внешние учетные записи с разрешениями владельца должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8456c1c-aa66-4dfb-861a-25d127b775c9) |Внешние учетные записи с разрешениями владельца должны быть удалены из подписки, чтобы предотвратить неотслеживаемый доступ. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWithOwnerPermissions_Audit.json) |

### <a name="elevated-privileges-are-assigned-to-a-different-user-id-from-those-used-for-normal-business-use-all-users-access-privileged-services-in-a-single-role-and-such-privileged-access-is-minimized"></a>Повышенные привилегии назначаются другому пользователю, идентификатор которого отличается от используемого для обычных бизнес-целей. Доступ всех пользователей к привилегированным службам осуществляется в одной роли. При этом такой привилегированный доступ сведен к минимуму.

**Идентификатор**. 1147.01c2System.456 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Устаревшие учетные записи с разрешениями владельца должны быть удалены из подписки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Febb62a0c-3560-49e1-89ed-27e074e9f8ad) |Устаревшие учетные записи с разрешениями владельца следует удалять из подписки.  Устаревшими считаются те учетные записи, для которых заблокирована возможность входа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveDeprecatedAccountsWithOwnerPermissions_Audit.json) |

### <a name="the-organization-restricts-access-to-privileged-functions-and-all-security-relevant-information"></a>Организация ограничивает доступ к привилегированным функциям и всем сведениям, относящимся к безопасности.

**Идентификатор**. 1148.01c2System.78 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит использования настраиваемых правил RBAC](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |Аудит встроенных ролей, таких как "Владелец", "Участник", "Читатель", вместо настраиваемых ролей RBAC, использование которых сопряжено с ошибками. Работа с пользовательскими ролями рассматривается как исключение и требует строгой проверки с моделированием угроз. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |
|[Компьютеры с Windows должны соответствовать требованиям для категории "Параметры безопасности — Учетные записи"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fee984370-154a-4ee8-9726-19d900e56fc0) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Параметры безопасности — учетные записи" для ограничения использования пустых паролей и состояния гостевой учетной записи в локальных учетных записях. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SecurityOptionsAccounts_AINE.json) |

### <a name="the-organization-facilitates-information-sharing-by-enabling-authorized-users-to-determine-a-business-partners-access-when-discretion-is-allowed-as-defined-by-the-organization-and-by-employing-manual-processes-or-automated-mechanisms-to-assist-users-in-making-information-sharingcollaboration-decisions"></a>Организация упрощает обмен информацией, позволяя полномочным пользователям определять права доступа бизнес-партнера, если такая свобода действия допускается организацией, и применять процессы, выполняемые вручную, или автоматические механизмы, помогающие пользователям в принятии решений по обмену информацией и совместной работе.

**Идентификатор**. 1149.01c2System.9 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В службах Kubernetes следует использовать управление доступом на основе ролей (RBAC)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |Чтобы обеспечить детальную фильтрацию действий, которые могут выполнять пользователи, используйте управление доступом на основе ролей (RBAC) для управления разрешениями в кластерах службы Kubernetes и настройте соответствующие политики авторизации. |Audit, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

### <a name="the-access-control-system-for-the-system-components-storing-processing-or-transmitting-covered-information-is-set-with-a-default-quotdeny-allquot-setting"></a>Система управления доступом для системных компонентов, хранящая, обрабатывающая или передающая соответствующие сведения, задается параметром по умолчанию &quot;Запрещено все&quot;.

**Идентификатор**. 1150.01c2System.10 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления на виртуальных машинах должны быть закрыты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22730e10-96f6-4aac-ad84-9383d35b5917) |Открытые порты удаленного управления создают для вашей виртуальной машины высокий уровень риска атак из Интернета. Эти атаки используют метод подбора учетных данных для получения доступа к компьютеру от имени администратора. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OpenManagementPortsOnVirtualMachines_Audit.json) |

### <a name="the-organization-limits-authorization-to-privileged-accounts-on-information-systems-to-a-pre-defined-subset-of-users"></a>Организация ограничивает авторизацию в привилегированных учетных записях в информационных системах предопределенным подмножеством пользователей.

**Идентификатор**. 1151.01c3System.1 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено не более 3 владельцев](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f11b553-d42e-4e3a-89be-32ca364cad4c) |Мы рекомендуем назначать подписке не более трех владельцев, чтобы снизить вероятность нарушения безопасности при компрометации владельца. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateLessThanXOwners_Audit.json) |

### <a name="the-organization-audits-the-execution-of-privileged-functions-on-information-systems-and-ensures-information-systems-prevent-non-privileged-users-from-executing-privileged-functions"></a>Организация проводит аудит выполнения привилегированных функций в информационных системах и следит за тем, чтобы непривилегированные пользователи не могли выполнять привилегированные функции.

**Идентификатор**. 1152.01c3System.2 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено более одного владельца](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09024ccc-0c5f-475e-9457-b7c0d9ed487b) |Мы рекомендуем назначить более одного владельца подписки, чтобы обеспечить избыточность для административного доступа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateMoreThanOneOwner_Audit.json) |

### <a name="all-file-system-access-not-explicitly-required-is-disabled-and-only-authorized-users-are-permitted-access-to-only-that-which-is-expressly-required-for-the-performance-of-the-users-job-duties"></a>Отключены все права доступа к файловой системе, кроме требуемых явно, и лишь полномочные пользователи получают доступ к тем ресурсам, которые явно требуются для выполнения их рабочих обязанностей.

**Идентификатор**. 1153.01c3System.35 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В службах Kubernetes следует использовать управление доступом на основе ролей (RBAC)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |Чтобы обеспечить детальную фильтрацию действий, которые могут выполнять пользователи, используйте управление доступом на основе ролей (RBAC) для управления разрешениями в кластерах службы Kubernetes и настройте соответствующие политики авторизации. |Audit, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

### <a name="contractors-are-provided-with-minimal-system-and-physical-access-only-after-the-organization-assesses-the-contractors-ability-to-comply-with-its-security-requirements-and-the-contractor-agrees-to-comply"></a>Системный и физический доступ предоставляется подрядчикам в минимальном объеме и только после того, как организация оценит их способность соблюдать требования к безопасности, а сами подрядчики согласятся их соблюдать.

**Идентификатор**. 1154.01c3System.4 — 01.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено не более 3 владельцев](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f11b553-d42e-4e3a-89be-32ca364cad4c) |Мы рекомендуем назначать подписке не более трех владельцев, чтобы снизить вероятность нарушения безопасности при компрометации владельца. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateLessThanXOwners_Audit.json) |

## <a name="user-authentication-for-external-connections"></a>Проверка подлинности пользователей для внешних подключений

### <a name="strong-authentication-methods-such-as-multi-factor-radius-or-kerberos-for-privileged-access-and-chap-for-encryption-of-credentials-for-dialup-methods-are-implemented-for-all-external-connections-to-the-organizations-network"></a>Используются надежные методы проверки подлинности, такие как многофакторная аутентификация, RADIUS или Kerberos (для привилегированного доступа) и CHAP (для шифрования учетных данных при коммутируемом доступе), для всех внешних подключений к сети Организации.

**Идентификатор**. 1116.01j1Organizational.145 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями владельца в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |Следует включить MFA для всех учетных записей подписки с разрешениями владельца, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |

### <a name="remote-access-by-vendors-and-business-partners-eg-for-remote-maintenance-is-disableddeactivated-when-not-in-use"></a>Если удаленный доступ не используется поставщиками и бизнес-партнерами (например, для удаленного обслуживания), он отключается или деактивируется.

**Идентификатор**. 1117.01j1Organizational.23 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на запись в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |Следует включить MFA для всех учетных записей подписки с разрешениями на запись, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |

### <a name="organizations-implement-encryption-eg-nbspvpn-solutions-or-private-lines-and-logs-remote-access-to-the-organizations-network-by-employees-contractors-or-third-party-eg-vendors"></a>Организации реализуют шифрование (например, &nbsp;VPN-решения или частные линии) и регистрируют удаленный доступ к сети организации со стороны сотрудников, подрядчиков или сторонних организаций (например, поставщиков).

**Идентификатор**. 1118.01j2Organizational.124 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на чтение в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |Следует включить многофакторную проверку подлинности (MFA) для всех учетных записей подписки с разрешениями на чтение, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |

### <a name="network-equipment-is-checked-for-unanticipated-dial-up-capabilities"></a>Сетевое оборудование проверяется на наличие непредвиденных возможностей коммутируемого подключения.

**Идентификатор**. 1119.01j2Organizational.3 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |

### <a name="remote-administration-sessions-are-authorized-encrypted-and-employ-increased-security-measures"></a>Для сеансов удаленного администрирования применяется авторизация, шифрование и повышенные меры безопасности.

**Идентификатор**. 1121.01j3Organizational.2 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями владельца в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |Следует включить MFA для всех учетных записей подписки с разрешениями владельца, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |

### <a name="if-encryption-is-not-used-for-dial-up-connections-the-cio-or-hisher-designated-representative-provides-specific-written-authorization"></a>Если для коммутируемых подключений не используется шифрование, директор по информационной безопасности или назначенный им представитель выдает явное письменное разрешение.

**Идентификатор**. 1173.01j1Organizational.6 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на запись в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |Следует включить MFA для всех учетных записей подписки с разрешениями на запись, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |

### <a name="the-organization-protects-wireless-access-to-systems-containing-sensitive-information-by-authenticating-both-users-and-devices"></a>Организация защищает беспроводный доступ к системам, содержащим конфиденциальные данные, проверяя подлинность как пользователей, так и устройств.

**Идентификатор**. 1174.01j1Organizational.7 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на чтение в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |Следует включить многофакторную проверку подлинности (MFA) для всех учетных записей подписки с разрешениями на чтение, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |

### <a name="remote-access-to-business-information-across-public-networks-only-takes-place-after-successful-identification-and-authentication"></a>Удаленный доступ к бизнес-информации в общедоступных сетях предоставляется только после успешной идентификации и проверки подлинности.

**Идентификатор**. 1175.01j1Organizational.8 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |

### <a name="the-organization-requires-a-callback-capability-with-re-authentication-to-verify-dial-up-connections-from-authorized-locations"></a>Организация требует наличия функции обратного вызова с повторной проверкой подлинности для проверки коммутируемых подключений из авторизованных расположений.

**Идентификатор**. 1176.01j2Organizational.5 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями владельца в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |Следует включить MFA для всех учетных записей подписки с разрешениями владельца, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |

### <a name="user-ids-assigned-to-vendors-are-reviewed-in-accordance-with-the-organizations-access-review-policy-at-a-minimum-annually"></a>Назначенные поставщикам идентификаторы пользователей проверяются не реже чем ежегодно в соответствии с политикой проверки доступа организации.

**Идентификатор**. 1177.01j2Organizational.6 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на запись в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |Следует включить MFA для всех учетных записей подписки с разрешениями на запись, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |

### <a name="node-authentication-including-cryptographic-techniques-eg-machine-certificates-serves-as-an-alternative-means-of-authenticating-groups-of-remote-users-where-they-are-connected-to-a-secure-shared-computer-facility"></a>Проверка подлинности узла, в том числе с использованием методов шифрования (например, сертификатов компьютеров), служит альтернативным средством проверки подлинности групп удаленных пользователей при подключении к защищенному общему компьютерному оборудованию.

**Идентификатор**. 1178.01j2Organizational.7 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на чтение в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |Следует включить многофакторную проверку подлинности (MFA) для всех учетных записей подписки с разрешениями на чтение, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |

### <a name="the-information-system-monitors-and-controls-remote-access-methods"></a>Информационная система отслеживает методы удаленного доступа и управляет ими.

**Идентификатор**. 1179.01j3Organizational.1 — 01.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |

## <a name="remote-diagnostic-and-configuration-port-protection"></a>Удаленная диагностика и защита портов конфигурации

### <a name="access-to-network-equipment-is-physically-protected"></a>Доступ к сетевому оборудованию физически защищен.

**Идентификатор**. 1192.01l1Organizational.1 — 01.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |

### <a name="controls-for-the-access-to-diagnostic-and-configuration-ports-include-the-use-of-a-key-lock-and-the-implementation-of-supporting-procedures-to-control-physical-access-to-the-port"></a>Элементы управления доступом к портам диагностики и конфигурации включают использование блокировки ключа и реализацию вспомогательных процедур для управления физическим доступом к порту.

**Идентификатор**. 1193.01l2Organizational.13 — 01.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Порты управления на виртуальных машинах должны быть закрыты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22730e10-96f6-4aac-ad84-9383d35b5917) |Открытые порты удаленного управления создают для вашей виртуальной машины высокий уровень риска атак из Интернета. Эти атаки используют метод подбора учетных данных для получения доступа к компьютеру от имени администратора. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OpenManagementPortsOnVirtualMachines_Audit.json) |

### <a name="ports-services-and-similar-applications-installed-on-a-computer-or-network-systems-which-are-not-specifically-required-for-business-functionality-are-disabled-or-removed"></a>Порты, службы и аналогичные приложения, установленные на компьютере или в сетевых системах, которые не требуются явным образом для выполнения бизнес-функций, отключены или удалены.

**Идентификатор**. 1194.01l2Organizational.2 — 01.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Удаленная отладка должна быть отключена для веб-приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcb510bfd-1cba-4d9f-a230-cb0976f4bb71) |Для удаленной отладки нужно, чтобы в веб-приложении были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_WebApp_Audit.json) |

### <a name="the-organization-reviews-the-information-system-within-every-three-hundred-and-sixty--five-365-days-to-identify-and-disables-unnecessary-and-non-secure-functions-ports-protocols-andor-services"></a>Организация проверяет информационную систему каждые 365 (триста шестьдесят пять) дней с целью выявления и отключения ненужных и небезопасных функций, портов, протоколов и (или) служб.

**Идентификатор**. 1195.01l3Organizational.1 — 01.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Удаленная отладка должна быть отключена для приложений-функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e60b895-3786-45da-8377-9c6b4b6ac5f9) |Для удаленной отладки нужно, чтобы в приложениях-функциях были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_FunctionApp_Audit.json) |

### <a name="the-organization-identifies-unauthorized-blacklisted-software-on-the-information-system-prevents-program-execution-in-accordance-with-a-list-of-unauthorized-blacklisted-software-programs-employs-an-allow-all-deny-by-exception-policy-to-prohibit-execution-of-known-unauthorized-blacklisted-software-and-reviews-and-updates-the-list-of-unauthorized-blacklisted-software-programs-annually"></a>Организация определяет неавторизованное программное обеспечение (из черного списка) в информационной системе, запрещает выполнение программ в соответствии со списком неавторизованных программ (из черного списка), применяет политику "Разрешено все, кроме запрещенного исключениями" для блокировки известного несанкционированного ПО (из черного списка), а также проверяет и обновляет список неавторизованных программ (из черного списка) раз в год.

**Идентификатор**. 1196.01l3Organizational.24 — 01.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Удаленная отладка должна быть отключена для приложений API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe9c8d085-d9cc-4b17-9cdc-059f1f01f19e) |Для удаленной отладки нужно, чтобы в приложениях API были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_ApiApp_Audit.json) |

### <a name="the-organization-disables-bluetooth-and-peer-to-peer-networking-protocols-within-the-information-system-determined-to-be-unnecessary-or-non-secure"></a>Организация отключает в информационной системе протоколы Bluetooth и одноранговые сетевые протоколы, которые определены как ненужные или небезопасные.

**Идентификатор**. 1197.01l3Organizational.3 — 01.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На компьютерах должны быть включены адаптивные элементы управления приложениями для определения безопасных приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Включите элементы управления приложениями, чтобы определить список проверенных безопасных приложений, выполняющихся на ваших компьютерах, и настроить получение оповещений о запуске других приложений. Это поможет защитить компьютеры от вредоносных программ. Чтобы упростить процесс настройки и обслуживания правил, Центр безопасности использует машинное обучение для анализа приложений, выполняющихся на каждом компьютере, и предлагает список проверенных безопасных приложений. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |

## <a name="segregation-in-networks"></a>Разделение в сетях

### <a name="the-organizations-security-gateways-eg-firewalls-enforce-security-policies-and-are-configured-to-filter-traffic-between-domains-block-unauthorized-access-and-are-used-to-maintain-segregation-between-internal-wired-internal-wireless-and-external-network-segments-eg-the-internet-including-dmzs-and-enforce-access-control-policies-for-each-of-the-domains"></a>В организации существуют шлюзы безопасности (например, брандмауэры), которые применяют политики безопасности и фильтруют трафик между доменами, блокируют несанкционированный доступ, поддерживают разделение между внутренней проводной, внутренней беспроводной и внешней (Интернет) сетями с организацией сетей периметра, а также применяют политики управления доступом для каждого домена.

**Идентификатор**. 0805.01m1Organizational.12 — 01.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба приложений должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2d21331d-a4c2-4def-a9ad-ee4e1e023beb) |Эта политика выполняет аудит для любой Службы приложений, не настроенной на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_AppService_AuditIfNotExists.json) |
|[Реестр контейнеров должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78) |Эта политика выполняет аудит всех реестров контейнеров, не настроенных для использования конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ContainerRegistry_Audit.json) |
|[Служба Cosmos DB должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe0a2b1a3-f7f9-4569-807f-2a9edebdf4d9) |Эта политика выполняет аудит всех Cosmos DB, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_CosmosDB_Audit.json) |
|[Концентратор событий должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd63edb4a-c612-454d-b47d-191a724fcbf0) |Эта политика выполняет аудит всех концентраторов событий, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_EventHub_AuditIfNotExists.json) |
|[Подсети шлюза не следует настраивать с использованием группы безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F35f9c03a-cc27-418e-9c0c-539ff999d010) |Эта политика запрещает настройку подсети шлюза с помощью группы безопасности сети. Назначение группы безопасности сети для подсети шлюза приведет к тому, что шлюз перестанет работать. |deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroupOnGatewaySubnet_Deny.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[Служба Key Vault должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea4d6841-2173-4317-9747-ff522a45120f) |Эта политика выполняет аудит всех Key Vault, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_KeyVault_Audit.json) |
|[Служебная шина Azure должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |Эта политика выполняет аудит всех служебных шин, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |
|[Служба SQL Server должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fae5d2f14-d830-42b6-9899-df6cfe9c71a3) |Эта политика выполняет аудит всех SQL Server, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_SQLServer_AuditIfNotExists.json) |
|[Учетная запись хранения должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F60d21c4f-21a3-4d94-85f4-b924e6aeeda4) |Эта политика выполняет аудит всех учетных записей хранения, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_StorageAccount_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |

### <a name="the-organizations-network-is-logically-and-physically-segmented-with-a-defined-security-perimeter-and-a-graduated-set-of-controls-including-subnetworks-for-publicly-accessible-system-components-that-are-logically-separated-from-the-internal-network-based-on-organizational-requirements-and-traffic-is-controlled-based-on-functionality-required-and-classification-of-the-datasystems-based-on-a-risk-assessment-and-their-respective-security-requirements"></a>Сеть организаций логически и физически сегментирована. При этом существует определенный периметр безопасности и дифференцированный набор элементов, включая подсети для общедоступных системных компонентов, которые логически отделены от внутренней сети с учетом требований организации. Управление трафиком осуществляется на основе необходимых функций, а также классификации данных и систем на основе оценки рисков и соответствующих требований к безопасности.

**Идентификатор**. 0806.01m2Organizational.12356 — 01.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба приложений должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2d21331d-a4c2-4def-a9ad-ee4e1e023beb) |Эта политика выполняет аудит для любой Службы приложений, не настроенной на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_AppService_AuditIfNotExists.json) |
|[Реестр контейнеров должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78) |Эта политика выполняет аудит всех реестров контейнеров, не настроенных для использования конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ContainerRegistry_Audit.json) |
|[Служба Cosmos DB должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe0a2b1a3-f7f9-4569-807f-2a9edebdf4d9) |Эта политика выполняет аудит всех Cosmos DB, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_CosmosDB_Audit.json) |
|[Концентратор событий должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd63edb4a-c612-454d-b47d-191a724fcbf0) |Эта политика выполняет аудит всех концентраторов событий, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_EventHub_AuditIfNotExists.json) |
|[Подсети шлюза не следует настраивать с использованием группы безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F35f9c03a-cc27-418e-9c0c-539ff999d010) |Эта политика запрещает настройку подсети шлюза с помощью группы безопасности сети. Назначение группы безопасности сети для подсети шлюза приведет к тому, что шлюз перестанет работать. |deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroupOnGatewaySubnet_Deny.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[Служба Key Vault должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea4d6841-2173-4317-9747-ff522a45120f) |Эта политика выполняет аудит всех Key Vault, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_KeyVault_Audit.json) |
|[Служебная шина Azure должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |Эта политика выполняет аудит всех служебных шин, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |
|[Служба SQL Server должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fae5d2f14-d830-42b6-9899-df6cfe9c71a3) |Эта политика выполняет аудит всех SQL Server, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_SQLServer_AuditIfNotExists.json) |
|[Учетная запись хранения должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F60d21c4f-21a3-4d94-85f4-b924e6aeeda4) |Эта политика выполняет аудит всех учетных записей хранения, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_StorageAccount_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |

### <a name="networks-are-segregated-from-production-level-networks-when-migrating-physical-servers-applications-or-data-to-virtualized-servers"></a>Сети изолируются от сетей производственного уровня при переносе физических серверов, приложений или данных на виртуализированные серверы.

**Идентификатор**. 0894.01m2Organizational.7 — 01.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба приложений должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2d21331d-a4c2-4def-a9ad-ee4e1e023beb) |Эта политика выполняет аудит для любой Службы приложений, не настроенной на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_AppService_AuditIfNotExists.json) |
|[Реестр контейнеров должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78) |Эта политика выполняет аудит всех реестров контейнеров, не настроенных для использования конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ContainerRegistry_Audit.json) |
|[Служба Cosmos DB должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe0a2b1a3-f7f9-4569-807f-2a9edebdf4d9) |Эта политика выполняет аудит всех Cosmos DB, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_CosmosDB_Audit.json) |
|[Развертывание наблюдателя за сетями при создании виртуальных сетей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa9b99dd8-06c5-4317-8629-9d86a3c6e7d9) |Эта политика создает ресурс наблюдателя за сетями в регионах с виртуальными сетями. Необходимо обеспечить наличие группы ресурсов с именем networkWatcherRG, которая будет использоваться для развертывания экземпляров наблюдателя за сетями. |DeployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkWatcher_Deploy.json) |
|[Концентратор событий должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd63edb4a-c612-454d-b47d-191a724fcbf0) |Эта политика выполняет аудит всех концентраторов событий, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_EventHub_AuditIfNotExists.json) |
|[Подсети шлюза не следует настраивать с использованием группы безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F35f9c03a-cc27-418e-9c0c-539ff999d010) |Эта политика запрещает настройку подсети шлюза с помощью группы безопасности сети. Назначение группы безопасности сети для подсети шлюза приведет к тому, что шлюз перестанет работать. |deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroupOnGatewaySubnet_Deny.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[Служба Key Vault должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea4d6841-2173-4317-9747-ff522a45120f) |Эта политика выполняет аудит всех Key Vault, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_KeyVault_Audit.json) |
|[Служебная шина Azure должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |Эта политика выполняет аудит всех служебных шин, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |
|[Служба SQL Server должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fae5d2f14-d830-42b6-9899-df6cfe9c71a3) |Эта политика выполняет аудит всех SQL Server, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_SQLServer_AuditIfNotExists.json) |
|[Учетная запись хранения должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F60d21c4f-21a3-4d94-85f4-b924e6aeeda4) |Эта политика выполняет аудит всех учетных записей хранения, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_StorageAccount_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |

## <a name="network-connection-control"></a>Управление сетевым подключением

### <a name="network-traffic-is-controlled-in-accordance-with-the-organizations-access-control-policy-through-firewall-and-other-network-related-restrictions-for-each-network-access-point-or-external-telecommunication-services-managed-interface"></a>Сетевой трафик контролируется в соответствии с корпоративной политикой управления доступом с применением брандмауэра или других сетевых средств ограничения для каждой точки доступа к сети или управляемого интерфейса внешней службы телекоммуникации.

**Идентификатор**. 0809.01n2Organizational.1234 — 01.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[В вашем приложении API необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="transmitted-information-is-secured-and-at-a-minimum-encrypted-over-open-public-networks"></a>Передаваемые данные защищены и, как минимум, зашифрованы через открытые общедоступные сети.

**Идентификатор**. 0810.01n2Organizational.5 — 01.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[В вашем приложении API необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="exceptions-to-the-traffic-flow-policy-are-documented-with-a-supporting-missionbusiness-need-duration-of-the-exception-and-reviewed-at-least-annually-traffic-flow-policy-exceptions-are-removed-when-no-longer-supported-by-an-explicit-missionbusiness-need"></a>Исключения из политики потока трафика задокументированы с указанием обусловившей его критической потребности или бизнес-потребности и длительности исключения. Они пересматриваются по меньшей мере ежегодно. Исключения политики потока трафика удаляются, если они больше не обусловлены явной критической потребностью или бизнес-потребностью.

**Идентификатор**. 0811.01n2Organizational.6 — 01.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[В вашем приложении API необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="remote-devices-establishing-a-non-remote-connection-are-not-allowed-to-communicate-with-external-remote-resources"></a>Удаленным устройствам, устанавливающим неудаленное подключение, не разрешается обмениваться данными с внешними (удаленными) ресурсами.

**Идентификатор**. 0812.01n2Organizational.8 — 01.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[В вашем приложении API необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="the-ability-of-users-to-connect-to-the-internal-network-is-restricted-using-a-deny-by-default-and-allow-by-exception-policy-at-managed-interfaces-according-to-the-access-control-policy-and-the-requirements-of-clinical-and-business-applications"></a>Возможность пользователей подключаться к внутренней сети ограничена с помощью политики "По умолчанию запрещено все, кроме разрешенного исключениями" на управляемых интерфейсах в соответствии с политикой управления доступом и требованиями бизнес-приложений и приложений для медицинской отрасли.

**Идентификатор**. 0814.01n1Organizational.12 — 01.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[В вашем приложении API необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |
|[Подсети должны быть связаны с группой безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Защитите подсеть от потенциальных угроз, ограничив доступ к ней с помощью группы безопасности сети (NSG). Эти группы содержат перечень правил списка контроля доступа (ACL), которые разрешают или запрещают передачу сетевого трафика в подсеть. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |
|[Виртуальные машины должны быть подключены к утвержденной виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd416745a-506c-48b6-8ab1-83cb814bcaa3) |Эта политика используется для аудита любой виртуальной машины, подключенной к виртуальной сети, которая не утверждена. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ApprovedVirtualNetwork_Audit.json) |
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

## <a name="user-identification-and-authentication"></a>Идентификация и проверка подлинности пользователей

### <a name="the-organization-ensures-that-redundant-user-ids-are-not-issued-to-other-users-and-that-all-users-are-uniquely-identified-and-authenticated-for-both-local-and-remote-access-to-information-systems"></a>Организация гарантирует, что избыточные идентификаторы пользователей не выдаются другим пользователям и все пользователи однозначно идентифицируются и проходят проверку подлинности как при локальном, так и при удаленном доступе к информационным системам.

**Идентификатор**. 11109.01q1Organizational.57 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями владельца в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |Следует включить MFA для всех учетных записей подписки с разрешениями владельца, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |

### <a name="non-organizational-users-all-information-system-users-other-than-organizational-users-such-as-patients-customers-contractors-or-foreign-nationals-or-processes-acting-on-behalf-of-non-organizational-users-determined-to-need-access-to-information-residing-on-the-organizations-information-systems-are-uniquely-identified-and-authenticated"></a>Если установлено, что пользователям, не являющимся организациями (к числу которых относятся все пользователи информационной системы, отличные от пользователей-организаций, например пациенты, клиенты, подрядчики или иностранные лица), либо процессам, выполняемым от их имени, необходим доступ к информации, размещенной в информационных системах организации, такие пользователи и процессы однозначно идентифицируются и проходят проверку подлинности.

**Идентификатор**. 11110.01q1Organizational.6 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на запись в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |Следует включить MFA для всех учетных записей подписки с разрешениями на запись, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |

### <a name="when-pki-based-authentication-is-used-the-information-system-validates-certificates-by-constructing-and-verifying-a-certification-path-to-an-accepted-trust-anchor-including-checking-certificate-status-information-enforces-access-to-the-corresponding-private-key-maps-the-identity-to-the-corresponding-account-of-the-individual-or-group-and-implements-a-local-cache-of-revocation-data-to-support-path-discovery-and-validation-in-case-of-an-inability-to-access-revocation-information-via-the-network"></a>При использовании проверки подлинности на основе PKI информационная система проверяет сертификаты, составляя и проверяя путь сертификации, ведущий к принятой привязке доверия. В частности, она проверяет сведения о состоянии сертификата, обеспечивает доступ к соответствующему закрытому ключу, сопоставляет удостоверение с соответствующей учетной записью пользователя или группы, а также реализует локальный кэш данных отзыва для поддержки обнаружения и проверки пути в случае невозможности получить доступ к сведениям об отзыве по сети.

**Идентификатор**. 11111.01q2System.4 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для учетных записей с разрешениями на чтение в вашей подписке должна быть включена многофакторная проверка подлинности](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |Следует включить многофакторную проверку подлинности (MFA) для всех учетных записей подписки с разрешениями на чтение, чтобы предотвратить нарушение безопасности учетных записей или ресурсов. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |

### <a name="the-information-system-employs-replay-resistant-authentication-mechanisms-such-as-nonce-one-time-passwords-or-time-stamps-to-secure-network-access-for-privileged-accounts-and-for-hardware-token-based-authentication-employs-mechanisms-that-satisfy-minimum-token-requirements-discussed-in-nist-sp-800-63-2-electronic-authentication-guideline"></a>Для защиты доступа к сети из привилегированных учетных записей информационная система использует механизмы проверки подлинности с защитой от повторных атак, такие как nonce, одноразовые пароли или отметки времени. Для проверки подлинности на основе маркеров оборудования в ней используются механизмы, удовлетворяющие минимальным требованиям к маркерам, указанным в положении NIST SP 800-63-2 Руководства по электронной аутентификации.

**Идентификатор**. 11112.01q2Organizational.67 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено не более 3 владельцев](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f11b553-d42e-4e3a-89be-32ca364cad4c) |Мы рекомендуем назначать подписке не более трех владельцев, чтобы снизить вероятность нарушения безопасности при компрометации владельца. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateLessThanXOwners_Audit.json) |

### <a name="the-organization-requires-that-electronic-signatures-unique-to-one-individual-cannot-be-reused-by-or-reassigned-to-anyone-else"></a>Организация требует, чтобы уникальные для одного лица электронные подписи не могли использоваться повторно или переназначаться другим пользователям.

**Идентификатор**. 11208.01q1Organizational.8 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Подписке должно быть назначено более одного владельца](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09024ccc-0c5f-475e-9457-b7c0d9ed487b) |Мы рекомендуем назначить более одного владельца подписки, чтобы обеспечить избыточность для административного доступа. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_DesignateMoreThanOneOwner_Audit.json) |

### <a name="electronic-signatures-and-handwritten-signatures-executed-to-electronic-records-shall-be-linked-to-their-respective-electronic-records"></a>Электронные подписи и рукописные подписи, выполненные для электронных записей, должны быть связаны с соответствующими электронными записями.

**Идентификатор**. 11210.01q2Organizational.10 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров Windows с указанными участниками в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69bf4abd-ca1e-4cf6-8b5a-762d42e61d4f) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если локальная группа администраторов содержит одного или нескольких участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToExclude_AINE.json) |

### <a name="signed-electronic-records-shall-contain-information-associated-with-the-signing-in-human-readable-format"></a>Подписанные электронные записи должны содержать связанные с подписыванием сведения в понятном для человека формате.

**Идентификатор**. 11211.01q2Organizational.11 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров с Windows без любого из указанных участников в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F30f71ea1-ac77-4f26-9fc5-2d926bbd4ba7) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если в локальной группе администраторов отсутствует один или несколько участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToInclude_AINE.json) |

### <a name="users-who-performed-privileged-functions-eg-system-administration-use-separate-accounts-when-performing-those-privileged-functions"></a>Пользователи, которые выполняют привилегированные функции (например, системное администрирование), используют отдельные учетные записи при выполнении этих привилегированных функций.

**Идентификатор**. 1123.01q1System.2 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров Windows с дополнительными учетными записями в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3d2a3320-2a72-4c67-ac5f-caa40fbee2b2) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если локальная группа администраторов содержит участников, не указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembers_AINE.json) |

### <a name="multi-factor-authentication-methods-are-used-in-accordance-with-organizational-policy-eg-for-remote-network-access"></a>Методы многофакторной проверки подлинности используются в соответствии с политикой организации (например, для удаленного доступа к сети).

**Идентификатор**. 1125.01q2System.1 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров Windows с указанными участниками в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69bf4abd-ca1e-4cf6-8b5a-762d42e61d4f) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если локальная группа администраторов содержит одного или нескольких участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToExclude_AINE.json) |

### <a name="where-tokens-are-provided-for-multi-factor-authentication-in-person-verification-is-required-prior-to-granting-access"></a>Если для многофакторной проверки подлинности предоставляются токены, то для предоставления доступа требуется проверка личности.

**Идентификатор**. 1127.01q2System.3 — 01.q **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров с Windows без любого из указанных участников в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F30f71ea1-ac77-4f26-9fc5-2d926bbd4ba7) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если в локальной группе администраторов отсутствует один или несколько участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToInclude_AINE.json) |

## <a name="identification-of-risks-related-to-external-parties"></a>Определение рисков, связанных с внешними сторонами

### <a name="access-to-the-organizations-information-and-systems-by-external-parties-is-not-permitted-until-due-diligence-has-been-conducted-the-appropriate-controls-have-been-implemented-and-a-contractagreement-reflecting-the-security-requirements-is-signed-acknowledging-they-understand-and-accept-their-obligations"></a>Внешним сторонам запрещен доступ к сведениям и системам организаций до проведения комплексной экспертизы, реализации соответствующих элементов управления и подписания контракта или соглашения, отражающего требования к безопасности, для подтверждения того, что они понимают и принимают свои обязательства.

**Идентификатор**. 1401.05i1Organizational.1239 — 05.i **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |

### <a name="remote-access-connections-between-the-organization-and-external-parties-are-encrypted"></a>Подключения для обеспечения удаленного доступа между организацией и внешними сторонами шифруются.

**Идентификатор**. 1402.05i1Organizational.45 — 05.i **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |

### <a name="access-granted-to-external-parties-is-limited-to-the-minimum-necessary-and-granted-only-for-the-duration-required"></a>Доступ, предоставленный внешним сторонам, ограничен необходимым минимумом и предоставляется только на необходимый срок.

**Идентификатор**. 1403.05i1Organizational.67 — 05.i **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="due-diligence-of-the-external-party-includes-interviews-document-review-checklists-certification-reviews-eg-hitrustnbspor-other-remote-means"></a>Комплексная экспертиза внешних сторон включает в себя интервью, проверку документов, составление контрольных списков, проверку сертификации (например, HITRUST)&nbsp;или применение других удаленных средств.

**Идентификатор**. 1404.05i2Organizational.1 — 05.i **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |

### <a name="the-identification-of-risks-related-to-external-party-access-takes-into-account-a-minimal-set-of-specifically-defined-issues"></a>При определении рисков, связанных с доступом внешних сторон, учитывается минимальный набор специально определенных проблем.

**Идентификатор**. 1418.05i1Organizational.8 — 05.i **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |

### <a name="the-organization-obtains-satisfactory-assurances-that-reasonable-information-security-exists-across-their-information-supply-chain-by-performing-an-annual-review-which-includes-all-partnersthird-party-providers-upon-which-their-information-supply-chain-depends"></a>Организация получает удовлетворительные гарантии того, что в цепочке поставок информации обеспечен надлежащий уровень информационной безопасности, выполняя годовую проверку, охватывающую всех партнеров и сторонних поставщиков, от которых зависит эта цепочка.

**Идентификатор**. 1450.05i2Organizational.2 — 05.i **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |

### <a name="cloud-service-providers-design-and-implement-controls-to-mitigate-and-contain-data-security-risks-through-proper-separation-of-duties-role-based-access-and-least-privilege-access-for-all-personnel-within-their-supply-chain"></a>Поставщики облачных служб разрабатывают и реализуют элементы управления для устранения и подавления угроз безопасности данных благодаря правильному разделению обязанностей, доступу на основе ролей и доступу с минимальными привилегиями для всех сотрудников в цепочке поставок.

**Идентификатор**. 1451.05iCSPOrganizational.2 — 05.i **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

## <a name="audit-logging"></a>Ведение журнала аудита

### <a name="a-secure-audit-record-is-created-for-all-activities-on-the-system-create-read-update-delete-involving-covered-information"></a>Защищенная запись аудита создается для всех действий в системе (создание, чтение, обновление, удаление), связанных с соответствующей информацией.

**Идентификатор**. 1202.09aa1System.1 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Azure Data Lake Storage должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |
|[В масштабируемых наборах виртуальных машин должны быть установлены обновления системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc3f317a7-a95c-4547-b7e7-11017ebdf2fe) |Аудит отсутствия обновлений для системы безопасности и критических обновлений, которые необходимо установить для защиты ваших масштабируемых наборов виртуальных машин с Windows и Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssMissingSystemUpdates_Audit.json) |

### <a name="audit-records-include-the-unique-user-id-unique-data-subject-id-function-performed-and-datetime-the-event-was-performed"></a>Записи аудита содержат уникальный идентификатор пользователя, уникальный идентификатор субъекта данных, сведения о выполняемой функции, а также дату и время наступления события.

**Идентификатор**. 1203.09aa1System.2 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Logic Apps должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34f95f76-5386-4de7-b824-0d8478470c9d) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_AuditDiagnosticLog_Audit.json) |

### <a name="the-activities-of-privileged-users-administrators-operators-etc-include-the-successfailure-of-the-event-time-the-event-occurred-the-account-involved-the-processes-involved-and-additional-information-about-the-event"></a>Сведения о действиях привилегированных пользователей (администраторов, операторов и т. д.) включают в себя данные об успешном или неуспешном осуществлении события, времени его наступления, используемых процессах и учетной записи, а также дополнительные сведения о событии.

**Идентификатор**. 1204.09aa1System.3 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Центре Интернета вещей должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F383856f8-de7f-44a2-81fc-e5135b5c2aa4) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[3.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Internet%20of%20Things/IoTHub_AuditDiagnosticLog_Audit.json) |

### <a name="logs-of-messages-sent-and-received-are-maintained-including-the-date-time-origin-and-destination-of-the-message-but-not-its-contents"></a>При ведении журналов отправляемых и получаемых сообщений в журналах записываются такие сведения, как дата, время, источник и назначение сообщения, но не его содержимое.

**Идентификатор**. 1205.09aa2System.1 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В учетных записях пакетной службы должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F428256e6-1fac-4f48-a757-df34c2b3336d) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Batch/Batch_AuditDiagnosticLog_Audit.json) |

### <a name="auditing-is-always-available-while-the-system-is-active-and-tracks-key-events-successfailed-data-access-system-security-configuration-changes-privileged-or-utility-use-any-alarms-raised-nbspactivation-and-de-activation-of-protection-systems-eg-av-and-ids-activation-and-deactivation-of-identification-and-authentication-mechanisms-and-creation-and-deletion-of-system-level-objects"></a>Аудит доступен всегда, пока система активна. При этом отслеживаются ключевые события, успешные и неудачные попытки доступа к данным, изменение конфигурации безопасности, использование привилегированных или служебных программ, срабатывание оповещений, &nbsp;активация и деактивация систем защиты (например, антивирусных программ и IDS), активация и деактивация механизмов идентификации и проверки подлинности, а также создание и удаление объектов на уровне системы.

**Идентификатор**. 1206.09aa2System.23 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Масштабируемых наборах виртуальных машин должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7c1b1214-f927-48bf-8882-84f0af6588b1) |Мы рекомендуем включить журналы для воссоздания следов действий и проведения исследования в случае инцидента или компрометации. |AuditIfNotExists, Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/ServiceFabric_and_VMSS_AuditVMSSDiagnostics.json) |

### <a name="audit-records-are-retained-for-90-days-and-older-audit-records-are-archived-for-one-year"></a>Записи аудита хранятся в течение 90 дней, а старые записи аудита архивируются сроком на один год.

**Идентификатор**. 1207.09aa2System.4 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Azure Stream Analytics должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9be5368-9bf5-4b84-9e0a-7850da98bb46) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Stream%20Analytics/StreamAnalytics_AuditDiagnosticLog_Audit.json) |
|[В Центре событий должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F83a214f7-d01a-484b-91a9-ed54470c9a6a) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Event%20Hub/EventHub_AuditDiagnosticLog_Audit.json) |

### <a name="audit-logs-are-maintained-for-management-activities-system-and-application-startupshutdownerrors-file-changes-and-security-policy-changes"></a>В журналы аудита записываются сведения об операциях управления, запуске, завершении работы и ошибках системы и приложений, изменениях файлов и изменениях политик безопасности.

**Идентификатор**. 1208.09aa3System.1 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В службах "Поиск" должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb4330a05-a843-4bc8-bf9a-cacce50c67f4) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Search/Search_AuditDiagnosticLog_Audit.json) |
|[В Служебной шине должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8d36e2f-389b-4ee4-898d-21aeb69a0f45) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Bus/ServiceBus_AuditDiagnosticLog_Audit.json) |

### <a name="the-information-system-generates-audit-records-containing-the-following-detailed-information-i-filename-accessed-ii-program-or-command-used-to-initiate-the-event-and-iii-source-and-destination-addresses"></a>Информационная система создает записи аудита, содержащие следующие подробные сведения: 1) имя файла, к которому был осуществлен доступ; 2) программа или команда, используемые для запуска события; 3) адреса источника и назначения.

**Идентификатор**. 1209.09aa3System.2 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В службах приложений должны быть включены журналы диагностики](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb607c5de-e7d9-4eee-9e5c-83f1bcee4fa0) |Аудит включения журналов диагностики для приложений. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditLoggingMonitoring_Audit.json) |

### <a name="all-disclosures-of-covered-information-within-or-outside-of-the-organization-are-logged-including-type-of-disclosure-datetime-of-the-event-recipient-and-sender"></a>В журнал заносятся данные обо всех случаях раскрытия соответствующей информации в пределах или за пределами организации, в том числе сведения о типе разглашения, дате и времени события, получателе и отправителе.

**Идентификатор**. 1210.09aa3System.3 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит параметров диагностики](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7f89b1eb-583c-429a-8828-af049802c1d9) |Аудит параметров диагностики для выбранных типов ресурсов |AuditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/DiagnosticSettingsForTypes_Audit.json) |
|[В Data Lake Analytics должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc95c74d9-38fe-4f0d-af86-0c7d626a315c) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeAnalytics_AuditDiagnosticLog_Audit.json) |

### <a name="the-organization-verifies-every-ninety-90-days-for-each-extract-of-covered-information-recorded-that-the-data-is-erased-or-its-use-is-still-required"></a>Каждые 90 (девяносто) дней организация проверяет, была ли удалена вся соответствующая извлеченная информация или же ее по-прежнему необходимо использовать.

**Идентификатор**. 1211.09aa3System.4 — 09.aa **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить аудит на сервере SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa6fb4358-5bf4-4ad7-ba82-2cd2f41ce5e9) |Для отслеживания действий во всех базах данных на сервере и сохранения их в журнал аудита должен быть включен аудит вашего SQL Server. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_Audit.json) |
|[В управляемом модуле HSM в Azure Key Vault необходимо включить журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa2a5b911-5617-447e-a49e-59dbe0e0434b) |Чтобы воссоздать следы действий для расследования инцидентов безопасности или несанкционированного доступа к сети, можно выполнить аудит, включив журналы ресурсов в управляемых модулях HSM. Следуйте инструкциям из этой статьи: [https://docs.microsoft.com/azure/key-vault/managed-hsm/logging](https://docs.microsoft.com/azure/key-vault/managed-hsm/logging). |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/ManagedHsm_AuditDiagnosticLog_Audit.json) |
|[В Key Vault должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf820ca0-f99e-4f3e-84fb-66e913812d21) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для анализа инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_AuditDiagnosticLog_Audit.json) |

## <a name="monitoring-system-use"></a>Мониторинг использования системы

### <a name="unauthorized-remote-connections-to-the-information-systems-are-monitored-and-reviewed-at-least-quarterly-and-appropriate-action-is-taken-if-an-unauthorized-connection-is-discovered"></a>Неавторизованные удаленные подключения к информационным системам отслеживаются и анализируются по крайней мере ежеквартально. При обнаружении неавторизованного подключения предпринимаются соответствующие меры.

**Идентификатор**. 1120.09ab3System.9 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Monitor должен собирать журналы действий из всех регионов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F41388f1c-2db0-4c25-95b2-35d7f5ccbfa9) |Эта политика осуществляет аудит профиля журнала Azure Monitor, который не экспортирует действия из всех поддерживаемых регионов Azure, включая глобальный. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllRegions.json) |

### <a name="the-organization-monitors-the-information-system-to-identify-irregularities-or-anomalies-that-are-indicators-of-a-system-malfunction-or-compromise-and-help-confirm-the-system-is-functioning-in-an-optimal-resilient-and-secure-state"></a>Организация осуществляет мониторинг информационной системы для выявления необычных событий или аномалий, которые являются индикаторами ее неисправности или компрометации. Это помогает убедиться, что система находится в оптимальном, отказоустойчивом и безопасном состоянии.

**Идентификатор**. 12100.09ab2System.15 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Агент Log Analytics должен быть установлен на виртуальных машинах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa70ca396-0a34-413a-88e1-b956c1e683be) |Эта политика выполняет аудит всех виртуальных машин Windows или Linux, на которых не установлен агент Log Analytics. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/VirtualMachines_LogAnalyticsAgent_AuditIfNotExists.json) |

### <a name="the-organization-specifies-how-often-audit-logs-are-reviewed-how-the-reviews-are-documented-and-the-specific-roles-and-responsibilities-of-the-personnel-conducting-the-reviews-including-the-professional-certifications-or-other-qualifications-required"></a>Организация указывает, как часто проверяются журналы аудита, как документируются проверки, а также какие именно роли и обязанности назначаются персоналу, осуществляющему проверки, в том числе профессиональные сертификации и другие необходимые квалификационные требования.

**Идентификатор**. 12101.09ab1Organizational.3 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Агент Log Analytics должен быть установлен в масштабируемом наборе виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fefbde977-ba53-4479-b8e9-10b957924fbf) |Эта политика выполняет аудит всех масштабируемых наборов виртуальных машин Windows или Linux, в которых не установлен агент Log Analytics. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/VMSS_LogAnalyticsAgent_AuditIfNotExists.json) |

### <a name="the-organization-shall-periodically-test-its-monitoring-and-detection-processes-remediate-deficiencies-and-improve-its-processes"></a>Организация периодически тестирует процессы мониторинга и обнаружения проблем, исправляет недостатки и улучшает соответствующие процессы.

**Идентификатор**. 12102.09ab1Organizational.4 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров с Windows без надлежащего подключения агента Log Analytics](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6265018c-d7e2-432f-a75d-094d5f6f4465) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если на них не установлен агент либо агент установлен, но COM-объект AgentConfigManager.MgmtSvcCfg сообщает, что этот агент зарегистрирован в рабочей области с идентификатором, отличным от указанного в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsLogAnalyticsAgentConnection_AINE.json) |

### <a name="all-applicable-legal-requirements-related-to-monitoring-authorized-access-and-unauthorized-access-attempts-are-met"></a>Удовлетворены все применимые юридические требования, связанные с мониторингом авторизованного доступа и попыток несанкционированного доступа.

**Идентификатор**. 1212.09ab1System.1 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Профиль журнала Azure Monitor должен собирать журналы по категориям "write", "delete" и "action"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1a4e592a-6a6e-44a5-9814-e36264ca96e7) |Эта политика заставляет профиль журнала собирать журналы по категориям "write" (запись), "delete" (удаление) и "action" (действие). |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllCategories.json) |

### <a name="automated-systems-deployed-throughout-the-organizations-environment-are-used-to-monitor-key-events-and-anomalous-activity-and-analyze-system-logs-the-results-of-which-are-reviewed-regularly"></a>Для наблюдения за основными событиями и аномальными действиями, а также анализа системных журналов и их регулярной проверки используются автоматизированные системы, развернутые в среде организации.

**Идентификатор**. 1213.09ab2System.128 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В подписке должна быть включена автоматическая подготовка агента Log Analytics](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |Чтобы отслеживать уязвимости и угрозы безопасности, Центр безопасности Azure собирает данные из виртуальных машин Azure. Для сбора данных используется агент Log Analytics (ранее — Microsoft Monitoring Agent, или MMA), который считывает разные конфигурации, связанные с безопасностью, и журналы событий с компьютера, а также копирует данные в рабочую область Log Analytics для анализа. Мы рекомендуем включить автоматическую подготовку для автоматического развертывания агента на всех поддерживаемых виртуальных машинах Azure, включая те, которые созданы недавно. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |

### <a name="monitoring-includes-privileged-operations-authorized-access-or-unauthorized-access-attempts-including-attempts-to-access-deactivated-accounts-and-system-alerts-or-failures"></a>Мониторинг охватывает привилегированные операции, авторизованный доступ или попытки несанкционированного доступа, в том числе попытки доступа к отключенным учетным записям, а также системные оповещения и сбои.

**Идентификатор**. 1214.09ab2System.3456 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Monitor должен собирать журналы действий из всех регионов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F41388f1c-2db0-4c25-95b2-35d7f5ccbfa9) |Эта политика осуществляет аудит профиля журнала Azure Monitor, который не экспортирует действия из всех поддерживаемых регионов Azure, включая глобальный. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllRegions.json) |

### <a name="auditing-and-monitoring-systems-employed-by-the-organization-support-audit-reduction-and-report-generation"></a>Системы аудита и мониторинга, используемые организацией, поддерживают сокращение аудита и создание отчетов.

**Идентификатор**. 1215.09ab2System.7 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Агент Log Analytics должен быть установлен на виртуальных машинах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa70ca396-0a34-413a-88e1-b956c1e683be) |Эта политика выполняет аудит всех виртуальных машин Windows или Linux, на которых не установлен агент Log Analytics. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/VirtualMachines_LogAnalyticsAgent_AuditIfNotExists.json) |

### <a name="automated-systems-are-used-to-review-monitoring-activities-of-security-systems-eg-ipsids-and-system-records-on-a-daily-basis-and-identify-and-document-anomalies"></a>Для ежедневной проверки действий мониторинга систем безопасности (например, IPS/IDS) и системных записей а также обнаружения и документирования аномалий используются автоматизированные системы.

**Идентификатор**. 1216.09ab3System.12 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Агент Log Analytics должен быть установлен в масштабируемом наборе виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fefbde977-ba53-4479-b8e9-10b957924fbf) |Эта политика выполняет аудит всех масштабируемых наборов виртуальных машин Windows или Linux, в которых не установлен агент Log Analytics. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/VMSS_LogAnalyticsAgent_AuditIfNotExists.json) |

### <a name="alerts-are-generated-for-technical-personnel-to-analyze-and-investigate-suspicious-activity-or-suspected-violations"></a>Для технического персонала создаются оповещения о необходимости анализа и исследования подозрительных действий или возможных нарушений.

**Идентификатор**. 1217.09ab3System.3 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров с Windows без надлежащего подключения агента Log Analytics](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6265018c-d7e2-432f-a75d-094d5f6f4465) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если на них не установлен агент либо агент установлен, но COM-объект AgentConfigManager.MgmtSvcCfg сообщает, что этот агент зарегистрирован в рабочей области с идентификатором, отличным от указанного в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsLogAnalyticsAgentConnection_AINE.json) |

### <a name="the-information-system-is-able-to-automatically-process-audit-records-for-events-of-interest-based-on-selectable-criteria"></a>Информационная система может автоматически обрабатывать записи аудита для интересующих событий на основе выбранных критериев.

**Идентификатор**. 1219.09ab3System.10 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Профиль журнала Azure Monitor должен собирать журналы по категориям "write", "delete" и "action"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1a4e592a-6a6e-44a5-9814-e36264ca96e7) |Эта политика заставляет профиль журнала собирать журналы по категориям "write" (запись), "delete" (удаление) и "action" (действие). |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllCategories.json) |

### <a name="monitoring-includes-inbound-and-outbound-communications-and-file-integrity-monitoring"></a>Выполняется мониторинг передачи входящих и исходящих данных и целостности файлов.

**Идентификатор**. 1220.09ab3System.56 — 09.ab **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В подписке должна быть включена автоматическая подготовка агента Log Analytics](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |Чтобы отслеживать уязвимости и угрозы безопасности, Центр безопасности Azure собирает данные из виртуальных машин Azure. Для сбора данных используется агент Log Analytics (ранее — Microsoft Monitoring Agent, или MMA), который считывает разные конфигурации, связанные с безопасностью, и журналы событий с компьютера, а также копирует данные в рабочую область Log Analytics для анализа. Мы рекомендуем включить автоматическую подготовку для автоматического развертывания агента на всех поддерживаемых виртуальных машинах Azure, включая те, которые созданы недавно. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |

## <a name="administrator-and-operator-logs"></a>Журналы администратора и оператора

### <a name="the-organization-ensures-proper-logging-is-enabled-in-order-to-audit-administrator-activities-and-reviews-system-administrator-and-operator-logs-on-a-regular-basis"></a>Организация обеспечивает правильное ведение журнала для аудита действий администратора и регулярно просматривает журналы системных администраторов и операторов.

**Идентификатор**. 1270.09ad1System.12 — 09.ad **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для выполнения определенных административных операций должно существовать оповещение журнала действий.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Эта политика выполняет аудит определенных административных операций без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="an-intrusion-detection-system-managed-outside-of-the-control-of-system-and-network-administrators-is-used-to-monitor-system-and-network-administration-activities-for-compliance"></a>Для мониторинга соответствия действий по администрированию системы и сети установленным требованиям используется система обнаружения вторжений, управление которой не контролируется системными и сетевыми администраторами.

**Идентификатор**. 1271.09ad1System.1 — 09.ad **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для выполнения определенных административных операций должно существовать оповещение журнала действий.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Эта политика выполняет аудит определенных административных операций без настроенных оповещений журнала действий. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

## <a name="segregation-of-duties"></a>Разделение обязанностей

### <a name="separation-of-duties-is-used-to-limit-the-risk-of-unauthorized-or-unintentional-modification-of-information-and-systems"></a>Используется разделение обязанностей для ограничения риска несанкционированного или непреднамеренного изменения информации и конфигурации систем.

**Идентификатор**. 1229.09c1Organizational.1 — 09.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В службах Kubernetes следует использовать управление доступом на основе ролей (RBAC)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |Чтобы обеспечить детальную фильтрацию действий, которые могут выполнять пользователи, используйте управление доступом на основе ролей (RBAC) для управления разрешениями в кластерах службы Kubernetes и настройте соответствующие политики авторизации. |Audit, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

### <a name="no-single-person-is-able-to-access-modify-or-use-information-systems-without-authorization-or-detection"></a>Пользователи не могут получать доступ к информационным системам, а также изменять или использовать их без авторизации или обнаружения.

**Идентификатор**. 1230.09c2Organizational.1 — 09.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит использования настраиваемых правил RBAC](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |Аудит встроенных ролей, таких как "Владелец", "Участник", "Читатель", вместо настраиваемых ролей RBAC, использование которых сопряжено с ошибками. Работа с пользовательскими ролями рассматривается как исключение и требует строгой проверки с моделированием угроз. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |

### <a name="access-for-individuals-responsible-for-administering-nbspaccess-controls-is-limited-to-the-minimum-necessary-based-upon-each-users-role-and-responsibilities-and-these-individuals-cannot-access-audit-functions-related-to-these-controls"></a>Для лиц, ответственных за администрирование &nbsp;управления доступом, предоставляется минимально необходимый уровень доступа с учетом ролей и обязанностей каждого пользователя, и у этих лиц нет доступа к функциям аудита, связанным с этими элементами управления.

**Идентификатор**. 1232.09c3Organizational.12 — 09.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Назначение прав пользователя"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe068b215-0026-4354-b347-8fb2766f73a2) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Назначение прав пользователя", разрешающие локальный вход в систему, RDP, доступ из сети и многие другие действия пользователей. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_UserRightsAssignment_AINE.json) |

### <a name="security-audit-activities-are-independent"></a>Действия аудита безопасности являются независимыми.

**Идентификатор**. 1276.09c2Organizational.2 — 09.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Роли владельца пользовательской подписки не должны существовать](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |Эта политика позволяет исключить наличие ролей владельца пользовательской подписки. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |

### <a name="the-initiation-of-an-event-is-separated-from-its-authorization-to-reduce-the-possibility-of-collusion"></a>Чтобы снизить вероятность сговора, инициация события отделена от его авторизации.

**Идентификатор**. 1277.09c2Organizational.4 — 09.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Параметры безопасности — контроль учетных записей"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F492a29ed-d143-4f03-b6a4-705ce081b463) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Параметры безопасности — контроль учетных записей" для режима администраторов, процедуры запроса на повышение прав, а также виртуализации сбоев при записи в файлы и реестр. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SecurityOptionsUserAccountControl_AINE.json) |

### <a name="the-organization-identifies-duties-that-require-separation-and-defines-information-system-access-authorizations-to-support-separation-of-duties-and-incompatible-duties-are-segregated-across-multiple-users-to-minimize-the-opportunity-for-misuse-or-fraud"></a>Организация определяет обязанности, для которых требуется разделение, и определяет способы авторизации доступа к информационной системе с целью разделения обязанностей. Чтобы свести к минимуму возможности для злоупотребления или мошенничества несовместимые обязанности распределяются между несколькими пользователями.

**Идентификатор**. 1278.09c2Organizational.56 — 09.c **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Роли владельца пользовательской подписки не должны существовать](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |Эта политика позволяет исключить наличие ролей владельца пользовательской подписки. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |

## <a name="controls-against-malicious-code"></a>Средства управления для защиты от вредоносного кода

### <a name="anti-virus-and-anti-spyware-are-installed-operating-and-updated-on-all-end-user-devices-to-conduct-periodic-scans-of-the-systems-to-identify-and-remove-unauthorized-software-server-environments-for-which-the-server-software-developer-specifically-recommends-not-installing-host-based-anti-virus-and-anti-spyware-software-may-address-the-requirement-via-a-network-based-malware-detection-nbmd-solution"></a>На всех устройствах конечных пользователей установлены, работают и обновляются антивирусные и антишпионские программы, выполняющие периодическую проверку систем для обнаружения и удаления неавторизованного программного обеспечения. В серверных средах, для которых разработчик серверного программного обеспечения явно рекомендует не устанавливать антивирусное и антишпионское программное обеспечение, это требование можно удовлетворить с помощью сетевого решения для обнаружения вредоносных программ (NBMD).

**Идентификатор**. 0201.09j1Organizational.124 — 09.j **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На компьютерах должны быть включены адаптивные элементы управления приложениями для определения безопасных приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Включите элементы управления приложениями, чтобы определить список проверенных безопасных приложений, выполняющихся на ваших компьютерах, и настроить получение оповещений о запуске других приложений. Это поможет защитить компьютеры от вредоносных программ. Чтобы упростить процесс настройки и обслуживания правил, Центр безопасности использует машинное обучение для анализа приложений, выполняющихся на каждом компьютере, и предлагает список проверенных безопасных приложений. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |
|[Развертывание расширения IaaSAntimalware (Майкрософт) по умолчанию для Windows Server](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2835b622-407b-4114-9198-6f7064cbe0dc) |Эта политика развертывает расширение IaaSAntimalware (Майкрософт) с конфигурацией по умолчанию, если для виртуальной машины не настроено расширение защиты от вредоносных программ. |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VMAntimalwareExtension_Deploy.json) |
|[В масштабируемых наборах виртуальных машин должно быть установлено решение Endpoint Protection](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F26a828e1-e88f-464e-bbb3-c134a282b9de) |Аудит наличия и работоспособности решения защиты конечных точек в ваших масштабируемых наборах виртуальных машин для защиты их от угроз и уязвимостей. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssMissingEndpointProtection_Audit.json) |
|[В Microsoft Antimalware для Azure должно быть настроено автоматическое обновление подписей защиты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc43e4a30-77cb-48ab-a4dd-93f175c63b57) |Эта политика выполняет аудит всех виртуальных машин Windows, на которых не настроено автоматическое обновление подписей защиты Microsoft Antimalware. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VirtualMachines_AntiMalwareAutoUpdate_AuditIfNotExists.json) |
|[Мониторинг отсутствия Endpoint Protection в Центре безопасности Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Серверы без установленного агента Endpoint Protection будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |
|[На компьютерах должны быть установлены обновления системы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86b3d65f-7626-441e-b690-81a8b71cff60) |Отсутствие обновлений системы безопасности на серверах будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingSystemUpdates_Audit.json) |

## <a name="back-up"></a>Резервное копирование

### <a name="backup-copies-of-information-and-software-are-made-and-tests-of-the-media-and-restoration-procedures-are-regularly-performed-at-appropriate-intervals"></a>Создаются резервные копии информации и программного обеспечения, а носители и процедуры восстановления проходят регулярное тестирование через приемлемые промежутки времени.

**Идентификатор**. 1616.09l1Organizational.16 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Базы данных SQL Azure должны использовать долгосрочное геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd38fc420-0735-4ef3-ac11-c806f651a570) |Эта политика выполняет аудит баз данных SQL Azure, для которых не включено долгосрочное геоизбыточное резервное копирование. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_SQLDatabase_AuditIfNotExists.json) |

### <a name="a-formal-definition-of-the-level-of-backup-required-for-each-system-is-defined-and-documented-including-how-each-system-will-be-restored-the-scope-of-data-to-be-imaged-frequency-of-imaging-and-duration-of-retention-based-on-relevant-contractual-legal-regulatory-and-business-requirements"></a>Для каждой системы задано и документировано формальное определение необходимого уровня резервного копирования, включая способ восстановления каждой системы, область данных для создания образа, частоту создания образов и продолжительность хранения. Это определение составляется на основе соответствующих договорных, юридических, нормативных и бизнес-требований.

**Идентификатор**. 1617.09l1Organizational.23 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для MySQL должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82339799-d096-41ae-8538-b108becf0970) |База данных Azure для MySQL позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForMySQL_Audit.json) |

### <a name="the-backups-are-stored-in-a-physically-secure-remote-location-at-a-sufficient-distance-to-make-them-reasonably-immune-from-damage-to-data-at-the-primary-site-and-reasonable-physical-and-environmental-controls-are-in-place-to-ensure-their-protection-at-the-remote-location"></a>Резервные копии хранятся в физически защищенном удаленном расположении на достаточном расстоянии для обеспечения приемлемого уровня их неуязвимости в случае повреждения данных на основной площадке. Для обеспечения защиты резервных копий в удаленном расположении предусмотрены надлежащие средства физического контроля или управления средой.

**Идентификатор**. 1618.09l1Organizational.45 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для PostgreSQL должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48af4db5-9b8b-401c-8e74-076be876a430) |База данных Azure для PostgreSQL позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForPostgreSQL_Audit.json) |

### <a name="inventory-records-for-the-backup-copies-including-content-and-current-location-are-maintained"></a>Сохраняются записи инвентаризации для резервных копий, включая содержимое и данные о текущем расположении.

**Идентификатор**. 1619.09l1Organizational.7 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для MariaDB должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ec47710-77ff-4a3d-9181-6aa50af424d0) |База данных Azure для MariaDB позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForMariaDB_Audit.json) |

### <a name="when-the-backup-service-is-delivered-by-the-third-party-the-service-level-agreement-includes-the-detailed-protections-to-control-confidentiality-integrity-and-availability-of-the-backup-information"></a>Если услуга резервного копирования предоставляется сторонним поставщиком, соглашение об уровне обслуживания включает в себя подробное описание средств защиты, позволяющих контролировать конфиденциальность, целостность и доступность данных резервных копий.

**Идентификатор**. 1620.09l1Organizational.8 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить Azure Backup для Виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Обеспечьте защиту виртуальных машин Azure, включив Azure Backup. Azure Backup — это безопасное и экономичное решение для защиты данных в Azure. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

### <a name="automated-tools-are-used-to-track-all-backups"></a>Для отслеживания всех резервных копий используются автоматизированные средства.

**Идентификатор**. 1621.09l2Organizational.1 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Базы данных SQL Azure должны использовать долгосрочное геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd38fc420-0735-4ef3-ac11-c806f651a570) |Эта политика выполняет аудит баз данных SQL Azure, для которых не включено долгосрочное геоизбыточное резервное копирование. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_SQLDatabase_AuditIfNotExists.json) |

### <a name="the-integrity-and-security-of-the-backup-copies-are-maintained-to-ensure-future-availability-and-any-potential-accessibility-problems-with-the-backup-copies-are-identified-and-mitigated-in-the-event-of-an-area-wide-disaster"></a>Для обеспечения доступности в будущем поддерживается целостность и безопасность резервных копий. При этом выявляются и устраняются все потенциальные проблемы с доступностью резервных копий в случае аварии на уровне области.

**Идентификатор**. 1622.09l2Organizational.23 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для MySQL должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82339799-d096-41ae-8538-b108becf0970) |База данных Azure для MySQL позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForMySQL_Audit.json) |

### <a name="covered-information-is-backed-up-in-an-encrypted-format-to-ensure-confidentiality"></a>Для обеспечения конфиденциальности для соответствующих сведений создаются резервные копии в зашифрованном формате.

**Идентификатор**. 1623.09l2Organizational.4 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для PostgreSQL должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48af4db5-9b8b-401c-8e74-076be876a430) |База данных Azure для PostgreSQL позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForPostgreSQL_Audit.json) |

### <a name="the-organization-performs-incremental-or-differential-backups-daily-and-full-backups-weekly-to-separate-media"></a>Организация выполняет добавочное или разностное резервное копирование ежедневно, а полное резервное копирование — еженедельно и на отдельные носители.

**Идентификатор**. 1624.09l3Organizational.12 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для MariaDB должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ec47710-77ff-4a3d-9181-6aa50af424d0) |База данных Azure для MariaDB позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForMariaDB_Audit.json) |

### <a name="three-3-generations-of-backups-full-plus-all-related-incremental-or-differential-backups-are-stored-off-site-and-both-on-site-and-off-site-backups-are-logged-with-name-date-time-and-action"></a>Удаленно хранится 3 (три) поколения резервных копий (полная и все связанные добавочные или разностные). При этом ведется журнал всех резервных копий, хранящихся как на месте, так и удаленно, в котором регистрируется имя, дата, время и действие.

**Идентификатор**. 1625.09l3Organizational.34 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить Azure Backup для Виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Обеспечьте защиту виртуальных машин Azure, включив Azure Backup. Azure Backup — это безопасное и экономичное решение для защиты данных в Azure. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

### <a name="the-organization-ensures-a-current-retrievable-copy-of-covered-information-is-available-before-movement-of-servers"></a>Организация гарантирует наличие доступной для извлечения копии соответствующей информации перед перемещением серверов.

**Идентификатор**. 1626.09l3Organizational.5 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для PostgreSQL должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48af4db5-9b8b-401c-8e74-076be876a430) |База данных Azure для PostgreSQL позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForPostgreSQL_Audit.json) |

### <a name="the-organization-tests-backup-information-following-each-backup-to-verify-media-reliability-and-information-integrity-and-at-least-annually-thereafter"></a>Организация тестирует данные резервных копий после каждого резервного копирования, чтобы проверить надежность носителей и целостность данных, а также повторяет эту процедуру в дальнейшем как минимум ежегодно.

**Идентификатор**. 1627.09l3Organizational.6 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для MariaDB должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ec47710-77ff-4a3d-9181-6aa50af424d0) |База данных Azure для MariaDB позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForMariaDB_Audit.json) |

### <a name="workforce-members-roles-and-responsibilities-in-the-data-backup-process-are-identified-and-communicated-to-the-workforce-in-particular-bring-your-own-device-byod-users-are-required-to-perform-backups-of-organizational-andor-client-data-on-their-devices"></a>Роли и обязанности сотрудников в процессе резервного копирования данных определены и известны сотрудникам. В частности, для создания резервных копий данных организации и (или) клиентов на своих устройствах сотрудники должны быть пользователями BYOD (участниками инициативы "Принеси свое устройство").

**Идентификатор**. 1699.09l1Organizational.10 — 09.l **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить Azure Backup для Виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Обеспечьте защиту виртуальных машин Azure, включив Azure Backup. Azure Backup — это безопасное и экономичное решение для защиты данных в Azure. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

## <a name="network-controls"></a>Элементы управления сетью

### <a name="the-organization-monitors-for-all-authorized-and-unauthorized-wireless-access-to-the-information-system-and-prohibits-installation-of-wireless-access-points-waps-unless-explicitly-authorized-in-writing-by-the-cio-or-hisher-designated-representative"></a>Организация отслеживает все случаи авторизованного и несанкционированного беспроводного доступа к информационной системе и запрещает установку точек беспроводного доступа (WAP) без явного письменного разрешения директора по информационной безопасности или назначенного им представителя.

**Идентификатор**. 0858.09m1Organizational.4 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Доступ ко всем сетевым портам должен быть ограничен в связанных с виртуальной машиной группах безопасности сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9daedab3-fb2d-461e-b861-71790eead4f6) |Центр безопасности Azure определил, что некоторые правила для входящих подключений в группах безопасности сети недостаточно строгие. Правила для входящих подключений не должны разрешать доступ из диапазонов "Любой" или "Интернет". В противном случае злоумышленники смогут атаковать ваши ресурсы. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnprotectedEndpoints_Audit.json) |
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |
|[Компьютеры с Windows должны соответствовать требованиям для категории "Свойства брандмауэра Windows"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F35d9882c-993d-44e6-87d2-db66ce21b636) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Свойства брандмауэра Windows" для состояния брандмауэра, подключений, управления правилами и уведомлений. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsFirewallProperties_AINE.json) |

### <a name="the-organization-ensures-the-security-of-information-in-networks-availability-of-network-services-and-information-services-using-the-network-and-the-protection-of-connected-services-from-unauthorized-access"></a>Организация обеспечивает безопасность информации в сетях, доступность сетевых и информационных служб, использующих сеть, а также защиту подключенных служб от несанкционированного доступа.

**Идентификатор**. 0859.09m1Organizational.78 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |

### <a name="the-organization-formally-manages-equipment-on-the-network-including-equipment-in-user-areas"></a>Организация официально управляет оборудованием в сети, в том числе оборудованием в областях пользователей.

**Идентификатор**. 0860.09m1Organizational.9 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Развертывание параметров диагностики для групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc9c29499-c1d1-4195-99bd-2ec9e3a9dc89) |Эта политика обеспечивает автоматическое развертывание параметров диагностики в группах безопасности сети. Будет автоматически создана учетная запись хранения с именем "{storagePrefixParameter}{NSGLocation}". |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/DiagnosticSettingsForNSG_Deploy.json) |
|[Служебная шина Azure должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |Эта политика выполняет аудит всех служебных шин, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |

### <a name="to-identify-and-authenticate-devices-on-local-andor-wide-area-networks-including-wireless-networks-nbspthe-information-system-uses-either-a-i-shared-known-information-solution-or-ii-an-organizational-authentication-solution-the-exact-selection-and-strength-of-which-is-dependent-on-the-security-categorization-of-the-information-system"></a>Для идентификации и проверки подлинности устройств в локальных и (или) глобальных сетях, в том числе беспроводных, &nbsp;информационная система использует 1) общее известное информационное решение или 2) решение организации для проверки подлинности, точный выбор и надежность которого зависят от категории безопасности информационной системы.

**Идентификатор**. 0861.09m2Organizational.67 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба приложений должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2d21331d-a4c2-4def-a9ad-ee4e1e023beb) |Эта политика выполняет аудит для любой Службы приложений, не настроенной на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_AppService_AuditIfNotExists.json) |
|[Компьютеры с Windows должны соответствовать требованиям для категории "Параметры безопасности — доступ к сети"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3ff60f98-7fa4-410a-9f7f-0b00f5afdbdd) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Параметры безопасности — доступ к сети", в том числе для доступа анонимных пользователей, локальных учетных записей и удаленного доступа к реестру. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SecurityOptionsNetworkAccess_AINE.json) |

### <a name="the-organization-ensures-information-systems-protect-the-confidentiality-and-integrity-of-transmitted-information-including-during-preparation-for-transmission-and-during-reception"></a>Организация гарантирует, что информационные системы защищают конфиденциальность и целостность передаваемых данных, в том числе во время подготовки к передаче и приема.

**Идентификатор**. 0862.09m2Organizational.8 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба SQL Server должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fae5d2f14-d830-42b6-9899-df6cfe9c71a3) |Эта политика выполняет аудит всех SQL Server, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_SQLServer_AuditIfNotExists.json) |

### <a name="the-organization-builds-a-firewall-configuration-that-restricts-connections-between-un-trusted-networks-and-any-system-components-in-the-covered-information-environment-and-any-changes-to-the-firewall-configuration-are-updated-in-the-network-diagram"></a>Организация создает конфигурацию брандмауэра, которая ограничивает подключения между ненадежными сетями и любыми компонентами системы в соответствующей информационной среде. Схема сети обновляется с учетом всех изменений в конфигурации брандмауэра.

**Идентификатор**. 0863.09m2Organizational.910 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Концентратор событий должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd63edb4a-c612-454d-b47d-191a724fcbf0) |Эта политика выполняет аудит всех концентраторов событий, не настроенных на использование конечной точки службы виртуальной сети. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_EventHub_AuditIfNotExists.json) |

### <a name="usage-restrictions-and-implementation-guidance-are-formally-defined-for-voip-including-the-authorization-and-monitoring-of-the-service"></a>Для протокола VoIP формально определены ограничения на использование и рекомендации по реализации, в том числе для авторизации и мониторинга службы.

**Идентификатор**. 0864.09m2Organizational.12 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба Cosmos DB должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe0a2b1a3-f7f9-4569-807f-2a9edebdf4d9) |Эта политика выполняет аудит всех Cosmos DB, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_CosmosDB_Audit.json) |

### <a name="the-organization-i-authorizes-connections-from-the-information-system-to-other-information-systems-outside-of-the-organization-through-the-use-of-interconnection-security-agreements-or-other-formal-agreement-ii-documents-each-connection-the-interface-characteristics-security-requirements-and-the-nature-of-the-information-communicated-iii-employs-a-deny-all-permit-by-exception-policy-for-allowing-connections-from-the-information-system-to-other-information-systems-outside-of-the-organization-and-iv-applies-a-default-deny-rule-that-drops-all-traffic-via-host-based-firewalls-or-port-filtering-tools-on-its-endpoints-workstations-servers-etc-except-those-services-and-ports-that-are-explicitly-allowed"></a>Организация 1) разрешает подключения из информационной системы к другим информационным системам за пределами организации, используя соглашения о безопасности при взаимном подключении или другое формальное соглашение; 2) документирует каждое подключение, характеристики интерфейса, требования безопасности и характер сведений, которыми обмениваются системы; 3) использует политику "Запрещено все, кроме разрешенного исключениями" для разрешения подключений из информационной системы к другим информационным системам за пределами организации; 4) применяет правило "Запрещено по умолчанию", которое удаляет весь трафик, проходящий через брандмауэры на узлах или средства фильтрации портов на ее конечных точках (рабочих станциях, серверах и т. д.), за исключением трафика тех служб и портов, которые разрешены явно.

**Идентификатор**. 0865.09m2Organizational.13 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба Key Vault должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fea4d6841-2173-4317-9747-ff522a45120f) |Эта политика выполняет аудит всех Key Vault, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_KeyVault_Audit.json) |

### <a name="the-organization-describes-the-groups-roles-and-responsibilities-for-the-logical-management-of-network-components-and-ensures-coordination-of-and-consistency-in-the-elements-of-the-network-infrastructure"></a>Организация описывает группы, роли и обязанности для логического управления сетевыми компонентами и обеспечивает координирование и согласованность в элементах сетевой инфраструктуры.

**Идентификатор**. 0866.09m3Organizational.1516 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Учетные записи хранения должны ограничивать доступ к сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34c877ad-507e-4c82-993e-3452a6e0ad3c) |Сетевой доступ к учетным записям хранения должен быть ограниченным. Настройте правила сети так, чтобы учетная запись хранения была доступна только приложениям из разрешенных сетей. Чтобы разрешить подключения от конкретных локальных клиентов и интернет-клиентов, вы можете открыть доступ для трафика из конкретных виртуальных сетей Azure или определенных диапазонов общедоступных IP-адресов. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_NetworkAcls_Audit.json) |

### <a name="wireless-access-points-are-placed-in-secure-areas-and-shut-down-when-not-in-use-eg-nights-weekends"></a>Точки беспроводного доступа размещаются в безопасных областях, а на время, когда они не используются (например, ночью или в выходные дни), их работа завершается.

**Идентификатор**. 0867.09m3Organizational.17 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Учетная запись хранения должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F60d21c4f-21a3-4d94-85f4-b924e6aeeda4) |Эта политика выполняет аудит всех учетных записей хранения, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_StorageAccount_Audit.json) |

### <a name="the-organization-builds-a-firewall-configuration-to-restrict-inbound-and-outbound-traffic-to-that-which-is-necessary-for-the-covered-data-environment"></a>Организация создает конфигурацию брандмауэра, ограничивающую входящий и исходящий трафик только тем, который необходим для соответствующей среды данных.

**Идентификатор**. 0868.09m3Organizational.18 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Реестр контейнеров должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78) |Эта политика выполняет аудит всех реестров контейнеров, не настроенных для использования конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ContainerRegistry_Audit.json) |

### <a name="the-router-configuration-files-are-secured-and-synchronized"></a>Файлы конфигурации маршрутизатора защищены и синхронизированы.

**Идентификатор**. 0869.09m3Organizational.19 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Реестр контейнеров должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78) |Эта политика выполняет аудит всех реестров контейнеров, не настроенных для использования конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ContainerRegistry_Audit.json) |

### <a name="access-to-all-proxies-is-denied-except-for-those-hosts-ports-and-services-that-are-explicitly-required"></a>Доступ ко всем прокси-компонентам запрещен, кроме тех узлов, портов и служб, которые требуются явным образом.

**Идентификатор**. 0870.09m3Organizational.20 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Реестр контейнеров должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78) |Эта политика выполняет аудит всех реестров контейнеров, не настроенных для использования конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ContainerRegistry_Audit.json) |

### <a name="authoritative-dns-servers-are-segregated-into-internal-and-external-roles"></a>Полномочные DNS-серверы разделяются на основе внутренних и внешних ролей.

**Идентификатор**. 0871.09m3Organizational.22 — 09.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Реестр контейнеров должен использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4857be7-912a-4c75-87e6-e30292bcdf78) |Эта политика выполняет аудит всех реестров контейнеров, не настроенных для использования конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ContainerRegistry_Audit.json) |

## <a name="security-of-network-services"></a>Безопасность сетевых служб

### <a name="agreed-services-provided-by-a-network-service-providermanager-are-formally-managed-and-monitored-to-ensure-they-are-provided-securely"></a>Согласованные службы, предоставляемые поставщиком или диспетчером сетевых услуг, официально управляются и отслеживаются для гарантии безопасности их предоставления.

**Идентификатор**. 0835.09n1Organizational.1 — 09.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах Windows должен быть установлен агент сбора данных сетевого трафика.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2f2ee1de-44aa-4762-b6bd-0893fc3f306d) |Центр безопасности использует Microsoft Dependency Agent для сбора данных сетевого трафика с ваших виртуальных машин Azure и реализации расширенных функций защиты, например визуализации трафика на карте сети, выдачи рекомендаций по улучшению безопасности и обработки конкретных сетевых угроз. |AuditIfNotExists, Disabled |[1.0.1 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ASC_Dependency_Agent_Audit_Windows.json) |
|[Виртуальные машины должны быть перенесены на новые ресурсы Azure Resource Manager](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1d84d5fb-01f6-4d12-ba4f-4a26081d403d) |Используйте для своих виртуальных машин новый выпуск Azure Resource Manager с улучшенными функциями безопасности, включая более строгий контроль доступа (RBAC), улучшенный аудит, развертывание и управление на основе Azure Resource Manager, доступ к управляемым удостоверениям и хранилищам ключей для секретов, проверку подлинности на основе Azure AD, а также поддержку тегов и групп ресурсов, упрощающих управление защитой. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/ClassicCompute_Audit.json) |

### <a name="the-organization-formally-authorizes-and-documents-the-characteristics-of-each-connection-from-an-information-system-to-other-information-systems-outside-the-organization"></a>Организация официально разрешает и документирует характеристики каждого подключения информационной системы к другим информационным системам за пределами организации.

**Идентификатор**. 0836.09.n2Organizational.1 — 09.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах Linux должен быть установлен агент сбора данных сетевого трафика.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F04c4380f-3fae-46e8-96c9-30193528f602) |Центр безопасности использует Microsoft Dependency Agent для сбора данных сетевого трафика с ваших виртуальных машин Azure и реализации расширенных функций защиты, например визуализации трафика на карте сети, выдачи рекомендаций по улучшению безопасности и обработки конкретных сетевых угроз. |AuditIfNotExists, Disabled |[1.0.1 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ASC_Dependency_Agent_Audit_Linux.json) |

### <a name="formal-agreements-with-external-information-system-providers-include-specific-obligations-for-security-and-privacy"></a>Формальные соглашения с внешними поставщиками информационных систем предусматривают определенные обязательства по обеспечению безопасности и конфиденциальности.

**Идентификатор**. 0837.09.n2Organizational.2 — 09.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить Наблюдатель за сетями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6e2945c-0b7b-40f5-9233-7a5323b5cdc6) |Наблюдатель за сетями — это региональная служба, обеспечивающая мониторинг и диагностику условий на уровне сетевого сценария на платформе Azure. Мониторинг на уровне сценария позволяет диагностировать проблемы в сети с помощью комплексного представления сетевого уровня. Инструменты диагностики сети и визуализации, доступные в Наблюдателе за сетями, помогают понять, как работает сеть в Azure, диагностировать ее и получить ценную информацию. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkWatcher_Enabled_Audit.json) |

### <a name="the-organization-reviews-and-updates-the-interconnection-security-agreements-on-an-ongoing-basis-verifying-enforcement-of-security-requirements"></a>Организация проверяет и обновляет соглашения о безопасности взаимодействия на постоянной основе, а также проверяет соблюдение требований безопасности.

**Идентификатор**. 0885.09n2Organizational.3 — 09.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах Linux должен быть установлен агент сбора данных сетевого трафика.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F04c4380f-3fae-46e8-96c9-30193528f602) |Центр безопасности использует Microsoft Dependency Agent для сбора данных сетевого трафика с ваших виртуальных машин Azure и реализации расширенных функций защиты, например визуализации трафика на карте сети, выдачи рекомендаций по улучшению безопасности и обработки конкретных сетевых угроз. |AuditIfNotExists, Disabled |[1.0.1 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ASC_Dependency_Agent_Audit_Linux.json) |

### <a name="the-organization-employs-and-documents-in-a-formal-agreement-or-other-document-either-i-allow-all-deny-by-exception-or-ii-deny-all-permit-by-exception-preferred-policy-for-allowing-specific-information-systems-to-connect-to-external-information-systems"></a>Организация применяет и документирует в официальном соглашении или другом документе политику 1) "Разрешено все, кроме запрещенного исключениями" или 2) "Запрещено все, кроме разрешенного исключениями", позволяющую определенным информационным системам подключаться к внешним информационным системам.

**Идентификатор**. 0886.09n2Organizational.4 — 09.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить Наблюдатель за сетями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6e2945c-0b7b-40f5-9233-7a5323b5cdc6) |Наблюдатель за сетями — это региональная служба, обеспечивающая мониторинг и диагностику условий на уровне сетевого сценария на платформе Azure. Мониторинг на уровне сценария позволяет диагностировать проблемы в сети с помощью комплексного представления сетевого уровня. Инструменты диагностики сети и визуализации, доступные в Наблюдателе за сетями, помогают понять, как работает сеть в Azure, диагностировать ее и получить ценную информацию. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkWatcher_Enabled_Audit.json) |

### <a name="the-organization-requires-externaloutsourced-service-providers-to-identify-the-specific-functions-ports-and-protocols-used-in-the-provision-of-the-externaloutsourced-services"></a>Организация требует, чтобы внешние поставщики услуг или аутсорсеры идентифицировали конкретные функции, порты и протоколы, используемые для внешних или аутсорсинговых служб.

**Идентификатор**. 0887.09n2Organizational.5 — 09.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах Windows должен быть установлен агент сбора данных сетевого трафика.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2f2ee1de-44aa-4762-b6bd-0893fc3f306d) |Центр безопасности использует Microsoft Dependency Agent для сбора данных сетевого трафика с ваших виртуальных машин Azure и реализации расширенных функций защиты, например визуализации трафика на карте сети, выдачи рекомендаций по улучшению безопасности и обработки конкретных сетевых угроз. |AuditIfNotExists, Disabled |[1.0.1 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ASC_Dependency_Agent_Audit_Windows.json) |

### <a name="the-contract-with-the-externaloutsourced-service-provider-includes-the-specification-that-the-service-provider-is-responsible-for-the-protection-of-covered-information-shared"></a>Контракт с внешним поставщиком или аутсорсером услуг включает положение о том, что поставщик услуг несет ответственность за защиту соответствующей информации, к которой ему предоставлен общий доступ.

**Идентификатор**. 0888.09n2Organizational.6 — 09.n **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить Наблюдатель за сетями](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6e2945c-0b7b-40f5-9233-7a5323b5cdc6) |Наблюдатель за сетями — это региональная служба, обеспечивающая мониторинг и диагностику условий на уровне сетевого сценария на платформе Azure. Мониторинг на уровне сценария позволяет диагностировать проблемы в сети с помощью комплексного представления сетевого уровня. Инструменты диагностики сети и визуализации, доступные в Наблюдателе за сетями, помогают понять, как работает сеть в Azure, диагностировать ее и получить ценную информацию. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkWatcher_Enabled_Audit.json) |

## <a name="management-of-removable-media"></a>Управление съемными носителями

### <a name="the-organization-based-on-the-data-classification-level-registers-media-including-laptops-prior-to-use-places-reasonable-restrictions-on-how-such-media-be-used-and-provides-an-appropriate-level-of-physical-and-logical-protection-including-encryption-for-media-containing-covered-information-until-properly-destroyed-or-sanitized"></a>На основе требований к уровню классификации данных организация регистрирует носители (включая ноутбуки) перед использованием, налагает обоснованные ограничения на использование этих носителей и обеспечивает соответствующий уровень физической и логической защиты (включая шифрование) для носителей, которые содержат защищаемые сведения, вплоть до момента их правильного уничтожения или очистки.

**Идентификатор**. 0301.09o1Organizational.123 — 09.o **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В базах данных SQL должно применяться прозрачное шифрование данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17k78e20-9358-41c9-923c-fb736d382a12) |Для защиты неактивных данных и обеспечения соответствия требованиям должно быть включено прозрачное шифрование данных. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlDBEncryption_Audit.json) |

### <a name="the-organization-protects-and-controls-media-containing-sensitive-information-during-transport-outside-of-controlled-areas"></a>Организация защищает и контролирует носители, содержащие конфиденциальные сведения, во время транспортировки за пределами контролируемых областей.

**Идентификатор**. 0302.09o2Organizational.1 — 09.o **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах должно применяться шифрование дисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) |Виртуальные машины без шифрования дисков будут отслеживаться Центром безопасности Azure (это требование носит рекомендательный характер). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnencryptedVMDisks_Audit.json) |

### <a name="digital-and-non-digital-media-requiring-restricted-use-and-the-specific-safeguards-used-to-restrict-their-use-are-identified"></a>Идентифицируются цифровые и нецифровые носители, использование которых должно быть ограничено с помощью определенных средств защиты.

**Идентификатор**. 0303.09o2Organizational.2 — 09.o **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Неподключенные диски должны быть зашифрованы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c89a2e5-7285-40fe-afe0-ae8654b92fb2) |Эта политика выполняет аудит всех неподключенных дисков, на которых не активировано шифрование. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/UnattachedDisk_Encryption_Audit.json) |

### <a name="the-organization-restricts-the-use-of-writable-removable-media-and-personally-owned-removable-media-in-organizational-systems"></a>Организация ограничивает использование съемных носителей с возможностью записи и персональных съемных носителей в системах организации.

**Идентификатор**. 0304.09o3Organizational.1 — 09.o **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Требовать шифрование учетных записей Data Lake Store](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7ff3161-0087-490a-9ad9-ad6217f4f43a) |Эта политика активирует шифрование во всех учетных записях Data Lake Storage |deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStoreEncryption_Deny.json) |
|[Управляемые экземпляры SQL должны использовать ключи, управляемые клиентом, для шифрования неактивных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F048248b0-55cd-46da-b1ff-39efd52db260) |Реализация прозрачного шифрования данных (TDE) с вашим собственным ключом обеспечивает повышенную прозрачность и контроль над предохранителем TDE, а также дополнительную защиту за счет использования внешней службы с поддержкой HSM и содействие разделению обязанностей. Эта рекомендация относится к организациям, имеющим связанное требование по соответствию. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |
|[Серверы SQL должны использовать управляемые клиентом ключи для шифрования неактивных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d134df8-db83-46fb-ad72-fe0c9428c8dd) |Реализация прозрачного шифрования данных (TDE) с вашим ключом обеспечивает повышенную прозрачность и контроль над предохранителем TDE, а также дополнительную защиту за счет использования внешней службы с поддержкой HSM и содействие разделению обязанностей. Эта рекомендация относится к организациям, имеющим связанное требование по соответствию. |AuditIfNotExists, Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |

## <a name="information-exchange-policies-and-procedures"></a>Политики и процедуры обмена информацией

### <a name="cloud-service-providers-use-an-industry-recognized-virtualization-platform-and-standard-virtualization-formats-eg-open-virtualization-format-ovf-to-help-ensure-interoperability-and-has-documented-custom-changes-made-to-any-hypervisor-in-use-and-all-solution-specific-virtualization-hooks-available-for-customer-review"></a>Поставщики облачных служб используют признанную в отрасли платформу виртуализации и стандартные форматы виртуализации (например, OVF — Open Virtualization Format) для поддержки взаимодействия, а также документируют все дополнительные изменения, внесенные в используемые низкоуровневые оболочки, и все доступные для проверки клиентом перехватчики виртуализации в своих решениях.

**Идентификатор**. 0662.09sCSPOrganizational.2 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Параметр "Сертификаты клиента (входящие сертификаты клиента)" для веб-приложения должен иметь значение "Вкл"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5bb220d9-2698-4ee4-8404-b9c30c9df609) |Сертификаты клиента позволяют приложению запрашивать сертификат для входящих запросов. Приложение будет доступно только клиентам с допустимым сертификатом. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_ClientCert.json) |

### <a name="the-organization-formally-addresses-multiple-safeguards-before-allowing-the-use-of-information-systems-for-information-exchange"></a>Организация формально принимает несколько мер защиты, прежде чем разрешить использование информационных систем для обмена информацией.

**Идентификатор**. 0901.09s1Organizational.1 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Функция CORS не должна разрешать всем ресурсам доступ к вашим веб-приложениям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5744710e-cc2f-4ee8-8809-3b11e89f4bc9) |Средства общего доступа к ресурсам независимо от источника (CORS) не должны разрешать доступ к вашему веб-приложению всем доменам. Разрешите взаимодействие с веб-приложением только необходимым доменам. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RestrictCORSAccess_WebApp_Audit.json) |

### <a name="remote-external-access-to-the-organizations-information-assets-and-access-to-external-information-assets-for-which-the-organization-has-no-control-is-based-on-clearly-defined-terms-and-conditions"></a>Удаленный (внешний) доступ к информационным ресурсам организации и доступ к внешним информационным ресурсам (которыми организация не управляет) основана на четко определенных условиях.

**Идентификатор**. 0902.09s2Organizational.13 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Функция CORS не должны разрешать всем ресурсам доступ к вашим приложениям-функциям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0820b7b9-23aa-4725-a1ce-ae4558f718e5) |Средства общего доступа к ресурсам независимо от источника (CORS) не должны разрешать доступ к вашему приложению-функции всем доменам. Разрешите взаимодействие с приложением-функцией только необходимым доменам. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RestrictCORSAccess_FuntionApp_Audit.json) |

### <a name="the-organization-establishes-terms-and-conditions-consistent-with-any-trust-relationship-established-with-other-organizations-owning-operating-andor-maintaining-external-information-systems-allowing-authorized-individuals-to-i-access-the-information-system-from-external-information-systems-and-ii-process-store-or-transmit-organization-controlled-information-using-external-information-systems"></a>Организация устанавливает условия, согласованные с любыми отношениями доверия с другими организациями, владеющими, эксплуатирующими и (или) обслуживающими внешние информационные системы, в соответствии с которыми уполномоченные пользователи могут 1) получать доступ к информационной системе из внешних информационных систем и 2) обрабатывать, хранить или передавать сведения, находящиеся под управлением организации, с помощью внешних информационных систем.

**Идентификатор**. 0911.09s1Organizational.2 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Средства CORS не должны разрешать всем ресурсам доступ к вашему приложению API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F358c20a6-3f9e-4f0e-97ff-c6ce485e2aac) |Средства общего доступа к ресурсам независимо от источника (CORS) не должны разрешать доступ к вашему приложению API всем доменам. Разрешите взаимодействие с приложением API только необходимым доменам. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RestrictCORSAccess_ApiApp_Audit.json) |

### <a name="cryptography-is-used-to-protect-the-confidentiality-and-integrity-of-remote-access-sessions-to-the-internal-network-and-to-external-systems"></a>Для защиты конфиденциальности и целостности сеансов удаленного доступа к внутренней сети и внешним системам используется шифрование.

**Идентификатор**. 0912.09s1Organizational.4 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Удаленная отладка должна быть отключена для веб-приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcb510bfd-1cba-4d9f-a230-cb0976f4bb71) |Для удаленной отладки нужно, чтобы в веб-приложении были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_WebApp_Audit.json) |

### <a name="strong-cryptography-protocols-are-used-to-safeguard-covered-information-during-transmission-over-less-trusted--open-public-networks"></a>Для защиты информации в процессе передачи по менее надежным или открытым общедоступным сетям используются надежные протоколы шифрования.

**Идентификатор**. 0913.09s1Organizational.5 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Удаленная отладка должна быть отключена для приложений-функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e60b895-3786-45da-8377-9c6b4b6ac5f9) |Для удаленной отладки нужно, чтобы в приложениях-функциях были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_FunctionApp_Audit.json) |

### <a name="the-organization-ensures-that-communication-protection-requirements-including-the-security-of-exchanges-of-information-is-the-subject-of-policy-development-and-compliance-audits"></a>Организация гарантирует, что в отношении требований к защите связи, в том числе безопасности обмена информацией, разработана соответствующая политика и проводится аудит их соблюдения.

**Идентификатор**. 0914.09s1Organizational.6 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Удаленная отладка должна быть отключена для приложений API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe9c8d085-d9cc-4b17-9cdc-059f1f01f19e) |Для удаленной отладки нужно, чтобы в приложениях API были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_ApiApp_Audit.json) |

### <a name="the-organization-limits-the-use-of-organization-controlled-portable-storage-media-by-authorized-individuals-on-external-information-systems"></a>Организация ограничивает использование переносимых запоминающих устройств, управляемых ею, уполномоченными пользователями из внешних информационных систем.

**Идентификатор**. 0915.09s2Organizational.2 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Параметр "Сертификаты клиента (входящие сертификаты клиента)" для веб-приложения должен иметь значение "Вкл"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5bb220d9-2698-4ee4-8404-b9c30c9df609) |Сертификаты клиента позволяют приложению запрашивать сертификат для входящих запросов. Приложение будет доступно только клиентам с допустимым сертификатом. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_ClientCert.json) |

### <a name="the-information-system-prohibits-remote-activation-of-collaborative-computing-devices-and-provides-an-explicit-indication-of-use-to-users-physically-present-at-the-devices"></a>Информационная система запрещает удаленную активацию для совместно используемых вычислительных устройств и явно оповещает об использовании пользователей с физическим доступом к устройствам.

**Идентификатор**. 0916.09s2Organizational.4 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Функция CORS не должна разрешать всем ресурсам доступ к вашим веб-приложениям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5744710e-cc2f-4ee8-8809-3b11e89f4bc9) |Средства общего доступа к ресурсам независимо от источника (CORS) не должны разрешать доступ к вашему веб-приложению всем доменам. Разрешите взаимодействие с веб-приложением только необходимым доменам. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RestrictCORSAccess_WebApp_Audit.json) |

### <a name="cloud-service-providers-use-secure-eg-non-clear-text-and-authenticated-standardized-network-protocols-for-the-import-and-export-of-data-and-to-manage-the-service-and-make-available-a-document-to-consumers-tenants-detailing-the-relevant-interoperability-and-portability-standards-that-are-involved"></a>Поставщики облачных служб используют безопасные (например, нечеткие текстовые и аутентифицированные) стандартизированные сетевые протоколы для импорта и экспорта данных и управления службой, а также предоставляют доступ к документам потребителям (клиентам) с подробными сведениями о применении соответствующих стандартов взаимодействия и переносимости.

**Идентификатор**. 0960.09sCSPOrganizational.1 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Функция CORS не должны разрешать всем ресурсам доступ к вашим приложениям-функциям](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0820b7b9-23aa-4725-a1ce-ae4558f718e5) |Средства общего доступа к ресурсам независимо от источника (CORS) не должны разрешать доступ к вашему приложению-функции всем доменам. Разрешите взаимодействие с приложением-функцией только необходимым доменам. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RestrictCORSAccess_FuntionApp_Audit.json) |

### <a name="personnel-are-appropriately-trained-on-leading-principles-and-practices-for-all-types-of-information-exchange-oral-paper-and-electronic"></a>Персонал должным образом обучен основным принципам и методикам для всех типов обмена информацией (устного, бумажного и электронного).

**Идентификатор**. 1325.09s1Organizational.3 — 09.s **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Удаленная отладка должна быть отключена для приложений-функций](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e60b895-3786-45da-8377-9c6b4b6ac5f9) |Для удаленной отладки нужно, чтобы в приложениях-функциях были открыты входящие порты. Удаленную отладку необходимо отключить. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_DisableRemoteDebugging_FunctionApp_Audit.json) |

## <a name="on-line-transactions"></a>Транзакции в Интернете

### <a name="data-involved-in-electronic-commerce-and-online-transactions-is-checked-to-determine-if-it-contains-covered-information"></a>Данные, используемые в электронной коммерции и интерактивных транзакциях, проверяются на наличие в них соответствующей информации.

**Идентификатор**. 0943.09y1Organizational.1 — 09.y **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Должно выполняться безопасное перемещение в учетные записи хранения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Настройка требования выполнять аудит для безопасного переноса в учетную запись хранения. Безопасная передача данных — это параметр, при включении которого ваша учетная запись хранения принимает запросы только с безопасных подключений (HTTPS). Протокол HTTPS обеспечивает проверку подлинности между сервером и службой, а также защищает перемещаемые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |

### <a name="protocols-used-to-communicate-between-all-involved-parties-are-secured-using-cryptographic-techniques-eg-ssl"></a>Протоколы, используемые для обмена данными между всеми участвующими сторонами, защищаются с помощью методов шифрования (например, SSL).

**Идентификатор**. 0945.09y1Organizational.3 — 09.y **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров Windows без указанных сертификатов в доверенном корневом центре сертификации](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F934345e1-4dfb-4c70-90d7-41990dc9608b) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если хранилище доверенных корневых сертификатов (CERT:\LocalMachine\Root) на них не содержит один или несколько сертификатов, указанных в параметре политики. |auditIfNotExists |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_WindowsCertificateInTrustedRoot_AINE.json) |

### <a name="the-organization-requires-the-use-of-encryption-between-and-the-use-of-electronic-signatures-by-each-of-the-parties-involved-in-the-transaction"></a>Организация требует использования шифрования при обмене данными между сторонами, вовлеченными в транзакцию, и использования электронных подписей каждой из них.

**Идентификатор**. 0946.09y2Organizational.14 — 09.y **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Использование только безопасных подключений к Кэшу Azure для Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |Аудит активации только подключений по протоколу SSL к кэшу Azure для Redis. Использование безопасных подключений обеспечивает проверку подлинности между сервером и службой, а также защищает переносимые данные от атак сетевого уровня, таких как "злоумышленник в середине", прослушивание трафика и перехват сеанса. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

### <a name="the-organization-ensures-the-storage-of-the-transaction-details-are-located-outside-of-any-publicly-accessible-environments-eg-on-a-storage-platform-existing-on-the-organizations-intranet-and-not-retained-and-exposed-on-a-storage-medium-directly-accessible-from-the-internet"></a>Организация гарантирует, что хранилище сведений о транзакциях находится за пределами общедоступных сред (например, на платформе хранения, существующей в интрасети организации) и что эти сведения не будут храниться и предоставляться на носителях для хранения данных, доступных через Интернет.

**Идентификатор**. 0947.09y2Organizational.2 — 09.y **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |

### <a name="where-a-trusted-authority-is-used-eg-for-the-purposes-of-issuing-and-maintaining-digital-signatures-andor-digital-certificates-security-is-integrated-and-embedded-throughout-the-entire-end-to-end-certificatesignature-management-process"></a>Если используется доверенный центр (например, для выдачи и обслуживания цифровых подписей и (или) цифровых сертификатов), защита полностью интегрирована и встроена в комплексный процесс управления сертификатами и подписями.

**Идентификатор**. 0948.09y2Organizational.3 — 09.y **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных MySQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |

### <a name="the-protocols-used-for-communications-are-enhanced-to-address-any-new-vulnerability-and-the-updated-versions-of-the-protocols-are-adopted-as-soon-as-possible"></a>Протоколы, используемые для обмена данными, улучшаются для защиты от любой новой уязвимости. Новые версии протоколов будут внедряться как можно скорее.

**Идентификатор**. 0949.09y2Organizational.5 — 09.y **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Приложение API должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb7ddfbdc-1260-477d-91fd-98bd9be789a6) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceApiApp_AuditHTTP_Audit.json) |
|[Приложение-функция должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6d555dd1-86f2-4f1c-8ed7-5abae7c6cbab) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceFunctionApp_AuditHTTP_Audit.json) |
|[В вашем приложении API необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[В вашем приложении-функции необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[В вашем веб-приложении необходимо использовать последнюю версию TLS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Обновите TLS до последней версии. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |
|[Веб-приложение должно быть доступно только по HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |Использование HTTPS обеспечивает проверку подлинности сервера и служб, а также защищает передаваемые данные от перехвата на сетевом уровне. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

## <a name="control-of-operational-software"></a>Управление операционным ПО

### <a name="only-authorized-administrators-are-allowed-to-implement-approved-upgrades-to-software-applications-and-program-libraries-based-on-business-requirements-and-the-security-implications-of-the-release"></a>Только уполномоченные администраторы могут внедрять утвержденные обновления программного обеспечения, приложений и библиотек программ, учитывая бизнес-требования и аспекты безопасности для выпуска.

**Идентификатор**. 0605.10h1System.12 — 10.h **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Уязвимости конфигурации безопасности на ваших компьютерах должны быть устранены](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1e5fd5d-3e4c-4ce1-8661-7d1873ae6b15) |Серверы, не соответствующие заданным базовым показателям, будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OSVulnerabilities_Audit.json) |
|[Компьютеры с Windows должны соответствовать требованиям для категории "Параметры безопасности — аудит"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F33936777-f2ac-45aa-82ec-07958ec9ade4) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Параметры безопасности — аудит" для принудительного применения подкатегории политики аудита и завершения работы в случае, если не удается зарегистрировать аудиты безопасности. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SecurityOptionsAudit_AINE.json) |
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — управление учетными записями"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F94d9aca8-3757-46df-aa51-f218c5f11954) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — управление учетными записями" для аудита приложений, безопасности, управления группами пользователей, а также других событий управления. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesAccountManagement_AINE.json) |

### <a name="applications-and-operating-systems-are-successfully-tested-for-usability-security-and-impact-prior-to-production"></a>Приложения и операционные системы перед выпуском успешно протестированы на удобство использования, безопасность и стороннее воздействие.

**Идентификатор**. 0606.10h2System.1 — 10.h **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо устранить уязвимости в конфигурациях безопасности контейнера.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe8cbc669-f12d-49eb-93e7-9273119e9933) |Аудит уязвимостей в конфигурации безопасности на компьютерах с установленным Docker и отображение результатов в качестве рекомендаций в Центре безопасности Azure. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ContainerBenchmark_Audit.json) |

### <a name="the-organization-uses-its-configuration-control-program-to-maintain-control-of-all-implemented-software-and-its-system-documentation-and-archive-prior-versions-of-implemented-software-and-associated-system-documentation"></a>Организация использует программу управления конфигурацией, чтобы поддерживать управление всем внедренным программным обеспечением и его системной документацией, а также архивировать предыдущие версии внедренного программного обеспечения и соответствующую документацию системы.

**Идентификатор**. 0607.10h2System.23 — 10.h **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На компьютерах должны быть включены адаптивные элементы управления приложениями для определения безопасных приложений](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Включите элементы управления приложениями, чтобы определить список проверенных безопасных приложений, выполняющихся на ваших компьютерах, и настроить получение оповещений о запуске других приложений. Это поможет защитить компьютеры от вредоносных программ. Чтобы упростить процесс настройки и обслуживания правил, Центр безопасности использует машинное обучение для анализа приложений, выполняющихся на каждом компьютере, и предлагает список проверенных безопасных приложений. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |
|[Уязвимости в конфигурации безопасности в масштабируемом наборе виртуальных машин должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3c735d8a-a4ba-4a3a-b7cf-db7754cf57f4) |Аудит наличия уязвимостей ОС в ваших масштабируемых наборах виртуальных машин для защиты их от атак. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssOSVulnerabilities_Audit.json) |

## <a name="change-control-procedures"></a>Процедуры управления изменениями

### <a name="managers-responsible-for-application-systems-are-also-responsible-for-the-strict-control-security-of-the-project-or-support-environment-and-ensure-that-all-proposed-system-changes-are-reviewed-to-check-that-they-do-not-compromise-the-security-of-either-the-system-or-the-operating-environment"></a>Менеджеры, ответственные за системы приложений, также отвечают за безопасность проекта или среды поддержки и управление ими. Они обеспечивают проверку всех предлагаемых системных изменений, чтобы эти изменения не нарушили безопасность системы или операционной среды.

**Идентификатор**. 0635.10k1Organizational.12 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="the-organization-formally-addresses-purpose-scope-roles-responsibilities-management-commitment-coordination-among-organizational-entities-and-compliance-for-configuration-management-eg-through-policies-standards-processes"></a>Организация формально определяет цель, область, роли, обязанности, обязательства по управлению, координацию между организационными подразделениями и меры для обеспечения соответствия требованиям к управлению конфигурацией (например, с помощью политик, стандартов, процессов).

**Идентификатор**. 0636.10k2Organizational.1 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="the-organization-has-developed-documented-and-implemented-a-configuration-management-plan-for-the-information-system"></a>Организация разработала, задокументировала и реализовала план управления конфигурацией для информационной системы.

**Идентификатор**. 0637.10k2Organizational.2 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="changes-are-formally-controlled-documented-and-enforced-in-order-to-minimize-the-corruption-of-information-systems"></a>Изменения формально контролируются, документируются и применяются для снижения риска повреждения информационных систем.

**Идентификатор**. 0638.10k2Organizational.34569 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="installation-checklists-and-vulnerability-scans-are-used-to-validate-the-configuration-of-servers-workstations-devices-and-appliances-and-ensure-the-configuration-meets-minimum-standards"></a>Контрольные списки установки и сканирования уязвимостей используются для проверки конфигурации серверов, рабочих станций, устройств и приборов, а также для обеспечения соответствия конфигурации минимальным стандартам.

**Идентификатор**. 0639.10k2Organizational.78 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="where-development-is-outsourced-change-control-procedures-to-address-security-are-included-in-the-contracts-and-specifically-require-the-developer-to-track-security-flaws-and-flaw-resolution-within-the-system-component-or-service-and-report-findings-to-organization-defined-personnel-or-roles"></a>Если разработка передается на аутсорсинг, процедуры управления изменениями для обеспечения безопасности включены в контракты, а разработчик обязан отслеживать недостатки системы безопасности и контролировать их устранение в системе, компоненте или службе, а также сообщать о результатах определенному организацией персоналу или ролям.

**Идентификатор**. 0640.10k2Organizational.1012 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="the-organization-does-not-use-automated-updates-on-critical-systems"></a>Организация не использует автоматические обновления в критически важных системах.

**Идентификатор**. 0641.10k2Organizational.11 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="the-organization-develops-documents-and-maintains-under-configuration-control-a-current-baseline-configuration-of-the-information-system-and-reviews-and-updates-the-baseline-as-required"></a>Организация разрабатывает, документирует и сохраняет в рамках управления конфигурацией текущую базовую конфигурацию информационной системы, а также проверяет и обновляет базовые показатели по мере необходимости.

**Идентификатор**. 0642.10k3Organizational.12 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="the-organization-i-establishes-and-documents-mandatory-configuration-settings-for-information-technology-products-employed-within-the-information-system-using-the-latest-security-configuration-baselines-ii-identifies-documents-and-approves-exceptions-from-the-mandatory-established-configuration-settings-for-individual-components-based-on-explicit-operational-requirements-and-iii-monitors-and-controls-changes-to-the-configuration-settings-in-accordance-with-organizational-policies-and-procedures"></a>Организация 1) устанавливает и документирует обязательные параметры конфигурации для продуктов в сфере информационных технологий, используемых в информационной системе, применяя последние конфигурационные базы для безопасности; 2) определяет, документирует и утверждает исключения из обязательных установленных параметров конфигурации для отдельных компонентов на основе явных операционных требований; 3) отслеживает изменения параметров конфигурации и управляет ими в соответствии с политиками и процедурами организации.

**Идентификатор**. 0643.10k3Organizational.3 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

### <a name="the-organization-employs-automated-mechanisms-to-i-centrally-manage-apply-and-verify-configuration-settings-ii-respond-to-unauthorized-changes-to-network-and-system-security-related-configuration-settings-and-iii-enforce-access-restrictions-and-auditing-of-the-enforcement-actions"></a>Организация применяет автоматические механизмы для 1) централизованных управления, применения и проверки параметров конфигурации; 2) реагирования на несанкционированные изменения параметров конфигурации, связанных с безопасностью сети и системы; 3) принудительного ограничения доступа и аудита принудительных действий.

**Идентификатор**. 0644.10k3Organizational.4 — 10.k **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Политики аудита системы — подробное отслеживание"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F58383b73-94a9-4414-b382-4146eb02611b) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Политики аудита системы — подробное отслеживание" для аудита API защиты данных, создания и завершения процессов, событий RPC и PNP-действий. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SystemAuditPoliciesDetailedTracking_AINE.json) |

## <a name="control-of-technical-vulnerabilities"></a>Управление техническими уязвимостями

### <a name="technical-vulnerabilities-are-identified-evaluated-for-risk-and-corrected-in-a-timely-manner"></a>Технические уязвимости выявляются, оцениваются с точки зрения риска и своевременно исправляются.

**Идентификатор**. 0709.10m1Organizational.1 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить решение для оценки уязвимостей на виртуальных машинах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F501541f7-f7e7-4cd6-868c-4190fdad3ac9) |Осуществляет аудит виртуальных машин, чтобы определить, работает ли на них поддерживаемое решение для оценки уязвимостей. Основным компонентом каждой программы безопасности и защиты от киберрисков является идентификация и анализ уязвимостей. Ценовая категория "Стандартный" Центра безопасности Azure предоставляет возможность выполнять проверку уязвимостей виртуальных машин без дополнительных затрат. Кроме того, Центр безопасности может автоматически развернуть этот инструмент. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ServerVulnerabilityAssessment_Audit.json) |
|[Необходимо устранить уязвимости в конфигурациях безопасности контейнера.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe8cbc669-f12d-49eb-93e7-9273119e9933) |Аудит уязвимостей в конфигурации безопасности на компьютерах с установленным Docker и отображение результатов в качестве рекомендаций в Центре безопасности Azure. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ContainerBenchmark_Audit.json) |
|[Уязвимости конфигурации безопасности на ваших компьютерах должны быть устранены](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1e5fd5d-3e4c-4ce1-8661-7d1873ae6b15) |Серверы, не соответствующие заданным базовым показателям, будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OSVulnerabilities_Audit.json) |
|[Уязвимости в конфигурации безопасности в масштабируемом наборе виртуальных машин должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3c735d8a-a4ba-4a3a-b7cf-db7754cf57f4) |Аудит наличия уязвимостей ОС в ваших масштабируемых наборах виртуальных машин для защиты их от атак. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssOSVulnerabilities_Audit.json) |
|[Уязвимости в базах данных SQL должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffeedbf84-6b99-488c-acc2-71c829aa5ffc) |Мониторинг результатов оценки уязвимостей и рекомендации по исправлению уязвимостей баз данных. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_SQLDbVulnerabilities_Audit.json) |
|[В Управляемых экземплярах SQL должна быть включена оценка уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1b7aa243-30e4-4c9e-bca8-d0d3022b634a) |Проверка каждого Управляемого экземпляра SQL с отключенной регулярной оценкой уязвимостей. Решение "Оценка уязвимостей" может обнаруживать, отслеживать и помогать исправлять потенциальные уязвимости баз данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnManagedInstance_Audit.json) |
|[На серверах SQL Server должна быть включена оценка уязвимости](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef2a8f2a-b3d9-49cd-a8a8-9a3aaaf647d9) |Аудит серверов Azure SQL, у которых отключена регулярная оценка уязвимостей. Решение "Оценка уязвимостей" может обнаруживать, отслеживать и помогать исправлять потенциальные уязвимости баз данных. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnServer_Audit.json) |
|[Компьютеры с Windows должны соответствовать требованиям для категории "Параметры безопасности — сервер для сетей Майкрософт"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcaf2d518-f029-4f6b-833b-d7081702f253) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Параметры безопасности — сервер для сетей Майкрософт" для отключения сервера SMB версии 1. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SecurityOptionsMicrosoftNetworkServer_AINE.json) |

### <a name="a-hardened-configuration-standard-exists-for-all-system-and-network-components"></a>Для всех системных и сетевых компонентов существует стандарт защищенной конфигурации.

**Идентификатор**. 0710.10m2Organizational.1 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемых экземплярах SQL должна быть включена оценка уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1b7aa243-30e4-4c9e-bca8-d0d3022b634a) |Проверка каждого Управляемого экземпляра SQL с отключенной регулярной оценкой уязвимостей. Решение "Оценка уязвимостей" может обнаруживать, отслеживать и помогать исправлять потенциальные уязвимости баз данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnManagedInstance_Audit.json) |

### <a name="a-technical-vulnerability-management-program-is-in-place-to-monitor-assess-rank-and-remediate-vulnerabilities-identified-in-systems"></a>Для отслеживания, оценки, ранжирования и устранения уязвимостей в системах используется техническая программа управления уязвимостями.

**Идентификатор**. 0711.10m2Organizational.23 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо включить решение для оценки уязвимостей на виртуальных машинах](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F501541f7-f7e7-4cd6-868c-4190fdad3ac9) |Осуществляет аудит виртуальных машин, чтобы определить, работает ли на них поддерживаемое решение для оценки уязвимостей. Основным компонентом каждой программы безопасности и защиты от киберрисков является идентификация и анализ уязвимостей. Ценовая категория "Стандартный" Центра безопасности Azure предоставляет возможность выполнять проверку уязвимостей виртуальных машин без дополнительных затрат. Кроме того, Центр безопасности может автоматически развернуть этот инструмент. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ServerVulnerabilityAssessment_Audit.json) |

### <a name="patches-are-tested-and-evaluated-before-they-are-installed"></a>Исправления проверяются и оцениваются до их установки.

**Идентификатор**. 0713.10m2Organizational.5 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Уязвимости конфигурации безопасности на ваших компьютерах должны быть устранены](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1e5fd5d-3e4c-4ce1-8661-7d1873ae6b15) |Серверы, не соответствующие заданным базовым показателям, будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OSVulnerabilities_Audit.json) |

### <a name="the-technical-vulnerability-management-program-is-evaluated-on-a-quarterly-basis"></a>Техническая программа управления уязвимостями оценивается ежеквартально.

**Идентификатор**. 0714.10m2Organizational.7 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Уязвимости в конфигурации безопасности в масштабируемом наборе виртуальных машин должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3c735d8a-a4ba-4a3a-b7cf-db7754cf57f4) |Аудит наличия уязвимостей ОС в ваших масштабируемых наборах виртуальных машин для защиты их от атак. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssOSVulnerabilities_Audit.json) |

### <a name="systems-are-appropriately-hardened-eg-configured-with-only-necessary-and-secure-services-ports-and-protocols-enabled"></a>Системы защищены соответствующим образом (например, путем настройки в них только необходимых и безопасных служб, портов и протоколов).

**Идентификатор**. 0715.10m2Organizational.8 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Необходимо устранить уязвимости в конфигурациях безопасности контейнера.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe8cbc669-f12d-49eb-93e7-9273119e9933) |Аудит уязвимостей в конфигурации безопасности на компьютерах с установленным Docker и отображение результатов в качестве рекомендаций в Центре безопасности Azure. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_ContainerBenchmark_Audit.json) |

### <a name="the-organization-conducts-an-enterprise-security-posture-review-as-needed-but-no-less-than-once-within-every-three-hundred-sixty-five-365-days-in-accordance-with-organizational-is-procedures"></a>Организация проводит проверку уровня безопасности предприятия по мере необходимости, но не менее одного раза в 365 (триста шестьдесят пять) дней в соответствии со своими процедурами.

**Идентификатор**. 0716.10m3Organizational.1 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Уязвимости в базах данных SQL должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffeedbf84-6b99-488c-acc2-71c829aa5ffc) |Мониторинг результатов оценки уязвимостей и рекомендации по исправлению уязвимостей баз данных. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_SQLDbVulnerabilities_Audit.json) |

### <a name="vulnerability-scanning-tools-include-the-capability-to-readily-update-the-information-system-vulnerabilities-scanned"></a>Инструменты проверки на уязвимости включают возможность быстрого обновления для защиты от выявленных уязвимостей информационной системы.

**Идентификатор**. 0717.10m3Organizational.2 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Уязвимости в конфигурации безопасности в масштабируемом наборе виртуальных машин должны быть устранены.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3c735d8a-a4ba-4a3a-b7cf-db7754cf57f4) |Аудит наличия уязвимостей ОС в ваших масштабируемых наборах виртуальных машин для защиты их от атак. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssOSVulnerabilities_Audit.json) |

### <a name="the-organization-scans-for-vulnerabilities-in-the-information-system-and-hosted-applications-to-determine-the-state-of-flaw-remediation-monthly-automatically-and-again-manually-or-automatically-when-new-vulnerabilities-potentially-affecting-the-systems-and-networked-environments-are-identified-and-reported"></a>Организация выполняет проверку на уязвимости в информационной системе и размещенных приложениях для определения состояния их исправления ежемесячно (автоматически) или повторно (вручную или автоматически) при обнаружении новых уязвимостей, которые потенциально могут влиять на системы и сетевые среды, и получении сведений о них.

**Идентификатор**. 0718.10m3Organizational.34 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Уязвимости конфигурации безопасности на ваших компьютерах должны быть устранены](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1e5fd5d-3e4c-4ce1-8661-7d1873ae6b15) |Серверы, не соответствующие заданным базовым показателям, будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OSVulnerabilities_Audit.json) |

### <a name="the-organization-updates-the-list-of-information-system-vulnerabilities-scanned-within-every-thirty-30-days-or-when-new-vulnerabilities-are-identified-and-reported"></a>Организация обновляет список выявленных уязвимостей информационной системы, каждые 30 (тридцать) дней или при обнаружении новых уязвимостей и получении сведений о них.

**Идентификатор**. 0719.10m3Organizational.5 — 10.m **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В Управляемых экземплярах SQL должна быть включена оценка уязвимостей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1b7aa243-30e4-4c9e-bca8-d0d3022b634a) |Проверка каждого Управляемого экземпляра SQL с отключенной регулярной оценкой уязвимостей. Решение "Оценка уязвимостей" может обнаруживать, отслеживать и помогать исправлять потенциальные уязвимости баз данных. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnManagedInstance_Audit.json) |

## <a name="business-continuity-and-risk-assessment"></a>Непрерывность бизнес-процессов и оценка рисков

### <a name="the-organization-identifies-the-critical-business-processes-requiring-business-continuity"></a>Организация определяет критически важные бизнес-процессы, которые требуют непрерывности.

**Идентификатор**. 1634.12b1Organizational.1 — 12.b **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит виртуальных машин без аварийного восстановления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0015ea4d-51ff-4ce3-8d8c-f3f8f0179a56) |Аудит виртуальных машин, на которых не настроено аварийное восстановление. Дополнительные сведения об аварийном восстановлении см. здесь: [https://aka.ms/asr-doc](https://aka.ms/asr-doc). |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/RecoveryServices_DisasterRecovery_Audit.json) |

### <a name="information-security-aspects-of-business-continuity-are-i-based-on-identifying-events-or-sequence-of-events-that-can-cause-interruptions-to-the-organizations-critical-business-processes-eg-equipment-failure-human-errors-theft-fire-natural-disasters-acts-of-terrorism-ii-followed-by-a-risk-assessment-to-determine-the-probability-and-impact-of-such-interruptions-in-terms-of-time-damage-scale-and-recovery-period-iii-based-on-the-results-of-the-risk-assessment-a-business-continuity-strategy-is-developed-to-identify-the-overall-approach-to-business-continuity-and-iv-once-this-strategy-has-been-created-endorsement-is-provided-by-management-and-a-plan-created-and-endorsed-to-implement-this-strategy"></a>Информационные аспекты безопасности для обеспечения непрерывности бизнес-процессов предусматривают: 1) определение этих аспектов путем выявления событий (или последовательности событий), которые могут вызвать прерывания критически важных бизнес-процессов организации (например, сбоя оборудования, человеческих ошибок, кражи, пожара, стихийных бедствий, террористических актов); 2) проведение оценки рисков с целью определения вероятности и влияния таких прерываний в работе с точки зрения времени, масштабов ущерба и периода восстановления; 3) разработку общей стратегии обеспечения непрерывности бизнес-процессов на основе результатов оценки рисков; 4) одобрение этой стратегии руководством после ее создания, а также составление плана ее реализации и одобрение этого плана.

**Идентификатор**. 1635.12b1Organizational.2 — 12.b **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для управляемого модуля HSM для Azure Key Vault должна быть включена защита от очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc39ba22d-4428-4149-b981-70acb31fc383) |Удаление управляемого модуля HSM для Azure Key Vault злоумышленником может привести к необратимой потере данных. Потенциальный злоумышленник в вашей организации может удалить и очистить управляемый модуль HSM для Azure Key Vault. Защита от очистки позволяет оградить вас от атак злоумышленников. Для этого применяется обязательный период хранения данных для управляемого модуля HSM для Azure Key Vault, который был обратимо удален. Ни корпорация Майкрософт, ни пользователи вашей организации не смогут очистить управляемый модуль HSM для Azure Key Vault во время периода хранения при обратимом удалении. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/ManagedHsm_Recoverable_Audit.json) |
|[В хранилищах ключей должна быть включена защита от очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |Удаление хранилища ключей злоумышленником может привести к необратимой потере данных. Потенциальный злоумышленник в вашей организации может удалить и очистить хранилища ключей. Защита от очистки позволяет оградить вас от атак злоумышленников. Для этого применяется обязательный период хранения данных для хранилищ ключей, которые были обратимо удалены. Ни корпорация Майкрософт, ни пользователи вашей организации не смогут очистить хранилища ключей во время периода хранения при обратимом удалении. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |

### <a name="business-impact-analysis-are-used-to-evaluate-the-consequences-of-disasters-security-failures-loss-of-service-and-service-availability"></a>Для оценки последствий аварийной ситуации, сбоев системы безопасности, службы или ее доступности используется анализ влияния на бизнес-процессы.

**Идентификатор**. 1637.12b2Organizational.2 — 12.b **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Компьютеры с Windows должны соответствовать требованиям для категории "Параметры безопасности — консоль восстановления"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff71be03e-e25b-4d0f-b8bc-9b3e309b66c0) |На компьютерах с Windows должны быть заданы параметры групповой политики в категории "Параметры безопасности — консоль восстановления" для разрешения копирования дискет и доступа ко всем дискам и папкам. Эта политика требует развертывания необходимых компонентов гостевой конфигурации в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_SecurityOptionsRecoveryconsole_AINE.json) |

### <a name="business-continuity-risk-assessments-i-are-carried-out-annually-with-full-involvement-from-owners-of-business-resources-and-processes-ii-consider-all-business-processes-and-is-not-limited-to-the-information-assets-but-includes-the-results-specific-to-information-security-and-iii-identifies-quantifies-and-prioritizes-risks-against-key-business-objectives-and-criteria-relevant-to-the-organization-including-critical-resources-impacts-of-disruptions-allowable-outage-times-and-recovery-priorities"></a>Оценки рисков для непрерывности бизнес-процессов 1) выполняются ежегодно с участием владельцев бизнес-ресурсов и процессов; 2) выполняются для всех бизнес-процессов, не ограничиваются информационными ресурсами, но включают результаты, относящиеся к информационной безопасности; 3) позволяют определить и количественно оценить риски, а также назначить их приоритеты с точки зрения ключевых бизнес-целей и критерии организации, включая критически важные ресурсы, влияние прерываний в работе, допустимое время простоя и приоритеты восстановления.

**Идентификатор**. 1638.12b2Organizational.345 — 12.b **Ответственность**: Customer

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит виртуальных машин без аварийного восстановления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0015ea4d-51ff-4ce3-8d8c-f3f8f0179a56) |Аудит виртуальных машин, на которых не настроено аварийное восстановление. Дополнительные сведения об аварийном восстановлении см. здесь: [https://aka.ms/asr-doc](https://aka.ms/asr-doc). |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/RecoveryServices_DisasterRecovery_Audit.json) |

> [!NOTE]
> Возможность использования отдельных определений Политики Azure может отличаться в Azure для государственных организаций и других национальных облаках.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные статьи о Политике Azure:

- Обзор [соответствия нормативным требованиям](../concepts/regulatory-compliance.md).
- См. [структуру определения инициативы](../concepts/initiative-definition-structure.md).
- См. другие [примеры шаблонов для Политики Azure](./index.md).
- Изучите [сведения о действии политик](../concepts/effects.md).
- Узнайте, как [исправлять несоответствующие ресурсы](../how-to/remediate-resources.md).

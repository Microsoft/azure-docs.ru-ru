---
title: Тестирование безопасности Azure v2 — привилегированный доступ
description: Привилегированный доступ к системе безопасности Azure версии 2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 29d4acc50ed872c37268a0b21c3e34837249a026
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102035422"
---
# <a name="security-control-v2-privileged-access"></a>Управление безопасностью v2: привилегированный доступ

Привилегированный доступ охватывает элементы управления для защиты привилегированного доступа к вашему клиенту и ресурсам Azure. Сюда входит ряд элементов управления для защиты административной модели, административных учетных записей и рабочих станций с привилегированным доступом к намеренным и случайным рискам.

Чтобы просмотреть подходящую встроенную политику Azure, ознакомьтесь со статьей [сведения о встроенной инициативе по обеспечению безопасности в Azure: привилегированный доступ](../../governance/policy/samples/azure-security-benchmark.md#privileged-access)

## <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1: защита и ограничение пользователей с высоким уровнем привилегий

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-1 | 4,3, 4,8 | AC-2 |

Ограничьте число учетных записей пользователей с высоким уровнем привилегий и защитите эти учетные записи на уровне повышенного уровня. Наиболее важными встроенными ролями в Azure AD являются глобальные администраторы и администратор привилегированных ролей, поскольку пользователи, которым назначены эти две роли, могут делегировать роли администратора. С этими привилегиями пользователи могут напрямую или косвенно читать и изменять каждый ресурс в среде Azure:

- Глобальный администратор. пользователи с этой ролью имеют доступ ко всем функциям администрирования в Azure AD, а также к службам, использующим удостоверения Azure AD.

- Администратор привилегированных ролей. пользователи с этой ролью могут управлять назначениями ролей в Azure AD, а также в Azure AD Privileged Identity Management (PIM). Кроме того, эта роль позволяет управлять всеми аспектами PIM и административными единицами.

Примечание. у вас могут быть другие критически важные роли, которыми необходимо управлять при использовании пользовательских ролей с назначенными привилегированными разрешениями. Кроме того, может потребоваться применить аналогичные элементы управления к учетной записи администратора критических бизнес-ресурсов.

Вы можете разрешить пользователям привилегированный JIT-доступ к ресурсам Azure и Azure AD с помощью Azure AD Privileged Identity Management (PIM). JIT-доступ предоставляет временные разрешения на выполнение привилегированных задач только в том случае, если это необходимо пользователям. PIM также может создавать оповещения безопасности при обнаружении подозрительных или небезопасных действий в организации Azure AD.

- [Разрешения роли администратора в Azure Active Directory](../../active-directory/roles/permissions-reference.md)

- [Использование оповещений системы безопасности в рамках управления привилегированными пользователями Azure](../../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md)

- [Защита привилегированного доступа для гибридных и облачных развертываний в Azure AD](../../active-directory/roles/security-planning.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Операции безопасности](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

## <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2: ограничение административного доступа к системам, критически важным для бизнеса

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-2 | 13,2, 2,10 | AC-2, SC-3, SC-7 |

Изолируйте доступ к критически важным для бизнеса системам, указав учетные записи, которым предоставляется привилегированный доступ к подпискам и группам управления, в которых они находятся. Убедитесь, что вы также ограничиваете доступ к системам управления, идентификации и безопасности, которые имеют административный доступ к важнейшим ресурсам предприятия, таким как контроллеры домен Active Directory (DCs), средства безопасности и средства управления системой с агентами, установленными в критически важных для бизнеса системах. Злоумышленники, которые нарушают эти системы управления и безопасности, могут немедленно веапонизе их для взлома важных для бизнеса ресурсов. 

Все типы элементов управления доступом должны быть согласованы с стратегией сегментации предприятия, чтобы обеспечить постоянный контроль доступа. 

Обязательно назначьте отдельные привилегированные учетные записи, отличающиеся от стандартных учетных записей пользователей, используемых для задач электронной почты, обзора и повышения производительности.

- [Компоненты и эталонная модель Azure](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Доступ к группе управления](../../governance/management-groups/overview.md#management-group-access)

- [Администраторы подписки Azure](../../cost-management-billing/manage/add-change-subscription-administrator.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

## <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3: регулярная проверка и сверка доступа пользователей

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-3 | 4,1, 16,9, 16,10 | AC-2 |

Регулярно просматривайте учетные записи пользователей и назначение доступа, чтобы убедиться, что учетные записи и их уровень доступа действительны. Вы можете использовать проверки доступа Azure AD для просмотра членства в группах, доступа к корпоративным приложениям и назначения ролей. Служба отчетов Azure AD может предоставлять журналы для облегчения поиска устаревших учетных записей. Можно также использовать Azure AD Privileged Identity Management для создания рабочего процесса отчета по проверке доступа, который упрощает процесс проверки.
Кроме того, управление привилегированными пользователями Azure можно настроить на оповещение при создании чрезмерного числа учетных записей администраторов и определении устаревших или неправильно настроенных учетных записей администраторов. 

Примечание. Некоторые службы Azure поддерживают локальных пользователей и роли, которые не управляются с помощью Azure AD. Эти пользователи необходимо управлять по отдельности.

- [Создание проверки доступа для ролей ресурсов Azure в управление привилегированными пользователями (PIM)](../../active-directory/privileged-identity-management/pim-resource-roles-start-access-review.md)

- [Использование проверок доступа для идентификации Azure AD](../../active-directory/governance/access-reviews-overview.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Безопасность приложений и DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4: настройка аварийного доступа в Azure AD

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-4 | 16 | AC-2, CP-2 |

Чтобы предотвратить случайную блокировку вашей организации Azure AD, настройте учетную запись аварийного доступа для доступа, если обычные административные учетные записи использовать нельзя. Учетные записи аварийного доступа обычно имеют высокий уровень привилегий и не должны назначаться конкретным пользователям. Учетные записи для аварийного доступа используются только в непредвиденных ситуациях, в которых невозможно использовать обычные учетные записи администратора.
Следует убедиться, что учетные данные (например, пароль, сертификат или смарт-карта) для учетных записей аварийного доступа защищены и известны только тем лицам, которые имеют право использовать их только в экстренной ситуации.

- [Управление учетными записями для аварийного доступа в Azure AD](../../active-directory/roles/security-emergency-access.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Безопасность приложений и DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Операции безопасности (SecOps)](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

## <a name="pa-5-automate-entitlement-management"></a>PA-5: Автоматизация управления назначениями

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-5 | 16 | AC-2, AC-5, PM-10 |

Используйте функции управления обслуживанием Azure AD для автоматизации рабочих процессов запросов на доступ, включая назначения доступа, обзоры и срок действия. Поддерживается также два или несколько промежуточных утверждений.
- [Что такое проверки доступа Azure AD](../../active-directory/governance/access-reviews-overview.md) 

- [Что такое управление назначениями Azure AD](../../active-directory/governance/entitlement-management-overview.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Безопасность приложений и DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="pa-6-use-privileged-access-workstations"></a>PA-6: использование рабочих станций с привилегированным доступом

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-6 | 4,6, 11,6, 12,12 | AC-2, SC-3, SC-7 |

Защищенные изолированные рабочие станции критически важны для защиты конфиденциальных ролей, таких как администратор, разработчик и оператор критической службы. Используйте высокозащищенные рабочие станции пользователей и (или) Azure бастиона для административных задач. Чтобы развернуть защищенную и управляемую рабочую станцию для административных задач, используйте Azure Active Directory, Advanced Threat Protection в Microsoft Defender и (или) Microsoft Intune. Защищенные рабочие станции можно централизованно управлять, чтобы обеспечить безопасную настройку, включая надежную проверку подлинности, базовые показатели программного обеспечения и оборудования, а также ограниченный логический и сетевой доступ. 

- [Общие сведения о рабочих станциях с привилегированным доступом](/security/compass/privileged-access-deployment)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность приложений и DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Операции безопасности (SecOps)](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

## <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7. Использование Just Enough Administration (принцип предоставления наименьших прав)

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-7 | 14,6 | AC-2, AC-3, SC-3 |

Управление доступом на основе ролей Azure (Azure RBAC) позволяет управлять доступом к ресурсам Azure с помощью назначений ролей. Эти роли можно назначить пользователям, субъектам групповых служб и управляемым удостоверениям. Существуют предварительно определенные встроенные роли для определенных ресурсов, и эти роли могут быть учтены или запрошены с помощью таких средств, как Azure CLI, Azure PowerShell и портал Azure. Привилегии, назначаемые ресурсам через Azure RBAC, всегда должны быть ограничены всеми требованиями ролей. Ограниченные привилегии дополняют JIT-подход Azure AD Privileged Identity Management (PIM), и эти привилегии следует периодически проверять.

Используйте встроенные роли для выделения разрешений и при необходимости создавайте пользовательские роли.

- [Что такое управление доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md)

- [Настройка Azure RBAC](../../role-based-access-control/role-assignments-portal.md)

- [Использование проверок доступа для идентификации Azure AD](../../active-directory/governance/access-reviews-overview.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность приложений и DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

## <a name="pa-8-choose-approval-process-for-microsoft-support"></a>PA-8: выберите процесс утверждения для службы поддержки Майкрософт 

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| PA-8 | 16 | AC-2 (AC-3), AC-4 |

В сценариях поддержки, в которых корпорация Майкрософт должна получить доступ к данным клиентов, защищенное хранилище предоставляет возможность явно просматривать и утверждать или отклонять каждый запрос на доступ к данным клиентов.

- [Общие сведения о защищенное хранилище](../fundamentals/customer-lockbox-overview.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность приложений и DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management) 

- [Управление удостоверениями и ключами](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)
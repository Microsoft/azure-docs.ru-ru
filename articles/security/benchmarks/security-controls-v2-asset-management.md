---
title: Тестирование безопасности Azure v2 — управление ресурсами
description: Управление ресурсами производительности Azure с производительностью 2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: fbeb88b6aa542666481458fde97d7c63f467fa30
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102051537"
---
# <a name="security-control-v2-asset-management"></a>Управление безопасностью версии 2: Управление активами

Управление активами охватывает элементы управления, обеспечивающие видимость и управление безопасностью ресурсов Azure. Сюда входят рекомендации по разрешениям для персонала по обеспечению безопасности, доступу к инвентаризации активов и управлению утверждениями для служб и ресурсов (инвентаризации, трассировки и исправления).

Чтобы просмотреть подходящую встроенную политику Azure, ознакомьтесь с [подробными сведениями о встроенной инициативе по обеспечению безопасности в Azure: Сетевая безопасность](../../governance/policy/samples/azure-security-benchmark.md#asset-management)

## <a name="am-1-ensure-security-team-has-visibility-into-risks-for-assets"></a>AM-1: обеспечение контроля над рисками для ресурсов группой безопасности

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| AM – 1 | 1,1, 1,2 | CM-8, PM-5 |

Убедитесь, что группам безопасности предоставлены разрешения читателя по безопасности в клиенте и подписках Azure, чтобы они могли отслеживать риски безопасности с помощью центра безопасности Azure.

В зависимости от структуры обязанностей группы безопасности, наблюдение за угрозами безопасности может быть обязанностью централизованной группы безопасности или локальной группы. С другой стороны, сведения о безопасности и угрозах безопасности всегда должны централизованно располагаться внутри организации. 

Разрешения читателя сведений о безопасности можно расширить для всего клиента (корневой группы управления) или ограничить до определенной группы управления или конкретных подписок. 

Примечание. Для наблюдения за рабочими нагрузками и службами могут потребоваться дополнительные разрешения.

- [Общие сведения о роли читателя сведений о безопасности](../../role-based-access-control/built-in-roles.md#security-reader)

- [Общие сведения о группах управления Azure](../../governance/management-groups/overview.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="am-2-ensure-security-team-has-access-to-asset-inventory-and-metadata"></a>AM-2: убедитесь, что группа безопасности имеет доступ к инвентаризации и метаданным активов.

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| AM – 2 | 1,1, 1,2, 1,4, 1,5, 9,1, 12,1 | CM-8, PM-5 |

Убедитесь, что группы безопасности имеют доступ к постоянно обновленной инвентаризации ресурсов в Azure. Группы безопасности часто нуждаются в этих сведениях, чтобы оценить потенциальную угрозу новых рисков, а также для использования их в качестве входных данных для непрерывного улучшения безопасности. 

Функция инвентаризации Центра безопасности Azure и граф ресурсов Azure позволяют запрашивать и обнаруживать все ресурсы в подписках, включая службы Azure, приложения и сетевые ресурсы.

Логически упорядочивайте активы в соответствии с классификацией вашей организации с помощью тегов, а также других метаданных в Azure (имя, описание и категория).

- [Как создавать запросы с помощью обозревателя Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

- [Управление инвентаризацией активов в центре безопасности Azure](../../security-center/asset-inventory.md)

- [Дополнительные сведения о разметке ресурсов см. в разделе Описание принятия решений по именованию и маркировке.](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=%2fazure%2fazure-resource-manager%2fmanagement%2ftoc.json)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="am-3-use-only-approved-azure-services"></a>AM-3: использование только утвержденных служб Azure

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| AM-3 | 2.3, 2.4 | CM-7, CM-8 |

Используйте политику Azure для аудита и ограничения служб, которые пользователи могут подготавливать в вашей среде. Используйте Azure Resource Graph для запроса и обнаружения ресурсов в своих подписках. Можно также использовать Azure Monitor, чтобы создать правила для активации оповещений при обнаружении неутвержденной службы.

- [Настройка и управление политикой Azure](../../governance/policy/tutorials/create-and-manage.md)

- [Как отказаться от определенного типа ресурса с помощью Политики Azure](../../governance/policy/samples/index.md)

- [Как создавать запросы с помощью обозревателя Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

## <a name="am-4-ensure-security-of-asset-lifecycle-management"></a>AM-4. Обеспечение безопасности для управления жизненным циклом ресурса

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| AM – 4 | 2,3, 2,4, 2,5 | CM-7, CM-8, CM-10, CM-11 |

Установите или обновите политики безопасности, которые устраняют процессы управления жизненным циклом активов для потенциально высоких изменений. Например, изменение поставщиков удостоверений и доступа, конфиденциальности данных, конфигурации сети и назначения прав администратора.

Удалите ресурсы Azure, если они больше не нужны.

- [Удаление группы ресурсов Azure и ее содержимого](../../azure-resource-manager/management/delete-resource-group.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>AM-5: ограничение возможности пользователей взаимодействовать с Azure Resource Manager

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| AM – 5 | 2,9 | AC-3 |

Используйте условный доступ Azure AD, чтобы ограничить возможность пользователей взаимодействовать с Azure Resource Manager путем настройки "блокировать доступ" для приложения "Управление Microsoft Azure".

- [Настройка условного доступа для блокировки доступа к диспетчеру ресурсов Azure](../../role-based-access-control/conditional-access-azure-management.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="am-6-use-only-approved-applications-in-compute-resources"></a>AM-6. использование только утвержденных приложений в ресурсах вычислений

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| AM-6 | 2,6, 2,7 | AC-3, CM-7, CM-8, CM-10, CM-11 |

Убедитесь, что выполняется только авторизованное программное обеспечение, и все несанкционированные программы блокируются на виртуальных машинах Azure.

Используйте адаптивные элементы управления приложением Azure Security Center (ASC) для обнаружения и создания списка разрешений приложения. Можно также использовать адаптивные элементы управления приложениями ASC, чтобы гарантировать выполнение только авторизованного программного обеспечения и блокирование выполнения всех несанкционированных программ на виртуальных машинах Azure.

Используйте Отслеживание изменений и инвентаризацию службы автоматизации Azure для автоматизации сбора данных инвентаризации с виртуальных машин Windows и Linux. Имя программного обеспечения, версия, издатель и время обновления доступны на портал Azure. Чтобы получить дату установки программного обеспечения и другие сведения, включите диагностику на уровне гостя и направьте журналы событий Windows в Log Analytics рабочую область.

В зависимости от типа сценариев можно использовать конфигурации операционной системы или сторонние ресурсы, чтобы ограничить возможность пользователей выполнять сценарии в ресурсах вычислений Azure.

Кроме того, можно использовать стороннее решение для обнаружения и определения неутвержденного программного обеспечения.

- [Как использовать адаптивные элементы управления приложениями в центре безопасности Azure](../../security-center/security-center-adaptive-application.md)

- [Общие сведения о Отслеживание изменений и инвентаризации службы автоматизации Azure](../../automation/change-tracking/overview.md)

- [Управление выполнением скриптов PowerShell в средах Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

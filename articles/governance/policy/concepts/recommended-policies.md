---
title: Рекомендуемые политики для служб Azure
description: Описывает, как найти и применить рекомендуемые политики для служб Azure, таких как виртуальные машины Azure.
ms.date: 09/02/2020
ms.topic: conceptual
ms.customer: generated
ms.openlocfilehash: 019541f034ce8f0c3728c38d2ae4425308b4e2a1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96350285"
---
# <a name="recommended-policies-for-azure-services"></a>Рекомендуемые политики для служб Azure

Клиенты, которые не знакомы с политикой Azure, часто ищут общие определения политик для управления ресурсами и их контроля. **Рекомендуемые** политики политики Azure предоставляют список общих определений политик, начинающихся с. **Рекомендуемые политики** для поддерживаемых ресурсов внедряются в интерфейс портала для этого ресурса.

Дополнительные встроенные расширения политики Azure см. [в статье встроенные определения политики Azure](../samples/built-in-policies.md).

## <a name="azure-virtual-machines"></a>Виртуальные машины Azure

**Рекомендуемые политики** для [виртуальных машин Azure](../../../virtual-machines/index.yml) находятся на странице **Обзор** виртуальных машин и на вкладке **возможности** . В карточке _политики Azure_ выберите текст "не настроено" или "# Assigned", чтобы открыть боковую область с рекомендуемыми политиками. Все определения политик, которые уже назначены области, членом которой является виртуальная машина, неактивны. Выберите Рекомендуемые политики для применения к этой виртуальной машине и щелкните **назначить политики** , чтобы создать назначение для каждого из них.

По мере того как организация приступит к работе с Организацией [ресурсов и иерархией ресурсов](/azure/cloud-adoption-framework/ready/azure-best-practices/organize-subscriptions), рекомендуется перенести эти назначения политики из одной на каждый ресурс в уровень подписки или [группы управления](../../management-groups/index.yml) .

### <a name="azure-virtual-machines-recommended-policies"></a>Рекомендуемые политики на виртуальных машинах Azure

|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит виртуальных машин без аварийного восстановления](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0015ea4d-51ff-4ce3-8d8c-f3f8f0179a56) |Аудит виртуальных машин, на которых не настроено аварийное восстановление. Дополнительные сведения об аварийном восстановлении см. здесь: [https://aka.ms/asr-doc](../../../site-recovery/index.yml). |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/RecoveryServices_DisasterRecovery_Audit.json) |
|[Аудит виртуальных машин, которые не используют управляемые диски](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F06a78e20-9358-41c9-923c-fb736d382a4d) |Эта политика выполняет аудит виртуальных машин, которые не используют управляемые диски. |аудит |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VMRequireManagedDisk_Audit.json) |
|[Необходимо включить Azure Backup для Виртуальных машин](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Эта политика помогает проводить аудит, если служба Azure Backup включена для всех виртуальных машин. Azure Backup — это экономичное решение для резервного копирования одним щелчком, которое упрощает восстановление данных и является более простым в использовании, чем другие облачные службы резервного копирования. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

## <a name="next-steps"></a>Дальнейшие действия

- См. другие [примеры шаблонов для службы Политика Azure](../samples/index.md).
- Изучите [сведения о действии политик](./effects.md).
- Узнайте, как [исправлять несоответствующие ресурсы](../how-to/remediate-resources.md).
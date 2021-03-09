---
title: Управление безопасностью Azure — безопасная конфигурация
description: Безопасная конфигурация управления безопасностью Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 44604f910b3ee6e6ce1b21c736c6ad1f0443e996
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102500042"
---
# <a name="security-control-secure-configuration"></a>Управление безопасностью: безопасная конфигурация

Установите, реализуйте и активно Управляйте (Отслеживайте, сообщайте об исправьте) конфигурацию безопасности ресурсов Azure, чтобы предотвратить взлом злоумышленниками уязвимых служб и параметров.

## <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1. Установка безопасных конфигураций для всех ресурсов Azure

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.1 | 5.1 | Customer |

Используйте псевдонимы политик Azure для создания настраиваемых политик для аудита или принудительного применения конфигурации ресурсов Azure. Вы также можете использовать встроенные определения политик Azure.

Кроме того, Azure Resource Manager может экспортировать шаблон в нотация объектов JavaScript (JSON), который следует проверить, чтобы убедиться, что конфигурации соответствуют требованиям безопасности Организации или превышают их.

Вы также можете использовать рекомендации из центра безопасности Azure в качестве защищенного шаблона базовой конфигурации для ресурсов Azure.

- [Просмотр доступных псевдонимов политик Azure](/powershell/module/az.resources/get-azpolicyalias)

- [Руководство по Создание политик и управление ими для обеспечения соответствия требованиям](../../governance/policy/tutorials/create-and-manage.md)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../../azure-resource-manager/templates/export-template-portal.md)

- [Рекомендации по безопасности — справочное руководство](../../security-center/recommendations-reference.md)

## <a name="72-establish-secure-operating-system-configurations"></a>7.2. Сохранение безопасных конфигураций для операционных систем

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.2 | 5.1 | Customer |

Используйте рекомендации центра безопасности Azure для сохранения конфигураций безопасности во всех ресурсах вычислений.  Кроме того, можно использовать пользовательские образы операционной системы или настройку состояния службы автоматизации Azure, чтобы установить конфигурацию безопасности операционной системы, требуемой для Организации.

- [Как отслеживать рекомендации центра безопасности Azure](../../security-center/security-center-recommendations.md)

- [Рекомендации по безопасности — справочное руководство](../../security-center/recommendations-reference.md)

- [Обзор настройки состояния службы автоматизации Azure](../../automation/automation-dsc-overview.md)

- [Отправка виртуального жесткого диска и его использование для создания новых виртуальных машин Windows в Azure](../../virtual-machines/windows/upload-generalized-managed.md)

- [Создание виртуальной машины Linux на основе пользовательского диска с помощью Azure CLI](../../virtual-machines/linux/upload-vhd.md)

## <a name="73-maintain-secure-azure-resource-configurations"></a>7.3. Сохранение безопасных конфигураций для ресурсов Azure

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.3 | 5,2 | Customer |

Используйте политику Azure [Deny] и [развернуть, если не существует], чтобы обеспечить безопасность параметров в ресурсах Azure.  Кроме того, вы можете использовать шаблоны Azure Resource Manager для поддержания конфигурации безопасности ресурсов Azure, необходимых вашей организации. 

- [Общие сведения о влиянии политики Azure](../../governance/policy/concepts/effects.md)

- [Создание политик и управление ими для обеспечения соответствия](../../governance/policy/tutorials/create-and-manage.md)

- [Общие сведения о шаблонах Azure Resource Manager](../../azure-resource-manager/templates/overview.md)

## <a name="74-maintain-secure-operating-system-configurations"></a>7.4. Сохранение безопасных конфигураций для операционных систем

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.4 | 5,2 | Shared |

Следуйте рекомендациям центра безопасности Azure, чтобы выполнять оценку уязвимостей в ресурсах вычислений Azure.  Кроме того, вы можете использовать шаблоны Azure Resource Manager, пользовательские образы операционной системы или настройку состояния службы автоматизации Azure для поддержания конфигурации безопасности операционной системы, необходимой для Организации.   Шаблоны виртуальных машин Майкрософт, Объединенные с конфигурацией требуемого состояния службы автоматизации Azure, могут помочь в собрании и обслуживании требований безопасности. 

Кроме того, обратите внимание, что образы виртуальных машин Azure Marketplace, опубликованные корпорацией Майкрософт, управляются и обслуживаются корпорацией Майкрософт. 

- [Реализация рекомендаций по оценке уязвимостей в центре безопасности Azure](../../security-center/deploy-vulnerability-assessment-vm.md)

- [Создание виртуальной машины Azure на основе шаблона Azure Resource Manager](../../virtual-machines/windows/ps-template.md)

- [Обзор настройки состояния службы автоматизации Azure](../../automation/automation-dsc-overview.md)

- [Создание виртуальной машины Windows на портале Azure](../../virtual-machines/windows/quick-create-portal.md)

- [Сведения о том, как скачать шаблон виртуальной машины](/previous-versions/azure/virtual-machines/windows/download-template)

- [Пример сценария для отправки VHD в Azure и создания виртуальной машины](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script)

## <a name="75-securely-store-configuration-of-azure-resources"></a>7.5. Безопасное хранение конфигурации ресурсов Azure

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7,5 % | 5,3 | Customer |

Используйте Azure DevOps для безопасного хранения и управления кодом, например пользовательскими политиками Azure, Azure Resource Manager шаблонами и скриптами настройки требуемого состояния. Чтобы получить доступ к ресурсам, которыми вы управляете в Azure DevOps, вы можете предоставить или отклонить разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps, или Active Directory, если они интегрированы с TFS.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

## <a name="76-securely-store-custom-operating-system-images"></a>7.6. Безопасное хранение пользовательских образов операционной системы

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.6 | 5,3 | Customer |

При использовании пользовательских образов используйте управление доступом на основе ролей Azure (Azure RBAC), чтобы обеспечить доступ к образам только полномочным пользователям. Используя коллекцию общих образов, вы можете делиться своими образами с другими пользователями, участниками службы или группами конструктора приложений в вашей организации.  Для образов контейнеров Храните их в реестре контейнеров Azure и используйте Azure RBAC, чтобы обеспечить доступ к образам только полномочным пользователям.  

- [Общие сведения об Azure RBAC](../../role-based-access-control/rbac-and-directory-admin-roles.md)

- [Общие сведения об Azure RBAC для реестра контейнеров](../../container-registry/container-registry-roles.md)

- [Настройка Azure RBAC](../../role-based-access-control/quickstart-assign-role-user-portal.md)

- [Общие сведения о коллекции общих образов](../../virtual-machines/shared-image-galleries.md)

## <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7,7: развертывание средств управления конфигурацией для ресурсов Azure

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.7 | 5.4 | Customer |

Определение и реализация стандартных конфигураций безопасности для ресурсов Azure с помощью политики Azure. Используйте псевдонимы политик Azure для создания настраиваемых политик для аудита или принудительного применения конфигурации сети ресурсов Azure. Вы также можете использовать встроенные определения политик, связанные с конкретными ресурсами.  Кроме того, вы можете использовать службу автоматизации Azure для развертывания изменений конфигурации.

- [Настройка Политики Azure и управление ею](../../governance/policy/tutorials/create-and-manage.md)

- [Использование псевдонимов](../../governance/policy/concepts/definition-structure.md#aliases)

## <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7,8: развертывание средств управления конфигурацией для операционных систем

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.8 | 5.4 | Customer |

Настройка состояния службы автоматизации Azure — это служба управления конфигурацией для узлов Desired State Configuration (DSC) в любом облачном или локальном центре обработки данных. Вы можете легко переносить компьютеры в облачную среду, присваивать им декларативные конфигурации, а также просматривать отчеты, отражающие соответствие каждого компьютера требуемому состоянию. 

- [Подключение компьютеров для управления с помощью службы "Настройка состояния службы автоматизации Azure"](../../automation/automation-dsc-onboarding.md)

## <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7,9. Реализация автоматического мониторинга конфигурации для ресурсов Azure

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.9 | 5.5 | Customer |

Используйте центр безопасности Azure для выполнения проверок базовых показателей ресурсов Azure.  Кроме того, используйте политику Azure для оповещения и аудита конфигураций ресурсов Azure.

- [Как исправить рекомендации в центре безопасности Azure](../../security-center/security-center-remediate-recommendations.md)

## <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10. Реализация автоматизированного мониторинга конфигурации для операционных систем

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.10 | 5.5 | Customer |

Используйте центр безопасности Azure для выполнения проверок базовых параметров ОС и DOCKER для контейнеров.

- [Общие сведения о рекомендациях центра безопасности Azure в отношении контейнеров](../../security-center/container-security.md)

## <a name="711-manage-azure-secrets-securely"></a>7.11. Безопасное управление секретами Azure

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.11 | Версия 13.1 | Customer |

Используйте Управляемое удостоверение службы в сочетании с Azure Key Vault, чтобы упростить и защитить управление секретами для облачных приложений.

- [Интеграция с управляемыми удостоверениями Azure](../../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Создание Key Vault](../../key-vault/secrets/quick-create-portal.md)

- [Проверка подлинности в Key Vault](../../key-vault/general/authentication.md)

- [Назначение политики доступа Key Vault](../../key-vault/general/assign-access-policy-portal.md)

## <a name="712-manage-identities-securely-and-automatically"></a>7.12. Безопасное и автоматическое управление удостоверениями

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7.12 | 4.1 | Customer |

Используйте управляемые удостоверения для предоставления служб Azure с автоматически управляемым удостоверением в Azure AD. Управляемое удостоверение можно использовать для проверки подлинности в любой службе, которая поддерживает проверку подлинности Azure AD, включая Key Vault, при этом не сохраняя каких-либо учетных данных в коде.

- [Настройка управляемых удостоверений](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

## <a name="713-eliminate-unintended-credential-exposure"></a>7.13. Устранение непреднамеренного раскрытия учетных данных

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 7,13 | 18,1, 18,7 | Customer |

Реализуйте средство проверки учетных данных для обнаружения учетных данных в коде. Сканер учетных данных также рекомендует перемещать обнаруженные учетные данные в более безопасные расположения, такие как Azure Key Vault. 

- [Как настроить сканер учетных данных](https://secdevtools.azurewebsites.net/helpcredscan.html)


## <a name="next-steps"></a>Дальнейшие действия

- См. следующий контроль безопасности:  [Защита от вредоносных программ](security-control-malware-defense.md)
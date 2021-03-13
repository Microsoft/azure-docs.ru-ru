---
title: Службы Azure с поддержкой управляемых удостоверений в Azure AD
description: Список служб, которые поддерживают управляемые удостоверения для ресурсов Azure и аутентификацию Azure AD.
services: active-directory
author: barclayn
ms.author: barclayn
ms.date: 01/28/2021
ms.topic: conceptual
ms.service: active-directory
ms.subservice: msi
manager: daveba
ms.collection: M365-identity-device-management
ms.custom: references_regions
ms.openlocfilehash: 0531305364b3e6f914a6de00614a3f9af4be3531
ms.sourcegitcommit: df1930c9fa3d8f6592f812c42ec611043e817b3b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/13/2021
ms.locfileid: "103418782"
---
# <a name="services-that-support-managed-identities-for-azure-resources"></a>Службы с поддержкой управляемых удостоверений для ресурсов Azure

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Управляемое удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня учетные данные в коде. Мы работаем над интеграцией управляемых удостоверений для ресурсов Azure и аутентификации Azure AD на платформе Azure. Почаще проверяйте наличие обновлений.

> [!NOTE]
> Управляемые удостоверения для ресурсов Azure — это новое название службы "Управляемое удостоверение службы" (MSI).

## <a name="azure-services-that-support-managed-identities-for-azure-resources"></a>Службы Azure с поддержкой управляемых удостоверений для ресурсов Azure

Ниже приведены службы Azure с поддержкой управляемых удостоверений для ресурсов Azure.


### <a name="azure-api-management"></a>Cлужба управления Azure API

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | Preview (Предварительный просмотр) | Preview (Предварительный просмотр) | Недоступно | Preview (Предварительный просмотр) |

Чтобы настроить управляемые удостоверения для Управления API Azure версии 2 (в регионах, где они доступны), см. следующий список.

- [Шаблон Azure Resource Manager](../../api-management/api-management-howto-use-managed-service-identity.md)

### <a name="azure-app-configuration"></a>Конфигурация приложений Azure

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check]  | Недоступно  | ![Доступно][check] |

См. следующий список, чтобы настроить управляемое удостоверение для конфигурации приложения Azure (в регионах, где это возможно):

- [Azure CLI](../../azure-app-configuration/overview-managed-identity.md)

### <a name="azure-app-service"></a>Служба приложений Azure

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check]  | ![Доступно][check]  | ![Доступно][check] |

Чтобы настроить управляемые удостоверения для Службы приложений Azure (в регионах, где они доступны), см. следующий список.

- [Портал Azure](../../app-service/overview-managed-identity.md#using-the-azure-portal)
- [Azure CLI](../../app-service/overview-managed-identity.md#using-the-azure-cli)
- [Azure PowerShell](../../app-service/overview-managed-identity.md#using-azure-powershell)
- [Шаблон Azure Resource Manager](../../app-service/overview-managed-identity.md#using-an-azure-resource-manager-template)

### <a name="azure-arc-enabled-kubernetes"></a>Azure Arc с поддержкой Kubernetes

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Preview (Предварительный просмотр) | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

В настоящее время Kubernetes с поддержкой Arc Azure [поддерживает назначенное системой удостоверение](../../azure-arc/kubernetes/connect-cluster.md#azure-arc-agents-for-kubernetes). Сертификат управляемого удостоверения службы используется всеми агентами Kubernetes, поддерживающими Azure Arc, для взаимодействия с Azure.

### <a name="azure-arc-enabled-servers"></a>Серверы с поддержкой Azure Arc

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | Недоступно |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

Все серверы с поддержкой Arc Azure имеют удостоверение, назначенное системой. Вы не можете отключить или изменить назначенное системой удостоверение на сервере с поддержкой дуги Azure. Дополнительные сведения об использовании управляемых удостоверений на серверах с поддержкой Arc Azure см. в следующих ресурсах:

- [Аутентификация в ресурсах Azure с помощью серверов с поддержкой Arc](../../azure-arc/servers/managed-identity-authentication.md)
- [Использование управляемого удостоверения с серверами с поддержкой Arc](../../azure-arc/servers/security-overview.md#using-a-managed-identity-with-arc-enabled-servers)

### <a name="azure-automanage"></a>Автоматическое управление Azure

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Preview (Предварительный просмотр) | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

Сведения о повторной настройке управляемого удостоверения при перемещении подписки на новый клиент см. в следующем документе:
* [Восстановление неработающей учетной записи автоуправления](../../automanage/repair-automanage-account.md)

### <a name="azure-blueprints"></a>Azure Blueprints

|Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | Недоступно |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check] | Недоступно | Недоступно |

В приведенном ниже списке указаны сведения об использовании управляемого удостоверения с [Azure Blueprints](../../governance/blueprints/overview.md):

- [Портал Azure — назначение схемы](../../governance/blueprints/create-blueprint-portal.md#assign-a-blueprint)
- [REST API — назначение схемы](../../governance/blueprints/create-blueprint-rest-api.md#assign-a-blueprint)


### <a name="azure-cognitive-search"></a>Когнитивный поиск Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

### <a name="azure-cognitive-services"></a>Azure Cognitive Services

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |


### <a name="azure-communication-services"></a>Службы коммуникации Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | ![Доступно][check] | Недоступно | Недоступно | Недоступно |


### <a name="azure-container-instances"></a>Экземпляры контейнеров Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Linux: Preview (Предварительный просмотр)<br>Windows: Недоступно | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Linux: Preview (Предварительный просмотр)<br>Windows: Недоступно | Недоступно | Недоступно | Недоступно |

Чтобы настроить управляемые удостоверения для Экземпляров контейнеров Azure (в регионах, где они доступны), см. следующий список.

- [Azure CLI](~/articles/container-instances/container-instances-managed-identity.md)
- [Шаблон Azure Resource Manager](~/articles/container-instances/container-instances-managed-identity.md#enable-managed-identity-using-resource-manager-template)
- [YAML](~/articles/container-instances/container-instances-managed-identity.md#enable-managed-identity-using-yaml-file)


### <a name="azure-container-registry-tasks"></a>Задачи Реестра контейнеров Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Preview (Предварительный просмотр) | Недоступно | Недоступно | Недоступно |

Чтобы настроить управляемое удостоверение для задачи Реестра контейнеров Azure (в доступных регионах), ознакомьтесь с приведенным ниже списком.

- [Azure CLI](~/articles/container-registry/container-registry-tasks-authentication-managed-identity.md)

### <a name="azure-data-explorer"></a>Azure Data Explorer

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

### <a name="azure-data-factory-v2"></a>Фабрика данных Azure версии 2

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

Чтобы настроить управляемые удостоверения для Фабрики данных Azure версии 2 (в регионах, где они доступны), см. следующий список.

- [Портал Azure](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity)
- [PowerShell](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity-using-powershell)
- [REST](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity-using-rest-api)
- [Пакет SDK](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity-using-sdk)

### <a name="azure-digital-twins"></a>Azure Digital Twins

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

См. следующий список, чтобы настроить управляемое удостоверение для Azure Digital двойников (в регионах, где это возможно):

- [Портал Azure](../../digital-twins/how-to-enable-managed-identities-portal.md)

### <a name="azure-event-grid"></a>Сетка событий Azure

Тип управляемого удостоверения |Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Preview (Предварительный просмотр) | Preview (Предварительный просмотр) | Недоступно | Preview (Предварительный просмотр) |
| Назначено пользователем | Недоступно | Недоступно  | Недоступно  | Недоступно |

### <a name="azure-firewall-policy"></a>Политика брандмауэра Azure

Тип управляемого удостоверения |Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Недоступно | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Preview (Предварительный просмотр) | Недоступно  | Недоступно  | Недоступно |

### <a name="azure-functions"></a>Функции Azure

Тип управляемого удостоверения |Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check]  | ![Доступно][check]  | ![Доступно][check]  |

Чтобы настроить управляемые удостоверения для Функций Azure (в регионах, где они доступны), см. следующий список.

- [Портал Azure](../../app-service/overview-managed-identity.md#using-the-azure-portal)
- [Azure CLI](../../app-service/overview-managed-identity.md#using-the-azure-cli)
- [Azure PowerShell](../../app-service/overview-managed-identity.md#using-azure-powershell)
- [Шаблон Azure Resource Manager](../../app-service/overview-managed-identity.md#using-an-azure-resource-manager-template)

### <a name="azure-iot-hub"></a>Центр Интернета вещей Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

См. следующий список, чтобы настроить управляемое удостоверение для центра Интернета вещей Azure (в регионах, где это возможно):

- [Портал Azure](../../iot-hub/virtual-network-support.md#turn-on-managed-identity-for-iot-hub)

### <a name="azure-importexport"></a>Импорт и экспорт Microsoft Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | --- | --- | --- | --- |
| Назначено системой | Доступно в регионе, где доступна служба импорта и экспорта Azure | Предварительный просмотр | Доступно | Доступно |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

### <a name="azure-kubernetes-service-aks"></a>Служба Azure Kubernetes (AKS)

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | Недоступно |
| Назначено пользователем | Preview (Предварительный просмотр) | Недоступно | Недоступно | Недоступно |


Дополнительные сведения см. в статье [Использование управляемых удостоверений в службе Kubernetes Azure](../../aks/use-managed-identity.md).

### <a name="azure-log-analytics-cluster"></a>Кластер Azure Log Analytics

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |

Дополнительные сведения см. [в разделе как работает удостоверение в Azure Monitor](../../azure-monitor/logs/customer-managed-keys.md)

### <a name="azure-logic-apps"></a>Azure Logic Apps

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check] | Недоступно | ![Доступно][check] |


Чтобы настроить управляемые удостоверения для Logic Apps (в регионах, где они доступны), см. следующий список.

- [Портал Azure](../../logic-apps/create-managed-service-identity.md#enable-system-assigned-identity-in-azure-portal)
- [Шаблон Azure Resource Manager](../../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)

### <a name="azure-machine-learning"></a>Машинное обучение Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Предварительный просмотр | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Preview (Предварительный просмотр) | Недоступно | Недоступно | Недоступно |

Дополнительные сведения см. [в статье Использование управляемых удостоверений с машинное обучение Azure](../../machine-learning/how-to-use-managed-identities.md).

### <a name="azure-policy"></a>Политика Azure

|Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

См. следующий список, чтобы настроить управляемое удостоверение для политики Azure (в регионах, где это возможно):

- [Портал Azure](../../governance/policy/tutorials/create-and-manage.md#assign-a-policy)
- [PowerShell](../../governance/policy/how-to/remediate-resources.md#create-managed-identity-with-powershell)
- [Azure CLI](/cli/azure/policy/assignment#az-policy-assignment-create)
- [Шаблоны диспетчера ресурсов Azure](/azure/templates/microsoft.authorization/policyassignments)
- [REST](/rest/api/resources/policyassignments/create)


### <a name="azure-service-fabric"></a>Azure Service Fabric

[Управляемое удостоверение для приложений Service Fabric](../../service-fabric/concepts-managed-identity.md) доступно во всех регионах.

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | ![Доступно][check] | Недоступно | Недоступно |Недоступно |

Чтобы настроить управляемое удостоверение для приложений Azure Service Fabric (в доступных регионах), ознакомьтесь с приведенным ниже списком.

- [Шаблон Azure Resource Manager](https://github.com/Azure-Samples/service-fabric-managed-identity/tree/anmenard-docs)

### <a name="azure-spring-cloud"></a>Azure Spring Cloud

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | Недоступно | Недоступно | ![Доступно][check] |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |


Дополнительные сведения см. [в статье Включение управляемого удостоверения, назначенного системой для облачного приложения Azure весны](~/articles/spring-cloud/spring-cloud-howto-enable-system-assigned-managed-identity.md).

### <a name="azure-stack-edge"></a>Azure Stack Edge

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | --- | --- | --- | --- |
| Назначено системой | Доступно в регионе, где доступна служба Azure Stack ребра | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

### <a name="azure-virtual-machine-scale-sets"></a>Масштабируемые наборы виртуальных машин Azure

|Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] |

Чтобы настроить управляемые удостоверения для масштабируемых наборов виртуальных машин Azure (в регионах, где они доступны), см. следующий список.

- [Портал Azure](qs-configure-portal-windows-vm.md)
- [PowerShell](qs-configure-powershell-windows-vm.md)
- [Azure CLI](qs-configure-cli-windows-vm.md)
- [Шаблоны диспетчера ресурсов Azure](qs-configure-template-windows-vm.md)
- [REST](qs-configure-rest-vm.md)



### <a name="azure-virtual-machines"></a>Виртуальные машины Azure

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] |
| Назначено пользователем | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] | ![Доступно][check] |

Чтобы настроить управляемые удостоверения для виртуальных машин Azure (в регионах, где они доступны), см. следующий список.

- [Портал Azure](qs-configure-portal-windows-vm.md)
- [PowerShell](qs-configure-powershell-windows-vm.md)
- [Azure CLI](qs-configure-cli-windows-vm.md)
- [Шаблоны диспетчера ресурсов Azure](qs-configure-template-windows-vm.md)
- [REST](qs-configure-rest-vm.md)
- [Пакеты SDK для Azure](qs-configure-sdk-windows-vm.md)


### <a name="azure-vm-image-builder"></a>Построитель образов виртуальных машин Azure

| Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Недоступно | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | [Доступно в поддерживаемых регионах](../../virtual-machines/image-builder-overview.md#regions) | Недоступно | Недоступно | Недоступно |

Сведения о настройке управляемого удостоверения для построителя образов виртуальных машин Azure (в регионах, где это возможно) см. в разделе [Общие сведения о построителе образов](../../virtual-machines/image-builder-overview.md#permissions).
### <a name="azure-signalr-service"></a>Служба SignalR Azure

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Preview (Предварительный просмотр) | Preview (Предварительный просмотр) | Недоступно | Preview (Предварительный просмотр) |
| Назначено пользователем | Preview (Предварительный просмотр) | Preview (Предварительный просмотр) | Недоступно | Preview (Предварительный просмотр) |

Сведения о настройке управляемого удостоверения для службы SignalR Azure (в регионах, где это возможно) см. в следующем списке:

- [Шаблон Azure Resource Manager](../../azure-signalr/howto-use-managed-identity.md)

### <a name="azure-resource-mover"></a>Azure Resource Mover

Тип управляемого удостоверения | Все общедоступные версии<br>Глобальные регионы Azure | Azure для государственных организаций | Azure для Германии | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Назначено системой | Доступно в регионах, где доступна служба для перемещения ресурсов Azure | Недоступно | Недоступно | Недоступно |
| Назначено пользователем | Недоступно | Недоступно | Недоступно | Недоступно |

Сведения об использовании службы "перемещение ресурсов Azure" см. в следующем документе:

- [Azure Resource Mover](../../resource-mover/overview.md)

## <a name="azure-services-that-support-azure-ad-authentication"></a>Службы Azure, поддерживающие аутентификацию Azure AD

Ниже приведены службы, поддерживающие аутентификацию Azure AD, которые были проверены с помощью служб клиента, использующих управляемые удостоверения для ресурсов Azure.

### <a name="azure-resource-manager"></a>Azure Resource Manager

Чтобы настроить доступ к Azure Resource Manager, ознакомьтесь с приведенным ниже списком.

- [Назначение доступа с помощью портала Azure](howto-assign-access-portal.md)
- [Назначение доступа с помощью PowerShell](howto-assign-access-powershell.md)
- [Назначение доступа с помощью Azure CLI](howto-assign-access-CLI.md)
- [Назначение доступа с помощью шаблона Azure Resource Manager](../../role-based-access-control/role-assignments-template.md)

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://management.azure.com/`| ![Доступно][check] |
| Azure для государственных организаций | `https://management.usgovcloudapi.net/` | ![Доступно][check] |
| Azure для Германии | `https://management.microsoftazure.de/` | ![Доступно][check] |
| Azure China 21Vianet | `https://management.chinacloudapi.cn` | ![Доступно][check] |

### <a name="azure-key-vault"></a>Azure Key Vault

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://vault.azure.net`| ![Доступно][check] |
| Azure для государственных организаций | `https://vault.usgovcloudapi.net` | ![Доступно][check] |
| Azure для Германии |  `https://vault.microsoftazure.de` | ![Доступно][check] |
| Azure China 21Vianet | `https://vault.azure.cn` | ![Доступно][check] |

### <a name="azure-data-lake"></a>Azure Data Lake;

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://datalake.azure.net/` | ![Доступно][check] |
| Azure для государственных организаций |  | Недоступно |
| Azure для Германии |   | Недоступно |
| Azure China 21Vianet |  | Недоступно |

### <a name="azure-sql"></a>Azure SQL

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://database.windows.net/` | ![Доступно][check] |
| Azure для государственных организаций | `https://database.usgovcloudapi.net/` | ![Доступно][check] |
| Azure для Германии | `https://database.cloudapi.de/` | ![Доступно][check] |
| Azure China 21Vianet | `https://database.chinacloudapi.cn/` | ![Доступно][check] |

### <a name="azure-data-explorer"></a>Azure Data Explorer

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://<account>.<region>.kusto.windows.net` | ![Доступно][check] |
| Azure для государственных организаций | `https://<account>.<region>.kusto.usgovcloudapi.net` | ![Доступно][check] |
| Azure для Германии | `https://<account>.<region>.kusto.cloudapi.de` | ![Доступно][check] |
| Azure China 21Vianet | `https://<account>.<region>.kusto.chinacloudapi.cn` | ![Доступно][check] |

### <a name="azure-event-hubs"></a>Центры событий Azure

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://eventhubs.azure.net` | ![Доступно][check] |
| Azure для государственных организаций |  | Недоступно |
| Azure для Германии |   | Недоступно |
| Azure China 21Vianet |  | Недоступно |

### <a name="azure-service-bus"></a>Служебная шина Azure

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://servicebus.azure.net`  | ![Доступно][check] |
| Azure для государственных организаций |  | ![Доступно][check] |
| Azure для Германии |   | Недоступно |
| Azure China 21Vianet |  | Недоступно |









### <a name="azure-storage-blobs-and-queues"></a>Большие двоичные объекты и очереди службы хранилища Azure

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://storage.azure.com/` <br /><br />`https://<account>.blob.core.windows.net` <br /><br />`https://<account>.queue.core.windows.net` | ![Доступно][check] |
| Azure для государственных организаций | `https://storage.azure.com/`<br /><br />`https://<account>.blob.core.usgovcloudapi.net` <br /><br />`https://<account>.queue.core.usgovcloudapi.net` | ![Доступно][check] |
| Azure для Германии | `https://storage.azure.com/`<br /><br />`https://<account>.blob.core.cloudapi.de` <br /><br />`https://<account>.queue.core.cloudapi.de` | ![Доступно][check] |
| Azure China 21Vianet | `https://storage.azure.com/`<br /><br />`https://<account>.blob.core.chinacloudapi.cn` <br /><br />`https://<account>.queue.core.chinacloudapi.cn` | ![Доступно][check] |

### <a name="azure-analysis-services"></a>Службы Azure Analysis Services

| Cloud | Идентификатор ресурса | Состояние |
|--------|------------|:-:|
| Azure (глобальный) | `https://*.asazure.windows.net` | ![Доступно][check] |
| Azure для государственных организаций | `https://*.asazure.usgovcloudapi.net` | ![Доступно][check] |
| Azure для Германии | `https://*.asazure.cloudapi.de` | ![Доступно][check] |
| Azure China 21Vianet | `https://*.asazure.chinacloudapi.cn` | ![Доступно][check] |

> [!Note]
> Microsoft Power BI также [поддерживает управляемые удостоверения](../../stream-analytics/powerbi-output-managed-identity.md).


[check]: media/services-support-managed-identities/check.png "Доступно"
---
title: Соответствие требованиям с помощью политики Azure
description: Назначение встроенных политик в политике Azure для аудита соответствия реестров контейнеров Azure
ms.topic: article
ms.date: 03/01/2021
ms.openlocfilehash: 0fed0c4132043e1eaed7e634e1f45b27f7c6e933
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103014304"
---
# <a name="audit-compliance-of-azure-container-registries-using-azure-policy"></a>Аудит соответствия реестров контейнеров Azure с помощью политики Azure

[Политика Azure](../governance/policy/overview.md) — это служба в Azure, которая используется для создания и назначения политик, а также управления ими. Эти политики гарантируют соблюдение различных правил и действий с ресурсами, что обеспечивает соответствие этих ресурсов корпоративным стандартам и соглашениям об уровне обслуживания.

В этой статье представлены встроенные политики для реестра контейнеров Azure. Используйте эти политики для аудита новых и существующих реестров для обеспечения соответствия.

Плата за использование политики Azure не взимается.

## <a name="built-in-policy-definitions"></a>Встроенные определения политик

Следующие встроенные определения политик относятся к реестру контейнеров Azure:

[!INCLUDE [azure-policy-reference-rp-containerreg](../../includes/policy/reference/byrp/microsoft.containerregistry.md)]

## <a name="assign-policies"></a>Назначение политик

* Назначьте политики с помощью [портал Azure](../governance/policy/assign-policy-portal.md), [Azure CLI](../governance/policy/assign-policy-azurecli.md), [Диспетчер ресурсов шаблона](../governance/policy/assign-policy-template.md)или пакетов SDK политики Azure.
* Определение области назначения политики для группы ресурсов, подписки или [группы управления Azure](../governance/management-groups/overview.md). Назначения политики реестра контейнеров применяются к существующим и новым реестрам контейнеров в области.
* Включение или отключение [принудительного применения политики](../governance/policy/concepts/assignment-structure.md#enforcement-mode) в любое время.

> [!NOTE]
> После назначения или обновления политики для применения назначения к ресурсам в определенной области потребуется некоторое время. См. сведения о [триггерах оценки политики](../governance/policy/how-to/get-compliance-data.md#evaluation-triggers).

## <a name="review-policy-compliance"></a>Проверка соответствия политике

Получите доступ к сведениям о соответствии, созданным назначениями политик, с помощью портал Azure, средств командной строки Azure или пакетов SDK политики Azure. Дополнительные сведения см. в разделе [Получение данных о соответствии ресурсов Azure](../governance/policy/how-to/get-compliance-data.md).

Если ресурс не соответствует требованиям, возможных причин этого множество. Чтобы определить причину или найти ответственного за изменение, см. раздел [Определение несоответствия](../governance/policy/how-to/determine-non-compliance.md).

### <a name="policy-compliance-in-the-portal"></a>Соответствие политике на портале:

1. Выберите **все службы** и выполните поиск по запросу **Политика**.
1. Выберите **соответствие**.
1. Используйте фильтры для ограничения состояний соответствия или поиска политик.

    ![Соответствие политике на портале](./media/container-registry-azure-policy/azure-policy-compliance.png)
    
1. Выберите политику для просмотра статистических сведений о соответствии и событиях. При необходимости выберите конкретный реестр для соответствия ресурсов.

### <a name="policy-compliance-in-the-azure-cli"></a>Соответствие политике в Azure CLI

Можно также использовать Azure CLI для получения данных о соответствии. Например, используйте команду [AZ Policy назначение List](/cli/azure/policy/assignment#az-policy-assignment-list) в CLI, чтобы получить идентификаторы политик, которые применяются к политикам реестра контейнеров Azure:

```azurecli
az policy assignment list --query "[?contains(displayName,'Container Registries')].{name:displayName, ID:id}" --output table
```

Пример результатов выполнения:

```
Name                                                                                   ID
-------------------------------------------------------------------------------------  --------------------------------------------------------------------------------------------------------------------------------
Container Registries should not allow unrestricted network access           /subscriptions/<subscriptionID>/providers/Microsoft.Authorization/policyAssignments/b4faf132dc344b84ba68a441
Container Registries should be encrypted with a Customer-Managed Key (CMK)  /subscriptions/<subscriptionID>/providers/Microsoft.Authorization/policyAssignments/cce1ed4f38a147ad994ab60a
```

Затем выполните команду [AZ Policy State List](/cli/azure/policy/state#az-policy-state-list) , чтобы вернуть состояние соответствия в формате JSON для всех ресурсов с указанным идентификатором политики:

```azurecli
az policy state list \
  --resource <policyID>
```

Или выполните команду [AZ Policy State List](/cli/azure/policy/state#az-policy-state-list) , чтобы вернуть состояние соответствия в формате JSON для определенного ресурса реестра, например *myregistry*:

```azurecli
az policy state list \
 --resource myregistry \
 --namespace Microsoft.ContainerRegistry \
 --resource-type registries \
 --resource-group myresourcegroup
```

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте больше о [определениях](../governance/policy/concepts/definition-structure.md) и [влиянии](../governance/policy/concepts/effects.md)политики Azure.

* Создайте [Определение настраиваемой политики](../governance/policy/tutorials/create-custom-policy-definition.md).

* Дополнительные сведения о [возможностях](../governance/index.yml) управления в Azure.

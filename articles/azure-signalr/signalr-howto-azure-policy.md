---
title: Соответствие требованиям с помощью политики Azure
description: Назначение встроенных политик в политике Azure для аудита соответствия ресурсам службы Azure SignalR.
author: JialinXin
ms.service: signalr
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: jixin
ms.openlocfilehash: 018033d3a6123948191a7261f5a1ee2ae526e25a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89295031"
---
# <a name="audit-compliance-of-azure-signalr-service-resources-using-azure-policy"></a>Аудит соответствия ресурсов службы Azure SignalR с помощью политики Azure

[Политика Azure](../governance/policy/overview.md) — это служба в Azure, которая используется для создания и назначения политик, а также управления ими. Эти политики гарантируют соблюдение различных правил и действий с ресурсами, что обеспечивает соответствие этих ресурсов корпоративным стандартам и соглашениям об уровне обслуживания.

В этой статье представлены встроенные политики (Предварительная версия) для службы Azure SignalR. Используйте эти политики для аудита новых и существующих ресурсов SignalR на соответствие.

Плата за использование политики Azure не взимается.

## <a name="built-in-policy-definitions"></a>Встроенные определения политик

Следующие встроенные определения политик относятся к службе Azure SignalR:

[!INCLUDE [azure-policy-reference-policies-signalr](../../includes/policy/reference/bycat/policies-signalr.md)]

## <a name="assign-policy-definitions"></a>Назначение определений политики

* Назначьте определения политик с помощью [портал Azure](../governance/policy/assign-policy-portal.md), [Azure CLI](../governance/policy/assign-policy-azurecli.md), [шаблона диспетчер ресурсов](../governance/policy/assign-policy-template.md)или пакетов SDK политики Azure.
* Определение области назначения политики для группы ресурсов, подписки или [группы управления Azure](../governance/management-groups/overview.md). Назначения политики SignalR применяются к существующим и новым ресурсам SignalR в пределах области.
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
   
    [![Соответствие политике на портале ](./media/signalr-howto-azure-policy/azure-policy-compliance.png)](./media/signalr-howto-azure-policy/azure-policy-compliance.png#lightbox)
2. Выберите политику для просмотра статистических сведений о соответствии и событиях. При необходимости выберите конкретный SignalR для соответствия ресурсов.

### <a name="policy-compliance-in-the-azure-cli"></a>Соответствие политике в Azure CLI

Можно также использовать Azure CLI для получения данных о соответствии. Например, используйте команду [AZ Policy назначение List](/cli/azure/policy/assignment#az-policy-assignment-list) в CLI, чтобы получить идентификаторы политик, которые применяются к политикам службы "Azure SignalR":

```azurecli
az policy assignment list --query "[?contains(displayName,'SignalR')].{name:displayName, ID:id}" --output table
```

Пример результатов выполнения:

```
Name                                                                                   ID
-------------------------------------------------------------------------------------  --------------------------------------------------------------------------------------------------------------------------------
[Preview]: Azure SignalR Service should use private links  /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/policyAssignments/<assignmentId>
```

Затем выполните команду [AZ Policy State List](/cli/azure/policy/state#az-policy-state-list) , чтобы вернуть состояние соответствия в формате JSON для всех ресурсов в определенной группе ресурсов:

```azurecli
az policy state list --g <resourceGroup>
```

Или выполните команду [AZ Policy State List](/cli/azure/policy/state#az-policy-state-list) , чтобы вернуть состояние соответствия в формате JSON для определенного ресурса SignalR:

```azurecli
az policy state list \
 --resource /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.SignalRService/SignalR/<resourceName> \
 --namespace Microsoft.SignalRService \
 --resource-group <resourceGroup>
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [определениях](../governance/policy/concepts/definition-structure.md) и [влиянии](../governance/policy/concepts/effects.md) политики Azure

* Создание [определения пользовательской политики](../governance/policy/tutorials/create-custom-policy-definition.md)

* Дополнительные сведения о [возможностях](../governance/index.yml) управления в Azure


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
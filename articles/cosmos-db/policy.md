---
title: Применение Политики Azure для систем и элементов управления ресурсами Azure Cosmos DB
description: Сведения о том, как применить Политику Azure для систем и элементов управления ресурсами Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 1390f5db6e0f0370788bef60d5a2cafee1e8a96d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93080657"
---
# <a name="use-azure-policy-to-implement-governance-and-controls-for-azure-cosmos-db-resources"></a>Применение Политики Azure для систем и элементов управления ресурсами Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

[Политика Azure](../governance/policy/overview.md) помогает поддерживать корпоративные стандарты управления, оценивать соответствие ресурсов и реализовать автоматическое исправление. Типичные цели использования для этого механизма: безопасность, управление затратами, согласованность конфигурации.

Политика Azure предоставляет встроенные определения политик. Но если эти встроенные определения не соответствуют вашим требованиям, вы можете создать пользовательские определения политик. Дополнительные сведения см. в [документации по Политике Azure](../governance/policy/overview.md).

> [!IMPORTANT]
> Политика Azure применяется на уровне поставщика ресурсов для служб Azure. Cosmos DB пакеты SDK могут выполнять большинство операций управления для баз данных, контейнеров и пропускной способности, которые обходят поставщик ресурсов Cosmos DB, тем самым игнорируя все политики, созданные с помощью политики Azure. Чтобы обеспечить соблюдение политик, см. раздел [предотвращение изменений в Azure Cosmos DBных](role-based-access-control.md#prevent-sdk-changes) пакетах SDK.

## <a name="assign-a-built-in-policy-definition"></a>Назначение определения политики

Определения политик описывают условия соответствия для ресурсов и действия, которые нужно выполнять при выполнении этих условий. _Назначения_ политик создаются из _определений_ политик. Для ресурсов Azure Cosmos DB можно использовать как встроенные, так и пользовательские определения политик. Назначения политик можно указать для группы управления Azure, подписки Azure или группы ресурсов, и они будут применяться для всех ресурсов в выбранной области. Также есть возможность исключить определенные ресурсы из области действия.

Назначения политик можно создавать с помощью [портала Azure](../governance/policy/assign-policy-portal.md), [Azure PowerShell](../governance/policy/assign-policy-powershell.md), [Azure CLI](../governance/policy/assign-policy-azurecli.md) или [шаблонов Resource Manager](../governance/policy/assign-policy-template.md).

Чтобы создать назначение политики из встроенного определения политики для Azure Cosmos DB с помощью портала Azure, выполните действия из [этой статьи](../governance/policy/assign-policy-portal.md).

На шаге выбора определения политики введите в поле поиска строку `Cosmos DB`, чтобы отфильтровать список доступных встроенных определений политики. Выберите одно из доступных встроенных определений политики, а затем щелкните **Выбрать**, чтобы продолжить создание назначения политики.

> [!TIP]
> Можно также использовать имена встроенных определений политик, которые отображаются в области **Доступные определения**, в командах Azure PowerShell, Azure CLI или шаблонах Resource Manager, чтобы создать назначения политик.

:::image type="content" source="./media/policy/available-definitions.png" alt-text="Поиск встроенных определений политик для Azure Cosmos DB":::

## <a name="create-a-custom-policy-definition"></a>Создание определения пользовательской политики

Для необычных сценариев, которые не охвачены встроенными политиками, вы можете создать [пользовательские определения политик](../governance/policy/tutorials/create-custom-policy-definition.md). Созданное _определение_ пользовательской политики вы сможете затем применить для создания _назначения_ политики.

### <a name="property-types-and-property-aliases-in-policy-rules"></a>Типы свойств и псевдонимы свойств в правилах политики

[Шаги для определения настраиваемой политики](../governance/policy/tutorials/create-custom-policy-definition.md) позволяют определить свойства ресурсов и псевдонимы свойств, которые потребуются для создания правил политики.

Чтобы выяснить псевдонимы свойств,имеющие отношение к Azure Cosmos DB, используйте пространство имен `Microsoft.DocumentDB` в сочетании с одним из методов, которые описаны в статье с шагами для определения пользовательского определения политики.

#### <a name="use-the-azure-cli"></a>С помощью Azure CLI:
```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Get Azure Policy aliases for namespace Microsoft.DocumentDB
az provider show --namespace Microsoft.DocumentDB --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
```

#### <a name="use-azure-powershell"></a>С помощью Azure PowerShell:
```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Use Get-AzPolicyAlias to list aliases for Microsoft.DocumentDB namespace
(Get-AzPolicyAlias -NamespaceMatch 'Microsoft.DocumentDB').Aliases
```

Эти команды выводят список псевдонимов свойств для указанного свойства Azure Cosmos DB. Ниже представлен фрагмент выходных данных этой команды:

```json
[
  "Microsoft.DocumentDB/databaseAccounts/sku.name",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*]",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*].id",
  "Microsoft.DocumentDB/databaseAccounts/isVirtualNetworkFilterEnabled",
  "Microsoft.DocumentDB/databaseAccounts/consistencyPolicy.defaultConsistencyLevel",
  "Microsoft.DocumentDB/databaseAccounts/enableAutomaticFailover",
  "Microsoft.DocumentDB/databaseAccounts/Locations",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*]",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*].locationName",
  "..."
]
```

Любой из псевдонимов свойств можно использовать в [правилах определения пользовательских политик](../governance/policy/tutorials/create-custom-policy-definition.md#policy-rule).

Ниже приведен пример определения политики, который проверяет, настроена ли учетная запись Azure Cosmos DB для нескольких расположений для записи. Определение пользовательской политики содержит два правила: одно для проверки конкретного типа псевдонима свойства, а второе — для конкретного свойства типа, в данном случае поле, в котором хранится параметр множественного расположения для записи. Оба правила используют псевдонимы.

```json
"policyRule": {
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.DocumentDB/databaseAccounts"
      },
      {
        "field": "Microsoft.DocumentDB/databaseAccounts/enableMultipleWriteLocations",
        "notEquals": true
      }
    ]
  },
  "then": {
    "effect": "Audit"
  }
}
```

Пользовательские определения политик можно точно так же использовать для создания назначений политик, как и встроенные определения политик.

## <a name="policy-compliance"></a>Соответствие политике

Когда вы создаете назначение политики, Политика Azure оценивает ресурсы в области назначения. Проводится проверка на _соответствие политике_ для каждого ресурса. Указанный в политике _эффект_ применяется ко всем несоответствующим ресурсам.

Результаты проверки соответствия и сведения об исправлении можно посмотреть на [портале Azure](../governance/policy/how-to/get-compliance-data.md#portal), через [Azure CLI](../governance/policy/how-to/get-compliance-data.md#command-line) или в [журналах Azure Monitor](../governance/policy/how-to/get-compliance-data.md#azure-monitor-logs).

На следующем снимке экрана приведены два примера назначений политики.

Одно из этих назначений основано на встроенном определении политики и проверяет, что ресурсы Azure Cosmos DB развертываются только в разрешенных регионах Azure. В соответствии с ресурсами отображается результат оценки политики (соответствует или не соответствует) для ресурсов в области.

Другое назначение основано на пользовательском определении политики. Это назначение проверяет, настроены ли учетные записи Cosmos DB для нескольких расположений для записи.

После развертывания назначений политики результаты оценки отображаются на панели мониторинга соответствия. Обратите внимание, что на это может потребоваться до 30 минут после развертывания назначения политики. Кроме того, после создания назначений политик [Просмотр оценки политики может быть запущен немедленно по требованию](../governance/policy/how-to/get-compliance-data.md#on-demand-evaluation-scan) .

На снимке экрана показаны следующие результаты оценки соответствия для Azure Cosmos DB учетных записей в области:

- Ноль двух учетных записей соответствует политике, которая должна быть настроена для фильтрации виртуальной сети (VNet).
- Ноль двух учетных записей соответствует политике, требующей, чтобы учетная запись была настроена для нескольких расположений для записи.
- Ноль двух учетных записей соответствует политике, которую ресурсы были развернуты в разрешенных регионах Azure.

:::image type="content" source="./media/policy/compliance.png" alt-text="Результаты соответствия требованиям для назначений политик Azure":::

Сведения об исправлении несоответствующих ресурсов см. в статье [как исправить ресурсы с помощью политики Azure](../governance/policy/how-to/remediate-resources.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Ознакомьтесь с примерами определений пользовательских политик для Azure Cosmos DB](https://github.com/Azure/azure-policy/tree/master/samples/CosmosDB), в том числе для нескольких расположений записи и политик фильтрации виртуальной сети, показанных выше.
- [Создание назначения политики на портале Azure](../governance/policy/assign-policy-portal.md)
- [Встроенные определения политик в Политике Azure для Azure Cosmos DB](./policy-reference.md)
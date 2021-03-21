---
title: Область по типам ресурсов расширения
description: Описывает использование свойства Scope при развертывании типов ресурсов расширения.
ms.topic: conceptual
ms.date: 01/13/2021
ms.openlocfilehash: ce08ca951e24c1c0a5450052cf814a68888837c2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99492167"
---
# <a name="setting-scope-for-extension-resources-in-arm-templates"></a>Настройка области для ресурсов расширения в шаблонах ARM

Ресурс расширения — это ресурс, который изменяет другой ресурс. Например, можно назначить роль для ресурса. Назначение роли является типом ресурса расширения.

Полный список типов ресурсов расширения см. в разделе [типы ресурсов, расширяющие возможности других ресурсов](../management/extension-resource-types.md).

В этой статье показано, как задать область для типа ресурса расширения при развертывании с помощью шаблона Azure Resource Manager (шаблон ARM). В нем описывается свойство Scope, доступное для ресурсов расширения при применении к ресурсу.

> [!NOTE]
> Свойство Scope доступно только для типов ресурсов расширения. Чтобы указать другую область для типа ресурса, который не является типом расширения, используйте вложенное или связанное развертывание. Дополнительные сведения см. в разделе [Развертывание группы ресурсов](deploy-to-resource-group.md), [развертывание подписок](deploy-to-subscription.md), [развертывание групп управления](deploy-to-management-group.md)и [развертывание клиентов](deploy-to-tenant.md).

## <a name="apply-at-deployment-scope"></a>Применить к области развертывания

Чтобы применить тип ресурса расширения в целевой области развертывания, добавьте ресурс в шаблон, как и для любого типа ресурсов. Доступные области: [Группа ресурсов](deploy-to-resource-group.md), [Подписка](deploy-to-subscription.md), [Группа управления](deploy-to-management-group.md)и [клиент](deploy-to-tenant.md). Область развертывания должна поддерживать тип ресурса.

Следующий шаблон развертывает блокировку.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/locktargetscope.json":::

При развертывании в группе ресурсов она блокирует группу ресурсов.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/locktargetscope.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
 New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/locktargetscope.json"
```

---

В следующем примере назначается роль.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/roletargetscope.json":::

При развертывании в подписке она назначает роль для подписки.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az deployment sub create \
  --name demoSubDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/roletargetscope.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzSubscriptionDeployment `
  -Name demoSubDeployment `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/scope/roletargetscope.json"
```

---

## <a name="apply-to-resource"></a>Применить к ресурсу

Чтобы применить ресурс расширения к ресурсу, используйте `scope` свойство. Задайте для свойства Scope имя ресурса, к которому добавляется расширение. Свойство Scope является корневым свойством для типа ресурса расширения.

В следующем примере создается учетная запись хранения и к ней применяется роль.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/storageandrole.json" highlight="56":::

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о том, как определить параметры в шаблоне, см. [в разделе Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Советы по устранению распространенных ошибок развертывания см. в разделе [Устранение распространенных ошибок развертывания в Azure с помощью Azure Resource Manager](common-deployment-errors.md).
* Сведения о развертывании шаблона, требующего маркер SAS, см. в статье [развертывание закрытого шаблона ARM с помощью маркера SAS](secure-template-with-sas-token.md).

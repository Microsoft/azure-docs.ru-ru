---
title: Краткое руководство. Создание назначения политики с помощью файла BICEP (предварительная версия)
description: В этом кратком руководстве с помощью файла BICEP (предварительная версия) создается назначение политики для обнаружения ресурсов, не соответствующих требованиям.
ms.date: 04/01/2021
ms.topic: quickstart
ms.custom: subject-bicepqs
ms.openlocfilehash: 17460f9a06d7225d50933608645ea8aea2f5e8f6
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106223869"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-by-using-a-bicep-file"></a>Создание назначения политики для обнаружения несоответствующих требованиям ресурсов с помощью файла BICEP

Чтобы понять, соответствуют ли ресурсы требованиям в Azure, прежде всего нужно определить их состояние.
В этом кратком руководстве описано, как с помощью файла [BICEP (предварительная версия)](https://github.com/Azure/bicep) создать назначение политики для определения виртуальных машин, которые не используют управляемые диски. Завершив работу, вы узнаете, какие виртуальные машины не используют управляемые диски, так как _не соответствуют_ назначению политики.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. На портале Azure откроется шаблон.

:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Кнопка для развертывания шаблона ARM для назначения политики Azure в Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurepolicy-assign-builtinpolicy-resourcegroup%2Fazuredeploy.json":::

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
- Установлен BICEP версии `0.3` или более поздней. Если у вас еще нет BICEP CLI или вам требуется обновление, см. статью об [Установке BICEP (предварительная версия)](../../azure-resource-manager/templates/bicep-install.md).

## <a name="review-the-bicep-file"></a>Проверка BICEP-файла

С помощью этого краткого руководства вы создадите назначение политики и назначите встроенное определение политики _Аудит виртуальных машин, которые не используют управляемые диски_ (`06a78e20-9358-41c9-923c-fb736d382a4d`). См. [полный список всех доступных встроенных политик Azure](./samples/index.md).

Создайте следующий файл BICEP `assignment.bicep`:

```bicep
param policyAssignmentName string = 'audit-vm-manageddisks'
param policyDefinitionID string = '/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d'

resource assignment 'Microsoft.Authorization/policyAssignments@2019-09-01' = {
    name: policyAssignmentName
    properties: {
        scope: subscriptionResourceId('Microsoft.Resources/resourceGroups', resourceGroup().name)
        policyDefinitionId: policyDefinitionID
    }
}

output assignmentId string = assignment.id
```

В шаблоне определен следующий ресурс:

- [Microsoft.Authorization/policyAssignments](/azure/templates/microsoft.authorization/policyassignments)

## <a name="deploy-the-template"></a>Развертывание шаблона

> [!NOTE]
> Служба "Политика Azure" предоставляется бесплатно. Дополнительные сведения см. в статье [Что такое служба "Политика Azure"?](./overview.md).

После установки и создания файла BICEP CLI можно развернуть файл BICEP с помощью:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroupDeployment `
  -Name PolicyDeployment `
  -ResourceGroupName PolicyGroup `
  -TemplateFile assignment.bicep
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az deployment group create \
  --name PolicyDeployment \
  --resource-group PolicyGroup \
  --template-file assignment.bicep
```

---

Ниже приведены некоторые дополнительные ресурсы.

- Дополнительные примеры шаблонов доступны [здесь](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Authorization&pageNumber=1&sort=Popular).
- Справочник по шаблонам Azure можно просмотреть [здесь](/azure/templates/microsoft.authorization/allversions).
- Чтобы узнать, как создавать шаблоны ARM, ознакомьтесь с [документацией по Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Чтобы узнать о развертывании на уровне подписки, ознакомьтесь со статьей [Создание групп ресурсов и ресурсов на уровне подписки](../../azure-resource-manager/templates/deploy-to-subscription.md).

## <a name="validate-the-deployment"></a>Проверка развертывания

Выберите **Соответствие** в левой части страницы и найдите ранее созданное назначение политики _Аудит виртуальных машин, которые не используют управляемые диски_.

:::image type="content" source="./media/assign-policy-template/policy-compliance.png" alt-text="Снимок экрана: сведения о соответствии политике на странице &quot;Соответствие политике&quot;." border="false":::

Существующие ресурсы, которые не соответствуют новому назначению, отображаются в разделе **Несоответствующие ресурсы**.

Дополнительные сведения см. в разделе [Как работает соответствие](./how-to/get-compliance-data.md#how-compliance-works).

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить созданное назначение, выполните следующие действия:

1. Выберите элемент **Соответствие** (или **Назначения**) в левой части страницы службы Политика Azure и найдите ранее созданное назначение политики _Аудит виртуальных машин, которые не используют управляемые диски_.

1. Щелкните правой кнопкой мыши назначение политики _Аудит виртуальных машин, которые не используют управляемые диски_ и выберите **Удалить назначение**.

   :::image type="content" source="./media/assign-policy-template/delete-assignment.png" alt-text="Снимок экрана: использование контекстного меню для удаления назначения на странице &quot;Соответствие&quot;." border="false":::

1. Удалите файл `assignment.bicep`.

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы назначили встроенное определение политики для области и изучили отчет о соответствии. Определение политики проверяет, все ли ресурсы в области соответствуют требованиям, и определяет, какие из них не соответствуют.

Следующее руководство серии содержит сведения о назначении политик для проверки новых ресурсов на соответствие требованиям:

> [!div class="nextstepaction"]
> [Создание политик и управление ими](./tutorials/create-and-manage.md)
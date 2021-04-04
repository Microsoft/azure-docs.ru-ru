---
title: Краткое руководство. Создание назначения политики с помощью REST API
description: В рамках этого краткого руководства с помощью REST API вы создадите назначение в службе "Политика Azure", позволяющее определить ресурсы, которые не соответствуют требованиям.
ms.date: 01/29/2021
ms.topic: quickstart
ms.openlocfilehash: 438d8004cd50e6e2ef7586c51adc63257f37978b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99219983"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-with-rest-api"></a>Краткое руководство. Создание назначения политики для идентификации ресурсов, не соответствующих требованиям, с помощью REST API

Чтобы понять, соответствуют ли ресурсы требованиям в Azure, прежде всего нужно определить их состояние.
В этом кратком руководстве описано, как создать назначение политики для определения виртуальных машин, которые не используют управляемые диски.

Завершив работу, вы узнаете, какие виртуальные машины не используют управляемые диски, так как _не соответствуют_ назначению политики.

REST API используется для создания ресурсов Azure, а также управления ими. В этом руководстве REST API используется для создания назначения политики для определения в среде Azure ресурсов, не соответствующих требованиям.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Установите [ARMClient](https://github.com/projectkudu/ARMClient), если его у вас еще нет. Это средство, которое отправляет HTTP-запросы к REST API на основе Azure Resource Manager. Кроме того, вы можете использовать функцию "Попробовать" в документации по REST или средствах, например PowerShell [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) или [Postman](https://www.postman.com).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-policy-assignment"></a>Создание назначения политики

С помощью этого краткого руководства вы создадите назначение политики и назначите определение **Аудит виртуальных машин, которые не используют управляемые диски** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Это определение политики идентифицирует ресурсы, которые не соответствуют заданным в нем условиям.

Чтобы создать назначение политики, выполните следующую команду:

   - Универсальный код ресурса (URI) REST API

     ```http
     PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/audit-vm-manageddisks?api-version=2019-09-01
     ```

   - Текст запроса

     ```json
     {
       "properties": {
         "displayName": "Audit VMs without managed disks Assignment",
         "description": "Shows all virtual machines not using managed disks",
         "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d",
         "nonComplianceMessages": [
             {
                 "message": "Virtual machines should use a managed disk"
             }
         ]
       }
     }
     ```

В предыдущей конечной точке и тексте запроса используются следующие сведения:

Универсальный код ресурса (URI) REST API:
- **Scope.** Область определяет, к каким ресурсам или группе ресурсов принудительно применяется назначение политики. Она может варьироваться от группы управления до отдельного ресурса. Обязательно замените `{scope}` на один из следующих шаблонов:
  - Группа управления: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Подписка: `/subscriptions/{subscriptionId}`
  - группу ресурсов `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`;
  - Ресурс: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]{resourceType}/{resourceName}`.
- **Name** — фактическое имя назначения. В этом примере использовалось имя _audit-vm-manageddisks_.

Тело запроса:
- **DisplayName** — отображаемое имя назначения политики. В этом случае используется определение _аудита виртуальных машин без назначения управляемых дисков_.
- **Description** — более подробное объяснение того, что делает политика или почему она относится к этой области.
- **policyDefinitionId** — идентификатор определения политики, на основе которой вы создаете назначение. В нашем случае это идентификатор определения политики _аудита виртуальных машин, которые не используют управляемые диски_.
- **nonComplianceMessages** — позволяет указать сообщение, которое будет отображаться, когда ресурс отклонен из-за несоответствия или был признан несоответствующим. Дополнительные сведения см. в разделе [Сообщения о несоответствии](./concepts/assignment-structure.md#non-compliance-messages).

## <a name="identify-non-compliant-resources"></a>Выявление несоответствующих ресурсов

Чтобы просмотреть ресурсы, которые не соответствуют этому новому назначению, выполните следующую команду и выведите идентификаторы несовместимых ресурсов в JSON-файле:

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$filter=IsCompliant eq false and PolicyAssignmentId eq 'audit-vm-manageddisks'&$apply=groupby((ResourceId))"
```

Результаты должны выглядеть примерно так:

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 3,
    "value": [{
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3Id>"
        }

    ]
}
```

Результаты похожи на содержимое списка **несовместимых ресурсов** в представлении портала Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить созданное назначение, выполните следующую команду:

```http
DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/audit-vm-manageddisks?api-version=2019-09-01
```

Замените `{scope}` той же областью, которую вы использовали при первом создании назначения политики.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы назначили определение политики для идентификации ресурсов, не соответствующих требованиям, в среде Azure.

Следующее руководство серии содержит сведения о назначении политик для проверки новых ресурсов на соответствие требованиям:

> [!div class="nextstepaction"]
> [Создание политик и управление ими](./tutorials/create-and-manage.md)

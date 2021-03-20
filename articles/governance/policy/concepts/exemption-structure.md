---
title: Сведения о структуре исключения политики
description: Описание определения исключения политики, используемого политикой Azure для исключения ресурсов из оценки инициатив или определений.
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: e6ced56c1dc65ca68998c5c58d3e985b63873e0b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91950182"
---
# <a name="azure-policy-exemption-structure"></a>Структура исключения политики Azure

Функция исключения из политики Azure (Предварительная версия) используется для _исключения_ иерархии ресурсов или отдельного ресурса из оценки инициатив или определений. Ресурсы, которые _не подпадают под_ общее соответствие, но не могут оцениваться или иметь временный отказ. Дополнительные сведения см. [в разделе сведения о области в политике Azure](./scope.md). Исключения политики Azure работают только с [режимами диспетчер ресурсов](./definition-structure.md#resource-manager-modes) и не работают с [режимами поставщика ресурсов](./definition-structure.md#resource-provider-modes).

> [!IMPORTANT]
> Эта функция предоставляется бесплатно во время **предварительной версии**. Сведения о ценах см. на странице [цен на политику Azure](https://azure.microsoft.com/pricing/details/azure-policy/). См. подробные сведения о [дополнительных условиях использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Для создания исключения политики используется JSON. Исключение политики содержит элементы для:

- display name
- description
- метаданные
- Назначение политики
- определения политик в рамках инициативы
- Категория исключения
- expiration

> [!NOTE]
> Исключение политики создается как дочерний объект в иерархии ресурсов или отдельный ресурс, которому было предоставлено исключение, поэтому целевой объект не включается в определение исключения.

Например, в следующем JSON показано исключение политики в категории **отказа** ресурса для назначения инициативы с именем `resourceShouldBeCompliantInit` . Ресурс _исключен_ только из двух определений политики в инициативе, `customOrgPolicy` определения настраиваемой политики (ссылки `requiredTags` ) и определения встроенной политики "разрешенные расположения" (идентификатор: `e56962a6-4747-49cd-b67b-bf8b01975c4c` , ссылка `allowedLocations` ):

```json
{
    "id": "/subscriptions/{subId}/resourceGroups/ExemptRG/providers/Microsoft.Authorization/policyExemptions/resourceIsNotApplicable",
    "name": "resourceIsNotApplicable",
    "type": "Microsoft.Authorization/policyExemptions",
    "properties": {
        "displayName": "This resource is scheduled for deletion",
        "description": "This resources is planned to be deleted by end of quarter and has been granted a waiver to the policy.",
        "metadata": {
            "requestedBy": "Storage team",
            "approvedBy": "IA",
            "approvedOn": "2020-07-26T08:02:32.0000000Z",
            "ticketRef": "4baf214c-8d54-4646-be3f-eb6ec7b9bc4f"
        },
        "policyAssignmentId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyAssignments/resourceShouldBeCompliantInit",
        "policyDefinitionReferenceIds": [
            "requiredTags",
            "allowedLocations"
        ],
        "exemptionCategory": "waiver",
        "expiresOn": "2020-12-31T23:59:00.0000000Z"
    }
}
```

Фрагмент связанной инициативы с сопоставлением, `policyDefinitionReferenceIds` используемым исключением политики:

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyDefinitions/customOrgPolicy",
        "policyDefinitionReferenceId": "requiredTags",
        "parameters": {
            "reqTags": {
                "value": "[parameters('init_reqTags')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
        "policyDefinitionReferenceId": "allowedLocations",
        "parameters": {
            "listOfAllowedLocations": {
                "value": "[parameters('init_listOfAllowedLocations')]"
            }
        }
    }
]
```

## <a name="display-name-and-description"></a>Отображаемое имя и описание

**DisplayName** и **Description** используются для обнаружения исключения политики и предоставления контекста для его использования с конкретным ресурсом. Максимальная длина **displayName** составляет _128_ символов, а **description** — _512_ символов.

## <a name="metadata"></a>Метаданные

Свойство **Metadata** позволяет создать любое дочернее свойство, необходимое для хранения релевантной информации. В приведенном выше примере свойства **requestedBy**, **аппроведби**, **аппроведон** и **тиккетреф** содержат значения клиента для предоставления сведений о том, кто запросил исключение, кто утвердил его и когда, а также внутренний билет отслеживания для запроса. Эти свойства **метаданных** являются примерами, но они не требуются, и **метаданные** не ограничиваются этими дочерними свойствами.

## <a name="policy-assignment-id"></a>Идентификатор назначения политики

Это поле должно быть полным путем к назначению политики или назначению инициативы.
`policyAssignmentId` является строкой, а не массивом. Это свойство определяет, из какого назначения будет _исключена_ родительская иерархия ресурсов или отдельный ресурс.

## <a name="policy-definition-ids"></a>Идентификаторы определения политики

Если объект `policyAssignmentId` предназначен для назначения инициативы, `policyDefinitionReferenceIds` можно использовать свойство, чтобы указать, какие определения политик в инициативе для ресурса субъекта имеют исключение. Так как ресурс может быть исключен из одного или нескольких определений включенных политик, это свойство является _массивом_. Значения должны соответствовать значениям в определении инициативы в `policyDefinitions.policyDefinitionReferenceId` полях.

## <a name="exemption-category"></a>Категория исключения

Существуют две категории исключения, которые используются для группирования исключений:

- **Устранено**. исключение предоставляется, поскольку цель политики достигается с помощью другого метода.
- **Отказ**: исключение предоставляется, так как состояние несоответствия ресурса временно принято. Другой причиной использования этой категории является иерархия ресурсов или ресурсов, которые должны быть исключены из одного или нескольких определений в инициативе, но не должны быть исключены из всей инициативы.

## <a name="expiration"></a>Окончание срока действия

Чтобы задать, когда иерархия ресурсов или отдельный ресурс _больше не исключается для назначения_ , установите `expiresOn` свойство. Это необязательное свойство должно иметь формат даты и времени в универсальном формате ISO 8601 `yyyy-MM-ddTHH:mm:ss.fffffffZ` .

> [!NOTE]
> Исключения политики не удаляются при `expiresOn` достижении даты. Объект сохраняется для сохранения записи, но исключение больше не учитывается.

## <a name="required-permissions"></a>Необходимые разрешения

Разрешения RBAC Azure, необходимые для управления объектами исключения политики, находятся в `Microsoft.Authorization/policyExemptions` группе операций. У [участника политики ресурсов](../../../role-based-access-control/built-in-roles.md#resource-policy-contributor) и [администратора безопасности](../../../role-based-access-control/built-in-roles.md#security-admin) встроенных ролей есть `read` разрешения и, `write` а также [средство записи данных Policy Insights (Предварительная версия)](../../../role-based-access-control/built-in-roles.md#policy-insights-data-writer-preview) `read` .

Исключения имеют дополнительные меры безопасности из-за влияния предоставления исключения. Помимо выполнения `Microsoft.Authorization/policyExemptions/write` операции с иерархией ресурсов или отдельным ресурсом, создатель исключения должен иметь `exempt/Action` команду на целевом назначении.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [структуре определения политик](./definition-structure.md).
- Узнайте о [программном создании политик](../how-to/programmatically-create.md).
- Узнайте, как [получать сведения о соответствии](../how-to/get-compliance-data.md).
- Узнайте, как [исправлять несоответствующие ресурсы](../how-to/remediate-resources.md).
- Дополнительные сведения о группе управления см. в статье [Упорядочивание ресурсов с помощью групп управления Azure](../../management-groups/overview.md).
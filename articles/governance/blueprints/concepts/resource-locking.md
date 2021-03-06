---
title: Концепция блокировки ресурсов
description: Сведения о вариантах блокировки в схемах Azure для защиты ресурсов при назначении схемы.
ms.date: 01/27/2021
ms.topic: conceptual
ms.openlocfilehash: b2004ad294ae0eec1b4f2fc6f49308efd32d652e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98920196"
---
# <a name="understand-resource-locking-in-azure-blueprints"></a>Общие сведения о блокировке ресурсов в Azure Blueprint

Создавать крупные согласованные среды имеет смысл только при наличии механизма, поддерживающего эту согласованность. В этой статье рассказывается, как работает блокировка ресурсов в Azure Blueprint. Пример блокировки ресурсов и применения _запрета назначений_ см. в учебнике [Защита новых ресурсов](../tutorials/protect-new-resources.md) .

> [!NOTE]
> Блокировки ресурсов, развернутые в проекте Azure, применяются только к ресурсам, развернутым с помощью назначения схемы. Существующие ресурсы, например те, которые уже существуют в существующих группах ресурсов, не имеют добавленных блокировок.

## <a name="locking-modes-and-states"></a>Режимы и состояния блокировки

Режим блокировки применяется к назначению схемы и имеет три параметра: **не блокировать**, **только чтение** или не **удалять**. Режим блокировки настраивается во время развертывания артефактов при назначении схемы. Вы можете задать другой режим блокировки, обновив назначение схемы.
Однако режимы блокировки не могут быть изменены за пределами схем Azure.

Ресурсы, созданные артефактами в назначении схемы, имеют четыре состояния: **не заблокировано**, **доступно только для чтения**, **не может быть изменено, удалено** или **не может быть удалено**. Каждый тип артефакта может находиться в состоянии **Не заблокировано**. Следующую таблицу можно использовать для определения состояния ресурса:

|Режим|Тип ресурса артефакта|Состояние|Описание|
|-|-|-|-|
|Не блокировать|*|Не заблокировано|Ресурсы не защищаются с помощью проектных проектов Azure. Это состояние также используется для ресурсов, добавленных к артефакту группы ресурсов **Только чтение** или **Do Not Delete** (Не удалять) за пределами назначения схемы.|
|Только для чтения|Группа ресурсов|Невозможно изменить/удалить|Группа ресурсов доступна только для чтения, и теги в этой группе недоступны для изменения. Ресурсы с состоянием **Не заблокировано** можно добавлять, перемещать, изменять или удалять из этой группы ресурсов.|
|Только для чтения|Нересурсная группа|Только для чтения|Ресурс не может быть изменен каким бы то ни было образом. Нет изменений, и его нельзя удалить.|
|Не удалять|*|Cannot Delete (Невозможно удалить)|Ресурсы можно изменить, но невозможно удалить. Ресурсы с состоянием **Не заблокировано** можно добавлять, перемещать, изменять или удалять из этой группы ресурсов.|

## <a name="overriding-locking-states"></a>Переопределение состояний блокировки

Как правило, для того, чтобы разрешить изменение или удаление любого ресурса в подписке, например роль "владелец", можно использовать соответствующий [Контроль доступа на основе ролей Azure (Azure RBAC)](../../../role-based-access-control/overview.md) . Это не так, когда планы Azure применяют блокировку как часть развернутого назначения. Если назначение было установлено с состоянием **Только чтение** или **Do Not Delete** (Не удалять), даже владелец подписки не сможет выполнять заблокированное действие для защищенного ресурса.

Эта мера предосторожности защищает согласованность определенной схемы и среды, для создания которой эта схема предназначается, от случайного или программного удаления либо изменения.

### <a name="assign-at-management-group"></a>Назначить в группе управления

Единственный вариант, чтобы запретить владельцам подписки удалять назначение схемы, — это назначить проект группе управления. В этом сценарии только **владельцы** группы управления имеют разрешения, необходимые для удаления назначения схемы.

Чтобы назначить схему группе управления, а не подписке, REST API вызвать изменения, чтобы выглядеть следующим образом:

```http
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{assignmentMG}/providers/Microsoft.Blueprint/blueprintAssignments/{assignmentName}?api-version=2018-11-01-preview
```

Группа управления, определенная, `{assignmentMG}` должна находиться либо в иерархии группы управления, либо в той же группе управления, в которой сохраняется определение схемы.

Текст запроса назначения схемы выглядит следующим образом:

```json
{
    "identity": {
        "type": "SystemAssigned"
    },
    "location": "eastus",
    "properties": {
        "description": "enforce pre-defined simpleBlueprint to this XXXXXXXX subscription.",
        "blueprintId": "/providers/Microsoft.Management/managementGroups/{blueprintMG}/providers/Microsoft.Blueprint/blueprints/simpleBlueprint",
        "scope": "/subscriptions/{targetSubscriptionId}",
        "parameters": {
            "storageAccountType": {
                "value": "Standard_LRS"
            },
            "costCenter": {
                "value": "Contoso/Online/Shopping/Production"
            },
            "owners": {
                "value": [
                    "johnDoe@contoso.com",
                    "johnsteam@contoso.com"
                ]
            }
        },
        "resourceGroups": {
            "storageRG": {
                "name": "defaultRG",
                "location": "eastus"
            }
        }
    }
}
```

Ключевым различием в этом тексте запроса и одной из назначенных подписок является `properties.scope` свойство. Для этого обязательного свойства необходимо задать подписку, к которой применяется назначение схемы. Подписка должна быть прямым дочерним элементом иерархии группы управления, в которой хранится назначение схемы.

> [!NOTE]
> Схема, назначенная области группы управления, по-прежнему действует как назначение на уровне подписки. Единственное отличие заключается в том, где хранится назначение схемы, чтобы не допустить владельцев подписки удалять назначение и связанные блокировки.

## <a name="removing-locking-states"></a>Удаление состояний блокировки

Если нужно изменить или удалить ресурс, защищенный назначением, это можно сделать двумя способами.

- Установить для назначения схемы режим блокировки с состоянием **Не блокировать**.
- Удалить назначение схемы.

При удалении назначения будут удалены блокировки, созданные с помощью проектных проектов Azure. Но при этом сам ресурс сохраняется. Его следует удалить обычными средствами.

## <a name="how-blueprint-locks-work"></a>Как работают блокировки схемы

Действие Deny не запрещает [назначение](../../../role-based-access-control/deny-assignments.md) Deny для ресурсов артефактов во время назначения схемы, если для назначения выбран параметр **только чтение** или не **Удаление** . Запрещающее действие добавляется управляемым удостоверением назначения схемы и может быть удалено из ресурсов артефактов только тем же управляемым удостоверением. Эта мера безопасности обеспечивает механизм блокировки и предотвращает снятие блокировки схемы за пределы чертежей Azure.

:::image type="content" source="../media/resource-locking/blueprint-deny-assignment.png" alt-text="Снимок экрана со страницей управления доступом (I а) и вкладкой &quot;отклонить назначения&quot; для группы ресурсов." border="false":::

Ниже перечислены [свойства назначения Deny](../../../role-based-access-control/deny-assignments.md#deny-assignment-properties) для каждого режима.

|Режим |Permissions. Actions |Разрешения. без изменений |Участники [i]. Тип |ЕксклудепринЦипалс [i]. Удостоверения | донотапплиточилдскопес |
|-|-|-|-|-|-|
|Только для чтения |**\** _ |_ *\* /Реад **<br />** Microsoft. Authorization/locks/удалить **<br />** Microsoft. Network/virtualNetwork/подсети/соединение/действие** |Системдефинед (все) |Назначение схемы и определяемые пользователем данные в **ексклудедпринЦипалс** |Группа ресурсов — _true_; Ресурс — _false_ |
|Не удалять |**\*/Delete** | **Microsoft.Authorization/locks/delete**<br />**Microsoft. Network/virtualNetwork/подсети/соединение/действие** |Системдефинед (все) |Назначение схемы и определяемые пользователем данные в **ексклудедпринЦипалс** |Группа ресурсов — _true_; Ресурс — _false_ |

> [!IMPORTANT]
> Azure Resource Manager кэширует сведения о назначении роли на срок до 30 минут. Это означает, что действие запрета назначений для ресурсов схемы может не сразу вступать в полную силу. В течение этого периода сохраняется возможность удалить ресурс, который должен быть защищен блокировками схемы.

## <a name="exclude-a-principal-from-a-deny-assignment"></a>Исключение участника из назначения Deny

В некоторых сценариях проектирования или безопасности может потребоваться исключить субъект из [назначения «Deny](../../../role-based-access-control/deny-assignments.md) », создаваемого назначением схемы. Этот шаг выполняется в REST API путем добавления до пяти значений в массив **ексклудедпринЦипалс** в свойстве **locks** при [создании назначения](/rest/api/blueprints/assignments/createorupdate). Следующее определение назначения является примером текста запроса, который содержит **ексклудедпринЦипалс**:

```json
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "eastus",
  "properties": {
    "description": "enforce pre-defined simpleBlueprint to this XXXXXXXX subscription.",
    "blueprintId": "/providers/Microsoft.Management/managementGroups/{mgId}/providers/Microsoft.Blueprint/blueprints/simpleBlueprint",
    "locks": {
        "mode": "AllResourcesDoNotDelete",
        "excludedPrincipals": [
            "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
            "38833b56-194d-420b-90ce-cff578296714"
        ]
    },
    "parameters": {
      "storageAccountType": {
        "value": "Standard_LRS"
      },
      "costCenter": {
        "value": "Contoso/Online/Shopping/Production"
      },
      "owners": {
        "value": [
          "johnDoe@contoso.com",
          "johnsteam@contoso.com"
        ]
      }
    },
    "resourceGroups": {
      "storageRG": {
        "name": "defaultRG",
        "location": "eastus"
      }
    }
  }
}
```

## <a name="exclude-an-action-from-a-deny-assignment"></a>Исключить действие из назначения Deny

Подобно [исключению субъекта](#exclude-a-principal-from-a-deny-assignment) в назначении [Deny](../../../role-based-access-control/deny-assignments.md) в назначении схемы, можно исключить определенные [операции поставщика ресурсов Azure](../../../role-based-access-control/resource-provider-operations.md). В блоке **Properties. locks** в том же месте, в котором **ексклудедпринЦипалс** , можно добавить **ексклудедактионс** :

```json
"locks": {
    "mode": "AllResourcesDoNotDelete",
    "excludedPrincipals": [
        "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
        "38833b56-194d-420b-90ce-cff578296714"
    ],
    "excludedActions": [
        "Microsoft.ContainerRegistry/registries/push/write",
        "Microsoft.Authorization/*/read"
    ]
},
```

Хотя **ексклудедпринЦипалс** должны быть явными, записи **ексклудедактионс** могут использовать `*` для поиска с подстановочными знаками операций поставщика ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

- Следуйте указаниям в учебнике [Защита новых ресурсов](../tutorials/protect-new-resources.md) .
- Ознакомьтесь со сведениями о [жизненном цикле схем](./lifecycle.md).
- Узнайте, как использовать [статические и динамические параметры](./parameters.md).
- Научитесь настраивать [последовательность схемы](./sequencing-order.md).
- Узнайте, как [обновлять существующие назначения](../how-to/update-existing-assignments.md).
- Устраняйте проблемы, возникающие во время назначения схемы, с помощью [общих инструкций по устранению неполадок](../troubleshoot/general.md).

---
title: Список запрещенных назначений Azure с помощью REST API-Azure RBAC
description: Узнайте, как получить список запрещенных назначений Azure для пользователей, групп и приложений с помощью REST API и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
documentationcenter: na
author: rolyon
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: role-based-access-control
ms.workload: multiple
ms.tgt_pltfrm: rest-api
ms.devlang: na
ms.topic: how-to
ms.date: 03/19/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 2f835c270930734bf9963a7c7c3168b873eddaf6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84791917"
---
# <a name="list-azure-deny-assignments-using-the-rest-api"></a>Список запрещенных назначений в Azure с помощью REST API

[Запрещенные назначения Azure](deny-assignments.md) запрещают пользователям выполнять определенные действия с ресурсами Azure, даже если назначение ролей предоставляет им доступ. В этой статье описывается, как вывести список запрещенных назначений с помощью REST API.

> [!NOTE]
> Вы не можете напрямую создавать собственные назначения Deny. Дополнительные сведения о создании назначений Deny см. в статье [запрещение назначений в Azure](deny-assignments.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы получить сведения о назначении Deny, необходимо следующее:

- `Microsoft.Authorization/denyAssignments/read` разрешение, которое входит в большинство [встроенных ролей Azure](built-in-roles.md).

## <a name="list-a-single-deny-assignment"></a>Вывод списка определенного запрета назначения

1. Можете начать со следующего запроса:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments/{deny-assignment-id}?api-version=2018-07-01-preview
    ```

1. Внутри URI замените *{scope}* областью, для которой требуется создать список запретов назначений.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId}` | Подписка |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Группа ресурсов |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/providers/Microsoft.Web/sites/mysite1` | Ресурс |

1. Замените *{deny-assignment-id}* идентификатором запрета назначения, которое необходимо получить.

## <a name="list-multiple-deny-assignments"></a>Вывод списка нескольких запретов назначений

1. Начните работу с любого из следующих запросов.

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview
    ```

    С использованием необязательных параметров:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter={filter}
    ```

1. Внутри URI замените *{scope}* областью, для которой требуется создать список запретов назначений.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId}` | Подписка |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Группа ресурсов |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/providers/Microsoft.Web/sites/mysite1` | Ресурс |

1. Замените *{filter}* условием, по которому требуется отфильтровать список запретов назначений.

    > [!div class="mx-tableFixed"]
    > | Filter | Описание |
    > | --- | --- |
    > | (без фильтра) | Список всех запретов назначений в указанной области, выше и ниже. |
    > | `$filter=atScope()` | Список запрещает назначения только для указанной области и более поздних версий. Не включаются запреты назначений во внутренних областях. |
    > | `$filter=assignedTo('{objectId}')` | Список запрещает назначения для указанного пользователя или субъекта-службы.<br/>Если пользователь является членом группы, которая имеет назначение Deny, это также приводит к запрету назначения. Этот фильтр является транзитивным для групп. Это означает, что если пользователь является членом группы, а эта группа является членом другой группы с назначенным запретом, то также будет указано, что оно запрещает назначение.<br/>Этот фильтр принимает только идентификатор объекта для пользователя или субъекта-службы. Невозможно передать идентификатор объекта для группы. |
    > | `$filter=atScope()+and+assignedTo('{objectId}')` | Список запрещает назначения для указанного пользователя или субъекта-службы в указанной области. |
    > | `$filter=denyAssignmentName+eq+'{deny-assignment-name}'` | Список запрещает назначения с указанным именем. |
    > | `$filter=principalId+eq+'{objectId}'` | Список запрещает назначения для указанного пользователя, группы или субъекта-службы. |

## <a name="list-deny-assignments-at-the-root-scope-"></a>Вывод списка запретов назначений в корневой области (/)

1. Повысьте уровень доступа, как описано в разделе [повышение уровня доступа для управления всеми подписками Azure и группами управления](elevate-access-global-admin.md).

1. Используйте следующий запрос:

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter={filter}
    ```

1. Замените *{filter}* условием, по которому требуется отфильтровать список запретов назначений. Необходимо указать фильтр.

    > [!div class="mx-tableFixed"]
    > | Filter | Описание |
    > | --- | --- |
    > | `$filter=atScope()` | Вывод списка запретов назначений только в корневой области. Не включаются запреты назначений во внутренних областях. |
    > | `$filter=denyAssignmentName+eq+'{deny-assignment-name}'` | Вывод списка всех запретов назначений с указанным названием. |

1. Удалите повышенный уровень доступа.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о назначении Deny в Azure](deny-assignments.md)
- [Повышение прав доступа для управления всеми подписками Azure и группами управления](elevate-access-global-admin.md)
- [Справочник по REST API Azure](/rest/api/azure/)

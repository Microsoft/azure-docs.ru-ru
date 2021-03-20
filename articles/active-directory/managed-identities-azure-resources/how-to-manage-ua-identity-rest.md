---
title: Управление назначенными пользователем управляемыми удостоверениями с помощью функции RESTFUL в Azure AD
description: Пошаговые инструкции по созданию, отчислению и удалению назначенного пользователем управляемого удостоверения для выполнения REST APIных вызовов.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/02/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 117f9a1c173f2083dd4621f4f3f41b6e83d1d46b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96546698"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-rest-api-calls"></a>Создание, перечисление или удаление назначенного пользователем управляемого удостоверения с помощью вызовов REST API

Управляемые удостоверения для ресурсов Azure предоставляют службам Azure возможность проверки подлинности в службах, поддерживающих проверку подлинности Azure AD, без необходимости учетных данных в коде. 

В этой статье вы узнаете, как создавать и удалять управляемые удостоверения, назначаемые пользователем, а также получать их список с помощью CURL для выполнения вызовов REST API.

## <a name="prerequisites"></a>Предварительные требования

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, изучите [общие сведения](overview.md). **Обратите внимание на [различие между управляемыми удостоверениями, назначаемыми системой и назначаемыми пользователями](overview.md#managed-identity-types)**.
- Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.
- Все команды, приведенные в этой статье, можно выполнить в облаке или локально:
    - для выполнения команд в облаке используйте [Azure Cloud Shell](../../cloud-shell/overview.md);
    - Для запуска в локальной среде установите [фигурные](https://curl.haxx.se/download.html) и [Azure CLI](/cli/azure/install-azure-cli).

## <a name="obtain-a-bearer-access-token"></a>Получение маркера доступа носителя

1. При локальном запуске Войдите в Azure с помощью Azure CLI:

    ```
    az login
    ```

1. Получите маркер доступа с помощью команды [AZ Account Get-Access-Token](/cli/azure/account#az_account_get_access_token) .

    ```azurecli-interactive
    az account get-access-token
    ```

## <a name="create-a-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначаемого пользователем 

Чтобы создать назначаемое пользователем управляемое удостоверение, учетной записи должна быть назначена роль [участника управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X PUT -d '{"loc
ation": "<LOCATION>"}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```

**Заголовки запроса**

|Заголовок запроса  |Описание  |
|---------|---------|
|*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
|*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        |

**Текст запроса**

|Имя  |Описание  |
|---------|---------|
|location     | Обязательный. Расположение ресурса.        |

## <a name="list-user-assigned-managed-identities"></a>Получение списка управляемых удостоверений, назначаемых пользователем

Чтобы получить список управляемых удостоверений, назначаемых пользователем, или отобразить их, учетной записи должна быть назначена роль [оператора управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator) или [участника управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview' -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview HTTP/1.1
```

|Заголовок запроса  |Описание  |
|---------|---------|
|*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
|*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        |

## <a name="delete-a-user-assigned-managed-identity"></a>Удаление управляемого удостоверения, назначаемого пользователем

Чтобы удалить назначаемое пользователем управляемое удостоверение, учетной записи должна быть назначена роль [участника управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

> [!NOTE]
> При удалении управляемого удостоверения, назначаемого пользователем, не удаляются ссылки из ресурсов, которым оно было назначено. Чтобы удалить управляемое удостоверение, назначаемое пользователем, из виртуальной машины с помощью CURL, обратитесь к разделу [Удаление назначенного пользователем удостоверения из виртуальной машины Azure](qs-configure-rest-vm.md#remove-a-user-assigned-managed-identity-from-an-azure-vm).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X DELETE -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
DELETE https://management.azure.com/subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourceGroups/TestRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```
|Заголовок запроса  |Описание  |
|---------|---------|
|*Content-Type*     | Обязательный элемент. Задайте значение `application/json`.        |
|*Авторизация*     | Обязательный элемент. Задайте допустимый маркер доступа для `Bearer`.        |

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как задать управляемое удостоверение, назначаемое пользователем, для виртуальной машины или масштабируемого набора виртуальных машин Azure, см. в разделах [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью вызовов REST API](qs-configure-rest-vm.md#user-assigned-managed-identity) и [Настройка управляемых удостоверений для ресурсов Azure в масштабируемом наборе виртуальных машин с помощью вызовов REST API](qs-configure-rest-vmss.md#user-assigned-managed-identity).

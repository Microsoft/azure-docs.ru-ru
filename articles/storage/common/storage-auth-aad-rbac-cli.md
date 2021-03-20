---
title: Назначение роли Azure для доступа к данным с помощью Azure CLI
titleSuffix: Azure Storage
description: Узнайте, как использовать Azure CLI для назначения разрешений субъекту безопасности Azure Active Directory с помощью управления доступом на основе ролей Azure (Azure RBAC). Служба хранилища Azure поддерживает встроенные и пользовательские роли Azure для проверки подлинности с помощью Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/10/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: c42061520b73966f2cd516716039d78c2b9cbeb8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100375992"
---
# <a name="use-azure-cli-to-assign-an-azure-role-for-access-to-blob-and-queue-data"></a>Использование Azure CLI для назначения роли Azure доступа к данным BLOB-объектов и очередей

Azure Active Directory (Azure AD) разрешает права доступа к защищенным ресурсам через [Управление доступом на основе ролей Azure (Azure RBAC)](../../role-based-access-control/overview.md). Служба хранилища Azure определяет набор встроенных ролей Azure, охватывающих общие наборы разрешений, используемых для доступа к данным BLOB-объектов или очередей.

Когда роль Azure назначается субъекту безопасности Azure AD, Azure предоставляет доступ к этим ресурсам для этого субъекта безопасности. Доступ может ограничиваться уровнем подписки, группой ресурсов, учетной записью хранения или отдельным контейнером или очередью. Субъект безопасности Azure AD может быть пользователем, группой, субъектом-службой приложения или [управляемым удостоверением для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md).

В этой статье описывается, как использовать Azure CLI для перечисления встроенных ролей Azure и назначения их пользователям. Дополнительные сведения об использовании Azure CLI см. в статье [интерфейс Command-Line Azure (CLI)](/cli/azure).

## <a name="azure-roles-for-blobs-and-queues"></a>Роли Azure для больших двоичных объектов и очередей

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

## <a name="determine-resource-scope"></a>Определение области действия ресурса

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="list-available-azure-roles"></a>Вывод списка доступных ролей Azure

Чтобы получить список доступных встроенных ролей Azure с Azure CLI, используйте команду [AZ Role Definition List](/cli/azure/role/definition#az-role-definition-list) .

```azurecli-interactive
az role definition list --out table
```

Вы увидите список встроенных ролей данных службы хранилища Azure вместе с другими встроенными ролями Azure:

```Example
Storage Blob Data Contributor             Allows for read, write and delete access to Azure Storage blob containers and data
Storage Blob Data Owner                   Allows for full access to Azure Storage blob containers and data, including assigning POSIX access control.
Storage Blob Data Reader                  Allows for read access to Azure Storage blob containers and data
Storage Queue Data Contributor            Allows for read, write, and delete access to Azure Storage queues and queue messages
Storage Queue Data Message Processor      Allows for peek, receive, and delete access to Azure Storage queue messages
Storage Queue Data Message Sender         Allows for sending of Azure Storage queue messages
Storage Queue Data Reader                 Allows for read access to Azure Storage queues and queue messages
```

## <a name="assign-an-azure-role-to-a-security-principal"></a>Назначение роли Azure субъекту безопасности

Чтобы назначить роль Azure субъекту безопасности, используйте команду [AZ Role назначение Create](/cli/azure/role/assignment#az-role-assignment-create) . Формат команды может различаться в зависимости от области назначения. В следующих примерах показано, как назначить роль пользователю в различных областях, но можно использовать ту же команду, чтобы назначить роль любому субъекту безопасности.

> [!IMPORTANT]
> При создании учетной записи хранения Azure вы не назначаете разрешения на доступ к данным через Azure AD автоматически. Необходимо явно назначить себе роль RBAC Azure для доступа к данным. Вы можете назначить ее на уровне подписки, группы ресурсов, учетной записи хранения, контейнера или очереди.
>
> Если учетная запись хранения заблокирована с блокировкой только для чтения Azure Resource Manager, блокировка не позволит назначить роли RBAC Azure, областью действия которых является учетная запись хранения или контейнер данных (контейнер BLOB-объектов или очередь).

### <a name="container-scope"></a>Область контейнера

Чтобы назначить роль для контейнера, укажите строку, содержащую область действия контейнера для `--scope` параметра. Область для контейнера имеет вид:

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container>
```

В следующем примере роль **участника данных BLOB-объекта хранилища** назначается пользователю в области действия уровня контейнера. Обязательно замените образцы значений и значения заполнителей в квадратных скобках собственными значениями:

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container>"
```

### <a name="queue-scope"></a>Область очереди

Чтобы назначить роль, ограниченную очередью, укажите строку, содержащую область действия очереди для `--scope` параметра. Область для очереди имеет вид:

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/<queue>
```

В следующем примере роль **участника данных очереди хранилища** назначается пользователю в области действия уровня очереди. Обязательно замените образцы значений и значения заполнителей в квадратных скобках собственными значениями:

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/<queue>"
```

### <a name="storage-account-scope"></a>Область учетной записи хранения

Чтобы назначить роль, ограниченную учетной записью хранения, укажите область ресурса учетной записи хранения для `--scope` параметра. Область для учетной записи хранения имеет вид:

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

В следующем примере показано, как назначить роль **модуля чтения данных большого двоичного объекта хранилища** пользователю на уровне учетной записи хранения. Обязательно замените образцы значений собственными значениями: \

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

### <a name="resource-group-scope"></a>Область группы ресурсов

Чтобы назначить роль, ограниченную группой ресурсов, укажите имя или идентификатор группы ресурсов для `--resource-group` параметра. В следующем примере роль **модуля чтения данных очереди хранилища** назначается пользователю на уровне группы ресурсов. Не забудьте заменить образцы значений и значения заполнителей в квадратных скобках собственными значениями:

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Reader" \
    --assignee <email> \
    --resource-group <resource-group>
```

### <a name="subscription-scope"></a>Область действия подписки

Чтобы назначить роль, ограниченную подпиской, укажите область действия подписки для `--scope` параметра. Область для подписки имеет вид:

```
/subscriptions/<subscription>
```

В следующем примере показано, как назначить роль **модуля чтения данных большого двоичного объекта хранилища** пользователю на уровне учетной записи хранения. Обязательно замените образцы значений собственными значениями: 

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Добавление или удаление назначений ролей Azure с помощью модуля Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
- [Использование модуля Azure PowerShell для назначения роли Azure доступа к данным BLOB-объектов и очередей](storage-auth-aad-rbac-powershell.md)
- [Назначение роли Azure для доступа к большим двоичным объектам и данным очереди с помощью портала Azure](storage-auth-aad-rbac-portal.md)

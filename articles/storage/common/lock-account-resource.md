---
title: Применение блокировки Azure Resource Manager к учетной записи хранения
titleSuffix: Azure Storage
description: Узнайте, как применить блокировку Azure Resource Manager к учетной записи хранения.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/09/2021
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: 9d80c0b8d4d913322c47d1ad278d6dbc033d2409
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102620153"
---
# <a name="apply-an-azure-resource-manager-lock-to-a-storage-account"></a>Применение блокировки Azure Resource Manager к учетной записи хранения

Корпорация Майкрософт рекомендует блокировать все учетные записи хранения с блокировкой Azure Resource Manager, чтобы предотвратить случайное или вредоносное удаление учетной записи хранения. Существует два типа Azure Resource Manager блокировок ресурсов:

- Блокировка **CannotDelete** запрещает пользователям удалять учетную запись хранения, но позволяет считывать и изменять ее конфигурацию.
- Блокировка **только для чтения** запрещает пользователям удалять учетную запись хранения или изменять ее конфигурацию, но позволяет считывать конфигурацию.

Дополнительные сведения о блокировках Azure Resource Manager см. [в разделе Блокировка ресурсов для предотвращения изменений](../../azure-resource-manager/management/lock-resources.md).

> [!IMPORTANT]
> При блокировке учетной записи хранения данные в этой учетной записи не защищаются от обновления или удаления.

## <a name="configure-an-azure-resource-manager-lock"></a>Настройка блокировки Azure Resource Manager

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы настроить блокировку учетной записи хранения с портал Azure, выполните следующие действия.

1. Войдите в свою учетную запись хранения на портале Azure.
1. В разделе **Параметры** выберите **блокировки**.
1. Выберите **Добавить**.
1. Укажите имя блокировки ресурса и укажите тип блокировки. При необходимости добавьте примечание о блокировке.

    :::image type="content" source="media/lock-account-resource/lock-storage-account.png" alt-text="Снимок экрана, показывающий, как заблокировать учетную запись хранения с блокировкой CannotDelete":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы настроить блокировку учетной записи хранения с помощью PowerShell, сначала убедитесь, что установлен [модуль AZ PowerShell](https://www.powershellgallery.com/packages/Az). Затем вызовите команду [New-азресаурцелокк](/powershell/module/az.resources/new-azresourcelock) и укажите тип блокировки, которую необходимо создать, как показано в следующем примере:

```azurepowershell
New-AzResourceLock -LockLevel CanNotDelete `
    -LockName <lock> `
    -ResourceName <storage-account> `
    -ResourceType Microsoft.Storage/storageAccounts `
    -ResourceGroupName <resource-group>
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы настроить блокировку учетной записи хранения с Azure CLI, вызовите команду [AZ Lock Create](/cli/azure/lock#az_lock_create) и укажите тип блокировки, которую необходимо создать, как показано в следующем примере:

```azurecli
az lock create \
    --name <lock> \
    --resource-group <resource-group> \
    --resource <storage-account> \
    --lock-type CanNotDelete \
    --resource-type Microsoft.Storage/storageAccounts
```

---

## <a name="authorizing-data-operations-when-a-readonly-lock-is-in-effect"></a>Авторизация операций с данными при включенной блокировке только для чтения

Если к учетной записи хранения применяется блокировка **только для чтения** , операция " [список ключей](/rest/api/storagerp/storageaccounts/listkeys) " блокируется для этой учетной записи хранения. Операция **List Keys** является ОПЕРАЦИЕЙ HTTPS POST, и все операции POST предотвращаются, если для учетной записи настроена блокировка **только для чтения** . Операция " **список ключей** " возвращает ключи доступа к учетной записи, которые затем можно использовать для чтения и записи данных в учетной записи хранения.

Если клиент использует ключи доступа к учетной записи во время применения блокировки к учетной записи хранения, этот клиент может продолжать использовать ключи для доступа к данным. Однако клиенты, не имеющие доступа к ключам, должны будут использовать учетные данные Azure Active Directory (Azure AD) для доступа к данным BLOB-объектов или очередей в учетной записи хранения.

Пользователи портал Azure могут быть затронуты при применении блокировки **только для чтения** , если они ранее обращались к данным большого двоичного объекта или очереди на портале с ключами доступа к учетной записи. После применения блокировки пользователи портала должны будут использовать учетные данные Azure AD для доступа к данным BLOB-объектов или очередей на портале. Для этого пользователю должны быть назначены по крайней мере две роли RBAC: роль модуля чтения Azure Resource Manager как минимум, и одна из ролей доступа к данным службы хранилища Azure. Дополнительные сведения см. в следующих статьях:

- [Выберите способ авторизации доступа к данным BLOB-объектов в портал Azure](../blobs/authorize-data-operations-portal.md)
- [Выберите способ авторизации доступа к данным очереди в портал Azure](../queues/authorize-data-operations-portal.md)

Данные в службе "файлы Azure" или "Служба таблиц" могут стать недоступными для клиентов, которые ранее обращались к ней с ключами учетной записи. Рекомендуется применять блокировку **только для чтения** к учетной записи хранения, а затем перемещать рабочие нагрузки службы файлов и таблиц Azure в учетную запись хранения, которая не заблокирована с блокировкой **ReadOnly** .

## <a name="next-steps"></a>Дальнейшие действия

[Блокировка ресурсов для предотвращения изменений](../../azure-resource-manager/management/lock-resources.md)

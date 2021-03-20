---
title: Определение модели ключа шифрования, используемой для учетной записи хранения
titleSuffix: Azure Storage
description: Используйте портал Azure, PowerShell или Azure CLI, чтобы проверить управление ключами шифрования для учетной записи хранения. Ключи могут управляться корпорацией Майкрософт (по умолчанию) или клиентом. Ключи, управляемые клиентом, должны храниться в Azure Key Vault.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f59e4238c983cdb336500a68c52730ae5346b1c7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91612427"
---
# <a name="determine-which-azure-storage-encryption-key-model-is-in-use-for-the-storage-account"></a>Определение модели ключа шифрования службы хранилища Azure, используемой для учетной записи хранения

Данные в учетной записи хранения автоматически шифруются службой хранилища Azure. Шифрование службы хранилища Azure предлагает два варианта управления ключами шифрования на уровне учетной записи хранения.

- **Ключи, управляемые корпорацией Майкрософт.** По умолчанию Корпорация Майкрософт управляет ключами, используемыми для шифрования учетной записи хранения.
- **Ключи, управляемые клиентом.** При необходимости можно выбрать управление ключами шифрования для учетной записи хранения. Ключи, управляемые клиентом, должны храниться в Azure Key Vault.

Кроме того, можно предоставить ключ шифрования на уровне отдельного запроса для некоторых операций с хранилищем BLOB-объектов. Если в запросе указан ключ шифрования, этот ключ переопределяет ключ шифрования, который активен в учетной записи хранения. Дополнительные сведения см. в разделе [Указание предоставленного клиентом ключа для запроса к хранилищу BLOB-объектов](../blobs/storage-blob-customer-provided-key.md).

Дополнительные сведения о ключах шифрования см. [в статье шифрование службы хранилища Azure для неактивных данных](storage-service-encryption.md).

## <a name="check-the-encryption-key-model-for-the-storage-account"></a>Проверка модели ключа шифрования для учетной записи хранения

Чтобы определить, использует ли учетная запись хранения ключи, управляемые корпорацией Майкрософт, или ключи, управляемые клиентом, для шифрования используйте один из следующих подходов.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы проверить модель шифрования для учетной записи хранения с помощью портал Azure, выполните следующие действия.

1. Войдите в свою учетную запись хранения на портале Azure.
1. Выберите параметр **шифрования** и обратите внимание на параметр.

На следующем рисунке показана учетная запись хранения, зашифрованная с помощью ключей, управляемых корпорацией Майкрософт:

![Просмотр учетной записи, зашифрованной с помощью ключей, управляемых корпорацией Майкрософт](media/storage-encryption-key-model-get/microsoft-managed-encryption-key-setting-portal.png)

На следующем рисунке показана учетная запись хранения, зашифрованная с помощью управляемых клиентом ключей.

![Снимок экрана, показывающий параметр ключа шифрования в портал Azure](media/storage-encryption-key-model-get/customer-managed-encryption-key-setting-portal.png)

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы проверить модель шифрования для учетной записи хранения с помощью PowerShell, вызовите команду [Get-азсторажеаккаунт](/powershell/module/az.storage/get-azstorageaccount) , а затем проверьте свойство **KeySource** для учетной записи.

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account>
$account.Encryption.KeySource
```

Если свойство **KeySource** имеет значение `Microsoft.Storage` , то учетная запись шифруется с помощью ключей, управляемых корпорацией Майкрософт. Если свойство **KeySource** имеет значение `Microsoft.Keyvault` , то учетная запись шифруется с помощью ключей, управляемых клиентом.

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы проверить модель шифрования для учетной записи хранения с помощью Azure CLI, вызовите команду [AZ Storage Account показывать](/cli/azure/storage/account#az-storage-account-show) , а затем проверьте свойство **keySource** для учетной записи.

```azurecli-interactive
key_source=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query encryption.keySource \
    --output tsv)
```

Если свойство **keySource** имеет значение `Microsoft.Storage` , то учетная запись шифруется с помощью ключей, управляемых корпорацией Майкрософт. Если свойство **keySource** имеет значение `Microsoft.Keyvault` , то учетная запись шифруется с помощью ключей, управляемых клиентом.

---

## <a name="next-steps"></a>Дальнейшие действия

- [Шифрование службы хранилища Azure для неактивных данных](storage-service-encryption.md)
- [Управляемые клиентом ключи для шифрования службы хранилища Azure](customer-managed-keys-overview.md)

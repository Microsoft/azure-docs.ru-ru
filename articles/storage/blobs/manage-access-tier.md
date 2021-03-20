---
title: Управление уровнем доступа большого двоичного объекта в учетной записи хранения Azure
description: Узнайте, как изменить уровень большого двоичного объекта в GPv2 или учетной записи хранилища BLOB-объектов.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 01/11/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.reviewer: klaasl
ms.openlocfilehash: 183593022c934eaf52ffe18649c23e8deced34d8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98166517"
---
# <a name="manage-the-access-tier-of-a-blob-in-an-azure-storage-account"></a>Управление уровнем доступа большого двоичного объекта в учетной записи хранения Azure

Каждый большой двоичный объект имеет уровень доступа по умолчанию: "горячий", "холодного" или "архивный". Большой двоичный объект принимает на себя уровень доступа по умолчанию учетной записи хранения Azure, в которой она создается. Учетные записи хранилища BLOB-объектов и GPv2 предоставляют атрибут **уровня доступа** на уровне учетной записи. Этот атрибут задает уровень доступа по умолчанию для любого большого двоичного объекта, который не задан явным образом на уровне объекта. Для объектов с уровнем, заданным на уровне объектов, уровень учетной записи не применяется. Уровень архива можно применять только на уровне объектов. Вы можете переключаться между уровнями доступа в любое время, выполнив следующие действия.

## <a name="change-the-tier-of-a-blob-in-a-gpv2-or-blob-storage-account"></a>Изменение уровня большого двоичного объекта в GPv2 или учетной записи хранилища BLOB-объектов

В следующих сценариях используется портал Azure или PowerShell:

# <a name="portal"></a>[Портал](#tab/portal)

1. Войдите на [портал Azure](https://portal.azure.com).

1. На портале Azure найдите и выберите **Все ресурсы**.

1. Затем выберите учетную запись хранения.

1. Выберите свой контейнер и большой двоичный объект.

1. В разделе **Свойства BLOB-объекта** выберите **Изменение уровня**.

1. Выберите " **горячий**", " **холодного**" или **архивный** уровень доступа. Если ваш большой двоичный объект в данный момент находится в архиве и вы хотите выполнить восстановление на уровне сети, можно также выбрать параметр "использовать приоритет" (" **стандартный** " или " **высокий**").

1. Щелкните **Сохранить** внизу.

![Изменение уровня BLOB-объекта в портал Azure](media/storage-tiers/blob-access-tier.png)

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Для изменения уровня BLOB-объектов можно использовать следующий сценарий PowerShell. Переменная `$rgName` должна быть инициализирована с помощью имени группы ресурсов. Переменная `$accountName` должна быть инициализирована с помощью имени учетной записи хранения. Переменная `$containerName` должна быть инициализирована с помощью имени контейнера. Переменная `$blobName` должна быть инициализирована с помощью имени большого двоичного объекта.

```powershell
#Initialize the following with your resource group, storage account, container, and blob names
$rgName = ""
$accountName = ""
$containerName = ""
$blobName == ""

#Select the storage account and get the context
$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

#Select the blob from a container
$blob = Get-AzStorageBlob -Container $containerName -Blob $blobName -Context $ctx

#Change the blob’s access tier to archive
$blob.ICloudBlob.SetStandardBlobTier("Archive")
```

---

## <a name="next-steps"></a>Дальнейшие действия

- [Как управлять уровнем доступа учетной записи по умолчанию для учетной записи хранения Azure](../common/manage-account-default-access-tier.md)
- [Дополнительные сведения о восстановлении данных больших двоичных объектов из уровня архива](storage-blob-rehydration.md)
- [Проверьте цены на "горячий", "холодный" и "Архив" в учетных записях хранилища BLOB-объектов по регионам](https://azure.microsoft.com/pricing/details/storage/)

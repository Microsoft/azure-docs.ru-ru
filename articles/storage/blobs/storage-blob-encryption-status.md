---
title: Проверка состояния шифрования большого двоичного объекта в службе хранилища Azure
description: Узнайте, как использовать портал Azure, PowerShell или Azure CLI, чтобы проверить, зашифрован ли заданный BLOB-объект. Если BLOB-объект не зашифрован, Узнайте, как использовать AzCopy для принудительного шифрования путем скачивания и повторной отправки большого двоичного объекта.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/26/2019
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: ee54357250e3f31ef9db633d933d897fff362f48
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98878567"
---
# <a name="check-the-encryption-status-of-a-blob"></a>Проверка состояния шифрования большого двоичного объекта

Каждый блочный большой двоичный объект, добавочный большой двоичный объект или страничный BLOB-объект, записанный в службу хранилища Azure после 20 октября 2017, шифруется с помощью шифрования службы хранилища Azure. Большие двоичные объекты, созданные до этой даты, продолжают шифроваться фоновым процессом.

В этой статье показано, как определить, зашифрован ли данный большой двоичный объект.

## <a name="check-a-blobs-encryption-status"></a>Проверка состояния шифрования BLOB-объекта

Чтобы определить, зашифрован ли большой двоичный объект без кода, используйте портал Azure, PowerShell или Azure CLI.

### <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы использовать портал Azure для проверки того, зашифрован ли большой двоичный объект, выполните следующие действия.

1. Войдите в свою учетную запись хранения на портале Azure.
1. Выберите **контейнеры** для перехода к списку контейнеров в учетной записи.
1. Перейдите к большому двоичному объекту и отобразите его вкладку **Обзор** .
1. Просмотр свойства, **зашифрованного сервером** . Если **значение true**, как показано на следующем рисунке, большой двоичный объект шифруется. Обратите внимание, что свойства большого двоичного объекта также включают дату и время создания большого двоичного объекта.

    ![Снимок экрана, показывающий, как проверить свойство, зашифрованное сервером, в портал Azure](media/storage-blob-encryption-status/blob-encryption-property-portal.png)

### <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы использовать PowerShell для проверки того, зашифрован ли большой двоичный объект, проверьте свойство **иссерверенкриптед** большого двоичного объекта. Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account>
$blob = Get-AzStorageBlob -Context $account.Context `
    -Container <container> `
    -Blob <blob>
$blob.ICloudBlob.Properties.IsServerEncrypted
```

Чтобы определить момент создания большого двоичного объекта, проверьте значение **созданного** свойства:

```powershell
$blob.ICloudBlob.Properties.IsServerEncrypted
```

### <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы использовать Azure CLI для проверки того, зашифрован ли большой двоичный объект, проверьте свойство **иссерверенкриптед** большого двоичного объекта. Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli-interactive
az storage blob show \
    --account-name <storage-account> \
    --container-name <container> \
    --name <blob> \
    --query "properties.serverEncrypted"
```

Чтобы определить момент создания большого двоичного объекта, проверьте значение **созданного** свойства.

---

## <a name="force-encryption-of-a-blob"></a>Принудительное шифрование большого двоичного объекта

Если большой двоичный объект, созданный до 20 октября, 2017 еще не зашифрован фоновым процессом, можно принудительно включить шифрование, загрузив и повторно отгрузив большой двоичный объект. Простой способ сделать это — с помощью AzCopy.

Чтобы скачать большой двоичный объект в локальную файловую систему с помощью AzCopy, используйте следующий синтаксис:

```
azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'

Example:
azcopy copy 'https://storagesamples.blob.core.windows.net/sample-container/blob1.txt' 'C:\temp\blob1.txt'
```

Чтобы повторно отправить большой двоичный объект в службу хранилища Azure с помощью AzCopy, используйте следующий синтаксис:

```
azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'

Example:
azcopy copy 'C:\temp\blob1.txt' 'https://storagesamples.blob.core.windows.net/sample-container/blob1.txt'
```

Дополнительные сведения об использовании AzCopy для копирования данных BLOB-объектов см. в статье [Перенос данных с помощью AzCopy и хранилища BLOB-объектов](../common/storage-use-azcopy-v10.md#transfer-data).

## <a name="next-steps"></a>Дальнейшие действия

[Шифрование службы хранилища Azure для неактивных данных](../common/storage-service-encryption.md)

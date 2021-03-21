---
title: Включение обратимого удаления — общие папки Azure
description: Узнайте, как включить обратимое удаление в общих папках Azure для восстановления данных и предотвращения случайного удаления.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 12/01/2020
ms.author: rogarana
ms.subservice: files
services: storage
ms.openlocfilehash: 77381700f4257006b50e56ab7ffc037ef99d297c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102218559"
---
# <a name="enable-soft-delete-on-azure-file-shares"></a>Включение обратимого удаления — общие папки Azure

Служба хранилища Azure теперь предоставляет возможность обратимого удаления для общих папок. Это упрощает восстановление данных, если они ошибочно удалены приложением или другим пользователем учетной записи хранения. Дополнительные сведения об обратимом удалении см. в статье [Как предотвратить случайное удаление общих папок Azure](storage-files-prevent-file-share-deletion.md).

В следующих разделах показано, как включить обратимое удаление для общих папок Azure в существующей учетной записи хранения.

# <a name="portal"></a>[Портал](#tab/azure-portal)

## <a name="getting-started"></a>Начало работы

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Перейдите к своей учетной записи хранения и выберите **Обратимое удаление** в разделе **Служба файлов**.
1. Выберите **включено** для параметра **обратимое удаление общих папок**.
1. Выберите **Срок хранения общей папки в днях** и введите число, которое вы выбрали.
1. Выберите **Сохранить**, чтобы подтвердить параметры хранения данных.

:::image type="content" source="media/storage-how-to-recover-deleted-account/enable-soft-delete-files.png" alt-text="Снимок экрана: область параметров обратимого удаления учетной записи хранения. Выделите раздел общих папок, включите переключатель, задайте срок хранения и сохраните. Это приведет к включению обратимого удаления для всех общих папок в вашей учетной записи хранения.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Командлеты обратимого удаления доступны в версии 2.1.3 и более новой версии [модуля Azure CLI](/cli/azure/install-azure-cli).

## <a name="getting-started-with-cli"></a>Начало работы с интерфейсом командной строки

Чтобы включить обратимое удаление, нужно обновить свойства службы клиента файлов. В следующем примере включено обратимое удаление для всех общих папок в учетной записи хранения:

```azurecli
az storage account file-service-properties update --enable-delete-retention true -n yourStorageaccount -g yourResourceGroup
```

Проверить, включено ли обратимое удаление, и просмотреть его политику хранения можно с помощью следующей команды:

```azurecli
az storage account file-service-properties show -n yourStorageaccount -g yourResourceGroup
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

## <a name="prerequisite"></a>Предварительное требование

Командлеты обратимого удаления доступны в 4.8.0 и более новых версиях модуля AZ. Storage. 

## <a name="getting-started-with-powershell"></a>Начало работы с PowerShell

Чтобы включить обратимое удаление, нужно обновить свойства службы клиента файлов. В следующем примере включено обратимое удаление для всех общих папок в учетной записи хранения:

```azurepowershell-interactive
$rgName = "yourResourceGroupName"
$accountName = "yourStorageAccountName"

Update-AzStorageFileServiceProperty -ResourceGroupName $rgName -StorageAccountName $accountName -EnableShareDeleteRetentionPolicy $true -ShareRetentionDays 7
```

Проверить, включено ли обратимое удаление, и просмотреть его политику хранения можно с помощью следующей команды:

```azurepowershell-interactive
Get-AzStorageFileServiceProperty -ResourceGroupName $rgName -StorageAccountName $accountName
```
---

## <a name="restore-soft-deleted-file-share"></a>Восстановление обратимо удаленных общих папок

# <a name="portal"></a>[Портал](#tab/azure-portal)

Для восстановление обратимо удаленной общей папки:

1. Перейдите к своей учетной записи хранения и выберите **Общие папки**.
1. В колонке общих папок включите **Показать удаленные общие папки**, чтобы отобразить все общие папки, которые были обратимо удалены.

    В результате будут отображены все общие папки, находящиеся в **удаленном** состоянии.

    :::image type="content" source="media/storage-how-to-recover-deleted-account/undelete-file-share.png" alt-text="Если в столбце состояния рядом со столбцом имени установлено значение Удалено, то общая папка находится в обратимо удаленном состоянии. Такие папки будут окончательно удалены по истечении указанного срока хранения.":::

1. Выберите папку и щелкните **отменить удаление**. Это приведет к восстановлению общей папки.

    Вы можете подтвердить, что общая папка восстановлена, так как ее состояние меняется на **Активна**.

    :::image type="content" source="media/storage-how-to-recover-deleted-account/restored-file-share.png" alt-text="Если в столбце Состояние рядом со столбцом имени установлено значение Активна, то общая папка была восстановлена.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Командлеты обратимого удаления доступны в 2.1.3 версии Azure CLI. Чтобы восстановить обратимо удаленную общую папку, необходимо сначала получить `--deleted-version` значение общего ресурса. Чтобы получить это значение, используйте следующую команду, чтобы вывести список всех удаленных общих ресурсов для вашей учетной записи хранения.

```azurecli
az storage share-rm list --storage-account yourStorageaccount --include-deleted
```

Определив общую папку, которую вы хотите восстановить, вы можете использовать ее с помощью следующей команды для ее восстановления:

```azurecli
az storage share-rm restore -n deletedshare --deleted-version 01D64EB9886F00C4 -g yourResourceGroup --storage-account yourStorageaccount
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Командлеты обратимого удаления доступны в 4.8.0 и более новых версиях модуля AZ. Storage. Чтобы восстановить обратимо удаленную общую папку, необходимо сначала получить `-DeletedShareVersion` значение общего ресурса. Чтобы получить это значение, используйте следующую команду, чтобы вывести список всех удаленных общих ресурсов для вашей учетной записи хранения.

```azurepowershell-interactive
Get-AzRmStorageShare -ResourceGroupName $rgname -StorageAccountName $accountName -IncludeDeleted
```

Определив общую папку, которую вы хотите восстановить, вы можете использовать ее с помощью следующей команды для ее восстановления:

```azurepowershell-interactive
Restore-AzRmStorageShare -ResourceGroupName $rgname -StorageAccountName $accountName -DeletedShareVersion 01D5E2783BDCDA97
```
---

## <a name="disable-soft-delete"></a>Отключение обратимого удаления

Если вы хотите отменить использование обратимого удаления или окончательно удалить общую папку, выполните следующие действия.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите к своей учетной записи хранения и выберите **Обратимое удаление** в разделе **Параметры**.
1. В разделе **Общие папки** выберите **Отключено** для параметра **Обратимое удаление файловых ресурсов**.
1. Выберите **Сохранить**, чтобы подтвердить параметры хранения данных.

    :::image type="content" source="media/storage-how-to-recover-deleted-account/disable-soft-delete-files.png" alt-text="Отключение обратимого удаления позволит немедленно и окончательно удалить все общие папки в вашей учетной записи хранения, когда вам заблагорассудится.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Командлеты обратимого удаления доступны в 2.1.3 версии Azure CLI. Чтобы отключить обратимое удаление в учетной записи хранения, можно использовать следующую команду:

```azurecli
az storage account file-service-properties update --enable-delete-retention false -n yourStorageaccount -g yourResourceGroup
```
# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Командлеты обратимого удаления доступны в 4.8.0 и более новых версиях модуля AZ. Storage. Чтобы отключить обратимое удаление в учетной записи хранения, можно использовать следующую команду:

```azurepowershell-interactive
Update-AzStorageFileServiceProperty -ResourceGroupName $rgName -StorageAccountName $accountName -EnableShareDeleteRetentionPolicy $false
```
---

## <a name="next-steps"></a>Дальнейшие действия

См. сведения о другом способе защиты и восстановления данных в статье [Общие сведения о моментальных снимках общих ресурсов для Файлов Azure](storage-snapshots-files.md).
---
title: Настройка и администрирование политик неизменяемости для хранилища BLOB-объектов — служба хранилища Azure
description: Узнайте, как использовать поддержку WORM для хранения больших двоичных объектов, что позволяет хранить данные в нестираемом и неизменяемом состоянии на протяжении определенного интервала времени.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/26/2019
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: a09dbd7d778a4f7ea2a9aac9ca07b9e6d06bc1ae
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84463643"
---
# <a name="set-and-manage-immutability-policies-for-blob-storage"></a>Настройка и администрирование политик неизменяемости для хранилища BLOB-объектов

Функция хранения данных в неизменяемом виде хранилища BLOB-объектов Azure дает пользователям возможность хранить критически важные для бизнеса объекты данных в состоянии WORM. Это состояние делает их нестираемыми и неизменяемыми в течение определенного пользователем интервала времени. В течение периода удержания большие двоичные объекты можно создать и прочитать, но не изменять или удалять. Хранение данных в неизменяемом виде доступно только в учетных записях хранилища BLOB-объектов и общего назначения версии 2 во всех регионах.

В этой статье показано, как настроить политики неизменяемости и удержания по юридическим причинам для данных в хранилище BLOB-объектов и администрировать их с помощью портала Azure, PowerShell или Azure CLI. Дополнительные сведения о неизменяемом хранилище см. в статье [Хранение критически важных для бизнеса данных большого двоичного объекта с помощью неизменяемого хранилища](storage-blob-immutable-storage.md).

## <a name="set-retention-policies-and-legal-holds"></a>Настройка политик хранения и удержаний по юридическим причинам

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. Создайте новый контейнер или выберите существующий контейнер для хранения BLOB-объектов, которые должны храниться в неизменяемом состоянии. Контейнер должен находиться в учетной записи хранения общего назначения версии 2 или учетной записи хранилища BLOB-объектов.

2. В параметрах контейнера выберите **Политика доступа**. Затем в разделе **Хранилище неизменяемых BLOB-объектов** нажмите **Добавить политику**.

    ![Параметры контейнера на портале](media/storage-blob-immutability-policies-manage/portal-image-1.png)

3. Чтобы включить временный период удержания, в раскрывающемся меню выберите **Хранение с учетом времени**.

    ![Параметр "Хранение с учетом времени", выбранный в разделе типа политики](media/storage-blob-immutability-policies-manage/portal-image-2.png)

4. Введите период удержания в днях (допустимые значения: от 1 до 146 000 дней).

    ![Поле "Изменить срок хранения на"](media/storage-blob-immutability-policies-manage/portal-image-5-retention-interval.png)

    Начальное состояние политики разблокировано, что позволяет протестировать функцию и внести изменения в политику перед ее блокировкой. Блокировка политики необходима для обеспечения соответствия требованиям, например SEC 17a-4.

5. Заблокируйте политику. Щелкните правой кнопкой мыши кнопку с многоточием ( **...** ), и появится следующее меню с дополнительными действиями.

    ![Параметр "Политика блокировки" в меню](media/storage-blob-immutability-policies-manage/portal-image-4-lock-policy.png)

6. Выберите **Блокировать политику** и подтвердите блокировку. После блокировки политику невозможно будет удалить и разрешено будет только продлевать период удержания. Удаление и переопределение BLOB-объектов запрещено. 

    ![Параметр "Блокировать политику" в меню](media/storage-blob-immutability-policies-manage/portal-image-5-lock-policy.png)

7. Чтобы включить удержания по юридическим причинам, нажмите кнопку **Добавить политику**. В раскрывающемся меню выберите **Удержание по юридическим причинам**.

    ![Параметр в меню "Удержание по юридическим причинам" под полем "Тип политики"](media/storage-blob-immutability-policies-manage/portal-image-legal-hold-selection-7.png)

8. Создайте юридическое удержание с одним или несколькими тегами.

    ![Поле "Имя тега" под полем "Тип политики"](media/storage-blob-immutability-policies-manage/portal-image-set-legal-hold-tags.png)

9. Чтобы очистить удержание по юридическим причинам, удалите примененный тег идентификатора удержания по юридическим причинам.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Эта возможность доступна в следующих группах команд: `az storage container immutability-policy` и `az storage container legal-hold`. Запустите в них `-h`, чтобы просмотреть команды.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Модуль Az.Storage поддерживает неизменяемое хранилище.  Чтобы включить эту функцию, сделайте следующее:

1. Установите последнюю версию PowerShellGet: `Install-Module PowerShellGet –Repository PSGallery –Force`.
2. Удалите все предыдущие версии Azure PowerShell.
3. Установка Azure PowerShell: `Install-Module Az –Repository PSGallery –AllowClobber`.

Пример сценария PowerShell ниже приводится для справки. Он создает учетную запись хранения и контейнер. Далее в нем показано, как задавать и снимать удержания по юридическим причинам, создавать и блокировать политику хранения на основе времени (известную как неизменяемая политика), а также продлевать период удержания.

Прежде всего создайте учетную запись хранения Azure.

```powershell
$resourceGroup = "<Enter your resource group>"
$storageAccount = "<Enter your storage account name>"
$container = "<Enter your container name>"
$location = "<Enter the storage account location>"

# Log in to Azure
Connect-AzAccount
Register-AzResourceProvider -ProviderNamespace "Microsoft.Storage"

# Create your Azure resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create your Azure storage account
$account = New-AzStorageAccount -ResourceGroupName $resourceGroup -StorageAccountName `
    $storageAccount -SkuName Standard_ZRS -Location $location -Kind StorageV2

# Create a new container using the context
$container = New-AzStorageContainer -Name $container -Context $account.Context

# List the containers in the account
Get-AzStorageContainer -Context $account.Context

# Remove a container
Remove-AzStorageContainer -Name $container -Context $account.Context
```

Установка и удаление юридических удержаний:

```powershell
# Set a legal hold
Add-AzRmStorageContainerLegalHold -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -Name $container -Tag <tag1>,<tag2>,...

# Clear a legal hold
Remove-AzRmStorageContainerLegalHold -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -Name $container -Tag <tag3>
```

Создание или обновление политик неизменяемости на основе времени:

```powershell
# Create a time-based immutability policy
Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container -ImmutabilityPeriod 10
```

Извлечение политик неизменяемости:

```powershell
# Get an immutability policy
Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container
```

Блокировка политик неизменяемости (добавление `-Force` для отклонения запроса):

```powershell
# Lock immutability policies
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container
Lock-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container `
    -Etag $policy.Etag
```

Расширение политик неизменяемости:

```powershell
# Extend immutability policies
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container

Set-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy `
    $policy -ImmutabilityPeriod 11 -ExtendPolicy
```

Удаление политики неизменяемости (добавление `-Force` для отклонения запроса):

```powershell
# Remove an unlocked immutability policy
$policy = Get-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName `
    $resourceGroup -StorageAccountName $storageAccount -ContainerName $container

Remove-AzRmStorageContainerImmutabilityPolicy -ImmutabilityPolicy $policy
```

---

## <a name="enabling-allow-protected-append-blobs-writes"></a>Включение разрешения на запись добавочных больших двоичных объектов

### <a name="portal"></a>[Портал](#tab/azure-portal)

![Параметр "Разрешить дополнительные добавления"](media/storage-blob-immutability-policies-manage/immutable-allow-additional-append-writes.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Эта возможность доступна в следующих группах команд: `az storage container immutability-policy` и `az storage container legal-hold`. Запустите в них `-h`, чтобы просмотреть команды.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# Create an immutability policy with appends allowed
Set-AzRmStorageContainerImmutabilityPolicy -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount -ContainerName $container -ImmutabilityPeriod 10 -AllowProtectedAppendWrite $true
```

---

## <a name="next-steps"></a>Дальнейшие действия

[Хранение критически важных для бизнеса данных большого двоичного объекта с помощью неизменяемого хранилища](storage-blob-immutable-storage.md)

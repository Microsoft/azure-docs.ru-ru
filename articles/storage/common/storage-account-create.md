---
title: Создание учетной записи хранения
titleSuffix: Azure Storage
description: Узнайте, как создать учетную запись хранения для хранения больших двоичных объектов, файлов, очередей и таблиц. Учетная запись хранения Azure предоставляет уникальное пространство имен в Microsoft Azure для чтения и записи данных.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 01/11/2021
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 73bef1648b08f5c1e0664ef1da15375bc44149cc
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105045354"
---
# <a name="create-a-storage-account"></a>Создание учетной записи хранения

Учетная запись хранения Azure содержит все объекты данных службы хранилища Azure: большие двоичные объекты, файлы, очереди, таблицы и диски. Учетная запись хранения предоставляет уникальное пространство имен для данных службы хранилища Azure, доступное из любой точки мира по протоколу HTTP или HTTPS. Данные в учетной записи хранения Azure являются устойчивыми и высокодоступными, безопасными и масштабируемыми.

В этом пошаговом руководстве вы узнаете, как создать учетную запись хранения с помощью [портал Azure](https://portal.azure.com/), [Azure PowerShell](/powershell/azure/), [Azure CLI](/cli/azure)или [шаблона Azure Resource Manager](../../azure-resource-manager/management/overview.md).  

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Отсутствует.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы создать учетную запись хранения Azure с помощью PowerShell, убедитесь, что установлен модуль Azure PowerShell AZ версии 0,7 или более поздней. Дополнительные сведения см. [в разделе Введение в модуль Azure PowerShell AZ](/powershell/azure/new-azureps-module-az).

Чтобы найти текущую версию, выполните следующую команду:

```powershell
Get-InstalledModule -Name "Az"
```

Сведения об установке или обновлении Azure PowerShell см. в разделе [install Azure PowerShell Module](/powershell/azure/install-Az-ps).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Вы можете войти в Azure и выполнить команды Azure CLI одним из двух способов:

- Выполнить команды CLI на портале Azure в Azure Cloud Shell.
- Установить CLI и выполнить команды CLI локально.

### <a name="use-azure-cloud-shell"></a>Использование Azure Cloud Shell

Azure Cloud Shell — это бесплатная оболочка Bash, которую можно запускать непосредственно на портале Azure. Предварительно установленный интерфейс Azure CLI настроен для использования с вашей учетной записью. Нажмите кнопку **Cloud Shell** в меню в правой верхней части портал Azure:

[![Cloud Shell](./media/storage-quickstart-create-account/cloud-shell-menu.png)](https://portal.azure.com)

Кнопка запускает интерактивную оболочку, которую можно использовать для выполнения действий, описанных в этой статье.

[![Снимок экрана с окном Azure Cloud Shell на портале](./media/storage-quickstart-create-account/cloud-shell.png)](https://portal.azure.com)

### <a name="install-the-cli-locally"></a>Установка CLI локально

Azure CLI также можно установить и применять локально. Для использования примеров в этой статье требуется Azure CLI версии 2.0.4 или более поздней. Выполните `az --version`, чтобы узнать установленную версию. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

# <a name="template"></a>[Шаблон](#tab/template)

Отсутствует.

---

## <a name="sign-in-to-azure"></a>Вход в Azure

# <a name="portal"></a>[Портал](#tab/azure-portal)

Войдите на [портал Azure](https://portal.azure.com).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнить проверку подлинности, войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

```powershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы запустить Azure Cloud Shell, войдите в [портал Azure](https://portal.azure.com).

Чтобы войти в локальную установку интерфейса командной строки, выполните команду [AZ login](/cli/azure/reference-index#az-login) :

```azurecli-interactive
az login
```

# <a name="template"></a>[Шаблон](#tab/template)

Н/Д

---

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

Каждая учетная запись хранения должна принадлежать группе ресурсов Azure. Группа ресурсов — это логический контейнер для группирования служб Azure. При создании учетной записи хранения у вас есть возможность создать новую или использовать существующую группу ресурсов. В этой статье показано, как создать группу ресурсов.

Учетная запись хранения **общего назначения версии 2** предоставляет доступ ко всем службам хранилища Azure: BLOB-объектам, файлам, очередям, таблицам и дискам. С помощью описанной здесь процедуры создается учетная запись хранения общего назначения версии 2, но создание любого типа учетной записи хранения происходит таким же образом. Дополнительные сведения о типах учетных записей хранения см. в статье [Общие сведения об учетной записи хранения](storage-account-overview.md).

# <a name="portal"></a>[Портал](#tab/azure-portal)

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Сначала создайте группу ресурсов с помощью PowerShell, используя команду [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

```powershell
# put resource group in a variable so you can use the same group name going forward,
# without hard-coding it repeatedly
$resourceGroup = "storage-resource-group"
$location = "westus"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

Если вы не уверены, какой регион необходимо задать для параметра `-Location`, вы можете получить список поддерживаемых регионов для своей подписки, выполнив команду [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

```powershell
Get-AzLocation | select Location
```

Затем создайте учетную запись хранения общего назначения версии 2 с геоизбыточным хранилищем с доступом на чтение (RA-GRS) с помощью команды [New-азсторажеаккаунт](/powershell/module/az.storage/New-azStorageAccount) . Помните, что имя вашей учетной записи хранения должно быть уникальным в пределах Azure, поэтому замените значение заполнителя в квадратных скобках своим уникальным значением:

```powershell
New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name <account-name> `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2
```

> [!IMPORTANT]
> Если вы планируете использовать [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), включите `-EnableHierarchicalNamespace $True` в этот список параметров.

Чтобы создать учетную запись хранения общего назначения версии 2 с другим вариантом репликации, замените требуемое значение в таблице ниже для параметра **SkuName** .

|Параметр репликации  |Параметр SkuName  |
|---------|---------|
|Локально избыточное хранилище (LRS)     |Standard_LRS         |
|хранилище, избыточное между зонами (ZRS);     |Standard_ZRS         |
|Геоизбыточное хранилище (GRS)     |Standard_GRS         |
|Геоизбыточное хранилище с доступом для чтения (RA-GRS)     |Standard_RAGRS         |
|Хранилище, геоизбыточное между зонами (GZRS)    |Standard_GZRS         |
|Хранилище, геоизбыточное между зонами, с доступом на чтение (RA-GZRS)    |Standard_RAGZRS         |

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Сначала создайте новую группу ресурсов с помощью Azure CLI, используя команду [az group create](/cli/azure/group#az_group_create).

```azurecli-interactive
az group create \
    --name storage-resource-group \
    --location westus
```

Если вы не уверены, какой регион необходимо задать для параметра `--location`, вы можете получить список поддерживаемых регионов для своей подписки, выполнив команду [az account list-locations](/cli/azure/account#az_account_list).

```azurecli-interactive
az account list-locations \
    --query "[].{Region:name}" \
    --out table
```

Затем создайте учетную запись хранения общего назначения версии 2 с геоизбыточным хранилищем с доступом на чтение с помощью команды [AZ Storage Account Create](/cli/azure/storage/account#az_storage_account_create) . Помните, что имя вашей учетной записи хранения должно быть уникальным в пределах Azure, поэтому замените значение заполнителя в квадратных скобках своим уникальным значением:

```azurecli-interactive
az storage account create \
    --name <account-name> \
    --resource-group storage-resource-group \
    --location westus \
    --sku Standard_RAGRS \
    --kind StorageV2
```

> [!IMPORTANT]
> Если вы планируете использовать [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), включите `--enable-hierarchical-namespace true` в этот список параметров.

Чтобы создать учетную запись хранения общего назначения версии 2 с другим вариантом репликации, замените требуемое значение в таблице ниже для параметра **SKU** .

|Параметр репликации  |параметр sku  |
|---------|---------|
|Локально избыточное хранилище (LRS)     |Standard_LRS         |
|хранилище, избыточное между зонами (ZRS);     |Standard_ZRS         |
|Геоизбыточное хранилище (GRS)     |Standard_GRS         |
|Геоизбыточное хранилище с доступом для чтения (RA-GRS)     |Standard_RAGRS         |
|Хранилище, геоизбыточное между зонами (GZRS)    |Standard_GZRS         |
|Хранилище, геоизбыточное между зонами, с доступом на чтение (RA-GZRS)    |Standard_RAGZRS         |

# <a name="template"></a>[Шаблон](#tab/template)

Чтобы развернуть шаблон диспетчер ресурсов для создания учетной записи хранения, можно использовать либо Azure PowerShell, либо Azure CLI. Шаблон, используемый в этой статье, относится к [Azure Resource Manager шаблонам](https://azure.microsoft.com/resources/templates/101-storage-account-create/)быстрого запуска. Чтобы запустить скрипты, выберите команду **попробовать** , чтобы открыть Azure Cloud Shell. Чтобы вставить сценарий, щелкните правой кнопкой мыши оболочку и выберите **Вставить**.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

> [!NOTE]
> Этот шаблон служит только в качестве примера. Существует множество параметров учетной записи хранения, которые не настроены как часть этого шаблона. Например, если вы хотите использовать [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), измените этот шаблон, задав `isHnsEnabledad` свойству `StorageAccountPropertiesCreateParameters` объекта значение `true` . 

Сведения о том, как изменить этот шаблон или создать новые, см. в следующих статьях:

- [Azure Resource Manager документация](../../azure-resource-manager/index.yml).
- [Microsoft.Storage resource types](/azure/templates/microsoft.storage/allversions) (Типы ресурсов Microsoft.Storage).
- [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Storage).

---

## <a name="delete-a-storage-account"></a>Удаление учетной записи хранения

Удаление учетной записи хранения приведет к удалению всей учетной записи, включая все данные в учетной записи, и не может быть отменено.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Перейдите к учетной записи хранения в [портал Azure](https://portal.azure.com).
1. Щелкните **Удалить**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы удалить учетную запись хранения, используйте команду [Remove-азсторажеаккаунт](/powershell/module/az.storage/remove-azstorageaccount) :

```powershell
Remove-AzStorageAccount -Name <storage-account> -ResourceGroupName <resource-group>
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы удалить учетную запись хранения, используйте команду [AZ Storage Account Delete](/cli/azure/storage/account#az-storage-account-delete) :

```azurecli-interactive
az storage account delete --name <storage-account> --resource-group <resource-group>
```

# <a name="template"></a>[Шаблон](#tab/template)

Чтобы удалить учетную запись хранения, используйте либо Azure PowerShell, либо Azure CLI.

```azurepowershell-interactive
$storageResourceGroupName = Read-Host -Prompt "Enter the resource group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"
Remove-AzStorageAccount -Name $storageAccountName -ResourceGroupName $storageResourceGroupName
```

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account delete --name storageAccountName --resource-group resourceGroupName
```

---

Кроме того, можно удалить группу ресурсов, которая удаляет учетную запись хранения и другие ресурсы в этой группе ресурсов. Дополнительные сведения об удалении группы ресурсов см. в разделе [Удаление группы ресурсов и ресурсов](../../azure-resource-manager/management/delete-resource-group.md).

> [!WARNING]
> Восстановить удаленную учетную запись хранения или ее содержимое невозможно. Создайте резервные копии нужных данных, прежде чем удалять учетную запись. Это касается также любых ресурсов в учетной записи. Восстановить удаленный BLOB-объект, таблицу, очередь или файл невозможно.
>
> Если вы попытаетесь удалить учетную запись хранения, связанную с виртуальной машиной Azure, может появиться сообщение об ошибке, уведомляющее, что учетная запись хранения используется. Сведения об устранении этой ошибки см. в разделе [Устранение ошибок при удалении учетных записей хранения](/troubleshoot/azure/virtual-machines/welcome-virtual-machines).

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об учетной записи хранения](storage-account-overview.md)
- [Обновление до учетной записи хранения общего назначения версии 2](storage-account-upgrade.md)
- [Перемещение учетной записи хранения Azure в другой регион](storage-account-move.md)
- [Восстановление удаленной учетной записи хранения.](storage-account-recover.md)
---
title: Использование PowerShell для управления данными в независимых облаках Azure
titleSuffix: Azure Storage
description: Управление хранилищем в облаке, правительственном облаке и немецком облаке с помощью Azure PowerShell.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2019
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e924a5f6c765b5b964fe3b1492393b063d9d23b4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92783578"
---
# <a name="managing-storage-in-the-azure-independent-clouds-using-powershell"></a>Управление службой хранилища в независимых от Azure облаках с помощью Azure PowerShell

Большинство людей используют общедоступное облако Azure для глобального развертывания Azure. Но есть и независимые развертывания Microsoft Azure для обеспечения автономности и других целей. Такие независимые развертывания называются средами. Ниже приведен список независимых облаков, доступных в настоящее время:

* [Azure Government Cloud (Облако Azure для государственных организаций)](https://azure.microsoft.com/features/gov/)
* [Облако Azure Китая 21Vianet, управляемое 21Vianet, в Китае](http://www.windowsazure.cn/)
* [Облако Azure для Германии.](../../germany/germany-welcome.md)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="using-an-independent-cloud"></a>Использование независимого облака

Чтобы использовать службу хранилища Azure в одном из независимых облаков, нужно подключиться к нему вместо общедоступного облака Azure. Чтобы использовать одно из независимых облаков Azure вместо общедоступного, выполните следующие инструкции:

* Укажите *среду*, к которой выполняется подключение.
* Определите и используйте доступные регионы.
* Используйте правильный суффикс конечной точки, который отличается от суффикса общедоступного облака Azure.

Для работы с этим руководством требуется модуль Az в Azure PowerShell 0.7 или более поздней версии. В окне PowerShell выполните `Get-Module -ListAvailable Az`, чтобы найти версию. Если версия не отображается или необходимо обновить модуль, см. статью об [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="log-in-to-azure"></a>Вход в Azure

Запустите командлет [Get-AzEnvironment](/powershell/module/az.accounts/get-azenvironment), чтобы просмотреть доступные среды Azure.

```powershell
Get-AzEnvironment
```

Войдите в учетную запись с доступом к облаку, к которому требуется подключиться, и укажите среду. В этом примере показано, как войти в учетную запись облака Azure для государственных организаций.   

```powershell
Connect-AzAccount –Environment AzureUSGovernment
```

Для доступа к облаку для Китая используйте среду **AzureChinaCloud**. Для доступа к облаку для Германии используйте среду **AzureGermanCloud**.

На этом этапе для создания учетной записи хранения иди другого ресурса вам может потребоваться список расположений в выбранном облаке. Запросите этот список с помощью командлета [Get AzLocation](/powershell/module/az.resources/get-azlocation).

```powershell
Get-AzLocation | select Location, DisplayName
```

В следующей таблице показаны расположения, доступные в облаке для Германии.

|Расположение | Отображаемое имя |
|----|----|
| `germanycentral` | Центральная Германия|
| `germanynortheast` | Северо-восточная Германия |


## <a name="endpoint-suffix"></a>Суффикс конечной точки

Суффикс конечной точки для каждой из этих сред отличается от суффикса конечной точки общедоступного облака Azure. Например, суффикс конечной точки большого двоичного объекта в общедоступном облаке Azure — **blob.core.windows.net**. В облаке Azure для государственных организаций суффикс конечной точки большого двоичного объекта — **blob.core.usgovcloudapi.net**.

### <a name="get-endpoint-using-get-azenvironment"></a>Получение конечной точки с помощью командлета Get-AzEnvironment

Получите суффикс конечной точки с помощью командлета [Get AzEnvironment](/powershell/module/az.accounts/get-azenvironment). Конечная точка — это свойство среды *StorageEndpointSuffix*.

В следующих фрагментах кода показано, как получить суффикс конечной точки. Все эти команды возвращают нечто вроде "core.cloudapp.net" или "core.cloudapi.de" и т. д. Добавьте суффикс в службу хранилища для доступа к этой службе. Например, конечная точка queue.core.cloudapi.de получит доступ к службе очередей в облаке для Германии.

Этот фрагмент кода позволяет получить все среды и суффикс конечной точки для каждой из них.

```powershell
Get-AzEnvironment | select Name, StorageEndpointSuffix 
```

Эта команда возвращает следующие результаты:

| Имя| StorageEndpointSuffix|
|----|----|
| AzureChinaCloud; | core.chinacloudapi.cn|
| AzureCloud; | core.windows.net |
| AzureGermanCloud | core.cloudapi.de|
| AzureUSGovernment; | core.usgovcloudapi.net |

Чтобы получить все свойства для указанной среды, вызовите **Get AzEnvironment** и укажите имя облака. Этот фрагмент кода позволяет получить список свойств. Найдите **StorageEndpointSuffix** в списке. Следующий пример касается облака для Германии.

```powershell
Get-AzEnvironment -Name AzureGermanCloud
```

Результаты похожи на следующие значения:

|Имя свойства|Значение|
|----|----|
| Имя | `AzureGermanCloud` |
| EnableAdfsAuthentication | `False` |
| ActiveDirectoryServiceEndpointResourceI | `http://management.core.cloudapi.de/` |
| GalleryURL | `https://gallery.cloudapi.de/` |
| ManagementPortalUrl | `https://portal.microsoftazure.de/` |
| ServiceManagementUrl | `https://manage.core.cloudapi.de/` |
| PublishSettingsFileUrl| `https://manage.microsoftazure.de/publishsettings/index` |
| ResourceManagerUrl | `http://management.microsoftazure.de/` |
| SqlDatabaseDnsSuffix | `.database.cloudapi.de` |
| **StorageEndpointSuffix** | `core.cloudapi.de` |
| ... | ... |

Чтобы получить только свойство суффикса конечной точки хранилища, обратитесь к определенному облаку и подайте запрос только на это свойство.

```powershell
$environment = Get-AzEnvironment -Name AzureGermanCloud
Write-Host "Storage EndPoint Suffix = " $environment.StorageEndpointSuffix
```

Эта команда возвращает приведенные ниже сведения.

`Storage Endpoint Suffix = core.cloudapi.de`

### <a name="get-endpoint-from-a-storage-account"></a>Получение конечной точки из учетной записи хранения

Вы также можете проверить свойства учетной записи хранения, чтобы получить конечные точки:

```powershell
# Get a reference to the storage account.
$resourceGroup = "myexistingresourcegroup"
$storageAccountName = "myexistingstorageaccount"
$storageAccount = Get-AzStorageAccount `
  -ResourceGroupName $resourceGroup `
  -Name $storageAccountName 
  # Output the endpoints.
Write-Host "blob endpoint = " $storageAccount.PrimaryEndPoints.Blob 
Write-Host "file endpoint = " $storageAccount.PrimaryEndPoints.File
Write-Host "queue endpoint = " $storageAccount.PrimaryEndPoints.Queue
Write-Host "table endpoint = " $storageAccount.PrimaryEndPoints.Table
```

Для учетной записи хранения в облаке для государственных организаций эта команда возвращает следующие выходные данные:

```
blob endpoint = http://myexistingstorageaccount.blob.core.usgovcloudapi.net/
file endpoint = http://myexistingstorageaccount.file.core.usgovcloudapi.net/
queue endpoint = http://myexistingstorageaccount.queue.core.usgovcloudapi.net/
table endpoint = http://myexistingstorageaccount.table.core.usgovcloudapi.net/
```

## <a name="after-setting-the-environment"></a>После настройки среды

Теперь вы можете использовать PowerShell для управления учетными записями хранения и доступа к данным больших двоичных объектов, очередей, файлов и таблиц. Дополнительные сведения см. в разделе [AZ. Storage](/powershell/module/az.storage).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы создали новую группу ресурсов и учетную запись хранения для этого упражнения, вы можете удалить оба ресурса, удалив группу ресурсов. Удаление группы ресурсов приведет к удалению всех ресурсов, которые она содержит.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

* [Использование учетных данных для входа в разных сеансах PowerShell](/powershell/azure/context-persistence).
* [Хранилище Azure для государственных организаций](../../azure-government/compare-azure-government-global-azure.md)
* [Руководство для разработчиков Microsoft Azure Government](../../azure-government/documentation-government-developer-guide.md)
* [Примечания для разработчиков для приложений Azure для Китая в Китае](https://msdn.microsoft.com/library/azure/dn578439.aspx)
* [Документация по Azure для Германии](../../germany/germany-welcome.md).
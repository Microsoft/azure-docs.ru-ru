---
title: Краткое руководство по управлению общими файловыми ресурсами Azure с помощью Azure PowerShell
description: Используйте это краткое руководство, чтобы узнать, как управлять общими файловыми ресурсами Azure с помощью Azure PowerShell.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/26/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 8b4bd9ece5f010f1294356ad4673543834e5076a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94626917"
---
# <a name="quickstart-create-and-manage-an-azure-file-share-with-azure-powershell"></a>Краткое руководство: создание общих файловых ресурсов Azure и управление ими с помощью Azure PowerShell 
В этом руководстве рассматриваются основы работы с [общими файловыми ресурсами Azure](storage-files-introduction.md) с помощью PowerShell. Общие файловые ресурсы Azure отличаются от других ресурсов тем, что хранятся в облаке и поддерживаются платформой Azure. Общие папки Azure поддерживают стандартный отраслевой протокол SMB и протокол NFS (предварительная версия), а также позволяют совместно использовать файлы на нескольких компьютерах, в нескольких приложениях и экземплярах. 

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Если вы хотите установить и использовать PowerShell локально для работы с этим руководством, вам понадобится Az модуля Azure PowerShell 0.7 или более поздней версии. Чтобы узнать, какую версию модуля Azure PowerShell вы используете, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить `Login-AzAccount`, чтобы войти в учетную запись Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Если у вас еще нет группы ресурсов Azure, вы можете создать ее с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). 

В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе "Западная часть США 2".

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$region = "westus2"

New-AzResourceGroup `
    -Name $resourceGroupName `
    -Location $region | Out-Null
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения
Учетная запись хранения представляет собой общий пул носителей, который можно использовать для развертывания общих папок Azure. Учетная запись хранения может содержать любое количество общих ресурсов, а общий ресурс может содержать любое количество файлов, насколько позволяет емкость учетной записи хранения. В этом примере создается версия 2 общего назначения (учетная запись хранения GPv2), которая может хранить общие папки Azure ценовой категории "Стандартный" или другие ресурсы хранения, такие как большие двоичные объекты или очереди, на вращающемся носителе жесткого диска (HDD). Служба "Файлы Azure" также поддерживает твердотельные накопители (SSD) ценовой категории "Премиум". Общие папки Azure этой категории можно создать в учетных записях хранения Файлов.

В этом примере создается учетная запись хранения с помощью командлета [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Учетная запись хранения называется *mystorageaccount\<random number>*, и к переменной **$storageAcct** добавляется ссылка на эту учетную запись. Имена учетных записей хранения должны быть уникальными. Чтобы сделать имя уникальным, используйте команду `Get-Random` для добавления к нему номера. 

```azurepowershell-interactive 
$storageAccountName = "mystorageacct$(Get-Random)"

$storageAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $region `
    -Kind StorageV2 `
    -SkuName Standard_ZRS `
    -EnableLargeFileShare
```

> [!Note]  
> Общие папки более 5 ТиБ (максимум до 100 ТиБ для папки) доступны только для локально избыточных (LRS) и избыточных в пределах зоны (ZRS) учетных записей хранения. Чтобы создать геоизбыточную (GRS) или избыточную в пределах зоны (GZRS) учетную запись хранения, удалите параметр `-EnableLargeFileShare`.

## <a name="create-an-azure-file-share"></a>создать файловый ресурс Azure;
Теперь можно создать первый файловый ресурс Azure. Вы можете создать файловый ресурс с помощью командлета [New-AzRmStorageShare](/powershell/module/az.storage/New-AzRmStorageShare). В этом примере создается общий ресурс с именем `myshare`.

```azurepowershell-interactive
$shareName = "myshare"

New-AzRmStorageShare `
    -StorageAccount $storageAcct `
    -Name $shareName `
    -EnabledProtocol SMB `
    -QuotaGiB 1024 | Out-Null
```

Имена общих ресурсов должны содержать только строчные буквы, цифры и отдельные дефисы, и они не могут начинаться с дефиса. Дополнительные сведения о присвоении имен общим папкам и файлам см. в статье [Именование общих ресурсов, каталогов, файлов и метаданных и ссылка на них](/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata).

## <a name="use-your-azure-file-share"></a>Использование общего файлового ресурса Azure
Служба файлов Azure обеспечивает два способа работы с файлами и папками в общем файловом ресурсе Azure: стандартный отраслевой [протокол Server Message Block (SMB)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) и [протокол File REST](/rest/api/storageservices/file-service-rest-api). 

Чтобы установить общий файловый ресурс с помощью SMB, ознакомьтесь со следующим документом (с учетом вашей ОС):
- [Windows](storage-how-to-use-files-windows.md)
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)

### <a name="using-an-azure-file-share-with-the-file-rest-protocol"></a>Использование общего файлового ресурса Azure с протоколом File REST 
С протоколом REST в службе "Файлы" вы можете работать напрямую (т. е. вручную обрабатывать вызовы HTTP REST), но самым распространенным способом использования этого протокола является применение модуля Azure PowerShell, [Azure CLI](storage-how-to-use-files-cli.md) или пакета SDK для службы хранилища Azure. Все это обеспечивает хорошую оболочку вокруг протокола на языке сценариев или программирования по вашему выбору.  

В большинстве случаев используется файловый ресурс Azure по протоколу SMB, так как это позволяет использовать имеющиеся приложения и инструменты, на использование которых у вас должна быть возможность, но есть несколько причин, по которым выгодно использовать File REST API, а не SMB, например:

- Вы подключаетесь к общему файловому ресурсу из PowerShell Cloud Shell (которая не может подключать общие файловые ресурсы по SMB).
- Вы используете бессерверные ресурсы, такие как [Функции Azure](../../azure-functions/functions-overview.md). 
- Вы создаете службу с добавленной стоимостью, которая будет взаимодействовать со многими общими папками Azure, например выполнять резервное копирование или антивирусное сканирование.

В следующих примерах показано, как использовать модуль Azure PowerShell для управления файловым ресурсом Azure с помощью протокола File REST. Параметр `-Context` используется, чтобы извлечь ключ учетной записи хранения для выполнения указанных действий с общей папкой. Чтобы получить ключ учетной записи хранения, вам должна быть назначена роль Azure `Owner` в учетной записи хранения.

#### <a name="create-directory"></a>Создание каталога
Чтобы создать каталог с именем *myDirectory* в корне файлового ресурса Azure, используйте командлет [New-AzStorageDirectory](/powershell/module/az.storage/New-AzStorageDirectory).

```azurepowershell-interactive
New-AzStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName $shareName `
   -Path "myDirectory"
```

#### <a name="upload-a-file"></a>Отправка файла
Чтобы продемонстрировать, как отправить файл с помощью командлета [Set-AzStorageFileContent](/powershell/module/az.storage/Set-AzStorageFileContent), сначала необходимо создать нужный файл на временном диске PowerShell Cloud Shell. 

Этот пример указывает текущую дату и время в новом файле на временном диске, а затем загружает этот файл в файловый ресурс.

```azurepowershell-interactive
# this expression will put the current date and time into a new file on your scratch drive
cd "~/CloudDrive/"
Get-Date | Out-File -FilePath "SampleUpload.txt" -Force

# this expression will upload that newly created file to your Azure file share
Set-AzStorageFileContent `
   -Context $storageAcct.Context `
   -ShareName $shareName `
   -Source "SampleUpload.txt" `
   -Path "myDirectory\SampleUpload.txt"
```   

При работе в PowerShell локально `~/CloudDrive/` следует заменить путем, который имеется на вашем компьютере.

После отправки файла вы можете использовать командлет [Get-AzStorageFile](/powershell/module/Az.Storage/Get-AzStorageFile), чтобы убедиться, что файл отправлен в файловый ресурс Azure. 

```azurepowershell-interactive
Get-AzStorageFile `
    -Context $storageAcct.Context `
    -ShareName $shareName `
    -Path "myDirectory\" 
```

#### <a name="download-a-file"></a>скачать файл;
С помощью командлета [Get-AzStorageFileContent](/powershell/module/az.storage/Get-AzStorageFilecontent) вы можете скачать копию только что отправленного файла на временный диск Cloud Shell.

```azurepowershell-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists because you've run this example before.
Remove-Item `
    -Path "SampleDownload.txt" `
    -Force `
    -ErrorAction SilentlyContinue

Get-AzStorageFileContent `
    -Context $storageAcct.Context `
    -ShareName $shareName `
    -Path "myDirectory\SampleUpload.txt" `
    -Destination "SampleDownload.txt"
```

После загрузки файла вы можете использовать команду `Get-ChildItem`, чтобы увидеть, что файл был загружен на временный диск PowerShell Cloud Shell.

```azurepowershell-interactive
Get-ChildItem | Where-Object { $_.Name -eq "SampleDownload.txt" }
``` 

#### <a name="copy-files"></a>Копирование файлов
Одна из распространенных задач заключается в копировании файлов из одной общей папки в другую. Чтобы продемонстрировать эту функцию, вы можете создать общий ресурс и скопировать только что отправленный в него файл с помощью командлета [Start-AzStorageFileCopy](/powershell/module/az.storage/Start-AzStorageFileCopy). 

```azurepowershell-interactive
$otherShareName = "myshare2"

New-AzRmStorageShare `
    -StorageAccount $storageAcct `
    -Name $otherShareName `
    -EnabledProtocol SMB `
    -QuotaGiB 1024 | Out-Null
  
New-AzStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName $otherShareName `
   -Path "myDirectory2"

Start-AzStorageFileCopy `
    -Context $storageAcct.Context `
    -SrcShareName $shareName `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestShareName $otherShareName `
    -DestFilePath "myDirectory2\SampleCopy.txt" `
    -DestContext $storageAcct.Context
```

Если сейчас вывести список файлов в новом файловом ресурсе, вы увидите скопированный файл.

```azurepowershell-interactive
Get-AzStorageFile `
    -Context $storageAcct.Context `
    -ShareName $otherShareName `
    -Path "myDirectory2" 
```

Хотя командлет `Start-AzStorageFileCopy` хорошо подходит для прямого перемещения файлов между общими папками Azure, для миграций и больших перемещений данных мы рекомендуем использовать `robocopy` в Windows и `rsync` в macOS и Linux. Для `robocopy` и `rsync` используется SMB, чтобы выполнять перемещения данных вместо REST API Файлов.

## <a name="create-and-manage-share-snapshots"></a>Создание моментальных снимков общих ресурсов и управление ими
Еще одна полезная задача, которую можно выполнить с файловым ресурсом Azure, — создать его моментальные снимки. Моментальный снимок сохраняет состояние файлового ресурса Azure на момент определенной точки во времени. Моментальные снимки общих ресурсов аналогичны тем, которые создаются с помощью уже знакомых вам технологий операционной системы, например:

- [служба теневого копирования томов (VSS)](/windows/desktop/VSS/volume-shadow-copy-service-portal) для файловых систем Windows, таких как NTFS и ReFS;
- [диспетчер логических томов (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) для систем Linux;
- [файловая система Apple (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) для macOS. 

Вы можете создать моментальный снимок, используя метод `Snapshot` для объекта PowerShell для общего файлового ресурса, полученного с помощью командлета [Get-AzStorageShare](/powershell/module/az.storage/Get-AzStorageShare). 

```azurepowershell-interactive
$share = Get-AzStorageShare -Context $storageAcct.Context -Name $shareName
$snapshot = $share.CloudFileShare.Snapshot()
```

### <a name="browse-share-snapshots"></a>Просмотр снимков общего ресурса
Вы можете просмотреть содержимое моментального снимка, передав ссылку на моментальный снимок (`$snapshot`) в параметр `-Share` командлета `Get-AzStorageFile`.

```azurepowershell-interactive
Get-AzStorageFile -Share $snapshot
```

### <a name="list-share-snapshots"></a>Вывод списка моментальных снимков общих ресурсов
С помощью следующей команды можно получить список сделанных моментальных снимков общего ресурса.

```azurepowershell-interactive
Get-AzStorageShare `
        -Context $storageAcct.Context | `
    Where-Object { $_.Name -eq $shareName -and $_.IsSnapshot -eq $true }
```

### <a name="restore-from-a-share-snapshot"></a>Восстановление из моментального снимка общего ресурса
Файл можно восстановить с помощью команды `Start-AzStorageFileCopy`, которая использовалась ранее. В целях этого краткого руководства мы сначала удалим ранее отправленный файл `SampleUpload.txt`. Теперь его можно восстановить из моментального снимка.

```azurepowershell-interactive
# Delete SampleUpload.txt
Remove-AzStorageFile `
    -Context $storageAcct.Context `
    -ShareName $shareName `
    -Path "myDirectory\SampleUpload.txt"

# Restore SampleUpload.txt from the share snapshot
Start-AzStorageFileCopy `
    -SrcShare $snapshot `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestContext $storageAcct.Context `
    -DestShareName $shareName `
    -DestFilePath "myDirectory\SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>Удаление моментального снимка общего ресурса
Моментальный снимок общего ресурса вы можете удалить с помощью командлета [Remove-AzStorageShare](/powershell/module/az.storage/Remove-AzStorageShare) с переменной, содержащей ссылку `$snapshot` в параметре `-Share`.

```azurepowershell-interactive
Remove-AzStorageShare `
    -Share $snapshot `
    -Confirm:$false `
    -Force
```

## <a name="clean-up-resources"></a>Очистка ресурсов
Закончив работу, вы можете удалить группу ресурсов и все связанные с ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). 

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

Кроме того, ресурсы можно удалить по одному.

- Чтобы удалить файловые ресурсы Azure, созданные для этого краткого руководства:

    ```azurepowershell-interactive
    Get-AzRmStorageShare -StorageAccount $storageAcct | Remove-AzRmStorageShare -Force
    ```

    > [!Note]  
    > Перед удалением общей папки Azure необходимо удалить все моментальные снимки общих ресурсов для общих папок Azure, которые вы создали.

- Чтобы удалить учетную запись хранения (при этом будут неявно удалены созданные файловые ресурсы Azure, а также другие созданные ресурсы хранения, такие как контейнер хранилища BLOB-объектов Azure).

    ```azurepowershell-interactive
    Remove-AzStorageAccount `
        -ResourceGroupName $storageAcct.ResourceGroupName `
        -Name $storageAcct.StorageAccountName
    ```

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Что представляет собой служба "Файлы Azure"?](storage-files-introduction.md)
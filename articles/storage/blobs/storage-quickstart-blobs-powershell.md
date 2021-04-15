---
title: Краткое руководство. Создание большого двоичного объекта с помощью PowerShell
titleSuffix: Azure Storage
description: В рамках этого краткого руководства вы используете Azure PowerShell в хранилище объектов (больших двоичных объектов). отправите большой двоичный объект в службу хранилища Azure, скачаете его и составите список больших двоичных объектов в контейнере с помощью Azure PowerShell.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 03/31/2020
ms.author: tamram
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f3fdeadcf90d94203a972946ccceefca74cd5c3d
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "107012487"
---
# <a name="quickstart-upload-download-and-list-blobs-with-powershell"></a>Краткое руководство. Отправка, скачивание и составление списка больших двоичных объектов с помощью PowerShell

С помощью модуля Azure PowerShell создайте ресурсы Azure и управляйте ими. Процессы создания ресурсов Azure и управления ими можно выполнить с помощью командной строки PowerShell или скриптов. В этом руководстве рассматривается использование PowerShell для передачи файлов между локальным диском и хранилищем BLOB-объектов Azure.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Для доступа к службе хранилища Azure требуется подписка Azure. Если у вас еще нет подписки, вы можете [создать бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Кроме того, вам потребуется роль участника данных Storage Blob для чтения, записи и удаления больших двоичных объектов и контейнеров службы хранилища Azure.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Для работы с этим кратким руководством требуется модуль Azure PowerShell 0.7 или более поздней версии. Чтобы узнать версию, выполните команду `Get-InstalledModule -Name Az -AllVersions | select Name,Version`. Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

[!INCLUDE [storage-quickstart-tutorial-intro-include-powershell](../../../includes/storage-quickstart-tutorial-intro-include-powershell.md)]

## <a name="create-a-container"></a>Создание контейнера

Большие двоичные объекты всегда отправляются в контейнер. Вы можете упорядочивать группы больших двоичных объектов аналогично организации файлов в папках на компьютере.

Задайте имя контейнера, а затем создайте контейнер с помощью командлета [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer). Задайте разрешения `blob`, чтобы предоставить открытый доступ к файлам. В этом примере контейнеру присвоено имя *quickstartblobs*.

```powershell
$containerName = "quickstartblobs"
New-AzStorageContainer -Name $containerName -Context $ctx -Permission blob
```

## <a name="upload-blobs-to-the-container"></a>Отправка BLOB-объектов в контейнер

Хранилище BLOB-объектов поддерживает блочные, добавочные и страничные BLOB-объекты. VHD-файлы, поддерживаемые для резервных виртуальных машин IaaS, являются страничными BLOB-объектами. Используйте добавочные большие двоичные объекты для ведения журнала, например, если требуется выполнять запись в файл и добавлять дополнительные сведения. Большинство файлов, находящихся в хранилище BLOB-объектов, представляют собой блочные BLOB-объекты. 

Для передачи файла в блочный BLOB-объект получите ссылку на контейнер, а затем получите ссылку на блочный BLOB-объект в этом контейнере. При наличии ссылки на большой двоичный объект вы можете отправлять в него данные с помощью командлета [Set-AzStorageBlobContent](/powershell/module/az.storage/set-azstorageblobcontent). Эта операция создает большой двоичный объект, если он еще не существует, или заменяет имеющийся.

В следующих примерах показана отправка файлов *Image001.jpg* и *Image002.png* из папки *D:\\_TestImages* на локальном диске в созданный контейнер.

```powershell
# upload a file to the default account (inferred) access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image000.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx 

# upload a file to the Hot access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image001.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx `
  -StandardBlobTier Hot

# upload another file to the Cool access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image002.png" `
  -Container $containerName `
  -Blob "Image002.png" `
  -Context $ctx `
  -StandardBlobTier Cool

# upload a file to a folder to the Archive access tier
Set-AzStorageBlobContent -File "D:\_TestImages\foldername\Image003.jpg" `
  -Container $containerName `
  -Blob "Foldername/Image003.jpg" `
  -Context $ctx `
  -StandardBlobTier Archive
```

Прежде чем продолжить, отправьте нужное количество файлов.

## <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Получите список больших двоичных объектов в контейнере с помощью командлета [Get-AzStorageBlob](/powershell/module/az.storage/get-azstorageblob). В этом примере показаны только имена отправленных BLOB-объектов.

```powershell
Get-AzStorageBlob -Container $ContainerName -Context $ctx | select Name
```

## <a name="download-blobs"></a>Скачивание больших двоичных объектов

Скачайте большие двоичные объекты на локальный диск. Для каждого скачиваемого большого двоичного объекта задайте имя и вызовите командлет [Get-AzStorageBlobContent](/powershell/module/az.storage/get-azstorageblobcontent).

В этом примере показано скачивание больших двоичных объектов на локальный диск *D:\\_TestImages\Downloads*. 

```powershell
# download first blob
Get-AzStorageBlobContent -Blob "Image001.jpg" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx 

# download another blob
Get-AzStorageBlobContent -Blob "Image002.png" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx
```

## <a name="data-transfer-with-azcopy"></a>Передача данных с помощью AzCopy

Служебная программа командной строки AzCopy обеспечивает высокопроизводительную передачу данных с поддержкой сценариев для службы хранилища Azure. AzCopy можно использовать для передачи данных из хранилища BLOB-объектов, Файлов Azure и обратно. Дополнительные сведения об AzCopy версии 10, которая является последней, см. в статье [Get started with AzCopy](../common/storage-use-azcopy-v10.md) (Начало работы с AzCopy). Дополнительные сведения об использовании AzCopy версии 10 с хранилищем BLOB-объектов см. в статье [Transfer data with AzCopy and Blob storage](../common/storage-use-azcopy-v10.md#transfer-data) (Передача данных с помощью AzCopy и хранилища BLOB-объектов).

В следующем примере AzCopy используется для передачи локального файла в большой двоичный объект. Не забудьте заменить примеры значений собственными значениями:

```powershell
azcopy login
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите все созданные ресурсы. Самым простым способом удалить ресурсы является удаление группы ресурсов. Удаление группы ресурсов также приведет к удалению всех ресурсов, которые она содержит. В следующем примере при удалении группы ресурсов удаляется учетная запись хранения и сама группа ресурсов.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы передали файлы между локальной файловой системой и хранилищем BLOB-объектов Azure. Чтобы узнать больше о работе с хранилищем BLOB-объектов с использованием PowerShell, изучите примеры Azure PowerShell для хранилища BLOB-объектов.

> [!div class="nextstepaction"]
> [Примеры Azure PowerShell для хранилища BLOB-объектов Azure](storage-samples-blobs-powershell.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

### <a name="microsoft-azure-powershell-storage-cmdlets-reference"></a>Справочник по командлетам PowerShell для службы хранилища Microsoft Azure

* [Командлеты PowerShell для службы хранилища](/powershell/module/az.storage)

### <a name="microsoft-azure-storage-explorer"></a>Обозреватель службы хранилища Microsoft Azure

* [Обозреватель хранилищ Microsoft Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.

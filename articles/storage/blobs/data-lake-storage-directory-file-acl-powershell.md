---
title: 'Управление данными с помощью PowerShell: Azure Data Lake Storage 2-го поколения'
description: Используйте командлеты PowerShell для управления каталогами и файлами в учетных записях хранения с включенным иерархическим пространством имен.
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 02/17/2021
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 552d53ff0257105ff61397e281504c5270512319
ms.sourcegitcommit: 87a6587e1a0e242c2cfbbc51103e19ec47b49910
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103573869"
---
# <a name="use-powershell-to-manage-directories-and-files-in-azure-data-lake-storage-gen2"></a>Использование PowerShell для управления каталогами и файлами в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать PowerShell для создания каталогов и файлов в учетных записях хранения с иерархическим пространством имен и управления ими.

Сведения о том, как получить, задать и обновить списки управления доступом (ACL) для каталогов и файлов, см. в разделе [Использование PowerShell для управления списками ACL в Azure Data Lake Storage 2-го поколения](data-lake-storage-acl-powershell.md).

[Справочные материалы](/powershell/module/Az.Storage/)  |  Сопоставление Gen1 с [Gen2](#gen1-gen2-map)  |  [Отправить отзыв](https://github.com/Azure/azure-powershell/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- Учетная запись хранения с включенным иерархическим пространством имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Платформа .NET Framework установлен или больше 4.7.2. См. раздел [Download платформа .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).

- Версия PowerShell `5.1` или выше.

## <a name="install-the-powershell-module"></a>Установка модуля PowerShell

1. Убедитесь, что установленная версия PowerShell `5.1` или более поздняя, с помощью следующей команды.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```

   Сведения об обновлении версии PowerShell см. в разделе [обновление существующих Windows PowerShell](/powershell/scripting/install/installing-windows-powershell#upgrading-existing-windows-powershell) .

2. Установите **AZ. Storage** Module.

   ```powershell
   Install-Module Az.Storage -Repository PSGallery -Force  
   ```

   Дополнительные сведения об установке модулей PowerShell см. [в статье Установка модуля Azure PowerShell](/powershell/azure/install-az-ps) .

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Выберите способ, которым команды должны получать права доступа к учетной записи хранения. 

### <a name="option-1-obtain-authorization-by-using-azure-active-directory-azure-ad"></a>Вариант 1. получение авторизации с помощью Azure Active Directory (Azure AD)

При таком подходе система гарантирует, что ваша учетная запись пользователя имеет соответствующие разрешения на управление доступом на основе ролей Azure (Azure RBAC) и ACL.

1. Откройте командное окно Windows PowerShell, а затем войдите в подписку Azure с помощью `Connect-AzAccount` команды и следуйте инструкциям на экране.

   ```powershell
   Connect-AzAccount
   ```

2. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку на подписку учетной записи хранения, которая будет создавать каталоги и управлять ими в. В этом примере замените `<subscription-id>` значение ЗАПОЛНИТЕЛЯ идентификатором подписки.

   ```powershell
   Select-AzSubscription -SubscriptionId <subscription-id>
   ``` 

3. Получение контекста учетной записи хранения.

   ```powershell
   $ctx = New-AzStorageContext -StorageAccountName '<storage-account-name>' -UseConnectedAccount
   ```

### <a name="option-2-obtain-authorization-by-using-the-storage-account-key"></a>Вариант 2. получение авторизации с помощью ключа учетной записи хранения

При таком подходе система не проверяет разрешения Azure RBAC или ACL. Получите контекст учетной записи хранения с помощью ключа учетной записи.

```powershell
$ctx = New-AzStorageContext -StorageAccountName '<storage-account-name>' -StorageAccountKey '<storage-account-key>'
```

## <a name="create-a-container"></a>Создание контейнера

Контейнер выступает в качестве файловой системы для файлов. Его можно создать с помощью `New-AzStorageContainer` командлета. 

В этом примере создается контейнер с именем `my-file-system` .

```powershell
$filesystemName = "my-file-system"
New-AzStorageContainer -Context $ctx -Name $filesystemName
```

## <a name="create-a-directory"></a>Создание каталога

Создайте ссылку на каталог с помощью `New-AzDataLakeGen2Item` командлета. 

В этом примере в контейнер добавляется каталог с именем `my-directory` .

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Directory
```

В этом примере добавляется тот же каталог, но также устанавливаются разрешения, umask, значения свойств и значения метаданных. 

```powershell
$dir = New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Directory -Permission rwxrwxrwx -Umask ---rwx---  -Property @{"ContentEncoding" = "UDF8"; "CacheControl" = "READ"} -Metadata  @{"tag1" = "value1"; "tag2" = "value2" }
```

## <a name="show-directory-properties"></a>Отображение свойств каталога

Этот пример получает каталог с помощью `Get-AzDataLakeGen2Item` командлета, а затем выводит значения свойств на консоль.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dir =  Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
$dir.Permissions
$dir.Group
$dir.Owner
$dir.Properties
$dir.Properties.Metadata
```

> [!NOTE]
> Чтобы получить корневой каталог контейнера, опустите `-Path` параметр.

## <a name="rename-or-move-a-directory"></a>Переименование или перемещение каталога

Переименуйте или переместите каталог с помощью `Move-AzDataLakeGen2Item` командлета.

В этом примере имя каталога переименовывается в имя `my-directory` `my-new-directory` .

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dirname2 = "my-new-directory/"
Move-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -DestFileSystem $filesystemName -DestPath $dirname2
```

> [!NOTE]
> Используйте `-Force` параметр, если требуется перезаписать без запросов.

В этом примере каталог с именем перемещается `my-directory` в подкаталог с `my-directory-2` именем `my-subdirectory` . 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dirname2 = "my-directory-2/my-subdirectory/"
Move-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -DestFileSystem $filesystemName -DestPath $dirname2
```

## <a name="delete-a-directory"></a>Удаление каталога

Удалите каталог с помощью `Remove-AzDataLakeGen2Item` командлета.

В этом примере удаляется каталог `my-directory`.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
Remove-AzDataLakeGen2Item  -Context $ctx -FileSystem $filesystemName -Path $dirname 
```

С помощью параметра можно `-Force` удалить файл без запроса.

## <a name="download-from-a-directory"></a>Скачивание из каталога

Скачайте файл из каталога с помощью `Get-AzDataLakeGen2ItemContent` командлета.

В этом примере из каталога `my-directory` скачивается файл `upload.txt`.

```powershell
$filesystemName = "my-file-system"
$filePath = "my-directory/upload.txt"
$downloadFilePath = "download.txt"
Get-AzDataLakeGen2ItemContent -Context $ctx -FileSystem $filesystemName -Path $filePath -Destination $downloadFilePath
```

## <a name="list-directory-contents"></a>Вывод содержимого каталогов

Перечислите содержимое каталога с помощью `Get-AzDataLakeGen2ChildItem` командлета. Можно использовать необязательный параметр, `-OutputUserPrincipalName` чтобы получить имя (вместо идентификатора объекта) пользователей.

В этом примере выводится содержимое каталога с именем `my-directory` .

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
Get-AzDataLakeGen2ChildItem -Context $ctx -FileSystem $filesystemName -Path $dirname -OutputUserPrincipalName
```

В следующем примере перечисляются `ACL` `Permissions` свойства,, `Group` и `Owner` для каждого элемента в каталоге. `-FetchProperty`Параметр необходим для получения значений для `ACL` Свойства. 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$properties = Get-AzDataLakeGen2ChildItem -Context $ctx -FileSystem $filesystemName -Path $dirname -Recurse -FetchProperty
$properties.ACL
$properties.Permissions
$properties.Group
$properties.Owner
```

> [!NOTE]
> Чтобы получить список содержимого корневого каталога контейнера, не указывайте `-Path` параметр.

## <a name="upload-a-file-to-a-directory"></a>Отправка файла в каталог

Отправьте файл в каталог с помощью `New-AzDataLakeGen2Item` командлета.

В этом примере в каталог `my-directory` отправляется файл `upload.txt`. 

```powershell
$localSrcFile =  "upload.txt"
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$destPath = $dirname + (Get-Item $localSrcFile).Name
New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $destPath -Source $localSrcFile -Force 
```

В этом примере загружается тот же файл, а затем задаются разрешения, umask, значения свойств и значения метаданных целевого файла. Этот пример также выводит эти значения на консоль.

```powershell
$file = New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $destPath -Source $localSrcFile -Permission rwxrwxrwx -Umask ---rwx--- -Property @{"ContentEncoding" = "UDF8"; "CacheControl" = "READ"} -Metadata  @{"tag1" = "value1"; "tag2" = "value2" }
$file1
$file1.Properties
$file1.Properties.Metadata

```

> [!NOTE]
> Чтобы передать файл в корневой каталог контейнера, опустите `-Path` параметр.

## <a name="show-file-properties"></a>Отображение свойств файла

Этот пример получает файл с помощью `Get-AzDataLakeGen2Item` командлета, а затем выводит значения свойств на консоль.

```powershell
$filepath =  "my-directory/upload.txt"
$filesystemName = "my-file-system"
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filepath
$file
$file.ACL
$file.Permissions
$file.Group
$file.Owner
$file.Properties
$file.Properties.Metadata
```

## <a name="delete-a-file"></a>Удаление файла

Удалите файл с помощью `Remove-AzDataLakeGen2Item` командлета.

В этом примере удаляется файл с именем `upload.txt` . 

```powershell
$filesystemName = "my-file-system"
$filepath = "upload.txt"
Remove-AzDataLakeGen2Item  -Context $ctx -FileSystem $filesystemName -Path $filepath 
```

С помощью параметра можно `-Force` удалить файл без запроса.

<a id="gen1-gen2-map"></a>

## <a name="gen1-to-gen2-mapping"></a>Сопоставление Gen1 с Gen2

В следующей таблице показано, как командлеты, используемые для Data Lake Storage 1-го поколения сопоставляться с командлетами для Data Lake Storage 2-го поколения.

|Командлет Data Lake Storage 1-го поколения| Командлет Data Lake Storage 2-го поколения| Примечания |
|--------|---------|-----|
|Get-AzDataLakeStoreChildItem|Get-AzDataLakeGen2ChildItem|По умолчанию командлет Get-AzDataLakeGen2ChildItem перечисляет только дочерние элементы первого уровня. Параметр-recursive Рекурсивно перечисляет дочерние элементы. |
|Get-AzDataLakeStoreItem<br>Get-AzDataLakeStoreItemAclEntry<br>Get-AzDataLakeStoreItemOwner<br>Get-AzDataLakeStoreItemPermission|Get-AzDataLakeGen2Item|Выходные элементы командлета Get-AzDataLakeGen2Item имеют следующие свойства: ACL, владелец, группа, разрешение.|
|Get-AzDataLakeStoreItemContent|Get-AzDataLakeGen2FileContent|Командлет Get-AzDataLakeGen2FileContent загрузить содержимое файла в локальный файл.|
|Move-AzDataLakeStoreItem|Move-AzDataLakeGen2Item||
|New-AzDataLakeStoreItem|New-AzDataLakeGen2Item|Этот командлет передает содержимое нового файла из локального файла.|
|Remove-AzDataLakeStoreItem|Remove-AzDataLakeGen2Item||
|Set-AzDataLakeStoreItemOwner<br>Set-AzDataLakeStoreItemPermission<br>Set-AzDataLakeStoreItemAcl|Update-AzDataLakeGen2Item|Командлет Update-AzDataLakeGen2Item обновляет только один элемент, а не рекурсивно. Если требуется обновлять рекурсивно, перечислите элементы с помощью командлета Get-AzDataLakeStoreChildItem, а затем выполните конвейер в командлете Update-AzDataLakeGen2Item.|
|Test-AzDataLakeStoreItem|Get-AzDataLakeGen2Item|Если элемент не существует, командлет Get-AzDataLakeGen2Item сообщит об ошибке.|

## <a name="see-also"></a>См. также раздел

- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Командлеты PowerShell для службы хранилища](/powershell/module/az.storage)

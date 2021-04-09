---
title: Краткое руководство по управлению файловыми ресурсами Azure с помощью Azure CLI
description: Из этого краткого руководства вы узнаете, как управлять службой "Файлы Azure" с помощью Azure CLI. Создайте группу ресурсов и учетную запись хранения, а затем создайте и используйте общую папку Azure.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/26/2018
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5611088b76d8acf785fc0951100dcd4a2f439250
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104593192"
---
# <a name="quickstart-create-and-manage-azure-file-shares-using-azure-cli"></a>Краткое руководство: создание файловых ресурсов Azure и управление ими с помощью Azure CLI
В этом руководстве рассматриваются основы работы с [файловыми ресурсами Azure](storage-files-introduction.md) с помощью Azure CLI. Общие файловые ресурсы Azure отличаются от других ресурсов тем, что хранятся в облаке и поддерживаются платформой Azure. Общие папки Azure поддерживают стандартный отраслевой протокол SMB и протокол NFS (предварительная версия), а также позволяют совместно использовать файлы на нескольких компьютерах, в нескольких приложениях и экземплярах. 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

- По умолчанию команды Azure CLI возвращают ответ в формате JSON. Это стандартный способ отправки и получения сообщений из REST API. Для облегчения работы с ответами в формате JSON в некоторых примерах в этой статье с командами Azure CLI используется параметр *запроса*. Этот параметр использует [язык запросов JMESPath](http://jmespath.org/) для синтаксического анализа JSON. Дополнительные сведения о том, как обрабатывать результаты выполнения команд Azure CLI с применением языка запросов JMESPath, см. в [руководстве по JMESPath](http://jmespath.org/tutorial.html).

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Если у вас еще нет группы ресурсов Azure, ее можно создать с помощью команды [az group create](/cli/azure/group). 

В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *Западная часть США 2*.

```azurecli-interactive 
export resourceGroupName="myResourceGroup"
region="westus2"

az group create \
    --name $resourceGroupName \
    --location $region \
    --output none
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения
Учетная запись хранения представляет собой общий пул носителей, который можно использовать для развертывания файловых ресурсов Azure или других ресурсов хранения, например больших двоичных объектов или очередей. Учетная запись хранения может содержать неограниченное количество файловых ресурсов. В общем ресурсе может храниться любое число файлов, насколько это позволяет емкость учетной записи хранения.

В следующем примере создается учетная запись хранения с использованием команды [az storage account create](/cli/azure/storage/account). Имена учетных записей хранения должны быть уникальными. Чтобы сделать имя уникальным, используйте команду `$RANDOM` для добавления к нему номера.

```azurecli-interactive 
export storageAccountName="mystorageacct$RANDOM"

az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --location $region \
    --kind StorageV2 \
    --sku Standard_LRS \
    --enable-large-file-share \
    --output none
```

> [!Note]  
> Общие папки более 5 ТиБ (максимум до 100 ТиБ для папки) доступны только для локально избыточных (LRS) и избыточных в пределах зоны (ZRS) учетных записей хранения. Чтобы создать геоизбыточную (GRS) или избыточную в пределах зоны (GZRS) учетную запись хранения, удалите параметр `--enable-large-file-share`.

### <a name="get-the-storage-account-key"></a>Получение ключа учетной записи хранения
Ключи учетной записи хранения определяют доступ к хранящихся в ней ресурсам. Эти ключи автоматически создаются при создании учетной записи хранения. Ключи учетной записи хранения можно получить с помощью команды [az storage account keys list](/cli/azure/storage/account/keys). 

```azurecli-interactive 
export storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroupName \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')
```

## <a name="create-an-azure-file-share"></a>создать файловый ресурс Azure;
Теперь можно создать первый файловый ресурс Azure. Используйте для этого команду [az storage share create](/cli/azure/storage/share). В этом примере создается файловый ресурс Azure с именем *myshare*. 

```azurecli-interactive
shareName="myshare"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --quota 1024 \
    --output none
```

Имена общих ресурсов должны содержать только строчные буквы, цифры и отдельные дефисы, и они не могут начинаться с дефиса. Дополнительные сведения о присвоении имен общим папкам и файлам см. в статье [Naming and Referencing Shares, Directories, Files, and Metadata](/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata) (Именование общих ресурсов, каталогов, файлов и метаданных и ссылка на них).

## <a name="use-your-azure-file-share"></a>Использование общего файлового ресурса Azure
Служба файлов Azure обеспечивает два способа работы с файлами и папками в общем файловом ресурсе Azure: стандартный отраслевой [протокол Server Message Block (SMB)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) и [протокол File REST](/rest/api/storageservices/file-service-rest-api). 

Чтобы установить общий файловый ресурс с помощью SMB, ознакомьтесь со следующим документом (с учетом вашей ОС):
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)
- [Windows](storage-how-to-use-files-windows.md)

### <a name="using-an-azure-file-share-with-the-file-rest-protocol"></a>Использование общего файлового ресурса Azure с протоколом File REST 
С протоколом File REST можно работать напрямую (вручную обрабатывать вызовы HTTP REST), но чаще всего он используется с помощью Azure CLI, [модуля Azure PowerShell](storage-how-to-use-files-powershell.md) или пакета SDK для службы хранилища Azure. Все это обеспечивает хорошую оболочку вокруг протокола File REST на любом языке скриптов или программирования.  

Ожидается, что в большинстве случаев служба файлов Azure будет использовать общий файловый ресурс Azure по протоколу SMB, так как это позволяет использовать имеющиеся приложения и инструменты, на использование которых у вас должна быть возможность, но есть несколько причин, по которым выгодно использовать File REST API, а не SMB, например:

- Вы подключаетесь к общему файловому ресурсу из Azure Bash Cloud Shell (которая не может подключать общие файловые ресурсы по SMB).
- Вы используете бессерверные ресурсы, такие как [Функции Azure](../../azure-functions/functions-overview.md). 
- Вы создаете службу с добавленной стоимостью, которая будет взаимодействовать со многими общими папками Azure, например выполнять резервное копирование или антивирусное сканирование.

В следующих примерах показано, как использовать Azure CLI для управления общим файловым ресурсом Azure с помощью протокола File REST. 

### <a name="create-a-directory"></a>Создание каталога
Чтобы создать каталог с именем *myDirectory* в корне файлового ресурса Azure, выполните команду [`az storage directory create`](/cli/azure/storage/directory).

```azurecli-interactive
az storage directory create \
   --account-name $storageAccountName \
   --account-key $storageAccountKey \
   --share-name $shareName \
   --name "myDirectory" \
   --output none
```

### <a name="upload-a-file"></a>Отправка файла
Чтобы продемонстрировать передачу файла с помощью команды [`az storage file upload`](/cli/azure/storage/file), сначала создайте файл для отправки на временном диске Cloud Shell. В следующем примере создается, а затем отправляется файл.

```azurecli-interactive
cd ~/clouddrive/
date > SampleUpload.txt

az storage file upload \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --source "SampleUpload.txt" \
    --path "myDirectory/SampleUpload.txt"
```

При работе в Azure CLI локально следует заменить `~/clouddrive` путем, который имеется на вашем компьютере.

После отправки файла можно использовать команду [`az storage file list`](/cli/azure/storage/file), чтобы убедиться, что файл отправлен в файловый ресурс Azure.

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory" \
    --output table
```

### <a name="download-a-file"></a>скачать файл;
С помощью команды [`az storage file download`](/cli/azure/storage/file) можно загрузить копию файла, отправленного на временный диск Cloud Shell.

```azurecli-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists, because you've run this example before
rm -f SampleDownload.txt

az storage file download \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory/SampleUpload.txt" \
    --dest "SampleDownload.txt" \
    --output none
```

### <a name="copy-files"></a>Копирование файлов
Одна из распространенных задач заключается в копировании файлов из одной общей папки в другую. Для демонстрации этой функциональности создайте новый файловый ресурс. Скопируйте файл, отправленный в только что созданный общий ресурс, с помощью команды [az storage file copy](/cli/azure/storage/file/copy). 

```azurecli-interactive
otherShareName="myshare2"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $otherShareName \
    --quota 1024 \
    --output none

az storage directory create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $otherShareName \
    --name "myDirectory2" \
    --output none

az storage file copy start \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --source-share $shareName \
    --source-path "myDirectory/SampleUpload.txt" \
    --destination-share $otherShareName \
    --destination-path "myDirectory2/SampleCopy.txt"
```

Если сейчас вывести список файлов в новом файловом ресурсе, вы увидите скопированный файл.

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $otherShareName \
    --path "myDirectory2" \
    --output table
```

Хотя команда `az storage file copy start` хорошо подходит для перемещения файлов между общими папками Azure, для миграций и больших перемещений данных мы рекомендуем использовать `rsync` в macOS и Linux и `robocopy` в Windows. Для `rsync` и `robocopy` используется SMB, чтобы выполнять перемещения данных вместо REST API Файлов.

## <a name="create-and-manage-share-snapshots"></a>Создание моментальных снимков общих ресурсов и управление ими
Еще одна полезная задача, которую можно выполнить с файловым ресурсом Azure, — создать его моментальные снимки. Моментальный снимок сохраняет состояние общего файлового ресурса Azure на момент определенной точки во времени. Моментальные снимки файловых ресурсов аналогичны тем, которые создаются с помощью уже знакомых вам технологий, например:

- [диспетчер логических томов (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) для систем Linux;
- [файловая система Apple (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) для macOS.
- [служба теневого копирования томов (VSS)](/windows/desktop/VSS/volume-shadow-copy-service-portal) для файловых систем Windows, таких как NTFS и ReFS.
 
Вы можете создать моментальный снимок общих ресурсов с помощью команды [`az storage share snapshot`](/cli/azure/storage/share).

```azurecli-interactive
snapshot=$(az storage share snapshot \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --query "snapshot" | tr -d '"')
```

### <a name="browse-share-snapshot-contents"></a>Просмотр содержимого моментального снимка общих ресурсов
Чтобы просмотреть содержимое моментального снимка общего ресурса, передайте его метку времени команде `az storage file list` в атрибуте переменной `$snapshot`.

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --snapshot $snapshot \
    --output table
```

### <a name="list-share-snapshots"></a>Вывод списка моментальных снимков общих ресурсов
Список имеющихся моментальных снимков общего ресурса можно получить с помощью следующей команды.

```azurecli-interactive
az storage share list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --include-snapshot \
    --query "[? name== '$shareName' && snapshot!=null].snapshot" \
    --output tsv
```

### <a name="restore-from-a-share-snapshot"></a>Восстановление из моментального снимка общего ресурса
Файл можно восстановить с помощью команды `az storage file copy start`, использовавшейся ранее. Сначала удалите отправленный файл SampleUpload.txt, чтобы его можно было восстановить из моментального снимка.

```azurecli-interactive
# Delete SampleUpload.txt
az storage file delete \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory/SampleUpload.txt" \
    --output none

# Build the source URI for a snapshot restore
URI=$(az storage account show \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --query "primaryEndpoints.file" | tr -d '"')

URI=$URI$shareName"/myDirectory/SampleUpload.txt?sharesnapshot="$snapshot

# Restore SampleUpload.txt from the share snapshot
az storage file copy start \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --source-uri $URI \
    --destination-share $shareName \
    --destination-path "myDirectory/SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>Удаление моментального снимка общего ресурса
Моментальный снимок общего ресурса можно удалить с помощью команды [`az storage share delete`](/cli/azure/storage/share). Воспользуйтесь переменной, содержащей ссылку `$SNAPSHOT` на параметр `--snapshot`.

```azurecli-interactive
az storage share delete \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --snapshot $snapshot \
    --output none
```

## <a name="clean-up-resources"></a>Очистка ресурсов
Закончив работу, удалите группу ресурсов и все связанные с ней ресурсы с помощью команды [`az group delete`](/cli/azure/group). 

```azurecli-interactive 
az group delete --name $resourceGroupName
```

Кроме того, ресурсы можно удалить по отдельности.
- Чтобы удалить файловые ресурсы Azure, созданные в ходе выполнения примеров из данной статьи:

    ```azurecli-interactive
    az storage share list \
            --account-name $storageAccountName \
            --account-key $storageAccountKey \
            --query "[].name" \
            --output tsv | \
        xargs -L1 bash -ec '\
            az storage share delete \
                --account-name "$storageAccountName" \
                --account-key "$storageAccountKey" \
                --name $0 \
                --delete-snapshots include \
                --output none'
    ```

- Чтобы удалить саму учетную запись хранения. (При этом опосредованно удаляются созданные файловые ресурсы Azure, а также другие созданные ресурсы хранения, например контейнер службы хранилища BLOB-объектов Azure.)

    ```azurecli-interactive
    az storage account delete \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --yes
    ```

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Что представляет собой служба "Файлы Azure"?](storage-files-introduction.md)

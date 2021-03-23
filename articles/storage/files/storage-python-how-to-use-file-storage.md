---
title: Разработка для файлов Azure с помощью Python | Документы Майкрософт
description: Узнайте, как разрабатывать приложения и службы Python, использующие файлы Azure для хранения данных файлов.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/08/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-python
ms.openlocfilehash: d45ce3a782d7ee145f769283b82e34647c78f26e
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104799876"
---
# <a name="develop-for-azure-files-with-python"></a>Разработка для файлов Azure с помощью Python

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Изучите основы использования Python для разработки приложений и служб, использующих службы файлов Azure для хранения данных файлов. Создайте простое консольное приложение и Узнайте, как выполнять основные действия с помощью Python и файлов Azure.

- создание файловых ресурсов Azure;
- Создание каталогов
- Перечисление файлов и каталогов в общей папке Azure
- Передача, загрузка и удаление файлов.
- Создание резервных копий файловых ресурсов с помощью моментальных снимков

> [!NOTE]
> Так как к файлам Azure можно обращаться по протоколу SMB, вы можете создавать простые приложения, которые получают доступ к файловым ресурсам Azure с использованием стандартных классов ввода-вывода и функций Python. В этой статье описывается написание приложений, использующих пакет SDK Python для хранилища файлов Azure, который использует службы [файлов azure REST API](/rest/api/storageservices/file-service-rest-api) для взаимодействия с файлами Azure.

## <a name="download-and-install-azure-storage-sdk-for-python"></a>Скачивание и установка пакета SDK службы хранилища Azure для Python

> [!NOTE]
> Если вы обновляете пакет SDK службы хранилища Azure для Python версии 0.36 или более ранней, удалите старый пакет SDK с помощью `pip uninstall azure-storage`, прежде чем устанавливать последнюю версию пакета.

# <a name="python-v12"></a>[Python v12](#tab/python)

[Клиентская библиотека хранилища файлов Azure версии 12. x для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-share) требует Python 2,7 или 3.6 +.

# <a name="python-v2"></a>[Python v2](#tab/python2)

[Для пакета SDK службы хранилища Azure для Python](https://github.com/azure/azure-storage-python) требуется Python 2,7 или 3.6 +.

---

## <a name="install-via-pypi"></a>Установка через PyPI

Для установки с помощью индекса пакетов Python (PyPI) введите:

# <a name="python-v12"></a>[Python v12](#tab/python)

```console
pip install azure-storage-file-share
```

# <a name="python-v2"></a>[Python v2](#tab/python2)

```console
pip install azure-storage-file
```

### <a name="view-the-sample-application"></a>Просмотр примера приложения

Чтобы просмотреть и запустить пример приложения, демонстрирующий использование Python с файлами Azure, см. статью Служба [хранилища Azure: Начало работы с файлами Azure в Python](https://github.com/Azure-Samples/storage-file-python-getting-started).

Чтобы запустить пример приложения, убедитесь, что установлены `azure-storage-file` `azure-storage-common` пакеты и.

---

## <a name="set-up-your-application-to-use-azure-files"></a>Настройка приложения для работы со службой файлов Azure

Добавьте следующий код в начало исходного файла Python, чтобы использовать фрагменты кода из этой статьи.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_Imports":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
from azure.storage.file import FileService
```

---

## <a name="set-up-a-connection-to-azure-files"></a>Настройка подключения к файлам Azure

# <a name="python-v12"></a>[Python v12](#tab/python)

[Шаресервицеклиент](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareserviceclient) позволяет работать с общими папками, каталогами и файлами. Следующий код создает объект, `ShareServiceClient` используя строку подключения учетной записи хранения.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateShareServiceClient":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Объект [FileService](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true) позволяет работать с общими папками, каталогами и файлами. Следующий код создает объект `FileService`, используя имя и ключ учетной записи хранения. Замените `<myaccount>` и `<mykey>` именем учетной записи и ключом.

```python
file_service = FileService(account_name='myaccount', account_key='mykey')
```

---

## <a name="create-an-azure-file-share"></a>Создание файлового ресурса Azure

# <a name="python-v12"></a>[Python v12](#tab/python)

В следующем примере кода используется объект [шареклиент](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient) для создания общего ресурса, если он не существует.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateFileShare":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

В следующем примере кода используется объект [FileService](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true) для создания общего ресурса, если он не существует.

```python
file_service.create_share('myshare')
```

---

## <a name="create-a-directory"></a>Создание каталога

Вы можете упорядочить файлы в хранилище, разместив их в подкаталогах, чтобы не захламлять корневой каталог.

# <a name="python-v12"></a>[Python v12](#tab/python)

Следующий метод создает каталог в корне указанного файлового ресурса с помощью объекта [шаредиректориклиент](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharedirectoryclient) .

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateDirectory":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

В следующем примере кода в корневом каталоге создается подкаталог с именем *sampledir*.

```python
file_service.create_directory('myshare', 'sampledir')
```

---

## <a name="upload-a-file"></a>Отправка файла

В этом разделе вы узнаете, как передать файл из локального хранилища в хранилище файлов Azure.

# <a name="python-v12"></a>[Python v12](#tab/python)

Следующий метод передает содержимое указанного файла в указанный каталог в указанной общей папке Azure.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_UploadFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Файловый ресурс Azure содержит по крайней мере корневой каталог, в котором могут размещаться файлы. Чтобы создать файл и передать данные, используйте методы [create_file_from_path](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-path-share-name--directory-name--file-name--local-file-path--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object---timeout-none-), [create_file_from_stream](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-stream-share-name--directory-name--file-name--stream--count--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--), [create_file_from_bytes](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-bytes-share-name--directory-name--file-name--file--index-0--count-none--content-settings-none--metadata-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--)или [create_file_from_text](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#create-file-from-text-share-name--directory-name--file-name--text--encoding--utf-8---content-settings-none--metadata-none--validate-content-false--timeout-none--file-permission-none--smb-properties--azure-storage-file-models-smbproperties-object--) . Они являются высокоуровневые методы, которые выполняют необходимое Фрагментирование, когда размер данных превышает 64 МБ.

Метод `create_file_from_path` передает содержимое файла из указанного пути, а метод `create_file_from_stream` передает содержимое уже открытого файла или потока. Метод `create_file_from_bytes` передает массив байтов, а `create_file_from_text` — указанное текстовое значение с использованием указанной кодировки (по умолчанию UTF-8).

В примере далее содержимое файла *sunset.png* передается в файл **myfile**.

```python
from azure.storage.file import ContentSettings
file_service.create_file_from_path(
    'myshare',
    None,  # We want to create this file in the root directory, so we specify None for the directory_name
    'myfile',
    'sunset.png',
    content_settings=ContentSettings(content_type='image/png'))
```

---

## <a name="enumerate-files-and-directories-in-an-azure-file-share"></a>Перечисление файлов и каталогов в общей папке Azure

# <a name="python-v12"></a>[Python v12](#tab/python)

Чтобы получить список файлов и каталогов в подкаталоге, используйте метод [list_directories_and_files](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient#list-directories-and-files-directory-name-none--name-starts-with-none--marker-none----kwargs-) . Этот метод возвращает Iterable автоматического разбиения на страницы. Следующий код выводит **имя** каждого файла и подкаталог в указанном каталоге в консоль.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_ListFilesAndDirs":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Чтобы получить список файлов и каталогов в общей папке, используйте метод [list_directories_and_files](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#list-directories-and-files-share-name--directory-name-none--num-results-none--marker-none--timeout-none--prefix-none--snapshot-none-) . Этот метод возвращает генератор. Приведенный далее код выводит в консоль **имя** каждого файла и каталога в общем ресурсе.

```python
generator = file_service.list_directories_and_files('myshare')
for file_or_dir in generator:
    print(file_or_dir.name)
```

---

## <a name="download-a-file"></a>Загрузка файла

# <a name="python-v12"></a>[Python v12](#tab/python)

Чтобы загрузить данные из файла, используйте [download_file](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharefileclient#download-file-offset-none--length-none----kwargs-).

В следующем примере демонстрируется использование `download_file` для получения содержимого указанного файла и его локального сохранения с **загрузкой** в начале файла.

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DownloadFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Чтобы загрузить данные из файла, используйте [get_file_to_path](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-path-share-name--directory-name--file-name--file-path--open-mode--wb---start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-), [get_file_to_stream](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-stream-share-name--directory-name--file-name--stream--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-), [get_file_to_bytes](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-bytes-share-name--directory-name--file-name--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-)или [get_file_to_text](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#get-file-to-text-share-name--directory-name--file-name--encoding--utf-8---start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--timeout-none--snapshot-none-). Они являются высокоуровневые методы, которые выполняют необходимое Фрагментирование, когда размер данных превышает 64 МБ.

В следующем примере показано использование метода `get_file_to_path` для загрузки содержимого файла **myfile** и его сохранения в файл *out-sunset.png*.

```python
file_service.get_file_to_path('myshare', None, 'myfile', 'out-sunset.png')
```

---

## <a name="create-a-share-snapshot"></a>Создание моментального снимка общего ресурса

Вы можете создать копию на момент времени всего общего файлового ресурса.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_CreateSnapshot":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
snapshot = file_service.snapshot_share(share_name)
snapshot_id = snapshot.snapshot
```

**Создание моментального снимка общего ресурса с метаданными**

```python
metadata = {"foo": "bar"}
snapshot = file_service.snapshot_share(share_name, metadata=metadata)
```

---

## <a name="list-shares-and-snapshots"></a>Список общих ресурсов и моментальных снимков

Вы можете вывести список всех моментальных снимков для определенного общего ресурса.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_ListSharesAndSnapshots":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
shares = list(file_service.list_shares(include_snapshots=True))
```

---

## <a name="browse-share-snapshot"></a>Просмотр снимков общего ресурса

Можно просмотреть каждый моментальный снимок общей папки, чтобы получить файлы и каталоги с этого момента времени.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_BrowseSnapshotDir":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
directories_and_files = list(
    file_service.list_directories_and_files(share_name, snapshot=snapshot_id))
```

---

## <a name="get-file-from-share-snapshot"></a>Получение файлов из моментальных снимков общих ресурсов

Файл можно скачать из моментального снимка общего ресурса. Это позволяет восстановить предыдущую версию файла.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DownloadSnapshotFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
with open(FILE_PATH, 'wb') as stream:
    file = file_service.get_file_to_stream(
        share_name, directory_name, file_name, stream, snapshot=snapshot_id)
```

---

## <a name="delete-a-single-share-snapshot"></a>Удаление одного моментального снимка общих ресурсов
Вы можете удалить один моментальный снимок общих ресурсов.

# <a name="python-v12"></a>[Python v12](#tab/python)

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteSnapshot":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

```python
file_service.delete_share(share_name, snapshot=snapshot_id)
```

---

## <a name="delete-a-file"></a>Удаление файла

# <a name="python-v12"></a>[Python v12](#tab/python)

Чтобы удалить файл, вызовите [delete_file](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.sharefileclient#delete-file---kwargs-).

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteFile":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Чтобы удалить файл, вызовите [delete_file](/python/api/azure-storage-file/azure.storage.file.fileservice.fileservice?view=azure-python-previous&preserve-view=true#delete-file-share-name--directory-name--file-name--timeout-none-).

```python
file_service.delete_file('myshare', None, 'myfile')
```

---

## <a name="delete-share-when-share-snapshots-exist"></a>Удаление общего ресурса с имеющимся моментальным снимком общих ресурсов

# <a name="python-v12"></a>[Python v12](#tab/python)

Чтобы удалить общую папку, содержащую моментальные снимки, вызовите [delete_share](/azure/developer/python/sdk/storage/azure-storage-file-share/azure.storage.fileshare.shareclient#delete-share-delete-snapshots-false----kwargs-) с помощью `delete_snapshots=True` .

:::code language="python" source="~/azure-storage-snippets/files/howto/python/python-v12/file_share_ops.py" id="Snippet_DeleteShare":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Нельзя удалить общий ресурс, содержащий моментальные снимки, пока все моментальные снимки не удалены.

```python
file_service.delete_share(share_name, delete_snapshots=DeleteSnapshot.Include)
```

---

## <a name="next-steps"></a>Следующие шаги

Теперь, когда вы узнали, как работать с файлами Azure с помощью Python, воспользуйтесь следующими ссылками для получения дополнительных сведений.

- [Центр по разработке для Python](/azure/developer/python/)
- [API-интерфейс REST служб хранилища Azure](/rest/api/azure/)
- [пакет SDK для службы хранилища Microsoft Azure для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage)

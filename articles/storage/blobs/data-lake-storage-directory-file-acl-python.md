---
title: Использование Python для управления данными в Azure Data Lake Storage 2-го поколения
description: Использование Python для управления каталогами и файлами в учетных записях хранения с включенным иерархическим пространством имен.
author: normesta
ms.service: storage
ms.date: 02/17/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-python
ms.openlocfilehash: a143c0aa19241b532cabff95fe6bf85679e4007c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100652298"
---
# <a name="use-python-to-manage-directories-and-files-in-azure-data-lake-storage-gen2"></a>Использование Python для управления каталогами и файлами в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать Python для создания каталогов и файлов в учетных записях хранения с иерархическим пространством имен и управления ими.

Сведения о том, как получить, задать и обновить списки управления доступом (ACL) для каталогов и файлов, см. в разделе [Использование Python для управления списками ACL в Azure Data Lake Storage 2-го поколения](data-lake-storage-acl-python.md).

[Пакет (индекс пакета Python)](https://pypi.org/project/azure-storage-file-datalake/)  |  [Примеры](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples)  |  [Справочник](/python/api/azure-storage-file-datalake/azure.storage.filedatalake)  |  по API Сопоставление Gen1 с [Gen2](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)  |  [Отправить отзыв](https://github.com/Azure/azure-sdk-for-python/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- Учетная запись хранения с включенным иерархическим пространством имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

## <a name="set-up-your-project"></a>Настройка проекта

Установите клиентскую библиотеку Azure Data Lake Storage для Python с помощью [PIP](https://pypi.org/project/pip/).

```
pip install azure-storage-file-datalake
```

Добавьте эти операторы импорта в начало файла кода.

```python
import os, uuid, sys
from azure.storage.filedatalake import DataLakeServiceClient
from azure.core._match_conditions import MatchConditions
from azure.storage.filedatalake._models import ContentSettings
```

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Чтобы использовать фрагменты кода в этой статье, необходимо создать экземпляр **даталакесервицеклиент** , представляющий учетную запись хранения. 

### <a name="connect-by-using-an-account-key"></a>Подключение с помощью ключа учетной записи

Это самый простой способ подключения к учетной записи. 

В этом примере создается экземпляр **даталакесервицеклиент** с помощью ключа учетной записи.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_AuthorizeWithKey":::

- Замените значение заполнителя `storage_account_name` именем вашей учетной записи хранения.

- Замените `storage_account_key` значение заполнителя ключом доступа к учетной записи хранения.

### <a name="connect-by-using-azure-active-directory-azure-ad"></a>Подключение с помощью Azure Active Directory (Azure AD)

Для проверки подлинности приложения в Azure AD можно использовать [клиентскую библиотеку удостоверений Azure для Python](https://pypi.org/project/azure-identity/) .

В этом примере создается экземпляр **даталакесервицеклиент** с использованием идентификатора клиента, секрета клиента и идентификатора клиента.  Чтобы получить эти значения, см. статью [Получение маркера из Azure AD для авторизации запросов из клиентского приложения](../common/storage-auth-aad-app.md).

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_AuthorizeWithAAD":::

> [!NOTE]
> Дополнительные примеры см. в документации по [клиентской библиотеке Azure Identity для Python](https://pypi.org/project/azure-identity/) .

## <a name="create-a-container"></a>Создание контейнера

Контейнер выступает в качестве файловой системы для файлов. Его можно создать, вызвав метод **FileSystemDataLakeServiceClient.create_file_system** .

В этом примере создается контейнер с именем `my-file-system` .

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_CreateContainer":::

## <a name="create-a-directory"></a>Создание каталога

Создайте ссылку на каталог, вызвав метод **FileSystemClient.create_directory** .

В этом примере в контейнер добавляется каталог с именем `my-directory` . 

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_CreateDirectory":::

## <a name="rename-or-move-a-directory"></a>Переименование или перемещение каталога

Переименуйте или переместите каталог, вызвав метод **DataLakeDirectoryClient.rename_directory** . Передайте путь к нужному каталогу параметру. 

В этом примере подкаталог переименовывается в имя `my-subdirectory-renamed` .

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_RenameDirectory":::

## <a name="delete-a-directory"></a>Удаление каталога

Удалите каталог, вызвав метод **DataLakeDirectoryClient.delete_directory** .

В этом примере удаляется каталог `my-directory`.  

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_DeleteDirectory":::

## <a name="upload-a-file-to-a-directory"></a>Отправка файла в каталог

Сначала создайте ссылку на файл в целевом каталоге, создав экземпляр класса **даталакефилеклиент** . Отправьте файл, вызвав метод **DataLakeFileClient.append_data** . Обязательно завершите передачу, вызвав метод **DataLakeFileClient.flush_data** .

В этом примере текстовый файл перегружается в каталог с именем `my-directory` .   

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_UploadFile":::

> [!TIP]
> Если размер файла большой, код должен выполнить несколько вызовов метода **DataLakeFileClient.append_data** . Вместо этого рекомендуется использовать метод **DataLakeFileClient.upload_data** . Таким образом, вы можете передать весь файл в одном вызове. 

## <a name="upload-a-large-file-to-a-directory"></a>Отправка большого файла в каталог

Используйте метод **DataLakeFileClient.upload_data** для отправки больших файлов без выполнения нескольких вызовов метода **DataLakeFileClient.append_data** .

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_UploadFileBulk":::

## <a name="download-from-a-directory"></a>Скачивание из каталога 

Откройте локальный файл для записи. Затем создайте экземпляр **даталакефилеклиент** , представляющий файл, который требуется скачать. Вызовите **DataLakeFileClient.read_file** , чтобы прочитать байты из файла, а затем запишите эти байты в локальный файл. 

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_DownloadFromDirectory":::

## <a name="list-directory-contents"></a>Вывод содержимого каталогов

Перечислите содержимое каталога, вызвав метод **FileSystemClient.get_paths** , а затем перечислите результаты.

В этом примере выводится путь к каждому подкаталогу и файлу, расположенному в каталоге с именем `my-directory` .

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_ListFilesInDirectory":::

## <a name="see-also"></a>См. также раздел

- [Справочная документация по API](/python/api/azure-storage-file-datalake/azure.storage.filedatalake)
- [Пакет (индекс пакета Python)](https://pypi.org/project/azure-storage-file-datalake/)
- [Примеры](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples)
- [Сопоставление 1-го и 2-го поколения](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)
- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Оставить отзыв](https://github.com/Azure/azure-sdk-for-python/issues)
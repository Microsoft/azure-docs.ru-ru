---
title: Python. Операции файловой системы в Azure Data Lake Storage 1-го поколения | Документация Майкрософт
description: Узнайте, как использовать пакет SDK для Python для работы с файловой системой Data Lake Storage 1-го поколения.
services: data-lake-store
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-python
ms.openlocfilehash: 8ca455a802b180163579f67104a61f455dd54f94
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92109227"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-python"></a>Операции файловой системы в Azure Data Lake Storage 1-го поколения с использованием пакета SDK для Python
> [!div class="op_single_selector"]
> * [Пакет SDK для .NET](data-lake-store-data-operations-net-sdk.md)
> * [пакет SDK для Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

В этой статье содержатся сведения об использовании пакета SDK для Python для выполнения операций файловой системы в Azure Data Lake Storage 1-го поколения. Дополнительные сведения о том, как выполнять операции управления учетными записями в Data Lake Storage 1-го поколения с помощью пакета SDK для Python, см. в [этой статье](data-lake-store-get-started-python.md).

## <a name="prerequisites"></a>Предварительные условия

* **Python**. Скачать Python можно [здесь](https://www.python.org/downloads/). В этой статье используется версия Python 3.6.2.

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Учетная запись Azure Data Lake Storage 1-го поколения**. Следуйте инструкциям из статьи [Начало работы с Azure Data Lake Storage Gen1 с помощью портала Azure](data-lake-store-get-started-portal.md).

## <a name="install-the-modules"></a>Установка модулей

Для работы с Data Lake Storage 1-го поколения с использованием Python необходимо установить три модуля.

* Модуль `azure-mgmt-resource`, который включает в себя модули Azure для Active Directory и т. д.
* Модуль `azure-mgmt-datalake-store`, который включает в себя операции по управлению учетной записью Azure Data Lake Storage 1-го поколения. Дополнительные сведения см. в [справочнике по модулю azure-mgmt-datalake-store](/python/api/azure-mgmt-datalake-store/).
* Модуль `azure-datalake-store`, который включает в себя операции с файловой системой Azure Data Lake Storage 1-го поколения. Дополнительные сведения см. в [справочнике по модулю azure-datalake-store file-system](/python/api/azure-datalake-store/azure.datalake.store.core/).

Чтобы установить модули, используйте следующие команды.

```console
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>Создание приложения Python

1. Для создания приложения Python используйте IDE по своему усмотрению, например **mysample.py**.

2. Чтобы импортировать необходимые модули, добавьте следующие строки.

   ```python
   ## Use this only for Azure AD service-to-service authentication
   from azure.common.credentials import ServicePrincipalCredentials

   ## Use this only for Azure AD end-user authentication
   from azure.common.credentials import UserPassCredentials

   ## Use this only for Azure AD multi-factor authentication
   from msrestazure.azure_active_directory import AADTokenCredentials

   ## Required for Azure Data Lake Storage Gen1 account management
   from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
   from azure.mgmt.datalake.store.models import DataLakeStoreAccount

   ## Required for Azure Data Lake Storage Gen1 filesystem management
   from azure.datalake.store import core, lib, multithread

   ## Common Azure imports
   from azure.mgmt.resource.resources import ResourceManagementClient
   from azure.mgmt.resource.resources.models import ResourceGroup

   ## Use these as needed for your application
   import logging, getpass, pprint, uuid, time
   ```

3. Сохраните изменения в mysample.py.

## <a name="authentication"></a>Аутентификация

В этом разделе мы рассмотрим различные способы проверки подлинности в Azure AD. Доступны следующие варианты.

* Дополнительные сведения о проверке подлинности пользователей в приложении см. в статье [Аутентификация пользователей в Data Lake Store с помощью Python](data-lake-store-end-user-authenticate-python.md).
* Дополнительные сведения о проверке подлинности между службами в приложении см. в статье [Аутентификация между службами в Data Lake Store с помощью Python](data-lake-store-service-to-service-authenticate-python.md).

## <a name="create-filesystem-client"></a>Создание файловой системы клиента

Приведенный ниже фрагмент кода сначала создает клиент учетной записи Data Lake Storage 1-го поколения. Затем он использует объект клиента для создания учетной записи Data Lake Storage 1-го поколения. И наконец, он создает объект клиента файловой системы.

```python
## Declare variables
subscriptionId = 'FILL-IN-HERE'
adlsAccountName = 'FILL-IN-HERE'

## Create a filesystem client object
adlsFileSystemClient = core.AzureDLFileSystem(adlCreds, store_name=adlsAccountName)
```

## <a name="create-a-directory"></a>Создание каталога

```python
## Create a directory
adlsFileSystemClient.mkdir('/mysampledirectory')
```

## <a name="upload-a-file"></a>Отправка файла

```python
## Upload a file
multithread.ADLUploader(adlsFileSystemClient, lpath='C:\\data\\mysamplefile.txt', rpath='/mysampledirectory/mysamplefile.txt', nthreads=64, overwrite=True, buffersize=4194304, blocksize=4194304)
```


## <a name="download-a-file"></a>Загрузка файла

```python
## Download a file
multithread.ADLDownloader(adlsFileSystemClient, lpath='C:\\data\\mysamplefile.txt.out', rpath='/mysampledirectory/mysamplefile.txt', nthreads=64, overwrite=True, buffersize=4194304, blocksize=4194304)
```

## <a name="delete-a-directory"></a>Удаление каталога

```python
## Delete a directory
adlsFileSystemClient.rm('/mysampledirectory', recursive=True)
```

## <a name="next-steps"></a>Дальнейшие действия
* [Операции управления учетными записями в Data Lake Storage 1-го поколения с помощью Python](data-lake-store-get-started-python.md).

## <a name="see-also"></a>См. также раздел

* [Справочник по Azure Data Lake Storage 1-го поколения для Python. Файловая система](/python/api/azure-datalake-store/azure.datalake.store.core)
* [Приложения больших данных с открытым исходным кодом, которые работают с Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md)
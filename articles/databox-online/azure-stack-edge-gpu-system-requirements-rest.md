---
title: Требования к хранилищу BLOB-объектов Microsoft Azure Stack Документация Майкрософт
description: Дополнительные сведения о поддерживаемых версиях для API, пакетов SDK и клиентских библиотек для Azure Stack пограничного хранилища BLOB-объектов
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/21/2020
ms.author: alkohli
ms.openlocfilehash: 7deba32eb9d0e098b75f98cc81fac2c01b8bb7f8
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105567259"
---
# <a name="azure-stack-edge-blob-storage-requirements"></a>Требования к хранилищу BLOB-объектов Azure Stack

В этой статье перечислены версии интерфейсов API Azure, клиентских библиотек Azure и средств, поддерживаемых в Azure Stack пограничном хранилище больших двоичных объектов. Azure Stack пограничное хранилище больших двоичных объектов обеспечивает функции управления BLOB-объектами с помощью семантики, соответствующей Azure. В этой статье также приводится сводка по известным отличиям хранилища больших двоичных объектов Azure Stack в службах хранилища Azure.

Рекомендуется внимательно просмотреть сведения перед подключением к Azure Stack пограничному хранилищу BLOB-объектов, а затем при необходимости вернуться к нему.

## <a name="storage-differences"></a>Различия хранилищ

|     Компонент                                             |     Хранилище Azure                                     |     Azure Stack пограничное хранилище BLOB-объектов |
|---------------------------------------------------------|-------------------------------------------------------|---------------------------|
|    Хранилище файлов Azure                                   |    Поддерживаются облачные общие папки с файлами SMB              |    Не поддерживается      |
|    Тип учетной записи хранения                                 |    Учетные записи хранения общего назначения и учетные записи хранения BLOB-объектов Azure    |    Только общего назначения версии 1|
|    Имя большого двоичного объекта                                            |    1024 символов (2048 байт)                     |    880 символов (1760 байт)|
|    Максимальный размер блока большого двоичного объекта                              |    4,75 ТБ (100 МБ х 50 000 блоков)                   |    4,75 ТБ (100 МБ x 50 000 блоков) для Azure Stackного периметра|
|    Максимальный размер страничного BLOB-объекта                               |    8 ТБ                                               |    1 TБ                   |
|    Размер страницы страничного BLOB-объекта                                  |    512 байт                                          |    4 КБ                   |

## <a name="supported-api-versions"></a>Поддерживаемые версии API

В Azure Stack пограничном хранилище BLOB-объектов поддерживаются следующие версии API службы хранилища Azure.

### <a name="azure-stack-edge-2113772170-onwards"></a>Azure Stack пограничных 2.1.1377.2170 в сторону

[!INCLUDE [data-box-rest-supported-api-versions](../../includes/data-box-rest-supported-api-versions.md)]

## <a name="supported-azure-client-libraries"></a>Поддерживаемые клиентские библиотеки Azure

Для Azure Stackного хранилища BLOB-объектов существуют определенные клиентские библиотеки и конкретные требования к суффиксу конечной точки. Конечные точки хранилища BLOB-объектов Azure Stack пограничных данных не имеют полной четности с последней версией REST API хранилища BLOB-объектов Azure. см. [Поддерживаемые версии API для Azure Stack пограничных](#supported-api-versions)устройств. Для клиентских библиотек хранилища вам необходимо знать версию, совместимую с REST API.

### <a name="azure-stack-edge-2113772170-onwards"></a>Azure Stack пограничных 2.1.1377.2170 в сторону

Для Azure Stack пограничного хранилища BLOB-объектов поддерживаются следующие версии клиентской библиотеки Azure.

[!INCLUDE [data-box-rest-supported-azure-client-libraries](../../includes/data-box-rest-supported-azure-client-libraries.md)]

### <a name="install-the-php-client-via-composer---current"></a>Установка клиента PHP с помощью компоновщика (Current)

Чтобы установить клиент PHP с помощью Composer, выполните следующие действия.

1. Создайте файл с именем composer.jsв корневом каталоге проекта со следующим кодом (пример использует службу Azure Storage Blob Service).

    ```
    {
    "require": {
    "Microsoft/azure-storage-blob":"1.2.0"
    }
    ```

2. Загрузите `composer.phar` в корневую папку проекта.

3. Выполните команду: установка php composer.phar.


## <a name="endpoint-declaration"></a>Объявление конечной точки

В пакете SDK для хранилища BLOB-объектов Azure Stack пограничным суффиксом конечной точки является `<device serial number>.microsoftdatabox.com` определение домена Azure Stack. Дополнительные сведения о конечной точке службы BLOB-объектов см. в подразделах [Перемещение данных с помощью учетных записей хранения с помощью Azure Stack ребра Pro GPU](./azure-stack-edge-gpu-deploy-add-storage-accounts.md).


## <a name="examples"></a>Примеры

### <a name="net"></a>.NET

Для Azure Stack пограничного хранилища BLOB-объектов в файле указывается суффикс конечной точки `app.config` :

```
<add key="StorageConnectionString"
value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;
EndpointSuffix=<<serial no. of the device>.microsoftdatabox.com  />
```

### <a name="java"></a>Java

Для Azure Stack пограничного хранилища BLOB-объектов в программе установки строки подключения указывается суффикс конечной точки:

```
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key;" +
    "EndpointSuffix=<serial no. of the device>.microsoftdatabox.com ";
```

### <a name="nodejs"></a>Node.js

Для Azure Stack пограничного хранилища BLOB-объектов в экземпляре объявления указывается суффикс конечной точки:

```
var blobSvc = azure.createBlobService('myaccount', 'mykey',
'myaccount.blob. <serial no. of the device>.microsoftdatabox.com ');
```

### <a name="c"></a>C++

Для Azure Stack пограничного хранилища BLOB-объектов в программе установки строки подключения указывается суффикс конечной точки:

```
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;
AccountName=your_storage_account;
AccountKey=your_storage_account_key;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com "));
```

### <a name="php"></a>PHP

Для Azure Stack пограничного хранилища BLOB-объектов в программе установки строки подключения указывается суффикс конечной точки:

```
$connectionString = 'BlobEndpoint=http://<storage account name>.blob.<serial no. of the device>.microsoftdatabox.com /;
AccountName=<storage account name>;AccountKey=<storage account key>'
```

### <a name="python"></a>Python

Для Azure Stack пограничного хранилища BLOB-объектов в экземпляре объявления указывается суффикс конечной точки:

```
block_blob_service = BlockBlobService(account_name='myaccount',
account_key='mykey',
endpoint_suffix=’<serial no. of the device>.microsoftdatabox.com’)
```

### <a name="ruby"></a>Ruby

Для Azure Stack пограничного хранилища BLOB-объектов в программе установки строки подключения указывается суффикс конечной точки:

```
set
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;
AccountName=myaccount;
AccountKey=mykey;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com
```

## <a name="next-steps"></a>Дальнейшие действия

* [Подготовка к развертыванию Azure Stack пограничных Pro с помощью GPU](azure-stack-edge-gpu-deploy-prep.md)
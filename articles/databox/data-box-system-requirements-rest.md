---
title: Требования к хранилищу BLOB-объектов Microsoft Azure Data Box | Документация Майкрософт
description: Дополнительные сведения о поддерживаемых версиях пакетов SDK, API-интерфейсов, клиентских библиотек для хранилища BLOB-объектов Azure Data Box
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 10/05/2020
ms.author: alkohli
ms.openlocfilehash: ac5f2de383066d6ee399dac3b0ad8c365b2e72bc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91744116"
---
# <a name="azure-data-box-blob-storage-requirements"></a>Требования к хранилищу BLOB-объектов Azure Data Box

В этой статье перечислены версии интерфейсов API Azure, клиентских библиотек Azure и средств, поддерживаемых в хранилище BLOB-объектов Data Box. Хранилище BLOB-объектов Data Box предоставляет функции управления большими двоичными объектами с помощью семантики, согласованной с Azure. В этой статье также перечислены известные различия между хранилищем BLOB-объектов Azure Data Box и службой хранилища Azure.

Прежде чем подключиться к хранилищу BLOB-объектов Data Box, советуем внимательно ознакомиться с приведенной информацией и при необходимости обращаться к ней.


## <a name="storage-differences"></a>Различия хранилищ

|     Компонент                                             |     Хранилище Azure                                     |     Хранилище BLOB-объектов службы Data Box |
|---------------------------------------------------------|-------------------------------------------------------|---------------------------|
|    Хранилище файлов Azure                                   |    Поддерживаются облачные общие папки с файлами SMB              |    Не поддерживается      |
|    Шифрование службы для неактивных данных                  |    256-битное шифрование AES                             |    256-битное шифрование AES |
|    Тип учетной записи хранения                                 |    Учетные записи хранения общего назначения и учетные записи хранения BLOB-объектов Azure    |    Только общего назначения версии 1|
|    Имя большого двоичного объекта                                            |    1024 символов (2048 байт)                     |    880 символов (1760 байт)|
|    Максимальный размер блока большого двоичного объекта                              |    4,75 ТБ (100 МБ х 50 000 блоков)                   |    4,75 ТБ (100 МБ x 50 000 блоков) для Azure Data Box v 3,0.|
|    Максимальный размер страничного BLOB-объекта                               |    8 ТБ                                               |    1 TБ                   |
|    Размер страницы страничного BLOB-объекта                                  |    512 байт                                          |    4 КБ                   |

## <a name="supported-api-versions"></a>Поддерживаемые версии API

В хранилище BLOB-объектов Data Box поддерживаются следующие версии API службы хранилища Azure.

### <a name="azure-data-box-30-onwards"></a>Azure Data Box 3,0 в сторону

[!INCLUDE [data-box-rest-supported-api-versions](../../includes/data-box-rest-supported-api-versions.md)]

## <a name="supported-azure-client-libraries"></a>Поддерживаемые клиентские библиотеки Azure

Для хранилища BLOB-объектов службы Data Box существуют определенные клиентские библиотеки и определенные требования к суффиксу конечной точки. Конечные точки хранилища BLOB-объектов Data Box не имеют полной четности с последней версией REST API хранилища BLOB-объектов Azure; Ознакомьтесь с [поддерживаемыми версиями для Azure Data Box 3,0, начиная с версии](#supported-api-versions). Для клиентских библиотек хранилища вам необходимо знать версию, совместимую с REST API.

### <a name="azure-data-box-30-onwards"></a>Azure Data Box 3,0 в сторону

Для хранилища BLOB-объектов Data Box поддерживаются следующие версии клиентской библиотеки Azure.

[!INCLUDE [data-box-rest-supported-azure-client-libraries](../../includes/data-box-rest-supported-azure-client-libraries.md)]

### <a name="install-php-client-via-composer---current"></a>Установка клиента PHP через компоновщик — текущая версия

Для установки через компоновщик сделайте следующее (возьмите для примера большой двоичный объект).
1. Создайте файл с именем composer.json в корневом каталоге проекта со следующим кодом.

    ```
    {
    "require": {
    "Microsoft/azure-storage-blob":"1.2.0"
    }
    ```

2. Загрузите `composer.phar` в корневую папку проекта.

3. Выполните команду: установка php composer.phar.

### <a name="endpoint-declaration"></a>Объявление конечной точки

В пакете SDK для хранилища BLOB-объектов Data Box суффикс конечной точки — `<device serial number>.microsoftdatabox.com` идентифицирует домен Data Box. Чтобы получить дополнительные сведения о конечной точке службы BLOB-объектов, перейдите к разделу [подключение через Data Box хранилище BLOB-объектов](data-box-deploy-copy-data-via-rest.md).
 
## <a name="examples"></a>Примеры

### <a name="net"></a>.NET

Для хранилища BLOB-объектов Data Box суффикс конечной точки указывается в файле `app.config`.

```
<add key="StorageConnectionString"
value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;
EndpointSuffix=<<serial no. of the device>.microsoftdatabox.com  />
```

### <a name="java"></a>Java

Для хранилища BLOB-объектов Data Box суффикс конечной точки указывается в настройке строки подключения.

```
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key;" +
    "EndpointSuffix=<serial no. of the device>.microsoftdatabox.com ";
```

### <a name="nodejs"></a>Node.js

Для хранилища BLOB-объектов Data Box суффикс конечной точки указывается в экземпляре объявления.

```
var blobSvc = azure.createBlobService('myaccount', 'mykey',
'myaccount.blob. <serial no. of the device>.microsoftdatabox.com ');
```

### <a name="c"></a>C++

Для хранилища BLOB-объектов Data Box суффикс конечной точки указывается в настройке строки подключения.

```
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;
AccountName=your_storage_account;
AccountKey=your_storage_account_key;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com "));
```

### <a name="php"></a>PHP

Для хранилища BLOB-объектов Data Box суффикс конечной точки указывается в настройке строки подключения.

```
$connectionString = 'BlobEndpoint=http://<storage account name>.blob.<serial no. of the device>.microsoftdatabox.com /;
AccountName=<storage account name>;AccountKey=<storage account key>'
```

### <a name="python"></a>Python

Для хранилища BLOB-объектов Data Box суффикс конечной точки указывается в экземпляре объявления.

```
block_blob_service = BlockBlobService(account_name='myaccount',
account_key='mykey',
endpoint_suffix=’<serial no. of the device>.microsoftdatabox.com’)
```

### <a name="ruby"></a>Ruby

Для хранилища BLOB-объектов Data Box суффикс конечной точки указывается в настройке строки подключения.

```
set
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;
AccountName=myaccount;
AccountKey=mykey;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com
```

## <a name="next-steps"></a>Дальнейшие действия

* [Развертывание Azure Data Box](data-box-deploy-ordered.md)

---
title: Копирование данных из Amazon S3 в службу хранилища Azure с помощью AzCopy | Документация Майкрософт
description: Используйте AzCopy для копирования данных из Amazon S3 в службу хранилища Azure. AzCopy — это служебная программа командной строки, которую можно использовать для копирования больших двоичных объектов или файлов в учетную запись хранения или из нее.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 07/27/2020
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: 34f54bb30e959ecc2fa27fba5ab7392b9eddc68e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103494518"
---
# <a name="copy-data-from-amazon-s3-to-azure-storage-by-using-azcopy"></a>Копирование данных из Amazon S3 в службу хранилища Azure с помощью AzCopy

AzCopy — это служебная программа командной строки, которую можно использовать для копирования больших двоичных объектов или файлов в учетную запись хранения или из нее. Эта статья поможет вам скопировать объекты, каталоги и контейнеры из Amazon Web Services (AWS) S3 в хранилище BLOB-объектов Azure с помощью AzCopy.

## <a name="choose-how-youll-provide-authorization-credentials"></a>Выбор порядка предоставления учетных данных для авторизации

* Для авторизации в службе хранилища Azure используйте Azure Active Directory (AD) или маркер подписанного URL-адрес (SAS).

* Для авторизации с помощью AWS S3 используйте ключ доступа AWS и секретный ключ доступа.

### <a name="authorize-with-azure-storage"></a>Авторизация в службе хранилища Azure

Ознакомьтесь с статьей начало [работы с AzCopy](storage-use-azcopy-v10.md) , чтобы скачать AzCopy, и выберите способ предоставления учетных данных авторизации для службы хранилища.

> [!NOTE]
> В примерах в этой статье предполагается, что вы проверили подлинность удостоверения с помощью `AzCopy login` команды. Затем AzCopy использует учетную запись Azure AD для авторизации доступа к данным в хранилище BLOB-объектов.
>
> Если вы предпочитаете использовать маркер SAS для авторизации доступа к данным большого двоичного объекта, можно добавить этот маркер к URL-адресу ресурса в каждой команде AzCopy.
>
> Например, `https://mystorageaccount.blob.core.windows.net/mycontainer?<SAS-token>`.

### <a name="authorize-with-aws-s3"></a>Авторизация с помощью AWS S3

Соберите ключ доступа AWS и секретный ключ доступа, а затем задайте следующие переменные среды:

| Операционная система | Команда  |
|--------|-----------|
| **Windows** | `set AWS_ACCESS_KEY_ID=<access-key>`<br>`set AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **Linux** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>` |
| **macOS** | `export AWS_ACCESS_KEY_ID=<access-key>`<br>`export AWS_SECRET_ACCESS_KEY=<secret-access-key>`|

## <a name="copy-objects-directories-and-buckets"></a>Копирование объектов, каталогов и контейнеров

AzCopy использует [блок размещения из API URL-адреса](/rest/api/storageservices/put-block-from-url) , поэтому данные копируются непосредственно между AWS S3 и серверами хранилища. Эти операции копирования не используют пропускную способность сети компьютера.

> [!TIP]
> В примерах этого раздела аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

 Эти примеры также работают с учетными записями, имеющими иерархическое пространство имен. [Многопротокольный доступ на Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) позволяет использовать один и тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для этих учетных записей. 

### <a name="copy-an-object"></a>Копирование объекта

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<object-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<blob-name>'` |
| **Пример** | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://s3.amazonaws.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |

> [!NOTE]
> В примерах в этой статье используются URL-адреса в стиле пути для контейнеров AWS S3 (например: `http://s3.amazonaws.com/<bucket-name>` ). 
>
> Кроме того, можно использовать виртуальные URL-адреса в стиле (например: `http://bucket.s3.amazonaws.com` ). 
>
> Дополнительные сведения о виртуальном размещении сегментов см. в статье [виртуальное размещение сегментов](https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html).

### <a name="copy-a-directory"></a>Копирование каталога

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<directory-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Пример** | `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

> [!NOTE]
> В этом примере добавляется `--recursive` флаг для копирования файлов во всех подкаталогах.

### <a name="copy-the-contents-of-a-directory"></a>Копирование содержимого каталога

Вы можете скопировать содержимое каталога, не копируя сам каталог, используя подстановочный знак (*).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>/<directory-name>/*' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Пример** | `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-a-bucket"></a>Копирование контейнера

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://s3.amazonaws.com/<bucket-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive=true` |
| **Пример** | `azcopy copy 'https://s3.amazonaws.com/mybucket' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://s3.amazonaws.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive=true` |

### <a name="copy-all-buckets-in-all-regions"></a>Копировать все контейнеры во всех регионах

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://s3.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Пример** | `azcopy copy 'https://s3.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://s3.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |

### <a name="copy-all-buckets-in-a-specific-s3-region"></a>Копирование всех контейнеров в заданном регионе S3

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://s3-<region-name>.amazonaws.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Пример** | `azcopy copy 'https://s3-rds.eu-north-1.amazonaws.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://s3.amazonaws.com/mybucket' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

## <a name="handle-differences-in-object-naming-rules"></a>Обработку различий в правилах именования объектов

AWS S3 имеет разный набор соглашений об именовании для имен контейнеров по сравнению с контейнерами больших двоичных объектов Azure. Сведения о них можно прочитать [здесь](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules). Если вы решили скопировать группу контейнеров в учетную запись хранения Azure, операция копирования может завершиться ошибкой из-за различий в именах.

AzCopy обрабатывает две наиболее распространенные проблемы, которые могут возникнуть. контейнеры, содержащие периоды и сегменты, которые содержат последовательные дефисы. Имена контейнеров S3 AWS могут содержать точки и последовательные дефисы, но контейнер в Azure не может. AzCopy заменяет точки дефисами и последовательными дефисами числом, представляющим количество последовательных дефисов (например, контейнер с именем `my----bucket` преобразуется в `my-4-bucket` . 

Кроме того, так как AzCopy выполняет копирование файлов, он проверяет наличие конфликтов именования и пытается разрешить их. Например, если есть контейнеры с именем `bucket-name` и `bucket.name` , AzCopy разрешает контейнер с именем `bucket.name` First в `bucket-name` и затем в `bucket-name-2` .

## <a name="handle-differences-in-object-metadata"></a>Обработку различий в метаданных объекта

AWS S3 и Azure позволяют использовать разные наборы символов в именах объектных ключей. Вы можете ознакомиться с символами, [которые использует AWS](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html#object-keys)S3. На стороне Azure ключи объектов больших двоичных объектов соответствуют правилам именования [идентификаторов C#](/dotnet/csharp/language-reference/).

В составе команды AzCopy `copy` можно указать необязательный `s2s-handle-invalid-metadata` флаг, указывающий, как вы хотите управлять файлами, в которых метаданные файла содержат несовместимые имена ключей. В следующей таблице описывается каждое значение флага.

| Значение флага | Описание  |
|--------|-----------|
| **ексклудеифинвалид** | (Параметр по умолчанию) Метаданные не включаются в передаваемый объект. AzCopy регистрирует предупреждение. |
| **фаилифинвалид** | Объекты не копируются. AzCopy регистрирует ошибку и включает эту ошибку в число сбоев, которое отображается в сводке по перемещению.  |
| **ренамеифинвалид**  | AzCopy разрешает недопустимый ключ метаданных и копирует объект в Azure с помощью разрешенной пары "ключ — значение метаданных". Чтобы узнать, какие действия AzCopy для переименования объектных ключей, ознакомьтесь с разделом [как AzCopy переименовывает ключи объектов](#rename-logic) ниже. Если AzCopy не удается переименовать ключ, объект не будет скопирован. |

<a id="rename-logic"></a>

### <a name="how-azcopy-renames-object-keys"></a>Как AzCopy переименовывает объектные ключи

AzCopy выполняет следующие действия:

1. Заменяет недопустимые символы символом "_".

2. Добавляет строку `rename_` в начало нового допустимого ключа.

   Этот ключ будет использоваться для сохранения исходного **значения** метаданных.

3. Добавляет строку `rename_key_` в начало нового допустимого ключа.
   Этот ключ будет использоваться для сохранения исходных метаданных недопустимого **ключа**.
   Вы можете использовать этот ключ, чтобы попытаться восстановить метаданные на стороне Azure, так как ключ метаданных сохраняется в качестве значения в службе хранилища BLOB-объектов.

## <a name="next-steps"></a>Следующие шаги

Дополнительные примеры приведены в любой из следующих статей:

- [Начало работы с AzCopy](storage-use-azcopy-v10.md)

- [Передача данных](storage-use-azcopy-v10.md#transfer-data)

- [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)

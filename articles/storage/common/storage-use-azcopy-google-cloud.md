---
title: Копирование данных из Google Cloud Storage в службу хранилища Azure с помощью AzCopy | Документация Майкрософт
description: Используйте AzCopy для копирования данных из Google Cloud Storage в службу хранилища Azure. AzCopy — это служебная программа командной строки, которую можно использовать для копирования больших двоичных объектов или файлов в учетную запись хранения или из нее.
services: storage
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 03/09/2021
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: c0f030683954ede013f769bf8584e6cf82bab69f
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103555672"
---
# <a name="copy-data-from-google-cloud-storage-to-azure-storage-by-using-azcopy-preview"></a>Копирование данных из Google Cloud Storage в службу хранилища Azure с помощью AzCopy (Предварительная версия)

AzCopy — это служебная программа командной строки, которую можно использовать для копирования больших двоичных объектов или файлов в учетную запись хранения или из нее. Эта статья поможет вам скопировать объекты, каталоги и контейнеры из Google Cloud Storage в хранилище BLOB-объектов Azure с помощью AzCopy.

> [!IMPORTANT]
> Копирование данных из Google Cloud Storage в службу хранилища Azure в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="choose-how-youll-provide-authorization-credentials"></a>Выбор порядка предоставления учетных данных для авторизации

* Для авторизации в службе хранилища Azure используйте Azure Active Directory (AD) или маркер подписанного URL-адрес (SAS).

* Чтобы авторизовать облачное хранилище Google, используйте ключ учетной записи службы.

### <a name="authorize-with-azure-storage"></a>Авторизация в службе хранилища Azure

Ознакомьтесь с статьей начало [работы с AzCopy](storage-use-azcopy-v10.md) , чтобы скачать AzCopy и узнать о способах предоставления учетных данных авторизации в службе хранилища.

> [!NOTE] 
> В примерах, приведенных в этой статье, предполагается, что вы указали учетные данные авторизации с помощью Azure Active Directory (Azure AD).
>
> Если вы предпочитаете использовать маркер SAS для авторизации доступа к данным большого двоичного объекта, можно добавить этот маркер к URL-адресу ресурса в каждой команде AzCopy. Например, так: `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

### <a name="authorize-with-google-cloud-storage"></a>Авторизация с помощью облачного хранилища Google

Чтобы авторизовать облачное хранилище Google, вы будете использовать ключ учетной записи службы. Сведения о создании ключа учетной записи службы см. в статье [Создание ключей учетной записи службы и управление ими](https://cloud.google.com/iam/docs/creating-managing-service-account-keys).

После получения ключа службы задайте `GOOGLE_APPLICATION_CREDENTIALS` для переменной среды абсолютный путь к файлу ключа учетной записи службы:

| Операционная система | Get-Help  |
|--------|-----------|
| **Windows** | `set GOOGLE_APPLICATION_CREDENTIALS=<path-to-service-account-key>` |
| **Linux** | `export GOOGLE_APPLICATION_CREDENTIALS=<path-to-service-account-key>` |
| **macOS** | `export GOOGLE_APPLICATION_CREDENTIALS=<path-to-service-account-key>` |

## <a name="copy-objects-directories-and-buckets"></a>Копирование объектов, каталогов и контейнеров

AzCopy использует [блок размещения из API URL](/rest/api/storageservices/put-block-from-url) , поэтому данные копируются непосредственно между облачным хранилищем Google и серверами хранилища. Эти операции копирования не используют пропускную способность сети компьютера.

> [!TIP]
> В примерах этого раздела аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

 Эти примеры также работают с учетными записями, имеющими иерархическое пространство имен. [Многопротокольный доступ на Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) позволяет использовать один и тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для этих учетных записей. 

### <a name="copy-an-object"></a>Копирование объекта

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://storage.cloud.google.com/<bucket-name>/<object-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<blob-name>'` |
| **Пример** | `azcopy copy 'https://storage.cloud.google.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://storage.cloud.google.com/mybucket/myobject' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myblob'` |


### <a name="copy-a-directory"></a>Копирование каталога

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://storage.cloud.google.com/<bucket-name>/<directory-name>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Пример** | `azcopy copy 'https://storage.cloud.google.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://storage.cloud.google.com/mybucket/mydirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

> [!NOTE]
> В этом примере добавляется `--recursive` флаг для копирования файлов во всех подкаталогах.

### <a name="copy-the-contents-of-a-directory"></a>Копирование содержимого каталога

Вы можете скопировать содержимое каталога, не копируя сам каталог, используя подстановочный знак (*).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://storage.cloud.google.com/<bucket-name>/<directory-name>/*' 'https://<storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive=true` |
| **Пример** | `azcopy copy 'https://storage.cloud.google.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://storage.cloud.google.com/mybucket/mydirectory/*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/mydirectory' --recursive=true` |

### <a name="copy-a-cloud-storage-bucket"></a>Копирование контейнера облачного хранилища

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://storage.cloud.google.com/<bucket-name>' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Пример** | `azcopy copy 'https://storage.cloud.google.com/mybucket' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://storage.cloud.google.com/mybucket' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |

### <a name="copy-all-buckets-in-a-google-cloud-project"></a>Копирование всех контейнеров в проекте Google Cloud 

Сначала задайте `GOOGLE_CLOUD_PROJECT` идентификатор проекта Google Cloud.

Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://storage.cloud.google.com/' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Пример** | `azcopy copy 'https://storage.cloud.google.com/' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://storage.cloud.google.com' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |

### <a name="copy-a-subset-of-buckets-in-a-google-cloud-project"></a>Копирование подмножества контейнеров в облачном проекте Google 

Сначала задайте `GOOGLE_CLOUD_PROJECT` идентификатор проекта Google Cloud.

Скопируйте подмножество контейнеров, используя символ-шаблон (*) в имени контейнера. Используйте тот же синтаксис URL-адреса ( `blob.core.windows.net` ) для учетных записей, имеющих иерархическое пространство имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://storage.cloud.google.com/<bucket*name>' 'https://<storage-account-name>.blob.core.windows.net' --recursive=true` |
| **Пример** | `azcopy copy 'https://storage.cloud.google.com/my*bucket' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |
| **Пример** (иерархическое пространство имен)| `azcopy copy 'https://storage.cloud.google.com/my*bucket' 'https://mystorageaccount.blob.core.windows.net' --recursive=true` |

## <a name="handle-differences-in-bucket-naming-rules"></a>Обработку различий в правилах именования контейнеров

Облачное хранилище Google имеет разный набор соглашений об именовании для имен контейнеров по сравнению с контейнерами больших двоичных объектов Azure. Сведения о них можно прочитать [здесь](https://cloud.google.com/storage/docs/naming-buckets). Если вы решили скопировать группу контейнеров в учетную запись хранения Azure, операция копирования может завершиться ошибкой из-за различий в именах.

AzCopy обрабатывает три из наиболее распространенных проблем, которые могут возникнуть. контейнеры, которые содержат периоды, контейнеры, содержащие последовательные дефисы, и контейнеры, содержащие символы подчеркивания. Имена контейнеров Google Cloud Storage могут содержать точки и последовательные дефисы, но контейнер в Azure не может. AzCopy заменяет точки дефисами и последовательными дефисами числом, представляющим количество последовательных дефисов (например, контейнер с именем `my----bucket` преобразуется в `my-4-bucket` . Если имя контейнера содержит символ подчеркивания ( `_` ), AzCopy заменяет подчеркивание дефисом (например: контейнер называется `my_bucket` `my-bucket` . 

## <a name="handle-differences-in-object-naming-rules"></a>Обработку различий в правилах именования объектов

Облачное хранилище Google имеет разный набор соглашений об именовании для имен объектов по сравнению с BLOB-объектами Azure. Сведения о них можно прочитать [здесь](https://cloud.google.com/storage/docs/naming-objects).

В службе хранилища Azure имена объектов (или любой сегмент в пути к виртуальному каталогу) не разрешается завершать точками в конце (например `my-bucket...` ,). Конечные точки обрезаются при выполнении операции копирования. 

## <a name="handle-differences-in-object-metadata"></a>Обработку различий в метаданных объекта

Облачное хранилище Google и Azure позволяют использовать разные наборы символов в именах объектных ключей. Информацию о метаданных в Google Cloud Storage можно прочитать [здесь](https://cloud.google.com/storage/docs/metadata). На стороне Azure ключи объектов больших двоичных объектов соответствуют правилам именования [идентификаторов C#](/dotnet/csharp/language-reference/).

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

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры приведены в любой из следующих статей:

- [Начало работы с AzCopy](storage-use-azcopy-v10.md)

- [Передача данных](storage-use-azcopy-v10.md#transfer-data)

- [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)

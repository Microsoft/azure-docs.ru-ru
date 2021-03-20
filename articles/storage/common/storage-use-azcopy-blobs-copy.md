---
title: Копирование BLOB-объектов между учетными записями хранения Azure с помощью AzCopy V10 | Документация Майкрософт
description: Эта статья содержит набор AzCopy примеров команд, которые помогают копировать большие двоичные объекты между учетными записями хранения.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/08/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 524e3f70f9588cfae2c739722fc1a44e683f9a7f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97617295"
---
# <a name="copy-blobs-between-azure-storage-accounts-by-using-azcopy-v10"></a>Копирование больших двоичных объектов между учетными записями хранения Azure с помощью AzCopy V10

Вы можете копировать большие двоичные объекты, каталоги и контейнеры между учетными записями хранения с помощью служебной программы командной строки AzCopy V10. 

Примеры других типов задач, таких как отправка файлов, Загрузка больших двоичных объектов и синхронизация с хранилищем BLOB-объектов, см. в ссылках, представленных в разделе [дальнейшие действия](#next-steps) этой статьи.

AzCopy использует [API](/rest/api/storageservices/put-page-from-url)" [сервер-сервер](/rest/api/storageservices/put-block-from-url) ", поэтому данные копируются непосредственно между серверами хранилища. Эти операции копирования не используют пропускную способность сети компьютера.

Чтобы скачать AzCopy и узнать о способах предоставления учетных данных авторизации в службе хранилища, см. статью Начало [работы с AzCopy](storage-use-azcopy-v10.md). 

## <a name="guidelines"></a>Рекомендации

Примените следующие рекомендации к командам AzCopy. 

- Клиент должен иметь доступ к сети как к исходной, так и к конечной учетной записи хранения. Сведения о настройке параметров сети для каждой учетной записи хранения см. в статье [Настройка брандмауэров службы хранилища Azure и виртуальных сетей](storage-network-security.md?toc=/azure/storage/blobs/toc.json).

- Добавьте маркер SAS к каждому исходному URL-адресу. 

  Если вы предоставляете учетные данные авторизации с помощью Azure Active Directory (Azure AD), маркер SAS можно опустить только из URL-адреса назначения. Убедитесь, что в целевой учетной записи настроены соответствующие роли. См. [вариант 1. использование Azure Active Directory](storage-use-azcopy-v10.md?toc=/azure/storage/blobs/toc.json#option-1-use-azure-active-directory). 

  В примерах в этой статье предполагается, что вы прошли проверку подлинности с помощью Azure AD, поэтому в примерах не используются маркеры SAS из URL-адреса назначения.

-  При копировании в учетную запись хранилища блочных BLOB-объектов уровня "Премиум" исключите уровень доступа большого двоичного объекта из операции копирования, задав для значение (например, `s2s-preserve-access-tier` `false` `--s2s-preserve-access-tier=false` ). Учетные записи хранения блочных BLOB-объектов уровня "Премиум" не поддерживают уровни доступа. 

- При копировании в или из учетной записи с иерархическим пространством имен используйте `blob.core.windows.net` вместо `dfs.core.windows.net` в синтаксисе URL-адреса. [Многопротокольный доступ на Data Lake Storage](../blobs/data-lake-storage-multi-protocol-access.md) позволяет использовать `blob.core.windows.net` , и это единственный поддерживаемый синтаксис учетной записи для сценариев копирования учетных записей. 

- Можно увеличить пропускную способность операций копирования, задав значение `AZCOPY_CONCURRENCY_VALUE` переменной среды. Дополнительные сведения см. в разделе [Оптимизация пропускной способности](storage-use-azcopy-configure.md#optimize-throughput). 

- Если у исходных больших двоичных объектов есть теги index и вы хотите сохранить эти теги, необходимо повторно применить их к целевым BLOB-объектам. Сведения о том, как задать теги индекса, см. в разделе [Copy blobs to другой учетная запись хранения с тегами индекса](#copy-between-accounts-and-add-index-tags) этой статьи.

## <a name="copy-a-blob"></a>Копирование большого двоичного объекта

Скопируйте большой двоичный объект в другую учетную запись хранения с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) . 

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<blob-path>'` |
| **Пример** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt'` |

Операция копирования является синхронной, поэтому когда команда возвращает результат, это означает, что все файлы скопированы. 

## <a name="copy-a-directory"></a>Копирование каталога

Скопируйте каталог в другую учетную запись хранения с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) . 

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-path><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Пример** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

Операция копирования является синхронной, поэтому когда команда возвращает результат, это означает, что все файлы скопированы.

## <a name="copy-a-container"></a>Копирование контейнера

Скопируйте контейнер в другую учетную запись хранения с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) .

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<container-name><SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Пример** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

Операция копирования является синхронной, поэтому когда команда возвращает результат, это означает, что все файлы скопированы.

## <a name="copy-containers-directories-and-blobs"></a>Копирование контейнеров, каталогов и больших двоичных объектов

Скопируйте все контейнеры, каталоги и большие двоичные объекты в другую учетную запись хранения с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) .

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/<SAS-token>' 'https://<destination-storage-account-name>.blob.core.windows.net/' --recursive` |
| **Пример** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive` |

Операция копирования является синхронной, поэтому когда команда возвращает результат, это означает, что все файлы скопированы.

<a id="copy-between-accounts-and-add-index-tags"></a>

## <a name="copy-blobs-and-add-index-tags"></a>Копирование больших двоичных объектов и добавление тегов индекса

Скопируйте большие двоичные объекты в другую учетную запись хранения и добавьте [теги индекса BLOB-объектов (Предварительная версия)](../blobs/storage-manage-find-blobs.md) в целевой большой двоичный объект.

Если вы используете авторизацию Azure AD, участнику безопасности должна быть назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) , или ему должно быть предоставлено разрешение на `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` [операцию поставщика ресурсов Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) через настраиваемую роль Azure. Если вы используете маркер подписанного URL-адрес (SAS), этот маркер должен предоставить доступ к тегам большого двоичного объекта через `t` разрешение SAS.

Чтобы добавить теги, используйте `--blob-tags` параметр вместе с парой "ключ-значение" в кодировке URL. 

Например, чтобы добавить ключ `my tag` и значение `my tag value` , необходимо добавить `--blob-tags='my%20tag=my%20tag%20value'` в параметр Destination. 

Разделите несколько тегов индекса с помощью амперсанда ( `&` ).  Например, если нужно добавить ключ `my second tag` и значение `my second tag value` , то полная строка параметра будет иметь вид `--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` .

В следующих примерах показано, как использовать `--blob-tags` параметр.

> [!TIP]
> В этих примерах аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Большой двоичный объект** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myTextFile.txt' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Каталог** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Контейнер** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags="--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Учетная запись** | `azcopy copy 'https://mysourceaccount.blob.core.windows.net/?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.blob.core.windows.net' --recursive --blob-tags="--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |

Операция копирования является синхронной, поэтому когда команда возвращает результат, это означает, что все файлы скопированы.

> [!NOTE]
> Если указать каталог, контейнер или учетную запись для источника, то все большие двоичные объекты, скопированные в назначение, будут иметь те же теги, которые вы указали в команде.

## <a name="copy-with-optional-flags"></a>Копирование с необязательными флагами

Вы можете настроить операцию копирования с помощью необязательных флагов. Вот несколько примеров.

|Сценарий|Флаг|
|---|---|
|Копирование больших двоичных объектов в виде блоков, страниц или добавочных BLOB-объектов.|**--BLOB-тип** = \[ BlockBlob \| PageBlob \| аппендблоб\]|
|Копирование на конкретный уровень доступа (например, уровень архива).|**--Block-BLOB-уровень** = \[ Нет \| горячий \| стильный \| Архив\]|
|Автоматическое распаковка файлов.|**--распаковка** = \[ \|деструктурирование gzip\]|

Полный список см. в разделе [Параметры](storage-ref-azcopy-copy.md#options). 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры см. в следующих статьях:

- [Примеры: Передать](storage-use-azcopy-blobs-upload.md)
- [Примеры: Скачивание](storage-use-azcopy-blobs-download.md)
- [Примеры: Synchronize](storage-use-azcopy-blobs-synchronize.md)
- [Примеры: Контейнеры Amazon S3](storage-use-azcopy-s3.md)
- [Примеры: службы файлов Azure](storage-use-azcopy-files.md)
- [Руководство. Перенос локальных данных в облачное хранилище с помощью AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)
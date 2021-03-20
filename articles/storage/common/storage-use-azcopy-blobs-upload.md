---
title: Отправка файлов в хранилище BLOB-объектов Azure с помощью AzCopy V10 | Документация Майкрософт
description: Эта статья содержит набор AzCopy примеров команд, которые помогают отправлять файлы в хранилище BLOB-объектов Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/11/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: ec88a3c740ceda7ccf352f8f32f94e2cd52d0988
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97358764"
---
# <a name="upload-files-to-azure-blob-storage-by-using-azcopy-v10"></a>Отправка файлов в хранилище BLOB-объектов Azure с помощью AzCopy V10

Вы можете передать файлы и каталоги в хранилище BLOB-объектов с помощью служебной программы командной строки AzCopy V10. 

Чтобы просмотреть примеры других типов задач, таких как загрузка больших двоичных объектов, синхронизация с хранилищем BLOB-объектов или копирование больших двоичных объектов между учетными записями, см. ссылки, представленные в разделе [дальнейшие действия](#next-steps) этой статьи.

## <a name="get-started"></a>Начало работы

Ознакомьтесь с статьей начало [работы с AzCopy](storage-use-azcopy-v10.md) , чтобы скачать AzCopy и узнать о способах предоставления учетных данных авторизации в службе хранилища.

> [!NOTE] 
> В примерах, приведенных в этой статье, предполагается, что вы указали учетные данные авторизации с помощью Azure Active Directory (Azure AD).
>
> Если вы предпочитаете использовать маркер SAS для авторизации доступа к данным большого двоичного объекта, можно добавить этот маркер к URL-адресу ресурса в каждой команде AzCopy. Например, `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="create-a-container"></a>Создание контейнера

Для создания контейнера можно использовать команду [azcopy make](storage-ref-azcopy-make.md) .

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy make 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>'` |
| **Пример** | `azcopy make 'https://mystorageaccount.blob.core.windows.net/mycontainer'` |
| **Пример** (иерархическое пространство имен) | `azcopy make 'https://mystorageaccount.dfs.core.windows.net/mycontainer'` |

Подробную справочную документацию см. в разделе [azcopy make](storage-ref-azcopy-make.md).

## <a name="upload-a-file"></a>Отправка файла

Отправьте файл с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) .

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'` |
| **Пример** | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt'` |

Можно также передать файл, используя подстановочный знак (*) в любом месте пути к файлу или имени файла. Например: `'C:\myDirectory\*.txt'` или `C:\my*\*.txt` .

## <a name="upload-a-directory"></a>Отправка каталога

Отправьте каталог с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) . 

В этом примере каталог (и все файлы в этом каталоге) копируется в контейнер больших двоичных объектов. Результатом является каталог в контейнере с тем же именем.

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --recursive` |
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --recursive` |

Чтобы скопировать в каталог в контейнере, просто укажите имя этого каталога в командной строке.

|    |     |
|--------|-----------|
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' --recursive` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' --recursive` |

Если указать имя каталога, который не существует в контейнере, AzCopy создает новый каталог с этим именем.

## <a name="upload-directory-contents"></a>Отправить содержимое каталога

Отправьте содержимое каталога с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) . Используйте символ-шаблон (*) для отправки содержимого без копирования самого содержащего его каталога.

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>'` |
| **Пример** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory'` |


Добавьте `--recursive` флаг для отправки файлов во все подкаталоги.

## <a name="upload-specific-files"></a>Отправка конкретных файлов

Вы можете передать определенные файлы с помощью полных имен файлов, частичных имен с подстановочными знаками (*) или с помощью дат и времени.

> [!TIP]
> В этих примерах аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

### <a name="specify-multiple-complete-file-names"></a>Указание нескольких полных имен файлов

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-path` параметром. Отдельные имена файлов разделяются точкой с запятой ( `;` ).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-path <semicolon-separated-file-list>` |
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-path 'photos;documents\myFile.txt' --recursive` |

В этом примере AzCopy передает `C:\myDirectory\photos` каталог и `C:\myDirectory\documents\myFile.txt` файл. Включите `--recursive` параметр для перемещения всех файлов в `C:\myDirectory\photos` каталоге.

Кроме того, можно исключить файлы с помощью `--exclude-path` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

### <a name="use-wildcard-characters"></a>Использовать подстановочные знаки

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-pattern` параметром. Укажите частичные имена, которые содержат подстановочные знаки. Разделяйте имена с помощью семиколин ( `;` ). 

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.dfs.core.windows.net/mycontainer' --include-pattern 'myFile*.txt;*.pdf*'` |

Кроме того, можно исключить файлы с помощью `--exclude-pattern` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

`--include-pattern`Параметры и `--exclude-pattern` применяются только к именам файлов, а не к пути.  Если необходимо скопировать все текстовые файлы, существующие в дереве каталогов, используйте `–recursive` параметр для получения всего дерева каталогов, а затем используйте `–include-pattern` и укажите, `*.txt` чтобы получить все текстовые файлы.

### <a name="upload-files-that-were-modified-before-or-after-a-date-and-time"></a>Передача файлов, которые были изменены до или после даты и времени 

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-before` `--include-after` параметром или. Укажите дату и время в формате ISO-8601 (например: `2020-08-19T15:04:00Z` ). 

В следующих примерах передаются файлы, которые были изменены или после указанной даты.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>'  --include-after <Date-Time-in-ISO-8601-format>` |
| **Пример** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory'  --include-after '2020-08-19T15:04:00Z'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory'   --include-after '2020-08-19T15:04:00Z'` |

Подробные справочные сведения см. в документации по [azcopy Copy](storage-ref-azcopy-copy.md) .

## <a name="upload-with-index-tags"></a>Отправка с помощью тегов индекса

Вы можете передать файл и добавить [теги индекса BLOB-объектов (Предварительная версия)](../blobs/storage-manage-find-blobs.md) в целевой большой двоичный объект.  

Если вы используете авторизацию Azure AD, участнику безопасности должна быть назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) , или ему должно быть предоставлено разрешение на `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` [операцию поставщика ресурсов Azure](../../role-based-access-control/resource-provider-operations.md#microsoftstorage) через настраиваемую роль Azure. Если вы используете маркер подписанного URL-адрес (SAS), этот маркер должен предоставить доступ к тегам большого двоичного объекта через `t` разрешение SAS.

Чтобы добавить теги, используйте `--blob-tags` параметр вместе с парой "ключ-значение" в кодировке URL. Например, чтобы добавить ключ `my tag` и значение `my tag value` , необходимо добавить `--blob-tags='my%20tag=my%20tag%20value'` в параметр Destination. 

Разделите несколько тегов индекса с помощью амперсанда ( `&` ).  Например, если нужно добавить ключ `my second tag` и значение `my second tag value` , то полная строка параметра будет иметь вид `--blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` .

В следующих примерах показано, как использовать `--blob-tags` параметр.

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Передача файла** | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |
| **Отправка каталога** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'`|
| **Отправить содержимое каталога** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' --blob-tags='my%20tag=my%20tag%20value&my%20second%20tag=my%20second%20tag%20value'` |

> [!NOTE]
> Если указать каталог для источника, то все большие двоичные объекты, скопированные в назначение, будут иметь те же теги, что указаны в команде.

## <a name="upload-with-optional-flags"></a>Отправка с необязательными флагами

Вы можете настроить операцию отправки с помощью необязательных флагов. Вот несколько примеров.

|Сценарий|Флаг|
|---|---|
|Отправьте файлы в виде добавочных или страничных BLOB-объектов.|**--BLOB-тип** = \[ BlockBlob \| PageBlob \| аппендблоб\]|
|Отправьте файлы на конкретный уровень хранилища (например, на уровень архива).|**--Block-BLOB-уровень** = \[ Нет \| горячий \| стильный \| Архив\]|

Полный список см. в разделе [Параметры](storage-ref-azcopy-copy.md#options).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры см. в следующих статьях:

- [Примеры: Скачивание](storage-use-azcopy-blobs-download.md)
- [Примеры: Копирование между учетными записями](storage-use-azcopy-blobs-copy.md)
- [Примеры: Synchronize](storage-use-azcopy-blobs-synchronize.md)
- [Примеры: Контейнеры Amazon S3](storage-use-azcopy-s3.md)
- [Примеры: службы файлов Azure](storage-use-azcopy-files.md)
- [Руководство. Перенос локальных данных в облачное хранилище с помощью AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)
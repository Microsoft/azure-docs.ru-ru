---
title: Скачивание больших двоичных объектов из хранилища BLOB-объектов Azure с помощью AzCopy V10 | Документация Майкрософт
description: Эта статья содержит коллекцию примеров AzCopy, которые помогут вам скачать большие двоичные объекты из хранилища BLOB-объектов Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/11/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 20da05399eed4cb9c5a4b69a82b0b1e799997751
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98880124"
---
# <a name="download-blobs-from-azure-blob-storage-by-using-azcopy-v10"></a>Скачивание больших двоичных объектов из хранилища BLOB-объектов Azure с помощью AzCopy V10

Вы можете скачать большие двоичные объекты и каталоги из хранилища BLOB-объектов с помощью служебной программы командной строки AzCopy V10. 

Чтобы просмотреть примеры других типов задач, таких как отправка файлов, синхронизация с хранилищем BLOB-объектов или копирование больших двоичных объектов между учетными записями, см. ссылки, представленные в разделе [дальнейшие действия](#next-steps) этой статьи.

## <a name="get-started"></a>Начало работы

Ознакомьтесь с статьей начало [работы с AzCopy](storage-use-azcopy-v10.md) , чтобы скачать AzCopy и узнать о способах предоставления учетных данных авторизации в службе хранилища.

> [!NOTE] 
> В примерах, приведенных в этой статье, предполагается, что вы указали учетные данные авторизации с помощью Azure Active Directory (Azure AD).
>
> Если вы предпочитаете использовать маркер SAS для авторизации доступа к данным большого двоичного объекта, можно добавить этот маркер к URL-адресу ресурса в каждой команде AzCopy. Например, `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="download-a-blob"></a>Загрузка больших двоичных объектов

Скачайте большой двоичный объект с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) .

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'` |
| **Пример** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |

> [!NOTE]
> Если `Content-md5` значение свойства большого двоичного объекта содержит хэш, AzCopy вычисляет хэш MD5 для скачанных данных и проверяет, соответствует ли хэш MD5, хранящийся в свойстве большого двоичного объекта `Content-md5` , соответствующему вычисляемому хэшу. Если эти значения не совпадают, загрузка завершается ошибкой, если это поведение не переопределено путем добавления `--check-md5=NoCheck` или `--check-md5=LogOnly` к команде Copy.

## <a name="download-a-directory"></a>Загрузка каталога

Скачайте каталог с помощью команды [azcopy Copy](storage-ref-azcopy-copy.md) .

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>' '<local-directory-path>' --recursive` |
| **Пример** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |

В этом примере создается каталог с именем `C:\myDirectory\myBlobDirectory` , который содержит все скачанные большие двоичные объекты.

## <a name="download-directory-contents"></a>Загрузить содержимое каталога

Вы можете скачать содержимое каталога, не копируя сам каталог, с помощью подстановочного знака (*).

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

> [!NOTE]
> В настоящее время этот сценарий поддерживается только для учетных записей, у которых нет иерархического пространства имен.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.blob.core.windows.net/<container-name>/*' '<local-directory-path>/'` |
| **Пример** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory/*' 'C:\myDirectory'` |

Добавьте `--recursive` флаг для скачивания файлов во всех подкаталогах.

## <a name="download-specific-blobs"></a>Скачивание конкретных больших двоичных объектов

Вы можете загрузить определенные большие двоичные объекты с помощью полных имен файлов, частичных имен с подстановочными знаками (*) или с помощью дат и времени. 

> [!TIP]
> В этих примерах аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

#### <a name="specify-multiple-complete-blob-names"></a>Указание нескольких полных имен больших двоичных объектов

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-path` параметром. Разделяйте отдельные имена больших двоичных объектов с помощью семиколин ( `;` ).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>'  --include-path <semicolon-separated-file-list>` |
| **Пример** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt' --recursive` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt'--recursive` |

В этом примере AzCopy передает `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos` каталог и `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/documents/myFile.txt` файл. Включите `--recursive` параметр для перемещения всех больших двоичных объектов в `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos` каталоге.

Кроме того, можно исключить большие двоичные объекты с помощью `--exclude-path` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

#### <a name="use-wildcard-characters"></a>Использовать подстановочные знаки

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-pattern` параметром. Укажите частичные имена, которые содержат подстановочные знаки. Разделяйте имена с помощью семиколин ( `;` ).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Пример** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |

Кроме того, можно исключить большие двоичные объекты с помощью `--exclude-pattern` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

`--include-pattern`Параметры и `--exclude-pattern` применяются только к именам больших двоичных объектов, а не к пути.  Если нужно скопировать все текстовые файлы (BLOB-объекты), которые существуют в дереве каталогов, используйте `–recursive` параметр, чтобы получить полное дерево каталогов, а затем используйте `–include-pattern` и укажите, `*.txt` чтобы получить все текстовые файлы.

#### <a name="download-blobs-that-were-modified-before-or-after-a-date-and-time"></a>Скачивание больших двоичных объектов, которые были изменены до или после даты и времени 

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-before` `--include-after` параметром или. Укажите дату и время в формате ISO-8601 (например: `2020-08-19T15:04:00Z` ). 

В следующих примерах загружаются файлы, которые были изменены или после указанной даты.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>/*' '<local-directory-path>' --include-after <Date-Time-in-ISO-8601-format>` |
| **Пример** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |

Подробные справочные сведения см. в документации по [azcopy Copy](storage-ref-azcopy-copy.md) .

#### <a name="download-previous-versions-of-a-blob"></a>Скачивание предыдущих версий большого двоичного объекта

Если вы включили [Управление версиями BLOB-объектов](../blobs/versioning-enable.md), вы можете скачать одну или несколько предыдущих версий большого двоичного объекта. 

Сначала создайте текстовый файл, содержащий список [идентификаторов версий](../blobs/versioning-overview.md). Каждый идентификатор версии должен находиться в отдельной строке. Пример: 

```
2020-08-17T05:50:34.2199403Z
2020-08-17T05:50:34.5041365Z
2020-08-17T05:50:36.7607103Z
```

Затем используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--list-of-versions` параметром. Укажите расположение текстового файла, содержащего список версий (например: `D:\\list-of-versions.txt` ).  

#### <a name="download-a-blob-snapshot"></a>Скачивание моментального снимка большого двоичного объекта

[Моментальный снимок BLOB-объекта](../blobs/snapshots-overview.md) можно скачать, ссылаясь на значение **DateTime** моментального снимка большого двоичного объекта. 

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>?sharesnapshot=<DateTime-of-snapshot>' '<local-file-path>'` |
| **Пример** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'` |
| **Пример** (иерархическое пространство имен) | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'` |

> [!NOTE]
> Если вы используете маркер SAS для авторизации доступа к данным большого двоичного объекта, добавьте моментальный снимок **DateTime** после маркера SAS. Например, `'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z'`.

## <a name="download-with-optional-flags"></a>Загрузка с дополнительными флагами

Вы можете настроить операцию загрузки с помощью необязательных флагов. Вот несколько примеров.

|Сценарий|Флаг|
|---|---|
|Автоматическое распаковка файлов.|**--Распаковка**|
|Укажите, насколько подробными должны быть записи журнала, связанные с копированием.|**--уровень** = \[ ведения журнала ПРЕДУПРЕЖДЕНИЕ \| об ошибке \| \| нет\]|
|Укажите, следует ли перезаписывать конфликтующие файлы и большие двоичные объекты в месте назначения.|**--overwrite** = \[ истинное \| ложный \| \| запрос ифсаурценевер\]|

Полный список см. в разделе [Параметры](storage-ref-azcopy-copy.md#options).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры см. в следующих статьях:

- [Примеры: Передать](storage-use-azcopy-blobs-upload.md)
- [Примеры: копирование между учетной записью](storage-use-azcopy-blobs-copy.md)
- [Примеры: Synchronize](storage-use-azcopy-blobs-synchronize.md)
- [Примеры: Контейнеры Amazon S3](storage-use-azcopy-s3.md)
- [Примеры: службы файлов Azure](storage-use-azcopy-files.md)
- [Руководство. Перенос локальных данных в облачное хранилище с помощью AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)
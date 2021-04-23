---
title: Перенос данных в службу файлов Azure или из нее с помощью AzCopy V10 | Документация Майкрософт
description: Перенос данных с помощью AzCopy и хранилища файлов. AzCopy — это программа командной строки для копирования больших двоичных объектов или файлов в учетную запись хранения или из нее. Используйте AzCopy с файлами Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/08/2020
ms.author: normesta
ms.subservice: common
ms.openlocfilehash: 2cca8a93330e5ddd965d27532895ed1d6702c123
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102179652"
---
# <a name="transfer-data-with-azcopy-and-file-storage"></a>Перенос данных с помощью AzCopy и хранилища файлов 

AzCopy — это служебная программа командной строки, которую можно использовать для копирования файлов в учетную запись хранения или из нее. Эта статья содержит примеры команд, которые работают с файлами Azure.

Прежде чем начать, ознакомьтесь со статьей начало [работы с AzCopy](storage-use-azcopy-v10.md) , чтобы скачать AzCopy и ознакомиться с этим инструментом.

> [!TIP]
> В примерах в этой статье аргументы пути заключаются в одинарные кавычки (''). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ('').

## <a name="create-file-shares"></a>Создание общих папок

Для создания общей папки можно использовать команду [azcopy make](storage-ref-azcopy-make.md) . В примере в этом разделе создается общая папка с именем `myfileshare` .

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy make 'https://<storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>'` |
| **Пример** | `azcopy make 'https://mystorageaccount.file.core.windows.net/myfileshare?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D'` |

Подробную справочную документацию см. в разделе [azcopy make](storage-ref-azcopy-make.md).

## <a name="upload-files"></a>Отправка файлов

Для отправки файлов и каталогов с локального компьютера можно использовать команду [azcopy Copy](storage-ref-azcopy-copy.md) .

Этот раздел содержит следующие примеры.

> [!div class="checklist"]
> * Отправка файла
> * Отправка каталога
> * Отправка содержимого каталога
> * Отправить указанный файл

> [!TIP]
> Вы можете настроить операцию отправки с помощью необязательных флагов. Вот несколько примеров.
>
> |Сценарий|Флаг|
> |---|---|
> |Скопируйте списки управления доступом (ACL) вместе с файлами.|**--Preserve-SMB-разрешения** = \[ true \| false\]|
> |Скопируйте сведения о свойстве SMB вместе с файлами.|**--Preserve-SMB-info** = \[ true \| false\]|
> 
> Полный список см. в разделе [Параметры](storage-ref-azcopy-copy.md#options).

> [!NOTE]
> AzCopy не вычисляет и не сохраняет хэш-код MD5 для файла автоматически. Если вы хотите, чтобы AzCopy это сделать, добавьте `--put-md5` флаг к каждой команде копирования. Таким образом, при скачивании файла AzCopy вычисляет хэш MD5 для скачанных данных и проверяет, соответствует ли хэш MD5, хранящийся в `Content-md5` свойстве файла, вычисляемому хэшу.

### <a name="upload-a-file"></a>Отправка файла

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-file-path>' 'https://<storage-account-name>.file.core.windows.net/<file-share-name>/<file-name><SAS-token>'` |
| **Пример** | `azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.file.core.windows.net/myfileshare/myTextFile.txt?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D'` |

Можно также передать файл, используя подстановочный знак (*) в любом месте пути к файлу или имени файла. Например: `'C:\myDirectory\*.txt'` или `C:\my*\*.txt` .

### <a name="upload-a-directory"></a>Отправка каталога

В этом примере каталог (и все файлы в этом каталоге) копируется в общую папку. В результате каталог будет находиться в общей папке с тем же именем.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>' --recursive` |
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.file.core.windows.net/myfileshare?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D' --recursive` |

Чтобы скопировать в каталог в общей папке, просто укажите имя этого каталога в командной строке.

|    |     |
|--------|-----------|
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.file.core.windows.net/myfileshare/myFileShareDirectory?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D' --recursive` |

Если указать имя каталога, который не существует в общей папке, AzCopy создает новый каталог с этим именем.

### <a name="upload-the-contents-of-a-directory"></a>Отправка содержимого каталога

Вы можете отправить содержимое каталога, не копируя сам каталог с помощью подстановочного знака (*).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>/*' 'https://<storage-account-name>.file.core.windows.net/<file-share-name>/<directory-path><SAS-token>` |
| **Пример** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.file.core.windows.net/myfileshare/myFileShareDirectory?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D"` |

> [!NOTE]
> Добавьте `--recursive` флаг для отправки файлов во все подкаталоги.

### <a name="upload-specific-files"></a>Отправка конкретных файлов

Вы можете передать определенные файлы с помощью полных имен файлов, частичных имен с подстановочными знаками (*) или с помощью дат и времени.

#### <a name="specify-multiple-complete-file-names"></a>Указание нескольких полных имен файлов

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-path` параметром. Отдельные имена файлов разделяются точкой с запятой ( `;` ).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.file.core.windows.net/<file-share-or-directory-name><SAS-token>' --include-path <semicolon-separated-file-list>` |
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.file.core.windows.net/myfileshare?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --include-path 'photos;documents\myFile.txt'` |

В этом примере AzCopy передает `C:\myDirectory\photos` каталог и `C:\myDirectory\documents\myFile.txt` файл. Необходимо включить `--recursive` параметр для перемещения всех файлов в `C:\myDirectory\photos` каталоге.

Кроме того, можно исключить файлы с помощью `--exclude-path` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

#### <a name="use-wildcard-characters"></a>Использовать подстановочные знаки

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-pattern` параметром. Укажите частичные имена, которые содержат подстановочные знаки. Разделяйте имена с помощью точки с запятой ( `;` ).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>' 'https://<storage-account-name>.file.core.windows.net/<file-share-or-directory-name><SAS-token>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Пример** | `azcopy copy 'C:\myDirectory' 'https://mystorageaccount.file.core.windows.net/myfileshare?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --include-pattern 'myFile*.txt;*.pdf*'` |

Кроме того, можно исключить файлы с помощью `--exclude-pattern` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

`--include-pattern`Параметры и `--exclude-pattern` применяются только к именам файлов, а не к пути.  Если необходимо скопировать все текстовые файлы, существующие в дереве каталогов, используйте `--recursive` параметр для получения всего дерева каталогов, а затем используйте `--include-pattern` и укажите, `*.txt` чтобы получить все текстовые файлы.

#### <a name="upload-files-that-were-modified-after-a-date-and-time"></a>Передача файлов, которые были изменены после даты и времени 

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-after` параметром. Укажите дату и время в формате ISO 8601 (например: `2020-08-19T15:04:00Z` ). 

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy '<local-directory-path>\*' 'https://<storage-account-name>.file.core.windows.net/<file-share-or-directory-name><SAS-token>'  --include-after <Date-Time-in-ISO-8601-format>` |
| **Пример** | `azcopy copy 'C:\myDirectory\*' 'https://mystorageaccount.file.core.windows.net/myfileshare?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --include-after '2020-08-19T15:04:00Z'` |

Подробные справочные сведения см. в документации по [azcopy Copy](storage-ref-azcopy-copy.md) .

## <a name="download-files"></a>Загрузка файлов

Для загрузки файлов, каталогов и файловых ресурсов на локальный компьютер можно использовать команду [azcopy Copy](storage-ref-azcopy-copy.md) .

Этот раздел содержит следующие примеры.

> [!div class="checklist"]
> * Загрузка файла
> * Загрузка каталога
> * Загрузка содержимого каталога
> * Загрузка конкретных файлов

> [!TIP]
> Вы можете настроить операцию загрузки с помощью необязательных флагов. Вот несколько примеров.
>
> |Сценарий|Флаг|
> |---|---|
> |Скопируйте списки управления доступом (ACL) вместе с файлами.|**--Preserve-SMB-разрешения** = \[ true \| false\]|
> |Скопируйте сведения о свойстве SMB вместе с файлами.|**--Preserve-SMB-info** = \[ true \| false\]|
> |Автоматическое распаковка файлов.|**--Распаковка**|
> 
> Полный список см. в разделе [Параметры](storage-ref-azcopy-copy.md#options).

> [!NOTE]
> Если `Content-md5` значение свойства файла содержит хэш, AzCopy вычисляет хэш MD5 для скачанных данных и проверяет, соответствует ли хэш MD5, хранящийся в `Content-md5` свойстве файла, вычисляемому хэшу. Если эти значения не совпадают, загрузка завершается ошибкой, если это поведение не переопределено путем добавления `--check-md5=NoCheck` или `--check-md5=LogOnly` к команде Copy.

### <a name="download-a-file"></a>Загрузка файла

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.file.core.windows.net/<file-share-name>/<file-path><SAS-token>' '<local-file-path>'` |
| **Пример** | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myfileshare/myTextFile.txt?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D' 'C:\myDirectory\myTextFile.txt'` |

### <a name="download-a-directory"></a>Загрузка каталога

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.file.core.windows.net/<file-share-name>/<directory-path><SAS-token>' '<local-directory-path>' --recursive` |
| **Пример** | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myfileshare/myFileShareDirectory?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D' 'C:\myDirectory'  --recursive` |

В этом примере создается каталог с именем `C:\myDirectory\myFileShareDirectory` , который содержит все скачанные файлы.

### <a name="download-the-contents-of-a-directory"></a>Загрузка содержимого каталога

Вы можете скачать содержимое каталога, не копируя сам каталог, с помощью подстановочного знака (*).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.file.core.windows.net/<file-share-name>/*<SAS-token>' '<local-directory-path>/'` |
| **Пример** | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myfileshare/myFileShareDirectory/*?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D' 'C:\myDirectory'` |

> [!NOTE]
> Добавьте `--recursive` флаг для скачивания файлов во всех подкаталогах.

### <a name="download-specific-files"></a>Загрузка конкретных файлов

Вы можете загрузить определенные файлы с помощью полных имен файлов, частичных имен с подстановочными знаками (*) или с помощью дат и времени.

#### <a name="specify-multiple-complete-file-names"></a>Указание нескольких полных имен файлов

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-path` параметром. Отдельные имена файлов разделяются точкой с запятой ( `;` ).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.file.core.windows.net/<file-share-or-directory-name><SAS-token>' '<local-directory-path>'  --include-path <semicolon-separated-file-list>` |
| **Пример** | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myFileShare/myDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt' --recursive` |

В этом примере AzCopy передает `https://mystorageaccount.file.core.windows.net/myFileShare/myDirectory/photos` каталог и `https://mystorageaccount.file.core.windows.net/myFileShare/myDirectory/documents/myFile.txt` файл. Включите `--recursive` параметр для перемещения всех файлов в `https://mystorageaccount.file.core.windows.net/myFileShare/myDirectory/photos` каталоге.

Кроме того, можно исключить файлы с помощью `--exclude-path` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

#### <a name="use-wildcard-characters"></a>Использовать подстановочные знаки

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-pattern` параметром. Укажите частичные имена, которые содержат подстановочные знаки. Разделяйте имена с помощью точки с запятой ( `;` ).

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.file.core.windows.net/<file-share-or-directory-name><SAS-token>' '<local-directory-path>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **Пример** | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myfileshare/myDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |

Кроме того, можно исключить файлы с помощью `--exclude-pattern` параметра. Дополнительные сведения см. в статье [azcopy Copy](storage-ref-azcopy-copy.md) Справочник по документам.

`--include-pattern`Параметры и `--exclude-pattern` применяются только к именам файлов, а не к пути.  Если необходимо скопировать все текстовые файлы, существующие в дереве каталогов, используйте `--recursive` параметр для получения всего дерева каталогов, а затем используйте `--include-pattern` и укажите, `*.txt` чтобы получить все текстовые файлы.

#### <a name="download-files-that-were-modified-after-a-date-and-time"></a>Скачать файлы, которые были изменены после даты и времени 

Используйте команду [azcopy Copy](storage-ref-azcopy-copy.md) с `--include-after` параметром. Укажите дату и время в формате ISO-8601 (например: `2020-08-19T15:04:00Z` ). 

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.file.core.windows.net/<file-share-or-directory-name>/*<SAS-token>' '<local-directory-path>'  --include-after <Date-Time-in-ISO-8601-format>` |
| **Пример** | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myfileshare/*?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'C:\myDirectory' --include-after '2020-08-19T15:04:00Z'` |


Подробные справочные сведения см. в документации по [azcopy Copy](storage-ref-azcopy-copy.md) .

#### <a name="download-from-a-share-snapshot"></a>Скачивание из моментального снимка общего ресурса

Вы можете скачать определенную версию файла или каталога, ссылаясь на значение **DateTime** моментального снимка общего ресурса. Дополнительные сведения о моментальных снимках общих ресурсов см. в статье [Общие сведения о моментальных снимках общих ресурсов для службы файлов Azure](../files/storage-snapshots-files.md). 

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<storage-account-name>.file.core.windows.net/<file-share-name>/<file-path-or-directory-name><SAS-token>&sharesnapshot=<DateTime-of-snapshot>' '<local-file-or-directory-path>'` |
| **Пример** (Загрузка файла) | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myfileshare/myTextFile.txt?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'` |
| **Пример** (Загрузка каталога) | `azcopy copy 'https://mystorageaccount.file.core.windows.net/myfileshare/myFileShareDirectory?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory'  --recursive`|


## <a name="copy-files-between-storage-accounts"></a>Копирование файлов между учетными записями хранения

AzCopy можно использовать для копирования файлов в другие учетные записи хранения. Операция копирования является синхронной, поэтому когда команда возвращает результат, это означает, что все файлы скопированы.

AzCopy использует [API](/rest/api/storageservices/put-page-from-url)" [сервер-сервер](/rest/api/storageservices/put-block-from-url) ", поэтому данные копируются непосредственно между серверами хранилища. Эти операции копирования не используют пропускную способность сети компьютера. Вы можете увеличить пропускную способность этих операций, задав значение `AZCOPY_CONCURRENCY_VALUE` переменной среды. Дополнительные сведения см. в разделе [Оптимизация пропускной способности](storage-use-azcopy-configure.md#optimize-throughput).

Также можно копировать определенные версии файлов, ссылаясь на значение **DateTime** моментального снимка общего ресурса. Дополнительные сведения о моментальных снимках общих ресурсов см. в статье [Общие сведения о моментальных снимках общих ресурсов для службы файлов Azure](../files/storage-snapshots-files.md). 

Этот раздел содержит следующие примеры.

> [!div class="checklist"]
> * Копирование файла в другую учетную запись хранения
> * Копирование каталога в другую учетную запись хранения
> * Копирование общей папки в другую учетную запись хранения
> * Копирование всех общих папок, каталогов и файлов в другую учетную запись хранения

> [!TIP]
> Вы можете настроить операцию копирования с помощью необязательных флагов. Вот несколько примеров.
>
> |Сценарий|Флаг|
> |---|---|
> |Скопируйте списки управления доступом (ACL) вместе с файлами.|**--Preserve-SMB-разрешения** = \[ true \| false\]|
> |Скопируйте сведения о свойстве SMB вместе с файлами.|**--Preserve-SMB-info** = \[ true \| false\]|
> 
> Полный список см. в разделе [Параметры](storage-ref-azcopy-copy.md#options).

### <a name="copy-a-file-to-another-storage-account"></a>Копирование файла в другую учетную запись хранения

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.file.core.windows.net/<file-share-name>/<file-path><SAS-token>' 'https://<destination-storage-account-name>.file.core.windows.net/<file-share-name>/<file-path><SAS-token>'` |
| **Пример** | `azcopy copy 'https://mysourceaccount.file.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.file.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D'` |
| **Пример** (моментальный снимок общего ресурса) | `azcopy copy 'https://mysourceaccount.file.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z' 'https://mydestinationaccount.file.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D'` |

### <a name="copy-a-directory-to-another-storage-account"></a>Копирование каталога в другую учетную запись хранения

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.file.core.windows.net/<file-share-name>/<directory-path><SAS-token>' 'https://<destination-storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>' --recursive` |
| **Пример** | `azcopy copy 'https://mysourceaccount.file.core.windows.net/myFileShare/myFileDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.file.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |
| **Пример** (моментальный снимок общего ресурса) | `azcopy copy 'https://mysourceaccount.file.core.windows.net/myFileShare/myFileDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z' 'https://mydestinationaccount.file.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |


### <a name="copy-a-file-share-to-another-storage-account"></a>Копирование общей папки в другую учетную запись хранения

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>' 'https://<destination-storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>' --recursive` |
| **Пример** | `azcopy copy 'https://mysourceaccount.file.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.file.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |
| **Пример** (моментальный снимок общего ресурса) | `azcopy copy 'https://mysourceaccount.file.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z' 'https://mydestinationaccount.file.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |


### <a name="copy-all-file-shares-directories-and-files-to-another-storage-account"></a>Копирование всех общих папок, каталогов и файлов в другую учетную запись хранения

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy copy 'https://<source-storage-account-name>.file.core.windows.net/<SAS-token>' 'https://<destination-storage-account-name>.file.core.windows.net/<SAS-token>' --recursive'` |
| **Пример** | `azcopy copy 'https://mysourceaccount.file.core.windows.net?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.file.core.windows.net?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |
| **Пример** (моментальный снимок общего ресурса) | `azcopy copy 'https://mysourceaccount.file.core.windows.net?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z' 'https://mydestinationaccount.file.core.windows.net?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |


## <a name="synchronize-files"></a>Синхронизировать файлы

Вы можете синхронизировать содержимое файлового ресурса с другой общей папкой. Кроме того, можно синхронизировать содержимое каталога в общей папке с содержимым каталога, расположенного в другой общей папке. Синхронизация является одним из способов. Иными словами, вы выбираете, какие из этих двух конечных точек являются источником, и какой из них является назначением. Синхронизация также использует API сервера-сервера.

> [!NOTE]
> В настоящее время этот сценарий поддерживается только для учетных записей, у которых нет иерархического пространства имен. Текущий выпуск AzCopy не синхронизируется между службой файлов Azure и хранилищем BLOB-объектов.

Команда [Sync](storage-ref-azcopy-sync.md) сравнивает имена файлов и метки времени последнего изменения. Установите `--delete-destination` необязательный флаг в значение `true` или, `prompt` чтобы удалить файлы в целевом каталоге, если эти файлы больше не существуют в исходном каталоге.

Если установить `--delete-destination` для флага значение `true` , AzCopy удаляет файлы без указания запроса. Если требуется, чтобы запрос отображался перед тем, как AzCopy удалит файл, установите `--delete-destination` для флага значение `prompt` .

> [!TIP]
> Вы можете настроить операцию синхронизации с помощью необязательных флагов. Вот несколько примеров.
>
> |Сценарий|Флаг|
> |---|---|
> |Скопируйте списки управления доступом (ACL) вместе с файлами.|**--Preserve-SMB-разрешения** = \[ true \| false\]|
> |Скопируйте сведения о свойстве SMB вместе с файлами.|**--Preserve-SMB-info** = \[ true \| false\]|
> |Исключение файлов на основе шаблона.|**--Exclude-путь**|
> |Укажите, насколько подробными должны быть записи журнала, связанные с синхронизацией.|**--уровень** = \[ ведения журнала ПРЕДУПРЕЖДЕНИЕ \| об ошибке \| \| нет\]|
> 
> Полный список см. в разделе [Параметры](storage-ref-azcopy-sync.md#options).

### <a name="update-a-file-share-with-changes-to-another-file-share"></a>Обновление общей папки с изменениями в другой общей папке

Первый файловый ресурс, отображаемый в этой команде, является источником. Второй — назначение.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy sync 'https://<source-storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>' 'https://<destination-storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>' --recursive` |
| **Пример** | `azcopy sync 'https://mysourceaccount.file.core.windows.net/myfileShare?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.file.core.windows.net/myfileshare?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |

### <a name="update-a-directory-with-changes-to-a-directory-in-another-file-share"></a>Обновление каталога с изменениями в каталоге в другой общей папке

Первым каталогом, который отображается в этой команде, является источник. Второй — назначение.

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy sync 'https://<source-storage-account-name>.file.core.windows.net/<file-share-name>/<directory-name><SAS-token>' 'https://<destination-storage-account-name>.file.core.windows.net/<file-share-name>/<directory-name><SAS-token>' --recursive` |
| **Пример** | `azcopy sync 'https://mysourceaccount.file.core.windows.net/myFileShare/myDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' 'https://mydestinationaccount.file.core.windows.net/myFileShare/myDirectory?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |

### <a name="update-a-file-share-to-match-the-contents-of-a-share-snapshot"></a>Обновление общей папки в соответствии с содержимым моментального снимка общего ресурса

Первый файловый ресурс, отображаемый в этой команде, является источником. В конце URI добавьте строку, `&sharesnapshot=` за которой следует значение **DateTime** моментального снимка. 

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy sync 'https://<source-storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>&sharesnapsot<snapshot-ID>' 'https://<destination-storage-account-name>.file.core.windows.net/<file-share-name><SAS-token>' --recursive` |
| **Пример** | `azcopy sync 'https://mysourceaccount.file.core.windows.net/myfileShare?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D&sharesnapshot=2020-03-03T20%3A24%3A13.0000000Z' 'https://mydestinationaccount.file.core.windows.net/myfileshare?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-07-04T05:30:08Z&st=2019-07-03T21:30:08Z&spr=https&sig=CAfhgnc9gdGktvB=ska7bAiqIddM845yiyFwdMH481QA8%3D' --recursive` |

Дополнительные сведения о моментальных снимках общих ресурсов см. в статье [Обзор моментальных снимков общих ресурсов для службы файлов Azure](../files/storage-snapshots-files.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры приведены в любой из следующих статей:

- [Начало работы с AzCopy](storage-use-azcopy-v10.md)

- [Передача данных](storage-use-azcopy-v10.md#transfer-data)

- [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)

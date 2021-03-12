---
title: Синхронизация с хранилищем BLOB-объектов Azure с помощью AzCopy V10 | Документация Майкрософт
description: Эта статья содержит набор AzCopy примеров команд, помогающих синхронизироваться с хранилищем BLOB-объектов Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/08/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: ab1da88899ba2b90e303da107631e3878b3a8b58
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102635881"
---
# <a name="synchronize-with-azure-blob-storage-by-using-azcopy-v10"></a>Синхронизация с хранилищем BLOB-объектов Azure с помощью AzCopy V10

Вы можете синхронизировать локальное хранилище с хранилищем BLOB-объектов Azure с помощью служебной программы командной строки AzCopy V10. 

Вы можете синхронизировать содержимое локальной файловой системы с контейнером больших двоичных объектов. Кроме того, можно синхронизировать контейнеры и виртуальные каталоги друг с другом. Синхронизация является одним из способов. Иными словами, вы выбираете, какие из этих двух конечных точек являются источником, и какой из них является назначением. Синхронизация также использует API сервера-сервера. Примеры, приведенные в этом разделе, также работают с учетными записями, имеющими иерархическое пространство имен. 

> [!NOTE]
> Текущий выпуск AzCopy не синхронизируется между другими источниками и назначениями (например, хранилище файлов или контейнеры Amazon Web Services (AWS) S3).

Для просмотра примеров других типов задач, таких как отправка файлов, Загрузка больших двоичных объектов или копирование больших двоичных объектов между учетными записями, см. ссылки, представленные в разделе [дальнейшие действия](#next-steps) этой статьи.

## <a name="get-started"></a>Начало работы

Ознакомьтесь с статьей начало [работы с AzCopy](storage-use-azcopy-v10.md) , чтобы скачать AzCopy и узнать о способах предоставления учетных данных авторизации в службе хранилища.

> [!NOTE] 
> В примерах, приведенных в этой статье, предполагается, что вы указали учетные данные авторизации с помощью Azure Active Directory (Azure AD).
>
> Если вы предпочитаете использовать маркер SAS для авторизации доступа к данным большого двоичного объекта, можно добавить этот маркер к URL-адресу ресурса в каждой команде AzCopy. Например: `'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`.

## <a name="guidelines"></a>Рекомендации

- Команда [Sync](storage-ref-azcopy-sync.md) сравнивает имена файлов и метки времени последнего изменения. Установите `--delete-destination` необязательный флаг в значение `true` или, `prompt` чтобы удалить файлы в целевом каталоге, если эти файлы больше не существуют в исходном каталоге.

- Если установить `--delete-destination` для флага значение `true` , AzCopy удаляет файлы без указания запроса. Если требуется, чтобы запрос отображался перед тем, как AzCopy удалит файл, установите `--delete-destination` для флага значение `prompt` .

- Чтобы предотвратить случайное удаление, обязательно включите функцию [обратимого удаления](../blobs/soft-delete-blob-overview.md) , прежде чем использовать `--delete-destination=prompt|true` флаг.

## <a name="update-a-container-with-changes-to-a-local-file-system"></a>Обновление контейнера с изменениями в локальной файловой системе

В этом случае контейнер является назначением, а локальная файловая система — источником. 

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy sync '<local-directory-path>' 'https://<storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Пример** | `azcopy sync 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive` |

## <a name="update-a-local-file-system-with-changes-to-a-container"></a>Обновление локальной файловой системы с изменениями в контейнере

В этом случае локальная файловая система является назначением, а контейнер — источником.

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy sync 'https://<storage-account-name>.blob.core.windows.net/<container-name>' 'C:\myDirectory' --recursive` |
| **Пример** | `azcopy sync 'https://mystorageaccount.blob.core.windows.net/mycontainer' 'C:\myDirectory' --recursive` |

## <a name="update-a-container-with-changes-in-another-container"></a>Обновление контейнера с изменениями в другом контейнере

Первый контейнер, который отображается в этой команде, является источником. Второй — назначение.

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>' --recursive` |
| **Пример** | `azcopy sync 'https://mysourceaccount.blob.core.windows.net/mycontainer' 'https://mydestinationaccount.blob.core.windows.net/mycontainer' --recursive` |

## <a name="update-a-directory-with-changes-to-a-directory-in-another-container"></a>Обновление каталога с изменениями в каталоге в другом контейнере

Первым каталогом, который отображается в этой команде, является источник. Второй — назначение.

> [!TIP]
> В этом примере аргументы пути заключаются в одинарные кавычки (' '). Используйте одинарные кавычки во всех командных оболочках, кроме командной оболочки Windows (cmd.exe). Если вы используете командную оболочку Windows (cmd.exe), заключите аргументы пути в двойные кавычки ("") вместо одинарных кавычек ("").

|    |     |
|--------|-----------|
| **Синтаксис** | `azcopy sync 'https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' 'https://<destination-storage-account-name>.blob.core.windows.net/<container-name>/<directory-name>' --recursive` |
| **Пример** | `azcopy sync 'https://mysourceaccount.blob.core.windows.net/<container-name>/myDirectory' 'https://mydestinationaccount.blob.core.windows.net/mycontainer/myDirectory' --recursive` |

## <a name="synchronize-with-optional-flags"></a>Синхронизация с необязательными флагами

Вы можете настроить операцию синхронизации с помощью необязательных флагов. Вот несколько примеров.

|Сценарий|Flag|
|---|---|
|Укажите, как следует проверять хэши с строгими MD5 при скачивании.|**--Check-MD5** = \[ Не проверять \| Вход \| фаилифдифферент \| фаилифдифферентормиссинг\]|
|Исключение файлов на основе шаблона.|**--Exclude-путь**|
|Укажите, насколько подробными должны быть записи журнала, связанные с синхронизацией.|**--уровень** = \[ ведения журнала ПРЕДУПРЕЖДЕНИЕ \| об ошибке \| \| нет\]|

Полный список см. в разделе [Параметры](storage-ref-azcopy-sync.md#options).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры см. в следующих статьях:

- [Примеры: Передать](storage-use-azcopy-blobs-upload.md)
- [Примеры: Скачивание](storage-use-azcopy-blobs-download.md)
- [Примеры: Копирование между учетными записями](storage-use-azcopy-blobs-copy.md)
- [Примеры: Контейнеры Amazon S3](storage-use-azcopy-s3.md)
- [Примеры: службы файлов Azure](storage-use-azcopy-files.md)
- [Руководство. Перенос локальных данных в облачное хранилище с помощью AzCopy](storage-use-azcopy-migrate-on-premises-data.md)
- [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)
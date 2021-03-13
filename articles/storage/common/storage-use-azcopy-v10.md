---
title: Копирование или перемещение данных в службу хранилища Azure с помощью AzCopy V10 | Документация Майкрософт
description: AzCopy — это служебная программа командной строки, которую можно использовать для копирования данных в учетные записи хранения, из или между ними. С помощью сведений из этой статьи вы сможете скачать AzCopy, подключиться к учетной записи хранения, а затем передавать файлы.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/08/2020
ms.author: normesta
ms.subservice: common
ms.custom: contperf-fy21q2
ms.openlocfilehash: a1651ef1b49ecedb8070f32c56e9d23a0f8fd0da
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103201590"
---
# <a name="get-started-with-azcopy"></a>Начало работы с AzCopy

AzCopy — это служебная программа командной строки, которую можно использовать для копирования больших двоичных объектов или файлов в учетную запись хранения или из нее. С помощью сведений из этой статьи вы сможете скачать AzCopy, подключиться к учетной записи хранения, а затем передавать файлы.

> [!NOTE]
> AzCopy **V10** — это Текущая поддерживаемая версия AzCopy.
>
> Если вам нужно использовать предыдущую версию AzCopy, см. раздел [Использование предыдущей версии AzCopy](#previous-version) этой статьи.

<a id="download-and-install-azcopy"></a>

## <a name="download-azcopy"></a>Скачивание AzCopy

Сначала скачайте исполняемый файл AzCopy V10 в любой каталог на компьютере. AzCopy V10 — это просто исполняемый файл, поэтому установка не требуется.

- [Windows 64-bit](https://aka.ms/downloadazcopy-v10-windows) (ZIP)
- [Windows 32-bit](https://aka.ms/downloadazcopy-v10-windows-32bit) (ZIP)
- [Linux x86-64](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [macOS](https://aka.ms/downloadazcopy-v10-mac) (ZIP)

Эти файлы сжимаются как ZIP-файл (Windows и Mac) или tar-файл (Linux). Чтобы скачать и распаковать TAR файл в Linux, см. документацию по дистрибутиву Linux.

> [!NOTE]
> Если вы хотите копировать данные в службу [хранилища таблиц Azure](../tables/table-storage-overview.md) и из нее, установите [AzCopy версии 7,3](https://aka.ms/downloadazcopynet).


## <a name="run-azcopy"></a>Запуск AzCopy

Для удобства рекомендуется добавить расположение каталога исполняемого файла AzCopy в системный путь для простоты использования. Таким образом можно ввести `azcopy` любой каталог в системе.

Если вы решили не добавлять каталог AzCopy в путь, необходимо будет изменить каталоги на расположение исполняемого файла AzCopy и ввести `azcopy` или `.\azcopy` в командной строки Windows PowerShell.

Чтобы просмотреть список команд, введите и нажмите `azcopy -h` клавишу ВВОД.

Чтобы узнать об определенной команде, просто включите имя команды (например: `azcopy list -h` ).

> [!div class="mx-imgBorder"]
> ![Встроенная справка](media/storage-use-azcopy-v10/azcopy-inline-help.png)


Подробную справочную документацию по каждой команде и параметру команды см. в разделе [azcopy](storage-ref-azcopy.md)

> [!NOTE] 
> Как владелец учетной записи хранения Azure, вы не назначаете разрешения на доступ к данным автоматически. Прежде чем выполнять какие-либо осмысленные действия с AzCopy, необходимо решить, как вы будете предоставлять учетные данные авторизации для службы хранилища. 

<a id="choose-how-youll-provide-authorization-credentials"></a>

## <a name="authorize-azcopy"></a>Авторизовать AzCopy

Учетные данные авторизации можно указать с помощью Azure Active Directory (AD) или с помощью маркера подписанного URL-адрес (SAS).

Используйте эту таблицу в качестве рекомендации:

| Тип хранилища | Текущий поддерживаемый метод авторизации |
|--|--|
|**Хранилище BLOB-объектов** | Azure AD и SAS |
|**Хранилище BLOB-объектов (иерархическое пространство имен)** | Azure AD и SAS |
|**Хранилище файлов** | Только SAS |

#### <a name="option-1-use-azure-active-directory"></a>Вариант 1. Использование Azure Active Directory

Этот параметр доступен только для хранилища BLOB-объектов. С помощью Azure Active Directory можно указать учетные данные один раз вместо того, чтобы добавлять маркер SAS к каждой команде.  

> [!NOTE]
> В текущем выпуске, если вы планируете копировать большие двоичные объекты между учетными записями хранения, необходимо добавить маркер SAS к каждому исходному URL-адресу. Маркер SAS можно опустить только из URL-адреса назначения. Примеры см. в разделе [копирование больших двоичных объектов между учетными записями хранения](#transfer-data).

Чтобы авторизовать доступ с помощью Azure AD, см. статью [авторизация доступа к BLOB-объектам с помощью AzCopy и Azure Active Directory (Azure AD)](storage-use-azcopy-authorize-azure-active-directory.md).

#### <a name="option-2-use-a-sas-token"></a>Вариант 2. Использование токена SAS

Маркер SAS можно добавить к каждому URL-адресу источника или назначения, который используется в командах AzCopy.

В этом примере команда рекурсивно копирует данные из локального каталога в контейнер больших двоичных объектов. Вымышленный маркер SAS добавляется в конец URL-адреса контейнера.

```azcopy
azcopy copy "C:\local\path" "https://account.blob.core.windows.net/mycontainer1/?sv=2018-03-28&ss=bjqt&srt=sco&sp=rwddgcup&se=2019-05-01T05:01:17Z&st=2019-04-30T21:01:17Z&spr=https&sig=MGCXiyEzbtttkr3ewJIh2AR8KrghSy1DGM9ovN734bQF4%3D" --recursive=true
```

Дополнительные сведения о маркерах SAS и их получении см. в разделе [использование подписанных URL-адресов (SAS)](./storage-sas-overview.md).

<a id="transfer-data"></a>

## <a name="transfer-data"></a>Передача данных

После авторизации удостоверения или получения маркера SAS можно начать передачу данных.

> [!NOTE]
> Параметр " [требуется безопасное перемещение](storage-require-secure-transfer.md) " учетной записи хранения определяет, защищено ли подключение к учетной записи хранения с помощью протокола TLS. Эта настройка включена по умолчанию.   

Примеры команд см. в любой из этих статей.

| Служба | Статья |
|--------|-----------|
|хранилище BLOB-объектов Azure |[Отправка файлов в хранилище BLOB-объектов Azure](storage-use-azcopy-blobs-upload.md)<br><br>[Скачивание больших двоичных объектов из хранилища BLOB-объектов Azure](storage-use-azcopy-blobs-download.md)<br><br>[Копирование больших двоичных объектов между учетными записями хранения Azure](storage-use-azcopy-blobs-copy.md)<br><br>[Синхронизация с хранилищем BLOB-объектов Azure](storage-use-azcopy-blobs-synchronize.md)|
|Файлы Azure |[Перенос данных с помощью AzCopy и хранилища файлов](storage-use-azcopy-files.md)|
|Amazon S3|[Передача данных с помощью AzCopy и контейнеров Amazon S3](storage-use-azcopy-s3.md)|
|Хранилище Azure Stack|[Перенос данных с помощью AzCopy и хранилища Azure Stack](/azure-stack/user/azure-stack-storage-transfer#azcopy)|

## <a name="use-in-a-script"></a>Использование в скрипте

#### <a name="obtain-a-static-download-link"></a>Получение статической ссылки для скачивания

Со временем [ссылка для скачивания](#download-and-install-azcopy) AzCopy будет указывать на новые версии AzCopy. Если сценарий скачивает AzCopy, скрипт может перестать работать, если более новая версия AzCopy изменяет функции, от которых зависит сценарий.

Чтобы избежать этих проблем, получите статическую (неизменную) ссылку на текущую версию AzCopy. Таким образом, ваш сценарий скачивает одну и ту же версию AzCopy при каждом запуске.

Чтобы получить ссылку, выполните следующую команду:

| Операционная система  | Команда |
|--------|-----------|
| **Linux** | `curl -s -D- https://aka.ms/downloadazcopy-v10-linux | grep ^Location` |
| **Windows** | `(curl https://aka.ms/downloadazcopy-v10-windows -MaximumRedirection 0 -ErrorAction silentlycontinue).headers.location` |

> [!NOTE]
> Для Linux `--strip-components=1` в `tar` команде удаляется папка верхнего уровня, содержащая имя версии, а вместо этого двоичный файл извлекается непосредственно в текущую папку. Это позволяет обновлять скрипт с новой версией `azcopy` , обновляя только `wget` URL-адрес.

URL-адрес отображается в выходных данных этой команды. Затем скрипт может скачать AzCopy с помощью этого URL-адреса.

| Операционная система  | Команда |
|--------|-----------|
| **Linux** | `wget -O azcopy_v10.tar.gz https://aka.ms/downloadazcopy-v10-linux && tar -xf azcopy_v10.tar.gz --strip-components=1` |
| **Windows** | `Invoke-WebRequest https://azcopyvnext.azureedge.net/release20190517/azcopy_windows_amd64_10.1.2.zip -OutFile azcopyv10.zip <<Unzip here>>` |

#### <a name="escape-special-characters-in-sas-tokens"></a>Экранирование специальных символов в маркерах SAS

В пакетных файлах с `.cmd` расширением необходимо экранировать `%` символы, отображаемые в маркерах SAS. Это можно сделать, добавив дополнительный `%` символ рядом с существующими `%` символами в строке токена SAS.

#### <a name="run-scripts-by-using-jenkins"></a>Выполнение скриптов с помощью Jenkins

Если вы планируете использовать [Jenkins](https://jenkins.io/) для выполнения скриптов, убедитесь, что в начале скрипта размещена следующая команда.

```
/usr/bin/keyctl new_session
```

## <a name="use-in-azure-storage-explorer"></a>Использование в Обозреватель службы хранилища Azure

[Обозреватель службы хранилища](https://azure.microsoft.com/features/storage-explorer/) использует AzCopy для выполнения всех операций по ее переносу данных. Вы можете использовать [Обозреватель службы хранилища](https://azure.microsoft.com/features/storage-explorer/) , если хотите использовать преимущества производительности AzCopy, но вместо командной строки вы предпочитаете работать с файлами с помощью графического пользовательского интерфейса.

Обозреватель службы хранилища использует ключ учетной записи для выполнения операций, поэтому после входа в Обозреватель службы хранилища вам не потребуется предоставлять дополнительные учетные данные для авторизации.

<a id="previous-version"></a>

## <a name="configure-optimize-and-fix"></a>Настройка, оптимизация и исправление

См. раздел [Настройка, оптимизация и устранение неполадок AzCopy](storage-use-azcopy-configure.md)

## <a name="use-a-previous-version"></a>Использовать предыдущую версию

Если вам нужно использовать предыдущую версию AzCopy, см. одну из следующих ссылок:

- [AzCopy в Windows (версия 8)](/previous-versions/azure/storage/storage-use-azcopy)

- [AzCopy в Linux (версии 7)](/previous-versions/azure/storage/storage-use-azcopy-linux)

## <a name="next-steps"></a>Дальнейшие действия

Если у вас есть вопросы, проблемы или общие отзывы, отправьте их [на страницу GitHub](https://github.com/Azure/azure-storage-azcopy) .

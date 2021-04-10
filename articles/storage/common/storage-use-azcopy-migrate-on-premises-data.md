---
title: Руководство по Миграция локальных данных в службу хранилища Azure с помощью AzCopy | Документация Майкрософт
description: В этом руководстве вы используете AzCopy для перемещения или копирования данных в большие двоичные объекты, таблицы, файлы и обратно. Вы можете легко перенести данные из локального хранилища в службу хранилища Azure.
author: normesta
ms.service: storage
ms.topic: tutorial
ms.date: 05/14/2019
ms.author: normesta
ms.reviewer: seguler
ms.subservice: common
ms.openlocfilehash: 682394329205d74859c1af0c0a68a37539da7872
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98881033"
---
#  <a name="tutorial-migrate-on-premises-data-to-cloud-storage-with-azcopy"></a>Руководство по Перенос локальных данных в облачное хранилище с помощью AzCopy

Средство командной строки AzCopy принимает простые команды, позволяющие копировать данные в хранилища BLOB-объектов Azure, файлов Azure и таблиц Azure, а также из этих хранилищ. Команды рассчитаны на оптимальную производительность. С помощью AzCopy вы можете перемещать данные между файловой системой и учетной записью хранения или между разными учетными записями хранения. AzCopy может использоваться для копирования локальных данных в учетную запись хранения.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создайте учетную запись хранения. 
> * использование AzCopy для отправки всех данных;
> * изменение данных для целей тестирования;
> * создание запланированных заданий или задач cron для поиска новых файлов для отправки.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить инструкции из этого руководства, скачайте последнюю версию AzCopy. См. подробнее о [начале работы с AzCopy](storage-use-azcopy-v10.md).

Если вы работаете в Windows, вам потребуется средство [Schtasks](/windows/win32/taskschd/schtasks), так как в этом руководстве оно используется для планирования задач. Пользователи Linux вместо этого будут использовать команду crontab.

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

## <a name="create-a-container"></a>Создание контейнера

Первый шаг — создать контейнер, потому что большие двоичные объекты должны всегда загружаться в контейнер. Контейнеры выступают средством упорядочения групп больших двоичных объектов подобно тому, как папки используются для упорядочения файлов на компьютере.

Чтобы создать контейнер, сделайте следующее.

1. Нажмите кнопку **Учетные записи хранения** на главной странице и выберите созданную учетную запись хранилища.
2. Выберите **Большие двоичные объекты** в разделе **Службы**, а затем выберите **Контейнер**.

   ![Снимок экрана, на котором показано создание контейнера](media/storage-azcopy-migrate-on-premises-data/CreateContainer.png)
 
Имя контейнера должно начинаться с буквы или цифры. Эти имена могут содержать только буквы, цифры и знак дефиса (-). Дополнительные сведения об именовании больших двоичных объектов и контейнеров см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

## <a name="download-azcopy"></a>Скачивание AzCopy

Скачайте исполняемый файл AzCopy версии 10.

- [Windows](https://aka.ms/downloadazcopy-v10-windows) (ZIP-файл)
- [Linux](https://aka.ms/downloadazcopy-v10-linux) (TAR-файл)
- [MacOS](https://aka.ms/downloadazcopy-v10-mac) (ZIP-файл)

Поместите файл AzCopy в любое расположение на компьютере. Добавьте расположение файла в системную переменную пути, чтобы можно было ссылаться на этот исполняемый файл из любой папки на компьютере.

## <a name="authenticate-with-azure-ad"></a>Аутентификация с помощью Azure AD

Сначала назначьте своему удостоверению роль [участника для данных больших двоичных объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-queue-data-contributor). См. статью о [назначении роли Azure для доступа к большим двоичным объектам и данным очереди с помощью портала Azure](./storage-auth-aad-rbac-portal.md).

Затем откройте командную строку, введите следующую команду и нажмите клавишу ВВОД.

```azcopy
azcopy login
```

Эта команда возвращает код проверки подлинности и URL-адрес веб-сайта. Откройте веб-сайт, укажите код и нажмите кнопку **Далее**.

![Снимок экрана, на котором показан запрос на вход](media/storage-use-azcopy-v10/azcopy-login.png)

Откроется окно входа. В этом окне войдите в свою учетную запись Azure с помощью соответствующих данных. Выполнив вход, можно закрыть окно браузера и начать работу с AzCopy.

## <a name="upload-contents-of-a-folder-to-blob-storage"></a>Отправка содержимого папки в хранилище BLOB-объектов

С помощью AzCopy вы можете передать все файлы из определенной папки в [Windows](./storage-use-azcopy-v10.md) или [Linux](./storage-use-azcopy-v10.md) в хранилище больших двоичных объектов. Чтобы передать все большие двоичные объекты, содержащиеся в папке, введите следующую команду AzCopy:

```AzCopy
azcopy copy "<local-folder-path>" "https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>" --recursive=true
```

* Замените заполнитель `<local-folder-path>` путем к папке, которая содержит файлы (например, `C:\myFolder` или `/mnt/myFolder`).

* Замените заполнитель `<storage-account-name>` именем вашей учетной записи хранения.

* Замените заполнитель `<container-name>` именем созданного контейнера.

Чтобы рекурсивно передать все содержимое определенного каталога в хранилище BLOB-объектов, укажите параметр `--recursive`. При запуске с этим параметром AzCopy передает все вложенные папки и файлы.

## <a name="upload-modified-files-to-blob-storage"></a>Отправка измененных файлов в хранилище BLOB-объектов

Вы можете использовать AzCopy для передачи файлов с учетом времени их последнего изменения. 

Чтобы проверить этот режим, измените или создайте несколько тестовых файлов в исходном каталоге. Затем выполните команду AzCopy `sync`.

```AzCopy
azcopy sync "<local-folder-path>" "https://<storage-account-name>.blob.core.windows.net/<container-name>" --recursive=true
```

* Замените заполнитель `<local-folder-path>` путем к папке, которая содержит файлы (например, `C:\myFolder` или `/mnt/myFolder`).

* Замените заполнитель `<storage-account-name>` именем вашей учетной записи хранения.

* Замените заполнитель `<container-name>` именем созданного контейнера.

См. подробнее о команде `sync` в руководстве по [синхронизации файлов](./storage-use-azcopy-v10.md#transfer-data).

## <a name="create-a-scheduled-task"></a>Создание запланированной задачи

Вы можете создать запланированную задачу или задание cron, которые выполняют скрипт с командой AzCopy. Этот скрипт обнаруживает и отправляет в облачное хранилище обновленные локальные данные через определенные промежутки времени.

Скопируйте команду AzCopy в текстовый редактор. Замените значения параметров в команде AzCopy подходящими значениями. Сохраните файл AzCopy с именем `script.sh` (для Linux) или `script.bat` (для Windows). 

В этих примерах используется имя папки `myFolder`, имя учетной записи хранения `mystorageaccount` и имя контейнера `mycontainer`.

> [!NOTE]
> В примере Linux добавляется маркер SAS. Вам потребуется указать его в команде. Текущая версия AzCopy V10 не поддерживает авторизацию Azure AD в заданиях Cron.

# <a name="linux"></a>[Linux](#tab/linux)

```bash
azcopy sync "/mnt/myfiles" "https://mystorageaccount.blob.core.windows.net/mycontainer?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-05-30T06:57:40Z&st=2019-05-29T22:57:40Z&spr=https&sig=BXHippZxxx54hQn%2F4tBY%2BE2JHGCTRv52445rtoyqgFBUo%3D" --recursive=true
```

# <a name="windows"></a>[Windows](#tab/windows)

```bash
azcopy sync "C:\myFolder" "https://mystorageaccount.blob.core.windows.net/mycontainer" --recursive=true
```

---

В этом руководстве мы применим [Schtasks](/windows/win32/taskschd/schtasks) для создания в Windows запланированной задачи. Команда [Crontab](http://crontab.org/) позволяет создать задание cron в Linux.

 **Schtasks** позволяет администратору создавать, удалять, просматривать, изменять, выполнять или завершать запланированные задачи на локальном или удаленном компьютере. **Cron** в Linux и Unix дает пользователям возможность выполнять команды или скрипты в определенное время определенного дня с помощью [выражений cron](https://en.wikipedia.org/wiki/Cron#CRON_expression).

# <a name="linux"></a>[Linux](#tab/linux)

Чтобы создать в Linux задание cron, введите в терминале следующую команду:

```bash
crontab -e
*/5 * * * * sh /path/to/script.sh
```

Если в команде указано выражение cron `*/5 * * * *`, то скрипт оболочки `script.sh` будет выполняться каждые пять минут. Вы можете назначить выполнение скрипта на определенное время ежедневно, ежемесячно или ежегодно. Дополнительные сведения о настройке даты и времени для выполнения заданий вы найдете в документации по [выражениям cron](https://en.wikipedia.org/wiki/Cron#CRON_expression).

# <a name="windows"></a>[Windows](#tab/windows)

Чтобы создать в Windows назначенную задачу, введите в командной строке или в PowerShell следующую команду:

В этом примере предполагается, что скрипт размещен в корневом диске компьютера, но его можно разместить в любом удобном расположении.

```cmd
schtasks /CREATE /SC minute /MO 5 /TN "AzCopy Script" /TR C:\script.bat
```

Эта команда использует следующие параметры:
- `/SC` позволяет указать расписание в минутах;
- `/MO` обозначает интервал длительностью пять минут;
- `/TN` позволяет указать имя задачи;
- `/TR` позволяет указать путь к файлу `script.bat`.

Дополнительные сведения о создании запланированной задачи в Windows см. в документации по [Schtasks](/previous-versions/orphan-topics/ws.10/cc772785(v=ws.10)#BKMK_minutes).

---

Чтобы проверить правильность работы запланированных задач и (или) заданий cron, создайте в каталоге `myFolder` новые файлы. Подождите пять минут, чтобы новые файлы успели переместиться в учетную запись хранения. Перейдите в каталог журнала и проверьте содержимое журналов для запланированной задачи или задания cron.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о способах перемещения данных из локальной среды в службу хранилища Azure и обратно см. по этой ссылке:

* [Перемещение данных в службу хранилища Azure и обратно](./storage-choose-data-transfer-solution.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)  

См. подробнее об AzCopy:

* [Get started with AzCopy](storage-use-azcopy-v10.md) (Начало работы с AzCopy)

* [Transfer data with AzCopy and blob storage](./storage-use-azcopy-v10.md#transfer-data) (Передача данных с помощью AzCopy и хранилища BLOB-объектов)

* [Transfer data with AzCopy and file storage](storage-use-azcopy-files.md) (Передача данных с помощью AzCopy и хранилища файлов)

* [Передача данных с помощью AzCopy и контейнеров Amazon S3](storage-use-azcopy-s3.md)
 
* [Configure, optimize, and troubleshoot AzCopy](storage-use-azcopy-configure.md) (Настройка, оптимизация и устранение неполадок с AzCopy)

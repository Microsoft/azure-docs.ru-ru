---
title: Доступ к ресурсам с помощью средств Data Lake
description: Узнайте, как использовать средства Azure Data Lake для доступа к ресурсам Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 02/09/2018
ms.openlocfilehash: d04f108b45070b27c4ff9ed833e8fb77b74cd597
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96754756"
---
# <a name="accessing-resources-with-azure-data-lake-tools"></a>Доступ к ресурсам с помощью средств Azure Data Lake

Вы можете легко получить доступ к Azure Data Lake Analytics ресурсам с помощью команд или действий средств обработки данных Azure в VS Code.

## <a name="integrate-with-azure-data-lake-analytics-through-a-command"></a>Интеграция с Azure Data Lake Analytics с помощью команды

Вы можете использовать ресурсы Azure Data Lake Analytics, чтобы отобразить учетные записи, получить доступ к метаданным и просмотреть задания аналитики.

### <a name="to-list-the-azure-data-lake-analytics-accounts-under-your-azure-subscription"></a>Получение списка учетных записей Azure Data Lake Analytics в подписке Azure

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите команду **ADL: List Accounts**. Учетные записи отображаются в области **Вывода**.

### <a name="to-access-azure-data-lake-analytics-metadata"></a>Доступ к метаданным Azure Data Lake Analytics

1. Нажмите клавиши CTRL+SHIFT+P, а затем введите **ADL: List Tables**.
2. Выберите одну из учетных записей Data Lake Analytics.
3. Выберите одну из баз данных Data Lake Analytics.
4. Выберите одну из схем. Можно просмотреть список таблиц.

### <a name="to-view-azure-data-lake-analytics-jobs"></a>Просмотр заданий Azure Data Lake Analytics

1. Откройте палитру команд (CTRL+SHIFT+P) и выберите **ADL: Show Jobs**.
2. Выберите Data Lake Analytics или локальную учетную запись.
3. Подождите, пока появится список заданий для учетной записи.
4. Выберите задание из списка. В правой области средств Data Lake откроется представление задания, а в выходных данных VS Code будут показаны некоторые сведения.

    ![Список заданий](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-show-job.png)

## <a name="integrate-with-azure-data-lake-store-through-a-command"></a>Интеграция с Azure Data Lake Store при помощи команды

С помощью команд для Azure Data Lake Store можно выполнять следующие действия:

- [Просмотр ресурсов Azure Data Lake Store](#list-the-storage-path)
- [Просмотр файла Azure Data Lake Store](#preview-the-storage-file)
- Отправка файла непосредственно в Azure Data Lake Store в VS Code
- Скачивание файла непосредственно из Azure Data Lake Store в VS Code

### <a name="list-the-storage-path"></a>Вывод пути к хранилищу

### <a name="to-list-the-storage-path-through-the-command-palette"></a>Вывод пути к хранилищу через палитру команд

1. Щелкните редактор сценариев правой кнопкой мыши и выберите **ADL: List Path** (ADL: путь к списку).
2. Выберите папку из списка либо выберите **Enter a path** или **Browse from root path**. В этом примере выбран вариант **Enter a path**.
3. Выберите учетную запись Data Lake Analytics.
4. Перейдите к папке хранилища или укажите путь к ней (например, /output/).  

В палитре команд выводятся сведения о пути на основе ваших записей.

![Результаты из пути к хранилищу](./media/data-lake-analytics-data-lake-tools-for-vscode/list-storage-path.png)

Более удобный способ вывода относительного пути — через контекстное меню.

### <a name="to-list-the-storage-path-through-the-shortcut-menu"></a>Вывод пути к хранилищу через контекстное меню

Щелкните строку пути правой кнопкой мыши и выберите **List Path**.

![Пункт "List Path" в контекстном меню](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-right-click-path.png)

### <a name="preview-the-storage-file"></a>Предварительный просмотр файла хранилища

1. Щелкните редактор сценариев правой кнопкой мыши и выберите **ADL: Preview File** (ADL: предварительный просмотр файла).
2. Выберите учетную запись Data Lake Analytics.
3. Введите путь к файлу службы хранилища Azure (например, /output/SearchLog.txt).

В VS Code откроется файл.

![Шаги и результат просмотра файла хранилища](./media/data-lake-analytics-data-lake-tools-for-vscode/preview-storage-file.png)

Просмотреть файл также можно через контекстное меню полного или относительного пути к файлу в редакторе скриптов.

### <a name="upload-a-file-or-folder"></a>Отправка файла или папки

1. Щелкните редактор сценариев правой кнопкой мыши и выберите **Upload File** (Отправить файл) или **Upload Folder** (Отправить папку).
2. Выберите один или несколько файлов (если выбрана команда **Upload File**) или целую папку (если выбрана команда **Upload Folder**). Затем нажмите кнопку **Отправить**.
3. Выберите папку хранилища из списка либо выберите **Enter a path** или **Browse from root path**. В этом примере выбран вариант **Enter a path**.
4. Выберите учетную запись Data Lake Analytics.
5. Перейдите к папке хранилища или укажите путь к ней (например, /output/).
6. Нажмите **Выбрать текущую папку**, чтобы указать цель отправки.

![Действия и результаты отправки файла или папки](./media/data-lake-analytics-data-lake-tools-for-vscode/upload-file.png)

Отправить файлы в хранилище также можно через контекстное меню полного или относительного пути к файлу в редакторе скриптов.

Вы можете [отслеживать состояние отправки](#check-storage-tasks-status).

### <a name="download-a-file"></a>Загрузка файла

Скачать файл можно с помощью команды **ADL: Download File** или **ADL: Download File (Advanced)**.

### <a name="to-download-a-file-through-the-adl-download-file-advanced-command"></a>Скачивание файла с помощью команды ADL: Download File (Advanced)

1. Щелкните правой кнопкой мыши редактор скриптов, а затем выберите **Download File (Advanced)** (Скачать файл (Дополнительно)).
2. В VS Code отобразится JSON-файл. Вы можете ввести несколько путей к файлам, чтобы загрузить несколько файлов одновременно. Инструкции отображаются в окне **вывод** . Чтобы продолжить скачивание файлов, сохраните (CTRL+S) JSON-файл.

    ![JSON-файл с путями для скачивания файлов](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-files.png)

В окне **Вывод** отобразится состояние скачивания файла.

![Окно вывода с состоянием скачивания](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-file-result.png)

Вы можете [отслеживать состояние скачивания](#check-storage-tasks-status).

### <a name="to-download-a-file-through-the-adl-download-file-command"></a>Скачивание файла с помощью команды ADL: Download File

1. Щелкните редактор сценариев правой кнопкой мыши, выберите **Download File**, а затем выберите целевую папку в диалоговом окне **Выбор папки**.

1. Выберите папку из списка либо выберите **Enter a path** или **Browse from root path**. В этом примере выбран вариант **Enter a path**.

1. Выберите учетную запись Data Lake Analytics.

1. Перейдите к папке хранилища или введите путь к ней (например, /output/) и выберите файл для скачивания.

![Действия и результат скачивания файла](./media/data-lake-analytics-data-lake-tools-for-vscode/download-file.png)

Скачать файлы хранилища также можно через контекстное меню полного или относительного пути к файлу в редакторе скриптов.

Вы можете [отслеживать состояние скачивания](#check-storage-tasks-status).

### <a name="check-storage-tasks-status"></a>Проверка состояния задач хранилища

В строке состояния отображается состояние отправки и скачивания. Выберите строку состояния, и состояние появится на вкладке **ВЫВОД**.

![Строка состояния и выходные данные](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-status.png)

## <a name="integrate-with-azure-data-lake-analytics-from-the-explorer"></a>Интеграция с Azure Data Lake Analytics из обозревателя

После входа все подписки для учетной записи Azure будут перечислены в области слева, в разделе **AZURE DATALAKE**.

![Обозреватель Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer.png)

### <a name="data-lake-analytics-metadata-navigation"></a>Навигация по метаданным Data Lake Analytics

Разверните узел своей подписки Azure. В узле **Базы данных U-SQL** вы можете просматривать свою базу данных U-SQL, в том числе такие папки, как **Схемы**, **Учетные данные**, **Сборки**, **Таблицы** и **Индекс**.

### <a name="data-lake-analytics-metadata-entity-management"></a>Управление объектами метаданных Data Lake Analytics

Разверните узел **Базы данных U-SQL**. Вы можете создать базу данных, схему, таблицу, тип таблицы, индекс или статистику, щелкнув соответствующий узел правой кнопкой мыши и выбрав **Скрипт для создания** в контекстном меню. На открывшейся странице отредактируйте скрипт в соответствии со своими потребностями. Затем отправьте задание, щелкнув его правой кнопкой мыши и выбрав команду **ADL: Submit Job**.

Завершив создание элемента, щелкните узел правой кнопкой мыши и выберите **Обновить**, чтобы элемент появился в списке. Вы также можете удалить элемент, открыв правой кнопкой мыши контекстное меню и выбрав **Удалить**.

![Команда "Скрипт для создания" в контекстном меню обозревателя Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create.png)

![Страница скрипта для нового элемента](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create-snippet.png)

### <a name="data-lake-analytics-assembly-registration"></a>Регистрация сборки Data Lake Analytics

Вы можете зарегистрировать сборку в соответствующей базе данных, щелкнув правой кнопкой мыши узел **Assemblies** и выбрав **Register assembly**.

![Команда "Зарегистрировать сборку" в контекстном меню узла Assemblies](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer-register-assembly.png)

## <a name="integrate-with-azure-data-lake-store-from-the-explorer"></a>Интеграция с Azure Data Lake Store из обозревателя

Перейдите в **Data Lake Store**:

- Вы можете щелкнуть узел папки правой кнопкой мыши, а затем использовать команды **Обновить**, **Удалить**, **Отправить**, **Загрузить папку**, **Копировать относительный путь** и **Копировать полный путь** в контекстном меню.

   ![Команды контекстного меню для узла папки в обозревателе Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-folder-menu.png)

- Вы можете щелкнуть узел файла правой кнопкой мыши, а затем использовать команды **Просмотреть**, **Скачать**, **Удалить**, **Создать скрипт EXTRACT** (доступна только для CSV-, TSV- и TXT-файлов), **Копировать относительный путь** и **Копировать полный путь** в контекстном меню.

   ![Команды контекстного меню для узла файла в обозревателе Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-extract.png)

## <a name="integrate-with-azure-blob-storage-from-the-explorer"></a>Интеграция с хранилищем BLOB-объектов Azure из обозревателя

Перейдите в хранилище BLOB-объектов:

- Вы можете щелкнуть правой кнопкой мыши узел контейнера BLOB-объектов, а затем использовать команды **Refresh**, **Delete Blob Container** и **Upload Blob** в контекстном меню.

   ![Команды контекстного меню для узла контейнера в хранилище BLOB-объектов](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-blob-container-node.png)

- Вы можете щелкнуть правой кнопкой мыши узел папки, а затем использовать команды **Обновить** и **Отправить BLOB-объект** в контекстном меню.

   ![Команды контекстного меню для узла папки в хранилище BLOB-объектов](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-folder-node.png)

- Вы можете щелкнуть узел файла правой кнопкой мыши, а затем использовать команды **Просмотреть/Изменить**, **Скачать**, **Удалить**, **Создать сценарий EXTRACT** (доступна только для CSV-, TSV- и TXT-файлов), **Копировать относительный путь** и **Копировать полный путь** в контекстном меню.

    ![Команды контекстного меню для узла файла в хранилище BLOB-объектов](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu-2.png)

## <a name="open-the-data-lake-explorer-in-the-portal"></a>Открытие обозревателя Data Lake на портале

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите команду **Open Web Azure Storage Explorer** либо щелкните правой кнопкой мыши относительный или полный путь в редакторе скриптов, а затем выберите пункт **Open Web Azure Storage Explorer**.
3. Выберите учетную запись Data Lake Analytics.

В средствах Data Lake откроется путь к хранилищу Azure на портале Azure. Можно найти путь и предварительно просмотреть файл в Интернете.

## <a name="additional-features"></a>Дополнительные функции

Средства Data Lake для VS Code поддерживают следующие функции:

- **Автозаполнение IntelliSense**. Предлагаемые варианты отображаются во всплывающих окнах вокруг ключевых слов, методов и переменных. Разные значки обозначают разные объекты:

  - тип данных Scala;
  - сложный тип данных;
  - встроенные пользовательские типы;
  - коллекции и классы .NET;
  - выражения C#;
  - встроенные функции и объекты C#, определяемые пользователем;
  - функции U-SQL;
  - функции U-SQL для работы с окнами;

    ![типы объектов IntelliSense.](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-objects.png)

- **Автозаполнение IntelliSense на Data Lake Analytics метаданных**. средства Data Lake загружают Data Lake Analytics данные метаданных локально. Функция IntelliSense автоматически заполняет объекты метаданными Data Lake Analytics. К этим объектам относятся база данных, схема, таблица, представление, функция с табличным значением, процедуры и сборки C#.

  ![Метаданные IntelliSense](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-metastore.png)

- **Маркер ошибки IntelliSense**. Средства Data Lake подчеркивают ошибки при редактировании файлов U-SQL и C#.
- **Выделение синтаксиса**. Средства Data Lake используют цвета для выделения переменных, ключевых слов, типов данных и функций.

    ![Синтаксис с различными цветами](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-syntax-highlights.png)

> [!NOTE]
> Рекомендуем обновить средства Azure Data Lake для Visual Studio до версии 2.3.3000.4 или выше. Предыдущие версии являются устаревшими и недоступными для скачивания.  

## <a name="next-steps"></a>Дальнейшие действия

- [Разработка U-SQL с помощью Python, R, и C Sharp для Azure Data Lake Analytics в VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
- [Локальный запуск и локальная отладка U-SQL в Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
- [Начало работы с Azure Data Lake Analytics с помощью портала Azure](data-lake-analytics-get-started-portal.md)
- [Разработка скриптов U-SQL с помощью средств Data Lake для Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
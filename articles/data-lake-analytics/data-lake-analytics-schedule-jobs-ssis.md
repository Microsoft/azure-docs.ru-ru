---
title: Планирование заданий U-SQL Azure Data Lake Analytics с использованием служб MSSQL Integration Services
description: Узнайте, как использовать SQL Server Integration Services для планирования заданий U-SQL с помощью встроенного скрипта или файлов запросов U-SQL.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 07/17/2018
ms.openlocfilehash: b080b433f5af49e970faba02003fb68e21a08365
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92221457"
---
# <a name="schedule-u-sql-jobs-using-sql-server-integration-services-ssis"></a>Планирование заданий U-SQL с использованием служб Integration Services (SSIS)

Этот документ содержит сведения о координации и создании задания U-SQL с помощью службы Integration Service (SSIS). 

## <a name="prerequisites"></a>Предварительные условия

[Пакет дополнительных компонентов Azure для служб Integration Services](/sql/integration-services/azure-feature-pack-for-integration-services-ssis#scenario-managing-data-in-the-cloud) предоставляет [задачу Azure Data Lake Analytics](/sql/integration-services/control-flow/azure-data-lake-analytics-task) и [диспетчер подключений Azure Data Lake Analytics](/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager), что позволяет подключиться к службе Azure Data Lake Analytics. Чтобы использовать эту задачу, убедитесь, что установлено следующее ПО.

- [Скачивание и установка SQL Server Data Tools (SSDT) для Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt)
- [Установите пакет дополнительных компонентов Azure для служб Integration Services (SSIS)](/sql/integration-services/azure-feature-pack-for-integration-services-ssis)

## <a name="azure-data-lake-analytics-task"></a>Задача Azure Data Lake Analytics

Задача Azure Data Lake Analytics дает возможность пользователям отправлять задания U-SQL в учетную запись Azure Data Lake Analytics. 

[Дополнительные сведения о настройке задачи Azure Data Lake Analytics](/sql/integration-services/control-flow/azure-data-lake-analytics-task).

![Задача Azure Data Lake Analytics в службах MSSQL Integration Services](./media/data-lake-analytics-schedule-jobs-ssis/data-lake-analytics-azure-data-lake-analytics-task-in-ssis.png)

Можно получить сценарий U-SQL из разных мест используя встроенные функции и задачи служб MSSQL Integration Services. В приведенных ниже сценариях показано, как настроить сценарии U-SQL для разных пользовательских случаев.

## <a name="scenario-1-use-inline-script-call-tvfs-and-stored-procs"></a>Сценарий 1. Использование встроенного сценария вызова tvfs и хранимой процедуры

В редакторе задач Azure Data Lake Analytics настройте **SourceType** как **DirectInput** и поместите инструкции U-SQL в **USQLStatement**.

Чтобы управление и обслуживание кода были легкими, поместите короткий сценарий U-SQL как встроенные сценарии, например, при вызове существующих таблично определенных функций и хранимых процедур баз данных U-SQL. 

![Изменение встроенного сценария U-SQL в задаче служб MSSQL Integration Services](./media/data-lake-analytics-schedule-jobs-ssis/edit-inline-usql-script-in-ssis.png)

Связанная статья о [способе передачи параметров для хранимых процедур](#scenario-6-pass-parameters-to-u-sql-script)

## <a name="scenario-2-use-u-sql-files-in-azure-data-lake-store"></a>Сценарий 2. Использование файлов U-SQL в хранилище Azure Data Lake Store

Можно также использовать файлы U-SQL в Azure Data Lake Store используя **задачу файловой системы Azure Data Lake Store** в пакете дополнительных компонентов Azure. Такой подход позволяет использовать сценарии, хранимые в облаке.

Выполните следующие действия для настройки подключения задачи файловой системы Azure Data Lake Store к заданию Azure Data Lake Analytics.

### <a name="set-task-control-flow"></a>Настройка задачи потока управления

В представлении конструктора пакетов SSIS добавьте **задачу файловой системы Azure Data Lake Store**, **контейнер "Цикл по каждому элементу"** и **задачу аналитики Azure Data Lake** в контейнере "Цикл по каждому элементу". Задача файловой системы Azure Data Lake Store позволяет загружать файлы во временную папку U-SQL в учетной записи ADLS. Контейнер "Цикл по каждому элементу" и задача Azure Data Lake Analytics помогают отправлять каждый файл U-SQL во временную папку учетной записи Azure Data Lake Analytics как задание U-SQL.

![Схема, на которой показана задача «Azure Data Lake Store файловая система», добавляемая в контейнер «цикл по каждому элементу».](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-data-lake-store.png)

### <a name="configure-azure-data-lake-store-file-system-task"></a>Настройка задачи файловой системы Azure Data Lake Store

1. Установите параметру **Операция** значение **CopyFromADLS**.
2. Настройте **AzureDataLakeConnection**, дополнительные сведения о [диспетчере подключений Azure Data Lake Store](/sql/integration-services/connection-manager/azure-data-lake-store-connection-manager).
3. Установите **AzureDataLakeDirectory**. Укажите папку для хранения сценариев U-SQL. Используйте относительный путь, который связан с корневой папкой учетной записи Azure Data Lake Store.
4. Установите параметр **Назначение** папке, которая кэширует загруженные сценарии U-SQL. Этот путь к папке будет использоваться в контейнере "Цикл по каждому элементу" для отправки задания U-SQL. 

![Настройка задачи файловой системы Azure Data Lake Store](./media/data-lake-analytics-schedule-jobs-ssis/configure-azure-data-lake-store-file-system-task.png)

[Дополнительные сведения о задаче файловой системы Azure Data Lake Store](/sql/integration-services/control-flow/azure-data-lake-store-file-system-task).

### <a name="configure-foreach-loop-container"></a>Настройка контейнера "Цикл по каждому элементу"

1. На странице **Коллекция** установите **Перечислитель** для **Перечислителя с циклом по каждому файлу**.

2. Задайте **Папку** в разделе группы **Настройка перечислителя** для временной папки, которая содержит загруженные скрипты U-SQL.

3. Задайте **Файлы** в разделе **Конфигурация перечислителя** для `*.usql` таким образом, чтобы контейнер цикла получал только файлы с расширением `.usql`.

    ![Снимок экрана, на котором показан редактор циклов по каждому элементу с выбранным параметром "Коллекция" и выделенные разделы конфигурации перечислителя и перечислителя.](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-collection.png)

4. На странице **Сопоставления переменной** добавьте определенную пользователем переменную для получения имени файла для каждого файла U-SQL. Установите значение 0 для параметра **Индекс**, чтобы получить имя файла. В этом примере определяется переменная с именем `User::FileName`. Эта переменная будет использоваться для получения динамического файла подключения скрипта U-SQL и задаст имя задания U-SQL в задаче Azure Data Lake Analytics.

    ![Настройка контейнера "Цикл по каждому элементу" для получения имени файла](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-variable-mapping.png)

### <a name="configure-azure-data-lake-analytics-task"></a>Настройка задачи Azure Data Lake Analytics 

1. Задайте значение **SourceType** для параметра **FileConnection**.

2. Задайте **FileConnection** для подключения файла, который указывает на объекты файла, возвращаемые контейнером "Цикл по каждому элементу".
    
    Создание файла подключения.

   1. Выберите **\<New Connection...>** параметр FileConnection.
   2. Задайте параметру **Тип использования** значение **Существующий файл**, а параметру **Файл** — любой путь к существующему файлу.

       ![Снимок экрана, на котором показан редактор диспетчера подключения файлов с "существующим файлом", выбранным для "тип использования".](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

   3. В представлении **Диспетчер подключений** щелкните правой кнопкой мыши недавно созданное подключение файла и выберите **Свойства**.

   4. В окне **Свойства** разверните вкладку **Выражения** и задайте переменной **ConnectionString** значение, определенное в контейнере "Цикл по каждому элементу", например, `@[User::FileName]`.

       ![Настройка контейнера "Цикл по каждому элементу"](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-property-for-foreach-loop-container.png)

3. Задайте параметру **AzureDataLakeAnalyticsConnection** учетную запись Azure Data Lake Analytics, в которую требуется отправлять задания. Дополнительные сведения о [диспетчере подключений Azure Data Lake Analytics](/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager).

4. Задайте другие конфигурации задания. [Подробнее](/sql/integration-services/control-flow/azure-data-lake-analytics-task)

5. Используйте **Выражения** для динамической настройки имени задания U-SQL.

    1. На странице **Выражения** добавьте новое выражение пары "ключ — значение" для **JobName**.
    2. Установите значение переменной JobName для определения переменной в контейнере "Цикл по каждому элементу", например, `@[User::FileName]`.
    
        ![Настройка выражения служб SSIS для имени задания U-SQL](./media/data-lake-analytics-schedule-jobs-ssis/configure-expression-for-u-sql-job-name.png)

## <a name="scenario-3-use-u-sql-files-in-azure-blob-storage"></a>Сценарий 3. Использование файла U-SQL в хранилище BLOB-объектов Azure

Можно использовать файлы U-SQL в хранилище BLOB-объектов Azure с помощью **задачи загрузки BLOB-объектов Azure** в пакете дополнительных компонентов Azure. Такой подход позволяет использовать сценарии, хранимые в облаке.

Эти шаги похожи на [Сценарий 2. Используйте файлы U-SQL в Azure Data Lake Store](#scenario-2-use-u-sql-files-in-azure-data-lake-store). Измените задачу файловой системы Azure Data Lake Store на задачу скачивания BLOB-объектов Azure. [Дополнительные сведения о задаче загрузки BLOB-объектов Azure](/sql/integration-services/control-flow/azure-blob-download-task).

Поток управления такой как показано ниже.

![Использование файлов U-SQL в хранилище Azure Data Lake Store](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-blob-storage.png)

## <a name="scenario-4-use-u-sql-files-on-the-local-machine"></a>Сценарий 4. Использование файлов U-SQL на локальном компьютере

Помимо использования файлов U-SQL, хранящихся в облаке, можно также использовать файлы на локальном компьютере или файлы, развернутые с помощью пакетов служб SSIS.

1. Щелкните правой кнопкой мыши **Диспетчер подключений** в проекте служб SSIS и выберите **Создать диспетчер подключений**.

2. Выберите тип **Файл** и нажмите кнопку **Добавить...**.

3. Задайте параметру **Тип использования** значение **Существующий файл**, а параметру **Файл** — путь к файлу на локальном компьютере.

    ![Добавление файла подключения к локальному файлу](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

4. Добавьте задачу **Azure Data Lake Analytics** и:
    1. Задайте значение **SourceType** для параметра **FileConnection**.
    2. Задайте параметр **FileConnection** для недавно созданного подключения файла.

5. Завершите другие настройки для задачи Azure Data Lake Analytics.

## <a name="scenario-5-use-u-sql-statement-in-ssis-variable"></a>Сценарий 5. Использование инструкции U-SQL в переменной SSIS

В некоторых случаях может потребоваться динамическое создание инструкций U-SQL. Можно использовать **переменную SSIS** с **выражением SSIS** и другие задачи служб, такие как задача "Сценарий", помогающие динамически создать инструкцию U-SQL.

1. Откройте окно инструментов "Переменные" с помощью меню верхнего уровня **SSIS > Переменные**.

2. Добавьте переменные SSIS и задайте значение напрямую или используйте **Выражение** для создания значения.

3. Добавьте задачу **Azure Data Lake Analytics** и сделайте следующее.
    1. Задайте значение **SourceType** для параметра **Переменная**.
    2. Задайте **SourceVariable** для недавно созданной переменной SSIS.

4. Завершите другие настройки для задачи Azure Data Lake Analytics.

## <a name="scenario-6-pass-parameters-to-u-sql-script"></a>Сценарий 6. Передача параметров сценарию U-SQL

В некоторых случаях может потребоваться динамически задавать значения переменной U-SQL в скрипте U-SQL. Функция **Сопоставление параметров** в задаче Azure Data Lake Analytics поможет в работе с этим сценарием. Существует два случая типичных действий пользователя.

- Задать динамические переменные пути ввода и вывода на основе текущей даты и времени.
- Задать параметр для хранимых процедур.

[Дополнительные сведения о настройке параметров для скрипта U-SQL](/sql/integration-services/control-flow/azure-data-lake-analytics-task#parameter-mapping-page-configuration).

## <a name="next-steps"></a>Дальнейшие действия

- [Запуск пакетов служб Integration Services в Azure](../data-factory/how-to-invoke-ssis-package-ssis-activity.md)
- [Пакет дополнительных компонентов Azure для служб Integration Services (SSIS)](/sql/integration-services/azure-feature-pack-for-integration-services-ssis#scenario-managing-data-in-the-cloud)
- [Преобразование данных с помощью сценариев U-SQL в Azure Data Lake Analytics](../data-factory/transform-data-using-data-lake-analytics.md)
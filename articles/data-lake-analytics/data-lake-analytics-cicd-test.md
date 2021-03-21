---
title: Тестирование кода Azure Data Lake Analytics
description: Сведения о добавлении тестовых случаев в расширенный код C# и U-SQL для Azure Data Lake Analytics.
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 08/30/2019
ms.openlocfilehash: 7310c67ef20a4134d4f613ea969c96802958bf62
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96015236"
---
# <a name="test-your-azure-data-lake-analytics-code"></a>Тестирование кода Azure Data Lake Analytics

Azure Data Lake предоставляет язык [U-SQL](data-lake-analytics-u-sql-get-started.md) . U-SQL сочетает декларативный SQL с императивным C# для обработки данных в любом масштабе. В этом документе вы узнаете, как создавать тестовые случаи для U-SQL и расширенного кода пользовательского оператора C# (UDO).

## <a name="test-u-sql-scripts"></a>Тестирование скриптов U-SQL

Скрипт U-SQL компилируется и оптимизируется для выполнения исполняемого кода в Azure или на локальном компьютере. В процессе компиляции и оптимизации обрабатывается весь сценарий U-SQL. Вы не можете выполнить традиционный модульный тест для каждой инструкции. Но используя тестовый пакет SDK для U-SQL и пакет SDK для запуска в локальной среде, можно выполнить тесты на уровне сценариев.

### <a name="create-test-cases-for-u-sql-script"></a>Создание тестовых случаев для скриптов U-SQL

Средства Azure Data Lake для Visual Studio позволяют создавать тестовые случаи для скриптов U-SQL.

1. В обозревателе решений щелкните правой кнопкой мыши сценарий U-SQL, а затем выберите **Создать модульный тест**.

1. Создайте новый тестовый проект или вставьте тестовый случай в существующий тестовый проект.

   ![Создание конфигурации тестового проекта U-SQL с помощью Средств Data Lake для Visual Studio](./media/data-lake-analytics-cicd-test/data-lake-tools-create-usql-test-project-configure.png)

### <a name="manage-the-test-data-source"></a>Управление источником тестовых данных

При тестировании сценариев U-SQL требуются файлы с тестовыми входными данными. Чтобы управлять тестовыми данными, в **Обозреватель решений** щелкните правой кнопкой мыши проект U-SQL и выберите пункт **свойства**. Источник можно указать в **источнике тестовых данных**.

![Data Lake Tools для Visual Studio — настройка источника тестовых данных проекта](./media/data-lake-analytics-cicd-test/data-lake-tools-configure-project-test-data-source.png)

При вызове `Initialize()` интерфейса в пакете SDK для тестирования U-SQL временная локальная папка данных создается в рабочем каталоге тестового проекта. Все файлы и папки в папке "тестовый источник данных" копируются во временную корневую папку локальных данных перед запуском тестовых случаев скрипта U-SQL. Можно добавить дополнительные папки источника тестовых данных, разделив пути к папкам с тестовыми данными точкой с запятой.

### <a name="manage-the-database-environment-for-testing"></a>Управление средой базы данных для тестирования

Если скрипты U-SQL используют или выполняют запросы к объектам базы данных U-SQL, необходимо инициализировать среду базы данных перед выполнением тестовых случаев U-SQL. Этот подход может потребоваться при вызове хранимых процедур. Интерфейс `Initialize()` в тестовом пакете SDK для U-SQL позволяет развернуть все базы данных, на которые ссылается проект U-SQL, во временную локальную корневую папку данных в рабочей папке тестового проекта.

Дополнительные сведения об управлении ссылками на проекты базы данных U-SQL для проекта U-SQL см. [в разделе ссылка на проект базы данных u-](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project)SQL.

### <a name="verify-test-results"></a>Проверка результатов теста

Интерфейс `Run()` возвращает результат выполнения задания. *0* означает успешное выполнение, а *1* означает сбой. Чтобы проверить выходные данные, можно также использовать функции assert в C#.

### <a name="run-test-cases-in-visual-studio"></a>Запуск тестовых случаев в Visual Studio

Тестовый проект сценария U-SQL создается на базе платформы модульного тестирования C#. После построения проекта выберите **тест**  >  **Windows**  >  **Test Explorer**. Тестовые случаи можно запускать из **обозревателя тестов**. Можно также щелкнуть правой кнопкой мыши CS-файл в модульном тесте и выбрать пункт **запустить тесты**.

## <a name="test-c-udos"></a>Тестирование определяемых пользователем операторов C#

### <a name="create-test-cases-for-c-udos"></a>Создание тестовых случаев для определяемых пользователем операторов C#

Платформу модульного тестирования C# можно использовать для тестирования определяемых пользователем операторов C# (Udo). При тестировании UDO необходимо подготовить соответствующий объект **IRowset** в качестве входных данных.

Существует два способа создания объекта **IRowset** .

- Загрузка данных из файла для создания **IRowset**:

    ```csharp
    //Schema: "a:int, b:int"
    USqlColumn<int> col1 = new USqlColumn<int>("a");
    USqlColumn<int> col2 = new USqlColumn<int>("b");
    List<IColumn> columns = new List<IColumn> { col1, col2 };
    USqlSchema schema = new USqlSchema(columns);

    //Generate one row with default values
    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Get data from file
    IRowset rowset = UnitTestHelper.GetRowsetFromFile(@"processor.txt", schema, output.AsReadOnly(), discardAdditionalColumns: true, rowDelimiter: null, columnSeparator: '\t');
    ```

- Использование данных из коллекции данных для создания **IRowset**:

    ```csharp
    //Schema: "a:int, b:int"
    USqlSchema schema = new USqlSchema(
        new USqlColumn<int>("a"),
        new USqlColumn<int>("b")
    );

    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Generate Rowset with specified values
    List<object[]> values = new List<object[]>{
        new object[2] { 2, 3 },
        new object[2] { 10, 20 }
    };

    IEnumerable<IRow> rows = UnitTestHelper.CreateRowsFromValues(schema, values);
    IRowset rowset = UnitTestHelper.GetRowsetFromCollection(rows, output.AsReadOnly());
    ```

### <a name="verify-test-results"></a>Проверка результатов теста

После вызова функций UDO результат можно проверить, используя схемы и проверку значения Rowset с помощью функций assert в C#. В решение можно добавить **проект модульного теста для Udo на языке C# U-SQL** . Для этого выберите **файл > новый проект >** в Visual Studio.

### <a name="run-test-cases-in-visual-studio"></a>Запуск тестовых случаев в Visual Studio

После построения проекта выберите **тест**  >  **Windows**  >  **Test Explorer**. Тестовые случаи можно запускать из **обозревателя тестов**. Можно также щелкнуть правой кнопкой мыши CS-файл в модульном тесте и выбрать пункт **запустить тесты**.

## <a name="run-test-cases-in-azure-pipelines"></a>Выполнение тестовых случаев в Azure Pipelines<a name="run-test-cases-in-azure-devops"></a>

Оба проекта, **тестовые проекты сценария U-SQL** и **тестовые проекты UDO C#**, наследуют проекты модульного теста C#. Тестовая [задача Visual Studio](/azure/devops/pipelines/test/getting-started-with-continuous-testing) в Azure pipelines может выполнять эти тестовые случаи.

### <a name="run-u-sql-test-cases-in-azure-pipelines"></a>Выполнение тестовых случаев U-SQL в Azure Pipelines

Для теста U-SQL убедитесь, что вы загрузили `CPPSDK` на компьютер сборки, а затем передадите `CPPSDK` путь к `USqlScriptTestRunner(cppSdkFolderFullPath: @"")` .

#### <a name="what-is-cppsdk"></a>Что такое CPPSDK?

CPPSDK — это пакет, включающий Microsoft Visual C++ 14 и Windows SDK 10.0.10240.0. Этот пакет включает среду, необходимую среде выполнения U-SQL. Этот пакет можно получить в папке установки Средств Azure Data Lake для Visual Studio.

- Для Visual Studio 2015: `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK`.
- Для Visual Studio 2017: `C:\Program Files (x86)\Microsoft Visual Studio\2017\<Visual Studio Edition>\SDK\ScopeCppSDK`.
- Для Visual Studio 2019 он находится в разделе `C:\Program Files (x86)\Microsoft Visual Studio\2019\<Visual Studio Edition>\SDK\ScopeCppSDK`

#### <a name="prepare-cppsdk-in-the-azure-pipelines-build-agent"></a>Подготовка CPPSDK в агенте сборки Azure Pipelines

Ниже приведен наиболее распространенный способ подготовки зависимости CPPSDK в Azure Pipelines.

1. Заархивировать папку, содержащую библиотеки CPPSDK.

1. Добавьте ZIP-файл в систему управления версиями ZIP-файл гарантирует возврат всех библиотек в папке CPPSDK, чтобы файлы не игнорировались из-за `.gitignore` файла.

1. Распакуйте ZIP-файл в конвейер сборки.

1. Наведите указатель `USqlScriptTestRunner` на эту распакованную папку на компьютере сборки.

### <a name="run-c-udo-test-cases-in-azure-pipelines"></a>Выполнение тестовых случаев C# UDO в Azure Pipelines

Для теста UDO C# обязательно сослаться на следующие сборки, которые необходимы для Udo.

- Microsoft.Analytics.Interfaces
- Microsoft.Analytics.Types
- Microsoft.Analytics.UnitTest

Если вы ссылаетесь на них с помощью [Microsoft.Azure.DataLake.USQL.Interfaces пакета Nuget ](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.Interfaces/), добавьте задачу восстановления NuGet в конвейер сборки.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка конвейера CI/CD для Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Запуск скриптов U-SQL на локальном компьютере](data-lake-analytics-data-lake-tools-local-run.md)
- [Разработка базы данных U-SQL с помощью проекта базы данных U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md)
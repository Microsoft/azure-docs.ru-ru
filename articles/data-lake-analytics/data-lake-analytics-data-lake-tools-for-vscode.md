---
title: Использование средств Azure Data Lake для Visual Studio Code
description: Узнайте, как с помощью средств Azure Data Lake для Visual Studio Code создавать, тестировать и выполнять скрипты U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 02/09/2018
ms.openlocfilehash: 40e3ce17e036312e7c3fdee95fcb42d06f5845e9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96751365"
---
# <a name="use-azure-data-lake-tools-for-visual-studio-code"></a>Использование средств Azure Data Lake для Visual Studio Code

Из этой статьи вы узнаете, как создавать, тестировать и запускать скрипты U-SQL, используя средства Azure Data Lake для Visual Studio Code (VS Code). Эти сведения также представлены в следующем видеоролике:

[![Проигрыватель видео: средства Azure Data Lake для VS Code](media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-video.png)](https://channel9.msdn.com/Series/AzureDataLake/Azure-Data-Lake-Tools-for-VSCode?term=ADL%20Tools%20for%20VSCode")

## <a name="prerequisites"></a>Предварительные условия

Средства Azure Data Lake для VS Code поддерживают Windows, Linux и MacOS. Локальные запуск и отладка U-SQL работают только в Windows.

- [Visual Studio Code](https://www.visualstudio.com/products/code-vs.aspx)

Для MacOS и Linux:

- [Пакет SDK для .NET Core 2,0](https://www.microsoft.com/net/download/core)
- [Mono 5.2.x](https://www.mono-project.com/download/).

## <a name="install-azure-data-lake-tools"></a>Установка средств озера данных Azure

После установки необходимых компонентов можно установить Средства Azure Data Lake для VS Code.

### <a name="to-install-azure-data-lake-tools"></a>Установка Средств Azure Data Lake

1. Откройте Visual Studio Code.
2. Выберите **Расширения** в области слева. В поле поиска введите **Средства Azure Data Lake**.
3. Выберите **Установить** рядом с элементом **Средства Azure Data Lake**.

   ![Выбранные параметры для установки средств Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extensions.png)

   Через несколько секунд кнопка **Установить** изменится на **Перезагрузить**.
4. Нажмите кнопку **Перезагрузить**, чтобы активировать расширение **Средства Azure Data Lake**.
5. Выберите **Перезагрузить окно** для подтверждения. На панели **расширения** можно просмотреть **Azure Data Lake инструменты** .

## <a name="activate-azure-data-lake-tools"></a>Активация Средств Azure Data Lake

Чтобы активировать расширение, создайте новый или откройте имеющийся USQL-файл.

## <a name="work-with-u-sql"></a>Работа с U-SQL

Для работы с U-SQL нужно открыть файл или папку U-SQL.

### <a name="to-open-the-sample-script"></a>Открытие примера скрипта

Откройте палитру команд (CTRL+SHIFT+P) и введите **ADL: Open Sample Script**. При этом откроется другой экземпляр этого примера. Вы также можете изменить, настроить и отправить скрипт в этом экземпляре.

### <a name="to-open-a-folder-for-your-u-sql-project"></a>Открытие папки для проекта U-SQL

1. В Visual Studio Code выберите меню **Файл**, а затем — **Открыть папку**.
2. Укажите папку и выберите **Выбрать папку**.
3. Выберите меню **Файл**, а затем **Создать**. К проекту будет добавлен файл с именем Untitled-1.
4. Введите следующий код в файле Untitled-1:

   ```usql
   @departments  =
       SELECT * FROM
           (VALUES
               (31,    "Sales"),
               (33,    "Engineering"),
               (34,    "Clerical"),
               (35,    "Marketing")
           ) AS
                 D( DepID, DepName );
   ```

   ВЫВОД @departments     в "/Output/departments.csv" с помощью Outputters.Csv ();

    Этот скрипт создает в папке/output файл departments.csv с некоторыми данными.

5. Сохраните файл в открытой папке, присвоив ему имя **myUSQL.usql**.

### <a name="to-compile-a-u-sql-script"></a>Компиляция скрипта U-SQL

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите **ADL: Compile Script**. Результаты компиляции отображаются в окне **Вывод**. Чтобы запустить компиляцию задания U-SQL можно также щелкнуть правой кнопкой мыши файл скрипта и выбрать **ADL: Compile Script**. Результат компиляции отобразится в области **Вывод**.

### <a name="to-submit-a-u-sql-script"></a>Отправка скрипта U-SQL

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите **ADL: Submit Job**. Также можно щелкнуть правой кнопкой мыши файл скрипта и выбрать **ADL: Submit Job**.

После отправки задания U-SQL отобразятся журналы отправки в окне **Вывод** в VS Code. В области справа появится представление задания. Если отправка пройдет успешно, также появится URL-адрес задания. URL-адрес задания можно открыть в веб-браузере, чтобы отслеживать состояние задания в реальном времени.

На вкладке **СВОДКА** в представлении задания отображаются сведения о задании. К основным функциям относятся повторная отправка, дублирование и открытие скрипта на портале. На вкладке **ДАННЫЕ** в представлении задания можно просмотреть входные и выходные файлы, а также файлы ресурсов. Файлы можно скачать на локальный компьютер.

![Вкладка "Сводка" в представлении задания](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-summary.png)

![Вкладка "Данные" в представлении задания](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-data.png)

### <a name="to-set-the-default-context"></a>Установка контекста по умолчанию

Если параметры для отдельных файлов не заданы, вы можете задать контекст по умолчанию, чтобы применять эти параметры ко всем файлам скриптов.

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите команду **ADL: Set Default Context**. Вы также можете щелкнуть редактор скриптов правой кнопкой мыши и выбрать **ADL: Set Default Context**.
3. Выберите учетную запись, базу данных и схему. Параметры сохранятся в файле конфигурации xxx_settings.json.

   ![Учетная запись, база данных и схема, заданные в качестве контекста по умолчанию](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-sequence.png)

### <a name="to-set-script-parameters"></a>Установка параметров скрипта

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите команду **ADL: Set Script Parameters**.
3. Откроется файл xxx_settings.json со следующими свойствами:

   - **account**. Учетная запись Azure Data Lake Analytics из подписки Azure, необходимая для компиляции и запуска заданий U-SQL. Необходимо настроить учетную запись вычислений, прежде чем компилировать и запускать задания U-SQL.
   - **база данных**: база данных под вашей учетной записью. Значение по умолчанию — **master**.
   - **схема**: схема в базе данных. Значение по умолчанию — **dbo**.
   - **optionalSettings**:
        - **priority**. Значение приоритета в диапазоне от 1 до 1000, где 1 — наивысший приоритет. Значение по умолчанию: **1000**.
        - **degreeOfParallelism**. Значение степени параллелизма в диапазоне от 1 до 150. Значением по умолчанию является максимально допустимый коэффициент параллелизма в учетной записи Azure Data Lake Analytics.

   ![Содержимое JSON-файла](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-setting.png)

> [!NOTE]
> Если контекст по умолчанию не задан, после сохранения конфигурации сведения об учетной записи, базе данных и схеме появятся в строке состояния в левом нижнем углу соответствующего USQL-файла.

### <a name="to-set-git-ignore"></a>Установка игнорирования Git

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите **ADL: Set Git Ignore**.

   - Если в вашей рабочей папке VS Code нет файла **.gitIgnore**, в ней будет создан файл с именем **.gitIgnore**. По умолчанию в файл будут добавлены четыре элемента (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache**, **obj**). По мере необходимости можно вносить другие изменения.
   - Если в рабочей папке VS Code уже есть файл **.gitIgnore**, то в файл **.gitIgnore** будут добавлены четыре элемента (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache**, **obj**), если они не были добавлены ранее.

   ![Элементы в файле .gitIgnore](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-gitignore.png)

## <a name="work-with-code-behind-files-c-sharp-python-and-r"></a>Работа с файлами кода программной части: С#, Python и R

Средства Azure Data Lake поддерживают разные пользовательские коды. Инструкции см. в статье [Разработка U-SQL с помощью Python, R, и C Sharp для Azure Data Lake Analytics в VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md).

## <a name="work-with-assemblies"></a>Работа со сборками

Дополнительные сведения о разработке сборок см. в статье [Разработка сборок U-SQL для заданий Azure Data Lake Analytics]().

Средства Data Lake можно использовать для регистрации сборки пользовательского кода в каталоге Data Lake Analytics.

### <a name="to-register-an-assembly"></a>Регистрация сборки

Зарегистрировать сборку можно с помощью команды **ADL: Register Assembly** или **ADL: Register Assembly (Advanced)**.

### <a name="to-register-through-the-adl-register-assembly-command"></a>Для регистрации с помощью команды ADL: Register Assembly

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите **ADL: Register Assembly**.
3. Укажите локальный путь к сборке.
4. Выберите учетную запись Data Lake Analytics.
5. Выберите базу данных.

В браузере откроется портал и начнется процесс регистрации сборки.  

Более удобный способ вызова команды **ADL: Register Assembly** — щелкнуть правой кнопкой мыши DLL-файл в обозревателе.

### <a name="to-register-through-the-adl-register-assembly-advanced-command"></a>Регистрация с помощью команды ADL: Register Assembly (Advanced)

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.
2. Введите команду **ADL: Register Assembly (Advanced)**.
3. Укажите локальный путь к сборке.
4. Отобразится JSON-файл. Просмотрите и при необходимости измените зависимости сборки и параметры ресурсов. Инструкции отображаются в окне **вывод** . Чтобы перейти к регистрации сборки, сохраните (CTRL+S) JSON-файл.

   ![JSON-файл с зависимостями сборки и параметрами ресурсов](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-register-assembly-advance.png)

>[!NOTE]
>
>- Средства Azure Data Lake автоматически определяют, есть ли у библиотеки DLL зависимости сборки. Обнаруженные зависимости отображаются в JSON-файле.
>- Ресурсы DLL (например, TXT, PNG и CSV) можно отправлять в ходе регистрации сборки.

Еще один способ запуска команды **ADL: Register Assembly (Advanced)** — щелкнуть правой кнопкой мыши DLL-файл в проводнике.

В приведенном ниже коде U-SQL показано, как вызвать сборку. В этом примере имя сборки — *test*.

```usql
REFERENCE ASSEMBLY [test];
@a =
    EXTRACT
        Iid int,
    Starts DateTime,
    Region string,
    Query string,
    DwellTime int,
    Results string,
    ClickedUrls string
    FROM @"Sample/SearchLog.txt"
    USING Extractors.Tsv();
@d =
    SELECT DISTINCT Region
    FROM @a;
@d1 =
    PROCESS @d
    PRODUCE
        Region string,
    Mkt string
    USING new USQLApplication_codebehind.MyProcessor();
OUTPUT @d1
    TO @"Sample/SearchLogtest.txt"
    USING Outputters.Tsv();
```

## <a name="use-u-sql-local-run-and-local-debug-for-windows-users"></a>Локальные запуск и отладка U-SQL для пользователей Windows

Локальное выполнение U-SQL тестирует локальные данные и проверяет скрипт локально перед публикацией кода в Data Lake Analytics. С помощью функции локальной отладки можно завершить следующие задачи перед отправкой кода в Data Lake Analytics:

- Отладка кода программной части C#.
- Пошаговая отладка кода.
- Проверка скрипта локально.

Функция локального запуска и локальной отладки работает только в средах Windows и не поддерживается в ОС macOS и Linux.

Инструкции по локальному выполнению и отладке см. в статье [Локальный запуск и локальная отладка U-SQL в Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md).

## <a name="connect-to-azure"></a>Подключение к Azure

Перед компиляцией и выполнением скрипта U-SQL в Data Lake Analytics необходимо подключиться к учетной записи Azure.

<a id="sign-in-by-command"></a>

### <a name="to-connect-to-azure-by-using-a-command"></a>Подключение к Azure с помощью команды

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд.

2. Введите текст **ADL: Login**. Сведения для входа появятся в правом нижнем углу.

   ![Ввод команды входа](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login.png)

   ![Уведомление о входе и аутентификации](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-login-info.png)

3. Выберите **Копировать и открыть**, чтобы открыть [веб-страницу входа](https://aka.ms/devicelogin). Вставьте код в поле, а затем нажмите **Продолжить**.

    ![Веб-страница входа](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login-paste-code.png)  

4. Следуйте инструкциям, предоставленным на веб-странице входа. Когда соединение установлено, имя учетной записи Azure отображается в левом нижнем углу окна VS Code.

> [!NOTE]
>
> - В следующий раз средства Data Lake автоматически выполнят вход, если вы не выйдете из системы.
> - Если для учетной записи используется двухфакторная проверка подлинности, мы советуем использовать проверку подлинности через телефон, а не PIN-код.

Для выхода введите команду **ADL: Logout**.

### <a name="to-connect-to-azure-from-the-explorer"></a>Подключение к Azure из обозревателя

Разверните узел **AZURE DATALAKE**, нажмите **Войти в Azure**, а затем выполните действия 3 и 4 из раздела [Подключение к Azure с помощью команды](#sign-in-by-command).

![Выбор команды "Войти в Azure" в обозревателе](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-sign-in-from-explorer.png )  

Выйти из обозревателя нельзя. Чтобы выйти, следуйте инструкциями из раздела [Подключение к Azure с помощью команды](#sign-in-by-command).

## <a name="create-an-extraction-script"></a>Создание скрипта извлечения

Вы можете создать скрипт для извлечения CSV-, TSV- и TXT-файлов с помощью команды **ADL: Create EXTRACT Script** или обозревателя Azure Data Lake.

### <a name="to-create-an-extraction-script-by-using-a-command"></a>Создание скрипта извлечения с помощью команды

1. Нажмите клавиши CTRL+SHIFT+P, чтобы открыть палитру команд, и введите **ADL: Create EXTRACT Script**.
2. Укажите полный путь к файлу службы хранилища Azure и нажмите клавишу ВВОД.
3. Выберите одну учетную запись.
4. Выберите разделитель, чтобы извлечь ТХТ-файл.

![Процесс создания скрипта извлечения](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-process.png)

Скрипт извлечения создается на основе ваших записей. Для скрипта, который не может обнаружить столбцы, выберите один из этих двух параметров. В противном случае будет создан только один скрипт.

![Результат создания скрипта извлечения](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-result.png)

### <a name="to-create-an-extraction-script-from-the-explorer"></a>Создание скрипта извлечения из обозревателя

Еще один способ создать скрипт извлечения — с помощью контекстного меню CSV-, TSV- или TXT-файла в Azure Data Lake Store или хранилище BLOB-объектов Azure.

![Команда "Создать скрипт EXTRACT" в контекстном меню](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Разработка U-SQL с помощью Python, R, и C Sharp для Azure Data Lake Analytics в VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
- [Локальный запуск и локальная отладка U-SQL в Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
- [Начало работы с Azure Data Lake Analytics с помощью портала Azure](data-lake-analytics-get-started-portal.md)
- [Разработка скриптов U-SQL с помощью средств Data Lake для Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
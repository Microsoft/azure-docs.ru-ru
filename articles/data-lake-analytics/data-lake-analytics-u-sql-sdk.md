---
title: Выполнение заданий U-SQL локально Azure Data Lake пакета SDK U-SQL
description: Узнайте, как запускать и тестировать задания U-SQL с помощью командной строки и программных интерфейсов на локальной рабочей станции.
ms.service: data-lake-analytics
author: yanacai
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 03/01/2017
ms.openlocfilehash: 8fb60e62a63bfc4562f19d483dc84c99c37676b0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92215541"
---
# <a name="run-and-test-u-sql-with-azure-data-lake-u-sql-sdk"></a>Выполнение и тестирование U-SQL с помощью пакета SDK Azure Data Lake для U-SQL

При разработке сценария U-SQL, прежде чем он отправляется в облако, как правило, сценарий запускается и тестируется в локальной среде. Для этого сценария Azure Data Lake предоставляет пакет Nuget, который называется пакетом SDK Azure Data Lake для U-SQL, с помощью которого можно легко масштабировать выполнение и тестирование заданий U-SQL. Можно также интегрировать этот тест U-SQL с системой CI (непрерывная интеграция), чтобы автоматизировать компиляцию и тестирование.

Если вас интересует, как вручную выполнить локальный запуск и отладку сценария U-SQL с помощью инструментов с графическим интерфейсом пользователя, то для этого можно использовать средства Azure Data Lake для Visual Studio. Дополнительные сведения см. [здесь](data-lake-analytics-data-lake-tools-local-run.md).

## <a name="install-azure-data-lake-u-sql-sdk"></a>Установка пакета SDK Azure Data Lake для U-SQL

Вы можете получить [Azure Data Lake пакет SDK](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) U-SQL на NuGet.org. И перед его использованием необходимо убедиться в наличии зависимостей, как показано ниже.

### <a name="dependencies"></a>Зависимости

Для пакета SDK U-SQL для Data Lake требуются следующие зависимости:

- [Microsoft .NET Framework 4.6 или более поздней версии](https://www.microsoft.com/download/details.aspx?id=17851).
- Microsoft Visual C++ 14 и пакет SDK для Windows 10.0.10240.0 или более поздняя версия (в этой статье он называется пакетом CppSDK). Пакет CppSDK можно получить двумя способами.

  - Установка выпуска [Visual Studio Community](https://developer.microsoft.com/downloads/vs-thankyou). В папке Program Files должна существовать папка \Windows Kits\10, например C:\Program Files (x86) \Windows Kits\10\. Кроме того, в папке \Windows Kits\10\Lib находится версия пакета SDK для Windows 10. Если у вас нет этих папок, переустановите Visual Studio и обязательно выберите пакет SDK для Windows 10 в процессе установки. Если он установлен вместе с Visual Studio, локальный компилятор U-SQL обнаружит его автоматически.

    ![Пакет SDK Windows 10 для локального выполнения средств Data Lake для Visual Studio](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-for-visual-studio-local-run-windows-10-sdk.png)

  - Установка [средств Data Lake для Visual Studio](https://aka.ms/adltoolsvs). Файлы пакета Visual C++ и Windows SDK можно найти по адресу `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\ADL Tools\X.X.XXXX.X\CppSDK.`

    В этом случае локальный компилятор U-SQL не сможет найти зависимости автоматически. Вам потребуется указать путь к CppSDK. Можно скопировать файлы в другое место или просто использовать стандартный путь.

## <a name="understand-basic-concepts"></a>Основные понятия

### <a name="data-root"></a>Корневая папка данных

Корневая папка данных служит "локальным хранилищем" для локальной учетной записи среды вычислений. Она действует так же, как учетная запись Azure Data Lake Store в учетной записи Data Lake Analytics. Переход на другую корневую папку данных аналогичен переходу на другую учетную запись хранения. Если вам нужен доступ к данным, которые совместно используются различными корневыми папками данных, следует использовать в скриптах абсолютные пути или создать в корневой папке данных символические ссылки файловой системы (например, с помощью команды **mklink** для файловой системы NTFS), указывающие на совместно используемые данные.

Корневая папка данных используется в следующих целях.

- Хранение локальных метаданных, включая базы данных, таблицы, функции с табличным значением (TVF) и сборки.
- Поиск путей ввода-вывода, которые определяются как относительные пути в U-SQL. Использование относительных путей упрощает развертывание проектов U-SQL в Azure.

### <a name="file-path-in-u-sql"></a>Путь к файлу в U-SQL

В скриптах U-SQL можно использовать как относительные, так и абсолютные локальные пути. Относительный путь задается относительно пути к выбранной корневой папке данных. Мы советуем использовать в качестве разделителя пути символ "/", чтобы обеспечить совместимость скриптов с выполнением на стороне сервера. Ниже приведены некоторые примеры относительных путей и их абсолютные эквиваленты. В этих примерах предполагается, что мы используем корневую папку данных C:\LocalRunDataRoot.

|Относительный путь|Абсолютный путь|
|-------------|-------------|
|/abc/def/input.csv |C:\LocalRunDataRoot\abc\def\input.csv|
|abc/def/input.csv  |C:\LocalRunDataRoot\abc\def\input.csv|
|D:/abc/def/input.csv |D:\abc\def\input.csv|

### <a name="working-directory"></a>Рабочий каталог

Если сценарий U-SQL выполняется локально, то рабочий каталог создается во время компиляции в текущем каталоге выполнения. В него записываются выходные данные процесса компиляции, а также здесь создается теневая копия всех файлов среды выполнения, которые требуются для локального выполнения. Корневая папка рабочего каталога называется ScopeWorkDir, и рабочий каталог содержит следующие файлы:

|Каталог/файл|Каталог/файл|Каталог/файл|Определение|Описание|
|--------------|--------------|--------------|----------|-----------|
|C6A101DDCB470506| | |Хэш-строка версии среды выполнения|Теневая копия файлов среды выполнения, необходимых для локального выполнения|
| |Script_66AE4909AA0ED06C| |Имя скрипта и хэш-строка пути к скрипту|Выходные данные компиляции и ведение журнала шагов выполнения|
| | |\_script\_.abr|Выходные данные компилятора|Файл Algebra|
| | |\_ScopeCodeGen\_.*|Выходные данные компилятора|Созданный управляемый код|
| | |\_ScopeCodeGenEngine\_.*|Выходные данные компилятора|Созданный собственный код|
| | |referenced assemblies|Ссылка на сборку|Связанные файлы сборок.|
| | |deployed_resources|Развертывание ресурсов|Файлы развертывания ресурсов|
| | |xxxxxxxx.xxx[1..n]\_\*.\*|Журнал выполнения|Журнал шагов выполнения|

## <a name="use-the-sdk-from-the-command-line"></a>Использование пакета SDK из командной строки

### <a name="command-line-interface-of-the-helper-application"></a>Интерфейс командной строки вспомогательного приложения

Файл LocalRunHelper.exe в каталоге directory\build\runtime пакета SDK содержит вспомогательное приложение командной строки, которое предоставляет интерфейсы для большинства распространенных функций локального выполнения. Обратите внимание, что в командах и параметрах аргументов учитывается регистр. Для вызова вспомогательного приложения выполните следующую команду:

```console
LocalRunHelper.exe <command> <Required-Command-Arguments> [Optional-Command-Arguments]
```

Если запустить LocalRunHelper.exe без аргументов или с параметром **help**, отобразятся справочные сведения:

```console
> LocalRunHelper.exe help
    Command 'help' :  Show usage information
    Command 'compile' :  Compile the script
    Required Arguments :
        -Script param
                Script File Path
    Optional Arguments :
        -Shallow [default value 'False']
                Shallow compile
```

Эта справочная информация содержит следующие элементы.

- **Command** определяет имя выполняемой команды.  
- **Required Argument** перечисляет обязательные аргументы.  
- **Optional Argument** перечисляет необязательные аргументы и приводит для них значения по умолчанию.  Если необязательный аргумент имеет тип логического значения, он не принимает дополнительные параметры, и тогда его наличие определяет поведение, противоположное значению по умолчанию.

### <a name="return-value-and-logging"></a>Возвращаемое значение и ведение журнала

Вспомогательное приложение возвращает значение **0** в случае успешного выполнения или **-1** в случае сбоя. По умолчанию вспомогательное приложение отправляет все служебные сообщения в текущую консоль. Однако большинство команд поддерживает необязательный аргумент **-MessageOut path_to_log_file**, который позволяет перенаправлять выходные данные в файл журнала.

### <a name="environment-variable-configuring"></a>Настройка переменной среды

Для локального выполнения U-SQL требуется указать корневую папку данных в качестве локальной учетной записи хранения, а также указать путь к пакету CppSDK для зависимостей. Эти значения можно задать с помощью аргумента в командной строке или переменной среды.

- Настройка переменной среды **SCOPE_CPP_SDK**.

  Если для получения Microsoft Visual C++ и пакета SDK для Windows вы установили средства Data Lake для Visual Studio, убедитесь в наличии следующей папки:

    `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK`

  Определите новую переменную среды с именем **SCOPE_CPP_SDK**, которая будет указывать на этот каталог. Можно также скопировать эту папку в другое расположение и указать новый путь к ней в переменной среды **SCOPE_CPP_SDK**.

  Кроме настройки переменной среды, вы можете использовать аргумент **-CppSDK** в командной строке. Этот аргумент переопределяет используемую по умолчанию переменную среды CppSDK.

- Задайте переменную среды **LOCALRUN_DATAROOT** .

  Определите новую переменную среды с именем **LOCALRUN_DATAROOT**, которая будет указывать на корневую папку данных.

  Кроме настройки переменной среды, вы можете использовать аргумент **-DataRoot** в командной строке, который указывает на корневую папку данных. Этот аргумент переопределяет используемую по умолчанию переменную среды, задающую путь к корневой папке данных. Этот аргумент необходимо добавлять во все выполняемые командные строки, чтобы переопределить используемую по умолчанию переменную среды, задающую путь к корневой папке данных, для всех операций.

### <a name="sdk-command-line-usage-samples"></a>Примеры использования командной строки пакета SDK

#### <a name="compile-and-run"></a>Компиляция и запуск

Команда **run** компилирует скрипт и выполняет программу, полученную в результате компиляции. Эта команда принимает все аргументы командной строки, которые доступны для команд **compile** и **execute**.

```console
LocalRunHelper run -Script path_to_usql_script.usql [optional_arguments]
```

Ниже приведены необязательные аргументы для команды **run**.

|Аргумент|Значение по умолчанию|Описание|
|--------|-------------|-----------|
|-CodeBehind|Неверно|Сценарий содержит код программной части .cs.|
|-CppSDK| |Каталог CppSDK.|
|-DataRoot| Переменная среды DataRoot|Корневая папка для локального выполнения. По умолчанию используется значение из переменной среды LOCALRUN_DATAROOT.|
|-MessageOut| |Сохранение в файл всех сообщений, предназначенных для вывода на консоль.|
|-Parallel|1|Запуск плана с указанным значением параллелизма.|
|-References| |Список путей к дополнительным справочным сборкам или файлам данных кода программной части с разделителем ";".|
|-UdoRedirect|Неверно|Создание конфигурации перенаправления сборки Udo.|
|-UseDatabase|master|База данных, в которой нужно регистрировать временную сборку кода программной части.|
|-Verbose|Неверно|Отображение подробных выходных данных среды выполнения.|
|-WorkDir|Текущий каталог|Задает каталог для работы и выходных данных компилятора.|
|-RunScopeCEP|0|Используемый режим ScopeCEP.|
|-ScopeCEPTempPath|temp|Временный путь для потоковой передачи данных.|
|-OptFlags| |Разделенный запятыми список флагов оптимизатора.|

Приведем пример:

`LocalRunHelper run -Script d:\test\test1.usql -WorkDir d:\test\bin -CodeBehind -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB –Parallel 5 -Verbose`

Операции **compile** и **execute** можно также выполнять по отдельности.

#### <a name="compile-a-u-sql-script"></a>Компиляция скрипта U-SQL

Команда **compile** позволяет выполнить компиляцию скрипта U-SQL в исполняемые файлы.

```console
LocalRunHelper compile -Script path_to_usql_script.usql [optional_arguments]
```

Ниже приведены необязательные аргументы для команды **compile**.

|Аргумент|Описание|
|--------|-----------|
| -CodeBehind [значение по умолчанию False]|Сценарий содержит код программной части .cs.|
| -CppSDK [значение по умолчанию "]|Каталог CppSDK.|
| -DataRoot [значение по умолчанию: переменная среды DataRoot]|Корневая папка для локального выполнения. По умолчанию используется значение из переменной среды LOCALRUN_DATAROOT.|
| -MessageOut [значение по умолчанию "]|Сохранение в файл всех сообщений, предназначенных для вывода на консоль.|
| -References [значение по умолчанию "]|Список путей к дополнительным справочным сборкам или файлам данных кода программной части с разделителем ";".|
| -Shallow [значение по умолчанию False]|Неполная компиляция.|
| -UdoRedirect [значение по умолчанию False]|Создание конфигурации перенаправления сборки Udo.|
| -UseDatabase [значение по умолчанию master]|База данных, в которой нужно регистрировать временную сборку кода программной части.|
| -WorkDir [значение по умолчанию: текущий каталог]|Задает каталог для работы и выходных данных компилятора.|
| -RunScopeCEP [значение по умолчанию: "0"]|Используемый режим ScopeCEP.|
| -ScopeCEPTempPath [значение по умолчанию: "temp"]|Временный путь для потоковой передачи данных.|
| -OptFlags [значение по умолчанию: "]|Разделенный запятыми список флагов оптимизатора.|

Вот несколько примеров использования команды.

Компиляция скрипта U-SQL:

```console
LocalRunHelper compile -Script d:\test\test1.usql
```

Компиляция скрипта U-SQL с определенной корневой папкой данных (обратите внимание, что этот аргумент переопределяет заданную переменную среды):

```console
LocalRunHelper compile -Script d:\test\test1.usql –DataRoot c:\DataRoot
```

Компиляция скрипта U-SQL с определенным рабочим каталогом и ссылками на сборку и базу данных:

```console
LocalRunHelper compile -Script d:\test\test1.usql -WorkDir d:\test\bin -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB
```

#### <a name="execute-compiled-results"></a>Выполнение скомпилированного результата

Команда **execute** используется для выполнения скомпилированного результата.

```console
LocalRunHelper execute -Algebra path_to_compiled_algebra_file [optional_arguments]
```

Ниже приведены необязательные аргументы для команды **execute**.

|Аргумент|Значение по умолчанию|Описание|
|--------|-------------|-----------|
|-DataRoot | '' |Корневая папка для обработки метаданных. По умолчанию используется переменная среды **LOCALRUN_DATAROOT**.|
|-MessageOut | '' |Сохранение в файл всех сообщений, предназначенных для вывода на консоль.|
|-Parallel | 1 |Показатель выполнения созданных шагов локального выполнения на указанном уровне параллелизма.|
|-Verbose | False |Показатель отображения подробных выходных данных среды выполнения.|

Пример использования этой команды:

```console
LocalRunHelper execute -Algebra d:\test\workdir\C6A101DDCB470506\Script_66AE4909AA0ED06C\__script__.abr –DataRoot c:\DataRoot –Parallel 5
```

## <a name="use-the-sdk-with-programming-interfaces"></a>Использование пакета SDK с программными интерфейсами

Все программные интерфейсы размещены в LocalRunHelper.exe. Они позволяют объединить функциональность пакетов SDK U-SQL и тестовой платформы C#, чтобы масштабировать локальное тестирование скрипта U-SQL. В этой статье я буду использовать стандартный проект модульного теста C#, чтобы показать, как использовать эти интерфейсы для тестирования сценария U-SQL.

### <a name="step-1-create-c-unit-test-project-and-configuration"></a>Шаг 1. Создание проекта модульного теста C# и его настройка

- Создайте проект модульного теста C#, выбрав "Файл" > "Создать" > "Проект" > "Visual C#" > "Тест" > "Проект модульного теста".
- Добавьте в проект ссылку на LocalRunHelper.exe. LocalRunHelper.exe находится в папке \build\runtime в пакете NuGet.

   ![Добавление ссылки на пакет SDK Azure Data Lake для U-SQL](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-add-reference.png)

- Пакет SDK для U-SQL поддерживает **только** среду x64, поэтому обязательно задайте в качестве цели платформу сборки x64. Это можно сделать, выбрав "Свойство проекта" > "Сборка" > "Целевая платформа".

   ![Настройка платформы x64 в проекте для использования пакета SDK Azure Data Lake для U-SQL](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-x64.png)

- Обязательно задайте тестовую среду x64. В Visual Studio это можно сделать, выбрав "Тест" > "Параметры теста" > " 	Архитектура процессора по умолчанию" > "x64".

   ![Настройка тестовой среды x64 для использования пакета SDK Azure Data Lake для U-SQL](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-test-x64.png)

- Обязательно скопируйте все файлы зависимостей из NugetPackage\build\runtime в рабочий каталог, который обычно находится в каталоге проекта ProjectFolder\bin\x64\Debug.

### <a name="step-2-create-u-sql-script-test-case"></a>Шаг 2. Создание тестового случая сценария U-SQL

Ниже приведен пример кода для теста сценария U-SQL. Для тестирования необходимо подготовить сценарии, входные файлы и ожидаемые выходные файлы.

```usql
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.IO;
using System.Text;
using System.Security.Cryptography;
using Microsoft.Analytics.LocalRun;
namespace UnitTestProject1
{
    [TestClass]
    public class USQLUnitTest
    {
        [TestMethod]
        public void TestUSQLScript()
        {
            //Specify the local run message output path
            StreamWriter MessageOutput = new StreamWriter("../../../log.txt");
            LocalRunHelper localrun = new LocalRunHelper(MessageOutput);
            //Configure the DateRoot path, Script Path and CPPSDK path
            localrun.DataRoot = "../../../";
            localrun.ScriptPath = "../../../Script/Script.usql";
            localrun.CppSdkDir = "../../../CppSDK";
            //Run U-SQL script
            localrun.DoRun();
            //Script output
            string Result = Path.Combine(localrun.DataRoot, "Output/result.csv");
            //Expected script output
            string ExpectedResult = "../../../ExpectedOutput/result.csv";
            Test.Helpers.FileAssert.AreEqual(Result, ExpectedResult);
            //Don't forget to close MessageOutput to get logs into file
            MessageOutput.Close();
        }
    }
}
namespace Test.Helpers
{
    public static class FileAssert
    {
        static string GetFileHash(string filename)
        {
            Assert.IsTrue(File.Exists(filename));
            using (var hash = new SHA1Managed())
            {
                var clearBytes = File.ReadAllBytes(filename);
                var hashedBytes = hash.ComputeHash(clearBytes);
                return ConvertBytesToHex(hashedBytes);
            }
        }
        static string ConvertBytesToHex(byte[] bytes)
        {
            var sb = new StringBuilder();
            for (var i = 0; i < bytes.Length; i++)
            {
                sb.Append(bytes[i].ToString("x"));
            }
            return sb.ToString();
        }
        public static void AreEqual(string filename1, string filename2)
        {
            string hash1 = GetFileHash(filename1);
            string hash2 = GetFileHash(filename2);
            Assert.AreEqual(hash1, hash2);
        }
    }
}
```

### <a name="programming-interfaces-in-localrunhelperexe"></a>Программные интерфейсы в LocalRunHelper.exe

LocalRunHelper.exe предоставляет программные интерфейсы для локальной компиляции, выполнения и т. д. в U-SQL. Интерфейсы перечислены ниже.

### <a name="constructor"></a>Конструктор

public LocalRunHelper([System.IO.TextWriter messageOutput = null])

|Параметр|Type|Описание|
|---------|----|-----------|
|messageOutput|System.IO.TextWriter|Для вывода сообщений задайте значение NULL, чтобы использовать консоль.|

### <a name="properties"></a>Свойства

|Свойство.|Type|Описание|
|--------|----|-----------|
|AlgebraPath|строка|Путь к файлу алгебры (он является одним из результатов компиляции).|
|CodeBehindReferences|строка|Если сценарий содержит дополнительные ссылки на код программной части, укажите эти пути, разделенные точкой с запятой ";".|
|CppSdkDir|строка|Каталог пакета CppSDK.|
|CurrentDir|строка|Текущий каталог.|
|DataRoot|строка|Путь к корневой папке данных.|
|DebuggerMailPath|строка|Путь к почтовому слоту отладчика.|
|GenerateUdoRedirect|bool|Указывает, нужно ли создавать конфигурацию, переопределяющую пути перенаправления для загрузки сборки|
|HasCodeBehind|bool|Позволяет указать, содержит ли сценарий код программной части.|
|InputDir|строка|Каталог для входных данных.|
|MessagePath|строка|Путь к файлу дампа сообщений.|
|OutputDir|строка|Каталог для выходных данных.|
|Parallelism|INT|Значение параллелизма для вычислений алгебры.|
|ParentPid|INT|Идентификатор процесса родительского объекта, который отслеживает служба для выполнения выхода. Задайте значение 0 или отрицательное значение, чтобы этот параметр игнорировался.|
|ResultPath|строка|Путь к файлу дампа результатов.|
|RuntimeDir|строка|Каталог среды выполнения.|
|ScriptPath|строка|Расположение сценария.|
|Shallow|bool|Позволяет задать выполнение неполной компиляции.|
|TempDir|строка|Каталог temp|
|UseDataBase|строка|Укажите базу данных для регистрации временной сборки кода программной части. По умолчанию: база данных master.|
|WorkDir|строка|Предпочитаемый рабочий каталог.|

### <a name="method"></a>Метод

|Метод|Описание|Возвращает|Параметр|
|------|-----------|------|---------|
|public bool DoCompile()|Компиляция сценария U-SQL.|В случае успешного выполнения возвращает True.| |
|public bool DoExec()|Выполнение скомпилированного результата.|В случае успешного выполнения возвращает True.| |
|public bool DoRun()|Выполнение сценария U-SQL (компиляция и выполнение).|В случае успешного выполнения возвращает True.| |
|public bool IsValidRuntimeDir(string path)|Проверяет, является ли заданный путь допустимым путем среды выполнения.|Значение True, если путь допустимый.|Путь к каталогу среды выполнения.|

## <a name="faq-about-common-issue"></a>Часто задаваемые вопросы о распространенных проблемах

### <a name="error-1"></a>Ошибка 1

E_CSC_SYSTEM_INTERNAL: Внутренняя ошибка! Не удалось загрузить файл или сборку "ScopeEngineManaged.dll" либо одну из ее зависимостей. Не найден указанный модуль.

Проверьте следующее:

- Убедитесь, что используется среда x64. Необходимо использовать целевую платформу сборки x64 и тестовую среду x64. Ознакомьтесь с разделом **Шаг 1. Создание проекта модульного теста C# и его настройка** выше.
- Убедитесь, что все файлы зависимостей из NugetPackage\build\runtime были скопированы в рабочий каталог проекта.

## <a name="next-steps"></a>Дальнейшие действия

- Для знакомства с U-SQL см. статью о [начале работы с языком U-SQL для Azure Data Lake Analytics](data-lake-analytics-u-sql-get-started.md).
- Сведения о том, как записывать диагностические данные в журнал, см. в статье [Доступ к журналам диагностики для Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md).
- Более сложный запрос см. в статье [Анализ журналов веб-сайта с помощью Azure Data Lake Analytics](data-lake-analytics-analyze-weblogs.md).
- Сведения о просмотре сведений о задании см. в статье [Использование обозревателя заданий и представления заданий для Azure Data Lake Analyticsных заданий](data-lake-analytics-data-lake-tools-view-jobs.md).
- Сведения об использовании представления выполнения вершин см. [в разделе Использование представления выполнения вершин в Data Lake инструментов для Visual Studio](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md).

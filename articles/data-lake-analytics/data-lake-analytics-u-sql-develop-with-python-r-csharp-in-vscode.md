---
title: Выполнение заданий U-SQL в Python, R и C# — Azure Data Lake Analytics
description: Сведения об использовании кода программной части Python, R и C# для отправки заданий в Azure Data Lake.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 11/22/2017
ms.custom: devx-track-python
ms.openlocfilehash: d6066bd6ec2a4c986ae17ad0cce3e7f6f73b21e7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92219978"
---
# <a name="develop-u-sql-with-python-r-and-c-for-azure-data-lake-analytics-in-visual-studio-code"></a>Разработка U-SQL с помощью Python, R, и C# для Azure Data Lake Analytics в Visual Studio Code
Сведения об использовании Visual Studio Code (VSCode) для записи кода программной части Python, R и C# с помощью U-SQL и отправки заданий в службу Azure Data Lake. Дополнительные сведения об использовании Средств Azure Data Lake для VSCode см. в статье [Использование средств Azure Data Lake для Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md).

Перед написанием пользовательского кода программной части необходимо открыть папку или рабочую области в VSCode.


## <a name="prerequisites-for-python-and-r"></a>Предварительные требования для Python и R
Зарегистрируйте сборки расширений Python и R для учетной записи ADL. 
1. Откройте учетную запись на портале.
   - Щелкните **Обзор**. 
   - Щелкните **Пример сценария**.
2. Нажмите кнопку **Дополнительно**.
3. Выберите **Установка расширений U-SQL**. 
4. После установки расширений U-SQL отображается сообщение с подтверждением. 

   ![Настройка среды для Python и R](./media/data-lake-analytics-data-lake-tools-for-vscode/setup-the-enrionment-for-python-and-r.png)

   > [!Note]
   > Установите расширения VSCode для Python и R, чтобы получить все возможности при использовании языковых служб Python и R. 

## <a name="develop-python-file"></a>Разработка файла Python
1. В рабочей области щелкните **Создать файл**.
2. Запишите свой код в U-SQL. Ниже приведен пример кода.
    ```U-SQL
    REFERENCE ASSEMBLY [ExtPython];
    @t  = 
        SELECT * FROM 
        (VALUES
            ("D1","T1","A1","@foo Hello World @bar"),
            ("D2","T2","A2","@baz Hello World @beer")
        ) AS 
            D( date, time, author, tweet );

    @m  =
        REDUCE @t ON date
        PRODUCE date string, mentions string
        USING new Extension.Python.Reducer("pythonSample.usql.py", pyVersion : "3.5.1");

    OUTPUT @m
        TO "/tweetmentions.csv"
        USING Outputters.Csv();
    ```
    
3. Щелкните файл сценария правой кнопкой мыши и выберите параметр **ADL: Generate Python Code Behind File** (ADL: создать файл с выделенным кодом Python). 
4. В рабочей области будет создан файл **xxx.usql.py**. Запишите свой код в файл Python. Ниже приведен пример кода.

    ```Python
    def get_mentions(tweet):
        return ';'.join( ( w[1:] for w in tweet.split() if w[0]=='@' ) )

    def usqlml_main(df):
        del df['time']
        del df['author']
        df['mentions'] = df.tweet.apply(get_mentions)
        del df['tweet']
        return df
    ```
5. Для выполнения задания щелкните файл **USQL** правой кнопкой мыши и выберите **Compile Script** (Компиляции сценария) или **Отправить задание**.

## <a name="develop-r-file"></a>Разработка файла R
1. В рабочей области щелкните **Создать файл**.
2. Запишите свой код в файл U-SQL. Ниже приведен пример кода.
    ```U-SQL
    DEPLOY RESOURCE @"/usqlext/samples/R/my_model_LM_Iris.rda";
    DECLARE @IrisData string = @"/usqlext/samples/R/iris.csv";
    DECLARE @OutputFilePredictions string = @"/my/R/Output/LMPredictionsIris.txt";
    DECLARE @PartitionCount int = 10;

    @InputData =
        EXTRACT SepalLength double,
                SepalWidth double,
                PetalLength double,
                PetalWidth double,
                Species string
        FROM @IrisData
        USING Extractors.Csv();

    @ExtendedData =
        SELECT Extension.R.RandomNumberGenerator.GetRandomNumber(@PartitionCount) AS Par,
            SepalLength,
            SepalWidth,
            PetalLength,
            PetalWidth
        FROM @InputData;

    // Predict Species

    @RScriptOutput =
        REDUCE @ExtendedData
        ON Par
        PRODUCE Par,
                fit double,
                lwr double,
                upr double
        READONLY Par
        USING new Extension.R.Reducer(scriptFile : "RClusterRun.usql.R", rReturnType : "dataframe", stringsAsFactors : false);
    OUTPUT @RScriptOutput
    TO @OutputFilePredictions
    USING Outputters.Tsv();
    ```
3. Щелкните файл **USQL** правой кнопкой мыши и выберите **ADL: Generate CS Code Behind File** (ADL: создать файл с выделенным кодом R). 
4. В рабочей области будет создан файл **xxx.usql.r**. Запишите свой код в файл R. Ниже приведен пример кода.

    ```R
    load("my_model_LM_Iris.rda")
    outputToUSQL=data.frame(predict(lm.fit, inputFromUSQL, interval="confidence"))
    ```
5. Для выполнения задания щелкните файл **USQL** правой кнопкой мыши и выберите **Compile Script** (Компиляции сценария) или **Отправить задание**.

## <a name="develop-c-file"></a>Разработка файла C#
Файл с выделенным кодом является файлом C#, который связан с одним скриптом U-SQL. В файле с выделенным кодом можно определить скрипт, который относится к UDO, UDA, UDT и UDF. Все эти объекты можно будет напрямую использовать в скрипте, не регистрируя для них сборку. Файл с выделенным кодом помещается в ту же папку, что и связанный с ним файл скрипта U-SQL. Например, для скрипта с именем xxx.usql, файл Code Behind будет иметь имя xxx.usql.cs. Если вручную удалить файл с выделенным кодом, функция выделенного кода будет отключена для связанного с ним скрипта U-SQL. Дополнительные сведения о написании пользовательского кода для скриптов U-SQL можно найти в записи блога [Writing and Using Custom Code in U-SQL – User-Defined Functions]( https://blogs.msdn.microsoft.com/visualstudio/2015/10/28/writing-and-using-custom-code-in-u-sql-user-defined-functions/) (Написание и использование пользовательского кода в U-SQL — определяемые пользователем функции).

1. В рабочей области щелкните **Создать файл**.
2. Запишите свой код в файл U-SQL. Ниже приведен пример кода.
    ```U-SQL
    @a = 
        EXTRACT 
            Iid int,
        Starts DateTime,
        Region string,
        Query string,
        DwellTime int,
        Results string,
        ClickedUrls string 
        FROM @"/Samples/Data/SearchLog.tsv" 
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
        TO @"/output/SearchLogtest.txt" 
        USING Outputters.Tsv();
    ```
3. Щелкните файл **USQL** правой кнопкой мыши и выберите **ADL: Generate CS Code Behind File** (ADL: создать файл с выделенным кодом CS). 
4. В рабочей области будет создан файл **xxx.usql.cs**. Запишите свой код в файл CS. Ниже приведен пример кода.

    ```CS
    namespace USQLApplication_codebehind
    {
        [SqlUserDefinedProcessor]

        public class MyProcessor : IProcessor
        {
            public override IRow Process(IRow input, IUpdatableRow output)
            {
                output.Set(0, input.Get<string>(0));
                output.Set(1, input.Get<string>(0));
                return output.AsReadOnly();
            } 
        }
    }
    ```
5. Для выполнения задания щелкните файл **USQL** правой кнопкой мыши и выберите **Compile Script** (Компиляции сценария) или **Отправить задание**.

## <a name="next-steps"></a>Дальнейшие действия
* [Использование средств Azure Data Lake для Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
* [Локальный запуск и локальная отладка U-SQL в Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
* [Приступая к работе с аналитикой озера данных с помощью PowerShell](data-lake-analytics-get-started-powershell.md)
* [Приступая к работе с аналитикой озера данных с помощью портала Azure](data-lake-analytics-get-started-portal.md)
* [Использование инструментов озера данных для Visual Studio для разработки приложений U-SQL](data-lake-analytics-data-lake-tools-get-started.md)
* [Использование каталога U-SQL Azure Data Lake Analytics](./data-lake-analytics-u-sql-get-started.md)
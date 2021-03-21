---
title: Руководство по программированию U-SQL для Azure Data Lake
description: Ознакомьтесь с обзором U-SQL и программированием UDF в Azure Data Lake Analytics, чтобы создать хороший скрипт USQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: d263f2616607a96a8aa14f1ad1c06330b1b44644
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96510868"
---
# <a name="u-sql-programmability-guide-overview"></a>Обзор руководств по программированию U-SQL

U-SQL — это специальный язык запросов для рабочих нагрузок, обрабатывающих большие данные. U-SQL отличается использованием SQL-подобного декларативного синтаксиса, а также расширяемостью и удобством программирования, характерными для C#. В этом руководстве мы рассмотрим расширяемость и программируемость языка U-SQL на основе C#.

## <a name="requirements"></a>Требования

Скачивание и установка [средств Azure Data Lake для Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504).

## <a name="get-started-with-u-sql"></a>Начало работы с U-SQL  

Рассмотрим следующий скрипт U-SQL:

```usql
@a  = 
  SELECT * FROM 
    (VALUES
       ("Contoso",   1500.0, "2017-03-39"),
       ("Woodgrove", 2700.0, "2017-04-10")
    ) AS D( customer, amount, date );

@results =
  SELECT
    customer,
    amount,
    date
  FROM @a;    
```

Этот скрипт определяет два набора строк: `@a` и `@results`. Набор строк `@results` определяется из `@a`.

## <a name="c-types-and-expressions-in-u-sql-script"></a>Типы и выражения C# в скрипте U-SQL

Выражение U-SQL — это выражение C# в сочетании с логическими операциями U-SQL, такими как `AND`, `OR`, и `NOT`. Выражения U-SQL можно использовать с инструкциями SELECT, EXTRACT, WHERE, HAVING, GROUP BY и DECLARE. Например, следующий скрипт анализирует строку как значение даты и времени (DateTime).

```usql
@results =
  SELECT
    customer,
    amount,
    DateTime.Parse(date) AS date
  FROM @a;    
```

Следующий фрагмент кода анализирует значение строки даты и времени (DateTime) в предложении DECLARE.

```usql
DECLARE @d = DateTime.Parse("2016/01/01");
```

### <a name="use-c-expressions-for-data-type-conversions"></a>Использование выражений C# для преобразования типов данных

В примере ниже описывается преобразование данных datetime с помощью выражений C#. В этом конкретном случае строковые данные о дате и времени преобразуются в стандартное значение datetime использованием формата времени полуночи (00:00:00).

```usql
DECLARE @dt = "2016-07-06 10:23:15";

@rs1 =
  SELECT 
    Convert.ToDateTime(Convert.ToDateTime(@dt).ToString("yyyy-MM-dd")) AS dt,
    dt AS olddt
  FROM @rs0;

OUTPUT @rs1 
  TO @output_file 
  USING Outputters.Text();
```

### <a name="use-c-expressions-for-todays-date"></a>Использование выражений C# для получения текущей даты

Получить текущую дату можно с помощью следующего выражения C#: `DateTime.Now.ToString("M/d/yyyy")`

Ниже приведен пример использования этого выражения в скрипте.

```usql
@rs1 =
  SELECT
    MAX(guid) AS start_id,
    MIN(dt) AS start_time,
    MIN(Convert.ToDateTime(Convert.ToDateTime(dt<@default_dt?@default_dt:dt).ToString("yyyy-MM-dd"))) AS start_zero_time,
    MIN(USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)) AS start_fiscalperiod,
    DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
    user,
    des
  FROM @rs0
  GROUP BY user, des;
```
## <a name="using-net-assemblies"></a>Использование сборок .NET

Модель расширяемости U-SQL реализована как возможность добавлять пользовательский код из сборок .NET. 

### <a name="register-a-net-assembly"></a>Регистрация сборки .NET

Используйте инструкцию `CREATE ASSEMBLY`, чтобы поместить сборку .NET в базу данных U-SQL. После этого скрипты U-SQL смогут применять эти сборки, используя инструкцию `REFERENCE ASSEMBLY`. 

Следующий код показывает, как зарегистрировать сборку:

```usql
CREATE ASSEMBLY MyDB.[MyAssembly]
   FROM "/myassembly.dll";
```

Следующий код показывает, как сослаться на сборку:

```usql
REFERENCE ASSEMBLY MyDB.[MyAssembly];
```

Дополнительные сведения о регистрации сборки см. в [этой статье](/archive/blogs/azuredatalake/how-to-register-u-sql-assemblies-in-your-u-sql-catalog).


### <a name="use-assembly-versioning"></a>Использование версий сборок
В настоящее время U-SQL использует платформа .NET Framework версии 4.7.2. Убедитесь, что ваши сборки совместимы с этой версией среды выполнения.

Как упоминалось выше, U-SQL выполняет код в 64-разрядном формате (x64). Поэтому код нужно компилировать так, чтобы он выполнялся в среде x64. В противном случае вы получите ошибку неправильного формата, как показано выше.

Любой передаваемый в хранилище файл (библиотеки DLL, файлы ресурсов, включая другие среды выполнения, машинные сборки, файлы конфигурации и т. д.) не может быть более 400 МБ. Общий размер всех ресурсов, развернутых с помощью инструкции DEPLOY RESOURCE или с использованием ссылок на сборки и связанные файлы, не может превышать 3 ГБ.

И наконец, важно помнить, что в каждой базе данных U-SQL может быть только одна версия любой сборки. Например, если требуется и версия 7, и версия 8 библиотеки NewtonSoft Json.NET, их необходимо зарегистрировать в двух разных базах данных. Кроме того, каждый скрипт может ссылаться только на одну версию библиотеки DLL для каждой сборки. В этом отношении U-SQL соблюдает семантику C# управления сборками и версиями.

## <a name="use-user-defined-functions-udf"></a>Использование определяемых пользователем функций
В языке U-SQL концепция определяемых пользователем функций обозначает подпрограммы, которые принимают параметры, выполняют действия (например, сложные вычисления) и возвращают результат этого действия в виде определенного значения. Определяемая пользователем функция может возвращать только одно скалярное значение. Основной скрипт U-SQL может вызывать такие функции, как и любую другую скалярную функцию C#.

Мы советуем инициализировать определяемые пользователем функции U-SQL как **общедоступные** (public) и **статические** (static).

```usql
public static string MyFunction(string param1)
{
    return "my result";
}
```

Сначала давайте рассмотрим простой пример создания определяемой пользователем функции.

В этом сценарии нужно определить финансовый период (квартал и месяц) для первого входа в систему определенного пользователя. Финансовый год для этого примера начинается в июне.

Чтобы вычислить финансовый период, мы создаем следующую функцию C#:

```usql
public static string GetFiscalPeriod(DateTime dt)
{
    int FiscalMonth=0;
    if (dt.Month < 7)
    {
    FiscalMonth = dt.Month + 6;
    }
    else
    {
    FiscalMonth = dt.Month - 6;
    }

    int FiscalQuarter=0;
    if (FiscalMonth >=1 && FiscalMonth<=3)
    {
    FiscalQuarter = 1;
    }
    if (FiscalMonth >= 4 && FiscalMonth <= 6)
    {
    FiscalQuarter = 2;
    }
    if (FiscalMonth >= 7 && FiscalMonth <= 9)
    {
    FiscalQuarter = 3;
    }
    if (FiscalMonth >= 10 && FiscalMonth <= 12)
    {
    FiscalQuarter = 4;
    }

    return "Q" + FiscalQuarter.ToString() + ":P" + FiscalMonth.ToString();
}
```

Она вычисляет финансовый месяц и квартал и возвращает строковое значение. Для июня (первый месяц первого финансового квартала) используются значения "Q1:P1". Для июля — "Q1:P2" и т. д.

В нашем проекте U-SQL мы будем использовать базовые функции C#.

Вот как выглядит раздел кода программной части для этого примера:

```usql
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace USQL_Programmability
{
    public class CustomFunctions
    {
        public static string GetFiscalPeriod(DateTime dt)
        {
            int FiscalMonth=0;
            if (dt.Month < 7)
            {
                FiscalMonth = dt.Month + 6;
            }
            else
            {
                FiscalMonth = dt.Month - 6;
            }

            int FiscalQuarter=0;
            if (FiscalMonth >=1 && FiscalMonth<=3)
            {
                FiscalQuarter = 1;
            }
            if (FiscalMonth >= 4 && FiscalMonth <= 6)
            {
                FiscalQuarter = 2;
            }
            if (FiscalMonth >= 7 && FiscalMonth <= 9)
            {
                FiscalQuarter = 3;
            }
            if (FiscalMonth >= 10 && FiscalMonth <= 12)
            {
                FiscalQuarter = 4;
            }

            return "Q" + FiscalQuarter.ToString() + ":" + FiscalMonth.ToString();
        }
    }
}
```

Теперь мы вызовем эту функцию из основного скрипта U-SQL. Для этого нам нужно предоставить полное имя функции, включая пространство имен, в формате: "Пространство_имен.Класс.Функция(параметр)".
```usql
USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)
```

Так будет выглядеть вызов в основном скрипте U-SQL:
```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid Guid,
        dt DateTime,
            user String,
            des String
    FROM @input_file USING Extractors.Tsv();

DECLARE @default_dt DateTime = Convert.ToDateTime("06/01/2016");

@rs1 =
    SELECT
        MAX(guid) AS start_id,
    MIN(dt) AS start_time,
        MIN(Convert.ToDateTime(Convert.ToDateTime(dt<@default_dt?@default_dt:dt).ToString("yyyy-MM-dd"))) AS start_zero_time,
        MIN(USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)) AS start_fiscalperiod,
        user,
        des
    FROM @rs0
    GROUP BY user, des;

OUTPUT @rs1 
    TO @output_file 
    USING Outputters.Text();
```

Ниже приведен файл выходных данных с результатом выполнения скрипта.

```output
0d8b9630-d5ca-11e5-8329-251efa3a2941,2016-02-11T07:04:17.2630000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User1",""

20843640-d771-11e5-b87b-8b7265c75a44,2016-02-11T07:04:17.2630000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User2",""

301f23d2-d690-11e5-9a98-4b4f60a1836f,2016-02-11T09:01:33.9720000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User3",""
```

Это простой пример использования встроенной определяемой пользователем функции в U-SQL.

### <a name="keep-state-between-udf-invocations"></a>Сохранение состояния между вызовами определяемой пользователем функции
Объекты, поддерживающие программирование C# в U-SQL, могут быть более сложными. Например, они могут интерактивно взаимодействовать через глобальные переменные кода программной части. Рассмотрим приведенный ниже сценарий использования.

В больших организациях пользователи могут переключаться между разными внутренними приложениями, К ним могут относиться Microsoft Dynamics CRM, Power BI и т. д. Клиенты могут захотеть применить анализ телеметрии для пользовательских переключений между различными приложениями, например оценить тенденции использования и т. д. Цель для бизнеса — оптимизация использования приложений. Они могут также пожелать объединить различные приложения или процедуры входа.

Чтобы достичь этой цели, мы должны определить идентификаторы сеансов и интервалы задержки между последними сеансами.

Нам нужно найти предыдущий вход и привязать этот вход ко всем сеансам, создаваемым для одного и того же приложения. Первая трудность заключается в том, что основной скрипт U-SQL не позволяет производить вычисления с функцией LAG над уже вычисленным столбцом. Вторая трудность — мы хотим сохранить определенный сеанс для всех сеансов за тот же интервал.

Чтобы решить эту проблему, мы используем глобальную переменную в разделе кода программной части `static public string globalSession;`.

Эта глобальная переменная применяется для всего набора строк во время выполнения скрипта.

Так будет выглядеть раздел кода программной части для программы U-SQL:

```csharp
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace USQLApplication21
{
    public class UserSession
    {
        static public string globalSession;
        static public string StampUserSession(string eventTime, string PreviousRow, string Session)
        {

            if (!string.IsNullOrEmpty(PreviousRow))
            {
                double timeGap = Convert.ToDateTime(eventTime).Subtract(Convert.ToDateTime(PreviousRow)).TotalMinutes;
                if (timeGap <= 60) {return Session;}
                else {return Guid.NewGuid().ToString();}
            }
            else {return Guid.NewGuid().ToString();}

        }

        static public string getStampUserSession(string Session)
        {
            if (Session != globalSession && !string.IsNullOrEmpty(Session)) { globalSession = Session; }
            return globalSession;
        }

    }
}
```

Как видно из этого примера, глобальная переменная `static public string globalSession;` используется внутри функции `getStampUserSession` и инициализируется заново каждый раз, когда изменяется значение параметра сеанса.

Основной скрипт U-SQL выглядит следующим образом:

```usql
DECLARE @in string = @"\UserSession\test1.tsv";
DECLARE @out1 string = @"\UserSession\Out1.csv";
DECLARE @out2 string = @"\UserSession\Out2.csv";
DECLARE @out3 string = @"\UserSession\Out3.csv";

@records =
    EXTRACT DataId string,
            EventDateTime string,           
            UserName string,
            UserSessionTimestamp string

    FROM @in
    USING Extractors.Tsv();

@rs1 =
    SELECT 
        EventDateTime,
        UserName,
    LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC) AS prevDateTime,          
        string.IsNullOrEmpty(LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)) AS Flag,           
        USQLApplication21.UserSession.StampUserSession
           (
            EventDateTime,
            LAG(EventDateTime, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC),
            LAG(UserSessionTimestamp, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)
           ) AS UserSessionTimestamp
    FROM @records;

@rs2 =
    SELECT 
        EventDateTime,
        UserName,
        LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC) AS prevDateTime,
        string.IsNullOrEmpty( LAG(EventDateTime, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)) AS Flag,
        USQLApplication21.UserSession.getStampUserSession(UserSessionTimestamp) AS UserSessionTimestamp
    FROM @rs1
    WHERE UserName != "UserName";

OUTPUT @rs2
    TO @out2
    ORDER BY UserName, EventDateTime ASC
    USING Outputters.Csv();
```

Функция `USQLApplication21.UserSession.getStampUserSession(UserSessionTimestamp)` вызывается здесь во время вычисления второго набора строк в памяти. Она передает столбец `UserSessionTimestamp` и возвращает значение до изменения `UserSessionTimestamp`.

Выходной файл будет выглядеть следующим образом:

```output
"2016-02-19T07:32:36.8420000-08:00","User1",,True,"72a0660e-22df-428e-b672-e0977007177f"
"2016-02-17T11:52:43.6350000-08:00","User2",,True,"4a0cd19a-6e67-4d95-a119-4eda590226ba"
"2016-02-17T11:59:08.8320000-08:00","User2","2016-02-17T11:52:43.6350000-08:00",False,"4a0cd19a-6e67-4d95-a119-4eda590226ba"
"2016-02-11T07:04:17.2630000-08:00","User3",,True,"51860a7a-1610-4f74-a9ea-69d5eef7cd9c"
"2016-02-11T07:10:33.9720000-08:00","User3","2016-02-11T07:04:17.2630000-08:00",False,"51860a7a-1610-4f74-a9ea-69d5eef7cd9c"
"2016-02-15T21:27:41.8210000-08:00","User3","2016-02-11T07:10:33.9720000-08:00",False,"4d2bc48d-bdf3-4591-a9c1-7b15ceb8e074"
"2016-02-16T05:48:49.6360000-08:00","User3","2016-02-15T21:27:41.8210000-08:00",False,"dd3006d0-2dcd-42d0-b3a2-bc03dd77c8b9"
"2016-02-16T06:22:43.6390000-08:00","User3","2016-02-16T05:48:49.6360000-08:00",False,"dd3006d0-2dcd-42d0-b3a2-bc03dd77c8b9"
"2016-02-17T16:29:53.2280000-08:00","User3","2016-02-16T06:22:43.6390000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-17T16:39:07.2430000-08:00","User3","2016-02-17T16:29:53.2280000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-17T17:20:39.3220000-08:00","User3","2016-02-17T16:39:07.2430000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-19T05:23:54.5710000-08:00","User3","2016-02-17T17:20:39.3220000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T05:48:37.7510000-08:00","User3","2016-02-19T05:23:54.5710000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T06:40:27.4830000-08:00","User3","2016-02-19T05:48:37.7510000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T07:27:37.7550000-08:00","User3","2016-02-19T06:40:27.4830000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T19:35:40.9450000-08:00","User3","2016-02-19T07:27:37.7550000-08:00",False,"3f385f0b-3e68-4456-ac74-ff6cef093674"
"2016-02-20T00:07:37.8250000-08:00","User3","2016-02-19T19:35:40.9450000-08:00",False,"685f76d5-ca48-4c58-b77d-bd3a9ddb33da"
"2016-02-11T09:01:33.9720000-08:00","User4",,True,"9f0cf696-c8ba-449a-8d5f-1ca6ed8f2ee8"
"2016-02-17T06:30:38.6210000-08:00","User4","2016-02-11T09:01:33.9720000-08:00",False,"8b11fd2a-01bf-4a5e-a9af-3c92c4e4382a"
"2016-02-17T22:15:26.4020000-08:00","User4","2016-02-17T06:30:38.6210000-08:00",False,"4e1cb707-3b5f-49c1-90c7-9b33b86ca1f4"
"2016-02-18T14:37:27.6560000-08:00","User4","2016-02-17T22:15:26.4020000-08:00",False,"f4e44400-e837-40ed-8dfd-2ea264d4e338"
"2016-02-19T01:20:31.4800000-08:00","User4","2016-02-18T14:37:27.6560000-08:00",False,"2136f4cf-7c7d-43c1-8ae2-08f4ad6a6e08"
```

В этом примере мы видим более сложный сценарий использования, в котором глобальная переменная используется в разделе кода программной части для всего набора строк в памяти.

## <a name="next-steps"></a>Дальнейшие действия
* [Инструкции по программированию U-SQL — определяемый пользователем тип и функции](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
* [Инструкции по программированию U-SQL — UDO](data-lake-analytics-u-sql-programmability-guide-UDO.md)
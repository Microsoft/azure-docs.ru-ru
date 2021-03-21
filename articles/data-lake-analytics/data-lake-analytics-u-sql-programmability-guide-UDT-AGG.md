---
title: Инструкции по программированию в U-SQL UDT и функции для Azure Data Lake
description: Узнайте о возможностях программирования функции и пользовательских типов U-SQL в Azure Data Lake Analytics, чтобы создать хороший скрипт USQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 10fcce9a667d9a08318f5adab804f482387052ff
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97606663"
---
# <a name="u-sql-programmability-guide---udt-and-udagg"></a>Инструкции по программированию U-SQL — определяемый пользователем тип и функции



## <a name="use-user-defined-types-udt"></a>Использование определяемых пользователем типов данных
Определяемые пользователем типы или UDT — это еще одно средство, поддерживающее возможность программирования в U-SQL. Определяемый пользователем тип в U-SQL действует так же, как аналогичный тип в C#. C# — это строго типизированный язык, в котором можно использовать встроенные и пользовательские типы данных.

U-SQL не может неявно сериализировать или десериализировать произвольные пользовательские типы данных, когда пользовательский тип данных передается между вершинами в наборах строк. Поэтому пользователь должен указать явный модуль форматирования с помощью интерфейса IFormatter. Таким образом, U-SQL получит методы сериализации и десериализации для пользовательских типов данных.

> [!NOTE]
> Встроенные средства извлечения и вывода U-SQL в настоящее время не могут сериализировать или десериализировать пользовательские типы данных в файлы или из них, даже если они используют набор IFormatter. Поэтому, если вы записываете пользовательские типы данных в файл с помощью инструкции OUTPUT или считываете их с помощью средств извлечения, вам необходимо передать их в качестве массива строк или байтов. Затем необходимо явным образом вызвать код сериализации и десериализации (метод пользовательских типов данных ToString()). Но пользовательские средства извлечения и вывода могут читать и записывать пользовательские типы данных.

При попытке использования пользовательского типа в средствах EXTRACTOR и OUTPUTTER (на основе инструкции SELECT), как показано ниже,

```usql
@rs1 =
    SELECT 
        MyNameSpace.Myfunction_Returning_UDT(filed1) AS myfield
    FROM @rs0;

OUTPUT @rs1 
    TO @output_file 
    USING Outputters.Text();
```

мы получим такую ошибку:

```output
Error   1   E_CSC_USER_INVALIDTYPEINOUTPUTTER: Outputters.Text was used to output column myfield of type
MyNameSpace.Myfunction_Returning_UDT.

Description:

Outputters.Text only supports built-in types.

Resolution:

Implement a custom outputter that knows how to serialize this type, or call a serialization method on the type in
the preceding SELECT.   C:\Users\sergeypu\Documents\Visual Studio 2013\Projects\USQL-Programmability\
USQL-Programmability\Types.usql 52  1   USQL-Programmability
```

Для работы с определяемым пользователем типом в средстве OUTPUTTER мы должны сериализовать данные в строку с помощью метода ToString() или создать пользовательское средство OUTPUTTER.

Сейчас определяемые пользователем типы нельзя использовать в инструкции GROUP BY. Если определяемый пользователем тип будет использован в GROUP BY, возникает следующая ошибка:

```output
Error   1   E_CSC_USER_INVALIDTYPEINCLAUSE: GROUP BY doesn't support type MyNameSpace.Myfunction_Returning_UDT
for column myfield

Description:

GROUP BY doesn't support UDT or Complex types.

Resolution:

Add a SELECT statement where you can project a scalar column that you want to use with GROUP BY.
C:\Users\sergeypu\Documents\Visual Studio 2013\Projects\USQL-Programmability\USQL-Programmability\Types.usql
62  5   USQL-Programmability
```

Чтобы определить пользовательский тип, сделайте следующее.

1. Добавьте приведенные ниже пространства имен.

```csharp
using Microsoft.Analytics.Interfaces
using System.IO;
```

2. Добавьте `Microsoft.Analytics.Interfaces` для интерфейсов определяемых пользователем типов. Кроме того, `System.IO` может потребоваться для определения интерфейса IFormatter.

3. Определите определяемый пользователем тип данных с помощью атрибута SqlUserDefinedType.

**SqlUserDefinedType** указывает, что определение типа в сборке представляет определяемый пользователем тип данных для U-SQL. Свойства этого атрибута отражают физические характеристики определяемого пользователем типа данных. Этот класс не наследуется.

SqlUserDefinedType является обязательным атрибутом для определяемых пользователем типов данных.

Ниже представлен конструктор для этого класса.  

* SqlUserDefinedTypeAttribute (модуль форматирования).

* Модуль форматирования — обязательный параметр, определяющий форматирование для определяемого пользователем типа данных. Здесь нужно передать тип интерфейса `IFormatter`.

```csharp
[SqlUserDefinedType(typeof(MyTypeFormatter))]
public class MyType
{ … }
```

* Типичные определяемые пользователем типы данных также требуют определения интерфейса IFormatter, как показано ниже.

```csharp
public class MyTypeFormatter : IFormatter<MyType>
{
    public void Serialize(MyType instance, IColumnWriter writer, ISerializationContext context)
    { … }

    public MyType Deserialize(IColumnReader reader, ISerializationContext context)
    { … }
}
```

`IFormatter`Интерфейс сериализует и десериализует граф объекта с корневым типом \<typeparamref name="T"> .

\<typeparam name="T">Корневой тип графа объекта для сериализации и десериализации.

* **Десериализация** — процесс, обратный сериализации данных в предоставленном потоке, в ходе которого воспроизводится граф объектов.

* **Сериализация** — процесс сериализации объекта или графа объектов с использованием заданного корня в предоставленный поток.

Экземпляр `MyType` — экземпляр типа.  
Модуль записи `IColumnWriter` или чтения `IColumnReader` — базовый поток столбца.  
Контекст `ISerializationContext` — перечисление, которое определяет набор флагов, указывающих контекст источника или назначения для потока во время сериализации.

* **Intermediate** — указывает, что исходный или целевой контекст не является материализованным хранилищем.

* **Persistent** — указывает, что исходный или целевой контекст является материализованным хранилищем.

Как и обычный тип C#, определение определяемого пользователем типа данных в U-SQL может переопределять такие операторы, как +/==/!=. Также оно может содержать статические методы. Например, если мы собираемся использовать этот определяемый пользователем тип данных в качестве параметра агрегатной функции MIN U-SQL, нужно переопределить оператор <.

Ранее в этой статье мы привели пример, в котором определяли финансовый период для конкретной даты в формате `Qn:Pn (Q1:P10)`. Следующий пример показывает, как использовать определяемый пользователем тип данных для значений финансового периода.

Ниже приведен пример раздела кода программной части с определяемым пользователем типом данных и интерфейсом IFormatter.

```csharp
[SqlUserDefinedType(typeof(FiscalPeriodFormatter))]
public struct FiscalPeriod
{
    public int Quarter { get; private set; }

    public int Month { get; private set; }

    public FiscalPeriod(int quarter, int month):this()
    {
    this.Quarter = quarter;
    this.Month = month;
    }

    public override bool Equals(object obj)
    {
    if (ReferenceEquals(null, obj))
    {
        return false;
    }

    return obj is FiscalPeriod && Equals((FiscalPeriod)obj);
    }

    public bool Equals(FiscalPeriod other)
    {
return this.Quarter.Equals(other.Quarter) && this.Month.Equals(other.Month);
    }

    public bool GreaterThan(FiscalPeriod other)
    {
return this.Quarter.CompareTo(other.Quarter) > 0 || this.Month.CompareTo(other.Month) > 0;
    }

    public bool LessThan(FiscalPeriod other)
    {
return this.Quarter.CompareTo(other.Quarter) < 0 || this.Month.CompareTo(other.Month) < 0;
    }

    public override int GetHashCode()
    {
    unchecked
    {
        return (this.Quarter.GetHashCode() * 397) ^ this.Month.GetHashCode();
    }
    }

    public static FiscalPeriod operator +(FiscalPeriod c1, FiscalPeriod c2)
    {
return new FiscalPeriod((c1.Quarter + c2.Quarter) > 4 ? (c1.Quarter + c2.Quarter)-4 : (c1.Quarter + c2.Quarter), (c1.Month + c2.Month) > 12 ? (c1.Month + c2.Month) - 12 : (c1.Month + c2.Month));
    }

    public static bool operator ==(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.Equals(c2);
    }

    public static bool operator !=(FiscalPeriod c1, FiscalPeriod c2)
    {
    return !c1.Equals(c2);
    }
    public static bool operator >(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.GreaterThan(c2);
    }
    public static bool operator <(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.LessThan(c2);
    }
    public override string ToString()
    {
    return (String.Format("Q{0}:P{1}", this.Quarter, this.Month));
    }

}

public class FiscalPeriodFormatter : IFormatter<FiscalPeriod>
{
    public void Serialize(FiscalPeriod instance, IColumnWriter writer, ISerializationContext context)
    {
    using (var binaryWriter = new BinaryWriter(writer.BaseStream))
    {
        binaryWriter.Write(instance.Quarter);
        binaryWriter.Write(instance.Month);
        binaryWriter.Flush();
    }
    }

    public FiscalPeriod Deserialize(IColumnReader reader, ISerializationContext context)
    {
    using (var binaryReader = new BinaryReader(reader.BaseStream))
    {
var result = new FiscalPeriod(binaryReader.ReadInt16(), binaryReader.ReadInt16());
        return result;
    }
    }
}
```

Определяемый тип содержит два числа: квартал и месяц. Здесь определены операторы `==/!=/>/<` и статический метод `ToString()`.

Как упоминалось выше, определяемый пользователем тип данных можно использовать в выражениях SELECT, но нельзя — в средствах OUTPUTTER и EXTRACTOR без настраиваемой сериализации. Нужно сериализовать его в строку с помощью метода `ToString()` или использовать пользовательское средство OUTPUTTER или EXTRACTOR.

Теперь давайте рассмотрим, как можно применять определяемый пользователем тип данных. В разделе кода программной части мы изменили функцию GetFiscalPeriod следующим образом:

```csharp
public static FiscalPeriod GetFiscalPeriodWithCustomType(DateTime dt)
{
    int FiscalMonth = 0;
    if (dt.Month < 7)
    {
    FiscalMonth = dt.Month + 6;
    }
    else
    {
    FiscalMonth = dt.Month - 6;
    }

    int FiscalQuarter = 0;
    if (FiscalMonth >= 1 && FiscalMonth <= 3)
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

    return new FiscalPeriod(FiscalQuarter, FiscalMonth);
}
```

Как видите, он возвращает значение с типом данных FiscalPeriod.

Ниже приведен пример его дальнейшего использования в основном скрипте U-SQL. В этом примере представлено несколько способов вызова определяемого пользователем типа данных из скрипта U-SQL.

```usql
DECLARE @input_file string = @"c:\work\cosmos\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"c:\work\cosmos\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        guid string,
        dt DateTime,
        user String,
        des String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
    SELECT 
        guid AS start_id,
        dt,
        DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).Quarter AS fiscalquarter,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).Month AS fiscalmonth,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt) + new USQL_Programmability.CustomFunctions.FiscalPeriod(1,7) AS fiscalperiod_adjusted,
        user,
        des
    FROM @rs0;

@rs2 =
    SELECT 
        start_id,
        dt,
        DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
        fiscalquarter,
        fiscalmonth,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).ToString() AS fiscalperiod,

       // This user-defined type was created in the prior SELECT.  Passing the UDT to this subsequent SELECT would have failed if the UDT was not annotated with an IFormatter.
           fiscalperiod_adjusted.ToString() AS fiscalperiod_adjusted,
           user,
           des
    FROM @rs1;

OUTPUT @rs2 
    TO @output_file 
    USING Outputters.Text();
```

Ниже приведен пример полного раздела кода программной части.

```csharp
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;

namespace USQL_Programmability
{
    public class CustomFunctions
    {
        static public DateTime? ToDateTime(string dt)
        {
            DateTime dtValue;

            if (!DateTime.TryParse(dt, out dtValue))
                return Convert.ToDateTime(dt);
            else
                return null;
        }

        public static FiscalPeriod GetFiscalPeriodWithCustomType(DateTime dt)
        {
            int FiscalMonth = 0;
            if (dt.Month < 7)
            {
                FiscalMonth = dt.Month + 6;
            }
            else
            {
                FiscalMonth = dt.Month - 6;
            }

            int FiscalQuarter = 0;
            if (FiscalMonth >= 1 && FiscalMonth <= 3)
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

            return new FiscalPeriod(FiscalQuarter, FiscalMonth);
        }        [SqlUserDefinedType(typeof(FiscalPeriodFormatter))]
        public struct FiscalPeriod
        {
            public int Quarter { get; private set; }

            public int Month { get; private set; }

            public FiscalPeriod(int quarter, int month):this()
            {
                this.Quarter = quarter;
                this.Month = month;
            }

            public override bool Equals(object obj)
            {
                if (ReferenceEquals(null, obj))
                {
                    return false;
                }

                return obj is FiscalPeriod && Equals((FiscalPeriod)obj);
            }

            public bool Equals(FiscalPeriod other)
            {
return this.Quarter.Equals(other.Quarter) &&    this.Month.Equals(other.Month);
            }

            public bool GreaterThan(FiscalPeriod other)
            {
return this.Quarter.CompareTo(other.Quarter) > 0 || this.Month.CompareTo(other.Month) > 0;
            }

            public bool LessThan(FiscalPeriod other)
            {
return this.Quarter.CompareTo(other.Quarter) < 0 || this.Month.CompareTo(other.Month) < 0;
            }

            public override int GetHashCode()
            {
                unchecked
                {
                    return (this.Quarter.GetHashCode() * 397) ^ this.Month.GetHashCode();
                }
            }

            public static FiscalPeriod operator +(FiscalPeriod c1, FiscalPeriod c2)
            {
return new FiscalPeriod((c1.Quarter + c2.Quarter) > 4 ? (c1.Quarter + c2.Quarter)-4 : (c1.Quarter + c2.Quarter), (c1.Month + c2.Month) > 12 ? (c1.Month + c2.Month) - 12 : (c1.Month + c2.Month));
            }

            public static bool operator ==(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.Equals(c2);
            }

            public static bool operator !=(FiscalPeriod c1, FiscalPeriod c2)
            {
                return !c1.Equals(c2);
            }
            public static bool operator >(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.GreaterThan(c2);
            }
            public static bool operator <(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.LessThan(c2);
            }
            public override string ToString()
            {
                return (String.Format("Q{0}:P{1}", this.Quarter, this.Month));
            }

        }

        public class FiscalPeriodFormatter : IFormatter<FiscalPeriod>
        {
public void Serialize(FiscalPeriod instance, IColumnWriter writer, ISerializationContext context)
            {
                using (var binaryWriter = new BinaryWriter(writer.BaseStream))
                {
                    binaryWriter.Write(instance.Quarter);
                    binaryWriter.Write(instance.Month);
                    binaryWriter.Flush();
                }
            }

public FiscalPeriod Deserialize(IColumnReader reader, ISerializationContext context)
            {
                using (var binaryReader = new BinaryReader(reader.BaseStream))
                {
var result = new FiscalPeriod(binaryReader.ReadInt16(), binaryReader.ReadInt16());
                    return result;
                }
            }
        }
    }
}
```

## <a name="use-user-defined-aggregates-udagg"></a>Использование определяемых пользователем статистических функций
Определяемые пользователем статистические выражения — это любые функции статистической обработки, не входящие в U-SQL. Например, это может быть статистическое выражение для пользовательских математических вычислений, объединения строк, манипуляций с ними и т. д.

Определение базового класса определяемого пользователем статистического выражения представлено ниже.

```csharp
    [SqlUserDefinedAggregate]
    public abstract class IAggregate<T1, T2, TResult> : IAggregate
    {
        protected IAggregate();

        public abstract void Accumulate(T1 t1, T2 t2);
        public abstract void Init();
        public abstract TResult Terminate();
    }
```

**SqlUserDefinedAggregate** указывает, что тип должен быть зарегистрирован как определяемая пользователем статистическая функция. Этот класс не наследуется.

Атрибут SqlUserDefinedType является **необязательным** для определения функции.


Базовый класс позволяет передать три абстрактных параметра — два входных и один в качестве результата. Типы данных представлены переменными, которые должны быть определены при наследовании классов:

```csharp
public class GuidAggregate : IAggregate<string, string, string>
{
    string guid_agg;

    public override void Init()
    { … }

    public override void Accumulate(string guid, string user)
    { … }

    public override string Terminate()
    { … }
}
```

* **Init** — метод, который вызывается один раз для каждой группы во время вычисления. Он предоставляет процедуры инициализации для каждой группы статистической обработки.  
* **Accumulate** — метод, который выполняется один раз для каждого значения. Он выполняет основную работу алгоритма статистической обработки. Его можно использовать для обработки значений с разными типами данных, определяемыми при наследовании класса. Он может принимать два параметра разных типов данных.
* **Terminate** — метод, который выполняется один раз на каждую группу статистической обработки в конце обработки, чтобы вывести результаты по каждой группе.

Чтобы объявить правильные типы входных и выходных данных, используйте определение класса, приведенное ниже.

```csharp
public abstract class IAggregate<T1, T2, TResult> : IAggregate
```

* T1 — первый параметр для метода Accumulate.
* T2: второй параметр для накопления
* TResult — возвращаемый тип для метода Terminate.

Пример:

```csharp
public class GuidAggregate : IAggregate<string, int, int>
```

или диспетчер конфигурации служб

```csharp
public class GuidAggregate : IAggregate<string, string, string>
```

### <a name="use-udagg-in-u-sql"></a>Использование определяемых пользователем статистических выражений в U-SQL
Чтобы применить определяемое пользователем статистическое выражение, сначала определите его в коде программной части или укажите на него ссылку из существующих библиотек DLL для программирования, как описано выше.

Затем используйте приведенный ниже синтаксис.

```csharp
AGG<UDAGG_functionname>(param1,param2)
```

Ниже приведен пример определяемого пользователем статистического выражения.

```csharp
public class GuidAggregate : IAggregate<string, string, string>
{
    string guid_agg;

    public override void Init()
    {
        guid_agg = "";
    }

    public override void Accumulate(string guid, string user)
    {
        if (user.ToUpper()== "USER1")
        {
        guid_agg += "{" + guid + "}";
        }
    }

    public override string Terminate()
    {
        return guid_agg;
    }

}
```

И основной скрипт U-SQL.

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @" \usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid string,
        dt DateTime,
            user String,
            des String
    FROM @input_file 
    USING Extractors.Tsv();

@rs1 =
    SELECT
        user,
        AGG<USQL_Programmability.GuidAggregate>(guid,user) AS guid_list
    FROM @rs0
    GROUP BY user;

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```

В этом сценарии мы объединяем значения идентификаторов класса для определенных пользователей.

## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о программировании U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
* [Инструкции по программированию U-SQL — UDO](data-lake-analytics-u-sql-programmability-guide-UDO.md)

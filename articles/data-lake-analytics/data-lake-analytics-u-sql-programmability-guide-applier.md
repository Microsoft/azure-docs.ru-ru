---
title: Рекомендации по программированию для пользовательских средств применения U-SQL для Azure Data Lake
description: Ознакомьтесь с руководством по программированию U-SQL UDO — определяемое пользователем средство применения.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 0842a2cfa021ef8ea45c19ec885c7dec371730de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96512643"
---
# <a name="use-user-defined-applier"></a>Использование определяемого пользователем средства применения 

## <a name="u-sql-udo-user-defined-applier"></a>U-SQL UDO: определяемое пользователем средство применения
Пользовательское средство применения U-SQL позволяет вызывать пользовательскую функцию C# для каждой строки, возвращаемой выражением запроса к внешней таблице. Входные данные справа оцениваются для каждой строки из входных данных слева, а затем созданные строки объединяются для итоговых выходных данных. Список столбцов, созданных оператором APPLY, — это сочетание набора столбцов из входных данных слева и справа.


## <a name="how-to-define-and-use-user-defined-applier"></a>Определение и использование определяемого пользователем средства применения
Пользовательские средства применения вызываются как часть выражения USQL SELECT.

Типичный вызов пользовательских средств применения выглядит следующим образом:

```usql
SELECT …
FROM …
CROSS APPLYis used to pass parameters
new MyScript.MyApplier(param1, param2) AS alias(output_param1 string, …);
```

Дополнительные сведения об использовании средств применения в выражении SELECT см. в статье [U-SQL SELECT, выбрав из CROSS APPLY и OUTER APPLY](/u-sql/statements-and-expressions/select/from/select-selecting-from-cross-apply-and-outer-apply).

Ниже представлено определение базового класса пользовательского средства применения.

```csharp
public abstract class IApplier : IUserDefinedOperator
{
protected IApplier();

public abstract IEnumerable<IRow> Apply(IRow input, IUpdatableRow output);
}
```

Чтобы определить пользовательское средство применения, нам необходимо создать интерфейс `IApplier` с атрибутом [`SqlUserDefinedApplier`], который не является обязательным для определения пользовательских средств применения.

```csharp
[SqlUserDefinedApplier]
public class ParserApplier : IApplier
{
    public ParserApplier()
    {
        …
    }

    public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
    {
        …
    }
}
```

* Операция применения вызывается для каждой строки внешней таблицы. Она возвращает выходной набор строк `IUpdatableRow`.
* Класс Constructor используется для передачи параметров в пользовательское средство применения.

**SqlUserDefinedApplier** указывает, что тип должен быть зарегистрирован как пользовательское средство применения. Этот класс не наследуется.

Атрибут **SqlUserDefinedApplier** является **необязательным** для определения пользовательского средства применения.


Ниже представлены основные объекты, поддерживающие программирование.

```csharp
public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
```

Входной набор строк передается в виде ввода `IRow`. Выходные строки создаются как интерфейс вывода `IUpdatableRow`.

Имена отдельных столбцов можно узнать, вызвав метод схемы `IRow`.

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Чтобы получить фактические значения данных из входящего метода `IRow`, мы используем метод Get() интерфейса `IRow`.

```csharp
mycolumn = row.Get<int>("mycolumn")
```

Или мы можем использовать имя столбца схемы:

```csharp
row.Get<int>(row.Schema[0].Name)
```

Выходные значения следует передавать в выход `IUpdatableRow`:

```csharp
output.Set<int>("mycolumn", mycolumn)
```

Важно понимать, что пользовательские средства применения только выводят столбцы и значения, определенные вызовом метода `output.Set`.

Фактический вывод данных из средства применения инициируется вызовом метода `yield return output.AsReadOnly();`.

Параметры пользовательских средств применения можно передать в конструктор. Средство применения может возвращать переменное число столбцов, которое должно быть определено при вызове средства применения из основного скрипта U-SQL.

```csharp
new USQL_Programmability.ParserApplier ("all") AS properties(make string, model string, year string, type string, millage int);
```

Ниже приведен пример использования пользовательского средства применения.

```csharp
[SqlUserDefinedApplier]
public class ParserApplier : IApplier
{
private string parsingPart;

public ParserApplier(string parsingPart)
{
    if (parsingPart.ToUpper().Contains("ALL")
    || parsingPart.ToUpper().Contains("MAKE")
    || parsingPart.ToUpper().Contains("MODEL")
    || parsingPart.ToUpper().Contains("YEAR")
    || parsingPart.ToUpper().Contains("TYPE")
    || parsingPart.ToUpper().Contains("MILLAGE")
    )
    {
    this.parsingPart = parsingPart;
    }
    else
    {
    throw new ArgumentException("Incorrect parameter. Please use: 'ALL[MAKE|MODEL|TYPE|MILLAGE]'");
    }
}

public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
{

    string[] properties = input.Get<string>("properties").Split(',');

    //  only process with correct number of properties
    if (properties.Count() == 5)
    {

    string make = properties[0];
    string model = properties[1];
    string year = properties[2];
    string type = properties[3];
    int millage = -1;

    // Only return millage if it is number, otherwise, -1
    if (!int.TryParse(properties[4], out millage))
    {
        millage = -1;
    }

    if (parsingPart.ToUpper().Contains("MAKE") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("make", make);
    if (parsingPart.ToUpper().Contains("MODEL") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("model", model);
    if (parsingPart.ToUpper().Contains("YEAR") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("year", year);
    if (parsingPart.ToUpper().Contains("TYPE") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("type", type);
    if (parsingPart.ToUpper().Contains("MILLAGE") || parsingPart.ToUpper().Contains("ALL")) output.Set<int>("millage", millage);
    }
    yield return output.AsReadOnly();            
}
}
```

И основной скрипт U-SQL для этого пользовательского средства применения:

```usql
DECLARE @input_file string = @"c:\usql-programmability\car_fleet.tsv";
DECLARE @output_file string = @"c:\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        stocknumber int,
        vin String,
        properties String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
    SELECT
        r.stocknumber,
        r.vin,
        properties.make,
        properties.model,
        properties.year,
        properties.type,
        properties.millage
    FROM @rs0 AS r
    CROSS APPLY
    new USQL_Programmability.ParserApplier ("all") AS properties(make string, model string, year string, type string, millage int);

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```

В этом сценарии пользовательское средство применения действует как анализатор значений с разделителями-запятыми для набора свойств автомобильного парка. Строки входного файла будут выглядеть так, как показано ниже.

```text
103 Z1AB2CD123XY45889   Ford,Explorer,2005,SUV,152345
303 Y0AB2CD34XY458890   Chevrolet,Cruise,2010,4Dr,32455
210 X5AB2CD45XY458893   Nissan,Altima,2011,4Dr,74000
```

Это стандартный TSV-файл со значениями с разделением знаками табуляции, в котором столбец свойств содержит такие свойства автомобилей, как производитель, модель и т. д. Эти свойства нужно проанализировать и преобразовать в столбцы таблицы. Предоставленное средство применения также позволяет создать динамическое число свойств в результирующем наборе строк на основе переданных параметров. Вы можете создать либо все свойства, либо определенный набор свойств.

```text
...USQL_Programmability.ParserApplier ("all")
...USQL_Programmability.ParserApplier ("make")
...USQL_Programmability.ParserApplier ("make&model")
```

Пользовательское средство применения может вызываться как новый экземпляр средства применения.

```usql
CROSS APPLY new MyNameSpace.MyApplier (parameter: "value") AS alias([columns types]…);
```

Или с вызовом метода фабрики оболочки.

```csharp
    CROSS APPLY MyNameSpace.MyApplier (parameter: "value") AS alias([columns types]…);
```


## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о программировании U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
* [Инструкции по программированию U-SQL — определяемый пользователем тип и функции](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
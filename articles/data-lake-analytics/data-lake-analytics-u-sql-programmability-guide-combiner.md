---
title: Рекомендации по программированию для пользовательских средств объединения U-SQL для Azure Data Lake
description: Ознакомьтесь с руководством по программированию U-SQL UDO — определяемое пользователем средство объединения.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: a6c560cf4ec11197183711656d69024591e7008c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96512696"
---
# <a name="use-user-defined-combiner"></a>Использование определяемого пользователем средства объединения

## <a name="u-sql-udo-user-defined-combiner"></a>UDO U-SQL: определяемый пользователем средство объединения
Пользовательское средство объединения позволяет объединить строки из левого и правого наборов строк, используя настраиваемую логику объединения. Пользовательское средство объединения используется с выражением COMBINE.

## <a name="how-to-define-and-use-user-defined-combiner"></a>Определение и использование определяемого пользователем средства объединения

Средство объединения вызывается из выражения COMBINE, которое предоставляет необходимые сведения о входных наборах строк, столбцах для группирования, схеме ожидаемого результата, а также дополнительные сведения.

Чтобы вызвать средство объединения из основного скрипта U-SQL, используйте следующий синтаксис:

```usql
Combine_Expression :=
    'COMBINE' Combine_Input
    'WITH' Combine_Input
    Join_On_Clause
    Produce_Clause
    [Readonly_Clause]
    [Required_Clause]
    USING_Clause.
```

Дополнительные сведения см. в [описании выражения COMBINE для U-SQL](/u-sql/statements-and-expressions/combine-expression).

Чтобы задать пользовательское средство объединения, нам необходимо создать интерфейс `ICombiner` с атрибутом [`SqlUserDefinedCombiner`], который не является обязательным для определения пользовательских средств объединения.

Определение базового класса `ICombiner`.

```csharp
public abstract class ICombiner : IUserDefinedOperator
{
protected ICombiner();
public virtual IEnumerable<IRow> Combine(List<IRowset> inputs,
       IUpdatableRow output);
public abstract IEnumerable<IRow> Combine(IRowset left, IRowset right,
       IUpdatableRow output);
}
```

Пользовательская реализация интерфейса `ICombiner` должна содержать определение для переопределения метода `IEnumerable<IRow>` Combine.

```csharp
[SqlUserDefinedCombiner]
public class MyCombiner : ICombiner
{

public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
    IUpdatableRow output)
{
    …
}
}
```

Атрибут **SqlUserDefinedCombiner** указывает, что этот тип должен быть зарегистрирован как пользовательское средство объединения. Этот класс не наследуется.

Метод **SqlUserDefinedCombiner** используется для определения свойства режима объединения. Он является необязательным атрибутом для определения пользовательских средств объединения.

Режимы CombinerMode

Перечисление CombinerMode может принимать следующие значения:

* Full (0). Каждая строка выходных данных может зависеть от всех входных строк слева и справа с одинаковым значением ключа.

* Left (1). Каждая строка выходных данных зависит от одной входной строки из левого входа (и потенциально от всех строк из правого входа с тем же значением ключа).

* Right (2) — каждая строка выходных данных зависит от одной входной строки из правого входа (и потенциально от всех строк из левого входа с тем же значением ключа).

* Inner (3). Каждая строка выходных данных зависит от одной строки из левого входа и одной строки из правого входа с тем же значением ключа.

Пример: [`SqlUserDefinedCombiner(Mode=CombinerMode.Left)`]


Ниже представлены основные объекты, поддерживающие программирование.

```csharp
    public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
        IUpdatableRow output
```

Входные наборы строк передаются в виде левого (**left**) и правого (**right) интерфейсов типа** `IRowset`. Они должны быть перечислены для обработки. Каждый интерфейс можно перечислить только один раз, поэтому для повторной обработки их нужно перечислять с использованием кэша.

В целях кэширования в \<T\> качестве результата выполнения запроса LINQ можно создать тип списка структуры памяти, а именно<`IRow`>. Также для перечисления можно использовать анонимные типы данных.

Дополнительные сведения о запросах LINQ см. [в разделе Введение в запросы LINQ (C#)](/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries) , а [ \<T\> интерфейс IEnumerable](/dotnet/api/system.collections.generic.ienumerable-1) — дополнительные сведения об \<T\> интерфейсе IEnumerable.

Чтобы получить фактические значения данных из входящего метода `IRowset`, мы используем метод Get() интерфейса `IRow`.

```csharp
mycolumn = row.Get<int>("mycolumn")
```

Имена отдельных столбцов можно узнать, вызвав метод схемы `IRow`.

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Или мы можем использовать имя столбца схемы:

```csharp
c# row.Get<int>(row.Schema[0].Name)
```

Общее перечисление с помощью LINQ выглядит следующим образом:

```csharp
var myRowset =
            (from row in left.Rows
                          select new
                          {
                              Mycolumn = row.Get<int>("mycolumn"),
                          }).ToList();
```

Перечислив оба набора строк, мы будем в цикле перебирать все строки, выбирая для каждой строки из левого набора все строки, которые соответствуют условию, заданному для средства объединения.

Выходные значения следует передавать в выход `IUpdatableRow`.

```csharp
output.Set<int>("mycolumn", mycolumn)
```

Фактический вывод данных инициируется вызовом метода `yield return output.AsReadOnly();`.

Ниже приведен пример средства объединения.

```csharp
[SqlUserDefinedCombiner]
public class CombineSales : ICombiner
{

public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
    IUpdatableRow output)
{
    var internetSales =
    (from row in left.Rows
          select new
          {
              ProductKey = row.Get<int>("ProductKey"),
              OrderDateKey = row.Get<int>("OrderDateKey"),
              SalesAmount = row.Get<decimal>("SalesAmount"),
              TaxAmt = row.Get<decimal>("TaxAmt")
          }).ToList();

    var resellerSales =
    (from row in right.Rows
     select new
     {
     ProductKey = row.Get<int>("ProductKey"),
     OrderDateKey = row.Get<int>("OrderDateKey"),
     SalesAmount = row.Get<decimal>("SalesAmount"),
     TaxAmt = row.Get<decimal>("TaxAmt")
     }).ToList();

    foreach (var row_i in internetSales)
    {
    foreach (var row_r in resellerSales)
    {

        if (
        row_i.OrderDateKey > 0
        && row_i.OrderDateKey < row_r.OrderDateKey
        && row_i.OrderDateKey == 20010701
        && (row_r.SalesAmount + row_r.TaxAmt) > 20000)
        {
        output.Set<int>("OrderDateKey", row_i.OrderDateKey);
        output.Set<int>("ProductKey", row_i.ProductKey);
        output.Set<decimal>("Internet_Sales_Amount", row_i.SalesAmount + row_i.TaxAmt);
        output.Set<decimal>("Reseller_Sales_Amount", row_r.SalesAmount + row_r.TaxAmt);
        }

    }
    }
    yield return output.AsReadOnly();
}
}
```

В этом сценарии использования мы создаем аналитический отчет для организации розничной торговли. Наша задача — найти все товары стоимостью более 20 000 долларов США, которые за определенный промежуток времени продавались через веб-сайт быстрее, чем через обычные точки розничной торговли.

Вот основной скрипт U-SQL: Вы можете сравнить логику обычной инструкции JOIN и средства объединения.

```sql
DECLARE @LocalURI string = @"\usql-programmability\";

DECLARE @input_file_internet_sales string = @LocalURI+"FactInternetSales.txt";
DECLARE @input_file_reseller_sales string = @LocalURI+"FactResellerSales.txt";
DECLARE @output_file1 string = @LocalURI+"output_file1.tsv";
DECLARE @output_file2 string = @LocalURI+"output_file2.tsv";

@fact_internet_sales =
EXTRACT
    ProductKey int ,
    OrderDateKey int ,
    DueDateKey int ,
    ShipDateKey int ,
    CustomerKey int ,
    PromotionKey int ,
    CurrencyKey int ,
    SalesTerritoryKey int ,
    SalesOrderNumber String ,
    SalesOrderLineNumber  int ,
    RevisionNumber int ,
    OrderQuantity int ,
    UnitPrice decimal ,
    ExtendedAmount decimal,
    UnitPriceDiscountPct float ,
    DiscountAmount float ,
    ProductStandardCost decimal ,
    TotalProductCost decimal ,
    SalesAmount decimal ,
    TaxAmt decimal ,
    Freight decimal ,
    CarrierTrackingNumber String,
    CustomerPONumber String
FROM @input_file_internet_sales
USING Extractors.Text(delimiter:'|', encoding: Encoding.Unicode);

@fact_reseller_sales =
EXTRACT
    ProductKey int ,
    OrderDateKey int ,
    DueDateKey int ,
    ShipDateKey int ,
    ResellerKey int ,
    EmployeeKey int ,
    PromotionKey int ,
    CurrencyKey int ,
    SalesTerritoryKey int ,
    SalesOrderNumber String ,
    SalesOrderLineNumber  int ,
    RevisionNumber int ,
    OrderQuantity int ,
    UnitPrice decimal ,
    ExtendedAmount decimal,
    UnitPriceDiscountPct float ,
    DiscountAmount float ,
    ProductStandardCost decimal ,
    TotalProductCost decimal ,
    SalesAmount decimal ,
    TaxAmt decimal ,
    Freight decimal ,
    CarrierTrackingNumber String,
    CustomerPONumber String
FROM @input_file_reseller_sales
USING Extractors.Text(delimiter:'|', encoding: Encoding.Unicode);

@rs1 =
SELECT
    fis.OrderDateKey,
    fis.ProductKey,
    fis.SalesAmount+fis.TaxAmt AS Internet_Sales_Amount,
    frs.SalesAmount+frs.TaxAmt AS Reseller_Sales_Amount
FROM @fact_internet_sales AS fis
     INNER JOIN @fact_reseller_sales AS frs
     ON fis.ProductKey == frs.ProductKey
WHERE
    fis.OrderDateKey < frs.OrderDateKey
    AND fis.OrderDateKey == 20010701
    AND frs.SalesAmount+frs.TaxAmt > 20000;

@rs2 =
COMBINE @fact_internet_sales AS fis
WITH @fact_reseller_sales AS frs
ON fis.ProductKey == frs.ProductKey
PRODUCE OrderDateKey int,
        ProductKey int,
        Internet_Sales_Amount decimal,
        Reseller_Sales_Amount decimal
USING new USQL_Programmability.CombineSales();

OUTPUT @rs1 TO @output_file1 USING Outputters.Tsv();
OUTPUT @rs2 TO @output_file2 USING Outputters.Tsv();
```

Пользовательское средство объединения может вызываться как новый экземпляр объекта применения.

```csharp
USING new MyNameSpace.MyCombiner();
```


Или с вызовом метода фабрики оболочки.

```csharp
USING MyNameSpace.MyCombiner();
```

## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о программировании U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
* [Инструкции по программированию U-SQL — определяемый пользователем тип и функции](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
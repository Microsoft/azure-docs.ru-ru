---
title: Date_Bucket (Transact-SQL) — Azure SQL ребро
description: Дополнительные сведения об использовании Date_Bucket в Azure SQL ребр
keywords: Date_Bucket, SQL Azure для пограничных вычислений
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 9d81419721e94a2e181f094c0e0e64b1b23544a8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93073525"
---
# <a name="date_bucket-transact-sql"></a>Date_Bucket (Transact-SQL)

Эта функция возвращает значение DateTime, соответствующее началу каждого контейнера DateTime, из метки времени, определенной `origin` параметром, или значения происхождения по умолчанию, `1900-01-01 00:00:00.000` Если параметр Origin не задан. 

Обзор всех типов данных и функций даты и времени в языке &#40;Transact-SQL&#41 см. в статье [Типы данных и функции даты и времени (Transact-SQL)](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql/).

[Синтаксические обозначения в Transact-SQL](/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql/)

## <a name="syntax"></a>Синтаксис

```sql
DATE_BUCKET (datePart, number, date, origin)
```

## <a name="arguments"></a>Аргументы

*datePart*

Часть *date* (дата), используемая с параметром "number". Например: Год, месяц, минута, секунда и т. д.

> [!NOTE]
> `DATE_BUCKET` не принимает эквивалентные переменные, определяемые пользователем, для аргументов *datepPart*.
  
|*datePart*|Сокращения|  
|---|---|
|**day**|**dd**, **d**|  
|**week**|**wk**, **ww**| 
|**month**|**mm**, **m**|
|**quarter**|**qq**, **q**|  
|**year**|**yy**, **yyyy**|  
|**hour**|**hh**|  
|**minute**|**mi**, **n**|  
|**second**|**ss**, **s**|  
|**millisecond**|**ms**|  

*number*

Целое число, которое определяет ширину контейнера в сочетании с аргументом *datePart*. Представляет ширину контейнеров dataPart из времени источника. **`This argument cannot be a negative integer value`**. 

*date*

Выражение, которое может быть разрешено в одно из следующих значений.

+ **date**
+ **datetime**
+ **datetimeoffset**
+ **datetime2**
+ **smalldatetime**
+ **time**

Для *date* (дата), `DATE_BUCKET` примет выражение столбца, выражение или пользовательскую переменную, если они разрешаются для любого из вышеперечисленных типов данных.

**Исходный домен** 

Необязательное выражение, которое может разрешаться в одно из следующих значений:

+ **date**
+ **datetime**
+ **datetimeoffset**
+ **datetime2**
+ **smalldatetime**
+ **time**

Тип данных для `Origin` должен соответствовать типу данных `Date` параметра. 

`DATE_BUCKET` использует значение даты источника по умолчанию `1900-01-01 00:00:00.000` , равное 12:00 AM в понедельник, 1 1900 января, если для функции не указано значение Origin.

## <a name="return-type"></a>Тип возвращаемых данных

Тип данных возвращаемого значения для этого метода является динамическим. Тип возвращаемого значения зависит от типа аргумента, переданного в `date`. Если для `date` предоставляется другой тип допустимых входных данных, `DATE_BUCKET` возвращает тот же тип данных. `DATE_BUCKET` вызывает ошибку, если для параметра `date` указан строковый литерал.

## <a name="return-values"></a>Возвращаемые значения

### <a name="understanding-the-output-from-date_bucket"></a>Основные сведения о выходных данных из `DATE_BUCKET`

`Date_Bucket` возвращает последнее значение даты или времени, соответствующее параметру "datePart" и "number". Например, в приведенных ниже выражениях `Date_Bucket` вернет выходное значение `2020-04-13 00:00:00.0000000`, так как выходные данные рассчитываются на основе временных сегментов по умолчанию для `1900-01-01 00:00:00.000`. Значение `2020-04-13 00:00:00.0000000` составляет 6276 недель от значения источника `1900-01-01 00:00:00.000`. 

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 1, @date)
```

Для всех приведенных ниже выражений будет возвращено одно и то же выходное значение `2020-04-13 00:00:00.0000000`. Это связано с тем, что `2020-04-13 00:00:00.0000000` составляет 6276 недель с даты начала и 6276 делится на 2, 3, 4 и 6.

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 2, @date)
Select DATE_BUCKET(wk, 3, @date)
Select DATE_BUCKET(wk, 4, @date)
Select DATE_BUCKET(wk, 6, @date)
```

Выходные данные для приведенного ниже выражения — это `2020-04-06 00:00:00.0000000` 6275 недель из времени происхождения по умолчанию `1900-01-01 00:00:00.000` .

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 5, @date)
```

Выходные данные для приведенного ниже выражения имеют значение `2020-06-09 00:00:00.0000000` , которое составляет 75 недель с указанного времени источника `2019-01-01 00:00:00` .

```sql
declare @date datetime2 = '2020-06-15 21:22:11'
declare @origin datetime2 = '2019-01-01 00:00:00'
Select DATE_BUCKET(wk, 5, @date, @origin)
```

## <a name="datepart-argument"></a>Аргумент datepart

Функции **dayofyear**, **day** и **weekday** возвращают одинаковое значение. Каждое выражение *datepart* и его краткие формы возвращают одно и то же значение.
  
## <a name="number-argument"></a>Аргумент number

Аргумент *number* не может выходить за диапазон положительных значений типа **int**. В приведенных ниже инструкциях аргумент *number* превышает диапазон типа данных **int** на 1. Следующая инструкция возвращает следующее сообщение об ошибке: "`Msg 8115, Level 16, State 2, Line 2. Arithmetic overflow error converting expression to data type int."`
  
```sql
declare @date datetime2 = '2020-04-30 00:00:00'
Select DATE_BUCKET(dd, 2147483648, @date)
```  

Если в функцию `Date_Bucket` передается отрицательное значение для "number", будет возвращена следующая ошибка: 

```sql
Msg 9834, Level 16, State 1, Line 1
Invalid bucket width value passed to date_bucket function. Only positive values are allowed.
````

## <a name="date-argument"></a>Аргумент date  

`DATE_BUCKET` возвращает базовое значение, соответствующее типу данных аргумента `date`. В следующем примере возвращается выходное значение с типом данных datetime2. 

```sql
Select DATE_BUCKET(dd, 10, SYSUTCDATETIME())
```

## <a name="origin-argument"></a>Аргумент источника  

Тип данных `origin` `date` аргументов и в должен быть одинаковым. Если используются разные типы данных, будет сформирована ошибка.

## <a name="remarks"></a>Remarks

Используйте `DATE_BUCKET` в следующих предложениях.

+ GROUP BY
+ HAVING
+ ORDER BY
+ SELECT \<list>
+ WHERE

## <a name="examples"></a>Примеры

### <a name="a-calculating-date_bucket-with-a-bucket-width-of-1-from-the-origin-time"></a>A. Вычисление Date_Bucket с шириной контейнера, равной 1, из времени источника

Каждая из этих инструкций увеличивает *date_bucket*, применяя ширину контейнера = 1, по сравнению с временем источника:

```sql
declare @date datetime2 = '2020-04-30 21:21:21'
Select 'Week',  DATE_BUCKET(wk, 1, @date)
Union All
Select 'Day',  DATE_BUCKET(dd, 1, @date)
Union All
Select 'Hour',  DATE_BUCKET(hh, 1, @date)
Union All
Select 'Minutes',  DATE_BUCKET(mi, 1, @date)
Union All
Select 'Seconds',  DATE_BUCKET(ss, 1, @date)
```

Результирующий набор:

```sql
Week    2020-04-27 00:00:00.0000000
Day     2020-04-30 00:00:00.0000000
Hour    2020-04-30 21:00:00.0000000
Minutes 2020-04-30 21:21:00.0000000
Seconds 2020-04-30 21:21:21.0000000
```

### <a name="b-using-expressions-as-arguments-for-the-number-and-date-parameters"></a>Б. Использование выражений в качестве аргументов number и date

В этих примерах выражения различного типа используются в качестве аргументов для параметров *number* и *date*. Эти примеры создаются с использованием базы данных "AdventureWorksDW2017".
  
#### <a name="specifying-user-defined-variables-as-number-and-date"></a>Указание пользовательских переменных в качестве аргументов number и date  

В этом примере в качестве аргументов *number* и *date* указываются пользовательские переменные.
  
```sql
DECLARE @days int = 365,
        @datetime datetime2 = '2000-01-01 01:01:01.1110000'; /* 2000 was a leap year */;  
SELECT Date_Bucket(day, @days, @datetime);
```

Результирующий набор:

```sql
---------------------------
1999-12-08 00:00:00.0000000

(1 row affected)
```  

#### <a name="specifying-a-column-as-date"></a>Указание столбца в качестве аргумента date

В примере, приведенном ниже, мы вычисляем сумму OrderQuantity и сумму UnitPrice, сгруппированные по контейнерам с датой за неделю.
  
```sql
SELECT
    Date_Bucket(week, 1 ,cast(Shipdate as datetime2)) AS ShippedDateBucket
    ,Sum(OrderQuantity)  As SumOrderQuantity
    ,Sum(UnitPrice) As SumUnitPrice
FROM dbo.FactInternetSales FIS
where Shipdate between '2011-01-03 00:00:00.000' and '2011-02-28 00:00:00.000'
Group by Date_Bucket(week, 1 ,cast(Shipdate as datetime2))
order by 1
```  

Результирующий набор:
  
```sql
ShippedDateBucket           SumOrderQuantity SumUnitPrice
--------------------------- ---------------- ---------------------
2011-01-03 00:00:00.0000000 21               65589.7546
2011-01-10 00:00:00.0000000 27               89938.5464
2011-01-17 00:00:00.0000000 31               104404.9064
2011-01-24 00:00:00.0000000 36               118525.6846
2011-01-31 00:00:00.0000000 39               123555.431
2011-02-07 00:00:00.0000000 35               109342.351
2011-02-14 00:00:00.0000000 32               107804.8964
2011-02-21 00:00:00.0000000 37               119456.3428
2011-02-28 00:00:00.0000000 9                28968.6982
```  

#### <a name="specifying-scalar-system-function-as-date"></a>Указание в качестве аргумента date скалярной системной функции

В этом примере для аргумента *date* указано значение `SYSDATETIME`. Точное возвращаемое значение зависит от дня и времени выполнения инструкции.
  
```sql
SELECT Date_Bucket(wk, 10, SYSDATETIME());  
```  

Результирующий набор:

```sql
---------------------------
2020-03-02 00:00:00.0000000

(1 row affected)
```  

#### <a name="specifying-scalar-subqueries-and-scalar-functions-as-number-and-date"></a>Указание в качестве аргументов number и date скалярных вложенных запросов и скалярных функций

В этом примере в качестве аргументов для *number* и *date* используются скалярные вложенные запросы `MAX(OrderDate)`. `(SELECT top 1 CustomerKey FROM dbo.DimCustomer where GeographyKey > 100)` является искусственным аргументом для числового параметра, показывающим способ выбора аргумента *number* из списка значений.
  
```sql
SELECT DATE_BUCKET(week,(SELECT top 1 CustomerKey FROM dbo.DimCustomer where GeographyKey > 100),  
    (SELECT MAX(OrderDate) FROM dbo.FactInternetSales));  
```  
  
#### <a name="specifying-numeric-expressions-and-scalar-system-functions-as-number-and-date"></a>Указание в качестве аргументов number и date числовых выражений и скалярных системных функций

В этом примере в качестве аргументов для number и date используется числовое выражение ((10/2)) и скалярные системные функции (SYSDATETIME).
  
```sql
SELECT Date_Bucket(week,(10/2), SYSDATETIME());
```

#### <a name="specifying-an-aggregate-window-function-as-number"></a>Указание в качестве аргумента number статистической оконной функции

В этом примере в качестве аргумента *number* используется агрегатная оконная функция.
  
```sql
Select 
    DISTINCT DATE_BUCKET(day, 30, Cast([shipdate] as datetime2)) as DateBucket,
    First_Value([SalesOrderNumber]) OVER (Order by DATE_BUCKET(day, 30, Cast([shipdate] as datetime2))) as First_Value_In_Bucket,
    Last_Value([SalesOrderNumber]) OVER (Order by DATE_BUCKET(day, 30, Cast([shipdate] as datetime2))) as Last_Value_In_Bucket
    from [dbo].[FactInternetSales]
Where ShipDate between '2011-01-03 00:00:00.000' and '2011-02-28 00:00:00.000'
order by DateBucket
GO  
``` 
### <a name="c-using-a-non-default-origin-value"></a>В. Использование значения источника, не являющегося источником по умолчанию

В этом примере используется значение оргин, отличное от значения по умолчанию, для создания сегментов даты. 

```sql
declare @date datetime2 = '2020-06-15 21:22:11'
declare @origin datetime2 = '2019-01-01 00:00:00'
Select DATE_BUCKET(hh, 2, @date, @origin)
```

## <a name="see-also"></a>См. также раздел

[Функции CAST и CONVERT (Transact-SQL)](/sql/t-sql/functions/cast-and-convert-transact-sql/)

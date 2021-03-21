---
title: Рекомендации по программированию для упрощения пользовательского интерфейса U-SQL для Azure Data Lake
description: Ознакомьтесь с руководством по программированию U-SQL UDO — определяемое пользователем сокращение.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 52d44685678c3e89dc820042a7925d284500cef8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96512631"
---
# <a name="use-user-defined-reducer"></a>Использование определяемого пользователем средства редукции

## <a name="u-sql-udo-user-defined-reducer"></a>U-SQL UDO: определяемый пользователем reduce

U-SQL позволяет создать пользовательские средства редукции для набора строк на языке C#, используя платформу расширения для определяемых пользователем операторов и реализацию интерфейса IReducer.

Определяемые пользователем редукторы можно использовать для исключения ненужных строк в процессе извлечения (импорта) данных. Также с помощью этого средства можно обрабатывать и оценивать строки и столбцы. Оно позволяет выбрать извлекаемые строки с использованием программируемой логики.

## <a name="how-to-define-and-use-user-defined-reducer"></a>Определение и использование определяемого пользователем сокращения
Чтобы определить класс пользовательского средства редукции, нужно создать интерфейс `IReducer` с необязательным атрибутом `SqlUserDefinedReducer`.

Этот интерфейс класса должен содержать определение, переопределяющее интерфейс объекта `IEnumerable` для набора строк.

```csharp
[SqlUserDefinedReducer]
public class EmptyUserReducer : IReducer
{

    public override IEnumerable<IRow> Reduce(IRowset input, IUpdatableRow output)
    {
        …
    }

}
```

Атрибут **SqlUserDefinedReducer** указывает, что этот тип должен быть зарегистрирован как пользовательское средство редукции. Этот класс не наследуется.
**SqlUserDefinedReducer** — это необязательный атрибут для определения пользовательских средств редукции. Он используется для определения свойства IsRecursive.

* bool IsRecursive.    
* Значение **true** указывает, является ли это средство редукции ассоциативным и коммуникативным.

Основные объекты, поддерживающие программирование, — это **input** (ввод) и **output** (вывод). Объект input используется для перечисления входных строк, а output — для вывода строк, полученных в результате действий по редукции.

Для перечисления входных строк мы используем метод `Row.Get`.

```csharp
foreach (IRow row in input.Rows)
{
    row.Get<string>("mycolumn");
}
```

Параметр для метода `Row.Get` — это столбец, который передается как часть класса `PRODUCE` инструкции `REDUCE` в основном скрипте U-SQL. Здесь необходимо использовать правильный тип данных.

Для выходных данных используйте метод `output.Set`.

Важно понимать, что пользовательские средства редукции возвращают только те выходные значения, которые определены в вызове метода `output.Set`.

```csharp
output.Set<string>("mycolumn", guid);
```

Фактический вывод данных из средства редукции инициируется вызовом метода `yield return output.AsReadOnly();`.

Ниже приведен пример средства редукции.

```csharp
[SqlUserDefinedReducer]
public class EmptyUserReducer : IReducer
{

    public override IEnumerable<IRow> Reduce(IRowset input, IUpdatableRow output)
    {
    string guid;
    DateTime dt;
    string user;
    string des;

    foreach (IRow row in input.Rows)
    {
        guid = row.Get<string>("guid");
        dt = row.Get<DateTime>("dt");
        user = row.Get<string>("user");
        des = row.Get<string>("des");

        if (user.Length > 0)
        {
        output.Set<string>("guid", guid);
        output.Set<DateTime>("dt", dt);
        output.Set<string>("user", user);
        output.Set<string>("des", des);

        yield return output.AsReadOnly();
        }
    }
    }

}
```

В этом сценарии использования средство редукции пропускает строки с пустым именем пользователя. Для каждой строки в наборе строк считывается каждый обязательный столбец, затем оценивается длина имени пользователя. Текущую строку средство передает на выход, только если длина имени пользователя превышает 0.

Ниже представлен основной скрипт U-SQL, который использует пользовательское средство редукции.

```usql
DECLARE @input_file string = @"\usql-programmability\input_file_reducer.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid string,
        dt DateTime,
            user String,
            des String
    FROM @input_file 
    USING Extractors.Tsv();

@rs1 =
    REDUCE @rs0 PRESORT guid
    ON guid  
    PRODUCE guid string, dt DateTime, user String, des String
    USING new USQL_Programmability.EmptyUserReducer();

@rs2 =
    SELECT guid AS start_id,
           dt AS start_time,
           DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
           USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).ToString() AS start_fiscalperiod,
           user,
           des
    FROM @rs1;

OUTPUT @rs2 
    TO @output_file 
    USING Outputters.Text();
```

## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о программировании U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
* [Инструкции по программированию U-SQL — определяемый пользователем тип и функции](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
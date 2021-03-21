---
title: Рекомендации по программированию пользовательских процессоров U-SQL для Azure Data Lake
description: Ознакомьтесь с руководством по программированию U-SQL UDO — определяемый пользователем процессор.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 6ff45c577e94a8c63bd7cb1e6603e4d5519af5c6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96512691"
---
# <a name="use-user-defined-processor"></a>Использование определяемого пользователем средства обработки

## <a name="u-sql-udo-user-defined-processor"></a>U-SQL UDO: определяемый пользователем обработчик
Пользовательское средство обработки — это тип определяемого пользователем объекта U-SQL, который позволяет обрабатывать входящие строки, применяя к ним средства программирования. Оно позволяет объединять столбцы, изменять значения или добавлять новые столбцы при необходимости. По сути оно позволяет получить нужные элементы данных, обрабатывая наборы строк.

## <a name="how-to-define-and-use-user-defined-processor"></a>Определение и использование определяемого пользователем процессора
Чтобы определить пользовательское средство обработки, нужно создать интерфейс `IProcessor` с атрибутом `SqlUserDefinedProcessor`, который является необязательным для этого средства.

Этот интерфейс должен содержать определение для переопределения набора строк интерфейса `IRow`, как показано в примере ниже.

```csharp
[SqlUserDefinedProcessor]
public class MyProcessor: IProcessor
{
public override IRow Process(IRow input, IUpdatableRow output)
 {
        …
 }
}
```

**SqlUserDefinedProcessor** указывает, что тип должен быть зарегистрирован как пользовательское средство обработки. Этот класс не наследуется.

Атрибут SqlUserDefinedProcessor является **необязательным** для определения пользовательского средства обработки.

Основные объекты, поддерживающие программирование, — это **input** (ввод) и **output** (вывод). Объект input используется для перечисления входных столбцов, а объект output — для вывода данных, полученных в результате действий по обработке.

Для перечисления входных столбцов мы используем метод `input.Get`.

```csharp
string column_name = input.Get<string>("column_name");
```

Параметр для метода `input.Get` — это столбец, представленный в предложении `PRODUCE` для инструкции `PROCESS` в основном скрипте U-SQL. Здесь необходимо использовать правильный тип данных.

Для выходных данных используйте метод `output.Set`.

Необходимо понимать, что пользовательское средство только выводит столбцы и значения, определяемые вызовом метода `output.Set`.

```csharp
output.Set<string>("mycolumn", mycolumn);
```

Фактический вывод данных из средства обработки инициируется вызовом метода `return output.AsReadOnly();`.

Ниже приведен пример использования средства обработки.

```csharp
[SqlUserDefinedProcessor]
public class FullDescriptionProcessor : IProcessor
{
public override IRow Process(IRow input, IUpdatableRow output)
 {
     string user = input.Get<string>("user");
     string des = input.Get<string>("des");
     string full_description = user.ToUpper() + "=>" + des;
     output.Set<string>("dt", input.Get<string>("dt"));
     output.Set<string>("full_description", full_description);
     output.Set<Guid>("new_guid", Guid.NewGuid());
     output.Set<Guid>("guid", input.Get<Guid>("guid"));
     return output.AsReadOnly();
 }
}
```

В этом сценарии использования средство обработки создает новый столбец с именем full_description путем объединения существующих столбцов — user в верхнем регистре и des. Также оно заново создает идентификатор GUID и возвращает его исходное и новое значения.

Как видно из примера выше, вы можете вызывать метод C# во время вызова метода `output.Set`.

Ниже приведен пример основного скрипта U-SQL, использующего пользовательское средство обработки.

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid Guid,
        dt String,
            user String,
            des String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
     PROCESS @rs0
     PRODUCE dt String,
             full_description String,
             guid Guid,
             new_guid Guid
     USING new USQL_Programmability.FullDescriptionProcessor();

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```


## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о программировании U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
* [Инструкции по программированию U-SQL — определяемый пользователем тип и функции](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
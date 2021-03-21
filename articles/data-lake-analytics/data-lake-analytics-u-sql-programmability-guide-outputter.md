---
title: Пользовательская программа предопределения программного средства вывода U-SQL для Azure Data Lake
description: Ознакомьтесь с руководством по программированию U-SQL UDO, определяемым пользователем.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 56b104b5cc8f8923445455c71fe2418e39539b8e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96512636"
---
# <a name="use-user-defined-outputter"></a>Использование определяемого пользователем средства вывода

## <a name="u-sql-udo-user-defined-outputter"></a>U-SQL UDO: определяемый пользователем средство вывода
Пользовательское средство вывода — это еще один определяемый пользователем объект U-SQL, который позволяет расширить встроенные возможности U-SQL. Как и средства извлечения, существует несколько встроенных средств вывода.

* *Outputters.Text()*: записывает данные в текстовые файлы с разделителями в разных кодировках.
* *Outputters.Csv()*: записывает данные в файлы с разделителями-запятыми (CSV) в разных кодировках.
* *Outputters.Tsv()*: записывает данные в файлы со значениями с разделением знаками табуляции (TSV) в разных кодировках.

Пользовательские средства вывода позволяют записывать данные в определяемом пользователем формате. Это может потребоваться для выполнения следующих задач:

* Запись данных в частично структурированные или неструктурированные файлы.
* Запись данных в неподдерживаемых кодировках.
* Преобразование выходных данных или добавление пользовательских атрибутов.

## <a name="how-to-define-and-use-user-defined-outputter"></a>Определение и использование определяемого пользователем средства вывода
Чтобы определить пользовательское средство вывода, нужно создать интерфейс `IOutputter`.

Ниже приведена базовая реализация класса `IOutputter`.

```csharp
public abstract class IOutputter : IUserDefinedOperator
{
    protected IOutputter();

    public virtual void Close();
    public abstract void Output(IRow input, IUnstructuredWriter output);
}
```

Все входные параметры средства вывода, такие как разделители столбцов и строк, кодировки и т. д., должны быть определены в конструкторе класса. Интерфейс `IOutputter` также должен содержать определение для переопределения `void Output`. При необходимости для атомарной обработки файлов можно задать атрибут `[SqlUserDefinedOutputter(AtomicFileProcessing = true)`. Дополнительные сведения см. ниже.

```csharp
[SqlUserDefinedOutputter(AtomicFileProcessing = true)]
public class MyOutputter : IOutputter
{

    public MyOutputter(myparam1, myparam2)
    {
      …
    }

    public override void Close()
    {
      …
    }

    public override void Output(IRow row, IUnstructuredWriter output)
    {
      …
    }
}
```

* Метод `Output` вызывается для каждой входной строки. Он возвращает набор строк `IUnstructuredWriter output`.
* Класс Constructor используется для передачи параметров в пользовательское средство вывода.
* Метод `Close` при необходимости можно переопределить, чтобы выходить из ресурсозатратных состояний или узнавать время записи последней строки.

Атрибут **SqlUserDefinedAggregate** указывает, что этот тип должен быть зарегистрирован как пользовательское средство вывода. Этот класс не наследуется.

SqlUserDefinedOutputter — это необязательный атрибут для определения пользовательских средств вывода. Он используется для определения свойства AtomicFileProcessing.

* bool AtomicFileProcessing.   

* **true** — указывает, что это средство вывода требует атомарные выходные файлы (JSON, XML и другие).
* **false** — указывает, что это средство вывода умеет работать с разделенными или распределенными файлами (CSV, SEQ и другие).

Основные объекты, поддерживающие программирование, — это **row** (строка) и **output** (вывод). Объект **row** используется для перечисления выходных данных в интерфейсе `IRow`, а объект **output** — для отправки выходных данных в конечный файл.

Обращение к выходным данным выполняется через интерфейс `IRow`. Выходные данные передаются построчно.

Отдельные значения перечисляются с помощью вызова метода Get интерфейса IRow.

```csharp
row.Get<string>("column_name")
```

Имена отдельных столбцов можно определить с помощью вызова метода `row.Schema`.

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Такой подход позволяет создавать гибкие средства вывода для любой схемы метаданных.

Выходные данные записываются в файл с помощью `System.IO.StreamWriter`. Для параметра потока задано значение `output.BaseStream` в структуре `IUnstructuredWriter output`.

Обратите внимание, что после каждой итерации строк необходимо очищать буфер данных для файла. Кроме того, объект `StreamWriter` должен использоваться с включенным (по умолчанию) атрибутом Disposable и с ключевым словом **using**.

```csharp
using (StreamWriter streamWriter = new StreamWriter(output.BaseStream, this._encoding))
{
…
}
```

В противном случае — после каждой итерации вызывайте явным образом метод Flush(). Это описано в примере ниже.

### <a name="set-headers-and-footers-for-user-defined-outputter"></a>Установка колонтитулов для пользовательского средства вывода
Чтобы задать верхний колонтитул, используйте процедуру однократного выполнения.

```csharp
public override void Output(IRow row, IUnstructuredWriter output)
{
 …
if (isHeaderRow)
{
    …                
}

 …
if (isHeaderRow)
{
    isHeaderRow = false;
}
 …
}
}
```

Код в первом блоке `if (isHeaderRow)` выполняется только один раз.

Для нижнего колонтитула используйте ссылку на экземпляр `System.IO.Stream` объекта `output.BaseStream`. Запишите нижний колонтитул в метод Close() интерфейса `IOutputter`.  (Дополнительные сведения см. ниже.)

Ниже приведен пример пользовательского средства вывода.

```csharp
[SqlUserDefinedOutputter(AtomicFileProcessing = true)]
public class HTMLOutputter : IOutputter
{
    // Local variables initialization
    private string row_delimiter;
    private char col_delimiter;
    private bool isHeaderRow;
    private Encoding encoding;
    private bool IsTableHeader = true;
    private Stream g_writer;

    // Parameters definition            
    public HTMLOutputter(bool isHeader = false, Encoding encoding = null)
    {
    this.isHeaderRow = isHeader;
    this.encoding = ((encoding == null) ? Encoding.UTF8 : encoding);
    }

    // The Close method is used to write the footer to the file. It's executed only once, after all rows
    public override void Close()
    {
    //Reference to IO.Stream object - g_writer
    StreamWriter streamWriter = new StreamWriter(g_writer, this.encoding);
    streamWriter.Write("</table>");
    streamWriter.Flush();
    streamWriter.Close();
    }

    public override void Output(IRow row, IUnstructuredWriter output)
    {
    System.IO.StreamWriter streamWriter = new StreamWriter(output.BaseStream, this.encoding);

    // Metadata schema initialization to enumerate column names
    ISchema schema = row.Schema;

    // This is a data-independent header--HTML table definition
    if (IsTableHeader)
    {
        streamWriter.Write("<table border=1>");
        IsTableHeader = false;
    }

    // HTML table attributes
    string header_wrapper_on = "<th>";
    string header_wrapper_off = "</th>";
    string data_wrapper_on = "<td>";
    string data_wrapper_off = "</td>";

    // Header row output--runs only once
    if (isHeaderRow)
    {
        streamWriter.Write("<tr>");
        for (int i = 0; i < schema.Count(); i++)
        {
        var col = schema[i];
        streamWriter.Write(header_wrapper_on + col.Name + header_wrapper_off);
        }
        streamWriter.Write("</tr>");
    }

    // Data row output
    streamWriter.Write("<tr>");                
    for (int i = 0; i < schema.Count(); i++)
    {
        var col = schema[i];
        string val = "";
        try
        {
        // Data type enumeration--required to match the distinct list of types from OUTPUT statement
        switch (col.Type.Name.ToString().ToLower())
        {
            case "string": val = row.Get<string>(col.Name).ToString(); break;
            case "guid": val = row.Get<Guid>(col.Name).ToString(); break;
            default: break;
        }
        }
        // Handling NULL values--keeping them empty
        catch (System.NullReferenceException)
        {
        }
        streamWriter.Write(data_wrapper_on + val + data_wrapper_off);
    }
    streamWriter.Write("</tr>");

    if (isHeaderRow)
    {
        isHeaderRow = false;
    }
    // Reference to the instance of the IO.Stream object for footer generation
    g_writer = output.BaseStream;
    streamWriter.Flush();
    }
}

// Define the factory classes
public static class Factory
{
    public static HTMLOutputter HTMLOutputter(bool isHeader = false, Encoding encoding = null)
    {
    return new HTMLOutputter(isHeader, encoding);
    }
}
```

И основной скрипт U-SQL.

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.html";

@rs0 =
    EXTRACT
            guid Guid,
        dt String,
            user String,
            des String
         FROM @input_file
         USING new USQL_Programmability.FullDescriptionExtractor(Encoding.UTF8);

OUTPUT @rs0 
    TO @output_file 
    USING new USQL_Programmability.HTMLOutputter(isHeader: true);
```

Это средство вывода для формата HTML. Оно создает HTML-файл с данными таблицы.

### <a name="call-outputter-from-u-sql-base-script"></a>Вызов средства вывода из основного скрипта U-SQL
Чтобы вызвать пользовательское средство вывода из основного скрипта U-SQL, нужно создать новый экземпляр объекта для средства вывода.

```sql
OUTPUT @rs0 TO @output_file USING new USQL_Programmability.HTMLOutputter(isHeader: true);
```

Чтобы не создавать экземпляр объекта в основном скрипте, мы можем создать оболочку функции, как было показано в примере выше.

```csharp
        // Define the factory classes
        public static class Factory
        {
            public static HTMLOutputter HTMLOutputter(bool isHeader = false, Encoding encoding = null)
            {
                return new HTMLOutputter(isHeader, encoding);
            }
        }
```

В этом случае исходный вызов будет выглядеть так:

```usql
OUTPUT @rs0 
TO @output_file 
USING USQL_Programmability.Factory.HTMLOutputter(isHeader: true);
```

## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о программировании U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
* [Инструкции по программированию U-SQL — определяемый пользователем тип и функции](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
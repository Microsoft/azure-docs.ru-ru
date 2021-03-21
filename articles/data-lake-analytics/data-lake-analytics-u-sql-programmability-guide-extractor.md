---
title: Программное средство извлечения, определяемое пользователем U-SQL, для Azure Data Lake
description: Ознакомьтесь с руководством по программированию U-SQL UDO — определяемое пользователем средство извлечения.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: ad7f6336753903533771033de21aec8262425a61
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97608023"
---
# <a name="use-user-defined-extractor"></a>Использование определяемого пользователем средства извлечения

## <a name="u-sql-udo-user-defined-extractor"></a>U-SQL UDO: определяемый пользователем средство извлечения
U-SQL позволяет импортировать внешние данные с помощью инструкции EXTRACT. Инструкция EXTRACT может использовать встроенные средства извлечения для определяемых пользователем объектов.  

* *Extractors.Text()*: извлекает данные из текстовых файлов с разделителями в разных кодировках.

* *Extractors.Csv()*: извлекает данные из файлов данных с разделителями-запятыми (CSV) в разных кодировках.

* *Extractors.Csv()*: извлекает данные из файлов со значением с разделением знаками табуляции (TSV), в разных кодировках.

Можно также разработать пользовательское средство извлечения. Например, если при импорте данных нам нужно выполнить одно из следующих действий:

* Изменить входные данные, разделяя столбцы или изменяя отдельные значения. Для объединения строк лучше всего использовать PROCESSOR.
* Выполнить синтаксический анализ неструктурированных данных, например веб-страниц или сообщений электронной почты, или частично структурированных данных, например документов XML или JSON.
* Выполнить анализ данных в неподдерживаемых кодировках.

## <a name="how-to-define-and-use-user-defined-extractor"></a>Определение и использование определяемого пользователем средства извлечения
Чтобы определить пользовательское средство извлечения, нужно создать интерфейс `IExtractor`. Все входные параметры средства извлечения, такие как разделители столбцов и строк, кодировки, должны быть определены в конструкторе класса. Интерфейс `IExtractor` также должен содержать определение для переопределения `IEnumerable<IRow>`:

```csharp
[SqlUserDefinedExtractor]
public class SampleExtractor : IExtractor
{
     public SampleExtractor(string row_delimiter, char col_delimiter)
     { … }

     public override IEnumerable<IRow> Extract(IUnstructuredReader input, IUpdatableRow output)
     { … }
}
```

Атрибут **SqlUserDefinedExtractor** указывает, что этот тип должен быть зарегистрирован как пользовательское средство извлечения. Этот класс не наследуется.

SqlUserDefinedExtractor — это необязательный атрибут для определения пользовательского средства извлечения. Он используется для определения свойства AtomicFileProcessing:

* bool AtomicFileProcessing.   

* Значение **true** указывает, что это средство извлечения требует атомарные входные файлы (JSON, XML и другие).
* Значение **false** указывает, что это средство извлечения умеет работать с разделенными или распределенными файлами (CSV, SEQ и другие).

Основные объекты в пользовательских объектах, поддерживающих программирование, — это **input** (ввод) и **output** (вывод). Объект input используется для перечисления входных данных как `IUnstructuredReader`, объект output — для вывода данных, полученных в результате действий по извлечению.

Получить доступ к входным данным можно через `System.IO.Stream` и `System.IO.StreamReader`.

Чтобы перечислить входные столбцы, необходимо сначала разбить входной поток, используя разделитель строк.

```csharp
foreach (Stream current in input.Split(my_row_delimiter))
{
…
}
```

Затем еще раз разбиваем входную строку на столбцы:

```csharp
foreach (Stream current in input.Split(my_row_delimiter))
{
…
    string[] parts = line.Split(my_column_delimiter);
    foreach (string part in parts)
    { … }
}
```

Чтобы сохранить выходные данные, мы используем метод `output.Set`.

Необходимо понимать, что пользовательское средство извлечения только выводит столбцы и значения, определяемые выходными данными. Вызов метода Set:

```csharp
output.Set<string>(count, part);
```

Фактический вывод данных из средства извлечения инициируется вызовом метода `yield return output.AsReadOnly();`.

Ниже приведен пример средства извлечения.

```csharp
[SqlUserDefinedExtractor(AtomicFileProcessing = true)]
public class FullDescriptionExtractor : IExtractor
{
     private Encoding _encoding;
     private byte[] _row_delim;
     private char _col_delim;

    public FullDescriptionExtractor(Encoding encoding, string row_delim = "\r\n", char col_delim = '\t')
    {
         this._encoding = ((encoding == null) ? Encoding.UTF8 : encoding);
         this._row_delim = this._encoding.GetBytes(row_delim);
         this._col_delim = col_delim;

    }

    public override IEnumerable<IRow> Extract(IUnstructuredReader input, IUpdatableRow output)
    {
         string line;
         //Read the input line by line
         foreach (Stream current in input.Split(_encoding.GetBytes("\r\n")))
         {
        using (System.IO.StreamReader streamReader = new StreamReader(current, this._encoding))
         {
             line = streamReader.ReadToEnd().Trim();
             //Split the input by the column delimiter
             string[] parts = line.Split(this._col_delim);
             int count = 0; // start with first column
             foreach (string part in parts)
             {
    if (count == 0)
             {  // for column “guid”, re-generated guid
                 Guid new_guid = Guid.NewGuid();
                 output.Set<Guid>(count, new_guid);
             }
             else if (count == 2)
             {
                 // for column “user”, convert to UPPER case
                 output.Set<string>(count, part.ToUpper());

             }
             else
             {
                 // keep the rest of the columns as-is
                 output.Set<string>(count, part);
             }
             count += 1;
             }

         }
         yield return output.AsReadOnly();
         }
         yield break;
     }
}
```

В этом сценарии средство извлечения восстанавливает идентификатор для столбца guid и преобразует значения столбца user в верхний регистр. Пользовательские средства извлечения могут создавать более сложные результаты, анализируя входные данные и преобразовывая их.

Ниже представлен основной скрипт U-SQL, который использует пользовательское средство извлечения.

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        guid Guid,
        dt String,
        user String,
        des String
    FROM @input_file
        USING new USQL_Programmability.FullDescriptionExtractor(Encoding.UTF8);

OUTPUT @rs0 TO @output_file USING Outputters.Text();
```


## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о программировании U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
* [Инструкции по программированию U-SQL — определяемый пользователем тип и функции](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
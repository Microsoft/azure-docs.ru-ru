---
title: Функции и системные переменные фабрики данных
description: Содержит список функций и системных переменных фабрики данных Azure.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: d47a26b46fad1ab8238f9fa7a1183aad20530ef5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363939"
---
# <a name="azure-data-factory---functions-and-system-variables"></a>Фабрика данных Azure — функции и системные переменные
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [системных переменных в службе "Фабрика данных Azure"](../control-flow-system-variables.md).

В этой статье содержатся сведения о функциях и переменных, поддерживаемых фабрикой данных Azure.

## <a name="data-factory-system-variables"></a>Системные переменные фабрики данных

| Имя переменной | Описание | Область объекта | Область JSON и примеры использования |
| --- | --- | --- | --- |
| WindowStart |Начало интервала времени для текущего окна запуска действия. |activity |<ol><li>Укажите запросы на выбор данных. Ознакомьтесь со статьями о соединителях, упомянутых в статье [Действия перемещения данных](data-factory-data-movement-activities.md).</li> |
| WindowEnd |Конец интервала времени для текущего окна запуска действия. |activity |то же, что и WindowStart. |
| SliceStart |Начало интервала времени для формируемого среза данных. |activity<br/>набор данных |<ol><li>Укажите динамические пути к папкам и имена файлов при работе с [большими двоичными объектами Azure](data-factory-azure-blob-connector.md) и [наборами данных файловой системы](data-factory-onprem-file-system-connector.md).</li><li>Укажите входные зависимости с помощью функций фабрики данных в коллекции входных данных действия.</li></ol> |
| SliceEnd |Конец интервала времени для текущего среза данных. |activity<br/>набор данных |То же, что и для SliceStart. |

> [!NOTE]
> В настоящее время для фабрики данных требуется, чтобы расписание, указанное в действии, в точности соответствовало расписанию, указанному в разделе availability выходного набора данных. Следовательно, WindowStart, WindowEnd, SliceStart и SliceEnd всегда сопоставляются с одним и тем же периодом времени и отдельным выходным срезом.
> 

### <a name="example-for-using-a-system-variable"></a>Пример использования системной переменной
В этом примере год, месяц, день и время **SliceStart** извлекаются в отдельные переменные, используемые в свойствах **folderPath** и **fileName**.

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```

## <a name="data-factory-functions"></a>Функции фабрики данных
Функции фабрики данных можно использовать с системными переменными для следующих целей:

1. Указание запросов выбора данных (см. статьи о соединителях, на которые ссылается статья [Действия перемещения данных](data-factory-data-movement-activities.md)).
   
   Для вызова функции фабрики данных используется следующий синтаксис: **$$\<function>** для запросов выбора данных и других свойств в наборе действий и DataSet.  
2. Указание входных зависимостей с помощью функций фабрики данных в коллекции входных данных действия.
   
    Префикс $$ не требуется при указании выражений входных зависимостей.     

В следующем примере свойству **sqlReaderQuery** в JSON-файле присваивается значение, возвращаемое функцией `Text.Format`. Кроме того, в этом примере используется системная переменная с именем **WindowStart**, которая отображает время начала для окна запуска действия.

```json
{
    "Type": "SqlSource",
    "sqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
}
```

В разделе [Строки настраиваемых форматов даты и времени](/dotnet/standard/base-types/custom-date-and-time-format-strings) описаны различные варианты форматирования, которые можно использовать (пример: гг и гггг). 

### <a name="functions"></a>Функции
В приведенных ниже таблицах перечислены все функции фабрики данных Azure.

| Category | Компонент | Параметры | Описание |
| --- | --- | --- | --- |
| Time |AddHours(X,Y) |X: DateTime  <br/><br/>Y: int |Добавляет Y часов к заданному времени X. <br/><br/>Например, `9/5/2013 12:00:00 PM + 2 hours = 9/5/2013 2:00:00 PM`. |
| время; |AddMinutes(X,Y) |X: DateTime  <br/><br/>Y: int |Добавляет Y минут к Х.<br/><br/>Например, `9/15/2013 12: 00:00 PM + 15 minutes = 9/15/2013 12: 15:00 PM`. |
| время; |StartOfHour(X) |X: DateTime  |Получает то время, когда начинался час, отображаемый компонентом часа в параметре X. <br/><br/>Например, `StartOfHour of 9/15/2013 05: 10:23 PM is 9/15/2013 05: 00:00 PM`. |
| Дата |AddDays(X,Y) |X: DateTime <br/><br/>Y: int |Добавляет Y дней к Х. <br/><br/>Пример: 15.09.2013 12:00:00 + 2 дня = 17.09.2013 12:00:00.<br/><br/>Можно также вычитать дни, указав в качестве Y отрицательное число.<br/><br/>Например, `9/15/2013 12:00:00 PM - 2 days = 9/13/2013 12:00:00 PM`. |
| Дата |AddMonths(X,Y) |X: DateTime <br/><br/>Y: int |Добавляет Y месяцев к Х.<br/><br/>`Example: 9/15/2013 12:00:00 PM + 1 month = 10/15/2013 12:00:00 PM`.<br/><br/>Можно также вычитать месяцы, указав в качестве Y отрицательное число.<br/><br/>Например, `9/15/2013 12:00:00 PM - 1 month = 8/15/2013 12:00:00 PM`.|
| Дата |AddQuarters(X,Y) |X: DateTime  <br/><br/>Y: int |Добавляет Y * 3 месяцев к Х.<br/><br/>Например, `9/15/2013 12:00:00 PM + 1 quarter = 12/15/2013 12:00:00 PM`. |
| Дата |AddWeeks(X,Y) |X: DateTime <br/><br/>Y: int |Добавляет Y (одна единица которого равна 7 дням) к X<br/><br/>Пример: 15.09.2013 12:00:00 + 1 неделя = 22.09.2013 12:00:00.<br/><br/>Можно также вычитать недели, указав в качестве Y отрицательное число.<br/><br/>Например, `9/15/2013 12:00:00 PM - 1 week = 9/7/2013 12:00:00 PM`. |
| Дата |AddYears(X,Y) |X: DateTime <br/><br/>Y: int |Добавляет Y лет к Х.<br/><br/>`Example: 9/15/2013 12:00:00 PM + 1 year = 9/15/2014 12:00:00 PM`<br/><br/>Можно также вычитать года, указав в качестве Y отрицательное число.<br/><br/>Например, `9/15/2013 12:00:00 PM - 1 year = 9/15/2012 12:00:00 PM`. |
| Дата |Day(X) |X: DateTime  |Возвращает значение дня для X.<br/><br/>Например, `Day of 9/15/2013 12:00:00 PM is 9`. |
| Дата |DayOfWeek(X) |X: DateTime  |Возвращает значение дня недели для X.<br/><br/>Например, `DayOfWeek of 9/15/2013 12:00:00 PM is Sunday`. |
| Дата |DayOfYear(X) |X: DateTime  |Получает день года, представленный значением года параметра X.<br/><br/>Примеры:<br/>`12/1/2015: day 335 of 2015`<br/>`12/31/2015: day 365 of 2015`<br/>`12/31/2016: day 366 of 2016 (Leap Year)` |
| Дата |DaysInMonth(X) |X: DateTime  |Получает количество дней в месяце, представленном значением месяца параметра X.<br/><br/>Например, `DaysInMonth of 9/15/2013 are 30 since there are 30 days in the September month`. |
| Дата |EndOfDay(X) |X: DateTime  |Получает значение даты и времени, которое соответствует окончанию суток (значения дня) в параметре X.<br/><br/>Например, `EndOfDay of 9/15/2013 05:10:23 PM is 9/15/2013 11:59:59 PM`. |
| Дата |EndOfMonth(X) |X: DateTime  |Получает значение конца месяца, представленного значением месяца параметра X. <br/><br/>Пример: `EndOfMonth of 9/15/2013 05:10:23 PM is 9/30/2013 11:59:59 PM` (дата и время конца сентября). |
| Дата |StartOfDay(X) |X: DateTime  |Получает начало суток, представленных значением дня в параметре X.<br/><br/>Например, `StartOfDay of 9/15/2013 05:10:23 PM is 9/15/2013 12:00:00 AM`. |
| Дата/время |From(X) |X: String |Разбор строки X в значение даты и времени. |
| Дата/время |Ticks(X) |X: DateTime  |Получает свойство тактов времени параметра X. Один такт равен 100 наносекундам. Значение этого свойства соответствует количеству тактов, прошедших с полуночи (24:00:00) 1 января 0001 года. |
| Текст |Format(X) |X: String (переменная) |Форматирует текст (используйте комбинацию `\\'` для экранирования символа `'`).|

> [!IMPORTANT]
> При использовании функции в другой функции не нужно использовать **$$** префикс для внутренней функции. Например, $$Text.Format('PartitionKey eq \\'my_pkey_filter_value\\' and RowKey ge \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(SliceStart, -6)). В этом примере обратите внимание, что **$$** префикс не используется для функции **time. AddHours** . 

#### <a name="example"></a>Пример
В следующем примере входные и выходные параметры для действия Hive определяются с помощью функции `Text.Format` и системной переменной SliceStart. 

```json  
{
    "name": "HiveActivitySamplePipeline",
        "properties": {
    "activities": [
            {
            "name": "HiveActivitySample",
            "type": "HDInsightHive",
            "inputs": [
                    {
                    "name": "HiveSampleIn"
                    }
            ],
            "outputs": [
                    {
                    "name": "HiveSampleOut"
                }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
                    "scriptPath": "adfwalkthrough\\scripts\\samplehive.hql",
                    "scriptLinkedService": "StorageLinkedService",
                    "defines": {
                        "Input": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/samplein/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)",
                        "Output": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/sampleout/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)"
                    },
                    "scheduler": {
                        "frequency": "Hour",
                        "interval": 1
                }
            }
            }
    ]
    }
}
```

### <a name="example-2"></a>Пример 2

В следующем примере параметр DateTime для действия хранимой процедуры определяется с помощью функции Text. Format и системной переменной SliceStart. 

```json
{
    "name": "SprocActivitySamplePipeline",
    "properties": {
        "activities": [
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "usp_sample",
                    "storedProcedureParameters": {
                        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)"
                    }
                },
                "outputs": [
                    {
                        "name": "sprocsampleout"
                    }
                ],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "SprocActivitySample"
            }
        ],
            "start": "2016-08-02T00:00:00Z",
            "end": "2016-08-02T05:00:00Z",
        "isPaused": false
    }
}
```

### <a name="example-3"></a>Пример 3
Для чтения данных за предыдущий день, а не день, указанный в SliceStart, используйте функцию AddDays, как показано в следующем примере. 

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-01-01T08:00:00",
        "end": "2017-01-01T11:00:00",
        "description": "hive activity",
        "activities": [
            {
                "name": "SampleHiveActivity",
                "inputs": [
                    {
                        "name": "MyAzureBlobInput",
                        "startTime": "Date.AddDays(SliceStart, -1)",
                        "endTime": "Date.AddDays(SliceEnd, -1)"
                    }
                ],
                "outputs": [
                    {
                        "name": "MyAzureBlobOutput"
                    }
                ],
                "linkedServiceName": "HDInsightLinkedService",
                "type": "HDInsightHive",
                "typeProperties": {
                    "scriptPath": "adftutorial\\hivequery.hql",
                    "scriptLinkedService": "StorageLinkedService",
                    "defines": {
                        "Year": "$$Text.Format('{0:yyyy}',WindowsStart)",
                        "Month": "$$Text.Format('{0:MM}',WindowStart)",
                        "Day": "$$Text.Format('{0:dd}',WindowStart)"
                    }
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 2,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

В разделе [Строки настраиваемых форматов даты и времени](/dotnet/standard/base-types/custom-date-and-time-format-strings) описаны различные варианты форматирования, которые можно использовать (пример: гг и гггг).
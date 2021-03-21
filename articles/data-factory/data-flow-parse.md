---
title: Анализ преобразования данных в потоке данных сопоставления
description: Анализ документов внедренных столбцов
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/08/2021
ms.openlocfilehash: 4db9503ea84ae13148a89a03048c73399413e5cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101710198"
---
# <a name="parse-transformation-in-mapping-data-flow"></a>Преобразование "анализ" в потоке данных сопоставления

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Используйте преобразование «синтаксический анализ» для анализа столбцов данных в форме документа. Текущие поддерживаемые типы внедренных документов, которые могут быть проанализированы, — это JSON и текст с разделителями.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RWykdO]

## <a name="configuration"></a>Конфигурация

На панели Конфигурация преобразования «анализ» сначала необходимо выбрать тип данных, содержащихся в столбцах, которые необходимо проанализировать. Преобразование «анализ» также содержит следующие параметры конфигурации.

![Параметры синтаксического анализа](media/data-flow/data-flow-parse-1.png "Синтаксический анализ")

### <a name="column"></a>Столбец

Как и в случае с производными столбцами и статистическими выражениями, здесь можно изменить выходной столбец, выбрав его в раскрывающемся списке выбора. Также можно ввести имя нового столбца. Файл ADF будет хранить проанализированные исходные данные в этом столбце.

### <a name="expression"></a>Expression

Используйте построитель выражений, чтобы задать источник для анализа. Это может быть просто просто выбором исходного столбца с автономными данными, которые необходимо проанализировать, или создавать сложные выражения для анализа.

### <a name="output-column-type"></a>Тип выходного столбца

Здесь вы настроите целевую выходную схему из синтаксического анализа, который будет записан в один столбец.

![Пример синтаксического анализа](media/data-flow/data-flow-parse-2.png "Пример синтаксического анализа")

В этом примере мы определили синтаксический анализ входящего поля «Жсонстринг», которое является обычным текстом, но отформатировано как структура JSON. Мы будем хранить проанализированные результаты в виде JSON в новом столбце с именем "JSON" с этой схемой:

```(trade as boolean, customers as string[])```

Чтобы убедиться, что выходные данные сопоставлены должным образом, обратитесь к вкладке Проверка и предварительный просмотр данных.

## <a name="examples"></a>Примеры

```
source(output(
        name as string,
        location as string,
        satellites as string[],
        goods as (trade as boolean, customers as string[], orders as (orderId as string, orderTotal as double, shipped as (orderItems as (itemName as string, itemQty as string)[]))[])
    ),
    allowSchemaDrift: true,
    validateSchema: false,
    ignoreNoFilesFound: false,
    documentForm: 'documentPerLine') ~> JsonSource
source(output(
        movieId as string,
        title as string,
        genres as string
    ),
    allowSchemaDrift: true,
    validateSchema: false,
    ignoreNoFilesFound: false) ~> CsvSource
JsonSource derive(jsonString = toString(goods)) ~> StringifyJson
StringifyJson parse(json = jsonString ? (trade as boolean,
        customers as string[]),
    format: 'json',
    documentForm: 'arrayOfDocuments') ~> ParseJson
CsvSource derive(csvString = 'Id|name|year\n\'1\'|\'test1\'|\'1999\'') ~> CsvString
CsvString parse(csv = csvString ? (id as integer,
        name as string,
        year as string),
    format: 'delimited',
    columnNamesAsHeader: true,
    columnDelimiter: '|',
    nullValue: '',
    documentForm: 'documentPerLine') ~> ParseCsv
ParseJson select(mapColumn(
        jsonString,
        json
    ),
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> KeepStringAndParsedJson
ParseCsv select(mapColumn(
        csvString,
        csv
    ),
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> KeepStringAndParsedCsv
```

## <a name="data-flow-script"></a>Скрипт потока данных

### <a name="syntax"></a>Синтаксис

### <a name="examples"></a>Примеры

```
parse(json = jsonString ? (trade as boolean,
                                customers as string[]),
                format: 'json',
                documentForm: 'singleDocument') ~> ParseJson

parse(csv = csvString ? (id as integer,
                                name as string,
                                year as string),
                format: 'delimited',
                columnNamesAsHeader: true,
                columnDelimiter: '|',
                nullValue: '',
                documentForm: 'documentPerLine') ~> ParseCsv
```    

## <a name="next-steps"></a>Дальнейшие действия

* Используйте [Преобразование «Сведение](data-flow-flatten.md) » для свертывания строк в столбцы.
* Используйте [Преобразование «Производный столбец](data-flow-derived-column.md) » для сведения столбцов к строкам.

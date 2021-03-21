---
title: Использование параметров и выражений в фабрике данных Azure
description: В этом руководстве содержатся сведения о выражениях и функциях, которые можно использовать при создании сущностей фабрики данных.
author: ssabat
ms.author: susabat
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/08/2020
ms.openlocfilehash: 8f22645eafa0969eac3d6c4c0645909f8c650cad
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103199812"
---
# <a name="how-to-use-parameters-expressions-and-functions-in-azure-data-factory"></a>Как использовать параметры, выражения и функции в фабрике данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-functions-variables.md)
> * [Текущая версия](how-to-expression-language-functions.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этом документе основное внимание будет уделено изучению фундаментальных концепций с различными примерами для изучения возможности создания параметризованных конвейеров данных в фабрике данных Azure. Параметризация и динамические выражения — это важные дополнения к ADF, поскольку они могут сэкономить много времени и обеспечить гораздо более гибкое решение для извлечения, преобразования, загрузки (ETL) или извлечения, загрузки, преобразования (ELT), что позволит значительно сократить затраты на обслуживание решений и ускорить реализацию новых функций в существующих конвейерах. Эти преимущества обусловлены тем, что параметризация уменьшает объем жесткого кода и увеличивает количество многократно используемых объектов и процессов в решении.

## <a name="azure-data-factory-ui-and-parameters"></a>Пользовательский интерфейс и параметры фабрики данных Azure

Если вы не знакомы с использованием параметра фабрики данных Azure в пользовательском интерфейсе ADF, проверьте [Пользовательский интерфейс фабрики данных для связанных служб с параметрами](https://docs.microsoft.com/azure/data-factory/parameterize-linked-services#data-factory-ui)  и [интерфейсом пользователя фабрики данных для конвейера, управляемого метаданными, с параметрами](https://docs.microsoft.com/azure/data-factory/how-to-use-trigger-parameterization#data-factory-ui) визуального объяснения.

## <a name="parameter-and-expression-concepts"></a>Основные понятия параметров и выражений 

Параметры можно использовать для передачи внешних значений в конвейеры, наборы данных, связанные службы и потоковые данные. После передачи параметра в ресурс его нельзя изменить. Путем параметризации ресурсов их можно повторно использовать с разными значениями каждый раз. Параметры можно использовать по отдельности или в составе выражений. Значения JSON в определении могут быть литералами или выражениями, которые оцениваются в среде выполнения.

Пример:  
  
```json
"name": "value"
```

 или диспетчер конфигурации служб  
  
```json
"name": "@pipeline().parameters.password"
```

Выражения могут встречаться в любом месте строкового значения JSON и всегда возвращают другое значение JSON. Здесь *Password* — это параметр конвейера в выражении. Если значение JSON является выражением, текст выражения извлекается без знака @ (\@). Если требуется строковый литерал, начинающийся с \@, его необходимо экранировать с помощью \@\@. В примерах ниже показано, как вычисляются выражения.  
  
|Значение JSON|Результат|  
|----------------|------------|  
|"parameters"|Возвращаются символы в виде 'parameters'.|  
|"parameters[1]"|Возвращаются символы в виде 'parameters[1]'.|  
|"\@\@"|Возвращается строка из 1 символа, содержащая символ "\@".|  
|" \@"|Возвращается строка из 2 символов, содержащая символ "\@".|  
  
 Выражения также могут содержаться внутри строк, где они заключаются в структуру `@{ ... }`, при использовании *интерполяции строк*. Например: `"name" : "First Name: @{pipeline().parameters.firstName} Last Name: @{pipeline().parameters.lastName}"`  
  
 При использовании интерполяции строки результатом всегда будет строка. Предположим, `myNumber` определено как `42`, а `myString` — как `foo`:  
  
|Значение JSON|Результат|  
|----------------|------------|  
|"\@pipeline().parameters.myString"| Возвращает `foo` как строку.|  
|"\@{pipeline().parameters.myString}"| Возвращает `foo` как строку.|  
|"\@pipeline().parameters.myNumber"| Возвращает `42` как *номер*.|  
|"\@{pipeline().parameters.myNumber}"| Возвращает `42` как *строку*.|  
|Answer is: @{pipeline().parameters.myNumber}| Возвращает строку `Answer is: 42`.|  
|"\@concat('Answer is: ', string(pipeline().parameters.myNumber))"| Возвращает строку `Answer is: 42`.|  
|"Answer is: \@\@{pipeline().parameters.myNumber}"| Возвращает строку `Answer is: @{pipeline().parameters.myNumber}`.|  

## <a name="examples-of-using-parameters-in-expressions"></a>Примеры использования параметров в выражениях 

### <a name="complex-expression-example"></a>Пример сложных выражений
Приведенный ниже пример содержит сложное выражение, которое ссылается на глубоко вложенное поле в выходных данных действия. Чтобы создать ссылку на параметр конвейера, который вычисляет вложенное поле, используйте синтаксис [] вместо оператора точки (.) (как subfield1 и subfield2 в нашем примере).

`@activity('*activityName*').output.*subfield1*.*subfield2*[pipeline().parameters.*subfield3*].*subfield4*`

### <a name="dynamic-content-editor"></a>Редактор динамического содержимого

Редактор динамического содержимого автоматически поменяет escape-символы в содержимом после завершения редактирования. Например, следующее содержимое в редакторе содержимого является интерполяцией строк с двумя функциями выражений. 

```json
{ 
  "type": "@{if(equals(1, 2), 'Blob', 'Table' )}",
  "name": "@{toUpper('myData')}"
}
```

Редактор динамического содержимого преобразует содержимое в выражение `"{ \n  \"type\": \"@{if(equals(1, 2), 'Blob', 'Table' )}\",\n  \"name\": \"@{toUpper('myData')}\"\n}"` . Результатом этого выражения является строка формата JSON, показанная ниже.

```json
{
  "type": "Table",
  "name": "MYDATA"
}
```

### <a name="a-dataset-with--parameters"></a>Набор данных с параметрами

В следующем примере BlobDataset принимает параметр с именем **path**. Его значение используется для задания значения свойства **folderPath** с помощью выражения: `dataset().path`. 

```json
{
    "name": "BlobDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": "@dataset().path"
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "path": {
                "type": "String"
            }
        }
    }
}
```

### <a name="a-pipeline-with--parameters"></a>Конвейер с параметрами

В следующем примере конвейер принимает параметры **inputPath** и **outputPath**. **Путь** к параметризованному набору данных большого двоичного объекта задается с помощью значений этих параметров. Синтаксис, используемый здесь: `pipeline().parameters.parametername`. 

```json
{
    "name": "Adfv2QuickStartPipeline",
    "properties": {
        "activities": [
            {
                "name": "CopyFromBlobToBlob",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.inputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.outputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                }
            }
        ],
        "parameters": {
            "inputPath": {
                "type": "String"
            },
            "outputPath": {
                "type": "String"
            }
        }
    }
}
```

  
## <a name="calling-functions-within-expressions"></a>Вызов функций внутри выражений 

Внутри выражений можно вызывать функции. В следующих разделах предоставлены сведения о функциях, которые могут использоваться в выражении.  

### <a name="string-functions"></a>Строковые функции  

Для работы со строками вы можете использовать эти строковые функции, а также некоторые [функции для коллекций](#collection-functions).
Строковые функции работают только со строками.

| Строковая функция | Задача |
| --------------- | ---- |
| [concat](control-flow-expression-language-functions.md#concat) | Объединяет две или более строк и возвращает объединенную строку. |
| [endsWith](control-flow-expression-language-functions.md#endswith) | Проверяет, заканчивается ли строка определенной подстрокой. |
| [guid](control-flow-expression-language-functions.md#guid) | Создает глобально уникальный идентификатор (GUID) в виде строки. |
| [indexOf](control-flow-expression-language-functions.md#indexof) | Возвращает начальную позицию подстроки. |
| [lastIndexOf](control-flow-expression-language-functions.md#lastindexof) | Возвращает начальную позицию последнего вхождения подстроки. |
| [replace](control-flow-expression-language-functions.md#replace) | Заменяет подстроку указанной строкой и возвращает обновленную строку. |
| [split](control-flow-expression-language-functions.md#split) | Возвращает массив, содержащий подстроки, разделенные запятыми, из большей строки, основываясь на указанном символе разделителя в исходной строке. |
| [startsWith](control-flow-expression-language-functions.md#startswith) | Проверяет, начинается ли строка с определенной подстроки. |
| [substring](control-flow-expression-language-functions.md#substring) | Возвращает символы из строки, начиная с указанной позиции. |
| [toLower](control-flow-expression-language-functions.md#toLower) | Возвращает строку символов в нижнем регистре. |
| [toUpper](control-flow-expression-language-functions.md#toUpper) | Возвращает строку символов в верхнем регистре. |
| [trim](control-flow-expression-language-functions.md#trim) | Удаляет все начальные и конечные пробелы и возвращает обновленную строку. |

### <a name="collection-functions"></a>Функции для коллекций

Для работы с коллекциями, обычно массивами, строками, а иногда и словарями, вы можете использовать следующие функции.

| Функция для коллекций | Задача |
| ------------------- | ---- |
| [contains](control-flow-expression-language-functions.md#contains) | Проверяет наличие определенного элемента в коллекции. |
| [empty](control-flow-expression-language-functions.md#empty) | Проверяет, является ли коллекция пустой. |
| [first](control-flow-expression-language-functions.md#first) | Возвращает первый элемент из коллекции. |
| [intersection](control-flow-expression-language-functions.md#intersection) | Возвращает коллекцию, которая содержит *только* общие элементы в указанных коллекциях. |
| [join](control-flow-expression-language-functions.md#join) | Возвращает строку, содержащую *все* элементы из массива, в которой каждый символ отделен разделителем. |
| [last](control-flow-expression-language-functions.md#last) | Возвращает последний элемент из коллекции. |
| [length](control-flow-expression-language-functions.md#length) | Возвращает число элементов в строке или массиве. |
| [skip](control-flow-expression-language-functions.md#skip) | Удаляет элементы из начала коллекции и возвращает *все другие элементы*. |
| [take](control-flow-expression-language-functions.md#take) | Возвращает элементы, расположенные в начале коллекции. |
| [union](control-flow-expression-language-functions.md#union) | Возвращает коллекцию, которая содержит *все* элементы из указанных коллекций. | 

### <a name="logical-functions"></a>Логические функции  

Эти функции подходят для условий и могут использоваться для определения логики любого типа.  
  
| Функция логического сравнения | Задача |
| --------------------------- | ---- |
| [and](control-flow-expression-language-functions.md#and) (и); | Проверяет, истинны ли все выражения. |
| [equals](control-flow-expression-language-functions.md#equals) | Проверяет, эквивалентны ли оба значения. |
| [greater](control-flow-expression-language-functions.md#greater) | Проверяет, является ли первое значение большим, чем второе. |
| [greaterOrEquals](control-flow-expression-language-functions.md#greaterOrEquals) | Проверяет, является ли первое значение большим, чем второе, или равным ему. |
| [if](control-flow-expression-language-functions.md#if) (если); | Проверьте, какое значение имеет выражение: true или false. Возвращает указанное значение на основе результата. |
| [less](control-flow-expression-language-functions.md#less) | Проверяет, является ли первое значение меньшим, чем второе. |
| [lessOrEquals](control-flow-expression-language-functions.md#lessOrEquals) | Проверяет, является ли первое значение меньшим, чем второе, или равным ему. |
| [not](control-flow-expression-language-functions.md#not) (не); | Проверяет, имеет ли выражение значение false. |
| [или диспетчер конфигурации служб](control-flow-expression-language-functions.md#or) | Проверяет, является ли хотя бы одно выражение истинным. |
  
### <a name="conversion-functions"></a>Функции преобразования  

 Эти функции используются для преобразования между собственными типами языка:  
-   строка
-   Целое число
-   FLOAT
-   Логическое
-   arrays;
-   dictionaries;

| Функция преобразования | Задача |
| ------------------- | ---- |
| [array](control-flow-expression-language-functions.md#array). | Возвращает массив из одного экземпляра указанных входных данных. Для использования нескольких входных данных см. раздел [createArray](control-flow-expression-language-functions.md#createArray). |
| [base64](control-flow-expression-language-functions.md#base64) | Возвращает версию строки с кодировкой base64 для заданной строки. |
| [base64ToBinary](control-flow-expression-language-functions.md#base64ToBinary) | Возвращает двоичную версию строки с кодировкой base64. |
| [base64ToString](control-flow-expression-language-functions.md#base64ToString) | Возвращает строковую версию строки с кодировкой base64. |
| [binary](control-flow-expression-language-functions.md#binary) | Возвращает двоичную версию входного значения. |
| [bool](control-flow-expression-language-functions.md#bool); | Возвращает логическую версию входного значения. |
| [coalesce](control-flow-expression-language-functions.md#coalesce) | Возвращает первое ненулевое значение из одного или нескольких параметров. |
| [createArray](control-flow-expression-language-functions.md#createArray) | Возвращает массив из нескольких экземпляров входных данных. |
| [dataUri](control-flow-expression-language-functions.md#dataUri) | Возвращает URI данных входного значения. |
| [dataUriToBinary](control-flow-expression-language-functions.md#dataUriToBinary) | Возвращает двоичную версию строки URI данных. |
| [dataUriToString](control-flow-expression-language-functions.md#dataUriToString) | Возвращает строковую версию URI данных. |
| [decodeBase64](control-flow-expression-language-functions.md#decodeBase64) | Возвращает строковую версию строки с кодировкой base64. |
| [decodeDataUri](control-flow-expression-language-functions.md#decodeDataUri) | Возвращает двоичную версию строки URI данных. |
| [decodeUriComponent](control-flow-expression-language-functions.md#decodeUriComponent) | Возвращает строку, которая заменяет escape-символы декодированными версиями. |
| [encodeUriComponent](control-flow-expression-language-functions.md#encodeUriComponent) | Возвращает строку, которая заменяет символы, опасные для URL-адреса, escape-символами. |
| [float](control-flow-expression-language-functions.md#float) | Возвращает значение с плавающей запятой в качестве входного значения. |
| [int](control-flow-expression-language-functions.md#int) | Возвращает целочисленную версию строки. |
| [json](control-flow-expression-language-functions.md#json) | Возвращает значение типа JSON либо объект для строки или XML. |
| [строка](control-flow-expression-language-functions.md#string) | Возвращает строковую версию входного значения. |
| [uriComponent](control-flow-expression-language-functions.md#uriComponent) | Возвращает кодированную версию URI для входного значения, заменив символы, опасные для URL-адреса, на escape-символы. |
| [uriComponentToBinary](control-flow-expression-language-functions.md#uriComponentToBinary) | Возвращает двоичную версию строки с закодированным URI. |
| [uriComponentToString](control-flow-expression-language-functions.md#uriComponentToString) | Возвращает строковую версию строки с закодированным URI. |
| [xml](control-flow-expression-language-functions.md#xml) | Возвращает XML-версию строки. |
| [xpath](control-flow-expression-language-functions.md#xpath) | Проверяет XML на наличие узлов или значений, которые соответствуют выражению XPath, и возвращает соответствующие узлы или значения. |

### <a name="math-functions"></a>Математические функции  
 Эти функции могут использоваться для любого типа чисел: **целых чисел** и **чисел с плавающей запятой**.  

| Математическая функция | Задача |
| ------------- | ---- |
| [добавление](control-flow-expression-language-functions.md#add) | Возвращает результат сложения двух чисел. |
| [div](control-flow-expression-language-functions.md#div) | Возвращает результат деления двух чисел. |
| [max](control-flow-expression-language-functions.md#max) | Возвращает наибольшее значение из набора чисел или массива. |
| [min](control-flow-expression-language-functions.md#min) | Возвращает наименьшее значение из набора чисел или массива. |
| [mod (модуль)](control-flow-expression-language-functions.md#mod) | Возвращает остаток результата деления двух чисел. |
| [mul](control-flow-expression-language-functions.md#mul) | Возвращает результат умножения двух чисел. |
| [rand](control-flow-expression-language-functions.md#rand) | Возвращает случайное целое число из указанного диапазона. |
| [range](control-flow-expression-language-functions.md#range) | Возвращает массив целых чисел, который начинается с заданного целого числа. |
| [sub](control-flow-expression-language-functions.md#sub) | Вычитает второе число из первого числа и возвращает результат. |
  
### <a name="date-functions"></a>Функции данных  

| Функция даты и времени | Задача |
| --------------------- | ---- |
| [addDays](control-flow-expression-language-functions.md#addDays) | Добавляет количество дней к метке времени. |
| [addHours](control-flow-expression-language-functions.md#addHours) | Добавляет количество часов к метке времени. |
| [addMinutes](control-flow-expression-language-functions.md#addMinutes) | Добавляет количество минут к метке времени. |
| [addSeconds](control-flow-expression-language-functions.md#addSeconds) | Добавляет количество секунд к метке времени. |
| [addToTime](control-flow-expression-language-functions.md#addToTime) | Добавляет количество единиц времени к метке времени. См. раздел [getFutureTime](control-flow-expression-language-functions.md#getFutureTime). |
| [convertFromUtc](control-flow-expression-language-functions.md#convertFromUtc) | Преобразовывает метку времени формата UTC в целевой часовой пояс. |
| [convertTimeZone](control-flow-expression-language-functions.md#convertTimeZone) | Преобразовывает метку времени из исходного часового пояса в целевой. |
| [convertToUtc](control-flow-expression-language-functions.md#convertToUtc) | Преобразует метку времени с исходным часовым поясом в формат UTC. |
| [dayOfMonth](control-flow-expression-language-functions.md#dayOfMonth) | Возвращает компонент дня месяца из метки времени. |
| [dayOfWeek](control-flow-expression-language-functions.md#dayOfWeek) | Возвращает компонент дня недели из метки времени. |
| [dayOfYear](control-flow-expression-language-functions.md#dayOfYear) | Возвращает компонент дня года из метки времени. |
| [formatDateTime](control-flow-expression-language-functions.md#formatDateTime) | Возвращает метку времени в виде строки в произвольном формате. |
| [getFutureTime](control-flow-expression-language-functions.md#getFutureTime) | Возвращает текущую метку времени, а также указанные единицы времени. См. раздел [addToTime](control-flow-expression-language-functions.md#addToTime). |
| [getPastTime](control-flow-expression-language-functions.md#getPastTime) | Возвращает текущую метку времени, вычитая указанные единицы времени. См. раздел [subtractFromTime](control-flow-expression-language-functions.md#subtractFromTime). |
| [startOfDay](control-flow-expression-language-functions.md#startOfDay) | Возвращает начало дня для метки времени. |
| [startOfHour](control-flow-expression-language-functions.md#startOfHour) | Возвращает начало часа для метки времени. |
| [startOfMonth](control-flow-expression-language-functions.md#startOfMonth) | Возвращает начало месяца для метки времени. |
| [subtractFromTime](control-flow-expression-language-functions.md#subtractFromTime) | Вычитает количество единиц времени из метки времени. См. раздел [getPastTime](control-flow-expression-language-functions.md#getPastTime). |
| [ticks](control-flow-expression-language-functions.md#ticks) | Возвращает значение свойства `ticks` для указанной метки времени. |
| [utcNow](control-flow-expression-language-functions.md#utcNow) | Возвращает текущую метку времени в виде строки. |

## <a name="detailed-examples-for-practice"></a>Подробные примеры для практики

### <a name="detailed-azure-data-factory-copy-pipeline-with-parameters"></a>Подробный конвейер копирования фабрики данных Azure с параметрами 

В этом [руководстве по передаче параметров конвейера копирования фабрики данных Azure](https://azure.microsoft.com/mediahandler/files/resourcefiles/azure-data-factory-passing-parameters/Azure%20data%20Factory-Whitepaper-PassingParameters.pdf) вы узнаете, как передавать параметры между конвейером и действием, а также между действиями.

### <a name="detailed--mapping-data-flow-pipeline-with-parameters"></a>Подробный конвейер потока данных сопоставления с параметрами 

Выполните [сопоставление потока данных с параметрами,](https://docs.microsoft.com/azure/data-factory/parameters-data-flow) чтобы получить исчерпывающий пример использования параметров в потоке данных.

### <a name="detailed-metadata-driven-pipeline-with-parameters"></a>Подробные сведения о конвейере, управляемом метаданными, с параметрами

Используйте [конвейер, управляемый метаданными, с параметрами](https://docs.microsoft.com/azure/data-factory/how-to-use-trigger-parameterization) , чтобы узнать больше об использовании параметров для проектирования конвейеров, управляемых метаданными. Это популярный вариант использования параметров.


## <a name="next-steps"></a>Дальнейшие действия
Список системных переменных, которые можно использовать в выражениях, см. в статье [System variables supported by Azure Data Factory](control-flow-system-variables.md) (Системные переменные, поддерживаемые фабрикой данных Azure).

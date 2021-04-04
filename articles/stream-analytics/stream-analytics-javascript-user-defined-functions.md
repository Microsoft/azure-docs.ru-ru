---
title: Определяемые пользователем функции JavaScript в Azure Stream Analytics
description: Эта статья содержит вводные сведения о определяемых пользователем функциях JavaScript в Stream Analytics.
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.service: stream-analytics
ms.topic: tutorial
ms.custom: mvc, devx-track-js
ms.date: 12/15/2020
ms.openlocfilehash: 70015ef24039694789ce96a6c4853221fe2377c3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98020389"
---
# <a name="javascript-user-defined-functions-in-azure-stream-analytics"></a>Определяемые пользователем функции JavaScript в Azure Stream Analytics
 
Azure Stream Analytics поддерживает определяемые пользователем функции, написанные на языке JavaScript. Благодаря обширному набору методов, которые предоставляют объекты JavaScript **String**, **RegExp**, **Math**, **Array** и **Date**, в заданиях Stream Analytics стало проще создавать сложные преобразования данных.

## <a name="overview"></a>Обзор

Определяемые пользователем функции JavaScript поддерживают скалярные вычислительные функции без отслеживания состояния, не требующие внешнего подключения. Возвращаемое значение функции может быть только скалярным (одиночным). Добавив в задание определяемую пользователем функцию JavaScript, вы сможете использовать ее в любом месте запроса, например во встроенной скалярной функции.

Ниже приведены некоторые сценарии, в которых могут пригодиться определяемые пользователем функции JavaScript.
* Анализ и обработка строки с использованием функций регулярных выражений, например **Regexp_Replace()** или **Regexp_Extract()** .
* Декодирование и кодирование данных, например преобразование двоичных данных в шестнадцатеричные.
* Математические вычисления с помощью **математических** функций JavaScript.
* Операции с массивами, такие как сортировка, присоединение, поиск и заполнение.

Вот некоторые действия, которые невозможно выполнить в Stream Analytics с помощью определяемой пользователем функции JavaScript.
* Вызов внешних конечных точек REST, например выполнение обратного разрешения IP-адресов или извлечение ссылочных данных из внешнего источника.
* Сериализация или десериализация входных и выходных данных в пользовательский формат сообщения.
* Создание пользовательских статистических функций.

Следует избегать использования таких функций, как **Date.GetDate()** или **Math.random()** , хотя они не запрещены в определении функции. Эти функции возвращают **разные** результаты при каждом вызове, а служба Azure Stream Analytics не ведет журнал вызовов и возвращаемых результатов для функций. Если функция возвращает разные результаты для одних и тех же событий, невозможно гарантировать повторяемость при перезапуске задания пользователем или службой Stream Analytics.

## <a name="add-a-javascript-user-defined-function-to-your-job"></a>Добавление определяемой пользователем функции JavaScript в задание

> [!NOTE]
> Эти действия подходят для заданий Stream Analytics, настроенных для работы в облаке. Если задание Stream Analytics настроено для работы в Azure IoT Edge, используйте вместо этого Visual Studio и [напишите определяемую пользователем функцию, используя C#](stream-analytics-edge-csharp-udf.md).

Чтобы создать определяемую пользователем функцию JavaScript в задании Stream Analytics, выберите **Функции** в разделе **Топология задания**. Затем выберите **JavaScript UDF** из раскрывающегося меню **+Add** (Добавить). 

![Добавление JavaScript UDF](./media/javascript/stream-analytics-jsudf-add.png)

Затем необходимо указать следующие свойства и выбрать **Сохранить**.

|Свойство|Описание|
|--------|-----------|
|Псевдоним функции|Введите имя для вызова функции в запросе.|
|Тип выходных данных|Тип, который будет возвращен определяемой пользователем функцией JavaScript в запрос Stream Analytics.|
|Определение функции|Реализация функции JavaScript, которая будет выполняться каждый раз при вызове определяемых пользователем функций из запроса.|

## <a name="test-and-troubleshoot-javascript-udfs"></a>Тестирование и устранение неполадок с функциями UDF JavaScript 

Вы можете тестировать и отлаживать логику UDF JavaScript в любом браузере. Отладка и тестирование логики этих определяемых пользователем функций сейчас не поддерживается на портале Stream Analytics. Когда функция заработает правильно, ее можно будет добавить в задание Stream Analytics, как упоминалось выше, а затем вызвать ее непосредственно из запроса. Логику запроса можно проверить с помощью JavaScript UDF, используя [средства Stream Analytics для Visual Studio](./stream-analytics-tools-for-visual-studio-install.md).

Ошибки времени выполнения JavaScript считаются неустранимыми и регистрируются в журнале действий. Чтобы получить этот журнал, перейдите на портале Azure к нужному заданию и щелкните **Журнал действий**.

## <a name="call-a-javascript-user-defined-function-in-a-query"></a>Вызов определяемой пользователем функции из запроса

Вы можете легко вызвать функцию JavaScript в запросе с помощью псевдонима функции с префиксом **udf**. Ниже приведен пример JavaScript UDF, которая преобразует шестнадцатеричные значения в целочисленное значение, вызываемое в запросе Stream Analytics.

```SQL
    SELECT
        time,
        UDF.hex2Int(offset) AS IntOffset
    INTO
        output
    FROM
        InputStream
```

## <a name="supported-javascript-objects"></a>Поддерживаемые объекты JavaScript

Определяемые пользователем функции JavaScript в Azure Stream Analytics могут использовать все стандартные встроенные объекты JavaScript. Список этих объектов вы найдете [здесь](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects).

### <a name="stream-analytics-and-javascript-type-conversion"></a>Преобразование типов Stream Analytics и JavaScript

Существуют различия между типами, которые поддерживаются в языке запросов Stream Analytics и в JavaScript. В следующей таблице перечислены сопоставления преобразования между этими типами.

Stream Analytics | JavaScript
--- | ---
BIGINT | Число (JavaScript может представлять целые числа только до значения 2^53).
Дата и время | Дата (JavaScript поддерживает только миллисекунды)
double | Number
nvarchar(MAX) | Строка
Записей | Объект
Array | Array
NULL | NULL

Ниже описаны преобразования из типов JavaScript в типы Stream Analytics.

JavaScript | Stream Analytics
--- | ---
Number | Значение типа bigint, если это целое число в диапазоне от long.MinValue до long.MaxValue. Иначе используется тип double.
Дата | Дата и время
Строка | nvarchar(MAX)
Объект | Записей
Array | Array
NULL, не определено | NULL
Любой другой тип (например, функция или ошибка) | Не поддерживается (возникает ошибка времени выполнения)

Язык JavaScript учитывает регистр, поэтому регистр полей объекта в коде JavaScript должен совпадать с регистром полей во входных данных. В заданиях с уровнем совместимости 1.0 в полях из инструкции SQL SELECT текст преобразуется в нижний регистр. При уровне совместимости 1.1 и выше поля из инструкции SELECT будут иметь тот же регистр, что и в запросе SQL.

## <a name="other-javascript-user-defined-function-patterns"></a>Другие методы использования определяемой пользователем функции

### <a name="write-nested-json-to-output"></a>Вывод вложенных значений JSON

Если вы используете этап обработки результатов, на котором входными данными являются выходные данные в формате JSON задания Stream Analytics, вам нужно записать строку JSON в выходные данные. Приведенный ниже пример вызывает функцию **JSON.stringify()** , которая упаковывает все полученные пары "имя — значение" и передает их единой строкой в качестве выходных данных.

**Определение определяемой пользователем функции JavaScript:**

```javascript
function main(x) {
return JSON.stringify(x);
}
```

**Пример запроса**
```SQL
SELECT
    DataString,
    DataValue,
    HexValue,
    UDF.jsonstringify(input) As InputEvent
INTO
    output
FROM
    input PARTITION BY PARTITIONID
```

### <a name="cast-string-to-json-object-to-process"></a>Приведение строки к объекту JSON для обработки

Если у вас есть строковое поле в формате JSON, которое вы хотите преобразовать в объект JSON для обработки в определяемой пользователем функции JavaScript, можно использовать функцию **JSON.parse()** для создания соответствующего объекта JSON.

**Определение определяемой пользователем функции JavaScript:**

```javascript
function main(x) {
var person = JSON.parse(x);  
return person.name;
}
```

**Пример запроса**
```SQL
SELECT
    UDF.getName(input) AS Name
INTO
    output
FROM
    input
```

### <a name="use-trycatch-for-error-handling"></a>Использование try/catch для обработки ошибок

Блоки try/catch помогают выявить проблемы с неправильно сформированными входными данными, передаваемыми в определяемую пользователем функцию JavaScript.

**Определение определяемой пользователем функции JavaScript:**

```javascript
function main(input, x) {
    var obj = null;

    try{
        obj = JSON.parse(x);
    }catch(error){
        throw input;
    }
    
    return obj.Value;
}
```

**Пример запроса: передача всей записи в качестве первого параметра для ее возврата при возникновении ошибки.**
```SQL
SELECT
    A.context.company AS Company,
    udf.getValue(A, A.context.value) as Value
INTO
    output
FROM
    input A
```

### <a name="tolocalestring"></a>toLocaleString()
Метод **toLocaleString** в JavaScript можно использовать для возврата строки с учетом языка, представляющей данные даты и времени для среды, из которой вызывается этот метод.
Хотя Azure Stream Analytics принимает в качестве системной метки времени только дату и время в формате UTC, этот метод можно использовать, чтобы преобразовать системную метку времени в данные с другим языковым стандартом и часовым поясом.
Этот метод соответствует тому же принципу реализации, что и метод в Internet Explorer.

**Определение определяемой пользователем функции JavaScript:**

```javascript
function main(datetime){
    const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
    return event.toLocaleDateString('de-DE', options);
}
```

**Пример запроса: передача даты и времени в качестве входного значения**
```SQL
SELECT
    udf.toLocaleString(input.datetime) as localeString
INTO
    output
FROM
    input
```

Результатом этого запроса будет входное значение времени и даты в **de-DE** с предоставленными параметрами.
```
Samstag, 28. Dezember 2019
```

## <a name="user-logging"></a>Ведение журнала пользователя
Механизм ведения журнала позволяет собирать определенные сведения во время выполнения задания. Данные журнала можно использовать для отладки или оценки правильности пользовательского кода в реальном времени. Этот механизм доступен через метод Console.Log().

```javascript
console.log('my error message');
```

Доступ к сообщениям журнала можно получить с помощью [журналов диагностики](data-errors.md).
## <a name="next-steps"></a>Дальнейшие действия

* [Определяемая пользователем функция в машинном обучении](./machine-learning-udf.md)
* [Определяемая пользователем функция C#](./stream-analytics-edge-csharp-udf-methods.md)

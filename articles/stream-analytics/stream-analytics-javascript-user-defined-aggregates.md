---
title: Определяемые пользователем вычисления JavaScript в Azure Stream Analytics
description: В этой статье описывается, как выполнять расширенные запросы с помощью определяемых пользователем вычислений JavaScript в Azure Stream Analytics.
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 10/28/2017
ms.custom: devx-track-js
ms.openlocfilehash: e9695a268d1c6ec43a737afd36536e4ec2a41a41
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98012484"
---
# <a name="azure-stream-analytics-javascript-user-defined-aggregates"></a>Пользовательские статистические выражения JavaScript в Azure Stream Analytics
 
Azure Stream Analytics поддерживает пользовательские статистические выражения (UDA) на языке JavaScript, что позволяет реализовывать сложную бизнес-логику с отслеживанием состояния. Внутри пользовательского статистического выражения предоставляется полный контроль над структурой данных состояния, накоплением состояния, сокращением состояния и вычислением статистических результатов. В этой статье представлены два разных интерфейса пользовательских статистических выражений (UDA) JavaScript, а также инструкции по созданию пользовательского статистического выражения и способы его использования в операциях на основе окна в запросе Stream Analytics.

## <a name="javascript-user-defined-aggregates"></a>Пользовательские статистические выражения JavaScript

Пользовательское статистическое выражение используется в указанном временном окне для выполнения статистического вычисления и создания одиночного результирующего значения. Существуют два типа интерфейсов UDA, поддерживаемых Stream Analytics в настоящее: AccumulateOnly и AccumulateDeaccumulate. Оба типа пользовательских статистических выражений можно использовать для "переворачивающегося", "прыгающего", скользящего окна и окна сеанса. Пользовательское статистическое выражение AccumulateDeaccumulate работает быстрее, чем AccumulateOnly, при использовании с "прыгающим", скользящим окном или окном сеанса. Можно выбрать один из двух типов в зависимости от используемого алгоритма.

### <a name="accumulateonly-aggregates"></a>Статистические выражения AccumulateOnly

Статистические выражения AccumulateOnly могут накапливать только новые события в своем состоянии, их алгоритм не допускает удаления значений. Выберите этот тип статистического выражения, если удаление данных события из значения состояния невозможно реализовать. Ниже приведен шаблон JavaScript для статистических выражений AccumulatOnly.

```JavaScript
// Sample UDA which state can only be accumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.computeResult = function () {
        return this.state;
    }
}
```

### <a name="accumulatedeaccumulate-aggregates"></a>Статистические выражения AccumulateDeaccumulate

Статистические выражения AccumulateDeaccumulate допускают удаление накопленного ранее значения из состояния, например удаление пары "ключ-значение" из списка значений события, или вычитание значения из состояния статистического выражения суммирования. Ниже приведен шаблон JavaScript для статистических выражений AccumulateDeaccumulate.

```JavaScript
// Sample UDA which state can be accumulated and deaccumulated.
function main() {
    this.init = function () {
        this.state = 0;
    }

    this.accumulate = function (value, timestamp) {
        this.state += value;
    }

    this.deaccumulate = function (value, timestamp) {
        this.state -= value;
    }

    this.deaccumulateState = function (otherState){
        this.state -= otherState.state;
    }

    this.computeResult = function () {
        return this.state;
    }
}
```

## <a name="uda---javascript-function-declaration"></a>Пользовательское статистическое выражение: объявление функции JavaScript

Каждое пользовательское статистическое выражение JavaScript определяется в объявлении объекта функции. Ниже приведены основные элементы в определении пользовательского статистического выражения.

### <a name="function-alias"></a>Псевдоним функции

Псевдоним функции — это идентификатор пользовательского статистического выражения. При вызове в запросе Stream Analytics всегда используйте псевдоним UDA вместе с "uda". .

### <a name="function-type"></a>Тип функции

Для пользовательского статистического выражения должен быть указан тип функции **Javascript UDA**.

### <a name="output-type"></a>Тип выходных данных

Определенный тип заданий, поддерживаемый Stream Analytics, или "Любой", если требуется обрабатывать тип в запросе.

### <a name="function-name"></a>Имя функции

Имя этого объекта функции. Имя функции должно соответствовать псевдониму пользовательского статистического выражения.

### <a name="method---init"></a>Метод init()

Метод init() инициализирует состояние статистического выражения. Этот метод вызывается в начале окна.

### <a name="method--accumulate"></a>Метод accumulate()

Метод accumulate() определяет состояние пользовательского статистического выражения на основе значений предыдущего состояния и текущего события. Этот метод вызывается, когда событие попадает во временное окно (TUMBLINGWINDOW, HOPPINGWINDOW, SLIDINGWINDOW или SESSIONWINDOW).

### <a name="method--deaccumulate"></a>Метод deaccumulate()

Метод deaccumulate() повторно определяет состояние на основе значений предыдущего состояния и текущего события. Этот метод вызывается, когда событие покидает окно SLIDINGWINDOW или SESSIONWINDOW.

### <a name="method--deaccumulatestate"></a>Метод deaccumulateState()

Метод deaccumulateState() повторно определяет состояние на основе предыдущего состояния и состояния перехода. Этот метод вызывается, когда набор событий покидает окно HOPPINGWINDOW.

### <a name="method--computeresult"></a>Метод computeResult()

Метод computeResult() возвращает результат статистического выражения на основе текущего состояния. Этот метод вызывается в конце временного окна (TUMBLINGWINDOW, HOPPINGWINDOW, SLIDINGWINDOW или SESSIONWINDOW).

## <a name="javascript-uda-supported-input-and-output-data-types"></a>Поддерживаемые типы входных и выходных данных для пользовательских статистических выражений JavaScript
Типы данных для пользовательских статистических выражений JavaScript приведены в разделе **Преобразование типов Stream Analytics и JavaScript** статьи [Определяемые пользователем функции JavaScript в Azure Stream Analytics](stream-analytics-javascript-user-defined-functions.md).

## <a name="adding-a-javascript-uda-from-the-azure-portal"></a>Добавление пользовательского статистического выражения JavaScript на портале Azure

Ниже рассматривается процесс создания пользовательского статистического выражения на портале. В примере, который здесь используется, вычисляются средневзвешенные по времени значения.

Теперь создадим пользовательское статистическое выражение JavaScript в существующем задании ASA, выполнив приведенные инструкции.

1. Выполните вход на портал Azure и найдите задание Stream Analytics.
1. Щелкните ссылку "Функции" в разделе **Топология задания**.
1. Щелкните значок **Добавить**, чтобы добавить новую функцию.
1. В представлении "Новая функция" выберите тип функции **JavaScript UDA**. В редакторе отобразится шаблон пользовательского статистического выражения по умолчанию.
1. Введите "TWA" в качестве псевдонима пользовательского статистического выражения и измените реализацию функции, как показано ниже.

    ```JavaScript
    // Sample UDA which calculate Time-Weighted Average of incoming values.
    function main() {
        this.init = function () {
            this.totalValue = 0.0;
            this.totalWeight = 0.0;
        }

        this.accumulate = function (value, timestamp) {
            this.totalValue += value.level * value.weight;
            this.totalWeight += value.weight;

        }

        // Uncomment below for AccumulateDeaccumulate implementation
        /*
        this.deaccumulate = function (value, timestamp) {
            this.totalValue -= value.level * value.weight;
            this.totalWeight -= value.weight;
        }

        this.deaccumulateState = function (otherState){
            this.state -= otherState.state;
            this.totalValue -= otherState.totalValue;
            this.totalWeight -= otherState.totalWeight;
        }
        */

        this.computeResult = function () {
            if(this.totalValue == 0) {
                result = 0;
            }
            else {
                result = this.totalValue/this.totalWeight;
            }
            return result;
        }
    }
    ```

1. Щелкните "Сохранить", и пользовательское статистическое выражение (UDA) появится в списке функций.

1. Выберите новую функцию TWA, и вы сможете проверить ее определение.

## <a name="calling-javascript-uda-in-asa-query"></a>Вызов пользовательского статистического выражения JavaScript в запросе ASA

На портале Azure откройте задание, измените запрос и вызовите функцию TWA() с обязательным префиксом "uda". Пример:

```SQL
WITH value AS
(
    SELECT
    NoiseLevelDB as level,
    DurationSecond as weight
FROM
    [YourInputAlias] TIMESTAMP BY EntryTime
)
SELECT
    System.Timestamp as ts,
    uda.TWA(value) as NoseDoseTWA
FROM value
GROUP BY TumblingWindow(minute, 5)
```

## <a name="testing-query-with-uda"></a>Проверка запроса с пользовательским статистическим выражением

Создайте локальный JSON-файл с приведенным ниже содержимым, передайте его в задание Stream Analytics и проверьте приведенный выше запрос.

```JSON
[
  {"EntryTime": "2017-06-10T05:01:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 22.0},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 81, "DurationSecond": 37.8},
  {"EntryTime": "2017-06-10T05:02:00-07:00", "NoiseLevelDB": 85, "DurationSecond": 26.3},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 95, "DurationSecond": 13.7},
  {"EntryTime": "2017-06-10T05:03:00-07:00", "NoiseLevelDB": 88, "DurationSecond": 10.3},
  {"EntryTime": "2017-06-10T05:05:00-07:00", "NoiseLevelDB": 103, "DurationSecond": 5.5},
  {"EntryTime": "2017-06-10T05:06:00-07:00", "NoiseLevelDB": 99, "DurationSecond": 23.0},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 1.76},
  {"EntryTime": "2017-06-10T05:07:00-07:00", "NoiseLevelDB": 79, "DurationSecond": 17.9},
  {"EntryTime": "2017-06-10T05:08:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 27.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 91, "DurationSecond": 17.1},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 115, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:09:00-07:00", "NoiseLevelDB": 80, "DurationSecond": 28.3},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 55, "DurationSecond": 18.2},
  {"EntryTime": "2017-06-10T05:10:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 25.8},
  {"EntryTime": "2017-06-10T05:11:00-07:00", "NoiseLevelDB": 83, "DurationSecond": 11.4},
  {"EntryTime": "2017-06-10T05:12:00-07:00", "NoiseLevelDB": 89, "DurationSecond": 7.9},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 112, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:15:00-07:00", "NoiseLevelDB": 93, "DurationSecond": 9.7},
  {"EntryTime": "2017-06-10T05:18:00-07:00", "NoiseLevelDB": 96, "DurationSecond": 3.7},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 108, "DurationSecond": 0.99},
  {"EntryTime": "2017-06-10T05:20:00-07:00", "NoiseLevelDB": 113, "DurationSecond": 25.1},
  {"EntryTime": "2017-06-10T05:22:00-07:00", "NoiseLevelDB": 110, "DurationSecond": 5.3}
]
```

## <a name="get-help"></a>Получить справку

Для получения дополнительной помощи воспользуйтесь [страницей вопросов и ответов о Microsoft Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Дальнейшие действия

* [Введение в Azure Stream Analytics](stream-analytics-introduction.md)
* [Приступая к работе с Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Масштабирование заданий в службе Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics query language reference](/stream-analytics-query/stream-analytics-query-language-reference) (Справочник по языку запросов Azure Stream Analytics).
* [Azure Stream Analytics management REST API reference](/rest/api/streamanalytics/) (Справочник по API-интерфейсу REST для управления Stream Analytics).
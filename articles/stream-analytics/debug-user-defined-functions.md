---
title: Отладка определяемых пользователем функций в Azure Stream Analytics
description: В этой статье описывается, как выполнять отладку определяемых пользователем функций в Azure Stream Analytics.
author: jenssuessmeyer
ms.author: jenss
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/24/2020
ms.custom: devx-track-js
ms.openlocfilehash: 7f5a24e4d351d7c133251900a80c70094b84cc4e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019437"
---
# <a name="debug-user-defined-functions-in-azure-stream-analytics"></a>Отладка определяемых пользователем функций в Azure Stream Analytics 

Если определяемые пользователем функции (UDF) не работают должным образом, их необходимо отладить для поиска проблемы. Вы можете выполнять отладку UDF для заданий Stream Analytics, когда задания выполняются локально с помощью [Visual Studio Code](visual-studio-code-local-run-live-input.md) или [Visual Studio](stream-analytics-vs-tools-local-run.md).

При локальном запуске задания в динамическом потоке входных данных он имитирует только выполнение облачной Azure Stream Analyticsной подсистемы на одном узле. Локальное тестирование данных в режиме реального времени не может заменить производительность и масштабируемость, выполняемое в облаке, но вы экономите время во время функционального тестирования, не требуя от отправки задания в облако каждый раз, когда вы хотите его протестировать. Кроме того, политики времени отключены для локального запуска с использованием локальных или демонстрационных данных, но политики времени поддерживаются для динамического тестирования данных.

## <a name="pick-your-language"></a>Выберите язык

Пользовательские функции можно писать для Azure Stream Analytics с помощью .NET (C#) или JavaScript. 

### <a name="functions-in-c"></a>Функции в C # 

При [написании UDF .NET с помощью Visual Studio](stream-analytics-edge-csharp-udf-methods.md)вы получаете тот же уровень поддержки, что и для любого проекта класса .NET. Эта поддержка включает следующее.

* Поддержка компиляции, например проверка синтаксиса и поддержка компилятора.

* Возможность добавления, сборки и ссылки на проект и артефакты C# в решении Stream Analytics. 

* Простое повторное использование кода, инкапсулированного в проекте, который является общим для совместного использования. 

* Поддержка отладки в Visual Studio напрямую. Задайте проект Stream Analytics как запуск проекта и задайте точки останова в коде C#. Затем нажмите клавишу **F5** для отладки кода c# так же, как и для любого другого проекта C#. 

### <a name="functions-in-javascript"></a>Функции в JavaScript

JavaScript — это еще один вариант для создания функций в Stream Analytics. Код JavaScript непосредственно помещается в область функций проекта Stream Analytics, что делает совместное использование нескольких проектов более сложным.

Компиляция происходит при компиляции или выполнении Stream Analytics проекта. Возможность найти проблему только во время выполнения будет выше. Поддержка отладки для функций JavaScript в Stream Analytics напрямую отсутствует.

## <a name="debug-options-for-javascript"></a>Параметры отладки для JavaScript

Поскольку отсутствует поддержка отладки для функций JavaScript в Stream Analytics напрямую, можно выполнить отладку, инкапсулировать функцию на HTML-сайте и получая выходные данные.

В следующем примере показано, как выполнить отладку определяемых пользователем функций JavaScript с некоторыми ограничениями в интегрированной среде выполнения в [Visual Studio Code](quick-create-visual-studio-code.md).

### <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, убедитесь, что проект Azure Stream Analytics содержит следующие элементы:

* Входные данные 
* Выходные данные 
* Stream Analyticsный запрос (. asaql) 
* Конфигурация задания Stream Analytics (JobConfig.jsна)
* Определяемая пользователем функция JavaScript

### <a name="prepare-files"></a>Подготовка файлов

На следующем рисунке файл запроса *. asaql* включает только вызов функции UDF, *фксчаркаунт*. Это изменение гарантирует, что вы по-прежнему сможете компилировать проект после внесения изменений.

> [!div class="mx-imgBorder"]
> ![Файл тестового запроса Stream Analytics](./media/debug-user-defined-functions/asaql-file.png)

Создайте дополнительную папку в **тестах** для размещения файла теста, который вызывается для выполнения теста с помощью функции JavaScript. В этом примере имя папки — *фксчаркаунт* , а имя теста — *Test_UDF.js*. 

На рисунке ниже показан код в тестовом файле, который загружает файл функции и выполняет функцию. Этот пример прост, но вы можете загрузить дополнительные файлы тестовых данных и циклически пройти все дополнительные тесты, чтобы получить выходные данные. Нотация вызова функции незначительно отличается от распространенных вызовов, так как на файл имеется ссылка и он не загружен в среду выполнения, что позволяет выполнять отладку. 

> [!div class="mx-imgBorder"]
> ![Файл теста Stream Analytics](./media/debug-user-defined-functions/test-file.png)

В функции добавьте следующие строки кода в файл, чтобы предоставить методы. Они не влияют на возможность компиляции кода в Visual Studio Code.

```javascript
var methods = {};
methods.fxchartCount = fxchartCount;
exports.data = methods;
``` 

> [!div class="mx-imgBorder"]
> ![Stream Analytics UDF JavaScript](./media/debug-user-defined-functions/udf-file.png)
  
## <a name="install-debug-support"></a>Установка поддержки отладки

Для отладки необходимо [загрузить](https://nodejs.org/en/download/) и установить **node.js**. Установите правильную версию в соответствии с используемой платформой. После установки среды выполнения node.js перезапустите Visual Studio Code, чтобы внести изменения. 

Выберите **выполнить и отладить** или нажмите клавиши **CTRL + SHIFT + D** , чтобы начать отладку. Появится поле со списком, в котором можно выбрать **node.js** в качестве среды выполнения. Если установлен только node.js, он используется по умолчанию. При необходимости вы сможете пошагово пройти код и во вспомогательный файл с помощью F11. 

> [!div class="mx-imgBorder"]
> ![Stream Analytics запуска и отладки определяемой пользователем функции](./media/debug-user-defined-functions/run-debug-udf.png)

### <a name="debug-user-defined-aggregates"></a>Отладка определяемых пользователем статистических выражений 

Для отладки определяемых пользователем статистических выражений можно использовать метод Debug для пользовательских функций JavaScript (UDA). В этом примере UDA добавляется в файл запроса *asaql* и в тестовый файл.

Как и в случае с определяемой пользователем функцией, вы включаете вызов UDA, чтобы убедиться, что проект будет компилироваться после внесения изменений. 

> [!div class="mx-imgBorder"]
> ![Добавление UDA в asaql](./media/debug-user-defined-functions/asaql-uda.png)

В файле *Test_UDA.js* вы ссылаетесь на файл UDA, как и с помощью определяемой пользователем функции. Кроме того, вы вызываете методы `main()` , `init()` и `accumulate()` . `accumulate()`Метод вызывается в цикле для помещения значений в стек состояния. `computeresult()`Метод вызывается для создания окончательного запроса. 

> [!div class="mx-imgBorder"]
> ![Файл теста UDA](./media/debug-user-defined-functions/uda-test.png)

Как и в примере UDF, некоторый код необходимо добавить в UDA, чтобы предоставить соответствующие методы.

```javascript
var methods = {};
methods.main = main;
methods.init = main.init;
methods.accumulate = main.accumulate;
methods.computeResult = main.computeResult;
exports.data = methods;
``` 

> [!div class="mx-imgBorder"]
> ![Код, добавленный в UDA](./media/debug-user-defined-functions/uda-expose-methods.png)

Выберите **выполнить и отладить** или нажмите клавиши **CTRL + SHIFT + D** , чтобы начать отладку. Появится поле со списком, в котором можно выбрать **node.js** в качестве среды выполнения. Если установлен только node.js, он используется по умолчанию. При необходимости вы сможете пошагово пройти код и во вспомогательный файл с помощью F11.

> [!div class="mx-imgBorder"]
> ![Stream Analytics запуска и отладки UDA](./media/debug-user-defined-functions/run-debug-uda.png)


## <a name="next-steps"></a>Дальнейшие действия

* [Разработка и отладка заданий Azure Stream Analytics локально](develop-locally.md)
* [Отладка запросов Azure Stream Analytics локально с помощью схемы заданий в Visual Studio](debug-locally-using-job-diagram.md)
* [Определяемые пользователем функции в Azure Stream Analytics](functions-overview.md)
 

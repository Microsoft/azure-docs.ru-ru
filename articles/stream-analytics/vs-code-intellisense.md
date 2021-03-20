---
title: IntelliSense в средствах Azure Stream Analytics для Visual Studio Code
description: В этой статье описывается, как использовать функции IntelliSense в Azure Stream Analytics инструментов для Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 4/11/2020
ms.topic: how-to
ms.openlocfilehash: 988b9d364f09a4c88bcaab1fe81ae80c8de5d4d9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96491576"
---
# <a name="intellisense-in-azure-stream-analytics-tools-for-visual-studio-code"></a>IntelliSense в средствах Azure Stream Analytics для Visual Studio Code

Технология IntelliSense доступна для [Stream Analytics языка запросов](/stream-analytics-query/stream-analytics-query-language-reference?bc=https%253a%2f%2fdocs.microsoft.com%2fazure%2fbread%2ftoc.json&toc=https%253a%2f%2fdocs.microsoft.com%2fazure%2fstream-analytics%2ftoc.json) в [Azure Stream Analytics tools for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-bigdatatools.vscode-asa&ssr=false#overview). IntelliSense — это вспомогательное средство для написания кода, включающее несколько возможностей: "Список членов", "Сведения о параметрах", "Краткие сведения" и "Завершить слово". Функции IntelliSense иногда вызываются другими именами, такими как "завершение кода", "помощь в содержимом" и "указанием кода".

![Демонстрационная версия IntelliSense](./media/vs-code-intellisense/intellisense.gif)

## <a name="intellisense-features"></a>функции IntelliSense

Функции IntelliSense в Stream Analytics средствах для VS Code на основе языковой службы. Языковая служба анализирует исходный код и предоставляет интеллектуальные завершения кода на основе семантики языка. Если языковая служба знает о возможных завершениях выполнения, при вводе появляется предложение IntelliSense. Если продолжить ввод, список членов, таких как переменные и методы, фильтруется, добавляя только те элементы, которые содержат введенные символы. При нажатии клавиш `Tab` или `Enter` IntelliSense вставляет выбранный элемент.

Вы можете активировать IntelliSense в любом окне редактора, введя символ триггера, например символ точки `.` .

![Автозавершение IntelliSense](./media/vs-code-intellisense/auto-completion.gif)

> [!TIP]
> Мини-приложение предложений поддерживает фильтрацию CamelCase. Для ограничения предложений можно ввести буквы в верхнем регистре в имени метода. Например, "кра" быстро отобразит "createApplication".

### <a name="types-of-completions"></a>Типы завершений

Stream Analytics средства для VS Code IntelliSense предлагает различные типы завершения, включая предложения языка сервера, фрагменты кода и простые текстовые автозавершения на основе слов.

|Completion     |  Type       |
| ----- | ------- |
| Ключевые слова | `keyword`
| Функции | `build-in function`, `user defined function`  |
| Имя набора данных| `input`, `output`, `intermediate result set`|
| Имя столбца набора данных|`input`, `intermediate result set`|

#### <a name="name-completion"></a>Завершение имени

Помимо автозавершения ключевых слов, Stream Analytics средства для VS Code считывает список входных и выходных имен задания, а также имена столбцов в источниках данных при их настройке. Расширение запоминает эти сведения для предоставления возможностей завершения имен, которые полезны для ввода инструкций с несколькими нажатиями клавиш:

При написании кода не нужно выходить из редактора, чтобы выполнять поиск по именам входных данных задания, имени вывода и именам столбцов. Вы можете оставаться в своем контексте, находить нужные сведения, вставлять элементы непосредственно в код и выполнять ввод с помощью IntelliSense.

Обратите внимание, что необходимо настроить локальный вход или вход в реальном времени и сохранить файл конфигурации, чтобы иметь возможность использовать завершение имен.

![завершение имени](./media/vs-code-intellisense/name-completion.gif)

### <a name="parameter-info"></a>Сведения о параметрах

Параметр IntelliSense **сведения о параметре** открывает список параметров, который предоставляет сведения о числе, именах и типах параметров, необходимых для функции. Параметр, выделенный полужирным шрифтом, указывает следующий параметр, необходимый при вводе функции.

Список параметров отображается также и для вложенных функций. Если функция вводится в качестве параметра другой функции, в списке параметров перечисляются параметры для этой вложенной функции. Затем, когда список параметров вложенной функции заполнен, в списке продолжается отображение параметров внешней функции.

![сведения о параметре](./media/vs-code-intellisense/parameter-info.gif)

### <a name="quick-info"></a>Краткие сведения

Как указано в языковой службе, можно просмотреть **краткие сведения** для каждого идентификатора в коде. Примеры идентификаторов: вход, выход, промежуточный результирующий набор или функция. При перемещении указателя мыши над идентификатором его объявление отображается во всплывающем окне. Отображаются свойства и схемы данных для входных данных, если они настроены, а также промежуточный набор.

![Краткие сведения](./media/vs-code-intellisense/quick-info.gif)

## <a name="troubleshoot-intellisense"></a>Устранение неполадок IntelliSense

Эта проблема вызвана отсутствием входной конфигурации, которая предоставляет данные. Можно проверить правильность настройки [локального ввода](visual-studio-code-local-run.md#define-a-local-input) или [ввода в реальном времени](visual-studio-code-local-run-live-input.md#define-a-live-stream-input) .

## <a name="next-steps"></a>Дальнейшие действия

* [Краткое руководство. Создание задания Azure Stream Analytics в Visual Studio Code](quick-create-visual-studio-code.md)
* [Локальное тестирование запросов Stream Analytics с использованием примера данных и Visual Studio Code](visual-studio-code-local-run.md)
* [Тестирование Stream Analytics запросов локально для входа в динамический поток с помощью Visual Studio Code](visual-studio-code-local-run-live-input.md)
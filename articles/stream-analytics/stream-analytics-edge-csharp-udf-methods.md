---
title: Разработка функций .NET Standard для Azure Stream Analyticsных заданий (Предварительная версия)
description: Сведения о написании определяемых пользователем функций C# для Stream Analyticsных заданий.
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/10/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 52fa6f05db5452a2e7b8ec4f93d65525873c8c7e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98020576"
---
# <a name="develop-net-standard-user-defined-functions-for-azure-stream-analytics-jobs-preview"></a>Разработка .NET Standard определяемых пользователем функций для Azure Stream Analytics заданий (Предварительная версия)

Azure Stream Analytics предлагает язык запросов, аналогичный SQL, для выполнения преобразований и вычислений в потоках данных событий. Существует множество встроенных функций, но некоторые сложные сценарии требуют дополнительных возможностей. С помощью пользовательских функций .NET Standard вы можете вызывать свои функции, написанные на любом языке .NET Standard (C#, F# и т. д.) для расширения языка запросов Stream Analytics. Пользовательские функции позволяют выполнять сложные математические вычисления, импортировать пользовательские модели ML с помощью ML.NET и использовать пользовательскую логику заполнения отсутствующих данных. Пользовательские функции для заданий Stream Analytics сейчас находятся на этапе предварительной версии, и их не следует использовать в рабочих нагрузках.

Определяемая пользователем функция .NET для облачных заданий доступна в:
* центрально-западная часть США
* Северная Европа
* Восточная часть США
* западная часть США
* восточная часть США 2
* Западная Европа

Если вы хотите использовать эту функцию в любом другом регионе, вы можете [запросить доступ](https://aka.ms/ccodereqregion).

## <a name="package-path"></a>Путь к пакету

Формат любых пакетов пользовательских функций имеет путь `/UserCustomCode/CLR/*`. Библиотеки динамической компоновки (DLL) и ресурсы копируются в папку `/UserCustomCode/CLR/*`, которая позволяет изолировать библиотеки DLL пользователей от системы и библиотек DLL Azure Stream Analytics. Этот путь к пакету используется для всех функций независимо от способа их использовать.

## <a name="supported-types-and-mapping"></a>Поддерживаемые типы и сопоставление

Для Azure Stream Analytics значений, используемых в C#, их необходимо маршалировать из одной среды в другую. Для всех входных параметров определяемой пользователем функции выполняется маршалирование. Каждый тип Azure Stream Analytics имеет соответствующий тип в C#, как показано в следующей таблице:

|**Тип Azure Stream Analytics** |**Тип C#** |
|---------|---------|
|BIGINT | long |
|FLOAT | double |
|nvarchar(max) | строка |
|DATETIME | Дата и время |
|Record | Dictionary\<string, object> |
|Array | Object [] |

То же самое справедливо и при необходимости маршалирования данных из C# в Azure Stream Analytics, что происходит с выходным значением определяемой пользователем функции. В следующей таблице показано, какие типы поддерживаются:

|**Тип C#**  |**Тип Azure Stream Analytics**  |
|---------|---------|
|long  |  BIGINT   |
|double  |  FLOAT   |
|строка  |  nvarchar(max)   |
|Дата/время  |  dateTime   |
|struct  |  Record   |
|object  |  Record   |
|Object []  |  Array   |
|Dictionary\<string, object>  |  Record   |

## <a name="develop-a-udf-in-visual-studio-code"></a>Разработка определяемой пользователем функции в Visual Studio Code

[Visual Studio Code средства для Azure Stream Analytics](quick-create-visual-studio-code.md) упрощают создание пользовательских функций, тестирование заданий локально (даже вне сети) и публикацию задания Stream Analytics в Azure.

Существует два способа реализации определяемых пользователем функций .NET Standard в Visual Studio Codeных инструментах.

* UDF из локальных библиотек DLL
* пользовательская функция из локального проекта;

### <a name="local-project"></a>Локальный проект

Пользовательские функции можно написать в сборке, на которую позже будет ссылаться запрос Azure Stream Analytics. Это рекомендуется для сложных функций, требующих всех возможностей языка .NET Standard помимо языка выражений, таких как процедурная логика и рекурсия. Пользовательские функции из локального проекта можно также использовать, если нужно предоставить логику функций в нескольких запросах Azure Stream Analytics. Добавление UDF в локальный проект позволяет выполнять отладку и тестирование функций локально.

Создание ссылки на локальный проект

1. Создайте новую библиотеку классов .NET Standard на локальном компьютере.
2. Напишите код в классе. Обратите внимание, что классы должны быть определены как *public*, а объекты — как *static public*.
3. Добавьте новый файл конфигурации функции CSharp в проект Azure Stream Analytics и сослаться на проект библиотеки классов CSharp.
4. Настройте путь к сборке в файле конфигурации задания, `JobConfig.json` **кустомкодестораже** разделе. Этот шаг не требуется для локального тестирования.

### <a name="local-dlls"></a>Локальные библиотеки DLL

Также можно ссылаться на локальные библиотеки DLL, включающие определяемые пользователем функции.

### <a name="example"></a>Пример

В этом примере **кшарпудфпрожект** является проектом библиотеки классов C#, а **асаудфдемо** — проектом Azure Stream Analytics, который будет ссылаться на **кшарпудфпрожект**.

:::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-demo.png" alt-text="Azure Stream Analytics проект в Visual Studio Code":::

Следующая определяемая пользователем функция имеет функцию, которая умножает целочисленное значение на целое, чтобы получить квадрат целого числа. Классы должны быть определены как *открытые* , а объекты должны быть определены как *статические открытые*.

```csharp
using System;

namespace CSharpUDFProject
{
    // 
    public class Class1
    {
        public static Int64 SquareFunction(Int64 a)
        {
            return a * a;
        }
    }
}
```

Ниже описано, как добавить функцию UDF C# в проект Stream Analytics.

1. Щелкните правой кнопкой мыши папку **функции** и выберите команду **Добавить элемент**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function.png" alt-text="Добавить новую функцию в Azure Stream Analytics проект":::

2. Добавьте функцию C# **скуарефунктион** в проект Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function-2.png" alt-text="Выбор функции CSharp из Stream Analytics проекта в VS Code":::

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function-name.png" alt-text="Введите имя функции CSharp в VS Code":::

3. В конфигурации функции C# выберите **выбрать библиотеку путь к проекту** , чтобы выбрать проект C# из раскрывающегося списка, и выберите **Build Project (собрать проект** ), чтобы построить проект. Затем выберите **выбрать класс** и **выберите метод** , чтобы выбрать связанный класс и имя метода из раскрывающегося списка. Для ссылки на методы, типы и функции в Stream Analytics запросе классы должны быть определены как *открытые* , а объекты должны быть определены как *статические открытые*.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-choose-project.png" alt-text="VS Code конфигурации Stream Analytics с резкими функциями":::

    Если вы хотите использовать определяемую пользователем функцию C# из библиотеки DLL, выберите выбрать **путь к библиотеке DLL** , чтобы выбрать библиотеку DLL. Затем выберите **выбрать класс** и **выберите метод** , чтобы выбрать связанный класс и имя метода из раскрывающегося списка.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-choose-dll.png" alt-text="Настройка функции C# Stream Analytics":::

4. Вызовите определяемую пользователем функцию в Azure Stream Analytics запросе.

   ```sql
    SELECT price, udf.SquareFunction(price)
    INTO Output
    FROM Input 
   ```

5. Перед отправкой задания в Azure настройте путь к пакету в файле конфигурации задания, `JobConfig.json` **кустомкодестораже** разделе. Используйте **SELECT из подписки** в CodeLens, чтобы выбрать подписку и выбрать учетную запись хранения и имя контейнера из раскрывающегося списка. Оставьте **путь** по умолчанию. Этот шаг не требуется для локального тестирования.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-configure-storage-account.png" alt-text="Выбор пути к библиотеке":::

## <a name="develop-a-udf-in-visual-studio"></a>Разработка определяемой пользователем функции в Visual Studio

Существует три способа реализации UDF в средствах Visual Studio.

* файлы CodeBehind в проекте ASA;
* пользовательская функция из локального проекта;
* существующий пакет из учетной записи хранения Azure.

### <a name="codebehind"></a>CodeBehind

Вы можете написать пользовательские функции в CodeBehind **Script.asql**. Инструменты Visual Studio автоматически скомпилируют файл CodeBehind в файл сборки. При отправке задания в Azure сборки будут упакованы в ZIP-файл и отправлены в учетную запись хранения. Чтобы узнать, как написать пользовательскую функцию на C# с помощью CodeBehind, воспользуйтесь руководством [Пользовательские функции на C# для заданий Edge в Stream Analytics](stream-analytics-edge-csharp-udf.md). 

### <a name="local-project"></a>Локальный проект

Для ссылки на локальный проект в Visual Studio:

1. Создание новой библиотеки классов .NET Standard в решении
2. Напишите код в классе. Обратите внимание, что классы должны быть определены как *public*, а объекты — как *static public*. 
3. Построить проект. Средства упакуют все артефакты в папке bin и передадут ZIP-файл в учетную запись хранения. Для внешних ссылок используйте ссылку на сборку, а не пакет NuGet.
4. Сошлитесь на новый класс в проекте Azure Stream Analytics.
5. Добавьте новую функцию в проект Azure Stream Analytics.
6. Настройте путь к сборке в файле конфигурации задания `JobConfig.json`. В качестве пути к сборке укажите **ссылку на локальный проект или код программной части**.
7. Выполните повторную сборку проекта функций и проекта Azure Stream Analytics.  

### <a name="example"></a>Пример

В этом примере **удфтест** является проектом библиотеки классов C#, а **асаудфдемо** — проектом Azure Stream Analytics, который будет ссылаться на **удфтест**.

:::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-demo.png" alt-text="Проект IoT Edge в Azure Stream Analytics в Visual Studio":::

1. Создайте проект на C#, который позволит добавить ссылку на пользовательскую функцию C# из запроса Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-build-project.png" alt-text="Создание проекта IoT Edge в Azure Stream Analytics в Visual Studio":::

2. Добавьте ссылку на проект C# в проекте ASA. Щелкните правой кнопкой мыши узел "Ссылки" и выберите "Добавить ссылку".

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-reference.png" alt-text="Добавление ссылки на проект C# в Visual Studio":::

3. Выберите имя проекта C# в списке.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-choose-project-name.png" alt-text="Выбор имени проекта C# в списке ссылок":::

4. В списке **Ссылки** в **обозревателе решений** должен отобразиться проект **UDFTest**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-added-reference.png" alt-text="Просмотр ссылки на пользовательскую функцию в обозревателе решений":::

5. Щелкните правой кнопкой мыши папку **Функции** и выберите **Новый элемент**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-csharp-function.png" alt-text="Добавление нового элемента в функции в решении Edge в Azure Stream Analytics":::

6. Добавьте функцию C# **SquareFunction.json** в проект Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-csharp-function-2.png" alt-text="Выбор функции C# в списке элементов Edge в Stream Analytics в Visual Studio":::

7. Дважды щелкните функцию в **обозревателе решений**, чтобы открыть диалоговое окно настройки.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-csharp-function-config.png" alt-text="Настройка функции C# в Visual Studio":::

8. В окне настройки функции C# выберите **Загрузить из ссылки на проект ASA** и щелкните имена связанных сборки, класса и метода в раскрывающемся списке. Для ссылки на методы, типы и функции в Stream Analytics запросе классы должны быть определены как *открытые* , а объекты должны быть определены как *статические открытые*.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-asa-csharp-function-config.png" alt-text="Настройка функции «Stream Analytics с резкими функциями в Visual Studio»":::

## <a name="existing-packages"></a>Существующие пакеты

Вы можете создать пользовательские функции .NET Standard в любой IDE на ваш выбор и вызвать их из запроса Azure Stream Analytics. Сначала скомпилируйте код и упакуйте все библиотеки DLL. Формат пакета имеет путь `/UserCustomCode/CLR/*`. После отправьте `UserCustomCode.zip` в корень контейнера в учетной записи хранения Azure.

После отправки ZIP-файлов пакетов в учетную запись хранения Azure вы можете использовать функции в запросах Azure Stream Analytics. Все, что нужно сделать, — включить сведения о хранилище в конфигурацию задания Stream Analytics. Протестировать функцию локально с помощью этого параметра невозможно, так как инструменты Visual Studio не смогут скачать ваш пакет. Путь к пакету анализируется непосредственно в службе. 

Настройте путь к сборке в файле конфигурации задания `JobConfig.json`:

Разверните раздел **Настройка пользовательского кода** и укажите следующие предложенные значения в конфигурации:

   |**Параметр**|**Рекомендуемое значение**|
   |-------|---------------|
   |Ресурс параметров глобального хранилища|Выберите источник данных из текущей учетной записи|
   |Подписка на параметры глобального хранилища| < your subscription >|
   |Учетная запись хранения параметров глобального хранилища| < ваша учетная запись хранения >|
   |Ресурс параметров хранилища настраиваемого кода|Выберите источник данных из текущей учетной записи|
   |Учетная запись хранения параметров хранилища настраиваемого кода|< ваша учетная запись хранения >|
   |Контейнер параметров хранилища настраиваемого кода|< ваш контейнер хранилища >|
   |Источник сборки пользовательского кода|Существующие пакеты сборки из облака|
   |Источник сборки пользовательского кода|UserCustomCode.zip|

## <a name="user-logging"></a>Ведение журнала пользователя

Механизм ведения журнала позволяет собирать определенные сведения во время выполнения задания. Данные журнала можно использовать для отладки или оценки правильности пользовательского кода в реальном времени.

`StreamingContext`Класс позволяет публиковать диагностические данные с помощью `StreamingDiagnostics.WriteError` функции. В приведенном ниже коде показан интерфейс, предоставляемый Azure Stream Analytics.

```csharp
public abstract class StreamingContext
{
    public abstract StreamingDiagnostics Diagnostics { get; }
}

public abstract class StreamingDiagnostics
{
    public abstract void WriteError(string briefMessage, string detailedMessage);
}
```

`StreamingContext` передается в качестве входного параметра в метод UDF и может использоваться в UDF для публикации пользовательских данных журнала. В приведенном ниже примере `MyUdfMethod` определяет входные **данные** , предоставляемые запросом, и входные данные **контекста** `StreamingContext` , предоставляемые подсистемой среды выполнения. 

```csharp
public static long MyUdfMethod(long data, StreamingContext context)
{
    // write log
    context.Diagnostics.WriteError("User Log", "This is a log message");
    
    return data;
}
```

`StreamingContext`Значение не должно передаваться SQL Query. Azure Stream Analytics предоставляет объект контекста автоматически при наличии входного параметра. Использование параметра не `MyUdfMethod` изменится, как показано в следующем запросе:

```sql
SELECT udf.MyUdfMethod(input.value) as udfValue FROM input
```

Доступ к сообщениям журнала можно получить с помощью [журналов диагностики](data-errors.md).

## <a name="limitations"></a>Ограничения

Предварительная версия пользовательских функций сейчас имеет следующие ограничения.

* .NET Standard UDF могут быть созданы только в Visual Studio Code или Visual Studio и опубликованы в Azure. Версии пользовательских функций .NET Standard только для чтения можно просмотреть в разделе **Функции** на портале Azure. Создание функций .NET Standard не поддерживается на портале Azure.

* Редактор запросов на портале Azure отображает ошибку при использовании пользовательской функции .NET Standard на портале. 

* Так как пользовательский код имеет общий контекст с подсистемой Azure Stream Analytics, он не может содержать ссылки на объекты с конфликтующим значением namespace/dll_name в коде Azure Stream Analytics. Например, ссылки на *Newtonsoft Json* запрещены.

* Вспомогательные файлы, включенные в проект, копируются в ZIP-файл пользовательского кода, который используется при публикации задания в облаке. Все файлы во вложенных папках копируются непосредственно в корень пользовательской папки пользовательского кода в облаке при распаковке. При распаковке ZIP-файл преобразуется в плоский формат.

* Пользовательский код пользовательского кода не поддерживает пустые папки. Не добавляйте пустые папки в вспомогательные файлы проекта.

## <a name="next-steps"></a>Дальнейшие действия

* [Руководство. Написание определяемой пользователем функции C# для задания Azure Stream Analytics (Предварительная версия)](stream-analytics-edge-csharp-udf.md)
* [Руководство. Определяемые пользователем функции JavaScript в Azure Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
* [Создание задания Azure Stream Analytics в Visual Studio Code](quick-create-visual-studio-code.md)
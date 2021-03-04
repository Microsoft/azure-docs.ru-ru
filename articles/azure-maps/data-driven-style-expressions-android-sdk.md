---
title: Выражения стиля на основе данных в картах Android | Карты Microsoft Azure
description: Сведения о выражениях в стиле, управляемых данными. Сведения об использовании этих выражений в Azure Maps пакет SDK для Android для настройки стилей в картах.
author: rbrundritt
ms.author: richbrun
ms.date: 2/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: 1babf1feb550109486089c45469ab4ce32f72cb3
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102097420"
---
# <a name="data-driven-style-expressions-android-sdk"></a>Выражения стиля на основе данных (пакет SDK для Android)

Выражения позволяют применять бизнес-логику к параметрам стилей, которые задают свойства, определенные в каждой фигуре источника данных. Выражения могут фильтровать данные в источнике данных или слое. Выражения могут состоять из условной логики, например if-операторы. Для работы с данными можно использовать следующие операторы: String, логические операторы и математические операторы.

Стили, управляемые данными, сокращают объем кода, необходимого для реализации бизнес-логики вокруг стилей. При использовании с слоями выражения вычисляются во время отрисовки в отдельном потоке. Эта функция обеспечивает повышенную производительность по сравнению с оценкой бизнес-логики в потоке пользовательского интерфейса.

Azure Maps пакет SDK для Android поддерживает почти все те же выражения стилей, что и веб-пакет SDK Azure Maps, поэтому все те же концепции, которые описаны в [выражениях в стиле на основе данных (веб-пакет SDK)](data-driven-style-expressions-web-sdk.md) , можно переносить в приложение Android. Все выражения стиля в пакет SDK для Androidах Azure Maps доступны в `com.microsoft.azure.maps.mapcontrol.options.Expression` пространстве имен. Существует множество различных типов выражений стилей.

| Тип выражений | Описание |
|---------------------|-------------|
| [Логические выражения](#boolean-expressions) | Логические выражения предоставляют набор выражений логических операторов для оценки логических сравнений. |
| [Выражения цвета](#color-expressions) | Выражения цвета упрощают создание цветовых значений и манипуляций с ними. |
| [Условные выражения](#conditional-expressions) | Условные выражения предоставляют логические операции, аналогичные операторам if. |
| [Выражения данных](#data-expressions) | Предоставляет доступ к данным свойств в компоненте. |
| [Выражения интерполяции и шага](#interpolate-and-step-expressions) | Выражения интерполяции и шага можно использовать для вычисления значений по кривой с интерполяцией или с пошаговыми функциями. |
| [Выражения на основе JSON](#json-based-expressions) | Упрощает повторное использование выражений на основе JSON необработанных стилей, созданных для веб-пакета SDK, с помощью пакет SDK для Android. |  
| [Выражения для конкретных слоев](#layer-specific-expressions) | Специальные выражения, применимые только к одному слою. |
| [Математические выражения](#math-expressions) | Предоставляет математические операторы для выполнения вычислений на основе данных в платформе выражений. |
| [Выражения строкового оператора](#string-operator-expressions) | Выражения строкового оператора выполняют операции преобразования строк, таких как объединение и преобразование варианта. |
| [Выражения типа](#type-expressions) | Выражения типа предоставляют средства для тестирования и преобразования различных типов данных, таких как строки, числа и логические значения. |
| [Выражения привязки переменных](#variable-binding-expressions) | Выражения привязки переменных хранят результаты вычисления в переменной и ссылаются на другие части выражения несколько раз без необходимости повторного вычисления сохраненного значения. |
| [Выражение масштаба](#zoom-expression) | Возвращает текущий уровень масштаба для отображения во время отрисовки. |

> [!NOTE]
> Синтаксис выражений во многом идентичен в Java и Котлин. Если у вас есть документация Котлин, но вы видите блоки кода для Java, код будет одинаковым на обоих языках.

Во всех примерах в этом разделе документа используется следующая функция для демонстрации различных способов использования этих выражений.

```json
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.13284, 47.63699]
    },
    "properties": {
        "id": 123,
        "entityType": "restaurant",
        "revenue": 12345,
        "subTitle": "Building 40", 
        "temperature": 64,
        "title": "Cafeteria", 
        "zoneColor": "purple",
        "abcArray": ["a", "b", "c"],
        "array2d": [["a", "b"], ["x", "y"]],
        "_style": {
            "fillColor": "red"
        }
    }
}
```

В следующем коде показано, как вручную создать эту функцию геоjson в приложении.

::: zone pivot="programming-language-java-android"

```Java
//Create a point feature.
Feature feature = Feature.fromGeometry(Point.fromLngLat(-100, 45));

//Add properties to the feature.
feature.addNumberProperty("id", 123);
feature.addStringProperty("entityType", "restaurant");
feature.addNumberProperty("revenue", 12345);
feature.addStringProperty("subTitle", "Building 40");
feature.addNumberProperty("temperature", 64);
feature.addStringProperty("title", "Cafeteria");
feature.addStringProperty("zoneColor", "purple");

JsonArray abcArray = new JsonArray();
abcArray.add("a");
abcArray.add("b");
abcArray.add("c");

feature.addProperty("abcArray", abcArray);

JsonArray array1 = new JsonArray();
array1.add("a");
array1.add("b");

JsonArray array2 = new JsonArray();
array1.add("x");
array1.add("y");

JsonArray array2d = new JsonArray();
array2d.add(array1);
array2d.add(array2);

feature.addProperty("array2d", array2d);

JsonObject style = new JsonObject();
style.addProperty("fillColor", "red");

feature.addProperty("_style", style);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Create a point feature.
val feature = Feature.fromGeometry(Point.fromLngLat(-100, 45))

//Add properties to the feature.
feature.addNumberProperty("id", 123)
feature.addStringProperty("entityType", "restaurant")
feature.addNumberProperty("revenue", 12345)
feature.addStringProperty("subTitle", "Building 40")
feature.addNumberProperty("temperature", 64)
feature.addStringProperty("title", "Cafeteria")
feature.addStringProperty("zoneColor", "purple")

val abcArray = JsonArray()
abcArray.add("a")
abcArray.add("b")
abcArray.add("c")

feature.addProperty("abcArray", abcArray)

val array1 = JsonArray()
array1.add("a")
array1.add("b")

val array2 = JsonArray()
array1.add("x")
array1.add("y")

val array2d = JsonArray()
array2d.add(array1)
array2d.add(array2)

feature.addProperty("array2d", array2d)

val style = JsonObject()
style.addProperty("fillColor", "red")

feature.addProperty("_style", style)
```

::: zone-end

В следующем коде показано, как выполнить десериализацию переведенные версии объекта JSON в функцию геоjson в приложении.

::: zone pivot="programming-language-java-android"

```java
String featureString = "{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[-122.13284,47.63699]},\"properties\":{\"id\":123,\"entityType\":\"restaurant\",\"revenue\":12345,\"subTitle\":\"Building 40\",\"temperature\":64,\"title\":\"Cafeteria\",\"zoneColor\":\"purple\",\"abcArray\":[\"a\",\"b\",\"c\"],\"array2d\":[[\"a\",\"b\"],[\"x\",\"y\"]],\"_style\":{\"fillColor\":\"red\"}}}";

Feature feature = Feature.fromJson(featureString);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val featureString = "{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[-122.13284,47.63699]},\"properties\":{\"id\":123,\"entityType\":\"restaurant\",\"revenue\":12345,\"subTitle\":\"Building 40\",\"temperature\":64,\"title\":\"Cafeteria\",\"zoneColor\":\"purple\",\"abcArray\":[\"a\",\"b\",\"c\"],\"array2d\":[[\"a\",\"b\"],[\"x\",\"y\"]],\"_style\":{\"fillColor\":\"red\"}}}"

val feature = Feature.fromJson(featureString)
```

::: zone-end

## <a name="json-based-expressions"></a>Выражения на основе JSON

Azure Maps веб-пакет SDK также поддерживает выражения стилей на основе данных, представленные с помощью массива JSON. Эти же выражения можно создать повторно с помощью собственного `Expression` класса в пакет SDK для Android. Кроме того, эти выражения на основе JSON можно преобразовать в строку с помощью веб-функции, такой как `JSON.stringify` и, передаваемой в `Expression.raw(String rawExpression)` метод. Например, возьмем следующее выражение JSON.

```javascript
var exp = ['get','title'];
JSON.stringify(exp); // = "['get','title']"
```

Переведенные версия приведенного выше выражения будет иметь `"['get','title']"` вид, и ее можно будет прочитать в пакет SDK для Android следующим образом.

::: zone pivot="programming-language-java-android"

```java
Expression exp = Expression.raw("['get','title']")
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val exp = Expression.raw("['get','title']")
```

::: zone-end

Такой подход позволяет легко использовать выражения стилей в мобильных и веб-приложениях, использующих Azure Maps.

В этом видеоролике представлен обзор стилей, основанных на данных, в Azure Maps.

</br>

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Data-Driven-Styling-with-Azure-Maps/player?format=ny]

## <a name="data-expressions"></a>Выражения данных

Выражения данных предоставляют доступ к данным свойств в компоненте.

| Выражение | Возвращаемый тип | Описание |
|------------|-------------|-------------|
| `accumulated()` | number | Возвращает значение свойства кластера, накопленного до сих пор. |
| `at(number | Expression, Expression)` | value | Извлекает элемент из массива. |
| `geometryType()` | строка | Возвращает тип геометрии компонента: Point, MultiPoint, LineString, MultiLineString, многоугольник, Polygon. |
| `get(string | Expression)` \| `get(string | Expression, Expression)` | value | Возвращает значение свойства из свойств указанного объекта. Возвращает значение null, если запрошенное свойство отсутствует. |
| `has(string | Expression)` \| `has(string | Expression, Expression)` | Логическое | Определяет, имеют ли свойства компонента указанное свойство. |
| `id()` | value | Возвращает идентификатор компонента, если он имеется. |
| `in(string | number | Expression, Expression)` | Логическое | Определяет, существует ли элемент в массиве |
| `length(string | Expression)` | number | Возвращает длину строки или массива. |
| `properties()`| value | Возвращает объект свойств компонента. |

Следующие выражения стиля веб-пакета SDK не поддерживаются в пакет SDK для Android:

- Индекс
- срез

**Примеры**

Доступ к свойствам функции можно получить непосредственно в выражении с помощью `get` выражения. В этом примере используется `zoneColor` значение функции для указания свойства Color пузырькового слоя.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    //Get the zoneColor value.
    bubbleColor(get("zoneColor"))
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    //Get the zoneColor value.
    bubbleColor(get("zoneColor"))
)
```

::: zone-end

Приведенный выше пример будет работать нормально, если все функции точки имеют `zoneColor` свойство. В противном случае цвет скорее всего вернется к «черному». Чтобы изменить цвет резервирования, используйте `switchCase` выражение в сочетании с `has` выражением, чтобы проверить, существует ли свойство. Если свойство не существует, возвращается резервный цвет.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        //Use a conditional case expression.
        switchCase(
            //Check to see if feature has a "zoneColor" 
            has("zoneColor"), 

            //If it does, use it.
            get("zoneColor"), 

            //If it doesn't, default to blue.
            literal("blue")
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        //Use a conditional case expression.
        switchCase(
            //Check to see if feature has a "zoneColor" 
            has("zoneColor"), 

            //If it does, use it.
            get("zoneColor"), 

            //If it doesn't, default to blue.
            literal("blue")
        )
    )
)
```

::: zone-end

Пузырьки и слои символов по умолчанию отображают координаты всех фигур в источнике данных. Это поведение может выделять вершины многоугольника или линии. `filter`Параметр слоя можно использовать для ограничения типа геометрии отображаемых функций с помощью `geometryType` выражения в логическом выражении. В следующем примере уровень пузырьков ограничивается таким образом, чтобы `Point` отображались только функции.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    filter(eq(geometryType(), "Point"))
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    filter(eq(geometryType(), "Point"))
)
```

::: zone-end

В следующем примере допускаются `Point` `MultiPoint` отрисовка и компонентов, и.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    filter(any(eq(geometryType(), "Point"), eq(geometryType(), "MultiPoint")))
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    filter(any(eq(geometryType(), "Point"), eq(geometryType(), "MultiPoint")))
)
```

::: zone-end

Точно так же контур многоугольников будет отображаться в слоях линий. Чтобы отключить это поведение на уровне линии, добавьте фильтр, допускающий только `LineString` функции и `MultiLineString` .  

Ниже приведены некоторые дополнительные примеры использования выражений данных.

```java
//Get item [2] from an array "properties.abcArray[1]" = "c"
at(2, get("abcArray"))

//Get item [0][1] from a 2D array "properties.array2d[0][1]" = "b"
at(1, at(0, get("array2d")))

//Check to see if a value is in an array "properties.abcArray.indexOf('a') !== -1" = true
in("a", get("abcArray"))

//Get the length of an array "properties.abcArray.length" = 3
length(get("abcArray"))

//Get the value of a subproperty "properties._style.fillColor" = "red"
get("fillColor", get("_style"))

//Check that "fillColor" exists as a subproperty of "_style".
has("fillColor", get("_style"))
```

## <a name="math-expressions"></a>Математические выражения

Математические выражения предоставляют математические операторы для выполнения вычислений на основе данных в платформе выражений.

| Выражение | Возвращаемый тип | Описание |
|------------|-------------|-------------|
| `abs(number | Expression)` | number | Вычисляет абсолютное значение заданного числа. |
| `acos(number | Expression)` | number | Вычисляет арккосинус указанного числа. |
| `asin(number | Expression)` | number | Вычисляет арксинус указанного числа. |
| `atan(number | Expression)` | number | Вычисляет арктангенс указанного числа. |
| `ceil(number | Expression)` | number | Округляет число до следующего целого числа. |
| `cos(number | Expression)` | number | Вычисляет COS указанного числа. |
| `division(number, number)` \| `division(Expression, Expression)` | number | Делит первое число на второе число. Выражение эквивалента веб-пакета SDK: `/` |
| `e()` | number | Возвращает математическую константу `e` . |
| `floor(number | Expression)` | number | Округляет число до предыдущего целого числа. |
| `log10(number | Expression)` | number | Вычисляет десятичный логарифм указанного числа. |
| `log2(number | Expression)` | number | Вычисляет логарифм указанного числа с основанием 2. |
| `ln(number | Expression)` | number | Вычисляет натуральный логарифм заданного числа. |
| `ln2()` | number | Возвращает математическую константу `ln(2)` . |
| `max(numbers... | expressions...)` | number | Вычисляет максимальное число в указанном наборе чисел. |
| `min(numbers... | expressions...)` | number | Вычисляет минимальное число в указанном наборе чисел. |
| `mod(number, number)` \| `mod(Expression, Expression)` | number | Вычисляет остаток при делении первого числа на второе число. Выражение эквивалента веб-пакета SDK: `%` |
| `pi()` | number | Возвращает математическую константу `PI` . |
| `pow(number, number)` \| `pow(Expression, Expression)` | number | Вычисляет значение первого значения, возведенного в степень второго числа. |
| `product(numbers... | expressions...)` | number | Умножает указанные числа вместе. Выражение эквивалента веб-пакета SDK: `*` |
| `round(number | Expression)` | number | Округляет число до ближайшего целого. Среднее значение округляется с нуля. Например, `round(-1.5)` принимает значение `-2` . |
| `sin(number | Expression)` | number | Вычисляет синус указанного числа. |
| `sqrt(number | Expression)` | number | Вычисляет квадратный корень заданного числа. |
| `subtract(number | Expression` | number | Вычитает 0 на указанное число. |
| `subtract(number | Expression, number | Expression)` | number | Вычитает первые цифры по второму числу. |
| `sum(numbers... | expressions...)` | number | Вычисляет сумму указанных чисел. |
| `tan(number | Expression)` | number | Вычисляет тангенс указанного числа. |

## <a name="boolean-expressions"></a>Логические выражения

Логические выражения предоставляют набор выражений логических операторов для оценки логических сравнений.

При сравнении значений сравнение строго типизировано. Значения различных типов всегда считаются неравными. Случаи, когда типы во время синтаксического анализа могут различаться, считаются недопустимыми и выдают ошибку синтаксического анализа.

| Выражение | Возвращаемый тип | Описание |
|------------|-------------|-------------|
| `all(Expression...)` | Логическое | Возвращает `true` значение, если все входные данные являются `true` , `false` в противном случае —. |
| `any(Expression...)` | Логическое | Возвращает `true` , если какой либо вход имеет `true` `false` значение, в противном случае —. |
| `eq(Expression compareOne, Expression | boolean | number | string compareTwo)` \| `eq(Expression compareOne, Expression | string compareTwo, Expression collator)` | Логическое | Возвращает значение `true` , если входные значения равны, `false` в противном случае —. Аргументы должны быть либо строками, либо обоими числами. |
| `gt(Expression compareOne, Expression | boolean | number | string compareTwo)` \| `gt(Expression compareOne, Expression | string compareTwo, Expression collator)` | Логическое | Возвращает `true` , если первые входные данные строго больше второго, `false` в противном случае —. Аргументы должны быть либо строками, либо обоими числами. |
| `gte(Expression compareOne, Expression | boolean | number | string compareTwo)` \| `gte(Expression compareOne, Expression | string compareTwo, Expression collator)` | Логическое | Возвращает `true` , если первый вход больше второго или равен ему; `false` в противном случае — значение. Аргументы должны быть либо строками, либо обоими числами. |
| `lt(Expression compareOne, Expression | boolean | number | string compareTwo)` \| `lt(Expression compareOne, Expression | string compareTwo, Expression collator)` | Логическое | Возвращает `true` , если первые входные данные строго меньше второго, `false` в противном случае —. Аргументы должны быть либо строками, либо обоими числами. |
| `lte(Expression compareOne, Expression | boolean | number | string compareTwo)` \| `lte(Expression compareOne, Expression | string compareTwo, Expression collator)` | Логическое | Возвращает `true` , если первый вход меньше второго или равен ему; `false` в противном случае — значение. Аргументы должны быть либо строками, либо обоими числами. |
| `neq(Expression compareOne, Expression | boolean | number | string compareTwo)` \| `neq(Expression compareOne, Expression | string compareTwo, Expression collator)` | Логическое | Возвращает значение `true` , если входные значения не равны; `false` в противном случае —. |
| `not(Expression | boolean)` | Логическое | Логическое отрицание. Возвращает `true` , если входные данные имеют значение `false` , и `false` Если вход имеет значение `true` . |

## <a name="conditional-expressions"></a>Условные выражения

Условные выражения предоставляют логические операции, аналогичные операторам if.

Следующие выражения выполняют условные операции логики над входными данными. Например, `switchCase` выражение предоставляет логику "if/then/else", а `match` выражение как "оператор switch".

### <a name="switch-case-expression"></a>Выражение варианта переключения

`switchCase`Выражение — это тип условного выражения, который предоставляет логику "if/then/else". Этот тип выражения проходит через список логических условий. Он возвращает выходное значение первого логического условия для вычисления значения true.

Следующий псевдокод определяет структуру `switchCase` выражения.

```java
switchCase(
    condition1: boolean expression, 
    output1: value,
    condition2: boolean expression, 
    output2: value,
    ...,
    fallback: value
)
```

**Пример**

В следующем примере выполняется выполнение различных логических условий до тех пор, пока не будет найдено выражение, результатом вычисления которого является `true` , а затем возвращается соответствующее значение. Если результат логического условия не равен `true` , будет возвращено резервное значение.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        switchCase(
            //Check to see if the first boolean expression is true, and if it is, return its assigned result.
            //If it has a zoneColor property, use its value as a color.
            has("zoneColor"), toColor(get("zoneColor")),

            //Check to see if the second boolean expression is true, and if it is, return its assigned result.
            //If it has a temperature property with a value greater than or equal to 100, make it red.
            all(has("temperature"), gte(get("temperature"), 100)), color(Color.RED),
            
            //Specify a default value to return. In this case green.
            color(Color.GREEN)
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        switchCase(
            //Check to see if the first boolean expression is true, and if it is, return its assigned result.
            //If it has a zoneColor property, use its value as a color.
            has("zoneColor"), toColor(get("zoneColor")),

            //Check to see if the second boolean expression is true, and if it is, return its assigned result.
            //If it has a temperature property with a value greater than or equal to 100, make it red.
            all(has("temperature"), gte(get("temperature"), 100)), color(Color.RED),
            
            //Specify a default value to return. In this case green.
            color(Color.GREEN)
        )
    )
)
```

::: zone-end

### <a name="match-expression"></a>Выражение match

`match`Выражение представляет собой тип условного выражения, предоставляющего оператор switch, например логику. Входными данными может быть любое выражение `get( "entityType")` , например, возвращающее строку или число. Каждая из них должна иметь метку, которая является либо одиночным литеральным значением, либо массивом литеральных значений, значения которых должны быть всеми строками или целыми числами. Входные данные соответствуют, если какое бы то ни было значение в массиве. Каждая заметка должна быть уникальной. Если тип входных данных не соответствует типу меток, то результатом будет значение отката по умолчанию.

Следующий псевдокод определяет структуру `match` выражения.

```java
match(Expression input, Expression defaultOutput, Expression.Stop... stops)
```

**Примеры**

В следующем примере рассматривается `entityType` свойство функции Point в пузырьковой слое, где выполняется поиск совпадения. Если найдено совпадение, возвращается указанное значение или возвращается резервное значение.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        match(
            //Get the input value to match.
            get("entityType"),

            //Specify a default value to return if no match is found.
            color(Color.BLACK),

            //List the values to match and the result to return for each match.

            //If value is "restaurant" return "red".
            stop("restaurant", color(Color.RED)),

            //If value is "park" return "green".
            stop("park", color(Color.GREEN))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        match(
            //Get the input value to match.
            get("entityType"),

            //Specify a default value to return if no match is found.
            color(Color.BLACK),

            //List the values to match and the result to return for each match.

            //If value is "restaurant" return "red".
            stop("restaurant", color(Color.RED)),

            //If value is "park" return "green".
            stop("park", color(Color.GREEN))
        )
    )
)
```

::: zone-end

В следующем примере массив используется для перечисления набора меток, которые должны возвращать одно и то же значение. Этот подход гораздо более эффективен, чем вывод каждой метки по отдельности. В этом случае, если `entityType` свойство имеет значение "Ресторан" или "grocery_store", будет возвращен красный цвет.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        match(
            //Get the input value to match.
            get("entityType"),

            //Specify a default value to return if no match is found.
            color(Color.BLACK),

            //List the values to match and the result to return for each match.

            //If value is "restaurant" or "grocery_store" return "red".
            stop(Arrays.asList("restaurant", "grocery_store"), color(Color.RED)),

            //If value is "park" return "green".
            stop("park", color(Color.GREEN))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        match(
            //Get the input value to match.
            get("entityType"),

            //Specify a default value to return if no match is found.
            color(Color.BLACK),

            //List the values to match and the result to return for each match.

            //If value is "restaurant" or "grocery_store" return "red".
            stop(arrayOf("restaurant", "grocery_store"), color(Color.RED)),

            //If value is "park" return "green".
            stop("park", color(Color.GREEN))
        )
    )
)
```

::: zone-end

### <a name="coalesce-expression"></a>Выражение объединения

`coalesce`Выражение проходит через набор выражений до тех пор, пока не будет получено первое значение, отличное от NULL, и не вернет это значение.

Следующий псевдокод определяет структуру `coalesce` выражения.

```java
coalesce(Expression... input)
```

**Пример**

В следующем примере выражение используется `coalesce` для задания `textField` параметра уровня символа. Если `title` свойство отсутствует в компоненте или имеет значение `null` , выражение попытается найти `subTitle` свойство, если оно отсутствует или `null` , после этого будет возвращаться к пустой строке.

::: zone pivot="programming-language-java-android"

```java
SymbolLayer layer = new SymbolLayer(source,
    textField(
        coalesce(
            //Try getting the title property.
            get("title"),

            //If there is no title, try getting the subTitle. 
            get("subTitle"),

            //Default to an empty string.
            literal("")
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = SymbolLayer(source,
    textField(
        coalesce(
            //Try getting the title property.
            get("title"),

            //If there is no title, try getting the subTitle. 
            get("subTitle"),

            //Default to an empty string.
            literal("")
        )
    )
)
```

::: zone-end

## <a name="type-expressions"></a>Выражения типа

Выражения типа предоставляют средства для тестирования и преобразования различных типов данных, таких как строки, числа и логические значения.

| Выражение | Возвращаемый тип | Описание |
|------------|-------------|-------------|
| `array(Expression)` | Object [] | Утверждает, что входные данные являются массивом. |
| `bool(Expression)` | Логическое | Утверждает, что входное значение является логическим. |
| `collator(boolean caseSensitive, boolean diacriticSensitive)` \| `collator(boolean caseSensitive, boolean diacriticSensitive, java.util.Locale locale)` \| `collator(Expression caseSensitive, Expression diacriticSensitive)` \| `collator(Expression caseSensitive, Expression diacriticSensitive, Expression locale)` | collator | Возвращает средство сортировки для использования в операциях сравнения, зависящих от языкового стандарта. Параметры с учетом регистра и диакритические знаки по умолчанию имеют значение false. Аргумент locale указывает тег языка IETF для используемого языкового стандарта. Если значение не указано, используется языковой стандарт по умолчанию. Если запрошенный языковой стандарт недоступен, то в параметрах сортировки будет использоваться определенный системой языковой стандарт. Используйте параметр resolved-locale для проверки результатов работы отката языкового стандарта.  |
| `literal(boolean \| number \| string \| Object \| Object[])` | Boolean \| Number, объект \| \| объекта \| [] | Возвращает литеральный массив или значение объекта. Используйте это выражение, чтобы предотвратить вычисление массива или объекта в качестве выражения. Это необходимо, если массив или объект должен возвращаться выражением. |
| `number(Expression)` | number | Утверждает, что входное значение является числом. |
| `object(Expression)` | Объект | Утверждает, что входное значение является объектом. |
| `string(Expression)` | строка | Утверждает, что входное значение является строкой. |
| `toArray(Expression)` | Object [] | Преобразует выражение в массив объектов JSON. |
| `toBool(Expression)` | Логическое | Преобразует входное значение в логическое. |
| `toNumber(Expression)` | number | По возможности преобразует входное значение в число. |
| `toString(Expression)` | строка | Преобразует входное значение в строку. |
| `typeoOf(Expression)` | строка | Возвращает строку, описывающую тип данного значения. |

## <a name="color-expressions"></a>Выражения цвета

Выражения цвета упрощают создание цветовых значений и манипуляций с ними.

| Выражение | Возвращаемый тип | Описание |
|------------|-------------|-------------|
| `color(int)` | color | Преобразует целочисленное значение цвета в выражение цвета. |
| `rgb(Expression red, Expression green, Expression blue)` \| `rgb(number red, number green, number blue)` | color | Создает значение цвета из *Красного*, *зеленого* и *синего* компонентов, которое должно находиться в диапазоне от `0` до и `255` альфа-компонент `1` . Если какой-либо компонент выходит за пределы диапазона, выражение является ошибкой. |
| `rgba(Expression red, Expression green, Expression blue, Expression alpha)` \| `rgba(number red, number green, number blue, number alpha)` | color | Создает значение цвета из *Красного*, *зеленого*, *синего* компонентов, которое должно находиться между `0` и `255` , и альфа-компонентом в диапазоне `0` и `1` . Если какой-либо компонент выходит за пределы диапазона, выражение является ошибкой. |
| `toColor(Expression)` | color  | Преобразует входное значение в цвет. |
| `toRgba(Expression)` | color | Возвращает массив из четырех элементов, содержащий красный, зеленый, синий и альфа-компоненты цвета ввода в указанном порядке. |

**Пример**

В следующем примере создается значение цвета RGB с *красным* значением `255` , а также *зеленым* и *синим* значениями, которые рассчитываются путем умножения на `2.5` значение `temperature` Свойства. При изменении температуры цвет изменится на другие оттенки *Красного* цвета.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        //Create a RGB color value.
        rgb(
            //Set red value to 255. Wrap with literal expression since using expressions for other values.
            literal(255f),    

            //Multiple the temperature by 2.5 and set the green value.
            product(literal(2.5f), get("temperature")), 

            //Multiple the temperature by 2.5 and set the blue value.
            product(literal(2.5f), get("temperature")) 
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        //Create a RGB color value.
        rgb(
            //Set red value to 255. Wrap with literal expression since using expressions for other values.
            literal(255f),    

            //Multiple the temperature by 2.5 and set the green value.
            product(literal(2.5f), get("temperature")), 

            //Multiple the temperature by 2.5 and set the blue value.
            product(literal(2.5f), get("temperature")) 
        )
    )
)
```

::: zone-end

Если все параметры цвета являются числами, нет необходимости заносить их в `literal` выражение. Пример:

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        //Create a RGB color value.
        rgb(
            255f,  //Set red value to 255.

            150f,  //Set green value to 150.

            0f     //Set blue value to 0.
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        //Create a RGB color value.
        rgb(
            255f,  //Set red value to 255.

            150f,  //Set green value to 150.

            0f     //Set blue value to 0.
        )
    )
)
```

::: zone-end

> [!TIP]
> Значения цвета строк можно преобразовать в цвет с помощью `android.graphics.Color.parseColor` метода. Следующий пример преобразует шестнадцатеричную цветовую строку в выражение цвета, которое можно использовать с слоем.
>
> ```java
> color(parseColor("#ff00ff"))
> ```

## <a name="string-operator-expressions"></a>Выражения строкового оператора

Выражения строкового оператора выполняют операции преобразования строк, таких как объединение и преобразование варианта.

| Выражение | Возвращаемый тип | Описание |
|------------|-------------|-------------|
| `concat(string...)` \| `concat(Expression...)` | строка | Объединяет несколько строк вместе. Каждое значение должно быть строкой. Используйте `toString` выражение типа для преобразования других типов значений в строку, если это необходимо. |
| `downcase(string)` \| `downcase(Expression)` | строка | Преобразует заданную строку в нижний регистр. |
| `isSupportedScript(string)` \| `isSupportedScript(Expression)`| Логическое | Определяет, использует ли входная строка кодировку, поддерживаемую текущим стеком шрифтов. Например: `isSupportedScript("ಗೌರವಾರ್ಥವಾಗಿ")` |
| `resolvedLocale(Expression collator)` | строка | Возвращает тег языка IETF для языкового стандарта, используемого указанными параметрами сортировки. Это можно использовать для определения национальной настройки системы по умолчанию или для определения успешной загрузки запрошенного языкового стандарта. |
| `upcase(string)` \| `upcase(Expression)` | строка | Преобразует заданную строку в верхний регистр. |

**Пример**

В следующем примере `temperature` свойство функции Point преобразуется в строку, а затем в конец строки объединяется "° f".

::: zone pivot="programming-language-java-android"

```java
SymbolLayer layer = new SymbolLayer(source,
    textField(
        concat(Expression.toString(get("temperature")), literal("°F"))
    ),

    //Some additional style options.
    textOffset(new Float[] { 0f, -1.5f }),
    textSize(12f),
    textColor("white")
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = SymbolLayer(source,
    textField(
        concat(Expression.toString(get("temperature")), literal("°F"))
    ),

    //Some additional style options.
    textOffset(new Float[] { 0f, -1.5f }),
    textSize(12f),
    textColor("white")
)
```

::: zone-end

Приведенное выше выражение выводит на карту закрепление с текстом "64 ° F", который накладывается поверх него, как показано на рисунке ниже.

![Пример выражения строкового оператора](media/how-to-expressions/string-operator-expression.png)

## <a name="interpolate-and-step-expressions"></a>Выражения интерполяции и шага

Выражения интерполяции и шага можно использовать для вычисления значений по кривой с интерполяцией или с пошаговыми функциями. Эти выражения принимают выражение, возвращающее числовое значение в качестве входных данных, например `get("temperature")` . Входное значение вычисляется на основе пар входных и выходных значений, чтобы определить значение, которое лучше подходит для интерполяции кривой или функции Step. Выходные значения называются «остановками». Входные значения для каждой из них должны быть числами и находиться в возрастающем порядке. Выходные значения должны быть числом, массивом чисел или цветом.

### <a name="interpolate-expression"></a>Выражение интерполяции

`interpolate`Выражение можно использовать для вычисления непрерывного гладкого набора значений путем интерполяции между значениями типа «конец». `interpolate`Выражение, возвращающее цветовые значения, создает цветовой градиент, в котором значения результатов выбираются из. `interpolate`Выражение имеет следующие форматы:

```java
//Stops consist of two expressions.
interpolate(Expression.Interpolator interpolation, Expression number, Expression... stops)

//Stop expression wraps two values.
interpolate(Expression.Interpolator interpolation, Expression number, Expression.Stop... stops)
```

Существует три типа методов интерполяции, которые можно использовать в `interpolate` выражении:

| Имя | Описание |
|------|-------------|
| `linear()` | Линейная интерполяция между парой остановок.  |
| `exponential(number)` \| `exponential(Expression)` | Выполняет экспоненциальную интерполяцию между остановками. Указывается "Base" и управляет скоростью, с которой увеличивается объем выходных данных. Более высокие значения приводят к увеличению объема выходных данных в большую сторону диапазона. Значение "Base", близкое к 1, создает выходные данные, которые более линейно увеличиваются.|
| `cubicBezier(number x1, number y1, number x2, number y2)` \| `cubicBezier(Expression x1, Expression y1, Expression x2, Expression y2)` | Выполняет интерполяцию с использованием [кривой Безье третьего порядка](https://developer.mozilla.org/docs/Web/CSS/timing-function) , определяемой заданными контрольными точками. |

`stop`Выражение имеет формат `stop(stop, value)` .

Ниже приведен пример того, как выглядят эти различные типы интерполяций.

| Линейная  | Экспоненциально | Кривая Безье третьего порядка |
|---------|-------------|--------------|
| ![Диаграмма линейной интерполяции](media/how-to-expressions/linear-interpolation.png) | ![Экспоненциальный граф интерполяции](media/how-to-expressions/exponential-interpolation.png) | ![Диаграмма интерполяции Безье третьего порядка](media/how-to-expressions/bezier-curve-interpolation.png) |

**Пример**

В следующем примере выражение используется `linear interpolate` для задания `bubbleColor` Свойства пузырькового слоя на основе `temperature` Свойства функции Point. Если `temperature` значение меньше 60, будет возвращен "Blue". Если это значение находится в диапазоне от 60 до 70, будет возвращен желтый цвет. Если он находится в диапазоне от 70 до 80, будет возвращен оранжевый ( `#FFA500` ). Если это 80 или более, будет возвращен красный символ.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        interpolate(
            linear(),
            get("temperature"),
            stop(50, color(Color.BLUE)),
            stop(60, color(Color.YELLOW)),
            stop(70, color(parseColor("#FFA500"))),
            stop(80, color(Color.RED))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        interpolate(
            linear(),
            get("temperature"),
            stop(50, color(Color.BLUE)),
            stop(60, color(Color.YELLOW)),
            stop(70, color(parseColor("#FFA500"))),
            stop(80, color(Color.RED))
        )
    )
)
```

::: zone-end

На следующем рисунке показано, как выбираются цвета для приведенного выше выражения.

![Пример выражения интерполяции](media/how-to-expressions/interpolate-expression-example.png)

### <a name="step-expression"></a>Выражение шага

`step`Выражение можно использовать для вычисления дискретных пошаговых значений результатов путем вычисления [функции кусочно--Constant](http://mathworld.wolfram.com/PiecewiseConstantFunction.html) , определенной в параметре Stop.

`interpolate`Выражение имеет следующие форматы:

```java
step(Expression input, Expression defaultOutput, Expression... stops)

step(Expression input, Expression defaultOutput, Expression.Stop... stops)

step(Expression input, number defaultOutput, Expression... stops)

step(Expression input, number defaultOutput, Expression.Stop... stops)

step(number input, Expression defaultOutput, Expression... stops)

step(number input, Expression defaultOutput, Expression.Stop... stops)

step(number input, number defaultOutput, Expression... stops)

step(number input, number defaultOutput, Expression.Stop... stops)
```

Выражения шага возвращают выходное значение завершения, непосредственно предшествующее входному значению, или первое входное значение, если входные данные меньше первой позиции.

**Пример**

В следующем примере выражение используется `step` для задания `bubbleColor` Свойства пузырькового слоя на основе `temperature` Свойства функции Point. Если `temperature` значение меньше 60, будет возвращен "Blue". Если он находится в диапазоне от 60 до 70, будет возвращен желтый. Если он находится в диапазоне от 70 до 80, будет возвращен оранжевый. Если это 80 или более, будет возвращен красный символ.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(
        step(
            get("temperature"),
            color(Color.BLUE),
            stop(60, color(Color.YELLOW)),
            stop(70, color(parseColor("#FFA500"))),
            stop(80, color(Color.RED))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(
        step(
            get("temperature"),
            color(Color.BLUE),
            stop(60, color(Color.YELLOW)),
            stop(70, color(parseColor("#FFA500"))),
            stop(80, color(Color.RED))
        )
    )
)
```

::: zone-end

На следующем рисунке показано, как выбираются цвета для приведенного выше выражения.

![Пример выражения шага](media/how-to-expressions/step-expression-example.png)

## <a name="layer-specific-expressions"></a>Выражения, относящиеся к слою

Специальные выражения, которые применяются только к определенным слоям.

### <a name="heat-map-density-expression"></a>Выражение плотности тепловой карт

Выражение плотности тепловой карт извлекает значение плотности тепловой карт для каждого пикселя на уровне тепловой карт и определяется как `heatmapDensity` . Это значение является числом между `0` и `1` . Он используется в сочетании с `interpolation` `step` выражением или для определения цветового градиента, используемого для раскрашивания тепловой карт. Это выражение можно использовать только в `heatmapColor` параметре слоя тепловой карт.

> [!TIP]
> Цвет с индексом 0, в выражении интерполяции или цвет по умолчанию для цвета шага определяет цвет области, в которой нет данных. Цвет с индексом 0 можно использовать для определения цвета фона. Многие пользователи предпочитают использовать для этого прозрачный или полупрозрачный черный цвет.

**Пример**

В этом примере используется выражение интерполяции для создания гладкой цветового градиента для визуализации тепловой карт.

::: zone pivot="programming-language-java-android"

```java
HeatMapLayer layer = new HeatMapLayer(source,
    heatmapColor(
        interpolate(
            linear(),
            heatmapDensity(),
            stop(0, color(Color.TRANSPARENT)),
            stop(0.01, color(Color.MAGENTA)),
            stop(0.5, color(parseColor("#fb00fb"))),
            stop(1, color(parseColor("#00c3ff")))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = HeatMapLayer(source,
    heatmapColor(
        interpolate(
            linear(),
            heatmapDensity(),
            stop(0, color(Color.TRANSPARENT)),
            stop(0.01, color(Color.MAGENTA)),
            stop(0.5, color(parseColor("#fb00fb"))),
            stop(1, color(parseColor("#00c3ff")))
        )
    )
)
```

::: zone-end

Помимо использования плавного градиента для выделения тепловой карте, цвета можно указать в наборе диапазонов с помощью `step` выражения. Использование `step` выражения для выделения цветом тепловой карт визуально разбивает плотность на диапазоны, похожие на карту расконтурного или лепесткового стиля.

::: zone pivot="programming-language-java-android"

```java
HeatMapLayer layer = new HeatMapLayer(source,
    heatmapColor(
        step(
            heatmapDensity(),
            color(Color.TRANSPARENT),
            stop(0.01, color(parseColor("#000080"))),
            stop(0.25, color(parseColor("#000080"))),
            stop(0.5, color(Color.GREEN)),
            stop(0.5, color(Color.YELLOW)),
            stop(1, color(Color.RED))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = HeatMapLayer(source,
    heatmapColor(
        step(
            heatmapDensity(),
            color(Color.TRANSPARENT),
            stop(0.01, color(parseColor("#000080"))),
            stop(0.25, color(parseColor("#000080"))),
            stop(0.5, color(Color.GREEN)),
            stop(0.5, color(Color.YELLOW)),
            stop(1, color(Color.RED))
        )
    )
)
```

::: zone-end

Дополнительные сведения см. в документации по [добавлению слоя тепловой карт](map-add-heat-map-layer-android.md) .

### <a name="line-progress-expression"></a>Выражение хода выполнения строки

Выражение хода выполнения строки получает ход выполнения вдоль линии градиента в слое линий и определяется как `lineProgress()` . Это значение является числом от 0 до 1. Он используется в сочетании с `interpolation` `step` выражением или. Это выражение можно использовать только с `strokeGradient` параметром линейного слоя.

> [!NOTE]
> Для `strokeGradient` параметра уровня линии необходимо, чтобы `lineMetrics` параметр источника данных был установлен в значение `true` .

**Пример**

В этом примере `lineProgress()` выражение используется для применения градиента цвета к штриху линии.

::: zone pivot="programming-language-java-android"

```java
LineLayer layer = new LineLayer(source,
    strokeGradient(
        interpolate(
            linear(),
            lineProgress(),
            stop(0, color(Color.BLUE)),
            stop(0.1, color(Color.argb(255, 65, 105, 225))), //Royal Blue
            stop(0.3, color(Color.CYAN)),
            stop(0.5, color(Color.argb(255,0, 255, 0))), //Lime
            stop(0.7, color(Color.YELLOW)),
            stop(1, color(Color.RED))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = LineLayer(source,
    strokeGradient(
        interpolate(
            linear(),
            lineProgress(),
            stop(0, color(Color.BLUE)),
            stop(0.1, color(Color.argb(255, 65, 105, 225))), //Royal Blue
            stop(0.3, color(Color.CYAN)),
            stop(0.5, color(Color.argb(255,0, 255, 0))), //Lime
            stop(0.7, color(Color.YELLOW)),
            stop(1, color(Color.RED))
        )
    )
)
```

::: zone-end

[См. пример в реальном времени](map-add-line-layer.md#line-stroke-gradient)

### <a name="text-field-format-expression"></a>Выражение формата текстового поля

`format`Выражение можно использовать с `textField` параметром слоя символов для обеспечения форматирования смешанного текста. Это выражение принимает одно или несколько `formatEntry` выражений, задающих строку и набор `formatOptions` для добавления к текстовому полю.

| Выражение | Описание |
|------------|-------------|
| `format(Expression...)` | Возвращает форматированный текст, содержащий заметки для использования в записях текстового поля в смешанном формате. |
| `formatEntry(Expression text)` \| `formatEntry(Expression text, Expression.FormatOption... formatOptions)` \| `formatEntry(String text)` \| `formatEntry(String text, Expression.FormatOption... formatOptions)` | Возвращает форматированную строку строки для использования в `format` выражении. |

Доступны следующие параметры формата:

| Выражение | Описание |
|------------|-------------|
| `formatFontScale(number)` \| `formatFontScale(Expression)` | Задает коэффициент масштабирования для размера шрифта. Если указано, это значение переопределит `textSize` свойство для отдельной строки. |
| `formatTextFont(string[])` \| `formatTextFont(Expression)` | Задает цвет, применяемый к тексту при подготовке к просмотру. |

**Пример**

В следующем примере текстовое поле форматируется путем добавления полужирного шрифта и масштабирования размера шрифта `title` Свойства функции. В этом примере также добавляется `subTitle` свойство функции на новую строку с масштабируемым размером шрифта.

::: zone pivot="programming-language-java-android"

```java
SymbolLayer layer = new SymbolLayer(source,
    textField(
        format(
            //Bold the title property and scale its font size up.
            formatEntry(
                get("title"),
                formatTextFont(new String[] { "StandardFont-Bold" }),
                formatFontScale(1.25)),

            //Add a new line without any formatting.
            formatEntry("\n"),

            //Scale the font size down of the subTitle property.
            formatEntry(
                get("subTitle"),
                formatFontScale(0.75))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = SymbolLayer(source,
    textField(
        format(
            //Bold the title property and scale its font size up.
            formatEntry(
                get("title"),
                formatTextFont(arrayOf("StandardFont-Bold")),
                formatFontScale(1.25)),

            //Add a new line without any formatting.
            formatEntry("\n"),

            //Scale the font size down of the subTitle property.
            formatEntry(
                get("subTitle"),
                formatFontScale(0.75))
        )
    )
)
```

::: zone-end

Этот слой будет отображать функцию Point, как показано на рисунке ниже:

![Изображение компонента Point с форматированным текстовым полем](media/how-to-expressions/text-field-format-expression.png)

## <a name="zoom-expression"></a>Выражение масштаба

`zoom`Выражение используется для получения текущего уровня масштаба отображения во время визуализации и определяется как `zoom()` . Это выражение возвращает число между минимальным и максимальным диапазонами масштаба для сопоставления. Azure Maps Интерактивные элементы управления картой для веб-и Android поддерживают 25 уровней масштабирования с номерами от 0 до 24. Использование `zoom` выражения позволяет динамически изменять стили при изменении масштаба на карте. `zoom`Выражение может использоваться только с `interpolate` `step` выражениями и.

**Пример**

По умолчанию радиусы точек данных, отображаемых на уровне тепловой карт, имеют фиксированный пиксельный радиус для всех уровней масштабирования. По мере масштабирования карт данные объединяются, и слой тепловой карт выглядит иначе. `zoom`Выражение можно использовать для масштабирования радиуса каждого уровня масштаба таким, чтобы каждая точка данных охватывает одну и ту же физическую область на карте. Благодаря этому слой тепловой карт будет выглядеть более статическим и последовательным. Каждый уровень масштаба на карте имеет вдвое больше пикселей по вертикали и горизонтали, чем предыдущий уровень масштаба. При масштабировании радиуса с каждым уровнем масштабирования создается тепловая схема, которая выглядит единообразно на всех уровнях масштабирования. Это можно сделать с помощью `zoom` выражения с `base 2 exponential interpolation` выражением с набором радиуса пикселов для минимального уровня масштаба и масштабируемым радиусом для максимального уровня масштаба, вычисленным как `2 * Math.pow(2, minZoom - maxZoom)` показано ниже.

::: zone pivot="programming-language-java-android"

```java
HeatMapLayer layer = new HeatMapLayer(source,
    heatmapRadius(
        interpolate(
            exponential(2),
            zoom(),

            //For zoom level 1 set the radius to 2 pixels.
            stop(1, 2),

            //Between zoom level 1 and 19, exponentially scale the radius from 2 pixels to 2 * (maxZoom - minZoom)^2 pixels.
            stop(19, 2 * Math.pow(2, 19 - 1))
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = HeatMapLayer(source,
    heatmapRadius(
        interpolate(
            exponential(2),
            zoom(),

            //For zoom level 1 set the radius to 2 pixels.
            stop(1, 2),

            //Between zoom level 1 and 19, exponentially scale the radius from 2 pixels to 2 * (maxZoom - minZoom)^2 pixels.
            stop(19, 2 * Math.pow(2, 19 - 1))
        )
    )
)
```

::: zone-end

## <a name="variable-binding-expressions"></a>Выражения привязки переменных

Выражения привязки переменных хранят результаты вычисления в переменной. Таким образом, можно несколько раз ссылаться на результаты вычисления в других местах выражения. Это полезная оптимизация для выражений, в которых участвует много вычислений.

| Выражение | Возвращаемый тип | Описание |
|--------------|---------------|--------------|
| `let(Expression... input)` | | Хранит одно или несколько значений как переменные для использования `var` выражением в дочернем выражении, возвращающем результат. |
| `var(Expression expression)` \| `var(string variableName)` | Объект | Ссылается на переменную, созданную с помощью `let` выражения. |

**Пример**

В этом примере используется выражение, которое вычисляет доход относительно коэффициента температуры, а затем использует `case` выражение для вычисления различных логических операций с этим значением. `let`Выражение используется для хранения дохода относительно коэффициента температуры, поэтому его необходимо вычислить только один раз. `var`Выражение ссылается на эту переменную так часто, как требуется, без необходимости повторного вычисления.

::: zone pivot="programming-language-java-android"

```java
BubbleLayer layer = new BubbleLayer(source,
    bubbleColor(           
        let(
            //Divide the point features `revenue` property by the `temperature` property and store it in a variable called `ratio`.
            literal("ratio"), division(get("revenue"), get("temperature")),

            //Evaluate the child expression in which the stored variable will be used.
            switchCase(
                //Check to see if the ratio is less than 100, return 'red'.
                lt(var("ratio"), 100), color(Color.RED),

                //Check to see if the ratio is less than 200, return 'green'.
                lt(var("ratio"), 200), color(Color.GREEN),

                //Return `blue` for values greater or equal to 200.
                color(Color.BLUE)
            )
        )
    )
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
val layer = BubbleLayer(source,
    bubbleColor(           
        let(
            //Divide the point features `revenue` property by the `temperature` property and store it in a variable called `ratio`.
            literal("ratio"), division(get("revenue"), get("temperature")),

            //Evaluate the child expression in which the stored variable will be used.
            switchCase(
                //Check to see if the ratio is less than 100, return 'red'.
                lt(var("ratio"), 100), color(Color.RED),

                //Check to see if the ratio is less than 200, return 'green'.
                lt(var("ratio"), 200), color(Color.GREEN),

                //Return `blue` for values greater or equal to 200.
                color(Color.BLUE)
            )
        )
    )
)
```

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о слоях, поддерживающих выражения:

> [!div class="nextstepaction"]
> [Добавление слоя символов](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление слоя пузырьков](map-add-bubble-layer-android.md)

> [!div class="nextstepaction"]
> [Добавление слоя линий](android-map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Добавление слоя многоугольников](how-to-add-shapes-to-android-map.md)

> [!div class="nextstepaction"]
> [Добавление тепловой карты](map-add-heat-map-layer-android.md)

---
title: Функции выражений в потоке данных для сопоставления
description: Сведения о функциях выражений в потоке данных для сопоставления.
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/10/2021
ms.openlocfilehash: 78a17ad6599e1ac4f455f2806a5f944675198b4b
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102631495"
---
# <a name="data-transformation-expressions-in-mapping-data-flow"></a>Выражения преобразования данных в потоке данных для сопоставления

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="expression-functions"></a>Функции выражений

В Фабрике данных для настройки преобразований данных используйте язык выражений функции потока данных для сопоставления.
___
### <code>abs</code>
<code><b>abs(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Абсолютное значение числа.  
* ``abs(-20) -> 20``  
* ``abs(10) -> 10``  
___   
### <code>acos</code>
<code><b>acos(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет арккосинус обратное значение.  
* ``acos(1) -> 0.0``  
___
### <code>add</code>
<code><b>add(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
Добавляет пару строк или чисел. Добавляет дату к числу дней. Добавляет длительность в метку времени. Добавляет один массив аналогичного типа к другому. То же, что и оператор +.  
* ``add(10, 20) -> 30``  
* ``10 + 20 -> 30``  
* ``add('ice', 'cream') -> 'icecream'``  
* ``'ice' + 'cream' + ' cone' -> 'icecream cone'``  
* ``add(toDate('2012-12-12'), 3) -> toDate('2012-12-15')``  
* ``toDate('2012-12-12') + 3 -> toDate('2012-12-15')``  
* ``[10, 20] + [30, 40] -> [10, 20, 30, 40]``  
* ``toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS') + (days(1) + hours(2) - seconds(10)) -> toTimestamp('2019-02-04 07:19:18.871', 'yyyy-MM-dd HH:mm:ss.SSS')``  
___
### <code>addDays</code>
<code><b>addDays(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;days to add&gt;</i> : integral) => datetime</b></code><br/><br/>
Добавляет дни к дате или метке времени. То же, что и оператор + для Date.  
* ``addDays(toDate('2016-08-08'), 1) -> toDate('2016-08-09')``  
___
### <code>addMonths</code>
<code><b>addMonths(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;months to add&gt;</i> : integral, [<i>&lt;value3&gt;</i> : string]) => datetime</b></code><br/><br/>
Добавляет месяцы к дате или метке времени. При необходимости можно передать часовой пояс.  
* ``addMonths(toDate('2016-08-31'), 1) -> toDate('2016-09-30')``  
* ``addMonths(toTimestamp('2016-09-30 10:10:10'), -1) -> toTimestamp('2016-08-31 10:10:10')``  
___
### <code>and</code>
<code><b>and(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
Логический оператор И. То же, что и &&.  
* ``and(true, false) -> false``  
* ``true && false -> false``  
___
### <code>asin</code>
<code><b>asin(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет обратное значение синуса.  
* ``asin(0) -> 0.0``  
___
### <code>atan</code>
<code><b>atan(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет обратное значение тангенса.  
* ``atan(0) -> 0.0``  
___
### <code>atan2</code>
<code><b>atan2(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
Возвращает угол в радианах между положительной осью x плоскости и точкой, заданной координатами.  
* ``atan2(0, 0) -> 0.0``  
___
### <code>between</code>
<code><b>between(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any, <i>&lt;value3&gt;</i> : any) => boolean</b></code><br/><br/>
Проверяет, находится ли первое значение между двумя другими значениями включительно. Можно сравнивать числовые, строковые и значения DateTime * ``between(10, 5, 24)``
* ``true``
* ``between(currentDate(), currentDate() + 10, currentDate() + 20)``
* ``false``
___
### <code>bitwiseAnd</code>
<code><b>bitwiseAnd(<i>&lt;value1&gt;</i> : integral, <i>&lt;value2&gt;</i> : integral) => integral</b></code><br/><br/>
Побитовый оператор и для целочисленных типов. Аналогично оператору & * ``bitwiseAnd(0xf4, 0xef)``
* ``0xe4``
* ``(0xf4 & 0xef)``
* ``0xe4``
___
### <code>bitwiseOr</code>
<code><b>bitwiseOr(<i>&lt;value1&gt;</i> : integral, <i>&lt;value2&gt;</i> : integral) => integral</b></code><br/><br/>
Побитовый оператор OR для целочисленных типов. То же, что и | станции * ``bitwiseOr(0xf4, 0xef)``
* ``0xff``
* ``(0xf4 | 0xef)``
* ``0xff``
___
### <code>bitwiseXor</code>
<code><b>bitwiseXor(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
Побитовый оператор OR для целочисленных типов. То же, что и | станции * ``bitwiseXor(0xf4, 0xef)``
* ``0x1b``
* ``(0xf4 ^ 0xef)``
* ``0x1b``
* ``(true ^ false)``
* ``true``
* ``(true ^ true)``
* ``false``
___
### <code>blake2b</code>
<code><b>blake2b(<i>&lt;value1&gt;</i> : integer, <i>&lt;value2&gt;</i> : any, ...) => string</b></code><br/><br/>
Вычисляет дайджест Blake2 набора столбцов различных примитивных типов данных с битовой длиной, которая может быть кратной 8 & 512. Его можно использовать для вычисления отпечатка для строки * ``blake2b(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4'))``
* ``'c9521a5080d8da30dffb430c50ce253c345cc4c4effc315dab2162dac974711d'``
___
### <code>blake2bBinary</code>
<code><b>blake2bBinary(<i>&lt;value1&gt;</i> : integer, <i>&lt;value2&gt;</i> : any, ...) => binary</b></code><br/><br/>
Вычисляет дайджест Blake2 набора столбцов различных примитивных типов данных с битовой длиной, которая может быть кратной 8 & 512. Его можно использовать для вычисления отпечатка для строки * ``blake2bBinary(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4'))``
* ``unHex('c9521a5080d8da30dffb430c50ce253c345cc4c4effc315dab2162dac974711d')``
___
### <code>case</code>
<code><b>case(<i>&lt;condition&gt;</i> : boolean, <i>&lt;true_expression&gt;</i> : any, <i>&lt;false_expression&gt;</i> : any, ...) => any</b></code><br/><br/>
На основе чередующихся условий применяется одно или другое значение. Если число входов четное, другое значение по умолчанию равно NULL для последнего условия.  
* ``case(10 + 20 == 30, 'dumbo', 'gumbo') -> 'dumbo'``  
* ``case(10 + 20 == 25, 'bojjus', 'do' < 'go', 'gunchus') -> 'gunchus'``  
* ``isNull(case(10 + 20 == 25, 'bojjus', 'do' > 'go', 'gunchus')) -> true``  
* ``case(10 + 20 == 25, 'bojjus', 'do' > 'go', 'gunchus', 'dumbo') -> 'dumbo'``  
___
### <code>cbrt</code>
<code><b>cbrt(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет корень Куба числа.  
* ``cbrt(8) -> 2.0``  
___
### <code>ceil</code>
<code><b>ceil(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Возвращает наименьшее целое число, не меньшее числа.  
* ``ceil(-0.1) -> 0``  
___
### <code>coalesce</code>
<code><b>coalesce(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
Возвращает из набора входных данных первое значение, не равное NULL. Все входные данные должны иметь один и тот же тип.  
* ``coalesce(10, 20) -> 10``  
* ``coalesce(toString(null), toString(null), 'dumbo', 'bo', 'go') -> 'dumbo'``  
___
### <code>collect</code>
<code><b>collect(<i>&lt;value1&gt;</i> : any) => array</b></code><br/><br/>
Собирает все значения выражения в агрегированной группе в массив. Во время этого процесса структуры можно собирать и преобразовывать в альтернативные структуры. Количество элементов будет равно количеству строк в этой группе и может содержать значения NULL. Число собранных элементов должно быть небольшим.  
* ``collect(salesPerson)``
* ``collect(firstName + lastName))``
* ``collect(@(name = salesPerson, sales = salesAmount) )``
___
### <code>columnNames</code>
<code><b>columnNames(<i>&lt;value1&gt;</i> : string) => array</b></code><br/><br/>
Получает все выходные столбцы для потока. В качестве второго аргумента вы можете передать необязательное имя потока.  
* ``columnNames()``
* ``columnNames('DeriveStream')``

___
### <code>columns</code>
<code><b>columns([<i>&lt;stream name&gt;</i> : string]) => any</b></code><br/><br/>
Получает все выходные столбцы для потока. В качестве второго аргумента вы можете передать необязательное имя потока.   
* ``columns()``
* ``columns('DeriveStream')``
___
### <code>compare</code>
<code><b>compare(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => integer</b></code><br/><br/>
Сравнивает два значения одного типа. Возвращает отрицательное целое число, если значение1 < value2, 0, если значение1 = = value2, положительное значение, если значение1 > value2.  
* ``(compare(12, 24) < 1) -> true``  
* ``(compare('dumbo', 'dum') > 0) -> true``  
___
### <code>concat</code>
<code><b>concat(<i>&lt;this&gt;</i> : string, <i>&lt;that&gt;</i> : string, ...) => string</b></code><br/><br/>
Объединяет переменное количество строк. То же, что и оператор + со строками.  
* ``concat('dataflow', 'is', 'awesome') -> 'dataflowisawesome'``  
* ``'dataflow' + 'is' + 'awesome' -> 'dataflowisawesome'``  
* ``isNull('sql' + null) -> true``  
___
### <code>concatWS</code>
<code><b>concatWS(<i>&lt;separator&gt;</i> : string, <i>&lt;this&gt;</i> : string, <i>&lt;that&gt;</i> : string, ...) => string</b></code><br/><br/>
Объединяет переменное количество строк с использованием разделителя. Первый параметр — это разделитель.  
* ``concatWS(' ', 'dataflow', 'is', 'awesome') -> 'dataflow is awesome'``  
* ``isNull(concatWS(null, 'dataflow', 'is', 'awesome')) -> true``  
* ``concatWS(' is ', 'dataflow', 'awesome') -> 'dataflow is awesome'``  
___
### <code>contains</code>
<code><b>contains(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => boolean</b></code><br/><br/>
Возвращает значение true, если какой-либо элемент в указанном массиве оценивается как true в предоставленном предикате. Функция Contains принимает ссылку на один элемент в функции предиката как #item.  
* ``contains([1, 2, 3, 4], #item == 3) -> true``  
* ``contains([1, 2, 3, 4], #item > 5) -> false``  
___
### <code>cos</code>
<code><b>cos(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет значение косинуса.  
* ``cos(10) -> -0.8390715290764524``  
___
### <code>cosh</code>
<code><b>cosh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет гиперболический косинус значения.  
* ``cosh(0) -> 1.0``  
___
### <code>crc32</code>
<code><b>crc32(<i>&lt;value1&gt;</i> : any, ...) => long</b></code><br/><br/>
Вычисляет хэш-код CRC32 набора столбцов различных примитивных типов данных с заданной разрядностью, которая может иметь только значения 0(256), 224, 256, 384, 512. Его можно использовать для вычисления отпечатка для строки.  
* ``crc32(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> 3630253689L``  
___
### <code>currentDate</code>
<code><b>currentDate([<i>&lt;value1&gt;</i> : string]) => date</b></code><br/><br/>
Возвращает текущую дату начала выполнения этого задания. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется местный часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. [https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html). 
* ``currentDate() == toDate('2250-12-31') -> false``  
* ``currentDate('PST')  == toDate('2250-12-31') -> false``  
* ``currentDate('America/New_York')  == toDate('2250-12-31') -> false``  
___
### <code>currentTimestamp</code>
<code><b>currentTimestamp() => timestamp</b></code><br/><br/>
Возвращает текущую метку времени, когда задание начинает выполняться с местным часовым поясом.  
* ``currentTimestamp() == toTimestamp('2250-12-31 12:12:12') -> false``  
___
### <code>currentUTC</code>
<code><b>currentUTC([<i>&lt;value1&gt;</i> : string]) => timestamp</b></code><br/><br/>
Возвращает текущую метку времени в формате UTC. Если вы хотите, чтобы текущее время было интерпретировано на другом часовом поясе, отличном от часового пояса кластера, можно передать дополнительный часовой пояс в формате "GMT", "PST", "UTC", "America/Кайман". По умолчанию используется текущий часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. [https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html). Чтобы преобразовать время в формате UTC в другой часовой пояс, используйте `fromUTC()` .  
* ``currentUTC() == toTimestamp('2050-12-12 19:18:12') -> false``  
* ``currentUTC() != toTimestamp('2050-12-12 19:18:12') -> true``  
* ``fromUTC(currentUTC(), 'Asia/Seoul') != toTimestamp('2050-12-12 19:18:12') -> true``  
___
### <code>dayOfMonth</code>
<code><b>dayOfMonth(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
Возвращает день месяца с учетом даты.  
* ``dayOfMonth(toDate('2018-06-08')) -> 8``  
___
### <code>dayOfWeek</code>
<code><b>dayOfWeek(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
Возвращает день недели для заданной даты. 1 – воскресенье, 2 – понедельник..., 7 — суббота.  
* ``dayOfWeek(toDate('2018-06-08')) -> 6``  
___
### <code>dayOfYear</code>
<code><b>dayOfYear(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
Возвращает день года, заданный в качестве даты.  
* ``dayOfYear(toDate('2016-04-09')) -> 100``  
___
### <code>days</code>
<code><b>days(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
Длительность в миллисекундах для количества дней.  
* ``days(2) -> 172800000L``  
___
### <code>degrees</code>
<code><b>degrees(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Преобразует радианы в градусы.  
* ``degrees(3.141592653589793) -> 180``  
___
### <code>divide</code>
<code><b>divide(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
Делит пару чисел. То же, что и `/` оператор.  
* ``divide(20, 10) -> 2``  
* ``20 / 10 -> 2``  
___
### <code>endsWith</code>
<code><b>endsWith(<i>&lt;string&gt;</i> : string, <i>&lt;substring to check&gt;</i> : string) => boolean</b></code><br/><br/>
Проверяет, заканчивается ли строка указанной строкой.  
* ``endsWith('dumbo', 'mbo') -> true``  
___
### <code>equals</code>
<code><b>equals(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
Оператор сравнения на равенство. Аналогичен оператору = =.  
* ``equals(12, 24) -> false``  
* ``12 == 24 -> false``  
* ``'bad' == 'bad' -> true``  
* ``isNull('good' == toString(null)) -> true``  
* ``isNull(null == null) -> true``  
___
### <code>equalsIgnoreCase</code>
<code><b>equalsIgnoreCase(<i>&lt;value1&gt;</i> : string, <i>&lt;value2&gt;</i> : string) => boolean</b></code><br/><br/>
Оператор сравнения на равенство без учета регистра. Аналогичен оператору <=>.  
* ``'abc'<=>'Abc' -> true``  
* ``equalsIgnoreCase('abc', 'Abc') -> true``  
___
### <code>escape</code>
<code><b>escape(<i>&lt;string_to_escape&gt;</i> : string, <i>&lt;format&gt;</i> : string) => string</b></code><br/><br/>
Обходит строку в соответствии с форматом. Литеральные значения допустимых форматов: "JSON", "XML", "ECMAScript", "HTML", "Java".
___
### <code>factorial</code>
<code><b>factorial(<i>&lt;value1&gt;</i> : number) => long</b></code><br/><br/>
Вычисляет факториал числа.  
* ``factorial(5) -> 120``  
___
### <code>false</code>
<code><b>false() => boolean</b></code><br/><br/>
Всегда возвращает значение false. Используйте функцию, `syntax(false())` если есть столбец с именем "false".  
* ``(10 + 20 > 30) -> false``  
* ``(10 + 20 > 30) -> false()``
___
### <code>floor</code>
<code><b>floor(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Возвращает максимальное целое число, которое не больше числа.  
* ``floor(-0.1) -> -1``  
___
### <code>fromBase64</code>
<code><b>fromBase64(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Кодирует заданную строку в Base64.  
* ``fromBase64('Z3VuY2h1cw==') -> 'gunchus'``  
___
### <code>fromUTC</code>
<code><b>fromUTC(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => timestamp</b></code><br/><br/>
Преобразует в метку времени из времени в формате UTC. При необходимости вы можете передать часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется текущий часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``fromUTC(currentTimestamp()) == toTimestamp('2050-12-12 19:18:12') -> false``  
* ``fromUTC(currentTimestamp(), 'Asia/Seoul') != toTimestamp('2050-12-12 19:18:12') -> true``  
___
### <code>greater</code>
<code><b>greater(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
Оператор сравнения "больше". Аналогичен оператору >.  
* ``greater(12, 24) -> false``  
* ``('dumbo' > 'dum') -> true``  
* ``(toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS') > toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS')) -> true``  
___
### <code>greaterOrEqual</code>
<code><b>greaterOrEqual(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
Оператор сравнения "больше чем или равно". То же, что и оператор >=.  
* ``greaterOrEqual(12, 12) -> true``  
* ``('dumbo' >= 'dum') -> true``  
___
### <code>greatest</code>
<code><b>greatest(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
Возвращает наибольшее значение из списка значений в качестве пропускающих значения NULL. Возвращает значение null, если все входные данные имеют значение null.  
* ``greatest(10, 30, 15, 20) -> 30``  
* ``greatest(10, toInteger(null), 20) -> 20``  
* ``greatest(toDate('2010-12-12'), toDate('2011-12-12'), toDate('2000-12-12')) -> toDate('2011-12-12')``  
* ``greatest(toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS'), toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS')) -> toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS')``  
___
### <code>hasColumn</code>
<code><b>hasColumn(<i>&lt;column name&gt;</i> : string, [<i>&lt;stream name&gt;</i> : string]) => boolean</b></code><br/><br/>
Проверяет значение столбца по имени в потоке. В качестве второго аргумента можно передать необязательное имя потока. Имена столбцов, известные во время проектирования, должны определяться только по имени. Вычисленные входные данные не поддерживаются, но вы можете использовать подстановки параметров.  
* ``hasColumn('parent')``  
___
### <code>hour</code>
<code><b>hour(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
Возвращает значение часа метки времени. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется местный часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``hour(toTimestamp('2009-07-30 12:58:59')) -> 12``  
* ``hour(toTimestamp('2009-07-30 12:58:59'), 'PST') -> 12``  
___
### <code>hours</code>
<code><b>hours(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
Длительность в миллисекундах для количества часов.  
* ``hours(2) -> 7200000L``  
___
### <code>iif</code>
<code><b>iif(<i>&lt;condition&gt;</i> : boolean, <i>&lt;true_expression&gt;</i> : any, [<i>&lt;false_expression&gt;</i> : any]) => any</b></code><br/><br/>
В зависимости от условия применяется одно или другое значение. Если другое значение не указано, считается что оно имеет значение NULL. Оба значения должны быть совместимыми (числовыми, строковыми...). * ``iif(10 + 20 == 30, 'dumbo', 'gumbo') -> 'dumbo'``  
* ``iif(10 > 30, 'dumbo', 'gumbo') -> 'gumbo'``  
* ``iif(month(toDate('2018-12-01')) == 12, 345.12, 102.67) -> 345.12``  
___
### <code>iifNull</code>
<code><b>iifNull(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : any], ...) => any</b></code><br/><br/>
Проверяет, имеет ли первый параметр значение NULL. Если значение не равно NULL, возвращается первый параметр. Если значение равно NULL, возвращается второй параметр. Если указаны три параметра, поведение аналогично iif (isNull(значение1), значение2, значение3), а третий параметр возвращается, если первое значение не равно NULL.  
* ``iifNull(10, 20) -> 10``  
* ``iifNull(null, 20, 40) -> 20``  
* ``iifNull('azure', 'data', 'factory') -> 'factory'``  
* ``iifNull(null, 'data', 'factory') -> 'data'``  
___
### <code>in</code>
<code><b>in(<i>&lt;array of items&gt;</i> : array, <i>&lt;item to find&gt;</i> : any) => boolean</b></code><br/><br/>
Проверяет, находится ли элемент в массиве.  
* ``in([10, 20, 30], 10) -> true``  
* ``in(['good', 'kid'], 'bad') -> false``  
___
### <code>initCap</code>
<code><b>initCap(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Преобразует первую букву каждого слова в верхний регистр. Слова обозначаются символами пробела.  
* ``initCap('cool iceCREAM') -> 'Cool Icecream'``  
___
### <code>instr</code>
<code><b>instr(<i>&lt;string&gt;</i> : string, <i>&lt;substring to find&gt;</i> : string) => integer</b></code><br/><br/>
Находит позицию (на основе 1) подстроки в строке. 0 возвращается, если не найден.  
* ``instr('dumbo', 'mbo') -> 3``  
* ``instr('microsoft', 'o') -> 5``  
* ``instr('good', 'bad') -> 0``  
___
### <code>isDelete</code>
<code><b>isDelete([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
Проверяет, помечена ли строка для удаления. Для преобразований, которые принимают более одного входного потока, можно передавать индекс потока (отсчитываемый от единицы). Индекс потока должен быть равен 1 или 2, а значение по умолчанию — 1.  
* ``isDelete()``  
* ``isDelete(1)``  
___
### <code>isError</code>
<code><b>isError([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
Проверяет, помечена ли строка как ошибка. Для преобразований, которые принимают более одного входного потока, можно передавать индекс потока (отсчитываемый от единицы). Индекс потока должен быть равен 1 или 2, а значение по умолчанию — 1.  
* ``isError()``  
* ``isError(1)``  
___
### <code>isIgnore</code>
<code><b>isIgnore([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
Проверяет, помечена ли строка для игнорирования. Для преобразований, которые принимают более одного входного потока, можно передавать индекс потока (отсчитываемый от единицы). Индекс потока должен быть равен 1 или 2, а значение по умолчанию — 1.  
* ``isIgnore()``  
* ``isIgnore(1)``  
___
### <code>isInsert</code>
<code><b>isInsert([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
Проверяет, помечена ли строка для вставки. Для преобразований, которые принимают более одного входного потока, можно передавать индекс потока (отсчитываемый от единицы). Индекс потока должен быть равен 1 или 2, а значение по умолчанию — 1.  
* ``isInsert()``  
* ``isInsert(1)``  
___
### <code>isMatch</code>
<code><b>isMatch([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
Проверяет соответствие строки при поиске. Для преобразований, которые принимают более одного входного потока, можно передавать индекс потока (отсчитываемый от единицы). Индекс потока должен быть равен 1 или 2, а значение по умолчанию — 1.  
* ``isMatch()``  
* ``isMatch(1)``  
___
### <code>isNull</code>
<code><b>isNull(<i>&lt;value1&gt;</i> : any) => boolean</b></code><br/><br/>
Проверяет, является ли значение нулевым.  
* ``isNull(NULL()) -> true``  
* ``isNull('') -> false``  
___
### <code>isUpdate</code>
<code><b>isUpdate([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
Проверяет, помечена ли строка для обновления. Для преобразований, которые принимают более одного входного потока, можно передавать индекс потока (отсчитываемый от единицы). Индекс потока должен быть равен 1 или 2, а значение по умолчанию — 1.  
* ``isUpdate()``  
* ``isUpdate(1)``  
___
### <code>isUpsert</code>
<code><b>isUpsert([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
Проверяет, помечена ли строка для вставки. Для преобразований, которые принимают более одного входного потока, можно передавать индекс потока (отсчитываемый от единицы). Индекс потока должен быть равен 1 или 2, а значение по умолчанию — 1.  
* ``isUpsert()``  
* ``isUpsert(1)``  
___
### <code>lastDayOfMonth</code>
<code><b>lastDayOfMonth(<i>&lt;value1&gt;</i> : datetime) => date</b></code><br/><br/>
Возвращает последнюю дату месяца с учетом даты.  
* ``lastDayOfMonth(toDate('2009-01-12')) -> toDate('2009-01-31')``  
___
### <code>least</code>
<code><b>least(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
Оператор сравнения "меньше чем или равно". То же, что и оператор <=.  
* ``least(10, 30, 15, 20) -> 10``  
* ``least(toDate('2010-12-12'), toDate('2011-12-12'), toDate('2000-12-12')) -> toDate('2000-12-12')``  
___
### <code>left</code>
<code><b>left(<i>&lt;string to subset&gt;</i> : string, <i>&lt;number of characters&gt;</i> : integral) => string</b></code><br/><br/>
Извлекает подстроку, начиная с индекса 1, с необходимым количеством знаков. То же, что и substring (STR, 1, n).  
* ``left('bojjus', 2) -> 'bo'``  
* ``left('bojjus', 20) -> 'bojjus'``  
___
### <code>length</code>
<code><b>length(<i>&lt;value1&gt;</i> : string) => integer</b></code><br/><br/>
Возвращает длину строки.  
* ``length('dumbo') -> 5``  
___
### <code>lesser</code>
<code><b>lesser(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
Оператор сравнения "меньше". Аналогичен оператору <.  
* ``lesser(12, 24) -> true``  
* ``('abcd' < 'abc') -> false``  
* ``(toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS') < toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS')) -> true``  
___
### <code>lesserOrEqual</code>
<code><b>lesserOrEqual(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
Оператор сравнения "меньше чем или равно". То же, что и оператор <=.  
* ``lesserOrEqual(12, 12) -> true``  
* ``('dumbo' <= 'dum') -> false``  
___
### <code>levenshtein</code>
<code><b>levenshtein(<i>&lt;from string&gt;</i> : string, <i>&lt;to string&gt;</i> : string) => integer</b></code><br/><br/>
Возвращает Левенштейна расстояние между двумя строками.  
* ``levenshtein('boys', 'girls') -> 4``  
___
### <code>like</code>
<code><b>like(<i>&lt;string&gt;</i> : string, <i>&lt;pattern match&gt;</i> : string) => boolean</b></code><br/><br/>
Шаблон представляет собой строку, которая сопоставляется буквально. Исключениями являются следующие специальные символы: _ соответствует любому одному знаку во входной строке (аналогично . в ```posix``` регулярных выражениях)% соответствует нулю или большему числу символов во входных данных (аналогично. * в ```posix``` регулярных выражениях).
Escape-символом является ''. Если escape-символ предшествует специальному символу или другому escape-символу, следующий знак сопоставляется буквально. Недопустимо экранировать любые другие знаки.  
* ``like('icecream', 'ice%') -> true``  
___
### <code>locate</code>
<code><b>locate(<i>&lt;substring to find&gt;</i> : string, <i>&lt;string&gt;</i> : string, [<i>&lt;from index - 1-based&gt;</i> : integral]) => integer</b></code><br/><br/>
Находит позицию (на основе 1) подстроки в строке, начиная определение с определенной позиции. Если позиция опущена, то она отсчитывается с начала строки. 0 возвращается, если не найден.  
* ``locate('mbo', 'dumbo') -> 3``  
* ``locate('o', 'microsoft', 6) -> 7``  
* ``locate('bad', 'good') -> 0``  
___
### <code>log</code>
<code><b>log(<i>&lt;value1&gt;</i> : number, [<i>&lt;value2&gt;</i> : number]) => double</b></code><br/><br/>
Вычисляет значение логарифма. При использовании необязательного базового объекта можно указать номер Эйлера.  
* ``log(100, 10) -> 2``  
___
### <code>log10</code>
<code><b>log10(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет значение журнала на основе 10 базовых значений.  
* ``log10(100) -> 2``  
___
### <code>lower</code>
<code><b>lower(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Строка в нижнем регистре.  
* ``lower('GunChus') -> 'gunchus'``  
___
### <code>lpad</code>
<code><b>lpad(<i>&lt;string to pad&gt;</i> : string, <i>&lt;final padded length&gt;</i> : integral, <i>&lt;padding&gt;</i> : string) => string</b></code><br/><br/>
Слева дополняет строку прилагаемым отступом определенной длины. Если строка больше или равна длине, то она усекается до длины.  
* ``lpad('dumbo', 10, '-') -> '-----dumbo'``  
* ``lpad('dumbo', 4, '-') -> 'dumb'``  
* ``lpad('dumbo', 8, '<>') -> '<><dumbo'``  
___
### <code>ltrim</code>
<code><b>ltrim(<i>&lt;string to trim&gt;</i> : string, [<i>&lt;trim characters&gt;</i> : string]) => string</b></code><br/><br/>
Обрезает строку начальных знаков слева. Если второй параметр не указан, удаляются пробелы. В противном случае все символы, указанные во втором параметре, обрезаются.  
* ``ltrim('  dumbo  ') -> 'dumbo  '``  
* ``ltrim('!--!du!mbo!', '-!') -> 'du!mbo!'``  
___
### <code>md5</code>
<code><b>md5(<i>&lt;value1&gt;</i> : any, ...) => string</b></code><br/><br/>
Вычисляет хэш-код MD5 набора столбцов с различными типами данных и возвращает шестнадцатеричную строку из 32 знаков. Его можно использовать для вычисления отпечатка для строки.  
* ``md5(5, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> '4ce8a880bd621a1ffad0bca905e1bc5a'``  
___
### <code>millisecond</code>
<code><b>millisecond(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
Возвращает значение миллисекунды даты. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется местный часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``millisecond(toTimestamp('2009-07-30 12:58:59.871', 'yyyy-MM-dd HH:mm:ss.SSS')) -> 871``  
___
### <code>milliseconds</code>
<code><b>milliseconds(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
Длительность в миллисекундах для числа миллисекунд.  
* ``milliseconds(2) -> 2L``  
___
### <code>minus</code>
<code><b>minus(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
Вычитает числа. Вычесть число дней с даты. Вычитает длительность из метки времени. Вычитает две метки времени, чтобы получить разницу в миллисекундах. То же, что и оператор-.  
* ``minus(20, 10) -> 10``  
* ``20 - 10 -> 10``  
* ``minus(toDate('2012-12-15'), 3) -> toDate('2012-12-12')``  
* ``toDate('2012-12-15') - 3 -> toDate('2012-12-12')``  
* ``toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS') + (days(1) + hours(2) - seconds(10)) -> toTimestamp('2019-02-04 07:19:18.871', 'yyyy-MM-dd HH:mm:ss.SSS')``  
* ``toTimestamp('2019-02-03 05:21:34.851', 'yyyy-MM-dd HH:mm:ss.SSS') - toTimestamp('2019-02-03 05:21:36.923', 'yyyy-MM-dd HH:mm:ss.SSS') -> -2072``  
___
### <code>minute</code>
<code><b>minute(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
Возвращает значение минуты метки времени. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется местный часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``minute(toTimestamp('2009-07-30 12:58:59')) -> 58``  
* ``minute(toTimestamp('2009-07-30 12:58:59'), 'PST') -> 58``  
___
### <code>minutes</code>
<code><b>minutes(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
Длительность в миллисекундах для количества минут.  
* ``minutes(2) -> 120000L``  
___
### <code>mod</code>
<code><b>mod(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
Модуль пары чисел. То же, что и оператор%.  
* ``mod(20, 8) -> 4``  
* ``20 % 8 -> 4``  
___
### <code>month</code>
<code><b>month(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
Возвращает значение месяца для даты или метки времени.  
* ``month(toDate('2012-8-8')) -> 8``  
___
### <code>monthsBetween</code>
<code><b>monthsBetween(<i>&lt;from date/timestamp&gt;</i> : datetime, <i>&lt;to date/timestamp&gt;</i> : datetime, [<i>&lt;roundoff&gt;</i> : boolean], [<i>&lt;time zone&gt;</i> : string]) => double</b></code><br/><br/>
Возвращает количество месяцев между двумя датами. Вычисление можно округлить. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется местный часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``monthsBetween(toTimestamp('1997-02-28 10:30:00'), toDate('1996-10-30')) -> 3.94959677``  
___
### <code>multiply</code>
<code><b>multiply(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
Умножает пару чисел. То же, что и оператор *.  
* ``multiply(20, 10) -> 200``  
* ``20 * 10 -> 200``  
___
### <code>negate</code>
<code><b>negate(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Инвертирует число. Превращает положительные числа в отрицательные и наоборот.  
* ``negate(13) -> -13``  
___
### <code>nextSequence</code>
<code><b>nextSequence() => long</b></code><br/><br/>
Возвращает следующую уникальную последовательность. Это число является последовательным только в пределах секции и имеет префикс partitionId.  
* ``nextSequence() == 12313112 -> false``  
___
### <code>normalize</code>
<code><b>normalize(<i>&lt;String to normalize&gt;</i> : string) => string</b></code><br/><br/>
Нормализует строковое значение для разделения символов Юникода с диакритическими знаками.  
* ``regexReplace(normalize('bo²s'), `\p{M}`, '') -> 'boys'``  
___
### <code>not</code>
<code><b>not(<i>&lt;value1&gt;</i> : boolean) => boolean</b></code><br/><br/>
Оператор логического отрицания.  
* ``not(true) -> false``  
* ``not(10 == 20) -> true``  
___
### <code>notEquals</code>
<code><b>notEquals(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
Оператор сравнения на неравенство. То же, что и оператор! =.  
* ``12 != 24 -> true``  
* ``'bojjus' != 'bo' + 'jjus' -> false``  
___
### <code>notNull</code>
<code><b>notNull(<i>&lt;value1&gt;</i> : any) => boolean</b></code><br/><br/>
Проверяет, является ли значение значением, не РАВНым NULL.  
* ``notNull(NULL()) -> false``  
* ``notNull('') -> true``  
___
### <code>null</code>
<code><b>null() => null</b></code><br/><br/>
Возвращает значение NULL. Используйте функцию, `syntax(null())` если есть столбец с именем NULL. Любая операция, которая использует, приведет к значению NULL.  
* ``isNull('dumbo' + null) -> true``  
* ``isNull(10 * null) -> true``  
* ``isNull('') -> false``  
* ``isNull(10 + 20) -> false``  
* ``isNull(10/0) -> true``  
___
### <code>or</code>
<code><b>or(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
Логический оператор ИЛИ. То же, что и | |.  
* ``or(true, false) -> true``  
* ``true || false -> true``  
___
### <code>pMod</code>
<code><b>pMod(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
Положительный модуль пары чисел.  
* ``pmod(-20, 8) -> 4``  
___
### <code>partitionId</code>
<code><b>partitionId() => integer</b></code><br/><br/>
Возвращает идентификатор текущей секции, в которой находится входная строка.  
* ``partitionId()``  
___
### <code>power</code>
<code><b>power(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
Порождает одно число в степень другого.  
* ``power(10, 2) -> 100``  
___
### <code>random</code>
<code><b>random(<i>&lt;value1&gt;</i> : integral) => long</b></code><br/><br/>
Возвращает случайное число, заданное необязательным начальным значением в секции. Начальное значение должно быть фиксированным значением и использоваться в сочетании с partitionId для получения случайных значений * ``random(1) == 1 -> false``
___
### <code>regexExtract</code>
<code><b>regexExtract(<i>&lt;string&gt;</i> : string, <i>&lt;regex to find&gt;</i> : string, [<i>&lt;match group 1-based index&gt;</i> : integral]) => string</b></code><br/><br/>
Извлекает соответствующую подстроку для данного шаблона регулярного выражения. Последний параметр идентифицирует группу для сопоставления и, если не указан, по умолчанию равен 1. <regex>Чтобы сопоставить строку без экранирования, используйте символ "" (Обратная кавычка).  
* ``regexExtract('Cost is between 600 and 800 dollars', '(\\d+) and (\\d+)', 2) -> '800'``  
* ``regexExtract('Cost is between 600 and 800 dollars', `(\d+) and (\d+)`, 2) -> '800'``  
___
### <code>regexMatch</code>
<code><b>regexMatch(<i>&lt;string&gt;</i> : string, <i>&lt;regex to match&gt;</i> : string) => boolean</b></code><br/><br/>
Проверяет, соответствует ли строка заданному шаблону регулярного выражения. <regex>Чтобы сопоставить строку без экранирования, используйте символ "" (Обратная кавычка).  
* ``regexMatch('200.50', '(\\d+).(\\d+)') -> true``  
* ``regexMatch('200.50', `(\d+).(\d+)`) -> true``  
___
### <code>regexReplace</code>
<code><b>regexReplace(<i>&lt;string&gt;</i> : string, <i>&lt;regex to find&gt;</i> : string, <i>&lt;substring to replace&gt;</i> : string) => string</b></code><br/><br/>
Заменить все вхождения шаблона регулярного выражения другой подстрокой в заданной строке использовать ' <regex> ' (Обратная кавычка) для сопоставления строки без экранирования.  
* ``regexReplace('100 and 200', '(\\d+)', 'bojjus') -> 'bojjus and bojjus'``  
* ``regexReplace('100 and 200', `(\d+)`, 'gunchus') -> 'gunchus and gunchus'``  
___
### <code>regexSplit</code>
<code><b>regexSplit(<i>&lt;string to split&gt;</i> : string, <i>&lt;regex expression&gt;</i> : string) => array</b></code><br/><br/>
Разделяет строку на основе разделителя на основе регулярного выражения и возвращает массив строк.  
* ``regexSplit('bojjusAgunchusBdumbo', `[CAB]`) -> ['bojjus', 'gunchus', 'dumbo']``  
* ``regexSplit('bojjusAgunchusBdumboC', `[CAB]`) -> ['bojjus', 'gunchus', 'dumbo', '']``  
* ``(regexSplit('bojjusAgunchusBdumboC', `[CAB]`)[1]) -> 'bojjus'``  
* ``isNull(regexSplit('bojjusAgunchusBdumboC', `[CAB]`)[20]) -> true``  
___
### <code>replace</code>
<code><b>replace(<i>&lt;string&gt;</i> : string, <i>&lt;substring to find&gt;</i> : string, [<i>&lt;substring to replace&gt;</i> : string]) => string</b></code><br/><br/>
Заменяет все вхождения подстроки другой подстрокой в ​​заданной строке. Если последний параметр пропущен, по умолчанию используется пустая строка.  
* ``replace('doggie dog', 'dog', 'cat') -> 'catgie cat'``  
* ``replace('doggie dog', 'dog', '') -> 'gie '``  
* ``replace('doggie dog', 'dog') -> 'gie '``  
___
### <code>reverse</code>
<code><b>reverse(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Изменяет направление строки на противоположную.  
* ``reverse('gunchus') -> 'suhcnug'``  
___
### <code>right</code>
<code><b>right(<i>&lt;string to subset&gt;</i> : string, <i>&lt;number of characters&gt;</i> : integral) => string</b></code><br/><br/>
Извлекает подстроку с количеством знаков справа. То же, что и substring (STR, LENGTH (STR)-n, n).  
* ``right('bojjus', 2) -> 'us'``  
* ``right('bojjus', 20) -> 'bojjus'``  
___
### <code>rlike</code>
<code><b>rlike(<i>&lt;string&gt;</i> : string, <i>&lt;pattern match&gt;</i> : string) => boolean</b></code><br/><br/>
Проверяет, соответствует ли строка заданному шаблону регулярного выражения.  
* ``rlike('200.50', `(\d+).(\d+)`) -> true``  
* ``rlike('bogus', `M[0-9]+.*`) -> false``  
___
### <code>round</code>
<code><b>round(<i>&lt;number&gt;</i> : number, [<i>&lt;scale to round&gt;</i> : number], [<i>&lt;rounding option&gt;</i> : integral]) => double</b></code><br/><br/>
Округляет число с учетом дополнительного масштаба и режима округления. Если масштаб не указан, по умолчанию используется значение 0. Если режим не указан, по умолчанию используется значение ROUND_HALF_UP(5). Округление значений включает 1-ROUND_UP 2-ROUND_DOWN 3-ROUND_CEILING 4-ROUND_FLOOR 5-ROUND_HALF_UP 6-ROUND_HALF_DOWN 7 — ROUND_HALF_EVEN 8 — ROUND_UNNECESSARY.  
* ``round(100.123) -> 100.0``  
* ``round(2.5, 0) -> 3.0``  
* ``round(5.3999999999999995, 2, 7) -> 5.40``  
___
### <code>rpad</code>
<code><b>rpad(<i>&lt;string to pad&gt;</i> : string, <i>&lt;final padded length&gt;</i> : integral, <i>&lt;padding&gt;</i> : string) => string</b></code><br/><br/>
Дополняет строку прилагаемым отступом справа до достижения определенной длины. Если строка больше или равна длине, то она усекается до длины.  
* ``rpad('dumbo', 10, '-') -> 'dumbo-----'``  
* ``rpad('dumbo', 4, '-') -> 'dumb'``  
* ``rpad('dumbo', 8, '<>') -> 'dumbo<><'``  
___
### <code>rtrim</code>rtrim</code>
<code><b>rtrim(<i>&lt;string to trim&gt;</i> : string, [<i>&lt;trim characters&gt;</i> : string]) => string</b></code><br/><br/>
Право обрезает строку конечных знаков. Если второй параметр не указан, удаляются пробелы. В противном случае все символы, указанные во втором параметре, обрезаются.  
* ``rtrim('  dumbo  ') -> '  dumbo'``  
* ``rtrim('!--!du!mbo!', '-!') -> '!--!du!mbo'``  
___
### <code>second</code>
<code><b>second(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
Возвращает второе значение даты. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется местный часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``second(toTimestamp('2009-07-30 12:58:59')) -> 59``  
___
### <code>seconds</code>
<code><b>seconds(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
Длительность в миллисекундах для количества секунд.  
* ``seconds(2) -> 2000L``  
___
### <code>sha1</code>
<code><b>sha1(<i>&lt;value1&gt;</i> : any, ...) => string</b></code><br/><br/>
Вычисляет хэш-код SHA-1 набора столбцов с различными типами данных и возвращает шестнадцатеричную строку из 40 знаков. Его можно использовать для вычисления отпечатка для строки.  
* ``sha1(5, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> '46d3b478e8ec4e1f3b453ac3d8e59d5854e282bb'``  
___
### <code>sha2</code>
<code><b>sha2(<i>&lt;value1&gt;</i> : integer, <i>&lt;value2&gt;</i> : any, ...) => string</b></code><br/><br/>
Вычисляет хэш-код SHA-2 набора столбцов различных примитивных типов данных с заданной разрядностью, которая может иметь только значения 0(256), 224, 256, 384, 512. Его можно использовать для вычисления отпечатка для строки.  
* ``sha2(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> 'afe8a553b1761c67d76f8c31ceef7f71b66a1ee6f4e6d3b5478bf68b47d06bd3'``  
___
### <code>sin</code>
<code><b>sin(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет значение синуса.  
* ``sin(2) -> 0.9092974268256817``  
___
### <code>sinh</code>
<code><b>sinh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет гиперболический синус значения.  
* ``sinh(0) -> 0.0``  
___
### <code>soundex</code>
<code><b>soundex(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Возвращает ```soundex``` код для строки.  
* ``soundex('genius') -> 'G520'``  
___
### <code>split</code>
<code><b>split(<i>&lt;string to split&gt;</i> : string, <i>&lt;split characters&gt;</i> : string) => array</b></code><br/><br/>
Разделяет строку на основе разделителя и возвращает массив строк.  
* ``split('bojjus,guchus,dumbo', ',') -> ['bojjus', 'guchus', 'dumbo']``  
* ``split('bojjus,guchus,dumbo', '|') -> ['bojjus,guchus,dumbo']``  
* ``split('bojjus, guchus, dumbo', ', ') -> ['bojjus', 'guchus', 'dumbo']``  
* ``split('bojjus, guchus, dumbo', ', ')[1] -> 'bojjus'``  
* ``isNull(split('bojjus, guchus, dumbo', ', ')[0]) -> true``  
* ``isNull(split('bojjus, guchus, dumbo', ', ')[20]) -> true``  
* ``split('bojjusguchusdumbo', ',') -> ['bojjusguchusdumbo']``  
___
### <code>sqrt</code>
<code><b>sqrt(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет квадратный корень числа.  
* ``sqrt(9) -> 3``  
___
### <code>startsWith</code>
<code><b>startsWith(<i>&lt;string&gt;</i> : string, <i>&lt;substring to check&gt;</i> : string) => boolean</b></code><br/><br/>
Проверяет, начинается ли строка с указанной строки.  
* ``startsWith('dumbo', 'du') -> true``  
___
### <code>subDays</code>
<code><b>subDays(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;days to subtract&gt;</i> : integral) => datetime</b></code><br/><br/>
Вычесть дни из даты или метки времени. То же, что и оператор-для Date.  
* ``subDays(toDate('2016-08-08'), 1) -> toDate('2016-08-07')``  
___
### <code>subMonths</code>
<code><b>subMonths(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;months to subtract&gt;</i> : integral) => datetime</b></code><br/><br/>
Вычитает месяцы из даты или метки времени.  
* ``subMonths(toDate('2016-09-30'), 1) -> toDate('2016-08-31')``  
___
### <code>substring</code>
<code><b>substring(<i>&lt;string to subset&gt;</i> : string, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of characters&gt;</i> : integral]) => string</b></code><br/><br/>
Извлекает подстроку определенной длины из заданной позиции. Позиция отсчитывается от единицы. Если длина опущена, по умолчанию используется конец строки.  
* ``substring('Cat in the hat', 5, 2) -> 'in'``  
* ``substring('Cat in the hat', 5, 100) -> 'in the hat'``  
* ``substring('Cat in the hat', 5) -> 'in the hat'``  
* ``substring('Cat in the hat', 100, 100) -> ''``  
___
### <code>tan</code>
<code><b>tan(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет значение тангенса.  
* ``tan(0) -> 0.0``  
___
### <code>tanh</code>
<code><b>tanh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Вычисляет гиперболический тангенс значения.  
* ``tanh(0) -> 0.0``  
___
### <code>translate</code>
<code><b>translate(<i>&lt;string to translate&gt;</i> : string, <i>&lt;lookup characters&gt;</i> : string, <i>&lt;replace characters&gt;</i> : string) => string</b></code><br/><br/>
Заменяет один набор знаков другим набором знаков в строке. Символы имеют от 1 до 1 замены.  
* ``translate('(bojjus)', '()', '[]') -> '[bojjus]'``  
* ``translate('(gunchus)', '()', '[') -> '[gunchus'``  
___
### <code>trim</code>
<code><b>trim(<i>&lt;string to trim&gt;</i> : string, [<i>&lt;trim characters&gt;</i> : string]) => string</b></code><br/><br/>
Обрезает строку начальных и конечных знаков. Если второй параметр не указан, удаляются пробелы. В противном случае все символы, указанные во втором параметре, обрезаются.  
* ``trim('  dumbo  ') -> 'dumbo'``  
* ``trim('!--!du!mbo!', '-!') -> 'du!mbo'``  
___
### <code>true</code>
<code><b>true() => boolean</b></code><br/><br/>
Всегда возвращает истинное значение. Используйте функцию, `syntax(true())` если есть столбец с именем "true".  
* ``(10 + 20 == 30) -> true``  
* ``(10 + 20 == 30) -> true()``  
___
### <code>typeMatch</code>
<code><b>typeMatch(<i>&lt;type&gt;</i> : string, <i>&lt;base type&gt;</i> : string) => boolean</b></code><br/><br/>
Сопоставляет тип столбца. Может использоваться только в выражениях шаблонов. число соответствует Short, Integer, Long, Double, float или Decimal, целочисленные совпадения Short, Integer, Long, дробные числа соответствуют Double, float, Decimal и DateTime соответствуют типу Date или timestamp.  
* ``typeMatch(type, 'number')``  
* ``typeMatch('date', 'datetime')``  
___
### <code>unescape</code>
<code><b>unescape(<i>&lt;string_to_escape&gt;</i> : string, <i>&lt;format&gt;</i> : string) => string</b></code><br/><br/>
Отменяет escape-последовательность строки в соответствии с форматом. Литеральные значения допустимых форматов: "JSON", "XML", "ECMAScript", "HTML", "Java".
* ```unescape('{\\\\\"value\\\\\": 10}', 'json')```
* ```'{\\\"value\\\": 10}'```
___
### <code>upper</code>
<code><b>upper(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Строка в верхнем регистре.  
* ``upper('bojjus') -> 'BOJJUS'``  
___
### <code>uuid</code>
<code><b>uuid() => string</b></code><br/><br/>
Возвращает созданный UUID.  
* ``uuid()``  
___
### <code>weekOfYear</code>
<code><b>weekOfYear(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
Возвращает неделю года с учетом даты.  
* ``weekOfYear(toDate('2008-02-20')) -> 8``  
___
### <code>weeks</code>
<code><b>weeks(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
Длительность в миллисекундах для количества недель.  
* ``weeks(2) -> 1209600000L``  
___
### <code>xor</code>
<code><b>xor(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
Логический оператор XOR. Аналогичен оператору ^.  
* ``xor(true, false) -> true``  
* ``xor(true, true) -> false``  
* ``true ^ false -> true``  
___
### <code>year</code>
<code><b>year(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
Возвращает значение года для даты.  
* ``year(toDate('2012-8-8')) -> 2012``  

## <a name="aggregate-functions"></a>Агрегатные функции
Следующие функции доступны только в преобразованиях «Статистическая обработка», «сведение», «отмена свертывания» и «окно».
___
### <code>avg</code>
<code><b>avg(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Возвращает среднее значение столбца.  
* ``avg(sales)``  
___
### <code>avgIf</code>
<code><b>avgIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
В зависимости от критерия Возвращает среднее значение столбца.  
* ``avgIf(region == 'West', sales)``  
___
### <code>count</code>
<code><b>count([<i>&lt;value1&gt;</i> : any]) => long</b></code><br/><br/>
Возвращает совокупное количество значений. Если указаны необязательные столбцы, то значения NULL в числе не учитываются.  
* ``count(custId)``  
* ``count(custId, custName)``  
* ``count()``  
* ``count(iif(isNull(custId), 1, NULL))``  
___
### <code>countDistinct</code>
<code><b>countDistinct(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : any], ...) => long</b></code><br/><br/>
Возвращает общее число различных значений набора столбцов.  
* ``countDistinct(custId, custName)``  
___
### <code>countIf</code>
<code><b>countIf(<i>&lt;value1&gt;</i> : boolean, [<i>&lt;value2&gt;</i> : any]) => long</b></code><br/><br/>
Возвращает совокупное количество значений на основе критериев. Если указан необязательный столбец, значения NULL игнорируются в числе.  
* ``countIf(state == 'CA' && commission < 10000, name)``  
___
### <code>covariancePopulation</code>
<code><b>covariancePopulation(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
Возвращает ковариацию Генеральной совокупности между двумя столбцами.  
* ``covariancePopulation(sales, profit)``  
___
### <code>covariancePopulationIf</code>
<code><b>covariancePopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number, <i>&lt;value3&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия получает ковариацию Генеральной совокупности двух столбцов.  
* ``covariancePopulationIf(region == 'West', sales)``  
___
### <code>covarianceSample</code>
<code><b>covarianceSample(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
Возвращает ковариацию выборки двух столбцов.  
* ``covarianceSample(sales, profit)``  
___
### <code>covarianceSampleIf</code>
<code><b>covarianceSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number, <i>&lt;value3&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия получает выборку ковариации двух столбцов.  
* ``covarianceSampleIf(region == 'West', sales, profit)``  
___
### <code>first</code>
<code><b>first(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : boolean]) => any</b></code><br/><br/>
Возвращает первое значение группы столбцов. Если второй параметр Игноренуллс опущен, он принимает значение false.  
* ``first(sales)``  
* ``first(sales, false)``  
___
### <code>kurtosis</code>
<code><b>kurtosis(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает эксцесс столбца.  
* ``kurtosis(sales)``  
___
### <code>kurtosisIf</code>
<code><b>kurtosisIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия получает эксцесс столбца.  
* ``kurtosisIf(region == 'West', sales)``  
___
### <code>last</code>
<code><b>last(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : boolean]) => any</b></code><br/><br/>
Возвращает последнее значение группы столбцов. Если второй параметр Игноренуллс опущен, он принимает значение false.  
* ``last(sales)``  
* ``last(sales, false)``  
___
### <code>max</code>
<code><b>max(<i>&lt;value1&gt;</i> : any) => any</b></code><br/><br/>
Возвращает максимальное значение столбца.  
* ``max(sales)``  
___
### <code>maxIf</code>
<code><b>maxIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
В зависимости от критерия получает максимальное значение столбца.  
* ``maxIf(region == 'West', sales)``  
___
### <code>mean</code>
<code><b>mean(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Возвращает среднее значение столбца. То же, что и AVG.  
* ``mean(sales)``  
___
### <code>meanIf</code>
<code><b>meanIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
Возвращает среднее значение столбца на основе критериев. То же, что и Авгиф.  
* ``meanIf(region == 'West', sales)``  
___
### <code>min</code>
<code><b>min(<i>&lt;value1&gt;</i> : any) => any</b></code><br/><br/>
Возвращает минимальное значение столбца.  
* ``min(sales)``  
___
### <code>minIf</code>
<code><b>minIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
В зависимости от критерия получает минимальное значение столбца.  
* ``minIf(region == 'West', sales)``  
___
### <code>skewness</code>
<code><b>skewness(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает асимметрию столбца.  
* ``skewness(sales)``  
___
### <code>skewnessIf</code>
<code><b>skewnessIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия получает асимметрию столбца.  
* ``skewnessIf(region == 'West', sales)``  
___
### <code>stddev</code>
<code><b>stddev(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает стандартное отклонение столбца.  
* ``stdDev(sales)``  
___
### <code>stddevIf</code>
<code><b>stddevIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия возвращает стандартное отклонение столбца.  
* ``stddevIf(region == 'West', sales)``  
___
### <code>stddevPopulation</code>
<code><b>stddevPopulation(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает среднеквадратичное отклонение совокупности для столбца.  
* ``stddevPopulation(sales)``  
___
### <code>stddevPopulationIf</code>
<code><b>stddevPopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия возвращает стандартное отклонение совокупности для столбца.  
* ``stddevPopulationIf(region == 'West', sales)``  
___
### <code>stddevSample</code>
<code><b>stddevSample(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает стандартное отклонение выборки для столбца.  
* ``stddevSample(sales)``  
___
### <code>stddevSampleIf</code>
<code><b>stddevSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия получает стандартное отклонение выборки для столбца.  
* ``stddevSampleIf(region == 'West', sales)``  
___
### <code>sum</code>
<code><b>sum(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Возвращает статистическую сумму числового столбца.  
* ``sum(col)``  
___
### <code>sumDistinct</code>
<code><b>sumDistinct(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
Возвращает статистическую сумму различных значений числового столбца.  
* ``sumDistinct(col)``  
___
### <code>sumDistinctIf</code>
<code><b>sumDistinctIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
Возвращает общую сумму числового столбца на основе критериев. Условие может основываться на любом столбце.  
* ``sumDistinctIf(state == 'CA' && commission < 10000, sales)``  
* ``sumDistinctIf(true, sales)``  
___
### <code>sumIf</code>
<code><b>sumIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
Возвращает общую сумму числового столбца на основе критериев. Условие может основываться на любом столбце.  
* ``sumIf(state == 'CA' && commission < 10000, sales)``  
* ``sumIf(true, sales)``  
___
### <code>variance</code>
<code><b>variance(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает дисперсию столбца.  
* ``variance(sales)``  
___
### <code>varianceIf</code>
<code><b>varianceIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия Получает дисперсию столбца.  
* ``varianceIf(region == 'West', sales)``  
___
### <code>variancePopulation</code>
<code><b>variancePopulation(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает дисперсию генеральной совокупности для столбца.  
* ``variancePopulation(sales)``  
___
### <code>variancePopulationIf</code>
<code><b>variancePopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия Получает дисперсию совокупности столбца.  
* ``variancePopulationIf(region == 'West', sales)``  
___
### <code>varianceSample</code>
<code><b>varianceSample(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
Возвращает несмещенную дисперсию столбца.  
* ``varianceSample(sales)``  
___
### <code>varianceSampleIf</code>
<code><b>varianceSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
В зависимости от критерия получает несмещенную дисперсию столбца.  
* ``varianceSampleIf(region == 'West', sales)``  

## <a name="array-functions"></a>Функции массива
Функции массивов выполняют преобразования в структурах данных, которые являются массивами. К ним относятся специальные ключевые слова для адресации элементов и индексов массива:

* ```#acc``` представляет значение, которое вы хотите включить в один выход при сокращении массива
* ```#index``` представляет текущий индекс массива вместе с номерами индексов массивов. ```#index2, #index3 ...```
* ```#item``` представляет значение текущего элемента в массиве

### <code>array</code>
<code><b>array([<i>&lt;value1&gt;</i> : any], ...) => array</b></code><br/><br/>
Создает массив элементов. Все элементы должны быть одного типа. Если элементы не указаны, по умолчанию используется пустой массив строк. Аналогичен оператору создания [].  
* ``array('Seattle', 'Washington')``
* ``['Seattle', 'Washington']``
* ``['Seattle', 'Washington'][1]``
* ``'Washington'``
___
### <code>filter</code>
<code><b>filter(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => array</b></code><br/><br/>
Фильтрует элементы массива, которые не соответствуют предоставленному предикату. Фильтр принимает ссылку на один элемент в функции предиката как #item.  
* ``filter([1, 2, 3, 4], #item > 2) -> [3, 4]``  
* ``filter(['a', 'b', 'c', 'd'], #item == 'a' || #item == 'b') -> ['a', 'b']``  
___
### <code>find</code>
<code><b>find(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => any</b></code><br/><br/>
Найти первый элемент из массива, соответствующего условию. Он принимает функцию фильтра, в которой можно обращаться к элементу в массиве #item. Для глубоко вложенных карт можно ссылаться на родительские карты, используя нотацию #item_n (#item_1, #item_2...).  
* ``find([10, 20, 30], #item > 10) -> 20``
* ``find(['azure', 'data', 'factory'], length(#item) > 4) -> 'azure'``
* ``find([
      @(
         name = 'Daniel',
         types = [
                   @(mood = 'jovial', behavior = 'terrific'),
                   @(mood = 'grumpy', behavior = 'bad')
                 ]
        ),
      @(
         name = 'Mark',
         types = [
                   @(mood = 'happy', behavior = 'awesome'),
                   @(mood = 'calm', behavior = 'reclusive')
                 ]
        )
      ],
      contains(#item.types, #item.mood=='happy')  /*Filter out the happy kid*/
    )``
* ``
     @(
           name = 'Mark',
           types = [
                     @(mood = 'happy', behavior = 'awesome'),
                     @(mood = 'calm', behavior = 'reclusive')
                   ]
      )
    ``  
___
### <code>map</code>
<code><b>map(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => any</b></код><BR/><BR/> сопоставляет каждый элемент массива с новым элементом, используя предоставленное выражение. Map принимает ссылку на один элемент в выражении эnction as #item.  
* ``map([1, 2, 3, 4], #item + 2) -> [3, 4, 5, 6]``  
* ``map(['a', 'b', 'c', 'd'], #item + '_processed') -> ['a_processed', 'b_processed', 'c_processed', 'd_processed']``  
___
### <code>mapIf</code>
<code><b>mapIf (<value1> : array, <value2> : binaryfunction, <value3>: binaryFunction) => any</b></код><BR/><BR/> условно сопоставляет массив с другим массивом с той же или меньшей длиной. Значения могут принадлежать любому типу данных, включая Структтипес. Он принимает функцию сопоставления, в которой можно обращаться к элементу в массиве как #item и текущий индекс как #index. Для глубоко вложенных карт можно ссылаться на скобки t maps using the ``#item_[n](#item_1 , #index_1...)`` notation.
*   ``mapIf([10, 20, 30], #item > 10, #item + 5) -> [25, 35]``
* ``mapIf(['icecream', 'cake', 'soda'], length(#item) > 4, upper(#item)) -> ['ICECREAM', 'CAKE']``
___
### <code>mapIndex</code>
<code><b>mapIndex(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => any</b></код><BR/><BR/> сопоставляет каждый элемент массива с новым элементом, используя предоставленное выражение. Функция map принимает ссылку на один элемент в функции выражения как #item и ссылку на элемент index as #index.  
* ``mapIndex([1, 2, 3, 4], #item + 2 + #index) -> [4, 6, 8, 10]``  
___
### <code>mapLoop</code>
<code><b>mapLoop(<value1> : integer, <value2> : unaryfunction) => any</b></код><BR/><BR/> перебирает от 1 до Length, чтобы создать массив этой длины. Он принимает функцию сопоставления, в которой можно обращаться к индексу в массиве #index. Для глубоко вложенных карт можно ссылаться на родительские карты с помощью #index_n (#index_1 #index_2...) notation.
*   ``mapLoop(3, #index * 10) -> [10, 20, 30]``
___
### <code>reduce</code>
<code><b>reduce(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : any, <i>&lt;value3&gt;</i> : binaryfunction, <i>&lt;value4&gt;</i> : unaryfunction) => any</b></Code><BR/><BR/> накапливает элементы в массиве. Функция Reduce ждет ссылку на совокупную единицу и один элемент в первой функции выражения как #acc и #item и ждет, что результирующее значение #resultся для использования во втором выражении.ession function.  
* ``toString(reduce(['1', '2', '3', '4'], '0', #acc + #item, #result)) -> '01234'``  
___
### <code>size</code>
<code><b>size(<i>&lt;value1&gt;</i> : any) => integer</b></Code><BR/><BR/> находит размерrray or map type  
* ``size(['element1', 'element2']) -> 2``
* ``size([1,2,3]) -> 3``
___
### <code>slice</code>
<code><b>slice(<i>&lt;array to slice&gt;</i> : array, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of items&gt;</i> : integral]) => array</b></Code><BR/><BR/> извлекает подмножество массива из расположения. Позиция отсчитывается от единицы. Если длина опущена, по умолчанию используется значение end of the string.  
* ``slice([10, 20, 30, 40], 1, 2) -> [10, 20]``  
* ``slice([10, 20, 30, 40], 2) -> [20, 30, 40]``  
* ``slice([10, 20, 30, 40], 2)[1] -> 20``  
* ``isNull(slice([10, 20, 30, 40], 2)[0]) -> true``  
* ``isNull(slice([10, 20, 30, 40], 2)[20]) -> true``  
* ``slice(['a', 'b', 'c', 'd'], 8) -> []``  
___
### <code>sort</code>
<code><b>sort(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => array</b></Code><BR/><BR/> сортирует массив с помощью предоставленной функции предиката. Функция Sort принимает ссылку на два последовательных элемента в функции выражения, как #item1 and #item2.  
* ``sort([4, 8, 2, 3], compare(#item1, #item2)) -> [2, 3, 4, 8]``  
* ``sort(['a3', 'b2', 'c1'], iif(right(#item1, 1) >= right(#item2, 1), 1, -1)) -> ['c1', 'b2', 'a3']``* ``
 @(
       name = 'Mark',
       types = [
                 @(mood = 'happy', behavior = 'awesome'),
                 @(mood = 'calm', behavior = 'reclusive')
               ]
  )
``  
___
### <code>map</code>
<code><b>map(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => any</b></code><br/><br/>
Maps each element of the array to a new element using the provided expression. Map expects a reference to one element in the expression function as #item.  
* ``map([1, 2, 3, 4], #item + 2) -> [3, 4, 5, 6]``  
* ``map(['a', 'b', 'c', 'd'], #item + '_processed') -> ['a_processed', 'b_processed', 'c_processed', 'd_processed']``  
___
### <code>mapIf</code>
<code><b>mapIf (<value1> : array, <value2> : binaryfunction, <value3>: binaryFunction) => any</b></code><br/><br/>
Conditionally maps an array to another array of same or smaller length. The values can be of any datatype including structTypes. It takes a mapping function where you can address the item in the array as #item and current index as #index. For deeply nested maps you can refer to the parent maps using the ``#item_[n](#item_1, #index_1...)`` notation.
*   ``mapIf([10, 20, 30], #item > 10, #item + 5) -> [25, 35]``
* ``mapIf(['icecream', 'cake', 'soda'], length(#item) > 4, upper(#item)) -> ['ICECREAM', 'CAKE']``
___
### <code>mapIndex</code>
<code><b>mapIndex(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => any</b></code><br/><br/>
Maps each element of the array to a new element using the provided expression. Map expects a reference to one element in the expression function as #item and a reference to the element index as #index.  
* ``mapIndex([1, 2, 3, 4], #item + 2 + #index) -> [4, 6, 8, 10]``  
___
### <code>mapLoop</code>
<code><b>mapLoop(<value1> : integer, <value2> : unaryfunction) => any</b></code><br/><br/>
Loops through from 1 to length to create an array of that length. It takes a mapping function where you can address the index in the array as #index. For deeply nested maps you can refer to the parent maps using the #index_n(#index_1, #index_2...) notation.
*   ``mapLoop(3, #index * 10) -> [10, 20, 30]``
___
### <code>reduce</code>
<code><b>reduce(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : any, <i>&lt;value3&gt;</i> : binaryfunction, <i>&lt;value4&gt;</i> : unaryfunction) => any</b></code><br/><br/>
Accumulates elements in an array. Reduce expects a reference to an accumulator and one element in the first expression function as #acc and #item and it expects the resulting value as #result to be used in the second expression function.  
* ``toString(reduce(['1', '2', '3', '4'], '0', #acc + #item, #result)) -> '01234'``  
___
### <code>size</code>
<code><b>size(<i>&lt;value1&gt;</i> : any) => integer</b></code><br/><br/>
Finds the size of an array or map type  
* ``size(['element1', 'element2']) -> 2``
* ``size([1,2,3]) -> 3``
___
### <code>slice</code>
<code><b>slice(<i>&lt;array to slice&gt;</i> : array, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of items&gt;</i> : integral]) => array</b></code><br/><br/>
Extracts a subset of an array from a position. Position is 1 based. If the length is omitted, it is defaulted to end of the string.  
* ``slice([10, 20, 30, 40], 1, 2) -> [10, 20]``  
* ``slice([10, 20, 30, 40], 2) -> [20, 30, 40]``  
* ``slice([10, 20, 30, 40], 2)[1] -> 20``  
* ``isNull(slice([10, 20, 30, 40], 2)[0]) -> true``  
* ``isNull(slice([10, 20, 30, 40], 2)[20]) -> true``  
* ``slice(['a', 'b', 'c', 'd'], 8) -> []``  
___
### <code>sort</code>
<code><b>sort(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => array</b></code><br/><br/>
Sorts the array using the provided predicate function. Sort expects a reference to two consecutive elements in the expression function as #item1 and #item2.  
* ``sort([4, 8, 2, 3], compare(#item1, #item2)) -> [2, 3, 4, 8]``  
* ``sort(['a3', 'b2', 'c1'], iif(right(#item1, 1) >= right(#item2, 1), 1, -1)) -> ['c1', 'b2', 'a3']``  

## <a name="cached-lookup-functions"></a>Кэшированные функции поиска
Следующие функции доступны только при использовании кэшированного поиска, если вы включили кэшированный приемник.
___
### <code>lookup</code>
<code><b>lookup(key, key2, ...) => complex[]</b></code><br/><br/>
Ищет первую строку из кэшированного приемника, используя указанные ключи, которые соответствуют ключам из кэшированного приемника.
* ``cacheSink#lookup(movieId)``  
___
### <code>mlookup</code>
<code><b>mlookup(key, key2, ...) => complex[]</b></code><br/><br/>
Ищет все совпадающие строки из кэшированного приемника с использованием указанных ключей, совпадающих с ключами из кэшированного приемника.
* ``cacheSink#mlookup(movieId)``  
___
### <code>output</code>
<code><b>output() => any</b></code><br/><br/>
Возвращает первую строку результатов приемника кэша * ``cacheSink#output()``  
___
### <code>outputs</code>
<code><b>output() => any</b></code><br/><br/>
Возвращает весь набор выходных строк результатов приемника кэша * ``cacheSink#outputs()``
___


## <a name="conversion-functions"></a>Функции преобразования

Функции преобразования используются для преобразования данных и проверки типов данных

### <code>isBoolean</code>
<code><b>isBoolean(<value1> : string) => boolean</b></code><br/><br/>
Проверяет, является ли строковое значение логическим значением в соответствии с правилами ``toBoolean()``
* ``isBoolean('true') -> true``
* ``isBoolean('no') -> true``
* ``isBoolean('microsoft') -> false``
___
### <code>isByte</code>
<code><b>isByte(<value1> : string) => boolean</b></code><br/><br/>
Проверяет, является ли строковое значение байтовым значением, заданным необязательным форматом в соответствии с правилами ``toByte()``
* ``isByte('123') -> true``
* ``isByte('chocolate') -> false``
___
### <code>isDate</code>
<code><b>isDate (<value1> : string, [<format>: string]) => boolean</b></code><br/><br/>
Проверяет, является ли входная строка даты датой, используя необязательный входной формат даты. Доступные форматы можно найти в SimpleDateFormat Java. Если входной формат даты не указан, используется формат по умолчанию ``yyyy-[M]M-[d]d`` . Допустимые форматы ``[ yyyy, yyyy-[M]M, yyyy-[M]M-[d]d, yyyy-[M]M-[d]dT* ]``
* ``isDate('2012-8-18') -> true``
* ``isDate('12/18--234234' -> 'MM/dd/yyyy') -> false``
___
### <code>isShort</code>
<code><b>isShort (<value1> : string, [<format>: string]) => boolean</b></code><br/><br/>
Проверки строкового значения — это короткое значение, заданное необязательным форматом в соответствии с правилами ``toShort()``
* ``isShort('123') -> true``
* ``isShort('$123' -> '$###') -> true``
* ``isShort('microsoft') -> false``
___
### <code>isInteger</code>
<code><b>isInteger (<value1> : string, [<format>: string]) => boolean</b></code><br/><br/>
Проверки строкового значения — это целочисленное значение, заданное необязательным форматом в соответствии с правилами ``toInteger()``
* ``isInteger('123') -> true``
* ``isInteger('$123' -> '$###') -> true``
* ``isInteger('microsoft') -> false``
___
### <code>isLong</code>
<code><b>isLong (<value1> : string, [<format>: string]) => boolean</b></code><br/><br/>
Проверки строкового значения — это длинное значение, заданное необязательным форматом в соответствии с правилами ``toLong()``
* ``isLong('123') -> true``
* ``isLong('$123' -> '$###') -> true``
* ``isLong('gunchus') -> false``
___
### <code>isFloat</code>
<code><b>isFloat (<value1> : string, [<format>: string]) => boolean</b></code><br/><br/>
Проверки строкового значения — это значение с плавающей запятой, заданное необязательным форматом в соответствии с правилами ``toFloat()``
* ``isFloat('123') -> true``
* ``isFloat('$123.45' -> '$###.00') -> true``
* ``isFloat('icecream') -> false``
___
### <code>isDouble</code>
<code><b>isDouble (<value1> : string, [<format>: string]) => boolean</b></code><br/><br/>
Проверки строкового значения — это значение типа Double, заданное необязательным форматом в соответствии с правилами ``toDouble()``
* ``isDouble('123') -> true``
* ``isDouble('$123.45' -> '$###.00') -> true``
* ``isDouble('icecream') -> false``
___
### <code>isDecimal</code>
<code><b>isDecimal (<value1> : string) => boolean</b></code><br/><br/>
Проверки строкового значения — это десятичное значение, заданное необязательным форматом в соответствии с правилами ``toDecimal()``
* ``isDecimal('123.45') -> true``
* ``isDecimal('12/12/2000') -> false``
___
### <code>isTimestamp</code>
<code><b>isTimestamp (<value1> : string, [<format>: string]) => boolean</b></code><br/><br/>
Проверяет, является ли входная строка даты меткой времени с использованием необязательного формата входной метки времени. Доступные форматы см. в Симпледатеформат Java. Если метка времени пропущена, используется шаблон по умолчанию ``yyyy-[M]M-[d]d hh:mm:ss[.f...]`` . Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". Отметка времени поддерживает точность до миллисекунд со значением 999. для доступных форматов можно использовать Симпледатеформат Java.
* ``isTimestamp('2016-12-31 00:12:00') -> true``
* ``isTimestamp('2016-12-31T00:12:00' -> 'yyyy-MM-dd\\'T\\'HH:mm:ss' -> 'PST') -> true``
* ``isTimestamp('2012-8222.18') -> false``
___
### <code>toBase64</code>
<code><b>toBase64(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Кодирует заданную строку в Base64.  
* ``toBase64('bojjus') -> 'Ym9qanVz'``  
___
### <code>toBinary</code>
<code><b>toBinary(<i>&lt;value1&gt;</i> : any) => binary</b></code><br/><br/>
Преобразует все числовые значения типа Date, timestamp и String в двоичное представление.  
* ``toBinary(3) -> [0x11]``  
___
### <code>toBoolean</code>
<code><b>toBoolean(<i>&lt;value1&gt;</i> : string) => boolean</b></code><br/><br/>
Преобразует значения ('t ', ' true ', ' y ', ' YES ', ' 1 ') в true и (' f ', ' false ', ' n ', ' No ', ' 0 ') в значение false и NULL для любого другого значения.  
* ``toBoolean('true') -> true``  
* ``toBoolean('n') -> false``  
* ``isNull(toBoolean('truthy')) -> true``  
___
### <code>toByte</code>
<code><b>toByte(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => byte</b></code><br/><br/>
Преобразует любое числовое или строковое значение в байтовый формат. Для преобразования может использоваться дополнительный десятичный формат Java.  
* ``toByte(123)``
* ``123``
* ``toByte(0xFF)``
* ``-1``
* ``toByte('123')``
* ``123``
___
### <code>toDate</code>
<code><b>toDate(<i>&lt;string&gt;</i> : any, [<i>&lt;date format&gt;</i> : string]) => date</b></code><br/><br/>
Преобразует строку даты ввода в дату, используя необязательный формат даты ввода. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. Если формат даты ввода не указан, формат по умолчанию — гггг-[М]М-[д]д. Допустимые форматы: [гггг, гггг-[M] M, гггг-[M] M-[d] d, гггг-[M] M-[d] dT *].  
* ``toDate('2012-8-18') -> toDate('2012-08-18')``  
* ``toDate('12/18/2012', 'MM/dd/yyyy') -> toDate('2012-12-18')``  
___
### <code>toDecimal</code>
<code><b>toDecimal(<i>&lt;value&gt;</i> : any, [<i>&lt;precision&gt;</i> : integral], [<i>&lt;scale&gt;</i> : integral], [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => decimal(10,0)</b></code><br/><br/>
Преобразует любое числовое значение или строку в десятичное значение. Если точность и масштаб не указаны, по умолчанию используется значение (10,2). Для преобразования можно использовать необязательный десятичный формат Java. Необязательный формат языкового стандарта в формате BCP47, например en-US, de, zh-CN.  
* ``toDecimal(123.45) -> 123.45``  
* ``toDecimal('123.45', 8, 4) -> 123.4500``  
* ``toDecimal('$123.45', 8, 4,'$###.00') -> 123.4500``  
* ``toDecimal('Ç123,45', 10, 2, 'Ç###,##', 'de') -> 123.45``  
___
### <code>toDouble</code>
<code><b>toDouble(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => double</b></code><br/><br/>
Преобразует любое числовое значение или строку в двойное значение. Для преобразования может использоваться дополнительный десятичный формат Java. Необязательный формат языкового стандарта в формате BCP47, например en-US, de, zh-CN.  
* ``toDouble(123.45) -> 123.45``  
* ``toDouble('123.45') -> 123.45``  
* ``toDouble('$123.45', '$###.00') -> 123.45``  
* ``toDouble('Ç123,45', 'Ç###,##', 'de') -> 123.45``  
___
### <code>toFloat</code>
<code><b>toFloat(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => float</b></code><br/><br/>
Преобразует любое числовое или строковое значение в плавающее. Для преобразования может использоваться дополнительный десятичный формат Java. Усекает любое двойное.  
* ``toFloat(123.45) -> 123.45f``  
* ``toFloat('123.45') -> 123.45f``  
* ``toFloat('$123.45', '$###.00') -> 123.45f``  
___
### <code>toInteger</code>
<code><b>toInteger(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => integer</b></code><br/><br/>
Преобразует любое числовое или строковое значение в целое. Для преобразования может использоваться дополнительный десятичный формат Java. Усекает любое длинное целое число типа float, Double.  
* ``toInteger(123) -> 123``  
* ``toInteger('123') -> 123``  
* ``toInteger('$123', '$###') -> 123``  
___
### <code>toLong</code>
<code><b>toLong(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => long</b></code><br/><br/>
Преобразует любое числовое или строковое значение в длинное. Для преобразования может использоваться дополнительный десятичный формат Java. Усекает любое число с плавающей запятой Double.  
* ``toLong(123) -> 123``  
* ``toLong('123') -> 123``  
* ``toLong('$123', '$###') -> 123``  
___
### <code>toShort</code>
<code><b>toShort(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => short</b></code><br/><br/>
Преобразует любое числовое или строковое значение в короткое. Для преобразования может использоваться дополнительный десятичный формат Java. Усекает любое целое число, Long, float и Double.  
* ``toShort(123) -> 123``  
* ``toShort('123') -> 123``  
* ``toShort('$123', '$###') -> 123``  
___
### <code>toString</code>
<code><b>toString(<i>&lt;value&gt;</i> : any, [<i>&lt;number format/date format&gt;</i> : string]) => string</b></code><br/><br/>
Преобразует примитивный тип данных в строку. Для чисел и даты можно указать формат. Если формат не задан, выбирается значение по умолчанию. Для чисел используется десятичный формат Java, Все возможные форматы даты см. в разделе Java Симпледатеформат. форматом по умолчанию является гггг-мм-дд.  
* ``toString(10) -> '10'``  
* ``toString('engineer') -> 'engineer'``  
* ``toString(123456.789, '##,###.##') -> '123,456.79'``  
* ``toString(123.78, '000000.000') -> '000123.780'``  
* ``toString(12345, '##0.#####E0') -> '12.345E3'``  
* ``toString(toDate('2018-12-31')) -> '2018-12-31'``  
* ``isNull(toString(toDate('2018-12-31', 'MM/dd/yy'))) -> true``  
* ``toString(4 == 20) -> 'false'``  
___
### <code>toTimestamp</code>
<code><b>toTimestamp(<i>&lt;string&gt;</i> : any, [<i>&lt;timestamp format&gt;</i> : string], [<i>&lt;time zone&gt;</i> : string]) => timestamp</b></code><br/><br/>
Преобразует строку в метку времени на основе указанного формата метки времени (необязательно). Если метка времени пропущена, используется шаблон по умолчанию yyyy-[M] M-[d] d чч: мм: СС [. f..]. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". Отметка времени поддерживает точность до миллисекунд со значением 999. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``toTimestamp('2016-12-31 00:12:00') -> toTimestamp('2016-12-31 00:12:00')``  
* ``toTimestamp('2016-12-31T00:12:00', 'yyyy-MM-dd\'T\'HH:mm:ss', 'PST') -> toTimestamp('2016-12-31 00:12:00')``  
* ``toTimestamp('12/31/2016T00:12:00', 'MM/dd/yyyy\'T\'HH:mm:ss') -> toTimestamp('2016-12-31 00:12:00')``  
* ``millisecond(toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS')) -> 871``  
___
### <code>toUTC</code>
<code><b>toUTC(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => timestamp</b></code><br/><br/>
Преобразует метку времени в формат UTC. Вы можете передать дополнительный часовой пояс в формате GMT, PST, UTC, "Острова Кайман". По умолчанию используется текущий часовой пояс. `SimpleDateFormat`Доступные форматы см. в разделе класс Java. https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``toUTC(currentTimestamp()) == toTimestamp('2050-12-12 19:18:12') -> false``  
* ``toUTC(currentTimestamp(), 'Asia/Seoul') != toTimestamp('2050-12-12 19:18:12') -> true``  

## <a name="metafunctions"></a>метафунктионс

Метафунктионс в основном функция для метаданных в потоке данных

### <code>byItem</code>
<code><b>byItem(<i>&lt;parent column&gt;</i> : any, <i>&lt;column name&gt;</i> : string) => any</b></code><br/><br/>
Найти вложенный элемент в структуре или массиве структуры при наличии нескольких совпадений возвращается первое совпадение. Если совпадений нет, возвращается значение NULL. Возвращаемое значение должно быть типом, преобразованным одним из действий преобразования типов (? Дата,? строка...).  Имена столбцов, известные во время разработки, должны быть адресованы только по имени. Вычисленные входные данные не поддерживаются, но можно использовать подстановки параметров * ``byItem( byName('customer'), 'orderItems') ? (itemName as string, itemQty as integer)``
* ````
* ``byItem( byItem( byName('customer'), 'orderItems'), 'itemName') ? string``
* ````
___
### <code>byOrigin</code>
<code><b>byOrigin(<i>&lt;column name&gt;</i> : string, [<i>&lt;origin stream name&gt;</i> : string]) => any</b></code><br/><br/>
Выбирает значение столбца по имени в исходном потоке. Вторым аргументом является имя исходного потока. Если найдено несколько совпадений, то возвращается первое совпадение. Если совпадений нет, возвращается значение NULL. Возвращаемое значение должно быть преобразовано с помощью одной из функций преобразования типа (TO_DATE, TO_STRING...). Имена столбцов, известные во время разработки, должны быть адресованы только по имени. Вычисленные входные данные не поддерживаются, но вы можете использовать подстановки параметров.  
* ``toString(byOrigin('ancestor', 'ancestorStream'))``
___
### <code>byOrigins</code>
<code><b>byOrigins(<i>&lt;column names&gt;</i> : array, [<i>&lt;origin stream name&gt;</i> : string]) => any</b></code><br/><br/>
Выбирает массив столбцов по имени в потоке. Второй аргумент — это поток, из которого он был получен. Если найдено несколько совпадений, то возвращается первое совпадение. Если совпадений нет, возвращается значение NULL. Возвращаемое значение должно быть преобразовано с помощью одной из функций преобразования типов (TO_DATE, TO_STRING...) Имена столбцов, известные во время разработки, должны быть адресованы только по имени. Вычисленные входные данные не поддерживаются, но вы можете использовать подстановки параметров.
* ``toString(byOrigins(['ancestor1', 'ancestor2'], 'ancestorStream'))``
___
### <code>byName</code>
<code><b>byName(<i>&lt;column name&gt;</i> : string, [<i>&lt;stream name&gt;</i> : string]) => any</b></code><br/><br/>
Выбирает значение столбца по имени в потоке. В качестве второго аргумента можно передать необязательное имя потока. Если найдено несколько совпадений, то возвращается первое совпадение. Если совпадений нет, возвращается значение NULL. Возвращаемое значение должно быть преобразовано типом одной из функций преобразования типов (TO_DATE, TO_STRING …).  Имена столбцов, известные во время проектирования, должны определяться только по имени. Вычисленные входные данные не поддерживаются, но вы можете использовать подстановки параметров.  
* ``toString(byName('parent'))``  
* ``toLong(byName('income'))``  
* ``toBoolean(byName('foster'))``  
* ``toLong(byName($debtCol))``  
* ``toString(byName('Bogus Column'))``  
* ``toString(byName('Bogus Column', 'DeriveStream'))``  
___
### <code>byNames</code>
<code><b>byNames(<i>&lt;column names&gt;</i> : array, [<i>&lt;stream name&gt;</i> : string]) => any</b></code><br/><br/>
Выберите массив столбцов по имени в потоке. В качестве второго аргумента можно передать необязательное имя потока. Если найдено несколько совпадений, то возвращается первое совпадение. Если для столбца нет совпадений, все выходные данные имеют значение NULL. Возвращаемое значение требует функции преобразования типов (toDate, toString, …).  Имена столбцов, известные во время проектирования, должны определяться только по имени. Вычисленные входные данные не поддерживаются, но вы можете использовать подстановки параметров.
* ``toString(byNames(['parent', 'child']))``
* ``byNames(['parent']) ? string``
* ``toLong(byNames(['income']))``
* ``byNames(['income']) ? long``
* ``toBoolean(byNames(['foster']))``
* ``toLong(byNames($debtCols))``
* ``toString(byNames(['a Column']))``
* ``toString(byNames(['a Column'], 'DeriveStream'))``
* ``byNames(['orderItem']) ? (itemName as string, itemQty as integer)``
___
### <code>byPosition</code>
<code><b>byPosition(<i>&lt;position&gt;</i> : integer) => any</b></code><br/><br/>
Выбирает значение столбца по его относительному расположению (начиная с 1) в потоке. Возвращает значение NULL, если расположение находится вне допустимого диапазона. Возвращаемое значение должно быть преобразовано с помощью одной из функций преобразования типов (TO_DATE, TO_STRING...) Вычисленные входные данные не поддерживаются, но можно использовать подстановки параметров.  
* ``toString(byPosition(1))``  
* ``toDecimal(byPosition(2), 10, 2)``  
* ``toBoolean(byName(4))``  
* ``toString(byName($colName))``  
* ``toString(byPosition(1234))``  
___
### <code>hex</code>
<code><b>hex(<value1>: binary) => string</b></code><br/><br/>
Возвращает шестнадцатеричное строковое представление двоичного значения * ``hex(toBinary([toByte(0x1f), toByte(0xad), toByte(0xbe)])) -> '1fadbe'``
___
### <code>unhex</code>
<code><b>unhex(<value1>: string) => binary</b></code><br/><br/>
Унхексес двоичное значение из его строкового представления. Это можно использовать в сочетании с SHA2, MD5 для преобразования строки в двоичное представление *   ``unhex('1fadbe') -> toBinary([toByte(0x1f), toByte(0xad), toByte(0xbe)])``
*   ``unhex(md5(5, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4'))) -> toBinary([toByte(0x4c),toByte(0xe8),toByte(0xa8),toByte(0x80),toByte(0xbd),toByte(0x62),toByte(0x1a),toByte(0x1f),toByte(0xfa),toByte(0xd0),toByte(0xbc),toByte(0xa9),toByte(0x05),toByte(0xe1),toByte(0xbc),toByte(0x5a)])``

## <a name="window-functions"></a>Оконные функции
Следующие функции доступны только в преобразованиях окна.
___
### <code>cumeDist</code>
<code><b>cumeDist() => integer</b></code><br/><br/>
Функция CumeDist вычисляет позицию значения относительно всех значений в разделе. Результатом является количество строк, предшествующих или равных текущей строке в упорядоченном наборе раздела, деленное на общее количество строк в разделе окна. Любые значения времени в упорядоченном наборе будут вычисляться в той же позиции.  
* ``cumeDist()``  
___
### <code>denseRank</code>
<code><b>denseRank() => integer</b></code><br/><br/>
Вычисляет ранг значения в группе значений, указанных в предложении order by окна. Результатом является единица плюс количество строк, предшествующих или равных текущей строке в упорядоченном наборе раздела. Значения не будут создавать промежутки в последовательности. Сжатый ранг работает даже в том случае, если данные не отсортированы и выполняется поиск изменений в значениях.  
* ``denseRank()``  
___
### <code>lag</code>
<code><b>lag(<i>&lt;value&gt;</i> : any, [<i>&lt;number of rows to look before&gt;</i> : number], [<i>&lt;default value&gt;</i> : any]) => any</b></code><br/><br/>
Возвращает значение первого параметра, вычислившего n строк перед текущей строкой. Второй параметр — это количество строк для ретроспективного анализа, значение по умолчанию — 1. Если число строк не превышает допустимого значения, возвращается значение null, если не указано значение по умолчанию.  
* ``lag(amount, 2)``  
* ``lag(amount, 2000, 100)``  
___
### <code>lead</code>
<code><b>lead(<i>&lt;value&gt;</i> : any, [<i>&lt;number of rows to look after&gt;</i> : number], [<i>&lt;default value&gt;</i> : any]) => any</b></code><br/><br/>
Возвращает значение первого параметра, вычислившего n строк после текущей строки. Второй параметр — это количество строк, идущих после текущей, значение по умолчанию — 1. Если число строк не превышает допустимого значения, возвращается значение null, если не указано значение по умолчанию.  
* ``lead(amount, 2)``  
* ``lead(amount, 2000, 100)``  
___
### <code>nTile</code>
<code><b>nTile([<i>&lt;value1&gt;</i> : integer]) => integer</b></code><br/><br/>
```NTile```Функция делит строки для каждой секции окна на `n` контейнеры в диапазоне от 1 до максимума `n` . Значения сегментов будут отличаться максимум на 1. Если количество строк в разделе не делится поровну на количество сегментов, то остальные значения распределяются по одному на сегмент, начиная с первого. ```NTile```Функция полезна для вычисления ```tertiles``` , квартилей, децили и других общих сводных статистических данных. Функция вычисляет две переменные во время инициализации: к размеру обычного сегмента будет добавлена одна дополнительная строка. Обе переменные зависят от размера текущего раздела. В процессе вычисления функция отслеживает текущее число строк, номер текущего сегмента и номер строки, на которой изменяется сегмент (bucketThreshold). Когда текущий номер строки достигает порога сегмента, значение сегмента увеличивается на единицу, а порог увеличивается на размер сегмента (плюс единица, если текущий сегмент заполняется).  
* ``nTile()``  
* ``nTile(numOfBuckets)``  
___
### <code>rank</code>
<code><b>rank() => integer</b></code><br/><br/>
Вычисляет ранг значения в группе значений, указанных в предложении order by окна. Результатом является единица плюс количество строк, предшествующих или равных текущей строке в упорядоченном наборе раздела. Значения будут создавать промежутки в последовательности. Ранжирование работает даже в том случае, если данные не отсортированы и выполняется поиск изменений в значениях.  
* ``rank()``  
___
### <code>rowNumber</code>
<code><b>rowNumber() => integer</b></code><br/><br/>
Назначает последовательную нумерацию строк в окне, начиная с 1.  
* ``rowNumber()``  

## <a name="next-steps"></a>Дальнейшие действия

[Создание выражений в потоке данных для сопоставления](concepts-data-flow-expression-builder.md)

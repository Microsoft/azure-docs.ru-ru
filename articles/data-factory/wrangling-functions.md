---
title: Функции структурирование данных в фабрике данных Azure
description: Обзор доступных функций структурирование данных в фабрике данных Azure
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/19/2021
ms.openlocfilehash: 659f6527d43e1b45a11fddf774050ca6d42bfe12
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98896669"
---
# <a name="transformation-functions-in-power-query-for-data-wrangling"></a>Функции преобразования в Power Query структурирование данных

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Структурирование данных в фабрике данных Azure позволяет выполнять динамическую подготовку данных и структурирование в масштабе облака, преобразуя Power Query ```M``` сценарии в сценарий потока данных. ADF интегрируется с [Power Query Online](/powerquery-m/power-query-m-reference) и делает Power Query ```M``` функции доступными для структурирование данных через выполнение Spark с помощью инфраструктуры Spark потока данных. 

> [!NOTE]
> Power Query в ADF в настоящее время доступных в общедоступной предварительной версии

В настоящее время не все функции Power Query M поддерживаются для структурирование данных, несмотря на то, что они доступны во время разработки. При создании гибридных веб-функций вам будет предложено следующее сообщение об ошибке, если функция не поддерживается:

`UserQuery : Expression.Error: The transformation logic is not supported as it requires dynamic access to rows of data, which cannot be scaled out.`

Ниже приведен список поддерживаемых функций Power Query M.

## <a name="column-management"></a>Управление столбцами

* Выбор: [Table. SelectColumns](/powerquery-m/table-selectcolumns)
* Удаление: [Table. ремовеколумнс](/powerquery-m/table-removecolumns)
* Переименование: [Table. RenameColumns](/powerquery-m/table-renamecolumns), [таблица. префиксколумнс](/powerquery-m/table-prefixcolumns), [Table. трансформколумннамес](/powerquery-m/table-transformcolumnnames)
* Изменение порядка: [Table. реордерколумнс](/powerquery-m/table-reordercolumns)

## <a name="row-filtering"></a>Фильтрация строк

Функция M [Table. селектровс](/powerquery-m/table-selectrows) используется для фильтрации по следующим условиям.

* Равенство и неравенство
* Сравнения чисел, текста и дат (но не даты и времени)
* Числовые данные, например [Number. четный](/powerquery-m/number-iseven), / [четные](/powerquery-m/number-iseven)
* Текстовое вложение с использованием [Text. Contains](/powerquery-m/text-contains), [Text. StartsWith](/powerquery-m/text-startswith)или [Text. EndsWith](/powerquery-m/text-endswith)
* Диапазоны дат, включая все функции "Исин' [Date](/powerquery-m/date-functions)". 
* Сочетания этих операторов с операторами AND, OR и not

## <a name="adding-and-transforming-columns"></a>Добавление и преобразование столбцов

Следующие функции M добавляют или преобразуют столбцы: [таблица. addColumn](/powerquery-m/table-addcolumn), [таблица. трансформколумнс](/powerquery-m/table-transformcolumns), [таблица. реплацевалуе](/powerquery-m/table-replacevalue), [Table. дупликатеколумн](/powerquery-m/table-duplicatecolumn). Ниже приведены поддерживаемые функции преобразования.

* Числовые арифметические операции
* Сцепление текста
* Date Андтиме арифметика (арифметические операторы, [Date. AddDays](/powerquery-m/date-adddays), [Date. аддмонсс](/powerquery-m/date-addmonths), [Date. аддкуартерс](/powerquery-m/date-addquarters), [Date. аддвикс](/powerquery-m/date-addweeks), [Date. аддеарс](/powerquery-m/date-addyears))
* Длительности можно использовать для арифметических операций с датами и временем, но они должны быть преобразованы в другой тип перед записью в приемник (арифметические операторы, [#duration](/powerquery-m/sharpduration), [продолжительность. Days](/powerquery-m/duration-days), [Duration. Hours](/powerquery-m/duration-hours), [Duration.](/powerquery-m/duration-minutes) [reтоталхаурс](/powerquery-m/duration-totalhours) [, Duration](/powerquery-m/duration-totalminutes). [Seconds](/powerquery-m/duration-seconds) [, Duration.](/powerquery-m/duration-totaldays) [тоталсекондс](/powerquery-m/duration-totalseconds))    
* Большинство стандартных, научных и тригонометрических числовых функций (всех функций в рамках [операций](/powerquery-m/number-functions#operations), [округления](/powerquery-m/number-functions#rounding)и [тригонометрических](/powerquery-m/number-functions#trigonometry) , *за исключением* Number. факториал, Number. перестановок и числа. сочетаний)
* Замена (замещаемый[. реплацетекст](/powerquery-m/replacer-replacetext), [Place. реплацевалуе](/powerquery-m/replacer-replacevalue), [Text. Replace](/powerquery-m/text-replace), [Text. Replace](/powerquery-m/text-remove))
* Извлечение позиционированного текста ([Text. поситионоф](/powerquery-m/text-positionof), [Text. length](/powerquery-m/text-length), [Text. Start](/powerquery-m/text-start), [Text. end](/powerquery-m/text-end), [Text. Ближний](/powerquery-m/text-middle), [Text. реплацеранже](/powerquery-m/text-replacerange), [Text. ремоверанже](/powerquery-m/text-removerange))
* Основное форматирование текста ([Text. Lower](/powerquery-m/text-lower), [Text. Upper](/powerquery-m/text-upper), [Text. Trim](/powerquery-m/text-trim) / [Начало](/powerquery-m/text-trimstart) / [](/powerquery-m/text-trimend), [Text. падстарт](/powerquery-m/text-padstart) / [End](/powerquery-m/text-padend), [Text. Reverse](/powerquery-m/text-reverse))
* Функции даты и времени ([Дата. день](/powerquery-m/date-day), [Дата. месяц](/powerquery-m/date-month), [Дата. год,](/powerquery-m/date-year) [время. час](/powerquery-m/time-hour), [время. минута](/powerquery-m/time-minute), [время. секунд](/powerquery-m/time-second), [Дата. DayOfWeek](/powerquery-m/date-dayofweek), [Date. DayOfYear](/powerquery-m/date-dayofyear), [Date. дайсинмонс](/powerquery-m/date-daysinmonth))
* Выражения if (но ветви должны иметь совпадающие типы)
* Фильтры строк в качестве логического столбца
* Константы числа, текста, логических, дат и даты и времени

<a name="mergingjoining-tables"></a>Слияние и объединение таблиц
----------------------
* Power Query создаст вложенное соединение (Table. Нестеджоин; пользователи могут также вручную написать [таблицу. адджоинколумн](/powerquery-m/table-addjoincolumn)).
    Пользователи должны развернуть вложенный столбец JOIN в невложенное соединение (Table. Експандтаблеколумн, не поддерживается в любом другом контексте).
* Таблица функций M   [. Join](/powerquery-m/table-join) может быть написана напрямую, чтобы избежать необходимости в дополнительном шаге расширения, но пользователь должен убедиться в отсутствии повторяющихся имен столбцов между соединяемыми таблицами.
* Поддерживаемые типы соединений:   [inner](/powerquery-m/joinkind-inner),   [LeftOuter](/powerquery-m/joinkind-leftouter),   [RightOuter](/powerquery-m/joinkind-rightouter),   [FullOuter](/powerquery-m/joinkind-fullouter)
* Оба   [значения. Equals](/powerquery-m/value-equals) и   [value. нуллабликуалс](/powerquery-m/value-nullableequals) поддерживаются как компараторы равенства ключей

## <a name="group-by"></a>Group by

Используйте [Table. Group](/powerquery-m/table-group) для статистической обработки значений.
* Должен использоваться с агрегатной функцией
* Поддерживаемые агрегатные функции:   [List. Sum](/powerquery-m/list-sum),   [List. Count](/powerquery-m/list-count),   [List. СРЗНАЧ](/powerquery-m/list-average),   [List. min](/powerquery-m/list-min),   [List. Max](/powerquery-m/list-max),   [List. стандартное отклонение](/powerquery-m/list-standarddeviation),   [List. First](/powerquery-m/list-first),   [List. Last](/powerquery-m/list-last)

## <a name="sorting"></a>Сортировка

Используйте [таблицу. Sort](/powerquery-m/table-sort) для сортировки значений.

## <a name="reducing-rows"></a>Сокращение количества строк

Сохраняются и удаляются первые, сохраняют диапазоны (соответствующие функции M, только вспомогательные счетчики, not Conditions: [Table. FirstN](/powerquery-m/table-firstn), [таблица. Skip](/powerquery-m/table-skip), [таблица. ремовефирстн](/powerquery-m/table-removefirstn), [таблица. диапазон](/powerquery-m/table-range), [таблица. Минн](/powerquery-m/table-minn), [Table. максн](/powerquery-m/table-maxn))

## <a name="known-unsupported-functions"></a>Известные неподдерживаемые функции

| Компонент | Состояние |
| -- | -- |
| Table.PromoteHeaders | Не поддерживается. Тот же результат можно получить, установив в наборе данных "первая строка как заголовок". |
| Table.CombineColumns | Это распространенный сценарий, который не поддерживается напрямую, но может быть достигнут путем добавления нового столбца, объединяющего два заданных столбца.  Например, Table. AddColumn (Ремовимаилколумн, "Name", each [FirstName] & "" & [LastName]) |
| Table.TransformColumnTypes | В большинстве случаев это поддерживается. Следующие сценарии не поддерживаются: преобразование строки в тип валюты, преобразование строки в тип времени, преобразование строки в процентный тип. |
| Table.NestedJoin | При простом соединении будет возникать ошибка проверки. Столбцы должны быть развернуты, чтобы они работали. |
| Table.Distinct | Удаление повторяющихся строк не поддерживается. |
| Table.RemoveLastN | Удаление нижних строк не поддерживается. |
| Table.RowCount | Не поддерживается, но может быть достигнуто путем добавления пользовательского столбца, содержащего значение 1, а затем статистической обработки этого столбца с помощью List. Sum. Таблица. Группа поддерживается. | 
| Обработка ошибок на уровне строк | Обработка ошибок на уровне строк в настоящее время не поддерживается. Например, чтобы отфильтровать нечисловые значения из столбца, одним из подходов будет преобразование текстового столбца в число. Каждая ячейка, для которой не выполняется преобразование, будет находиться в состоянии ошибки и должна быть отфильтрована. Этот сценарий невозможен в масштабировании M. |
| Table.Transpose | Не поддерживается |
| Table.Pivot | Не поддерживается |

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [создать Power Query данных структурирование в ADF](wrangling-tutorial.md).

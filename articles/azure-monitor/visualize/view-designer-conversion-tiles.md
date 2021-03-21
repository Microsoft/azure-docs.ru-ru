---
title: Azure Monitor конструктора представлений для преобразования плиток в книгах
description: Сведения для преобразования плиток в книги при переходе из представлений в Azure Monitor.
author: austonli
ms.author: aul
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: 9b8d1840c0b1cea4c03312d38ed892fde94faa4c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102043343"
---
# <a name="azure-monitor-view-designer-tile-conversions"></a>Преобразования плиток конструктора представлений Azure Monitor
[Конструктор представлений](view-designer.md) — это функция Azure Monitor, которая позволяет создавать пользовательские представления, помогающие визуализировать данные в рабочей области log Analytics, с диаграммами, списками и временными шкалами. Они выводятся и заменяются книгами, которые предоставляют дополнительные функциональные возможности. В этой статье приводятся сведения о преобразовании различных плиток в книги.

## <a name="donut--list-tile"></a>Плитка со списком & кольца

![Список кольца](media/view-designer-conversion-tiles/donut-list.png)

Повторное создание элемента кольца & списка в книгах включает две отдельные визуализации. Для части кольца существует два варианта.
Для начала нажмите кнопку **Добавить запрос** и вставьте исходный запрос из конструктора представлений в ячейку.

**Вариант 1.** Выбор **круговой диаграммы** из раскрывающегося списка **визуализации** : ![ меню визуализации круговой диаграммы](media/view-designer-conversion-tiles/pie-chart.png)

**Вариант 2.** Выберите **задать по запросу** в раскрывающемся списке **визуализация** и добавьте `| render piechart` в запрос:

 ![Меню визуализации](media/view-designer-conversion-tiles/set-by-query.png)

**Пример**

Исходный запрос
```KQL
search * 
| summarize AggregatedValue = count() by Type 
| order by AggregatedValue desc
```

Обновленный запрос
```KQL
search * 
| summarize AggregatedValue = count() by Type 
| order by AggregatedValue desc 
| render piechart
```

Сведения о создании списка и включении спарклайнов см. в статье [Общие задачи](view-designer-conversion-tasks.md).

Ниже приведен пример того, как плитка & списка кольца может быть повторно интерпретирована в книгах:

![Книги в виде кольцевых списков](media/view-designer-conversion-tiles/donut-workbooks.png)

## <a name="line-chart--list-tile"></a>Плитка графика & списка
![Список графиков](media/view-designer-conversion-tiles/line-list.png) 

Чтобы повторно создать график, обновите запрос следующим образом:

Исходный запрос
```KQL
search * 
| summarize AggregatedValue = count() by Type
```

Обновленный запрос
```KQL
search * 
| make-series Count = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type
```

Существует два варианта визуализации графика

**Вариант 1.** Выберите **график** из раскрывающегося списка **визуализация** :
 
 ![Меню графика](media/view-designer-conversion-tiles/line-visualization.png)

**Вариант 2.** Выберите **задать по запросу** в раскрывающемся списке **визуализация** и добавьте `| render linechart` в запрос:

 ![Меню визуализации](media/view-designer-conversion-tiles/set-by-query.png)

**Пример**

```KQL
search * 
| make-series Count = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type 
| render linechart_
```

Сведения о создании списка и включении спарклайнов см. в статье [Общие задачи](view-designer-conversion-tasks.md).

Ниже приведен пример повторной интерпретации элемента графика & списка в книгах:

![График списка книг](media/view-designer-conversion-tiles/line-workbooks.png)

## <a name="number--list-tile"></a>Плитка числа & списка

 ![Список плиток](media/view-designer-conversion-tiles/tile-list-example.png)

Для плитки число обновите запрос следующим образом:

Исходный запрос
```KQL
search * 
| summarize AggregatedValue = count() by Computer | count
```

Обновленный запрос
```KQL
search *
| summarize AggregatedValue = count() by Computer 
| summarize Count = count()
```

Измените раскрывающийся список визуализации на **плитки** , а затем выберите **Параметры плитки**.
 ![Визуализация плитки](media/view-designer-conversion-tiles/tile-visualization.png)

Оставьте **заголовок** раздела пустым и выберите **слева**. Измените значение параметра **использовать столбец:** для параметра **число**, а для модуля **подготовки столбцов** — **большое число**:

![Параметры плитки](media/view-designer-conversion-tiles/tile-settings.png)

 
Сведения о создании списка и включении спарклайнов см. в статье [Общие задачи](view-designer-conversion-tasks.md).

Ниже приведен пример того, как в книгах может быть повторно интерпретирована плитка числа & списка:

![Книги списка чисел](media/view-designer-conversion-tiles/number-workbooks.png)

## <a name="timeline--list"></a>Временная шкала и список

 ![Список временной шкалы](media/view-designer-conversion-tiles/time-list.png)

Для временной шкалы обновите запрос следующим образом:

Исходный запрос
```KQL
search * 
| sort by TimeGenerated desc
```

Обновленный запрос
```KQL
search * 
| summarize Count = count() by Computer, bin(TimeGenerated,{TimeRange:grain})
```

Существует два варианта визуализации запроса в виде линейчатой диаграммы.

**Вариант 1.** Выберите **линейчатую диаграмму** из раскрывающегося списка **визуализация** : ![ визуализация барчарт](media/view-designer-conversion-tiles/bar-visualization.png)
 
**Вариант 2.** Выберите **задать по запросу** в раскрывающемся списке **визуализация** и добавьте `| render barchart` в запрос:

 ![Меню визуализации](media/view-designer-conversion-tiles/set-by-query.png)

 
Сведения о создании списка и включении спарклайнов см. в статье [Общие задачи](view-designer-conversion-tasks.md).

Ниже приведен пример того, как в книгах может быть повторно интерпретирована плитка & списка временной шкалы:

![Книги из списка временной шкалы](media/view-designer-conversion-tiles/time-workbooks.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор конструктора представлений для перехода к книгам](view-designer-conversion-overview.md)

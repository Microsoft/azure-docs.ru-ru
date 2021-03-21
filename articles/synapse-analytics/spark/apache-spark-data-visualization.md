---
title: Визуализации
description: Использование записных книжек Azure синапсе для визуализации данных
author: midesa
ms.author: midesa
ms.reviewer: euang
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 09/13/2020
ms.openlocfilehash: f11693b34048b11c02668e086561b9a6521a5213
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98121531"
---
# <a name="visualize-data"></a>Визуализируйте данные
Azure синапсе — это интегрированная служба аналитики, которая ускоряет анализ данных в разных хранилищах и системах анализа больших данных. Визуализация данных — ключевой компонент, позволяющий получить представление о данных. Это упрощает понимание больших и небольших данных. Это также упрощает обнаружение шаблонов, тенденций и выбросов в группах данных. 

При использовании Apache Spark в Azure синапсе Analytics доступны различные встроенные параметры, которые помогут вам визуализировать данные, в том числе параметры Синапсеной записной книжки, доступ к популярным библиотекам с открытым исходным кодом и интеграцию с синапсе SQL и Power BI.

## <a name="notebook-chart-options"></a>Параметры диаграммы записной книжки
При использовании записной книжки Azure синапсе можно превратить представление табличных результатов в настроенную диаграмму с помощью параметров диаграммы. Здесь можно визуализировать данные без написания кода. 

### <a name="displaydf-function"></a>функция дисплея (DF)
```display```Функция позволяет ВКЛЮЧАТЬ SQL запросы и Apache Spark кадры данных и RDD в визуализации с богатыми данными. ```display```Функцию можно использовать для кадров данных или RDD, созданных в PySpark, Scala, Java и .NET.

Для доступа к параметрам диаграммы выполните следующие действия.
1. Выходные данные ```%%sql``` команд Magic по умолчанию отображаются в представлении отображаемой таблицы. Можно также вызвать ```display(df)``` функцию для таблиц данных Spark или отказоустойчивого распределенного набора данных (RDD), чтобы получить представление отображаемой таблицы. 
   
2. После отображения табличного представления переключитесь в представление диаграммы.
   ![Встроенные диаграммы](./media/apache-spark-development-using-notebooks/synapse-built-in-charts.png#lightbox)

3. Теперь можно настроить визуализацию, указав следующие значения:
   | Конфигурация | Описание |
   |--|--| 
   | Тип диаграммы | ```display```Функция поддерживает широкий спектр типов диаграмм, включая линейчатые диаграммы, точечные, графикы и многое другое. |
   | Ключ | Укажите диапазон значений для оси x|
   | Значение | Укажите диапазон значений для значений по оси y |
   | Группа «Ряды» | Используется для определения групп для статистической обработки | 
   | Агрегирование | Метод для агрегирования данных в визуализации| 
   
   
    > [!NOTE]
    > По умолчанию ```display(df)``` функция принимает только первые 1000 строк данных для отрисовки диаграмм. Проверьте **агрегирование всех результатов** и нажмите кнопку **Применить** . вы примените создание диаграммы из всего набора данных. Задание Spark будет запущено при изменении параметра диаграммы. Обратите внимание, что выполнение вычислений может занять несколько минут, и диаграмма будет отображена.
   
4. После этого вы сможете просматривать окончательную визуализацию и взаимодействовать с ней!

### <a name="displaydf-statistic-details"></a>Отображение данных статистики (DF)
Можно использовать <code>display(df, summary = true)</code> для проверки сводки статистики по заданному Apache Spark кадре данных, включающего в себя имя столбца, тип столбца, уникальные значения и отсутствующие значения для каждого столбца. Можно также выбрать конкретный столбец, чтобы увидеть его минимальное значение, максимальное значение, среднее значение и стандартное отклонение.
    ![Встроенные диаграммы — сводка](./media/apache-spark-development-using-notebooks/synapse-built-in-charts-summary.png#lightbox)
   
### <a name="displayhtmldf-option"></a>Дисплайхтмл (DF), параметр
Записные книжки Azure синапсе Analytics поддерживают HTML-графику с помощью ```displayHTML``` функции.

На следующем рисунке показан пример создания визуализаций с помощью [D3.js](https://d3js.org/).

   ![D3-JS-пример](./media/apache-spark-data-viz/d3-boxplot.png#lightbox)

Выполните следующий код, чтобы создать визуализацию выше.

```python
displayHTML("""<!DOCTYPE html>
<meta charset="utf-8">

<!-- Load d3.js -->
<script src="https://d3js.org/d3.v4.js"></script>

<!-- Create a div where the graph will take place -->
<div id="my_dataviz"></div>
<script>

// set the dimensions and margins of the graph
var margin = {top: 10, right: 30, bottom: 30, left: 40},
  width = 400 - margin.left - margin.right,
  height = 400 - margin.top - margin.bottom;

// append the svg object to the body of the page
var svg = d3.select("#my_dataviz")
.append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
.append("g")
  .attr("transform",
        "translate(" + margin.left + "," + margin.top + ")");

// Create Data
var data = [12,19,11,13,12,22,13,4,15,16,18,19,20,12,11,9]

// Compute summary statistics used for the box:
var data_sorted = data.sort(d3.ascending)
var q1 = d3.quantile(data_sorted, .25)
var median = d3.quantile(data_sorted, .5)
var q3 = d3.quantile(data_sorted, .75)
var interQuantileRange = q3 - q1
var min = q1 - 1.5 * interQuantileRange
var max = q1 + 1.5 * interQuantileRange

// Show the Y scale
var y = d3.scaleLinear()
  .domain([0,24])
  .range([height, 0]);
svg.call(d3.axisLeft(y))

// a few features for the box
var center = 200
var width = 100

// Show the main vertical line
svg
.append("line")
  .attr("x1", center)
  .attr("x2", center)
  .attr("y1", y(min) )
  .attr("y2", y(max) )
  .attr("stroke", "black")

// Show the box
svg
.append("rect")
  .attr("x", center - width/2)
  .attr("y", y(q3) )
  .attr("height", (y(q1)-y(q3)) )
  .attr("width", width )
  .attr("stroke", "black")
  .style("fill", "#69b3a2")

// show median, min and max horizontal lines
svg
.selectAll("toto")
.data([min, median, max])
.enter()
.append("line")
  .attr("x1", center-width/2)
  .attr("x2", center+width/2)
  .attr("y1", function(d){ return(y(d))} )
  .attr("y2", function(d){ return(y(d))} )
  .attr("stroke", "black")
</script>

"""
)

```

## <a name="popular-libraries"></a>Популярные библиотеки
Когда дело доходит до визуализации данных, Python предлагает несколько библиотек для построения диаграмм, которые упаковываются с множеством различных функций. По умолчанию каждый пул Apache Spark в Azure синапсе Analytics содержит набор проверенных и популярных библиотек с открытым исходным кодом. Вы также можете добавлять дополнительные библиотеки & версий или управлять ими с помощью возможностей управления библиотекой Azure синапсе Analytics. 

### <a name="bokeh"></a>Bokeh
Можно визуализировать HTML или интерактивные библиотеки, например **Боке**, с помощью ```displayHTML(df)``` . 

Ниже приведен пример построения глифов на карте с использованием **bokeh**.

   ![bokeh-example](./media/apache-spark-development-using-notebooks/synapse-bokeh-image.png#lightbox)

Выполните следующий пример кода, чтобы нарисовать изображение выше.

```python
from bokeh.plotting import figure, output_file
from bokeh.tile_providers import get_provider, Vendors
from bokeh.embed import file_html
from bokeh.resources import CDN
from bokeh.models import ColumnDataSource

tile_provider = get_provider(Vendors.CARTODBPOSITRON)

# range bounds supplied in web mercator coordinates
p = figure(x_range=(-9000000,-8000000), y_range=(4000000,5000000),
           x_axis_type="mercator", y_axis_type="mercator")
p.add_tile(tile_provider)

# plot datapoints on the map
source = ColumnDataSource(
    data=dict(x=[ -8800000, -8500000 , -8800000],
              y=[4200000, 4500000, 4900000])
)

p.circle(x="x", y="y", size=15, fill_color="blue", fill_alpha=0.8, source=source)

# create an html document that embeds the Bokeh plot
html = file_html(p, CDN, "my plot1")

# display this html
displayHTML(html)
```

### <a name="matplotlib"></a>Matplotlib
Вы можете визуализировать стандартные библиотеки построения графиков, например Matplotlib, используя встроенные функции отрисовки для каждой библиотеки.

На следующем рисунке показан пример создания линейчатой диаграммы с помощью **Matplotlib**.
   ![Пример графа линий.](./media/apache-spark-data-viz/matplotlib-example.png#lightbox)

Выполните следующий пример кода, чтобы нарисовать изображение выше.

```python
# Bar chart

import matplotlib.pyplot as plt

x1 = [1, 3, 4, 5, 6, 7, 9]
y1 = [4, 7, 2, 4, 7, 8, 3]

x2 = [2, 4, 6, 8, 10]
y2 = [5, 6, 2, 6, 2]

plt.bar(x1, y1, label="Blue Bar", color='b')
plt.bar(x2, y2, label="Green Bar", color='g')
plt.plot()

plt.xlabel("bar number")
plt.ylabel("bar height")
plt.title("Bar Chart Example")
plt.legend()
plt.show()
```

### <a name="additional-libraries"></a>Дополнительные библиотеки 
Помимо этих библиотек, среда выполнения Azure синапсе Analytics также включает следующий набор библиотек, которые часто используются для визуализации данных:
- [Matplotlib](https://matplotlib.org/)
- [Bokeh](https://bokeh.org/)
- [Seaborn](https://seaborn.pydata.org/) 

Вы можете посетить [документацию](./spark/../apache-spark-version-support.md) по среде выполнения Azure синапсе Analytics, чтобы получить актуальные сведения о доступных библиотеках и версиях.

## <a name="connect-to-power-bi-using-apache-spark--sql-on-demand"></a>Подключение к Power BI с помощью Apache Spark & SQL по запросу
Azure синапсе Analytics тесно интегрируется с Power BI, позволяя инженерам по анализу данных создавать решения аналитики.

Azure синапсе Analytics позволяет различным вычислительным системам рабочей области совместно использовать базы данных и таблицы между своими пулами Spark и независимым от сервера пулом SQL. С помощью [общей модели метаданных](../metadata/overview.md)можно выполнять запросы к Apache Sparkным таблицам с помощью SQL по запросу. После этого можно подключить конечную точку SQL по запросу, чтобы Power BI, чтобы легко выполнять запросы к синхронизированным таблицам Spark.


## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о настройке соединителя хранилища данных Spark SQL: [СИНАПСЕ SQL Connector](./spark/../synapse-spark-sql-pool-import-export.md)
- Просмотр библиотек по умолчанию: [Среда выполнения Azure синапсе Analytics](../spark/apache-spark-version-support.md)
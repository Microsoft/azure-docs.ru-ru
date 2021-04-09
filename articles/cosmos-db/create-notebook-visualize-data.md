---
title: Руководство по Создание записной книжки в Azure Cosmos DB для анализа и визуализации данных
description: Руководство по Узнайте, как использовать встроенные записные книжки Jupyter для импорта данных в Azure Cosmos DB, анализа данных и визуализации результатов.
author: deborahc
ms.topic: tutorial
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 11/05/2019
ms.author: dech
ms.reviewer: sngun
ms.openlocfilehash: beb58922172a045242f4f9bcaf647b3cfc8b5551
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100380803"
---
# <a name="tutorial-create-a-notebook-in-azure-cosmos-db-to-analyze-and-visualize-the-data"></a>Руководство по Создание записной книжки в Azure Cosmos DB для анализа и визуализации данных
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описано, как использовать встроенные записные книжки Jupyter для импорта демонстрационных данных розничной торговли в Azure Cosmos DB. Вы узнаете, как использовать магические команды SQL и Azure Cosmos DB для выполнения запросов, анализа данных и визуализации результатов.

## <a name="prerequisites"></a>Предварительные требования

* [Включенные записные книжки для учетной записи Azure Cosmos.](enable-notebooks.md)

## <a name="create-the-resources-and-import-data"></a>Создание ресурсов и импорт данных
 
В этом разделе вы создадите базу данных Azure Cosmos, контейнер и импортируете розничные данные в контейнер.

1. Перейдите к своей учетной записи Azure Cosmos и откройте меню **Data Explorer**.

1. Перейдите на вкладку **Записная книжка**, выберите `…` рядом с **Мои записные книжки** и создайте **Новую записную книжку**. Выберите **Python 3** в качестве ядра по умолчанию.

   :::image type="content" source="./media/create-notebook-visualize-data/create-new-notebook.png" alt-text="Создание записной книжки":::

1. После создания записной книжки ее можно переименовать на нечто вроде **VisualizeRetailData.ipynb.**

1. Далее необходимо создать базу данных с именем "RetailDemo" и контейнер с именем "WebsiteData" для хранения розничных данных. В качестве ключа секции можно использовать /CartID. Скопируйте и вставьте следующий код в новую ячейку записной книжки и запустите ее.

   ```python
   import azure.cosmos
   from azure.cosmos.partition_key import PartitionKey

   database = cosmos_client.create_database_if_not_exists('RetailDemo')
   print('Database RetailDemo created')

   container = database.create_container_if_not_exists(id='WebsiteData', partition_key=PartitionKey(path='/CartID'))
   print('Container WebsiteData created')
   ```

   Чтобы запустить ячейку, выберите `Shift + Enter` или выделите ячейку и выберите параметр **Запустить активную ячейку** на панели навигации обозревателя данных.

   :::image type="content" source="./media/create-notebook-visualize-data/run-active-cell.png" alt-text="Запуск активной ячейки":::

   База данных и контейнер создаются в текущей учетной записи Azure Cosmos. Контейнер подготавливается с 400 единицами запросов в секунду. После создания базы данных и контейнера вы увидите следующие выходные данные. 

   ```console
    Database RetailDemo created
    Container WebsiteData created
   ```

   Вы также можете обновить вкладку **Данные** и просмотреть только что созданные ресурсы.

   :::image type="content" source="media/create-notebook-visualize-data/refresh-data-tab.png" alt-text="Обновление вкладки Данные для просмотра нового контейнера":::

1. Далее вы импортируете примеры розничных данных в контейнер Azure Cosmos. Ниже приведен формат элемента из данных розничной торговли.

   ```json
    {
       "CartID":5399,
       "Action":"Viewed",
       "Item":"Cosmos T-shirt",
       "Price":350,
       "UserName":"Demo.User10",
       "Country":"Iceland",
       "EventDate":"2015-06-25T00:00:00",
       "Year":2015,"Latitude":-66.8673,
       "Longitude":-29.8214,
       "Address":"852 Modesto Loop, Port Ola, Iceland",
       "id":"00ffd39c-7e98-4451-9b91-b2bcf2f9a32d"
    }
   ```

   Для целей обучения образец розничных данных хранится в хранилище BLOB-объектов Azure. Вы можете импортировать его в контейнер Azure Cosmos, вставив следующий код в новую ячейку. Вы можете убедиться, что данные успешно импортированы, выполнив запрос для выбора количества элементов.

   ```python
    # Read data from storage
    import urllib.request, json

    with urllib.request.urlopen("https://cosmosnotebooksdata.blob.core.windows.net/notebookdata/websiteData.json") as url:
      data = json.loads(url.read().decode())

    print("Importing data. This will take a few minutes...\n")

    for event in data:
     try: 
        container.upsert_item(body=event)
     except errors.CosmosHttpResponseError as e:
        raise
        
    ## Run a query against the container to see number of documents
    query = 'SELECT VALUE COUNT(1) FROM c'
    result = list(container.query_items(query, enable_cross_partition_query=True))

    print('Container with id \'{0}\' contains \'{1}\' items'.format(container.id, result[0]))
   ```

   При выполнении предыдущего запроса он возвращает приведенные ниже выходные данные.

   ```console
   Importing data. This will take a few minutes...

   Container with id 'WebsiteData' contains '2654' items
   ```

## <a name="get-your-data-into-a-dataframe"></a>Получение данных в кадр данных

Перед выполнением запросов для анализа данных вы можете считать данные из контейнера в [кадр данных Pandas](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html) для анализа. Используйте следующую магическую команду SQL, чтобы считать данные в кадр данных.

```bash
%%sql --database {database_id} --container {container_id} --output outputDataframeVar 
{Query text}
```

Дополнительные сведения см. в статье о [командах и компонентах записной книжки в Azure Cosmos DB](use-python-notebook-features-and-commands.md). Вы выполните запрос – `SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c`. Результаты будут сохранены в кадр данных Pandas с именем df_cosmos. Вставьте приведенную ниже команду в ячейку записной книжки и запустите ее.

```python
%%sql --database RetailDemo --container WebsiteData --output df_cosmos
SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c
```

В новой ячейке записной книжки запустите следующий код, чтобы считать первые 10 элементов из выходных данных:

```python
# See a sample of the result
df_cosmos.head(10)
```

:::image type="content" source="./media/create-notebook-visualize-data/run-query-get-top10-items.png" alt-text="Выполнение запроса для получения первых десяти элементов":::

## <a name="run-queries-and-analyze-your-data"></a>Выполнение запросов и анализ данных

В этом разделе вы запустите некоторые запросы к извлеченным данным.

* **Запрос 1:** Выполните запрос "Группировать по" к кадру данных, чтобы получить сумму дохода от продаж по каждой стране или каждому региону и отобразить 5 элементов из результатов. Выполните приведенную ниже команду в ячейке записной книжки.

   ```python
   df_revenue = df_cosmos.groupby("Country").sum().reset_index()
   display(df_revenue.head(5))
   ```

   :::image type="content" source="./media/create-notebook-visualize-data/total-sales-revenue-output.png" alt-text="Сумма дохода от продаж в выходных данных":::

* **Запрос 2**. Чтобы получить список первых пяти приобретенных элементов, откройте новую ячейку записной книжки и выполните приведенный ниже код.

   ```python
   import pandas as pd

   ## What are the top 5 purchased items?
   pd.DataFrame(df_cosmos[df_cosmos['Action']=='Purchased'].groupby('Item').size().sort_values(ascending=False).head(5), columns=['Count'])
   ```

   :::image type="content" source="./media/create-notebook-visualize-data/top5-purchased-items.png" alt-text="Пять ведущих товаров по объему продаж":::

## <a name="visualize-your-data"></a>Визуализация данных  

1. Теперь, когда у нас есть данные о доходах от контейнера Azure Cosmos, вы можете визуализировать данные с помощью библиотеки визуализации по своему усмотрению. В этом руководстве используется библиотека Bokeh. Откройте новую ячейку записной книжки и выполните следующий код, чтобы установить библиотеку Bokeh. Библиотека будет установлена после соблюдения всех требований.

   ```python
   import sys
   !{sys.executable} -m pip install bokeh --user
   ```

1. Далее выполните подготовку для графического представления данных на карте. Присоедините данные в Azure Cosmos DB с информацией о стране или регионе, расположенном в хранилище BLOB-объектов Azure, и конвертируйте результат в формат GeoJSON. Скопируйте приведенный ниже код в ячейку записной книжки и выполните его.

   ```python
   import urllib.request, json
   import geopandas as gpd

   # Load country/region information for mapping
   countries = gpd.read_file("https://cosmosnotebooksdata.blob.core.windows.net/notebookdata/countries.json")

   # Merge the countries/regions dataframe with our data in Azure Cosmos DB, joining on country/region code
   df_merged = countries.merge(df_revenue, left_on = 'admin', right_on = 'Country', how='left')

   # Convert to GeoJSON so bokeh can plot it
   merged_json = json.loads(df_merged.to_json())
   json_data = json.dumps(merged_json)
   ```

1. Визуализируйте доход от продаж различных стран или регионов на карте мира, выполнив следующий код в новой ячейке записной книжки:

   ```python
   from bokeh.io import output_notebook, show
   from bokeh.plotting import figure
   from bokeh.models import GeoJSONDataSource, LinearColorMapper, ColorBar
   from bokeh.palettes import brewer
    
   #Input GeoJSON source that contains features for plotting.
   geosource = GeoJSONDataSource(geojson = json_data)
    
   #Choose our choropleth color palette: https://bokeh.pydata.org/en/latest/docs/reference/palettes.html
   palette = brewer['YlGn'][8]
    
   #Reverse color order so that dark green is highest revenue
   palette = palette[::-1]
    
   #Instantiate LinearColorMapper that linearly maps numbers in a range, into a sequence of colors.
   color_mapper = LinearColorMapper(palette = palette, low = 0, high = 1000)
    
   #Define custom tick labels for color bar.
   tick_labels = {'0': '$0', '250': '$250', '500':'$500', '750':'$750', '1000':'$1000', '1250':'$1250', '1500':'$1500','1750':'$1750', '2000': '>$2000'}
    
   #Create color bar. 
   color_bar = ColorBar(color_mapper=color_mapper, label_standoff=8,width = 500, height = 20,
   border_line_color=None,location = (0,0), orientation = 'horizontal', major_label_overrides = tick_labels)
    
   #Create figure object.
   p = figure(title = 'Sales revenue by country', plot_height = 600 , plot_width = 1150, toolbar_location = None)
   p.xgrid.grid_line_color = None
   p.ygrid.grid_line_color = None
    
   #Add patch renderer to figure. 
   p.patches('xs','ys', source = geosource,fill_color = {'field' :'ItemRevenue', 'transform' : color_mapper},
              line_color = 'black', line_width = 0.25, fill_alpha = 1)
    
   #Specify figure layout.
   p.add_layout(color_bar, 'below')
    
   #Display figure inline in Jupyter Notebook.
   output_notebook()
   
   #Display figure.
   show(p)
   ```

   В выходных данных отображается геосхема мира с различными цветами. Более светлое цветовое значение представляет страны или регионы от наибольшего до наименьшего дохода.

   :::image type="content" source="./media/create-notebook-visualize-data/countries-revenue-map-visualization.png" alt-text="Визуализация дохода для стран или регионов":::

1. Рассмотрим еще один вариант визуализации данных. Контейнер WebsiteData содержит записи о пользователях, которые просмотрели товар, добавили его в корзину и приобрели элемент. Давайте построим график коэффициента конверсии приобретенных товаров. Выполните следующий код в новой ячейке, чтобы визуализировать коэффициент конверсии для каждого элемента:

   ```python
   from bokeh.io import show, output_notebook
   from bokeh.plotting import figure
   from bokeh.palettes import Spectral3
   from bokeh.transform import factor_cmap
   from bokeh.models import ColumnDataSource, FactorRange
       
   # Get the top 10 items as an array
   top_10_items = df_cosmos[df_cosmos['Action']=='Purchased'].groupby('Item').size().sort_values(ascending=False)[:10].index.values.tolist()
    
   # Filter our data to only these 10 items
   df_top10 = df_cosmos[df_cosmos['Item'].isin(top_10_items)]
    
   # Group by Item and Action, sorting by event count
   df_top10_sorted = df_top10.groupby(['Item', 'Action']).count().rename(columns={'Country':'ResultCount'}, inplace=False).reset_index().sort_values(['Item', 'ResultCount'], ascending = False).set_index(['Item', 'Action'])
    
   # Get sorted X-axis values - this way, we can display the funnel of view -> add -> purchase
   x_axis_values = df_top10_sorted.index.values.tolist()
    
   group = df_top10_sorted.groupby(['Item', 'Action'])
    
   # Specifiy colors for X axis
   index_cmap = factor_cmap('Item_Action', palette=Spectral3, factors=sorted(df_top10.Action.unique()), start=1, end=2)
    
   # Create the plot
    
   p = figure(plot_width=1200, plot_height=500, title="Conversion rate of items from View -> Add to cart -> Purchase", x_range=FactorRange(*x_axis_values), toolbar_location=None, tooltips=[("Number of events", "@ResultCount_max"), ("Item, Action", "@Item_Action")])
    
   p.vbar(x='Item_Action', top='ItemRevenue_max', width=1, source=group,
           line_color="white", fill_color=index_cmap, )
    
   #Configure how the plot looks
   p.y_range.start = 0
   p.x_range.range_padding = 0.05
   p.xgrid.grid_line_color = None
   p.xaxis.major_label_orientation = 1.2
   p.outline_line_color = "black"
   p.xaxis.axis_label = "Item"
   p.yaxis.axis_label = "Count"
    
   #Display figure inline in Jupyter Notebook.
   output_notebook()
    
   #Display figure.
   show(p)
   ```

   :::image type="content" source="./media/create-notebook-visualize-data/visualize-purchase-conversion-rate.png" alt-text="Визуализация коэффициента конверсии покупок":::

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о командах записной книжки Python см. в статье [Использование встроенных команд и компонентов записной книжки Python в Azure Cosmos DB](use-python-notebook-features-and-commands.md).

---
title: Использование встроенных команд и функций записных книжек Python в Azure Cosmos DB (предварительная версия)
description: Узнайте, как использовать встроенные команды и функции для выполнения стандартных операций с помощью встроенных записных книжек Python в Azure Cosmos DB.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/19/2020
ms.author: dech
ms.openlocfilehash: b89fcf32ed033f359b4db601e36cc69bb899944d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98165830"
---
# <a name="use-built-in-notebook-commands-and-features-in-azure-cosmos-db-python-notebooks-preview"></a>Использование встроенных команд и функций записных книжек Python в Azure Cosmos DB (предварительная версия)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Встроенные записные книжки Jupyter в Azure Cosmos DB позволяют анализировать и визуализировать данные с портала Azure. В этой статье описывается, как использовать встроенные команды и функции записной книжки для выполнения стандартных операций в записных книжках Python.

## <a name="install-a-new-package"></a>Установка нового пакета
После включения поддержки записных книжек для учетных записей Azure Cosmos можно открыть новую записную книжку и установить пакет.

В новой ячейке кода вставьте и выполните приведенный ниже код, заменив ``PackageToBeInstalled`` соответствующим пакетом Python.
```python
import sys
!{sys.executable} -m pip install PackageToBeInstalled --user
```
Этот пакет будет доступен для использования из любой записной книжки в рабочей области учетной записи Azure Cosmos. 

> [!TIP]
> Если записной книжке требуется пользовательский пакет, мы рекомендуем добавить ячейку в записную книжку, чтобы установить этот пакет, так как при [сбросе рабочей области](#reset-notebooks-workspace) пакеты удаляются.  

## <a name="run-a-sql-query"></a>Выполнение SQL-запроса

Вы можете использовать магическую команду ``%%sql``, чтобы выполнить [SQL-запрос](sql-query-getting-started.md) для любого контейнера в учетной записи. Используйте следующий синтаксис:

```python
%%sql --database {database_id} --container {container_id}
{Query text}
```

- Замените ``{database_id}`` и ``{container_id}`` именами базы данных и контейнера в своей учетной записи Cosmos. Если аргументы ``--database`` и ``--container`` не указаны, запрос будет выполнен для [базы данных и контейнера по умолчанию](#set-default-database-for-queries)
- Можно выполнить любой SQL-запрос, допустимый в Azure Cosmos DB. Текст запроса должен быть размещен в новой строке.

Пример: 
```python
%%sql --database RetailDemo --container WebsiteData
SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c
```
Выполните ```%%sql?``` в ячейке, чтобы просмотреть справочную документацию по магической команде SQL в записной книжке.

## <a name="run-a-sql-query-and-output-to-a-pandas-dataframe"></a>Выполнение SQL-запроса и вывод данных в кадр данных Pandas

Результаты запроса ``%%sql`` можно вывести в [кадр данных Pandas](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame). Используйте следующий синтаксис: 

```python
%%sql --database {database_id} --container {container_id} --output {outputDataFrameVar}
{Query text}
```
- Замените ``{database_id}`` и ``{container_id}`` именами базы данных и контейнера в своей учетной записи Cosmos. Если аргументы ``--database`` и ``--container`` не указаны, запрос будет выполнен для [базы данных и контейнера по умолчанию](#set-default-database-for-queries)
- Замените ``{outputDataFrameVar}`` именем переменной кадра данных, которая будет содержать результаты.
- Можно выполнить любой SQL-запрос, допустимый в Azure Cosmos DB. Текст запроса должен быть размещен в новой строке. 

Пример:

```python
%%sql --database RetailDemo --container WebsiteData --output df_cosmos
SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c
```
```python
df_cosmos.head(10)

    Action    ItemRevenue    Country/Region    Item
0    Viewed    9.00    Tunisia    Black Tee
1    Viewed    19.99    Antigua and Barbuda    Flannel Shirt
2    Added    3.75    Guinea-Bissau    Socks
3    Viewed    3.75    Guinea-Bissau    Socks
4    Viewed    55.00    Czech Republic    Rainjacket
5    Viewed    350.00    Iceland    Cosmos T-shirt
6    Added    19.99    Syrian Arab Republic    Button-Up Shirt
7    Viewed    19.99    Syrian Arab Republic    Button-Up Shirt
8    Viewed    33.00    Tuvalu    Red Top
9    Viewed    14.00    Cabo Verde    Flip Flop Shoes
```
## <a name="set-default-database-for-queries"></a>Указание базы данных по умолчанию для запросов
Вы можете задать базу данных по умолчанию, которую будут использовать команды ```%%sql``` для записной книжки. Замените ```{database_id}``` на имя вашей базы данных.

```python
%database {database_id}
```
Выполните ```%database?``` в ячейке, чтобы просмотреть документацию в записной книжке.

## <a name="set-default-container-for-queries"></a>Указание контейнера по умолчанию для запросов
Вы можете задать контейнер по умолчанию, который будут использовать команды ```%%sql``` для записной книжки. Замените ```{container_id}``` именем своего контейнера.

```python
%container {container_id}
```
Выполните ```%container?``` в ячейке, чтобы просмотреть документацию в записной книжке.

## <a name="upload-json-items-to-a-container"></a>Отправка элементов JSON в контейнер
Магическую команду ``%%upload`` можно использовать для передачи данных из JSON-файла в указанный контейнер Azure Cosmos. Используйте для отправки элементов следующую команду:

```python
%%upload --databaseName {database_id} --containerName {container_id} --url {url_location_of_file}
```

- Замените ``{database_id}`` и ``{container_id}`` именами базы данных и контейнера в своей учетной записи Azure Cosmos. Если аргументы ``--database`` и ``--container`` не указаны, запрос будет выполнен для [базы данных и контейнера по умолчанию](#set-default-database-for-queries)
- Замените ``{url_location_of_file}`` расположением JSON-файла. Файл должен представлять собой массив действительных объектов JSON и должен быть доступен через общедоступный Интернет.

Пример:

```python
%%upload --database databaseName --container containerName --url 
https://contoso.com/path/to/data.json
```
```
Documents successfully uploaded to ContainerName
Total number of documents imported : 2654
Total time taken : 00:00:38.1228087 hours
Total RUs consumed : 25022.58
```
С помощью выходной статистики можно вычислить эффективный показатель (единиц запросов в секунду) для отправки элементов. Например, если 25 000 единиц запросов было использовано за 38 секунд, то эффективный показатель равен 25 000 единиц запросов / 38 секунд = 658 единиц запросов/с.

Файлы (например, CSV или JSON) можно сохранить в локальной рабочей области записной книжки. Для сохранения файлов рекомендуется добавить в записную книжку ячейку. Эти файлы можно просмотреть в интегрированном терминале в среде записной книжки. Для просмотра сохраненных файлов можно использовать команду "ls". Однако эти файлы удаляются при сбросе рабочей области. Поэтому лучше использовать постоянное хранилище, например GitHub или учетную запись хранения, а не локальную рабочую область.

## <a name="run-another-notebook-in-current-notebook"></a>Запуск другой записной книжки в данной записной книжке 
Можно использовать магическую команду ``%%run`` для запуска из текущей записной книжки другой записной книжки в своем рабочем пространстве. Используйте следующий синтаксис:

```python
%%run ./path/to/{notebookName}.ipynb
```
Замените ``{notebookName}`` именем записной книжки, которую нужно запустить. Записная книжка должна находиться в текущей рабочей области "Мои записные книжки". 

## <a name="use-built-in-nteract-data-explorer"></a>Использование встроенного обозревателя данных nteract
Для фильтрации и визуализации кадра данных можно использовать встроенный [обозреватель данных nteract](https://blog.nteract.io/designing-the-nteract-data-explorer-f4476d53f897). Чтобы включить эту функцию, задайте для параметра ``pd.options.display.html.table_schema`` значение ``True``, а для ``pd.options.display.max_rows`` — подходящее вам значение (можно задать ``pd.options.display.max_rows`` ``None``, чтобы отобразить все результаты).

```python
import pandas as pd
pd.options.display.html.table_schema = True
pd.options.display.max_rows = None

df_cosmos.groupby("Item").size()
```
:::image type="content" source="media/use-notebook-features-and-commands/nteract-built-in-chart.png" alt-text="Обозреватель данных nteract":::

## <a name="use-the-built-in-python-sdk"></a>Использование встроенного пакета SDK для Python
Версия 4 [пакета SDK Python для Azure Cosmos DB для API SQL](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos) установлена и добавлена в среду записных книжек для учетной записи Azure Cosmos.

Используйте встроенный экземпляр ``cosmos_client`` для выполнения любой операции SDK. 

Пример:

```python
## Import modules as needed
from azure.cosmos.partition_key import PartitionKey

## Create a new database if it doesn't exist
database = cosmos_client.create_database_if_not_exists('RetailDemo')

## Create a new container if it doesn't exist
container = database.create_container_if_not_exists(id='WebsiteData', partition_key=PartitionKey(path='/CartID'))
```
Ознакомьтесь с [примерами для пакета SDK для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos/samples). 

> [!IMPORTANT]
> Встроенный пакет SDK для Python поддерживается только для учетных записей API SQL (Core). Для других API необходимо [установить драйвер Python](#install-a-new-package), соответствующий конкретному API. 

## <a name="create-a-custom-instance-of-cosmos_client"></a>Создание пользовательского экземпляра ``cosmos_client``
Для повышения гибкости можно создать пользовательский экземпляр ``cosmos_client``, чтобы:

- настроить [политику подключения](/python/api/azure-cosmos/azure.cosmos.documents.connectionpolicy?preserve-view=true&view=azure-python-preview);
- выполнять операции с другой учетной записью Azure Cosmos, отличной от вашей.

Вы можете получить строку подключения и первичный ключ текущей учетной записи с помощью [переменных среды](#access-the-account-endpoint-and-primary-key-env-variables). 

```python
import azure.cosmos.cosmos_client as cosmos
import azure.cosmos.documents as documents

# These should be set to a region you've added for Azure Cosmos DB
region_1 = "Central US" 
region_2 = "East US 2"

custom_connection_policy = documents.ConnectionPolicy()
custom_connection_policy.PreferredLocations = [region_1, region_2] # Set the order of regions the SDK will route requests to. The regions should be regions you've added for Cosmos, otherwise this will error.

# Create a new instance of CosmosClient, getting the endpoint and key from the environment variables
custom_client = cosmos.CosmosClient(url=COSMOS.ENDPOINT, credential=COSMOS.KEY, connection_policy=custom_connection_policy)

```
## <a name="access-the-account-endpoint-and-primary-key-env-variables"></a>Доступ к переменным среды конечной точки и первичного ключа учетной записи
Вы можете получить доступ к встроенной конечной точке и ключу учетной записи, в которой размещена ваша записная книжка.

```python
endpoint = COSMOS.ENDPOINT
primary_key = COSMOS.KEY
```
> [!IMPORTANT]
> Переменные ``COSMOS.ENDPOINT`` и ``COSMOS.KEY`` применимы только для API SQL. Для других API найдите конечную точку и ключ в колонке **Строки подключения** или **Ключи** учетной записи Azure Cosmos.  

## <a name="reset-notebooks-workspace"></a>Сброс рабочей области записных книжек
Чтобы сбросить рабочую область записных книжек до параметров по умолчанию, выберите **Сбросить рабочую область** на панели команд. При этом будут удалены все установленные пользовательские пакеты и перезапущен сервер Jupyter. Ваши записные книжки, файлы и ресурсы Azure Cosmos затронуты не будут.  

:::image type="content" source="media/use-notebook-features-and-commands/reset-workspace.png" alt-text="Сброс рабочей области записных книжек":::

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о преимуществах использования [записных книжек Jupyter в Azure Cosmos DB](cosmosdb-jupyter-notebooks.md).
- Узнайте больше о [пакете SDK Python для Azure Cosmos DB для API SQL](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos).
---
title: Создание наборов данных Машинного обучения Azure
titleSuffix: Azure Machine Learning
description: Узнайте, как создать Машинное обучение Azure наборы данных для доступа к данным для запуска экспериментов машинного обучения.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1, data4ml
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 07/31/2020
ms.openlocfilehash: a8f1ca1da54c816199a0504eb17fa0a7bbfc441b
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102522195"
---
# <a name="create-azure-machine-learning-datasets"></a>Создание наборов данных Машинного обучения Azure

Из этой статьи вы узнаете, как создавать наборы данных Машинное обучение Azure для доступа к данным для локальных и удаленных экспериментов с помощью пакета SDK для Машинное обучение Azure Python. Чтобы узнать, где находятся наборы данных в общем рабочем процессе доступа к данным Машинное обучение Azure, ознакомьтесь со статьей [безопасный доступ к](concept-data.md#data-workflow) данным.

Создавая набор данных, вы создаете ссылку на расположение источника данных, а также копию его метаданных. Поскольку данные остаются в существующем расположении, не взимается дополнительная плата за хранение и не рискуйте целостность источников данных. Кроме того, наборы данных отложены по производительности, что помогает в скорости рабочего процесса. Наборы данных можно создавать на основе хранилищ данных, общедоступных URL-адресов и [открытых наборов данных Azure](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md).

Для работы с низким кодом [создайте машинное обучение Azure наборы данных с помощью машинное обучение Azure Studio.](how-to-connect-data-ui.md#create-datasets)

С помощью Машинное обучение Azure наборов данных можно:

* Сохранение одной копии данных в хранилище, на которые ссылаются наборы данных.

* Беспроблемный доступ к данным во время обучения модели, не беспокоясь о строках подключения или путях к данным. [Узнайте больше об обучении с наборами данных](how-to-train-with-datasets.md).

* Совместное использование данных и совместная работа с другими пользователями.

## <a name="prerequisites"></a>Предварительные условия

Для создания наборов данных и работы с ними требуется:

* Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начать работу. Попробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).

* [Рабочая область машинное обучение Azure](how-to-manage-workspace.md).

* [Установленный пакет SDK для машинное обучение Azure для Python](/python/api/overview/azure/ml/install), включающий пакет azureml-DataSets.

    * Создание [машинное обучение Azure вычислительного экземпляра](how-to-create-manage-compute-instance.md), который представляет собой полностью настроенную и управляемую среду разработки, включающую интегрированные записные книжки и уже установленный пакет SDK.

    **OR**

    * Поработайте с собственной записной книжкой Jupyter и установите пакет SDK самостоятельно с помощью [этих инструкций](/python/api/overview/azure/ml/install).

> [!NOTE]
> Некоторые классы наборов данных имеют зависимости от пакета [azureml-](/python/api/azureml-dataprep/) DataMarket, который совместим только с 64-разрядным Python. Для пользователей Linux эти классы поддерживаются только в следующих дистрибутивах: Red Hat Enterprise Linux (7, 8), Ubuntu (14,04, 16,04, 18,04), Fedora (27, 28), Debian (8, 9) и CentOS (7). Если вы используете неподдерживаемый дистрибутивов, воспользуйтесь [этим руководством](/dotnet/core/install/linux) , чтобы установить .net Core 2,1, чтобы продолжить. 

## <a name="compute-size-guidance"></a>Руководство по размеру вычислений

При создании набора данных проверьте вычислительную мощность и размер данных в памяти. Размер данных в хранилище не совпадает с размером данных в кадре. Например, данные в CSV-файлах могут расширяться до 10 раз в кадре данных, поэтому файл CSV размером 1 ГБ может стать 10 ГБ в кадре данных. 

Если данные сжаты, их можно расширить. 20 ГБ относительно разреженных данных, хранящихся в сжатом формате Parquet, можно увеличить до ~ 800 ГБ в памяти. Поскольку файлы Parquet хранят данные в формате столбцов, если требуется только половина столбцов, необходимо загрузить в память ~ 400 ГБ.

Дополнительные [сведения об оптимизации обработки данных в машинное обучение Azure](concept-optimize-data-processing.md).

## <a name="dataset-types"></a>Типы наборов данных

Существует два типа наборов данных в зависимости от того, как пользователи их используют в обучении; Филедатасетс и Табулардатасетс. Оба типа можно использовать в Машинное обучение Azure рабочих процессах обучения, в которых участвуют, оценивающие, Аутомл, а также устройства и конвейеры. 

### <a name="filedataset"></a>филедатасет

[Филедатасет](/python/api/azureml-core/azureml.data.file_dataset.filedataset) ссылается на один или несколько файлов в хранилищах данных или общедоступных URL-адресах. Если данные уже очищены и готовы к использованию в учебных экспериментах, вы можете [скачать или подключить](how-to-train-with-datasets.md#mount-vs-download) файлы в Compute в качестве объекта филедатасет. 

Мы рекомендуем Филедатасетс для рабочих процессов машинного обучения, так как исходные файлы могут иметь любой формат, что позволяет использовать более широкий спектр сценариев машинного обучения, включая глубокое обучение.

Создайте Филедатасет с помощью [пакета SDK для Python](#create-a-filedataset) или [машинное обучение Azure Studio](how-to-connect-data-ui.md#create-datasets) .
### <a name="tabulardataset"></a>Табличный набор данных

[Табулардатасет](/python/api/azureml-core/azureml.data.tabulardataset) представляет данные в табличном формате путем синтаксического анализа указанного файла или списка файлов. Это дает возможность материализовать данные в таблицу данных Pandas или Spark, чтобы вы могли работать с привычными библиотеками подготовки и обучения данных, не покидая записной книжки. Вы можете создать `TabularDataset` объект из файлов. csv,. TSV,. Parquet,. JSON и из [результатов SQL-запроса](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-sql-query-query--validate-true--set-column-types-none--query-timeout-30-).

С помощью Табулардатасетс можно указать отметку времени из столбца данных или из любого места, где хранятся данные шаблона пути, чтобы включить признаки временных рядов. Эта спецификация обеспечивает простую и эффективную фильтрацию по времени. Пример см. в статье [Демонстрация API, относящегося к табличному ряду, с данными о погоде NOAA](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/timeseries-datasets/tabular-timeseries-dataset-filtering.ipynb).

Создание Табулардатасет с помощью [пакета SDK для Python](#create-a-tabulardataset) или [машинное обучение Azure Studio](how-to-connect-data-ui.md#create-datasets).

>[!NOTE]
> [Автоматизированные](concept-automated-ml.md) рабочие процессы ml, созданные с помощью машинное обучение Azure Studio, в настоящее время поддерживают только табулардатасетс. 

## <a name="access-datasets-in-a-virtual-network"></a>Доступ к наборам данных в виртуальной сети

Если Рабочая область находится в виртуальной сети, необходимо настроить набор данных для пропуска проверки. Дополнительные сведения об использовании хранилищ и наборов данных в виртуальной сети см. в статье [Защита рабочей области и связанных ресурсов](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

<a name="datasets-sdk"></a>

## <a name="create-datasets-from-datastores"></a>Создание наборов данных из хранилищ данных

Чтобы данные были доступны по Машинное обучение Azure, необходимо создать наборы данных по путям в [машинное обучение Azure хранилищах данных](how-to-access-data.md) или веб-URL. 

> [!TIP] 
> Наборы данных можно создавать непосредственно из URL-адресов хранилища с доступом к данным на основе удостоверений. Дополнительные сведения см. в статье [Подключение к хранилищу с доступом к данным на основе удостоверений (Предварительная версия)](how-to-identity-based-data-access.md) .<br><br>
Эта возможность является [экспериментальной](/python/api/overview/azure/ml/#stable-vs-experimental) функцией предварительной версии и может быть изменена в любое время. 

 
Чтобы создать наборы данных из хранилища данных с помощью пакета SDK для Python, сделайте следующее:

1. Убедитесь, что у вас есть `contributor` `owner` доступ к базовой службе хранилища зарегистрированного машинное обучение Azure хранилища данных. [Проверьте разрешения учетной записи хранения в портал Azure](../role-based-access-control/check-access.md).

1. Создайте набор данных, ссылаясь на пути в хранилище данных. Набор данных можно создать из нескольких путей в нескольких хранилищах. Нет жесткого ограничения на количество файлов или объем данных, из которых можно создать набор данных. 

> [!NOTE]
> Для каждого пути данных в службу хранилища будут отправлены несколько запросов, чтобы проверить, указывает ли она на файл или папку. Эта дополнительная нагрузка может привести к снижению производительности или сбою. Набор данных, ссылающийся на одну папку с 1000 файлами в, считается ссылкой на один путь к данным. Для оптимальной производительности рекомендуется создавать наборы данных, ссылающиеся на менее 100 путей в хранилищах.

### <a name="create-a-filedataset"></a>Создание FileDataset

Используйте [`from_files()`](/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory#from-files-path--validate-true-) метод `FileDatasetFactory` класса для загрузки файлов в любом формате и создания незарегистрированного филедатасет. 

Если хранилище находится за виртуальной сетью или брандмауэром, задайте параметр `validate=False` в `from_files()` методе. Это обходит этап начальной проверки и гарантирует, что вы сможете создать набор данных на основе этих защищенных файлов. Узнайте больше о том, как [использовать хранилища и наборы данных в виртуальной сети](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

```Python
# create a FileDataset pointing to files in 'animals' folder and its subfolders recursively
datastore_paths = [(datastore, 'animals')]
animal_ds = Dataset.File.from_files(path=datastore_paths)

# create a FileDataset from image and label files behind public web urls
web_paths = ['https://azureopendatastorage.blob.core.windows.net/mnist/train-images-idx3-ubyte.gz',
             'https://azureopendatastorage.blob.core.windows.net/mnist/train-labels-idx1-ubyte.gz']
mnist_ds = Dataset.File.from_files(path=web_paths)
```
Для повторного использования наборов данных и совместного использования их в рамках эксперимента в рабочей области [зарегистрируйте свой DataSet](#register-datasets). 

> [!TIP] 
> Отправьте файлы из локального каталога и создайте Филедатасет в одном методе с помощью общедоступного метода предварительной версии [upload_directory ()](/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory#upload-directory-src-dir--target--pattern-none--overwrite-false--show-progress-true-). Этот метод является [экспериментальной](/python/api/overview/azure/ml/#stable-vs-experimental) функцией предварительной версии и может измениться в любое время. 
> 
>  Этот метод передает данные в базовое хранилище, что приводит к затратам на хранение. 

### <a name="create-a-tabulardataset"></a>Создание Табулардатасет

Используйте [`from_delimited_files()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory) метод `TabularDatasetFactory` класса для чтения файлов в формате CSV или TSV и для создания незарегистрированного табулардатасет. Если выполняется чтение из нескольких файлов, результаты будут объединены в одно табличное представление. 

Если хранилище находится за виртуальной сетью или брандмауэром, задайте параметр `validate=False` в `from_delimited_files()` методе. Это обходит этап начальной проверки и гарантирует, что вы сможете создать набор данных на основе этих защищенных файлов. Узнайте больше о том, как использовать [хранилища и наборы данных в виртуальной сети](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

Следующий код возвращает существующую рабочую область и требуемое хранилище данных по имени. Затем передает хранилище данных и расположения файлов в `path` параметр для создания нового табулардатасет, `weather_ds` .

```Python
from azureml.core import Workspace, Datastore, Dataset

datastore_name = 'your datastore name'

# get existing workspace
workspace = Workspace.from_config()
    
# retrieve an existing datastore in the workspace by name
datastore = Datastore.get(workspace, datastore_name)

# create a TabularDataset from 3 file paths in datastore
datastore_paths = [(datastore, 'weather/2018/11.csv'),
                   (datastore, 'weather/2018/12.csv'),
                   (datastore, 'weather/2019/*.csv')]

weather_ds = Dataset.Tabular.from_delimited_files(path=datastore_paths)
```
### <a name="set-data-schema"></a>Настройка схемы данных

По умолчанию при создании Табулардатасет типы данных столбца выводятся автоматически. Если выводимые типы не соответствуют ожиданиям, можно обновить схему набора данных, указав типы столбцов с помощью следующего кода. Параметр `infer_column_type` применим только для наборов данных, созданных из файлов с разделителями. Дополнительные [сведения о поддерживаемых типах данных](/python/api/azureml-core/azureml.data.dataset_factory.datatype).


```Python
from azureml.core import Dataset
from azureml.data.dataset_factory import DataType

# create a TabularDataset from a delimited file behind a public web url and convert column "Survived" to boolean
web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path, set_column_types={'Survived': DataType.to_bool()})

# preview the first 3 rows of titanic_ds
titanic_ds.take(3).to_pandas_dataframe()
```

|Номер|PassengerId|Survived|пкласс|Имя|Пол|возраст;|сибсп|парч|Билет|Плата|кабин|Предпринимались
-|-----------|--------|------|----|---|---|-----|-----|------|----|-----|--------|
0|1|False|3|Браунд, Mr. О'мэлли Owen Харрис|Мужской|22,0|1|0|A/5 21171|7,2500||S
1|2|True|1|Кумингс, Mrs. Джон Кирилл (Флоренция Бриггс TH...|Женский|38,0|1|0|PC 17599|71,2833|C85|C
2|3|True|3|Хеиккинен, промах. лаина|Женский|26,0|0|0|СТОН/O2. 3101282|7,9250||S

Чтобы повторно использовать наборы данных и обмениваться ими между экспериментами в рабочей области, [зарегистрируйте свой DataSet](#register-datasets).

## <a name="explore-data"></a>Анализ данных

После создания и [регистрации](#register-datasets) набора данных его можно загрузить в записную книжку, чтобы исследовать данные перед обучением модели. Если вам не нужно выполнять какие бы то ни было исследования данных, см. статью как использовать DataSet в сценариях обучения для отправки экспериментов ML в процессе [обучения с наборами](how-to-train-with-datasets.md).

Для Филедатасетс можно **подключить** или **скачать** набор данных, а также применить библиотеки Python, которые обычно используются для исследования данных. [Дополнительные сведения о подключении VS download](how-to-train-with-datasets.md#mount-vs-download).

```python
# download the dataset 
dataset.download(target_path='.', overwrite=False) 

# mount dataset to the temp directory at `mounted_path`

import tempfile
mounted_path = tempfile.mkdtemp()
mount_context = dataset.mount(mounted_path)

mount_context.start()
```

Для Табулардатасетс используйте метод, [`to_pandas_dataframe()`](/python/api/azureml-core/azureml.data.tabulardataset#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) чтобы просмотреть данные в кадре данных. 

```python
# preview the first 3 rows of titanic_ds
titanic_ds.take(3).to_pandas_dataframe()
```

|Номер|PassengerId|Survived|пкласс|Имя|Пол|возраст;|сибсп|парч|Билет|Плата|кабин|Предпринимались
-|-----------|--------|------|----|---|---|-----|-----|------|----|-----|--------|
0|1|False|3|Браунд, Mr. О'мэлли Owen Харрис|Мужской|22,0|1|0|A/5 21171|7,2500||S
1|2|True|1|Кумингс, Mrs. Джон Кирилл (Флоренция Бриггс TH...|Женский|38,0|1|0|PC 17599|71,2833|C85|C
2|3|True|3|Хеиккинен, промах. лаина|Женский|26,0|0|0|СТОН/O2. 3101282|7,9250||S

## <a name="create-a-dataset-from-pandas-dataframe"></a>Создание набора данных на основе Pandas

Чтобы создать Табулардатасет из кадра данных Pandas в памяти, запишите данные в локальный файл, например в CSV-файл, и создайте набор данных из этого файла. Этот рабочий процесс демонстрируется в следующем коде.

```python
# azureml-core of version 1.0.72 or higher is required
# azureml-dataprep[pandas] of version 1.1.34 or higher is required

from azureml.core import Workspace, Dataset
local_path = 'data/prepared.csv'
dataframe.to_csv(local_path)

# upload the local file to a datastore on the cloud

subscription_id = 'xxxxxxxxxxxxxxxxxxxxx'
resource_group = 'xxxxxx'
workspace_name = 'xxxxxxxxxxxxxxxx'

workspace = Workspace(subscription_id, resource_group, workspace_name)

# get the datastore to upload prepared data
datastore = workspace.get_default_datastore()

# upload the local file from src_dir to the target_path in datastore
datastore.upload(src_dir='data', target_path='data')

# create a dataset referencing the cloud location
dataset = Dataset.Tabular.from_delimited_files(path = [(datastore, ('data/prepared.csv'))])
```

> [!TIP]
> Создайте и зарегистрируйте Табулардатасет в кадре данных в памяти Spark или Pandas с помощью одного метода с общедоступными методами предварительной версии [`register_spark_dataframe()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#methods) и [`register_pandas_dataframe()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#methods) . Эти методы регистрации являются [экспериментальными](/python/api/overview/azure/ml/#stable-vs-experimental) функциями предварительной версии и могут быть изменены в любое время. 
> 
>  Эти методы отправляют данные в базовое хранилище, что приводит к затратам на хранение. 

## <a name="register-datasets"></a>Регистрация наборов данных

Чтобы завершить процесс создания, зарегистрируйте свои наборы данных в рабочей области. Используйте [`register()`](/python/api/azureml-core/azureml.data.abstract_dataset.abstractdataset#&preserve-view=trueregister-workspace--name--description-none--tags-none--create-new-version-false-) метод для регистрации наборов данных в рабочей области, чтобы поделиться ими с другими пользователями и использовать их в разных экспериментах в рабочей области:

```Python
titanic_ds = titanic_ds.register(workspace=workspace,
                                 name='titanic_ds',
                                 description='titanic training data')
```

## <a name="create-datasets-using-azure-resource-manager"></a>Создание наборов данных с помощью Azure Resource Manager

Существует ряд шаблонов [https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-dataset-create-*](https://github.com/Azure/azure-quickstart-templates/tree/master/) , которые можно использовать для создания наборов данных.

Сведения об использовании этих шаблонов см. в разделе [Использование шаблона Azure Resource Manager для создания рабочей области для машинное обучение Azure](how-to-create-workspace-template.md).


## <a name="create-datasets-from-azure-open-datasets"></a>Создание наборов данных из открытых наборов данных Azure

[Открытые наборы данных Azure](https://azure.microsoft.com/services/open-datasets/) — это проверенные общедоступные наборы данных, которые можно использовать для добавления функций конкретных сценариев в решения машинного обучения для создания более точных моделей. Наборы данных включают открытые данные о погоде, численности населения, праздниках, общественной безопасности и расположениях, которые помогают вам обучать модели машинного обучения и обогащать прогностические решения. Открытые наборы данных находятся в облаке на Microsoft Azure и включены как в пакет SDK, так и в студию.

Узнайте, как создавать [наборы данных машинное обучение Azure из открытых наборов данных Azure](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md). 

## <a name="train-with-datasets"></a>Обучение с наборами данных

Используйте свои наборы данных в экспериментах машинного обучения для обучения моделей ML. [Узнайте больше об обучении с наборами данных](how-to-train-with-datasets.md).

## <a name="version-datasets"></a>Наборы данных версий

Вы можете зарегистрировать новый набор данных с тем же именем, создав новую версию. Версия набора данных — это способ закладки состояния данных, чтобы можно было применить определенную версию набора данных для эксперимента или будущего воспроизведения. Дополнительные сведения о [версиях наборов данных](how-to-version-track-datasets.md).
```Python
# create a TabularDataset from Titanic training data
web_paths = ['https://dprepdata.blob.core.windows.net/demo/Titanic.csv',
             'https://dprepdata.blob.core.windows.net/demo/Titanic2.csv']
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_paths)

# create a new version of titanic_ds
titanic_ds = titanic_ds.register(workspace = workspace,
                                 name = 'titanic_ds',
                                 description = 'new titanic training data',
                                 create_new_version = True)
```

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, [как обучаться с помощью наборов данных](how-to-train-with-datasets.md).
* Используйте автоматическое машинное обучение для [обучения с табулардатасетс](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb).
* Дополнительные примеры обучения наборов данных см. в [примерах записных книжек](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).

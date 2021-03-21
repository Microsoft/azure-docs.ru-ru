---
title: Прием данных с помощью Фабрики данных Azure
titleSuffix: Azure Machine Learning
description: Узнайте о доступных вариантах создания конвейера приема данных с помощью фабрики данных Azure и преимуществ каждого из них.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: iefedore
author: eedorenko
manager: davete
ms.reviewer: larryfr
ms.date: 01/26/2021
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.openlocfilehash: b842934ea6bb458a59a53ea7068aa9ece98aacf1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98796149"
---
# <a name="data-ingestion-with-azure-data-factory"></a>Прием данных с помощью Фабрики данных Azure

В этой статье вы узнаете о доступных вариантах создания конвейера приема данных с помощью [фабрики данных Azure](../data-factory/introduction.md). Этот конвейер фабрики данных Azure используется для приема данных для использования с [машинное обучение Azure](overview-what-is-azure-ml.md). Фабрика данных позволяет легко извлекать, преобразовывать и загружать данные (ETL). После преобразования и загрузки данных в хранилище их можно использовать для обучения моделей машинного обучения в Машинное обучение Azure.

Простое преобразование данных можно обрабатывать с помощью собственных действий фабрики данных и инструментов, таких как [поток данных](../data-factory/control-flow-execute-data-flow-activity.md). Когда речь идет о более сложных сценариях, данные могут обрабатываться с помощью пользовательского кода. Например, код Python или R.

## <a name="compare-azure-data-factory-data-ingestion-pipelines"></a>Сравнение конвейеров приема данных в фабрике данных Azure 
Существует несколько распространенных способов использования фабрики данных для преобразования данных во время приема. Каждый метод имеет свои преимущества и недостатки, которые помогают определить, подходит ли он для конкретного варианта использования:

| Метод | Преимущества | Недостатки |
| ----- | ----- | ----- |
| Фабрика данных + функции Azure | <li> Низкая задержка, бессерверные вычислений<li>Функции с отслеживанием состояния<li>Многократно используемые функции | Хорошо работает только для кратковременной обработки |
| Фабрика данных + пользовательский компонент | <li>Крупномасштабные параллельные вычисления<li>Подходит для интенсивных алгоритмов | <li>Требует переноса кода в исполняемый файл<li>Сложность обработки зависимостей и операций ввода-вывода |
| Фабрика данных и Azure Databricks Записная книжка |<li> Apache Spark<li>Собственная среда Python |<li>Может быть дорогостоящим<li>Создание кластеров изначально занимает некоторое время и увеличивает задержку

## <a name="azure-data-factory-with-azure-functions"></a>Фабрика данных Azure с помощью функций Azure

Функции Azure позволяют запускать небольшие фрагменты кода (функции), не беспокоясь о инфраструктуре приложений. В этом случае данные обрабатываются с помощью пользовательского кода Python, упакованного в функцию Azure. 

Функция вызывается с помощью [действия функции Azure фабрики данных Azure](../data-factory/control-flow-azure-function-activity.md). Этот подход является хорошим вариантом для упрощенных преобразований данных. 

![На схеме показан конвейер фабрики данных Azure с помощью функции Azure и запуска конвейера машинного обучения, а также конвейер Машинное обучение Azure, с обучением модели и их взаимодействия с необработанными данными и подготовленными данными.](media/how-to-data-ingest-adf/adf-function.png)



* Преимущества:
    * Данные обрабатываются в бессерверных расчетах с относительно низкой задержкой.
    * Конвейер фабрики данных может вызывать [устойчивую функцию Azure](../azure-functions/durable/durable-functions-overview.md) , которая может реализовать сложный поток преобразования данных. 
    * Сведения о преобразовании данных являются абстрактными для функции Azure, которую можно повторно использовать и вызывать из других мест.
* Недостатки
    * Функции Azure необходимо создать перед использованием с ADF
    * Функции Azure хороши только для кратковременной обработки данных.

## <a name="azure-data-factory-with-custom-component-activity"></a>Фабрика данных Azure с действием настраиваемых компонентов

В этом случае данные обрабатываются с помощью пользовательского кода Python, упакованного в исполняемый файл. Она вызывается с помощью [ действия настраиваемого компонента фабрики данных Azure](../data-factory/transform-data-using-dotnet-custom-activity.md). Этот подход лучше подходит для больших данных, чем предыдущий метод.

![На схеме показан конвейер фабрики данных Azure с пользовательским компонентом и запуск конвейера M L, а также конвейер Машинное обучение Azure, с обучением модели и способ взаимодействия с необработанными данными и подготовленными данными.](media/how-to-data-ingest-adf/adf-customcomponent.png)

* Преимущества:
    * Данные обрабатываются в пуле [пакетной службы Azure](../batch/batch-technical-overview.md) , который обеспечивает крупномасштабные параллельные и высокопроизводительные вычисления.
    * Может использоваться для запуска интенсивных алгоритмов и обработки значительных объемов данных.
* Недостатки:
    * Пул пакетной службы Azure необходимо создать перед использованием с фабрикой данных
    * Помимо проектирования, связанной с переносом кода Python в исполняемый файл. Сложность обработки зависимостей и входных и выходных параметров

## <a name="azure-data-factory-with-azure-databricks-python-notebook"></a>Фабрика данных Azure с записной книжкой Azure Databricks Python

[Azure Databricks](https://azure.microsoft.com/services/databricks/) — это платформа аналитики на основе Apache Spark в Microsoft Cloud.

В этом методе преобразование данных выполняется в [записной книжке Python](../data-factory/transform-data-using-databricks-notebook.md), которая выполняется в кластере Azure Databricks. Возможно, это самый распространенный подход, использующий всю мощь службы Azure Databricks. Он предназначен для распределенной обработки данных в масштабе.

![На схеме показан конвейер фабрики данных Azure с Azure Databricks Python и выполнением конвейера M L, а также конвейером Машинное обучение Azure, с обучением модели и способом взаимодействия с необработанными данными и подготовленными данными.](media/how-to-data-ingest-adf/adf-databricks.png)

* Преимущества:
    * Данные преобразуются в самую мощную службу Azure для обработки данных, резервная копия которой задается средой Apache Spark
    * Собственная поддержка Python вместе с платформами и библиотеками для обработки и анализа данных, включая TensorFlow, PyTorch и scikit-учиться
    * Нет необходимости упаковывать код Python в функции или исполняемые модули. Код работает как есть.
* Недостатки:
    * Перед использованием фабрики данных необходимо создать Azure Databricks инфраструктуру
    * Может быть дорогостоящей в зависимости от конфигурации Azure Databricks
    * Вращение кластеров вычислений из "холодного" режима занимает некоторое время, что приводит к высокой задержке решения. 
    

## <a name="consume-data-in-azure-machine-learning"></a>Использование данных в Машинное обучение Azure 

Конвейер фабрики данных сохраняет подготовленные данные в облачном хранилище (например, в большом двоичном объекте Azure или Azure Data Lake). <br>
Использование подготовленных данных в Машинное обучение Azure с помощью, 

* Вызов конвейера Машинное обучение Azure из конвейера фабрики данных.<br>**OR**
* Создание [машинное обучение Azure хранилища](how-to-access-data.md#create-and-register-datastores) [данных и набора машинное обучение Azure](how-to-create-register-datasets.md) для дальнейшего использования.

### <a name="invoke-azure-machine-learning-pipeline-from-data-factory"></a>Вызов конвейера Машинное обучение Azure из фабрики данных

Этот метод рекомендуется для [рабочих процессов машинное обучениеных операций (млопс)](concept-model-management-and-deployment.md#what-is-mlops). Если вы не хотите настраивать конвейер Машинное обучение Azure, см. [статью чтение данных непосредственно из хранилища](#read-data-directly-from-storage).

При каждом запуске конвейера фабрики данных 

1. Данные сохраняются в другом расположении в хранилище. 
1. Чтобы передать расположение в Машинное обучение Azure, конвейер фабрики данных вызывает [конвейер машинное обучение Azure](concept-ml-pipelines.md). При вызове конвейера машинного обучения расположение данных и идентификатор запуска отправляются в качестве параметров. 
1. Затем конвейер машинного обучения может создать Машинное обучение Azure хранилище данных и набор данных с расположением. Дополнительные сведения см. в статье [выполнение конвейеров машинное обучение Azure в фабрике данных](../data-factory/transform-data-machine-learning-service.md).

![На схеме показан конвейер фабрики данных Azure и конвейер Машинное обучение Azure, а также взаимодействие с необработанными данными и подготовленными данными. Конвейер фабрики данных передает данные в подготовленную базу данных, которая передает данные в хранилище данных, которые являются каналами в рабочей области Машинное обучение.](media/how-to-data-ingest-adf/aml-dataset.png)

> [!TIP]
> Наборы данных [поддерживают управление версиями](./how-to-version-track-datasets.md), поэтому конвейер машинного обучения может зарегистрировать новую версию DataSet, которая указывает на самые последние данные из конвейера ADF.

После доступа к данным через хранилище данных или набор данных их можно использовать для обучения модели машинного обучения. Процесс обучения может быть частью того же конвейера ML, который вызывается из ADF. Или это может быть отдельный процесс, например экспериментирование в записной книжке Jupyter.

Так как наборы данных поддерживают управление версиями, и каждый запуск из конвейера создает новую версию, легко понять, какая версия используется для обучения модели.

### <a name="read-data-directly-from-storage"></a>Чтение данных непосредственно из хранилища

Если вы не хотите создавать конвейер машинного обучения, вы можете получить доступ к данным непосредственно из учетной записи хранения, в которой подготовленные данные сохраняются с хранилищем данных Машинное обучение Azure и набором данных. 

В следующем коде Python показано, как создать хранилище данных, которое подключается к хранилищу Azure с поколением 2. Дополнительные [сведения о хранилищах данных и о том, где найти разрешения субъекта-службы](how-to-access-data.md#create-and-register-datastores).

```python
ws = Workspace.from_config()
adlsgen2_datastore_name = '<ADLS gen2 storage account alias>'  #set ADLS Gen2 storage account alias in AML

subscription_id=os.getenv("ADL_SUBSCRIPTION", "<ADLS account subscription ID>") # subscription id of ADLS account
resource_group=os.getenv("ADL_RESOURCE_GROUP", "<ADLS account resource group>") # resource group of ADLS account

account_name=os.getenv("ADLSGEN2_ACCOUNTNAME", "<ADLS account name>") # ADLS Gen2 account name
tenant_id=os.getenv("ADLSGEN2_TENANT", "<tenant id of service principal>") # tenant id of service principal
client_id=os.getenv("ADLSGEN2_CLIENTID", "<client id of service principal>") # client id of service principal
client_secret=os.getenv("ADLSGEN2_CLIENT_SECRET", "<secret of service principal>") # the secret of service principal

adlsgen2_datastore = Datastore.register_azure_data_lake_gen2(
    workspace=ws,
    datastore_name=adlsgen2_datastore_name,
    account_name=account_name, # ADLS Gen2 account name
    filesystem='<filesystem name>', # ADLS Gen2 filesystem
    tenant_id=tenant_id, # tenant id of service principal
    client_id=client_id, # client id of service principal
```

Затем создайте набор данных для ссылки на файлы, которые вы хотите использовать в задаче машинного обучения. 

Следующий код создает Табулардатасет из CSV-файла `prepared-data.csv` . Дополнительные сведения о [типах наборов данных и допустимых форматах файлов](how-to-create-register-datasets.md#dataset-types). 

```python
from azureml.core import Workspace, Datastore, Dataset
from azureml.core.experiment import Experiment
from azureml.train.automl import AutoMLConfig

# retrieve data via AML datastore
datastore = Datastore.get(ws, adlsgen2_datastore)
datastore_path = [(datastore, '/data/prepared-data.csv')]
        
prepared_dataset = Dataset.Tabular.from_delimited_files(path=datastore_path)
```

Здесь `prepared_dataset` можно использовать для ссылки на подготовленные данные, например в сценариях обучения. Узнайте, как [обучить модели с помощью наборов данных в машинное обучение Azure](./how-to-train-with-datasets.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Запуск записной книжки кирпичей в фабрике данных Azure](../data-factory/transform-data-using-databricks-notebook.md)
* [Доступ к данным в службах хранилища Azure](./how-to-access-data.md#create-and-register-datastores)
* [Обучение моделей с помощью наборов данных в машинное обучение Azure](./how-to-train-with-datasets.md).
* [DevOps для конвейера приема данных](./how-to-cicd-data-ingestion.md)
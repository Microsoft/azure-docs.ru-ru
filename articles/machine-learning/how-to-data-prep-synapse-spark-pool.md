---
title: Структурирование данных с помощью пулов Apache Spark (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как присоединить и запустить пулы Apache Spark для структурирование данных с помощью Azure синапсе Analytics и Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 03/02/2021
ms.custom: how-to, devx-track-python, data4ml, synapse-azureml
ms.openlocfilehash: 9ced4da7f71a0499e538e499644d89240611f1ea
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104956219"
---
# <a name="attach-apache-spark-pools-powered-by-azure-synapse-analytics-for-data-wrangling-preview"></a>Присоединение пулов Apache Spark (на платформе Azure синапсе Analytics) для структурирование данных (Предварительная версия)

Из этой статьи вы узнаете, как присоединить и запустить пул Apache Spark на платформе [Azure синапсе Analytics](/synapse-analytics/overview-what-is.md) для структурирование данных в масштабе. 

Эта статья содержит рекомендации по интерактивному выполнению задач структурирование данных в рамках выделенного сеанса синапсе в записной книжке Jupyter. Если вы предпочитаете использовать конвейеры Машинное обучение Azure, см. статью [использование Apache Spark (на платформе Azure синапсе Analytics) в конвейере машинного обучения (Предварительная версия)](how-to-use-synapsesparkstep.md).

>[!IMPORTANT]
> Машинное обучение Azure и интеграция Azure синапсе Analytics находятся на этапе предварительной версии. Возможности, представленные в этой статье, `azureml-synapse` применяют пакет, который содержит функции [экспериментальной](/python/api/overview/azure/ml/#stable-vs-experimental) предварительной версии, которые могут изменяться в любое время.

## <a name="azure-machine-learning-and-azure-synapse-analytics-integration-preview"></a>Интеграция Машинное обучение Azure и Azure синапсе Analytics (Предварительная версия)

Интеграция Azure синапсе Analytics с Машинное обучение Azure (Предварительная версия) позволяет подключать пул Apache Spark, поддерживаемый Azure синапсе для интерактивного исследования и подготовки данных. Благодаря этой интеграции вы можете использовать выделенные ресурсы для структурирование данных в том же блокноте Python, который используется для обучения моделей машинного обучения.

## <a name="prerequisites"></a>Предварительные условия

* [Создайте рабочую область Машинного обучения Azure](how-to-manage-workspace.md?tabs=python).

* [Создайте рабочую область Azure синапсе Analytics в портал Azure](../synapse-analytics/quickstart-create-workspace.md).

* [Создание пула Apache Spark с помощью портал Azure, веб-средств или синапсе Studio](../synapse-analytics/quickstart-create-apache-spark-pool-portal.md)

* [Настройте среду разработки](how-to-configure-environment.md) , чтобы установить пакет sdk для машинное обучение Azure, или используйте [машинное обучение Azure вычислительный экземпляр](concept-compute-instance.md#create) с уже установленным пакетом SDK. 

* Установите `azureml-synapse` пакет (Предварительная версия) со следующим кодом:

  ```python
  pip install azureml-synapse
  ```

* [Свяжите машинное обучение Azure рабочей области и рабочей области Azure синапсе Analytics](how-to-link-synapse-ml-workspaces.md).

## <a name="get-an-existing-linked-service"></a>Получение существующей связанной службы
Прежде чем присоединить выделенное вычисление для структурирование данных, необходимо иметь рабочую область ML, связанную с рабочей областью Azure синапсе Analytics, которая называется связанной службой. 

Для получения и использования существующей связанной службы требуются разрешения **пользователя или участника** рабочей области Azure синапсе Analytics.

Просмотрите все связанные службы, связанные с рабочей областью машинного обучения. 

```python
LinkedService.list(ws)
```

В этом примере из рабочей области извлекается существующая связанная служба `synapselink1` `ws` с [`get()`](/python/api/azureml-core/azureml.core.linkedservice#get-workspace--name-) методом.
```python
linked_service = LinkedService.get(ws, 'synapselink1')
```
 
## <a name="attach-synapse-spark-pool-as-a-compute"></a>Подключение пула синапсе Spark в качестве вычислений

После получения связанной службы Подключите пул Apache Spark синапсе в качестве выделенного ресурса вычислений для задач структурирование данных. 

Пулы Apache Spark можно подключать с помощью,
* Студия машинного обучения Azure.
* [Шаблоны Azure Resource Manager (ARM)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json)
* Пакет SDK для Python 

### <a name="attach-a-pool-via-the-studio"></a>Подключение пула с помощью студии
Выполните указанные ниже действия. 

1. Войдите в [машинное обучение Azure Studio](https://ml.azure.com/).
1. Выберите **связанные службы** в разделе **Управление** левой панели.
1. Выберите рабочую область синапсе.
1. Выберите **подключенные пулы Spark** в верхнем левом углу. 
1. Выберите **Подключить**. 
1. Выберите из списка Пул Apache Spark и укажите имя.  
    1. В этом списке указаны доступные пулы синапсе Spark, которые можно присоединить к вычислению. 
    1. Сведения о создании нового пула Spark синапсе см. [в статье Создание пула Apache Spark в синапсе Studio](../synapse-analytics/quickstart-create-apache-spark-pool-portal.md) .
1. Выберите пункт **присоединить выбранное**. 

### <a name="attach-a-pool-with-the-python-sdk"></a>Подключение пула с помощью пакета SDK для Python

Можно также использовать **пакет SDK для Python** , чтобы подключить пул Apache Spark. 

Следующий код 
1. Настраивает Синапсекомпуте с помощью,

   1. LinkedService, `linked_service` созданный или полученный на предыдущем шаге. 
   1. Тип целевого объекта вычислений, который необходимо присоединить, `SynapseSpark`
   1. Имя пула Apache Spark. Это должно совпадать с существующим пулом Apache Spark, который находится в рабочей области Azure синапсе Analytics.
   
1. Создает ComputeTarget машинного обучения, передавая, 
   1. Рабочая область машинного обучения, которую вы хотите использовать, `ws`
   1. Имя, которое вы хотите использовать для вычислений в рабочей области Машинное обучение Azure. 
   1. Attach_configuration, указанный при настройке вычислений синапсе.
       1. Вызов ComputeTarget. Attach () выполняется асинхронно, поэтому образец блокируется до завершения вызова.

```python
from azureml.core.compute import SynapseCompute, ComputeTarget

attach_config = SynapseCompute.attach_configuration(linked_service, #Linked synapse workspace alias
                                                    type='SynapseSpark', #Type of assets to attach
                                                    pool_name="<Synapse Spark pool name>") #Name of Synapse spark pool 

synapse_compute = ComputeTarget.attach(workspace= ws,                
                                       name="<Synapse Spark pool alias in Azure ML>", 
                                       attach_configuration=attach_config
                                      )

synapse_compute.wait_for_completion()
```

Проверьте, присоединен ли пул Apache Spark.

```python
ws.compute_targets['Synapse Spark pool alias']
```

## <a name="launch-synapse-spark-pool-for-data-preparation-tasks"></a>Запуск пула Spark синапсе для задач подготовки данных

Чтобы начать подготовку данных с помощью пула Apache Spark, укажите имя пула Apache Spark.

> [!IMPORTANT]
> Чтобы продолжить использование пула Apache Spark, необходимо указать, какой ресурс вычислений использовать в задачах структурирование данных, с помощью `%synapse` одной строки кода и `%%synapse` нескольких строк. 

```python
%synapse start -c SynapseSparkPoolAlias
```

После запуска сеанса можно проверить метаданные сеанса.

```python
%synapse meta
```

Вы можете указать [машинное обучение Azure среду](concept-environments.md) , которая будет использоваться во время сеанса Apache Spark. Вступят в силу будут только зависимости Conda, указанные в среде. Образ DOCKER не поддерживается.

>[!WARNING]
>  Зависимости Python, указанные в зависимости Conda окружения, не поддерживаются в пулах Apache Spark. В настоящее время поддерживаются только фиксированные версии Python. Проверьте версию Python, включив  `sys.version_info` в скрипт.

Следующий код создает среду, `myenv` которая устанавливает `azureml-core` версию 1.20.0 и `numpy` версию 1.17.0 до начала сеанса. Затем эту среду можно включить в инструкцию Apache Spark Session `start` .

```python

from azureml.core import Workspace, Environment

# creates environment with numpy and azureml-core dependencies
ws = Workspace.from_config()
env = Environment(name="myenv")
env.python.conda_dependencies.add_pip_package("azureml-core==1.20.0")
env.python.conda_dependencies.add_conda_package("numpy==1.17.0")
env.register(workspace=ws)
```

Чтобы начать подготовку данных с помощью пула Apache Spark и настраиваемой среды, укажите имя пула Apache Spark и среду, которая будет использоваться во время сеанса Apache Spark. Кроме того, можно указать идентификатор подписки, группу ресурсов рабочей области машинного обучения и имя рабочей области машинного обучения.

```python
%synapse start -c SynapseSparkPoolAlias -e myenv -s AzureMLworkspaceSubscriptionID -r AzureMLworkspaceResourceGroupName -w AzureMLworkspaceName
```
## <a name="load-data-from-storage"></a>Загрузка данных из хранилища

После запуска сеанса Apache Spark прочтите данные, которые вы хотите подготовить. Загрузка данных поддерживается для хранилища BLOB-объектов Azure и Azure Data Lake Storage поколений 1 и 2.

Существует два способа загрузки данных из следующих служб хранилища: 

* Непосредственная загрузка данных из хранилища с помощью пути к распределенным файлам Hadoop (HDFS).

* Чтение данных из существующего [машинное обучение Azure набора данных](how-to-create-register-datasets.md).

Для доступа к этим службам хранилища требуются разрешения **читателя данных BLOB-объекта хранилища** . Если вы планируете записывать данные обратно в эти службы хранилища, необходимы разрешения **участника данных BLOB-объекта хранилища** . [Дополнительные сведения о разрешениях и ролях хранилища](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues).

### <a name="load-data-with-hadoop-distributed-files-system-hdfs-path"></a>Загрузка данных с помощью пути к распределенным файлам Hadoop (HDFS)

Для загрузки и чтения данных из хранилища с соответствующим путем HDFS необходимо иметь доступ к учетным данным для проверки подлинности. Эти учетные данные различаются в зависимости от типа хранилища.  

В следующем коде показано, как считывать данные из **хранилища BLOB-объектов Azure** в кадр данных Spark с помощью токена подписанного URL-адрес (SAS) или ключа доступа. 

```python
%%synapse

# setup access key or SAS token
sc._jsc.hadoopConfiguration().set("fs.azure.account.key.<storage account name>.blob.core.windows.net", "<access key>")
sc._jsc.hadoopConfiguration().set("fs.azure.sas.<container name>.<storage account name>.blob.core.windows.net", "<sas token>")

# read from blob 
df = spark.read.option("header", "true").csv("wasbs://demo@dprepdata.blob.core.windows.net/Titanic.csv")
```

В следующем коде показано, как считывать данные из **Azure Data Lake Storage поколения 1 (ADLS Gen 1)** с учетными данными субъекта-службы. 

```python

# setup service principal which has access of the data
sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.access.token.provider.type","ClientCredential")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.client.id", "<client id>")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.credential", "<client secret>")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.refresh.url",
https://login.microsoftonline.com/<tenant id>/oauth2/token)

df = spark.read.csv("adl://<storage account name>.azuredatalakestore.net/<path>")

```

В следующем коде показано, как считывать данные из **Azure Data Lake Storage поколения 2 (ADLS Gen 2)** с учетными данными субъекта-службы. 

```python
# setup service principal which has access of the data
sc._jsc.hadoopConfiguration().set("fs.azure.account.auth.type.<storage account name>.dfs.core.windows.net","OAuth")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth.provider.type.<storage account name>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.id.<storage account name>.dfs.core.windows.net", "<client id>")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.secret.<storage account name>.dfs.core.windows.net", "<client secret>")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.endpoint.<storage account name>.dfs.core.windows.net",
https://login.microsoftonline.com/<tenant id>/oauth2/token)


df = spark.read.csv("abfss://<container name>@<storage account>.dfs.core.windows.net/<path>")

```

### <a name="read-in-data-from-registered-datasets"></a>Считывает данные из зарегистрированных наборов данных

Вы также можете получить имеющийся зарегистрированный набор данных в рабочей области и выполнить для него подготовку данных, преобразовав ее в таблицу данных Spark.

В следующем примере выполняется проверка подлинности в рабочей области, получение зарегистрированного Табулардатасет, `blob_dset` который ссылается на файлы в хранилище BLOB-объектов, и преобразует его в кадр данных Spark. При преобразовании наборов данных в таблицу данных Spark можно использовать `pyspark` библиотеки для изучения и подготовки.  

``` python

%%synapse
from azureml.core import Workspace, Dataset

subscription_id = "<enter your subscription ID>"
resource_group = "<enter your resource group>"
workspace_name = "<enter your workspace name>"

ws = Workspace(workspace_name = workspace_name,
               subscription_id = subscription_id,
               resource_group = resource_group)

dset = Dataset.get_by_name(ws, "blob_dset")
spark_df = dset.to_spark_dataframe()
```

## <a name="perform-data-wrangling-tasks"></a>Выполнение задач структурирование данных

После извлечения и изучения данных можно выполнять задачи структурирование данных.

Следующий код разворачивается по примеру HDFS в предыдущем разделе и фильтрует данные в таблице данных Spark, `df` на основе столбца и групп **выжившую»** , которые перечислены по **возрасту** .

```python
%%synapse
from pyspark.sql.functions import col, desc

df.filter(col('Survived') == 1).groupBy('Age').count().orderBy(desc('count')).show(10)

df.show()

```

## <a name="save-data-to-storage-and-stop-spark-session"></a>Сохранение данных в хранилище и завершение сеанса Spark

После завершения изучения и подготовки данных храните подготовленные данные для последующего использования в учетной записи хранения в Azure.

В следующем примере подготовленные данные записываются обратно в хранилище BLOB-объектов Azure и перезаписывают исходный `Titanic.csv` файл в `training_data` каталоге. Для обратной записи в хранилище требуются разрешения **участника данных BLOB-объекта хранилища** . [Дополнительные сведения о разрешениях и ролях хранилища](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues).

```python
%% synapse

df.write.format("csv").mode("overwrite").save("wasbs://demo@dprepdata.blob.core.windows.net/training_data/Titanic.csv")
```

Когда вы завершите подготовку данных и сохранили подготовленные данные в хранилище, завершите использование пула Apache Spark с помощью следующей команды.

```python
%synapse stop
```

## <a name="create-dataset-to-represent-prepared-data"></a>Создание набора данных для представления подготовленных данных

Когда вы будете готовы использовать подготовленные данные для обучения модели, подключитесь к хранилищу с помощью [хранилища данных машинное обучение Azure](how-to-access-data.md)и укажите, какие файлы нужно использовать с [набором машинное обучение Azure](how-to-create-register-datasets.md).

Следующий пример кода:

* Предполагается, что хранилище данных, которое подключается к службе хранилища, в которой были сохранены подготовленные данные, уже создано.  
* Возвращает существующее хранилище данных `mydatastore` из рабочей области `ws` с помощью метода Get ().
* Создает [филедатасет](how-to-create-register-datasets.md#filedataset), `train_ds` который ссылается на подготовленные файлы данных, расположенные в `training_data` каталоге в `mydatastore` .  
* Создает переменную `input1` , которую можно использовать позднее, чтобы сделать файлы данных `train_ds` набора данных доступными для целевого объекта вычислений.

```python
from azureml.core import Datastore, Dataset

datastore = Datastore.get(ws, datastore_name='mydatastore')

datastore_paths = [(datastore, '/training_data/')]
train_ds = Dataset.File.from_files(path=datastore_paths, validate=True)
input1 = train_ds.as_mount()

```

## <a name="example-notebook"></a>Пример записной книжки

Подробный пример того, как выполнить подготовку данных и обучение модели из одной записной книжки с помощью Azure синапсе Analytics и Машинное обучение Azure, см. в этой [сквозной записной книжке](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-synapse/spark_job_on_synapse_spark_pool.ipynb) .

## <a name="next-steps"></a>Дальнейшие действия

* [Обучение модели](how-to-set-up-training-targets.md).
* [Обучение с помощью набора данных Машинное обучение Azure](how-to-train-with-datasets.md)

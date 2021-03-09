---
title: Отслеживание Млфлов для экспериментов Azure Databricks ML
titleSuffix: Azure Machine Learning
description: Настройте Млфлов с Машинное обучение Azure, чтобы регистрировать метрики и артефакты Azure Databricks экспериментов машинного обучения.
services: machine-learning
author: nibaccam
ms.author: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 09/22/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: a894a46ce7c78b65dde80c52f9516fcbe4d27bcb
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521005"
---
# <a name="track-azure-databricks-ml-experiments-with-mlflow-and-azure-machine-learning-preview"></a>Следите за экспериментами Azure Databricks ML с помощью Млфлов и Машинное обучение Azure (Предварительная версия)

Из этой статьи вы узнаете, как включить Млфлов для отслеживания URI и API ведения журналов, который называется [отслеживанием млфлов](https://mlflow.org/docs/latest/quickstart.html#using-the-tracking-api), для подключения экспериментов Azure DATABRICKS (ADB), млфлов и машинное обучение Azure.

[MLflow](https://www.mlflow.org) — это библиотека с открытым кодом для управления жизненным циклом экспериментов машинного обучения. Отслеживание Млфлов — это компонент Млфлов, который регистрирует и отслеживает метрики и артефакты обучающего запуска. Дополнительные сведения о [Azure Databricks и млфлов](/azure/databricks/applications/mlflow/). 

Дополнительные сведения об интеграции функций Млфлов и Машинное обучение Azure см. [в статье мониторинг запуска экспериментов с помощью млфлов и машинное обучение Azure](how-to-use-mlflow.md) .

>[!NOTE]
> Как библиотека с открытым исходным кодом, Млфлов часто меняются. Таким образом, функции, предоставляемые через Машинное обучение Azure и интеграцию с Млфлов, следует рассматривать как предварительную версию, а не полностью поддерживать корпорацией Майкрософт.

> [!TIP]
> Сведения в этом документе предназначены главным образом для специалистов по обработке и анализу данных и разработчиков, желающих отслеживать процесс обучения модели. Если вы являетесь администратором, который заинтересован в наблюдении из Машинного обучения Azure за использованием ресурсов и событиями, такими как квоты, завершенные обучающие запуски или завершенные развертывания моделей, ознакомьтесь с разделом [Мониторинг Машинного обучения Azure](monitor-azure-machine-learning.md).

## <a name="prerequisites"></a>Предварительные условия

* Установите пакет `azureml-mlflow`. 
    * Этот пакет автоматически переносится в пакет `azureml-core` [SDK для машинное обучение Azure Python](/python/api/overview/azure/ml/install), который обеспечивает подключение для млфлов к рабочей области.
* [Рабочая область и кластер Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal).
* [Создайте рабочую область Машинного обучения Azure](how-to-manage-workspace.md).
    * Узнайте, какие [разрешения доступа требуются для выполнения операций млфлов с рабочей областью](how-to-assign-roles.md#mlflow-operations).

## <a name="track-azure-databricks-runs"></a>Отслеживание выполнений Azure Databricks

Отслеживание Млфлов с помощью Машинное обучение Azure позволяет хранить зарегистрированные метрики и артефакты из Azure Databricks выполняется как в: 

* Рабочая область Azure Databricks.
* Рабочая область службы "Машинное обучение Azure"

После создания рабочей области и кластера Azure Databricks 

1. Установите библиотеку *azureml-млфлов* из PyPi, чтобы предоставить кластеру доступ к необходимым функциям и классам.

1. Настройте записную книжку эксперимента.

1. Подключите рабочую область Azure Databricks и Машинное обучение Azure рабочую область.

Дополнительные сведения об этих действиях приведены в следующих разделах, в которых можно успешно запустить эксперименты Млфлов с Azure Databricks. 

## <a name="install-libraries"></a>Установка библиотек

Чтобы установить библиотеки в кластере, перейдите на вкладку **библиотеки** и выберите **установить новый** .

 ![млфлов с модулями Azure](./media/how-to-use-mlflow-azure-databricks/azure-databricks-cluster-libraries.png)

В поле **пакет** введите azureml-млфлов и нажмите кнопку установить. Повторите этот шаг, чтобы установить другие дополнительные пакеты в кластер для своего эксперимента, если это требуется.

 ![Библиотека млфлов для установки базы данных Azure](./media/how-to-use-mlflow-azure-databricks/install-libraries.png)

## <a name="set-up-your-notebook"></a>Настройка записной книжки 

После настройки кластера ADB 
1. В области навигации слева выберите **рабочие области** . 
1. Разверните раскрывающееся меню рабочие области и выберите **Импорт** .
1. Перетащите или найдите записную книжку своего эксперимента, чтобы импортировать рабочую область ADB.
1. Выберите **Импортировать**. Записная книжка эксперимента откроется автоматически.
1. Под заголовком записной книжки в верхнем левом углу выберите кластер, который нужно подключить к записной книжке эксперимента. 

## <a name="connect-your-azure-databricks-and-azure-machine-learning-workspaces"></a>Связывание рабочих областей Azure Databricks и Машинного обучения Azure

Связывание рабочей области ADB с рабочей областью Машинное обучение Azure позволяет отслеживанию данных экспериментов в рабочей области Машинное обучение Azure.

Чтобы связать рабочую область ADB с новой или существующей рабочей областью Машинное обучение Azure, 
1. Войдите на [портал Azure](https://ms.portal.azure.com).
1. Перейдите на страницу **обзора** рабочей области ADB.
1. Нажмите кнопку **Link машинное обучение Azure Workspace (связать с рабочей областью** ) в правом нижнем углу. 

 ![Связывание рабочих областей Azure DB и Машинное обучение Azure](./media/how-to-use-mlflow-azure-databricks/link-workspaces.png)

## <a name="mlflow-tracking-in-your-workspaces"></a>Отслеживание Млфлов в рабочих областях

После создания экземпляра рабочей области отслеживание Млфлов автоматически задается для отслеживания в следующих местах:

* Связанная Рабочая область Машинное обучение Azure.
* Исходная Рабочая область ADB. 

Все ваши эксперименты в управляемой службе отслеживания Машинное обучение Azure.

Следующий код должен находиться в записной книжке эксперимента, чтобы получить связанную рабочую область Машинное обучение Azure. 

Этот код, 

*  Получение сведений о подписке Azure для создания экземпляра рабочей области Машинное обучение Azure. 

* Предполагается, что у вас есть группа ресурсов и Рабочая область Машинное обучение Azure. в противном случае ее можно [создать](how-to-manage-workspace.md). 

* Задает имя эксперимента. Это `user_name` согласуется с тем, который `user_name` связан с рабочей областью Azure Databricks.

```python
import mlflow
import mlflow.azureml
import azureml.mlflow
import azureml.core

from azureml.core import Workspace

subscription_id = 'subscription_id'

# Azure Machine Learning resource group NOT the managed resource group
resource_group = 'resource_group_name' 

#Azure Machine Learning workspace name, NOT Azure Databricks workspace
workspace_name = 'workspace_name'  

# Instantiate Azure Machine Learning workspace
ws = Workspace.get(name=workspace_name,
                   subscription_id=subscription_id,
                   resource_group=resource_group)

#Set MLflow experiment. 
experimentName = "/Users/{user_name}/{experiment_folder}/{experiment_name}" 
mlflow.set_experiment(experimentName) 

```

### <a name="set-mlflow-tracking-to-only-track-in-your-azure-machine-learning-workspace"></a>Настройка отслеживания Млфлов для отслеживания только в рабочей области Машинное обучение Azure

Если вы предпочитаете управлять отслеживаемыми экспериментами в централизованном расположении, можно настроить отслеживание Млфлов для отслеживания **только** в рабочей области машинное обучение Azure. 

Включите в скрипт следующий код:

```python
uri = ws.get_mlflow_tracking_uri()
mlflow.set_tracking_uri(uri)
```

Импортируйте `mlflow` в сценарий обучения, чтобы использовать интерфейсы API ведения журнала MLflow, и начните вести журнал метрик выполнения. В следующем примере записывается метрика потерь эпохи. 

```python
import mlflow 
mlflow.log_metric('epoch_loss', loss.item()) 
```

## <a name="register-models-with-mlflow"></a>Регистрация моделей с помощью Млфлов

После обучения модели можно регистрировать и регистрировать модели на сервере отслеживания серверной части с помощью `mlflow.<model_flavor>.log_model()` метода. `<model_flavor>`, ссылается на платформу, связанную с моделью. [Узнайте, какие разновидности модели поддерживаются](https://mlflow.org/docs/latest/models.html#model-api). 

Сервер отслеживания внутреннего сервера по умолчанию является рабочей областью Azure Databricks; Если вы не решили [установить отслеживание млфлов только в рабочей области машинное обучение Azure](#set-mlflow-tracking-to-only-track-in-your-azure-machine-learning-workspace), сервер отслеживания внутреннего сервера является рабочей областью машинное обучение Azure.   

* **Если Зарегистрированная модель с таким именем не существует**, метод регистрирует новую модель, создает версию 1 и возвращает объект моделверсион млфлов. 

* **Если Зарегистрированная модель с таким именем уже существует**, метод создает новую версию модели и возвращает объект Version. 

```python
mlflow.spark.log_model(model, artifact_path = "model", 
                       registered_model_name = 'model_name')  

mlflow.sklearn.log_model(model, artifact_path = "model", 
                         registered_model_name = 'model_name') 
```

## <a name="create-endpoints-for-mlflow-models"></a>Создание конечных точек для моделей Млфлов

Когда вы будете готовы создать конечную точку для моделей машинного обучения. Можно развернуть как, 

* Веб-служба Машинное обучение Azure Request-Response для интерактивного оценки. Это развертывание позволяет использовать и применять Машинное обучение Azure управления моделями, а также возможности обнаружения отклонения данных в рабочих моделях. 

* Объекты модели Млфлов, которые могут использоваться в потоковых или пакетных конвейерах как функции Python или Pandas UDF в Azure Databricks рабочей области.

### <a name="deploy-models-to-azure-machine-learning-endpoints"></a>Развертывание моделей в Машинное обучение Azure конечных точках 
Вы можете использовать API [млфлов. azureml. deploy](https://www.mlflow.org/docs/latest/python_api/mlflow.azureml.html#mlflow.azureml.deploy) для развертывания модели в рабочей области машинное обучение Azure. Если вы зарегистрировали модель только в рабочей области Azure Databricks, как описано в разделе [Регистрация моделей с помощью млфлов](#register-models-with-mlflow) , укажите `model_name` параметр для регистрации модели в машинное обучение Azure рабочей области. 

Azure Databricksные запуски можно развернуть в следующих конечных точках. 
* [Экземпляр контейнера Azure](how-to-deploy-mlflow-models.md#deploy-to-azure-container-instance-aci)
* [Служба Azure Kubernetes](how-to-deploy-mlflow-models.md#deploy-to-azure-kubernetes-service-aks)

### <a name="deploy-models-to-adb-endpoints-for-batch-scoring"></a>Развертывание моделей в конечных точках ADB для пакетной оценки 

Для пакетной оценки можно выбрать Azure Databricks кластеры. Модель Млфлов загружается и используется в качестве пользовательской функции Spark Pandas для оценки новых данных. 

```python
from pyspark.sql.types import ArrayType, FloatType 

model_uri = "runs:/"+last_run_id+ {model_path} 

#Create a Spark UDF for the MLFlow model 

pyfunc_udf = mlflow.pyfunc.spark_udf(spark, model_uri) 

#Load Scoring Data into Spark Dataframe 

scoreDf = spark.table({table_name}).where({required_conditions}) 


#Make Prediction 

preds = (scoreDf 

           .withColumn('target_column_name', pyfunc_udf('Input_column1', 'Input_column2', ' Input_column3', …)) 

        ) 

display(preds) 
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не планируете использовать зарегистрированные метрики и артефакты в рабочей области, возможность удалить их по отдельности в данный момент недоступна. Вместо этого удалите группу ресурсов, содержащую учетную запись хранения и рабочую область, чтобы не взиматься плата.

1. На портале Azure выберите **Группы ресурсов** в левой части окна.

   ![Удаление ресурсов на портале Azure](./media/how-to-use-mlflow-azure-databricks/delete-resources.png)

1. В списке выберите созданную группу ресурсов.

1. Выберите **Удалить группу ресурсов**.

1. Введите имя группы ресурсов. Теперь щелкните **Удалить**.

## <a name="example-notebooks"></a>Примеры записных книжек

В [млфлов машинное обучение Azure с записными книжками](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow) демонстрируются и развертываются концепции, представленные в этой статье.

## <a name="next-steps"></a>Дальнейшие действия
* [Развертывание моделей млфлов в качестве веб-службы Azure](how-to-deploy-mlflow-models.md). 
* [Управляйте своими моделями](concept-model-management-and-deployment.md).
* [Следите за запуском экспериментов с помощью млфлов и машинное обучение Azure](how-to-use-mlflow.md). 
* Дополнительные сведения о [Azure Databricks и млфлов](/azure/databricks/applications/mlflow/).
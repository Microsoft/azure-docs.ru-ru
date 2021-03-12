---
title: Настройка целевых объектов для обучения & вывода
titleSuffix: Azure Machine Learning
description: Добавление в рабочую область ресурсов вычислений (целевых объектов вычислений) для использования при обучении машинного обучения и выводе
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 10/02/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperf-fy21q1
ms.openlocfilehash: 9fa6a1758bc2e2a76291efc3bb239c5249a6e21e
ms.sourcegitcommit: 6776f0a27e2000fb1acb34a8dddc67af01ac14ac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103149347"
---
# <a name="set-up-compute-targets-for-model-training-and-deployment"></a>Настройка целевых объектов вычислений для обучения и развертывания модели

Узнайте, как подключить ресурсы вычислений Azure к рабочей области Машинное обучение Azure.  Затем эти ресурсы можно использовать в качестве [целевых](concept-compute-target.md) показателей для обучения и вывода в задачах машинного обучения.

В этой статье вы узнаете, как настроить рабочую область для использования этих ресурсов вычислений:

* Локальный компьютер
* Удаленные виртуальные машины
* Azure HDInsight
* Пакетная служба Azure
* Azure Databricks
* Аналитика озера данных Azure
* Экземпляр контейнера Azure

Чтобы использовать целевые объекты вычислений, управляемые с помощью Машинное обучение Azure, см.:


* [Вычислительная операция Машинного обучения Azure](how-to-create-manage-compute-instance.md)
* [Вычислительный кластер Машинного обучения Azure](how-to-create-attach-compute-cluster.md)
* [Кластер службы Kubernetes Azure](how-to-create-attach-kubernetes.md)

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

* [Расширение Azure CLI для службы машинное обучение](reference-azure-machine-learning-cli.md), [машинное обучение Azure пакет SDK для Python](/python/api/overview/azure/ml/intro)или [расширение машинное обучение Azure Visual Studio Code](tutorial-setup-vscode-extension.md).

## <a name="limitations"></a>Ограничения

* **Не создавайте несколько одновременных вложений для одного и того же вычислений** из рабочей области. Например, можно подключить один кластер службы Azure Kubernetes к рабочей области, используя два разных имени. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.

    Если необходимо повторно подключить целевой объект вычислений, например для изменения конфигурации TLS или другого кластера, необходимо сначала удалить существующее вложение.

## <a name="whats-a-compute-target"></a>Что такое целевой объект вычислений?

С помощью решения "Машинное обучение Azure" вы можете обучать модель, используя разные вычислительные ресурсы или среды, которые вместе называются [__целевыми объектами вычислений__](concept-azure-machine-learning-architecture.md#compute-targets). Они могут быть локальными или облачными. Например, это может быть Вычислительная среда Машинного обучения Azure, Azure HDInsight или удаленная виртуальная машина.  Вы также используете целевые объекты вычислений для развертывания модели, как описано в разделе ["где и как развертывать модели"](how-to-deploy-and-where.md).


## <a name="local-computer"></a><a id="local"></a>Локальный компьютер

При использовании локального компьютера для **обучения** нет необходимости создавать целевой объект вычислений.  Просто [отправьте обучающий запуск](how-to-set-up-training-targets.md) с локального компьютера.

При использовании локального компьютера для **вывода** необходимо установить DOCKER. Чтобы выполнить развертывание, используйте [LocalWebservice.deploy_configuration ()](/python/api/azureml-core/azureml.core.webservice.local.localwebservice#deploy-configuration-port-none-) , чтобы определить порт, который будет использовать веб-служба. Затем используйте нормальный процесс развертывания, как описано в разделе [Развертывание моделей с помощью машинное обучение Azure](how-to-deploy-and-where.md).

## <a name="remote-virtual-machines"></a><a id="vm"></a>Удаленные виртуальные машины

Машинное обучение Azure также поддерживает подключение виртуальной машины Azure. Виртуальная машина должна быть виртуальной машиной Azure для обработки и анализа данных (DSVM). Эта виртуальная машина является предварительно настроенной обработкой и анализом данных и средой разработки искусственного интеллекта в Azure. Виртуальная машина предлагает выбор средств и платформ для разработки машинного обучения в течение всего жизненного цикла. Дополнительных сведения об использовании DSVM со Службой машинного обучения Azure см. в статье [Настройка среды разработки для Машинного обучения Azure](./how-to-configure-environment.md#dsvm).

1. **Создание**. Создайте DSVM перед ее использованием для обучения модели. Подробные сведения о создании этого ресурса см. в разделе [Подготовка виртуальной машины Linux (Ubuntu) для обработки и анализа данных](./data-science-virtual-machine/dsvm-ubuntu-intro.md).

    > [!WARNING]
    > Машинное обучение Azure поддерживает только виртуальные машины под управлением **Ubuntu**. Поэтому создаваемая виртуальная или существующая виртуальная машина, которую вы выбираете, должна работать под управлением Ubuntu.
    > 
    > Для Машинное обучение Azure также требуется, чтобы виртуальная машина соимела __общедоступный IP-адрес__.

1. **Присоединение**. Чтобы присоединить имеющуюся виртуальную машину как целевой объект вычислений, необходимо указать для нее ИД ресурса, имя входа и пароль. Идентификатор ресурса виртуальной машины можно создать с помощью идентификатора подписки, имени группы ресурсов и имени виртуальной машины, используя следующий строковый формат: `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Compute/virtualMachines/<vm_name>`

 
   ```python
   from azureml.core.compute import RemoteCompute, ComputeTarget

   # Create the compute config 
   compute_target_name = "attach-dsvm"
   
   attach_config = RemoteCompute.attach_configuration(resource_id='<resource_id>',
                                                   ssh_port=22,
                                                   username='<username>',
                                                   password="<password>")

   # Attach the compute
   compute = ComputeTarget.attach(ws, compute_target_name, attach_config)

   compute.wait_for_completion(show_output=True)
   ```

   Кроме того, можно присоединить DSVM к рабочей области, [используя Студию машинного обучения Azure](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > Не создавайте несколько одновременных вложений для одного и того же DSVM из рабочей области. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.

1. **Настройка**. Создайте конфигурацию запуска для целевого объекта вычислений DSVM. Для создания и настройки среды обучения на DSVM используются Docker и Conda.

   ```python
   from azureml.core import ScriptRunConfig
   from azureml.core.environment import Environment
   from azureml.core.conda_dependencies import CondaDependencies
   
   # Create environment
   myenv = Environment(name="myenv")
   
   # Specify the conda dependencies
   myenv.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
   
   # If no base image is explicitly specified the default CPU image "azureml.core.runconfig.DEFAULT_CPU_IMAGE" will be used
   # To use GPU in DSVM, you should specify the default GPU base Docker image or another GPU-enabled image:
   # myenv.docker.enabled = True
   # myenv.docker.base_image = azureml.core.runconfig.DEFAULT_GPU_IMAGE
   
   # Configure the run configuration with the Linux DSVM as the compute target and the environment defined above
   src = ScriptRunConfig(source_directory=".", script="train.py", compute_target=compute, environment=myenv) 
   ```

## <a name="azure-hdinsight"></a><a id="hdinsight"></a>Azure HDInsight 

Azure HDInsight — это популярная платформа для анализа больших данных. Она предоставляет Apache Spark, который можно использовать для обучения модели.

1. **Создание**.  Прежде чем использовать кластер HDInsight для обучения модели, создайте его. Инструкции по созданию Spark в кластере HDInsight см. в статье [Краткое руководство. Создание кластера Apache Spark в HDInsight с помощью шаблона](../hdinsight/spark/apache-spark-jupyter-spark-sql.md). 

    > [!WARNING]
    > Для Машинное обучение Azure требуется, чтобы кластер HDInsight имел __общедоступный IP-адрес__.

    При создании кластера необходимо указать имя пользователя SSH и пароль. Запишите эти значения — они вам потребуются при использовании HDInsight в качестве целевого объекта вычислений.
    
    После создания кластера подключитесь к нему с помощью имени узла \<clustername> -SSH.azurehdinsight.NET, где \<clustername> — это имя, указанное для кластера. 

1. **Присоединение**. Чтобы присоединить кластер HDInsight в качестве целевого объекта вычислений, необходимо указать ИД ресурса, имя пользователя и пароль для кластера HDInsight. Идентификатор ресурса кластера HDInsight можно создать с помощью идентификатора подписки, имени группы ресурсов и имени кластера HDInsight, используя следующий строковый формат: `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.HDInsight/clusters/<cluster_name>`

    ```python
   from azureml.core.compute import ComputeTarget, HDInsightCompute
   from azureml.exceptions import ComputeTargetException

   try:
    # if you want to connect using SSH key instead of username/password you can provide parameters private_key_file and private_key_passphrase

    attach_config = HDInsightCompute.attach_configuration(resource_id='<resource_id>',
                                                          ssh_port=22, 
                                                          username='<ssh-username>', 
                                                          password='<ssh-pwd>')
    hdi_compute = ComputeTarget.attach(workspace=ws, 
                                       name='myhdi', 
                                       attach_configuration=attach_config)

   except ComputeTargetException as e:
    print("Caught = {}".format(e.message))

   hdi_compute.wait_for_completion(show_output=True)
   ```

   Кроме того, можно присоединить кластер HDInsight к рабочей области, [используя Студию машинного обучения Azure](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > Не создавайте несколько одновременных вложений в одном HDInsight из рабочей области. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.

1. **Настройка**. Создайте конфигурацию запуска для целевого объекта вычислений HDI. 

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/hdi.py?name=run_hdi)]


Теперь, после подключения вычислительной среды и настройки запуска, следующий шаг — [отправить запрос на запуск обучения](how-to-set-up-training-targets.md).

## <a name="azure-batch"></a><a id="azbatch"></a>Пакетная служба Azure 

Пакетная служба Azure используется для эффективного запуска приложений для крупномасштабных параллельных и высокопроизводительных вычислений (HPC) в облаке. В конвейере Машинного обучения Azure для отправки заданий в пул виртуальных машин пакетной службы Azure можно использовать AzureBatchStep.

Чтобы присоединить пакетную службу Azure в качестве целевого объекта вычислений, используйте пакет SDK Машинного обучения Azure и укажите следующие сведения.

-    **Azure Batch compute name** (Имя пакетного вычисления пакетной службы Azure). Понятное имя, используемое для вычислений в рабочей области.
-    **Имя учетной записи для пакетной службы Azure**. Имя учетной записи пакетной службы Azure.
-    **Группа ресурсов**. Группа ресурсов, в которой содержится учетная запись пакетной службы Azure.

Следующий код демонстрирует, как присоединить пакетную службу в качестве целевого объекта вычислений.

```python
from azureml.core.compute import ComputeTarget, BatchCompute
from azureml.exceptions import ComputeTargetException

# Name to associate with new compute in workspace
batch_compute_name = 'mybatchcompute'

# Batch account details needed to attach as compute to workspace
batch_account_name = "<batch_account_name>"  # Name of the Batch account
# Name of the resource group which contains this account
batch_resource_group = "<batch_resource_group>"

try:
    # check if the compute is already attached
    batch_compute = BatchCompute(ws, batch_compute_name)
except ComputeTargetException:
    print('Attaching Batch compute...')
    provisioning_config = BatchCompute.attach_configuration(
        resource_group=batch_resource_group, account_name=batch_account_name)
    batch_compute = ComputeTarget.attach(
        ws, batch_compute_name, provisioning_config)
    batch_compute.wait_for_completion()
    print("Provisioning state:{}".format(batch_compute.provisioning_state))
    print("Provisioning errors:{}".format(batch_compute.provisioning_errors))

print("Using Batch compute:{}".format(batch_compute.cluster_resource_id))
```

> [!WARNING]
> Не создавайте несколько одновременных вложений в один пакет Azure из рабочей области. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.

### <a name="azure-databricks"></a><a id="databricks"></a>Azure Databricks

Azure Databricks — это среда, которая лежит в основе Apache Spark в облаке Azure. Ее можно использовать как целевой объект вычислений с помощью конвейера Машинного обучения Azure.

Прежде чем ее использовать, создайте рабочую область Azure Databricks. Чтобы создать ресурс рабочей области, см. статью [Запуск задания Spark в Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal) документе.

Введите следующие данные, чтобы вложить Azure Databricks в качестве целевого объекта вычислений.

* __Имя для вычислений для кирпичей__. имя, которое вы хотите назначить этому ресурсу вычислений.
* __Имя рабочей области кирпичей__ данных: имя рабочей области Azure Databricks.
* __Маркер доступа к модулям__. маркер доступа используется для проверки подлинности в Azure Databricks. Чтобы создать маркер доступа, см. документ [Проверка подлинности](/azure/databricks/dev-tools/api/latest/authentication).

В следующем примере кода показано, как присоединить Azure Databricks в качестве целевого объекта вычислений с Машинное обучение Azureным пакетом SDK (__Рабочая область "кирпичы" должна находиться в той же подписке, что и Рабочая область AML__):

```python
import os
from azureml.core.compute import ComputeTarget, DatabricksCompute
from azureml.exceptions import ComputeTargetException

databricks_compute_name = os.environ.get(
    "AML_DATABRICKS_COMPUTE_NAME", "<databricks_compute_name>")
databricks_workspace_name = os.environ.get(
    "AML_DATABRICKS_WORKSPACE", "<databricks_workspace_name>")
databricks_resource_group = os.environ.get(
    "AML_DATABRICKS_RESOURCE_GROUP", "<databricks_resource_group>")
databricks_access_token = os.environ.get(
    "AML_DATABRICKS_ACCESS_TOKEN", "<databricks_access_token>")

try:
    databricks_compute = ComputeTarget(
        workspace=ws, name=databricks_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('databricks_compute_name {}'.format(databricks_compute_name))
    print('databricks_workspace_name {}'.format(databricks_workspace_name))
    print('databricks_access_token {}'.format(databricks_access_token))

    # Create attach config
    attach_config = DatabricksCompute.attach_configuration(resource_group=databricks_resource_group,
                                                           workspace_name=databricks_workspace_name,
                                                           access_token=databricks_access_token)
    databricks_compute = ComputeTarget.attach(
        ws,
        databricks_compute_name,
        attach_config
    )

    databricks_compute.wait_for_completion(True)
```

Более подробный пример см. в [примере записной книжки](https://aka.ms/pl-databricks) на сайте GitHub.

> [!WARNING]
> Не создавайте несколько одновременных вложений для одного и того же Azure Databricks из рабочей области. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.

### <a name="azure-data-lake-analytics"></a><a id="adla"></a>Аналитика озера данных Azure

Azure Data Lake Analytics — это платформа аналитики больших данных в облаке Azure. Ее можно использовать как целевой объект вычислений с помощью конвейера Машинного обучения Azure.

Прежде чем ее использовать, создайте учетную запись Azure Data Lake Analytics. Чтобы создать этот ресурс, см. статью [Начало работы с Azure Data Lake Analytics с помощью портала Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md).

Чтобы вложить Azure Data Lake Analytics в качестве целевого объекта вычислений, используйте пакет SDK машинного обучения Azure и укажите следующие сведения.

* __Имя вычисления__. Имя, которое вы хотите назначить как вычислительный ресурс.
* __Группа ресурсов__. Группа ресурсов, содержащая учетную запись Data Lake Analytics.
* __Имя учетной записи__. имя учетной записи Data Lake Analytics.

Следующий код демонстрирует, как вложить Data Lake Analytics в качестве целевого объекта вычислений.

```python
import os
from azureml.core.compute import ComputeTarget, AdlaCompute
from azureml.exceptions import ComputeTargetException


adla_compute_name = os.environ.get(
    "AML_ADLA_COMPUTE_NAME", "<adla_compute_name>")
adla_resource_group = os.environ.get(
    "AML_ADLA_RESOURCE_GROUP", "<adla_resource_group>")
adla_account_name = os.environ.get(
    "AML_ADLA_ACCOUNT_NAME", "<adla_account_name>")

try:
    adla_compute = ComputeTarget(workspace=ws, name=adla_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('adla_compute_name {}'.format(adla_compute_name))
    print('adla_resource_id {}'.format(adla_resource_group))
    print('adla_account_name {}'.format(adla_account_name))
    # create attach config
    attach_config = AdlaCompute.attach_configuration(resource_group=adla_resource_group,
                                                     account_name=adla_account_name)
    # Attach ADLA
    adla_compute = ComputeTarget.attach(
        ws,
        adla_compute_name,
        attach_config
    )

    adla_compute.wait_for_completion(True)
```

Более подробный пример см. в [примере записной книжки](https://aka.ms/pl-adla) на сайте GitHub.

> [!WARNING]
> Не создавайте несколько одновременных вложений для одного и того же ADLA из рабочей области. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.

> [!TIP]
> Конвейеры машинного обучения Azure работают только с хранимыми данными в хранилище данных учетной записи Data Lake Analytics по умолчанию. Если данные, с которыми необходимо работать, находятся в хранилище, отличном от хранилища по умолчанию, можно использовать [`DataTransferStep`](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep) для копирования данных перед обучением.

## <a name="azure-container-instance"></a><a id="aci"></a>Экземпляр контейнера Azure

Службы "экземпляры контейнеров Azure" (ACI) создаются динамически при развертывании модели. Вы не можете создать или присоединить ACI к рабочей области любым другим способом. Дополнительные сведения см. в статье [развертывание модели в службе "экземпляры контейнеров Azure](how-to-deploy-azure-container-instance.md)".

## <a name="azure-kubernetes-service"></a>Служба Azure Kubernetes

Служба Azure Kubernetes Service (AKS) позволяет использовать различные параметры конфигурации при использовании с Машинное обучение Azure. Дополнительные сведения см. [в статье Создание и подключение службы Kubernetes Azure](how-to-create-attach-kubernetes.md).


## <a name="notebook-examples"></a>Примеры записных книжек

Посмотрите примеры обучения различных целевых объектов вычислений в следующих записных книжках.
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Используйте ресурс вычислений для [настройки и отправки обучающего запуска](how-to-set-up-training-targets.md).
* [Руководство. по обучению модели классификации изображений с помощью Службы машинного обучения Azure](tutorial-train-models-with-aml.md). В нем используется управляемый целевой объект вычислений для обучения модели.
* Узнайте, как [эффективно настроить гиперпараметры](how-to-tune-hyperparameters.md) для создания улучшенных моделей.
* После обучения модели узнайте о [способах и расположениях развертывания моделей](how-to-deploy-and-where.md).
* [Использование Машинного обучения Azure с виртуальными сетями Microsoft Azure](./how-to-network-security-overview.md)
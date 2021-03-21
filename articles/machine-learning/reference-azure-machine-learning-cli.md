---
title: Установка и использование расширения CLI для Машинного обучения Azure
description: Узнайте, как использовать расширение Azure CLI для машинного обучения для создания & управления ресурсами, такими как Рабочая область, хранилища данных, DataSet, конвейеры, модели и развертывания.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.author: jordane
author: jpe316
ms.date: 06/22/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 3e073310d62bfb772ea1120bd379cdc277137da0
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519118"
---
# <a name="install--use-the-cli-extension-for-azure-machine-learning"></a>Установка и использование расширения CLI для Машинного обучения Azure


CLI для Машинного обучения Azure является расширением кроссплатформенного интерфейса командной строки для платформы Azure ([Azure CLI](/cli/azure/)). Это расширение предоставляет команды CLI для работы с Машинным обучением Azure. С его помощью можно автоматизировать операции машинного обучения. В следующем списке приведены примеры действий, которые можно выполнить с помощью расширения CLI:

+ выполнять эксперименты для создания моделей машинного обучения;

+ регистрировать модели машинного обучения для использования клиентами;

+ упаковывать, развертывать и отслеживать жизненный цикл моделей машинного обучения.

Интерфейс командной строки не является заменой пакета SDK для Машинного обучения Azure. Это дополнительное средство, оптимизированное для обработки очень параметризованных задач, которые хорошо поддаются автоматизации.

## <a name="prerequisites"></a>Предварительные требования

* Для использования интерфейса командной строки необходима подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.

* Чтобы выполнять приведенные в этом документе команды CLI в **локальной среде**, вам потребуется [Azure CLI](/cli/azure/install-azure-cli).

    Если вы используете [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/), интерфейс командной строки доступен через браузер и находится в облаке.

## <a name="full-reference-docs"></a>Полная справочная документация

Ознакомьтесь с [полной справочной документацией по расширению azure-cli-ml (Azure CLI)](/cli/azure/ext/azure-cli-ml/).

## <a name="connect-the-cli-to-your-azure-subscription"></a>Подключение интерфейса командной строки к своей подписке Azure

> [!IMPORTANT]
> Если вы используете Azure Cloud Shell, этот раздел можно пропустить. Cloud Shell автоматически выполняет аутентификацию, используя учетную запись, с помощью которой вы вошли в подписку Azure.

Существует несколько способов аутентификации в подписке Azure с помощью CLI. Самый простой — выполнить интерактивную проверку подлинности с помощью браузера. Чтобы выполнить проверку подлинности в интерактивном режиме, откройте командную строку или терминал и используйте следующую команду:

```azurecli-interactive
az login
```

Если CLI сможет запустить браузер по умолчанию, он откроет в браузере страницу входа. Или откройте браузер и выполните инструкции из командной строки. В инструкции входит переход к [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и ввод кода авторизации.

[!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)]

Другие методы аутентификации см. в статье [Вход с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

## <a name="install-the-extension"></a>Установка расширения

Чтобы установить расширение интерфейса командной строки Машинного обучения, выполните следующую команду:

```azurecli-interactive
az extension add -n azure-cli-ml
```

> [!TIP]
> Примеры файлов, которые можно использовать с приведенными ниже командами, приведены [здесь](https://aka.ms/azml-deploy-cloud).

При появлении запроса выберите `y`, чтобы установить расширение.

Чтобы убедиться, что расширение установлено, используйте следующую команду для отображения списка подкоманд Машинного обучения:

```azurecli-interactive
az ml -h
```

## <a name="update-the-extension"></a>Обновление расширения

Чтобы обновить расширение CLI для Машинного обучения, выполните следующую команду:

```azurecli-interactive
az extension update -n azure-cli-ml
```


## <a name="remove-the-extension"></a>Удаление расширения

Чтобы удалить расширение интерфейса командной строки, используйте следующую команду:

```azurecli-interactive
az extension remove -n azure-cli-ml
```

## <a name="resource-management"></a>Управление ресурсами

Следующие команды демонстрируют использование интерфейса командной строки для управления ресурсами, используемыми службой "Машинное обучение Azure".

+ Если у вас нет группы ресурсов, создайте ее:

    ```azurecli-interactive
    az group create -n myresourcegroup -l westus2
    ```

+ Создайте рабочую область Машинного обучения Azure:

    ```azurecli-interactive
    az ml workspace create -w myworkspace -g myresourcegroup
    ```

    См. дополнительные сведения о команде [az ml workspace create](/cli/azure/ext/azure-cli-ml/ml/workspace#ext-azure-cli-ml-az-ml-workspace-create).

+ Подключите конфигурацию рабочей области к папке, чтобы включить поддержку контекста в CLI.

    ```azurecli-interactive
    az ml folder attach -w myworkspace -g myresourcegroup
    ```

    Эта команда создает подкаталог `.azureml`, содержащий примеры файлов runconfig и среды Conda. Он также содержит файл `config.json`, который используется для взаимодействия с рабочей областью Машинного обучения Azure.

    См. дополнительные сведения о команде [az ml folder attach](/cli/azure/ext/azure-cli-ml/ml/folder#ext-azure-cli-ml-az-ml-folder-attach).

+ Прикрепите контейнер BLOB-объектов Azure в качестве хранилища данных.

    ```azurecli-interactive
    az ml datastore attach-blob  -n datastorename -a accountname -c containername
    ```

    См. дополнительные сведения о команде [az ml datastore attach-blob](/cli/azure/ext/azure-cli-ml/ml/datastore#ext-azure-cli-ml-az-ml-datastore-attach-blob).

+ Передайте файлы в хранилище данных.

    ```azurecli-interactive
    az ml datastore upload  -n datastorename -p sourcepath
    ```

    См. дополнительные сведения о команде [az ml datastore upload](/cli/azure/ext/azure-cli-ml/ml/datastore#ext-azure-cli-ml-az-ml-datastore-upload).

+ Подключите кластер AKS в качестве целевого объекта вычислений.

    ```azurecli-interactive
    az ml computetarget attach aks -n myaks -i myaksresourceid -g myresourcegroup -w myworkspace
    ```

    См. дополнительные сведения о команде [az ml computetarget attach aks](/cli/azure/ext/azure-cli-ml/ml/computetarget/attach#ext-azure-cli-ml-az-ml-computetarget-attach-aks).

### <a name="compute-clusters"></a>Вычислительные кластеры

+ Создайте новый управляемый кластер вычислений.

    ```azurecli-interactive
    az ml computetarget create amlcompute -n cpu --min-nodes 1 --max-nodes 1 -s STANDARD_D3_V2
    ```



+ Создание нового управляемого кластера вычислений с управляемым удостоверением

  + Управляемое удостоверение, назначаемое пользователем

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
    ```

  + Управляемое удостоверение, назначаемое системой

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '[system]'
    ```
+ Добавление управляемого удостоверения в существующий кластер:

    + Управляемое удостоверение, назначаемое пользователем
        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
        ```
    + Управляемое удостоверение, назначаемое системой

        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '[system]'
        ```

См. дополнительные сведения о команде [az ml computetarget create amlcompute](/cli/azure/ext/azure-cli-ml/ml/computetarget/create#ext-azure-cli-ml-az-ml-computetarget-create-amlcompute).

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-note.md)]

<a id="computeinstance"></a>

### <a name="compute-instance"></a>Вычислительная операция
Управление экземплярами вычислений.  Во всех приведенных ниже примерах имя вычислительного экземпляра — **CPU** .

+ Создайте новый компутеинстанце.

    ```azurecli-interactive
    az ml computetarget create computeinstance -n cpu -s "STANDARD_D3_V2" -v
    ```

    Дополнительные сведения см. в разделе [AZ ML computetarget Create компутеинстанце](/cli/azure/ext/azure-cli-ml/ml/computetarget/create#ext-azure-cli-ml-az-ml-computetarget-create-computeinstance).

+ Останавливает компутеинстанце.

    ```azurecli-interactive
    az ml computetarget computeinstance stop -n cpu -v
    ```

    Дополнительные сведения см. в разделе [AZ ML computetarget компутеинстанце останавливаться](/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance#ext-azure-cli-ml-az-ml-computetarget-computeinstance-stop).

+ Запустите компутеинстанце.

    ```azurecli-interactive
    az ml computetarget computeinstance start -n cpu -v
    ```

    Дополнительные сведения см. в статье [AZ ML computetarget компутеинстанце Start](/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance#ext-azure-cli-ml-az-ml-computetarget-computeinstance-start).

+ Перезапустите компутеинстанце.

    ```azurecli-interactive
    az ml computetarget computeinstance restart -n cpu -v
    ```

    Дополнительные сведения см. в статье [AZ ML computetarget компутеинстанце Restart](/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance#ext-azure-cli-ml-az-ml-computetarget-computeinstance-restart).

+ Удаление компутеинстанце.

    ```azurecli-interactive
    az ml computetarget delete -n cpu -v
    ```

    Дополнительные сведения см. в разделе [AZ ML computetarget Delete компутеинстанце](/cli/azure/ext/azure-cli-ml/ml/computetarget#ext-azure-cli-ml-az-ml-computetarget-delete).


## <a name="run-experiments"></a><a id="experiments"></a>Запуск экспериментов

* Запустите выполнение эксперимента. При использовании этой команды укажите имя файла runconfig (текст перед \*.runconfig в вашей файловой системе) в параметре -c.

    ```azurecli-interactive
    az ml run submit-script -c sklearn -e testexperiment train.py
    ```

    > [!TIP]
    > Команда `az ml folder attach` создает подкаталог `.azureml`, который содержит два примера файлов runconfig. 
    >
    > При наличии скрипта Python, который программно создает объект конфигурации запуска, вы можете использовать метод [RunConfig.Save()](/python/api/azureml-core/azureml.core.runconfiguration#save-path-none--name-none--separate-environment-yaml-false-), чтобы сохранить его как файл runconfig.
    >
    > Полную схему runconfig можно найти в этом [файле JSON](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json). Схема содержит самодокументирующийся код за счет использования ключа `description` для каждого объекта. Кроме того, для каждого объекта предусмотрены перечисления возможных значений, а в конце приведен фрагмент кода с шаблоном.

    См. дополнительные сведения о команде [az ml run submit-script](/cli/azure/ext/azure-cli-ml/ml/run#ext-azure-cli-ml-az-ml-run-submit-script).

* Просмотрите список экспериментов:

    ```azurecli-interactive
    az ml experiment list
    ```

    См. дополнительные сведения о команде [az ml experiment list](/cli/azure/ext/azure-cli-ml/ml/experiment#ext-azure-cli-ml-az-ml-experiment-list).

### <a name="hyperdrive-run"></a>Запуск HyperDrive

Используя HyperDrive с Azure CLI, можно запускать выполнение для настройки параметров. Сначала создайте файл конфигурации HyperDrive в следующем формате. Дополнительные сведения о параметрах настройки гиперпараметров см. в статье [Настройка гиперпараметров модели](how-to-tune-hyperparameters.md).

```yml
# hdconfig.yml
sampling: 
    type: random # Supported options: Random, Grid, Bayesian
    parameter_space: # specify a name|expression|values tuple for each parameter.
    - name: --penalty # The name of a script parameter to generate values for.
      expression: choice # supported options: choice, randint, uniform, quniform, loguniform, qloguniform, normal, qnormal, lognormal, qlognormal
      values: [0.5, 1, 1.5] # The list of values, the number of values is dependent on the expression specified.
policy: 
    type: BanditPolicy # Supported options: BanditPolicy, MedianStoppingPolicy, TruncationSelectionPolicy, NoTerminationPolicy
    evaluation_interval: 1 # Policy properties are policy specific. See the above link for policy specific parameter details.
    slack_factor: 0.2
primary_metric_name: Accuracy # The metric used when evaluating the policy
primary_metric_goal: Maximize # Maximize|Minimize
max_total_runs: 8 # The maximum number of runs to generate
max_concurrent_runs: 2 # The number of runs that can run concurrently.
max_duration_minutes: 100 # The maximum length of time to run the experiment before cancelling.
```

Добавьте этот файл вместе с файлами конфигурации запуска. Затем отправьте запрос на запуск HyperDrive с помощью следующей команды.
```azurecli
az ml run submit-hyperdrive -e <experiment> -c <runconfig> --hyperdrive-configuration-name <hdconfig> my_train.py
```

Запишите данные раздела *arguments* в схеме runconfig и *пространство параметров* в файле конфигурации HyperDrive. Они содержат аргументы командной строки, передаваемые в скрипт обучения. Значение в схеме runconfig остается неизменным для каждой итерации, в то время как диапазон в файле конфигурации HyperDrive меняется. Не указывайте один и тот же аргумент в обоих файлах.

## <a name="dataset-management"></a>Управление наборами данных

Следующие команды показывают, как работать с наборами данных в службе "Машинное обучение Azure".

+ Регистрация набора данных:

    ```azurecli-interactive
    az ml dataset register -f mydataset.json
    ```

    Для получения сведений о формате файла JSON, используемого для определения набора данных, используйте `az ml dataset register --show-template`.

    См. дополнительные сведения о команде [az ml dataset register](/cli/azure/ext/azure-cli-ml/ml/dataset#ext-azure-cli-ml-az-ml-dataset-register).

+ Получите список всех наборов данных в рабочей области:

    ```azurecli-interactive
    az ml dataset list
    ```

    См. дополнительные сведения о команде [az ml dataset list](/cli/azure/ext/azure-cli-ml/ml/dataset#ext-azure-cli-ml-az-ml-dataset-list).

+ Получите сведения о наборе данных:

    ```azurecli-interactive
    az ml dataset show -n dataset-name
    ```

    См. дополнительные сведения о команде [az ml dataset show](/cli/azure/ext/azure-cli-ml/ml/dataset#ext-azure-cli-ml-az-ml-dataset-show).

+ Отмените регистрацию набора данных:

    ```azurecli-interactive
    az ml dataset unregister -n dataset-name
    ```

    См. дополнительные сведения о команде [az ml dataset unregister](/cli/azure/ext/azure-cli-ml/ml/dataset#ext-azure-cli-ml-az-ml-dataset-archive).

## <a name="environment-management"></a>Управление средой

Следующие команды демонстрируют создание, регистрацию и перечисление [сред](how-to-configure-environment.md) Машинного обучения Azure для рабочей области.

+ Создайте файлы формирования шаблонов для среды:

    ```azurecli-interactive
    az ml environment scaffold -n myenv -d myenvdirectory
    ```

    См. дополнительные сведения о команде [az ml environment scaffold](/cli/azure/ext/azure-cli-ml/ml/environment#ext-azure-cli-ml-az-ml-environment-scaffold).

+ Зарегистрируйте среду:

    ```azurecli-interactive
    az ml environment register -d myenvdirectory
    ```

    См. дополнительные сведения о команде [az ml environment register](/cli/azure/ext/azure-cli-ml/ml/environment#ext-azure-cli-ml-az-ml-environment-register).

+ Получите список зарегистрированных сред:

    ```azurecli-interactive
    az ml environment list
    ```

    См. дополнительные сведения о команде [az ml environment list](/cli/azure/ext/azure-cli-ml/ml/environment#ext-azure-cli-ml-az-ml-environment-list).

+ Загрузите зарегистрированную среду:

    ```azurecli-interactive
    az ml environment download -n myenv -d downloaddirectory
    ```

    См. дополнительные сведения о команде [az ml environment download](/cli/azure/ext/azure-cli-ml/ml/environment#ext-azure-cli-ml-az-ml-environment-download).

### <a name="environment-configuration-schema"></a>Схема конфигурации среды

Если вы использовали команду `az ml environment scaffold`, она создает файл шаблона `azureml_environment.json`, который можно изменить и использовать для создания настраиваемых конфигураций среды с помощью интерфейса командной строки. Объект верхнего уровня нестрого сопоставлен с классом [`Environment`](/python/api/azureml-core/azureml.core.environment%28class%29) в пакете SDK для Python. 

```json
{
    "name": "testenv",
    "version": null,
    "environmentVariables": {
        "EXAMPLE_ENV_VAR": "EXAMPLE_VALUE"
    },
    "python": {
        "userManagedDependencies": false,
        "interpreterPath": "python",
        "condaDependenciesFile": null,
        "baseCondaEnvironment": null
    },
    "docker": {
        "enabled": false,
        "baseImage": "mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04",
        "baseDockerfile": null,
        "sharedVolumes": true,
        "shmSize": "2g",
        "arguments": [],
        "baseImageRegistry": {
            "address": null,
            "username": null,
            "password": null
        }
    },
    "spark": {
        "repositories": [],
        "packages": [],
        "precachePackages": true
    },
    "databricks": {
        "mavenLibraries": [],
        "pypiLibraries": [],
        "rcranLibraries": [],
        "jarLibraries": [],
        "eggLibraries": []
    },
    "inferencingStackVersion": null
}
```

В следующей таблице приводятся подробные сведения о каждом поле верхнего уровня в JSON-файле, его тип и описание. Если тип объекта связан с классом из пакета SDK для Python, каждому полю JSON нестрого сопоставлено имя общедоступной переменной в классе Python. В некоторых случаях поле может сопоставляться с аргументом конструктора, а не с переменной класса. Например, поле `environmentVariables` сопоставляется с переменной `environment_variables` в классе [`Environment`](/python/api/azureml-core/azureml.core.environment%28class%29).

| Поле JSON | Тип | Описание |
|---|---|---|
| `name` | `string` | Имя среды. Не начинайте имя с **Microsoft** или **AzureML**. |
| `version` | `string` | Версия среды. |
| `environmentVariables` | `{string: string}` | Хэш-карта имен и значений переменных среды. |
| `python` | [`PythonSection`](/python/api/azureml-core/azureml.core.environment.pythonsection)Hat определяет среду и интерпретатор Python для использования в целевом ресурсе вычислений. |
| `docker` | [`DockerSection`](/python/api/azureml-core/azureml.core.environment.dockersection) | Определяет параметры для настройки образа Docker, созданного согласно спецификациям среды. |
| `spark` | [`SparkSection`](/python/api/azureml-core/azureml.core.environment.sparksection) | В разделе настраиваются параметры Spark. Он используется только в том случае, если используется платформа PySpark. |
| `databricks` | [`DatabricksSection`](/python/api/azureml-core/azureml.core.databricks.databrickssection) | Настраивает зависимости библиотеки Databricks. |
| `inferencingStackVersion` | `string` | Указывает версию стека вывода, добавленного в образ. Чтобы не добавлять стек вывода, оставьте для этого поля значение `null`. Допустимое значение: "latest". |

## <a name="ml-pipeline-management"></a>Управление конвейером Машинного обучения

Следующие команды показывают, как работать с конвейерами машинного обучения.

+ Создайте конвейер машинного обучения:

    ```azurecli-interactive
    az ml pipeline create -n mypipeline -y mypipeline.yml
    ```

    См. дополнительные сведения о команде [az ml pipeline create](/cli/azure/ext/azure-cli-ml/ml/pipeline#ext-azure-cli-ml-az-ml-pipeline-create).

    Дополнительные сведения о файле конвейера YAML см. в статье [Определение конвейеров машинного обучения в YAML](reference-pipeline-yaml.md).

+ Выполните конвейер:

    ```azurecli-interactive
    az ml run submit-pipeline -n myexperiment -y mypipeline.yml
    ```

    См. дополнительные сведения о команде [az ml run submit-pipeline](/cli/azure/ext/azure-cli-ml/ml/run#ext-azure-cli-ml-az-ml-run-submit-pipeline).

    Дополнительные сведения о файле конвейера YAML см. в статье [Определение конвейеров машинного обучения в YAML](reference-pipeline-yaml.md).

+ Запланируйте выполнение конвейера:

    ```azurecli-interactive
    az ml pipeline create-schedule -n myschedule -e myexpereiment -i mypipelineid -y myschedule.yml
    ```

    См. дополнительные сведения о команде [az ml pipeline create-schedule](/cli/azure/ext/azure-cli-ml/ml/pipeline#ext-azure-cli-ml-az-ml-pipeline-create-schedule).

    Дополнительные сведения о файле расписания конвейера YAML см. в статье [Определение конвейеров машинного обучения в YAML](reference-pipeline-yaml.md#schedules).

## <a name="model-registration-profiling-deployment"></a>Регистрация модели, профилирование, развертывание

Следующие команды демонстрируют регистрацию обученной модели, а затем ее развертывание в качестве рабочей службы:

+ Зарегистрируйте модель с помощью Машинного обучения Azure.

    ```azurecli-interactive
    az ml model register -n mymodel -p sklearn_regression_model.pkl
    ```

    См. дополнительные сведения о команде [az ml model register](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-register).

+ **НЕОБЯЗАТЕЛЬНО.** Выполните профилирование модели, чтобы получить оптимальные значения нагрузки на ЦП и память для развертывания.
    ```azurecli-interactive
    az ml model profile -n myprofile -m mymodel:1 --ic inferenceconfig.json -d "{\"data\": [[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}" -t myprofileresult.json
    ```

    См. дополнительные сведения о команде [az ml model profile](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-profile).

+ Развертывание модели в AKS
    ```azurecli-interactive
    az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json --ct akscomputetarget
    ```
    
    Дополнительные сведения о схеме файла конфигурации вывода см. в разделе [Схема конфигурации вывода](#inferenceconfig).
    
    Дополнительные сведения о схеме файла конфигурации развертывания см. в разделе [Схема конфигурации развертывания](#deploymentconfig).

    См. дополнительные сведения о команде [az ml model deploy](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-deploy).

<a id="inferenceconfig"></a>

## <a name="inference-configuration-schema"></a>Схема конфигурации вывода

[!INCLUDE [inferenceconfig](../../includes/machine-learning-service-inference-config.md)]

<a id="deploymentconfig"></a>

## <a name="deployment-configuration-schema"></a>Схема конфигурации развертывания

### <a name="local-deployment-configuration-schema"></a>Схема конфигурации локального развертывания

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-local-deploy-config.md)]

### <a name="azure-container-instance-deployment-configuration-schema"></a>Схема конфигурации для развертывания экземпляра контейнера Azure 

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aci-deploy-config.md)]

### <a name="azure-kubernetes-service-deployment-configuration-schema"></a>Схема конфигурации для развертывания экземпляра Службы Azure Kubernetes

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aks-deploy-config.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Справочник по командам для расширения CLI Машинного обучения](/cli/azure/ext/azure-cli-ml/ml)

* [Обучение и развертывание моделей машинного обучения с помощью Azure Pipelines](/azure/devops/pipelines/targets/azure-machine-learning)

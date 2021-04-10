---
title: Обучение и развертывание моделей из интерфейса командной строки
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать расширение машинного обучения для Azure CLI, чтобы обучать, регистрировать и развертывать модели из командной строки.
ms.author: larryfr
author: Blackmist
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 04/02/2021
ms.openlocfilehash: e518ea83608472753d3463a980aae169b291d4de
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221302"
---
# <a name="tutorial-train-and-deploy-a-model-from-the-cli"></a>Руководство по Обучение и развертывание моделей из интерфейса командной строки


В этом учебнике вы будете использовать расширение машинного обучения для Azure CLI, чтобы обучать, регистрировать и развертывать модели.

В сценариях обучения Python в этом учебнике используется [scikit-learn](https://scikit-learn.org/) для обучения простой модели. Основное внимание уделяется не сценариям или модели, а процессу использования интерфейса командной строки для работы с Машинным обучением Azure.

Узнайте, как выполнять следующие действия:

> [!div class="checklist"]
> * Установка расширения машинного обучения
> * Создание рабочей области машинного обучения Azure
> * Создание вычислительного ресурса для обучения модели
> * Определение и регистрация набора данных, используемого для обучения модели
> * Запуск обучающего прогона
> * Регистрация и загрузка модели
> * Развертывание модели в качестве веб-службы.
> * Оценка данных с помощью веб-службы

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.

* Чтобы выполнять приведенные в этом документе команды CLI в **локальной среде**, вам потребуется [Azure CLI](/cli/azure/install-azure-cli).

    Если вы используете [Azure Cloud Shell](https://azure.microsoft.com//features/cloud-shell/), интерфейс командной строки доступен через браузер и находится в облаке.

## <a name="download-the-example-project"></a>Загрузка примера проекта

Для выполнения заданий в этом учебнике загрузите проект [https://github.com/microsoft/MLOps](https://github.com/microsoft/MLOps). Файлы в каталоге `examples/cli-train-deploy` используются в действиях, описанных в этом учебнике.

Чтобы получить локальную копию файлов, [загрузите ZIP-архив](https://github.com/microsoft/MLOps/archive/master.zip) или выполните следующую команду Git, чтобы клонировать репозиторий:

```azurecli-interactive
git clone https://github.com/microsoft/MLOps.git
```

### <a name="training-files"></a>Файлы для обучения

Каталог `examples/cli-train-deploy` из проекта содержит следующие файлы, которые используются при обучении модели:

* `.azureml\mnist.runconfig`: файл __конфигурации запуска__. Этот файл определяет среду выполнения, необходимую для обучения модели. В этом примере он также подключает данные, используемые для обучения модели, в среду обучения.
* `scripts\train.py`: скрипт обучения. Этот файл обучает модель.
* `scripts\utils.py`: вспомогательный файл, используемый сценарием обучения.
* `.azureml\conda_dependencies.yml`: определяет зависимости программного обеспечения, необходимые для запуска сценария обучения.
* `dataset.json`: определение набора данных. Используется для регистрации набора данных MNIST в рабочей области Машинного обучения Azure.

### <a name="deployment-files"></a>Файлы развертывания

Репозиторий содержит следующие файлы, которые используются для развертывания обученной модели в качестве веб-службы:

* `aciDeploymentConfig.yml`: файл __конфигурации развертывания__. Этот файл определяет среду размещения, необходимую для модели.
* `inferenceConfig.json`: файл __конфигурации вывода__. Этот файл определяет программную среду, используемую службой для оценки данных с помощью модели.
* `score.py`: скрипт Python, который принимает входящие данные, оценивает их с помощью модели, а затем возвращает ответ.
* `scoring-env.yml`: зависимости conda, необходимые для запуска модели и скрипта `score.py`.
* `testdata.json`: файл данных, который можно использовать для тестирования развернутой веб-службы.

## <a name="connect-to-your-azure-subscription"></a>Подключение к подписке Azure

Существует несколько способов проверки подлинности в подписке Azure с помощью интерфейса командной строки. Самый простой — выполнить интерактивную аутентификацию с помощью браузера. Чтобы выполнить проверку подлинности в интерактивном режиме, откройте командную строку или терминал и используйте следующую команду:

```azurecli-interactive
az login
```

Если CLI сможет запустить браузер по умолчанию, он откроет в браузере страницу входа. Или откройте браузер и выполните инструкции из командной строки. В инструкции входит переход к [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и ввод кода авторизации.

[!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это контейнер ресурсов на платформе Azure. При работе с Машинным обучением Azure группа ресурсов будет содержать рабочую область Машинного обучения Azure. Она также будет содержать другие службы Azure, используемые рабочей областью. Например, при обучении модели с помощью облачного ресурса вычислений этот ресурс создается в группе ресурсов.

Выполните следующую команду, чтобы __создать новую группу ресурсов__. Замените `<resource-group-name>` именем, которое будет использоваться для этой группы ресурсов. Замените `<location>` регионом Azure, который будет использоваться для этой группы ресурсов:

> [!TIP]
> Необходимо выбрать регион, в котором доступно Машинное обучение Azure. Дополнительные сведения см. в статье [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-service).

```azurecli-interactive
az group create --name <resource-group-name> --location <location>
```

Результат этой команды аналогичен следующему коду JSON:

```json
{
  "id": "/subscriptions/<subscription-GUID>/resourceGroups/<resourcegroupname>",
  "location": "<location>",
  "managedBy": null,
  "name": "<resource-group-name>",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": null
}
```

Дополнительные сведения о работе с группами ресурсов см. в разделе [az group](/cli/azure/group).

## <a name="create-a-workspace"></a>Создание рабочей области

Создайте рабочую область с помощью следующей команды. Замените `<workspace-name>` именем, которое вы хотите использовать для этой рабочей области. Замените `<resource-group-name>` именем группы ресурсов:

```azurecli-interactive
az ml workspace create -w <workspace-name> -g <resource-group-name>
```

Результат этой команды аналогичен следующему коду JSON:

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>",
  "containerRegistry": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.containerregistry/registries/<acr-name>",
  "creationTime": "2019-08-30T20:24:19.6984254+00:00",
  "description": "",
  "friendlyName": "<workspace-name>",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

## <a name="connect-local-project-to-workspace"></a>Подключение локального проекта к рабочей области

В терминале или командной строке используйте следующие команды, чтобы изменить каталоги на каталог `cli-train-deploy`, а затем подключиться к рабочей области:

```azurecli-interactive
cd ~/MLOps/examples/cli-train-deploy
az ml folder attach -w <workspace-name> -g <resource-group-name>
```

Результат этой команды аналогичен следующему коду JSON:

```json
{
  "Experiment name": "model-training",
  "Project path": "/home/user/MLOps/examples/cli-train-deploy",
  "Resource group": "<resource-group-name>",
  "Subscription id": "<subscription-id>",
  "Workspace name": "<workspace-name>"
}
```

Эта команда создает файл `.azureml/config.json`, который содержит сведения, необходимые для подключения к рабочей области. Остальные команды `az ml`, используемые в этом учебнике, будут использовать этот файл, поэтому вам не нужно добавлять рабочую область и группу ресурсов во все команды.

## <a name="create-the-compute-target-for-training"></a>Создание целевого объекта вычислений для обучения

В этом примере для обучения модели используется кластер Вычислительной среды Машинного обучения Azure. Создайте вычислительный кластер с помощью следующей команды:

```azurecli-interactive
az ml computetarget create amlcompute -n cpu-cluster --max-nodes 4 --vm-size Standard_D2_V2
```

Результат этой команды аналогичен следующему коду JSON:

```json
{
  "location": "<location>",
  "name": "cpu-cluster",
  "provisioningErrors": null,
  "provisioningState": "Succeeded"
}
```

Эта команда создает новый целевой объект вычислений с именем `cpu-cluster` и максимум четырьмя узлами. Выбранный размер виртуальной машины предоставляет виртуальную машину с ресурсом GPU. Сведения о размере виртуальной машины см. в разделе [Типы и размеры виртуальных машин].

> [!IMPORTANT]
> Имя целевого объекта вычислений (`cpu-cluster` в данном случае) важно; на него ссылается файл `.azureml/mnist.runconfig`, используемый в следующем разделе.

## <a name="define-the-dataset"></a>Определение набора данных

Для обучения модели можно предоставить обучающие данные с помощью набора данных. Чтобы создать набор данных из интерфейса командной строки, необходимо указать файл определения набора данных. Файл `dataset.json`, указанный в репозитории, создает новый набор данных с помощью данных MNIST. Создаваемый набор данных называется `mnist-dataset`.

Чтобы зарегистрировать набор данных с помощью файла `dataset.json`, используйте следующую команду:

```azurecli-interactive
az ml dataset register -f dataset.json --skip-validation
```

Результат этой команды аналогичен следующему коду JSON:

```json
{
  "definition": [
    "GetFiles"
  ],
  "registration": {
    "description": "mnist dataset",
    "id": "a13a4034-02d1-40bd-8107-b5d591a464b7",
    "name": "mnist-dataset",
    "tags": {
      "sample-tag": "mnist"
    },
    "version": 1,
    "workspace": "Workspace.create(name='myworkspace', subscription_id='mysubscriptionid', resource_group='myresourcegroup')"
  },
  "source": [
    "http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz",
    "http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz",
    "http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz",
    "http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz"
  ]
}
```

> [!IMPORTANT]
> Скопируйте значение записи `id`, так как оно используется в следующем разделе.

Чтобы просмотреть более полный шаблон для набора данных, используйте следующую команду:

```azurecli-interactive
az ml dataset register --show-template
```

## <a name="reference-the-dataset"></a>Ссылка на набор данных

Чтобы сделать набор данных доступным в среде обучения, необходимо сослаться на него из файла runconfig. Файл `.azureml/mnist.runconfig` содержит следующие записи YAML:

```yaml
# The arguments to the script file.
arguments:
- --data-folder
- DatasetConsumptionConfig:mnist

.....

# The configuration details for data.
data:
  mnist:
# Data Location
    dataLocation:
# the Dataset used for this run.
      dataset:
# Id of the dataset.
        id: a13a4034-02d1-40bd-8107-b5d591a464b7
# the DataPath used for this run.
      datapath:
# Whether to create new folder.
    createOutputDirectories: false
# The mode to handle
    mechanism: mount
# Point where the data is download or mount or upload.
    environmentVariableName: mnist
# relative path where the data is download or mount or upload.
    pathOnCompute:
# Whether to overwrite the data if existing.
    overwrite: false
```

Измените значение записи `id` в соответствии со значением, возвращаемым при регистрации набора данных. Это значение используется для загрузки данных в целевой объект вычислений во время обучения.

Этот YAML выполняет следующие действия во время обучения.

* Подключает набор данных (на основе идентификатора набора данных) в среде обучения и сохраняет путь к точке подключения в переменной среды `mnist`.
* Передает расположение данных (точку подключения) в среде обучения в скрипт, используя аргумент `--data-folder`.

Файл runconfig также содержит сведения, используемые для настройки среды, в которой происходит обучающий запуск. Если просмотреть этот файл, вы увидите, что он ссылается на целевой объект вычислений `cpu-compute`, созданный ранее. В нем также указано количество узлов, используемых при обучении (`"nodeCount": "4"`), и содержится раздел `"condaDependencies"`, где перечислены пакеты Python, необходимые для запуска сценария обучения.

> [!TIP]
> Хотя можно вручную создать файл runconfig, в этом примере он был создан с помощью файла `generate-runconfig.py`, включенного в репозиторий. Этот файл получает ссылку на зарегистрированный набор данных, создает конфигурацию запуска программным способом, а затем сохраняет ее в файле.

Дополнительные сведения о файлах конфигурации запуска см. в разделе [Использование целевых объектов вычислений для обучения модели](how-to-set-up-training-targets.md#whats-a-run-configuration). Полный справочник по JSON см. в [runconfigschema.json](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json).

## <a name="submit-the-training-run"></a>Отправка обучающего запуска

Чтобы начать обучающий запуск на целевом объекте вычислений `cpu-cluster`, используйте следующую команду:

```azurecli-interactive
az ml run submit-script -c mnist -e tutorial-cli --source-directory scripts -t runoutput.json
```

Эта команда задает имя эксперимента (`tutorial-cli`). В эксперименте хранятся сведения об этом запуске в рабочей области.

Параметр `-c mnist` указывает файл `.azureml/mnist.runconfig`.

Параметр `-t` сохраняет ссылку на этот запуск в JSON-файле и будет использоваться на следующих шагах для регистрации и загрузки модели.

По мере выполнения обучающего запуска он передает потоковую информацию из сеанса обучения на удаленном ресурсе вычислений. Часть этой информации аналогична следующей:

```output
Predict the test set
Accuracy is 0.9185
```

Этот текст заносится в журнал из обучающего скрипта и отображает точность модели. Другие модели будут иметь отличные метрики производительности.

Если изучить обучающий скрипт, вы заметите, что он также использует альфа-значение при сохранении обученной модели в `outputs/sklearn_mnist_model.pkl`.

Модель была сохранена в каталоге `./outputs` в целевом объекте вычислений, где она была обучена. В этом примере используется экземпляр Вычислительной среды Машинного обучения Azure в облаке Azure. Процесс обучения автоматически отправляет содержимое каталога `./outputs` из целевого объекта вычислений, в котором выполняется обучение в рабочей области Машинного обучения Azure. Он хранится в составе эксперимента (`tutorial-cli` в этом примере).

## <a name="register-the-model"></a>Регистрация модели

Чтобы зарегистрировать модель непосредственно из сохраненной версии в эксперименте, используйте следующую команду:

```azurecli-interactive
az ml model register -n mymodel -f runoutput.json --asset-path "outputs/sklearn_mnist_model.pkl" -t registeredmodel.json
```

Эта команда регистрирует файл `outputs/sklearn_mnist_model.pkl`, созданный обучающим запуском, в качестве регистрации новой модели с именем `mymodel`. `--assets-path` ссылается на путь в эксперименте. В этом случае сведения об эксперименте и запуске загружаются из файла `runoutput.json`, созданного командой обучения. `-t registeredmodel.json` создает JSON-файл, который ссылается на новую зарегистрированную модель, созданную этой командой, и используется другими командами интерфейса командной строки, работающими с зарегистрированными моделями.

Результат этой команды аналогичен следующему коду JSON:

```json
{
  "createdTime": "2019-09-19T15:25:32.411572+00:00",
  "description": "",
  "experimentName": "tutorial-cli",
  "framework": "Custom",
  "frameworkVersion": null,
  "id": "mymodel:1",
  "name": "mymodel",
  "properties": "",
  "runId": "tutorial-cli_1568906070_5874522d",
  "tags": "",
  "version": 1
}
```

### <a name="model-versioning"></a>управления версиями моделей;

Обратите внимание на номер версии, возвращенный для модели. Версия увеличивается каждый раз при регистрации новой модели с таким именем. Например, можно скачать модель и зарегистрировать ее из локального файла с помощью следующих команд:

```azurecli-interactive
az ml model download -i "mymodel:1" -t .
az ml model register -n mymodel -p "sklearn_mnist_model.pkl"
```

Первая команда скачивает зарегистрированную модель в текущий каталог. Имя файла — `sklearn_mnist_model.pkl`. Это файл, на который указывает ссылка при регистрации модели. Вторая команда регистрирует локальную модель (`-p "sklearn_mnist_model.pkl"`) с тем же именем, что и предыдущая регистрация (`mymodel`). На этот раз в возвращенных данных JSON указана версия 2.

## <a name="deploy-the-model"></a>Развертывание модели

Выполните следующую команду, чтобы развернуть модель:

```azurecli-interactive
az ml model deploy -n myservice -m "mymodel:1" --ic inferenceConfig.json --dc aciDeploymentConfig.yml
```

> [!NOTE]
> Вы можете получить предупреждение об ошибке "Не удалось проверить существование LocalWebservice" или "Не удалось создать клиент Docker". Вы можете спокойно проигнорировать это предупреждение, так как вы не развертываете локальную веб-службу.

Эта команда развертывает новую службу с именем `myservice`, используя версию 1 модели, которая была зарегистрирована ранее.

Файл `inferenceConfig.yml` содержит сведения об использовании модели для вывода. Например, он ссылается на начальный сценарий (`score.py`) и зависимости программного обеспечения.

Дополнительные сведения о структуре этого файла см. в статье [Схема конфигурации вывода](reference-azure-machine-learning-cli.md#inference-configuration-schema). Дополнительные сведения о начальных сценариях см. в статье [Развертывание моделей в Машинном обучении Azure](how-to-deploy-and-where.md).

`aciDeploymentConfig.yml` описывает среду развертывания, используемую для размещения службы. Конфигурация развертывания зависит от типа вычислений, используемого для развертывания. В этом случае используется экземпляр контейнера Azure. Дополнительные сведения см. разделе [Схема конфигурации развертывания](reference-azure-machine-learning-cli.md#deployment-configuration-schema).

Процесс развертывания занимает несколько минут.

> [!TIP]
> В этом примере используются Экземпляры контейнеров Azure. Развертывания в ACI автоматически создают необходимый ресурс ACI. Если вместо этого вы выполняете развертывание в службу Azure Kubernetes, необходимо заранее создать кластер AKS и указать его как часть команды `az ml model deploy`. Пример развертывания в AKS см. в разделе [Развертывание модели в кластер службы Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md).

Через несколько минут будет возвращена информация, аналогичная приведенному ниже JSON:

```json
ACI service creation operation finished, operation "Succeeded"
{
  "computeType": "ACI",
  {...ommitted for space...}
  "runtimeType": null,
  "scoringUri": "http://6c061467-4e44-4f05-9db5-9f9a22ef7a5d.eastus2.azurecontainer.io/score",
  "state": "Healthy",
  "tags": "",
  "updatedAt": "2019-09-19T18:22:32.227401+00:00"
}
```

### <a name="the-scoring-uri"></a>URI оценки

`scoringUri`, возвращенный из развертывания, является конечной точкой REST для модели, развернутой в виде веб-службы. Вы также можете получить этот URI с помощью следующей команды:

```azurecli-interactive
az ml service show -n myservice
```

Эта команда возвращает тот же документ JSON, включая `scoringUri`.

Конечную точку REST можно использовать для отправки данных в службу. Сведения о создании клиентского приложения, отправляющего данные в службу, см. в разделе [Использование модели Машинного обучения Azure, развернутой в качестве веб-службы](how-to-consume-web-service.md)

### <a name="send-data-to-the-service"></a>Отправка данных в службу

Хотя вы можете создать клиентское приложение для вызова конечной точки, CLI машинного обучения предоставляет программу, которая может использоваться в качестве тестового клиента. Используйте следующую команду, чтобы отправить данные из файла `testdata.json` в службу:

```azurecli-interactive
az ml service run -n myservice -d @testdata.json
```

> [!TIP]
> При использовании PowerShell выполните приведенную ниже команду:
>
> ```azurecli-interactive
> az ml service run -n myservice -d `@testdata.json
> ```

Ответ команды аналогичен `[ 3 ]`.

## <a name="clean-up-resources"></a>Очистка ресурсов

> [!IMPORTANT]
> Созданные ресурсы могут использоваться в качестве необходимых компонентов при работе с другими руководствами по Машинному обучению Azure.

### <a name="delete-deployed-service"></a>Удаление развернутой службы

Если вы планируете продолжить использование рабочей области Машинного обучения Azure, но хотите избавиться от развернутой службы, чтобы снизить затраты, используйте следующую команду:

```azurecli-interactive
az ml service delete -n myservice
```

Эта команда возвращает документ JSON, который содержит имя удаленной службы. Удаление службы может занять несколько минут.

### <a name="delete-the-training-compute"></a>Удаление обучающих вычислений

Если вы планируете продолжить использовать рабочую область Машинного обучения Azure, но хотите избавиться от целевого объекта вычислений `cpu-cluster`, созданного для обучения, используйте следующую команду:

```azurecli-interactive
az ml computetarget delete -n cpu-cluster
```

Эта команда возвращает документ JSON, который содержит идентификатор удаленного целевого объекта вычислений. Удаление целевого объекта вычислений может занять несколько минут.

### <a name="delete-everything"></a>Удаление всех ресурсов

Если вы не планируете использовать созданные ресурсы, удалите их, чтобы с вас не взималась плата.

Чтобы удалить группу ресурсов и все ресурсы Azure, созданные в этом документе, используйте следующую команду. Замените значение `<resource-group-name>` именем вашей группы ресурсов, созданной ранее:

```azurecli-interactive
az group delete -g <resource-group-name> -y
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства по Машинному обучению Azure с помощью CLI машинного обучения вы выполнили следующие задачи:

> [!div class="checklist"]
> * Установка расширения машинного обучения
> * Создание рабочей области машинного обучения Azure
> * Создание вычислительного ресурса для обучения модели
> * Определение и регистрация набора данных, используемого для обучения модели
> * Запуск обучающего прогона
> * Регистрация и загрузка модели
> * Развертывание модели в качестве веб-службы.
> * Оценка данных с помощью веб-службы

Дополнительные сведения об использовании интерфейса командной строки см. в разделе [Использование расширения CLI для Машинного обучения Azure](reference-azure-machine-learning-cli.md).
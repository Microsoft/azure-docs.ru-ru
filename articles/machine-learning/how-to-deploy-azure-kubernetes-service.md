---
title: Развертывание моделей ML в службе Kubernetes
titleSuffix: Azure Machine Learning
description: Узнайте, как развернуть модели Машинное обучение Azure в качестве веб-службы с помощью службы Azure Kubernetes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1, deploy
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 09/01/2020
ms.openlocfilehash: 7ba01139e365b2f0023ef0784b6ed83e7bde609a
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/31/2020
ms.locfileid: "97831739"
---
# <a name="deploy-a-model-to-an-azure-kubernetes-service-cluster"></a>Развертывание модели в кластере службы Azure Kubernetes

Узнайте, как использовать Машинное обучение Azure для развертывания модели в качестве веб-службы в службе Azure Kubernetes Service (AKS). Служба Azure Kubernetes подходит для крупномасштабных производственных развертываний. Используйте службу Kubernetes Azure, если требуется одна или несколько из следующих возможностей:

- __Быстрое время отклика__
- __Автомасштабирование__ развернутой службы
- __Ведение журнала__
- __Сбор данных модели__
- __Аутентификация__
- __Терминирование TLS__
- Такие параметры __аппаратного ускорения__ , как GPU и программируемые на поля массивы ШЛЮЗОВ (FPGA)

При развертывании в службе Kubernetes Azure развертывание выполняется в кластере AKS, __подключенном к рабочей области__. Сведения о подключении кластера AKS к рабочей области см. в статье [Создание и подключение кластера службы Kubernetes Azure](how-to-create-attach-kubernetes.md).

> [!IMPORTANT]
> Рекомендуется выполнять отладку локально перед развертыванием в веб-службе. Дополнительные сведения см. в разделе [Локальная отладка](./how-to-troubleshoot-deployment-local.md) .
>
> Вы также можете ознакомиться со статьей о Машинном обучении Azure: [Развертывание в локальный Notebook](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment/deploy-to-local)

## <a name="prerequisites"></a>Предварительные требования

- Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

- Модель машинного обучения, зарегистрированная в вашей рабочей области. Если у вас нет зарегистрированной модели, см. раздел [как и где развертывать модели](how-to-deploy-and-where.md).

- [Расширение Azure CLI для службы машинное обучение](reference-azure-machine-learning-cli.md), [машинное обучение Azure пакет SDK для Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py)или [расширение машинное обучение Azure Visual Studio Code](tutorial-setup-vscode-extension.md).

- В фрагментах кода __Python__ в этой статье предполагается, что установлены следующие переменные:

    * `ws` — Укажите рабочую область.
    * `model` — Укажите вашу зарегистрированную модель.
    * `inference_config` — Задайте в качестве конфигурации вывода для модели.

    Дополнительные сведения об установке этих переменных см. в разделе [как и где развертываются модели](how-to-deploy-and-where.md).

- В фрагментах кода __CLI__ в этой статье предполагается, что вы создали `inferenceconfig.json` документ. Дополнительные сведения о создании этого документа см. в разделе [как и где развертываются модели](how-to-deploy-and-where.md).

- Кластер службы Kubernetes Azure, подключенный к рабочей области. Дополнительные сведения см. в статье [Создание и подключение кластера службы Kubernetes Azure](how-to-create-attach-kubernetes.md).

    - Если вы хотите развернуть модели на узлах GPU или узлах FPGA (или на любом конкретном номере SKU), необходимо создать кластер с конкретным номером SKU. Создание пула вторичных узлов в существующем кластере и развертывание моделей в пуле вторичных узлов не поддерживаются.

## <a name="understand-the-deployment-processes"></a>Общие сведения о процессах развертывания

Слово «Deployment» используется как в Kubernetes, так и в Машинное обучение Azure. В этих двух контекстах значение "Deployment" имеет разные значения. В Kubernetes `Deployment` — конкретная сущность, указанная с помощью декларативного файла YAML. Kubernetes `Deployment` имеет определенный жизненный цикл и конкретные связи с другими сущностями Kubernetes, такими как `Pods` и `ReplicaSets` . Вы можете узнать о Kubernetes из документации и видеороликов о том [, что такое Kubernetes?](https://aka.ms/k8slearning).

В Машинное обучение Azure "развертывание" используется в более общем смысле предоставления доступа и очистки ресурсов проекта. Ниже приведены действия, которые Машинное обучение Azure рассматривает часть развертывания.

1. Переключить файлы в папку проекта, игнорируя их, указанные в файле. амлигноре или. gitignore
1. Масштабирование кластера вычислений (относится к Kubernetes)
1. Сборка или скачивание dockerfile на расчетный узел (относится к Kubernetes)
    1. Система вычисляет хэш: 
        - Базовый образ 
        - Пользовательские шаги DOCKER (см. раздел [развертывание модели с помощью пользовательского базового образа DOCKER](./how-to-deploy-custom-docker-image.md)).
        - Conda Definition YAML (см. раздел [создание & использование программных сред в машинное обучение Azure](./how-to-use-environments.md))
    1. Система использует этот хэш в качестве ключа при поиске в рабочей области реестра контейнеров Azure (запись контроля доступа).
    1. Если он не найден, он ищет соответствие в глобальной записи контроля доступа.
    1. Если он не найден, система создает новый образ (который будет кэширован и отправлен в запись контроля доступа к рабочей области).
1. Скачивание ZIP-файла проекта во временное хранилище на кластерном узле
1. Распаковка файла проекта
1. Выполненный узел вычислений `python <entry script> <arguments>`
1. Сохранение журналов, файлов моделей и других файлов, записанных в `./outputs` учетную запись хранения, связанную с рабочей областью
1. Уменьшение масштаба вычислений, включая удаление временного хранилища (относится к Kubernetes)

### <a name="azure-ml-router"></a>Маршрутизатор машинного обучения Azure

Интерфейсный компонент (azureml-Fe), который направляет входящие запросы вывода на развернутые службы, автоматически масштабируется по мере необходимости. Масштабирование azureml-Fe основано на назначении и размере кластера AKS (число узлов). Назначение и узлы кластера настраиваются при [создании или присоединении кластера AKS](how-to-create-attach-kubernetes.md). Существует одна служба azureml-Fe на кластер, которая может выполняться на нескольких модулях.

> [!IMPORTANT]
> При использовании кластера, настроенного как __dev-test__, автоматическое масштабирование **отключено**.

Azureml-Fe масштабируется (вертикально), чтобы использовать больше ядер, и out (горизонтально) для использования большего числа модулей. При принятии решения об увеличении масштаба используется время, необходимое для маршрутизации входящих запросов вывода. Если это время превышает пороговое значение, происходит масштабирование. Если время нахождения входящих запросов по-своему превышает пороговое значение, происходит горизонтальное масштабирование.

При уменьшении масштаба и в используется использование ЦП. Если пороговое значение загрузки ЦП достигнуто, внешний интерфейс сначала будет масштабироваться. Если загрузка ЦП падает до порогового значения масштабирования, выполняется операция масштабирования. Увеличение и уменьшение масштаба выполняется только при наличии достаточного количества ресурсов кластера.

## <a name="understand-connectivity-requirements-for-aks-inferencing-cluster"></a>Общие сведения о требованиях к подключению для AKS в кластере

Когда Машинное обучение Azure создает или присоединяет кластер AKS, кластер AKS развертывается с помощью одной из следующих двух сетевых моделей:
* Сеть kubenet — ресурсы обычно создаются и настраиваются при развертывании кластера AKS.
* Сеть Azure Container Networking Interface (CNI) — кластер AKS подключен к имеющимся ресурсам виртуальной сети и конфигурациям.

В первом сетевом режиме сетевые подключения создаются и правильно настроены для службы Машинное обучение Azure. Для второго сетевого режима, так как кластер подключен к существующей виртуальной сети, особенно если для существующей виртуальной сети используется настраиваемая служба DNS, клиенту необходимо уделять особое внимание требованиям к подключению для AKSного создания кластера и обеспечить разрешение DNS и исходящее подключение для AKS.

Следующая схема фиксирует все требования к подключению для AKS. Черные стрелки представляют собой фактическое взаимодействие, а синие — имена доменов, которые должны быть разрешены в управляемом клиентом DNS.

 ![Требования к подключению для AKS](./media/how-to-deploy-aks/aks-network.png)

### <a name="overall-dns-resolution-requirements"></a>Общие требования к разрешению DNS
Разрешение DNS в существующей виртуальной сети находится под контролем клиента. Следующие DNS-записи должны быть разрешимы:
* Сервер API AKS в формате \<cluster\> . HCP. \<region\> . azmk8s.io
* Реестр контейнеров (Майкрософт) (мкр): mcr.microsoft.com
* Реестр контейнеров Azure (ARC) клиента в формате \<ACR name\> . azurecr.IO
* Учетная запись хранения Azure в формате \<account\> . Table.Core.Windows.NET и \<account\> . BLOB.Core.Windows.NET
* Используемых Для проверки подлинности AAD: api.azureml.ms
* Оценка доменного имени конечной точки, созданного автоматически МАШИНным именем Azure или с помощью пользовательского доменного имени. Автоматически созданное доменное имя будет выглядеть следующим образом: \<leaf-domain-label \+ auto-generated suffix\> . \<region\> . cloudapp.azure.com

### <a name="connectivity-requirements-in-chronological-order-from-cluster-creation-to-model-deployment"></a>Требования к подключению в хронологическом порядке: от создания кластера до развертывания модели

В процессе создания или присоединения AKS маршрутизатор машинного обучения Azure (azureml-Fe) развертывается в кластере AKS. Чтобы развернуть маршрутизатор Azure ML, узел AKS должен иметь следующие возможности:
* Разрешение DNS для сервера API AKS
* Разрешение DNS для мкр для скачивания образов DOCKER для маршрутизатора машинного обучения Azure
* Загрузка изображений из мкр, где требуется исходящее подключение

Сразу после развертывания azureml-Fe будет предпринята попытка запуститься, и для этого потребуется:
* Разрешение DNS для сервера API AKS
* Запросите сервер API AKS для обнаружения других экземпляров (это служба с несколькими Pod)
* Подключение к другим экземплярам

После запуска azureml-FE для правильной работы требуется дополнительное подключение:
* Подключение к службе хранилища Azure для скачивания динамической конфигурации
* Разрешите DNS для сервера проверки подлинности AAD api.azureml.ms и взаимодействуют с ним, когда развернутая служба использует проверку подлинности AAD.
* Запрос сервера API AKS для обнаружения развернутых моделей
* Обмен данными с развернутыми модулями модели

Во время развертывания модели для успешного развертывания модели AKS узел должен иметь следующие возможности: 
* Разрешение DNS для записи контроля доступа клиента
* Загрузка изображений из записи контроля доступа клиента
* Разрешение DNS для больших двоичных объектов Azure, где хранится модель
* Загрузка моделей из больших двоичных объектов Azure

После развертывания модели и запуска службы azureml-Fe автоматически обнаружит ее с помощью API AKS и будет готова к маршрутизации запроса. Она должна иметь возможность взаимодействовать с модулями Pod модели.
>[!Note]
>Если для развернутой модели требуется подключение (например, запросы к внешней базе данных или другой службе RESTFUL, Загрузка блога и т. д.), то необходимо включить оба разрешения DNS и исходящий трафик для этих служб.

## <a name="deploy-to-aks"></a>Развертывание в AKS

Чтобы развернуть модель в службе Kubernetes Azure, создайте __конфигурацию развертывания__ , которая описывает требуемые ресурсы вычислений. Например, число ядер и память. Также необходима __Конфигурация вывода__, описывающая среду, необходимую для размещения модели и веб-службы. Дополнительные сведения о создании конфигурации вывода см. в разделе [как и где развертываются модели](how-to-deploy-and-where.md).

> [!NOTE]
> Число развертываемых моделей ограничено до 1 000 моделей на развертывание (для каждого контейнера).

<a id="using-the-cli"></a>

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import AksWebservice, Webservice
from azureml.core.model import Model

aks_target = AksCompute(ws,"myaks")
# If deploying to a cluster configured for dev/test, ensure that it was created with enough
# cores and memory to handle this deployment configuration. Note that memory is also used by
# things such as dependencies and AML components.
deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config, aks_target)
service.wait_for_deployment(show_output = True)
print(service.state)
print(service.get_logs())
```

Дополнительные сведения о классах, методах и параметрах, используемых в этом примере, см. в следующих справочных документах:

* [акскомпуте](/python/api/azureml-core/azureml.core.compute.aks.akscompute?preserve-view=true&view=azure-ml-py)
* [AksWebservice.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration?preserve-view=true&view=azure-ml-py)
* [Модель. Развертывание](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-)
* [Webservice.wait_for_deployment](/python/api/azureml-core/azureml.core.webservice%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truewait-for-deployment-show-output-false-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для развертывания с помощью интерфейса командной строки используйте следующую команду. Замените на `myaks` имя целевого объекта вычислений AKS. Замените на `mymodel:1` имя и версию зарегистрированной модели. Замените `myservice` именем для предоставления этой службе:

```azurecli-interactive
az ml model deploy -ct myaks -m mymodel:1 -n myservice -ic inferenceconfig.json -dc deploymentconfig.json
```

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aks-deploy-config.md)]

Дополнительные сведения см. в справочнике по [развертыванию модели языка AZ ML](/cli/azure/ext/azure-cli-ml/ml/model?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy) .

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Сведения об использовании VS Code см. в разделе [развертывание в AKS с помощью расширения VS Code](tutorial-train-deploy-image-classification-model-vscode.md#deploy-the-model).

> [!IMPORTANT]
> Для развертывания с помощью VS Code необходимо заранее создать кластер AKS или подключить его к рабочей области.

---

### <a name="autoscaling"></a>Автомасштабирование

Компонент, обрабатывающий Автомасштабирование для развертываний модели машинного обучения Azure, — azureml-Fe, который является маршрутизатором Smart. Так как все запросы на вывод проходят через него, у него есть необходимые данные для автоматического масштабирования развернутых моделей.

> [!IMPORTANT]
> * **Не включайте Kubernetes горизонтальный Автомасштабирование Pod (hPa) для развертывания моделей**. Это приведет к тому, что два компонента автоматического масштабирования будут конкурировать друг с другом. Azureml-Fe предназначен для автоматического масштабирования моделей, развернутых в МАШИНном обучении Azure, где HPA должен угадать или приблизительно использовать модель из общей метрики, такой как загрузка ЦП или Настраиваемая конфигурация метрик.
> 
> * **Azureml-Fe не масштабирует количество узлов в кластере AKS**, так как это может привести к увеличению затрат на непредвиденные затраты. Вместо этого **он масштабирует количество реплик для модели** в пределах границ физического кластера. Если необходимо масштабировать количество узлов в кластере, можно вручную масштабировать кластер или [настроить Автомасштабирование кластера AKS](../aks/cluster-autoscaler.md).

Автомасштабирование можно контролировать с помощью параметров `autoscale_target_utilization` , `autoscale_min_replicas` и `autoscale_max_replicas` для веб-службы AKS. В следующем примере показано, как включить Автомасштабирование.

```python
aks_config = AksWebservice.deploy_configuration(autoscale_enabled=True, 
                                                autoscale_target_utilization=30,
                                                autoscale_min_replicas=1,
                                                autoscale_max_replicas=4)
```

Решения для увеличения или уменьшения масштаба основываются на использовании текущих реплик контейнеров. Количество занятых реплик (обработка запроса), деленное на общее число текущих реплик, является текущим использованием. Если это число превышает `autoscale_target_utilization` , то создаются дополнительные реплики. Если он меньше, то реплики будут сокращены. По умолчанию использование целевого объекта составляет 70%.

Решения по добавлению реплик выполняются быстро и быстрее (около 1 секунды). Решения об удалении реплик являются консервативными (около 1 минуты).

Необходимые реплики можно вычислить с помощью следующего кода:

```python
from math import ceil
# target requests per second
targetRps = 20
# time to process the request (in seconds)
reqTime = 10
# Maximum requests per container
maxReqPerContainer = 1
# target_utilization. 70% in this example
targetUtilization = .7

concurrentRequests = targetRps * reqTime / targetUtilization

# Number of container replicas
replicas = ceil(concurrentRequests / maxReqPerContainer)
```

Дополнительные сведения о настройке `autoscale_target_utilization` , `autoscale_max_replicas` и см `autoscale_min_replicas` . в справочнике по модулю [аксвебсервице](/python/api/azureml-core/azureml.core.webservice.akswebservice?preserve-view=true&view=azure-ml-py) .

## <a name="deploy-models-to-aks-using-controlled-rollout-preview"></a>Развертывание моделей в AKS с помощью контролируемого развертывания (Предварительная версия)

Анализируйте и передвигайте версии модели в контролируемом виде с помощью конечных точек. Можно развернуть до шести версий за одну конечную точку. Конечные точки предоставляют следующие возможности.

* Настройте __процент трафика оценки, отправляемого в каждую конечную точку__. Например, маршрут 20% трафика к конечной точке "Test" и 80% в "Production".

    > [!NOTE]
    > Если не учитывать 100% трафика, оставшийся процент направляется к версии конечной точки __по умолчанию__ . Например, если настроить конечную точку "Test" для получения 10% трафика, а "произ." — 30%, оставшиеся 60% будут отправлены в конечную точку по умолчанию.
    >
    > Первая созданная версия конечной точки автоматически настраивается по умолчанию. Это можно изменить, задав параметр `is_default=True` при создании или обновлении версии конечной точки.
     
* Разметка версии конечной точки как __управления__ или __обработки__. Например, текущая версия рабочей конечной точки может быть элементом управления, а потенциальные новые модели развертываются как версии лечения. После оценки производительности версий лечения, если одна из них выполняет текущий элемент управления, она может быть перенесена в новый рабочий или элемент управления.

    > [!NOTE]
    > У вас может быть только __один__ элемент управления. Можно использовать несколько использование.

Вы можете включить Application Insights, чтобы просматривать операционные метрики конечных точек и развернутых версий.

### <a name="create-an-endpoint"></a>Создание конечной точки
Когда вы будете готовы к развертыванию моделей, создайте конечную точку оценки и разверните свою первую версию. В следующем примере показано, как развернуть и создать конечную точку с помощью пакета SDK. Первое развертывание будет определено как версия по умолчанию. Это означает, что неопределенный процент по всем версиям будет переходить к версии по умолчанию.  

> [!TIP]
> В следующем примере конфигурация устанавливает начальную версию конечной точки, чтобы обрабатывалась 20% трафика. Так как это первая конечная точка, она также является версией по умолчанию. А так как у нас нет других версий для других 80% трафика, он также направляется по умолчанию. До тех пор, пока не будут развернуты другие версии, которые принимают процент трафика, он получает 100% трафика.

```python
import azureml.core,
from azureml.core.webservice import AksEndpoint
from azureml.core.compute import AksCompute
from azureml.core.compute import ComputeTarget
# select a created compute
compute = ComputeTarget(ws, 'myaks')

# define the endpoint and version name
endpoint_name = "mynewendpoint"
version_name= "versiona"
# create the deployment config and define the scoring traffic percentile for the first deployment
endpoint_deployment_config = AksEndpoint.deploy_configuration(cpu_cores = 0.1, memory_gb = 0.2,
                                                              enable_app_insights = True,
                                                              tags = {'sckitlearn':'demo'},
                                                              description = "testing versions",
                                                              version_name = version_name,
                                                              traffic_percentile = 20)
 # deploy the model and endpoint
 endpoint = Model.deploy(ws, endpoint_name, [model], inference_config, endpoint_deployment_config, compute)
 # Wait for he process to complete
 endpoint.wait_for_deployment(True)
 ```

### <a name="update-and-add-versions-to-an-endpoint"></a>Обновление и добавление версий в конечную точку

Добавьте еще одну версию в конечную точку и настройте оценку трафика оценки до версии. Существует два типа версий: элемент управления и версия лечения. Для сравнения с одной версией элемента управления может существовать несколько версий лечения.

> [!TIP]
> Вторая версия, созданная следующим фрагментом кода, принимает 10% трафика. Первая версия настраивается на 20%, поэтому для конкретных версий настраивается только 30% трафика. Остальные 70% отправляются в первую версию конечной точки, так как она также является версией по умолчанию.

 ```python
from azureml.core.webservice import AksEndpoint

# add another model deployment to the same endpoint as above
version_name_add = "versionb"
endpoint.create_version(version_name = version_name_add,
                        inference_config=inference_config,
                        models=[model],
                        tags = {'modelVersion':'b'},
                        description = "my second version",
                        traffic_percentile = 10)
endpoint.wait_for_deployment(True)
```

Обновите существующие версии или удалите их в конечной точке. Можно изменить тип по умолчанию для версии, тип элемента управления и значение интенсивности трафика. В следующем примере вторая версия увеличивает трафик на 40% и теперь является значением по умолчанию.

> [!TIP]
> После выполнения следующего фрагмента кода вторая версия будет по умолчанию. Теперь она настроена на 40%, а исходная версия по-прежнему настроена на 20%. Это означает, что 40% трафика не учитывается конфигурациями версий. Остаточный трафик будет направляться во вторую версию, так как теперь он по умолчанию. Он эффективно получает 80% трафика.

 ```python
from azureml.core.webservice import AksEndpoint

# update the version's scoring traffic percentage and if it is a default or control type
endpoint.update_version(version_name=endpoint.versions["versionb"].name,
                        description="my second version update",
                        traffic_percentile=40,
                        is_default=True,
                        is_control_version_type=True)
# Wait for the process to complete before deleting
endpoint.wait_for_deployment(true)
# delete a version in an endpoint
endpoint.delete_version(version_name="versionb")

```

## <a name="web-service-authentication"></a>Проверка подлинности веб-службы

При развертывании в службе Kubernetes Azure проверка подлинности __на основе ключей__ включена по умолчанию. Также можно включить проверку подлинности __на основе маркеров__ . Для аутентификации на основе маркеров клиенты должны использовать учетную запись Azure Active Directory для запроса маркера проверки подлинности, который используется для выполнения запросов к развернутой службе.

Чтобы __Отключить__ проверку подлинности, задайте `auth_enabled=False` параметр при создании конфигурации развертывания. В следующем примере отключается проверка подлинности с помощью пакета SDK:

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, auth_enabled=False)
```

Сведения о проверке подлинности в клиентском приложении см. в разделе [использование машинное обучение Azureной модели, развернутой в качестве веб-службы](how-to-consume-web-service.md).

### <a name="authentication-with-keys"></a>Проверка подлинности с помощью ключей

Если включена проверка подлинности ключа, можно использовать `get_keys` метод для получения первичного и вторичного ключа проверки подлинности:

```python
primary, secondary = service.get_keys()
print(primary)
```

> [!IMPORTANT]
> Если необходимо повторно создать ключ, используйте [`service.regen_key`](/python/api/azureml-core/azureml.core.webservice%28class%29?preserve-view=true&view=azure-ml-py)

### <a name="authentication-with-tokens"></a>Аутентификация с помощью токенов

Чтобы включить проверку подлинности на маркере, задайте `token_auth_enabled=True` параметр при создании или обновлении развертывания. В следующем примере активируется проверка подлинности на основе маркеров с помощью пакета SDK:

```python
deployment_config = AksWebservice.deploy_configuration(cpu_cores=1, memory_gb=1, token_auth_enabled=True)
```

Если включена проверка подлинности токенов, можно использовать `get_token` метод для получения маркера JWT и срока действия маркера:

```python
token, refresh_by = service.get_token()
print(token)
```

> [!IMPORTANT]
> После времени маркера потребуется запросить новый токен `refresh_by` .
>
> Корпорация Майкрософт настоятельно рекомендует создать рабочую область Машинное обучение Azure в том же регионе, что и кластер службы Azure Kubernetes. Для проверки подлинности с помощью маркера веб-служба выполняет вызов в регион, в котором создана рабочая область Машинного обучения Azure. Если регион рабочей области недоступен, вы не сможете получить маркер для веб-службы даже в том случае, если кластер находится в регионе, отличном от региона рабочей области. Это фактически приводит к недоступности проверки подлинности на основе маркеров, пока регион рабочей области не будет снова доступен. Кроме того, чем больше расстояние между регионом кластера и регионом рабочей области, тем дольше будет получена лексема.
>
> Чтобы получить маркер, необходимо использовать пакет SDK для Машинное обучение Azure или команду [AZ ML Service Get-Access-Token](/cli/azure/ext/azure-cli-ml/ml/service?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-service-get-access-token) .


### <a name="vulnerability-scanning"></a>Сканирование уязвимостей

Центр безопасности Azure обеспечивает унифицированное управление безопасностью и расширенную защиту от угроз для гибридных облачных рабочих нагрузок. Вы должны разрешить центру безопасности Azure проверять ресурсы и следовать рекомендациям. Дополнительные сведения см. в статье [Интеграция Azure Kubernetes Services с центром безопасности](../security-center/defender-for-kubernetes-introduction.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Использование Azure RBAC для авторизации Kubernetes](../aks/manage-azure-rbac.md)
* [Безопасная организация в среде с виртуальной сетью Azure](how-to-secure-inferencing-vnet.md)
* [Развертывание модели с помощью пользовательского образа DOCKER](how-to-deploy-custom-docker-image.md)
* [Устранение неполадок развертывания](how-to-troubleshoot-deployment.md)
* [Обновление веб-службы](how-to-deploy-update-web-service.md)
* [Использование TLS для защиты веб-службы с помощью Машинного обучения Azure](how-to-secure-web-service.md).
* [Использование модели Машинного обучения Azure, развернутой в качестве веб-службы](how-to-consume-web-service.md)
* [Мониторинг моделей Машинное обучение Azure с помощью Application Insights](how-to-enable-app-insights.md)
* [Сбор данных для моделей в рабочей среде](how-to-enable-data-collection.md)

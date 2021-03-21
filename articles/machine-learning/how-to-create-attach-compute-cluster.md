---
title: Создание вычислительных кластеров
titleSuffix: Azure Machine Learning
description: Узнайте, как создавать кластеры вычислений в рабочей области Машинное обучение Azure. Используйте кластер вычислений в качестве целевого объекта вычислений для обучения или вывода.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.author: sgilley
author: sdgilley
ms.reviewer: sgilley
ms.date: 10/02/2020
ms.openlocfilehash: c11176f0c7760e76b755406bda96b72b302f8857
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102506945"
---
# <a name="create-an-azure-machine-learning-compute-cluster"></a>Создание вычислительного кластера Машинного обучения Azure

Узнайте, как создать и управлять [кластером](concept-compute-target.md#azure-machine-learning-compute-managed) в рабочей области машинное обучение Azure.

Вы можете использовать Машинное обучение Azure вычислительный кластер для распределения процесса обучения или выполнения пакетного вывода в кластере вычислительных узлов ЦП или GPU в облаке. Дополнительные сведения о размерах виртуальных машин, которые содержат GPU, см. в статье [Размеры виртуальных машин, оптимизированных для GPU](../virtual-machines/sizes-gpu.md). 

В этой статье раскрываются следующие темы:

* Создание вычислительного кластера
* Снижение стоимости вычислений в кластере
* Настройка [управляемого удостоверения](../active-directory/managed-identities-azure-resources/overview.md) для кластера

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

* [Расширение Azure CLI для службы машинное обучение](reference-azure-machine-learning-cli.md), [машинное обучение Azure пакет SDK для Python](/python/api/overview/azure/ml/intro)или [расширение машинное обучение Azure Visual Studio Code](tutorial-setup-vscode-extension.md).

## <a name="what-is-a-compute-cluster"></a>Что такое Кластер вычислений?

Кластер Машинное обучение Azure COMPUTE — это управляемая инфраструктура вычислений, которая позволяет легко создавать одно или несколько вычислений. Она создается в регионе вашей рабочей области и является ресурсом, который можно использовать совместно с другими пользователями в этой рабочей области. Вычислительная среда автоматически масштабируется при отправке задания, а также может быть размещена в виртуальной сети Azure. Она выполняется в контейнерной среде, упаковывая зависимости вашей модели в [контейнер Docker](https://www.docker.com/why-docker).

Кластеры вычислений могут безопасно выполнять задания в [среде виртуальной сети](how-to-secure-training-vnet.md), не требуя, чтобы предприятия открывали порты SSH. Задание выполняется в контейнерной среде и упаковывает зависимости модели в контейнер DOCKER. 

## <a name="limitations"></a>Ограничения

* **Не создавайте несколько одновременных вложений для одного и того же вычислений** из рабочей области. Например, можно присоединить один кластер вычислений к рабочей области, используя два разных имени. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.

    Если требуется повторно подключить целевой объект вычислений, например для изменения параметров конфигурации кластера, необходимо сначала удалить существующее вложение.

* Некоторые сценарии, перечисленные в этом документе, помечены как __Предварительная версия__. Функции предварительной версии предоставляются без соглашения об уровне обслуживания и не рекомендуются для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

* Вычислительная среда Машинного обучения Azure имеет ограничения по умолчанию, такие как количество ядер, которые могут быть выделены. Дополнительные сведения см. в статье [Управление квотами для ресурсов Azure и их запрашивание](how-to-manage-quotas.md).

* Azure позволяет размещать _блокировки_ ресурсов, чтобы их нельзя было удалить или использовать только для чтения. __Не применяйте блокировки ресурсов к группе ресурсов, содержащей рабочую область__. Применение блокировки к группе ресурсов, содержащей рабочую область, не позволит выполнять операции масштабирования для вычислений с помощью кластеров машинного обучения Azure. Дополнительные сведения о блокировке ресурсов см. [в разделе Блокировка ресурсов для предотвращения непредвиденных изменений](../azure-resource-manager/management/lock-resources.md).

> [!TIP]
> Кластеры обычно масштабируются до 100 узлов при наличии достаточной квоты для требуемого числа ядер. По умолчанию кластеры настроены с поддержкой взаимодействия между их узлами, например для поддержки заданий MPI. Однако вы можете масштабировать кластеры до тысяч узлов, просто [вызывая запрос в службу поддержки](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)и запрашивая разрешение на получение списка подписки, рабочей области или конкретного кластера для отключения взаимодействия между узлами. 


## <a name="create"></a>Создать

**Оценка времени**: приблизительно 5 минут.

Вычислительную среду Машинного обучения Azure можно многократно использовать при различных запусках. Результаты вычислений могут использоваться совместно с другими пользователями в рабочей области. Они сохраняются для последующих запусков, а число узлов в них автоматически масштабируется в зависимости от числа отправленных запросов на запуск и заданного для кластера значения max_nodes. С помощью параметра min_nodes можно контролировать минимально доступное количество узлов.

Выделенные ядра для каждого региона на квоту семейства виртуальных машин и общая региональная квота, применяемая к созданию вычислительных кластеров, объединены и совместно используются в рамках квоты на вычислительные экземпляры Машинное обучение Azure обучения. 

[!INCLUDE [min-nodes-note](../../includes/machine-learning-min-nodes.md)]

Когда вычислительная среда не используется, она автоматически масштабируется до нуля узлов.   Выделенные виртуальные машины создаются для выполнения заданий по мере необходимости.
    
# <a name="python"></a>[Python](#tab/python)

Чтобы создать постоянный ресурс Вычислительной среды машинного обучения Azure в Python, необходимо указать свойства **vm_size** и **max_nodes**. Затем Машинное обучение Azure использует интеллектуальные значения по умолчанию для остальных свойств. 
    
* **vm_size**. Семейство виртуальных машин узлов, создаваемых Вычислительной средой Машинного обучения Azure.
* **max_nodes**. Максимальное количество узлов для автомасштабирования при запуске задания в Вычислительной среде Машинного обучения Azure.


[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=cpu_cluster)]

При создании Вычислительной среды машинного обучения Azure вы можете также настроить несколько дополнительных свойств. Эти свойства позволяют создать постоянный кластер фиксированного размера, который может размещаться в существующей виртуальной сети Azure в вашей подписке.  Дополнительные сведения см. в разделе о [классе AmlCompute](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute).


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)


```azurecli-interactive
az ml computetarget create amlcompute -n cpu --min-nodes 1 --max-nodes 1 -s STANDARD_D3_V2
```

См. дополнительные сведения о команде [az ml computetarget create amlcompute](/cli/azure/ext/azure-cli-ml/ml/computetarget/create#ext-azure-cli-ml-az-ml-computetarget-create-amlcompute).

# <a name="studio"></a>[Студия](#tab/azure-studio)

Сведения о создании кластера вычислений в студии см. в статье [Создание целевых объектов вычислений в машинное обучение Azure Studio](how-to-create-attach-compute-studio.md#amlcompute).

---

 ## <a name="lower-your-compute-cluster-cost"></a><a id="low-pri-vm"></a> Снижение стоимости вычислений в кластере

Вы также можете использовать [виртуальные машины с низким приоритетом](concept-plan-manage-cost.md#low-pri-vm) для выполнения некоторых или всех рабочих нагрузок. Эти виртуальные машины не обеспечены гарантиями доступности и могут быть замещены при использовании. Вам потребуется перезапустить задание с вытеснением. 

Используйте любой из этих способов, чтобы указать виртуальную машину с низким приоритетом:
    
# <a name="python"></a>[Python](#tab/python)
    
```python
compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            vm_priority='lowpriority',
                                                            max_nodes=4)
```
    
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Задайте `vm-priority` :
    
```azurecli-interactive
az ml computetarget create amlcompute --name lowpriocluster --vm-size Standard_NC6 --max-nodes 5 --vm-priority lowpriority
```

# <a name="studio"></a>[Студия](#tab/azure-studio)

В студии при создании виртуальной машины выберите **низкий приоритет** .

--- 

## <a name="set-up-managed-identity"></a><a id="managed-identity"></a> Настройка управляемого удостоверения

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-intro.md)]

# <a name="python"></a>[Python](#tab/python)

* Настройте управляемое удостоверение в конфигурации подготовки:  

    * Управляемое удостоверение, назначенное системой:
        ```python
        # configure cluster with a system-assigned managed identity
        compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                                max_nodes=5,
                                                                identity_type="SystemAssigned",
                                                                )
        ```
    
    * Управляемое удостоверение, назначаемое пользователем:
    
        ```python
        # configure cluster with a user-assigned managed identity
        compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                                max_nodes=5,
                                                                identity_type="UserAssigned",
                                                                identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])
    
        cpu_cluster_name = "cpu-cluster"
        cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
        ```

* Добавление управляемого удостоверения в существующий кластер вычислений 
    
    * Управляемое удостоверение, назначаемое системой:
    
        ```python
        # add a system-assigned managed identity
        cpu_cluster.add_identity(identity_type="SystemAssigned")
        ````
    
    * Управляемое удостоверение, назначаемое пользователем:
    
        ```python
        # add a user-assigned managed identity
        cpu_cluster.add_identity(identity_type="UserAssigned", 
                                    identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])
        ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

* Создание нового управляемого кластера вычислений с управляемым удостоверением

  * Управляемое удостоверение, назначаемое пользователем

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
    ```

  * Управляемое удостоверение, назначаемое системой

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '[system]'
    ```
* Добавление управляемого удостоверения в существующий кластер:

    * Управляемое удостоверение, назначаемое пользователем
        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
        ```
    * Управляемое удостоверение, назначаемое системой

        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '[system]'
        ```

# <a name="studio"></a>[Студия](#tab/azure-studio)

См. раздел [Настройка управляемого удостоверения в студии](how-to-create-attach-compute-studio.md#managed-identity).

---

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-note.md)]

### <a name="managed-identity-usage"></a>Использование управляемого удостоверения

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-default.md)]

## <a name="troubleshooting"></a>Устранение неполадок

Существует вероятность того, что некоторые пользователи, которые создали свою Машинное обучение Azureную рабочую область из портал Azure до общедоступного выпуска, не смогут создать Амлкомпуте в этой рабочей области. Вы можете либо отправить запрос в службу поддержки, либо создать новую рабочую область на портале или с помощью пакета SDK, чтобы немедленно разблокировать пользователя.

Если при изменении размера узла (0-> 0) в кластере Машинное обучение Azure будет заблокировано, это может быть вызвано блокировкой ресурсов Azure.

[!INCLUDE [resource locks](../../includes/machine-learning-resource-lock.md)]

## <a name="next-steps"></a>Дальнейшие действия

Используйте в качестве среды вычислений кластер:

* [Отправить обучающий запуск](how-to-set-up-training-targets.md) 
* [Выполнение вывода пакета](./tutorial-pipeline-batch-scoring-classification.md).

---
title: Создание вычислительного экземпляра и управление им
titleSuffix: Azure Machine Learning
description: Узнайте, как создать вычислительный экземпляр Машинное обучение Azure и управлять им. Используйте в качестве среды разработки или в качестве целевого объекта вычислений для целей разработки и тестирования.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.author: sgilley
author: sdgilley
ms.reviewer: sgilley
ms.date: 10/02/2020
ms.openlocfilehash: 2778f52b312e5d2fda7879b834fcd204285b7144
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105628957"
---
# <a name="create-and-manage-an-azure-machine-learning-compute-instance"></a>Создание Машинное обучение Azure вычислительного экземпляра и управление им

Узнайте, как создать [вычислительный экземпляр](concept-compute-instance.md) и управлять им в рабочей области машинное обучение Azure.

Используйте вычислительный экземпляр в качестве полностью настроенной и управляемой среды разработки в облаке. Для разработки и тестирования можно также использовать экземпляр в качестве [цели для обучающих вычислений](concept-compute-target.md#train) или для [целевого объекта вывода](concept-compute-target.md#deploy).   Вычислительный экземпляр может параллельно запускать несколько заданий и иметь очередь заданий. В качестве среды разработки вычислительный экземпляр не может использоваться совместно с другими пользователями в рабочей области.

Вы узнаете, как выполнять следующие задачи:

* Создание экземпляра вычислений 
* Управление (запуск, завершение, перезапуск, удаление) вычислительного экземпляра
* Доступ к окну терминала 
* Установка пакетов R или Python
* Создание новых сред или ядер Jupyter

Экземпляры вычислений могут безопасно выполнять задания в [среде виртуальной сети](how-to-secure-training-vnet.md), не требуя, чтобы предприятия открывали порты SSH. Задание выполняется в контейнерной среде и упаковывает зависимости модели в контейнер DOCKER. 

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

* [Расширение Azure CLI для службы машинное обучение](reference-azure-machine-learning-cli.md), [машинное обучение Azure пакет SDK для Python](/python/api/overview/azure/ml/intro)или [расширение машинное обучение Azure Visual Studio Code](tutorial-setup-vscode-extension.md).

## <a name="create"></a>Создать

**Оценка времени**: приблизительно 5 минут.

Создание вычислительного экземпляра выполняется один раз для рабочей области. Можно повторно использовать COMPUTE как рабочую станцию разработки или как целевой объект вычислений для обучения. К рабочей области можно присоединить несколько экземпляров вычислений.

Выделенные ядра для каждого региона на каждую квоту семейства виртуальных машин и общая региональная квота, применяемая к созданию вычислительных экземпляров, объединены и используются совместно с Машинное обучение Azureной квотой вычислительных кластеров для обучения. При остановке вычислительного экземпляра квота не освобождается для того, чтобы вы могли перезапустить вычислительный экземпляр. Обратите внимание, что после создания вычислительного экземпляра невозможно изменить размер виртуальной машины.

В следующем примере показано, как создать вычислительный экземпляр:

# <a name="python"></a>[Python](#tab/python)

```python
import datetime
import time

from azureml.core.compute import ComputeTarget, ComputeInstance
from azureml.core.compute_target import ComputeTargetException

# Choose a name for your instance
# Compute instance name should be unique across the azure region
compute_name = "ci{}".format(ws._workspace_id)[:10]

# Verify that instance does not exist already
try:
    instance = ComputeInstance(workspace=ws, name=compute_name)
    print('Found existing instance, use it.')
except ComputeTargetException:
    compute_config = ComputeInstance.provisioning_configuration(
        vm_size='STANDARD_D3_V2',
        ssh_public_access=False,
        # vnet_resourcegroup_name='<my-resource-group>',
        # vnet_name='<my-vnet-name>',
        # subnet_name='default',
        # admin_user_ssh_public_key='<my-sshkey>'
    )
    instance = ComputeInstance.create(ws, compute_name, compute_config)
    instance.wait_for_completion(show_output=True)
```

Дополнительные сведения о классах, методах и параметрах, используемых в этом примере, см. в следующих справочных документах:

* [Класс Компутеинстанце](/python/api/azureml-core/azureml.core.compute.computeinstance.computeinstance)
* [ComputeTarget. Create](/python/api/azureml-core/azureml.core.compute.computetarget#create-workspace--name--provisioning-configuration-)
* [ComputeInstance.wait_for_completion](/python/api/azureml-core/azureml.core.compute.computeinstance(class)#wait-for-completion-show-output-false--is-delete-operation-false-)


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az ml computetarget create computeinstance  -n instance -s "STANDARD_D3_V2" -v
```

Дополнительные сведения см. в справке по команде [AZ ML computetarget Create компутеинстанце](/cli/azure/ext/azure-cli-ml/ml/computetarget/create#ext_azure_cli_ml_az_ml_computetarget_create_computeinstance) .

# <a name="studio"></a>[Studio](#tab/azure-studio)

В рабочей области Машинное обучение Azure Studio создайте новый вычислительный экземпляр из раздела **вычислений** или **записных книжек** , когда будете готовы запустить одну из ваших записных книжек.

Сведения о создании вычислительного экземпляра в студии см. в статье [Создание целевых объектов вычислений в машинное обучение Azure Studio](how-to-create-attach-compute-studio.md#compute-instance).

---

Вы также можете создать вычислительный экземпляр с помощью [шаблона Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-computeinstance). 

### <a name="create-on-behalf-of-preview"></a>Создание от имени (Предварительная версия)

Как администратор вы можете создать вычислительный экземпляр от имени анализу данных и присвоить ему экземпляр с помощью:
* [Шаблон Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-computeinstance).  Дополнительные сведения о том, как найти ИД клиента и идентификатор объекта, необходимые в этом шаблоне, см. в разделе [Поиск кодов объектов удостоверений для конфигурации проверки подлинности](../healthcare-apis/fhir/find-identity-object-ids.md).  Эти значения также можно найти на портале Azure Active Directory.
* REST API

Анализу данных, для которых создается вычислительный экземпляр, должны быть предоставлены следующие разрешения на [Управление доступом на основе ролей (Azure RBAC)](../role-based-access-control/overview.md) : 
* *Microsoft. Мачинелеарнингсервицес/рабочие области/вычислений/запуск/действие*
* *Microsoft. Мачинелеарнингсервицес/рабочие области/вычислений/действие*
* *Microsoft. Мачинелеарнингсервицес/рабочие области/вычислений/перезагрузка/действие*
* *Microsoft. Мачинелеарнингсервицес/рабочие области/вычислений/аппликатионакцесс/действие*

Анализу данных может запускать, прекращать и перезапускать вычислительный экземпляр. Они могут использовать вычислительный экземпляр для:
* Jupyter
* JupyterLab
* RStudio
* Интегрированные записные книжки

## <a name="manage"></a>Управление

Запуск, завершение, перезапуск и удаление вычислительного экземпляра. Вычислительный экземпляр не масштабируется автоматически, поэтому убедитесь, что ресурс остановлен, чтобы предотвратить постоянную плату.

> [!TIP]
> Размер диска ОС вычислительного экземпляра составляет 120 ГБ. Если на диске заканчивается свободное место, [Используйте терминал](how-to-access-terminal.md) , чтобы очистить по крайней мере 1-2 ГБ перед остановкой или перезапуском вычислительного экземпляра.

# <a name="python"></a>[Python](#tab/python)

В приведенных ниже примерах имя вычислительного экземпляра — **instance** .

* Получение состояния

    ```python
    # get_status() gets the latest status of the ComputeInstance target
    instance.get_status()
    ```

* Stop

    ```python
    # stop() is used to stop the ComputeInstance
    # Stopping ComputeInstance will stop the billing meter and persist the state on the disk.
    # Available Quota will not be changed with this operation.
    instance.stop(wait_for_completion=True, show_output=True)
    ```

* Начать

    ```python
    # start() is used to start the ComputeInstance if it is in stopped state
    instance.start(wait_for_completion=True, show_output=True)
    ```

* Перезагрузить

    ```python
    # restart() is used to restart the ComputeInstance
    instance.restart(wait_for_completion=True, show_output=True)
    ```

* Удалить

    ```python
    # delete() is used to delete the ComputeInstance target. Useful if you want to re-use the compute name 
    instance.delete(wait_for_completion=True, show_output=True)
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

В приведенных ниже примерах имя вычислительного экземпляра — **instance** .

* Stop

    ```azurecli-interactive
    az ml computetarget stop computeinstance -n instance -v
    ```

    Дополнительные сведения см. в разделе [AZ ML computetarget останавливает компутеинстанце](/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance#ext-azure-cli-ml-az-ml-computetarget-computeinstance-stop).

* Начать 

    ```azurecli-interactive
    az ml computetarget start computeinstance -n instance -v
    ```

    Дополнительные сведения см. в статье [AZ ML computetarget Start компутеинстанце](/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance#ext-azure-cli-ml-az-ml-computetarget-computeinstance-start).

* Перезагрузить 

    ```azurecli-interactive
    az ml computetarget restart computeinstance -n instance -v
    ```

    Дополнительные сведения см. в статье [AZ ML computetarget Restart компутеинстанце](/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance#ext-azure-cli-ml-az-ml-computetarget-computeinstance-restart).

* Удалить

    ```azurecli-interactive
    az ml computetarget delete -n instance -v
    ```

    Дополнительные сведения см. в разделе [AZ ML computetarget Delete компутеинстанце](/cli/azure/ext/azure-cli-ml/ml/computetarget#ext-azure-cli-ml-az-ml-computetarget-delete).

# <a name="studio"></a>[Studio](#tab/azure-studio)

В рабочей области в студии Машинного обучения Azure выберите **Вычисление**, а затем вверху выберите **Вычислительный экземпляр**.

![Управление вычислительным экземпляром](./media/concept-compute-instance/manage-compute-instance.png)

Вы можете выполнять следующие действия.

* Создание нового вычислительного экземпляра 
* Обновите вкладку "вычисленные экземпляры".
* Запустите, приостанавливает и перезапустите вычислительный экземпляр.  Вы платите за экземпляр при его запуске. Останавливает вычислительный экземпляр, если он не используется для снижения затрат. Остановка вычислительного экземпляра освобождает его. При необходимости снова запустите ее.
* Удаление вычислительного экземпляра.
* Отфильтруйте список экземпляров вычислений, чтобы отображались только созданные вами.

Для каждого вычислительного экземпляра в рабочей области, созданного (или созданного для вас), можно:

* Доступ к Jupyter, JupyterLab или RStudio в вычислительном экземпляре.
* Подключение к вычислительному экземпляру по протоколу SSH. Доступ по протоколу SSH по умолчанию отключен, но его можно включить во время создания вычислительного экземпляра. Доступ по протоколу SSH осуществляется с помощью открытого и закрытого ключей. На вкладке приводятся подробные сведения об SSH-подключении, такие как IP-адрес, имя пользователя и номер порта.
* Получение сведений об определенном вычислительном экземпляре, таких как IP-адрес и регион.

---


[Azure RBAC](../role-based-access-control/overview.md) позволяет контролировать, какие пользователи в рабочей области могут создавать, удалять, запускать, останавливаться и перезапускать вычислительные экземпляры. Все пользователи с ролью участника или владельца рабочей области могут создавать, удалять, запускать, останавливать и перезапускать вычислительные экземпляры в рабочей области. Однако доступ к Jupyter, JupyterLab и RStudio на этом вычислительном экземпляре разрешен только создателю определенного вычислительного экземпляра или назначенному пользователю, если он был создан от их имени. Вычислительный экземпляр предназначен для одного пользователя с правами root и может проходить через Jupyter/JupyterLab/RStudio. В вычислительном экземпляре будет входить однопользовательский вход, и все действия будут использовать удостоверение этого пользователя для Azure RBAC и его атрибуты. Доступ по протоколу SSH контролируется с помощью открытого и закрытого ключей.

Эти действия можно контролировать с помощью Azure RBAC:
* *Microsoft.MachineLearningServices/workspaces/computes/read*
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/computes/delete*
* *Microsoft. Мачинелеарнингсервицес/рабочие области/вычислений/запуск/действие*
* *Microsoft. Мачинелеарнингсервицес/рабочие области/вычислений/действие*
* *Microsoft. Мачинелеарнингсервицес/рабочие области/вычислений/перезагрузка/действие*

## <a name="next-steps"></a>Дальнейшие шаги

* [Доступ к терминалу вычислительных экземпляров](how-to-access-terminal.md)
* [Создание файлов и управление ими](how-to-manage-files.md)
* [Отправить обучающий запуск](how-to-set-up-training-targets.md)

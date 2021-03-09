---
title: Настройка частной конечной точки
titleSuffix: Azure Machine Learning
description: Используйте частную ссылку Azure для безопасного доступа к рабочей области Машинное обучение Azure из виртуальной сети.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 02/09/2021
ms.openlocfilehash: 752f38c0a44d40f2bf7e7b5961dca7743fbf4b76
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102488041"
---
# <a name="configure-azure-private-link-for-an-azure-machine-learning-workspace"></a>Настройка частной ссылки Azure для Машинное обучение Azure рабочей области

В этом документе вы узнаете, как использовать частную связь Azure с рабочей областью Машинное обучение Azure. Сведения о создании виртуальной сети для Машинное обучение Azure см. в статье [Общие сведения о изоляции и конфиденциальности виртуальной сети](how-to-network-security-overview.md) .

Частная ссылка Azure позволяет подключаться к рабочей области с помощью частной конечной точки. Частная конечная точка — это набор частных IP-адресов в виртуальной сети. Затем можно ограничить доступ к рабочей области, чтобы она выполнялась только по частным IP-адресам. Частная ссылка помогает снизить риск утечка данных. Дополнительные сведения о частных конечных точках см. в статье [Приватный канал Azure](../private-link/private-link-overview.md).

> [!IMPORTANT]
> Частная ссылка Azure не влияет на плоскость управления Azure (операции управления), такие как удаление рабочей области или управление ресурсами вычислений. Например, создание, обновление или удаление целевого объекта вычислений. Эти операции выполняются через общедоступный Интернет в нормальном режиме. Операции с плоскостью данных, такие как использование Машинное обучение Azure Studio, API (включая опубликованные конвейеры) или пакет SDK, используют закрытую конечную точку.
>
> При использовании Mozilla Firefox могут возникнуть проблемы при попытке доступа к закрытой конечной точке рабочей области. Эта проблема может быть связана с DNS по протоколу HTTPS в Mozilla. В качестве обходного пути рекомендуется использовать Microsoft ребро Google Chrome.

## <a name="prerequisites"></a>Предварительные требования

* Если вы планируете использовать рабочую область с включенной частной связью с ключом, управляемым клиентом, необходимо запросить эту функцию с помощью запроса в службу поддержки. Дополнительные сведения см. в статье [Управление квотами и их увеличение](how-to-manage-quotas.md#private-endpoint-and-private-dns-quota-increases).

* Для создания частной конечной точки в необходимо наличие существующей виртуальной сети. Необходимо также [отключить сетевые политики для частных конечных точек](../private-link/disable-private-endpoint-network-policy.md) перед добавлением частной конечной точки.
## <a name="limitations"></a>Ограничения

* Использование рабочей области Машинное обучение Azure с частной ссылкой недоступна в регионах Azure для государственных организаций.
* Если вы включили общий доступ для рабочей области, защищенной с помощью частного канала, и используете Машинное обучение Azure Studio через общедоступный Интернет, некоторые функции, такие как конструктор, могут не получить доступ к данным. Эта проблема возникает, когда данные хранятся в службе, защищенной за пределы виртуальной сети. Например, учетная запись хранения Azure.

## <a name="create-a-workspace-that-uses-a-private-endpoint"></a>Создание рабочей области, использующей закрытую конечную точку

Используйте один из следующих методов, чтобы создать рабочую область с частной конечной точкой. Для каждого из этих методов __требуется существующая виртуальная сеть__.

> [!TIP]
> Если вы хотите одновременно создать рабочую область, частную конечную точку и виртуальную сеть, см. раздел [Использование шаблона Azure Resource Manager для создания рабочей области для машинное обучение Azure](how-to-create-workspace-template.md).

# <a name="python"></a>[Python](#tab/python)

Пакет SDK для Машинное обучение Azure Python предоставляет класс [приватиндпоинтконфиг](/python/api/azureml-core/azureml.core.privateendpointconfig?view=azure-ml-py) , который можно использовать с [рабочей областью. Create ()](/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---tags-none--friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--adb-workspace-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--private-endpoint-config-none--private-endpoint-auto-approval-true--exist-ok-false--show-output-true-) для создания рабочей области с закрытой конечной точкой. Для этого класса требуется существующая виртуальная сеть.

```python
from azureml.core import Workspace
from azureml.core import PrivateEndPointConfig

pe = PrivateEndPointConfig(name='myprivateendpoint', vnet_name='myvnet', vnet_subnet_name='default')
ws = Workspace.create(name='myworkspace',
    subscription_id='<my-subscription-id>',
    resource_group='myresourcegroup',
    location='eastus2',
    private_endpoint_config=pe,
    private_endpoint_auto_approval=True,
    show_output=True)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Расширение Azure CLI для машинного обучения](reference-azure-machine-learning-cli.md) предоставляет команду [AZ ML Workspace Create](/cli/azure/ext/azure-cli-ml/ml/workspace#ext_azure_cli_ml_az_ml_workspace_create) . Следующие параметры этой команды можно использовать для создания рабочей области с частной сетью, но для нее требуется существующая виртуальная сеть.

* `--pe-name`— Имя создаваемой частной конечной точки.
* `--pe-auto-approval`— Следует ли автоматически утверждать подключения частной конечной точки к рабочей области.
* `--pe-resource-group`: Группа ресурсов для создания частной конечной точки в. Должна быть той же группой, которая содержит виртуальную сеть.
* `--pe-vnet-name`— Существующая виртуальная сеть для создания частной конечной точки в.
* `--pe-subnet-name`— Имя подсети, в которой создается частная конечная точка. Значение по умолчанию — `default`.

Эти параметры помимо других обязательных параметров для команды Create. Например, следующая команда создает новую рабочую область в регионе "Западная часть США", используя существующую группу ресурсов и виртуальную сеть:

```azurecli
az ml workspace create -r myresourcegroup \
    -l westus \
    -n myworkspace \
    --pe-name myprivateendpoint \
    --pe-auto-approval \
    --pe-resource-group myresourcegroup \
    --pe-vnet-name myvnet \
    --pe-subnet-name mysubnet
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

Вкладка " __сеть__ " в машинное обучение Azure Studio позволяет настроить частную конечную точку. Однако для этого требуется существующая виртуальная сеть. Дополнительные сведения см. [в статье Создание рабочих областей на портале](how-to-manage-workspace.md).

---

## <a name="add-a-private-endpoint-to-a-workspace"></a>Добавление частной конечной точки в рабочую область

Чтобы добавить частную конечную точку в существующую рабочую область, используйте один из следующих методов.

> [!WARNING]
>
> Если у вас есть существующие целевые объекты вычислений, связанные с этой рабочей областью, и они не находятся за той же виртуальной сетью, в которой Tha частная конечная точка, они не будут работать.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core import Workspace
from azureml.core import PrivateEndPointConfig

pe = PrivateEndPointConfig(name='myprivateendpoint', vnet_name='myvnet', vnet_subnet_name='default')
ws = Workspace.from_config()
ws.add_private_endpoint(private_endpoint_config=pe, private_endpoint_auto_approval=True, show_output=True)
```

Дополнительные сведения о классах и методах, используемых в этом примере, см. в разделе [приватиндпоинтконфиг](/python/api/azureml-core/azureml.core.privateendpointconfig?view=azure-ml-py) and [Workspace.add_private_endpoint](/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#add-private-endpoint-private-endpoint-config--private-endpoint-auto-approval-true--location-none--show-output-true--tags-none-).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Расширение Azure CLI для машинного обучения](reference-azure-machine-learning-cli.md) предоставляет команду [AZ ML Workspace Private-Endpoint Add](/cli/azure/ext/azure-cli-ml/ml/workspace/private-endpoint#ext_azure_cli_ml_az_ml_workspace_private_endpoint_add) .

```azurecli
az ml workspace private-endpoint add -w myworkspace  --pe-name myprivateendpoint --pe-auto-approval true --pe-vnet-name myvnet
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

В рабочей области Машинное обучение Azure на портале выберите __подключения к частным конечным точкам__ и выберите __+ Частная конечная точка__. Используйте поля для создания новой частной конечной точки.

* При выборе __региона__ выберите тот же регион, что и для виртуальной сети. 
* При выборе __типа ресурса__ используйте __Microsoft. мачинелеарнингсервицес или рабочие области__. 
* Задайте __ресурсу__ имя рабочей области.

Наконец, выберите __создать__ , чтобы создать частную конечную точку.

---

## <a name="remove-a-private-endpoint"></a>Удаление частной конечной точки

Чтобы удалить закрытую конечную точку из рабочей области, используйте один из следующих методов.

# <a name="python"></a>[Python](#tab/python)

Чтобы удалить закрытую конечную точку, используйте [Workspace.delete_private_endpoint_connection](/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#delete-private-endpoint-connection-private-endpoint-connection-name-) .

```python
from azureml.core import Workspace

ws = Workspace.from_config()
# get the connection name
_, _, connection_name = ws.get_details()['privateEndpointConnections'][0]['id'].rpartition('/')
ws.delete_private_endpoint_connection(private_endpoint_connection_name=connection_name)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Расширение Azure CLI для машинного обучения](reference-azure-machine-learning-cli.md) предоставляет команду [AZ ML Workspace Private-Endpoint Delete](/cli/azure/ext/azure-cli-ml/ml/workspace/private-endpoint#ext_azure_cli_ml_az_ml_workspace_private_endpoint_delete) .

# <a name="portal"></a>[Портал](#tab/azure-portal)

В рабочей области Машинное обучение Azure на портале выберите __подключения к частной конечной точке__, а затем выберите конечную точку, которую нужно удалить. Наконец, выберите __Удалить__.

---

## <a name="using-a-workspace-over-a-private-endpoint"></a>Использование рабочей области в частной конечной точке

Так как взаимодействие с рабочей областью разрешено только из виртуальной сети, все среды разработки, использующие эту рабочую область, должны быть членами виртуальной сети. Например, виртуальная машина в виртуальной сети.

> [!IMPORTANT]
> Чтобы избежать временного сбоя подключения, корпорация Майкрософт рекомендует сбрасывать кэш DNS на компьютерах, подключающихся к рабочей области, после включения частной связи. 

Сведения о виртуальных машинах Azure см. в [документации по виртуальным машинам](../virtual-machines/index.yml).

## <a name="enable-public-access"></a>Включить общий доступ

В некоторых ситуациях может потребоваться разрешить кому-либо подключаться к защищенному рабочему пространству через общедоступную конечную точку, а не через виртуальную сеть. После настройки рабочей области с помощью частной конечной точки можно дополнительно включить общий доступ к рабочей области. Это не приведет к удалению закрытой конечной точки. Все связи между компонентами, расположенными за виртуальной сетью, остаются защищенными. Он обеспечивает общий доступ только к рабочей области, а также к частному доступу через виртуальную сеть.

> [!WARNING]
> При подключении через общедоступную конечную точку некоторые функции Studio не смогут получить доступ к данным. Эта проблема возникает, когда данные хранятся в службе, защищенной за пределы виртуальной сети. Например, учетная запись хранения Azure.

Чтобы включить общий доступ к рабочей области с поддержкой закрытых ссылок, выполните следующие действия.

# <a name="python"></a>[Python](#tab/python)

Чтобы удалить закрытую конечную точку, используйте [Workspace.delete_private_endpoint_connection](/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#delete-private-endpoint-connection-private-endpoint-connection-name-) .

```python
from azureml.core import Workspace

ws = Workspace.from_config()
ws.update(allow_public_access_when_behind_vnet=True)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Расширение Azure CLI для машинного обучения](reference-azure-machine-learning-cli.md) предоставляет команду [AZ ML Workspace Update](/cli/azure/ext/azure-cli-ml/ml/workspace#ext_azure_cli_ml_az_ml_workspace_update) . Чтобы включить общий доступ к рабочей области, добавьте параметр `--allow-public-access true` .

# <a name="portal"></a>[Портал](#tab/azure-portal)

В настоящее время невозможно включить эту функцию с помощью портала.

---


## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о защите рабочей области Машинное обучение Azure см. в статье [Общие сведения о изоляции и конфиденциальности виртуальной сети](how-to-network-security-overview.md) .

* Если вы планируете использовать настраиваемое решение DNS в виртуальной сети, см. статью [Использование рабочей области с пользовательским DNS-сервером](how-to-custom-dns.md).

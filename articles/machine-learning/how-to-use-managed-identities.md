---
title: Использование управляемых удостоверений для контроля доступа (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать управляемые удостоверения для управления доступом к ресурсам Azure из Машинное обучение Azure рабочей области.
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: a7efd57100ad89fa9824b7a635e11698515e13ae
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521022"
---
# <a name="use-managed-identities-with-azure-machine-learning-preview"></a>Использование управляемых удостоверений с Машинное обучение Azure (Предварительная версия)

[Управляемые удостоверения](../active-directory/managed-identities-azure-resources/overview.md) позволяют настроить рабочую область с *минимально необходимыми разрешениями для доступа к ресурсам*. 

При ненадежной настройке рабочей области Машинное обучение Azure важно убедиться, что разные службы, связанные с рабочей областью, имеют правильный уровень доступа. Например, во время рабочего процесса машинного обучения рабочей области требуется доступ к реестру контейнеров Azure (записи контроля доступа) для образов DOCKER и учетных записей хранения для обучающих данных. 

Более того, управляемые удостоверения обеспечивают точный контроль над разрешениями, например, вы можете предоставить или отозвать доступ к конкретным ресурсам в определенной записи контроля доступа.

В этой статье вы узнаете, как использовать управляемые удостоверения в следующих случаях:

 * Настройте и используйте запись контроля доступа для рабочей области Машинное обучение Azure без включения прав администратора на доступ к записи.
 * Доступ к закрытой записи контроля доступа, внешней для рабочей области, для извлечения базовых образов для обучения или вывода.
 * Создайте рабочую область с назначенным пользователем управляемым удостоверением для доступа к связанным ресурсам.

> [!IMPORTANT]
> Использование управляемых удостоверений для управления доступом к ресурсам с Машинное обучение Azure в настоящее время находится на этапе предварительной версии. Функция предварительной версии предоставляется "как есть", без гарантии поддержки или соглашения об уровне обслуживания. Дополнительные сведения см. в дополнительных [условиях использования для предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
 
## <a name="prerequisites"></a>Предварительные требования

- Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).
- [Расширение Azure CLI для службы машинное обучение](reference-azure-machine-learning-cli.md)
- [Пакет SDK для машинное обучение Azure Python](/python/api/overview/azure/ml/intro).
- Для назначения ролей имя входа для подписки Azure должно иметь роль [управляемого оператора идентификации](../role-based-access-control/built-in-roles.md#managed-identity-operator) или другую роль, которая предоставляет необходимые действия (например, __owner__).
- Вы должны быть знакомы с созданием [управляемых удостоверений](../active-directory/managed-identities-azure-resources/overview.md)и работой с ними.

## <a name="configure-managed-identities"></a>Настройка управляемых удостоверений

В некоторых ситуациях необходимо запретить доступ пользователей с правами администратора к реестру контейнеров Azure. Например, запись контроля доступа может быть общей и необходимо запретить доступ администратора другим пользователям. Либо создание записи контроля доступа с включенным пользователем с правами администратора запрещена политикой уровня подписки.

> [!IMPORTANT]
> При использовании Машинное обучение Azure для определения в экземпляре контейнера Azure (ACI) __требуется__ административный доступ к записи контроля доступа. Не отключайте его, если планируется развертывание моделей в ACI для вывода.

Если вы создаете запись контроля доступа, не разрешая пользователю административный доступ, управляемые удостоверения используются для получения доступа к записи записи после создания образов DOCKER и их извлечения.

Вы можете создать собственную запись контроля доступа с отключенным пользователем с правами администратора при создании рабочей области. Кроме того, можно Машинное обучение Azure создать запись контроля доступа в рабочей области и отключить пользователя с правами администратора.

### <a name="bring-your-own-acr"></a>Взять собственную запись контроля доступа

Если администратор записи контроля доступа запрещен политикой подписки, необходимо сначала создать запись контроля доступа без учетной записи администратора, а затем связать ее с рабочей областью. Кроме того, если у вас есть учетная запись контроля доступа с отключенным администратором, вы можете присоединить ее к рабочей области.

[Создайте запись контроля доступа из Azure CLI](../container-registry/container-registry-get-started-azure-cli.md) без ```--admin-enabled``` аргумента параметра или из портал Azure без включения пользователя администратора. Затем при создании Машинное обучение Azure рабочей области Укажите идентификатор ресурса Azure для записи контроля доступа. В следующем примере показано создание рабочей области машинного обучения Azure, которая использует существующую запись контроля доступа:

> [!TIP]
> Чтобы получить значение для `--container-registry` параметра, используйте команду AZ запись [контроля](/cli/azure/acr#az_acr_show) доступа, чтобы отобразить сведения о записи контроля доступа. `id`Поле содержит идентификатор ресурса для записи контроля доступа.

```azurecli-interactive
az ml workspace create -w <workspace name> \
-g <workspace resource group> \
-l <region> \
--container-registry /subscriptions/<subscription id>/resourceGroups/<acr resource group>/providers/Microsoft.ContainerRegistry/registries/<acr name>
```

### <a name="let-azure-machine-learning-service-create-workspace-acr"></a>Разрешить Машинное обучение Azure службе создать запись контроля доступа для рабочей области

Если вы не выберете собственную запись контроля доступа, Машинное обучение Azure служба создаст ее при выполнении какой бы то ни было операции. Например, отправьте обучающий запуск в Вычислительная среда Машинного обучения, создайте среду или разверните конечную точку веб-службы. Запись контроля доступа, созданная рабочей областью, будет включать пользователя Admin, и вам потребуется отключить пользователя с правами администратора вручную.

1. Создание рабочей области

    ```azurecli-interactive
    az ml workspace show -n <my workspace> -g <my resource group>
    ```

1. Выполните действие, для которого требуется запись контроля доступа. Например, [руководство по обучению модели](tutorial-train-models-with-aml.md).

1. Получите имя записи контроля доступа, созданное кластером:

    ```azurecli-interactive
    az ml workspace show -w <my workspace> \
    -g <my resource group>
    --query containerRegistry
    ```

    Эта команда возвращает значение, аналогичное следующему тексту. Требуется только последняя часть текста, которая является именем экземпляра записи контроля доступа:

    ```output
    /subscriptions/<subscription id>/resourceGroups/<my resource group>/providers/MicrosoftContainerReggistry/registries/<ACR instance name>
    ```

1. Обновите запись контроля доступа, чтобы отключить учетную запись администратора:

    ```azurecli-interactive
    az acr update --name <ACR instance name> --admin-enabled false
    ```

### <a name="create-compute-with-managed-identity-to-access-docker-images-for-training"></a>Создание вычислений с управляемым удостоверением для доступа к образам DOCKER для обучения

Чтобы получить доступ к записи контроля доступа рабочей области, создайте кластерный ресурс машинного обучения с включенным системой управляемым удостоверением. Удостоверение можно включить портал Azure или Studio при создании вычислений или из Azure CLI с помощью приведенных ниже. Дополнительные сведения см. в разделе [использование управляемого удостоверения с COMPUTE Clusters](how-to-create-attach-compute-cluster.md#managed-identity).

# <a name="python"></a>[Python](#tab/python)

При создании кластера вычислений с [амлкомпутепровисионингконфигуратион](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcomputeprovisioningconfiguration)используйте `identity_type` параметр, чтобы задать управляемый тип удостоверения.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interaction
az ml computetarget create amlcompute --name cpucluster -w <workspace> -g <resource group> --vm-size <vm sku> --assign-identity '[system]'
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

Сведения о настройке управляемого удостоверения при создании кластера вычислений в студии см. в разделе [Настройка управляемого удостоверения](how-to-create-attach-compute-studio.md#managed-identity).

---

Управляемому удостоверению автоматически предоставляется роль Акрпулл в записи контроля доступа к рабочей области, чтобы разрешить извлечение образов DOCKER для обучения.

> [!NOTE]
> При создании вычислений сначала до создания записи контроля доступа рабочей области необходимо назначить роль Акрпулл вручную.

## <a name="access-base-images-from-private-acr"></a>Доступ к базовым образам из закрытой записи контроля доступа

По умолчанию Машинное обучение Azure использует базовые образы DOCKER, которые поступают из общедоступного репозитория, управляемого Майкрософт. Затем он создает среду обучения или вывода на этих изображениях. Дополнительные сведения см. в статье [что такое среды ML?](concept-environments.md).

Чтобы использовать пользовательский базовый образ для предприятия, вы можете использовать управляемые удостоверения для доступа к вашей частной записи. Существует два варианта использования.

 * Использовать базовый образ для обучения, как есть.
 * Создание Машинное обучение Azure управляемого образа с помощью пользовательского образа в качестве основы.

### <a name="pull-docker-base-image-to-machine-learning-compute-cluster-for-training-as-is"></a>Базовый образ DOCKER для машинного обучения в кластере вычислений для обучения

Создайте кластер вычислений машинного обучения с включенным системой управляемым удостоверением, как описано выше. Затем определите идентификатор субъекта управляемого удостоверения.

```azurecli-interactive
az ml computetarget amlcompute identity show --name <cluster name> -w <workspace> -g <resource group>
```

При необходимости можно обновить кластер вычислений для назначения управляемого удостоверения, назначенного пользователем:

```azurecli-interactive
az ml computetarget amlcompute identity assign --name cpucluster \
-w $mlws -g $mlrg --identities <my-identity-id>
```

Чтобы разрешить вытягивание базовых образов в кластере вычислений, предоставьте управляемому удостоверению службы Акрпулл роль в закрытой записи контроля доступа.

```azurecli-interactive
az role assignment create --assignee <principal ID> \
--role acrpull \
--scope "/subscriptions/<subscription ID>/resourceGroups/<private ACR resource group>/providers/Microsoft.ContainerRegistry/registries/<private ACR name>"
```

Наконец, при отправке обучающего запуска укажите расположение базового образа в [определении среды](how-to-use-environments.md#use-existing-environments).

```python
from azureml.core import Environment
env = Environment(name="private-acr")
env.docker.base_image = "<ACR name>.azurecr.io/<base image repository>/<base image version>"
env.python.user_managed_dependencies = True
```

> [!IMPORTANT]
> Чтобы гарантировать, что базовый образ помещается непосредственно в ресурс вычислений, установите `user_managed_dependencies = True` и не указывайте Dockerfile. В противном случае Машинное обучение Azure Служба попытается создать новый образ DOCKER и завершится сбоем, поскольку доступ к базовому образу из записи контроля доступа будет получен только в кластере вычислений.

### <a name="build-azure-machine-learning-managed-environment-into-base-image-from-private-acr-for-training-or-inference"></a>Создание Машинное обучение Azure управляемой среды в базовом образе из закрытой записи контроля доступа для обучения или вывода

В этом сценарии Машинное обучение Azure служба создает среду обучения или вывода поверх базового образа, который предоставляется из закрытой записи контроля доступа. Так как задача сборки образа выполняется в записи контроля доступа в рабочей области с помощью задач контроля учетных записей, необходимо выполнить дополнительные действия, чтобы разрешить доступ.

1. Создайте __управляемое удостоверение, назначаемое пользователем__ , и предоставьте удостоверению акрпулл доступ к __закрытой записи контроля__ доступа.  
1. Предоставьте управляемому системой рабочей области, __назначенному системе__ , управляемому удостоверению роль оператора идентификации в __управляемом удостоверении, назначенном пользователем__ на предыдущем шаге. Эта роль позволяет рабочей области назначать управляемое пользователем удостоверение задаче записи контроля доступа для создания управляемой среды. 

    1. Получите идентификатор субъекта управляемого удостоверения, назначенного системой:

        ```azurecli-interactive
        az ml workspace show -w <workspace name> -g <resource group> --query identityPrincipalId
        ```

    1. Предоставьте роль управляемого оператора идентификации:

        ```azurecli-interactive
        az role assignment create --assignee <principal ID> --role managedidentityoperator --scope <UAI resource ID>
        ```

        Идентификатор ресурса УАИ — это идентификатор ресурса Azure, которому назначено удостоверение пользователя, в формате `/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAI name>` .

1. Укажите внешнюю запись контроля доступа и идентификатор клиента для __назначаемого пользователем управляемого удостоверения__ в подключениях к рабочей области с помощью [метода Workspace.set_connection](/python/api/azureml-core/azureml.core.workspace.workspace#set-connection-name--category--target--authtype--value-):

    ```python
    workspace.set_connection(
        name="privateAcr", 
        category="ACR", 
        target = "<acr url>", 
        authType = "RegistryConnection", 
        value={"ResourceId": "<UAI resource id>", "ClientId": "<UAI client ID>"})
    ```

После завершения настройки можно использовать базовые образы из частной записи контроля доступа при создании сред для обучения или вывода. В следующем фрагменте кода показано, как указать запись контроля доступа базового образа и имя образа в определении среды:

```python
from azureml.core import Environment

env = Environment(name="my-env")
env.docker.base_image = "<acr url>/my-repo/my-image:latest"
```

При необходимости можно указать URL-адрес управляемого ресурса удостоверения и идентификатор клиента в определении среды с помощью [регистридентити](/python/api/azureml-core/azureml.core.container_registry.registryidentity). Если вы явно используете удостоверение реестра, он переопределяет все подключения к рабочей области, указанные ранее:

```python
from azureml.core.container_registry import RegistryIdentity

identity = RegistryIdentity()
identity.resource_id= "<UAI resource ID>"
identity.client_id="<UAI client ID>”
env.docker.base_image_registry.registry_identity=identity
env.docker.base_image = "my-acr.azurecr.io/my-repo/my-image:latest"
```

## <a name="use-docker-images-for-inference"></a>Использование образов DOCKER для вывода

После настройки записи контроля доступа без учетной записи администратора, как описано выше, можно получить доступ к образам DOCKER для вывода без ключей администратора из службы Azure Kubernetes (AKS). При создании или присоединении AKS к рабочей области субъекту-службе кластера автоматически назначается доступ Акрпулл к записи контроля доступа к рабочей области.

> [!NOTE]
> Если вы перенесете собственный кластер AKS, то вместо управляемого удостоверения кластеру должен быть включен субъект-служба.

## <a name="create-workspace-with-user-assigned-managed-identity"></a>Создание рабочей области с управляемым удостоверением, назначенным пользователем

При создании рабочей области можно указать назначаемое пользователем управляемое удостоверение, которое будет использоваться для доступа к связанным ресурсам: запись контроля учетных записей, KeyVault, хранилище и App Insights.

Сначала [Создайте назначаемое пользователем управляемое удостоверение](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli])и запишите идентификатор ресурса ARM управляемого удостоверения.

Затем используйте Azure CLI или пакет SDK для Python, чтобы создать рабочую область. При использовании интерфейса командной строки укажите идентификатор с помощью `--primary-user-assigned-identity` параметра. При использовании пакета SDK используйте `primary_user_assigned_identity` . Ниже приведены примеры использования Azure CLI и Python для создания новой рабочей области с использованием следующих параметров.

__Azure CLI__

```azurecli-interactive
az ml workspace create -w <workspace name> -g <resource group> --primary-user-assigned-identity <managed identity ARM ID>
```

__Python__

```python
from azureml.core import Workspace

ws = Workspace.create(name="workspace name", 
    subscription_id="subscription id", 
    resource_group="resource group name",
    primary_user_assigned_identity="managed identity ARM ID")
```

Можно также использовать [шаблон ARM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced) для создания рабочей области с назначенным пользователем управляемым удостоверением.

> [!IMPORTANT]
> Если вы понесете собственные связанные ресурсы, вместо того, чтобы создавать Машинное обучение Azure службы, необходимо предоставить управляемые роли идентификации в этих ресурсах. Для выполнения назначений используйте [шаблон ARM для назначения ролей](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-dependencies-role-assignment) .

Для рабочей области с (ключами, управляемыми клиентом для шифрования) [ https://docs.microsoft.com/azure/machine-learning/concept-data-encryption ] можно передать управляемое пользователем удостоверение для проверки подлинности из хранилища в Key Vault. Для передачи управляемого удостоверения используйте аргумент __User-назначил-Identity-for-CMK-Encryption__ (CLI) или __user_assigned_identity_for_cmk_encryption__ (SDK). Это управляемое удостоверение может совпадать с именем управляемого удостоверения, назначенным основному пользователю рабочей области, или совпадать с ним.

Если у вас есть рабочая область, ее можно обновить с назначенного пользователем управляемого удостоверения с помощью ```az ml workspace update``` команды CLI или ```Workspace.update``` метода Python SDK.


## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [корпоративной безопасности в машинное обучение Azure](concept-enterprise-security.md).

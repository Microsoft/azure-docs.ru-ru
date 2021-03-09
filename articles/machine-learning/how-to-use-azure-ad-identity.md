---
title: Использование удостоверения Azure AD с веб-службой
titleSuffix: Azure Machine Learning
description: Используйте удостоверение Azure AD с веб-службой в службе Kubernetes Azure для доступа к облачным ресурсам во время оценки.
services: machine-learning
ms.author: larryfr
author: BlackMist
ms.reviewer: aashishb
ms.service: machine-learning
ms.subservice: core
ms.date: 11/16/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: b6e6288f125da2a29a8eff56b64f327914f90cb4
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102520478"
---
# <a name="use-azure-ad-identity-with-your-machine-learning-web-service-in-azure-kubernetes-service"></a>Использование удостоверения Azure AD с веб-службой машинного обучения в Службе Azure Kubernetes

В этом пошаговом окне вы узнаете, как назначить удостоверение Azure Active Directory (Azure AD) для развернутой модели машинного обучения в службе Azure Kubernetes. Проект [удостоверения Pod Azure AD](https://github.com/Azure/aad-pod-identity) позволяет приложениям безопасно получать доступ к облачным ресурсам с помощью Azure AD, используя [управляемые удостоверения](../active-directory/managed-identities-azure-resources/overview.md) и примитивы Kubernetes. Это позволяет веб-службе безопасно получать доступ к ресурсам Azure без необходимости внедрять учетные данные или управлять маркерами непосредственно внутри `score.py` скрипта. В этой статье объясняется, как создать и установить удостоверение Azure в кластере службы Azure Kubernetes и назначить удостоверение для развернутой веб-службы.

## <a name="prerequisites"></a>Предварительные условия

- [Расширение Azure CLI для службы машинное обучение](reference-azure-machine-learning-cli.md), [машинное обучение Azure SDK для Python](/python/api/overview/azure/ml/intro)или [расширение машинное обучение Azure Visual Studio Code](tutorial-setup-vscode-extension.md).

- Доступ к кластеру AKS с помощью `kubectl` команды. Дополнительные сведения см. в разделе [Подключение к кластеру](../aks/kubernetes-walkthrough.md#connect-to-the-cluster) .

- Машинное обучение Azure веб-службы, развернутой в кластере AKS.

## <a name="create-and-install-an-azure-identity"></a>Создание и установка удостоверения Azure

1. Чтобы определить, включен ли кластер AKS Kubernetes RBAC, используйте следующую команду:

    ```azurecli-interactive
    az aks show --name <AKS cluster name> --resource-group <resource group name> --subscription <subscription id> --query enableRbac
    ```

    Эта команда возвращает значение, `true` Если включен KUBERNETES RBAC. Это значение определяет команду, которая будет использоваться на следующем шаге.

1. Установите [удостоверение Azure AD Pod](https://azure.github.io/aad-pod-identity/docs/getting-started/installation/) в кластере AKS.

1. [Создайте удостоверение в Azure](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#2-create-an-identity-on-azure) , выполнив действия, показанные на странице проекта удостоверений Pod Azure AD.

1. [Разверните азуреидентити](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#3-deploy-azureidentity) , выполнив действия, показанные на странице проекта удостоверений Pod Azure AD.

1. [Разверните азуреидентитибиндинг](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#5-deploy-azureidentitybinding) , выполнив действия, показанные на странице проекта удостоверений Pod Azure AD.

1. Если удостоверение Azure, созданное на предыдущем шаге, находится не в той же группе ресурсов кластера AKS, выполните действия по [назначению роли](https://azure.github.io/aad-pod-identity/docs/getting-started/role-assignment/#user-assigned-identities-that-are-not-within-the-node-resource-group) , показанные на странице проекта удостоверений Pod Azure AD.

## <a name="assign-azure-identity-to-web-service"></a>Назначение удостоверения Azure веб-службе

Следующие шаги используют удостоверение Azure, созданное в предыдущем разделе, и назначение его веб-службе AKS с помощью **метки Selector**.

Сначала укажите имя и пространство имен развертывания в кластере AKS, которому требуется назначить удостоверение Azure. Эти сведения можно получить, выполнив следующую команду. Пространства имен должны быть Машинное обучение Azure именем рабочей области, а имя развертывания должно быть именем конечной точки, как показано на портале.

```azurecli-interactive
kubectl get deployment --selector=isazuremlapp=true --all-namespaces --show-labels
```

Добавьте метку селектора удостоверения Azure в развертывание, изменив спецификацию развертывания. Значение селектора должно быть задано в шаге 5 раздела [deploy азуреидентитибиндинг](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#5-deploy-azureidentitybinding).

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentityBinding
metadata:
  name: demo1-azure-identity-binding
spec:
  AzureIdentity: <a-idname>
  Selector: <label value to match>
```

Измените развертывание, чтобы добавить метку селектора удостоверений Azure. Перейдите к следующему разделу в разделе `/spec/template/metadata/labels` . Вы должны увидеть такие значения, как `isazuremlapp: “true”` . Добавьте метку AAD-Pod-Identity, как показано ниже.

```azurecli-interactive
    kubectl edit deployment/<name of deployment> -n azureml-<name of workspace>
```

```yaml
spec:
  template:
    metadata:
      labels:
       aadpodidbinding: "<value of Selector in AzureIdentityBinding>"
      ...
```

Чтобы проверить правильность добавления метки, выполните следующую команду. Кроме того, должны отобразиться состояния только что созданных модулей Pod.

```azurecli-interactive
   kubectl get pod -n azureml-<name of workspace> --show-labels
```

После того как модули Pod будут работать, веб-службы для этого развертывания смогут получить доступ к ресурсам Azure через удостоверение Azure без необходимости внедрять учетные данные в код.

## <a name="assign-roles-to-your-azure-identity"></a>Назначение ролей удостоверениям Azure

[Назначьте управляемое удостоверение Azure с соответствующими ролями](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) для доступа к другим ресурсам Azure. Убедитесь, что назначенные роли имеют правильные действия с **данными**. Например, [роль читателя данных BLOB-объекта хранилища](../role-based-access-control/built-in-roles.md#storage-blob-data-reader) будет иметь разрешения на чтение BLOB-объекта хранилища, а универсальная [роль читателя](../role-based-access-control/built-in-roles.md#reader) — нет.

## <a name="use-azure-identity-with-your-web-service"></a>Использование удостоверения Azure для веб-службы

Разверните модель в кластере AKS. `score.py`Скрипт может содержать операции, указывающие на ресурсы Azure, к которым имеет доступ удостоверение Azure. Убедитесь, что установлены необходимые зависимости клиентской библиотеки для ресурса, к которому вы пытаетесь получить доступ. Ниже приведено несколько примеров того, как можно использовать удостоверение Azure для доступа к различным ресурсам Azure из службы.

### <a name="access-key-vault-from-your-web-service"></a>Доступ к Key Vault из веб-службы

Если вы предоставили удостоверению Azure доступ на чтение к секрету в **Key Vault**, вы `score.py` можете получить к нему доступ, используя следующий код.

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

my_vault_name = "yourkeyvaultname"
my_vault_url = "https://{}.vault.azure.net/".format(my_vault_name)
my_secret_name = "sample-secret"

# This will use your Azure Managed Identity
credential = DefaultAzureCredential()
secret_client = SecretClient(
    vault_url=my_vault_url,
    credential=credential)
secret = secret_client.get_secret(my_secret_name)
```

> [!IMPORTANT]
> В этом примере используется Дефаултазурекредентиал. Сведения о предоставлении доступа к удостоверению с помощью определенной политики доступа см. в разделе [Назначение политики доступа Key Vault с помощью Azure CLI](../key-vault/general/assign-access-policy-cli.md).

### <a name="access-blob-from-your-web-service"></a>Доступ к большому двоичному объекту из веб-службы

Если вы задаете удостоверению Azure доступ на чтение данных внутри **большого двоичного объекта хранилища**, вы `score.py` можете получить к нему доступ, используя следующий код.

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

my_storage_account_name = "yourstorageaccountname"
my_storage_account_url = "https://{}.blob.core.windows.net/".format(my_storage_account_name)

# This will use your Azure Managed Identity
credential = DefaultAzureCredential()
blob_service_client = BlobServiceClient(
    account_url=my_storage_account_url,
    credential=credential
)
blob_client = blob_service_client.get_blob_client(container="some-container", blob="some_text.txt")
blob_data = blob_client.download_blob()
blob_data.readall()
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об использовании клиентской библиотеки удостоверений Azure для Python см. в [репозитории](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/identity/azure-identity#azure-identity-client-library-for-python) на сайте GitHub.
* Подробное руководство по развертыванию моделей в кластерах службы Kubernetes Azure [см. в](how-to-deploy-azure-kubernetes-service.md)этом разделе.
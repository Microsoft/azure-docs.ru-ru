---
title: Управление Azure RBAC в Kubernetes из Azure
titleSuffix: Azure Kubernetes Service
description: Узнайте, как использовать Azure RBAC для авторизации Kubernetes в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 09/21/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: 9ce8bc71139b52d690893734435e6bfee090062d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102184378"
---
# <a name="use-azure-rbac-for-kubernetes-authorization-preview"></a>Использование Azure RBAC для авторизации Kubernetes (предварительная версия)

Сейчас вы уже можете использовать [встроенную проверку подлинности между Azure Active Directory (Azure AD) и AKS](managed-aad.md). Если эта интеграция включена, клиенты могут использовать пользователей, группы или субъекты-службы Azure AD в качестве субъектов в Kubernetes RBAC. Дополнительные сведения см. [здесь](azure-ad-rbac.md).
Эта функция освобождает вас от необходимости отдельно управлять удостоверениями пользователей и учетными данными для Kubernetes. Тем не менее, по-прежнему необходимо настроить и управлять Azure RBAC и Kubernetes RBAC отдельно. Дополнительные сведения о проверке подлинности и авторизации с помощью RBAC в AKS см. [здесь](concepts-identity.md).

В этом документе рассматривается новый подход, обеспечивающий унифицированный контроль и управление доступом к ресурсам Azure, AKS и Kubernetes.

## <a name="before-you-begin"></a>Перед началом

Возможность управления RBAC для ресурсов Kubernetes из Azure дает возможность управлять RBAC для ресурсов кластера с помощью Azure или собственных механизмов Kubernetes. Если этот параметр включен, субъекты Azure AD будут проверяться исключительно в Azure RBAC, а обычные пользователи Kubernetes и учетные записи служб будут проверены только с помощью Kubernetes RBAC. Дополнительные сведения о проверке подлинности и авторизации с помощью RBAC в AKS см. [здесь](concepts-identity.md#azure-rbac-for-kubernetes-authorization-preview).

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="prerequisites"></a>Предварительные условия 
- Убедитесь, что у вас Azure CLI версии 2.9.0 или более поздней.
- Убедитесь, что `EnableAzureRBACPreview` включен флаг компонента.
- Убедитесь, что установлено `aks-preview` [расширение CLI][az-extension-add] v 0.4.55 или более поздней версии.
- Убедитесь, что установлен [kubectl v 1.18.3 +][az-aks-install-cli].

#### <a name="register-enableazurerbacpreview-preview-feature"></a>`EnableAzureRBACPreview`Функция регистрации предварительной версии

Чтобы создать кластер AKS, использующий Azure RBAC для авторизации Kubernetes, необходимо включить `EnableAzureRBACPreview` флаг компонента в подписке.

Зарегистрируйте `EnableAzureRBACPreview` флаг функции с помощью команды [AZ Feature Register][az-feature-register] , как показано в следующем примере:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "EnableAzureRBACPreview"
```

 Состояние регистрации можно проверить с помощью команды [az feature list][az-feature-list].

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableAzureRBACPreview')].{Name:name,State:properties.state}"
```

Когда все будет готово, обновите регистрацию поставщика ресурсов *Microsoft. ContainerService* с помощью команды [AZ Provider Register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

#### <a name="install-aks-preview-cli-extension"></a>Установка расширения интерфейса командной строки предварительной версии AKS

Чтобы создать кластер AKS, использующий Azure RBAC, требуется расширение CLI *AKS-Preview* версии 0.4.55 или выше. Установите расширение Azure CLI *AKS-Preview* с помощью команды [AZ Extension Add][az-extension-add] или установите все доступные обновления с помощью команды [AZ Extension Update][az-extension-update] .

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="limitations"></a>Ограничения

- Требуется [управляемая интеграция с Azure AD](managed-aad.md).
- Вы не можете интегрировать Azure RBAC для авторизации Kubernetes в существующие кластеры во время предварительной версии, но вы сможете получить общедоступную версию.
- Используйте [kubectl v 1.18.3 +][az-aks-install-cli].
- Если у вас есть КРДС и вы вносите пользовательские определения ролей, единственный способ охватить КРДС сегодня — предоставить `Microsoft.ContainerService/managedClusters/*/read` . AKS работает над предоставлением более детализированных разрешений для КРДС. Для остальных объектов можно использовать определенные группы API, например: `Microsoft.ContainerService/apps/deployments/read` .
- Новые назначения ролей могут занимать до 5 мин для распространения и обновления сервером авторизации.
- Требует, чтобы клиент Azure AD, настроенный для проверки подлинности, совпадал с клиентом для подписки, в которой находится кластер AKS. 

## <a name="create-a-new-cluster-using-azure-rbac-and-managed-azure-ad-integration"></a>Создание нового кластера с помощью Azure RBAC и управляемой интеграции Azure AD

Создайте кластер AKS, используя следующие команды интерфейса командной строки.

Создайте группу ресурсов Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Создайте кластер AKS с управляемой интеграцией Azure AD и Azure RBAC для авторизации Kubernetes.

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g MyResourceGroup -n MyManagedCluster --enable-aad --enable-azure-rbac
```

Успешное создание кластера с интеграцией Azure AD и Azure RBAC для авторизации Kubernetes имеет следующий раздел в тексте ответа:

```json
"AADProfile": {
    "adminGroupObjectIds": null,
    "clientAppId": null,
    "enableAzureRbac": true,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "****-****-****-****-****"
  }
```

## <a name="create-role-assignments-for-users-to-access-cluster"></a>Создание назначений ролей для доступа пользователей к кластеру

AKS предоставляет следующие четыре встроенных роли:


| Роль                                | Описание  |
|-------------------------------------|--------------|
| Модуль чтения RBAC службы Azure Kubernetes  | Разрешает доступ только для чтения для просмотра большинства объектов в пространстве имен. Он не позволяет просматривать роли или привязки ролей. Эта роль не разрешает просмотр `Secrets` , так как чтение содержимого секретов обеспечивает доступ к учетным данным учетная запись службы в пространстве имен, что позволило бы доступу через API как любой учетная запись службы в пространстве имен (форма укрупнения привилегий).  |
| Модуль записи RBAC службы Azure Kubernetes | Разрешает доступ для чтения и записи к большинству объектов в пространстве имен. Эта роль не позволяет просматривать или изменять роли или привязки ролей. Однако эта роль позволяет получать доступ к классам Pod `Secrets` и запускать их как любые учетная запись службы в пространстве имен, поэтому его можно использовать для получения уровней доступа API любых учетная запись службы в пространстве имен. |
| Администратор RBAC службы Azure Kubernetes  | Предоставляет административный доступ, предназначенный для предоставления в пространстве имен. Разрешает доступ для чтения и записи к большинству ресурсов в пространстве имен (или области кластера), включая возможность создания ролей и привязок ролей в пространстве имен. Эта роль не разрешает доступ на запись к квоте ресурса или пространству имен. |
| Администратор кластера RBAC службы Azure Kubernetes  | Разрешает доступ суперпользователя для выполнения любых действий с любым ресурсом. Он предоставляет полный контроль над каждым ресурсом в кластере и во всех пространствах имен. |


Назначение ролей для **всего кластера AKS** можно выполнить либо в колонке управления доступом (IAM) ресурса кластера на портал Azure, либо с помощью Azure CLIных команд, как показано ниже:

```bash
# Get your AKS Resource ID
AKS_ID=$(az aks show -g MyResourceGroup -n MyManagedCluster --query id -o tsv)
```

```azurecli-interactive
az role assignment create --role "Azure Kubernetes Service RBAC Admin" --assignee <AAD-ENTITY-ID> --scope $AKS_ID
```

где `<AAD-ENTITY-ID>` может быть именем пользователя (например, user@contoso.com ) или даже ClientID субъекта-службы.

Кроме того, можно создать назначения ролей для определенного **пространства имен** в кластере:

```azurecli-interactive
az role assignment create --role "Azure Kubernetes Service RBAC Viewer" --assignee <AAD-ENTITY-ID> --scope $AKS_ID/namespaces/<namespace-name>
```

Сейчас назначения ролей для пространств имен должны быть настроены с помощью Azure CLI.


### <a name="create-custom-roles-definitions"></a>Создание определений пользовательских ролей

При необходимости вы можете создать собственное определение роли, а затем назначить его, как описано выше.

Ниже приведен пример определения роли, позволяющей пользователю читать только развертывания и ничего другого. Полный список возможных действий можно просмотреть [здесь](../role-based-access-control/resource-provider-operations.md#microsoftcontainerservice).


Скопируйте приведенный ниже код JSON в файл с именем `deploy-view.json` .

```json
{
    "Name": "AKS Deployment Viewer",
    "Description": "Lets you view all deployments in cluster/namespace.",
    "Actions": [],
    "NotActions": [],
    "DataActions": [
        "Microsoft.ContainerService/managedClusters/apps/deployments/read"
    ],
    "NotDataActions": [],
    "assignableScopes": [
        "/subscriptions/<YOUR SUBSCRIPTION ID>"
    ]
}
```

Замените `<YOUR SUBSCRIPTION ID>` идентификатором из подписки, который можно получить, выполнив:

```azurecli-interactive
az account show --query id -o tsv
```


Теперь можно создать определение роли, выполнив следующую команду из папки, в которой вы сохранили `deploy-view.json` :

```azurecli-interactive
az role definition create --role-definition @deploy-view.json 
```

Теперь, когда у вас есть определение роли, вы можете назначить его пользователю или другому удостоверению, выполнив команду:

```azurecli-interactive
az role assignment create --role "AKS Deployment Viewer" --assignee <AAD-ENTITY-ID> --scope $AKS_ID
```

## <a name="use-azure-rbac-for-kubernetes-authorization-with-kubectl"></a>Использование Azure RBAC для авторизации Kubernetes с помощью `kubectl`

> [!NOTE]
> Убедитесь, что у вас есть последняя версия kubectl, выполнив следующую команду:
>
> ```azurecli-interactive
> az aks install-cli
> ```
> Может потребоваться запустить его с `sudo` правами доступа. 

Теперь вы назначили нужную роль и разрешения. Можно запустить вызов API Kubernetes, например из `kubectl` .

Для этого сначала нужно получить kubeconfig кластера, используя следующую команду:

```azurecli-interactive
az aks get-credentials -g MyResourceGroup -n MyManagedCluster
```

> [!IMPORTANT]
> Для выполнения описанного выше шага потребуется встроенная роль [пользователя кластера службы Azure Kubernetes](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-user-role) .

Теперь можно использовать kubectl для, например, для перечисления узлов в кластере. При первом запуске потребуется выполнить вход, а последующие команды будут использовать соответствующий маркер доступа.

```azurecli-interactive
kubectl get nodes
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.

NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-93451573-vmss000000   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000001   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000002   Ready    agent   3h6m   v1.15.11
```


## <a name="use-azure-rbac-for-kubernetes-authorization-with-kubelogin"></a>Использование Azure RBAC для авторизации Kubernetes с помощью `kubelogin`

Чтобы разблокировать дополнительные сценарии, такие как неинтерактивные входы в систему, старые `kubectl` версии или использование единого входа в нескольких кластерах без необходимости входа в новый кластер, при этом AKS создал подключаемый модуль exec [`kubelogin`](https://github.com/Azure/kubelogin) .

Его можно использовать, выполнив:

```bash
export KUBECONFIG=/path/to/kubeconfig
kubelogin convert-kubeconfig
``` 

В первый раз вам придется войти в интерактивном режиме, как и обычные kubectl, но после этого вам больше не потребуется даже для новых кластеров Azure AD (если токен по-прежнему действителен).

```bash
kubectl get nodes
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.

NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-93451573-vmss000000   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000001   Ready    agent   3h6m   v1.15.11
aks-nodepool1-93451573-vmss000002   Ready    agent   3h6m   v1.15.11
```


## <a name="clean-up"></a>Очистка

### <a name="clean-role-assignment"></a>Очистить назначение роли

```azurecli-interactive
az role assignment list --scope $AKS_ID --query [].id -o tsv
```
Скопируйте ИДЕНТИФИКАТОРы или идентификаторы из всех назначений, а затем.

```azurecli-interactive
az role assignment delete --ids <LIST OF ASSIGNMENT IDS>
```

### <a name="clean-up-role-definition"></a>Очистка определения роли

```azurecli-interactive
az role definition delete -n "AKS Deployment Viewer"
```

### <a name="delete-cluster-and-resource-group"></a>Удаление кластера и группы ресурсов

```azurecli-interactive
az group delete -n MyResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о проверке подлинности AKS, авторизации, Kubernetes RBAC и Azure RBAC см. [здесь](concepts-identity.md).
- Дополнительные сведения об Azure RBAC см. [здесь](../role-based-access-control/overview.md).
- Узнайте больше о всех действиях, которые можно использовать для детального определения настраиваемых ролей Azure [для авторизации Kubernetes](../role-based-access-control/resource-provider-operations.md#microsoftcontainerservice).


<!-- LINKS - Internal -->
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-aks-install-cli]: /cli/azure/aks#az-aks-install-cli
[az-provider-register]: /cli/azure/provider#az-provider-register

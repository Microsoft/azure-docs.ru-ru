---
title: Ограничение доступа к kubeconfig в Службе Azure Kubernetes (AKS)
description: Узнайте, как управлять доступом к файлу конфигурации Kubernetes (kubeconfig) для администраторов и пользователей кластера.
services: container-service
ms.topic: article
ms.date: 05/06/2020
ms.openlocfilehash: 77b9988557106ef460d3b222ef85eb29e08f31c8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97693984"
---
# <a name="use-azure-role-based-access-control-to-define-access-to-the-kubernetes-configuration-file-in-azure-kubernetes-service-aks"></a>Использование управления доступом на основе ролей Azure для определения доступа к файлу конфигурации Kubernetes в службе Kubernetes Azure (AKS)

Вы можете взаимодействовать с кластерами Kubernetes с помощью инструмента `kubectl`. Azure CLI позволяет без труда получить учетные данные для доступа и сведения о конфигурации, чтобы подключиться к кластерам AKS с помощью `kubectl`. Чтобы ограничить количество пользователей, которые могут получать сведения о конфигурации Kubernetes (*kubeconfig*) и ограничить разрешения, которые они имеют, можно использовать управление доступом на основе ролей Azure (Azure RBAC).

В этой статье показано, как назначить роли Azure, которые ограничивают, кто может получить сведения о конфигурации для кластера AKS.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

В этой статье также предполагается, что вы используете Azure CLI версии 2.0.65 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="available-cluster-roles-permissions"></a>Доступные разрешения ролей кластера

При взаимодействии с кластером AKS с помощью инструмента `kubectl` используется файл конфигурации, который определяет сведения о подключении кластера. Этот файл конфигурации обычно хранится в *~/.KUBE/config.*. В этом файле *kubeconfig* можно определить несколько кластеров. Вы можете переключаться между кластерами с помощью команды [kubectl config use-context][kubectl-config-use-context].

Команда [az aks get-credentials][az-aks-get-credentials] позволяет получить учетные данные для доступа к кластеру AKS и объединить их в файл *kubeconfig*. Для управления доступом к этим учетным данным можно использовать управление доступом на основе ролей Azure (Azure RBAC). Эти роли Azure позволяют определить, кто может извлекать файл *kubeconfig* и какие разрешения они имеют в кластере.

Ниже описаны две встроенные роли:

* **Роль администратора кластера в Службе Azure Kubernetes**  
  * Разрешение доступа к вызову API *Microsoft.ContainerService/managedClusters/listClusterAdminCredential/action*. Этот вызов API [позволяет перечислить учетные данные администратора кластера][api-cluster-admin].
  * Скачивание файла *kubeconfig* для роли *clusterAdmin*.
* **Роль пользователя кластера в Службе Azure Kubernetes**
  * Разрешение доступа к вызову API *Microsoft.ContainerService/managedClusters/listClusterUserCredential/action*. Этот вызов API [позволяет перечислить учетные данные пользователя кластера][api-cluster-user].
  * Скачивание файла *kubeconfig* для роли *clusterUser*.

Эти роли Azure можно применять к пользователю или группе Azure Active Directory (AD).

> [!NOTE]
> В кластерах, использующих Azure AD, пользователи с ролью *клустерусер* имеют пустой файл *kubeconfig* , который запрашивает вход в систему. После входа пользователи получают доступ на основе параметров пользователя или группы Azure AD. Пользователи с ролью *клустерадмин* имеют доступ администратора.
>
> Кластеры, не использующие Azure AD, используют только роль *клустерадмин* .

## <a name="assign-role-permissions-to-a-user-or-group"></a>Назначение разрешений роли пользователю или группе

Чтобы назначить одну из доступных ролей, необходимо получить идентификатор ресурса кластера AKS и идентификатор учетной записи пользователя или группы Azure AD. В следующих примерах команд:

* Получите идентификатор ресурса кластера с помощью команды [AZ AKS показывать][az-aks-show] для кластера с именем *myAKSCluster* в группе ресурсов *myResourceGroup* . При необходимости укажите имя вашей группы ресурсов и кластера.
* Используйте команды [AZ Account шоу][az-account-show] и [AZ AD user демонстрация][az-ad-user-show] , чтобы получить идентификатор пользователя.
* Наконец, назначьте роль с помощью команды [AZ Role назначение Create][az-role-assignment-create] .

В следующем примере *роль администратора кластера Azure Kubernetes* назначается отдельной учетной записи пользователя:

```azurecli-interactive
# Get the resource ID of your AKS cluster
AKS_CLUSTER=$(az aks show --resource-group myResourceGroup --name myAKSCluster --query id -o tsv)

# Get the account credentials for the logged in user
ACCOUNT_UPN=$(az account show --query user.name -o tsv)
ACCOUNT_ID=$(az ad user show --id $ACCOUNT_UPN --query objectId -o tsv)

# Assign the 'Cluster Admin' role to the user
az role assignment create \
    --assignee $ACCOUNT_ID \
    --scope $AKS_CLUSTER \
    --role "Azure Kubernetes Service Cluster Admin Role"
```

> [!IMPORTANT]
> В некоторых случаях *User.Name* в учетной записи отличается от *userPrincipalName*, например с гостевыми пользователями Azure AD:
>
> ```output
> $ az account show --query user.name -o tsv
> user@contoso.com
> $ az ad user list --query "[?contains(otherMails,'user@contoso.com')].{UPN:userPrincipalName}" -o tsv
> user_contoso.com#EXT#@contoso.onmicrosoft.com
> ```
>
> В этом случае присвойте параметру *ACCOUNT_UPN* значение *userPrincipalName* из пользователя Azure AD. Например, если ваша учетная запись *User.Name* — *пользователь \@ contoso.com*:
> 
> ```azurecli-interactive
> ACCOUNT_UPN=$(az ad user list --query "[?contains(otherMails,'user@contoso.com')].{UPN:userPrincipalName}" -o tsv)
> ```

> [!TIP]
> Если вы хотите назначить разрешения для группы Azure AD, измените `--assignee` параметр, показанный в предыдущем примере, на идентификатор объекта для *группы* , а не для *пользователя*. Чтобы получить идентификатор объекта для группы, используйте команду [AZ AD Group показ][az-ad-group-show] . В следующем примере возвращается идентификатор объекта для группы Azure AD с именем *AppDev*: `az ad group show --group appdev --query objectId -o tsv`

При необходимости можно изменить предыдущее назначение на *роль пользователя кластера*.

В следующем примере выходных данных показано, что назначение ролей выполнено успешно:

```
{
  "canDelegate": null,
  "id": "/subscriptions/<guid>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/providers/Microsoft.Authorization/roleAssignments/b2712174-5a41-4ecb-82c5-12b8ad43d4fb",
  "name": "b2712174-5a41-4ecb-82c5-12b8ad43d4fb",
  "principalId": "946016dd-9362-4183-b17d-4c416d1f8f61",
  "resourceGroup": "myResourceGroup",
  "roleDefinitionId": "/subscriptions/<guid>/providers/Microsoft.Authorization/roleDefinitions/0ab01a8-8aac-4efd-b8c2-3ee1fb270be8",
  "scope": "/subscriptions/<guid>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="get-and-verify-the-configuration-information"></a>Получение и проверка сведений о конфигурации

Используя назначенные роли Azure, используйте команду [AZ AKS Get-Credential][az-aks-get-credentials] , чтобы получить определение *KUBECONFIG* для кластера AKS. В приведенном ниже примере возвращаются учетные данные *--admin*, которые будут работать правильно при наличии *роли администратора кластера*:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin
```

Затем с помощью команды [kubectl config view][kubectl-config-view] можно проверить, показано ли в *context*, что применены сведения о конфигурации администратора:

```
$ kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://myaksclust-myresourcegroup-19da35-4839be06.hcp.eastus.azmk8s.io:443
  name: myAKSCluster
contexts:
- context:
    cluster: myAKSCluster
    user: clusterAdmin_myResourceGroup_myAKSCluster
  name: myAKSCluster-admin
current-context: myAKSCluster-admin
kind: Config
preferences: {}
users:
- name: clusterAdmin_myResourceGroup_myAKSCluster
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
    token: e9f2f819a4496538b02cefff94e61d35
```

## <a name="remove-role-permissions"></a>Удаление разрешений ролей

Чтобы удалить назначения роли, используйте команду [az role assignment delete][az-role-assignment-delete]. Укажите идентификатор учетной записи и идентификатор ресурса кластера, полученные в предыдущих командах. Если роль назначается группе, а не пользователю, укажите соответствующий идентификатор объекта группы, а не идентификатор объекта учетной записи для `--assignee` параметра:

```azurecli-interactive
az role assignment delete --assignee $ACCOUNT_ID --scope $AKS_CLUSTER
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы повысить уровень безопасности при доступе к кластерам AKS, [интегрируйте аутентификацию Azure Active Directory][aad-integration].

<!-- LINKS - external -->
[kubectl-config-use-context]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config
[kubectl-config-view]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[azure-rbac]: ../role-based-access-control/overview.md
[api-cluster-admin]: /rest/api/aks/managedclusters/listclusteradmincredentials
[api-cluster-user]: /rest/api/aks/managedclusters/listclusterusercredentials
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-account-show]: /cli/azure/account#az-account-show
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-role-assignment-delete]: /cli/azure/role/assignment#az-role-assignment-delete
[aad-integration]: ./azure-ad-integration-cli.md
[az-ad-group-show]: /cli/azure/ad/group#az-ad-group-show

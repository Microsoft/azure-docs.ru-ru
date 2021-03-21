---
title: Использование Azure AD и Kubernetes RBAC для кластеров
titleSuffix: Azure Kubernetes Service
description: Узнайте, как использовать членство в группах Azure Active Directory, чтобы ограничить доступ к ресурсам кластера с помощью управления доступом на основе ролей Kubernetes (Kubernetes RBAC) в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 03/17/2021
ms.openlocfilehash: 72b2c456d62b899f2b04041929434da668cad82d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104583536"
---
# <a name="control-access-to-cluster-resources-using-kubernetes-role-based-access-control-and-azure-active-directory-identities-in-azure-kubernetes-service"></a>Управление доступом к ресурсам кластера с помощью управления доступом на основе ролей Kubernetes и удостоверений Azure Active Directory в службе Kubernetes Azure

Службу Azure Kubernetes (AKS) можно настроить на использование Azure Active Directory (AD) для проверки подлинности пользователей. В этой конфигурации вы входите в кластер AKS с помощью маркера проверки подлинности Azure AD. Можно также настроить Kubernetes управления доступом на основе ролей (Kubernetes RBAC), чтобы ограничить доступ к ресурсам кластера на основе удостоверения пользователя или членства в группе.

В этой статье показано, как использовать членство в группах Azure AD для управления доступом к пространствам имен и ресурсам кластера с помощью Kubernetes RBAC в кластере AKS. Примеры групп и пользователей создаются в Azure AD, а затем роли и Ролебиндингс создаются в кластере AKS, чтобы предоставить соответствующие разрешения на создание и просмотр ресурсов.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас уже есть кластер AKS с поддержкой интеграции с Azure AD. Если вам нужен кластер AKS, см. статью [интеграция Azure Active Directory с AKS][azure-ad-aks-cli].

Необходимо установить и настроить Azure CLI версии 2.0.61 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="create-demo-groups-in-azure-ad"></a>Создание демонстрационных групп в Azure AD

В этой статье мы создадим две роли пользователей, которые можно использовать, чтобы продемонстрировать, как Kubernetes RBAC и Azure AD контролируют доступ к ресурсам кластера. Используются следующие два примера ролей:

* **Разработчик приложений**
    * Пользователь с именем *аксдев* , который является частью группы *AppDev* .
* **Инженер по надежности сайта**
    * Пользователь с именем *акссре* , который является частью группы *опссре* .

В рабочей среде можно использовать существующих пользователей и группы в клиенте Azure AD.

Сначала получите идентификатор ресурса кластера AKS с помощью команды [AZ AKS показывать][az-aks-show] . Назначьте идентификатор ресурса переменной с именем *AKS_ID* , чтобы на нее можно было ссылаться в дополнительных командах.

```azurecli-interactive
AKS_ID=$(az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --query id -o tsv)
```

Создайте группу первого примера в Azure AD для разработчиков приложений, используя команду [AZ AD Group Create][az-ad-group-create] . В следующем примере создается группа с именем *AppDev*:

```azurecli-interactive
APPDEV_ID=$(az ad group create --display-name appdev --mail-nickname appdev --query objectId -o tsv)
```

Теперь создайте назначение роли Azure для группы *AppDev* с помощью команды [AZ Role назначение Create][az-role-assignment-create] . Это назначение позволяет любому члену группы использовать `kubectl` для взаимодействия с кластером AKS, предоставляя им *роль пользователя кластера службы Kubernetes Azure*.

```azurecli-interactive
az role assignment create \
  --assignee $APPDEV_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

> [!TIP]
> Если появляется сообщение об ошибке `Principal 35bfec9328bd4d8d9b54dea6dac57b82 does not exist in the directory a5443dcd-cd0e-494d-a387-3039b419f0d5.` , например, подождите несколько секунд, пока идентификатор объекта группы Azure AD будет передан через каталог, а затем повторите `az role assignment create` команду.

Создайте второй пример группы, такой как для Срес с именем *опссре*:

```azurecli-interactive
OPSSRE_ID=$(az ad group create --display-name opssre --mail-nickname opssre --query objectId -o tsv)
```

Опять же, создайте назначение роли Azure, чтобы предоставить членам группы *роль пользователя кластера Azure Kubernetes Service*:

```azurecli-interactive
az role assignment create \
  --assignee $OPSSRE_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

## <a name="create-demo-users-in-azure-ad"></a>Создание демонстрационных пользователей в Azure AD

С помощью двух примеров групп, созданных в Azure AD для разработчиков приложений и Срес, теперь можно создать двух примеров пользователей. Чтобы протестировать интеграцию Kubernetes RBAC в конце статьи, войдите в кластер AKS с этими учетными записями.

Задайте имя участника-пользователя (UPN) и пароль для разработчиков приложений. Следующая команда предложит ввести имя участника-пользователя и задаст его *AAD_DEV_UPN* для использования в более поздней команде (Помните, что команды в этой статье введены в оболочку Bash). В качестве имени участника-пользователя необходимо указать проверенное доменное имя клиента, например `aksdev@contoso.com` .

```azurecli-interactive
echo "Please enter the UPN for application developers: " && read AAD_DEV_UPN
```

Следующая команда предложит ввести пароль и задаст его *AAD_DEV_PW* для использования в более поздней команде.

```azurecli-interactive
echo "Please enter the secure password for application developers: " && read AAD_DEV_PW
```

Создайте первую учетную запись пользователя в Azure AD с помощью команды [AZ AD user Create][az-ad-user-create] .

В следующем примере создается пользователь с отображаемым именем *AKS dev* и UPN и защищенный пароль, используя значения в *AAD_DEV_UPN* и *AAD_DEV_PW*.

```azurecli-interactive
AKSDEV_ID=$(az ad user create \
  --display-name "AKS Dev" \
  --user-principal-name $AAD_DEV_UPN \
  --password $AAD_DEV_PW \
  --query objectId -o tsv)
```

Теперь добавьте пользователя в группу *AppDev* , созданную в предыдущем разделе, с помощью команды [AZ Active Group Member Add][az-ad-group-member-add] :

```azurecli-interactive
az ad group member add --group appdev --member-id $AKSDEV_ID
```

Задайте имя участника-пользователя и пароль для Срес. Следующая команда предложит ввести имя участника-пользователя и задаст его *AAD_SRE_UPN* для использования в более поздней команде (Помните, что команды в этой статье введены в оболочку Bash). В качестве имени участника-пользователя необходимо указать проверенное доменное имя клиента, например `akssre@contoso.com` .

```azurecli-interactive
echo "Please enter the UPN for SREs: " && read AAD_SRE_UPN
```

Следующая команда предложит ввести пароль и задаст его *AAD_SRE_PW* для использования в более поздней команде.

```azurecli-interactive
echo "Please enter the secure password for SREs: " && read AAD_SRE_PW
```

Создайте вторую учетную запись пользователя. В следующем примере создается пользователь с отображаемым именем *AKS выполняются* , UPN и защищенный пароль, используя значения в *AAD_SRE_UPN* и *AAD_SRE_PW*.

```azurecli-interactive
# Create a user for the SRE role
AKSSRE_ID=$(az ad user create \
  --display-name "AKS SRE" \
  --user-principal-name $AAD_SRE_UPN \
  --password $AAD_SRE_PW \
  --query objectId -o tsv)

# Add the user to the opssre Azure AD group
az ad group member add --group opssre --member-id $AKSSRE_ID
```

## <a name="create-the-aks-cluster-resources-for-app-devs"></a>Создание ресурсов кластера AKS для приложения разработчики

Теперь группы и пользователи Azure AD созданы. Назначения ролей Azure были созданы для того, чтобы участники группы подключались к кластеру AKS в качестве обычного пользователя. Теперь настроим кластер AKS, чтобы разрешить этим различным группам доступ к конкретным ресурсам.

Сначала получите учетные данные администратора кластера с помощью команды [AZ AKS Get-Credential][az-aks-get-credentials] . В одном из следующих разделов вы получите учетные данные обычного кластера *пользователей* , чтобы увидеть последовательность проверки подлинности Azure AD в действии.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin
```

Создайте пространство имен в кластере AKS с помощью команды [kubectl Create Namespace][kubectl-create] . В следующем примере создается имя пространства имен *dev*:

```console
kubectl create namespace dev
```

В Kubernetes *роли* определяют разрешения, которые необходимо предоставить, и *ролебиндингс* их применение к нужным пользователям или группам. Эти назначения могут применяться для определенного пространства имен или в масштабах всего кластера. Дополнительные сведения см. [в разделе Использование авторизации KUBERNETES RBAC][rbac-authorization].

Сначала создайте роль для пространства имен *dev* . Эта роль предоставляет полные разрешения на пространство имен. В рабочей среде можно указать более детализированные разрешения для разных пользователей или групп.

Создайте файл с именем `role-dev-namespace.yaml` и вставьте следующий манифест YAML:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-full-access
  namespace: dev
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
```

Создайте роль с помощью команды [kubectl Apply][kubectl-apply] и укажите имя файла манифеста YAML:

```console
kubectl apply -f role-dev-namespace.yaml
```

Затем получите идентификатор ресурса для группы *AppDev* с помощью команды [AZ AD Group демонстрация][az-ad-group-show] . Эта группа задается в качестве субъекта Ролебиндинг на следующем шаге.

```azurecli-interactive
az ad group show --group appdev --query objectId -o tsv
```

Теперь создайте Ролебиндинг для группы *AppDev* , чтобы использовать ранее созданную роль для доступа к пространству имен. Создайте файл `rolebinding-dev-namespace.yaml` и вставьте в него следующий манифест YAML. В последней строке замените *граупобжектид*  на выходные данные идентификатора объекта Group из предыдущей команды:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-access
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-user-full-access
subjects:
- kind: Group
  namespace: dev
  name: groupObjectId
```

Создайте Ролебиндинг с помощью команды [kubectl Apply][kubectl-apply] и укажите имя файла манифеста YAML:

```console
kubectl apply -f rolebinding-dev-namespace.yaml
```

## <a name="create-the-aks-cluster-resources-for-sres"></a>Создание ресурсов кластера AKS для Срес

Теперь повторите предыдущие шаги, чтобы создать пространство имен, роль и Ролебиндинг для Срес.

Сначала создайте пространство имен для *выполняются* с помощью команды [kubectl Create Namespace][kubectl-create] :

```console
kubectl create namespace sre
```

Создайте файл с именем `role-sre-namespace.yaml` и вставьте следующий манифест YAML:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sre-user-full-access
  namespace: sre
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
```

Создайте роль с помощью команды [kubectl Apply][kubectl-apply] и укажите имя файла манифеста YAML:

```console
kubectl apply -f role-sre-namespace.yaml
```

Получите идентификатор ресурса для группы *опссре* с помощью команды [AZ AD Group показывать][az-ad-group-show] .

```azurecli-interactive
az ad group show --group opssre --query objectId -o tsv
```

Создайте Ролебиндинг для группы *опссре* , чтобы использовать ранее созданную роль для доступа к пространству имен. Создайте файл `rolebinding-sre-namespace.yaml` и вставьте в него следующий манифест YAML. В последней строке замените *граупобжектид*  на выходные данные идентификатора объекта Group из предыдущей команды:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sre-user-access
  namespace: sre
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: sre-user-full-access
subjects:
- kind: Group
  namespace: sre
  name: groupObjectId
```

Создайте Ролебиндинг с помощью команды [kubectl Apply][kubectl-apply] и укажите имя файла манифеста YAML:

```console
kubectl apply -f rolebinding-sre-namespace.yaml
```

## <a name="interact-with-cluster-resources-using-azure-ad-identities"></a>Взаимодействие с ресурсами кластера с помощью удостоверений Azure AD

Теперь давайте протестируем ожидаемые разрешения при создании ресурсов в кластере AKS и управлении ими. В этих примерах планируется и выполняется просмотр модулей Pod в назначенном пользователю пространстве имен. Затем вы пытаетесь запланировать и просмотреть модули Pod за пределами назначенного пространства имен.

Сначала сбросьте контекст *kubeconfig* с помощью команды [AZ AKS Get-Credential][az-aks-get-credentials] . В предыдущем разделе вы настроили контекст с помощью учетных данных администратора кластера. Пользователь с правами администратора обходит запросы на вход в Azure AD. Без `--admin` параметра применяется пользовательский контекст, требующий проверки подлинности всех запросов с помощью Azure AD.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing
```

Запланируйте базовый модуль NGINX с помощью команды [kubectl Run][kubectl-run] в пространстве имен *dev* :

```console
kubectl run nginx-dev --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --namespace dev
```

В качестве запроса на вход введите учетные данные для собственной `appdev@contoso.com` учетной записи, созданной в начале статьи. После успешного входа маркер учетной записи кэшируется для будущих `kubectl` команд. Расписание NGINX успешно запланировано, как показано в следующем примере выходных данных:

```console
$ kubectl run nginx-dev --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --namespace dev

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code B24ZD6FP8 to authenticate.

pod/nginx-dev created
```

Теперь используйте команду [kubectl Get][kubectl-get] Pod для просмотра модулей Pod в пространстве имен *dev* .

```console
kubectl get pods --namespace dev
```

Как показано в следующем примере выходных данных, модуль NGINX успешно *выполняется*:

```console
$ kubectl get pods --namespace dev

NAME        READY   STATUS    RESTARTS   AGE
nginx-dev   1/1     Running   0          4m
```

### <a name="create-and-view-cluster-resources-outside-of-the-assigned-namespace"></a>Создание и просмотр ресурсов кластера за пределами назначенного пространства имен

Теперь попробуйте просмотреть модули Pod за пределами пространства имен *dev* . Снова используйте команду [kubectl Get][kubectl-get] Pod, на следующий раз чтобы увидеть `--all-namespaces` следующее:

```console
kubectl get pods --all-namespaces
```

Членство пользователя в группе не имеет роли Kubernetes, которая разрешает это действие, как показано в следующем примере выходных данных:

```console
$ kubectl get pods --all-namespaces

Error from server (Forbidden): pods is forbidden: User "aksdev@contoso.com" cannot list resource "pods" in API group "" at the cluster scope
```

Аналогичным образом попробуйте запланировать Pod в другом пространстве имен, например в пространстве имен *выполняются* . Членство пользователя в группе не соответствует роли Kubernetes и Ролебиндинг для предоставления этих разрешений, как показано в следующем примере выходных данных:

```console
$ kubectl run nginx-dev --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --namespace sre

Error from server (Forbidden): pods is forbidden: User "aksdev@contoso.com" cannot create resource "pods" in API group "" in the namespace "sre"
```

### <a name="test-the-sre-access-to-the-aks-cluster-resources"></a>Проверка доступа выполняются к ресурсам кластера AKS

Чтобы убедиться, что членство в группе Azure AD и Kubernetes RBAC правильно работают между разными пользователями и группами, попробуйте выполнить предыдущие команды при входе от имени пользователя *опссре* .

Сбросьте контекст *kubeconfig* с помощью команды [AZ AKS Get-Credential][az-aks-get-credentials] , которая очищает ранее кэшированный маркер проверки подлинности для пользователя *аксдев* :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing
```

Попытайтесь запланировать и просмотреть модули Pod в назначенном пространстве имен *выполняются* . При появлении запроса войдите с помощью собственных `opssre@contoso.com` учетных данных, созданных в начале статьи:

```console
kubectl run nginx-sre --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --namespace sre
kubectl get pods --namespace sre
```

Как показано в следующем примере выходных данных, можно успешно создавать и просматривать модули Pod:

```console
$ kubectl run nginx-sre --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --namespace sre

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BM4RHP3FD to authenticate.

pod/nginx-sre created

$ kubectl get pods --namespace sre

NAME        READY   STATUS    RESTARTS   AGE
nginx-sre   1/1     Running   0
```

Теперь попробуйте просмотреть или запланировать модули Pod за пределами назначенного пространства имен выполняются:

```console
kubectl get pods --all-namespaces
kubectl run nginx-sre --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --namespace dev
```

Эти `kubectl` команды завершаются ошибкой, как показано в следующем примере выходных данных. Членство пользователя в группах и роль Kubernetes и Ролебиндингс не предоставляют разрешения на создание ресурсов и управление ресурсами в других пространствах имен:

```console
$ kubectl get pods --all-namespaces
Error from server (Forbidden): pods is forbidden: User "akssre@contoso.com" cannot list pods at the cluster scope

$ kubectl run nginx-sre --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --namespace dev
Error from server (Forbidden): pods is forbidden: User "akssre@contoso.com" cannot create pods in the namespace "dev"
```

## <a name="clean-up-resources"></a>Очистка ресурсов

В этой статье вы создали ресурсы в кластере AKS, а также пользователи и группы в Azure AD. Чтобы очистить все эти ресурсы, выполните следующие команды:

```azurecli-interactive
# Get the admin kubeconfig context to delete the necessary cluster resources
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin

# Delete the dev and sre namespaces. This also deletes the pods, Roles, and RoleBindings
kubectl delete namespace dev
kubectl delete namespace sre

# Delete the Azure AD user accounts for aksdev and akssre
az ad user delete --upn-or-object-id $AKSDEV_ID
az ad user delete --upn-or-object-id $AKSSRE_ID

# Delete the Azure AD groups for appdev and opssre. This also deletes the Azure role assignments.
az ad group delete --group appdev
az ad group delete --group opssre
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о защите кластеров Kubernetes см. в разделе [параметры доступа и удостоверений для AKS][rbac-authorization].

Рекомендации по управлению удостоверениями и ресурсами см. [в разделе рекомендации по проверке подлинности и авторизации в AKS][operator-best-practices-identity].

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-run]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

<!-- LINKS - internal -->
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[install-azure-cli]: /cli/azure/install-azure-cli
[azure-ad-aks-cli]: azure-ad-integration-cli.md
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-ad-group-create]: /cli/azure/ad/group#az-ad-group-create
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-ad-user-create]: /cli/azure/ad/user#az-ad-user-create
[az-ad-group-member-add]: /cli/azure/ad/group/member#az-ad-group-member-add
[az-ad-group-show]: /cli/azure/ad/group#az-ad-group-show
[rbac-authorization]: concepts-identity.md#kubernetes-role-based-access-control-kubernetes-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md

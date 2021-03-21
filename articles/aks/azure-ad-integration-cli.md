---
title: Интеграция Azure Active Directory со службой Kubernetes Azure (устаревшая служба)
description: Узнайте, как использовать Azure CLI для создания кластера Azure Kubernetes Service (AKS) с поддержкой Azure Active Directory (прежних версий).
services: container-service
author: TomGeske
ms.topic: article
ms.date: 07/20/2020
ms.author: thomasge
ms.openlocfilehash: 055afe24cb72b331e64e1a2b3d786503ef31a105
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102176405"
---
# <a name="integrate-azure-active-directory-with-azure-kubernetes-service-using-the-azure-cli-legacy"></a>Интеграция Azure Active Directory со службой Azure Kubernetes с помощью Azure CLI (прежних версий)

Службу Azure Kubernetes (AKS) можно настроить на использование Azure Active Directory (AD) для проверки подлинности пользователей. В этой конфигурации вы можете войти в кластер AKS с помощью маркера проверки подлинности Azure AD. Операторы кластера также могут настраивать Kubernetes управления доступом на основе ролей (Kubernetes RBAC) на основе удостоверения пользователя или членства в группе каталогов.

В этой статье показано, как создать необходимые компоненты Azure AD, а затем развернуть кластер с поддержкой Azure AD и создать базовую роль Kubernetes в кластере AKS.

Полный пример скрипта, используемый в этой статье, см. в разделе [Azure CLI Samples AKS Integration by Azure AD][complete-script].

> [!Important]
> AKS имеет новый усовершенствованный интерфейс [Azure AD, управляемый AKS][managed-aad] , который не требует управления сервером или клиентским приложением. Если вы хотите выполнить миграцию, следуйте приведенным [здесь][managed-aad-migrate]инструкциям.

## <a name="the-following-limitations-apply"></a>Действительны следующие ограничения.

- Azure AD можно включить только в кластере с поддержкой Kubernetes RBAC.
- Устаревшую интеграцию Azure AD можно включить только во время создания кластера.

## <a name="before-you-begin"></a>Перед началом

Необходимо установить и настроить Azure CLI версии 2.0.61 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

Перейдите по адресу [https://shell.azure.com](https://shell.azure.com), чтобы открыть Cloud Shell в браузере.

Для обеспечения согласованности и для выполнения команд в этой статье создайте переменную для нужного имени кластера AKS. В следующем примере используется имя *myakscluster*:

```console
aksname="myakscluster"
```

## <a name="azure-ad-authentication-overview"></a>Обзор аутентификации Azure AD

Кластеры AKS проходят аутентификацию Azure AD с помощью OpenID Connect. OpenID Connect представляет собой уровень идентификации на основе протокола OAuth 2.0. Дополнительные сведения о OpenID Connect Connect см. в [документации по Open ID Connect][open-id-connect].

Внутри кластера Kubernetes проверка подлинности на основе маркера веб-перехватчика используется для проверки маркеров проверки подлинности. Настройка такой проверка подлинности и ее управление являются частью кластера AKS. Дополнительные сведения о проверке подлинности на основе маркера веб-перехватчика см. в [Документации по аутентификации веб-перехватчика][kubernetes-webhook].

> [!NOTE]
> При настройке Azure AD для аутентификации AKS настраиваются два приложения Azure AD. Эту операцию должен выполнять администратор клиента Azure.

## <a name="create-azure-ad-server-component"></a>Создание компонента сервера Azure AD

Для интеграции с AKS вы создаете и используете приложение Azure AD, которое выступает в качестве конечной точки для запросов на идентификацию. Первое необходимое приложение Azure AD получает членство в группе Azure AD для пользователя.

Создайте компонент серверного приложения с помощью команды [AZ AD App Create][az-ad-app-create] , а затем обновите утверждения членства в группе с помощью команды [AZ AD App Update][az-ad-app-update] . В следующем примере используется переменная *акснаме* , определенная в разделе [перед началом](#before-you-begin) выполнения, и создается переменная.

```azurecli-interactive
# Create the Azure AD application
serverApplicationId=$(az ad app create \
    --display-name "${aksname}Server" \
    --identifier-uris "https://${aksname}Server" \
    --query appId -o tsv)

# Update the application group membership claims
az ad app update --id $serverApplicationId --set groupMembershipClaims=All
```

Теперь создайте субъект-службу для серверного приложения с помощью команды [AZ AD SP Create][az-ad-sp-create] . Этот субъект-служба используется для проверки подлинности в платформе Azure. Затем получите секрет субъекта-службы с помощью команды [AZ AD SP Credential Reset][az-ad-sp-credential-reset] и назначьте переменной с именем *сервераппликатионсекрет* для использования в одном из следующих шагов:

```azurecli-interactive
# Create a service principal for the Azure AD application
az ad sp create --id $serverApplicationId

# Get the service principal secret
serverApplicationSecret=$(az ad sp credential reset \
    --name $serverApplicationId \
    --credential-description "AKSPassword" \
    --query password -o tsv)
```

Субъекту-службе Azure AD требуются разрешения для выполнения следующих действий:

* Чтение данных каталога
* Вход в систему и чтение профиля пользователя

Назначьте эти разрешения с помощью команды [AZ AD App Permission Add][az-ad-app-permission-add] :

```azurecli-interactive
az ad app permission add \
    --id $serverApplicationId \
    --api 00000003-0000-0000-c000-000000000000 \
    --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope 06da0dbc-49e2-44d2-8312-53f166ab848a=Scope 7ab1d382-f21e-4acd-a863-ba3e13f7da61=Role
```

Наконец, предоставьте разрешения, назначенные на предыдущем шаге для серверного приложения, с помощью команды [AZ AD App Grant][az-ad-app-permission-grant] . Этот шаг завершается ошибкой, если текущая учетная запись не является администратором клиента. Кроме того, необходимо добавить разрешения для приложения Azure AD, чтобы запросить сведения, которые в противном случае требуют согласия администратора, с помощью команды [AZ AD App Permission Permissions с согласия администратора][az-ad-app-permission-admin-consent]:

```azurecli-interactive
az ad app permission grant --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000
az ad app permission admin-consent --id  $serverApplicationId
```

## <a name="create-azure-ad-client-component"></a>Создание клиентского компонента Azure AD

Второе приложение Azure AD используется, когда пользователь входит в кластер AKS с помощью интерфейса командной строки Kubernetes ( `kubectl` ). Это клиентское приложение принимает запрос на проверку подлинности от пользователя и проверяет свои учетные данные и разрешения. Создайте приложение Azure AD для клиентского компонента с помощью команды [AZ AD App Create][az-ad-app-create] .

```azurecli-interactive
clientApplicationId=$(az ad app create \
    --display-name "${aksname}Client" \
    --native-app \
    --reply-urls "https://${aksname}Client" \
    --query appId -o tsv)
```

Создайте субъект-службу для клиентского приложения с помощью команды [AZ AD SP Create][az-ad-sp-create] .

```azurecli-interactive
az ad sp create --id $clientApplicationId
```

Получите идентификатор oAuth2 для серверного приложения, чтобы разрешить поток проверки подлинности между двумя компонентами приложения с помощью команды [AZ AD App демонстрация][az-ad-app-show] . Этот идентификатор oAuth2 используется на следующем шаге.

```azurecli-interactive
oAuthPermissionId=$(az ad app show --id $serverApplicationId --query "oauth2Permissions[0].id" -o tsv)
```

Добавьте разрешения для клиентских приложений и компонентов серверного приложения, чтобы использовать поток обмена данными oAuth2 с помощью команды [AZ AD App Permission Add][az-ad-app-permission-add] . Затем предоставьте разрешения на обмен данными между клиентским приложением и серверным приложением с помощью команды [AZ AD App Grant][az-ad-app-permission-grant] .

```azurecli-interactive
az ad app permission add --id $clientApplicationId --api $serverApplicationId --api-permissions ${oAuthPermissionId}=Scope
az ad app permission grant --id $clientApplicationId --api $serverApplicationId
```

## <a name="deploy-the-cluster"></a>Развертывание кластера

Создав два приложения Azure AD, создадим сам кластер AKS. Сначала создайте группу ресурсов с помощью команды [AZ Group Create][az-group-create] . В следующем примере создается группа ресурсов в регионе *EastUS* :

Создайте группу ресурсов для кластера.

```azurecli-interactive
az group create --name myResourceGroup --location EastUS
```

Получите идентификатор клиента подписки Azure с помощью команды [AZ Account показывать][az-account-show] . Затем создайте кластер AKS с помощью команды [AZ AKS Create][az-aks-create] . Команда для создания кластера AKS предоставляет идентификаторы серверных и клиентских приложений, секрет субъекта службы серверного приложения и идентификатор клиента:

```azurecli-interactive
tenantId=$(az account show --query tenantId -o tsv)

az aks create \
    --resource-group myResourceGroup \
    --name $aksname \
    --node-count 1 \
    --generate-ssh-keys \
    --aad-server-app-id $serverApplicationId \
    --aad-server-app-secret $serverApplicationSecret \
    --aad-client-app-id $clientApplicationId \
    --aad-tenant-id $tenantId
```

Наконец, получите учетные данные администратора кластера с помощью команды [AZ AKS Get-Credential][az-aks-get-credentials] . В одном из следующих шагов вы получите учетные данные регулярного кластера *пользователей* , чтобы увидеть последовательность проверки подлинности Azure AD в действии.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name $aksname --admin
```

## <a name="create-kubernetes-rbac-binding"></a>Создание привязки RBAC Kubernetes

Прежде чем использовать учетную запись Azure Active Directory с кластером AKS, необходимо создать привязку роли или привязку роли кластера. *Роли* определяют разрешения для предоставления, а *привязки* применяют их к желаемым пользователям. Эти назначения могут применяться для определенного пространства имен или в масштабах всего кластера. Дополнительные сведения см. [в разделе Использование авторизации KUBERNETES RBAC][rbac-authorization].

Получите имя участника-пользователя (UPN) для пользователя, вошедшего в систему, с помощью команды [AZ AD выполнившего вход — User демонстрация][az-ad-signed-in-user-show] . В следующем шаге эта учетная запись пользователя включена для интеграции с Azure AD.

```azurecli-interactive
az ad signed-in-user show --query userPrincipalName -o tsv
```

> [!IMPORTANT]
> Если пользователь, которому вы предоставляете привязку Kubernetes RBAC для, находится в том же клиенте Azure AD, назначьте разрешения на основе *userPrincipalName*. Если пользователь находится в другом клиенте Azure AD, вместо него следует запрашивать и использовать свойство *ObjectID* .

Создайте манифест YAML с именем `basic-azure-ad-binding.yaml` и вставьте следующее содержимое. В последней строке замените *userPrincipalName_or_objectId*  на выходные данные имени участника-пользователя или идентификатора объекта из предыдущей команды:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contoso-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: userPrincipalName_or_objectId
```

Создайте Клустерролебиндинг с помощью команды [kubectl Apply][kubectl-apply] и укажите имя файла манифеста YAML:

```console
kubectl apply -f basic-azure-ad-binding.yaml
```

## <a name="access-cluster-with-azure-ad"></a>Доступ к кластеру с помощью Azure AD

Теперь давайте протестируем интеграцию аутентификации Azure AD для кластера AKS. Задайте `kubectl` для контекста конфигурации использование обычных учетных данных пользователя. Этот контекст передает все запросы проверки подлинности обратно через Azure AD.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name $aksname --overwrite-existing
```

Теперь используйте команду [kubectl Get][kubectl-get] Pod для просмотра модулей Pod во всех пространствах имен:

```console
kubectl get pods --all-namespaces
```

Вы получаете запрос на вход для аутентификации с помощью учетных данных Azure AD с помощью веб-браузера. После успешной проверки подлинности `kubectl` команда отображает модули Pod в кластере AKS, как показано в следующем примере выходных данных:

```console
kubectl get pods --all-namespaces
```

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BYMK7UXVD to authenticate.

NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-754f947b4-2v75r                 1/1     Running   0          23h
kube-system   coredns-754f947b4-tghwh                 1/1     Running   0          23h
kube-system   coredns-autoscaler-6fcdb7d64-4wkvp      1/1     Running   0          23h
kube-system   heapster-5fb7488d97-t5wzk               2/2     Running   0          23h
kube-system   kube-proxy-2nd5m                        1/1     Running   0          23h
kube-system   kube-svc-redirect-swp9r                 2/2     Running   0          23h
kube-system   kubernetes-dashboard-847bb4ddc6-trt7m   1/1     Running   0          23h
kube-system   metrics-server-7b97f9cd9-btxzz          1/1     Running   0          23h
kube-system   tunnelfront-6ff887cffb-xkfmq            1/1     Running   0          23h
```

Токен проверки подлинности, полученный для `kubectl` , кэшируется. После истечения срока действия маркера или повторного создания файла конфигурации Kubernetes будет предложено только выполнить вход.

Если вы видите сообщение об ошибке авторизации после успешного входа с помощью веб-браузера, как показано в следующем примере выходных данных, проверьте следующие возможные проблемы.

```output
error: You must be logged in to the server (Unauthorized)
```

* Вы определили соответствующий идентификатор объекта или имя участника-пользователя в зависимости от того, находится ли учетная запись пользователя в том же клиенте Azure AD или нет.
* Пользователь не может быть членом более чем 200 групп.
* Секрет, определенный в регистрации приложения для сервера, соответствует значению, заданному с помощью `--aad-server-app-secret`

## <a name="next-steps"></a>Дальнейшие действия

Полный сценарий, содержащий команды, приведенные в этой статье, см. в разделе [сценарий интеграции Azure AD в репозитории примеров AKS][complete-script].

Сведения об использовании пользователей и групп Azure AD для управления доступом к ресурсам кластера см. в статье [Управление доступом к ресурсам кластера с помощью управления доступом на основе ролей Kubernetes и удостоверений Azure AD в AKS][azure-ad-rbac].

Дополнительные сведения о защите кластеров Kubernetes см. в разделе [параметры доступа и удостоверений для AKS][rbac-authorization].

Рекомендации по управлению удостоверениями и ресурсами см. [в разделе рекомендации по проверке подлинности и авторизации в AKS][operator-best-practices-identity].

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[complete-script]: https://github.com/Azure-Samples/azure-cli-samples/tree/master/aks/azure-ad-integration/azure-ad-integration.sh

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]: ../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[az-ad-app-create]: /cli/azure/ad/app#az-ad-app-create
[az-ad-app-update]: /cli/azure/ad/app#az-ad-app-update
[az-ad-sp-create]: /cli/azure/ad/sp#az-ad-sp-create
[az-ad-app-permission-add]: /cli/azure/ad/app/permission#az-ad-app-permission-add
[az-ad-app-permission-grant]: /cli/azure/ad/app/permission#az-ad-app-permission-grant
[az-ad-app-permission-admin-consent]: /cli/azure/ad/app/permission#az-ad-app-permission-admin-consent
[az-ad-app-show]: /cli/azure/ad/app#az-ad-app-show
[az-group-create]: /cli/azure/group#az-group-create
[az-account-show]: /cli/azure/account#az-account-show
[az-ad-signed-in-user-show]: /cli/azure/ad/signed-in-user#az-ad-signed-in-user-show
[install-azure-cli]: /cli/azure/install-azure-cli
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset
[rbac-authorization]: concepts-identity.md#kubernetes-role-based-access-control-kubernetes-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[managed-aad]: managed-aad.md
[managed-aad-migrate]: managed-aad.md#upgrading-to-aks-managed-azure-ad-integration

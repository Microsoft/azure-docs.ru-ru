---
title: Использование Azure AD в службе Kubernetes Azure
description: Узнайте, как использовать Azure AD в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 08/26/2020
ms.author: thomasge
ms.openlocfilehash: 534c355961bb87a816f5ba50a3cc2d397e544a15
ms.sourcegitcommit: dd24c3f35e286c5b7f6c3467a256ff85343826ad
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/29/2021
ms.locfileid: "99072360"
---
# <a name="aks-managed-azure-active-directory-integration"></a>Интеграция Azure Active Directory с управляемым AKS

Интеграция Azure AD с AKS разработана для упрощения процесса интеграции Azure AD, где пользователям ранее требовалось было создавать клиентское приложение, серверное приложение и требовалось клиент Azure AD для предоставления разрешений на чтение каталога. В новой версии поставщик ресурсов AKS управляет клиентскими и серверными приложениями.

## <a name="azure-ad-authentication-overview"></a>Обзор аутентификации Azure AD

Администраторы кластера могут настроить Kubernetes управления доступом на основе ролей (Kubernetes RBAC) на основе удостоверения пользователя или членства в группе каталогов. Кластеры AKS проходят аутентификацию Azure AD с помощью OpenID Connect. OpenID Connect представляет собой уровень идентификации на основе протокола OAuth 2.0. Дополнительные сведения о OpenID Connect Connect см. в [документации по Open ID Connect][open-id-connect].

Дополнительные сведения о потоке интеграции Azure AD см. в [документации по основным понятиям интеграции Azure Active Directory](concepts-identity.md#azure-active-directory-integration).

## <a name="limitations"></a>Ограничения 

* Невозможно отключить управляемую AKS интеграцию Azure AD
* кластеры с поддержкой Kubernetes RBAC не поддерживаются для интеграции с Azure AD, управляемой AKS
* Изменение клиента Azure AD, связанного с интеграцией Azure AD, управляемой AKS, не поддерживается

## <a name="prerequisites"></a>Предварительные требования

* Azure CLI версии 2.11.0 или более поздней.
* Kubectl с минимальной версией [1.18.1](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#v1181) или [кубелогин](https://github.com/Azure/kubelogin)
* Если используется [Helm](https://github.com/helm/helm), минимальная версия Helm 3,3.

> [!Important]
> Необходимо использовать Kubectl с минимальной версией 1.18.1 или кубелогин. Если вы не используете правильную версию, вы увидите проблемы с проверкой подлинности.

Чтобы установить kubectl и кубелогин, используйте следующие команды:

```azurecli-interactive
sudo az aks install-cli
kubectl version --client
kubelogin --version
```

Используйте [эти инструкции](https://kubernetes.io/docs/tasks/tools/install-kubectl/) для других операционных систем.

## <a name="before-you-begin"></a>Перед началом работы

Для кластера требуется группа Azure AD. Эта группа необходима в качестве группы администраторов кластера для предоставления разрешений администратора кластера. Вы можете использовать существующую группу Azure AD или создать новую. Запишите идентификатор объекта вашей группы Azure AD.

```azurecli-interactive
# List existing groups in the directory
az ad group list --filter "displayname eq '<group-name>'" -o table
```

Чтобы создать новую группу Azure AD для администраторов кластера, используйте следующую команду:

```azurecli-interactive
# Create an Azure AD group
az ad group create --display-name myAKSAdminGroup --mail-nickname myAKSAdminGroup
```

## <a name="create-an-aks-cluster-with-azure-ad-enabled"></a>Создание кластера AKS с включенной службой Azure AD

Создайте кластер AKS, используя следующие команды интерфейса командной строки.

Создайте группу ресурсов Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location centralus
```

Создание кластера AKS и включение административного доступа для вашей группы Azure AD

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Успешное создание кластера Azure AD, управляемого AKS, содержит следующий раздел в тексте ответа.
```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

После создания кластера можно приступить к его созданию.

## <a name="access-an-azure-ad-enabled-cluster"></a>Доступ к кластеру с поддержкой Azure AD

Для выполнения следующих действий потребуется встроенная роль [пользователя кластера службы Kubernetes Azure](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-user-role) .

Получите учетные данные пользователя для доступа к кластеру:
 
```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```
Следуйте инструкциям по входу.

Используйте команду kubectl Get Nodes, чтобы просмотреть узлы в кластере:

```azurecli-interactive
kubectl get nodes

NAME                       STATUS   ROLES   AGE    VERSION
aks-nodepool1-15306047-0   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-1   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-2   Ready    agent   102m   v1.15.10
```
Настройте [Управление доступом на основе ролей Azure (Azure RBAC)](./azure-ad-rbac.md) , чтобы настроить дополнительные группы безопасности для кластеров.

## <a name="troubleshooting-access-issues-with-azure-ad"></a>Устранение проблем с доступом к Azure AD

> [!Important]
> Описанные ниже действия предназначены для обхода обычной проверки подлинности группы Azure AD. Используйте их только в экстренной ситуации.

Если вы постоянно блокируете доступ к действующей группе Azure AD с доступом к кластеру, вы по-прежнему можете получить учетные данные администратора для прямого доступа к кластеру.

Чтобы выполнить эти действия, необходимо иметь доступ к встроенной роли [администратора кластера службы Kubernetes Azure](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-admin-role) .

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster --admin
```

## <a name="enable-aks-managed-azure-ad-integration-on-your-existing-cluster"></a>Включение управляемой AKS интеграции Azure AD в имеющемся кластере

Вы можете включить управляемую AKS интеграцию Azure AD в существующем кластере с поддержкой Kubernetes RBAC. Убедитесь, что Группа администраторов настроена для сохранения доступа к кластеру.

```azurecli-interactive
az aks update -g MyResourceGroup -n MyManagedCluster --enable-aad --aad-admin-group-object-ids <id-1> [--aad-tenant-id <id>]
```

Успешная активация кластера Azure AD, управляемого AKS, содержит следующий раздел в тексте ответа.

```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Скачайте учетные данные пользователя еще раз, чтобы получить доступ к кластеру, выполнив действия, описанные [здесь][access-cluster].

## <a name="upgrading-to-aks-managed-azure-ad-integration"></a>Обновление до управляемой AKS интеграции Azure AD

Если кластер использует устаревшую интеграцию Azure AD, вы можете выполнить обновление до управляемой AKS интеграции Azure AD.

```azurecli-interactive
az aks update -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Успешная миграция кластера Azure AD, управляемого AKS, содержит следующий раздел в тексте ответа.

```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Если вы хотите получить доступ к кластеру, выполните действия, описанные [здесь][access-cluster].

## <a name="non-interactive-sign-in-with-kubelogin"></a>Неинтерактивный вход с помощью кубелогин

Существуют некоторые неинтерактивные сценарии, например конвейеры непрерывной интеграции, которые в настоящее время недоступны в kubectl. Вы можете использовать [`kubelogin`](https://github.com/Azure/kubelogin) для доступа к кластеру с неинтерактивным входом субъекта-службы.

## <a name="use-conditional-access-with-azure-ad-and-aks"></a>Использование условного доступа с помощью Azure AD и AKS

При интеграции Azure AD с кластером AKS можно также использовать [Условный доступ][aad-conditional-access] для управления доступом к кластеру.

> [!NOTE]
> Условный доступ Azure AD является Azure AD Premiumной возможностью.

Чтобы создать пример политики условного доступа для использования с AKS, выполните следующие действия.

1. В верхней части портал Azure найдите и выберите Azure Active Directory.
1. В меню для Azure Active Directory на левой стороне выберите *корпоративные приложения*.
1. В меню для корпоративных приложений с левой стороны выберите *Условный доступ*.
1. В меню для условного доступа на левой стороне выберите *политики* , а затем — *Новая политика*.
    :::image type="content" source="./media/managed-aad/conditional-access-new-policy.png" alt-text="Добавление политики условного доступа":::
1. Введите имя политики, например *AKS-Policy*.
1. Выберите *Пользователи и группы*, а затем в разделе *включить* выберите *выбрать пользователей и группы*. Выберите пользователей и группы, для которых необходимо применить политику. В этом примере выберите ту же группу Azure AD с административным доступом к кластеру.
    :::image type="content" source="./media/managed-aad/conditional-access-users-groups.png" alt-text="Выбор пользователей или групп для применения политики условного доступа":::
1. Выберите *облачные приложения или действия*, а затем в разделе *включить* выберите *выберите приложения*. Найдите *службу Azure Kubernetes* и выберите *Azure Kubernetes Service AAD Server*.
    :::image type="content" source="./media/managed-aad/conditional-access-apps.png" alt-text="Выбор Azure Kubernetes Service AD для применения политики условного доступа":::
1. В разделе *Элементы управления доступом* выберите *Предоставить*. Выберите *предоставить доступ* , а затем *устройство должно быть помечено как соответствующее*.
    :::image type="content" source="./media/managed-aad/conditional-access-grant-compliant.png" alt-text="Включение только соответствующих устройств для политики условного доступа":::
1. В разделе *включить политику* выберите *включить* , а затем — *создать*.
    :::image type="content" source="./media/managed-aad/conditional-access-enable-policy.png" alt-text="Включение политики условного доступа":::

Получите учетные данные пользователя для доступа к кластеру, например:

```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```

Следуйте инструкциям по входу.

Используйте `kubectl get nodes` команду для просмотра узлов в кластере:

```azurecli-interactive
kubectl get nodes
```

Следуйте инструкциям, чтобы войти снова. Обратите внимание на сообщение об ошибке, сообщающее, что вы успешно выполнили вход, но администратору требуется, чтобы устройство, запрашивающее доступ, управлялось Azure AD для доступа к ресурсу.

В портал Azure перейдите к Azure Active Directory, выберите *корпоративные приложения* и в разделе *действие* выберите *входы*. Обратите внимание на запись в верхней части с *состоянием* *Failed (сбой* ) и *условным доступом* *успешно*. Выберите запись, а затем в области *сведения* выберите *Условный доступ* . Обратите внимание, что в списке указана политика условного доступа.

:::image type="content" source="./media/managed-aad/conditional-access-sign-in-activity.png" alt-text="Ошибка входа в систему из-за политики условного доступа":::

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об [интеграции с Azure RBAC для авторизации Kubernetes][azure-rbac-integration]
* Сведения об [интеграции Azure AD с KUBERNETES RBAC][azure-ad-rbac].
* Используйте [кубелогин](https://github.com/Azure/kubelogin) для доступа к функциям для проверки подлинности Azure, которые недоступны в kubectl.
* Узнайте больше о [концепциях идентификации AKS и Kubernetes][aks-concepts-identity].
* Используйте [шаблоны Azure Resource Manager (ARM) ][aks-arm-template] для создания кластеров с поддержкой Azure AD, управляемых AKS.

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters

<!-- LINKS - Internal -->
[aad-conditional-access]: ../active-directory/conditional-access/overview.md
[azure-rbac-integration]: manage-azure-rbac.md
[aks-concepts-identity]: concepts-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]:../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[rbac-authorization]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[azure-ad-cli]: azure-ad-integration-cli.md
[access-cluster]: #access-an-azure-ad-enabled-cluster
[aad-migrate]: #upgrading-to-aks-managed-azure-ad-integration

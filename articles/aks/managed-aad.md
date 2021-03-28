---
title: Использование Azure AD в службе Kubernetes Azure
description: Узнайте, как использовать Azure AD в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 02/1/2021
ms.author: miwithro
ms.openlocfilehash: b7918ecc31fe152bd25153ac8c899ce3ff8fdacb
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105640590"
---
# <a name="aks-managed-azure-active-directory-integration"></a>Интеграция Azure Active Directory с управляемым AKS

Интеграция Azure AD с AKS разработана для упрощения процесса интеграции Azure AD, где пользователям ранее требовалось было создавать клиентское приложение, серверное приложение и требовалось клиент Azure AD для предоставления разрешений на чтение каталога. В новой версии поставщик ресурсов AKS управляет клиентскими и серверными приложениями.

## <a name="azure-ad-authentication-overview"></a>Обзор аутентификации Azure AD

Администраторы кластера могут настроить Kubernetes управления доступом на основе ролей (Kubernetes RBAC) на основе удостоверения пользователя или членства в группе каталогов. Кластеры AKS проходят аутентификацию Azure AD с помощью OpenID Connect. OpenID Connect представляет собой уровень идентификации на основе протокола OAuth 2.0. Дополнительные сведения о OpenID Connect Connect см. в [документации по Open ID Connect][open-id-connect].

Дополнительные сведения о потоке интеграции Azure AD см. в [документации по основным понятиям интеграции Azure Active Directory](concepts-identity.md#azure-active-directory-integration).

## <a name="limitations"></a>Ограничения 

* Невозможно отключить управляемую AKS интеграцию Azure AD
* Изменение управляемого AKS кластера Azure AD до устаревшего AAD не поддерживается
* кластеры с поддержкой Kubernetes RBAC не поддерживаются для интеграции с Azure AD, управляемой AKS
* Изменение клиента Azure AD, связанного с интеграцией Azure AD, управляемой AKS, не поддерживается

## <a name="prerequisites"></a>Предварительные условия

* Azure CLI версии 2.11.0 или более поздней.
* Kubectl с минимальной версией [1.18.1](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#v1181) или [кубелогин](https://github.com/Azure/kubelogin)
* Если используется [Helm](https://github.com/helm/helm), минимальная версия Helm 3,3.

> [!Important]
> Необходимо использовать Kubectl с минимальной версией 1.18.1 или кубелогин. Разница между дополнительными версиями Kubernetes и kubectl не должна превышать 1 версию. Если вы не используете правильную версию, вы увидите проблемы с проверкой подлинности.

Чтобы установить kubectl и кубелогин, используйте следующие команды:

```azurecli-interactive
sudo az aks install-cli
kubectl version --client
kubelogin --version
```

Используйте [эти инструкции](https://kubernetes.io/docs/tasks/tools/install-kubectl/) для других операционных систем.

## <a name="before-you-begin"></a>Подготовка

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

## <a name="configure-just-in-time-cluster-access-with-azure-ad-and-aks"></a>Настройка JIT-доступа к кластеру с помощью Azure AD и AKS

Другим вариантом управления доступом к кластеру является использование управление привилегированными пользователями (PIM) для JIT-запросов.

>[!NOTE]
> PIM — это Azure AD Premiumная возможность, для которой необходим SKU уровня "Премиум P2". Дополнительные сведения о SKU Azure AD см. в разделе [ценовые указания][aad-pricing].

Чтобы интегрировать JIT-запросы доступа к кластеру AKS с помощью управляемой AKS интеграции Azure AD, выполните следующие действия.

1. В верхней части портал Azure найдите и выберите Azure Active Directory.
1. Запишите идентификатор клиента, который указан для остальных инструкций, как `<tenant-id>` :::image type="content" source="./media/managed-aad/jit-get-tenant-id.png" alt-text="в веб-браузере, портал Azure экран для Azure Active Directory отображается с ВЫДЕЛЕНным идентификатором клиента.":::
1. В меню для Azure Active Directory на левой стороне в разделе *Управление* выбор *групп* выберите *Новая группа*.
    :::image type="content" source="./media/managed-aad/jit-create-new-group.png" alt-text="Отображает экран портал Azure Active Directory групп с выделенным параметром &quot;создать группу&quot;.":::
1. Убедитесь, что выбран тип группы *Безопасность* , и введите имя группы, например *мижитграуп*. В разделе *роли Azure AD могут быть назначены этой группе (Предварительная версия)* выберите *Да*. Наконец, выберите *Создать*.
    :::image type="content" source="./media/managed-aad/jit-new-group-created.png" alt-text="Отображает экран создания новой группы портал Azure.":::
1. Будет возвращена страница " *группы* ". Выберите созданную группу и запишите идентификатор объекта, на который ссылаются остальные инструкции как `<object-id>` .
    :::image type="content" source="./media/managed-aad/jit-get-object-id.png" alt-text="Отображение портал Azure экрана только что созданной группы с выделением идентификатора объекта":::
1. Разверните кластер AKS с интеграцией Azure AD, управляемой AKS, используя `<tenant-id>` `<object-id>` значения и из предыдущего:
    ```azurecli-interactive
    az aks create -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <object-id> --aad-tenant-id <tenant-id>
    ```
1. Вернитесь в портал Azure в меню *действия* на левой стороне выберите *привилегированный доступ (Предварительная версия)* и выберите *включить привилегированный доступ*.
    :::image type="content" source="./media/managed-aad/jit-enabling-priv-access.png" alt-text="Отобразится страница портал Azure привилегированный доступ (Предварительная версия) с выделенным параметром &quot;включить привилегированный доступ&quot;.":::
1. Выберите *добавить назначения* , чтобы начать предоставление доступа.
    :::image type="content" source="./media/managed-aad/jit-add-active-assignment.png" alt-text="Отобразится экран портал Azure привилегированный доступ (Предварительная версия) после включения. Выделен параметр &quot;добавить назначения&quot;.":::
1. Выберите роль *член* и выберите пользователей и группы, которым вы хотите предоставить доступ к кластеру. Эти назначения могут быть изменены администратором группы в любое время. Когда вы будете готовы к переходу, нажмите кнопку *Далее*.
    :::image type="content" source="./media/managed-aad/jit-adding-assignment.png" alt-text="Отобразится экран &quot;Добавление назначения&quot; портал Azure с примером пользователя, выбранным для добавления в качестве члена. Параметр &quot;Далее&quot; выделен.":::
1. Выберите тип назначения *Активная*, желаемая длительность и укажите обоснование. Когда вы будете готовы к продолжению, нажмите кнопку *назначить*. Дополнительные сведения о типах назначений см. [в разделе Назначение права для привилегированной группы доступа (Предварительная версия) в Управление привилегированными пользователями][aad-assignments].
    :::image type="content" source="./media/managed-aad/jit-set-active-assignment-details.png" alt-text="Появится экран настройки добавления назначения портал Azure. Выбран тип назначения &quot;Active&quot;, и указано обоснование. Параметр &quot;Assign&quot; выделен.":::

После выполнения назначений убедитесь, что JIT-доступ работает путем доступа к кластеру. Пример:

```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```

Выполните действия для входа.

Используйте `kubectl get nodes` команду для просмотра узлов в кластере:

```azurecli-interactive
kubectl get nodes
```

Обратите внимание на требование проверки подлинности и выполните действия по проверке подлинности. В случае успеха вы увидите результат, аналогичный приведенному ниже:

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AAAAAAAAA to authenticate.
NAME                                STATUS   ROLES   AGE     VERSION
aks-nodepool1-61156405-vmss000000   Ready    agent   6m36s   v1.18.14
aks-nodepool1-61156405-vmss000001   Ready    agent   6m42s   v1.18.14
aks-nodepool1-61156405-vmss000002   Ready    agent   6m33s   v1.18.14
```

### <a name="troubleshooting"></a>Устранение неполадок

Если функция `kubectl get nodes` возвращает ошибку, аналогичную следующей:

```output
Error from server (Forbidden): nodes is forbidden: User "aaaa11111-11aa-aa11-a1a1-111111aaaaa" cannot list resource "nodes" in API group "" at the cluster scope
```

Убедитесь, что администратор группы безопасности предоставил вашей учетной записи *активное* назначение.

## <a name="next-steps"></a>Дальнейшие шаги

* Сведения об [интеграции с Azure RBAC для авторизации Kubernetes][azure-rbac-integration]
* Сведения об [интеграции Azure AD с KUBERNETES RBAC][azure-ad-rbac].
* Используйте [кубелогин](https://github.com/Azure/kubelogin) для доступа к функциям для проверки подлинности Azure, которые недоступны в kubectl.
* Узнайте больше о [концепциях идентификации AKS и Kubernetes][aks-concepts-identity].
* Используйте [шаблоны Azure Resource Manager (ARM) ][aks-arm-template] для создания кластеров с поддержкой Azure AD, управляемых AKS.

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters
[aad-pricing]: https://azure.microsoft.com/pricing/details/active-directory/

<!-- LINKS - Internal -->
[aad-conditional-access]: ../active-directory/conditional-access/overview.md
[azure-rbac-integration]: manage-azure-rbac.md
[aks-concepts-identity]: concepts-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]:../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[rbac-authorization]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[azure-ad-cli]: azure-ad-integration-cli.md
[access-cluster]: #access-an-azure-ad-enabled-cluster
[aad-migrate]: #upgrading-to-aks-managed-azure-ad-integration
[aad-assignments]: ../active-directory/privileged-identity-management/groups-assign-member-owner.md#assign-an-owner-or-member-of-a-group

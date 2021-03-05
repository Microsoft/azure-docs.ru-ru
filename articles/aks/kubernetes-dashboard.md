---
title: Управление кластером Службы Azure Kubernetes с помощью веб-панели мониторинга
description: Узнайте, как использовать панель мониторинга пользовательского веб-интерфейса Kubernetes для управления кластером Службы Azure Kubernetes (AKS)
services: container-service
author: mlearned
ms.topic: article
ms.date: 06/03/2020
ms.author: mlearned
ms.openlocfilehash: acaeaa2e5338c86fa59d0e2941719f8fa2708ef1
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102176827"
---
# <a name="access-the-kubernetes-web-dashboard-in-azure-kubernetes-service-aks"></a>Подключение веб-панели мониторинга Kubernetes в Службе Azure Kubernetes (AKS)

Служба Kubernetes включает в себя веб-панель мониторинга, которая может использоваться для базовых операций управления. Эта панель мониторинга позволяет просматривать базовое состояние работоспособности и метрики для приложений, создавать и развертывать службы, а также изменять существующие приложения. В этом документе рассматривается запуск панели мониторинга Kubernetes с помощью Azure CLI, а также некоторые основные операции с ней.

Дополнительные сведения о панели мониторинга Kubernetes см. в разделе [панель мониторинга пользовательского веб-интерфейса Kubernetes][kubernetes-dashboard]. AKS использует панель мониторинга с открытым исходным кодом версии 2,0 и более поздней.

> [!WARNING]
> **Надстройка панели мониторинга AKS задается для устаревания. Вместо этого используйте [представление ресурсов Kubernetes в портал Azure (Предварительная версия)][kubernetes-portal] .** 
> * Панель мониторинга Kubernetes включена по умолчанию для кластеров с версией Kubernetes менее 1,18.
> * Надстройка панели мониторинга будет отключена по умолчанию для всех новых кластеров, созданных в Kubernetes 1,18 или более поздней версии. 
 > * Начиная с Kubernetes 1,19 в предварительной версии AKS больше не будет поддерживать установку управляемой надстройки KUBE-Dashboard. 
 > * Существующие кластеры с включенной надстройкой не будут затронуты. Пользователи будут по-прежнему иметь возможность вручную устанавливать панель мониторинга с открытым исходным кодом как программное обеспечение, установленное пользователем.

## <a name="before-you-begin"></a>Перед началом

В шагах, описанных в этом документе, предполагается, что вы создали кластер AKS и установили `kubectl` Подключение к кластеру. Если вам нужно создать кластер AKS, см. раздел [Краткое руководство. Развертывание кластера службы Kubernetes Azure с помощью Azure CLI][aks-quickstart].

Также требуется Azure CLI версии 2.6.0 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="disable-the-kubernetes-dashboard"></a>Отключение панели мониторинга Kubernetes

Надстройка KUBE-Dashboard **включена по умолчанию для кластеров старше K8s 1,18**. Надстройку можно отключить, выполнив следующую команду.

``` azurecli
az aks disable-addons -g myRG -n myAKScluster -a kube-dashboard
```

## <a name="start-the-kubernetes-dashboard"></a>Запуск панели мониторинга Kubernetes

> [!WARNING]
> Надстройка панели мониторинга AKS устарела для версий 1.19 +. Вместо этого используйте [представление ресурсов Kubernetes в портал Azure (Предварительная версия)][kubernetes-portal] . 
> * Следующая команда теперь открывает представление ресурсов портала Azure вместо панели мониторинга kubernetes для версий 1,19 и выше.

Чтобы запустить панель мониторинга Kubernetes в кластере, используйте команду [AZ AKS Browse][az-aks-browse] . Для этой команды требуется установка надстройки KUBE-Dashboard в кластере, которая включена по умолчанию в кластерах, где запущена любая версия, отличная от Kubernetes 1,18.

В следующем примере создается панель мониторинга для кластера с именем *myAKSCluster* в группе ресурсов *myResourceGroup*.

```azurecli
az aks browse --resource-group myResourceGroup --name myAKSCluster
```

Эта команда создает прокси-сервер между системой разработки и API Kubernetes и открывает панель мониторинга Kubernetes в веб-браузере. Если панель мониторинга Kubernetes не открывается, скопируйте и вставьте URL-адрес, указанный в Azure CLI (обычно `http://127.0.0.1:8001`).

> [!NOTE]
> Если панель мониторинга не отображается, `http://127.0.0.1:8001` можно вручную направить на следующие адреса. Кластеры на 1,16 или более поздней версии используют HTTPS и должны иметь отдельную конечную точку.
> * K8s 1,16 или более поздней версии: `http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy`
> * K8s 1,15 и ниже: `http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy`

<!--
![The login page of the Kubernetes web dashboard](./media/kubernetes-dashboard/dashboard-login.png)

You have the following options to sign in to your cluster's dashboard:

* A [kubeconfig file][kubeconfig-file]. You can generate a kubeconfig file using [az aks get-credentials][az-aks-get-credentials].
* A token, such as a [service account token][aks-service-accounts] or user token. On [AAD-enabled clusters][aad-cluster], this token would be an AAD token. You can use `kubectl config view` to list the tokens in your kubeconfig file. For more details on creating an AAD token for use with an AKS cluster see [Integrate Azure Active Directory with Azure Kubernetes Service using the Azure CLI][aad-cluster].
* The default dashboard service account, which is used if you click *Skip*.

> [!WARNING]
> Never expose the Kubernetes dashboard publicly, regardless of the authentication method used.
> 
> When setting up authentication for the Kubernetes dashboard, it is recommended that you use a token over the default dashboard service account. A token allows each user to use their own permissions. Using the default dashboard service account may allow a user to bypass their own permissions and use the service account instead.
> 
> If you do choose to use the default dashboard service account and your AKS cluster uses Kubernetes RBAC, a *ClusterRoleBinding* must be created before you can correctly access the dashboard. By default, the Kubernetes dashboard is deployed with minimal read access and displays Kubernetes RBAC access errors. A cluster administrator can choose to grant additional access to the *kubernetes-dashboard* service account, however this can be a vector for privilege escalation. You can also integrate Azure Active Directory authentication to provide a more granular level of access.
>
> To create a binding, use the [kubectl create clusterrolebinding][kubectl-create-clusterrolebinding] command as shown in the following example. **This sample binding does not apply any additional authentication components and may lead to insecure use.**
>
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```
> 
> You can now access the Kubernetes dashboard in your Kubernetes RBAC-enabled cluster. To start the Kubernetes dashboard, use the [az aks browse][az-aks-browse] command as detailed in the previous step.
>
> If your cluster does not use Kubernetes RBAC, it is not recommended to create a *ClusterRoleBinding*.
> 
> For more information on using the different authentication methods, see the Kubernetes dashboard wiki on [access controls][dashboard-authentication].

After you choose a method to sign in, the Kubernetes dashboard is displayed. If you chose to use *token* or *skip*, the Kubernetes dashboard will use the permissions of the currently logged in user to access the cluster.

> [!IMPORTANT]
> If your AKS cluster uses Kubernetes RBAC, a *ClusterRoleBinding* must be created before you can correctly access the dashboard. By default, the Kubernetes dashboard is deployed with minimal read access and displays Kubernetes RBAC access errors. The Kubernetes dashboard does not currently support user-provided credentials to determine the level of access, rather it uses the roles granted to the service account. A cluster administrator can choose to grant additional access to the *kubernetes-dashboard* service account, however this can be a vector for privilege escalation. You can also integrate Azure Active Directory authentication to provide a more granular level of access.
> 
> To create a binding, use the [kubectl create clusterrolebinding][kubectl-create-clusterrolebinding] command. The following example shows how to create a sample binding, however, this sample binding does not apply any additional authentication components and may lead to insecure use. The Kubernetes dashboard is open to anyone with access to the URL. Do not expose the Kubernetes dashboard publicly.
>
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```
> 
> For more information on using the different authentication methods, see the Kubernetes dashboard wiki on [access controls][dashboard-authentication].
-->

## <a name="sign-in-to-the-dashboard-kubernetes-116"></a>Вход на панель мониторинга (kubernetes 1.16 +)

> [!IMPORTANT]
> Начиная с [v 1.10.1 панели мониторинга Kubernetes](https://github.com/kubernetes/dashboard/releases/tag/v1.10.1) или Kubernetes v 1.16 + учетная запись службы "Kubernetes-Dashboard" больше не может использоваться для получения ресурсов из-за [исправления безопасности в этом выпуске](https://github.com/kubernetes/dashboard/pull/3400). В результате запросы без сведений о проверке подлинности возвращают [ошибку 401](https://github.com/Azure/AKS/issues/1573#issuecomment-703040998). Токен носителя, полученный из учетной записи службы, по-прежнему можно использовать как в этом [примере панели мониторинга Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui), но это влияет на поток входа надстройки панели мониторинга по сравнению с более старыми версиями.
>
>Если вы по-прежнему запускаете версию до 1,16, вы по-прежнему можете предоставить разрешения учетной записи службы kubernetes-Dashboard, но это **не рекомендуется**:
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```

На начальном экране требуется kubeconfig или токен. Для обоих параметров требуются разрешения на доступ к ресурсам для показа этих ресурсов на панели мониторинга.

![экран входа](./media/kubernetes-dashboard/login.png)

**Использование kubeconfig**

Как для Azure AD, так и для кластеров, не использующих Azure AD, можно передать kubeconfig. Убедитесь, что маркеры доступа допустимы, если срок действия маркеров истек, вы можете обновить маркеры через kubectl.

1. Задайте для kubeconfig администратора значение `az aks get-credentials -a --resource-group <RG_NAME> --name <CLUSTER_NAME>`
1. Выберите `Kubeconfig` и щелкните, `Choose kubeconfig file` чтобы открыть средство выбора файлов
1. Выберите файл kubeconfig (по умолчанию $HOME/.KUBE/config.)
1. Щелкните `Sign In`.

**Использование маркера**

1. Для **кластера, не поддерживающего Azure AD**, запустите `kubectl config view` и скопируйте маркер, связанный с учетной записью пользователя кластера.
1. Вставьте в параметр token при входе.    
1. Щелкните `Sign In`.

Для кластеров с поддержкой Azure AD извлеките токен AAD с помощью следующей команды. Проверьте, что вы заменили имя группы ресурсов и кластера в команде.

```
## Update <RESOURCE_GROUP and <AKS_NAME> with your input.

kubectl config view -o jsonpath='{.users[?(@.name == "clusterUser_<RESOURCE GROUP>_<AKS_NAME>")].user.auth-provider.config.access-token}'
```

После успешного выполнения отобразится страница, аналогичная приведенной ниже.

![Страница "Обзор" веб-панели мониторинга Kubernetes](./media/kubernetes-dashboard/dashboard-overview.png)

## <a name="create-an-application"></a>Создание приложения

Для выполнения следующих действий необходимо, чтобы пользователь имел разрешения для соответствующих ресурсов. 

Давайте создадим приложение, чтобы увидеть, как панель мониторинга Kubernetes позволяет упростить задачи управления. Приложение можно создать на панели мониторинга Kubernetes, предоставив текстовые входные данные (файл YAML), или через мастер с графическим интерфейсом.

Чтобы создать приложение, выполните указанные ниже действия:

1. Нажмите кнопку **Создать** в правом верхнем углу окна.
1. Чтобы использовать мастер с графическим интерфейсом, выберите **Create an app** (Создать приложение).
1. Укажите имя для развертывания, например *nginx*.
1. Введите имя, которое будет использоваться для образа контейнера, например *nginx:1.15.5*.
1. Чтобы настроить порт 80 для веб-трафика, создайте службу Kubernetes. В разделе **Служба** выберите **External** (Внешняя), а затем введите **80** для порта и целевого порта.
1. Когда все будет готово, нажмите кнопку **Развернуть**, чтобы создать приложение.

![Развертывание приложения на веб-панели мониторинга Kubernetes](./media/kubernetes-dashboard/create-app.png)

Назначение общедоступного внешнего IP-адреса службе Kubernetes занимает пару минут. Слева в разделе **Discovery and Load Balancing** (Обнаружение и балансировка нагрузки) выберите **Службы**. Вы увидите службу приложения, а также *внешние конечные точки*, как показано в следующем примере:

![Просмотр списка служб и конечных точек](./media/kubernetes-dashboard/view-services.png)

Выберите адрес конечной точки, чтобы открыть окно веб-браузера на странице NGINX по умолчанию:

![Просмотр страницы NGINX по умолчанию развернутого приложения](./media/kubernetes-dashboard/default-nginx.png)

## <a name="view-pod-information"></a>Просмотр сведений о pod

На панели мониторинга Kubernetes могут содержаться основные метрики мониторинга и сведения об устранении неполадок, в частности журналы.

Чтобы просмотреть дополнительные сведения о группах pod приложения, выберите **Pods** (Группы pod) в меню слева. Отобразится список доступных pod. Выберите свой pod *nginx*, чтобы просмотреть такие сведения, как потребление ресурсов:

![Просмотр сведений о pod](./media/kubernetes-dashboard/view-pod-info.png)

## <a name="edit-the-application"></a>Изменение приложения

Помимо создания и просмотра приложений панель мониторинга Kubernetes можно использовать для изменения и обновления развертываний приложений. Давайте увеличим число реплик NGINX, чтобы обеспечить дополнительную избыточность для приложения.

Чтобы изменить развертывание:

1. Щелкните **Развертывания** в меню слева, а затем выберите свое развертывание *nginx*.
1. Выберите **Изменить** на верхней правой панели навигации.
1. Найдите значение `spec.replica` в строке 20. Чтобы увеличить количество реплик для приложения, измените это значение с *1* на *3*.
1. Выберите **Обновить** по окончании.

![Изменение развертывания для обновления количества реплик](./media/kubernetes-dashboard/edit-deployment.png)

Создание новых pod в наборе реплик занимает несколько секунд. В меню слева выберите **Replica Sets** (Наборы реплик), а затем выберите набор реплик *nginx*. Список pod теперь отражает обновленное число реплик, как показано в следующем примере выходных данных:

![Просмотр сведений о наборе реплик](./media/kubernetes-dashboard/view-replica-set.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о веб-панели мониторинга Kubernetes см. в [этой статье][kubernetes-dashboard].

<!-- LINKS - external -->
[dashboard-authentication]: https://github.com/kubernetes/dashboard/wiki/Access-control
[kubeconfig-file]: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
[kubectl-create-clusterrolebinding]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-clusterrolebinding-em-
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubernetes-dashboard]: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

<!-- LINKS - internal -->
[aad-cluster]: ./azure-ad-integration-cli.md
[aks-quickstart]: ./kubernetes-walkthrough.md
[aks-service-accounts]: ./concepts-identity.md#kubernetes-service-accounts
[az-account-get-access-token]: /cli/azure/account#az-account-get-access-token
[az-aks-browse]: /cli/azure/aks#az-aks-browse
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[install-azure-cli]: /cli/azure/install-azure-cli
[kubernetes-portal]: ./kubernetes-portal.md

---
title: Установка существующих приложений с помощью Helm в AKS
description: Узнайте, как использовать средство упаковки Helm для развертывания контейнеров в кластере службы Kubernetes Azure (AKS).
services: container-service
author: zr-msft
ms.topic: article
ms.date: 12/07/2020
ms.author: zarhoads
ms.openlocfilehash: f12dffe0b538738a8f6dd00cd3d87d44da828f21
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96779173"
---
# <a name="install-existing-applications-with-helm-in-azure-kubernetes-service-aks"></a>Установка существующих приложений с помощью Helm в службе Kubernetes Azure (AKS)

[Helm][helm] — это средство упаковки с открытым исходным кодом, помогающее в установке и управлении жизненным циклом приложений Kubernetes. Аналогично диспетчерам пакетов Linux, таким как *APT* и *Yum*, Helm используется для управления диаграммами Kubernetes, которые представляют собой пакеты предварительно настроенных ресурсов Kubernetes.

В этой статье показано, как настроить и использовать Helm в кластере Kubernetes в AKS.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Также необходимо установить Helm CLI, который является клиентом, работающим в системе разработки. Она позволяет запускать, прекращать работу приложений и управлять ими с помощью Helm. Если вы используете Azure Cloud Shell, интерфейс командной строки Helm уже установлен. Инструкции по установке на локальной платформе см. в разделе [Installing Helm][helm-install].

> [!IMPORTANT]
> Helm предназначен для работы на узлах Linux. Если в кластере есть узлы Windows Server, необходимо убедиться, что Helm Pod планируется запускать только на узлах Linux. Также необходимо убедиться, что все устанавливаемые Helm диаграммы запланированы на правильных узлах. В командах, приведенных в этой статье, используются [селекторы узлов] [K8S-node-Selector], чтобы убедиться, что модули Pod запланированы на правильные узлы, но не все Helm диаграммы могут предоставлять селектор узла. Можно также использовать другие параметры в кластере, например [таинтс][taints].

## <a name="verify-your-version-of-helm"></a>Проверка версии Helm

Используйте `helm version` команду, чтобы убедиться, что установлен Helm 3:

```console
helm version
```

В следующем примере показана установленная версия 3.0.0 Helm:

```console
$ helm version

version.BuildInfo{Version:"v3.0.0", GitCommit:"e29ce2a54e96cd02ccfce88bee4f58bb6e2a28b6", GitTreeState:"clean", GoVersion:"go1.13.4"}
```

## <a name="install-an-application-with-helm-v3"></a>Установка приложения с помощью Helm v3

### <a name="add-helm-repositories"></a>Добавление репозиториев Helm

Используйте команду [репозитория Helm][helm-repo-add] , чтобы добавить репозиторий *nginx* .

```console
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

### <a name="find-helm-charts"></a>Поиск чартов Helm

Чарты Helm используются для развертывания приложений в кластере Kubernetes. Чтобы найти предварительно созданные диаграммы Helm, используйте команду [поиска Helm][helm-search] :

```console
helm search repo ingress-nginx
```

В следующем кратком примере выходные данные содержат некоторые из доступных для использования чартов Helm:

```console
$ helm search repo ingress-nginx

NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
ingress-nginx/ingress-nginx     2.12.0          0.34.1          Ingress controller for Kubernetes using NGINX a...
```

Чтобы обновить список чартов, используйте команду [helm repo update][helm-repo-update].

```console
helm repo update
```

Ниже приведен пример успешного обновления репозитория:

```console
$ helm repo update

Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ingress-nginx" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

### <a name="run-helm-charts"></a>Выполнение чартов Helm

Чтобы установить диаграммы с помощью Helm, используйте команду [Helm Install][helm-install-command] и укажите имя выпуска и имя диаграммы для установки. Чтобы увидеть, как установить Helm диаграмму в действии, давайте установим базовое развертывание nginx с помощью диаграммы Helm.

```console
helm install my-nginx-ingress ingress-nginx/ingress-nginx \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

В следующем кратком примере выходные данные содержат состояние развертывания ресурсов Kubernetes, созданное чартом Helm:

```console
$ helm install my-nginx-ingress ingress-nginx/ingress-nginx \
>     --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
>     --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux

NAME: my-nginx-ingress
LAST DEPLOYED: Fri Nov 22 10:08:06 2019
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w my-nginx-ingress-ingress-nginx-controller'
...
```

Используйте `kubectl get services` команду, чтобы получить *внешний IP-адрес* службы.

```console
kubectl --namespace default get services -o wide -w my-nginx-ingress-ingress-nginx-controller
```

Например, в приведенной ниже команде показан *внешний IP-адрес* для службы *My-nginx-входящий-входящий-nginx-Controller* :

```console
$ kubectl --namespace default get services -o wide -w my-nginx-ingress-ingress-nginx-controller

NAME                                        TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)                      AGE   SELECTOR
my-nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.2.237   <EXTERNAL-IP>    80:31380/TCP,443:32239/TCP   72s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=my-nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

### <a name="list-releases"></a>Вывод списка выпусков

Чтобы просмотреть список выпусков, установленных в кластере, используйте `helm list` команду.

```console
helm list
```

В следующем примере показан выпуск *My-nginx-* входящий, развернутый на предыдущем шаге:

```console
$ helm list

NAME                NAMESPACE   REVISION    UPDATED                                 STATUS      CHART                   APP VERSION
my-nginx-ingress    default     1           2019-11-22 10:08:06.048477 -0600 CST    deployed    nginx-ingress-1.25.0    0.26.1 
```

### <a name="clean-up-resources"></a>Очистка ресурсов

При развертывании диаграммы Helm создается несколько ресурсов Kubernetes. К ним относятся элементы pod, развертывания и службы. Чтобы очистить эти ресурсы, используйте команду [удаления Helm][helm-cleanup] и укажите имя выпуска, как указано в предыдущей `helm list` команде.

```console
helm uninstall my-nginx-ingress
```

В следующем примере показан выпуск с именем *My-nginx-* входящий, который был удален:

```console
$ helm uninstall my-nginx-ingress

release "my-nginx-ingress" uninstalled
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управлении развертываниями приложений Kubernetes с помощью Helm см. в документации по Helm.

> [!div class="nextstepaction"]
> [Документация по Helm][helm-documentation]

<!-- LINKS - external -->
[helm]: https://github.com/kubernetes/helm/
[helm-cleanup]: https://helm.sh/docs/intro/using_helm/#helm-uninstall-uninstalling-a-release
[helm-documentation]: https://helm.sh/docs/
[helm-install]: https://helm.sh/docs/intro/install/
[helm-install-command]: https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package
[helm-repo-add]: https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
[helm-search]: https://helm.sh/docs/intro/using_helm/#helm-search-finding-charts
[helm-repo-update]: https://helm.sh/docs/intro/using_helm/#helm-repo-working-with-repositories
            
<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[taints]: operator-best-practices-advanced-scheduler.md

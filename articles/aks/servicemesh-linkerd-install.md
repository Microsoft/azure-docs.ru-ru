---
title: Установка Linkerd в службе Kubernetes Azure (AKS)
description: Узнайте, как установить и использовать Linkerd для создания сетки службы в кластере службы Kubernetes Azure (AKS).
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
zone_pivot_groups: client-operating-system
ms.openlocfilehash: d5b2ec220bbf14a79247ed2c78a0e84f4717fae3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94684242"
---
# <a name="install-linkerd-in-azure-kubernetes-service-aks"></a>Установка Linkerd в службе Kubernetes Azure (AKS)

[Linkerd][linkerd-github] — это сервисная сеть с открытым исходным кодом и [проект инкубация кнкф][linkerd-cncf]. Linkerd — это ултралигхт служба, которая предоставляет функции, включающие управление трафиком, идентификацию службы, безопасность, надежность и наблюдаемость. Дополнительные сведения о Linkerd см. в официальной документации по [Linkerd и часто задаваемым][linkerd-faq] вопросам по [архитектуре Linkerd][linkerd-architecture] .

В этой статье показано, как установить Linkerd. `linkerd`Двоичный файл клиента Linkerd устанавливается на клиентский компьютер, а компоненты Linkerd устанавливаются в кластер Kubernetes на AKS.

> [!NOTE]
> Эти инструкции ссылаются на версию Linkerd `stable-2.6.0` .
>
> Linkerd `stable-2.6.x` можно использовать в версиях Kubernetes `1.13+` . Дополнительные стабильные и граничные версии Linkerd можно найти в [выпусках GitHub-Linkerd][linkerd-github-releases].

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Скачивание и установка двоичного файла клиента Linkerd Linkerd
> * Установка Linkerd на AKS
> * Проверка установки Linkerd
> * Доступ к панели мониторинга
> * Удаление Linkerd из AKS

## <a name="before-you-begin"></a>Перед началом

В шагах, описанных в этой статье, предполагается, что вы создали кластер AKS (Kubernetes `1.13` и выше, с включенным KUBERNETES RBAC) и установили `kubectl` Подключение к кластеру. Дополнительные сведения см. в [кратком руководстве по AKS][aks-quickstart].

Все Linkerd Pod должны быть запланированы для запуска на узлах Linux. Эта настройка используется по умолчанию в описанном ниже методе установки и не требует дополнительной настройки.

В этой статье Руководство по установке Linkerd разделено на несколько отдельных этапов. Результат аналогичен в структуре в качестве официального [руководства][linkerd-getting-started]по началу работы с Linkerd.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download and install client binary](includes/servicemesh/linkerd/install-client-binary-linux.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [MacOS - download and install client binary](includes/servicemesh/linkerd/install-client-binary-macos.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download and install client binary](includes/servicemesh/linkerd/install-client-binary-windows.md)]

::: zone-end

## <a name="install-linkerd-on-aks"></a>Установка Linkerd на AKS

Перед установкой Linkerd мы выполним проверки перед установкой, чтобы определить, можно ли установить плоскость управления в нашем кластере AKS:

```console
linkerd check --pre
```

Чтобы указать, что кластер AKS является допустимым целевым объектом установки для Linkerd, вы должны увидеть примерно следующее:

```console
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version
√ is running the minimum kubectl version

pre-kubernetes-setup
--------------------
√ control plane namespace does not already exist
√ can create Namespaces
√ can create ClusterRoles
√ can create ClusterRoleBindings
√ can create CustomResourceDefinitions
√ can create PodSecurityPolicies
√ can create ServiceAccounts
√ can create Services
√ can create Deployments
√ can create CronJobs
√ can create ConfigMaps
√ no clock skew detected

pre-kubernetes-capability
-------------------------
√ has NET_ADMIN capability
√ has NET_RAW capability

pre-linkerd-global-resources
----------------------------
√ no ClusterRoles exist
√ no ClusterRoleBindings exist
√ no CustomResourceDefinitions exist
√ no MutatingWebhookConfigurations exist
√ no ValidatingWebhookConfigurations exist
√ no PodSecurityPolicies exist

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

Status check results are √
```

Теперь пора установить компоненты Linkerd. Используйте `linkerd` `kubectl` двоичные файлы и для установки компонентов Linkerd в кластер AKS. `linkerd`Пространство имен будет создано автоматически, и компоненты будут установлены в это пространство имен.

```console
linkerd install | kubectl apply -f -
```

Linkerd развертывает несколько объектов. Вы увидите список из выходных данных `linkerd install` команды выше. Для завершения развертывания компонентов Linkerd в зависимости от среды кластера потребуется около одной минуты.

На этом этапе вы развернули Linkerd в кластере AKS. Чтобы убедиться в успешном развертывании Linkerd, перейдем к следующему разделу, чтобы [проверить установку Linkerd](#validate-the-linkerd-installation).

## <a name="validate-the-linkerd-installation"></a>Проверка установки Linkerd

Убедитесь, что ресурсы были успешно созданы. Используйте команды [kubectl Get SVC][kubectl-get] и [kubectl Get Pod][kubectl-get] для запроса `linkerd` пространства имен, где компоненты Linkerd были установлены с помощью `linkerd install` команды:

```console
kubectl get svc --namespace linkerd --output wide
kubectl get pod --namespace linkerd --output wide
```

В следующем примере выходных данных показаны службы и модули Pod (запланированные на узлах Linux), которые теперь должны выполняться:

```console
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE  SELECTOR
linkerd-controller-api   ClusterIP   10.0.110.67    <none>        8085/TCP            66s  linkerd.io/control-plane-component=controller
linkerd-destination      ClusterIP   10.0.224.29    <none>        8086/TCP            66s  linkerd.io/control-plane-component=controller
linkerd-dst              ClusterIP   10.0.225.148   <none>        8086/TCP            66s  linkerd.io/control-plane-component=destination
linkerd-grafana          ClusterIP   10.0.61.124    <none>        3000/TCP            65s  linkerd.io/control-plane-component=grafana
linkerd-identity         ClusterIP   10.0.6.104     <none>        8080/TCP            67s  linkerd.io/control-plane-component=identity
linkerd-prometheus       ClusterIP   10.0.27.168    <none>        9090/TCP            65s  linkerd.io/control-plane-component=prometheus
linkerd-proxy-injector   ClusterIP   10.0.100.133   <none>        443/TCP             64s  linkerd.io/control-plane-component=proxy-injector
linkerd-sp-validator     ClusterIP   10.0.221.5     <none>        443/TCP             64s  linkerd.io/control-plane-component=sp-validator
linkerd-tap              ClusterIP   10.0.18.14     <none>        8088/TCP,443/TCP    64s  linkerd.io/control-plane-component=tap
linkerd-web              ClusterIP   10.0.37.108    <none>        8084/TCP,9994/TCP   66s  linkerd.io/control-plane-component=web

NAME                                      READY   STATUS    RESTARTS   AGE   IP            NODE                            NOMINATED NODE   READINESS GATES
linkerd-controller-66ddc9f94f-cm9kt       3/3     Running   0          66s   10.240.0.50   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-destination-c94bc454-qpkng        2/2     Running   0          66s   10.240.0.78   aks-linux-16165125-vmss000002   <none>           <none>
linkerd-grafana-6868fdcb66-4cmq2          2/2     Running   0          65s   10.240.0.69   aks-linux-16165125-vmss000002   <none>           <none>
linkerd-identity-74d8df4b85-tqq8f         2/2     Running   0          66s   10.240.0.48   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-prometheus-699587cf8-k8ghg        2/2     Running   0          65s   10.240.0.41   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-proxy-injector-6556447f64-n29wr   2/2     Running   0          64s   10.240.0.32   aks-linux-16165125-vmss000000   <none>           <none>
linkerd-sp-validator-56745cd567-v4x7h     2/2     Running   0          64s   10.240.0.6    aks-linux-16165125-vmss000000   <none>           <none>
linkerd-tap-5cd9fc566-ct988               2/2     Running   0          64s   10.240.0.15   aks-linux-16165125-vmss000000   <none>           <none>
linkerd-web-774c79b6d5-dhhwf              2/2     Running   0          65s   10.240.0.70   aks-linux-16165125-vmss000002   <none>           <none>
```

Linkerd предоставляет команду через `linkerd` клиентский двоичный файл, чтобы проверить успешность установки и настройки плоскости управления Linkerd.

```console
linkerd check
```

Чтобы указать, что установка прошла успешно, вы увидите нечто вроде следующего:

```console
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version
√ is running the minimum kubectl version

linkerd-config
--------------
√ control plane Namespace exists
√ control plane ClusterRoles exist
√ control plane ClusterRoleBindings exist
√ control plane ServiceAccounts exist
√ control plane CustomResourceDefinitions exist
√ control plane MutatingWebhookConfigurations exist
√ control plane ValidatingWebhookConfigurations exist
√ control plane PodSecurityPolicies exist

linkerd-existence
-----------------
√ 'linkerd-config' config map exists
√ heartbeat ServiceAccount exist
√ control plane replica sets are ready
√ no unschedulable pods
√ controller pod is running
√ can initialize the client
√ can query the control plane API

linkerd-api
-----------
√ control plane pods are ready
√ control plane self-check
√ [kubernetes] control plane can talk to Kubernetes
√ [prometheus] control plane can talk to Prometheus
√ no invalid service profiles

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

control-plane-version
---------------------
√ control plane is up-to-date
√ control plane and cli versions match

Status check results are √
```

## <a name="access-the-dashboard"></a>Доступ к панели мониторинга

Linkerd поставляется с панелью мониторинга, которая предоставляет представление о сети и рабочих нагрузках службы. Чтобы получить доступ к панели мониторинга, используйте `linkerd dashboard` команду. Эта команда использует [kubectl порт-Forward][kubectl-port-forward] для создания безопасного подключения между клиентским компьютером и соответствующими модулями Pod в кластере AKS. Затем панель мониторинга будет автоматически открыта в браузере по умолчанию.

```console
linkerd dashboard
```

Команда также создает переадресацию порта и возвращает ссылку на панели мониторинга Grafana.

```console
Linkerd dashboard available at:
http://127.0.0.1:50750
Grafana dashboard available at:
http://127.0.0.1:50750/grafana
Opening Linkerd dashboard in the default browser
```

## <a name="uninstall-linkerd-from-aks"></a>Удаление Linkerd из AKS

> [!WARNING]
> Удаление Linkerd из работающей системы может привести к проблемам с трафиком между Вашими службами. Перед продолжением убедитесь, что вы выполнили предварительные условия для правильной работы системы без Linkerd.

Сначала необходимо удалить прокси-серверы плоскости данных. Удалите все автоматические [аннотации][linkerd-automatic-proxy-injection] введения прокси-сервера из пространств имен рабочей нагрузки и развертывание рабочих нагрузок. У рабочих нагрузок больше нет связанных компонентов плоскости данных.

Наконец, удалите плоскость управления, как показано ниже.

```console
linkerd install --ignore-cluster | kubectl delete -f -
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об установке и параметрах конфигурации для Linkerd см. в следующих официальных руководствах по Linkerd:

- [Linkerd — установка Helm][linkerd-install-with-helm]
- [Linkerd. Установка в нескольких стадиях для разрешения ролей][linkerd-multi-stage-installation]

Кроме того, можно выполнять дополнительные сценарии, используя:

- [Демонстрация Linkerd еможивото][linkerd-demo-emojivoto]
- [Демонстрация Linkerd Books][linkerd-demo-books]

<!-- LINKS - external -->

[linkerd]: https://linkerd.io/
[linkerd-cncf]: https://landscape.cncf.io/selected=linkerd
[linkerd-faq]: https://linkerd.io/2/faq/
[linkerd-architecture]: https://linkerd.io/2/reference/architecture/
[linkerd-getting-started]: https://linkerd.io/2/getting-started/
[linkerd-overview]: https://linkerd.io/2/overview/
[linkerd-github]: https://github.com/linkerd/linkerd2
[linkerd-github-releases]: https://github.com/linkerd/linkerd2/releases/

[linkerd-install-with-helm]: https://linkerd.io/2/tasks/install-helm/
[linkerd-multi-stage-installation]: https://linkerd.io/2/tasks/install/#multi-stage-install
[linkerd-automatic-proxy-injection]: https://linkerd.io/2/features/proxy-injection/

[linkerd-demo-emojivoto]: https://linkerd.io/2/getting-started/#step-5-install-the-demo-app
[linkerd-demo-books]: https://linkerd.io/2/tasks/books/

[helm]: https://helm.sh

[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md

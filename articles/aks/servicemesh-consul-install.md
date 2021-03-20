---
title: Установка Consul в Службе Azure Kubernetes (AKS)
description: Узнайте, как установить и использовать Consul для создания сетки службы в кластере службы Kubernetes Azure (AKS).
author: dstrebel
ms.topic: article
ms.date: 10/09/2019
ms.author: dastrebe
zone_pivot_groups: client-operating-system
ms.openlocfilehash: 7c5ad53c0040009e9ed1f28072540b46ce7b0b9a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94683925"
---
# <a name="install-and-use-consul-in-azure-kubernetes-service-aks"></a>Установка и использование Consul в службе Kubernetes Azure (AKS)

[Consul][consul-github] — это сетка служб с открытым исходным кодом, которая предоставляет набор ключевых функций для микрослужб в кластере Kubernetes. Эти функции включают обнаружение служб, проверку работоспособности, сегментацию службы и наблюдаемость. Дополнительные сведения о Consul см. в официальной документации о [Consul?][consul-docs-concepts] .

В этой статье показано, как установить Consul. Компоненты Consul устанавливаются в кластер Kubernetes на AKS.

> [!NOTE]
> Эти инструкции ссылаются на версию Consul `1.6.0` и используют по меньшей мере Helm версию `2.14.2` .
>
> `1.6.x`Выпуски Consul могут выполняться для версий Kubernetes `1.13+` . Дополнительные версии Consul можно найти в [выпусках GitHub-Consul][consul-github-releases] , а сведения о каждой из выпусков см. в [заметках о выпуске Consul][consul-release-notes].

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Установка компонентов Consul на AKS
> * Проверка установки Consul
> * Удаление Consul из AKS

## <a name="before-you-begin"></a>Перед началом

В шагах, описанных в этой статье, предполагается, что вы создали кластер AKS (Kubernetes `1.13` и выше, с включенным KUBERNETES RBAC) и установили `kubectl` Подключение к кластеру. Дополнительные сведения см. в [кратком руководстве по AKS][aks-quickstart]. Убедитесь, что кластер содержит по крайней мере 3 узла в пуле узлов Linux.

Вам потребуется [Helm][helm] , чтобы выполнить эти инструкции и установить Consul. Рекомендуется правильно установить и настроить последнюю стабильную версию в кластере. Если вам нужна помощь с установкой Helm, см. [руководство по установке AKS Helm][helm-install]. Все Consul Pod также должны быть запланированы для запуска на узлах Linux.

В этой статье Руководство по установке Consul разделено на несколько отдельных этапов. Конечный результат аналогичен в структуре в качестве официального [руководства][consul-install-k8]по установке Consul.

### <a name="install-the-consul-components-on-aks"></a>Установка компонентов Consul на AKS

Начнем с загрузки версии `v0.10.0` диаграммы Consul Helm. Эта версия диаграммы включает версию Consul `1.6.0` .

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download](includes/servicemesh/consul/download-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [macOS - download](includes/servicemesh/consul/download-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download](includes/servicemesh/consul/download-powershell.md)]

::: zone-end

Используйте Helm и скачанную `consul-helm` диаграмму для установки компонентов Consul в `consul` пространство имен в кластере AKS. 

> [!NOTE]
> **Варианты установки**
> 
> В процессе установки мы используем следующие варианты:
> - `connectInject.enabled=true` — разрешить внедрение прокси-серверов в модули Pod
> - `client.enabled=true` — Разрешить клиентам Consul выполняться на каждом узле;
> - `client.grpc=true` — включить прослушиватель gRPC для Коннектинжект
> - `syncCatalog.enabled=true` -Sync Kubernetes and Consul Services
>
> **Селекторы узлов**
>
> Consul в настоящее время необходимо запланировать запуск на узлах Linux. Если в кластере есть узлы Windows Server, необходимо убедиться, что Consul Pods планируется запускать только на узлах Linux. Мы будем использовать [селекторы узлов][kubernetes-node-selectors] , чтобы убедиться, что для модулей Pod запланированы правильные узлы.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Bash - install Istio components](includes/servicemesh/consul/install-components-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [Bash - install Istio components](includes/servicemesh/consul/install-components-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [PowerShell - install Istio components](includes/servicemesh/consul/install-components-powershell.md)]

::: zone-end

`Consul`Helm диаграмма развертывает несколько объектов. Список можно просмотреть на основе выходных данных `helm install` команды выше. Для завершения развертывания компонентов Consul в зависимости от среды кластера может потребоваться около 3 минут.

На этом этапе вы развернули Consul в кластере AKS. Чтобы убедиться в успешном развертывании Consul, перейдем к следующему разделу, чтобы проверить установку Consul.

## <a name="validate-the-consul-installation"></a>Проверка установки Consul

Убедитесь, что ресурсы были успешно созданы. Используйте команды [kubectl Get SVC][kubectl-get] и [kubectl Get Pod][kubectl-get] для запроса `consul` пространства имен, где компоненты Consul были установлены с помощью `helm install` команды:

```console
kubectl get svc --namespace consul --output wide
kubectl get pod --namespace consul --output wide
```

В следующем примере выходных данных показаны службы и модули Pod (запланированные на узлах Linux), которые теперь должны выполняться:

```output
NAME                                 TYPE           CLUSTER-IP    EXTERNAL-IP             PORT(S)                                                                   AGE     SELECTOR
consul                               ExternalName   <none>        consul.service.consul   <none>                                                                    38s     <none>
consul-consul-connect-injector-svc   ClusterIP      10.0.98.102   <none>                  443/TCP                                                                   3m26s   app=consul,component=connect-injector,release=consul
consul-consul-dns                    ClusterIP      10.0.46.194   <none>                  53/TCP,53/UDP                                                             3m26s   app=consul,hasDNS=true,release=consul
consul-consul-server                 ClusterIP      None          <none>                  8500/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP,8600/UDP   3m26s   app=consul,component=server,release=consul
consul-consul-ui                     ClusterIP      10.0.50.188   <none>                  80/TCP                                                                    3m26s   app=consul,component=server,release=consul

NAME                                                              READY   STATUS    RESTARTS   AGE    IP            NODE                            NOMINATED NODE   READINESS GATES
consul-consul-connect-injector-webhook-deployment-99f74fdbcr5zj   1/1     Running   0          3m9s   10.240.0.68   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-jbksc                                               1/1     Running   0          3m9s   10.240.0.44   aks-linux-92468653-vmss000001   <none>           <none>
consul-consul-jkwtq                                               1/1     Running   0          3m9s   10.240.0.70   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-server-0                                            1/1     Running   0          3m9s   10.240.0.91   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-server-1                                            1/1     Running   0          3m9s   10.240.0.38   aks-linux-92468653-vmss000001   <none>           <none>
consul-consul-server-2                                            1/1     Running   0          3m9s   10.240.0.10   aks-linux-92468653-vmss000000   <none>           <none>
consul-consul-sync-catalog-d846b79c-8ssr8                         1/1     Running   2          3m9s   10.240.0.94   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-tz2t5                                               1/1     Running   0          3m9s   10.240.0.12   aks-linux-92468653-vmss000000   <none>           <none>
```

Все модули Pod должны отображать состояние `Running` . Если группы pod имеют другое состояние, подождите пару минут, чтобы отобразилось нужное. Если для групп pod отображаются проблемы, выполните команду [describe pod][kubectl-describe], чтобы просмотреть выходные данные и состояние.

## <a name="accessing-the-consul-ui"></a>Доступ к пользовательскому интерфейсу Consul

Пользовательский интерфейс Consul был установлен в нашей настройке выше и предоставляет конфигурацию на основе пользовательского интерфейса для Consul. Пользовательский интерфейс для Consul недоступен через внешний IP-адрес. Чтобы получить доступ к пользовательскому интерфейсу Consul, используйте команду [перенаправления порта kubectl][kubectl-port-forward] . Эта команда создает безопасное подключение между вашим клиентским компьютером и соответствующим Pod в кластере AKS.

```console
kubectl port-forward -n consul svc/consul-consul-ui 8080:80
```

Теперь можно открыть браузер и указать, чтобы `http://localhost:8080/ui` открыть пользовательский интерфейс Consul. При открытии пользовательского интерфейса вы должны увидеть следующее:

![Пользовательский интерфейс Consul](./media/servicemesh/consul/consul-ui.png)

## <a name="uninstall-consul-from-aks"></a>Удаление Consul из AKS

> [!WARNING]
> Удаление Consul из работающей системы может привести к проблемам с трафиком между Вашими службами. Перед продолжением убедитесь, что вы выполнили предварительные условия для правильной работы системы без Consul.

### <a name="remove-consul-components-and-namespace"></a>Удалить компоненты и пространство имен Consul

Чтобы удалить Consul из кластера AKS, используйте следующие команды. Команда удалит `helm delete` `consul` диаграмму, и `kubectl delete namespace` команда удалит `consul` пространство имен.

```console
helm delete --purge consul
kubectl delete namespace consul
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об установке и параметрах конфигурации для Consul см. в следующих официальных статьях по Consul:

- [Consul — руководства по установке Helm][consul-install-k8]
- [Параметры установки Consul-Helm][consul-install-helm-options]

Кроме того, можно выполнять дополнительные сценарии, используя:

- [Пример приложения Consul][consul-app-example]
- [Эталонная архитектура Consul Kubernetes][consul-reference]
- [Consul сетчатые шлюзы][consul-mesh-gateways]

<!-- LINKS - external -->
[Hashicorp]: https://hashicorp.com
[cosul-github]: https://github.com/hashicorp/consul
[helm]: https://helm.sh

[consul-docs-concepts]: https://www.consul.io/docs/platform/k8s/index.html
[consul-github]: https://github.com/hashicorp/consul
[consul-github-releases]: https://github.com/hashicorp/consul/releases
[consul-release-notes]: https://github.com/hashicorp/consul/blob/master/CHANGELOG.md
[consul-install-download]: https://www.consul.io/downloads.html
[consul-install-k8]: https://learn.hashicorp.com/consul/kubernetes/kubernetes-deployment-guide
[consul-install-helm-options]: https://www.consul.io/docs/platform/k8s/helm.html#configuration-values-
[consul-mesh-gateways]: https://learn.hashicorp.com/consul/kubernetes/mesh-gateways
[consul-reference]: https://learn.hashicorp.com/consul/kubernetes/kubernetes-reference
[consul-app-example]: https://learn.hashicorp.com/consul?track=gs-consul-service-mesh#gs-consul-service-mesh
[install-wsl]: /windows/wsl/install-win10

[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward
[kubernetes-node-selectors]: ./concepts-clusters-workloads.md#node-selectors

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
[consul-scenario-mtls]: ./consul-mtls.md
[helm-install]: ./kubernetes-helm.md

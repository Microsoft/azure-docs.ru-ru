---
title: Установка Istio в Службе Azure Kubernetes (AKS)
description: Сведения об установке и применении Istio для создания сетки службы в кластере Службы Azure Kubernetes (AKS)
author: paulbouwer
ms.topic: article
ms.date: 10/02/2020
ms.author: pabouwer
zone_pivot_groups: client-operating-system
ms.openlocfilehash: c42e9c31397e9313898d7029366bc8de169d368e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94683834"
---
# <a name="install-and-use-istio-in-azure-kubernetes-service-aks"></a>Установка и использование Istio в Службе Azure Kubernetes (AKS)

Сетка [Istio][istio-github] — это решение с открытым исходным кодом, которое предоставляет набор ключевых функций для микрослужб в кластере Kubernetes, включая управление трафиком, удостоверения служб и защиту служб, а также применение политик и возможность мониторинга. См. дополнительные сведения об [Istio][istio-docs-concepts].

В этой статье описано, как установить Istio. `istioctl`Двоичный файл клиента Istio устанавливается на клиентский компьютер, а компоненты Istio устанавливаются в кластер Kubernetes на AKS.

> [!NOTE]
> Приведенные ниже инструкции ссылаются на версию Istio `1.7.3` .
>
> `1.7.x`Выпуски Istio были протестированы группой Istio по сравнению с версией Kubernetes `1.16+` . Дополнительные версии Istio можно найти в разделе [GitHub-Istio][istio-github-releases], сведения о каждой из выпусков в [Istio News][istio-release-notes] и поддерживаемых версиях Kubernetes в [Istio общие вопросы и ответы][istio-faq].

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Скачивание и установка двоичного файла клиента Istio истиоктл
> * Установка Istio на AKS
> * Проверка установки Istio
> * Доступ к надстройкам
> * Удаление Istio из AKS

## <a name="before-you-begin"></a>Перед началом

В шагах, описанных в этой статье, предполагается, что вы создали кластер AKS (Kubernetes `1.16` и выше, с включенным KUBERNETES RBAC) и установили `kubectl` Подключение к кластеру. Дополнительные сведения см. в [кратком руководстве по AKS][aks-quickstart].

Убедитесь, что вы прочитали документацию по [производительности и масштабируемости Istio](https://istio.io/docs/concepts/performance-and-scalability/) , чтобы понять требования к дополнительным ресурсам для запуска Istio в кластере AKS. Требования к основному и оперативной памяти будут зависеть от конкретной рабочей нагрузки. Выберите подходящее количество узлов и размер виртуальной машины для настройки.

В этой статье инструкции по установке Istio разделены на несколько этапов. Результат установки будет идентичным результату согласно [официальной инструкции][istio-install-istioctl].

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download and install client binary](includes/servicemesh/istio/install-client-binary-linux.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [macOS - download and install client binary](includes/servicemesh/istio/install-client-binary-macos.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download and install client binary](includes/servicemesh/istio/install-client-binary-windows.md)]

::: zone-end

## <a name="install-the-istio-operator-on-aks"></a>Установка оператора Istio в AKS

Istio предоставляет [оператор][istio-install-operator] для управления установкой и обновлениями компонентов Istio в кластере AKS. Мы установим оператор Istio с помощью `istioctl` клиентского двоичного файла.

```bash
istioctl operator init
```

Вы увидите примерно следующий результат, чтобы убедиться, что оператор Istio установлен.

```console
Using operator Deployment image: docker.io/istio/operator:1.7.3
✔ Istio operator installed
✔ Installation complete
```

Оператор Istio устанавливается в `istio-operator` пространство имен. Запросите пространство имен.

```bash
kubectl get all -n istio-operator
```

Вы должны увидеть развернутые следующие компоненты.

```console
NAME                                  READY   STATUS    RESTARTS   AGE
pod/istio-operator-6d7958b7bf-wxgdc   1/1     Running   0          2m43s

NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/istio-operator   ClusterIP   10.0.8.57    <none>        8383/TCP   2m43s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-operator   1/1     1            1           2m43s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-operator-6d7958b7bf   1         1         1       2m43s
```

Вы можете узнать больше о шаблоне оператора и о том, как он может помочь автоматизировать сложные задачи с помощью [kubernetes.IO][kubernetes-operator].


### <a name="install-istio-components"></a>Установка компонентов Istio

Теперь, когда мы успешно установили оператор Istio в нашем кластере AKS, пришло время установить компоненты Istio. 

Мы будем использовать `default` [профиль конфигурации Istio][istio-configuration-profiles] для создания [спецификации оператора Istio][istio-control-plane].

Можно выполнить следующую команду, `istioctl` чтобы просмотреть конфигурацию для `default` профиля конфигурации Istio.

```bash
istioctl profile dump default
```

> [!NOTE]
> Istio в настоящее время необходимо запланировать запуск на узлах Linux. Если в кластере есть узлы Windows Server, необходимо убедиться, что Istio Pods планируется запускать только на узлах Linux. Мы будем использовать [селекторы узлов][kubernetes-node-selectors] , чтобы убедиться, что для модулей Pod запланированы правильные узлы.

> [!CAUTION]
> Функции [ISTIO CNI][istio-feature-cni] Istio в настоящее время относятся к [альфа-каналу][istio-feature-stages], поэтому перед включением их следует придумать. 

Создайте файл с именем `istio.aks.yaml` со следующим содержимым. Этот файл будет содержать [спецификацию оператора Istio][istio-control-plane] для настройки Istio.

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-control-plane
spec:
  # Use the default profile as the base
  # More details at: https://istio.io/docs/setup/additional-setup/config-profiles/
  profile: default
  # Enable the addons that we will want to use
  addonComponents:
    grafana:
      enabled: true
    prometheus:
      enabled: true
    tracing:
      enabled: true
    kiali:
      enabled: true
  values:
    global:
      # Ensure that the Istio pods are only scheduled to run on Linux nodes
      defaultNodeSelector:
        beta.kubernetes.io/os: linux
    kiali:
      dashboard:
        auth:
          strategy: anonymous 
```

Создайте `istio-system` пространство имен и разверните спецификацию оператора Istio в этом пространстве имен. Оператор Istio будет отслеживать спецификацию оператора Istio и будет использовать его для установки и настройки Istio в кластере AKS.

```bash
kubectl create ns istio-system

kubectl apply -f istio.aks.yaml 
```

На этом этапе вы развернули Istio в кластере AKS. Чтобы убедиться в успешном развертывании Istio, перейдем к следующему разделу, чтобы [проверить установку Istio](#validate-the-istio-installation).

## <a name="validate-the-istio-installation"></a>Проверка установки Istio

Запросите `istio-system` пространство имен, где компоненты Istio и Add-On были установлены оператором Istio:

```bash
kubectl get all -n istio-system
```

Вы должны увидеть следующие компоненты:

- `istio*` — компоненты Istio
- `jaeger-*`надстройка, `tracing` и `zipkin` -Трассировка
- `prometheus` — Надстройка метрик
- `grafana` — Надстройка "аналитика и мониторинг"
- `kiali` — Надстройка панели мониторинга для сетки службы

```console
NAME                                        READY   STATUS    RESTARTS   AGE
pod/grafana-7cf9794c74-mpfbp                1/1     Running   0          5m53s
pod/istio-ingressgateway-86b5dbdcb9-ndrp5   1/1     Running   0          5m57s
pod/istio-tracing-c98f4b8fc-zqklg           1/1     Running   0          82s
pod/istiod-6965c56995-4ph9h                 1/1     Running   0          6m15s
pod/kiali-7b44985d68-p87zh                  1/1     Running   0          81s
pod/prometheus-6868989549-5ghzz             1/1     Running   0          81s

NAME                                TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                                                      AGE
service/grafana                     ClusterIP      10.0.226.39    <none>         3000/TCP                                                     5m54s
service/istio-ingressgateway        LoadBalancer   10.0.143.56    20.53.72.254   15021:32166/TCP,80:31684/TCP,443:31302/TCP,15443:30863/TCP   5m57s
service/istiod                      ClusterIP      10.0.211.228   <none>         15010/TCP,15012/TCP,443/TCP,15014/TCP,853/TCP                6m16s
service/jaeger-agent                ClusterIP      None           <none>         5775/UDP,6831/UDP,6832/UDP                                   82s
service/jaeger-collector            ClusterIP      10.0.7.62      <none>         14267/TCP,14268/TCP,14250/TCP                                82s
service/jaeger-collector-headless   ClusterIP      None           <none>         14250/TCP                                                    82s
service/jaeger-query                ClusterIP      10.0.52.172    <none>         16686/TCP                                                    82s
service/kiali                       ClusterIP      10.0.71.179    <none>         20001/TCP                                                    82s
service/prometheus                  ClusterIP      10.0.171.151   <none>         9090/TCP                                                     82s
service/tracing                     ClusterIP      10.0.195.137   <none>         80/TCP                                                       82s
service/zipkin                      ClusterIP      10.0.136.111   <none>         9411/TCP                                                     82s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana                1/1     1            1           5m54s
deployment.apps/istio-ingressgateway   1/1     1            1           5m58s
deployment.apps/istio-tracing          1/1     1            1           83s
deployment.apps/istiod                 1/1     1            1           6m16s
deployment.apps/kiali                  1/1     1            1           83s
deployment.apps/prometheus             1/1     1            1           82s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-7cf9794c74                1         1         1       5m54s
replicaset.apps/istio-ingressgateway-86b5dbdcb9   1         1         1       5m58s
replicaset.apps/istio-tracing-c98f4b8fc           1         1         1       83s
replicaset.apps/istiod-6965c56995                 1         1         1       6m16s
replicaset.apps/kiali-7b44985d68                  1         1         1       82s
replicaset.apps/prometheus-6868989549             1         1         1       82s

NAME                                                       REFERENCE                         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   7%/80%    1         5         1          5m57s
horizontalpodautoscaler.autoscaling/istiod                 Deployment/istiod                 1%/80%    1         5         1          6m16s
```

Вы также можете получить дополнительные сведения об установке, просмотрев журналы для оператора Istio.

```bash
kubectl logs -n istio-operator -l name=istio-operator -f
```

Если `istio-ingressgateway` отображает значение `<pending>` в поле внешнего IP-адреса, подождите несколько минут, пока сеть Azure не назначит новый IP-адрес.

Все модули Pod должны отображать состояние `Running` . Если группы pod имеют другое состояние, подождите пару минут, чтобы отобразилось нужное. Если для групп pod отображаются проблемы, выполните команду [describe pod][kubectl-describe], чтобы просмотреть выходные данные и состояние.

## <a name="accessing-the-add-ons"></a>Доступ к надстройкам

Несколько надстроек были установлены оператором Istio, обеспечивающим дополнительные функциональные возможности. Веб-приложения для надстроек **не** предоставляются публично через внешний IP-адрес. 

Чтобы получить доступ к пользовательским интерфейсам надстройки, используйте `istioctl dashboard` команду. Эта команда использует [kubectl порта перенаправления][kubectl-port-forward] и случайный порт для создания безопасного подключения между клиентским компьютером и соответствующим модулем Pod в кластере AKS. После этого будет автоматически открыто веб-приложение надстройки в браузере по умолчанию.

### <a name="grafana"></a>Grafana

Информационные панели аналитики и мониторинга для Istio предоставляются [Grafana][grafana]. При появлении запроса не забудьте использовать учетные данные, созданные с помощью секрета Grafana ранее. Безопасно откройте панель мониторинга Grafana следующим образом:

```console
istioctl dashboard grafana
```

### <a name="prometheus"></a>Prometheus

Метрики для Istio предоставляются [Prometheus][prometheus]. Безопасно откройте панель мониторинга Prometheus следующим образом:

```console
istioctl dashboard prometheus
```

### <a name="jaeger"></a>Jaeger

Трассировка в Istio предоставляется [жаежер][jaeger]. Безопасно откройте панель мониторинга Жаежер следующим образом:

```console
istioctl dashboard jaeger
```

### <a name="kiali"></a>Kiali

Панель мониторинга для сетки служб предоставляется [Kiali][kiali]. При появлении запроса не забудьте использовать учетные данные, созданные с помощью секрета Киали ранее. Безопасно откройте панель мониторинга Киали следующим образом:

```console
istioctl dashboard kiali
```

### <a name="envoy"></a>Envoy

Доступен простой интерфейс для прокси-серверов [делегата][envoy] . Он предоставляет сведения о конфигурации и метрики для прокси-сервера делегата, работающего в указанном Pod. Безопасно Откройте интерфейс делегата следующим образом:

```console
istioctl dashboard envoy <pod-name>.<namespace>
```

## <a name="uninstall-istio-from-aks"></a>Удаление Istio из AKS

> [!WARNING]
> Удаление Istio из работающей системы может привести к проблемам с трафиком между Вашими службами. Перед продолжением убедитесь, что вы выполнили предварительные условия для правильной работы системы без Istio.

### <a name="remove-istio"></a>Удалить Istio

Чтобы удалить Istio из кластера AKS, удалите `IstioOperator` ресурс с именем `istio-control-plane` , который мы добавили ранее. Оператор Istio распознает, что спецификация оператора Istio была удалена, а затем удалите все связанные компоненты Istio.

```bash
kubectl delete istiooperator istio-control-plane -n istio-system
```

Чтобы проверить, были ли удалены все компоненты Istio, выполните следующую команду:

```bash
kubectl get all -n istio-system
```

### <a name="remove-istio-operator"></a>Удалить оператор Istio

После успешного удаления Istio можно также удалить оператор Istio.

```bash
istioctl operator remove
```

И наконец, удалите `istio-` пространства имен.

```bash
kubectl delete ns istio-system
kubectl delete ns istio-operator
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об установке и параметрах конфигурации для Istio см. в следующих официальных руководствах по Istio:

- [Istio — руководства по установке][istio-installation-guides]

Кроме того, можно выполнять дополнительные сценарии, используя:

- [Пример приложения Istio BookInfo][istio-bookinfo-example]

<!-- LINKS - external -->
[istio]: https://istio.io
[helm]: https://helm.sh

[istio-faq]: https://istio.io/faq/general/
[istio-docs-concepts]: https://istio.io/docs/concepts/what-is-istio/
[istio-github]: https://github.com/istio/istio
[istio-github-releases]: https://github.com/istio/istio/releases
[istio-release-notes]: https://istio.io/news/
[istio-installation-guides]: https://istio.io/docs/setup/install/
[istio-install-download]: https://istio.io/docs/setup/kubernetes/download-release/
[istio-install-istioctl]: https://istio.io/docs/setup/install/istioctl/
[istio-install-operator]: https://istio.io/latest/docs/setup/install/operator/
[istio-configuration-profiles]: https://istio.io/docs/setup/additional-setup/config-profiles/
[istio-control-plane]: https://istio.io/docs/reference/config/istio.operator.v1alpha1/
[istio-bookinfo-example]: https://istio.io/docs/examples/bookinfo/

[istio-feature-stages]: https://istio.io/about/feature-stages/
[istio-feature-sds]: https://istio.io/docs/tasks/traffic-management/ingress/secure-ingress-sds/
[istio-feature-cni]: https://istio.io/docs/setup/additional-setup/cni/

[kubernetes-operator]: https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
[kubernetes-feature-sa-projected-volume]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-token-volume-projection
[kubernetes-crd]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions
[kubernetes-secrets]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-node-selectors]: ./concepts-clusters-workloads.md#node-selectors
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/
[jaeger]: https://www.jaegertracing.io/
[kiali]: https://www.kiali.io/
[envoy]: https://www.envoyproxy.io/

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md

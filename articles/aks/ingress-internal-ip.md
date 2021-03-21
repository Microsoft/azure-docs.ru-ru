---
title: Входной контроллер во внутренней сети
titleSuffix: Azure Kubernetes Service
description: Сведения об установке и настройке контроллера входящего трафика NGINX для внутренней частной сети в кластере Службы Azure Kubernetes (AKS).
services: container-service
ms.topic: article
ms.date: 03/16/2021
ms.openlocfilehash: 3201f510db9970b7db548ee6a3348fa68d278248
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104601471"
---
# <a name="create-an-ingress-controller-to-an-internal-virtual-network-in-azure-kubernetes-service-aks"></a>Создание контроллера входящего трафика для внутренней виртуальной сети в Службе Azure Kubernetes (AKS)

Контроллер входящего трафика является программным компонентом, предоставляющим для служб Kubernetes обратный прокси-сервер, настраиваемую маршрутизацию трафика и обработку подключений TLS для последующей передачи. Ресурсы входящего трафика Kubernetes используются при настройке правил входящего трафика и маршрутов для отдельных служб Kubernetes. Применяя контроллер и правила входящего трафика, для маршрутизации трафика в несколько служб в кластере Kubernetes можно использовать один IP-адрес.

В этой статье описывается, как установить и настроить [контроллер входящего трафика NGINX][nginx-ingress] в кластере Службы Azure Kubernetes (AKS). Контроллер входящего трафика настроен для внутренней частной виртуальной сети и IP-адреса. Внешний доступ запрещен. Затем в кластере AKS запускаются два приложения, каждое из которых доступно по одному IP-адресу.

Также можно:

- [Создать базовый контроллер входящего трафика с внешним сетевым подключением.][aks-ingress-basic]
- [Включить надстройку маршрутизации приложений HTTP.][aks-http-app-routing]
- [Создать контроллер входящего трафика, который использует ваши собственные сертификаты TLS][aks-ingress-own-tls]
- Создать контроллер входящего трафика, использующий службу Let's Encrypt для автоматического создания сертификатов TLS [с динамическим общедоступным IP-адресом][aks-ingress-tls] или [со статическим общедоступным IP-адресом][aks-ingress-static-tls].

## <a name="before-you-begin"></a>Перед началом

В этой статье для установки контроллера входящих данных NGINX используется [Helm 3][helm] . Убедитесь, что вы используете последний выпуск Helm и имеете доступ к репозиторию Helm входящего трафика *-nginx* . Дополнительную информацию о настройке и использовании Helm см. в статье [Использование Helm со службой Azure Kubernetes][use-helm].

В этой статье также предполагается, что вы используете Azure CLI версии 2.0.64 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="create-an-ingress-controller"></a>Создание контроллера входящего трафика

По умолчанию контроллер входящего трафика NGINX создается с динамическим общедоступным назначением IP-адресов. Общим требованием к конфигурации является использование внутренней частной сети и IP-адреса. Такой подход позволяет ограничить доступ к службам внутренними пользователями без внешнего доступа.

Создайте файл с именем *internal-ingress.yaml*, используя приведенный ниже пример файла манифеста. Этот пример назначает IP-адрес *10.240.0.42* ресурсу *loadBalancerIP*. Укажите собственный внутренний IP-адрес для использования с контроллером входящего трафика. Убедитесь, что этот IP-адрес не используется в вашей виртуальной сети. Кроме того, если вы используете существующую виртуальную сеть и подсеть, необходимо настроить кластер AKS с правильными разрешениями для управления виртуальной сетью и подсетью. Дополнительные сведения см. [в статье Использование сети кубенет со своими диапазонами IP-адресов в службе Kubernetes Azure (AKS)][aks-configure-kubenet-networking] или [Настройка сети CNI Azure в службе Azure Kubernetes (AKS)][aks-configure-advanced-networking] .

```yaml
controller:
  service:
    loadBalancerIP: 10.240.0.42
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-internal: "true"
```

Разверните диаграмму *nginx ingress* с помощью Helm. Чтобы использовать файл манифеста, созданный на предыдущем шаге, добавьте параметр `-f internal-ingress.yaml`. Для обеспечения дополнительной избыточности развертываются две реплики контроллеров входящего трафика NGINX с использованием параметра `--set controller.replicaCount`. Чтобы максимально эффективно использовать реплики контроллера входящего трафика, убедитесь, что в кластере AKS используется несколько узлов.

Контроллер Ingress также необходимо запланировать на узле Linux. Узлы Windows Server не должны запускать контроллер Ingress. Селектор узла указывается с помощью параметра `--set nodeSelector`, чтобы сообщить планировщику Kubernetes о необходимости запуска контроллера NGINX Ingress на узле под управлением Linux.

> [!TIP]
> В следующем примере создается пространство имен Kubernetes для входящих ресурсов с именем входящие *-Basic*. При необходимости укажите пространство имен для своей среды. Если в кластере AKS не включен Kubernetes RBAC, добавьте `--set rbac.create=false` в команды Helm.

> [!TIP]
> Если вы хотите включить [Сохранение IP-адреса источника клиента][client-source-ip] для запросов к контейнерам в кластере, добавьте `--set controller.service.externalTrafficPolicy=Local` команду Helm install. Исходный IP-адрес клиента хранится в заголовке запроса в разделе *X-forwardd-for*. При использовании контроллера входящего трафика с включенным сохранением IP-адресов источника клиента передача TLS не будет работать.

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    -f internal-ingress.yaml \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."beta\.kubernetes\.io/os"=linux
```

При создании службы балансировщика нагрузки Kubernetes для контроллера входящих данных NGINX внутренний IP-адрес назначается. Чтобы получить общедоступный IP-адрес, используйте команду `kubectl get service`.

```console
kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller
```

Назначение IP-адреса службе занимает несколько минут, как показано в следующем примере выходных данных:

```
$ kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller

NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.74.133   EXTERNAL_IP     80:32486/TCP,443:30953/TCP   44s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

Так как правила входящего трафика не созданы, при переходе к внутреннему IP-адресу по умолчанию будет отображаться страница контроллера входящего трафика NGINX с ошибкой "404 — страница не найдена". В следующих шагах настраиваются правила входящего трафика.

## <a name="run-demo-applications"></a>Запуск демонстрационных версий приложений

Чтобы увидеть контроллер входящего трафика в действии, запустите два демонстрационных приложения в кластере AKS. В этом примере используется `kubectl apply` для развертывания двух экземпляров простого приложения *Hello World* .

Создайте файл *AKS-HelloWorld. YAML* и скопируйте в следующем примере YAML:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld
  template:
    metadata:
      labels:
        app: aks-helloworld
    spec:
      containers:
      - name: aks-helloworld
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld
```

Создайте файл *входной-Demo. YAML* и скопируйте в следующем примере YAML:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ingress-demo
  template:
    metadata:
      labels:
        app: ingress-demo
    spec:
      containers:
      - name: ingress-demo
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "AKS Ingress Demo"
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-demo
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: ingress-demo
```

Запустите два демонстрационных приложения с помощью `kubectl apply` :

```console
kubectl apply -f aks-helloworld.yaml --namespace ingress-basic
kubectl apply -f ingress-demo.yaml --namespace ingress-basic
```

## <a name="create-an-ingress-route"></a>Создание маршрута для входящего трафика

Оба приложения запущены в кластере Kubernetes. Для маршрутизации трафика для каждого приложения создайте ресурс входящего трафика Kubernetes. Ресурс входящего трафика настраивает правила, по которым осуществляется маршрутизация трафика к одному из двух приложений.

В следующем примере трафик по адресу `http://10.240.0.42/` направляется в службу `aks-helloworld`. Трафик по адресу `http://10.240.0.42/hello-world-two` направляется в службу `ingress-demo`.

Создайте файл `hello-world-ingress.yaml` и скопируйте в него следующий пример кода YAML:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /hello-world-one(/|$)(.*)
      - backend:
          serviceName: ingress-demo
          servicePort: 80
        path: /hello-world-two(/|$)(.*)
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /(.*)
```

Создайте ресурс входящего трафика, используя команду `kubectl apply -f hello-world-ingress.yaml`.

```console
kubectl apply -f hello-world-ingress.yaml
```

В следующем примере выходных данных показано, что создан ресурс входящих.

```
$ kubectl apply -f hello-world-ingress.yaml

ingress.extensions/hello-world-ingress created
```

## <a name="test-the-ingress-controller"></a>Тестирование контроллера входящего трафика

Чтобы проверить маршруты для контроллера входящего трафика, перейдите в два приложения с помощью веб-клиента. При необходимости вы можете быстро проверить эту внутреннюю функцию системного модуля pod в кластере AKS. Создайте тестовый системный модуль pod и свяжите с ним сеанс терминала.

```console
kubectl run -it --rm aks-ingress-test --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --namespace ingress-basic
```

Установите `curl` в модуле pod с помощью `apt-get`:

```console
apt-get update && apt-get install -y curl
```

Теперь получите доступ к адресу входящего контроллера Kubernetes с помощью `curl` , например *http://10.240.0.42* . Укажите свой внутренний IP-адрес, указанный при развертывании контроллера входящего трафика на первом шаге, описанном в этой статье.

```console
curl -L http://10.240.0.42
```

С адресом не был предоставлен дополнительный путь, поэтому для контроллера входящих данных по умолчанию используется */* маршрут. Первое демонстрационное приложение возвращается, как показано в следующем сокращенном примере выходных данных:

```
$ curl -L http://10.240.0.42

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <link rel="stylesheet" type="text/css" href="/static/default.css">
    <title>Welcome to Azure Kubernetes Service (AKS)</title>
[...]
```

Теперь добавьте к адресу путь */hello-world-two*, например *http://10.240.0.42/hello-world-two*. Второе демонстрационное приложение с настраиваемым заголовком возвращается, как показано в следующем сокращенном примере выходных данных:

```
$ curl -L -k http://10.240.0.42/hello-world-two

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <link rel="stylesheet" type="text/css" href="/static/default.css">
    <title>AKS Ingress Demo</title>
[...]
```

## <a name="clean-up-resources"></a>Очистка ресурсов

В этой статье мы использовали Helm для установки входящих компонентов. При развертывании диаграммы Helm создается несколько ресурсов Kubernetes. К ним относятся элементы pod, развертывания и службы. Чтобы очистить эти ресурсы, можно либо удалить весь пример пространства имен, либо отдельные ресурсы.

### <a name="delete-the-sample-namespace-and-all-resources"></a>Удаление образца пространства имен и всех ресурсов

Чтобы удалить весь пример пространства имен, используйте `kubectl delete` команду и укажите имя пространства имен. Все ресурсы в пространстве имен удаляются.

```console
kubectl delete namespace ingress-basic
```

### <a name="delete-resources-individually"></a>Удаление ресурсов по отдельности

Кроме того, более детализированный подход заключается в удалении отдельных созданных ресурсов. Выведите список выпусков Helm с помощью `helm list` команды. 

```console
helm list --namespace ingress-basic
```

Найдите диаграммы *nginx-ingress* и *aks-helloworld*, как показано в следующем примере выходных данных.

```
$ helm list --namespace ingress-basic

NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
nginx-ingress           ingress-basic   1               2020-01-06 19:55:46.358275 -0600 CST    deployed        nginx-ingress-1.27.1    0.26.1  
```

Удалите выпуски с помощью `helm uninstall` команды.

```console
helm uninstall nginx-ingress --namespace ingress-basic
```

В следующем примере удаляется NGINX входящее развертывание.

```
$ helm uninstall nginx-ingress --namespace ingress-basic

release "nginx-ingress" uninstalled
```

Затем удалите два примера приложений:

```console
kubectl delete -f aks-helloworld.yaml --namespace ingress-basic
kubectl delete -f ingress-demo.yaml --namespace ingress-basic
```

Удалите маршрут входящего трафика, направлявший трафик в пример приложения.

```console
kubectl delete -f hello-world-ingress.yaml
```

Наконец, можно удалить само пространство имен. Используйте `kubectl delete` команду и укажите имя пространства имен:

```console
kubectl delete namespace ingress-basic
```

## <a name="next-steps"></a>Дальнейшие действия

В данной статье упоминаются некоторые внешние компоненты для AKS. Чтобы узнать больше об этих компонентах, см. следующие страницы проекта:

- [Интерфейс командной строки Helm][helm-cli]
- [Контроллер входящего трафика NGINX][nginx-ingress]

Также можно:

- [Создать базовый контроллер входящего трафика с внешним сетевым подключением.][aks-ingress-basic]
- [Включить надстройку маршрутизации приложений HTTP.][aks-http-app-routing]
- [Создать контроллер входящего трафика с динамическим общедоступным IP-адресом и настроить шифрование для автоматического создания TLS-сертификатов.][aks-ingress-tls]
- [Создать контроллер входящего трафика со статическим общедоступным IP-адресом и настроить шифрование для автоматического создания TLS-сертификатов.][aks-ingress-static-tls]

<!-- LINKS - external -->
[helm]: https://helm.sh/
[helm-cli]: ./kubernetes-helm.md
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
[client-source-ip]: concepts-network.md#ingress-controllers
[aks-configure-kubenet-networking]: configure-kubenet.md
[aks-configure-advanced-networking]: configure-azure-cni.md
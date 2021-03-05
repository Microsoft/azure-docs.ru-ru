---
title: Надстройка маршрутизации приложений HTTP в Службе Azure Kubernetes (AKS)
description: Используйте надстройку маршрутизации приложений HTTP для доступа к приложениям, развернутым в службе Kubernetes Azure (AKS).
services: container-service
author: lachie83
ms.topic: article
ms.date: 07/20/2020
ms.author: laevenso
ms.openlocfilehash: 25fc021a48e8936f242df35f7485fc59a93bba13
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102172806"
---
# <a name="http-application-routing"></a>Маршрутизация приложений HTTP

Решение маршрутизации приложений HTTP упрощает доступ к приложениям, развернутым в кластере Службы Azure Kubernetes (AKS). Если решение включено, оно настраивает [входной контроллер](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) в кластере AKS. Когда приложения развернуты, решение также создает общедоступные DNS-имена для конечных точек приложений.

При включении этой надстройки в подписке создается зона DNS. Дополнительные сведения о стоимости DNS см. [на этой странице][dns-pricing].

> [!CAUTION]
> Надстройка маршрутизации приложения HTTP предназначена для быстрого создания входного контроллера и доступа к вашим приложениям. Эта надстройка в настоящее время не предназначена для использования в рабочей среде и не рекомендуется для использования в рабочем окружении. Дополнительные сведения о готовых к развертыванию приложений, включающих несколько реплик и поддержку TLS, см. раздел [Создание входного контроллера HTTPS](./ingress-tls.md).

## <a name="http-routing-solution-overview"></a>Обзор решения маршрутизации HTTP-трафика

Надстройка развертывает два компонента: контроллер входящего трафика [Kubernetes][ingress] и контроллер [внешнего DNS][external-dns] .

- **Контроллер входящего трафика**. Доступ к контроллеру входящего трафика в Интернете предоставляется с помощью Службы Azure Kubernetes типа LoadBalancer. Входной контроллер отслеживает и реализует Kubernetes входящие [ресурсы][ingress-resource], которые создают маршруты к конечным точкам приложения.
- **Контроллер внешних DNS**. Отслеживает ресурсы входящего трафика Службы Azure Kubernetes и создает записи A DNS в зоне DNS с определенным кластером.

## <a name="deploy-http-routing-cli"></a>Развертывание маршрутизации HTTP-трафика: CLI

Надстройку для маршрутизации приложений HTTP можно активировать с помощью Azure CLI при развертывании кластера AKS. Для этого используйте команду [az aks create][az-aks-create] с аргументом `--enable-addons`.

```azurecli
az aks create --resource-group myResourceGroup --name myAKSCluster --enable-addons http_application_routing
```

> [!TIP]
> Если вы хотите включить несколько надстроек, укажите их в виде списка через запятую. Например, чтобы включить маршрутизацию и мониторинг приложений HTTP, используйте формат `--enable-addons http_application_routing,monitoring`.

Можно также включить маршрутизацию HTTP для существующего кластера AKS с помощью команды [az aks enable-addons][az-aks-enable-addons]. Чтобы включить маршрутизацию HTTP в существующем кластере, добавьте параметр `--addons` и укажите *http_application_routing*, как показано в следующем примере.

```azurecli
az aks enable-addons --resource-group myResourceGroup --name myAKSCluster --addons http_application_routing
```

После развертывания или обновления кластера получите имя зоны DNS с помощью команды [az aks show][az-aks-show].

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName -o table
```

Это имя необходимо для развертывания приложений в кластере AKS и показано в следующем примере выходных данных:

```console
9f9c1fe7-21a1-416d-99cd-3543bb92e4c3.eastus.aksapp.io
```

## <a name="deploy-http-routing-portal"></a>Развертывание маршрутизации HTTP-трафика: портал

При развертывании кластера службы AKS надстройку для маршрутизации приложений HTTP можно активировать на портале Azure.

![Включение функции маршрутизации HTTP-трафика](media/http-routing/create.png)

После развертывания кластера перейдите к автоматически созданной группе ресурсов службы AKS и выберите зону DNS. Запишите имя зоны DNS. Оно потребуется, чтобы развертывать приложения в кластер службы AKS.

![Получение имени зоны DNS](media/http-routing/dns.png)

## <a name="connect-to-your-aks-cluster"></a>Подключение к кластеру AKS

Для подключения к кластеру Kubernetes с локального компьютера используйте средство [kubectl][kubectl] (клиент командной строки Kubernetes).

Если вы используете Azure Cloud Shell, `kubectl` уже установлен. Его также можно установить локально с помощью команды [az aks install-cli][]:

```azurecli
az aks install-cli
```

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials][]. Следующий пример получает учетные данные для кластера AKS с именем *MyAKSCluster* в *MyResourceGroup*:

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKSCluster
```

## <a name="use-http-routing"></a>Использование маршрутизации HTTP-трафика

Решение маршрутизации приложений HTTP можно активировать только в ресурсах входящего трафика помеченных следующим образом.

```yaml
annotations:
  kubernetes.io/ingress.class: addon-http-application-routing
```

Создайте файл с именем **samples-http-application-routing.yaml** и скопируйте его в следующий YAML. В строке 43 обновите `<CLUSTER_SPECIFIC_DNS_ZONE>`, указав имя зоны DNS, полученное на предыдущем шаге этой статьи.

```yaml
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
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: aks-helloworld
  annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
spec:
  rules:
  - host: aks-helloworld.<CLUSTER_SPECIFIC_DNS_ZONE>
    http:
      paths:
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /
```

Выполните команду [kubectl apply][kubectl-apply], чтобы создать ресурсы.

```bash
kubectl apply -f samples-http-application-routing.yaml
```

В следующем примере показаны созданные ресурсы:

```bash
$ kubectl apply -f samples-http-application-routing.yaml

deployment.apps/aks-helloworld created
service/aks-helloworld created
ingress.networking.k8s.io/aks-helloworld created
```

Откройте веб-браузер в *AKS-HelloWorld. \<CLUSTER_SPECIFIC_DNS_ZONE\>* например, *AKS-HelloWorld.9f9c1fe7-21a1-416d-99cd-3543bb92e4c3.eastus.aksapp.IO* и убедитесь, что вы видите демонстрационное приложение. На отображение приложения может потребоваться несколько минут.

## <a name="remove-http-routing"></a>Удаление маршрутизации HTTP

Решение для маршрутизации HTTP можно удалить с помощью Azure CLI. Для этого выполните следующую команду, заменив свой кластер AKS и имя группы ресурсов.

```azurecli
az aks disable-addons --addons http_application_routing --name myAKSCluster --resource-group myResourceGroup --no-wait
```

При отключении надстройки маршрутизации приложений HTTP некоторые ресурсы Kubernetes могут оставаться в кластере. Эти ресурсы включают объекты *configMap* и *секреты* и создаются в пространстве имен *kube-system*. Чтобы поддерживать чистоту кластера, можно удалить эти ресурсы.

Найдите ресурсы *addon-http-application-routing* с помощью следующих команд [kubectl get][kubectl-get].

```console
kubectl get deployments --namespace kube-system
kubectl get services --namespace kube-system
kubectl get configmaps --namespace kube-system
kubectl get secrets --namespace kube-system
```

В следующем примере выходных данных показаны объекты configMap, подлежащие удалению.

```
$ kubectl get configmaps --namespace kube-system

NAMESPACE     NAME                                                       DATA   AGE
kube-system   addon-http-application-routing-nginx-configuration         0      9m7s
kube-system   addon-http-application-routing-tcp-services                0      9m7s
kube-system   addon-http-application-routing-udp-services                0      9m7s
```

Чтобы удалить ресурсы, используйте команду [kubectl delete][kubectl-delete]. Укажите тип ресурса, имя ресурса и пространство имен. В следующем примере удаляется один из предыдущих объектов configmap.

```console
kubectl delete configmaps addon-http-application-routing-nginx-configuration --namespace kube-system
```

Повторите предыдущий шаг `kubectl delete` для всех ресурсов *addon-http-application-routing*, которые остаются в кластере.

## <a name="troubleshoot"></a>Диагностика

Выполните команду [kubectl logs][kubectl-logs], чтобы просмотреть журналы приложений для внешнего приложения DNS. В журналах должно быть указано, что записи A и TXT DNS успешно созданы.

```
$ kubectl logs -f deploy/addon-http-application-routing-external-dns -n kube-system

time="2018-04-26T20:36:19Z" level=info msg="Updating A record named 'aks-helloworld' to '52.242.28.189' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
time="2018-04-26T20:36:21Z" level=info msg="Updating TXT record named 'aks-helloworld' to '"heritage=external-dns,external-dns/owner=default"' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
```

Эти записи также можно увидеть в ресурсе зоны DNS на портале Azure.

![Получение записей DNS](media/http-routing/clippy.png)

Используйте команду [kubectl logs][kubectl-logs] для просмотра журналов приложений для контроллера входящих данных nginx. В журналах должно быть указано, что `CREATE` ресурса входящего трафика создан и контроллер перезагружен. Все действия по протоколу HTTP регистрируются в журнале.

```bash
$ kubectl logs -f deploy/addon-http-application-routing-nginx-ingress-controller -n kube-system

-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.13.0
  Build:      git-4bc943a
  Repository: https://github.com/kubernetes/ingress-nginx
-------------------------------------------------------------------------------

I0426 20:30:12.212936       9 flags.go:162] Watching for ingress class: addon-http-application-routing
W0426 20:30:12.213041       9 flags.go:165] only Ingress with class "addon-http-application-routing" will be processed by this ingress controller
W0426 20:30:12.213505       9 client_config.go:533] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0426 20:30:12.213752       9 main.go:181] Creating API client for https://10.0.0.1:443
I0426 20:30:12.287928       9 main.go:225] Running in Kubernetes Cluster version v1.8 (v1.8.11) - git (clean) commit 1df6a8381669a6c753f79cb31ca2e3d57ee7c8a3 - platform linux/amd64
I0426 20:30:12.290988       9 main.go:84] validated kube-system/addon-http-application-routing-default-http-backend as the default backend
I0426 20:30:12.294314       9 main.go:105] service kube-system/addon-http-application-routing-nginx-ingress validated as source of Ingress status
I0426 20:30:12.426443       9 stat_collector.go:77] starting new nginx stats collector for Ingress controller running in namespace  (class addon-http-application-routing)
I0426 20:30:12.426509       9 stat_collector.go:78] collector extracting information from port 18080
I0426 20:30:12.448779       9 nginx.go:281] starting Ingress controller
I0426 20:30:12.463585       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-nginx-configuration", UID:"2588536c-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"559", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-nginx-configuration
I0426 20:30:12.466945       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-tcp-services", UID:"258ca065-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"561", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-tcp-services
I0426 20:30:12.467053       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-udp-services", UID:"259023bc-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"562", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-udp-services
I0426 20:30:13.649195       9 nginx.go:302] starting NGINX process...
I0426 20:30:13.649347       9 leaderelection.go:175] attempting to acquire leader lease  kube-system/ingress-controller-leader-addon-http-application-routing...
I0426 20:30:13.649776       9 controller.go:170] backend reload required
I0426 20:30:13.649800       9 stat_collector.go:34] changing prometheus collector from  to default
I0426 20:30:13.662191       9 leaderelection.go:184] successfully acquired lease kube-system/ingress-controller-leader-addon-http-application-routing
I0426 20:30:13.662292       9 status.go:196] new leader elected: addon-http-application-routing-nginx-ingress-controller-5cxntd6
I0426 20:30:13.763362       9 controller.go:179] ingress backend successfully reloaded...
I0426 21:51:55.249327       9 event.go:218] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"aks-helloworld", UID:"092c9599-499c-11e8-a5e1-0a58ac1f0ef2", APIVersion:"extensions", ResourceVersion:"7346", FieldPath:""}): type: 'Normal' reason: 'CREATE' Ingress default/aks-helloworld
W0426 21:51:57.908771       9 controller.go:775] service default/aks-helloworld does not have any active endpoints
I0426 21:51:57.908951       9 controller.go:170] backend reload required
I0426 21:51:58.042932       9 controller.go:179] ingress backend successfully reloaded...
167.220.24.46 - [167.220.24.46] - - [26/Apr/2018:21:53:20 +0000] "GET / HTTP/1.1" 200 234 "" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)" 197 0.001 [default-aks-helloworld-80] 10.244.0.13:8080 234 0.004 200
```

## <a name="clean-up"></a>Очистка

Удалите связанные объекты Kubernetes, созданные в этой статье, с помощью `kubectl delete` .

```bash
kubectl delete -f samples-http-application-routing.yaml
```

В выходных данных примера показаны объекты Kubernetes, которые были удалены.

```bash
$ kubectl delete -f samples-http-application-routing.yaml

deployment "aks-helloworld" deleted
service "aks-helloworld" deleted
ingress "aks-helloworld" deleted
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения об установке контроллера защищенного входящего HTTPS-трафика в AKS см. в статье [Входящий трафик HTTPS в Службе Azure Kubernetes (AKS)][ingress-https].

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-show]: /cli/azure/aks#az-aks-show
[ingress-https]: ./ingress-tls.md
[az-aks-enable-addons]: /cli/azure/aks#az-aks-enable-addons
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials

<!-- LINKS - external -->
[dns-pricing]: https://azure.microsoft.com/pricing/details/dns/
[external-dns]: https://github.com/kubernetes-incubator/external-dns
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[ingress-resource]: https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource

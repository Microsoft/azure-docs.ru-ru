---
title: Использование пользовательского контроллера входящих данных NGINX и настройка HTTPS
services: azure-dev-spaces
ms.date: 12/10/2019
ms.topic: conceptual
description: Узнайте, как настроить Azure Dev Spaces для использования пользовательского контроллера входящих данных NGINX и настройки HTTPS с помощью этого контроллера входящего трафика.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, containers, Helm, service mesh, service mesh routing, kubectl, k8s
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: c5ef14074c6e601bcd8a23ce62921d67b01ba3bb
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102198138"
---
# <a name="use-a-custom-nginx-ingress-controller-and-configure-https"></a>Использование пользовательского контроллера входящих данных NGINX и настройка HTTPS

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

В этой статье показано, как настроить Azure Dev Spaces для использования пользовательского контроллера входящего трафика NGINX. В этой статье также показано, как настроить этот настраиваемый контроллер входящего трафика для использования протокола HTTPS.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если ее нет, можно создать [бесплатную учетную запись][azure-account-create].
* [Установленный Azure CLI][az-cli].
* Кластер Azure Kubernetes Service (AKS) с включенным Azure Dev Spaces.
* [kubectl][kubectl] установлен.
* [Установленная версия Helm 3][helm-installed].
* [Личный домен][custom-domain] с [зоной DNS][dns-zone].  В этой статье предполагается, что личный домен и зона DNS находятся в той же группе ресурсов, что и кластер AKS, но можно использовать личный домен и зону DNS в другой группе ресурсов.

## <a name="configure-a-custom-nginx-ingress-controller"></a>Настройка пользовательского контроллера входящего трафика NGINX

Подключитесь к кластеру с помощью [kubectl][kubectl], клиента командной строки Kubernetes. Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials][az-aks-get-credentials]. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKS
```

Чтобы проверить подключение к кластеру, используйте команду [kubectl get][kubectl-get] для получения списка узлов кластера.

```console
kubectl get nodes
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.14.1
```

Добавьте [официальный стабильный репозиторий Helm][helm-stable-repo], который содержит диаграмму Helm для контроллера входящих данных nginx.

```console
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

Создайте пространство имен Kubernetes для контроллера входящих данных NGINX и установите его с помощью `helm` .

```console
kubectl create ns nginx
helm install nginx stable/nginx-ingress --namespace nginx --version 1.27.0
```

> [!NOTE]
> В приведенном выше примере создается общедоступная конечная точка для контроллера входящего трафика. Если вместо этого необходимо использовать частную конечную точку для контроллера входящего трафика, добавьте параметр *--Set Controller. Service. Annotations. Service \\ . Beta \\ . kubernetes \\ . IO/Azure — Load-балансировщик — внутренний параметр "= true"* для команды *установки Helm* . Пример:
> ```console
> helm install nginx stable/nginx-ingress --namespace nginx --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"=true --version 1.27.0
> ```
> Эта частная конечная точка предоставляется в виртуальной сети, в которой развернут кластер AKS.

Получите IP-адрес службы NGINX входящего контроллера с помощью [kubectl Get][kubectl-get].

```console
kubectl get svc -n nginx --watch
```

В примере выходных данных показаны IP-адреса всех служб в пространстве имен *nginx* .

```console
NAME                                  TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
nginx-nginx-ingress-controller        LoadBalancer   10.0.19.39     <pending>        80:31314/TCP,443:30521/TCP   10s
nginx-nginx-ingress-default-backend   ClusterIP      10.0.210.231   <none>           80/TCP                       10s
...
nginx-nginx-ingress-controller        LoadBalancer   10.0.19.39     MY_EXTERNAL_IP   80:31314/TCP,443:30521/TCP   26s
```

Добавьте запись *A* в зону DNS с внешним IP-адресом службы nginx с помощью команды [AZ Network DNS запись-Set A-Record][az-network-dns-record-set-a-add-record].

```azurecli
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name *.nginx \
    --ipv4-address MY_EXTERNAL_IP
```

В приведенном выше примере запись *A* добавляется в зону DNS *MY_CUSTOM_DOMAIN* .

В этой статье на примере [приложения Azure Dev Spaces для аренды велосипедов](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp) вы изучите применение Azure Dev Spaces. Клонируйте приложение с GitHub и перейдите в его каталог:

```cmd
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/BikeSharingApp/charts
```

Откройте [Values. YAML][values-yaml] и внесите следующие обновления:
* Замените все экземпляры *<REPLACE_ME_WITH_HOST_SUFFIX>* *nginx. MY_CUSTOM_DOMAIN* использование домена для *MY_CUSTOM_DOMAIN*. 
* Замените *kubernetes.IO/Ingress.class: траефик-аздс # dev Spaces, относящийся* к *kubernetes.IO/Ingress.class: nginx # Custom входной* вход. 

Ниже приведен пример обновленного `values.yaml` файла.

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
    hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
    hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
```

Сохраните изменения и закройте файл.

Создайте пространство *разработки* с помощью примера приложения, используя `azds space select` .

```console
azds space select -n dev -y
```

Разверните пример приложения с помощью `helm install` .

```console
helm install bikesharingsampleapp . --dependency-update --namespace dev --atomic
```

В приведенном выше примере пример приложения развертывается в пространстве имен *dev* .

Отображение URL-адресов для доступа к образцу приложения с помощью `azds list-uris` .

```console
azds list-uris
```

В приведенных ниже выходных данных показаны примеры URL-адресов из `azds list-uris` .

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/  Available
http://dev.gateway.nginx.MY_CUSTOM_DOMAIN/         Available
```

Перейдите к службе *bikesharingweb* по общедоступному URL-адресу, который вам предоставила команда `azds list-uris`. В примере выше для службы *bikesharingweb* используется общедоступный URL-адрес `http://dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/`.

> [!NOTE]
> Если вместо службы *бикешарингвеб* отображается страница ошибки, убедитесь **, что вы обновили** заметку *kubernetes.IO/Ingress.class* и узел в файле *Values. YAML* .

Используйте `azds space select` команду, чтобы создать дочернее пространство в разделе *dev* и вывести список URL-адресов для доступа к дочернему пространству разработки.

```console
azds space select -n dev/azureuser1 -y
azds list-uris
```

В приведенных ниже выходных данных показаны примеры URL-адресов из `azds list-uris` для доступа к образцу приложения в дочерней области разработки *azureuser1* .

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/  Available
http://azureuser1.s.dev.gateway.nginx.MY_CUSTOM_DOMAIN/         Available
```

Перейдите к службе *бикешарингвеб* в пространстве дочернего пространства разработки *azureuser1* , открыв общедоступный URL-адрес из `azds list-uris` команды. В приведенном выше примере общедоступный URL-адрес службы *бикешарингвеб* в пространстве дочернего разработчика *azureuser1* — `http://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/` .

## <a name="configure-the-nginx-ingress-controller-to-use-https"></a>Настройка контроллера входящего трафика NGINX для использования протокола HTTPS

Используйте [Диспетчер сертификатов][cert-manager] для автоматизации управления сертификатом TLS при настройке контроллера входящих подключений Nginx для использования протокола HTTPS. Используйте `helm` для установки диаграммы *цертманажер* .

```console
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml --namespace nginx
kubectl label namespace nginx certmanager.k8s.io/disable-validation=true
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager --namespace nginx --version v0.12.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt --set ingressShim.defaultIssuerKind=ClusterIssuer
```

Создайте `letsencrypt-clusterissuer.yaml` файл и обновите поле электронной почты, указав свой адрес электронной почты.

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: MY_EMAIL_ADDRESS
    privateKeySecretRef:
      name: letsencrypt
    solvers:
      - http01:
          ingress:
            class: nginx
```

> [!NOTE]
> Для тестирования также существует [промежуточный сервер][letsencrypt-staging-issuer] , который можно использовать для *клустериссуер*.

Используйте `kubectl` для применения `letsencrypt-clusterissuer.yaml` .

```console
kubectl apply -f letsencrypt-clusterissuer.yaml --namespace nginx
```

Обновите [Values. YAML][values-yaml] , чтобы включить сведения об использовании *ДИСПЕТЧЕРА сертификатов* и HTTPS. Ниже приведен пример обновленного `values.yaml` файла.

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN
      secretName: dev-bikesharingweb-secret

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN
      secretName: dev-gateway-secret
```

Обновите пример приложения с помощью `helm` :

```console
helm upgrade bikesharingsampleapp . --namespace dev --atomic
```

Перейдите к образцу приложения в дочернем пространстве *dev/azureuser1* и обратите внимание, что вы перенаправлялись на использование протокола HTTPS. Также обратите внимание, что страница загружается, но в браузере отображаются некоторые ошибки. При открытии консоли браузера отображается ошибка, связанная с HTTPS-страницей, пытающейся загрузить ресурсы HTTP. Пример:

```console
Mixed Content: The page at 'https://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/devsignin' was loaded over HTTPS, but requested an insecure resource 'http://azureuser1.s.dev.gateway.nginx.MY_CUSTOM_DOMAIN/api/user/allUsers'. This request has been blocked; the content must be served over HTTPS.
```

Чтобы устранить эту ошибку, обновите [бикешарингвеб/аздс. YAML][azds-yaml] , как показано ниже:

```yaml
...
    ingress:
      annotations:
        kubernetes.io/ingress.class: nginx
        cert-manager.io/cluster-issuer: letsencrypt
      hosts:
      # This expands to [space.s.][rootSpace.]bikesharingweb.<random suffix>.<region>.azds.io
      - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.nginx.MY_CUSTOM_DOMAIN
      tls:
      - hosts:
        - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.nginx.MY_CUSTOM_DOMAIN
        secretName: dev-bikesharingweb-secret
...
```

Обновление [бикешарингвеб/package.json][package-json] с зависимостью от пакета *URL-адреса* .

```json
{
...
    "react-responsive": "^6.0.1",
    "universal-cookie": "^3.0.7",
    "url": "0.11.0"
  },
...
```

Обновите метод *жетапихостасинк* в [бикешарингвеб/lib/helpers.js][helpers-js] , чтобы использовать HTTPS:

```javascript
...
    getApiHostAsync: async function() {
        const apiRequest = await fetch('/api/host');
        const data = await apiRequest.json();
        
        var urlapi = require('url');
        var url = urlapi.parse(data.apiHost);

        console.log('apiHost: ' + "https://"+url.host);
        return "https://"+url.host;
    },
...
```

Перейдите в `BikeSharingWeb` каталог и используйте `azds up` для запуска обновленной службы *бикешарингвеб* .

```console
cd ../BikeSharingWeb/
azds up
```

Перейдите к образцу приложения в дочернем пространстве *dev/azureuser1* и обратите внимание, что вы перенаправлялись на использование протокола HTTPS без ошибок.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о принципах работы Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Принцип работы Azure Dev Spaces](../how-dev-spaces-works.md)


[az-cli]: /cli/azure/install-azure-cli
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-network-dns-record-set-a-add-record]: /cli/azure/network/dns/record-set/a#az-network-dns-record-set-a-add-record
[custom-domain]: ../../app-service/manage-custom-dns-buy-domain.md#buy-an-app-service-domain
[dns-zone]: ../../dns/dns-getstarted-cli.md
[azds-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/azds.yaml
[azure-account-create]: https://azure.microsoft.com/free
[cert-manager]: https://cert-manager.io/
[helm-installed]: https://helm.sh/docs/intro/install/
[helm-stable-repo]: https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
[helpers-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/lib/helpers.js#L7
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[letsencrypt-staging-issuer]: https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer
[package-json]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/package.json
[values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml

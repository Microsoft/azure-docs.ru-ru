---
title: Использовать входной контроллер с статическим IP-адресом
titleSuffix: Azure Kubernetes Service
description: Сведения об установке и настройке контроллера входящего трафика NGINX со статическим общедоступным IP-адресом в кластере Службы Azure Kubernetes (AKS).
services: container-service
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: fa6572ddc694cb892f48cb3e618c176f087524f6
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102506571"
---
# <a name="create-an-ingress-controller-with-a-static-public-ip-address-in-azure-kubernetes-service-aks"></a>Создание контроллера входящего трафика со статическим общедоступным IP-адресом в Службе Azure Kubernetes (AKS)

Контроллер входящего трафика является программным компонентом, предоставляющим для служб Kubernetes обратный прокси-сервер, настраиваемую маршрутизацию трафика и обработку подключений TLS для последующей передачи. Ресурсы входящего трафика Kubernetes используются при настройке правил входящего трафика и маршрутов для отдельных служб Kubernetes. Применяя контроллер и правила входящего трафика, для маршрутизации трафика в несколько служб в кластере Kubernetes можно использовать один IP-адрес.

В этой статье описывается, как установить и настроить [контроллер входящего трафика NGINX][nginx-ingress] в кластере Службы Azure Kubernetes (AKS). Контроллер входящего трафика настраивается со статическим общедоступным IP-адресом. Для автоматического создания и настройки сертификатов [Let's Encrypt][lets-encrypt] используется проект [cert-manager][cert-manager]. Наконец, в кластере AKS запущено два приложения, каждое из которых доступно по одному IP-адресу.

Кроме того, вы можете сделать следующее:

- [Создать базовый контроллер входящего трафика с внешним сетевым подключением.][aks-ingress-basic]
- [Включить надстройку маршрутизации приложений HTTP.][aks-http-app-routing]
- [Создать контроллер входящего трафика, который использует ваши собственные сертификаты TLS][aks-ingress-own-tls]
- [Создать контроллер входящего трафика, который использует службу Let's Encrypt для автоматического создания сертификатов TLS со динамическим общедоступным IP-адресом][aks-ingress-tls]

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

В этой статье используется [Helm 3][helm] для установки контроллера входящих данных nginx и диспетчера сертификатов. Убедитесь, что вы используете последний выпуск Helm и имеете доступ к репозиториям входящих *-nginx* и *жетстакк* Helm. Инструкции по обновлению см. в документации по [установке Helm][helm-install]. Дополнительные сведения о настройке и использовании Helm см. [в статье Установка приложений с помощью Helm в службе Kubernetes Azure (AKS)][use-helm].

В этой статье также предполагается, что вы используете Azure CLI версии 2.0.64 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="create-an-ingress-controller"></a>Создание контроллера входящего трафика

По умолчанию контроллер входящего трафика NGINX создается с новым назначением общедоступного IP-адреса. Этот общедоступный IP-адрес является статическим только на время жизненного цикла контроллера входящего трафика и теряется, если контроллер удаляется и создается повторно. Общее требование конфигурации — предоставить контроллеру входящего трафика NGINX имеющийся статический общедоступный IP-адрес. Статический общедоступный IP-адрес останется, если контроллер входящего трафика будет удален. Такой подход позволяет согласованно использовать имеющиеся записи DNS и сетевые конфигурации на протяжении жизненного цикла приложений.

Если необходимо создать статический общедоступный IP-адрес, сначала нужно получить имя группы ресурсов кластера AKS с помощью команды [az aks show][az-aks-show].

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
```

Создайте общедоступный IP-адрес с помощью метода *статического* выделения, используя команду [az network public-ip create][az-network-public-ip-create]. В следующем примере создается общедоступный IP-адрес *myPublicIP* в группе ресурсов AKS, о которой говорилось на предыдущих шагах.

```azurecli-interactive
az network public-ip create --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP --sku Standard --allocation-method static --query publicIp.ipAddress -o tsv
```

> [!NOTE]
> Приведенные выше команды создают IP-адрес, который будет удален при удалении кластера AKS. Кроме того, можно создать IP-адрес в другой группе ресурсов, которую можно управлять отдельно от кластера AKS. При создании IP-адреса в другой группе ресурсов убедитесь, что удостоверение кластера, используемое кластером AKS, имеет делегированные разрешения на доступ к другой группе ресурсов, например к *участнику сети*. Дополнительные сведения см. [в статье Использование статического общедоступного IP-адреса и метки DNS с подсистемой балансировки нагрузки AKS][aks-static-ip].

Разверните диаграмму *nginx ingress* с помощью Helm. Для обеспечения дополнительной избыточности развертываются две реплики контроллеров входящего трафика NGINX с использованием параметра `--set controller.replicaCount`. Чтобы максимально эффективно использовать реплики контроллера входящего трафика, убедитесь, что в кластере AKS используется несколько узлов.

В выпуске Helm необходимо передать два дополнительных параметра, чтобы контроллер входящего трафика знал как статический IP-адрес подсистемы балансировки нагрузки, выделенной службе контроллера входящих данных, так и метку DNS-имени, применяемую к ресурсу общедоступного IP-адреса. Чтобы сертификаты HTTPS работали правильно, для настройки полного доменного имени для IP-адреса контроллера входящих данных используется метка DNS-имени.

1. Добавьте `--set controller.service.loadBalancerIP` параметр. Укажите собственный общедоступный IP-адрес, созданный на предыдущем шаге.
1. Добавьте `--set controller.service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"` параметр. Укажите метку DNS-имени, которая будет применяться к общедоступному IP-адресу, созданному на предыдущем шаге.

Контроллер Ingress также необходимо запланировать на узле Linux. Узлы Windows Server не должны запускать контроллер Ingress. Селектор узла указывается с помощью параметра `--set nodeSelector`, чтобы сообщить планировщику Kubernetes о необходимости запуска контроллера NGINX Ingress на узле под управлением Linux.

> [!TIP]
> В следующем примере создается пространство имен Kubernetes для входящих ресурсов с именем входящие *-Basic*. При необходимости укажите пространство имен для своей среды. Если в кластере AKS не включен Kubernetes RBAC, добавьте `--set rbac.create=false` в команды Helm.

> [!TIP]
> Если вы хотите включить [Сохранение IP-адреса источника клиента][client-source-ip] для запросов к контейнерам в кластере, добавьте `--set controller.service.externalTrafficPolicy=Local` команду Helm install. Исходный IP-адрес клиента хранится в заголовке запроса в разделе *X-forwardd-for*. При использовании контроллера входящего трафика с включенным сохранением IP-адресов источника клиента передача TLS не будет работать.

Обновите следующий скрипт, указав **IP-адрес** контроллера входящего трафика, и **уникальное имя** , которое вы хотите использовать для префикса FQDN.

> [!IMPORTANT]
> При выполнении команды необходимо обновить заменяющие *STATIC_IP* и *DNS_LABEL* своими IP-адресами и уникальными именами.

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.service.loadBalancerIP="STATIC_IP" \
    --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"="DNS_LABEL"
```

При создании службы распределения нагрузки Kubernetes входящего контроллера NGINX назначается статический IP-адрес, как показано в следующем примере выходных данных.

```
$ kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller

NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.74.133   EXTERNAL_IP     80:32486/TCP,443:30953/TCP   44s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

Так как правила входящего трафика не созданы, при переходе к общедоступному IP-адресу по умолчанию будет отображаться страница контроллеров входящего трафика NGINX с ошибкой "404 — страница не найдена". В следующих шагах настраиваются правила входящего трафика.

Чтобы проверить, применена ли метка DNS-имени, запросите полное доменное имя общедоступного IP-адреса следующим образом:

```azurecli-interactive
az network public-ip list --resource-group MC_myResourceGroup_myAKSCluster_eastus --query "[?ipAddress=='myAKSPublicIP'].[dnsSettings.fqdn]" -o tsv
```

Контроллер входящего трафика теперь доступен через IP-адрес или полное доменное имя.

## <a name="install-cert-manager"></a>Установка cert-manager

Контроллер входящего трафика NGINX поддерживает обработку подключений TLS для последующей передачи. Получить и настроить сертификаты для HTTPS можно несколькими способами. В этой статье демонстрируется использование проекта [cert-manager][cert-manager], который обеспечивает автоматические функции создания сертификатов [Lets Encrypt][lets-encrypt] и управления ими.

> [!NOTE]
> В этой статье используется среда для Let's Encrypt `staging`. В производственных развертываниях используйте `letsencrypt-prod` и `https://acme-v02.api.letsencrypt.org/directory` в определениях ресурсов и при установке диаграммы Helm.

Чтобы установить контроллер диспетчера сертификатов в кластере с поддержкой Kubernetes RBAC, используйте следующую `helm install` команду:

```console
# Label the cert-manager namespace to disable resource validation
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  cert-manager \
  --namespace ingress-basic \
  --version v0.16.1 \
  --set installCRDs=true \
  --set nodeSelector."beta\.kubernetes\.io/os"=linux \
  jetstack/cert-manager
```

Дополнительные сведения о конфигурации cert-manager см. в описании [проекта cert-manager][cert-manager].

## <a name="create-a-ca-cluster-issuer"></a>Создание издателя кластера ЦС

Для выдачи сертификатов средству cert-manager нужен ресурс [Issuer][cert-manager-issuer] или [ClusterIssuer][cert-manager-cluster-issuer]. Эти ресурсы Kubernetes имеют аналогичную функциональность, однако `Issuer` работает в отдельном пространстве имен, а `ClusterIssuer` — во всех. Дополнительные сведения см. в документации по [издателю cert-manager][cert-manager-issuer].

Создайте издатель кластера, например `cluster-issuer.yaml`, используя приведенный ниже пример манифеста. Измените адрес электронной почты на действительный адрес вашей организации:

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: user@contoso.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
          podTemplate:
            spec:
              nodeSelector:
                "kubernetes.io/os": linux
```

Чтобы создать издатель, используйте команду `kubectl apply`.

```
kubectl apply -f cluster-issuer.yaml --namespace ingress-basic
```

Результат должен быть похож на следующий пример:

```
clusterissuer.cert-manager.io/letsencrypt-staging created
```

## <a name="run-demo-applications"></a>Запуск демонстрационных версий приложений

Контроллер входящего трафика и решение по управлению сертификатами настроены. Теперь запустите две демонстрационные версии приложения в своем кластере AKS. В этом примере для развертывания двух экземпляров простого приложения Hello World применяется Helm.

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

Теперь оба приложения запущены на кластере Kubernetes, несмотря на то что они настроены с помощью службы типа `ClusterIP`. Таким образом приложения не доступны через Интернет. Чтобы сделать их доступными, создайте ресурс входящего трафика Kubernetes. Ресурс входящего трафика настраивает правила, по которым осуществляется маршрутизация трафика к одному из двух приложений.

В следующем примере трафик по адресу `https://demo-aks-ingress.eastus.cloudapp.azure.com/` направляется в службу `aks-helloworld`. Трафик по адресу `https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two` направляется в службу `ingress-demo`. Обновите *узлы* и *узел*, указав DNS-имя, которое вы создали на предыдущем шаге.

Создайте файл `hello-world-ingress.yaml` и скопируйте в него следующий пример кода YAML:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-staging
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  tls:
  - hosts:
    - demo-aks-ingress.eastus.cloudapp.azure.com
    secretName: tls-secret
  rules:
  - host: demo-aks-ingress.eastus.cloudapp.azure.com
    http:
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

Создайте ресурс входящего трафика, используя команду `kubectl apply`.

```
kubectl apply -f hello-world-ingress.yaml --namespace ingress-basic
```

Результат должен быть похож на следующий пример:

```
ingress.extensions/hello-world-ingress created
```

## <a name="create-a-certificate-object"></a>Создание объекта сертификата

Далее нужно создать ресурс сертификата. Этот ресурс сертификата определяет необходимый сертификат X.509. Дополнительные сведения см. в разделе [Сертификаты-менеджеры сертификатов][cert-manager-certificates].

cert-manager скорее всего автоматически создал объект сертификата с помощью ingress-shim — оболочки совместимости, которая автоматически развертывается с помощью cert-manager, начиная с версии 0.2.2. Дополнительные сведения см. в [документации по ingress-shim][ingress-shim].

Чтобы убедиться, что сертификат был успешно создан, используйте команду `kubectl describe certificate tls-secret --namespace ingress-basic`.

Если сертификат был выдан, отобразятся выходные данные, аналогичные этим.
```
Type    Reason          Age   From          Message
----    ------          ----  ----          -------
  Normal  CreateOrder     11m   cert-manager  Created new ACME order, attempting validation...
  Normal  DomainVerified  10m   cert-manager  Domain "demo-aks-ingress.eastus.cloudapp.azure.com" verified with "http-01" validation
  Normal  IssueCert       10m   cert-manager  Issuing certificate...
  Normal  CertObtained    10m   cert-manager  Obtained certificate from ACME server
  Normal  CertIssued      10m   cert-manager  Certificate issued successfully
```

Если вам нужно создать дополнительный ресурс сертификата, вы можете это сделать с помощью приведенного ниже примера манифеста. Обновите *dnsNames* и *домены*, указав для них DNS-имя, которое вы создали на предыдущем шаге. Если вы используете только внутренний контроллер входящего трафика, укажите внутреннее имя DNS для службы.

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: tls-secret
  namespace: ingress-basic
spec:
  secretName: tls-secret
  dnsNames:
  - demo-aks-ingress.eastus.cloudapp.azure.com
  acme:
    config:
    - http01:
        ingressClass: nginx
      domains:
      - demo-aks-ingress.eastus.cloudapp.azure.com
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

Чтобы создать ресурс сертификата, используйте команду `kubectl apply`.

```
$ kubectl apply -f certificates.yaml

certificate.cert-manager.io/tls-secret created
```

## <a name="test-the-ingress-configuration"></a>Проверка конфигурации входящего трафика

Откройте в веб-браузере полное доменное имя контроллера входящего трафика Kubernetes, например *`https://demo-aks-ingress.eastus.cloudapp.azure.com`* .

Как и в этих примерах используется `letsencrypt-staging` , выданный сертификат TLS/SSL не является доверенным для браузера. Чтобы продолжить работу с приложением, примите предупреждение. Информация о сертификате свидетельствует о том, что этот сертификат *Fake LE Intermediate X1* выдан Let's Encrypt. Этот поддельный сертификат указывает на то, что `cert-manager` правильно обработал запрос и получил сертификат от поставщика.

![Промежуточный сертификат Let's Encrypt](media/ingress/staging-certificate.png)

Когда вы меняете настройки Let's Encrypt, чтобы использовать `prod` вместо `staging`, применяется доверенный сертификат, выданный Let's Encrypt, как показано в следующем примере:

![Сертификат Let's Encrypt](media/ingress/certificate.png)

В браузере отображается демонстрационная версия приложения:

![Первый пример приложения](media/ingress/app-one.png)

Теперь добавьте к FQDN путь */hello-world-two*, например *`https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two`*. Отобразится второе демонстрационное приложение с пользовательским заголовком:

![Второй пример приложения](media/ingress/app-two.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

В этой статье для установки компонентов обработки входящего трафика, сертификатов и примеров приложений используется Helm. При развертывании диаграммы Helm создается несколько ресурсов Kubernetes. К ним относятся элементы pod, развертывания и службы. Чтобы очистить эти ресурсы, можно либо удалить весь пример пространства имен, либо отдельные ресурсы.

### <a name="delete-the-sample-namespace-and-all-resources"></a>Удаление образца пространства имен и всех ресурсов

Чтобы удалить весь пример пространства имен, используйте `kubectl delete` команду и укажите имя пространства имен. Все ресурсы в пространстве имен удаляются.

```console
kubectl delete namespace ingress-basic
```

### <a name="delete-resources-individually"></a>Удаление ресурсов по отдельности

Кроме того, более детализированный подход заключается в удалении отдельных созданных ресурсов. Сначала удалите ресурсы сертификата:

```console
kubectl delete -f certificates.yaml
kubectl delete -f cluster-issuer.yaml
```

Теперь получите список выпусков Helm с помощью команды `helm list`. Найдите диаграммы с именами *nginx-* in и *CERT-Manager* , как показано в следующем примере выходных данных:

```
$ helm list --all-namespaces

NAME                    NAMESPACE       REVISION        UPDATED                        STATUS          CHART                   APP VERSION
nginx-ingress           ingress-basic   1               2020-01-11 14:51:03.454165006  deployed        nginx-ingress-1.28.2    0.26.2
cert-manager            ingress-basic   1               2020-01-06 21:19:03.866212286  deployed        cert-manager-v0.13.0    v0.13.0
```

Удалите выпуски с помощью `helm uninstall` команды. В следующем примере удаляются развертывания NGINX входящего трафика и диспетчера сертификатов.

```
$ helm uninstall nginx-ingress cert-manager -n ingress-basic

release "nginx-ingress" deleted
release "cert-manager" deleted
```

Затем удалите два примера приложений:

```console
kubectl delete -f aks-helloworld.yaml --namespace ingress-basic
kubectl delete -f ingress-demo.yaml --namespace ingress-basic
```

Удалите само пространство имен. Используйте `kubectl delete` команду и укажите имя пространства имен:

```console
kubectl delete namespace ingress-basic
```

Наконец, удалите статический общедоступный IP-адрес, созданный для контроллера входящего трафика. Укажите имя группы ресурсов кластера *MC_*, полученное на первом шаге в этой статье, например *MC_myResourceGroup_myAKSCluster_eastus*.

```azurecli-interactive
az network public-ip delete --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP
```

## <a name="next-steps"></a>Дальнейшие действия

В данной статье упоминаются некоторые внешние компоненты для AKS. Чтобы узнать больше об этих компонентах, см. следующие страницы проекта:

- [Интерфейс командной строки Helm][helm-cli]
- [Контроллер входящего трафика NGINX][nginx-ingress]
- [cert-manager][cert-manager]

Кроме того, вы можете сделать следующее:

- [Создать базовый контроллер входящего трафика с внешним сетевым подключением.][aks-ingress-basic]
- [Включить надстройку маршрутизации приложений HTTP.][aks-http-app-routing]
- [Создать контроллер входящего трафика, который использует внутреннюю, частную сети и IP-адрес.][aks-ingress-internal]
- [Создать контроллер входящего трафика, который использует ваши собственные сертификаты TLS][aks-ingress-own-tls]
- [Создать контроллер входящего трафика с динамическим общедоступным IP-адресом и настроить шифрование для автоматического создания TLS-сертификатов.][aks-ingress-tls]

<!-- LINKS - external -->
[helm-cli]: ./kubernetes-helm.md
[cert-manager]: https://github.com/jetstack/cert-manager
[cert-manager-certificates]: https://cert-manager.readthedocs.io/en/latest/reference/certificates.html
[cert-manager-cluster-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/clusterissuers.html
[cert-manager-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/issuers.html
[lets-encrypt]: https://letsencrypt.org/
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
[helm]: https://helm.sh/
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm
[ingress-shim]: https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[client-source-ip]: concepts-network.md#ingress-controllers
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-static-ip]: static-ip.md

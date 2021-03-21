---
title: Устранение неполадок контроллера входящего трафика шлюза приложений
description: Эта статья содержит документацию по устранению распространенных вопросов и/или проблем с контроллером входящего трафика шлюза приложений.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 06/18/2020
ms.author: caya
ms.openlocfilehash: f2b9f79f0914e645c736f8a577c46baa42587332
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94874616"
---
# <a name="troubleshoot-common-questions-or-issues-with-ingress-controller"></a>Устранение распространенных вопросов и проблем с контроллером входящего трафика

[Azure Cloud Shell](https://shell.azure.com/) — это наиболее удобный способ устранения проблем с установкой AKS и агик. Запустите оболочку из [Shell.Azure.com](https://shell.azure.com/) или щелкните ссылку:

[![Внедрение запуска](https://shell.azure.com/images/launchcloudshell.png "Запуск Azure Cloud Shell")](https://shell.azure.com)


## <a name="test-with-a-simple-kubernetes-app"></a>Тестирование с помощью простого приложения Kubernetes

Ниже предполагается, что выполняются следующие действия.
  - У вас есть кластер AKS с включенной поддержкой расширенных сетей
  - АГИК установлен в кластере AKS
  - У вас уже есть шлюз приложений в виртуальной сети, совместно используемой с кластером AKS

Чтобы убедиться, что установка шлюза приложений + AKS + АГИК правильно настроена, разверните простейшее возможное приложение:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test-agic-app-pod
  labels:
    app: test-agic-app
spec:
  containers:
  - image: "mcr.microsoft.com/dotnet/core/samples:aspnetapp"
    name: aspnetapp-image
    ports:
    - containerPort: 80
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: test-agic-app-service
spec:
  selector:
    app: test-agic-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-agic-app-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
    - host: test.agic.contoso.com
      http:
        paths:
          - path: /
            backend:
              serviceName: test-agic-app-service
              servicePort: 80
EOF
```

Скопируйте и вставьте все строки из приведенного выше сценария в [Azure Cloud Shell](https://shell.azure.com/). Убедитесь, что скопирована вся команда — начиная с `cat` и включая последнюю `EOF` .

![apply](./media/application-gateway-ingress-controller-troubleshooting/tsg--apply-config.png)

После успешного развертывания приложения, расположенного выше, в кластере AKS будет создан новый модуль Pod, служба и входящий трафик.

Получите список модулей Pod с [Cloud Shell](https://shell.azure.com/): `kubectl get pods -o wide` .
Мы планируем создать Pod с именем Test-агик-App-Pod. Он будет иметь IP-адрес. Этот адрес должен находиться в виртуальной сети шлюза приложений, используемом с AKS.

![Снимок экрана: окно Bash в Azure Cloud Shell показывающее список модулей Pod, включающих Test-агик-App-Tester в списке.](./media/application-gateway-ingress-controller-troubleshooting/tsg--get-pods.png)

Получить список служб: `kubectl get services -o wide` . Мы планируем Просмотреть службу с именем Test-агик-App-Service.

![Снимок экрана: окно Bash в Azure Cloud Shell показывающее список служб, содержащих Test-агик-App-Pod в списке.](./media/application-gateway-ingress-controller-troubleshooting/tsg--get-services.png)

Получите список передает: `kubectl get ingress` . Предполагается, что для входящего ресурса с именем Test-агик-App-Authenticator были созданы. Ресурс будет иметь имя узла "test.agic.contoso.com".

![Снимок экрана: окно Bash в Azure Cloud Shell показывающее список передает, включающих в список тест-агик-App-in.](./media/application-gateway-ingress-controller-troubleshooting/tsg--get-ingress.png)

Один из модулей Pod будет АГИК. `kubectl get pods` отобразит список модулей Pod, один из которых будет начинаться с "входящий трафик — Azure". Получите все журналы этого модуля, `kubectl logs <name-of-ingress-controller-pod>` чтобы убедиться в успешном развертывании. Успешное развертывание добавило в журнал следующие строки:
```
I0927 22:34:51.281437       1 process.go:156] Applied Application Gateway config in 20.461335266s
I0927 22:34:51.281585       1 process.go:165] cache: Updated with latest applied config.
I0927 22:34:51.282342       1 process.go:171] END AppGateway deployment
```

Кроме того, с [Cloud Shell](https://shell.azure.com/) мы можем извлекать только строки, указывающие на успешную настройку шлюза приложений с помощью `kubectl logs <ingress-azure-....> | grep 'Applied App Gateway config in'` , где `<ingress-azure....>` должно быть точным именем Pod агик.

К шлюзу приложений будет применена следующая конфигурация:

- Прослушиватель: ![ прослушиватель](./media/application-gateway-ingress-controller-troubleshooting/tsg--listeners.png)

- Правило маршрутизации: ![ routing_rule](./media/application-gateway-ingress-controller-troubleshooting/tsg--rule.png)

- Внутренний пул:
  - В серверном пуле адресов будет один IP-адрес, который будет соответствовать IP-адресу модуля, который мы наблюдали ранее с `kubectl get pods -o wide` 
 ![ backend_pool](./media/application-gateway-ingress-controller-troubleshooting/tsg--backendpools.png)


Наконец, можно использовать `cURL` команду из [Cloud Shell](https://shell.azure.com/) , чтобы установить HTTP-подключение к только что развернутому приложению:

1. Использование `kubectl get ingress` для получения общедоступного IP-адреса шлюза приложений
2. Используйте `curl -I -H 'test.agic.contoso.com' <publitc-ip-address-from-previous-command>`.

![Снимок экрана: окно Bash в Azure Cloud Shell показывающее команду с фигурным сообщением об успешном подключении к тестовому приложению по протоколу HTTP.](./media/application-gateway-ingress-controller-troubleshooting/tsg--curl.png)

Результат `HTTP/1.1 200 OK` означает, что система "шлюз приложений + AKS + агик" работает должным образом.


## <a name="inspect-kubernetes-installation"></a>Проверка установки Kubernetes

### <a name="pods-services-ingress"></a>Модули Pod, службы, входящие
Контроллер входящего трафика шлюза приложений (АГИК) постоянно отслеживает следующие ресурсы Kubernetes: [развертывание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment) или [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/#what-is-a-pod), [Служба](https://kubernetes.io/docs/concepts/services-networking/service/), [входящий](https://kubernetes.io/docs/concepts/services-networking/ingress/) трафик


Для правильной работы АГИК необходимо следующее:
  1. AKS должен иметь один или несколько работоспособных **модулей**.
     Проверьте, не [Cloud Shell](https://shell.azure.com/) с чем `kubectl get pods -o wide --show-labels` , если у вас есть Pod с `apsnetapp` , выходные данные могут выглядеть следующим образом:
     ```bash
     delyan@Azure:~$ kubectl get pods -o wide --show-labels

     NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE                       NOMINATED NODE   READINESS GATES   LABELS
     aspnetapp              1/1     Running   0          17h   10.0.0.6    aks-agentpool-35064155-1   <none>           <none>            app=aspnetapp
     ```

  2. Одна или несколько **служб**, ссылающихся на модули, приведенные выше, с помощью соответствующих `selector` меток.
     Проверьте, не [Cloud Shell](https://shell.azure.com/) ли это с помощью `kubectl get services -o wide`
     ```bash
     delyan@Azure:~$ kubectl get services -o wide --show-labels

     NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR        LABELS
     aspnetapp           ClusterIP   10.2.63.254    <none>        80/TCP    17h   app=aspnetapp   <none>     
     ```

  3. Входящий **(с** заметкам `kubernetes.io/ingress.class: azure/application-gateway` ), ссылающийся на указанную выше службу проверьте это с [Cloud Shell](https://shell.azure.com/)`kubectl get ingress -o wide --show-labels`
     ```bash
     delyan@Azure:~$ kubectl get ingress -o wide --show-labels

     NAME        HOSTS   ADDRESS   PORTS   AGE   LABELS
     aspnetapp   *                 80      17h   <none>
     ```

  4. Просмотрите аннотации входящего трафика, приведенного выше: `kubectl get ingress aspnetapp -o yaml` (замените `aspnetapp` именем своего входящего трафика).
     ```bash
     delyan@Azure:~$ kubectl get ingress aspnetapp -o yaml

     apiVersion: extensions/v1beta1
     kind: Ingress
     metadata:
       annotations:
         kubernetes.io/ingress.class: azure/application-gateway
       name: aspnetapp
     spec:
       backend:
         serviceName: aspnetapp
         servicePort: 80
     ```

     У ресурса входящих данных должен быть заметка `kubernetes.io/ingress.class: azure/application-gateway` .
 

### <a name="verify-observed-namespace"></a>Проверка наблюдаемого пространства имен

* Получите существующие пространства имен в кластере Kubernetes. В каком пространстве имен выполняется ваше приложение? АГИК ли наблюдение за этим пространством имен? Сведения о правильной настройке наблюдаемых пространств имен см. в документации по [поддержке нескольких пространств имен](./ingress-controller-multiple-namespace-support.md#enable-multiple-namespace-support) .

    ```bash
    # What namespaces exist on your cluster
    kubectl get namespaces
    
    # What pods are currently running
    kubectl get pods --all-namespaces -o wide
    ```


* Модуль АГИК должен находиться в `default` пространстве имен (см. столбец `NAMESPACE` ). Работоспособный Pod будет `Running` в `STATUS` столбце. Должен существовать хотя бы один модуль АГИК.

    ```bash
    # Get a list of the Application Gateway Ingress Controller pods
    kubectl get pods --all-namespaces --selector app=ingress-azure
    ```


* Если модуль АГИК не является работоспособным ( `STATUS` столбец из команды выше не является `Running` ):
  - получите журналы, чтобы понять, почему: `kubectl logs <pod-name>`
  - для предыдущего экземпляра Pod: `kubectl logs <pod-name> --previous`
  - Опишите модуль, чтобы получить дополнительный контекст: `kubectl describe pod <pod-name>`


* У вас есть [Служба](https://kubernetes.io/docs/concepts/services-networking/service/) [Kubernetes и входящие](https://kubernetes.io/docs/concepts/services-networking/ingress/) ресурсы?
    
    ```bash
    # Get all services across all namespaces
    kubectl get service --all-namespaces -o wide
    
    # Get all ingress resources across all namespaces
    kubectl get ingress --all-namespaces -o wide
    ```


* Является ли ваш [входной](https://kubernetes.io/docs/concepts/services-networking/ingress/) код заметкам: `kubernetes.io/ingress.class: azure/application-gateway` ? АГИК будет отслеживать только те ресурсы Kubernetes, которые имеют эту аннотацию.
    
    ```bash
    # Get the YAML definition of a particular ingress resource
    kubectl get ingress --namespace  <which-namespace?>  <which-ingress?>  -o yaml
    ```


* АГИК выдает события Kubernetes для определенных критических ошибок. Вы можете просмотреть следующие данные:
  - в терминале через `kubectl get events --sort-by=.metadata.creationTimestamp`
  - в браузере с помощью [веб-интерфейса Kubernetes (панель мониторинга)](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)


## <a name="logging-levels"></a>Уровни ведения журнала

АГИК имеет 3 уровня ведения журнала. Уровень 1 — это значение по умолчанию, которое показывает минимальное количество строк журнала.
Уровень 5, с другой стороны, отображает все журналы, включая исключенное содержимое конфигурации, применяемое к ARM.

Сообщество Kubernetes установило 9 уровней ведения журнала для средства [kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-output-verbosity-and-debugging) . В этом репозитории используется три из этих трех вариантов с аналогичной семантикой:


| Уровень детализации | Описание |
|-----------|-------------|
|  1        | Уровень ведения журнала по умолчанию; отображаются сведения о запуске, предупреждения и ошибки |
|  3        | Расширенные сведения о событиях и изменениях; списки созданных объектов |
|  5        | Заносит в журнал упакованные объекты; показывает исключенную конфигурацию JSON, примененную к ARM |


Уровни детализации изменяются с помощью `verbosityLevel` переменной в файле [Helm-config. YAML](#sample-helm-config-file) . Увеличьте уровень детализации, чтобы `5` получить конфигурацию JSON, отправленную в [ARM](../azure-resource-manager/management/overview.md):
  - Добавьте `verbosityLevel: 5` строку отдельно в [Helm-config. YAML](#sample-helm-config-file) и повторите установку.
  - получение журналов с помощью `kubectl logs <pod-name>`

### <a name="sample-helm-config-file"></a>Пример файла конфигурации Helm
```yaml
# This file contains the essential configs for the ingress controller helm chart

# Verbosity level of the App Gateway Ingress Controller
verbosityLevel: 3

################################################################################
# Specify which application gateway the ingress controller will manage
#
appgw:
    subscriptionId: <subscriptionId>
    resourceGroup: <resourceGroupName>
    name: <applicationGatewayName>

    # Setting appgw.shared to "true" will create an AzureIngressProhibitedTarget CRD.
    # This prohibits AGIC from applying config for any host/path.
    # Use "kubectl get AzureIngressProhibitedTargets" to view and change this.
    shared: false

################################################################################
# Specify which kubernetes namespace the ingress controller will watch
# Default value is "default"
# Leaving this variable out or setting it to blank or empty string would
# result in Ingress Controller observing all acessible namespaces.
#
# kubernetes:
#   watchNamespace: <namespace>

################################################################################
# Specify the authentication with Azure Resource Manager
#
# Two authentication methods are available:
# - Option 1: AAD-Pod-Identity (https://github.com/Azure/aad-pod-identity)
armAuth:
    type: aadPodIdentity
    identityResourceID: <identityResourceId>
    identityClientID:  <identityClientId>

## Alternatively you can use Service Principal credentials
# armAuth:
#    type: servicePrincipal
#    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --subscription <subscription-uuid> --sdk-auth | base64 -w0" >>

################################################################################
# Specify if the cluster is Kubernetes RBAC enabled or not
rbac:
    enabled: false # true/false

# Specify aks cluster related information. THIS IS BEING DEPRECATED.
aksClusterConfiguration:
    apiServerAddress: <aks-api-server-address>
```
---
title: Настройка Кореднс для службы Kubernetes Azure (AKS)
description: Узнайте, как настроить Кореднс для добавления поддоменов или расширения пользовательских конечных точек DNS с помощью службы Kubernetes Azure (AKS).
services: container-service
author: palma21
ms.topic: article
ms.date: 03/15/2019
ms.author: jpalma
ms.openlocfilehash: 5b13931bc6a13d988c21f728b996c51270769e0c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97368687"
---
# <a name="customize-coredns-with-azure-kubernetes-service"></a>Настройка CoreDNS с использованием Службы Azure Kubernetes

Служба Azure Kubernetes Service (AKS) использует проект [кореднс][coredns] для управления DNS кластера и разрешения для всех кластеров *1.12. x* и более поздних версий. Ранее использовался проект KUBE-DNS. Этот проект KUBE-DNS теперь устарел. Дополнительные сведения о настройке Кореднс и Kubernetes см. в [официальной вышестоящей документации][corednsk8s].

Так как AKS является управляемой службой, вы не можете изменить основную конфигурацию Кореднс ( *корефиле*). Вместо этого используйте Kubernetes *ConfigMap* для переопределения параметров по умолчанию. Чтобы просмотреть AKS Кореднс Конфигмапс по умолчанию, используйте `kubectl get configmaps --namespace=kube-system coredns -o yaml` команду.

В этой статье показано, как использовать Конфигмапс для базовых параметров настройки Кореднс в AKS. Этот подход отличается от настройки Кореднс в других контекстах, таких как использование Корефиле. Проверьте используемую версию Кореднс, так как значения конфигурации могут изменяться в разных версиях.

> [!NOTE]
> `kube-dns` предложены различные [варианты настройки][kubednsblog] через карту конфигурации Kubernetes. Кореднс **не** поддерживает обратную совместимость с KUBE-DNS. Все ранее использовавшиеся настройки должны быть обновлены для использования с Кореднс.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

При создании конфигурации, как показано в приведенных ниже примерах, имена в разделе *данных* должны заканчиваться на *. Server* или *. override*. Это соглашение об именовании определено в AKS Кореднс по умолчанию ConfigMap, которое можно просмотреть с помощью `kubectl get configmaps --namespace=kube-system coredns -o yaml` команды.

## <a name="what-is-supportedunsupported"></a>Поддерживаемые и неподдерживаемые

Поддерживаются все встроенные подключаемые модули Кореднс. Не поддерживаются надстройки и подключаемые модули сторонних производителей.

## <a name="rewrite-dns"></a>Перезапись DNS

Один из сценариев — выполнить перезапись DNS-имени на лету. В следующем примере замените на `<domain to be written>` собственное полное доменное имя. Создайте файл с именем `corednsms.yaml` и вставьте следующий пример конфигурации:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  test.server: | # you may select any name here, but it must end with the .server file extension
    <domain to be rewritten>.com:53 {
        errors
        cache 30
        rewrite name substring <domain to be rewritten>.com default.svc.cluster.local
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        forward .  /etc/resolv.conf # you can redirect this to a specific DNS server such as 10.0.0.10, but that server must be able to resolve the rewritten domain name
    }
```

> [!IMPORTANT]
> При перенаправлении на DNS-сервер, например IP-адрес службы Кореднс, этот DNS-сервер должен иметь возможность разрешить перезапись доменного имени.

Создайте ConfigMap с помощью команды [kubectl Apply ConfigMap][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl apply -f corednsms.yaml
```

Чтобы проверить, применены ли настройки, используйте [kubectl Get конфигмапс][kubectl-get] и укажите *кореднс-Custom* ConfigMap:

```
kubectl get configmaps --namespace=kube-system coredns-custom -o yaml
```

Теперь принудительно перезагрузите ConfigMap Кореднс. Команда [kubectl Delete Pod][kubectl delete] не является обратимой и не приводит к простою. `kube-dns`Удаляются модули, а затем планировщик Kubernetes воссоздает их. Эти новые модули Pod содержат изменение значения TTL.

```console
kubectl delete pod --namespace kube-system -l k8s-app=kube-dns
```

> [!Note]
> Указанная выше команда верна. Пока мы изменим `coredns` , развертывание находится под именем **KUBE-DNS** .

## <a name="custom-forward-server"></a>Пользовательский сервер пересылки

Если необходимо указать сервер пересылки для сетевого трафика, можно создать ConfigMap для настройки DNS. В следующем примере обновите `forward` имя и адрес, указав значения для своей среды. Создайте файл с именем `corednsms.yaml` и вставьте следующий пример конфигурации:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  test.server: | # you may select any name here, but it must end with the .server file extension
    <domain to be rewritten>.com:53 {
        forward foo.com 1.1.1.1
    }
```

Как и в предыдущих примерах, создайте ConfigMap с помощью команды [kubectl Apply ConfigMap][kubectl-apply] и укажите имя манифеста YAML. Затем принудительно Кореднс перезагрузите ConfigMap с помощью [kubectl Delete Pod][kubectl delete] , чтобы планировщик Kubernetes воссоздать их.

```console
kubectl apply -f corednsms.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
```

## <a name="use-custom-domains"></a>Использовать личные домены

Может потребоваться настроить пользовательские домены, которые могут разрешаться только внутри. Например, может потребоваться разрешить личный домен *пуглифе. local*, который не является допустимым доменом верхнего уровня. Без пользовательского домена ConfigMap кластер AKS не сможет разрешить адрес.

В следующем примере измените личный домен и IP-адрес, чтобы направить трафик на значения для вашей среды. Создайте файл с именем `corednsms.yaml` и вставьте следующий пример конфигурации:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  puglife.server: | # you may select any name here, but it must end with the .server file extension
    puglife.local:53 {
        errors
        cache 30
        forward . 192.11.0.1  # this is my test/dev DNS server
    }
```

Как и в предыдущих примерах, создайте ConfigMap с помощью команды [kubectl Apply ConfigMap][kubectl-apply] и укажите имя манифеста YAML. Затем принудительно Кореднс перезагрузите ConfigMap с помощью [kubectl Delete Pod][kubectl delete] , чтобы планировщик Kubernetes воссоздать их.

```console
kubectl apply -f corednsms.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
```

## <a name="stub-domains"></a>Домены-заглушки

Кореднс также можно использовать для настройки доменов-заглушек. В следующем примере обновите личные домены и IP-адреса, указав значения для своей среды. Создайте файл с именем `corednsms.yaml` и вставьте следующий пример конфигурации:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  test.server: | # you may select any name here, but it must end with the .server file extension
    abc.com:53 {
        errors
        cache 30
        forward . 1.2.3.4
    }
    my.cluster.local:53 {
        errors
        cache 30
        forward . 2.3.4.5
    }

```

Как и в предыдущих примерах, создайте ConfigMap с помощью команды [kubectl Apply ConfigMap][kubectl-apply] и укажите имя манифеста YAML. Затем принудительно Кореднс перезагрузите ConfigMap с помощью [kubectl Delete Pod][kubectl delete] , чтобы планировщик Kubernetes воссоздать их.

```console
kubectl apply -f corednsms.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
```

## <a name="hosts-plugin"></a>Подключаемый модуль узлов

Так как поддерживаются все встроенные подключаемые модули, это означает, что также можно настроить подключаемый модуль Кореднс [hosts][coredns hosts] .

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom # this is the name of the configmap you can overwrite with your changes
  namespace: kube-system
data:
    test.override: | # you may select any name here, but it must end with the .override file extension
          hosts example.hosts example.org { # example.hosts must be a file
              10.0.0.1 example.org
              fallthrough
          }
```

## <a name="enable-logging-for-dns-query-debugging"></a>Включение ведения журнала для отладки запросов DNS 

Чтобы включить ведение журнала запросов DNS, примените следующую конфигурацию в кореднс-Custom ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  log.override: | # you may select any name here, but it must end with the .override file extension
        log
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье показано несколько примеров сценариев для настройки Кореднс. Сведения о проекте Кореднс см. [на странице вышестоящего проекта кореднс][coredns].

Дополнительные сведения о основных понятиях сети см. [в разделе Основные понятия сети для приложений в AKS][concepts-network].

<!-- LINKS - external -->
[kubednsblog]: https://www.danielstechblog.io/using-custom-dns-server-for-domain-specific-name-resolution-with-azure-kubernetes-service/
[coredns]: https://coredns.io/
[corednsk8s]: https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/#coredns
[dnscache]: https://coredns.io/plugins/cache/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[coredns hosts]: https://coredns.io/plugins/hosts/

<!-- LINKS - internal -->
[concepts-network]: concepts-network.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md

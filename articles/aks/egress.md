---
title: Использовать статический IP-адрес для исходящего трафика
titleSuffix: Azure Kubernetes Service
description: Узнайте, как создать и использовать статический общедоступный IP-адрес для исходящего трафика в кластере Службы Azure Kubernetes (AKS).
services: container-service
ms.topic: article
ms.date: 03/16/2021
ms.openlocfilehash: e1f81bf4c4d35108557449a8bebd126bdf744191
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104592376"
---
# <a name="use-a-static-public-ip-address-for-egress-traffic-with-a-basic-sku-load-balancer-in-azure-kubernetes-service-aks"></a>Использовать статический общедоступный IP-адрес для исходящего трафика с *базовой* подсистемой балансировки нагрузки в службе Kubernetes Azure (AKS)

По умолчанию IP-адрес исходящего трафика из кластера Службы Azure Kubernetes (AKS) назначается случайным образом. Эта конфигурация не обеспечивает точность, например, когда необходимо идентифицировать IP-адрес для доступа ко внешним службам. Вместо этого может потребоваться добавить статический IP-адрес в разрешенных для доступа к службе.

Узнайте, как создать и использовать статический общедоступный IP-адрес для исходящего трафика в кластере AKS.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что вы используете базовый Load Balancer Azure.  Мы рекомендуем использовать [Load Balancer (цен. Категория "Стандартный") Azure](../load-balancer/load-balancer-overview.md), а также более широкие возможности для [управления трафиком исходящего трафика AKS](./limit-egress-traffic.md).

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Кроме того, нужно установить и настроить Azure CLI версии 2.0.59 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

> [!IMPORTANT]
> В этой статье используется балансировщик нагрузки *базового* SKU с одним пулом узлов. Эта конфигурация недоступна для нескольких пулов узлов, так как балансировщик нагрузки *базового* SKU не поддерживается для нескольких пулов узлов. Дополнительные сведения об использовании балансировщика нагрузки уровня " *стандартный* " см. в статье [использование общедоступного Load Balancer (цен. Категория "Стандартный") в службе Azure Kubernetes (AKS)][slb] .

## <a name="egress-traffic-overview"></a>Обзор исходящего трафика

Исходящий трафик из кластера AKS регулируется на основе [соглашений Azure Load Balancer][outbound-connections]. До создания первой службы Kubernetes типа `LoadBalancer` узлы агента в кластере AKS не являются частью любого пула Azure Load Balancer. В этой конфигурации у узлов нет общедоступного IP-адреса уровня экземпляра. Azure преобразует исходящий поток в общедоступный исходный IP-адрес, который не является настраиваемым или детерминированным.

После создания службы Kubernetes типа `LoadBalancer` узлы агента добавляются в пул Azure Load Balancer. Если есть несколько интерфейсов (общедоступных) IP-адресов, которые можно использовать для исходящих потоков, Load Balancer категории "Базовый" выберет один интерфейс. Выбор нельзя настроить, а его алгоритм следует рассматривать как случайный. Этот общедоступный IP-адрес действует только на протяжении существования этого ресурса. Если вы удалите службу LoadBalancer Kubernetes, связанные с ней подсистема балансировки нагрузки и IP-адрес также будут удалены. Если вы хотите назначить определенный IP-адрес или сохранить IP-адрес для повторно развернутых служб Kubernetes, можно создать и использовать статический общедоступный IP-адрес.

## <a name="create-a-static-public-ip"></a>Создание статического общедоступного IP-адреса

Получите имя группы ресурсов, выполнив команду [az aks show][az-aks-show] и добавив параметр запроса `--query nodeResourceGroup`. В следующем примере возвращается группа ресурсов узла для имени кластера AKS *myAKSCluster* в группе ресурсов *myResourceGroup*.

```azurecli-interactive
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

Затем выполните команду [az network public-ip create][az-network-public-ip-create], чтобы создать статический общедоступный IP-адрес. Укажите имя группы ресурсов узла, полученное в предыдущей команде, а затем имя ресурса IP-адреса, например *myAKSPublicIP*:

```azurecli-interactive
az network public-ip create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --name myAKSPublicIP \
    --allocation-method static
```

Отобразится IP-адрес, как показано в следующем сжатом примере выходных данных:

```json
{
  "publicIp": {
    "dnsSettings": null,
    "etag": "W/\"6b6fb15c-5281-4f64-b332-8f68f46e1358\"",
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Network/publicIPAddresses/myAKSPublicIP",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "40.121.183.52",
    [..]
  }
```

Позже можно получить общедоступный IP-адрес с помощью команды [az network public-ip list][az-network-public-ip-list]. Укажите имя группы ресурсов узла, а затем запросите *ipAddress*, как показано в следующем примере:

```azurecli-interactive
$ az network public-ip list --resource-group MC_myResourceGroup_myAKSCluster_eastus --query [0].ipAddress --output tsv

40.121.183.52
```

## <a name="create-a-service-with-the-static-ip"></a>Создание службы со статическим IP-адресом

Чтобы создать службу со статическим общедоступным IP-адресом, добавьте свойство `loadBalancerIP` и значение статического общедоступного IP-адреса в манифест YAML. Создайте файл `egress-service.yaml` и скопируйте в него следующий код YAML. Укажите собственный общедоступный IP-адрес, созданный на предыдущем шаге.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-egress
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
```

Выполните команду `kubectl apply` для создания службы и развертывания.

```console
kubectl apply -f egress-service.yaml
```

Эта служба настраивает новый интерфейсный IP-адрес в Azure Load Balancer. Если у вас нет других настроенных IP-адресов, тогда этот адрес должен использоваться **всем** исходящим трафиком. Если на Azure Load Balancer настроено несколько адресов, любой из этих общедоступных IP-адресов является кандидатом для исходящих потоков, и один из них выбирается случайным образом.

## <a name="verify-egress-address"></a>Проверка адреса исходящего трафика

Проверить, что используется статический общедоступный IP-адрес, можно с помощью службы поиска DNS, такой как `checkip.dyndns.org`.

Выполните запуск и подключитесь к базовому *модулю Debian*:

```console
kubectl run -it --rm aks-ip --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11
```

Чтобы получить доступ к веб-сайту в контейнере, используйте `apt-get` для установки `curl` в контейнер.

```console
apt-get update && apt-get install curl -y
```

Теперь используйте curl для доступа к сайту *checkip.dyndns.org*. Отобразится IP-адрес исходящего трафика, как показано в следующем примере выходных данных: Этот IP-адрес соответствует статическому общедоступному IP-адресу, созданному и определенному для службы loadBalancer:

```console
$ curl -s checkip.dyndns.org

<html><head><title>Current IP Check</title></head><body>Current IP Address: 40.121.183.52</body></html>
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы избежать поддержки нескольких общедоступных IP-адресов в Azure Load Balancer, можно использовать контроллер входящего трафика. Контроллеры входящего трафика обеспечивают такие дополнительные преимущества, как обработка подключений SSL и TLS, поддержка повторных записей URI и восходящее шифрование протоколов SSL и TLS. Дополнительные сведения см. в статье [Создание контроллера входящего трафика в Службе Azure Kubernetes (AKS)][ingress-aks-cluster].

<!-- LINKS - internal -->
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[az-aks-show]: /cli/azure/aks#az-aks-show
[azure-cli-install]: /cli/azure/install-azure-cli
[ingress-aks-cluster]: ./ingress-basic.md
[outbound-connections]: ../load-balancer/load-balancer-outbound-connections.md#scenarios
[public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[slb]: load-balancer-standard.md

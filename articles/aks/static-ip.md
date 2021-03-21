---
title: Использовать статический IP-адрес с подсистемой балансировки нагрузки
titleSuffix: Azure Kubernetes Service
description: Сведения о создании и использовании статического IP-адреса с подсистемой балансировки нагрузки Службы Azure Kubernetes (AKS).
services: container-service
ms.topic: article
ms.date: 11/14/2020
ms.openlocfilehash: 102df48ca22fb996e0f4d9c402b8ce8f0fa80f2c
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102509478"
---
# <a name="use-a-static-public-ip-address-and-dns-label-with-the-azure-kubernetes-service-aks-load-balancer"></a>Использование статического общедоступного IP-адреса и метки DNS с подсистемой балансировки нагрузки Azure Kubernetes Service (AKS)

По умолчанию общедоступный IP-адрес, назначенный ресурсу подсистемы балансировки нагрузки, созданному кластером AKS, действителен только на протяжении существования этого ресурса. Если вы удалите службу Kubernetes, связанные с ней подсистема балансировки нагрузки и IP-адрес также будут удалены. Если вы хотите назначить определенный IP-адрес или сохранить IP-адрес для повторно развернутых служб Kubernetes, можно создать и использовать статический общедоступный IP-адрес.

В этой статье описывается создание статического общедоступного IP-адреса и его назначение службе Kubernetes.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Кроме того, нужно установить и настроить Azure CLI версии 2.0.59 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

В этой статье рассматривается использование *стандартного* IP-адреса SKU с подсистемой балансировки нагрузки *стандартного* SKU. Дополнительные сведения см. в статье [Типы IP-адресов и методы их распределения в Azure][ip-sku].

## <a name="create-a-static-ip-address"></a>Создание статического IP-адреса

Создайте статический общедоступный IP-адрес с помощью команды [AZ Network public IP Create][az-network-public-ip-create] . В следующем примере создается ресурс статического IP-адреса с именем *мякспублиЦип* в группе ресурсов *myResourceGroup* :

```azurecli-interactive
az network public-ip create \
    --resource-group myResourceGroup \
    --name myAKSPublicIP \
    --sku Standard \
    --allocation-method static
```

> [!NOTE]
> При использовании подсистемы балансировки нагрузки уровня " *базовый* " в кластере AKS используйте параметр " *базовый* " для параметра *SKU* при определении общедоступного IP-адреса. Только *основные* IP-адреса SKU работают с подсистемой балансировки нагрузки уровня " *базовый* " и поддерживаются только *стандартные* IP-адреса SKU с подсистемами балансировки нагрузки " *стандартный* " 

Отобразится IP-адрес, как показано в приведенном ниже примере вывода с уплотнением:

```json
{
  "publicIp": {
    ...
    "ipAddress": "40.121.183.52",
    ...
  }
}
```

Позже можно получить общедоступный IP-адрес с помощью команды [az network public-ip list][az-network-public-ip-list]. Укажите имя группы ресурсов узла и созданный вами общедоступный IP-адрес, а затем запросите *ipAddress*, как показано в следующем примере:

```azurecli-interactive
$ az network public-ip show --resource-group myResourceGroup --name myAKSPublicIP --query ipAddress --output tsv

40.121.183.52
```

## <a name="create-a-service-using-the-static-ip-address"></a>Создание службы со статическим IP-адресом

Перед созданием службы убедитесь, что удостоверение кластера, используемое кластером AKS, имеет делегированные разрешения на доступ к другой группе ресурсов. Пример:

```azurecli-interactive
az role assignment create \
    --assignee <Client ID> \
    --role "Network Contributor" \
    --scope /subscriptions/<subscription id>/resourceGroups/<resource group name>
```

> [!IMPORTANT]
> Если вы настроили исходящий IP-адрес, убедитесь, что у удостоверения кластера есть разрешения на исходящий общедоступный IP-адрес и этот входящий общедоступный IP-адрес

Чтобы создать службу *балансировки нагрузки* со статическим общедоступным IP-адресом, добавьте `loadBalancerIP` свойство и значение статического общедоступного IP-адреса в манифест YAML. Создайте файл `load-balancer-service.yaml` и скопируйте в него следующий код YAML. Укажите собственный общедоступный IP-адрес, созданный на предыдущем шаге. В следующем примере также задается заметка для группы ресурсов с именем *myResourceGroup*. Укажите собственное имя группы ресурсов.

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-resource-group: myResourceGroup
  name: azure-load-balancer
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-load-balancer
```

Выполните команду `kubectl apply` для создания службы и развертывания.

```console
kubectl apply -f load-balancer-service.yaml
```

## <a name="apply-a-dns-label-to-the-service"></a>Применение метки DNS к службе

Если служба использует динамический или статический общедоступный IP-адрес, можно использовать заметку службы `service.beta.kubernetes.io/azure-dns-label-name` для задания общедоступной метки DNS. При этом будет опубликовано полное доменное имя службы с помощью общедоступных DNS-серверов Azure и домена верхнего уровня. Значение аннотации должно быть уникальным в расположении Azure, поэтому рекомендуется использовать достаточно полную метку.   

Azure автоматически добавит подсеть по умолчанию, например `<location>.cloudapp.azure.com` (где расположение — выбранный регион), в имя, которое вы задаете, чтобы создать полное DNS-имя. Пример:

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/azure-dns-label-name: myserviceuniquelabel
  name: azure-load-balancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-load-balancer
```

> [!NOTE] 
> Сведения о публикации службы в собственном домене см. в разделе [Azure DNS][azure-dns-zone] и проект [External-DNS][external-dns] .

## <a name="troubleshoot"></a>Диагностика

Если статический IP-адрес, определенный в свойстве *лоадбаланцерип* манифеста службы Kubernetes, не существует или не был создан в группе ресурсов узла и не настроены дополнительные делегирования, то создание службы балансировщика нагрузки завершится ошибкой. Для устранения неполадок просмотрите события создания службы, выполнив команду [kubectl describe][kubectl-describe]. Введите имя службы, указанное в манифесте YAML, как показано в следующем примере:

```console
kubectl describe service azure-load-balancer
```

Отобразятся сведения о ресурсе службы Kubernetes. В разделе *Events* в конце выходных данных в следующем примере содержатся сведения о том, что *указанный пользователем IP-адрес найти не удалось*. В этих сценариях необходимо убедиться, что статический общедоступный IP-адрес создан в группе ресурсов узла и что в манифесте службы Kubernetes указан правильный IP-адрес.

```
Name:                     azure-load-balancer
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=azure-load-balancer
Type:                     LoadBalancer
IP:                       10.0.18.125
IP:                       40.121.183.52
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32582/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type     Reason                      Age               From                Message
  ----     ------                      ----              ----                -------
  Normal   CreatingLoadBalancer        7s (x2 over 22s)  service-controller  Creating load balancer
  Warning  CreatingLoadBalancerFailed  6s (x2 over 12s)  service-controller  Error creating load balancer (will retry): Failed to create load balancer for service default/azure-load-balancer: user supplied IP Address 40.121.183.52 was not found
```

## <a name="next-steps"></a>Дальнейшие действия

Для дополнительного управления входящим трафиком приложений можно [создать контроллер входящего трафика][aks-ingress-basic]. Вы также можете [создать контроллер входящего трафика со статическим общедоступным IP-адресом][aks-static-ingress].

<!-- LINKS - External -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[azure-dns-zone]: https://azure.microsoft.com/services/dns/
[external-dns]: https://github.com/kubernetes-sigs/external-dns

<!-- LINKS - Internal -->
[aks-faq-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[az-aks-show]: /cli/azure/aks#az-aks-show
[aks-ingress-basic]: ingress-basic.md
[aks-static-ingress]: ingress-static-ip.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[ip-sku]: ../virtual-network/public-ip-addresses.md#sku

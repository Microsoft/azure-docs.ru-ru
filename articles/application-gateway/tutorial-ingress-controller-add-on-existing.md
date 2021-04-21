---
title: Руководство по Включение надстройки контроллера объекта ingress для имеющегося кластера AKS со Шлюзом приложений Azure
description: В этом учебнике описано, как включить надстройку контроллера объекта ingress для имеющегося кластера AKS со Шлюзом приложений Azure.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: tutorial
ms.date: 03/02/2021
ms.author: caya
ms.openlocfilehash: 2fe615da256099c3135f607a7b6f8095bb93b442
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772853"
---
# <a name="tutorial-enable-application-gateway-ingress-controller-add-on-for-an-existing-aks-cluster-with-an-existing-application-gateway"></a>Руководство по включению надстройки контроллера объекта ingress для имеющегося кластера AKS со Шлюзом приложений

Вы можете использовать Azure CLI или портал, чтобы включить надстройку [контроллера объекта ingress Шлюза приложений (AGIC)](ingress-controller-overview.md) для существующего кластера [Службы Azure Kubernetes (AKS)](https://azure.microsoft.com/services/kubernetes-service/). В этом учебнике вы узнаете, как использовать надстройку AGIC, чтобы предоставить приложение Kubernetes в имеющемся кластере AKS с помощью Шлюза приложений, развернутого в разных виртуальных сетях. Начнем с создания кластера AKS в одной виртуальной сети и Шлюза приложений в отдельной виртуальной сети для имитации существующих ресурсов. Затем вы включите надстройку AGIC, создадите пиринговое подключение между двумя виртуальными сетями и развернете пример приложения, которое будет предоставляться через Шлюз приложений с помощью надстройки AGIC. Если вы включаете надстройку AGIC для имеющегося Шлюза приложений и кластера AKS в той же виртуальной сети, можно пропустить шаг создания пирингового подключения ниже. Надстройка обеспечивает более быстрый способ развертывания AGIC для кластера AKS, чем [при использовании Helm](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on). Она также предоставляет полностью управляемый интерфейс.  

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание группы ресурсов 
> * Создание кластера AKS. 
> * Создание Шлюза приложений. 
> * Включение надстройки AGIC в существующем кластере AKS с помощью Azure CLI 
> * Включение надстройки AGIC в существующем кластере AKS с помощью портала 
> * Создание пирингового подключения между виртуальной сетью Шлюза приложений и виртуальной сетью кластера AKS.
> * Развертывание примера приложения с использованием AGIC для объекта ingress в кластере AKS.
> * Проверка доступности приложения через Шлюз приложений.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="create-a-resource-group"></a>Создание группы ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Создайте группу ресурсов, используя команду [az group create](/cli/azure/group#az_group_create). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении (регионе) *CanadaCentral*. 

```azurecli-interactive
az group create --name myResourceGroup --location canadacentral
```

## <a name="deploy-a-new-aks-cluster"></a>Развертывание нового кластера AKS

Теперь можно развернуть новый кластер AKS, чтобы имитировать наличие имеющегося кластера AKS, для которого требуется включить надстройку AGIC.  

В следующем примере вы развернете новый кластер AKS с именем *myCluster*, используя [Azure CNI](../aks/concepts-network.md#azure-cni-advanced-networking) и [управляемые удостоверения](../aks/use-managed-identity.md) в созданной группе ресурсов *myResourceGroup*.

```azurecli-interactive
az aks create -n myCluster -g myResourceGroup --network-plugin azure --enable-managed-identity 
```

Информацию для настройки дополнительных параметров команды `az aks create` см. по [этим ссылкам](/cli/azure/aks#az_aks_create). 

## <a name="deploy-a-new-application-gateway"></a>Развертывание нового Шлюза приложений 

Теперь вы развернете новый шлюз приложений, чтобы имитировать наличие имеющегося Шлюза приложений, который будет использоваться для балансировки нагрузки трафика в кластере AKS (*myCluster*). Имя шлюза приложений будет *myApplicationGateway*, но необходимо сначала создать ресурс общедоступного IP-адреса с именем *myPublicIp* и новую виртуальную сеть с именем *myVnet* и диапазоном IP-адресов 11.0.0.0/8. Затем следует создать подсеть с диапазоном IP-адресов 11.1.0.0/16 с именем *mySubnet* и развернуть шлюз приложений в *mySubnet* с помощью команды *myPublicIp*. 

При использовании кластера AKS и Шлюза приложений в отдельных виртуальных сетях диапазоны IP-адресов двух виртуальных сетей не должны перекрываться. Диапазон IP-адресов по умолчанию, в котором развертывается кластер AKS, — 10.0.0.0/8, поэтому мы устанавливаем префикс адреса виртуальной сети шлюза приложений — 11.0.0.0/8. 

```azurecli-interactive
az network public-ip create -n myPublicIp -g MyResourceGroup --allocation-method Static --sku Standard
az network vnet create -n myVnet -g myResourceGroup --address-prefix 11.0.0.0/8 --subnet-name mySubnet --subnet-prefix 11.1.0.0/16 
az network application-gateway create -n myApplicationGateway -l canadacentral -g myResourceGroup --sku Standard_v2 --public-ip-address myPublicIp --vnet-name myVnet --subnet mySubnet
```

> [!NOTE]
> Надстройка контроллера объекта ingress (AGIC) Шлюза приложений поддерживает **только** номера SKU Шлюза приложений версии 2 (Стандартный и WAF), а **не** версии 1. 

## <a name="enable-the-agic-add-on-in-existing-aks-cluster-through-azure-cli"></a>Включение надстройки AGIC в существующем кластере AKS с помощью Azure CLI 

Если вы хотите продолжить использование Azure CLI, можете включить надстройку AGIC в созданном кластере AKS (*myCluster*) и указать надстройку AGIC, чтобы использовать созданный Шлюз приложений (*myApplicationGateway*).

```azurecli-interactive
appgwId=$(az network application-gateway show -n myApplicationGateway -g myResourceGroup -o tsv --query "id") 
az aks enable-addons -n myCluster -g myResourceGroup -a ingress-appgw --appgw-id $appgwId
```

## <a name="enable-the-agic-add-on-in-existing-aks-cluster-through-portal"></a>Включение надстройки AGIC в существующем кластере AKS с помощью портала 

Если вы хотите использовать портал Azure для включения надстройки AGIC, перейдите по адресу [(https://aka.ms/azure/portal/aks/agic)](https://aka.ms/azure/portal/aks/agic) и перейдите к своему кластеру AKS через ссылку на портале. После этого перейдите на вкладку "Сети" в кластере AKS. Вы увидите раздел "Application Gateway ingress controller" (Контроллер объекта ingress Шлюза приложений), который позволяет включить или отключить надстройку контроллера объекта ingress с помощью пользовательского интерфейса портала. Установите флажок рядом с пунктом "Enable ingress controller" (Включить контроллер объекта ingress) и выберите созданный Шлюз приложений *myApplicationGateway* в раскрывающемся меню. 

![Портал контроллера объекта ingress Шлюза приложений](./media/tutorial-ingress-controller-add-on-existing/portal-ingress-controller-add-on.png)

## <a name="peer-the-two-virtual-networks-together&quot;></a>Создание пирингового подключения между двумя виртуальными сетями

Так как мы развернули кластер AKS в своей виртуальной сети, а шлюз приложений в другой, необходимо объединить эти две виртуальные сети, чтобы трафик перенаправлялся из Шлюза приложений в модули Pod кластера. Для пирингового подключения между двумя виртуальными сетями требуется выполнить команду Azure CLI два раза, чтобы обеспечить двустороннюю связь. Первая команда создаст пиринговое подключение из виртуальной сети Шлюза приложений к виртуальной сети AKS. Вторая команда создаст пиринговое подключение в обратном направлении.

```azurecli-interactive
nodeResourceGroup=$(az aks show -n myCluster -g myResourceGroup -o tsv --query &quot;nodeResourceGroup")
aksVnetName=$(az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name")

aksVnetId=$(az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id")
az network vnet peering create -n AppGWtoAKSVnetPeering -g myResourceGroup --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access

appGWVnetId=$(az network vnet show -n myVnet -g myResourceGroup -o tsv --query "id")
az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access
```

## <a name="deploy-a-sample-application-using-agic"></a>Развертывание примера приложения с помощью AGIC 

Теперь можно развернуть пример приложения в созданном кластере AKS, который будет использовать надстройку AGIC для объекта ingress, и подключить Шлюз приложений к кластеру AKS. Сначала получите учетные данные для развернутого кластера AKS с помощью команды `az aks get-credentials`. 

```azurecli-interactive
az aks get-credentials -n myCluster -g myResourceGroup
```

Теперь, когда у вас есть учетные данные для созданного кластера, выполните указанную ниже команду, чтобы настроить пример приложения, использующего AGIC для ingress в кластере. AGIC обновит Шлюз приложений, настроенный вами ранее с соответствующими правилами маршрутизации для нового развертываемого примера приложения.  

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml 
```

## <a name="check-that-the-application-is-reachable"></a>Проверка доступности приложения

Теперь, когда Шлюз приложений настроен для обслуживания трафика в кластере AKS, давайте проверим доступность приложения. Сначала получите IP-адрес объекта ingress. 

```azurecli-interactive
kubectl get ingress
```

Убедитесь, что созданный пример приложения запущен и работает. Для этого посетите IP-адрес шлюза приложений, полученный при выполнении указанной выше команды, либо используйте `curl`. Для обновления Шлюзу приложений может потребоваться около минуты. Поэтому, если шлюз приложений по-прежнему находится в состоянии обновления на портале, дождитесь завершения, прежде чем пытаться связаться с IP-адресом. 

## <a name="clean-up-resources"></a>Очистка ресурсов

При необходимости вы можете удалить группу ресурсов, шлюз приложений и все связанные ресурсы.

```azurecli-interactive
az group delete --name myResourceGroup 
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения об отключении надстройки AGIC для кластера AKS](./ingress-controller-disable-addon.md)

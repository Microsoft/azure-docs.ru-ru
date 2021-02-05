---
title: Руководство по Включение надстройки контроллера объекта ingress для имеющегося кластера AKS со Шлюзом приложений Azure
description: В этом учебнике описано, как включить надстройку контроллера объекта ingress для имеющегося кластера AKS со Шлюзом приложений Azure.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: tutorial
ms.date: 09/24/2020
ms.author: caya
ms.openlocfilehash: d491b714c7d553fbd89d72315f46e6927d437717
ms.sourcegitcommit: f377ba5ebd431e8c3579445ff588da664b00b36b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/05/2021
ms.locfileid: "99593824"
---
# <a name="tutorial-enable-application-gateway-ingress-controller-add-on-for-an-existing-aks-cluster-with-an-existing-application-gateway-through-azure-cli-preview"></a>Руководство по включению надстройки контроллера объекта ingress для имеющегося кластера AKS со Шлюзом приложений с помощью Azure CLI (предварительная версия)

Используйте Azure CLI, чтобы включить надстройку [контроллера объекта ingress Шлюза приложений (AGIC)](ingress-controller-overview.md) (предварительная версия) для кластера [службы Azure Kubernetes (AKS)](https://azure.microsoft.com/services/kubernetes-service/). В этом учебнике вы узнаете, как использовать надстройку AGIC, чтобы предоставить приложение Kubernetes в имеющемся кластере AKS с помощью Шлюза приложений, развернутого в разных виртуальных сетях. Начнем с создания кластера AKS в одной виртуальной сети и Шлюза приложений в отдельной виртуальной сети для имитации существующих ресурсов. Затем включите надстройку AGIC, создайте пиринговое подключение между двумя виртуальными сетями и разверните пример приложения, которое будет предоставляться через Шлюз приложений с помощью надстройки AGIC. Если вы включаете надстройку AGIC для имеющегося Шлюза приложений и кластера AKS в той же виртуальной сети, можно пропустить шаг создания пирингового подключения ниже. Надстройка обеспечивает более быстрый способ развертывания AGIC для кластера AKS, чем [при использовании Helm](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on). Она также предоставляет полностью управляемый интерфейс.  

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание группы ресурсов 
> * Создание кластера AKS. 
> * Создание Шлюза приложений. 
> * Включение надстройки AGIC в имеющемся кластере AKS с помощью Шлюза приложений. 
> * Создание пирингового подключения между виртуальной сетью Шлюза приложений и виртуальной сетью кластера AKS.
> * Развертывание примера приложения с использованием AGIC для объекта ingress в кластере AKS.
> * Проверка доступности приложения через Шлюз приложений.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим учебником требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

 - Зарегистрируйте флаг функции *AKS-IngressApplicationGatewayAddon* с помощью команды [az feature register](/cli/azure/feature#az-feature-register), как показано в указанном ниже примере. Это необходимо сделать только один раз для каждой подписки, пока надстройка еще находится на этапе предварительной версии.
     ```azurecli-interactive
     az feature register --name AKS-IngressApplicationGatewayAddon --namespace microsoft.containerservice
     ```
    Через несколько минут отобразится состояние "Зарегистрировано". Состояние регистрации можно проверить с помощью команды [az feature list](/cli/azure/feature#az-feature-register).
     ```azurecli-interactive
     az feature list -o table --query "[?contains(name, 'microsoft.containerservice/AKS-IngressApplicationGatewayAddon')].{Name:name,State:properties.state}"
     ```

 - Когда все будет готово, обновите регистрацию поставщика ресурсов Microsoft.ContainerService с помощью команды [az provider register](/cli/azure/provider#az-provider-register).
    ```azurecli-interactive
    az provider register --namespace Microsoft.ContainerService
    ```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Создайте группу ресурсов, используя команду [az group create](/cli/azure/group#az-group-create). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении (регионе) *CanadaCentral*. 

```azurecli-interactive
az group create --name myResourceGroup --location canadacentral
```

## <a name="deploy-a-new-aks-cluster"></a>Развертывание нового кластера AKS

Теперь можно развернуть новый кластер AKS, чтобы имитировать наличие имеющегося кластера AKS, для которого требуется включить надстройку AGIC.  

В следующем примере вы развернете новый кластер AKS с именем *myCluster*, используя [Azure CNI](../aks/concepts-network.md#azure-cni-advanced-networking) и [управляемые удостоверения](../aks/use-managed-identity.md) в созданной группе ресурсов *myResourceGroup*.    

```azurecli-interactive
az aks create -n myCluster -g myResourceGroup --network-plugin azure --enable-managed-identity 
```

Информацию для настройки дополнительных параметров команды `az aks create` см. по [этим ссылкам](/cli/azure/aks#az-aks-create). 

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

## <a name="enable-the-agic-add-on-in-existing-aks-cluster-with-existing-application-gateway"></a>Включение надстройки AGIC в имеющемся кластере AKS с помощью Шлюза приложений 

Теперь включите надстройку AGIC в созданном кластере AKS (*myCluster*) и укажите надстройку AGIC, чтобы использовать созданный Шлюз приложений (*myApplicationGateway*). Убедитесь, что вы добавили или обновили расширение aks-preview в начале этого учебника. 

```azurecli-interactive
appgwId=$(az network application-gateway show -n myApplicationGateway -g myResourceGroup -o tsv --query "id") 
az aks enable-addons -n myCluster -g myResourceGroup -a ingress-appgw --appgw-id $appgwId
```

## <a name="peer-the-two-virtual-networks-together"></a>Создание пирингового подключения между двумя виртуальными сетями

Так как мы развернули кластер AKS в своей виртуальной сети, а шлюз приложений в другой, необходимо объединить эти две виртуальные сети, чтобы трафик перенаправлялся из Шлюза приложений в модули Pod кластера. Для пирингового подключения между двумя виртуальными сетями требуется выполнить команду Azure CLI два раза, чтобы обеспечить двустороннюю связь. Первая команда создаст пиринговое подключение из виртуальной сети Шлюза приложений к виртуальной сети AKS. Вторая команда создаст пиринговое подключение в обратном направлении. 

```azurecli-interactive
nodeResourceGroup=$(az aks show -n myCluster -g myResourceGroup -o tsv --query "nodeResourceGroup")
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

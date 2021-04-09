---
title: Руководство по включению надстройки контроллера объекта ingress для нового кластера AKS с новым Шлюзом приложений Azure
description: Узнайте, как включить надстройку контроллера объекта ingress для нового кластера AKS с новым экземпляром Шлюза приложений Azure.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: tutorial
ms.date: 03/02/2021
ms.author: caya
ms.openlocfilehash: c37168c5165f5402dd4f57c8557bc2b7b3603533
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101720194"
---
# <a name="tutorial-enable-the-ingress-controller-add-on-for-a-new-aks-cluster-with-a-new-application-gateway-instance"></a>Руководство по включению надстройки контроллера объекта ingress для нового кластера AKS с новым экземпляром Шлюза приложений

Вы можете использовать Azure CLI, чтобы включить надстройку [контроллера объекта ingress Шлюза приложений (AGIC)](ingress-controller-overview.md) для нового кластера [Службы Azure Kubernetes (AKS)](https://azure.microsoft.com/services/kubernetes-service/).

В этом учебнике описано, как создавать кластер AKS с включенной надстройкой AGIC. При создании кластера автоматически создается используемый экземпляр Шлюза приложений Azure. Затем вы развернете пример приложения, которое будет использовать надстройку для предоставления доступа к приложению через Шлюз приложений. 

Надстройка обеспечивает более быстрый способ развертывания AGIC для кластера AKS, чем [при использовании Helm](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on). Она также предоставляет полностью управляемый интерфейс.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создайте группу ресурсов. 
> * Создайте кластер AKS с включенной надстройкой AGIC.
> * Разверните пример приложения с помощью AGIC для объекта ingress в кластере AKS.
> * Убедитесь, что приложение доступно через Шлюз приложений.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="create-a-resource-group"></a>Создание группы ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Создайте группу ресурсов, используя команду [az group create](/cli/azure/group#az-group-create). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении (регионе) *CanadaCentral*: 

```azurecli-interactive
az group create --name myResourceGroup --location canadacentral
```

## <a name="deploy-an-aks-cluster-with-the-add-on-enabled"></a>Развертывание кластера AKS с включенной надстройкой

Теперь разверните кластер AKS с включенной надстройкой AGIC. Если вы не указали существующий экземпляр Шлюза приложений для использования в этом процессе, мы автоматически создадим и настроим экземпляр Шлюза приложений для обслуживания трафика в кластере AKS.  

> [!NOTE]
> Надстройка контроллера объекта ingress Шлюза приложений поддерживает *только* номер SKU Шлюза приложений версии 2 (Стандартный и WAF), а *не* версии 1. При развертывании нового экземпляра Шлюза приложений через надстройку AGIC вы можете развернуть только Шлюз приложений для SKU Standard_v2. Если вы хотите включить надстройку для WAF в SKU версии 2 Шлюза приложения, используйте один из следующих методов:
>
> - На портале включите WAF в Шлюзе приложений. 
> - Сначала создайте WAF версии 2 экземпляра Шлюза приложений, а затем следуйте инструкциям по [включению надстройки AGIC с существующим кластером AKS и экземпляром Шлюза приложений](tutorial-ingress-controller-add-on-existing.md). 

В следующем примере вы развернете новый кластер AKS с именем *myCluster* с помощью [Azure CNI](../aks/concepts-network.md#azure-cni-advanced-networking) и [управляемых удостоверений](../aks/use-managed-identity.md). Надстройка AGIC будет включена в созданной вами группе ресурсов *myResourceGroup*. 

Развертывание нового кластера AKS с включенной надстройкой AGIC без указания существующего экземпляра Шлюза приложений приведет к автоматическому созданию экземпляра SKU уровня "Стандартный" версии 2 экземпляра Шлюза приложений. Поэтому потребуется указать имя и диапазон адресов подсети экземпляра Шлюза приложений. Имя экземпляра Шлюза приложений будет *myApplicationGateway*, а используемый диапазон адресов подсети — 10.2.0.0/16.

```azurecli-interactive
az aks create -n myCluster -g myResourceGroup --network-plugin azure --enable-managed-identity -a ingress-appgw --appgw-name myApplicationGateway --appgw-subnet-cidr "10.2.0.0/16" --generate-ssh-keys
```

Информацию для настройки дополнительных параметров команды `az aks create` см. по [этим ссылкам](/cli/azure/aks#az-aks-create). 

> [!NOTE]
> Созданный вами кластер AKS отобразится в группе ресурсов *myResourceGroup*. Но автоматически созданный экземпляр Шлюза приложений будет находиться в группе ресурсов узла, в котором находятся пулы агентов. По умолчанию группа ресурсов узла имеет имя *MC_resource-group-name_cluster-name_location* (можно изменить). 

## <a name="deploy-a-sample-application-by-using-agic"></a>Развертывание примера приложения с помощью AGIC

Теперь вы можете развернуть пример приложения в созданном кластере AKS. Приложение использует надстройку AGIC для ingress и подключит экземпляр Шлюза приложений к кластеру AKS. 

Сначала получите учетные данные для кластера AKS с помощью команды `az aks get-credentials`: 

```azurecli-interactive
az aks get-credentials -n myCluster -g myResourceGroup
```

Теперь, когда у вас есть учетные данные, выполните указанную ниже команду, чтобы настроить пример приложения, использующего AGIC для ingress в кластере. AGIC обновит экземпляр Шлюза приложений, настроенный вами ранее с соответствующими правилами маршрутизации для нового развертываемого примера приложения.  

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml 
```

## <a name="check-that-the-application-is-reachable"></a>Проверка доступности приложения

Теперь, когда экземпляр Шлюза приложений настроен для обслуживания трафика в кластере AKS, давайте проверим доступность приложения. Сначала получите IP-адрес объекта ingress: 

```azurecli-interactive
kubectl get ingress
```

Убедитесь, что созданный вами пример приложения запускается одним из следующих способов:

- Используя IP-адрес экземпляра Шлюза приложений, полученного при выполнении предыдущей команды.
- При помощи `curl`. 

Для обновления Шлюзу приложений может потребоваться около минуты. Если Шлюз приложений на портале по-прежнему находится в состоянии **Обновление**, дождитесь окончания процедуры, прежде чем подключаться к IP-адресу. 

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите группу ресурсов, экземпляр Шлюза приложений и все связанные ресурсы, если они больше не нужны:

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения об отключении надстройки AGIC](./ingress-controller-disable-addon.md)

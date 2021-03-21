---
title: Настройка параметров маршрутизации для службы Azure Kubernetes с помощью Azure CLI
titlesuffix: Azure Virtual Network
description: Узнайте, как настроить кластер AKS с предпочтениями маршрутизации с помощью Azure CLI.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2021
ms.author: mnayak
ms.openlocfilehash: ac70f48a3c484f8865c54e09c59662a14a259e74
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101680342"
---
# <a name="configure-routing-preference-for-a-kubernetes-cluster-using-azure-cli"></a>Настройка параметров маршрутизации для кластера Kubernetes с помощью Azure CLI

В этой статье показано, как настроить параметры маршрутизации через сеть ISP (параметр **Интернета** ) для кластера Kubernetes с помощью Azure CLI. Параметры маршрутизации задаются путем создания общедоступного IP-адреса типа предпочтения маршрутизации * * Интернет * * * * и его последующего использования при создании кластера AKS.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется версия 2.0.49 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create). В следующем примере создается группа ресурсов в регионе Azure **Восточная часть США**.

```azurecli
  az group create --name myResourceGroup --location eastus
```
## <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса

Создайте общедоступный IP-адрес с предпочтением маршрутизации типа **Интернет** с помощью команды [AZ Network public-IP Create](/cli/azure/network/public-ip#az-network-public-ip-create).

Следующая команда создает новый общедоступный IP-адрес с параметром предпочтительного варианта маршрутизации **Интернет** в регионе Azure **Восточная часть США**.

```azurecli
az network public-ip create \
--name MyRoutingPrefIP \
--resource-group MyResourceGroup \
--location eastus \
--ip-tags 'RoutingPreference=Internet' \
--sku STANDARD \
--allocation-method static \
--version IPv4
```
> [!NOTE]
>  Сейчас при выборе предпочтительного варианта маршрутизации поддерживаются только общедоступные IP-адреса IPV4.

## <a name="get-the-id-of-public-ip-address"></a>Получение идентификатора общедоступного IP-адреса

Следующая команда возвращает идентификатор общедоступного IP-адреса, созданный в предыдущем разделе:
```azurecli
az network public-ip show \
--resource-group myResourceGroup \
--name myRoutingPrefIP \
--query id
```
## <a name="create-kubernetes-cluster-with-the-public-ip"></a>Создание кластера Kubernetes с общедоступным IP-адресом

Следующая команда создает кластер AKS с общедоступным IP-адресом, созданным в предыдущем разделе:

```azurecli
az aks create \
--resource-group MyResourceGroup \
--name MyAKSCluster \
--load-balancer-outbound-ips "Enter the public IP ID from previous step" --generate-ssh-key
```

>[!NOTE]
>Развертывание кластера AKS займет несколько минут.

Чтобы выполнить проверку, выполните поиск общедоступного IP-адреса, созданного на предыдущем шаге, в портал Azure. Вы увидите, что IP-адрес связан с подсистемой балансировки нагрузки, связанной с кластером Kubernetes, как показано ниже:

 ![Общедоступный IP-адрес предпочтения маршрутизации для Kubernetes](./media/routing-preference-azure-kubernetes-service-cli/routing-preference-azure-kubernetes-service.png)


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [предпочтительном варианте маршрутизации в общедоступных IP-адресах](routing-preference-overview.md). 
- [Настройка предпочтительного варианта маршрутизации для виртуальной машины с помощью Azure CLI.](configure-routing-preference-virtual-machine-cli.md)


---
title: Интерфейс командной строки. Развертывание частной конечной точки для веб-приложения с помощью Azure CLI
description: Узнайте, как с помощью Azure CLI развернуть частную конечную точку для веб-приложения
author: ericgre
ms.assetid: a56faf72-7237-41e7-85ce-da8346f2bcaa
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/06/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: acebc75b579b13ebb2cfad0e18057245781165ad
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102175283"
---
# <a name="create-an-app-service-app-and-deploy-private-endpoint-using-azure-cli"></a>Создание приложения Службы приложений и развертывание частной конечной точки с помощью Azure CLI

Этот пример скрипта создает в Службе приложений приложение со связанными ресурсами, а затем развертывает частную конечную точку.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим руководством требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Перед созданием любого ресурса необходимо создать группу ресурсов, которая будет содержать веб-приложение, виртуальную сеть и другие сетевые компоненты. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Этот пример создает группу ресурсов с именем *myResourceGroup* в расположении *francecentral*.

```azurecli-interactive
az group create --name myResourceGroup --location francecentral 
```

## <a name="create-an-app-service-plan"></a>Создание плана службы приложений

Вам нужно создать план Службы приложений для размещения веб-приложения.
Для этого выполните команду [az appservice plan create](/cli/azure/appservice/plan#az-appservice-plan-create).
Этот пример создает план плужбы приложений с именем *myAppServicePlan* в расположении *francecentral* с ценовой категорией *P1V2* и одним рабочим процессом. 

```azurecli-interactive
az appservice plan create \
--name myAppServicePlan \
--resource-group myResourceGroup \
--location francecentral \
--sku P1V2 \
--number-of-workers 1
```

## <a name="create-a-web-app"></a>Создание веб-приложения

Теперь, когда у вас есть план Службы приложений, вы можете развернуть веб-приложение.
Для создания веб-приложения выполните команду [az appservice plan create](/cli/azure/webapp#az-webapp-create.
Этот пример создает веб-приложение с именем *mySiteName* в плане с именем *myAppServicePlan*.

```azurecli-interactive
az webapp create \
--name mySiteName \
--resource-group myResourceGroup \
--plan myAppServicePlan
```

## <a name="create-a-vnet"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet). Этот пример создает виртуальную сеть по умолчанию с именем *myVNet* и одной подсетью с именем *mySubnet*.

```azurecli-interactive
az network vnet create \
--name myVNet \
--resource-group myResourceGroup \
--location francecentral \
--address-prefixes 10.8.0.0/16 \
--subnet-name mySubnet \
--subnet-prefixes 10.8.100.0/24
```

## <a name="configure-the-subnet"></a>Настройка подсети 

Вам нужно изменить настройки подсети, чтобы отключить сетевые политики для частной конечной точки. Обновите конфигурацию подсети *mySubnet* с помощью команды [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update).

```azurecli-interactive
az network vnet subnet update \
--name mySubnet \
--resource-group myResourceGroup \
--vnet-name myVNet \
--disable-private-endpoint-network-policies true
```

## <a name="create-the-private-endpoint"></a>Создание частной конечной точки

Создайте частную конечную точку для веб-приложения с помощью команды [az network private-endpoint create](/cli/azure/network/private-endpoint). Этот пример создает частную конечную точку с именем *myPrivateEndpoint* в виртуальной сети *myVNet* и подсети *mySubnet* с подключением *myConnectionName* к идентификатору ресурса веб-приложения /subscriptions/SubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/myWebApp, а также определяет параметр группы *sites*, который обозначает тип веб-приложения. 

```azurecli-interactive
az network private-endpoint create \
--name myPrivateEndpoint \
--resource-group myResourceGroup \
--vnet-name myVNet \
--subnet mySubnet \
--connection-name myConnectionName \
--private-connection-resource-id /subscriptions/SubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/myWebApp \
--group-id sites
```

## <a name="configure-the-private-zone"></a>Настройка частной зоны

Вам осталось создать частную зону DNS с именем *privatelink.azurewebsites.net*, связанную с виртуальной сетью, чтобы разрешать DNS-имя этого веб-приложения.


```azurecli-interactive
az network private-dns zone create \
--name privatelink.azurewebsites.net \
--resource-group myResourceGroup

az network private-dns link vnet create \
--name myDNSLink \
--resource-group myResourceGroup \
--registration-enabled false \
--virtual-network myVNet \
--zone-name privatelink.azurewebsites.net

az network private-endpoint dns-zone-group create \
--name myZoneGroup \
--resource-group myResourceGroup \
--endpoint-name myPrivateEndpoint \
--private-dns-zone privatelink.azurewebsites.net \
--zone-name privatelink.azurewebsites.net
```






[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
- Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по службе приложений Azure](../samples-cli.md).

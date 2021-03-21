---
title: Создание, изменение или удаление виртуальной сети TAP — Azure CLI
description: Узнайте, как создать, изменить или удалить TAP виртуальной сети с помощью Azure CLI.
services: virtual-network
documentationcenter: na
author: karthikananth
manager: ganesr
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/18/2018
ms.author: kaanan
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8e3a56e4a6eb1fb6eb633021178ef78f8ac7287d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96014794"
---
# <a name="work-with-a-virtual-network-tap-using-the-azure-cli"></a>Работа с TAP виртуальной сети с помощью Azure CLI

TAP (точка доступа к терминалу) виртуальной сети Azure позволяет непрерывно передавать сетевой трафик виртуальной машины в сборщик сетевых пакетов или средство аналитики. Средство сбора данных или анализа предоставляется партнером по [сетевым виртуальным устройствам](https://azure.microsoft.com/solutions/network-appliances/) . Список партнерских решений, работающих с TAP виртуальной сети, см. в разделе о [партнерских решениях](virtual-network-tap-overview.md#virtual-network-tap-partner-solutions). 

## <a name="create-a-virtual-network-tap-resource"></a>Создание ресурса TAP виртуальной сети

Ознакомьтесь с [предварительными требованиями](virtual-network-tap-overview.md#prerequisites) перед созданием ресурса TAP виртуальной сети. Вы можете выполнить приведенные ниже команды в [Azure Cloud Shell](https://shell.azure.com/bash) или Azure CLI на своем компьютере. Azure Cloud Shell — это бесплатная интерактивная оболочка, которая не требует установки Azure CLI на компьютере. Вам потребуется войти в Azure с учетной записью, имеющей соответствующие [разрешения](virtual-network-tap-overview.md#permissions). Для этой статьи требуется Azure CLI 2.0.46 или более поздней версии. Выполните командлет `az --version`, чтобы узнать установленную версию. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli). В настоящее время в качестве расширения можно выбрать виртуальную сеть. Для установки расширения необходимо запустить `az extension add -n virtual-network-tap` . Если Azure CLI запущен локально, необходимо также выполнить командлет `az login`, чтобы создать подключение к Azure.

1. Извлеките идентификатор подписки в переменную, которая используется позднее:

   ```azurecli-interactive
   subscriptionId=$(az account show \
   --query id \
   --out tsv)
   ```

2. Задайте идентификатор подписки, который будет использоваться для создания ресурса TAP виртуальной сети.

   ```azurecli-interactive
   az account set --subscription $subscriptionId
   ```

3. Повторно зарегистрируйте идентификатор подписки, который будет использоваться для создания ресурса TAP виртуальной сети. Если при создании ресурса TAP возникнет ошибка регистрации, выполните следующую команду:

   ```azurecli-interactive
   az provider register --namespace Microsoft.Network --subscription $subscriptionId
   ```

4. Если целевым назначением TAP виртуальной сети является сетевой интерфейс на сетевом виртуальном устройстве для сборщика или средства аналитики:

   - Извлеките IP-конфигурацию сетевого интерфейса сетевого виртуального устройства в переменную, которая используется позднее. Идентификатор является конечной точкой, которая будет агрегировать трафик TAP. В следующем примере извлекается идентификатор IP-конфигурации *ipconfig1* для сетевого интерфейса с именем *myNetworkInterface* в группе ресурсов с именем *myResourceGroup*:

      ```azurecli-interactive
       IpConfigId=$(az network nic ip-config show \
       --name ipconfig1 \
       --nic-name myNetworkInterface \
       --resource-group myResourceGroup \
       --query id \
       --out tsv)
      ```

   - Создайте TAP виртуальной сети в регионе Azure westcentralus, используя идентификатор IP-конфигурации в качестве целевого назначения и необязательное свойство port. Свойство port указывает порт назначения для IP-конфигурации сетевого интерфейса, по которому будет приниматься трафик TAP:  

      ```azurecli-interactive
       az network vnet tap create \
       --resource-group myResourceGroup \
       --name myTap \
       --destination $IpConfigId \
       --port 4789 \
       --location westcentralus
      ```

5. Если целевым назначением для TAP виртуальной сети является внутренняя подсистема балансировки нагрузки Azure:
  
   - Извлеките IP-конфигурацию внешнего интерфейса внутренней подсистемы балансировки нагрузки Azure в переменную, которая используется позднее. Идентификатор является конечной точкой, которая будет агрегировать трафик TAP. В следующем примере извлекается идентификатор IP-конфигурации внешнего интерфейса *frontendipconfig1* для подсистемы балансировки нагрузки с именем *myInternalLoadBalancer* в группе ресурсов с именем *myResourceGroup*:

      ```azurecli-interactive
      FrontendIpConfigId=$(az network lb frontend-ip show \
      --name frontendipconfig1 \
      --lb-name myInternalLoadBalancer \
      --resource-group myResourceGroup \
      --query id \
      --out tsv)
      ```

   - Создайте TAP виртуальной сети, используя идентификатор IP-конфигурации внешнего интерфейса в качестве целевого назначения и необязательное свойство port. Свойство port указывает порт назначения для IP-конфигурации внешнего интерфейса, по которому будет приниматься трафик TAP:  

      ```azurecli-interactive
      az network vnet tap create \
      --resource-group myResourceGroup \
      --name myTap \
      --destination $FrontendIpConfigId \
      --port 4789 \
     --location westcentralus
     ```

6. Подтвердите создание TAP виртуальной сети:

   ```azurecli-interactive
   az network vnet tap show \
   --resource-group myResourceGroup
   --name myTap
   ```

## <a name="add-a-tap-configuration-to-a-network-interface"></a>Добавление конфигурации TAP для сетевого интерфейса

1. Извлеките идентификатор существующего ресурса TAP виртуальной сети. В следующем примере извлекается TAP виртуальной сети с именем *myTap* в группе ресурсов *myResourceGroup*:

   ```azurecli-interactive
   tapId=$(az network vnet tap show \
   --name myTap \
   --resource-group myResourceGroup \
   --query id \
   --out tsv)
   ```

2. Создайте конфигурацию TAP для сетевого интерфейса отслеживаемой виртуальной машины. В следующем примере создается конфигурация TAP для сетевого интерфейса с именем *myNetworkInterface*:

   ```azurecli-interactive
   az network nic vtap-config create \
   --resource-group myResourceGroup \
   --nic myNetworkInterface \
   --vnet-tap $tapId \
   --name mytapconfig \
   --subscription subscriptionId
   ```

3. Подтвердите создание конфигурации TAP:

   ```azurecli-interactive
   az network nic vtap-config show \
   --resource-group myResourceGroup \
   --nic-name myNetworkInterface \
   --name mytapconfig \
   --subscription subscriptionId
   ```

## <a name="delete-the-tap-configuration-on-a-network-interface"></a>Удаление конфигурации TAP для сетевого интерфейса

   ```azurecli-interactive
   az network nic vtap-config delete \
   --resource-group myResourceGroup \
   --nic myNetworkInterface \
   --name myTapConfig \
   --subscription subscriptionId
   ```

## <a name="list-virtual-network-taps-in-a-subscription"></a>Перечисление TAP виртуальных сетей в подписке

   ```azurecli-interactive
   az network vnet tap list
   ```

## <a name="delete-a-virtual-network-tap-in-a-resource-group"></a>Удаление TAP виртуальной сети в группе ресурсов

   ```azurecli-interactive
   az network vnet tap delete \
   --resource-group myResourceGroup \
   --name myTap
   ```

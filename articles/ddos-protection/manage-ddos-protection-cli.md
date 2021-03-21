---
title: Создание и Настройка плана защиты Azure от атак DDoS с помощью Azure CLI
description: Узнайте, как создать план защиты от атак DDoS с помощью Azure CLI
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: yitoh
ms.openlocfilehash: 59c5ca9ce9e95319b36e002da0b5d1438ef3fdd1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102203782"
---
# <a name="quickstart-create-and-configure-azure-ddos-protection-standard-using-azure-cli"></a>Краткое руководство. Создание и Настройка стандарта защиты от атак DDoS Azure с помощью Azure CLI

Приступая к работе со стандартом защиты Azure от атак DDoS с помощью Azure CLI. 

План защиты от атак DDoS определяет набор виртуальных сетей, в которых включен стандарт защиты от атак DDoS, для подписок. Вы можете настроить один план защиты от атак DDoS для своей организации и привязать к нему виртуальные сети из нескольких подписок. 

В этом кратком руководстве вы создадите план защиты от атак DDoS и свяжете его с виртуальной сетью. 

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure CLI или Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать CLI локально, для выполнения инструкций из этого руководства вам потребуется Azure CLI версии 2.0.28 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-ddos-protection-plan"></a>Создание плана защиты от атак DDoS

В Azure выделите связанные ресурсы группе ресурсов. Вы можете выбрать существующую группу ресурсов или создать новую.

Для создания группы ресурсов используйте команду [az group create](/cli/azure/group#az-group-create). В этом примере мы назовем нашу группу ресурсов _MyResourceGroup_ и используем расположение _восточной части США_ :

```azurecli-interactive
az group create \
    --name MyResourceGroup \
    --location eastus
```

Теперь создайте план защиты от атак DDoS с именем _миддоспротектионплан_:

```azurecli-interactive
az network ddos-protection create \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
```

## <a name="enable-ddos-protection-for-a-virtual-network"></a>Включение защиты от атак DDoS для виртуальной сети

### <a name="enable-ddos-protection-for-a-new-virtual-network"></a>Включение защиты от атак DDoS для новой виртуальной сети

Вы можете включить защиту от атак DDoS при создании виртуальной сети. В этом примере мы назовем нашу виртуальную сеть _MyVnet_: 

```azurecli-interactive
az network vnet create \
    --resource-group MyResourceGroup \
    --name MyVnet \
    --location eastus \
    --ddos-protection true
    --ddos-protection-plan MyDdosProtectionPlan
```

Невозможно переместить виртуальную сеть в другую группу ресурсов или подписку, если для этой сети включена защита от атак DDoS уровня "Стандартный". Если необходимо перенести такую виртуальную сеть, сначала отключите защиту от атак DDoS уровня "Стандартный", переместите виртуальную сеть и затем включите защиту от атак DDoS уровня "Стандартный". После перемещения сбрасываются автоматически настраиваемые пороговые значения политик для всех защищенных общедоступных IP-адресов в виртуальной сети.

### <a name="enable-ddos-protection-for-an-existing-virtual-network"></a>Включение защиты от атак DDoS для существующей виртуальной сети

При [создании плана защиты от атак DDoS](#create-a-ddos-protection-plan)можно связать с планом одну или несколько виртуальных сетей. Чтобы добавить несколько виртуальных сетей, просто перечислите имена или идентификаторы, разделенные пробелами. В этом примере мы добавим _MyVnet_:

```azurecli-interactive
az group create \
    --name MyResourceGroup \
    --location eastus

az network ddos-protection create \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
    --vnets MyVnet
```

Кроме того, можно включить защиту от атак DDoS для заданной виртуальной сети.

```azurecli-interactive
az network vnet update \
    --resource-group MyResourceGroup \
    --name MyVnet \
    --ddos-protection true
    --ddos-protection-plan MyDdosProtectionPlan
```

## <a name="validate-and-test"></a>Проверка и тестирование

Сначала проверьте сведения о плане защиты от атак DDoS:

```azurecli-interactive
az network ddos-protection show \
    --resource-group MyResourceGroup \
    --name MyDdosProtectionPlan
```

Убедитесь, что команда возвращает правильные сведения о плане защиты от атак DDoS.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете разместить ресурсы для следующего руководства. Удалите группу ресурсов _MyResourceGroup_ , если она больше не нужна. При удалении группы ресурсов также удаляется план защиты от атак DDoS и все связанные с ним ресурсы. 

Чтобы удалить группу ресурсов, выполните команду [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
az group delete \
--name MyResourceGroup 
```

Обновите заданную виртуальную сеть, чтобы отключить защиту от атак DDoS:

```azurecli-interactive
az network vnet update \
    --resource-group MyResourceGroup \
    --name MyVnet \
    --ddos-protection false
    --ddos-protection-plan ""
```

Если вы хотите удалить план защиты от атак DDoS, сначала необходимо отменить связь со всеми виртуальными сетями. 

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как просмотреть и настроить данные телеметрии для плана защиты от атак DDoS, перейдите к следующим руководствам.

> [!div class="nextstepaction"]
> [Настройка телеметрии защиты от атак DDoS и просмотр данных телеметрии](telemetry.md)

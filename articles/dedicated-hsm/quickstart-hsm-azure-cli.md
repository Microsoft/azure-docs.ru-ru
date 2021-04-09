---
title: Краткое руководство. Создание Выделенного устройства HSM Azure с помощью Azure CLI
description: Создание, отображение, перечисление, обновление и удаление Выделенных устройств HSM Azure с помощью Azure CLI.
services: dedicated-hsm
author: msmbaldwin
ms.author: mbaldwin
ms.topic: quickstart
ms.service: key-vault
ms.devlang: azurecli
ms.date: 01/06/2021
ms.custom: devx-track-azurecli
ms.openlocfilehash: e07bc758b1ef86b3d8c605cbce72f6db564a355f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98020863"
---
# <a name="quickstart-create-an-azure-dedicated-hsm-by-using-the-azure-cli"></a>Краткое руководство. Создание Выделенного устройства HSM Azure с использованием Azure CLI

В этой статье описано создание Выделенного устройства HSM Azure и управление им с помощью расширения Azure CLI [az dedicated-hsm](/cli/azure/ext/hardware-security-modules/dedicated-hsm).

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Вы можете создать [бесплатную учетную запись](https://azure.microsoft.com/free/), если у вас еще ее нет.
  
  Если у вас несколько подписок Azure, настройте подписку, которая будет использоваться для выставления счетов, с помощью команды Azure CLI [az account set](/cli/azure/account#az_account_set).
  
  ```azurecli-interactive
  az account set --subscription 00000000-0000-0000-0000-000000000000
  ```
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]  
  
- Выполнение всех требований для выделенного устройства HSM, включая регистрацию и утверждение, а также обеспечение наличия виртуальной сети и виртуальной машины, которые будут использоваться для подготовки. Дополнительные сведения о требованиях для выделенного устройства HSM и предварительных требованиях см. в статье [Руководство. Развертывание устройств HSM в существующей виртуальной сети с помощью Azure CLI](tutorial-deploy-hsm-cli.md).
  

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[Группа ресурсов Azure](../azure-resource-manager/management/overview.md) — это логический контейнер для развертывания ресурсов Azure и управления ими как группой. Если у вас еще нет группы ресурсов для выделенного устройства HSM, создайте ее с помощью команды [az group create](/cli/azure/group#az_group_create). В следующем примере создается группа ресурсов с именем `myRG` в регионе Azure `westus`:

```azurecli-interactive
az group create --name myRG --location westus
```

## <a name="create-a-dedicated-hsm"></a>Создание выделенного устройства HSM

Чтобы создать выделенное устройство HSM, используйте команду [az dedicated-hsm create](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_create). В приведенном ниже примере подготавливается выделенное устройство HSM с именем `hsm1` в регионе `westus`, группа ресурсов `myRG`, а также указанная подписка, виртуальная сеть и подсеть. Обязательные параметры: `name`, `location` и `resource group`.

```azurecli-interactive
az dedicated-hsm create \
   --resource-group myRG \
   --name "hsm1" \
   --location "westus" \
   --network-profile-network-interfaces private-ip-address="1.0.0.1" \
   --subnet id="/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/hsm-group/providers/Microsoft.Network/virtualNetworks/MyHSM-vnet/subnets/MyHSM-vnet" \
   --stamp-id "stamp1" \
   --sku name="SafeNet Luna Network HSM A790" \
   --tags resourceType="hsm" Environment="test" \
   --zones "AZ1"
```

Развертывание занимает примерно 25–30 минут.

## <a name="get-a-dedicated-hsm"></a>Получение сведений о выделенном устройстве HSM

Чтобы получить сведения о текущем выделенном устройстве HSM, выполните команду [az dedicated-hsm show](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_show). В следующем примере извлекаются сведения о выделенном устройстве HSM `hsm1`, входящем в группу ресурсов `myRG`.

```azurecli-interactive
az dedicated-hsm show --resource-group myRG --name hsm1
```

## <a name="update-a-dedicated-hsm"></a>Обновление выделенного устройства HSM

Команда [az dedicated-hsm update](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_update) позволяет обновить выделенное устройство HSM. В следующем примере обновляется выделенное устройство HSM `hsm1`, входящее в группу ресурсов `myRG`, и его теги:

```azurecli-interactive
az dedicated-hsm update --resource-group myRG –-name hsm1 --tags resourceType="hsm" Environment="prod" Slice="A"
```

## <a name="list-dedicated-hsms"></a>Вывод списка выделенных устройств HSM

Чтобы получить сведения о текущих выделенных устройствах HSM, выполните команду [az dedicated-hsm list](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_list). В следующем примере перечисляются выделенные устройства HSM, входящие в группу ресурсов `myRG`:

```azurecli-interactive
az dedicated-hsm list --resource-group myRG
```

## <a name="remove-a-dedicated-hsm"></a>Удаление выделенного устройства HSM

Чтобы удалить выделенное устройство HSM, используйте команду [az dedicated-hsm delete](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_delete). В следующем примере выделенное устройство HSM `hsm1` удаляется из группы ресурсов `myRG`:

```azurecli-interactive
az dedicated-hsm delete --resource-group myRG –-name hsm1
```

## <a name="delete-the-resource-group"></a>Удаление группы ресурсов

Если группа ресурсов, созданная для выделенного устройства HSM, больше не нужна, ее можно удалить. Для этого выполните команду [az group delete](/cli/azure/group#az_group_delete). При выполнении этой команды удаляется группа и все содержащиеся в ней ресурсы, включая те, которые не связаны с выделенным устройством HSM. В следующем примере удаляется группа ресурсов `myRG` и все содержащиеся в ней ресурсы:

```azurecli-interactive
az group delete --name myRG
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье о [Выделенном устройстве HSM Azure](overview.md).

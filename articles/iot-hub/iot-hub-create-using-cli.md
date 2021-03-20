---
title: Создание Центра Интернета вещей с помощью Azure CLI | Документация Майкрософт
description: Узнайте, как использовать команды Azure CLI, чтобы создать группу ресурсов, а затем создать центр Интернета вещей в группе ресурсов. Также Узнайте, как удалить концентратор.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/23/2018
ms.author: robinsh
ms.openlocfilehash: e9757b94330f889869080a0e961ab2728cee86f6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94659936"
---
# <a name="create-an-iot-hub-using-the-azure-cli"></a>Создание Центра Интернета вещей с помощью Azure CLI

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

В этой статье показано, как создать Центр Интернета вещей с помощью Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="create-an-iot-hub"></a>Создание центра Интернета вещей

Создайте группу ресурсов и добавьте Центр Интернета вещей с помощью Azure CLI.

1. Создайте Центр Интернета вещей в группе ресурсов. Используйте имеющуюся группу ресурсов либо выполните следующую [команду для создания группы ресурсов](/cli/azure/resource):
    
   ```azurecli-interactive
   az group create --name {your resource group name} --location westus
   ```

   > [!TIP]
   > В предыдущем примере создается группа ресурсов в расположении западной части США. Список доступных расположений можно просмотреть, выполнив эту команду: 
   >
   > ```azurecli-interactive
   > az account list-locations -o table
   > ```
   >

2. Выполните в своей группе ресурсов следующую [команду для создания Центра Интернета вещей](/cli/azure/iot/hub#az-iot-hub-create), указав глобально уникальное имя своего Центра Интернета вещей:
    
   ```azurecli-interactive
   az iot hub create --name {your iot hub name} \
      --resource-group {your resource group name} --sku S1
   ```

   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]


Предыдущая команда создает Центр Интернета вещей в оплачиваемой ценовой категории S1. Дополнительные сведения см. на странице с [ценами на Центр Интернета вещей Azure](https://azure.microsoft.com/pricing/details/iot-hub/).

## <a name="remove-an-iot-hub"></a>Удаление Центра Интернета вещей

Вы можете использовать Azure CLI, чтобы [удалить отдельный ресурс](/cli/azure/resource), например Центр Интернета вещей, или группу ресурсов со всеми ресурсами, включая любые Центры Интернета вещей.

Чтобы [удалить центр Интернета вещей](/cli/azure/iot/hub#az-iot-hub-delete), выполните следующую команду:

```azurecli-interactive
az iot hub delete --name {your iot hub name} -\
  -resource-group {your resource group name}
```

Чтобы [Удалить группу ресурсов](/cli/azure/group#az-group-delete) и все ее ресурсы, выполните следующую команду:

```azurecli-interactive
az group delete --name {your resource group name}
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании Центра Интернета вещей см. в следующих статьях:

* [Руководство разработчика для Центра Интернета вещей](iot-hub-devguide.md)
* [Создание Центра Интернета вещей с помощью портала Azure](iot-hub-create-through-portal.md)
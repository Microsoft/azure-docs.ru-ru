---
title: Подключение существующей учетной записи Azure Cosmos к конечным точкам службы виртуальной сети
description: Подключение существующей учетной записи Azure Cosmos к конечным точкам службы виртуальной сети
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: f66d219eff1919e62088e5c4f7aa72aab97ea4f9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94566237"
---
# <a name="connect-an-existing-azure-cosmos-account-with-virtual-network-service-endpoints-using-azure-cli"></a>Подключение существующей учетной записи Azure Cosmos к конечным точкам службы виртуальной сети с помощью Azure CLI
[!INCLUDE[appliesto-all-apis](../../../includes/appliesto-all-apis.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.9.1 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="sample-script"></a>Пример скрипта

В этом примере показано, как подключить существующую учетную запись Azure Cosmos к существующей новой виртуальной сети, в которой подсеть еще не настроена для конечных точек службы, с помощью параметра `ignore-missing-vnet-service-endpoint`. Это позволяет без ошибок завершить настройку учетной записи Cosmos до завершения настройки подсети виртуальной сети. После завершения настройки подсети учетная запись Cosmos будет доступна через настроенную подсеть.

> [!NOTE]
> В этом примере демонстрируется использование учетной записи API SQL (Core). Чтобы использовать этот пример для других API, примените параметры `enable-virtual-network` и `virtual-network-rules`, приведенные в скрипте ниже, в своем скрипте для конкретного API.

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/common/service-endpoints-ignore-missing-vnet.sh "Create an Azure Cosmos account with service endpoints.")]

## <a name="clean-up-deployment"></a>Очистка развертывания

После выполнения примера сценария можно удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```azurecli-interactive
az group delete --name $resourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) | Создает виртуальную сеть Azure. |
| [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) | Создает подсеть для виртуальной сети Azure. |
| [az network vnet subnet show](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-show) | Возвращает подсеть для виртуальной сети Azure. |
| [az cosmosdb create](/cli/azure/cosmosdb#az-cosmosdb-create) | Создает учетную запись Azure Cosmos DB. |
| [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) | Обновляет подсеть для виртуальной сети Azure. |
| [az group delete](/cli/azure/resource#az-resource-delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure Cosmos DB CLI см. в [этой документации](/cli/azure/cosmosdb).

Все примеры сценариев Azure Cosmos DB CLI можно найти в [этом репозитории на сайте GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
